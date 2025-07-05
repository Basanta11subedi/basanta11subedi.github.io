---
title: "Lottery (solidity)"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---


```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Lottery{
    address public owner;
    uint256 public ticketPrice;
    uint256 public totalCommission;
    uint256 public lotteryEndTime;
    bool public lotteryActive;
    uint256 private ticketCounter;
    uint256 public winningTicket;

    mapping(address => uint256[]) public tickets;
    mapping(uint256 => address) public ticketOwners;
    address[] public players;

    constructor(){
        owner= msg.sender;
        lotteryActive= false;
        ticketCounter= 0;
        winningTicket= 0;
    }
    modifier onlyOwner(){
        require(msg.sender== owner,"Not the contract owner");
        _;
    }
    modifier whenLotteryActive(){
        require(lotteryActive,"Lottery is not active");
        _;
    }
    modifier whenLotteryEnded(){
        require(block.timestamp>= lotteryEndTime,"Lottery hasnot ended yet");
        _;
    }
    //function to start the lottery
    function startLottery(uint256 _ticketPrice, uint256 _durationInSeconds) public onlyOwner() {
        require(!lotteryActive,"A lottery is already active");
        ticketPrice = _ticketPrice;
        lotteryActive = true;
        lotteryEndTime = block.timestamp + _durationInSeconds;
        winningTicket= 0;
    }
    //functions to buy a tickets(10 tickets at a time not more than that)
    function buyTickets(uint256 _numTickets) public payable whenLotteryActive(){
        require(block.timestamp < lotteryEndTime,"Lottery has ended");
        require(_numTickets > 0 && _numTickets <=10,"Can buy 1-10 tickets only");
        require(msg.value == _numTickets * ticketPrice, "Incorrect payement");

        if (tickets[msg.sender].length == 0){
            players.push(msg.sender);
        }
        for (uint256 i =0; i < _numTickets; i++){
            ticketCounter++;
            tickets[msg.sender].push(ticketCounter);
            ticketOwners[ticketCounter] = msg.sender;
        }
    }
    //function to end the lottery by the owner and pick the random number for winner
    function endLottery() public onlyOwner whenLotteryActive whenLotteryEnded{
        require(ticketCounter > 0,"No tickets sold");

        //Generate a random winning ticket
        winningTicket = uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao, ticketCounter))) % ticketCounter + 1; 

        //Determine the winner
        address winner = ticketOwners[winningTicket];

        //calculate prize distribution
        uint256 prize = address(this).balance * 90/100;
        uint256 commission = address(this).balance - prize;
        totalCommission += commission;

        //transfer prize to winner
        payable(winner).transfer(prize);

        // Reset the lottery
        delete players;
        lotteryActive = false;
    }
    //function for the owner to withdraw the total commisiion
    function withdrawCommission(uint amount) public onlyOwner{
        require(amount > 0 && amount <= totalCommission,"Withdraw amount should be greated than zero");
        payable(owner).transfer(amount);
        totalCommission -= amount;
    }
    function getPlayerTickets(address _player) public view returns(uint256[] memory){
        return tickets[_player];
    }
    //view function to check the winning tickets
    function getWinningTickets() public view returns(uint256){
        require(!lotteryActive,"Lottery is still active");
        return winningTicket;
    }
    //view function to check the contract balance
    function getContractbalance() public view returns(uint256){
        return address(this).balance;
    }

}

```

## 🎲 Smart Contract: `Lottery` (Explained)

### 📜 License and Version

```solidity

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

```

- **License**: The contract is unlicensed, meaning it's not open-source unless stated otherwise.
- **Solidity version**: Compatible with version 0.8.13 or higher.

---

### 🏛️ State Variables

```solidity

address public owner;
uint256 public ticketPrice;
uint256 public totalCommission;
uint256 public lotteryEndTime;
bool public lotteryActive;
uint256 private ticketCounter;
uint256 public winningTicket;

```

- `owner`: The person who deployed the contract (admin).
- `ticketPrice`: Cost of one lottery ticket.
- `totalCommission`: Earnings retained by the owner (10% of total funds).
- `lotteryEndTime`: Timestamp when lottery ends.
- `lotteryActive`: Tracks if a lottery is ongoing.
- `ticketCounter`: Unique counter for ticket IDs.
- `winningTicket`: The randomly selected winning ticket number.

---

### 👥 Mappings and Arrays

```solidity

mapping(address => uint256[]) public tickets;
mapping(uint256 => address) public ticketOwners;
address[] public players;

```

- `tickets`: Maps a player's address to a list of their ticket numbers.
- `ticketOwners`: Maps each ticket number to its owner.
- `players`: List of all unique players who bought at least one ticket.

---

### 🛠️ Constructor: Sets the Contract Owner

```solidity

constructor(){
    owner= msg.sender;
    lotteryActive= false;
    ticketCounter= 0;
    winningTicket= 0;
}

```

- Runs once when the contract is deployed.
- Initializes the owner and sets default values.

---

### 🔐 Modifiers: Access Control

```solidity

modifier onlyOwner() { ... }
modifier whenLotteryActive() { ... }
modifier whenLotteryEnded() { ... }

```

- `onlyOwner`: Restricts function to owner/admin.
- `whenLotteryActive`: Ensures a lottery is ongoing.
- `whenLotteryEnded`: Ensures the lottery duration has passed.

---

### 🚀 Start the Lottery

```solidity

function startLottery(uint256 _ticketPrice, uint256 _durationInSeconds) public onlyOwner { ... }

```

- Starts a new lottery.
- Owner sets:
    - Ticket price
    - Duration (in seconds)
- Resets the winning ticket.

---

### 🎟️ Buy Lottery Tickets

```solidity

function buyTickets(uint256 _numTickets) public payable whenLotteryActive { ... }

```

- Buy **1 to 10 tickets** at once.
- Validates correct payment (`_numTickets * ticketPrice`).
- Adds buyer to `players` if it’s their first time.
- Mints new tickets and assigns them.

---

### 🛑 End the Lottery and Pick Winner

```solidity

function endLottery() public onlyOwner whenLotteryActive whenLotteryEnded { ... }

```

- Can only be called after lottery duration ends.
- Generates a **random winner** using:

```solidity

uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao, ticketCounter))) % ticketCounter + 1;

```

- 90% of funds go to the winner.
- 10% goes to the owner as commission.
- Resets the state for the next lottery.

---

### 💵 Withdraw Commission (Owner Only)

```solidity

function withdrawCommission(uint amount) public onlyOwner { ... }

```

- Lets the owner withdraw earned commissions.

---

### 👁️ View Player Tickets

```solidity

function getPlayerTickets(address _player) public view returns(uint256[] memory)

```

- Returns all ticket numbers owned by a player.

---

### 🏆 View Winning Ticket

```solidity

function getWinningTickets() public view returns(uint256)

```

- Only callable after lottery ends.
- Returns the winning ticket number.

---

### 💰 View Contract Balance

```solidity

function getContractbalance() public view returns(uint256)

```

- Shows the current balance (ETH) held by the contract.

---

---

### 🎲 **Lottery Contract Overview**

This smart contract allows users to participate in a decentralized lottery where:

- Players buy tickets using ETH.
- A random winner is selected after the deadline.
- The winner receives **90%** of the pot.
- The owner collects a **10% commission**.

---

### 🔐 Admin Controls:

- **Start** and **end** the lottery.
- **Set ticket price** and **duration**.
- **Withdraw commission** after lottery ends.

---

### 💡 How Randomness Works:

The winner is picked using:

```solidity

keccak256(abi.encodePacked(block.timestamp, block.prevrandao, ticketCounter))

```

This creates a pseudo-random hash based on:

- The time
- A blockchain randomness source
- Ticket count

---

### 💳 Rules:

- Buy up to **10 tickets per transaction**
- Must pay exact amount: `ticketPrice * number of tickets`
- One winner per round