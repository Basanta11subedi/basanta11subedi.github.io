---
title: "FreeLancing Platform (solidity)"
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
pragma solidity ^0.8.19;

/**
 * @title FreelancePlatform
 * @dev Smart contract for a decentralized freelance platform with escrow functionality
 */
contract FreelancePlatform {
    // State variables
    address public owner;
    uint256 public platformFee; // Platform fee in basis points (1% = 100)
    uint256 public jobCount;
    uint256 public proposalCount;
    uint256 public contractCount;

    // Enums
    enum JobStatus { Open, InProgress, Completed, Cancelled }
    enum ContractStatus { Created, Started, Completed, Cancelled, Disputed }
    enum DisputeStatus { None, InProgress, ResolvedForClient, ResolvedForFreelancer }

    // Structs
    struct Profile {
        string name;
        string description;
        string[] skills;
        bool isRegistered;
        bool isClient;
        bool isFreelancer;
        uint256 clientRating; // Average rating as client (out of 100)
        uint256 clientRatingCount;
        uint256 freelancerRating; // Average rating as freelancer (out of 100)
        uint256 freelancerRatingCount;
        uint256[] completedJobs;
    }

    struct Job {
        uint256 id;
        address client;
        string title;
        string description;
        string[] requiredSkills;
        uint256 budget;
        uint256 deadline; // Unix timestamp
        JobStatus status;
        uint256[] proposalIds;
        uint256 selectedProposal;
        uint256 contractId;
        bool isFixed; // Fixed price vs hourly
        uint256 createdAt;
    }

    struct Proposal {
        uint256 id;
        uint256 jobId;
        address freelancer;
        uint256 price; // Total price for fixed jobs OR hourly rate for hourly jobs
        string description;
        uint256 estimatedTime; // In days for fixed jobs OR hours for hourly jobs
        uint256 createdAt;
        bool selected;
    }

    struct WorkContract {
        uint256 id;
        uint256 jobId;
        uint256 proposalId;
        address client;
        address freelancer;
        uint256 amount;
        uint256 deposit; // Client's deposit
        ContractStatus status;
        uint256 startDate;
        uint256 endDate;
        uint256 createdAt;
        uint256 completedAt;
        DisputeStatus disputeStatus;
        string disputeReason;
        uint256 milestones; // Number of milestones
        uint256 completedMilestones; // Number of completed milestones
        mapping(uint256 => Milestone) milestoneDetails;
    }

    struct Milestone {
        string description;
        uint256 amount;
        bool completed;
        bool paid;
    }

    // Mappings
    mapping(address => Profile) public profiles;
    mapping(uint256 => Job) public jobs;
    mapping(uint256 => Proposal) public proposals;
    mapping(uint256 => WorkContract) public contracts;
    mapping(address => uint256[]) public clientJobs; // Jobs posted by a client
    mapping(address => uint256[]) public freelancerProposals; // Proposals submitted by a freelancer
    mapping(address => uint256[]) public freelancerContracts; // Contracts a freelancer is involved in
    mapping(address => uint256[]) public clientContracts; // Contracts a client is involved in

    // Events
    event ProfileCreated(address indexed user, bool isClient, bool isFreelancer);
    event JobPosted(uint256 indexed jobId, address indexed client, string title, uint256 budget);
    event ProposalSubmitted(uint256 indexed proposalId, uint256 indexed jobId, address indexed freelancer);
    event ProposalSelected(uint256 indexed jobId, uint256 indexed proposalId, address indexed freelancer);
    event ContractCreated(uint256 indexed contractId, uint256 indexed jobId, address client, address freelancer);
    event MilestoneAdded(uint256 indexed contractId, uint256 milestoneIndex, string description, uint256 amount);
    event MilestoneCompleted(uint256 indexed contractId, uint256 milestoneIndex);
    event MilestonePaid(uint256 indexed contractId, uint256 milestoneIndex, uint256 amount);
    event ContractStarted(uint256 indexed contractId);
    event ContractCompleted(uint256 indexed contractId);
    event ContractCancelled(uint256 indexed contractId);
    event DisputeRaised(uint256 indexed contractId, string reason);
    event DisputeResolved(uint256 indexed contractId, DisputeStatus resolution);
    event FundsDeposited(uint256 indexed contractId, address indexed sender, uint256 amount);
    event FundsWithdrawn(uint256 indexed contractId, address indexed recipient, uint256 amount);
    event RatingSubmitted(address indexed rated, address indexed rater, uint256 rating, bool isClientRating);

    // Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }

    modifier onlyRegistered() {
        require(profiles[msg.sender].isRegistered, "User not registered");
        _;
    }

    modifier onlyClient() {
        require(profiles[msg.sender].isClient, "Only clients can call this function");
        _;
    }

    modifier onlyFreelancer() {
        require(profiles[msg.sender].isFreelancer, "Only freelancers can call this function");
        _;
    }

    modifier onlyJobClient(uint256 _jobId) {
        require(jobs[_jobId].client == msg.sender, "Only job client can call this function");
        _;
    }

    modifier onlyContractParticipant(uint256 _contractId) {
        require(
            contracts[_contractId].client == msg.sender || contracts[_contractId].freelancer == msg.sender,
            "Only contract participants can call this function"
        );
        _;
    }

    // Constructor
    constructor() {
        owner = msg.sender;
        platformFee = 100; // Set as basis points (100 = 1%)
        jobCount = 0;
        proposalCount = 0;
        contractCount = 0;
    }

    // Function to update platform fee (only owner)
    function updatePlatformFee(uint256 _newFee) external onlyOwner {
        require(_newFee <= 1000, "Fee cannot exceed 10%"); // 1000 basis points = 10%
        platformFee = _newFee;
    }

    // Function to register a new user
    function registerProfile(
        string memory _name,
        string memory _description,
        string[] memory _skills,
        bool _isClient,
        bool _isFreelancer
    ) external {
        require(!profiles[msg.sender].isRegistered, "Profile already registered");
        require(_isClient || _isFreelancer, "Must register as client or freelancer");

        Profile storage newProfile = profiles[msg.sender];
        newProfile.name = _name;
        newProfile.description = _description;
        newProfile.skills = _skills;
        newProfile.isRegistered = true;
        newProfile.isClient = _isClient;
        newProfile.isFreelancer = _isFreelancer;
        newProfile.clientRating = 0;
        newProfile.clientRatingCount = 0;
        newProfile.freelancerRating = 0;
        newProfile.freelancerRatingCount = 0;

        emit ProfileCreated(msg.sender, _isClient, _isFreelancer);
    }

    // Function to update a user's profile
    function updateProfile(
        string memory _name,
        string memory _description,
        string[] memory _skills,
        bool _isClient,
        bool _isFreelancer
    ) external onlyRegistered {
        require(_isClient || _isFreelancer, "Must be client or freelancer");

        Profile storage profile = profiles[msg.sender];
        profile.name = _name;
        profile.description = _description;
        profile.skills = _skills;
        profile.isClient = _isClient;
        profile.isFreelancer = _isFreelancer;
    }

    // Function to post a new job (only clients)
    function postJob(
        string memory _title,
        string memory _description,
        string[] memory _requiredSkills,
        uint256 _budget,
        uint256 _deadline,
        bool _isFixed
    ) external onlyRegistered onlyClient {
        require(bytes(_title).length > 0, "Title cannot be empty");
        require(_budget > 0, "Budget must be greater than 0");
        require(_deadline > block.timestamp, "Deadline must be in the future");

        uint256 newJobId = jobCount++;
        Job storage newJob = jobs[newJobId];
        newJob.id = newJobId;
        newJob.client = msg.sender;
        newJob.title = _title;
        newJob.description = _description;
        newJob.requiredSkills = _requiredSkills;
        newJob.budget = _budget;
        newJob.deadline = _deadline;
        newJob.status = JobStatus.Open;
        newJob.isFixed = _isFixed;
        newJob.createdAt = block.timestamp;

        clientJobs[msg.sender].push(newJobId);

        emit JobPosted(newJobId, msg.sender, _title, _budget);
    }

    // Function to submit a proposal for a job (only freelancers)
    function submitProposal(
        uint256 _jobId,
        uint256 _price,
        string memory _description,
        uint256 _estimatedTime
    ) external onlyRegistered onlyFreelancer {
        Job storage job = jobs[_jobId];
        require(job.status == JobStatus.Open, "Job is not open for proposals");
        require(job.deadline > block.timestamp, "Job deadline has passed");
        require(_price > 0, "Price must be greater than 0");
        require(bytes(_description).length > 0, "Description cannot be empty");
        require(_estimatedTime > 0, "Estimated time must be greater than 0");
        require(job.client != msg.sender, "Cannot submit proposal to own job");

        uint256 newProposalId = proposalCount++;
        Proposal storage newProposal = proposals[newProposalId];
        newProposal.id = newProposalId;
        newProposal.jobId = _jobId;
        newProposal.freelancer = msg.sender;
        newProposal.price = _price;
        newProposal.description = _description;
        newProposal.estimatedTime = _estimatedTime;
        newProposal.createdAt = block.timestamp;
        newProposal.selected = false;

        job.proposalIds.push(newProposalId);
        freelancerProposals[msg.sender].push(newProposalId);

        emit ProposalSubmitted(newProposalId, _jobId, msg.sender);
    }

    // Function to select a proposal (only job client)
    function selectProposal(uint256 _jobId, uint256 _proposalId) external onlyRegistered onlyJobClient(_jobId) {
        Job storage job = jobs[_jobId];
        Proposal storage proposal = proposals[_proposalId];

        require(job.status == JobStatus.Open, "Job is not open");
        require(proposal.jobId == _jobId, "Proposal not for this job");
        require(!proposal.selected, "Proposal already selected");

        job.selectedProposal = _proposalId;
        proposal.selected = true;

        // Create a contract
        uint256 newContractId = contractCount++;
        WorkContract storage newContract = contracts[newContractId];
        newContract.id = newContractId;
        newContract.jobId = _jobId;
        newContract.proposalId = _proposalId;
        newContract.client = job.client;
        newContract.freelancer = proposal.freelancer;
        newContract.amount = proposal.price;
        newContract.status = ContractStatus.Created;
        newContract.createdAt = block.timestamp;
        newContract.disputeStatus = DisputeStatus.None;

        job.contractId = newContractId;
        job.status = JobStatus.InProgress;

        clientContracts[job.client].push(newContractId);
        freelancerContracts[proposal.freelancer].push(newContractId);

        emit ProposalSelected(_jobId, _proposalId, proposal.freelancer);
        emit ContractCreated(newContractId, _jobId, job.client, proposal.freelancer);
    }

    // Function to add milestones to a contract (only client)
    function addMilestones(
        uint256 _contractId,
        string[] memory _descriptions,
        uint256[] memory _amounts
    ) external onlyRegistered {
        WorkContract storage workContract = contracts[_contractId];
        require(workContract.client == msg.sender, "Only client can add milestones");
        require(workContract.status == ContractStatus.Created, "Contract already started");
        require(_descriptions.length == _amounts.length, "Arrays must have same length");
        require(_descriptions.length > 0, "Must add at least one milestone");
        
        uint256 totalAmount = 0;
        for (uint256 i = 0; i < _amounts.length; i++) {
            totalAmount += _amounts[i];
        }
        require(totalAmount == workContract.amount, "Total milestone amounts must equal contract amount");

        // Add milestones
        for (uint256 i = 0; i < _descriptions.length; i++) {
            workContract.milestoneDetails[i] = Milestone({
                description: _descriptions[i],
                amount: _amounts[i],
                completed: false,
                paid: false
            });
            emit MilestoneAdded(_contractId, i, _descriptions[i], _amounts[i]);
        }

        workContract.milestones = _descriptions.length;
    }

    // Function for client to deposit funds to start the contract
    function depositFunds(uint256 _contractId) external payable onlyRegistered {
        WorkContract storage workContract = contracts[_contractId];
        require(workContract.client == msg.sender, "Only client can deposit funds");
        require(workContract.status == ContractStatus.Created, "Contract not in created state");
        require(msg.value == workContract.amount, "Deposit must equal contract amount");

        workContract.deposit = msg.value;
        workContract.status = ContractStatus.Started;
        workContract.startDate = block.timestamp;

        emit FundsDeposited(_contractId, msg.sender, msg.value);
        emit ContractStarted(_contractId);
    }

    // Function for freelancer to mark a milestone as completed
    function completeMilestone(uint256 _contractId, uint256 _milestoneIndex) external onlyRegistered {
        WorkContract storage workContract = contracts[_contractId];
        require(workContract.freelancer == msg.sender, "Only freelancer can complete milestone");
        require(workContract.status == ContractStatus.Started, "Contract not in started state");
        require(_milestoneIndex < workContract.milestones, "Invalid milestone index");
        require(!workContract.milestoneDetails[_milestoneIndex].completed, "Milestone already completed");

        workContract.milestoneDetails[_milestoneIndex].completed = true;
        workContract.completedMilestones++;

        emit MilestoneCompleted(_contractId, _milestoneIndex);
    }

    // Function for client to release payment for a milestone
    function releaseMilestonePayment(uint256 _contractId, uint256 _milestoneIndex) external onlyRegistered {
        WorkContract storage workContract = contracts[_contractId];
        require(workContract.client == msg.sender, "Only client can release payment");
        require(workContract.status == ContractStatus.Started, "Contract not in started state");
        require(_milestoneIndex < workContract.milestones, "Invalid milestone index");
        require(workContract.milestoneDetails[_milestoneIndex].completed, "Milestone not completed");
        require(!workContract.milestoneDetails[_milestoneIndex].paid, "Milestone already paid");

        Milestone storage milestone = workContract.milestoneDetails[_milestoneIndex];
        uint256 platformFeeAmount = (milestone.amount * platformFee) / 10000; // Calculate platform fee
        uint256 paymentAmount = milestone.amount - platformFeeAmount; // Calculate freelancer payment

        milestone.paid = true;

        // Transfer payment to freelancer
        (bool success, ) = workContract.freelancer.call{value: paymentAmount}("");
        require(success, "Transfer to freelancer failed");

        // Transfer platform fee to owner
        (bool platformSuccess, ) = owner.call{value: platformFeeAmount}("");
        require(platformSuccess, "Transfer of platform fee failed");

        emit MilestonePaid(_contractId, _milestoneIndex, milestone.amount);

        // Check if all milestones are completed and paid
        bool allCompleted = true;
        for (uint256 i = 0; i < workContract.milestones; i++) {
            if (!workContract.milestoneDetails[i].paid) {
                allCompleted = false;
                break;
            }
        }

        // If all milestones are completed and paid, complete the contract
        if (allCompleted) {
            workContract.status = ContractStatus.Completed;
            workContract.completedAt = block.timestamp;
            workContract.endDate = block.timestamp;
            
            // Update job status
            jobs[workContract.jobId].status = JobStatus.Completed;
            
            // Add job to completed jobs for both client and freelancer
            profiles[workContract.client].completedJobs.push(workContract.jobId);
            profiles[workContract.freelancer].completedJobs.push(workContract.jobId);
            
            emit ContractCompleted(_contractId);
        }
    }

    // Function to cancel a contract (only client before work starts)
    function cancelContract(uint256 _contractId) external onlyRegistered {
        WorkContract storage workContract = contracts[_contractId];
        require(
            workContract.client == msg.sender || workContract.freelancer == msg.sender,
            "Only contract participants can cancel"
        );
        require(workContract.status == ContractStatus.Created || workContract.status == ContractStatus.Started, 
                "Cannot cancel contract in current state");

        // If contract has started (funds deposited), only refund if no milestones are paid
        if (workContract.status == ContractStatus.Started) {
            bool anyMilestonePaid = false;
            for (uint256 i = 0; i < workContract.milestones; i++) {
                if (workContract.milestoneDetails[i].paid) {
                    anyMilestonePaid = true;
                    break;
                }
            }
            require(!anyMilestonePaid, "Cannot cancel after milestones are paid");
            
            // Refund deposit to client
            (bool success, ) = workContract.client.call{value: workContract.deposit}("");
            require(success, "Refund to client failed");
        }

        workContract.status = ContractStatus.Cancelled;
        jobs[workContract.jobId].status = JobStatus.Cancelled;

        emit ContractCancelled(_contractId);
    }

    // Function to raise a dispute (only contract participants)
    function raiseDispute(uint256 _contractId, string memory _reason) external onlyRegistered onlyContractParticipant(_contractId) {
        WorkContract storage workContract = contracts[_contractId];
        require(workContract.status == ContractStatus.Started, "Contract not in started state");
        require(workContract.disputeStatus == DisputeStatus.None, "Dispute already raised");
        
        workContract.disputeStatus = DisputeStatus.InProgress;
        workContract.disputeReason = _reason;
        
        emit DisputeRaised(_contractId, _reason);
    }

    // Function to resolve a dispute (only owner - platform admin)
    function resolveDispute(
        uint256 _contractId, 
        DisputeStatus _resolution, 
        uint256[] memory _clientRefundMilestones,
        uint256[] memory _freelancerPaymentMilestones
    ) external onlyOwner {
        WorkContract storage workContract = contracts[_contractId];
        require(workContract.disputeStatus == DisputeStatus.InProgress, "No active dispute");
        require(
            _resolution == DisputeStatus.ResolvedForClient || _resolution == DisputeStatus.ResolvedForFreelancer,
            "Invalid resolution status"
        );

        workContract.disputeStatus = _resolution;
        
        // Process refunds to client
        for (uint256 i = 0; i < _clientRefundMilestones.length; i++) {
            uint256 milestoneIndex = _clientRefundMilestones[i];
            require(milestoneIndex < workContract.milestones, "Invalid milestone index");
            Milestone storage milestone = workContract.milestoneDetails[milestoneIndex];
            
            if (!milestone.paid) {
                uint256 refundAmount = milestone.amount;
                
                // Mark as paid to prevent double payments
                milestone.paid = true;
                
                // Transfer refund to client
                (bool success, ) = workContract.client.call{value: refundAmount}("");
                require(success, "Refund to client failed");
            }
        }
        
        // Process payments to freelancer
        for (uint256 i = 0; i < _freelancerPaymentMilestones.length; i++) {
            uint256 milestoneIndex = _freelancerPaymentMilestones[i];
            require(milestoneIndex < workContract.milestones, "Invalid milestone index");
            Milestone storage milestone = workContract.milestoneDetails[milestoneIndex];
            
            if (!milestone.paid && milestone.completed) {
                uint256 platformFeeAmount = (milestone.amount * platformFee) / 10000;
                uint256 paymentAmount = milestone.amount - platformFeeAmount;
                
                // Mark as paid to prevent double payments
                milestone.paid = true;
                
                // Transfer payment to freelancer
                (bool success, ) = workContract.freelancer.call{value: paymentAmount}("");
                require(success, "Payment to freelancer failed");
                
                // Transfer platform fee to owner
                (bool platformSuccess, ) = owner.call{value: platformFeeAmount}("");
                require(platformSuccess, "Transfer of platform fee failed");
            }
        }
        
        // Update contract status
        workContract.status = ContractStatus.Completed;
        workContract.completedAt = block.timestamp;
        workContract.endDate = block.timestamp;
        
        // Update job status
        jobs[workContract.jobId].status = JobStatus.Completed;
        
        emit DisputeResolved(_contractId, _resolution);
        emit ContractCompleted(_contractId);
    }

    // Function to rate a user after contract completion
    function rateUser(address _user, uint256 _rating, bool _isRatingClient) external onlyRegistered {
        require(_rating >= 0 && _rating <= 100, "Rating must be between 0 and 100");
        require(_user != msg.sender, "Cannot rate yourself");
        require(profiles[_user].isRegistered, "User not registered");
        
        // Verify that rater and rated have worked together
        bool hasWorkedTogether = false;
        if (_isRatingClient) {
            // Freelancer rating client
            require(profiles[_user].isClient, "Rated user is not a client");
            require(profiles[msg.sender].isFreelancer, "Only freelancers can rate clients");
            
            for (uint256 i = 0; i < freelancerContracts[msg.sender].length; i++) {
                uint256 contractId = freelancerContracts[msg.sender][i];
                if (contracts[contractId].client == _user && 
                    contracts[contractId].status == ContractStatus.Completed) {
                    hasWorkedTogether = true;
                    break;
                }
            }
        } else {
            // Client rating freelancer
            require(profiles[_user].isFreelancer, "Rated user is not a freelancer");
            require(profiles[msg.sender].isClient, "Only clients can rate freelancers");
            
            for (uint256 i = 0; i < clientContracts[msg.sender].length; i++) {
                uint256 contractId = clientContracts[msg.sender][i];
                if (contracts[contractId].freelancer == _user && 
                    contracts[contractId].status == ContractStatus.Completed) {
                    hasWorkedTogether = true;
                    break;
                }
            }
        }
        
        require(hasWorkedTogether, "You have not worked with this user");
        
        // Update ratings
        if (_isRatingClient) {
            // Update client rating
            Profile storage profile = profiles[_user];
            uint256 totalRating = profile.clientRating * profile.clientRatingCount;
            profile.clientRatingCount++;
            profile.clientRating = (totalRating + _rating) / profile.clientRatingCount;
        } else {
            // Update freelancer rating
            Profile storage profile = profiles[_user];
            uint256 totalRating = profile.freelancerRating * profile.freelancerRatingCount;
            profile.freelancerRatingCount++;
            profile.freelancerRating = (totalRating + _rating) / profile.freelancerRatingCount;
        }
        
        emit RatingSubmitted(_user, msg.sender, _rating, _isRatingClient);
    }

    // View functions
    function getProfileData(address _user) external view returns (
        string memory name,
        string memory description,
        bool isClient,
        bool isFreelancer,
        uint256 clientRating,
        uint256 clientRatingCount,
        uint256 freelancerRating,
        uint256 freelancerRatingCount,
        uint256 completedJobCount
    ) {
        Profile storage profile = profiles[_user];
        return (
            profile.name,
            profile.description,
            profile.isClient,
            profile.isFreelancer,
            profile.clientRating,
            profile.clientRatingCount,
            profile.freelancerRating,
            profile.freelancerRatingCount,
            profile.completedJobs.length
        );
    }

    function getProfileSkills(address _user) external view returns (string[] memory) {
        return profiles[_user].skills;
    }

    function getCompletedJobs(address _user) external view returns (uint256[] memory) {
        return profiles[_user].completedJobs;
    }

    function getJobProposals(uint256 _jobId) external view returns (uint256[] memory) {
        return jobs[_jobId].proposalIds;
    }

    function getClientJobs(address _client) external view returns (uint256[] memory) {
        return clientJobs[_client];
    }

    function getFreelancerProposals(address _freelancer) external view returns (uint256[] memory) {
        return freelancerProposals[_freelancer];
    }

    function getContractDetails(uint256 _contractId) external view returns (
        uint256 id,
        uint256 jobId,
        uint256 proposalId,
        address client,
        address freelancer,
        uint256 amount,
        uint256 deposit,
        ContractStatus status,
        uint256 startDate,
        uint256 endDate,
        uint256 completedMilestones,
        uint256 totalMilestones,
        DisputeStatus disputeStatus
    ) {
        WorkContract storage workContract = contracts[_contractId];
        return (
            workContract.id,
            workContract.jobId,
            workContract.proposalId,
            workContract.client,
            workContract.freelancer,
            workContract.amount,
            workContract.deposit,
            workContract.status,
            workContract.startDate,
            workContract.endDate,
            workContract.completedMilestones,
            workContract.milestones,
            workContract.disputeStatus
        );
    }
    // Function to get job details
    function getJobDetails(uint256 _jobId) external view returns (
        uint256 id,
        address client,
        string memory title,
        string memory description,
        string[] memory requiredSkills,
        uint256 budget,
        uint256 deadline,
        JobStatus status,
        uint256[] memory proposalIds,
        uint256 selectedProposal,
        uint256 contractId,
        bool isFixed,
        uint256 createdAt
        ) {
        Job storage job = jobs[_jobId];
        return (
            job.id,
            job.client,
            job.title,
            job.description,
            job.requiredSkills,
            job.budget,
            job.deadline,
            job.status,
            job.proposalIds,
            job.selectedProposal,
            job.contractId,
            job.isFixed,
            job.createdAt
        );
    }

    function getMilestoneDetails(uint256 _contractId, uint256 _milestoneIndex) external view returns (
        string memory description,
        uint256 amount,
        bool completed,
        bool paid
    ) {
        require(_milestoneIndex < contracts[_contractId].milestones, "Invalid milestone index");
        Milestone storage milestone = contracts[_contractId].milestoneDetails[_milestoneIndex];
        return (
            milestone.description,
            milestone.amount,
            milestone.completed,
            milestone.paid
        );
    }

    // Emergency withdrawal function for platform owner
    function emergencyWithdraw() external onlyOwner {
        uint256 balance = address(this).balance;
        require(balance > 0, "No funds to withdraw");
        (bool success, ) = owner.call{value: balance}("");
        require(success, "Withdrawal failed");
    }

    // Function to get contract between specific client and freelancer for a job
    function getContractByJob(uint256 _jobId) external view returns (uint256) {
        return jobs[_jobId].contractId;
    }

    // Fallback and receive functions
    receive() external payable {}
    fallback() external payable {}
}

```

### 🧾 File Header & License

```solidity

// SPDX-License-Identifier: UNLICENSED

```

This line declares the license type. `UNLICENSED` means you're **not granting others rights to use or copy** the code by default.

```solidity

pragma solidity ^0.8.19;

```

Specifies the **Solidity compiler version** requirement — this contract must be compiled with version `0.8.19` or higher but less than `0.9.0`.

---

### 🏗 Contract Declaration

```solidity

contract FreelancePlatform {

```

Defines a new contract named `FreelancePlatform`. This is like a **class** in other languages (e.g., Java, Python), and contains **variables, functions, and structs**.

---

### 🔐 State Variables

```solidity

address public owner;

```

Stores the address of the contract's creator/owner. `public` generates a getter automatically.

```solidity

uint256 public platformFee;
uint256 public jobCount;
uint256 public proposalCount;
uint256 public contractCount;

```

- `uint256`: 256-bit unsigned integer (common in Solidity).
- These variables track overall **platform configuration** and counters for IDs.

---

### 🧾 Enums

```solidity

enum JobStatus { Open, InProgress, Completed, Cancelled }

```

Defines possible statuses for a job — like an **enum in C/C++/Java**. Each value is assigned a number (starting from 0).

Similarly:

```solidity

enum ContractStatus { Created, Started, Completed, Cancelled, Disputed }
enum DisputeStatus { None, InProgress, ResolvedForClient, ResolvedForFreelancer }

```

---

### 📦 Structs (Custom Data Types)

These are **like objects** or records that hold multiple related fields.

### 📁 Profile

```solidity

struct Profile {
    string name;
    string description;
    string[] skills;
    bool isRegistered;
    bool isClient;
    bool isFreelancer;
    ...
}

```

Each user has a profile with basic info, roles (client/freelancer), and ratings.

### 📋 Job

```solidity

struct Job {
    uint256 id;
    address client;
    string title;
    ...
}

```

Each job posted has a unique `id`, a `client` address, description, skills required, status, and tracking fields.

### 💬 Proposal

```solidity

struct Proposal {
    uint256 id;
    uint256 jobId;
    address freelancer;
    ...
}

```

This represents an offer by a freelancer to do a job.

### 📜 WorkContract

```solidity

struct WorkContract {
    uint256 id;
    ...
    mapping(uint256 => Milestone) milestoneDetails;
}

```

Represents a **binding agreement** between client and freelancer, includes payment info, status, and milestones.

### ✅ Milestone

```solidity

struct Milestone {
    string description;
    uint256 amount;
    bool completed;
    bool paid;
}

```

Each milestone is a part of a contract’s work — it defines **partial goals**, helping break down larger tasks.

---

### 🔗 Mappings

Mappings are like **hash maps** or **dictionaries**:

```solidity

mapping(address => Profile) public profiles;

```

Maps each address to a `Profile`.

Others include:

- `mapping(uint256 => Job)` → job ID to Job struct
- `mapping(address => uint256[])` → address to a list of job/contract IDs

---

### 🔊 Events

```solidity

event ProfileCreated(address indexed user, bool isClient, bool isFreelancer);

```

Events are **logs** emitted to the blockchain. Off-chain apps (like UIs or subgraphs) listen to them.

`indexed` allows filtering by that field.

---

### 🛡 Modifiers

Modifiers are **reusable checks** before function logic runs:

```solidity

modifier onlyOwner() {
    require(msg.sender == owner, "Only owner can call this function");
    _;
}

```

- `require(...)` halts execution if condition fails.
- `_` is a placeholder — the wrapped function’s code goes here.

Examples:

- `onlyRegistered`
- `onlyFreelancer`
- `onlyJobClient(uint256 _jobId)` → requires sender to be the job’s creator

---

### 🏗 Constructor

```solidity

constructor() {
    owner = msg.sender;
    platformFee = 100;
    ...
}

```

Runs **once** when the contract is deployed. Sets:

- `owner` = address that deployed the contract.
- `platformFee` = 100 (basis points = 1%).

---

### ⚙ Functions (Partially implemented)

### 📌 `updatePlatformFee`

```solidity

function updatePlatformFee(uint256 _newFee) external onlyOwner {
    require(_newFee <= 1000, "Fee cannot exceed 10%");
    platformFee = _newFee;
}

```

- `external` = can be called from outside only.
- `onlyOwner` = only owner can change the platform fee.

### 👤 `registerProfile`

```solidity

function registerProfile(...) external {
    require(!profiles[msg.sender].isRegistered, "Already registered");
    require(_isClient || _isFreelancer, "Must choose role");
    ...
}

```

Registers a new user profile. Partially written — needs assignment and event emit.

---

That’s the full breakdown **up to where the code stops**.

---

### 🧠 Summary: Solidity Concepts Used

| Concept | Meaning |
| --- | --- |
| `contract` | Smart contract definition |
| `struct` | Group multiple variables under one name |
| `enum` | Finite list of named options |
| `mapping` | Key-value storage |
| `modifier` | Pre-condition logic wrappers |
| `require()` | Ensures conditions are met |
| `msg.sender` | The caller's address |
| `address` | Ethereum wallet address |
| `uint256` | 256-bit unsigned integer |
| `event` | Blockchain log for tracking actions |
| `constructor()` | Runs once at deployment |
| `external` | Function visible externally |
| `public` | Accessible from everywhere + getter |
| `bool` | Boolean true/false |