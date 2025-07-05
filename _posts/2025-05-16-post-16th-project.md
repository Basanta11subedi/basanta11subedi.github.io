---
title: "MarketPlace (solidity)"
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

contract RealEstateMarketplace {
    address public owner;
    
    struct Property {
        uint256 id;
        address currentOwner;
        uint256 price;
        bool isAvailable;
    }
    
    uint256 public propertyCount;
    uint256 public accumulatedFees;  // Accumulated 2% fees
    mapping(uint256 => Property) public properties;
    mapping(address => uint256) public balances;
    
    event PropertyListed(uint256 indexed propertyId, address indexed owner, uint256 price);
    event PropertySold(uint256 indexed propertyId, address indexed buyer, uint256 price);
    event FeesWithdrawn(address indexed owner, uint256 amount);
    event ContractFunded(address indexed funder, uint256 amount);
    
    modifier onlyOwner() {
        require(msg.sender == owner, "You are not the owner.");
        _;
    }

    modifier onlyOwnerOfProperty(uint256 propertyId) {
        require(properties[propertyId].currentOwner == msg.sender, "You are not the owner of this property.");
        _;
    }

    modifier isAvailable(uint256 propertyId) {
        require(properties[propertyId].isAvailable, "This property is not available for sale.");
        _;
    }

    modifier sufficientFunds(uint256 propertyId) {
        require(msg.value >= properties[propertyId].price, "Insufficient funds to purchase the property.");
        _;
    }

    constructor() {
        owner = msg.sender;
        propertyCount = 0;
        accumulatedFees = 0;
    }

    // Function to list a property
    function listProperty(uint256 price) external payable {
        require(price > 0, "Price must be greater than 0.");
        require(msg.value == (price * 2) / 100, "You must pay a 2% fee for listing this property.");

        // Collect the 2% fee and add it to the accumulated fees
        accumulatedFees += msg.value;
        
        propertyCount++;
        properties[propertyCount] = Property({
            id: propertyCount,
            currentOwner: msg.sender,
            price: price,
            isAvailable: true
        });
        
        emit PropertyListed(propertyCount, msg.sender, price);
    }

    // Function to buy a property
    function buyProperty(uint256 propertyId) external payable isAvailable(propertyId) sufficientFunds(propertyId) {
        address seller = properties[propertyId].currentOwner;
        uint256 price = properties[propertyId].price;

        // Collect the 2% fee from the buyer
        uint256 fee = (price * 2) / 100;
        accumulatedFees += fee;

        // Transfer the property price to the seller
        payable(seller).transfer(price - fee);

        // Transfer ownership of the property
        properties[propertyId].currentOwner = msg.sender;
        properties[propertyId].isAvailable = false;
        
        emit PropertySold(propertyId, msg.sender, price);
    }

    // Function to withdraw accumulated fees by the owner
    function withdrawFees() external onlyOwner {
        uint256 amount = accumulatedFees;
        require(amount > 0, "No fees to withdraw.");
        
        // Reset the accumulated fees after withdrawal
        accumulatedFees = 0;
        
        payable(owner).transfer(amount);

        emit FeesWithdrawn(owner, amount);
    }

    // Function to remove a listing (owner can remove the property from sale)
    function removeListing(uint256 propertyId) external onlyOwnerOfProperty(propertyId) {
        properties[propertyId].isAvailable = false;
    }

    // Function to view property details
    function viewProperty(uint256 propertyId) external view returns (uint256, address, uint256, bool) {
        Property memory prop = properties[propertyId];
        return (prop.id, prop.currentOwner, prop.price, prop.isAvailable);
    }

        // Function to fund the contract
    function fundContract() external payable onlyOwner(){
        require(msg.value > 0, "You must send Ether to fund the contract.");
        emit ContractFunded(msg.sender, msg.value);
    }

}

```
---

## 📜 File Header

```solidity

// SPDX-License-Identifier: MIT

```

- Software license identifier. "MIT" is an open-source license.

```solidity

pragma solidity ^0.8.0;

```

- Declares the Solidity compiler version used. Code works with **v0.8.0 or above**.

---

## 🏢 Contract Declaration

```solidity

contract RealEstateMarketplace {

```

- Starts definition of the contract named `RealEstateMarketplace`.

---

## 🔐 State Variables

```solidity

address public owner;

```

- Stores the address of the **contract deployer (super admin)**.
- `public`: auto-generates a getter.

---

## 🧱 Structs

```solidity

struct Property {
    uint256 id;
    address currentOwner;
    uint256 price;
    bool isAvailable;
}

```

- Describes a **real estate property**:
    - `id`: unique property ID.
    - `currentOwner`: current owner’s address.
    - `price`: sale price in wei.
    - `isAvailable`: whether it's listed for sale.

---

## 🔢 Counters and Mappings

```solidity

uint256 public propertyCount;
uint256 public accumulatedFees;

```

- `propertyCount`: total number of properties listed.
- `accumulatedFees`: holds total 2% platform fees (from listing and sales).

```solidity

mapping(uint256 => Property) public properties;

```

- Maps a property ID to its `Property` struct.

```solidity

mapping(address => uint256) public balances;

```

- Not used in the current logic (can be used for user balances or escrow tracking). Possibly leftover or for future use.

---

## 📣 Events

```solidity

event PropertyListed(uint256 indexed propertyId, address indexed owner, uint256 price);
event PropertySold(uint256 indexed propertyId, address indexed buyer, uint256 price);
event FeesWithdrawn(address indexed owner, uint256 amount);
event ContractFunded(address indexed funder, uint256 amount);

```

- Events log important state changes for frontend UIs or indexers like The Graph.
- `indexed`: makes arguments filterable in event logs.

---

## 🔐 Modifiers

### `onlyOwner()`

```solidity

modifier onlyOwner() {
    require(msg.sender == owner, "You are not the owner.");
    _;
}

```

- Restricts function to contract owner.

### `onlyOwnerOfProperty(uint256 propertyId)`

```solidity

modifier onlyOwnerOfProperty(uint256 propertyId) {
    require(properties[propertyId].currentOwner == msg.sender, "You are not the owner of this property.");
    _;
}

```

- Restricts function to **property owner**.

### `isAvailable(uint256 propertyId)`

```solidity

modifier isAvailable(uint256 propertyId) {
    require(properties[propertyId].isAvailable, "This property is not available for sale.");
    _;
}

```

- Ensures property is currently listed.

### `sufficientFunds(uint256 propertyId)`

```solidity

modifier sufficientFunds(uint256 propertyId) {
    require(msg.value >= properties[propertyId].price, "Insufficient funds to purchase the property.");
    _;
}

```

- Requires buyer to send enough ETH to cover property price.

---

## 🏗️ Constructor

```solidity

constructor() {
    owner = msg.sender;
    propertyCount = 0;
    accumulatedFees = 0;
}

```

- Sets `owner` to the address that deploys the contract.
- Initializes counters to 0.

---

## 🏠 FUNCTION: `listProperty`

```solidity

function listProperty(uint256 price) external payable {
    require(price > 0, "Price must be greater than 0.");
    require(msg.value == (price * 2) / 100, "You must pay a 2% fee for listing this property.");

```

- Lists a property for sale.
- Requires a non-zero price.
- Requires a **2% listing fee**, calculated as `price * 2 / 100`.

```solidity

    accumulatedFees += msg.value;

```

- Stores the listing fee.

```solidity

    propertyCount++;
    properties[propertyCount] = Property({
        id: propertyCount,
        currentOwner: msg.sender,
        price: price,
        isAvailable: true
    });

```

- Creates a new property and stores it in the `properties` mapping.

```solidity

    emit PropertyListed(propertyCount, msg.sender, price);
}

```

- Emits event for off-chain listeners.

---

## 🛒 FUNCTION: `buyProperty`

```solidity

function buyProperty(uint256 propertyId) external payable isAvailable(propertyId) sufficientFunds(propertyId) {

```

- User can buy a listed property.
- Must pass `isAvailable` and `sufficientFunds` modifiers.

```solidity

    address seller = properties[propertyId].currentOwner;
    uint256 price = properties[propertyId].price;

```

- Stores seller info and price locally.

```solidity

    uint256 fee = (price * 2) / 100;
    accumulatedFees += fee;

```

- Takes another **2% platform fee** (this time from the buyer).

```solidity

    payable(seller).transfer(price - fee);

```

- Transfers **(price - fee)** to the seller.

```solidity

    properties[propertyId].currentOwner = msg.sender;
    properties[propertyId].isAvailable = false;

```

- Updates property ownership and marks as sold.

```solidity

    emit PropertySold(propertyId, msg.sender, price);
}

```

- Emits sale event.

---

## 💸 FUNCTION: `withdrawFees`

```solidity

function withdrawFees() external onlyOwner {
    uint256 amount = accumulatedFees;
    require(amount > 0, "No fees to withdraw.");
    accumulatedFees = 0;
    payable(owner).transfer(amount);
    emit FeesWithdrawn(owner, amount);
}

```

- Contract owner withdraws all platform fees.
- Resets fee counter to 0.
- Transfers to the owner's wallet.

---

## ❌ FUNCTION: `removeListing`

```solidity

function removeListing(uint256 propertyId) external onlyOwnerOfProperty(propertyId) {
    properties[propertyId].isAvailable = false;
}

```

- Allows a property owner to unlist (deactivate) a property.

---

## 🔍 FUNCTION: `viewProperty`

```solidity

function viewProperty(uint256 propertyId) external view returns (uint256, address, uint256, bool) {
    Property memory prop = properties[propertyId];
    return (prop.id, prop.currentOwner, prop.price, prop.isAvailable);
}

```

- Read-only function to return details of a property.

---

## 💰 FUNCTION: `fundContract`

```solidity

function fundContract() external payable onlyOwner() {
    require(msg.value > 0, "You must send Ether to fund the contract.");
    emit ContractFunded(msg.sender, msg.value);
}

```

- Lets the contract owner send ETH into the contract (e.g. for future payments).
- Emits a funding event.

---

## ✅ Summary

| Feature | Description |
| --- | --- |
| List Property | Owner pays 2% fee, lists a new property |
| Buy Property | Buyer pays full price; 2% fee is deducted, rest sent to seller |
| Fee Management | Owner can withdraw accumulated fees |
| Property Management | Owner can view, remove listing |
| Contract Funding | Owner can fund the contract |
| Event Logs | Used for off-chain interaction with UIs |