```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract StudentRecords {
    address public owner;

    struct Student {
        uint256 id;
        string name;
        uint8 age;
        string course;
        address addedBy;
    }

    Student[] private students;
    mapping(uint256 => uint256) private idToIndex;
    mapping(uint256 => bool) private exists;

    event StudentAdded(uint256 indexed id, string name, address indexed addedBy);
    event StudentUpdated(uint256 indexed id, string name);
    event Received(address indexed sender, uint256 amount);
    event Withdrawn(address indexed to, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function addStudent(uint256 _id, string calldata _name, uint8 _age, string calldata _course) external {
        require(!exists[_id], "Student already exists");
        students.push(Student({id: _id, name: _name, age: _age, course: _course, addedBy: msg.sender}));
        idToIndex[_id] = students.length - 1;
        exists[_id] = true;
        emit StudentAdded(_id, _name, msg.sender);
    }

    function getStudent(uint256 _id) external view returns (uint256 id, string memory name, uint8 age, string memory course, address addedBy) {
        require(exists[_id], "Student not found");
        Student storage s = students[idToIndex[_id]];
        return (s.id, s.name, s.age, s.course, s.addedBy);
    }

    function getAllStudents() external view returns (Student[] memory) {
        return students;
    }

    function updateStudent(uint256 _id, string calldata _name, uint8 _age, string calldata _course) external {
        require(exists[_id], "Student not found");
        Student storage s = students[idToIndex[_id]];
        s.name = _name;
        s.age = _age;
        s.course = _course;
        emit StudentUpdated(_id, _name);
    }

    // Allow contract to receive ETH (testnet funds for observing gas/tx fee)
    receive() external payable {
        emit Received(msg.sender, msg.value);
    }

    // Fallback in case calldata does not match any function
    fallback() external payable {
        emit Received(msg.sender, msg.value);
    }

    function contractBalance() external view returns (uint256) {
        return address(this).balance;
    }

    function withdraw(uint256 _amount) external {
        require(msg.sender == owner, "Only owner");
        require(address(this).balance >= _amount, "Insufficient balance");
        payable(owner).transfer(_amount);
        emit Withdrawn(owner, _amount);
    }
}
```
