---
title: "  Semi Fungible token Example "
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

```

;; Simple SIP-013 Semi-Fungible Token Implementation
;; Clarity Version 3

(impl-trait 'ST1ZK4MRVTQQJMVAAJQWBV2WPQ87QV2851YCTHD7X.sip-013-trait-sft-standard.sip-013-trait)

;; Define native token for post condition support
(define-fungible-token sft-token)
(define-non-fungible-token sft-token-id {token-id: uint, owner: principal})

;; Constants
(define-constant CONTRACT-OWNER tx-sender)
(define-constant ERR-OWNER-ONLY (err u100))
(define-constant ERR-INSUFFICIENT-BALANCE (err u1))
(define-constant ERR-SAME-SENDER-RECIPIENT (err u2))
(define-constant ERR-ZERO-AMOUNT (err u3))
(define-constant ERR-NOT-AUTHORIZED (err u4))
(define-constant ERR-TOKEN-NOT-FOUND (err u404))

;; Storage
(define-map token-balances {token-id: uint, owner: principal} uint)
(define-map token-supplies uint uint)
(define-map token-decimals uint uint)
(define-map token-uris uint (string-ascii 256))

;; Read-only functions

;; Get balance of a specific token for a principal
(define-read-only (get-balance (token-id uint) (who principal))
  (ok (default-to u0 (map-get? token-balances {token-id: token-id, owner: who})))
)

;; Get overall balance across all tokens for a principal
(define-read-only (get-overall-balance (who principal))
  (ok (ft-get-balance sft-token who))
)

;; Get total supply of a specific token
(define-read-only (get-total-supply (token-id uint))
  (ok (default-to u0 (map-get? token-supplies token-id)))
)

;; Get overall supply across all tokens
(define-read-only (get-overall-supply)
  (ok (ft-get-supply sft-token))
)

;; Get decimals for a token
(define-read-only (get-decimals (token-id uint))
  (ok (default-to u0 (map-get? token-decimals token-id)))
)

;; Get token URI
(define-read-only (get-token-uri (token-id uint))
  (ok (map-get? token-uris token-id))
)

;; Private helper functions

;; Tag NFT for post condition support
(define-private (tag-nft-token-id (nft-token-id {token-id: uint, owner: principal}))
  (begin
    (and
      (is-some (nft-get-owner? sft-token-id nft-token-id))
      (try! (nft-burn? sft-token-id nft-token-id (get owner nft-token-id)))
    )
    (nft-mint? sft-token-id nft-token-id (get owner nft-token-id))
  )
)

;; Update token balance
(define-private (set-balance (token-id uint) (owner principal) (new-balance uint))
  (if (> new-balance u0)
    (map-set token-balances {token-id: token-id, owner: owner} new-balance)
    (map-delete token-balances {token-id: token-id, owner: owner})
  )
)

;; Public functions

;; Transfer tokens
(define-public (transfer (token-id uint) (amount uint) (sender principal) (recipient principal))
  (let
    (
      (sender-balance (unwrap-panic (get-balance token-id sender)))
    )
    ;; Validate inputs
    (asserts! (> amount u0) ERR-ZERO-AMOUNT)
    (asserts! (not (is-eq sender recipient)) ERR-SAME-SENDER-RECIPIENT)
    (asserts! (>= sender-balance amount) ERR-INSUFFICIENT-BALANCE)
    (asserts! (or (is-eq sender tx-sender) (is-eq sender contract-caller)) ERR-NOT-AUTHORIZED)
    
    ;; Update balances
    (set-balance token-id sender (- sender-balance amount))
    (set-balance token-id recipient (+ (unwrap-panic (get-balance token-id recipient)) amount))
    
    ;; Transfer fungible tokens for post conditions
    (try! (ft-transfer? sft-token amount sender recipient))
    
    ;; Tag NFTs for post conditions
    (try! (tag-nft-token-id {token-id: token-id, owner: sender}))
    (try! (tag-nft-token-id {token-id: token-id, owner: recipient}))
    
    ;; Emit transfer event
    (print {type: "sft_transfer", token-id: token-id, amount: amount, sender: sender, recipient: recipient})
    
    (ok true)
  )
)

;; Transfer with memo
(define-public (transfer-memo (token-id uint) (amount uint) (sender principal) (recipient principal) (memo (buff 34)))
  (begin
    (try! (transfer token-id amount sender recipient))
    (print memo)
    (ok true)
  )
)

;; Admin functions

;; Mint new tokens (only owner)
(define-public (mint (token-id uint) (amount uint) (recipient principal))
  (begin
    (asserts! (is-eq tx-sender CONTRACT-OWNER) ERR-OWNER-ONLY)
    (asserts! (> amount u0) ERR-ZERO-AMOUNT)
    
    ;; Update balance and supply
    (set-balance token-id recipient (+ (unwrap-panic (get-balance token-id recipient)) amount))
    (map-set token-supplies token-id (+ (unwrap-panic (get-total-supply token-id)) amount))
    
    ;; Mint fungible tokens for post conditions
    (try! (ft-mint? sft-token amount recipient))
    
    ;; Tag NFT for post conditions
    (try! (tag-nft-token-id {token-id: token-id, owner: recipient}))
    
    ;; Emit mint event
    (print {type: "sft_mint", token-id: token-id, amount: amount, recipient: recipient})
    
    (ok true)
  )
)

;; Burn tokens
(define-public (burn (token-id uint) (amount uint) (sender principal))
  (let
    (
      (sender-balance (unwrap-panic (get-balance token-id sender)))
    )
    (asserts! (> amount u0) ERR-ZERO-AMOUNT)
    (asserts! (>= sender-balance amount) ERR-INSUFFICIENT-BALANCE)
    (asserts! (or (is-eq sender tx-sender) (is-eq sender contract-caller)) ERR-NOT-AUTHORIZED)
    
    ;; Update balance and supply
    (set-balance token-id sender (- sender-balance amount))
    (map-set token-supplies token-id (- (unwrap-panic (get-total-supply token-id)) amount))
    
    ;; Burn fungible tokens for post conditions
    (try! (ft-burn? sft-token amount sender))
    
    ;; Tag NFT for post conditions
    (try! (tag-nft-token-id {token-id: token-id, owner: sender}))
    
    ;; Emit burn event
    (print {type: "sft_burn", token-id: token-id, amount: amount, sender: sender})
    
    (ok true)
  )
)

;; Set token metadata (only owner)
(define-public (set-token-uri (token-id uint) (uri (string-ascii 256)))
  (begin
    (asserts! (is-eq tx-sender CONTRACT-OWNER) ERR-OWNER-ONLY)
    (map-set token-uris token-id uri)
    (ok true)
  )
)

;; Set token decimals (only owner)
(define-public (set-decimals (token-id uint) (decimals uint))
  (begin
    (asserts! (is-eq tx-sender CONTRACT-OWNER) ERR-OWNER-ONLY)
    (map-set token-decimals token-id decimals)
    (ok true)
  )
)

```
## 🔧 **Contract Metadata and Setup**

### 1. `(impl-trait ...)`

```

(impl-trait 'ST1ZK4MRVTQQJMVAAJQWBV2WPQ87QV2851YCTHD7X.sip-013-trait-sft-standard.sip-013-trait)

```

> Implements a trait (interface/standard) defined elsewhere. This ensures this contract complies with SIP-013, Stacks' SFT standard.
> 

---

## 🪙 **Token Definitions**

### 2. `define-fungible-token` and `define-non-fungible-token`

```

(define-fungible-token sft-token)
(define-non-fungible-token sft-token-id {token-id: uint, owner: principal})

```

> sft-token: A fungible token for total supply/balance tracking.sft-token-id: An NFT used for tagging balances for post-conditions.Structure: Each has a token-id and owner.
> 

---

## ⚙️ **Constants**

```

(define-constant CONTRACT-OWNER tx-sender)

```

> Set the deployer as the contract owner.
> 

Other constants are **error codes**:

```

(define-constant ERR-OWNER-ONLY (err u100))
(define-constant ERR-INSUFFICIENT-BALANCE (err u1))
...
(define-constant ERR-TOKEN-NOT-FOUND (err u404))

```

> Used with asserts! to enforce logic rules.
> 

---

## 🧠 **Storage Maps**

```
(define-map token-balances {token-id: uint, owner: principal} uint)

```

> Tracks how many units of a token a user owns.
> 

```

(define-map token-supplies uint uint)

```

> Total supply for each token ID.
> 

```

(define-map token-decimals uint uint)

```

> Number of decimal places for each token ID.
> 

```

(define-map token-uris uint (string-ascii 256))

```

> Metadata URI (e.g., image or metadata file) for each token.
> 

---

## 📖 **Read-Only Functions**

These functions **don’t modify blockchain state** and can be called without paying fees.

### `get-balance`

```

(get-balance (token-id uint) (who principal))

```

> Returns how much of token-id a user owns.
> 

### `get-overall-balance`

```

(get-overall-balance (who principal))

```

> Returns balance of sft-token (total fungible tokens held by user).
> 

### `get-total-supply` / `get-overall-supply`

```

(get-total-supply (token-id uint))
(get-overall-supply)

```

> Total minted tokens per ID or across all.
> 

### `get-decimals`, `get-token-uri`

```

(get-decimals (token-id uint))
(get-token-uri (token-id uint))

```

> Get token metadata (decimals/URI).
> 

---

## 🧰 **Private Helper Functions**

### `tag-nft-token-id`

```

(tag-nft-token-id {token-id: uint, owner: principal})

```

> Used for post-condition support:
> 
1. Burn existing NFT if it exists.
2. Mint new one with the same data.

> This enables tracking ownership changes on-chain.
> 

### `set-balance`

```

(set-balance token-id owner new-balance)

```

> Updates balance map:
> 
- If balance > 0 → store it.
- If balance = 0 → delete it.

---

## 🔄 **Public Functions**

### ✅ `transfer`

```

(transfer token-id amount sender recipient)

```

> Transfers amount of a token ID:
> 
1. Validates input:
    - Amount must be > 0.
    - Sender ≠ recipient.
    - Sender has enough tokens.
    - Sender is authorized (must be `tx-sender` or `contract-caller`).
2. Updates balances.
3. Transfers associated fungible tokens.
4. Tags NFTs.
5. Logs event.

### ✅ `transfer-memo`

```

(transfer-memo token-id amount sender recipient memo)

```

> Same as transfer but logs a memo (buffer of 34 bytes).
> 

---

## 🏗️ **Admin Functions**

Only the **contract owner** (deployer) can use these.

### 🪙 `mint`

```

(mint token-id amount recipient)

```

> Mints new SFT units:
> 
1. Only owner can mint.
2. Increases balance and supply.
3. Mints corresponding fungible tokens.
4. Tags NFT.
5. Logs event.

### 🔥 `burn`

```

(burn token-id amount sender)

```

> Burns tokens:
> 
1. Validates ownership and amount.
2. Decreases balance and supply.
3. Burns corresponding fungible tokens.
4. Tags NFT.
5. Logs event.

### 🧾 `set-token-uri`

```

(set-token-uri token-id uri)

```

> Set metadata URI (only by owner).
> 

### 🧮 `set-decimals`

```

(set-decimals token-id decimals)

```

> Set number of decimal places (only by owner).
> 

---

### **1. What is SIP-013?**

A standard for **Semi-Fungible Tokens (SFTs)** that act like fungible tokens with distinct IDs.

### **2. Key Components of the Contract**

- Implements the SIP-013 trait.
- Uses both fungible and non-fungible logic.
- Stores balances, supplies, metadata.

### **3. Smart Features**

- Fungible/NFT tagging for post-conditions.
- Error handling and balance cleanup.
- Built-in mint, transfer, burn, and metadata management.
