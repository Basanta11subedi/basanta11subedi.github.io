---
title: " Multiplayer Counter contract (clarity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

```

;; Multiplayer Counter contract

(define-map counters principal uint)

(define-read-only (get-count (who principal))
    (default-to u0 (map-get? counters who))
)

(define-public (count-up) 
    (ok (map-set counters tx-sender (+ (get-count tx-sender) u1))) 
)

```

### 🧠 Brief Overview

- **Purpose**: Each user (principal) gets their own unique counter.
- **Data storage**: A key-value map that stores each user's counter.
- **Functions**:
    - `get-count`: Read-only function to check a user’s count.
    - `count-up`: Public function to increase the sender's count by 1

---

### 🔍 Code Breakdown

```

;; Multiplayer Counter contract

(define-map counters principal uint)

```

- Declares a **map** named `counters`.
- Key: `principal` (the user's blockchain address).
- Value: `uint` (unsigned integer — the counter value).

---

```

(define-read-only (get-count (who principal))
    (default-to u0 (map-get? counters who))
)

```

- This is a **read-only** function. It does not modify blockchain state.
- Input: `who` — a user address.
- Action: Checks if the user has a counter.
    - If yes: returns the value.
    - If no: returns `u0` (default value 0).

---

```

(define-public (count-up)
    (ok (map-set counters tx-sender (+ (get-count tx-sender) u1)))
)

```

- A **public function** (anyone can call it).
- `tx-sender`: The address of the user calling this function.
- Gets the sender's current count using `get-count`.
- Adds 1 to it.
- Updates the `counters` map with the new value.
- Wraps the result in an `ok` response.

---

### 🧭 Step-by-Step: How This Contract Works

Let’s walk through an example interaction:

### 1. 🧍 User A checks their counter:

```

(get-count tx-sender)

```

- If User A is new, this returns `0` because there's no entry yet.

### 2. 🆙 User A calls `count-up`

```

(count-up)

```

- Contract fetches User A's current count (e.g., `0`).
- Adds 1 → new value = `1`.
- Stores it in the `counters` map: `counters[User A] = 1`.

### 3. 🔁 User A calls `count-up` again

- Now it fetches `1`, adds 1 → value becomes `2`, and updates the map.

### 4. 🧑‍🤝‍🧑 User B joins and starts at 0

- Their `get-count` will return `0`.
- They can now increment their own counter without affecting User A’s.

---

### 📘 Summary

| Function | Type | Description |
| --- | --- | --- |
| `counters` | Map | Stores each user's counter. |
| `get-count` | Read-only | Returns current count or 0 if not found. |
| `count-up` | Public | Increments the caller’s counter by 1. |

---

### 🧩 Real-World Analogy

Imagine a scoreboard where every player has their own slot. When a player hits a button, only their score increases. No one can touch your slot but you.