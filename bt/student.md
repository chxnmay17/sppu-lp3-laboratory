# Solidity contract + deployment & gas-observation guide

Below you’ll find a compact, well-commented Solidity contract that stores **Student** records using `struct`, `array`, `mapping` (for fast lookup), and includes both `receive()` and `fallback()` to satisfy your *Fallback* requirement. After the contract there are **two ways** to deploy & observe transaction fee / gas values:

1. Quick — **Remix + MetaMask** (visual, easiest).
2. Reproducible — **Hardhat script** that deploys + calls functions and prints `gasUsed`, `effectiveGasPrice`, and the transaction fee in ETH.

---

# 1) Solidity: `StudentRegistry.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/// @title StudentRegistry
/// @notice Store simple student data using struct + array + mapping. Includes receive() and fallback().
contract StudentRegistry {
    // Student structure
    struct Student {
        uint256 id;
        string name;
        uint8 age;
        string grade;
        bool exists;
    }

    // Dynamic array of student IDs (preserves insertion order)
    uint256[] public studentIds;

    // Mapping from ID -> Student struct
    mapping(uint256 => Student) private students;

    // Mapping from ID -> index in studentIds array (plus 1). Zero means not present.
    mapping(uint256 => uint256) private idToIndexPlusOne;

    // Owner for simple access control
    address public owner;

    // Last call data for fallback demonstration
    bytes public lastCallData;
    address public lastCaller;

    // Events
    event StudentAdded(uint256 indexed id, string name);
    event StudentUpdated(uint256 indexed id, string name);
    event StudentRemoved(uint256 indexed id);
    event Received(address indexed sender, uint256 amount);
    event FallbackCalled(address indexed sender, uint256 value, bytes data);

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    /// @notice Add a new student. Reverts if id already exists.
    function addStudent(
        uint256 id,
        string calldata name,
        uint8 age,
        string calldata grade
    ) external onlyOwner {
        require(id != 0, "id cannot be zero");
        require(!students[id].exists, "Student exists");

        students[id] = Student({id: id, name: name, age: age, grade: grade, exists: true});
        studentIds.push(id);
        idToIndexPlusOne[id] = studentIds.length; // store index+1

        emit StudentAdded(id, name);
    }

    /// @notice Update existing student fields (only owner).
    function updateStudent(
        uint256 id,
        string calldata name,
        uint8 age,
        string calldata grade
    ) external onlyOwner {
        require(students[id].exists, "Student not found");
        students[id].name = name;
        students[id].age = age;
        students[id].grade = grade;

        emit StudentUpdated(id, name);
    }

    /// @notice Remove a student by id (only owner). Keeps array compact by swapping.
    function removeStudent(uint256 id) external onlyOwner {
        require(students[id].exists, "Student not found");

        // find index (stored as +1)
        uint256 idxPlusOne = idToIndexPlusOne[id];
        uint256 idx = idxPlusOne - 1;
        uint256 lastIndex = studentIds.length - 1;

        if (idx != lastIndex) {
            uint256 lastId = studentIds[lastIndex];
            // swap in array
            studentIds[idx] = lastId;
            idToIndexPlusOne[lastId] = idx + 1;
        }
        // pop last
        studentIds.pop();

        // delete mappings
        delete idToIndexPlusOne[id];
        delete students[id];

        emit StudentRemoved(id);
    }

    /// @notice Return student details for an id.
    function getStudent(uint256 id) external view returns (uint256, string memory, uint8, string memory, bool) {
        Student storage s = students[id];
        return (s.id, s.name, s.age, s.grade, s.exists);
    }

    /// @notice Return all student IDs (useful to iterate off-chain)
    function getAllStudentIds() external view returns (uint256[] memory) {
        return studentIds;
    }

    /// @notice Return number of students stored.
    function totalStudents() external view returns (uint256) {
        return studentIds.length;
    }

    /* ------------------ Fallback & Receive ------------------ */

    /// @notice Accept plain ETH transfers to the contract.
    receive() external payable {
        // store minimal info & emit event
        lastCaller = msg.sender;
        lastCallData = "";
        emit Received(msg.sender, msg.value);
    }

    /// @notice Called when no other function matches, or when calldata is non-empty but no function matches.
    fallback() external payable {
        lastCaller = msg.sender;
        lastCallData = msg.data;
        emit FallbackCalled(msg.sender, msg.value, msg.data);
    }

    /* ------------------ Misc ------------------ */

    /// @notice Owner can withdraw ETH from contract.
    function withdrawETH(address payable to, uint256 amount) external onlyOwner {
        require(address(this).balance >= amount, "Insufficient balance");
        (bool ok, ) = to.call{value: amount}("");
        require(ok, "Transfer failed");
    }

    /// @notice Read last fallback calldata and caller
    function getLastCallInfo() external view returns (address, bytes memory) {
        return (lastCaller, lastCallData);
    }
}
```

---

# 2) How to deploy & observe gas / fee (two methods)

## A — Using **Remix + MetaMask** (visual)

1. Open [https://remix.ethereum.org](https://remix.ethereum.org).
2. Create `StudentRegistry.sol` and paste the contract above.
3. Compile with Solidity `^0.8.20`.
4. In *Deploy & Run Transactions* panel:

   * Environment: **Injected Provider — MetaMask** (or JS VM for local testing).
   * Connect MetaMask to a testnet (e.g., **Sepolia**). (Goerli is deprecated.)
5. Deploy the contract (confirm in MetaMask). After the transaction is mined:

   * Click the deployed contract in Remix's *Deployed Contracts* list.
   * Run `addStudent(...)` (owner only — use the account that deployed) — provide arguments and click **transact**. MetaMask will prompt to sign and then show the transaction in the wallet.
6. In Remix transaction log (or in your wallet / block explorer like Etherscan for Sepolia), click the transaction details. You will see:

   * **Gas Used** — the actual gas consumed by the transaction.
   * **Gas Limit** — the maximum gas you allowed.
   * **Effective Gas Price** — the gas price (wei per gas unit) that was used.
   * **Transaction Fee** = `Gas Used × Effective Gas Price`. Some UIs already display the fee in ETH.

Remix and block explorers show these values directly (gas used, gas price, fee). If you want the fee in ETH: convert from wei using `wei / 1e18`.

---

## B — Using **Hardhat** (scripted; prints the gasUsed and fee)

### 1) Minimal project setup

(You likely already have Node.js.)

```bash
mkdir student-registry
cd student-registry
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox ethers
npx hardhat # choose "Create a basic sample project"
```

Replace `contracts/Lock.sol` with `contracts/StudentRegistry.sol` (paste the Solidity code above).

### 2) Example script `scripts/deploy-and-measure.js`

Create `scripts/deploy-and-measure.js`:

```js
const hre = require("hardhat");
const { ethers } = hre;

async function main() {
  const [deployer, other] = await ethers.getSigners();
  console.log("Deploying with account:", deployer.address);

  const Factory = await ethers.getContractFactory("StudentRegistry");
  const contract = await Factory.deploy();
  const deployReceipt = await contract.deployTransaction.wait();
  console.log("Deployed at:", contract.address);
  console.log("Deploy transaction gasUsed:", deployReceipt.gasUsed.toString());
  console.log("Deploy effectiveGasPrice (wei):", deployReceipt.effectiveGasPrice.toString());
  const deployFeeWei = deployReceipt.gasUsed.mul(deployReceipt.effectiveGasPrice);
  console.log("Deploy fee (ETH):", ethers.utils.formatEther(deployFeeWei));

  // Example: call addStudent (owner-only) and measure gas
  const tx = await contract.addStudent(1, "Alice", 20, "A+");
  const receipt = await tx.wait();
  console.log("addStudent gasUsed:", receipt.gasUsed.toString());
  console.log("addStudent effectiveGasPrice (wei):", receipt.effectiveGasPrice.toString());
  const feeWei = receipt.gasUsed.mul(receipt.effectiveGasPrice);
  console.log("addStudent fee (ETH):", ethers.utils.formatEther(feeWei));

  // Make another transaction that hits fallback (send data that doesn't match any function)
  const rawTx = {
    to: contract.address,
    data: "0x12345678", // random data to trigger fallback
    value: ethers.utils.parseEther("0.001"),
  };
  const tx2 = await deployer.sendTransaction(rawTx);
  const r2 = await tx2.wait();
  console.log("fallback tx gasUsed:", r2.gasUsed.toString());
  console.log("fallback tx fee (ETH):", ethers.utils.formatEther(r2.gasUsed.mul(r2.effectiveGasPrice)));
}

main()
  .then(() => process.exit(0))
  .catch((err) => {
    console.error(err);
    process.exit(1);
  });
```

### 3) Run the script

* For a local Hardhat node (fast & free):
  `npx hardhat node` (in one terminal)
  In another terminal:
  `npx hardhat run scripts/deploy-and-measure.js --network localhost`

* For a testnet (e.g., Sepolia):

  * Add provider + private key to `hardhat.config.js` (use Infura/Alchemy endpoint and a test wallet with Sepolia funds).
  * Then run:
    `npx hardhat run scripts/deploy-and-measure.js --network sepolia`

The script prints `gasUsed`, `effectiveGasPrice` (wei), and the transaction fee in ETH. The formula is:

```
transactionFeeWei = receipt.gasUsed * receipt.effectiveGasPrice
transactionFeeEth = ethers.utils.formatEther(transactionFeeWei)
```

---

# 3) What to look for when observing gas & fee

* **Gas Used**: actual units consumed by the EVM for that transaction. Dependent on complexity (storage writes are expensive; `SSTORE` costs a lot).
* **Gas Limit**: the maximum gas the sender authorized for the tx (not usually equal to gas used).
* **Effective Gas Price** (wei): the price per gas unit paid. On modern EIP-1559 chains you’ll see `baseFee` + `tip`, and explorers often display `effectiveGasPrice`.
* **Fee (ETH)** = `GasUsed * EffectiveGasPrice` (convert wei→ETH).
* **Storage writes** (first-time `SSTORE`) are the most expensive ops — adding a `Student` that modifies storage will consume substantially more gas than a `view` call.
* **Deploy gas** is typically much larger than simple calls because the entire contract bytecode is stored.

---

# 4) Why this contract is good for measuring gas

* `addStudent` performs new storage writes (SSTORE) — good to measure cost of creating records.
* `updateStudent` modifies existing storage (cheaper than first write).
* `removeStudent` involves array manipulation and `SSTORE` deletes — also interesting to measure.
* `receive()` and `fallback()` let you measure the cost of plain ETH transfers and fallback behavior.
* Deploy demonstrates contract-size & constructor costs.

---

# 5) Example (expected) gas behaviour — rough idea (will vary by chain & time)

* Contract deploy gasUsed: typically **hundreds of thousands** to **a few million** gas depending on contract size.
* `addStudent` (one SSTORE, array push) — often around **~80k–150k gas** (varies).
* `updateStudent` (modify existing storage) — lower, maybe **~30k–50k gas**.
* `receive()` when sent small ETH with no storage changes — typically **~21k–50k gas** (base transaction cost + any event logs).
* These are approximate — measure on your target testnet with the steps above to get exact numbers.

---

