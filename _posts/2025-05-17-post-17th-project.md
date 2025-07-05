---
title: "Supply Chain (solidity)"
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
pragma solidity ^0.8.20;
contract supply {
    struct product {
        uint id;
        string name;
        uint manufacturingdate;
        address manufacturer;
        address distributor;
        address retailer;
        string status;
    }
    mapping(uint => product) public products;
    uint public productcount;

    event manufactured(uint id, string name, address indexed manufacturer, uint manufacturingdate);
    event distributed(uint id, address indexed distributor);
    event delivered(uint id, address indexed retailer);
    
    modifier onlymanufacturer(uint _id) {
        require(products[_id].manufacturer == msg.sender, "Only the manufacturer can perform this action");
        _;
    }
    modifier onlydistributor(uint _id) {
        require(products[_id].distributor == msg.sender, "only distributor can perform this task");
        _;
    }
    function addproduct(string memory _name) public {
        productcount++;
        products[productcount] = product({
            id: productcount,
            name: _name,
            manufacturingdate: block.timestamp,
            manufacturer: msg.sender,
            distributor: address(0),
            retailer: address(0),
            status: "Product added"
        });
        emit manufactured(productcount, _name, msg.sender, block.timestamp);
    }
    function shipproduct(uint _id, address _distributor) public onlymanufacturer(_id) {
        products[_id].distributor = _distributor;
        products[_id].status = "Shipped";
        emit distributed(_id, _distributor);
    }
    function delivertoretailer(uint _id, address _retailer) public onlydistributor(_id) {
        products[_id].retailer = _retailer;
        products[_id].status = "Delivered";
        emit delivered(_id, _retailer);
    }
    
    function getproductdetail(uint _id) public view returns (uint, string memory, address, string memory) {
        product memory p = products[_id];
        return (p.id, p.name, p.manufacturer, p.status);
    }
}

```
Here's a **line-by-line explanation** of your Solidity smart contract named **`supply`**, which models a **simple supply chain** for tracking products from **manufacturer → distributor → retailer**.

---

## ✅ File Info

```solidity

// SPDX-License-Identifier: MIT

```

- License declaration: Makes the contract open-source under the MIT License.

```solidity

pragma solidity ^0.8.20;

```

- Specifies the Solidity compiler version (at least `0.8.20`).

---

## 🏗 Contract Declaration

```solidity

contract supply {

```

- Declares the contract named `supply`.

---

## 📦 Struct Definition: `product`

```solidity

    struct product {
        uint id;
        string name;
        uint manufacturingdate;
        address manufacturer;
        address distributor;
        address retailer;
        string status;
    }

```

Defines a **product** with the following fields:

- `id`: Unique ID for each product
- `name`: Product name
- `manufacturingdate`: Timestamp of creation
- `manufacturer`: Address that created the product
- `distributor`: Address to which the product is shipped
- `retailer`: Address of the final delivery
- `status`: Current status (e.g., "Product added", "Shipped", "Delivered")

---

## 🗺 State Variables

```solidity

    mapping(uint => product) public products;

```

- Stores all `product` structs by their `id`.

```solidity

    uint public productcount;

```

- Counter to assign unique product IDs.

---

## 📣 Events

These are used for logging actions on-chain, visible to frontend apps:

```solidity

    event manufactured(uint id, string name, address indexed manufacturer, uint manufacturingdate);
    event distributed(uint id, address indexed distributor);
    event delivered(uint id, address indexed retailer);

```

- `manufactured`: Emitted when a product is created.
- `distributed`: Emitted when shipped to a distributor.
- `delivered`: Emitted when delivered to a retailer.

---

## 🛡️ Modifiers

Modifiers are reusable access-control rules.

### 👨‍🏭 Only Manufacturer

```solidity

    modifier onlymanufacturer(uint _id) {
        require(products[_id].manufacturer == msg.sender, "Only the manufacturer can perform this action");
        _;
    }

```

- Ensures that only the product's manufacturer can perform the action.

### 🛒 Only Distributor

```solidity

    modifier onlydistributor(uint _id) {
        require(products[_id].distributor == msg.sender, "only distributor can perform this task");
        _;
    }

```

- Ensures that only the assigned distributor can perform the next step.

---

## 🛠️ Core Functions

### 1️⃣ Add Product (Create)

```solidity

    function addproduct(string memory _name) public {
        productcount++;

```

- Increments product ID counter.

```solidity

        products[productcount] = product({
            id: productcount,
            name: _name,
            manufacturingdate: block.timestamp,
            manufacturer: msg.sender,
            distributor: address(0),
            retailer: address(0),
            status: "Product added"
        });

```

- Creates a new `product`:
    - `msg.sender` is set as the manufacturer.
    - `distributor` and `retailer` are initialized as `address(0)` (i.e., unset).
    - Status is set to `"Product added"`.

```solidity

        emit manufactured(productcount, _name, msg.sender, block.timestamp);
    }

```

- Emits `manufactured` event.

---

### 2️⃣ Ship Product to Distributor

```solidity

    function shipproduct(uint _id, address _distributor) public onlymanufacturer(_id) {

```

- Only the manufacturer of the product can call this.

```solidity

        products[_id].distributor = _distributor;
        products[_id].status = "Shipped";
        emit distributed(_id, _distributor);
    }

```

- Assigns the `distributor` address.
- Updates the status.
- Emits `distributed` event.

---

### 3️⃣ Deliver Product to Retailer

```solidity

    function delivertoretailer(uint _id, address _retailer) public onlydistributor(_id) {

```

- Only the assigned distributor can call this.

```solidity

        products[_id].retailer = _retailer;
        products[_id].status = "Delivered";
        emit delivered(_id, _retailer);
    }

```

- Assigns the retailer.
- Updates status.
- Emits `delivered` event.

---

### 4️⃣ View Product Info

```solidity

    function getproductdetail(uint _id) public view returns (uint, string memory, address, string memory) {
        product memory p = products[_id];
        return (p.id, p.name, p.manufacturer, p.status);
    }

```

- Allows **anyone** to view key product details: `id`, `name`, `manufacturer`, and current `status`.

---

## ✅ Summary Table

| Feature | Description |
| --- | --- |
| `addproduct()` | Manufacturer creates a new product |
| `shipproduct()` | Ships product from manufacturer to distributor |
| `delivertoretailer()` | Distributor delivers product to retailer |
| `getproductdetail()` | View basic info about the product |
| `onlymanufacturer` | Access control for manufacturer-only actions |
| `onlydistributor` | Access control for distributor-only actions |
| `status` | Tracks progress: `"Product added"` → `"Shipped"` → `"Delivered"` |
| `events` | Used for tracking and integration with frontends/dApps |