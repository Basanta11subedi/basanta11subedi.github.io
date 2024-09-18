---
title: "Fourth project in Solidity"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  
  - readability
  - standard
---

A simple contract between a landlord and tenant for renting property. It could handle rent payments, security deposits, and lease termination terms.

```

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract RentalAgreement {
address public landlord;
address public tenant;
uint public rentAmount;
bool public isPaid;
uint public depositAmount;

constructor(address _tenant, uint _rentAmount, uint _depositAmount) {
    landlord = msg.sender;
    tenant = _tenant;
    rentAmount = _rentAmount;
    depositAmount = _depositAmount;
}

function payRent() public payable {
    require(msg.sender == tenant, "Only the tenant can pay");
    require(msg.value == rentAmount, "Incorrect rent amount");
    isPaid = true;
}

function refundDeposit() public {
    require(msg.sender == landlord, "Only the landlord can refund");
    require(isPaid == true, "Rent must be paid before refund");
    payable(tenant).transfer(depositAmount);
}

```

}

Explanation:

```solidity
solidity
Copy code
// SPDX-License-Identifier: MIT

```

- **SPDX-License-Identifier**: Specifies the license for the contract. The `MIT` license is a permissive free software license, allowing others to use, modify, and distribute the code.

```solidity
solidity
Copy code
pragma solidity ^0.8.0;

```

- **Pragma Directive**: This line specifies that the Solidity version used to compile this contract should be `0.8.0` or higher, but not breaking backward compatibility.

```solidity
solidity
Copy code
contract RentalAgreement {

```

- **Contract Declaration**: Defines the `RentalAgreement` contract. Inside this, we declare variables and functions for handling the rental process.

### **State Variables**

State variables store data on the blockchain, and their values persist as long as the contract is active.

```solidity
solidity
Copy code
    address public landlord;
    address public tenant;
    uint public rentAmount;
    bool public isPaid;
    uint public depositAmount;

```

- **landlord**: Stores the Ethereum address of the landlord (the person deploying the contract). It's marked `public`, so anyone can access it externally.
- **tenant**: Stores the Ethereum address of the tenant (who will pay the rent). It is also marked `public`.
- **rentAmount**: This `uint` (unsigned integer) stores the agreed rent amount. Also publicly accessible.
- **isPaid**: A boolean flag to track whether the rent has been paid. Initially `false`, and once the tenant pays the rent, it is set to `true`.
- **depositAmount**: A `uint` representing the deposit amount that the tenant gives to the landlord as part of the rental agreement. This is usually refundable under certain conditions.

### **Constructor**

The constructor is a special function that is called only once when the contract is deployed. It sets up the initial values.

```solidity
solidity
Copy code
    constructor(address _tenant, uint _rentAmount, uint _depositAmount) {
        landlord = msg.sender;
        tenant = _tenant;
        rentAmount = _rentAmount;
        depositAmount = _depositAmount;
    }

```

- **msg.sender**: This global variable refers to the address that is interacting with the contract. In this case, the person who deploys the contract becomes the `landlord`.
- **_tenant**: This parameter specifies the tenant's address passed when the contract is deployed. It is assigned to the `tenant` state variable.
- **_rentAmount**: This parameter defines the rent to be paid. It is set to the `rentAmount` state variable.
- **_depositAmount**: This parameter defines the deposit amount. It is assigned to `depositAmount` state variable.

### **payRent Function**

```solidity
solidity
Copy code
    function payRent() public payable {
        require(msg.sender == tenant, "Only the tenant can pay");
        require(msg.value == rentAmount, "Incorrect rent amount");
        isPaid = true;
    }

```

- **payRent**: This is a public function that allows the tenant to pay rent. It is `payable`, meaning it can receive Ether as payment.
- **require(msg.sender == tenant, "Only the tenant can pay")**: This condition ensures that only the tenant (who was set in the constructor) can call this function to pay the rent. If someone else tries, the transaction is reverted with the error message "Only the tenant can pay."
- **require(msg.value == rentAmount, "Incorrect rent amount")**: This condition ensures that the amount of Ether sent with the transaction matches the `rentAmount`. If the tenant sends an incorrect amount, the transaction will fail with the message "Incorrect rent amount."
- **isPaid = true**: Once the correct rent is paid, this line sets the `isPaid` boolean to `true`, indicating that the rent has been paid.

### **refundDeposit Function**

```solidity
solidity
Copy code
    function refundDeposit() public {
        require(msg.sender == landlord, "Only the landlord can refund");
        require(isPaid == true, "Rent must be paid before refund");
        payable(tenant).transfer(depositAmount);
    }

```

- **refundDeposit**: This function allows the landlord to refund the deposit after the rent has been paid.
- **require(msg.sender == landlord, "Only the landlord can refund")**: Ensures that only the landlord can trigger the refund process. If anyone else tries, the transaction is reverted with the error message "Only the landlord can refund."
- **require(isPaid == true, "Rent must be paid before refund")**: This ensures that the tenant must pay the rent first before the deposit can be refunded. If the rent hasn't been paid, the transaction fails with the message "Rent must be paid before refund."
- **payable(tenant).transfer(depositAmount)**: This transfers the `depositAmount` back to the tenant's wallet address after the rent has been paid. The `payable` keyword allows the contract to send Ether.