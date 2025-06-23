---
title: "Time Transaction (solidity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

contract TimeTransactions {
    struct Transaction {
        address creator;
        address recipient;
        uint256 amount;
        uint256 createdAt;
        uint256 executionTime;
        uint256 tip;
        Status status;
        uint256 transactionFee; // Added to track fee per transaction
    }

    enum Status {
        Pending,
        Executed,
        Reverted
    }

    event TransactionCreated(uint256 indexed transactionCount, address creator, address recipient, uint256 amount, uint256 executionTime, uint256 tip);
    event TransactionReverted(uint256 indexed transactionId);
    event TransactionExecuted(uint256 indexed transactionId, address executor);
    event FundsWithdrawn(address admin, uint256 amount);

    mapping(uint256 => Transaction) public transactions;
    uint256 public transactionCount;
    uint256 public totalExecutedTransactionFees; // Tracks fees from executed transactions
    address public admin;

    modifier onlyAdmin {
        require(msg.sender == admin, "Only admin can call this function");
        _;
    }

    constructor() {
        admin = msg.sender;
    }

    function createTransaction(address _recipient, uint256 _amount, uint256 _executionTime, uint256 _tip) external payable {
        require(_recipient != address(0), "Invalid recipient address");
        require(_amount + _tip > 0, "Amount and tip must be greater than zero");
        require(_executionTime > block.timestamp, "Execution time must be in the future");

        uint256 transactionFee = calculateTransactionFee(_amount);

        require(msg.value == _amount + _tip + transactionFee, "Sent value must equal amount, tip, and transaction fee");

        transactions[transactionCount] = Transaction({
            creator: msg.sender,
            recipient: _recipient,
            amount: _amount,
            createdAt: block.timestamp,
            executionTime: _executionTime,
            tip: _tip,
            status: Status.Pending,
            transactionFee: transactionFee // Store the transaction fee
        });

        transactionCount++;
        emit TransactionCreated(transactionCount - 1, msg.sender, _recipient, _amount, _executionTime, _tip);
    }

    function RevertTransaction(uint256 _transactionId) external {
        Transaction storage transaction = transactions[_transactionId];

        require(transaction.creator == msg.sender, "Only creator can revert the transaction");
        require(transaction.status == Status.Pending, "Transaction is not pending");
        require(block.timestamp < transaction.executionTime, "Revert time has expired");

        transaction.status = Status.Reverted;

        // Refund the full transaction (including fee)
        payable(msg.sender).transfer(transaction.amount + transaction.tip + transaction.transactionFee);

        emit TransactionReverted(_transactionId);
    }

    function ExecuteTransaction(uint256 _transactionId) external {
        Transaction storage transaction = transactions[_transactionId];

        require(transaction.status == Status.Pending, "Transaction is not pending");
        require(transaction.recipient != msg.sender, "Recipient cannot execute this transaction");
        require(block.timestamp >= transaction.executionTime, "Transaction is not yet executable");

        transaction.status = Status.Executed;

        // Transfer the amount to the recipient and tip to the executor
        payable(transaction.recipient).transfer(transaction.amount);
        payable(msg.sender).transfer(transaction.tip);

        // Accumulate executed transaction fee for withdrawal
        totalExecutedTransactionFees += transaction.transactionFee;

        emit TransactionExecuted(_transactionId, msg.sender);
    }

    function calculateTransactionFee(uint256 amount) internal pure returns (uint256) {
        return (amount * 1) / 100; // 1% fee
    }

    // Admin can only withdraw the total fees from executed transactions
    function withdrawTransactionFees(uint256 _amount) external onlyAdmin {
        require(_amount <= totalExecutedTransactionFees, "Insufficient transaction fees to withdraw");
        totalExecutedTransactionFees -= _amount; // Deduct the withdrawn fees
        payable(admin).transfer(_amount);

        emit FundsWithdrawn(admin, _amount);
    }

    // Admin can view the contract's balance
    function getContractBalance() external view onlyAdmin returns (uint256) {
        return totalExecutedTransactionFees; // Return only the fees from executed transactions
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

    function getTransactionsByDateRange(uint256 startTimestamp, uint256 endTimestamp) public view returns (Transaction[] memory) {
        bool returnAll = (startTimestamp == 0 && endTimestamp == 0);

        uint256 count = 0;
        for (uint256 i = 0; i < transactionCount; i++) {
            if (returnAll || (transactions[i].createdAt >= startTimestamp && transactions[i].createdAt <= endTimestamp)) {
                count++;
            }
        }

        Transaction[] memory result = new Transaction[](count);
        uint256 index = 0;
        for (uint256 i = 0; i < transactionCount; i++) {
            if (returnAll || (transactions[i].createdAt >= startTimestamp && transactions[i].createdAt <= endTimestamp)) {
                result[index] = transactions[i];
                index++;
            }
        }

        return result;
    }

    receive() external payable {}
}

```

Here's a **complete explanation** of your `TimeTransactions` smart contract — with **comments, purpose, and behavior of each part of the code**, to help you fully understand what's going on.

---

### 📄 **1. Header and Compiler Version**

```solidity

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

```

- `SPDX-License-Identifier: UNLICENSED`: Code isn't open-source under any specific license.
- `pragma`: Uses Solidity compiler version 0.8.20 or higher.

---

### 🧱 **2. Contract Declaration**

```solidity

contract TimeTransactions {

```

Defines the contract named `TimeTransactions`.

---

### 📦 **3. Structs and Enums**

### Transaction Structure

```solidity

struct Transaction {
    address creator;          // Sender of the transaction
    address recipient;        // Intended recipient
    uint256 amount;           // Main amount to send
    uint256 createdAt;        // When transaction was created
    uint256 executionTime;    // Scheduled future time to execute
    uint256 tip;              // Bonus to executor
    Status status;            // Current status
    uint256 transactionFee;   // 1% fee
}

```

### Transaction Status

```solidity

enum Status {
    Pending,
    Executed,
    Reverted
}

```

Used to track whether a transaction is still waiting (`Pending`), completed (`Executed`), or cancelled (`Reverted`).

---

### 📣 **4. Events**

```solidity

event TransactionCreated(...);
event TransactionReverted(...);
event TransactionExecuted(...);
event FundsWithdrawn(...);

```

These log events help external applications (like frontends or indexers) know when a transaction was:

- Created
- Reverted
- Executed
- Admin withdrew contract fees

---

### 🧾 **5. State Variables**

```solidity

mapping(uint256 => Transaction) public transactions;
uint256 public transactionCount;
uint256 public totalExecutedTransactionFees;
address public admin;

```

- `transactions`: Stores all transactions by their ID.
- `transactionCount`: Number of transactions created.
- `totalExecutedTransactionFees`: Sum of 1% fees collected from executed transactions (admin can withdraw this).
- `admin`: The contract owner/admin (who can withdraw fees).

---

### 🔐 **6. Access Control Modifier**

```solidity

modifier onlyAdmin {
    require(msg.sender == admin, "Only admin can call this function");
    _;
}

```

Ensures only the `admin` address can execute certain functions.

---

### 🏗 **7. Constructor**

```solidity

constructor() {
    admin = msg.sender;
}

```

Sets the **deployer** of the contract as the `admin`.

---

### ➕ **8. Creating a Timed Transaction**

```solidity

function createTransaction(address _recipient, uint256 _amount, uint256 _executionTime, uint256 _tip) external payable

```

### Purpose:

Schedules a transaction for future execution.

### Logic:

1. Requires a valid recipient and a time in the future.
2. Calculates a **1% transaction fee**.
3. Verifies that `msg.value` equals `amount + tip + fee`.
4. Saves transaction with `Pending` status.

```solidity

emit TransactionCreated(...);

```

Logs the transaction creation.

---

### 🔁 **9. Reverting a Pending Transaction**

```solidity

function RevertTransaction(uint256 _transactionId) external

```

### Purpose:

Allows the creator to **cancel** their own transaction if it's **still pending** and **before execution time**.

### Logic:

- Checks the sender is the creator.
- Verifies the transaction hasn’t been executed.
- Refunds full value (`amount + tip + fee`).
- Updates status to `Reverted`.
- Emits event.

---

### ✅ **10. Executing the Transaction**

```solidity

function ExecuteTransaction(uint256 _transactionId) external

```

### Purpose:

Allows **any user except the recipient** to execute the transaction **after the scheduled time**.

### Logic:

- Checks it's still `Pending` and `block.timestamp >= executionTime`.
- Transfers:
    - `amount` to `recipient`
    - `tip` to `executor` (as a reward)
- Adds the fee to `totalExecutedTransactionFees`.
- Marks status as `Executed`.
- Emits event.

---

### 🧮 **11. Calculate Transaction Fee**

```solidity

function calculateTransactionFee(uint256 amount) internal pure returns (uint256)

```

Returns a 1% fee of the amount.

---

### 💸 **12. Admin Withdraws Fees**

```solidity

function withdrawTransactionFees(uint256 _amount) external onlyAdmin

```

Allows the admin to withdraw only the **accumulated 1% fees** from **executed transactions**.

---

### 👀 **13. View Contract Balance**

```solidity

function getContractBalance() external view onlyAdmin returns (uint256)

```

Returns only the withdrawable **executed transaction fees**, not full contract balance.

---

### 🔍 **14. Get a Specific Transaction**

```solidity

function getTransaction(uint256 _transactionId) external view returns (...)

```

Returns core transaction data for a specific ID.

---

### 📆 **15. Get Transactions Within a Date Range**

```solidity

function getTransactionsByDateRange(uint256 startTimestamp, uint256 endTimestamp)

```

Returns a list of transactions created between `startTimestamp` and `endTimestamp`.

If both inputs are `0`, it returns **all transactions**.

---

### ⚠️ **16. Fallback Function**

```solidity

receive() external payable {}

```

Allows the contract to **receive ETH directly**, though it doesn't affect any functionality here.

---

## 🧠 Summary Table

| Feature | Description |
| --- | --- |
| `createTransaction()` | Schedules a future payment with optional tip |
| `ExecuteTransaction()` | Anyone (except recipient) can execute after time and earn the tip |
| `RevertTransaction()` | Creator can cancel before execution time |
| `calculateTransactionFee()` | 1% fee added on creation |
| `withdrawTransactionFees()` | Admin-only fee withdrawal |
| `getTransactionsByDateRange()` | Query for transactions by date |
| `transaction.status` | Tracks Pending, Executed, Reverted |
| `tip` | Reward for the executor |