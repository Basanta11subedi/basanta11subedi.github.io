---
title: "Introduction to Solidity"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  
  - readability
  - standard
---

### 

Create the contract that recieve the data and retrive the data.
        - Be able to do following
          1. Receive the information, 2. Store the inforamtion, 3. Show the information

```solidity
pragma solidity >=0.8.2 <0.9.0;
// create the contract that recieve the data and retrive the data.
// be able to do following
//1. receive the information, 2. store the inforamtion, 3. show the information

contract simpleStorage{   // our contract name is simpleStorage
    uint storevalue;   // variables are the reserved memory location to store value.
    // Function is a group of resueable code that can be used anywhere in our application. They perform a specific task.
    // For this contract we need two function. One is to set the value and another is to get the value.
    // to make a function, "Funtion" keyword is used.
    function set(uint _storedvalue) public {    // here public enables the visibility so that function can be call from outside the function.
        storevalue= _storedvalue;               
    }
    function get() public view returns(uint){// view is modifier which prevent the function to change the state.
        return storevalue;                    // reuturns specifies the return type of the function.
    }
}

```

- **`pragma solidity ...;`**:
    - The `pragma` keyword is a directive that provides additional information to the compiler. In this case, it's used to specify the version of Solidity that should be used to compile the contract.
    - `solidity` indicates that this pragma is specifically for Solidity, the programming language used to write smart contracts on the Ethereum blockchain.
- **`>=0.8.2`**:
    - `>=` means "greater than or equal to."
    - `0.8.2` refers to version 0.8.2 of the Solidity compiler.
    - This part of the pragma specifies that the contract can be compiled with Solidity version 0.8.2 or any newer version.
- **`<0.9.0`**:
    - `<` means "less than."
    - `0.9.0` refers to version 0.9.0 of the Solidity compiler.
    - This part of the pragma specifies that the contract cannot be compiled with Solidity version 0.9.0 or any later version.

### Detailed Explanation:

1. **Contract Declaration**:
    - `contract simpleStorage { ... }`:
        - This declares a new contract named `simpleStorage`. A contract in Solidity is similar to a class in object-oriented programming and contains state variables, functions, and other types of data.
2. **State Variable**:
    - `uint storevalue;`:
        - `storevalue` is a state variable of type `uint` (unsigned integer), which is stored on the blockchain. This variable holds the value that is set by the `set` function and retrieved by the `get` function.
        - State variables are persistent, meaning their values are stored on the blockchain and persist between function calls.
3. **Function Definition**:
    - **`set` function**:
        - `function set(uint _storedvalue) public { ... }`:
            - `function` is the keyword used to define a function.
            - `set` is the name of the function.
            - The function takes one argument, `_storedvalue`, of type `uint`.
            - The `public` keyword makes the function accessible from outside the contract.
            - Inside the function, the value passed to `_storedvalue` is assigned to the state variable `storevalue`.
    - **`get` function**:
        - `function get() public view returns (uint) { ... }`:
            - `function` is the keyword used to define a function.
            - `get` is the name of the function.
            - The function does not take any arguments.
            - The `public` keyword makes the function accessible from outside the contract.
            - The `view` modifier indicates that the function will not modify the contract’s state (i.e., it will not change any state variables).
            - `returns (uint)` specifies that the function returns a value of type `uint`.
            - The function returns the value stored in `storevalue`.

### Summary:

- **State Variable (`storevalue`)**: Stores an unsigned integer value that persists on the blockchain.
- **`set` Function**: Allows the user to set the value of `storevalue`.
- **`get` Function**: Allows the user to retrieve the value of `storevalue` without modifying the blockchain state.