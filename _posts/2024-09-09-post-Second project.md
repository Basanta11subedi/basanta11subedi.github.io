---
title: "Second project in Solidity"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  
  - readability
  - standard
---
The distribution of an inheritance after the owner of the contract has passed away. 

```
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.2 <0.9.0;

contract will{
    address owner;
    uint fortune;
    bool deceased;

    constructor() payable { 
        owner= msg.sender;
        fortune= msg.value;
        deceased= false;
    }
    
    modifier onlyowner{
       require(owner == msg.sender);
       _;
    }
    modifier mustbedeceased{
        require(deceased== true);
        _;
    }

    address payable [] familywallets;  // list of the family members wallet address

    mapping (address=> uint) inheritance;

    //set inheritance for each address

    function setInheritance(address payable wallet, uint amount)public onlyowner{
        familywallets.push(wallet);
        inheritance[wallet]= amount;
    }

    function payout() private mustbedeceased {
        for( uint i=0; i<familywallets.length;i++){
            familywallets[i].transfer(inheritance[familywallets[i]]);
        }

    } 

    function hasdeceased() public onlyowner{
        deceased= true;
        payout();
    }
}
```

### Key Components

1. **State Variables**:
    - `address owner;`: Stores the address of the contract owner (the person who created the contract).
    - `uint fortune;`: Represents the total amount of Ether (the fortune) the contract holds.
    - `bool deceased;`: A flag to indicate whether the contract owner is deceased.
2. **Constructor**:
    - This constructor is marked as `payable`, allowing it to receive Ether when the contract is deployed.
    - The `owner` is set to the address that deploys the contract (`msg.sender`).
    - The `fortune` is set to the amount of Ether sent during deployment (`msg.value`).
    - The `deceased` flag is initialized to `false`, indicating the owner is alive.
3. **Modifiers**:
    - `onlyowner`: Ensures that certain functions can only be called by the contract owner.
    - `mustbedeceased`: Ensures that certain functions can only be executed if the owner is deceased.
4. **Family Wallets**:
    - An array that stores the addresses of the family members who will inherit the Ether.
5. **Inheritance Mapping**:
    - A mapping that associates each family member's address with the amount of Ether they will inherit.

### Functions

1. **`setInheritance`**:
    - This function allows the owner to set the inheritance amount for each family member.
    - It adds the family member's wallet address to the `familywallets` array and sets their inheritance amount in the `inheritance` mapping.
    - The function is restricted to the owner via the `onlyowner` modifier.
2. **`payout`**:
    - This private function transfers the specified inheritance amount to each family member's wallet.
    - It is restricted by the `mustbedeceased` modifier, ensuring that the payout only occurs after the owner is confirmed deceased.
3. **`hasdeceased`**:
    - This function is used by the owner to declare themselves deceased, which triggers the inheritance distribution.
    - It sets the `deceased` flag to `true` and then calls the `payout` function to distribute the Ether to the family members.

### Summary

- **Purpose**: The contract is intended to manage and distribute the owner's Ether to specified family members upon their death.
- **Control**: The contract owner has full control over setting the inheritance and can only trigger the payout once they have marked themselves as deceased.
- **Security**: The use of modifiers ensures that only the owner can set inheritance and declare themselves deceased, preventing unauthorized access.

#############

Note:

There are two types of accounts, one is external account which is determined by the public key and another is solidity contract which is determined duinf contract deployment.

#######