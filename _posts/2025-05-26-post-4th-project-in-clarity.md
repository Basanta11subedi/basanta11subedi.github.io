---
title: " Simple Voting Contract (clarity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

```
(define-map votes { candidate: (string-utf8 20) } uint)


;; #[allow(unchecked_data)]
(define-public (vote (candidate (string-utf8 20)))
  (let ((count (default-to u0 (map-get? votes { candidate: candidate }))))
    (begin
      (map-set votes { candidate: candidate } (+ count u1))
      (ok true)
    )
  )
)

(define-public (get-votes (candidate (string-utf8 20)))
  (ok (default-to u0 (map-get? votes { candidate: candidate })))
)
```

This contract allows users to **vote for candidates**, with votes stored **on-chain**. Anyone can cast a vote for a candidate by name, and anyone can check how many votes a candidate has received.

---

### 🧠 Brief Overview

- **Purpose**: Tally votes for candidates by name.
- **Storage**: A map that links each candidate’s name to their vote count.
- **Functions**:
    - `vote`: Increases vote count for a candidate.
    - `get-votes`: Returns the total votes for a candidate.

---

## 📄 Code Breakdown

### 🗃️ 1. Define the Votes Map

```
(define-map votes { candidate: (string-utf8 20) } uint)

```

- A **map** called `votes` tracks vote totals.
- Key: `{ candidate: string-utf8(20) }` — candidate name (max 20 chars).
- Value: `uint` — the number of votes received.

---

### ✅ 2. Cast a Vote

```
(define-public (vote (candidate (string-utf8 20)))
  (let ((count (default-to u0 (map-get? votes { candidate: candidate }))))
    (begin
      (map-set votes { candidate: candidate } (+ count u1))
      (ok true)
    )
  )
)

```

- **Public function** — anyone can call it to vote.
- Input: `candidate` — the name of the candidate being voted for.
- Reads current vote count using `map-get?`.
    - If none yet: defaults to `0` using `default-to`.
- Adds 1 to the vote count.
- Stores the new count in the map.
- Returns `(ok true)` as confirmation.

---

### 📊 3. Get Vote Count

```
(define-public (get-votes (candidate (string-utf8 20)))
  (ok (default-to u0 (map-get? votes { candidate: candidate })))
)

```

- Takes the `candidate` name as input.
- Fetches the number of votes from the map.
- If no votes exist yet, returns `0`.
- Always returns result wrapped in `ok`.

---

## 🧭 Step-by-Step: How It Works

Let’s walk through an example:

### 🧑 User A votes for "Alice":

```
(vote "Alice")

```

- The contract:
    - Checks if "Alice" has votes → returns `0` (if first vote).
    - Adds 1 → total becomes `1`.
    - Stores `{ candidate: "Alice" } → 1`.

---

### 🧑‍🤝‍🧑 User B votes for "Bob":

```
(vote "Bob")

```

- Same logic → `{ candidate: "Bob" } → 1`.

---

### 🧑 User A votes for "Alice" again:

```
(vote "Alice")

```

- Reads current count: `1`.
- Adds 1 → becomes `2`.
- Updates map.

---

### 📊 Check how many votes "Alice" has:

```
(get-votes "Alice")

```

- Returns:

```
(ok u2)

```

---

## 📘 Summary Table

| Function | Purpose |
| --- | --- |
| `votes` map | Stores each candidate’s vote count |
| `vote` | Increments a candidate’s vote count |
| `get-votes` | Retrieves vote count for any candidate |
| `default-to` | Ensures candidates with no votes return 0 |

---

## 🧩 Real-World Analogy

Think of a public election box:

- Each candidate’s name is a folder.
- Every time someone votes, a ballot goes into that folder.
- Anyone can check how many ballots (votes) are in any folder.