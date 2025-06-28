---
title: "How to build NFT MarketPlace"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

## 1. Introduction

NFTs—short for *non-fungible tokens*—are digital assets with unique identifiers. Non-fungible means that the tokens are not equivalent to one another. NFTs are created using **Clarity smart contracts**, a secure and predictable language designed for high-stakes blockchain applications. The most widely used NFT standard on Stacks is the **SIP-009** (Stacks Improvement Proposal 009), which defines the basic functionality and metadata structure required for NFTs on the network.

An **NFT marketplace** is a digital platform where users can mint, buy, sell, and trade NFTs. These marketplaces serve as hubs for creators and collectors to exchange unique digital assets.

## 2. Prerequisites

1. Familiar with Clarity  
   [Clarity Book](https://book.clarity-lang.org/) or [Clarity Docs](https://docs.hiro.so/stacks/clarity)

2. Familiar with Stacks  
   [Stacks Docs](https://docs.stacks.co/) or [Stacks.js Docs](https://docs.hiro.so/stacks/stacks.js)

3. Familiar with NFTs and SIP-009  
   [SIP-009 NFT Standard](https://book.clarity-lang.org/ch10-01-sip009-nft-standard.html)


## 3. Implementing the NFT MarketPlace

To implement the NFT marketplace, I wrote two smart contracts. One is for minting and transfer NFTs using the NFT trait, and the other is for the marketplace, where users can list, unlist, buy, and transfer NFTs between accounts. I will explain all the code in detail and provide the contract address link so you can review it.

### 3.1 SimpleNFT.clar

```
;; SimpleNFT - A basic NF contract for stacks blockchain
;; implements sip-009 nft standard
;; clarity version- 3
```

- The contract name is SimpleNFt which implements sip-009 nft standard. The smart contract code is written in clarity version 3.

🔗 **Trait Implementation**

```jsx
(impl-trait 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM.nft-trait.nft-trait)

```

 **The SIP009 NFT trait on testnet**

```
    
  (define-trait sip009-nft-trait (
		
    ;; Last token ID, limited to uint range
		(get-last-token-id () (response uint uint))
    
    ;; URI for metadata associated with the token
    (get-token-uri (uint) (response (optional (string-ascii 256)) uint))

    ;; Owner of a given token identifier
    (get-owner (uint) (response (optional principal) uint))

    ;; Transfer from the sender to a new principal
    (transfer (uint principal principal) (response bool uint))
  )
)

```

- **Purpose**: Imports and implements the **SIP-009 NFT trait** (interface).
- This ensures the contract conforms to the NFT standard (required functions and behavior).

❗ **Error Constants**

```
;; errors
(define-constant ERR-NOT-AUTHORIZED (err u100))
(define-constant ERR-NOT-FOUND (err u101))
(define-constant ERR-METADATA-FROZEN (err u102))

```

- Predefined **error codes** to use throughout the contract for clarity and consistency.


📦 **State Variables**

```
;; Data variables
(define-data-var contract-owner principal tx-sender)
(define-data-var last-token-id uint u0)
(define-data-var metadata-frozen bool  false)
```
- `contract-owner`: Stores the address of the contract owner.
- `last-token-id`: Keeps track of the latest minted token ID.
- `metadata-frozen`: Prevents changes to metadata after it's locked.

🗺️ **Data Maps**

```
;; nft metadata mapping
(define-map token-uris uint (string-ascii 256))

;;token ownership
(define-map token-owners uint principal)
```

- `token-uris`: Maps token ID to its metadata URI .
- `token-owners`: Maps token ID to the wallet address (principal) that owns it.

## 📖 **Read-Only Functions**

These functions **don’t modify blockchain state**, only return data.

### `get-last-token-id`

```
;;get the last token id
(define-read-only (get-last-token-id)
    (ok (var-get last-token-id))
)
```

- Returns the latest token ID minted.

### `get-token-uri`

```
;;get token uri
(define-read-only (get-token-uri (token-id uint))
    (ok (map-get? token-uris token-id))
```

- Returns the metadata URI for a specific token.

### `get-owner`

```
;; get the owner of the specified token
(define-read-only (get-owner (token-id uint))
    (ok (map-get? token-owners token-id))
)
```

- Returns the owner of a specific token.

### `get-total-supply`

```
;get total supply
(define-read-only (get-total-supply)
  (ok (var-get last-token-id))
)
```

- Since token IDs are sequential, this returns the total NFTs minted.

## 🚀 **Public Functions**

These **change state** on the blockchain.

### `mint`

```
;; mint nft
(define-public (mint (metadata-uri (string-ascii 256)))
  (let (
      (token-id (+ (var-get last-token-id) u1))
      (owner tx-sender)
    )
    (var-set last-token-id token-id)
    (map-set token-owners token-id owner)
    (map-set token-uris token-id metadata-uri)
    ;; (try! (nft-mint? asset-name asset-identifier recipient))
    (ok token-id)
  )
)
```

- **Mints a new NFT**:
    - Increments token ID.
    - Sets the sender as the owner.
    - Stores the metadata URI.
    - Returns the new token ID.

### `transfer`

```
(define-public (transfer (token-id uint) (sender principal) (recipient principal))
  (begin
    (asserts! (is-eq sender tx-sender) ERR-NOT-AUTHORIZED)
    (asserts! (is-some (map-get? token-owners token-id)) ERR-NOT-FOUND)
    (asserts! (is-eq (some sender) (map-get? token-owners token-id)) ERR-NOT-AUTHORIZED)
    
    (ok (map-set token-owners token-id recipient))
    
  )
)
```

- Transfers a token from one user to another.
- Includes checks to ensure:
    - Caller is the `sender`.
    - Token exists.
    - `sender` is actually the owner.

### `set-token-uri`

```
;; Metadata Management Functions
(define-public (set-token-uri (token-id uint) (new-uri (string-ascii 256)))
  (begin
    (asserts! (not (var-get metadata-frozen)) ERR-METADATA-FROZEN)
    (asserts! (is-some (map-get? token-owners token-id)) ERR-NOT-FOUND)
    (asserts! (is-eq (some tx-sender) (map-get? token-owners token-id)) ERR-NOT-AUTHORIZED)
    
    (map-set token-uris token-id new-uri)
    (ok true)
  )
)
```

- Allows the owner of a token to update its metadata URI.
- Only works **if metadata is not frozen**.

### `freeze-metadata`

```
(define-public (freeze-metadata)
  (begin
    (asserts! (is-eq tx-sender (var-get contract-owner tx-sender)) ERR-NOT-AUTHORIZED) ;; 
    (var-set metadata-frozen true)
    (ok true)
  )
)
```

- Freezes metadata updates globally.
- Prevents future `set-token-uri` calls.


## SimpleNFT

```
;; SimpleNFT - A basic NF contract for stacks blockchain
;; implements sip-009 nft standard
;; clarity version- 3

(impl-trait 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM.nft-trait.nft-trait)

;; errors
(define-constant ERR-NOT-AUTHORIZED (err u100))
(define-constant ERR-NOT-FOUND (err u101))
(define-constant ERR-METADATA-FROZEN (err u102))

;; Data variables
(define-data-var last-token-id uint u0)
(define-data-var metadata-frozen bool  false)

;; nft metadata mapping
(define-map token-uris uint (string-ascii 256))

;;token ownership
(define-map token-owners uint principal)

;;SIP-009 functions
;; read only functions
;;get the last token id
(define-read-only (get-last-token-id)
    (ok (var-get last-token-id))
)

;;get token uri
(define-read-only (get-token-uri (token-id uint))
    (ok (map-get? token-uris token-id))
)

;; get the owner of the specified token
(define-read-only (get-owner (token-id uint))
    (ok (map-get? token-owners token-id))
)

;;get total supply
(define-read-only (get-total-supply)
  (ok (var-get last-token-id))
)

;;public functions

;; mint nft
(define-public (mint (metadata-uri (string-ascii 256)))
  (let (
      (token-id (+ (var-get last-token-id) u1))
      (owner tx-sender)
    )
    (var-set last-token-id token-id)
    (map-set token-owners token-id owner)
    (map-set token-uris token-id metadata-uri)
    ;; (try! (nft-mint? asset-name asset-identifier recipient))
    (ok token-id)
  )
)

(define-public (transfer (token-id uint) (sender principal) (recipient principal))
  (begin
    (asserts! (is-eq sender tx-sender) ERR-NOT-AUTHORIZED)
    (asserts! (is-some (map-get? token-owners token-id)) ERR-NOT-FOUND)
    (asserts! (is-eq (some sender) (map-get? token-owners token-id)) ERR-NOT-AUTHORIZED)
    
    (ok (map-set token-owners token-id recipient))
    
  )
)

;; Metadata Management Functions
(define-public (set-token-uri (token-id uint) (new-uri (string-ascii 256)))
  (begin
    (asserts! (not (var-get metadata-frozen)) ERR-METADATA-FROZEN)
    (asserts! (is-some (map-get? token-owners token-id)) ERR-NOT-FOUND)
    (asserts! (is-eq (some tx-sender) (map-get? token-owners token-id)) ERR-NOT-AUTHORIZED)
    
    (map-set token-uris token-id new-uri)
    (ok true)
  )
)

(define-public (freeze-metadata)
  (begin
    (asserts! (is-eq tx-sender tx-sender) ERR-NOT-AUTHORIZED) ;; Replace with your owner check if needed
    (var-set metadata-frozen true)
    (ok true)
  )
)
```
- Contract address: ST390VFVZJA4WP7QSZN0RTSGQDAG2P9NPN3X1ATDX.SimpleNFT

### 3.2 MarketPlace.clar

### 🔒 Contract-Level Constants

```jsx
(define-constant contract-owner tx-sender)
(define-constant PLATFORM-FEE-BPS  u250) ;;2.5% basic fee for now
```

- Sets the **contract owner** at deployment time (you can replace with a specific address for real-world use).
- Used to restrict sensitive actions (like withdrawing fees).
- 2.5% platform fee defined in **basis points** (BPS).
- 250 BPS = 2.5% of the sale price.

---

### ❗ Error Constants

These are predefined **error codes** for common issues:

```
;;ERROR
(define-constant ERR-NOT-AUTHORIZED (err u401))
(define-constant ERR-LISTING-EXPIRED (err u402))
(define-constant ERR-PRICE-MISMATCHED (err u403))
(define-constant ERR-ALREADY-LISTED (err u404))
(define-constant ERR-NFT-TRANSFER-FAILED (err u405))
(define-constant ERR-NOT-FOUND (err u406))
(define-constant ERR-OWNER-CANNOT-BUY (err u407))
(define-constant ERR-INSUFFICIENT-FUNDS (err u408))
```

---

### 🗺️ Data Structures

```
;;Data structures
(define-map listings {token-id: uint} 
    {seller: principal, price: uint, expiry: uint, is-active: bool}
)
```

- Tracks all **NFT listings**.
- Keys: `token-id`, Values: seller info, price, expiry block, and listing status.

```
;;Track paltform fee
(define-data-var total-platform-fees  uint u0)
```

- Accumulates total platform fees collected from sales.

---

## 🧠 Read-Only Functions

### ✅ `is-listed`

```
;check if token is listed
(define-read-only (is-listed (token-id uint)) 
    (match (map-get? listings { token-id: token-id})
        listing (and (get is-active listing)
            (< stacks-block-height (get expiry listing)))
        false
    )
)
```

**Checks if a token is currently listed and hasn’t expired.**

Returns `true` if:

- Token is in the map,
- `is-active` is true,
- `expiry` is greater than current block height.

---

### 📜 `get-listing`

```
;;get listing details if active
(define-read-only (get-listing (token-id uint))
    (map-get? listings {token-id: token-id})
)

```

**Fetches full listing info for a token if it exists.**

Returns:

- Listing object if found (even if expired/inactive),
- `none` otherwise.

---

### 💸 `calculate-platform-fee`

```
;;calculate fees for the given amount
(define-read-only (calculate-platform-fee (amount uint)) 
    (/ (* amount PLATFORM-FEE-BPS) u10000)
)
```

- Calculates platform fee from total price.
- Example: If `amount = 1000`, fee = 2.5% = 25.

---

### 💰 `get-total-platform-fees`

```
;;get total platform fee of the platform
(define-read-only (get-total-platform-fees) 
    (var-get total-platform-fees)
)
```

- Returns the accumulated fees collected by the platform so far.

---

## 🛍️ Listing Functions

---

### 📦 `list-nft`

```
;; list nft for sale
(define-public (list-nft (token-id uint) (price uint) (expiry uint))
  (let (
    (nft-owner-opt (unwrap! (contract-call? .SimpleNFT get-owner token-id) (err u0)))
  )
    ;; Check if the owner is none
    (asserts! (is-some nft-owner-opt) ERR-NOT-AUTHORIZED)
    (let (
      (nft-owner (unwrap! nft-owner-opt (err u0)))
    )
      ;; Validations
      (asserts! (is-eq tx-sender nft-owner) ERR-NOT-AUTHORIZED)
      (asserts! (> expiry stacks-block-height) ERR-LISTING-EXPIRED)
      (asserts! (> price u0) ERR-PRICE-MISMATCHED)
      (asserts! (not (is-listed token-id)) ERR-ALREADY-LISTED)

      ;; Transfer NFT to this contract (escrow)
      (unwrap! (contract-call? .SimpleNFT transfer token-id tx-sender (as-contract tx-sender)) ERR-NFT-TRANSFER-FAILED)
      
      ;; Create the listing
      (map-set listings
        { token-id: token-id }
        { 
          seller: tx-sender, 
          price: price, 
          expiry: expiry, 
          is-active: true
        }
      )
      
      (ok true)
    )
  )
)
```

**Lists a token for sale and transfers NFT to escrow (Marketplace contract).**

Includes checks to ensure:

- Token exists and caller is the owner (via `SimpleNFT.get-owner`).
- Price is positive.
- Expiry is in the future.
- Token isn’t already listed.
- NFT is transferred to this contract via `SimpleNFT.transfer`.

Stores listing in the `listings` map as active.

---

### ❌ `cancel-listing`

```
;;cancel a listings
(define-public (cancel-listing (token-id uint))
    (let (
        (listing (unwrap! (map-get? listings { token-id: token-id}) ERR-NOT-FOUND))
    )   
    ;;only seller can cancel
    (asserts! (is-eq tx-sender (get seller listing)) ERR-NOT-AUTHORIZED)
    ;;ensure listing is active
    (asserts! (get is-active listing) ERR-NOT-FOUND)

    ;;Mark lisitng is inactive
    (map-set listings {token-id: token-id} (merge listing {is-active: false}))

    ;;Return nft to seller from escrow
    (as-contract (contract-call? .SimpleNFT transfer token-id tx-sender (get seller listing))))
)
```

**Cancels an active listing and returns NFT to seller.**

Includes:

- Validation that caller is the original seller.
- Checks if listing is still active.
- Marks the listing inactive.
- Transfers NFT from contract back to seller using `SimpleNFT.transfer`.

---

## 💵 Purchase Functions

---

### 🛒 `buy-nft`

```
;; buy listed nft
(define-public (buy-nft (token-id uint))
  (let (
    (listing (unwrap! (map-get? listings { token-id: token-id }) ERR-NOT-FOUND))
    (price (get price listing))
    (seller (get seller listing))
    (platform-fee (calculate-platform-fee price))
    (seller-amount (- price platform-fee))
  )
    ;; Check conditions
    (asserts! (get is-active listing) ERR-NOT-FOUND)
    (asserts! (< stacks-block-height (get expiry listing)) ERR-LISTING-EXPIRED)
    (asserts! (not (is-eq tx-sender seller)) ERR-OWNER-CANNOT-BUY)
    
    ;; Update platform fees
    (var-set total-platform-fees (+ (var-get total-platform-fees) platform-fee))
    
    ;; Mark listing as inactive
    (map-set listings
      { token-id: token-id }
      (merge listing { is-active: false })
    )
    
    ;; Pay seller
    (unwrap! (stx-transfer? seller-amount tx-sender seller) ERR-INSUFFICIENT-FUNDS)
    
    ;; Pay platform fee
    (unwrap! (stx-transfer? platform-fee tx-sender contract-owner) ERR-INSUFFICIENT-FUNDS)
    
    ;; Transfer NFT to buyer from escrow
    (as-contract 
      (contract-call? .SimpleNFT transfer token-id tx-sender tx-sender)
    )
  )
)
```

**Allows a buyer to purchase a listed NFT.**

Includes:

- Checks that listing is active, not expired, and buyer isn’t seller.
- Calculates fee and seller amount.
- Adds fee to `total-platform-fees`.
- Transfers STX to seller and platform.
- Transfers NFT to buyer from contract using `SimpleNFT.transfer`.

### 💲 `update-listing-price`

```
;; Update a listing's price
(define-public (update-listing-price (token-id uint) (new-price uint))
  (let (
    (listing (unwrap! (map-get? listings { token-id: token-id }) ERR-NOT-FOUND))
  )
    ;; Validations
    (asserts! (is-eq tx-sender (get seller listing)) ERR-NOT-AUTHORIZED)
    (asserts! (get is-active listing) ERR-NOT-FOUND)
    (asserts! (< stacks-block-height (get expiry listing)) ERR-LISTING-EXPIRED)
    (asserts! (> new-price u0) ERR-PRICE-MISMATCHED)
    
    ;; Update the listing price
    (map-set listings
      { token-id: token-id }
      (merge listing { price: new-price })
    )
    
    (ok true)
  )
)
```

**Updates the price of an active, unexpired listing.**

Validates:

- Caller is the seller.
- Listing is still active and not expired.
- New price is positive.

Then updates listing with the new price.

---

## 🏦 Admin Function

---

### 💼 `withdraw-platform-fee`

```
;; Administration function to withdraw paltform fee

(define-public (withdraw-platform-fee) 
 (let (
    (fee-amount (var-get total-platform-fees))
 )
 (asserts! (is-eq tx-sender contract-owner) ERR-NOT-AUTHORIZED)
 (asserts! (> fee-amount u0) ERR-INSUFFICIENT-FUNDS)

 ;;Reset counter
 (var-set total-platform-fees u0)

 ;; transfer funds to contract owner
 (as-contract (stx-transfer? fee-amount tx-sender contract-owner))
 )
)
```

**Allows the contract owner to withdraw collected platform fees.**

Includes:

- Validates caller is the contract owner.
- Fee amount is greater than zero.
- Transfers total fees to contract owner.
- Resets fee counter to zero.

## MarketPlace code

```
;; Makrketplace contract where users can list, unlist and can sell their nfts
;; clarity version 3

(define-constant contract-owner tx-sender)
(define-constant PLATFORM-FEE-BPS  u250) ;;2.5% basic fee for now

;;eRROR
(define-constant ERR-NOT-AUTHORIZED (err u401))
(define-constant ERR-LISTING-EXPIRED (err u402))
(define-constant ERR-PRICE-MISMATCHED (err u403))
(define-constant ERR-ALREADY-LISTED (err u404))
(define-constant ERR-NFT-TRANSFER-FAILED (err u405))
(define-constant ERR-NOT-FOUND (err u406))
(define-constant ERR-OWNER-CANNOT-BUY (err u407))
(define-constant ERR-INSUFFICIENT-FUNDS (err u408))

;;Data structures
(define-map listings {token-id: uint} 
    {seller: principal, price: uint, expiry: uint, is-active: bool}
)

;;Track paltform fee
(define-data-var total-platform-fees  uint u0)

;;check if token is listed
(define-read-only (is-listed (token-id uint)) 
    (match (map-get? listings { token-id: token-id})
        listing (and (get is-active listing)
            (< stacks-block-height (get expiry listing)))
        false
    )
)

;;get listing details if active
(define-read-only (get-listing (token-id uint))
    (map-get? listings {token-id: token-id})
)

;;calculate fees for the given amount
(define-read-only (calculate-platform-fee (amount uint)) 
    (/ (* amount PLATFORM-FEE-BPS) u10000)
)
;;get total platform fee of the platform
(define-read-only (get-total-platform-fees) 
    (var-get total-platform-fees)
)

;;listing functions
;; list nft for sale
(define-public (list-nft (token-id uint) (price uint) (expiry uint))
  (let (
    (nft-owner-opt (unwrap! (contract-call? .SimpleNFT get-owner token-id) (err u0)))
  )
    ;; Check if the owner is none
    (asserts! (is-some nft-owner-opt) ERR-NOT-AUTHORIZED)
    (let (
      (nft-owner (unwrap! nft-owner-opt (err u0)))
    )
      ;; Validations
      (asserts! (is-eq tx-sender nft-owner) ERR-NOT-AUTHORIZED)
      (asserts! (> expiry stacks-block-height) ERR-LISTING-EXPIRED)
      (asserts! (> price u0) ERR-PRICE-MISMATCHED)
      (asserts! (not (is-listed token-id)) ERR-ALREADY-LISTED)

      ;; Transfer NFT to this contract (escrow)
      (unwrap! (contract-call? .SimpleNFT transfer token-id tx-sender (as-contract tx-sender)) ERR-NFT-TRANSFER-FAILED)
      
      ;; Create the listing
      (map-set listings
        { token-id: token-id }
        { 
          seller: tx-sender, 
          price: price, 
          expiry: expiry, 
          is-active: true
        }
      )
      
      (ok true)
    )
  )
)

;;cancel a listings
(define-public (cancel-listing (token-id uint))
    (let (
        (listing (unwrap! (map-get? listings { token-id: token-id}) ERR-NOT-FOUND))
    )   
    ;;only seller can cancel
    (asserts! (is-eq tx-sender (get seller listing)) ERR-NOT-AUTHORIZED)
    ;;ensure listing is active
    (asserts! (get is-active listing) ERR-NOT-FOUND)

    ;;Mark lisitng is inactive
    (map-set listings {token-id: token-id} (merge listing {is-active: false}))

    ;;Return nft to seller from escrow
    (as-contract (contract-call? .SimpleNFT transfer token-id tx-sender (get seller listing))))
)

;; Purchase functions

;; buy listed nft
(define-public (buy-nft (token-id uint))
  (let (
    (listing (unwrap! (map-get? listings { token-id: token-id }) ERR-NOT-FOUND))
    (price (get price listing))
    (seller (get seller listing))
    (platform-fee (calculate-platform-fee price))
    (seller-amount (- price platform-fee))
  )
    ;; Check conditions
    (asserts! (get is-active listing) ERR-NOT-FOUND)
    (asserts! (< stacks-block-height (get expiry listing)) ERR-LISTING-EXPIRED)
    (asserts! (not (is-eq tx-sender seller)) ERR-OWNER-CANNOT-BUY)
    
    ;; Update platform fees
    (var-set total-platform-fees (+ (var-get total-platform-fees) platform-fee))
    
    ;; Mark listing as inactive
    (map-set listings
      { token-id: token-id }
      (merge listing { is-active: false })
    )
    
    ;; Pay seller
    (unwrap! (stx-transfer? seller-amount tx-sender seller) ERR-INSUFFICIENT-FUNDS)
    
    ;; Pay platform fee
    (unwrap! (stx-transfer? platform-fee tx-sender contract-owner) ERR-INSUFFICIENT-FUNDS)
    
    ;; Transfer NFT to buyer from escrow
    (as-contract 
      (contract-call? .SimpleNFT transfer token-id tx-sender tx-sender)
    )
  )
)

;; Update a listing's price
(define-public (update-listing-price (token-id uint) (new-price uint))
  (let (
    (listing (unwrap! (map-get? listings { token-id: token-id }) ERR-NOT-FOUND))
  )
    ;; Validations
    (asserts! (is-eq tx-sender (get seller listing)) ERR-NOT-AUTHORIZED)
    (asserts! (get is-active listing) ERR-NOT-FOUND)
    (asserts! (< stacks-block-height (get expiry listing)) ERR-LISTING-EXPIRED)
    (asserts! (> new-price u0) ERR-PRICE-MISMATCHED)
    
    ;; Update the listing price
    (map-set listings
      { token-id: token-id }
      (merge listing { price: new-price })
    )
    
    (ok true)
  )
)

;; Administration function to withdraw paltform fee

(define-public (withdraw-platform-fee) 
 (let (
    (fee-amount (var-get total-platform-fees))
 )
 (asserts! (is-eq tx-sender contract-owner) ERR-NOT-AUTHORIZED)
 (asserts! (> fee-amount u0) ERR-INSUFFICIENT-FUNDS)

 ;;Reset counter
 (var-set total-platform-fees u0)

 ;; transfer funds to contract owner
 (as-contract (stx-transfer? fee-amount tx-sender contract-owner))
 )
)
```
- Contract address: ST390VFVZJA4WP7QSZN0RTSGQDAG2P9NPN3X1ATDX.MarketPlace