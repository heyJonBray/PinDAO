# PinDAO: A Community-Driven IPFS Pinning Protocol

## Abstract

PinDAO is a decentralized protocol designed to incentivize distributed file persistence on the InterPlanetary File System (IPFS) through tokenized rewards on EVM-compatible blockchains. By leveraging a pin-for-pin mechanism and an auction-based queue system, PinDAO ensures reliable file availability while generating intrinsic value for its underlying token ($PIN).

## Introduction

IPFS lacks a native economic model for long-term file persistence. While centralized pinning services exist, they undermine the principles of decentralization that distributed technology is founded upon. While decentralized solutions like Filecoin exist, they operate on their own siloed networks outside of blockchains that utilize IPFS for storage. PinDAO introduces a tokenized, trustless solution where nodes are rewarded for pinning files while enabling users to prioritize content storage through market-driven mechanisms. Integrating PinDAO directly on EVM-compatible networks enhances composability, accessibility, and efficiency for smart contracts that rely on IPFS storage.

## Benefits of EVM Integration

Unlike solutions like Filecoin that operate on separate networks, PinDAO's integration with EVM-compatible blockchains offers distinct advantages:

### 1. Direct Smart Contract Composability

- Many NFT platforms, DAOs, and DeFi projects already store metadata and assets on IPFS, with smart contracts referencing these CIDs.
- By running on EVM, contracts can interact with pinning directly via on-chain logic, eliminating reliance on third-party services.
- For example, an NFT minting contract can automatically trigger a pinning request and attach a bounty for nodes to ensure its metadata persists.

### 2. Permissionless Integration

- EVM dApps can call PinDAO's smart contract directly, whereas Filecoin requires custom APIs and deal negotiations.
- Any smart contract can use PinDAO's auction mechanism to prioritize pinning in a trustless way.
- A DAO managing research papers on IPFS can programmatically allocate treasury funds to ensure long-term availability.

### 3. DeFi Ecosystem Compatibility

- $PIN tokenomics can integrate with DeFi protocols for staking, lending, and yield farming.
- Liquidity pools allow users to hedge against storage costs, while Filecoin's economic model relies more on off-chain agreements.
- Lending protocols could offer reduced collateral requirements for users staking $PIN tokens.

### 4. Real-Time Settlement and Participation

- Filecoin requires complex storage deals involving bidding, miner selection, and off-chain computation.
- On EVM, storage payments and rewards settle in real-time, providing a more fluid and predictable system.
- Content platforms can use PinDAO's auction-based pinning to ensure viral content is widely replicated immediately.

### 5. Democratized Storage Participation

- Filecoin's storage model favors large storage providers due to its technical requirements and long-term contracts.
- PinDAO allows smaller, distributed nodes to contribute storage without complex deal negotiations.
- Individual users running home IPFS nodes can earn rewards trustlessly rather than competing against industrial-scale miners.

### Feature Comparison: PinDAO vs. Filecoin

| Feature                          | PinDAO (EVM)                      | Filecoin                                 |
| -------------------------------- | --------------------------------- | ---------------------------------------- |
| Integration with Smart Contracts | Native & permissionless           | Requires off-chain agreements            |
| Instant Token Payments           | Yes, through $PIN on-chain        | Requires Filecoin deal flow              |
| Composability with DeFi          | Can stake, lend, yield farm $PIN  | No DeFi integration                      |
| Storage Model                    | P2P with auction flexibility      | Long-term contract-based                 |
| Barriers to Entry                | Anyone with an IPFS node can earn | Storage miners require large-scale infra |
| Decentralization                 | Distributed individual nodes      | Primarily large storage providers        |

## Core Mechanisms

### 1. Wallet-Linked IPFS Nodes

- Each participating IPFS node must connect an EVM wallet.
- Nodes must stake a minimum amount of $PIN tokens to participate in the protocol.
- Nodes receive $PIN tokens for pinning files from a shared queue.
- Smart contracts track node participation, staking, and rewards distribution.

### 2. Pin-for-Pin Exchange & Reputation System

- Nodes must pin files in proportion to their own requested storage.
- A reputation system penalizes nodes that unpin files prematurely.
- Nodes that automatically stake a percentage of their incoming $PIN payments receive reputation boosts.
- Higher reputation nodes receive better queue priority and higher rewards.
- Nodes with repeated misbehavior can be temporarily or permanently banned from the network.

### 3. Auction-Based Pinning Queue

- Users who need guaranteed pinning can bid $PIN tokens for priority storage.
- The higher the bid, the faster their files get distributed across nodes.
- Nodes that pin high-bid files receive bonus token rewards.
- All auction payments are held in escrow and vest linearly over the pinning duration.

### 4. Direct IPFS Node Rewards

- Users can pay nodes directly in $PIN to secure long-term pinning.
- All direct payments are held in escrow and vest linearly over the agreed pinning period.
- This creates a decentralized marketplace for storage beyond the pin-for-pin model.

### 5. Escrow & Protection Mechanism

- All pinning rewards vest linearly over the pinning duration to prevent "take payment and run" scenarios.
- If a node unpins content prematurely, 95% of remaining escrowed tokens are returned to the payer.
  - 5% of remaining tokens are awarded to the user who discovers and reports the misbehavior, creating a bounty system for network monitoring.
  - This incentivizes community policing and ensures nodes are held accountable for their commitments.

### 6. Node Banning Mechanism

- Nodes that consistently fail to meet protocol requirements can be banned from participating.
- Banning is progressive:
  - First offense: Short-term ban (e.g., 1 week)
  - Repeated offenses: Increasingly longer bans
  - Severe or multiple serious violations can result in permanent exclusion
- Banned nodes:
  - Lose access to the pinning queue
  - Forfeit pending rewards
  - Have their stake significantly reduced or completely slashed
- The ban duration increases exponentially with each subsequent ban
- Nodes can appeal bans through a decentralized governance process

## On-Chain Smart Contract Design

### Node Registration & Reputation

```solidity
struct Node {
    address wallet;              // Ethereum wallet address of the IPFS node operator
    string ipfsPeerId;           // Unique identifier for the IPFS node in the network
    uint256 reputation;          // Cumulative score representing the node's reliability and performance
    uint256 stakedAmount;        // Total amount of $PIN tokens locked as collateral in the protocol
    uint256 autoStakePercentage; // Percentage of earned rewards automatically re-staked to boost reputation
    uint256 lastActive;          // Timestamp of the node's most recent network activity or pin assignment
    bool isActive;               // True if node is currently eligible to participate
    uint256 bannedUntil;         // Timestamp specifying when a temporary ban expires (0 if not banned)
    uint256 banCount;            // Number of times the node has been banned, used for escalating penalties
}
```

### File Pinning Queue & Auctions

```solidity
struct FilePinRequest {
    bytes32 cid;        // Content Identifier (CID) of the file to be pinned on IPFS
    address requester;  // Ethereum address of the user or contract requesting the pin
    uint256 duration;   // Total time period for which the file should remain pinned
    uint256 minReward;  // Minimum token reward required to incentivize node participation
    uint256 bidAmount;  // Actual amount of $PIN tokens bid for prioritizing this pin request
    uint256 createdAt;  // Timestamp when the pin request was submitted to the queue
    bool fulfilled;     // Indicates whether the pin request has been successfully assigned to a node
}
```

### Node Pin Assignments & Rewards

```solidity
struct NodePinAssignment {
    bytes32 cid;            // Content Identifier of the pinned file
    address node;           // Ethereum address of the node responsible for pinning
    uint256 pinTimestamp;   // Timestamp when the file was initially pinned
    uint256 duration;       // Agreed-upon duration for maintaining the pin
    uint256 totalReward;    // Total $PIN tokens allocated for this pin assignment
    uint256 claimedReward;  // Amount of rewards already claimed through the vesting schedule
    bool completed;         // Indicates if the pin assignment has been fully fulfilled
}
```

### Purchase Pinning Directly

```solidity
struct DirectPayment {
    address user;           // Ethereum address of the user paying for pinning
    address node;           // Ethereum address of the node providing the pinning service
    bytes32 cid;            // Content Identifier of the file being pinned
    uint256 amount;         // Total amount of $PIN tokens paid for pinning
    uint256 duration;       // Agreed-upon duration for maintaining the pin
    uint256 startTimestamp; // Timestamp when the direct payment and pinning began
    uint256 claimedAmount;  // Amount of tokens already claimed through the vesting schedule
    bool completed;         // Indicates if the direct payment has been fully processed
}
```

### Vesting & Escrow

```solidity
struct VestingSchedule {
    address beneficiary;    // Ethereum address of the node receiving vested tokens
    address payer;          // Ethereum address of the user who initiated the payment
    bytes32 cid;            // Content Identifier associated with this vesting schedule
    uint256 startTimestamp; // Timestamp when the vesting schedule begins
    uint256 endTimestamp;   // Timestamp when the vesting schedule completes
    uint256 totalAmount;    // Total amount of $PIN tokens to be vested
    uint256 claimedAmount;  // Amount of tokens already claimed by the beneficiary
    bool active;            // Indicates if the vesting schedule is currently valid
}
```

## Smart Contract Functions

### Node Registration & Staking

Nodes must register with the PinDAO contract and stake a minimum amount of $PIN tokens to join the protocol.

```solidity
function registerNode(string memory ipfsPeerId, uint256 stakeAmount) external;
function setAutoStakePercentage(uint256 percentage) external;
function stakeAdditional(uint256 amount) external;
function unstake(uint256 amount) external;
```

### Submit Pin Request

Auction requests to pin content are submitted to the queue can come from anyone who provides a valid CID. Payment (in $PIN) is included when the request is made.

```solidity
function submitPinRequest(bytes32 cid, uint256 duration, uint256 minReward, uint256 bidAmount) external;
```

### Assign Pin Request to Nodes

Core PinDAO contract assigns a requested pin to a node, whether decided through an auction or direct payment to a node. This creates a vesting schedule for the payment.

```solidity
function assignPin(bytes32 cid, address node, uint256 reward, uint256 duration) internal;
```

### Verify Pinning

⚠️ Frequency of verification TBD. Allows any participant to verify that content is still being pinned.

```solidity
function verifyPin(bytes32 cid, address node) external;
```

### Claim Vested Rewards

Nodes can claim their vested rewards as they accrue over time.

```solidity
function claimVestedRewards(bytes32 cid) external;
function claimAllVestedRewards() external;
```

### Slash Nodes for Early Unpinning

If nodes are found to be unpinning content before the agreed upon expiry, their future rewards and staked $PIN will be slashed. 95% of remaining tokens are returned to the payer, and 5% are awarded to the reporter.

```solidity
function reportUnpinnedContent(address node, bytes32 cid) external;
function slashNode(address node, bytes32 cid, address reporter) internal;
```

### Direct Payments for Pinning

Users who require pinned content can pay nodes directly to pin their content. The amount is agreed upon by both parties and held in escrow, vesting linearly.

```solidity
function payForPinning(address node, bytes32 cid, uint256 amount, uint256 duration) external;
```

### Node Banning Mechanisms

Provides functions to manage node banning for protocol violations.

```solidity
function banNode(address node, uint256 duration) external;

// Check if a node is currently banned
function isNodeBanned(address node) external view returns (bool);

// Get the remaining ban duration for a node
function getBanDuration(address node) external view returns (uint256);

// Allow a banned node to appeal their ban
function appealBan() external;

// Governance function to review and potentially lift or modify a ban
function reviewNodeBan(address node, bool shouldLiftBan) external;
```

### Node Reputation Management

```solidity
// Increase a node's reputation for good behavior
function increaseNodeReputation(address node, uint256 amount) internal;

// Decrease a node's reputation for violations
function decreaseNodeReputation(address node, uint256 amount) internal;

// Get current node reputation
function getNodeReputation(address node) external view returns (uint256);
```

## Security & Sustainability

- **Stake-Based Participation**: Required staking ensures nodes have skin in the game, reducing Sybil attacks.
- **Linear Vesting Model**: Rewards vest over time, aligning node operators with long-term file persistence.
- **Community Policing**: The 5% bounty for reporting unpinned content creates a decentralized verification network.
- **Smart Contract Verification**: All transactions and reward mechanisms operate transparently on-chain.
- **Economic Penalties**: Slashing mechanisms ensure significant economic consequences for malicious behavior.
- **Progressive Node Banning**:
  - Nodes that repeatedly violate protocol rules face escalating penalties
  - Banning mechanism provides a graduated response to misconduct
  - Prevents persistent bad actors from exploiting the network
- **Decentralized Arbitration**: Disputes over uptime and availability are resolved via community governance.
- **Reputation-Based Access**: Node participation is dynamically controlled by their historical performance and adherence to protocol rules.
