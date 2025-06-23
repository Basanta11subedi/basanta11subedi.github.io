---
title: "CrowdFunding"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---


```
/ SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Crowdfunding {
struct Campaign {
address payable creator;
string title;
string description;
string evidence;
uint fundingGoal;
uint deadline;
uint amountRaised;
bool isCompleted;
mapping(address => uint) contributions;
}

address public owner;
uint public campaignCount;
uint public totalFeesCollected;
mapping(uint => Campaign) public campaigns;

event CampaignCreated(
    uint campaignId,
    address creator,
    string title,
    string evidence,
    uint fundingGoal,
    uint deadline
);
event ContributionMade(uint campaignId, address contributor, uint amount);
event FundsWithdrawn(uint campaignId, uint amountToCreator, uint feeToContract);
event ContractFeeWithdrawn(uint amount);

modifier onlyOwner() {
    require(msg.sender == owner, "Only the owner can perform this action");
    _;
}

modifier isCampaignActive(uint _campaignId) {
    Campaign storage campaign = campaigns[_campaignId];
    require(block.timestamp < campaign.deadline, "Campaign deadline passed");
    require(!campaign.isCompleted, "Campaign is completed");
    _;
}

constructor() {
    owner = msg.sender;
}

function createCampaign(
    string memory _title,
    string memory _description,
    string memory _evidence,
    uint _fundingGoal,
    uint _duration
) external {
    require(_fundingGoal > 0, "Funding goal must be greater than 0");
    require(_duration > 0, "Duration must be greater than 0");

    Campaign storage newCampaign = campaigns[campaignCount];
    newCampaign.creator = payable(msg.sender);
    newCampaign.title = _title;
    newCampaign.description = _description;
    newCampaign.evidence = _evidence;
    newCampaign.fundingGoal = _fundingGoal;
    newCampaign.deadline = block.timestamp + _duration;

    emit CampaignCreated(campaignCount, msg.sender, _title, _evidence, _fundingGoal, newCampaign.deadline);

    campaignCount++;
}

function contribute(uint _campaignId) external payable isCampaignActive(_campaignId) {
    Campaign storage campaign = campaigns[_campaignId];
    require(msg.value > 0, "Contribution must be greater than 0");
    require(campaign.amountRaised < campaign.fundingGoal, "Funding goal already reached");

    campaign.contributions[msg.sender] += msg.value;
    campaign.amountRaised += msg.value;

    emit ContributionMade(_campaignId, msg.sender, msg.value);
}

function withdrawFunds(uint _campaignId) external {
    Campaign storage campaign = campaigns[_campaignId];
    require(msg.sender == campaign.creator, "Only the creator can withdraw funds");
    require(!campaign.isCompleted, "Funds already withdrawn");
    require(
        block.timestamp >= campaign.deadline || campaign.amountRaised >= campaign.fundingGoal,
        "Campaign is still active"
    );

    uint fee = (campaign.amountRaised * 2) / 100; // 2% fee
    uint amountToCreator = campaign.amountRaised - fee;

    campaign.isCompleted = true;
    totalFeesCollected += fee;

    (bool success, ) = campaign.creator.call{value: amountToCreator}("");
    require(success, "Transfer to creator failed");

    emit FundsWithdrawn(_campaignId, amountToCreator, fee);
}

function withdrawContractFees() external onlyOwner {
    require(totalFeesCollected > 0, "No fees available to withdraw");
    uint amount = totalFeesCollected;
    totalFeesCollected = 0;

    (bool success, ) = owner.call{value: amount}("");
    require(success, "Transfer to owner failed");

    emit ContractFeeWithdrawn(amount);
}

function getCampaignDetails(uint _campaignId) external view returns (
    address creator,
    string memory title,
    string memory description,
    string memory evidence,
    uint fundingGoal,
    uint deadline,
    uint amountRaised,
    bool isCompleted
) {
    Campaign storage campaign = campaigns[_campaignId];
    return (
        campaign.creator,
        campaign.title,
        campaign.description,
        campaign.evidence,
        campaign.fundingGoal,
        campaign.deadline,
        campaign.amountRaised,
        campaign.isCompleted
    );
}}

```

## 💰 Solidity Smart Contract: `Crowdfunding`

> 📍Deployed on Polygon Testnet
> 
> 
> This contract enables decentralized crowdfunding with campaign creation, contribution handling, and fee-based fund withdrawals.
> 

---

### ✅ **License & Version**

```solidity

// SPDX-License-Identifier: MIT

```

- Declares the open-source license (MIT).

```solidity

pragma solidity ^0.8.0;

```

- Requires Solidity version 0.8.0 or higher.
- Ensures compatibility and access to the latest features and safety improvements.

---

### 🏗️ **Contract Declaration**

```solidity

contract Crowdfunding {

```

- Defines a contract named `Crowdfunding`.

---

### 🧱 **Struct: Campaign**

```solidity

struct Campaign {
    address payable creator;
    string title;
    string description;
    string evidence;
    uint fundingGoal;
    uint deadline;
    uint amountRaised;
    bool isCompleted;
    mapping(address => uint) contributions;
}

```

Defines the structure for a single campaign:

| Field | Type | Description |
| --- | --- | --- |
| `creator` | `address payable` | Campaign owner, allowed to receive funds. |
| `title` | `string` | Campaign title. |
| `description` | `string` | Description of the campaign. |
| `evidence` | `string` | Proof or evidence (e.g., IPFS link, image URL). |
| `fundingGoal` | `uint` | Funding target in wei. |
| `deadline` | `uint` | Campaign end time (timestamp). |
| `amountRaised` | `uint` | Amount contributed so far. |
| `isCompleted` | `bool` | True once funds have been withdrawn. |
| `contributions` | `mapping` | Tracks how much each address has contributed. |

🧠 Note: `mapping` inside a `struct` cannot be returned directly — it’s internal storage only.

---

### 📦 **Global Variables**

```solidity

address public owner;
uint public campaignCount;
uint public totalFeesCollected;
mapping(uint => Campaign) public campaigns;

```

- `owner`: Contract deployer; can withdraw platform fees.
- `campaignCount`: Tracks number of campaigns.
- `totalFeesCollected`: 2% fee from campaigns.
- `campaigns`: Maps a campaign ID (`uint`) to its data (`Campaign` struct).

---

### 📣 **Events**

```solidity

event CampaignCreated(...);
event ContributionMade(...);
event FundsWithdrawn(...);
event ContractFeeWithdrawn(...);

```

These allow external interfaces (like dApps) to listen for changes:

| Event | Description |
| --- | --- |
| `CampaignCreated` | Triggered when a new campaign is made. |
| `ContributionMade` | Logs a new contribution. |
| `FundsWithdrawn` | Logs fund withdrawal by a creator. |
| `ContractFeeWithdrawn` | Logs withdrawal of contract fees by the owner. |

---

### 🔐 **Access Control Modifiers**

```solidity

modifier onlyOwner() {
    require(msg.sender == owner, "Only the owner can perform this action");
    _;
}

```

- Restricts a function to the contract `owner`.

```solidity

modifier isCampaignActive(uint _campaignId) {
    Campaign storage campaign = campaigns[_campaignId];
    require(block.timestamp < campaign.deadline, "Campaign deadline passed");
    require(!campaign.isCompleted, "Campaign is completed");
    _;
}

```

- Ensures a campaign is **still running** before allowing contributions.

---

### 🏁 **Constructor**

```solidity

constructor() {
    owner = msg.sender;
}

```

- Sets the contract deployer as `owner`.

---

### 🆕 **Function: `createCampaign(...)`**

```solidity

function createCampaign(...) external { ... }

```

| Parameter | Type | Description |
| --- | --- | --- |
| `_title` | string | Campaign name. |
| `_description` | string | Campaign details. |
| `_evidence` | string | Link or hash to proof (images, documents). |
| `_fundingGoal` | uint | Minimum funding target in wei. |
| `_duration` | uint | Campaign length in seconds. |

Steps:

- Validates goal and duration.
- Initializes a new campaign in the `campaigns` mapping.
- Emits `CampaignCreated`.

---

### 💸 **Function: `contribute(...)`**

```solidity

function contribute(uint _campaignId) external payable isCampaignActive(_campaignId)

```

- Requires payment (`payable`) and checks if the campaign is active.
- Verifies non-zero contribution and funding goal isn't already reached.
- Adds contribution to campaign’s `amountRaised`.
- Tracks how much each sender contributed.
- Emits `ContributionMade`.

---

### 🏦 **Function: `withdrawFunds(...)`**

```solidity

function withdrawFunds(uint _campaignId) external

```

Only callable by campaign **creator** if:

- Campaign is **complete** (deadline passed or goal reached).
- Campaign not already withdrawn.

**Fee Structure:**

- 2% platform fee taken from total raised.
- Remaining 98% sent to the creator using `call{value: ...}` (modern recommended pattern).

**Updates:**

- Marks campaign as `isCompleted = true`.
- Adds fee to `totalFeesCollected`.
- Emits `FundsWithdrawn`.

---

### 🧾 **Function: `withdrawContractFees()`**

```solidity

function withdrawContractFees() external onlyOwner

```

- Lets contract `owner` collect accumulated 2% fees.
- Uses `call` to transfer fees.
- Emits `ContractFeeWithdrawn`.

---

### 🔍 **Function: `getCampaignDetails(...)`**

```solidity

function getCampaignDetails(...) external view returns (...)

```

Returns read-only information about a campaign, excluding internal mappings:

- `creator`
- `title`
- `description`
- `evidence`
- `fundingGoal`
- `deadline`
- `amountRaised`
- `isCompleted`

Useful for front-end UIs.

---

## 📚 Summary Table

| Concept | Description |
| --- | --- |
| `struct` | Used to model a campaign’s data. |
| `mapping` | Associates campaign IDs with campaigns, and contributors with amounts. |
| `modifier` | Reusable conditions (like access control). |
| `event` | Emits logs for external applications to track changes. |
| `call{value: ...}` | Secure and flexible Ether transfer method. |
| `block.timestamp` | Current time used to compare against deadlines. |
| `payable` | Marks functions or addresses that can send/receive Ether. |