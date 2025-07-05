---
title: "Learning Clarity Functions"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

### 🔢 Arithmetic & Math

### 1. **add**

Adds two numbers.

**Example:** `(add 2 3)` → `5`

### 2. **subtract**

Subtracts the second number from the first.

**Example:** `(subtract 5 2)` → `3`

### 3. **multiply**

Multiplies two numbers.

**Example:** `(multiply 4 3)` → `12`

### 4. **divide**

Divides the first number by the second. Integer division.

**Example:** `(divide 10 2)` → `5`

### 5. **mod**

Returns the remainder after division.

**Example:** `(mod 7 3)` → `1`

### 6. **pow**

Raises a number to the power of another.

**Example:** `(pow 2 3)` → `8`

---

### 🧠 Logical Operators

### 7. **and**

Returns true if all expressions are true.

**Example:** `(and true true)` → `true`

### 8. **or**

Returns true if any expression is true.

**Example:** `(or false true)` → `true`

### 9. **not**

Logical negation.

**Example:** `(not false)` → `true`

---

### 🔍 Comparisons

### 10. **greater than (`>`)**

Checks if the first number is greater.

**Example:** `(> 5 3)` → `true`

### 11. **greater than or equal (`>=`)**

Checks if the first number is greater or equal.

**Example:** `(>= 5 5)` → `true`

### 12. **less than (`<`)**

Checks if the first number is less.

**Example:** `(< 2 5)` → `true`

### 13. **less than or equal (`<=`)**

Checks if the first number is less or equal.

**Example:** `(<= 2 2)` → `true`

---

### 🧩 Bitwise Operators

### 14. **bit-and**

Bitwise AND between integers.

**Example:** `(bit-and 3 6)` → `2`

### 15. **bit-or**

Bitwise OR between integers.

**Example:** `(bit-or 3 6)` → `7`

### 16. **bit-not**

Bitwise NOT (inverts bits).

**Example:** `(bit-not 0)` → `-1`

### 17. **bit-xor**

Bitwise exclusive OR.

**Example:** `(bit-xor 3 6)` → `5`

### 18. **bit-shift-left**

Shifts bits left.

**Example:** `(bit-shift-left 1 u2)` → `4`

### 19. **bit-shift-right**

Shifts bits right.

**Example:** `(bit-shift-right 4 u1)` → `2`

---

### 🧮 Integer Conversion

### 20. **buff-to-int-be**

Converts a buffer to signed integer (big-endian).

**Example:** `(buff-to-int-be 0x00000001)` → `1`

### 21. **buff-to-int-le**

Buffer to signed integer (little-endian).

**Example:** `(buff-to-int-le 0x01000000)` → `1`

### 22. **buff-to-uint-be**

Buffer to unsigned integer (big-endian).

**Example:** `(buff-to-uint-be 0x00000001)` → `u1`

### 23. **buff-to-uint-le**

Buffer to unsigned integer (little-endian).

**Example:** `(buff-to-uint-le 0x01000000)` → `u1`

---

### 📚 Buffers & Lists

### 24. **append**

Adds an element to a list.

**Example:** `(append (list 1 2) 3)` → `(1 2 3)`

### 25. **concat**

Concatenates two lists or buffers.

**Example:** `(concat (list 1 2) (list 3))` → `(1 2 3)`

### 26. **len**

Returns the length of a list, string, or buffer.

**Example:** `(len (list 1 2 3))` → `3`

---

### ✅ Result Handling

### 27. **ok**

Wraps a value in an `ok` response.

**Example:** `(ok 42)` → `(ok 42)`

### 28. **err**

Wraps a value in an `err` response.

**Example:** `(err u401)` → `(err u401)`

### 29. **is-ok**

Returns `true` if input is `ok`.

**Example:** `(is-ok (ok u1))` → `true`

### 30. **is-err**

Returns `true` if input is `err`.

**Example:** `(is-err (err u1))` → `tru`

### 31. **is-some**

Returns true if the input is a `some` value.

**Example:** `(is-some (some u1))` → `true`

### 32. **is-none**

Returns true if the input is `none`.

**Example:** `(is-none none)` → `true`

### 33. **some**

Wraps a value in a `some`.

**Example:** `(some u10)` → `(some u10)`

### 34. **default-to**

Returns the value or a fallback if input is none.

**Example:** `(default-to u0 none)` → `u0`

---

### 🧵 List/Buffer/String Tools

### 35. **element-at?**

Gets element at index if exists.

**Example:** `(element-at? (list 10 20 30) u1)` → `(some 20)`

### 36. **index-of**

Returns the index of an element if found.

**Example:** `(index-of u2 (list u1 u2 u3))` → `(some u1)`

### 37. **replace-at?**

Replaces a value at a list index.

**Example:** `(replace-at? (list u1 u2 u3) u1 u9)` → `(some (list u1 u9 u3))`

### 38. **slice?**

Returns sublist between two indexes.

**Example:** `(slice? (list u1 u2 u3 u4) u1 u3)` → `(some (list u2 u3))`

---

### 🧰 Mapping, Filtering, Folding

### 39. **map**

Applies a function to each item in a list.

**Example:** `(map (lambda (x) (* x x)) (list 1 2 3))` → `(1 4 9)`

### 40. **filter**

Filters items by a condition.

**Example:** `(filter (lambda (x) (> x u1)) (list u1 u2 u3))` → `(2 3)`

### 41. **fold**

Reduces a list to a single value using a function.

**Example:** `(fold (lambda (x acc) (+ x acc)) (list 1 2 3) 0)` → `6`

---

### 🔐 Variable Binding & Scoping

### 42. **let**

Binds values to names locally.

**Example:**

```

(let ((x 3)) (* x x))

```

→ `9`

### 43. **begin**

Evaluates expressions sequentially and returns the last.

**Example:**

```

(begin (print "hi") (+ 1 1))

```

→ `2`

---

### 🧱 Constants & Variables

### 44. **define-constant**

Defines an immutable constant.

**Example:** `(define-constant max-supply u1000000)` → usable as `max-supply`

### 45. **define-data-var**

Creates a mutable data variable.

**Example:** `(define-data-var counter int 0)`

### 46. **var-get**

Reads a data variable.

**Example:** `(var-get counter)` → `0`

### 47. **var-set**

Updates a data variable.

**Example:** `(var-set counter 10)` → `10`

---

### 🧬 Tuples

### 48. **tuple**

Creates a named set of fields.

**Example:** `(tuple (name "clarity") (version u1))`

### 49. **get**

Accesses a field in a tuple.

**Example:** `(get name (tuple (name "clarity")))` → `"clarity"`

---

### 🧩 Contracts & Structure

### 50. **define-map**

Defines a named map for storing key-value pairs.

**Example:** `(define-map user-balances {user: principal} {balance: uint})`

### 51. **map-insert**

Inserts a key-value pair into a map.

**Example:** `(map-insert user-balances {user: tx-sender} {balance: u100})`

### 52. **map-set**

Updates an existing key in the map.

**Example:** `(map-set user-balances {user: tx-sender} {balance: u200})`

### 53. **map-delete**

Removes a key from the map.

**Example:** `(map-delete user-balances {user: tx-sender})`

### 54. **map-get?**

Retrieves a value from the map.

**Example:** `(map-get? user-balances {user: tx-sender})` → `(some {balance: u200})`

---

### 🧵 Pattern Matching

### 55. **match**

Pattern matches `some` or `none`.

**Example:**

```

(match (some u5)
  val val
  none u0)

```

→ `5`

### 56. **as-contract**

Returns the contract principal of the current contract.

**Example:** `(as-contract tx-sender)` → `SP...` (principal)

### 57. **as-max-len?**

Returns `some` if list or string length ≤ max, else `none`.

**Example:** `(as-max-len? (list 1 2 3) u5)` → `(some (list 1 2 3))`

### 58. **at-block**

Executes code at a specific block height.

**Example:** `(at-block u100 (ok u1))` → Runs `(ok u1)` at block 100

---

### ⛓ Contract Calls & Info

### 59. **contract-call?**

Calls a public function on another contract.

**Example:** `(contract-call? 'SP1.contract 'get-balance tx-sender)` → `(ok u100)`

### 60. **contract-of**

Returns the contract principal for a function call.

**Example:** `(contract-of 'SP1.contract)` → `SP1.contract`

### 61. **get-block-info?**

Fetches info about the current block.

**Example:** `(get-block-info?)` → Tuple with block data

### 62. **get-burn-block-info?**

Returns burn block info (bitcoin anchored).

**Example:** `(get-burn-block-info?)` → burn block tuple

### 63. **get-stacks-block-info?**

Returns Stacks chain block info.

**Example:** `(get-stacks-block-info?)` → stacks block tuple

---

### 🔢 Number Conversion & String Helpers

### 64. **int-to-ascii**

Converts integer to ASCII string.

**Example:** `(int-to-ascii 65)` → `"A"`

### 65. **int-to-utf8**

Converts integer to UTF-8 string.

**Example:** `(int-to-utf8 8364)` → `"€"`

### 66. **string-to-int?**

Parses string to integer, returns optional.

**Example:** `(string-to-int? "123")` → `(some 123)`

### 67. **string-to-uint?**

Parses string to unsigned integer optional.

**Example:** `(string-to-uint? "456")` → `(some u456)`

---

### 🔒 Cryptographic Hashes

### 68. **hash160**

Performs HASH160 (RIPEMD-160 of SHA-256).

**Example:** `(hash160 0x01)` → `0x...`

### 69. **keccak256**

Computes Keccak-256 hash.

**Example:** `(keccak256 0x01)` → `0x...`

### 70. **sha256**

Computes SHA-256 hash.

**Example:** `(sha256 0x01)` → `0x...`

### 71. **sha512**

Computes SHA-512 hash.

**Example:** `(sha512 0x01)` → `0x...`

### 72. **sha512/256**

Computes SHA-512/256 hash.

**Example:** `(sha512/256 0x01)` → `0x...`

---

### 📊 Fungible Token (FT) Functions

### 73. **define-fungible-token**

Defines a fungible token with a total supply.

**Example:** `(define-fungible-token my-token u1000)`

### 74. **ft-get-balance**

Returns FT balance of a principal.

**Example:** `(ft-get-balance? 'my-token tx-sender)` → `u500`

### 75. **ft-get-supply**

Returns total FT supply.

**Example:** `(ft-get-supply? 'my-token)` → `u1000`

### 76. **ft-transfer?**

Transfers FT tokens.

**Example:** `(ft-transfer? 'my-token u10 tx-sender 'SP2.recipient)` → `(ok true)`

### 77. **ft-mint?**

Mints FT tokens.

**Example:** `(ft-mint? 'my-token u100 tx-sender)` → `(ok true)`

### 78. **ft-burn?**

Burns FT tokens.

**Example:** `(ft-burn? 'my-token u50 tx-sender)` → `(ok true)`

### 79. **define-non-fungible-token**

Defines a new NFT type.

**Example:** `(define-non-fungible-token my-nft)`

### 80. **nft-mint?**

Mints a new NFT to a principal.

**Example:** `(nft-mint? my-nft u1 tx-sender)` → `(ok true)`

### 81. **nft-transfer?**

Transfers NFT to another principal.

**Example:** `(nft-transfer? my-nft u1 tx-sender 'SP2.recipient)` → `(ok true)`

### 82. **nft-get-owner?**

Returns the owner of an NFT.

**Example:** `(nft-get-owner? my-nft u1)` → `(some tx-sender)`

### 83. **nft-burn?**

Burns an NFT.

**Example:** `(nft-burn? my-nft u1 tx-sender)` → `(ok true)`

---

### 🔧 Contract Function Definition & Traits

### 84. **define-private**

Defines a private contract function.

**Example:** `(define-private (helper (x int)) (+ x u1))`

### 85. **define-public**

Defines a public callable function.

**Example:** `(define-public (add-one (x int)) (ok (+ x u1)))`

### 86. **define-read-only**

Defines a read-only function that can’t change state.

**Example:** `(define-read-only (get-counter) (var-get counter))`

### 87. **define-trait**

Defines a contract trait (interface).

**Example:** `(define-trait my-trait (print-hello ()))`

### 88. **impl-trait**

Implements a trait in a contract.

**Example:** `(impl-trait my-trait)`

### 89. **use-trait**

Uses a trait’s functions inside a contract.

**Example:** `(use-trait my-trait)`

---

### 🔄 Control Flow

### 90. **if**

Conditional branching.

**Example:** `(if (> x u5) (ok "big") (ok "small"))`

### 91. **match**

Pattern matching (already covered, but important).

**Example:**

```

(match some-val
  val (do-something val)
  none (handle-none))

```

---

### 🔐 Principals

### 92. **principal-construct?**

Creates a principal from strings.

**Example:** `(principal-construct? 'SP1' 'contract')` → principal

### 93. **principal-destruct?**

Extracts info from a principal.

**Example:** `(principal-destruct? tx-sender)` → tuple

### 94. **principal-of?**

Returns principal of the current contract.

**Example:** `(principal-of? 'contract-name)` → principal

---

### 🖨️ Logging & Debugging

### 95. **print**

Prints a value for debugging.

**Example:** `(print "Hello world")` → prints `"Hello world"`

---

### 🧩 Tuple Helpers

### 96. **merge**

Merges two tuples into one.

**Example:** `(merge (tuple (a u1)) (tuple (b u2)))` → `(tuple (a u1) (b u2))`

---

### 🔢 Numeric Helpers

### 97. **log2**

Computes base-2 logarithm (integer).

**Example:** `(log2 u8)` → `3`

### 98. **sqrti**

Integer square root.

**Example:** `(sqrti u16)` → `4`

---

### 🧩 Result Utilities

### 99. **unwrap!**

Unwraps an `ok` or aborts.

**Example:** `(unwrap! (ok u1))` → `u1`

### 100. **unwrap-err!**

Unwraps an `err` or aborts.

**Example:** `(unwrap-err! (err u1))` → `u1`

### 🔀 Control Flow & Error Handling

### 101. **try!**

Tries to unwrap an `ok` value or aborts with `err`.

**Example:** `(try! (ok u10))` → `u10`

---

### 🧩 List & Tuple Operations

### 102. **list**

Creates a list of values.

**Example:** `(list u1 u2 u3)` → `(1 2 3)`

### 103. **tuple**

Creates a tuple (key-value pairs).

**Example:** `(tuple (name "Alice") (age u30))`

---

### 🔄 Map Utilities

### 104. **map-delete**

Deletes a key from a map.

**Example:** `(map-delete my-map {key: u1})`

### 105. **map-get?**

Retrieves a value from a map if key exists.

**Example:** `(map-get? my-map {key: u1})` → `(some value)`

### 106. **map-insert**

Inserts a key-value pair into a map.

**Example:** `(map-insert my-map {key: u1} {value: u10})`

### 107. **map-set**

Updates the value for an existing key in a map.

**Example:** `(map-set my-map {key: u1} {value: u20})`

---

### 🔢 Integer Conversion & Buffer Helpers

### 108. **to-int**

Converts a value to an integer (signed).

**Example:** `(to-int u10)` → `10`

### 109. **to-uint**

Converts a value to an unsigned integer.

**Example:** `(to-uint 10)` → `u10`

### 110. **to-consensus-buff?**

Converts a value to a consensus buffer (used for signing).

**Example:** `(to-consensus-buff? u10)` → `0x0a`

### 111. **from-consensus-buff?**

Converts a consensus buffer to a value.

**Example:** `(from-consensus-buff? 0x0a)` → `u10`

---

### 🎭 Option & Result Utilities

### 112. **unwrap-panic**

Unwraps `ok` or panics.

**Example:** `(unwrap-panic (ok u5))` → `5`

### 113. **unwrap-err-panic**

Unwraps `err` or panics.

**Example:** `(unwrap-err-panic (err u1))` → `1`

---

### 📜 Traits & Modules

### 114. **use-trait**

Imports trait functions into contract.

**Example:** `(use-trait my-trait)`

---

### 🔐 STX Token Functions

### 115. **stx-get-balance**

Returns STX balance of an account.

**Example:** `(stx-get-balance tx-sender)` → `u1000`

### 116. **stx-transfer?**

Transfers STX tokens to another account.

**Example:** `(stx-transfer? u100 tx-sender 'SP2.recipient)` → `(ok true)`

### 117. **stx-transfer-memo?**

Transfers STX with a memo.

**Example:** `(stx-transfer-memo? u100 tx-sender 'SP2.recipient "Thanks!")` → `(ok true)`

### 118. **stx-burn?**

Burns STX tokens.

**Example:** `(stx-burn? u10 tx-sender)` → `(ok true)`

### 119. **stx-account**

Returns STX account info for a principal.

**Example:** `(stx-account tx-sender)` → account tuple

---

### 🔣 String Utilities

### 120. **print**

Prints a message to debug console.

**Example:** `(print "Debugging")`

### 121. **replace-at?**

Replaces an element in a list at an index.

**Example:** `(replace-at? (list u1 u2 u3) u1 u9)` → `(some (list u1 u9 u3))`

---

### ⚙️ Other Helpers

### 122. **default-to**

Returns a default value if input is `none`.

**Example:** `(default-to u0 none)` → `0`

### 123. **not**

Logical negation.

**Example:** `(not false)` → `true`

### 124. **xor**

Bitwise XOR between two numbers.

**Example:** `(xor u3 u6)` → `5`

### ⚙️ Bitwise Operations

### 125. **bit-and**

Performs bitwise AND on two integers.

**Example:** `(bit-and u6 u3)` → `2`

Explanation: `6 (110) AND 3 (011) = 2 (010)`

### 126. **bit-or**

Performs bitwise OR on two integers.

**Example:** `(bit-or u6 u3)` → `7`

Explanation: `6 (110) OR 3 (011) = 7 (111)`

### 127. **bit-xor**

Performs bitwise XOR on two integers.

**Example:** `(bit-xor u6 u3)` → `5`

Explanation: `6 (110) XOR 3 (011) = 5 (101)`

### 128. **bit-not**

Performs bitwise NOT on an integer.

**Example:** `(bit-not u6)` → `-7` (two’s complement)

### 129. **bit-shift-left**

Shifts bits left by n positions.

**Example:** `(bit-shift-left u3 u2)` → `12`

Explanation: `3 (0011) << 2 = 12 (1100)`

### 130. **bit-shift-right**

Shifts bits right by n positions.

**Example:** `(bit-shift-right u12 u2)` → `3`

Explanation: `12 (1100) >> 2 = 3 (0011)`

---

### 🧮 Arithmetic Operations

### 131. **add**

Adds two numbers.

**Example:** `(add u5 u10)` → `15`

### 132. **subtract**

Subtracts second number from first.

**Example:** `(subtract u10 u3)` → `7`

### 133. **multiply**

Multiplies two numbers.

**Example:** `(multiply u4 u5)` → `20`

### 134. **divide**

Divides first number by second (integer division).

**Example:** `(divide u10 u3)` → `3`

### 135. **mod**

Computes modulus (remainder).

**Example:** `(mod u10 u3)` → `1`

### 136. **pow**

Computes power (exponentiation).

**Example:** `(pow u2 u3)` → `8`

---

### 🔢 Length & Concatenation

### 137. **len**

Returns length of a list or string.

**Example:** `(len (list u1 u2 u3))` → `3`

### 138. **concat**

Concatenates two lists or strings.

**Example:** `(concat (list u1 u2) (list u3 u4))` → `(1 2 3 4)`

---

### 🔍 List Search & Logic

### 139. **and**

Logical AND of two boolean values.

**Example:** `(and true false)` → `false`

### 140. **or**

Logical OR of two boolean values.

**Example:** `(or true false)` → `true`

### 141. **greater than**

Checks if first is greater than second.

**Example:** `(> u5 u3)` → `true`

### 142. **greater than or equal**

Checks if first is ≥ second.

**Example:** `(>= u5 u5)` → `true`

### 143. **less than**

Checks if first is less than second.

**Example:** `(< u3 u5)` → `true`

### 144. **less than or equal**

Checks if first is ≤ second.

**Example:** `(<= u5 u5)` → `true`

---

### 🔍 Equality & Error Handling

### 145. **is-eq**

Checks equality of two values.

**Example:** `(is-eq u5 u5)` → `true`

### 146. **is-err**

Checks if a result is an error.

**Example:** `(is-err (err u1))` → `true`

### 147. **is-ok**

Checks if a result is ok.

**Example:** `(is-ok (ok u1))` → `true`

### 148. **is-none**

Checks if an optional value is `none`.

**Example:** `(is-none none)` → `true`

`(is-none (some u1))` → `false`

### 149. **is-some**

Checks if an optional value is `some`.

**Example:** `(is-some (some u1))` → `true`

`(is-some none)` → `false`

---

### 📜 Trait & Contract Utilities

### 150. **default-to**

Returns a default value if optional is `none`.

**Example:** `(default-to u0 none)` → `0`

`(default-to u0 (some u5))` → `5`

### 151. **define-constant**

Defines a constant value.

**Example:** `(define-constant PI u3)`

### 152. **define-data-var**

Defines a mutable data variable.

**Example:** `(define-data-var counter int u0)`

---

### 🔀 Variable & State Management

### 153. **var-get**

Gets the current value of a data variable.

**Example:** `(var-get counter)` → current counter value

### 154. **var-set**

Sets a new value for a data variable.

**Example:** `(var-set counter u10)` → `(ok u10)`

---

### ⚙️ Buffers & Conversion

### 155. **buff-to-int-be**

Converts big-endian buffer to int.

**Example:** `(buff-to-int-be 0x0000000a)` → `10`

### 156. **buff-to-int-le**

Converts little-endian buffer to int.

**Example:** `(buff-to-int-le 0x0a000000)` → `10`

### 157. **buff-to-uint-be**

Converts big-endian buffer to unsigned int.

**Example:** `(buff-to-uint-be 0x0000000a)` → `u10`

### 158. **buff-to-uint-le**

Converts little-endian buffer to unsigned int.

**Example:** `(buff-to-uint-le 0x0a000000)` → `u10`

### 159. **to-int**

Converts value to signed integer.

**Example:** `(to-int u10)` → `10`

### 160. **to-uint**

Converts value to unsigned integer.

**Example:** `(to-uint 10)` → `u10`

---

### 🔐 Signature & Crypto Verification

### 161. **secp256k1-verify**

Verifies a secp256k1 signature.

**Example:** `(secp256k1-verify pubkey message signature)` → `true/false`

### 162. **secp256k1-recover?**

Recovers public key from a signature.

**Example:** `(secp256k1-recover? message signature recovery-id)` → `some pubkey`

---

### 🖨️ Debugging & Logging

### 163. **log2**

Returns integer base-2 logarithm.

**Example:** `(log2 u16)` → `4`

---

### Miscellaneous

### 164. **some**

Wraps a value in a `some` optional.

**Example:** `(some u10)`

### 165. **ok**

Wraps a value in an `ok` result.

**Example:** `(ok u10)`

### 166. **err**

Wraps a value in an `err` result.

**Example:** `(err u1)`