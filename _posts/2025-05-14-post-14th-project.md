---
title: "Child Projection (solidity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---



```// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ChildProtection {
    enum ReportType { MISSING, CHILD_LABOR }
    
    struct BasicInfo {
        uint256 id;
        ReportType reportType;
        address reporter;
        string reporterContact;
        string name;
        uint8 age;
    }

    struct DetailedInfo {
        string description;
        string[] photos;
        uint256 timestamp;
        bool resolved;
        uint256 rewardPool;
    }

    struct LaborInfo {
        string location;
        string employerDetails;
        string workType;
    }

    struct StatusInfo {
        address finder;
        string finderStatement;
    }

    struct Report {
        uint256 id;
        ReportType reportType;
        address reporter;
        string reporterContact;
        string name;
        uint8 age;
        string description;
        string[] photos;
        uint256 timestamp;
        bool resolved;
        address finder;
        string finderStatement;
        uint256 rewardPool;
        string location;
        string employerDetails;
        string workType;
    }

    uint256 public reportCounter;
    mapping(uint256 => Report) public reports;
    mapping(uint256 => bool) public rewardClaimed;

    // Events remain the same
    event ReportFiled(
        uint256 id,
        ReportType reportType,
        address indexed reporter,
        string name,
        uint8 age,
        uint256 timestamp
    );

    event ChildLaborReported(
        uint256 id,
        string location,
        string workType
    );

    event FinderDeclared(
        uint256 id,
        address indexed finder,
        string statement
    );

    event ReportResolved(
        uint256 id,
        address indexed finder
    );

    event RewardAdded(
        uint256 id,
        address indexed contributor,
        uint256 amount
    );

    event RewardClaimed(
        uint256 id,
        address indexed finder,
        uint256 amount
    );

    // Modifiers remain the same
    modifier validatePhotos(string[] memory photos) {
        require(photos.length >= 1, "At least 1 photo is required");
        require(photos.length <= 5, "Maximum 5 photos allowed");
        require(bytes(photos[0]).length > 0, "First photo cannot be empty");
        _;
    }

    modifier onlyReporter(uint256 id) {
        require(msg.sender == reports[id].reporter, "Only the reporter can perform this action.");
        _;
    }

    function fileReport(
        ReportType reportType,
        string memory name,
        uint8 age,
        string memory description,
        string[] memory photos,
        string memory reporterContact
    ) public validatePhotos(photos) {
        require(bytes(name).length > 0, "Name is required.");
        require(age > 0, "Age must be greater than 0.");
        require(bytes(description).length > 0, "Description is required.");
        require(bytes(reporterContact).length > 0, "Reporter contact is required.");

        reportCounter++;

        reports[reportCounter] = Report({
            id: reportCounter,
            reportType: reportType,
            reporter: msg.sender,
            reporterContact: reporterContact,
            name: name,
            age: age,
            description: description,
            photos: photos,
            timestamp: block.timestamp,
            resolved: false,
            finder: address(0),
            finderStatement: "",
            rewardPool: 0,
            location: "",
            employerDetails: "",
            workType: ""
        });

        emit ReportFiled(reportCounter, reportType, msg.sender, name, age, block.timestamp);
    }

    function fileChildLaborReport(
        string memory name,
        uint8 age,
        string memory description,
        string[] memory photos,
        string memory reporterContact,
        string memory location,
        string memory employerDetails,
        string memory workType
    ) public validatePhotos(photos) {
        require(bytes(location).length > 0, "Location is required.");
        require(bytes(employerDetails).length > 0, "Employer details are required.");
        require(bytes(workType).length > 0, "Work type is required.");

        reportCounter++;

        reports[reportCounter] = Report({
            id: reportCounter,
            reportType: ReportType.CHILD_LABOR,
            reporter: msg.sender,
            reporterContact: reporterContact,
            name: name,
            age: age,
            description: description,
            photos: photos,
            timestamp: block.timestamp,
            resolved: false,
            finder: address(0),
            finderStatement: "",
            rewardPool: 0,
            location: location,
            employerDetails: employerDetails,
            workType: workType
        });

        emit ReportFiled(reportCounter, ReportType.CHILD_LABOR, msg.sender, name, age, block.timestamp);
        emit ChildLaborReported(reportCounter, location, workType);
    }

    function addReward(uint256 id) public payable {
        require(id > 0 && id <= reportCounter, "Invalid report ID.");
        require(msg.value > 0, "Reward amount must be greater than zero.");
        require(!reports[id].resolved, "Cannot add reward to a resolved report.");

        reports[id].rewardPool += msg.value;

        emit RewardAdded(id, msg.sender, msg.value);
    }

    function declareFound(uint256 id, string memory statement) public {
        require(id > 0 && id <= reportCounter, "Invalid report ID.");
        require(!reports[id].resolved, "Report is already resolved.");
        require(bytes(statement).length > 0, "Finder statement is required.");

        reports[id].finder = msg.sender;
        reports[id].finderStatement = statement;

        emit FinderDeclared(id, msg.sender, statement);
    }

    function resolveReport(uint256 id) public onlyReporter(id) {
        require(!reports[id].resolved, "Report already resolved.");
        require(reports[id].finder != address(0), "No finder has been declared.");

        reports[id].resolved = true;

        emit ReportResolved(id, reports[id].finder);
    }

    function claimReward(uint256 id) public {
        require(reports[id].resolved, "Report is not resolved yet.");
        require(reports[id].finder == msg.sender, "Only the finder can claim the reward.");
        require(!rewardClaimed[id], "Reward already claimed.");

        uint256 rewardAmount = reports[id].rewardPool;
        rewardClaimed[id] = true;

        payable(msg.sender).transfer(rewardAmount);

        emit RewardClaimed(id, msg.sender, rewardAmount);
    }

    function getBasicInfo(uint256 id) public view returns (BasicInfo memory) {
        require(id > 0 && id <= reportCounter, "Invalid report ID.");
        Report storage report = reports[id];
        return BasicInfo(
            report.id,
            report.reportType,
            report.reporter,
            report.reporterContact,
            report.name,
            report.age
        );
    }

    function getDetailedInfo(uint256 id) public view returns (DetailedInfo memory) {
        require(id > 0 && id <= reportCounter, "Invalid report ID.");
        Report storage report = reports[id];
        return DetailedInfo(
            report.description,
            report.photos,
            report.timestamp,
            report.resolved,
            report.rewardPool
        );
    }

    function getLaborInfo(uint256 id) public view returns (LaborInfo memory) {
        require(id > 0 && id <= reportCounter, "Invalid report ID.");
        Report storage report = reports[id];
        return LaborInfo(
            report.location,
            report.employerDetails,
            report.workType
        );
    }

    function getStatusInfo(uint256 id) public view returns (StatusInfo memory) {
        require(id > 0 && id <= reportCounter, "Invalid report ID.");
        Report storage report = reports[id];
        return StatusInfo(
            report.finder,
            report.finderStatement
        );
    }
}
```

The `ChildProtection` smart contract is designed to **report and track cases of missing children and child labor** using the Ethereum blockchain. It adds transparency, immutability, and the possibility to **incentivize reports through rewards**. Below is a detailed breakdown:

---

## 🧱 **Core Concepts & Structures**

### Enum:

```solidity

enum ReportType { MISSING, CHILD_LABOR }

```

- Represents two types of reports: `MISSING` children or `CHILD_LABOR` cases.

---

### Main Struct: `Report`

Combines all relevant information:

```solidity

struct Report {
    uint256 id;
    ReportType reportType;
    address reporter;
    string reporterContact;
    string name;
    uint8 age;
    string description;
    string[] photos;
    uint256 timestamp;
    bool resolved;
    address finder;
    string finderStatement;
    uint256 rewardPool;
    string location;
    string employerDetails;
    string workType;
}

```

> Note: There are helper structs like BasicInfo, DetailedInfo, etc., used for viewing subsets of data more cleanly.
> 

---

## 🔧 **Functional Overview**

### 1. **Filing Reports**

### a. `fileReport(...)` (For MISSING children)

- Caller provides basic child info, a description, contact, and photos.
- Assigns a new report ID and stores the data.
- Emits `ReportFiled`.

### b. `fileChildLaborReport(...)`

- In addition to `fileReport`, includes:
    - `location`, `employerDetails`, and `workType`
- Emits:
    - `ReportFiled`
    - `ChildLaborReported`

---

### 2. **Reward Mechanism**

### `addReward(uint256 id)`

- Anyone can contribute Ether to incentivize the resolution of a case.
- Increases `rewardPool` of the report.
- Emits `RewardAdded`.

---

### 3. **Finder Declaration & Case Resolution**

### `declareFound(uint256 id, string statement)`

- A user claims to have found the missing child / resolved the labor issue.
- Saves their address and statement.
- Emits `FinderDeclared`.

### `resolveReport(uint256 id)`

- **Only the original reporter** can mark the report as resolved.
- Requires a finder to have declared first.
- Emits `ReportResolved`.

---

### 4. **Claiming Rewards**

### `claimReward(uint256 id)`

- Only the declared `finder` can claim the Ether reward.
- Report must be resolved first.
- Can only be claimed once.
- Emits `RewardClaimed`.

---

### 5. **Data Retrieval Helpers**

Each of these functions helps fetch parts of the report based on the report ID:

- `getBasicInfo(id)` → Name, age, contact, etc.
- `getDetailedInfo(id)` → Description, photos, timestamp, etc.
- `getLaborInfo(id)` → For child labor-specific data.
- `getStatusInfo(id)` → Who claimed to have found the child and their statement.

---

## ⚙️ **Modifiers**

- `validatePhotos(...)` – Ensures photo array is 1–5 in length and not empty.
- `onlyReporter(id)` – Restricts function use to the original reporter.

---

## 📡 **Events for Transparency**

Emitted during:

- Report creation
- Child labor specifics
- Finder declaration
- Report resolution
- Reward addition and claim

This makes the contract easily trackable by off-chain apps or UIs.

---

## 🔐 **Security and Logic Considerations**

- Only the **original reporter** can resolve a case.
- Only the **declared finder** can claim the reward.
- **Reward can't be claimed twice.**
- Photos are required for legitimacy.
- Contracts can be **auditable and extendable** by connecting to oracles, law enforcement backends, or IPFS.

---

## 🚀 Real-world Applications

This contract could serve as the backend for:

- A **decentralized reporting system** for child protection NGOs.
- A **bounty platform** encouraging the public to help locate missing children.
- An **immutable registry** to keep cases on-chain with proof.