# Smart contract — simple Bank account (Solidity)

Below is a simple, safe, well-commented Solidity contract that represents a single customer **bank account**. It allows:

* Depositing ETH (anyone can deposit to the account),
* Owner-only withdrawing,
* Reading the account balance,
* Emits events for deposits & withdrawals,
* Has a `receive()` so it can accept plain transfers.

You can deploy this to any Ethereum test network (Goerli, Sepolia, etc.) using Remix + MetaMask or Hardhat/Foundry.

---

# Contract code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/// @title SimpleBankAccount
/// @notice A minimal bank-style account for a single customer (owner).
///         - deposit: anyone can send ETH to the account
///         - withdraw: only owner can withdraw to any address (default owner)
///         - getBalance: anyone can read the account balance
contract SimpleBankAccount {
    address public owner;

    // Events
    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(address indexed to, uint256 amount);

    /// @notice Set contract deployer as owner (the customer).
    constructor(address _owner) {
        // allow specifying owner on deploy; if zero address passed, use deployer
        if (_owner == address(0)) {
            owner = msg.sender;
        } else {
            owner = _owner;
        }
    }

    /// @notice Modifier to restrict functions to the owner (customer).
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner allowed");
        _;
    }

    /// @notice Deposit funds to this bank account.
    /// @dev This is payable so ETH can be sent. Emits Deposited event.
    function deposit() external payable {
        require(msg.value > 0, "Send some ETH to deposit");
        emit Deposited(msg.sender, msg.value);
    }

    /// @notice Withdraw a specified amount (in wei) to an address.
    /// @param to Recipient address (can be owner or any other address).
    /// @param amount Amount in wei to withdraw.
    function withdraw(address payable to, uint256 amount) external onlyOwner {
        require(amount > 0, "Amount must be > 0");
        uint256 current = address(this).balance;
        require(current >= amount, "Insufficient balance in account");

        // Use call to forward gas and handle reentrancy safely.
        // (This function has no state changes after external call, so it's safe as-is,
        //  but if you add state later consider reentrancy guard.)
        (bool success, ) = to.call{value: amount}("");
        require(success, "Transfer failed");

        emit Withdrawn(to, amount);
    }

    /// @notice Get the current balance of the account (in wei).
    /// @return balance Account balance in wei.
    function getBalance() external view returns (uint256 balance) {
        return address(this).balance;
    }

    /// @notice Allow contract to receive ETH via plain transfer.
    receive() external payable {
        require(msg.value > 0, "Zero-value transfer not allowed");
        emit Deposited(msg.sender, msg.value);
    }

    /// @notice Convenience: change owner (customer) to a new address.
    /// @param newOwner Address of the new owner.
    function changeOwner(address newOwner) external onlyOwner {
        require(newOwner != address(0), "Zero address not allowed");
        owner = newOwner;
    }
}
```

---

# How it works — short explanation

* `constructor(address _owner)`: set the customer. If you want the deployer to be owner, pass `0x000...0` (or omit constructor parameter in Remix and set to deployer).
* `deposit()` / `receive()`: add ETH to the contract balance. `deposit()` is explicit, `receive()` handles direct transfers.
* `withdraw(to, amount)`: only `owner` can withdraw. You can withdraw to any address (owner's wallet or another).
* `getBalance()`: returns current contract balance in **wei**. Convert to ETH when showing to users (1 ETH = 10^18 wei).
* `changeOwner()`: optional: transfer account ownership to another address.

---

# Deploying on a test network (Quick steps using Remix + MetaMask)

1. Open [https://remix.ethereum.org](https://remix.ethereum.org).
2. Create a new file `SimpleBankAccount.sol` and paste the contract above.
3. Compile with Solidity `^0.8.20` (select compiler version matching `pragma`).
4. In the **Deploy & Run Transactions** panel:

   * Environment: `Injected Web3` (connects to MetaMask).
   * Make sure MetaMask is connected to a testnet (Sepolia / Goerli if available on your provider).
5. Deployment args: you can provide the owner address or `0x0000000000000000000000000000000000000000` to set deployer as owner.
6. Click `Deploy` and confirm the transaction in MetaMask.
7. After deployment:

   * Use `deposit()` with value (set ETH amount in Remix value field) to deposit.
   * Use `getBalance()` to read the balance (returns wei).
   * Use `withdraw(to, amount)` (owner must call) to withdraw.

---

# Example values / conversions

* To deposit 0.01 ETH in Remix, set Value = `0.01` ETH and call `deposit()` or send directly to contract address.
* `getBalance()` returns an integer in **wei**. Convert to ETH: `eth = wei / 1e18`.

---

# Minimal Hardhat test example (JS)

If you want a quick automated test using Hardhat (node.js), here's an example test:

```js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("SimpleBankAccount", function () {
  it("accepts deposits and allows owner withdrawal", async function () {
    const [deployer, other] = await ethers.getSigners();
    const Bank = await ethers.getContractFactory("SimpleBankAccount");
    const bank = await Bank.deploy(deployer.address); // owner = deployer
    await bank.deployed();

    // Deposit 1 ETH from other
    await other.sendTransaction({ to: bank.address, value: ethers.utils.parseEther("1.0") });

    const bal = await bank.getBalance();
    expect(bal).to.equal(ethers.utils.parseEther("1.0"));

    // Owner withdraws 0.4 ETH to other
    await bank.connect(deployer).withdraw(other.address, ethers.utils.parseEther("0.4"));

    const finalBal = await bank.getBalance();
    expect(finalBal).to.equal(ethers.utils.parseEther("0.6"));
  });
});
```

---

# Security notes (small)

* This contract is minimal and intended for learning / testnets only.
* For production you’d add:

  * ReentrancyGuard (or checks-effects-interactions pattern),
  * Proper access controls & multi-sig for bank functionality,
  * Accounting for tokens vs native ETH,
  * Limits, interest handling, withdrawal delays, KYC off-chain, etc.

---


