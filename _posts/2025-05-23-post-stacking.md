---
title: " Stacking"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---


# **Understanding Stacking in the Stacks Blockchain**

## **Introduction**

Stacking is a fundamental mechanism in the Stacks blockchain that allows STX holders to participate in the Proof of Transfer (PoX) consensus model. By locking up their STX tokens, stackers contribute to network security and earn Bitcoin (BTC) rewards in return.

In this blog post, we'll explore how stacking is implemented as a smart contract in Clarity, the programming language of Stacks. We'll cover:

- Solo Stacking
- Delegated Stacking
- Reward Distribution
- Common Errors

If you're new to stacking, check out the [official Stacks documentation](https://docs.stacks.co/) to understand the basics before diving into the technical details.

---

## **Stacking Smart Contract Overview**

The current implementation of stacking is handled by the **`pox-4`** contract, which can be viewed on the [Stacks Explorer](https://explorer.stacks.co/).

Stacking operations include:

- **Solo Stacking** – Locking STX directly.
- **Delegated Stacking** – Delegating STX to a pool operator.
- **Pool Operations** – Managing multiple delegated stackers.

Let’s break down each of these processes.

---

## **1. Solo Stacking with `stack-stx`**

Solo stacking is the simplest way to participate. The **`stack-stx`** function locks STX for a specified period.

### **Key Parameters:**

- **`amount-ustx`** – Amount to lock (in micro-STX).
- **`pox-addr`** – Bitcoin address for rewards (encoded as a tuple).
- **`start-burn-ht`** – Bitcoin block height to start stacking.
- **`lock-period`** – Number of reward cycles (1-12).
- **`signer-sig`** & **`signer-key`** – Cryptographic proof of ownership.
- **`max-amount`** & **`auth-id`** – Security measures to prevent replay attacks.

### **How It Works:**

1. **Checks:**
    - Validates the caller’s permissions.
    - Ensures the stacker isn’t already stacking or delegating.
    - Confirms sufficient STX balance.
    - Verifies the PoX address and signer key.
2. **Execution:**
    - Locks the STX for the given period.
    - Registers the PoX address for rewards.
    - Updates the **`stacking-state`** map.

### **Supported Bitcoin Address Types:**

The **`pox-addr`** must specify a version (e.g., P2PKH, P2SH, SegWit). Example:



```
(define-constant ADDRESS_VERSION_P2PKH 0x00)
(define-constant ADDRESS_VERSION_P2SH 0x01)
(define-constant ADDRESS_VERSION_P2WPKH 0x02)
```

---

## **2. Delegated Stacking**

Delegated stacking allows users to delegate their STX to a pool operator, who handles the stacking process.

### **Step 1: `delegate-stx` (Delegator Calls)**

- Delegators authorize a pool operator to stack on their behalf.
- Optional **`pox-addr`** forces rewards to a specific Bitcoin address.

### **Step 2: `delegate-stack-stx` (Pool Operator Calls)**

- The pool operator locks the delegated STX.
- Validates delegation permissions and PoX address.

### **Step 3: `stack-aggregation-commit-indexed` (Finalize Delegation)**

- Aggregates all partially stacked STX.
- Requires a valid **`signer-sig`** to prove ownership.

---

## **3. Reward Distribution**

Rewards are distributed via Bitcoin transactions.

- **Solo stackers** receive BTC directly to their **`pox-addr`**.
- **Delegated stackers** rely on the pool operator to distribute rewards fairly.

### **Key Considerations:**

- If a delegator provides a **`pox-addr`**, the pool **must** send rewards there.
- Without a specified **`pox-addr`**, the pool operator decides distribution.

---

## **Common Stacking Errors**

| **Error Code** | **Description** | **Solution** |
| --- | --- | --- |
| **ERR_INVALID_SIGNATURE_PUBKEY (35)** | Mismatch in signer key or parameters. | Verify signature generation inputs. |
| **ERR_STACKING_NO_SUCH_PRINCIPAL (4)** | Invalid lookup in partial stacking. | Ensure correct **`pox-addr`**, **`stacker`**, and **`reward-cycle`**. |
| **ERR_INVALID_START_BURN_HEIGHT (24)** | Invalid **`start-burn-ht`** (past/future cycle). | Use the current cycle’s burn height. |

---

## **Tools & Resources**

- [**Leather Wallet (Earn)**](https://leather.io/earn) – User-friendly stacking UI.
- [**Stacks.js Stacking Library**](https://github.com/stacks-network/stacks.js) – JavaScript helper for stacking.
- [**Hiro Stacking Guide**](https://hiro.so/stacking) – Detailed walkthrough.

---

## **Conclusion**

Stacking in Stacks is a powerful way to earn Bitcoin while securing the network. Whether you're solo stacking or delegating, understanding the **`pox-4`** contract ensures smooth participation.

For more details, check out the [Stacks documentation](https://docs.stacks.co/) and experiment with the [Stacks Explorer](https://explorer.stacks.co/).

Happy stacking! 🚀