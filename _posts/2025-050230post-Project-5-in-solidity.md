---
title: "Cafe"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Cafe {
address public owner;
struct MenuItem {
    string name;
    string description;
    uint256 price; // price in Wei (smallest unit of Ether)
}

struct Order {
    uint256 orderId;
    address customer;
    uint256 itemId;
    uint256 timestamp;
    bool isCompleted;
}

mapping(uint256 => MenuItem) public menu; // Menu ID to Menu Item
mapping(uint256 => Order) public orders; // Order ID to Order
uint256 public menuCount;
uint256 public orderCount;

event MenuItemAdded(uint256 itemId, string name, uint256 price);
event OrderPlaced(uint256 orderId, address customer, uint256 itemId, uint256 price);
event OrderCompleted(uint256 orderId);

modifier onlyOwner() {
    require(msg.sender == owner, "Only owner can perform this action");
    _;
}

modifier onlyCustomer(uint256 orderId) {
    require(orders[orderId].customer == msg.sender, "You are not the customer for this order");
    _;
}

constructor() {
    owner = msg.sender;
}

// Add a menu item (only accessible by owner)
function addMenuItem(string memory name, string memory description, uint256 price) public onlyOwner {
    menuCount++;
    menu[menuCount] = MenuItem(name, description, price);
    emit MenuItemAdded(menuCount, name, price);
}

// Remove a menu item (only accessible by owner)
function removeMenuItem(uint256 itemId) public onlyOwner {
    delete menu[itemId];
}

// Place an order
function placeOrder(uint256 itemId) public payable {
    require(itemId > 0 && itemId <= menuCount, "Invalid menu item");
    MenuItem memory item = menu[itemId];
    require(msg.value >= item.price, "Insufficient payment");

    orderCount++;
    orders[orderCount] = Order(orderCount, msg.sender, itemId, block.timestamp, false);

    payable(owner).transfer(msg.value); // Send the payment to the owner

    emit OrderPlaced(orderCount, msg.sender, itemId, item.price);
}

// Mark an order as completed (only accessible by owner)
function completeOrder(uint256 orderId) public onlyOwner {
    require(!orders[orderId].isCompleted, "Order is already completed");

    orders[orderId].isCompleted = true;
    emit OrderCompleted(orderId);
}

// Get order details
function getOrder(uint256 orderId) public view returns (Order memory) {
    return orders[orderId];
}

// Get menu item details
function getMenuItem(uint256 itemId) public view returns (MenuItem memory) {
    return menu[itemId];
}

// Get the total number of orders
function getTotalOrders() public view returns (uint256) {
    return orderCount;
}

// Get the total number of menu items
function getTotalMenuItems() public view returns (uint256) {
    return menuCount;
}
}
```

## ☕ Solidity Smart Contract: `Cafe`

---

### ✅ **License and Version**

```solidity

// SPDX-License-Identifier: MIT

```

- Declares the license type for the contract (MIT).
- It's a best practice to include a license identifier for open-source clarity.

```solidity

pragma solidity ^0.8.0;

```

- Specifies the compiler version (Solidity 0.8.0 or later).
- `pragma` is a directive that ensures compatibility.

---

### 🏛️ **Contract Declaration**

```solidity

contract Cafe {

```

- Declares a new contract named `Cafe`.
- A contract in Solidity is similar to a class in OOP languages.

---

### 👤 **State Variables**

```solidity
address public owner;

```

- `address`: a 20-byte Ethereum address type.
- `public`: creates a getter function automatically.
- `owner`: stores the Ethereum address of the contract's owner.

---

### 📋 **Structs (Custom Data Types)**

```solidity

struct MenuItem {
    string name;
    string description;
    uint256 price; // price in Wei (smallest unit of Ether)
}

```

- Defines a menu item with a name, description, and price.
- `uint256`: unsigned integer, 256 bits, used for gas efficiency.

```solidity

struct Order {
    uint256 orderId;
    address customer;
    uint256 itemId;
    uint256 timestamp;
    bool isCompleted;
}

```

- Represents an order placed by a customer.
- Includes order metadata: ID, customer address, menu item ID, time of order, and status.

---

### 🗺️ **Mappings (Key-Value Storage)**

```solidity

mapping(uint256 => MenuItem) public menu;

```

- Maps a unique item ID to its `MenuItem` struct.
- `public`: makes it readable from outside.

```solidity

mapping(uint256 => Order) public orders;

```

- Maps a unique order ID to its `Order` struct.

```solidity

uint256 public menuCount;
uint256 public orderCount;

```

- Counters to keep track of menu items and orders.

---

### 📣 **Events**

```solidity

event MenuItemAdded(uint256 itemId, string name, uint256 price);
event OrderPlaced(uint256 orderId, address customer, uint256 itemId, uint256 price);
event OrderCompleted(uint256 orderId);

```

- `event`s allow off-chain applications (like dApps) to listen for important contract actions.

---

### 🛡️ **Modifiers (Access Control)**

```solidity

modifier onlyOwner() {
    require(msg.sender == owner, "Only owner can perform this action");
    _;
}

```

- Restricts function access to the contract owner.
- `msg.sender`: address calling the function.
- `_`: represents the function body that uses the modifier.

```solidity

modifier onlyCustomer(uint256 orderId) {
    require(orders[orderId].customer == msg.sender, "You are not the customer for this order");
    _;
}

```

- Restricts access to the customer who placed the specific order.

---

### ⚙️ **Constructor**

```solidity
constructor() {
    owner = msg.sender;
}

```

- Runs once during deployment.
- Assigns the contract deployer as the owner.

---

### 🍽️ **Function: Add Menu Item**

```solidity

function addMenuItem(string memory name, string memory description, uint256 price) public onlyOwner {
    menuCount++;
    menu[menuCount] = MenuItem(name, description, price);
    emit MenuItemAdded(menuCount, name, price);
}

```

- `memory`: temporary storage (vs. `storage`).
- Increments the menu counter.
- Saves a new `MenuItem` into the mapping.
- Emits an event for tracking.

---

### 🧹 **Function: Remove Menu Item**

```solidity

function removeMenuItem(uint256 itemId) public onlyOwner {
    delete menu[itemId];
}

```

- Deletes a menu item by setting its struct values to default (empty).
- Only owner can do this.

---

### 🛒 **Function: Place Order**

```solidity

function placeOrder(uint256 itemId) public payable {
    require(itemId > 0 && itemId <= menuCount, "Invalid menu item");
    MenuItem memory item = menu[itemId];
    require(msg.value >= item.price, "Insufficient payment");

    orderCount++;
    orders[orderCount] = Order(orderCount, msg.sender, itemId, block.timestamp, false);

    payable(owner).transfer(msg.value);

    emit OrderPlaced(orderCount, msg.sender, itemId, item.price);
}

```

- `payable`: allows the function to receive Ether.
- Checks the menu item is valid and that payment is sufficient.
- Creates a new order and transfers the Ether to the owner.
- `block.timestamp`: the time the block was mined.
- `payable(owner).transfer(...)`: sends Ether.

---

### ✅ **Function: Complete Order**

```solidity

function completeOrder(uint256 orderId) public onlyOwner {
    require(!orders[orderId].isCompleted, "Order is already completed");

    orders[orderId].isCompleted = true;
    emit OrderCompleted(orderId);
}

```

- Marks an order as completed.
- Prevents re-completing the same order.

---

### 🔍 **Read Functions (View)**

```solidity

function getOrder(uint256 orderId) public view returns (Order memory) {
    return orders[orderId];
}

```

- Returns full details of a specific order.

```solidity

function getMenuItem(uint256 itemId) public view returns (MenuItem memory) {
    return menu[itemId];
}

```

- Returns full details of a specific menu item.

```solidity

function getTotalOrders() public view returns (uint256) {
    return orderCount;
}

```

- Returns total number of orders.

```solidity

function getTotalMenuItems() public view returns (uint256) {
    return menuCount;
}

```

- Returns total number of menu items.

---

## 🧾 Summary

| Concept | Usage |
| --- | --- |
| `address` | Stores Ethereum addresses (e.g., owner, customers) |
| `struct` | Groups multiple variables (e.g., `MenuItem`, `Order`) |
| `mapping` | Key-value storage for menu and orders |
| `event` | Logs key actions for front-end and monitoring |
| `modifier` | Restricts access to specific users (owner/customer) |
| `constructor` | Initializes the contract state |
| `payable` | Enables function to receive/send Ether |
| `block.timestamp` | Provides the current block time |