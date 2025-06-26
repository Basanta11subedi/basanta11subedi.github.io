---
title: " Token Vault (clarity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

```
(define-map balances { account: principal } uint)

;; #[allow(unchecked_data)]
(define-public (deposit (amount uint))
  (let ((current (default-to u0 (map-get? balances { account: tx-sender }))))
    (begin
      (map-set balances { account: tx-sender } (+ current amount))
      (ok true)
    )
  )
)

(define-public (withdraw (amount uint))
  (let ((current (default-to u0 (map-get? balances { account: tx-sender }))))
    (if (>= current amount)
        (begin
          (map-set balances { account: tx-sender } (- current amount))
          (ok true)
        )
        (err u402) ;; insufficient funds
    )
  )
)
```


### 🧠 Brief Overview

- **Purpose**: Track deposits and withdrawals of users on-chain.
- **Data Storage**: Each user's balance is stored using a map.
- **Functions**:
    - `deposit`: Add tokens to the sender’s balance.
    - `withdraw`: Subtract tokens if the sender has enough.

---

## 📄 Code Breakdown

### 🗃️ 1. Define the Balance Map

```

(define-map balances { account: principal } uint)

```

- A **map** named `balances` stores user balances.
- Key: `{ account: principal }` — each user’s address.
- Value: `uint` — the unsigned integer balance.

---

### 💸 2. Deposit Function

```
(define-public (deposit (amount uint))
  (let ((current (default-to u0 (map-get? balances { account: tx-sender }))))
    (begin
      (map-set balances { account: tx-sender } (+ current amount))
      (ok true)
    )
  )
)

```

- **Public function** — any user can call it.
- Accepts an `amount` to deposit.
- Fetches the current balance:
    - If none exists, defaults to `0`.
- Adds the deposit amount to the current balance.
- Updates the map.
- Returns `(ok true)` to confirm success.

---

### 🏧 3. Withdraw Function

```
(define-public (withdraw (amount uint))
  (let ((current (default-to u0 (map-get? balances { account: tx-sender }))))
    (if (>= current amount)
        (begin
          (map-set balances { account: tx-sender } (- current amount))
          (ok true)
        )
        (err u402) ;; insufficient funds
    )
  )
)

```

- Also **public** — allows users to withdraw.
- Gets the user’s current balance.
- Checks: is the balance >= withdrawal amount?
    - ✅ If yes: subtracts and updates the balance.
    - ❌ If no: returns `(err u402)` — an error for insufficient funds.

---

## 🔄 Step-by-Step: How This Contract Works

Let’s walk through a real-world scenario:

### 🧍 User A wants to deposit 100 tokens:

1. Calls:

```
(deposit u100)

```

1. Contract:
    - Reads balance (e.g., `u0` if new).
    - Adds `100`.
    - Updates the map: `{ account: UserA } → 100`.

---

### 🏧 Now User A wants to withdraw 30 tokens:

1. Calls:

```
(withdraw u30)

```

1. Contract:
    - Checks balance: 100.
    - 100 ≥ 30 → allowed.
    - Subtracts 30 → new balance = 70.
    - Updates the map.

---

### ⛔ User A tries to withdraw 100 (but only has 70):

- Fails the check.
- Returns:

```

(err u402)

```

Which can be interpreted in your frontend as **“Insufficient Funds”**.

---

### 🧩 Real-World Analogy

Think of it like a **digital piggy bank** for each user:

- You drop in coins (`deposit`).
- You can take out coins if there's enough (`withdraw`).
- If it's empty or has too little, you get an error.

---

## 📘 Summary Table

| Component | Purpose |
| --- | --- |
| `balances` | Map storing user balances |
| `deposit` | Adds tokens to the user’s vault |
| `withdraw` | Removes tokens if user has enough |
| `err u402` | Error code for “Insufficient Funds” |