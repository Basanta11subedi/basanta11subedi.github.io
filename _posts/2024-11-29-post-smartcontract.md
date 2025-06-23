---
title: "Smart Contract Example"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

## Task

 Create a smart contract of folowing points included

1. User can create timebound transactions
2. List all the transaction in one pool with reverted, executed and pending 
3. User can revert the transaction if the time has passed
4. Third user or any other user in the pool( except the user who is receiving the transaction ) allow to execute the transaction can
5. In transaction there should be amount included, the time period, tip for executing.

### The solidity contract for this task is:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

contract TimeTransactions{
    struct Transaction{
        address creator;
        address recipient;
        uint256 amount;
        uint256 createdAt;
        uint256 executionTime;
        uint256 tip;
        Status status;
    }

    enum Status{
        Pending,
        Executed,
        Reverted
    }
    event TransactionCreated(uint256 indexed transactionCount, address creator, address recipient,uint256 amount,uint256 executionTime, uint256 tip);
    event TransactionReverted(uint256 indexed transactionId);
    event TransactionExecuted(uint256 indexed transactionId, address executor);
    
    mapping(uint256=> Transaction) public transactions;
    uint256 public transactionCount;

    function createTransaction(address _recipient, uint256 _amount, uint256 _executionTime, uint256 _tip) external payable{
        require(_recipient != address(0), "Invalid Transaction");
        require(_amount>0,"Amount must be greater than zero");
        require(msg.value == _amount + _tip,"Sent value must match amount and tip");
        require(_executionTime > block.timestamp,"Execution time must be in future");

        transactions[transactionCount]= Transaction({
            creator: msg.sender,
            recipient: _recipient,
            amount: _amount,
            createdAt: block.timestamp,
            executionTime: _executionTime,
            tip: _tip,
            status: Status.Pending

        });
        emit TransactionCreated(transactionCount, msg.sender, _recipient, _amount, _executionTime, _tip);
        transactionCount++;

    }
    function RevertTransaction(uint256 _transactionId) external {
        Transaction storage transaction= transactions[_transactionId];

        require(transaction.creator == msg.sender,"Only creator can Revert the transaction");
        require(transaction.status == Status.Pending,"Transaction not in pending status");
        require(block.timestamp < transaction.executionTime, "Revert time experied");

        transaction.status= Status.Reverted;

        payable(msg.sender).transfer(transaction.amount + transaction.tip);

        emit TransactionReverted(_transactionId);
    }

    function ExecuteTransaction(uint256 _transactionId) external{
        Transaction storage transaction= transactions[_transactionId];

        require(transaction.status== Status.Pending,"Transaction Not Pending");
        require(transaction.recipient != msg.sender,"Executor cannot perform this task");
        require(block.timestamp>= transaction.executionTime,"Transaction not yet executable");

        transaction.status= Status.Executed;

        payable(transaction.recipient).transfer(transaction.amount);
        
        payable(msg.sender).transfer(transaction.tip);

        emit TransactionExecuted(_transactionId, msg.sender);

    }
    function getTransaction(uint256 _transactionId) external view returns (
        address creator,
        address recipient,
        uint256 amount,
        uint256 createdAt,
        uint256 executionTime,
        uint256 tip,
        Status status
    ) {
        Transaction storage transaction = transactions[_transactionId];
        return (
            transaction.creator,
            transaction.recipient,
            transaction.amount,
            transaction.createdAt,
            transaction.executionTime,
            transaction.tip,
            transaction.status
        );
    }
    function getAllTransactions() external view returns (Transaction[] memory) {
        Transaction[] memory allTransactions = new Transaction[](transactionCount);
        
        for (uint256 i = 0; i < transactionCount; i++) {
            allTransactions[i] = transactions[i];
        }
        
        return allTransactions;
    }

}
```

### The test code is:

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/TimeTransactions.sol";

contract TimeTransactionsTest is Test{
    TimeTransactions public timetransactions;
    address creator;
    address recipient;
    address executor;

    function setUp() public{
        timetransactions= new TimeTransactions();
        creator= makeAddr("creator");
        recipient= makeAddr("recipient");
        executor= makeAddr("executor");

        vm.deal(creator, 10 ether);
        vm.deal(executor, 10 ether);
    }
    function testcreatTransaction() public{

        uint256 executionTime= block.timestamp + 1 minutes;
        uint256 amount= 0.5 ether;
        uint256 tip= 0.1 ether;

        vm.prank(creator);
        timetransactions.createTransaction{value: amount + tip  }(recipient, amount, executionTime, tip);

        (address txCreator,,,,uint256 txExecutionTime,uint256 txTip ,) = timetransactions.getTransaction(0);
        assertEq(txCreator, creator);
        assertEq(txExecutionTime, executionTime);
        assertEq(txTip,tip);
    }
    function testRevertTransaction() public{
        uint256 executionTime= block.timestamp + 1 minutes;
        uint256 amount = 0.5 ether;
        uint256 tip = 0.1 ether;

        vm.prank(creator);
        timetransactions.createTransaction{value: amount + tip }(
            recipient, 
            amount, 
            executionTime,
            tip
        );
        vm.prank(creator);
        timetransactions.RevertTransaction(0);

        (, , , , , , TimeTransactions.Status status) = timetransactions.getTransaction(0);
        assertEq(uint256(status), uint256(TimeTransactions.Status.Reverted));

    }
    function testExecutedTransaction() public {
        uint256 executionTime= block.timestamp + 1 minutes;
        uint256 amount = 0.5 ether;
        uint256 tip = 0.1 ether;

         vm.prank(creator);
        timetransactions.createTransaction{value: amount + tip }(
            recipient, 
            amount, 
            executionTime,
            tip
        );
        vm.warp(executionTime + 1 minutes);

        vm.prank(creator);
        timetransactions.ExecuteTransaction(0);

        (, , , , , , TimeTransactions.Status status) = timetransactions.getTransaction(0);
        assertEq(uint256(status), uint256(TimeTransactions.Status.Executed));
    }
    function testInsufficientFunds() public {
        uint256 executionTime = block.timestamp + 1 minutes;
        uint256 amount = 0.5 ether;
        uint256 tip = 0.1 ether;
        
        vm.prank(creator);
        vm.expectRevert("Sent value must match amount and tip");
        timetransactions.createTransaction{value: amount}(
            recipient, 
            amount, 
            executionTime,
            tip
        );
    }
    function testCannotExecuteBeforeExecutionTime() public {
        uint256 executionTime = block.timestamp + 1 minutes;
        uint256 amount = 0.5 ether;
        uint256 tip = 0.1 ether;
        
        vm.prank(creator);
        timetransactions.createTransaction{value: amount + tip}(
            recipient, 
            amount, 
            executionTime,
            tip
        );
        
        vm.prank(executor);
        vm.expectRevert("Transaction not yet executable");
        timetransactions.ExecuteTransaction(0);
    }

    
}

```

### The deploy code is:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

import "forge-std/Script.sol";
import "../src/TimeTransactions.sol";

contract DeployTimeTransactions is Script {
    TimeTransactions public timetransactions;

    function run() public {

        vm.startBroadcast();
        timetransactions = new TimeTransactions();
        vm.stopBroadcast();
    
    }
}
```

It is deployed in sepolia testnet using RPC_URL of sepolia and PRIVATE_KEY of my metamask account.