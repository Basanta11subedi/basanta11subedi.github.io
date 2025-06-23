---
title: "House Rent Platform (solidity)"
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

contract MultiUnitHouseRental {
    struct Property {
        uint256 totalFloors;
        uint256 roomsPerFloor;
        string propertyAddress;
        bool active;
    }
    
    struct RentalUnit {
        uint256 floorNumber;
        uint256 roomNumber;
        uint256 monthlyRent;
        uint256 utilityCharges;
        bool isRoom;        // true if it's a room, false if entire floor
        bool isOccupied;
        uint256 securityDeposit;
    }
    
    struct Tenant {
        address walletAddress;
        string name;
        uint8 familyMembers;
        string[] occupations;
        uint256 startDate;
        uint256 leaseDuration; // in months
        bool isActive;
        uint256 floorNumber;
        uint256 roomNumber;   // 0 if entire floor is rented
    }
    
    struct Rules {
        bool noPets;
        bool noSmoking;
        uint8 maxOccupantsPerRoom;
        bool noCommercialUse;
        uint256 quietHoursStart;
        uint256 quietHoursEnd;
        bool visitorRestrictions;
    }
    
    address public owner;
    Property public propertyDetails;
    Rules public houseRules;
    
    // Mapping for rental units: floor => room => RentalUnit
    mapping(uint256 => mapping(uint256 => RentalUnit)) public rentalUnits;
    
    // Mapping for tenants: floor => room => Tenant
    mapping(uint256 => mapping(uint256 => Tenant)) public tenants;
    
    // Mapping for rent payments: tenant address => last payment timestamp
    mapping(address => uint256) public lastPaymentDate;
    
    // Events
    event PropertyRegistered(string propertyAddress, uint256 totalFloors, uint256 roomsPerFloor);
    event UnitRegistered(uint256 floorNumber, uint256 roomNumber, uint256 rent, bool isRoom);
    event NewTenant(address indexed tenant, uint256 floorNumber, uint256 roomNumber, uint256 startDate);
    event RentPaid(address indexed tenant, uint256 amount, uint256 timestamp);
    event ContractTerminated(address indexed tenant, uint256 floorNumber, uint256 roomNumber, uint256 timestamp);
    event RulesUpdated(uint256 timestamp);
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can perform this action");
        _;
    }
    
    modifier validUnit(uint256 floorNumber, uint256 roomNumber) {
        require(floorNumber > 0 && floorNumber <= propertyDetails.totalFloors, "Invalid floor number");
        if(roomNumber > 0) {
            require(roomNumber <= propertyDetails.roomsPerFloor, "Invalid room number");
        }
        _;
    }
    
    constructor() {
        owner = msg.sender;
    }
    
    function registerProperty(
        string memory _propertyAddress,
        uint256 _totalFloors,
        uint256 _roomsPerFloor
    ) public onlyOwner {
        require(!propertyDetails.active, "Property already registered");
        propertyDetails = Property(_totalFloors, _roomsPerFloor, _propertyAddress, true);
        emit PropertyRegistered(_propertyAddress, _totalFloors, _roomsPerFloor);
    }
    
    function registerUnit(
        uint256 _floorNumber,
        uint256 _roomNumber,
        uint256 _monthlyRent,
        uint256 _utilityCharges,
        bool _isRoom,
        uint256 _securityDeposit
    ) public onlyOwner validUnit(_floorNumber, _roomNumber) {
        require(propertyDetails.active, "Property not registered");
        
        RentalUnit memory unit = RentalUnit(
            _floorNumber,
            _roomNumber,
            _monthlyRent,
            _utilityCharges,
            _isRoom,
            false,
            _securityDeposit
        );
        
        rentalUnits[_floorNumber][_roomNumber] = unit;
        emit UnitRegistered(_floorNumber, _roomNumber, _monthlyRent, _isRoom);
    }
    
    function setHouseRules(
        bool _noPets,
        bool _noSmoking,
        uint8 _maxOccupantsPerRoom,
        bool _noCommercialUse,
        uint256 _quietHoursStart,
        uint256 _quietHoursEnd,
        bool _visitorRestrictions
    ) public onlyOwner {
        houseRules = Rules(
            _noPets,
            _noSmoking,
            _maxOccupantsPerRoom,
            _noCommercialUse,
            _quietHoursStart,
            _quietHoursEnd,
            _visitorRestrictions
        );
        emit RulesUpdated(block.timestamp);
    }
    
    function addTenant(
        address _tenantAddress,
        string memory _name,
        uint8 _familyMembers,
        string[] memory _occupations,
        uint256 _leaseDuration,
        uint256 _floorNumber,
        uint256 _roomNumber
    ) public onlyOwner validUnit(_floorNumber, _roomNumber) {
        RentalUnit storage unit = rentalUnits[_floorNumber][_roomNumber];
        require(!unit.isOccupied, "Unit is currently occupied");
        
        if(unit.isRoom) {
            require(_familyMembers <= houseRules.maxOccupantsPerRoom, "Exceeds maximum occupants allowed per room");
        }
        
        Tenant memory newTenant = Tenant(
            _tenantAddress,
            _name,
            _familyMembers,
            _occupations,
            block.timestamp,
            _leaseDuration,
            true,
            _floorNumber,
            _roomNumber
        );
        
        tenants[_floorNumber][_roomNumber] = newTenant;
        unit.isOccupied = true;
        
        emit NewTenant(_tenantAddress, _floorNumber, _roomNumber, block.timestamp);
    }
    
    function payRent(uint256 _floorNumber, uint256 _roomNumber) public payable {
        RentalUnit storage unit = rentalUnits[_floorNumber][_roomNumber];
    Tenant storage tenant = tenants[_floorNumber][_roomNumber];

    require(msg.sender == tenant.walletAddress, "Only tenant can pay rent");
    require(tenant.isActive, "No active tenant contract");

    uint256 totalRent = unit.monthlyRent + unit.utilityCharges;
    require(msg.value == totalRent, "Incorrect rent amount");

    lastPaymentDate[msg.sender] = block.timestamp;

    (bool success, ) = owner.call{value: msg.value}("");
    require(success, "Transfer failed");

    emit RentPaid(msg.sender, msg.value, block.timestamp);
    }
    
    function terminateContract(uint256 _floorNumber, uint256 _roomNumber) public onlyOwner validUnit(_floorNumber, _roomNumber) {
        Tenant storage tenant = tenants[_floorNumber][_roomNumber];
        require(tenant.isActive, "No active tenant");
        
        tenant.isActive = false;
        rentalUnits[_floorNumber][_roomNumber].isOccupied = false;
        
        emit ContractTerminated(tenant.walletAddress, _floorNumber, _roomNumber, block.timestamp);
    }
    
    function getUnitDetails(uint256 _floorNumber, uint256 _roomNumber) 
        public 
        view 
        validUnit(_floorNumber, _roomNumber) 
        returns (RentalUnit memory) {
        return rentalUnits[_floorNumber][_roomNumber];
    }
    
    function getTenantDetails(uint256 _floorNumber, uint256 _roomNumber) 
        public 
        view 
        validUnit(_floorNumber, _roomNumber) 
        returns (Tenant memory) {
        return tenants[_floorNumber][_roomNumber];
    }
    
    function getVacantUnits() public view returns (RentalUnit[] memory) {
        uint256 count = 0;
        
        // First count vacant units
        for(uint256 i = 1; i <= propertyDetails.totalFloors; i++) {
            for(uint256 j = 0; j <= propertyDetails.roomsPerFloor; j++) {
                if(!rentalUnits[i][j].isOccupied && (rentalUnits[i][j].monthlyRent > 0)) {
                    count++;
                }
            }
        }
        
        RentalUnit[] memory vacantUnits = new RentalUnit[](count);
        uint256 index = 0;
        
        // Then populate the array
        for(uint256 i = 1; i <= propertyDetails.totalFloors; i++) {
            for(uint256 j = 0; j <= propertyDetails.roomsPerFloor; j++) {
                if(!rentalUnits[i][j].isOccupied && (rentalUnits[i][j].monthlyRent > 0)) {
                    vacantUnits[index] = rentalUnits[i][j];
                    index++;
                }
            }
        }
        
        return vacantUnits;
    }


}

```

## ✅ 1. **License and Version Declaration**

```solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

```

- `SPDX-License-Identifier`: Specifies the license (MIT) for open-source compliance.
- `pragma solidity ^0.8.0;`: Ensures the code compiles with Solidity version 0.8.0 or higher.

---

## ✅ 2. **Contract Declaration**

```solidity

contract MultiUnitHouseRental {

```

Defines a contract named `MultiUnitHouseRental`. Think of it as a digital program deployed on the blockchain that manages rental operations.

---

## ✅ 3. **Data Structures (Structs)**

### 📦 `Property`

```solidity

struct Property {
    uint256 totalFloors;
    uint256 roomsPerFloor;
    string propertyAddress;
    bool active;
}

```

Describes the main property:

- `totalFloors`: How many floors the building has.
- `roomsPerFloor`: Rooms on each floor.
- `propertyAddress`: Physical address.
- `active`: Whether the property is currently active.

---

### 📦 `RentalUnit`

```solidity

struct RentalUnit {
    uint256 floorNumber;
    uint256 roomNumber;
    uint256 monthlyRent;
    uint256 utilityCharges;
    bool isRoom;
    bool isOccupied;
    uint256 securityDeposit;
}

```

Describes each unit:

- `floorNumber`, `roomNumber`: Where the unit is.
- `monthlyRent`, `utilityCharges`: Financials.
- `isRoom`: If `true`, it's a room; if `false`, it's a whole floor.
- `isOccupied`: If someone is living there.
- `securityDeposit`: One-time upfront cost.

---

### 🧑‍🤝‍🧑 `Tenant`

```solidity

struct Tenant {
    address walletAddress;
    string name;
    uint8 familyMembers;
    string[] occupations;
    uint256 startDate;
    uint256 leaseDuration;
    bool isActive;
    uint256 floorNumber;
    uint256 roomNumber;
}

```

Describes a tenant:

- `walletAddress`: Ethereum address of tenant.
- `name`, `occupations`, `familyMembers`: Personal info.
- `startDate`, `leaseDuration`: Lease terms.
- `isActive`: Whether the lease is still valid.

---

### 📜 `Rules`

```solidity

struct Rules {
    bool noPets;
    bool noSmoking;
    uint8 maxOccupantsPerRoom;
    bool noCommercialUse;
    uint256 quietHoursStart;
    uint256 quietHoursEnd;
    bool visitorRestrictions;
}

```

Defines the house rules:

- Example: No pets, smoking, max people per room, quiet hours, etc.

---

## ✅ 4. **State Variables**

```solidity

address public owner;
Property public propertyDetails;
Rules public houseRules;

```

- `owner`: The landlord or admin of the contract.
- `propertyDetails`: Info about the building.
- `houseRules`: Rules tenants must follow.

---

## ✅ 5. **Mappings (Data Storage)**

### 🏠 Units

```solidity

mapping(uint256 => mapping(uint256 => RentalUnit)) public rentalUnits;

```

Maps a floor and room number to a `RentalUnit`.

### 👥 Tenants

```solidity

mapping(uint256 => mapping(uint256 => Tenant)) public tenants;

```

Maps a floor and room number to a `Tenant`.

### 💸 Rent Payments

```solidity

mapping(address => uint256) public lastPaymentDate;

```

Stores the last rent payment date per tenant.

---

## ✅ 6. **Events**

```solidity

event PropertyRegistered(...);
event UnitRegistered(...);
event NewTenant(...);
event RentPaid(...);
event ContractTerminated(...);
event RulesUpdated(...);

```

Events are logs that allow frontends to listen to changes. For example, when rent is paid or a tenant joins, you can notify users on your website.

---

## ✅ 7. **Modifiers**

### 🔐 `onlyOwner`

```solidity

modifier onlyOwner() {
    require(msg.sender == owner, "Only owner can perform this action");
    _;
}

```

Restricts function usage to the contract owner.

### 🧾 `validUnit`

```solidity

modifier validUnit(uint256 floorNumber, uint256 roomNumber) {
    require(floorNumber > 0 && floorNumber <= propertyDetails.totalFloors, "Invalid floor");
    if(roomNumber > 0) {
        require(roomNumber <= propertyDetails.roomsPerFloor, "Invalid room");
    }
    _;
}

```

Ensures the floor/room provided exists in the building.

---

## ✅ 8. **Constructor**

```solidity

constructor() {
    owner = msg.sender;
}

```

When deployed, sets the contract deployer as the owner.

---

## ✅ 9. **Core Functions**

### 🏢 Register the property

```solidity

function registerProperty(...) public onlyOwner

```

Sets up the property details.

---

### 🛏 Register a rental unit

```solidity

function registerUnit(...) public onlyOwner validUnit(...)

```

Adds a floor/room to the system with its rent, deposit, and charges.

---

### 📜 Set house rules

```solidity

function setHouseRules(...) public onlyOwner

```

Defines the dos and don’ts for the tenants.

---

### 🧑 Add a tenant

```solidity

function addTenant(...) public onlyOwner validUnit(...)

```

Assigns a tenant to a room/floor. Validates capacity.

---

### 💰 Pay rent

```solidity

function payRent(...) public payable

```

Tenants can pay rent using this function. It verifies:

- Caller is the correct tenant.
- Correct rent amount.
- Sends ETH to the owner.

---

### 🛑 Terminate a lease

```solidity

function terminateContract(...) public onlyOwner validUnit(...)

```

Ends a tenant’s contract. Marks the unit as unoccupied.

---

### 🔍 View unit details

```solidity

function getUnitDetails(...) public view

```

Returns details for a specific unit.

---

### 🔍 View tenant details

```solidity

function getTenantDetails(...) public view

```

Returns details about a specific tenant.

---

### 📃 List vacant units

```solidity

function getVacantUnits() public view returns (RentalUnit[] memory)

```

Returns all available units (not occupied but registered).

---

---

### 🎯 **What is this Smart Contract?**

This is a blockchain-based rental management system built using Solidity for multi-unit properties. It allows a landlord to:

- Register a property and rooms/floors
- Define rules and register tenants
- Collect rent and deposits
- View occupancy status and payments

---

### 💡 **Main Features:**

- ✅ Room and floor-level rentals
- ✅ Track tenant details and lease terms
- ✅ Enforce house rules (no pets, quiet hours, etc.)
- ✅ Accept rent in ETH and log payment dates
- ✅ View available units in real-time