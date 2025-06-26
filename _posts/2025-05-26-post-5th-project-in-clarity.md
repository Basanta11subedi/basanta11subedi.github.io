---
title: " Time-locked STX Vault  (clarity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

```

;; Time-Locked STX Vault - A contract that allows users to deposit STX and lock them for a specified period
;; Users can only withdraw once the specified block height is reached

;; Define storage for the vault deposits
;; Map: user-address -> (amount, unlock-height)
(define-map vaults
  { owner: principal }
  {
    amount: uint,
    unlock-height: uint
  }
)

;; Public function to deposit STX and set a lock time
(define-public (deposit (amount uint) (lock-blocks uint))
  (let
    (
      (current-height stacks-block-height)
    )
    (asserts! (> amount u0) (err u1))
    (asserts! (and (>= lock-blocks u1) (<= lock-blocks u52560)) (err u5))
    (asserts! (is-none (map-get? vaults {owner: tx-sender})) (err u2))
    (let 
      ((unlock-at (+ current-height lock-blocks)))
      (map-set vaults
        {owner: tx-sender}
        {
          amount: amount,
          unlock-height: unlock-at
        }
      )
      (try! (stx-transfer? amount tx-sender (as-contract tx-sender)))
      (ok unlock-at)
    )
  )
)


;; Public function to withdraw STX after lock period
(define-public (withdraw)
  (let
    (
      (vault-data (unwrap! (map-get? vaults {owner: tx-sender}) (err u3)))
      (amount (get amount vault-data))
      (unlock-height (get unlock-height vault-data))
    )
    (asserts! (>= stacks-block-height unlock-height) (err u4))
    (map-delete vaults {owner: tx-sender})
    (try! (stx-transfer? amount (as-contract tx-sender) tx-sender))
    (ok amount) ;; Return the amount withdrawn
  )
)


;; Read-only function to get information about a user's vault
(define-read-only (get-vault-info (user principal))
  (map-get? vaults {owner: user})
)

;; Read-only function to check if withdrawal is possible
(define-read-only (can-withdraw (user principal))
  (match (map-get? vaults {owner: user})
    vault (>= stacks-block-height (get unlock-height vault))
    false
  )
)

;; Read-only function to check how many blocks remain until withdrawal is possible
(define-read-only (blocks-until-unlock (user principal))
  (match (map-get? vaults {owner: user})
    vault (if (>= stacks-block-height (get unlock-height vault))
            u0
            (- (get unlock-height vault) stacks-block-height))
    u0
  )
)

```

This smart contract enables users to **deposit STX tokens** and lock them for a certain number of **block heights**. Once the time has passed, they can withdraw their tokens.

---

### 🧠 Brief Overview

- **Purpose**: Lock STX tokens until a future block height.
- **Use Case**: Save funds for later (delayed vesting, time-based release).
- **Key Features**:
    - Users deposit STX and choose how many blocks to lock.
    - Withdrawal is only allowed after the unlock time.
    - Includes helpful read-only utilities for vault status.

---

## 📄 Code Breakdown

### 🗃️ 1. Define Vault Storage

```
(define-map vaults
  { owner: principal }
  {
    amount: uint,
    unlock-height: uint
  }
)

```

- `vaults` is a map that stores each user’s deposit and unlock time.
- Key: `{ owner: principal }` — user’s wallet address.
- Value: `{ amount, unlock-height }` — how much they deposited and when it unlocks.

---

### 💰 2. Deposit STX with a Lock

```
(define-public (deposit (amount uint) (lock-blocks uint))

```

### What it does:

- Lets users deposit STX and lock them for a certain number of blocks.

### How it works:

```
(asserts! (> amount u0) (err u1)) ;; Must deposit more than 0
(asserts! (and (>= lock-blocks u1) (<= lock-blocks u52560)) (err u5)) ;; Lock duration between 1 and ~1 year
(asserts! (is-none (map-get? vaults {owner: tx-sender})) (err u2)) ;; User can only have one active vault

```

- It calculates the unlock time:

```
(let ((unlock-at (+ current-height lock-blocks)))

```

- Stores the vault:

```
(map-set vaults {owner: tx-sender} { amount: amount, unlock-height: unlock-at })

```

- Transfers STX from the user to the contract:

```
(try! (stx-transfer? amount tx-sender (as-contract tx-sender)))

```

- Returns the unlock block height:

```
(ok unlock-at)

```

---

### 🏧 3. Withdraw After Unlock

```
(define-public (withdraw)

```

### What it does:

- Allows users to withdraw **only after the lock period ends**.

### Steps:

1. Load the vault:

```
(let ((vault-data (unwrap! (map-get? vaults {owner: tx-sender}) (err u3)))

```

1. Check unlock time:

```
(asserts! (>= stacks-block-height unlock-height) (err u4))

```

1. Remove vault record:

```
(map-delete vaults {owner: tx-sender})

```

1. Send STX back:

```
(try! (stx-transfer? amount (as-contract tx-sender) tx-sender))

```

1. Return amount withdrawn:

```
(ok amount)

```

---

### 🧾 4. Read-Only Functions

These don't modify state and are used to **check vault info**:

### 🔎 Get Vault Info

```
(define-read-only (get-vault-info (user principal))
  (map-get? vaults {owner: user})
)

```

- Returns a user's vault data (amount + unlock-height), or `none`.

---

### ⌛ Check If Withdrawal Is Allowed

```
(define-read-only (can-withdraw (user principal))
  (match (map-get? vaults {owner: user})
    vault (>= stacks-block-height (get unlock-height vault))
    false
  )
)

```

- Returns `true` if user’s lock time has passed.

---

### 🕒 Check Remaining Time

```
(define-read-only (blocks-until-unlock (user principal))
  (match (map-get? vaults {owner: user})
    vault (if (>= stacks-block-height (get unlock-height vault))
            u0
            (- (get unlock-height vault) stacks-block-height))
    u0
  )
)

```

- Tells how many blocks are left until the user can withdraw.

---

## 🧭 Step-by-Step Example

### 🧑 Alice Deposits 100 STX for 1 Day (~144 blocks)

```
(deposit u100 u144)

```

- Current block: 1000
- Unlock block: 1000 + 144 = 1144
- STX is locked in contract.

---

### 🧑 Alice tries to withdraw at block 1100

```
(withdraw)

```

- Fails: `stacks-block-height < unlock-height`
- Returns error: `err u4` (Too early)

---

### ✅ Alice tries at block 1145

- Succeeds!
- STX is sent back.
- Vault entry is deleted.

---

## 📘 Summary Table

| Function | Purpose |
| --- | --- |
| `deposit` | Lock STX for a set number of blocks |
| `withdraw` | Retrieve STX after unlock height |
| `get-vault-info` | View stored amount and unlock time |
| `can-withdraw` | Check if user can withdraw yet |
| `blocks-until-unlock` | See how many blocks are left to wait |
| `vaults` map | Stores each user’s locked STX & unlock time |

---

## 🧩 Real-World Analogy

Imagine putting money in a **timed safe**:

- You set the lock for 24 hours.
- Once the time is up, you can unlock it and take the money.
- You can't change your mind or unlock early.


## This is the simple frontend code that I have written.

```
import { useState, useEffect } from 'react';
import { connect, disconnect, isConnected, getLocalStorage, request } from '@stacks/connect';
import { broadcastTransaction, Cl, fetchCallReadOnlyFunction, cvToValue } from '@stacks/transactions';

export default function App() {
  const [walletAddress, setWalletAddress] = useState('');
  const [amount, setAmount] = useState();
  const [lockBlocks, setLockBlocks] = useState();
  const [vaultInfo, setVaultInfo] = useState(null);
  const [blocksUntilUnlock, setBlocksUntilUnlock] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');
  const [success, setSuccess] = useState('');


  const CONTRACT_ADDRESS = 'ST2WD8TKH9C3VKX4C355RGPPWFGYRAA9WT29SFQZY.time-vault3';
  

  useEffect(() => {
    const checkConnection = async () => {
      try {
        if (isConnected()) {
          const data = getLocalStorage();
          const stxAddress = data?.addresses?.stx?.[0]?.address;
          setWalletAddress(stxAddress);
          console.log('hello',typeof stxAddress)
          console.log("address:", stxAddress);
          if (stxAddress) {
            try{
            await fetchVaultInfo(stxAddress);
            }catch (err) {
              console.error("fetchVault failed", err)
            }
          }
        }
      } catch (err) {
        console.error('Connection check failed:', err);
        setError('Failed to check wallet connection');
      }
    };
    checkConnection();
  }, [walletAddress]);

  const handleConnect = async () => {
    try {
      setLoading(true);
      setError('');
      await connect();
      const data = getLocalStorage();
      const stxAddress = data?.addresses?.stx?.[0]?.address || '';
     
      setWalletAddress(stxAddress);
      await fetchVaultInfo(stxAddress);
      setSuccess('Wallet connected successfully');
      setTimeout(() => setSuccess(''), 3000);
    } catch (error) {
      console.error('Connection failed:', error);
      setError('Wallet connection failed');
    } finally {
      setLoading(false);
    }
  };

  const handleDisconnect = () => {
    try {
      disconnect();
      setWalletAddress('');
      setVaultInfo(null);
      setBlocksUntilUnlock(null);
      setSuccess('Wallet disconnected');
      setTimeout(() => setSuccess(''), 3000);
    } catch (err) {
      console.error('Disconnect failed:', err);
      setError('Failed to disconnect wallet');
    }
  };

  const depositSTX = async () => {
    if (!amount || !lockBlocks || Number(amount) <= 0 || Number(lockBlocks) <= 0) {
      setError('Please enter valid amount and lock period');
      return;
    }

    try {
      setLoading(true);
      setError('');
      const amountMicroSTX = (amount * 1000000);
      const blocks = (lockBlocks);

      console.log("Depositing:", {
        amountSTX: amount,
        amountMicroSTX,
        lockBlocks: typeof blocks
      });

      const cvAmount = Cl.uint((amountMicroSTX)) // convert STX to micro-STX
      const cvBlock = Cl.uint((blocks))

      console.log("lala", {cvAmount, Block: cvBlock})

      console.log("amount:", amount * 1000000);
      const response = await request('stx_callContract', {
        contract: CONTRACT_ADDRESS,
        functionName: 'deposit',
        functionArgs: [
          cvAmount,
          cvBlock
        ],
        network: 'testnet',
        postConditionMode: 'allow'
      });
      console.log("amount:", amount * 1000000);
      console.log('Deposit success:', response);
      setSuccess('Deposit transaction submitted');
      setTimeout(() => setSuccess(''), 3000);
      // Wait a few seconds then refresh vault info
      setTimeout(() => fetchVaultInfo(walletAddress), 5000);
    } catch (error) {
      console.error('Deposit failed:', error);
      setError('Deposit failed: ' + (error.message || 'Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  const withdrawSTX = async () => {
    try {
      setLoading(true);
      setError('');
      const response = await request('stx_callContract', {
        contract: CONTRACT_ADDRESS,
        functionName: 'withdraw',
        functionArgs: [],
        network: 'testnet'
      });
      console.log('Withdraw success:', response);
      setSuccess('Withdrawal transaction submitted');
      setTimeout(() => setSuccess(''), 3000);
      // Wait a few seconds then refresh vault info
      setTimeout(() => fetchVaultInfo(walletAddress), 5000);
    } catch (error) {
      console.error('Withdraw failed:', error);
      setError('Withdrawal failed: ' + (error.message || 'Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  const fetchVaultInfo = async (address) => {
    try {
      setLoading(true);
      setError('');
      
      // Get vault inf0

      const VaultTxOptions = {
        contractName: 'time-vault3',
        contractAddress: 'ST2WD8TKH9C3VKX4C355RGPPWFGYRAA9WT29SFQZY',
        functionName: 'get-vault-info',
        functionArgs: [Cl.standardPrincipal(address)],
        senderAddress: address,
        network: 'testnet'
      }

      const vaultTransaction = await fetchCallReadOnlyFunction(VaultTxOptions);
      
      console.log(vaultTransaction);
      const readable = cvToValue(vaultTransaction)
      console.log(readable.value);
      console.log(readable.value['unlock-height'].value);
      


      
      
      // Get blocks until unlock
      const BlockTxOptions = {
        contractName: 'time-vault3',
        contractAddress: 'ST2WD8TKH9C3VKX4C355RGPPWFGYRAA9WT29SFQZY',
        functionName: 'blocks-until-unlock',
        functionArgs: [Cl.standardPrincipal(address)],
        senderAddress: address,
        network: 'testnet'
      }

      const blockTransaction = await fetchCallReadOnlyFunction(BlockTxOptions);
      
      console.log(blockTransaction);
      
      const blockreadable = cvToValue(blockTransaction);
      console.log(blockreadable);



      // Check if vault exists (response will be null if no vault)
      if (vaultTransaction === null) {
        setVaultInfo(null);
        setBlocksUntilUnlock(null);
      } else {
        setVaultInfo(readable);
        setBlocksUntilUnlock();
      }
    } catch (error) {
      console.error('Fetch Vault Info failed:', error);
      setError('No Vault');
    } finally {
      setLoading(false);
    }
  };

  const formatSTX = (microstx) => {
    return (microstx / 1000000).toFixed(6);
  };

  return (
    <div className="min-h-screen bg-gray-100 flex flex-col items-center justify-center p-4">
      <div className="max-w-md w-full bg-white shadow-lg rounded-lg p-6 space-y-6">
        <h1 className="text-2xl font-bold text-center text-indigo-600">STX Time Vault</h1>

        {/* Status messages */}
        {error && (
          <div className="p-3 bg-red-100 text-red-700 rounded">
            {error}
            <button onClick={() => setError('')} className="float-right font-bold">
              ×
            </button>
          </div>
        )}
        {success && (
          <div className="p-3 bg-green-100 text-green-700 rounded">
            {success}
            <button onClick={() => setSuccess('')} className="float-right font-bold">
              ×
            </button>
          </div>
        )}

        {loading && (
          <div className="text-center py-4">
            <div className="inline-block animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-indigo-600"></div>
            <p className="mt-2 text-gray-600">Processing...</p>
          </div>
        )}

        {!walletAddress ? (
          <button
            onClick={handleConnect}
            disabled={loading}
            className="w-full bg-indigo-600 text-white py-2 rounded hover:bg-indigo-700 disabled:bg-indigo-300"
          >
            Connect Wallet
          </button>
        ) : (
          <>
            <div className="space-y-2">
              <p className="text-gray-700 text-sm break-words">Connected: {walletAddress}</p>
              <button
                onClick={handleDisconnect}
                disabled={loading}
                className="w-full bg-red-500 text-white py-2 rounded hover:bg-red-600 disabled:bg-red-300"
              >
                Disconnect
              </button>
            </div>

            <div className="space-y-4">
              <h2 className="text-lg font-semibold text-gray-800">Deposit STX</h2>
              <input
                type="number"
                placeholder="Amount (STX)"
                className="w-full p-2 border rounded"
                value={amount}
                onChange={(e) => setAmount(e.target.value)}
                disabled={loading}
                min="0.000001"
                step="0.000001"
              />
              <input
                type="number"
                placeholder="Lock Blocks (1 block ≈ 10 minutes)"
                className="w-full p-2 border rounded"
                value={lockBlocks}
                onChange={(e) => setLockBlocks(e.target.value)}
                disabled={loading}
                min="1"
              />
              <button
                onClick={depositSTX}
                disabled={loading || !amount || !lockBlocks}
                className="w-full bg-green-500 text-white py-2 rounded hover:bg-green-600 disabled:bg-green-300"
              >
                Deposit
              </button>
            </div>

            {vaultInfo ? (
              <div className="space-y-2 p-4 bg-gray-50 rounded">
                <h2 className="text-lg font-semibold text-gray-800">Vault Info</h2>
                <p>Amount Locked: {formatSTX(vaultInfo.value.amount.value)} STX</p>
                <p>Unlock Block Height: {vaultInfo.value['unlock-height'].value}</p>
                {blocksUntilUnlock == 0 ? 
                <>
                <p>Blocks Remaining: {blocksUntilUnlock} b</p>
                <p className="text-sm text-gray-500">
                  Approx. {(blocksUntilUnlock * 10 / 60).toFixed(1)} hours remaining
                </p>
                </> : <><p>You can withdraw your stx</p></> }
              </div>
            ) : (
              <div className="p-4 bg-gray-50 rounded text-center">
                <p className="text-gray-600">No active vault found for this address</p>
              </div>
            )}

            <button
              onClick={withdrawSTX}
              disabled={loading || !vaultInfo}
              className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600 disabled:bg-blue-300"
            >
              Withdraw
            </button>
          </>
        )}
      </div>
    </div>
  );
}
```