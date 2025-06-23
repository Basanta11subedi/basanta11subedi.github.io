---
title: "Types of Token"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---
## **1. ERC-20: Fungible Tokens**

### Overview

ERC-20 is a standard for **fungible tokens**, where all tokens are identical and can be exchanged at a 1:1 ratio. These are the most common type of tokens and are extensively used in **Decentralized Finance (DeFi)** and other blockchain-based applications.

### Key Concepts:

1. **Fungibility**:
    - All ERC-20 tokens are interchangeable. For example, 1 USDT (a stablecoin) is always equal to another 1 USDT.
    - They do not have unique properties like serial numbers or distinct characteristics.
2. **Token Operations**
    1. **`balanceOf(address owner)`**: Returns the number of tokens held by a specific address.
    2. **`transfer(address to, uint256 value)`**: Transfers tokens from the caller's account to another address.
    3. **`approve(address spender, uint256 value)`**: Approves another address to spend tokens on behalf of the caller.
    4. **`transferFrom(address from, address to, uint256 value)`**: Transfers tokens from one account to another, using the approved allowance.
    5. **`allowance(address owner, address spender)`**: Shows the remaining tokens that a spender is allowed to transfer on behalf of the owner.
3. **Events**:
    - Transfer events are emitted to keep the blockchain updated with changes in token balances.
    - Approval events notify the blockchain when a user approves another entity to spend tokens on their behalf.

### Use Cases:

1. **Cryptocurrencies**: Many altcoins are built using the ERC-20 standard.
2. **DeFi Protocols**: Lending platforms like Aave or liquidity pools on Uniswap rely on ERC-20 tokens.
3. **Crowdfunding**: ERC-20 tokens are often used in Initial Coin Offerings (ICOs) to distribute project shares.

### Limitations:

1. **Batch Operations**: ERC-20 does not support batch token transfers, leading to higher gas costs.
2. **Approval Risks**: If an external contract is compromised, approved tokens can be drained.

---

## **2. ERC-721: Non-Fungible Tokens (NFTs)**

### Overview

ERC-721 is a standard for **non-fungible tokens (NFTs)**, where each token is **unique** and has its own value. NFTs are widely used for representing ownership of digital or real-world assets.

### Key Concepts:

1. **Non-Fungibility**:
    - Unlike ERC-20, ERC-721 tokens are not interchangeable.
    - Each token has a unique identifier (`tokenId`) that distinguishes it from others.
2. **Ownership**:
    - ERC-721 tracks the ownership of each token and allows transferring it between accounts.
    - It ensures that only the rightful owner can transfer a token unless explicit permission is granted.
3. **Metadata**:
    - NFTs have metadata that provides additional information about the token (e.g., artwork details, game character stats, or a property deed). This data is often stored off-chain on systems like IPFS.
4. **Use Cases**:
    - **Digital Art**: Artists can tokenize their creations and sell them directly as NFTs.
    - **Gaming**: Unique game characters, skins, or weapons can be represented as ERC-721 tokens.
    - **Real Estate**: ERC-721 can be used to tokenize property deeds, enabling fractional ownership.
5. **Key Properties of ERC-721:**
    - **`ownerOf(uint256 tokenId)`**: Returns the owner of a specific NFT.
    - **`safeTransferFrom(address from, address to, uint256 tokenId)`**: Safely transfers ownership of an NFT.
    - **`approve(address approved, uint256 tokenId)`**: Approves another address to transfer a specific NFT.
    - **`getApproved(uint256 tokenId)`**: Gets the approved address for an NFT.

### Challenges:

1. **High Gas Costs**: Each NFT transfer is a separate transaction, leading to high fees in busy networks.
2. **Metadata Storage**: Since metadata is stored off-chain, token value is partially dependent on the reliability of external systems.

---

## **3. ERC-1155: Hybrid Tokens**

### Overview

ERC-1155 is a **multi-token standard** that supports fungible, non-fungible, and even semi-fungible tokens in a single contract. It was designed to overcome the inefficiencies of ERC-20 and ERC-721.

### Key Concepts:

1. **Fungibility Flexibility**:
    - A single ERC-1155 contract can manage multiple token types.
    - For example, fungible tokens like "gold coins" and non-fungible tokens like "a rare sword" can coexist in the same contract.
2. **Batch Operations**:
    - ERC-1155 supports batch transfers, allowing multiple token types to be sent in a single transaction. This drastically reduces gas costs.
    - Useful for games where players often deal with multiple items simultaneously.
3. **Token Metadata**:
    - Metadata is defined using placeholders (e.g., `{id}`) for each token type, making it easier to manage various tokens with shared metadata structures.
4. **Use Cases**:
    - **Gaming**: A game can use ERC-1155 to manage fungible tokens like gold and non-fungible tokens like unique weapons.
    - **Marketplaces**: Digital art platforms can use ERC-1155 to handle diverse NFT collections.
    - **DeFi**: Enables efficient transfer of multiple token types in a single operation.

### Advantages over ERC-20 and ERC-721:

1. **Gas Efficiency**: By batching operations, ERC-1155 reduces transaction costs.
2. **Flexibility**: A single smart contract can represent a broad range of token types.
3. **Scalability**: Suitable for applications with high-frequency token transfers or diverse asset types.

---

## **Comparison Table**

| Feature | ERC-20 (Fungible) | ERC-721 (Non-Fungible) | ERC-1155 (Hybrid) |
| --- | --- | --- | --- |
| **Token Type** | Fully fungible | Fully non-fungible | Both fungible and non-fungible |
| **Key Property** | Interchangeable and divisible | Unique and indivisible | Flexible token type management |
| **Metadata** | Not applicable | Unique for each token | Shared and flexible |
| **Use Cases** | Cryptocurrencies, DeFi | Digital art, gaming, real estate | Gaming, marketplaces, DeFi |
| **Batching** | Not supported | Not supported | Supported |
| **Gas Efficiency** | Moderate | Expensive for multiple tokens | Highly efficient |

---

## **Summary**

1. **ERC-20** is critical for fungible assets like cryptocurrencies and tokens in DeFi.
2. **ERC-721** revolutionized the world of digital ownership and collectibles with unique assets.
3. **ERC-1155** merges the best of both worlds, offering scalability and efficiency, particularly for gaming and complex marketplaces.