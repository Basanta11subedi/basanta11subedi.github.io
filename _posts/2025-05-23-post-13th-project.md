---
title: "Marrige and Prenup (solidity)"
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
pragma solidity ^0.8.19;

import "./PrenuptialAgreement.sol";

contract MarriageRegistry {
    struct Spouse {
        string fullName;
        uint256 birthDate;
        string fatherName;
        string motherName;
        string birthPlace;
        string nationality;
        string occupation;
        string residentialAddress;
    }

    struct Marriage {
        Spouse spouse1;
        Spouse spouse2;
        uint256 marriageDate;
        string marriageLocation;
        string ceremonyType;
        address[] witnesses;
        bytes32 prenupId;  // Links to prenup contract
        bool isRegistered;
        bool isPrenupRequired;
    }

    PrenuptialAgreement public prenupContract;
    
    mapping(bytes32 => Marriage) public marriages;
    mapping(address => bytes32) public personToMarriage;
    
    event MarriageRegistered(bytes32 indexed marriageId,address indexed spouse1Address,address indexed spouse2Address,uint256 marriageDate);
    event MarriageVoided(bytes32 indexed marriageId);

    error AlreadyMarried();
    error PrenupNotSigned();
    error NotAuthorized();
    error MarriageNotFound();
    error PrenupRequired();

    constructor(address _prenupContractAddress) {
        prenupContract = PrenuptialAgreement(_prenupContractAddress);
    }

    function registerMarriage(address spouse2Address,Spouse memory _spouse1Details,Spouse memory _spouse2Details,string memory _marriageLocation,string memory _ceremonyType,address[] memory _witnesses,bool _requirePrenup,bytes32 _prenupId) external returns (bytes32) {
        // Check if either party is already married
        if (personToMarriage[msg.sender] != bytes32(0) || 
            personToMarriage[spouse2Address] != bytes32(0)) {
            revert AlreadyMarried();
        }

        // If prenup is required, verify it's signed by both parties
        if (_requirePrenup) {
            (,,,,,,,bool isSignedByFirst, bool isSignedBySecond,,bool isActive) = 
                prenupContract.getPrenupDetails(_prenupId);
            
            if (!isSignedByFirst || !isSignedBySecond || !isActive) {
                revert PrenupNotSigned();
            }
        }

        bytes32 marriageId = keccak256(abi.encodePacked(msg.sender, spouse2Address, block.timestamp));

        Marriage storage newMarriage = marriages[marriageId];
        newMarriage.spouse1 = _spouse1Details;
        newMarriage.spouse2 = _spouse2Details;
        newMarriage.marriageDate = block.timestamp;
        newMarriage.marriageLocation = _marriageLocation;
        newMarriage.ceremonyType = _ceremonyType;
        newMarriage.witnesses = _witnesses;
        newMarriage.prenupId = _prenupId;
        newMarriage.isRegistered = true;
        newMarriage.isPrenupRequired = _requirePrenup;

        personToMarriage[msg.sender] = marriageId;
        personToMarriage[spouse2Address] = marriageId;

        emit MarriageRegistered(marriageId,msg.sender,spouse2Address,block.timestamp);

        return marriageId;
    }

    function getMarriageDetails(bytes32 _marriageId) external view returns (Spouse memory spouse1,Spouse memory spouse2,uint256 marriageDate,string memory marriageLocation,string memory ceremonyType,address[] memory witnesses,bytes32 prenupId,bool isRegistered,bool isPrenupRequired) {
        
        Marriage storage marriage = marriages[_marriageId];
        return (marriage.spouse1,marriage.spouse2,marriage.marriageDate,marriage.marriageLocation,marriage.ceremonyType,marriage.witnesses,marriage.prenupId,marriage.isRegistered,marriage.isPrenupRequired);
    }

    function voidMarriage(bytes32 _marriageId) external {
        Marriage storage marriage = marriages[_marriageId];
        if (!marriage.isRegistered) revert MarriageNotFound();
        
        address spouse1Address = msg.sender;
        address spouse2Address;
        
        // Find spouse2 address from prenup contract
        (spouse1Address, spouse2Address,,,,,,,,,) = prenupContract.getPrenupDetails(marriage.prenupId);
        
        if (msg.sender != spouse1Address && msg.sender != spouse2Address) {
            revert NotAuthorized();
        }

        marriage.isRegistered = false;
        delete personToMarriage[spouse1Address];
        delete personToMarriage[spouse2Address];

        emit MarriageVoided(_marriageId);
    }
}

```

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract PrenuptialAgreement {
    struct AssetDeclaration {
        string description;
        uint256 approximateValue;
        bool isPremaritalAsset;
        address owner;
    }

    struct FinancialTerms {
        string propertyDivision;        // How property will be divided
        string alimonyTerms;            // Terms for spousal support
        string retirementAccounts;      // How retirement accounts will be handled
        string businessAssets;          // Treatment of business interests
        string intellectualProperty;    // Handling of intellectual property
        string investmentAccounts;      // Treatment of investment accounts
        string futureSavings;           // How future savings will be handled
    }

    struct DebtTerms {
        string premaritalDebts;         // How existing debts are handled
        string futureDebts;             // How future debts will be handled
        string studentLoans;            // Specific terms for student loans
        string mortgageTerms;           // How mortgages will be handled
    }

    struct SpecialProvisions {
        string inheritanceTerms;        // How inheritances will be handled
        string giftTerms;               // Treatment of gifts received
        string incomeTerms;             // How income will be handled
        string estatePlanning;          // Estate planning provisions
        string childrenProvisions;      // Provisions regarding future children
    }

    struct PrenupDetails {
        address spouse1;
        address spouse2;
        AssetDeclaration[] assets;
        FinancialTerms financialTerms;
        DebtTerms debtTerms;
        SpecialProvisions specialProvisions;
        bool isOptedIn;                 // True if both parties want prenup
        bool isSignedByFirst;
        bool isSignedBySecond;
        uint256 creationDate;
        bool isActive;
    }

    mapping(bytes32 => PrenupDetails) public prenups;
    mapping(address => bytes32) public personToPrenup;

    event PrenupCreated(bytes32 indexed prenupId, address indexed spouse1, address indexed spouse2);
    event PrenupSigned(bytes32 indexed prenupId, address indexed signer);
    event PrenupCancelled(bytes32 indexed prenupId);
    event AssetAdded(bytes32 indexed prenupId, string description, address owner);
    event OptedOut(bytes32 indexed prenupId);

    error AlreadyHasPrenup();
    error NotAuthorized();
    error AlreadySigned();
    error PrenupNotFound();
    error BothPartiesNotSigned();

    function createPrenup(address _spouse2,FinancialTerms memory _financialTerms,DebtTerms memory _debtTerms,SpecialProvisions memory _specialProvisions) external returns (bytes32) {
        if (personToPrenup[msg.sender] != bytes32(0)) revert AlreadyHasPrenup();
        if (personToPrenup[_spouse2] != bytes32(0)) revert AlreadyHasPrenup();

        bytes32 prenupId = keccak256(abi.encodePacked(msg.sender, _spouse2, block.timestamp));

        PrenupDetails storage newPrenup = prenups[prenupId];
        newPrenup.spouse1 = msg.sender;
        newPrenup.spouse2 = _spouse2;
        newPrenup.financialTerms = _financialTerms;
        newPrenup.debtTerms = _debtTerms;
        newPrenup.specialProvisions = _specialProvisions;
        newPrenup.creationDate = block.timestamp;
        newPrenup.isOptedIn = true;
        newPrenup.isActive = true;

        personToPrenup[msg.sender] = prenupId;
        personToPrenup[_spouse2] = prenupId;

        emit PrenupCreated(prenupId, msg.sender, _spouse2);
        return prenupId;
    }

    function addAsset(bytes32 _prenupId,string memory _description,uint256 _value,bool _isPremarital) external {
        PrenupDetails storage prenup = prenups[_prenupId];
        if (msg.sender != prenup.spouse1 && msg.sender != prenup.spouse2) {
            revert NotAuthorized();
        }

        AssetDeclaration memory newAsset = AssetDeclaration({
            description: _description,
            approximateValue: _value,
            isPremaritalAsset: _isPremarital,
            owner: msg.sender
        });

        prenup.assets.push(newAsset);
        emit AssetAdded(_prenupId, _description, msg.sender);
    }

    function signPrenup(bytes32 _prenupId) external {
        PrenupDetails storage prenup = prenups[_prenupId];
        
        if (msg.sender == prenup.spouse1) {
            if (prenup.isSignedByFirst) revert AlreadySigned();
            prenup.isSignedByFirst = true;
        } else if (msg.sender == prenup.spouse2) {
            if (prenup.isSignedBySecond) revert AlreadySigned();
            prenup.isSignedBySecond = true;
        } else {
            revert NotAuthorized();
        }

        emit PrenupSigned(_prenupId, msg.sender);
    }

    function optOut(bytes32 _prenupId) external {
        PrenupDetails storage prenup = prenups[_prenupId];
        if (msg.sender != prenup.spouse1 && msg.sender != prenup.spouse2) {
            revert NotAuthorized();
        }
        
        prenup.isOptedIn = false;
        prenup.isActive = false;
        
        delete personToPrenup[prenup.spouse1];
        delete personToPrenup[prenup.spouse2];

        emit OptedOut(_prenupId);
    }

    function getPrenupDetails(bytes32 _prenupId) external view returns (address spouse1,address spouse2,AssetDeclaration[] memory assets,FinancialTerms memory financialTerms,DebtTerms memory debtTerms,SpecialProvisions memory specialProvisions,bool isOptedIn,bool isSignedByFirst,bool isSignedBySecond,uint256 creationDate,bool isActive) {
        PrenupDetails storage prenup = prenups[_prenupId];
        return (prenup.spouse1,prenup.spouse2,prenup.assets,prenup.financialTerms,prenup.debtTerms,prenup.specialProvisions,prenup.isOptedIn,prenup.isSignedByFirst,prenup.isSignedBySecond,prenup.creationDate,prenup.isActive);
    }
}

```

# 💍 **MarriageRegistry & PrenuptialAgreement Smart Contracts Explained**

---

## 🏛️ **1. MarriageRegistry Contract**

This contract handles **on-chain marriage registration**, **storage of personal details**, **verification of prenuptial agreements**, and **voiding of marriages**.

### 🔗 Connects to:

- An external `PrenuptialAgreement` contract to enforce legal terms before marriage.

---

### 📦 Key Structures:

### 👤 `Spouse`

Stores personal info about each person:

```solidity

struct Spouse {
    string fullName;
    uint256 birthDate;
    string fatherName;
    string motherName;
    string birthPlace;
    string nationality;
    string occupation;
    string residentialAddress;
}

```

### 💍 `Marriage`

All details of a marriage record:

```solidity

struct Marriage {
    Spouse spouse1;
    Spouse spouse2;
    uint256 marriageDate;
    string marriageLocation;
    string ceremonyType;
    address[] witnesses;
    bytes32 prenupId;
    bool isRegistered;
    bool isPrenupRequired;
}

```

---

### 🧠 Core Logic:

### ✅ `registerMarriage(...)`

- Inputs: spouse2’s address, both spouses' details, marriage metadata, witnesses, prenup flag, and ID.
- Checks if either party is already married (`personToMarriage` mapping).
- If a **prenup is required**, validates:
    - Both have signed.
    - The prenup is active.
- Generates a `marriageId` using a hash of both addresses and `block.timestamp`.
- Stores and links data in mappings.
- Emits `MarriageRegistered`.

### 📄 `getMarriageDetails(marriageId)`

Returns all data about a marriage.

### ❌ `voidMarriage(marriageId)`

- Can be called by either spouse.
- Validates sender is one of the spouses using the prenup info.
- Deletes both parties’ marriage links.
- Marks the marriage as unregistered.
- Emits `MarriageVoided`.

---

### 🔐 Errors:

- `AlreadyMarried()`: One or both addresses are already married.
- `PrenupNotSigned()`: Required prenup is unsigned or inactive.
- `NotAuthorized()`: Caller is not a spouse.
- `MarriageNotFound()`: ID doesn’t point to a valid registered marriage.

---

### 📊 Mappings:

- `marriages`: Stores each marriage by `bytes32` ID.
- `personToMarriage`: Quickly looks up someone’s current marriage.

---

## 📃 **2. PrenuptialAgreement Contract**

This contract manages **prenuptial agreements** including assets, debts, financial terms, and signatures from both parties.

---

### 📦 Key Structures:

### 📑 `AssetDeclaration`

```solidity

struct AssetDeclaration {
    string description;
    uint256 approximateValue;
    bool isPremaritalAsset;
    address owner;
}

```

### 💸 `FinancialTerms`, 🧾 `DebtTerms`, and 📜 `SpecialProvisions`

Define property division, debt handling, estate planning, and children-related clauses.

### 📂 `PrenupDetails`

```solidity

struct PrenupDetails {
    address spouse1;
    address spouse2;
    AssetDeclaration[] assets;
    FinancialTerms financialTerms;
    DebtTerms debtTerms;
    SpecialProvisions specialProvisions;
    bool isOptedIn;
    bool isSignedByFirst;
    bool isSignedBySecond;
    uint256 creationDate;
    bool isActive;
}

```

---

### 🧠 Core Logic:

### ✅ `createPrenup(...)`

- Accepts the partner’s address and agreement terms.
- Ensures neither party already has a prenup.
- Creates and stores a new prenup using a unique `bytes32` ID.

### ➕ `addAsset(...)`

- Either party can add assets to the prenup.
- Validates that the caller is one of the spouses.

### ✍️ `signPrenup(prenupId)`

- Each party can **sign** only once.
- Changes their signature status (`isSignedByFirst/Second`).

### ❌ `optOut(prenupId)`

- Cancels the prenup if either party no longer agrees.
- Resets the mapping and deactivates the prenup.

### 🔍 `getPrenupDetails(...)`

- Returns full details of a prenup, including signatory status and asset declarations.

---

### 📊 Mappings:

- `prenups`: All prenups by ID.
- `personToPrenup`: Track each person’s associated prenup (one per person).

---

### ⚠️ Errors:

- `AlreadyHasPrenup()`: Prevents duplicates.
- `NotAuthorized()`: Caller isn’t a party to the prenup.
- `AlreadySigned()`: Prevents double-signing.
- `PrenupNotFound()`: Invalid ID.
- `BothPartiesNotSigned()`: Used when required signatures are missing (enforced in `MarriageRegistry`).

---

## 🔄 How They Work Together

| Step | Action | Contract |
| --- | --- | --- |
| 1. | Party A creates prenup and adds terms/assets | `PrenuptialAgreement` |
| 2. | Both parties sign prenup | `PrenuptialAgreement` |
| 3. | Either party initiates `registerMarriage(...)` | `MarriageRegistry` |
| 4. | If prenup required, signatures and status are checked | `MarriageRegistry` calls `PrenuptialAgreement` |
| 5. | Marriage data is stored and mapped | `MarriageRegistry` |
| 6. | Either party can later void the marriage | `MarriageRegistry` |

---

## ✅ Use Case Summary

| Feature | Description |
| --- | --- |
| **Marriage Registration** | Secure, verifiable, and time-stamped |
| **Prenup Integration** | Enforces agreement before allowing marriage |
| **Voiding Marriages** | Allows annulment by either spouse |
| **Asset Declarations** | Transparent handling of personal property |
| **Event Logging** | All critical actions emit events (for frontend) |