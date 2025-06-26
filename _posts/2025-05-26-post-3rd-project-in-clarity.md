---
title: " Message Board Contract (clarity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

```
(define-map messages { user: principal } {message: (string-utf8 100)})


(define-public (write-message (msg (string-utf8 100)))
  (begin
    (map-set messages { user: tx-sender } { message: msg })
    (ok true)
  )
)

(define-public (read-message (user principal))
  (match (map-get? messages { user: user })
    value (ok (get message value))
    (err u404)
  )
)

(define-public (read-message (user principal))
  (match (map-get? messages { user: user })
    value (ok (get message value))
    (err u404)
  )
)
```

This smart contract allows users to write a **single message** tied to their address (like a profile status), and allows others to **read those messages**. All data is stored **on-chain**, meaning it's transparent and permanent until overwritten.

---

### 🧠 Brief Overview

- **Purpose**: Store one personal message per user on the blockchain.
- **Data Storage**: A map that links user addresses to their messages.
- **Functions**:
    - `write-message`: Save/update your message.
    - `read-message`: Read another user’s message.
    - `see-message`: Low-level view of a user’s message record (for dev/debug).

---

## 🔍 Code Breakdown

### 🗃️ 1. Define the Message Map

```

(define-map messages { user: principal } {message: (string-utf8 100)})

```

- This map stores messages per user.
- Key: `{ user: principal }` — user’s blockchain address.
- Value: `{ message: (string-utf8 100) }` — a UTF-8 string up to 100 characters.

---

### ✍️ 2. Write a Message

```
(define-public (write-message (msg (string-utf8 100)))
  (begin
    (map-set messages { user: tx-sender } { message: msg })
    (ok true)
  )
)

```

- **Public function** that lets users write/update their message.
- Uses `tx-sender` to identify the writer.
- Stores `{ message: msg }` in the `messages` map.
- Returns `(ok true)` to confirm the update.

---

### 📖 3. Read Someone’s Message

```
(define-public (read-message (user principal))
  (match (map-get? messages { user: user })
    value (ok (get message value))
    (err u404)
  )
)

```

- Takes a `user` (their address) as input.
- Tries to get their message from the map.
- If found: extracts the `message` and returns it with `ok`.
- If not: returns `(err u404)` (message not found).

---

### 🔍 4. Developer View: See Raw Message Record

```
(define-read-only (see-message (recipient principal))
	(ok (map-get? messages recipient))
)

```

- Read-only helper function.
- Returns the **full record** stored in the map, not just the message string.
- Useful for debugging or exploring the raw map value (including key/field names).

---

## 🧭 Step-by-Step: How It Works

### 🧍 User A writes a message

```
(write-message "Hello, Stacks!")

```

- The contract saves:
    
    `{ user: UserA } → { message: "Hello, Stacks!" }`
    

---

### 🧑‍🤝‍🧑 User B reads User A's message

```
(read-message UserA)

```

- If User A has written a message: returns `ok "Hello, Stacks!"`.
- If not: returns `err u404`.

---

### 🧪 Developer reads raw data

```
(see-message UserA)

```

- Output:
    
    `ok (some { message: "Hello, Stacks!" })`
    
    or `ok none` if there's no message.
    

---

## 📘 Summary Table

| Component | Purpose |
| --- | --- |
| `messages` map | Stores one message per user |
| `write-message` | Lets users post or update their message |
| `read-message` | Lets anyone read another user’s message |
| `see-message` | Returns the full map entry (debug-style) |
| `err u404` | Error if no message found for a user |

---

## 🧩 Real-World Analogy

Imagine a global public board where:

- Each user gets **one sticky note** to post a message.
- Anyone can read your note.
- You can update it any time.
- Others can't change your note — only you can.