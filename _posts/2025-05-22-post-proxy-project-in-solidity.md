---
title: "  Proxy Contract Example Code: Bank (Solidity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

1. ✅ `Proxy`: Delegates calls to another logic contract (upgradeable).
2. 🏦 `Bank`: The original logic contract with deposit/withdraw.
3. 🔁 `Bank2`: The upgraded version of the Bank with an added transfer function.

---

## 🧠 Goal: Upgradeable Smart Contracts Using Proxy Pattern

This structure allows you to:

- Deploy **one permanent proxy** that users interact with.
- Swap the **underlying logic contract** (Bank → Bank2) over time.
- Keep all user balances and contract state persistent via `delegatecall`.

---

## 📁 1. Proxy Contract

```solidity

contract Proxy {
    address public implementation;
    address public admin;

    constructor(address _implementation) {
        implementation = _implementation;
        admin = msg.sender;
    }

    function upgrade(address newImplementation) public {
        implementation = newImplementation;
    }

    fallback() external payable {
        address impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }

    receive() external payable {}
}

```

### 🔍 What It Does:

- Stores the address of the **logic contract** in `implementation`.
- Lets the `admin` upgrade to a new logic contract using `upgrade()`.
- Forwards any calls it **doesn’t recognize** to `implementation` using `delegatecall`.

> delegatecall runs the logic contract in the context of the proxy, meaning the proxy’s storage is used — not the logic contract’s.
> 

---

## 🏦 2. `Bank` (Initial Logic Contract)

```solidity

contract Bank {
    mapping(address => uint) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint amount) public payable {
        require(balances[msg.sender] >= amount, "Insufficient balances to withdraw");
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
}

```

### 💡 What It Does:

- Users can **deposit Ether** and it updates their balance.
- Users can **withdraw Ether** from their balance.
- Data is stored in the `balances` mapping.

When called through the **proxy**, the balances are **stored in the proxy’s storage** — this is crucial for upgradability.

---

## 🔁 3. `Bank2` (Upgraded Logic Contract)

```solidity

contract Bank2 {
    mapping(address => uint) public balances;

    function deposit(uint amount) public payable {
        balances[msg.sender] += amount;
    }

    function withdraw(uint256 amount) public payable {
        require(balances[msg.sender] >= amount, "Insufficient balances");
        balances[msg.sender] -= amount;
    }

    function transfer(address recipient, uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance to send");
        balances[msg.sender] -= amount;
        balances[recipient] += amount;
    }
}

```

### 🆕 What’s New:

- Same `balances` layout — **this is crucial** so storage mapping works with the proxy.
- Adds a **`transfer()`** function so users can send balance directly to others.
- `withdraw()` doesn't transfer Ether here — possibly a placeholder or error.

> Always ensure the storage layout matches when upgrading logic contracts to avoid storage corruption.
> 

---

## 🧪 Example Usage Flow

### 🔧 Deployment Flow

1. Deploy `Bank` contract.
2. Deploy `Proxy`, passing `Bank`'s address to constructor.
3. Interact with proxy:
    - `Proxy.deposit()` delegates to `Bank.deposit()`.
    - User balance stored in proxy contract.

---

### 🔁 Upgrade to `Bank2`

1. Call `Proxy.upgrade(Bank2 address)` (ideally restricted to admin).
2. Now, all calls to proxy are delegated to `Bank2`.

### 📤 Transfer Funds

User calls:

```solidity

Proxy.transfer(address recipient, amount)

```

- Executes `Bank2.transfer()` via delegatecall.
- Balances still stored in proxy, but logic has changed.

---

## 🔒 Important Considerations

| Concern | Solution |
| --- | --- |
| 💥 Anyone can upgrade | Add `require(msg.sender == admin)` check |
| 🧠 Storage mismatch on upgrade | Maintain exact layout across logic versions |
| ⚠️ Missing `initialize` logic | Add init function for first-time setup |
| 🔍 Debugging delegatecall | Use proxy logs or events in logic contract |

---

## ✅ Final Thoughts

Proxy contracts are essential in any mature smart contract ecosystem because:

- **They allow upgrades without redeployment.**
- **They separate logic from data**, making your architecture modular.
- **They offer gas savings** via contract reuse.

---

## 🛠 Summary Table

| Contract | Role |
| --- | --- |
| Proxy | Stores state & delegates calls |
| Bank | Initial logic (deposit/withdraw) |
| Bank2 | Upgraded logic (adds transfer) |