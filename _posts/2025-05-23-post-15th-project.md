---
title: "Party Manager (solidity)"
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

import "lib/openzeppelin-contracts/contracts/access/Ownable.sol";
import "lib/openzeppelin-contracts/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "lib/openzeppelin-contracts/contracts/utils/ReentrancyGuard.sol";

contract PartyManager is ERC721URIStorage, Ownable, ReentrancyGuard {
    struct Party {
        uint256 id;
        string name;
        string description;
        string venue;
        uint256 startTime;
        uint256 endTime;
        uint256 entryFee;
        uint256 maxParticipants;
        address admin;
        address host;
        bool approved;
        address[] participants;
        mapping(address => bool) hasMintedNFT;
    }

    mapping(uint256 => Party) public parties;
    address[] public adminList;
    mapping(address => bool) public admins;
    mapping(uint256 => mapping(uint256 => uint256)) public salePrices; // partyId => tokenId => price
    mapping(uint256 => mapping(uint256 => address)) public nftOwners; // partyId => tokenId => owner

    uint256 public partyCount;
    uint256 public nftCounter;
    uint256 public constant SUPERADMIN_NFT_FEE = 10; // 10% fee on NFT sales

    event AdminAdded(address indexed admin);
    event AdminRemoved(address indexed admin);
    event PartyCreated(uint256 indexed partyId, address indexed admin);
    event PartyApproved(uint256 indexed partyId);
    event PartyDeleted(uint256 indexed partyId);
    event UserJoinedParty(uint256 indexed partyId, address indexed user);
    event NFTMinted(uint256 indexed partyId, address indexed user, uint256 nftId);
    event NFTListed(uint256 indexed partyId, uint256 indexed tokenId, uint256 price);
    event NFTUnlisted(uint256 indexed partyId, uint256 indexed tokenId);
    event NFTSold(uint256 indexed partyId, uint256 indexed tokenId, address indexed buyer, uint256 price);

    modifier onlyAdmin() {
        require(admins[msg.sender], "Not an admin");
        _;
    }

    modifier onlyDuringParty(uint256 _partyId) {
        require(block.timestamp >= parties[_partyId].startTime && block.timestamp <= parties[_partyId].endTime, "Party time over");
        _;
    }

    modifier onlyBeforeParty(uint256 _partyId) {
        require(block.timestamp < parties[_partyId].startTime, "Party already started");
        _;
    }

    modifier onlyApprovedParty(uint256 _partyId) {
        require(parties[_partyId].approved, "Party not approved");
        _;
    }

    constructor() ERC721("PartyNFT", "PNFT") Ownable(msg.sender) {}

    // SuperAdmin Functions
    function addAdmin(address _admin) external onlyOwner {
        require(!admins[_admin], "Already an admin");
        admins[_admin] = true;
        adminList.push(_admin);
        emit AdminAdded(_admin);
    }

    function removeAdmin(address _admin) external onlyOwner {
        require(admins[_admin], "Not an admin");
        admins[_admin] = false;
        // Remove from adminList array
        for (uint i = 0; i < adminList.length; i++) {
            if (adminList[i] == _admin) {
                adminList[i] = adminList[adminList.length - 1];
                adminList.pop();
                break;
            }
        }
        emit AdminRemoved(_admin);
    }

    function approveParty(uint256 _partyId) external onlyOwner {
        parties[_partyId].approved = true;
        emit PartyApproved(_partyId);
    }

    function deleteParty(uint256 _partyId) external onlyOwner {
        delete parties[_partyId];
        emit PartyDeleted(_partyId);
    }

    // Admin Functions
    function createParty(
        string memory _name,
        string memory _description,
        string memory _venue,
        uint256 _startTime,
        uint256 _endTime,
        uint256 _entryFee,
        uint256 _maxParticipants,
        address _host
    ) external onlyAdmin {
        require(_startTime < _endTime, "Invalid time range");

        partyCount++;
        Party storage party = parties[partyCount];
        party.id = partyCount;
        party.name = _name;
        party.description = _description;
        party.venue = _venue;
        party.startTime = _startTime;
        party.endTime = _endTime;
        party.entryFee = _entryFee;
        party.maxParticipants = _maxParticipants;
        party.admin = msg.sender;
        party.host = _host;
        party.approved = false;

        emit PartyCreated(partyCount, msg.sender);
    }

    // User Functions
    function joinParty(uint256 _partyId) external payable onlyApprovedParty(_partyId) onlyBeforeParty(_partyId) {
        Party storage party = parties[_partyId];
        require(msg.value == party.entryFee, "Incorrect entry fee");
        require(party.participants.length < party.maxParticipants, "Party full");

        payable(party.admin).transfer(msg.value);
        party.participants.push(msg.sender);

        emit UserJoinedParty(_partyId, msg.sender);
    }

    function mintNFT(uint256 _partyId, string memory _tokenURI) external onlyDuringParty(_partyId) onlyApprovedParty(_partyId) {
        Party storage party = parties[_partyId];
        require(!party.hasMintedNFT[msg.sender], "NFT already minted");

        nftCounter++;
        _mint(msg.sender, nftCounter);
        _setTokenURI(nftCounter, _tokenURI);
        nftOwners[_partyId][nftCounter] = msg.sender;
        party.hasMintedNFT[msg.sender] = true;

        emit NFTMinted(_partyId, msg.sender, nftCounter);
    }

    // NFT Marketplace Functions
    function listNFT(uint256 _partyId, uint256 _tokenId, uint256 price) external {
        require(ownerOf(_tokenId) == msg.sender, "Not NFT owner");
        require(price > 0, "Price must be greater than zero");

        salePrices[_partyId][_tokenId] = price;
        emit NFTListed(_partyId, _tokenId, price);
    }

    function unlistNFT(uint256 _partyId, uint256 _tokenId) external {
        require(nftOwners[_partyId][_tokenId] == msg.sender, "Not seller");

        delete salePrices[_partyId][_tokenId];
        emit NFTUnlisted(_partyId, _tokenId);
    }

    function buyNFT(uint256 _partyId, uint256 _tokenId) external payable {
        uint256 price = salePrices[_partyId][_tokenId];
        require(price > 0, "NFT not for sale");
        require(msg.value == price, "Incorrect price");

        address seller = nftOwners[_partyId][_tokenId];
        uint256 superAdminFee = (price * SUPERADMIN_NFT_FEE) / 100;
        uint256 sellerAmount = price - superAdminFee;

        payable(owner()).transfer(superAdminFee);
        payable(seller).transfer(sellerAmount);

        _transfer(seller, msg.sender, _tokenId);
        nftOwners[_partyId][_tokenId] = msg.sender;
        delete salePrices[_partyId][_tokenId];

        emit NFTSold(_partyId, _tokenId, msg.sender, price);
    }

    // Fetching Admin List
    function getAllAdmins() external view returns (address[] memory) {
        return adminList;
    }


    function getParty(uint256 _partyId) external view returns (uint256 id,string memory name,string memory description,string memory venue,uint256 startTime,uint256 endTime,uint256 entryFee,uint256 maxParticipants,address admin,address host,bool approved,uint256 participantCount) {
    Party storage party = parties[_partyId];

    return (
        party.id,
        party.name,
        party.description,
        party.venue,
        party.startTime,
        party.endTime,
        party.entryFee,
        party.maxParticipants,
        party.admin,
        party.host,
        party.approved,
        party.participants.length
    );
   }

}
```

This `PartyManager` smart contract is a **party management and NFT ticketing system** built with Solidity. It supports:

- Party creation & approval (by admins/owner),
- User participation with entry fees,
- NFT minting for event attendance,
- NFT resale via an internal marketplace.

It uses **OpenZeppelin** libraries for secure access control and token management:

- `Ownable` – manages super admin (contract owner) privileges.
- `ERC721URIStorage` – for minting NFTs with metadata URIs.
- `ReentrancyGuard` – protects against reentrancy attacks in sensitive functions (e.g., `buyNFT()`).

---

## 🔧 CONTRACT STRUCTURE & STATE VARIABLES

### ✅ Struct: `Party`

Represents an event. Contains:

- `id`, `name`, `description`, `venue`, `startTime`, `endTime`
- `entryFee` – amount users must pay to join
- `maxParticipants`, `participants` – limit and list of attendees
- `admin`, `host` – managing and organizing addresses
- `approved` – must be `true` for users to join or mint
- `hasMintedNFT` – mapping per party to track if a user minted an NFT (1/user/party)

### 🔁 Storage Mappings:

- `parties`: party ID → `Party` struct
- `adminList`: array of current admin addresses
- `admins`: address → bool (isAdmin)
- `salePrices`: party ID + token ID → price (for NFT marketplace)
- `nftOwners`: party ID + token ID → current owner

---

## 📌 CONTRACT FUNCTIONS – EXPLAINED IN FULL

---

### 🏛️ **SuperAdmin Functions (Owner only)**

### `addAdmin(address _admin)`

- Adds a new admin to `adminList` and sets their `admins[_admin] = true`.
- Emits `AdminAdded`.

### `removeAdmin(address _admin)`

- Removes an admin from the mapping and from the `adminList` array.
- Emits `AdminRemoved`.

### `approveParty(uint256 _partyId)`

- Marks a party as approved so that users can join and mint NFTs.
- Emits `PartyApproved`.

### `deleteParty(uint256 _partyId)`

- Deletes the party from storage.
- Emits `PartyDeleted`.

---

### 🔧 **Admin Functions (Only `admins`)**

### `createParty(...)`

Creates a new party.

- Must set:
    - Valid time (`startTime < endTime`)
    - Entry fee and participant cap
    - Event metadata: name, venue, description, etc.
- Emits `PartyCreated`.

---

### 🙋‍♂️ **User Functions**

### `joinParty(uint256 _partyId)`

- User sends ETH equal to the entry fee.
- Allowed **only before** the party starts & **only if approved**.
- Fee is forwarded to the party's admin.
- Adds user to the `participants` list.
- Emits `UserJoinedParty`.

### `mintNFT(uint256 _partyId, string memory _tokenURI)`

- Allowed **only during** the party & only if approved.
- One NFT per user per party.
- Mints ERC721 NFT with metadata.
- Tracks owner and mints count.
- Emits `NFTMinted`.

---

### 🛒 **NFT Marketplace Functions**

### `listNFT(uint256 _partyId, uint256 _tokenId, uint256 price)`

- NFT owner lists their token for sale at a given price.
- Price must be > 0.
- Emits `NFTListed`.

### `unlistNFT(uint256 _partyId, uint256 _tokenId)`

- Removes the NFT from sale.
- Only callable by current owner.
- Emits `NFTUnlisted`.

### `buyNFT(uint256 _partyId, uint256 _tokenId)`

- Buyer must pay the exact listed price.
- Transfers NFT to buyer.
- Sends:
    - 90% to seller
    - 10% fee to contract owner (`SUPERADMIN_NFT_FEE = 10`)
- Cleans up ownership and sale data.
- Emits `NFTSold`.

> 🛡️ ReentrancyGuard protects this function.
> 

---

### 📚 **View Functions**

### `getAllAdmins()`

- Returns the current list of admins.

### `getParty(uint256 _partyId)`

- Returns all details of a party, including number of participants.
- Helpful for off-chain UI display.

---

## ⚠️ MODIFIERS

These restrict when and who can call certain functions.

- `onlyAdmin`: only addresses marked as admins.
- `onlyDuringParty(_partyId)`: current block time must be between start & end.
- `onlyBeforeParty(_partyId)`: current block time must be before party starts.
- `onlyApprovedParty(_partyId)`: the party must be approved by owner.

---

## 📦 EVENTS – For DApps or Indexers

All major actions emit events:

- Admin additions/removals
- Party lifecycle (create/approve/delete)
- User joins
- NFT minting, listing, sale

---

## 💸 ECONOMIC FLOW

- **Users pay entry fee** → sent to party `admin`
- **Users mint NFTs** → free but limited to one per user
- **Users sell NFTs** → seller receives 90%, contract owner receives 10%

---

## 🔐 SECURITY CONSIDERATIONS

- Uses OpenZeppelin `Ownable` for access control
- Uses `ReentrancyGuard` in `buyNFT`
- NFT minting is restricted per user per party
- Entry fees are transferred immediately to prevent locked funds

---

## ✅ USE CASES

- **IRL or Virtual Events** with limited access
- Ticketing where entry is tied to NFT mint
- NFT trading for event memorabilia or access tokens