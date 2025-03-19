# PinDAO: A Community-Driven IPFS Pinning Protocol 📌🌐

## Abstract 🔬

PinDAO is a decentralized protocol designed to incentivize distributed file persistence on the InterPlanetary File System (IPFS) through tokenized rewards on EVM-compatible blockchains. By leveraging reciprocal pinning and an auction-based queue system, PinDAO ensures reliable file availability while generating value for node operators and DAO participants.

## Introduction 🌍

IPFS lacks a native economic model for long-term file persistence. Many centralized pinning services exist, but undermine the principles of decentralization that distributed technology is founded upon. Other solutions like Filecoin and Arweave have stepped in to fill this gap, but they operate on their own siloed networks outside of EVM blockchains which often utilize IPFS for storage.

PinDAO introduces an EVM-native solution where nodes are rewarded for pinning files while enabling users to prioritize content storage through market mechanisms. Running PinDAO infrastructure directly on EVM networks enhances composability, accessibility, and efficiency for smart contracts that already rely on IPFS storage.

## Benefits of EVM Integration 🤖

Unlike other solutions for IPFS persistence like Filecoin that operate on a proprietary network, PinDAO's integration with EVM chains offers notable advantages:

### 1. Direct Smart Contract Composability

- Many NFT platforms, DAOs, and DeFi projects already store metadata and assets on IPFS, with smart contracts referencing these CIDs.
  - For example, an NFT minting app can upload a file to IPFS on the frontend then automatically trigger a pinning request and attach a bounty for nodes to ensure its metadata persists.
- Contracts can interact with pinning directly via on-chain logic, eliminating reliance on third-party services.
- Chain-of-custody for IPFS uploads that originated onchain.

### 2. Permissionless Integration

- EVM dApps can call PinDAO's smart contract directly, whereas Filecoin requires custom APIs and deal negotiations.
- Any smart contract can use PinDAO's auction mechanism to prioritize pinning in a trustless manner.
- Accounts can directly pay nodes for storage costs onchain.
- A DAO managing files on IPFS can programmatically allocate treasury funds to ensure long-term availability.

### 3. DeFi Ecosystem Compatibility

- $PIN tokenomics can integrate with DeFi protocols for staking, lending, and yield farming.
- Liquidity Pools allow users to hedge against storage costs.

### 4. Real-Time Settlement and Participation

- Filecoin requires complex storage deals involving bidding, miner selection, and off-chain computation.
- Platforms can use auction-based pinning to ensure content is widely replicated immediately.

### 5. Democratized Storage Participation

- PinDAO allows smaller distributed nodes to contribute storage without complex deal negotiations or additional overhead required with protocols like Filecoin.
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

## Core Mechanisms 🛠️

### Wallet-Linked IPFS Nodes

- Each participating IPFS node must connect an EVM wallet.
- Nodes must stake a minimum amount of $PIN tokens to participate in the protocol.
- Nodes receive $PIN tokens for pinning files from a shared queue.
- Smart contracts track node participation, staking, and rewards distribution.

### Reciprocal Exchange & Reputation System

- Nodes who pin files in proportion to their own requested storage receive a higher reputation.
- The reputation system penalizes nodes that unpin files prematurely.
- Higher reputation nodes receive better queue priority and higher rewards.
- Nodes with repeated misbehavior can be temporarily or permanently banned from the network.

### Auction-Based Pinning Queue

- Users who need guaranteed pinning can bid $PIN tokens for priority storage.

### Direct IPFS Node Rewards

- Users can pay nodes directly in $PIN to secure long-term pinning, creating a decentralized marketplace for storage beyond the reciprocal pinning model.

### 5. Escrow & Protection Mechanism

- All pinning payments vest linearly over the pinning duration to prevent "take the money and run" scenarios.
- If a node unpins content prematurely, 95% of remaining escrowed tokens are returned to the payer, and 5% is awarded to the user who discovers and reports the misbehavior.
  - This incentivizes community policing and ensures nodes are held accountable for their commitments.

### 6. Node Banning Mechanism

- Nodes that consistently fail to meet protocol requirements can be banned from participating.
- Banning is progressive:

  🟡 **First offense:** Short-term ban (e.g., 1 week)

  🟠 **Repeated offenses:** Increasingly longer bans

  🔴 **Severe or multiple serious violations can result in permanent exclusion**

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
    address wallet;              // Wallet address of node operator
    string ipfsPeerId;           // Unique identifier for node
    uint256 reputation;          // Cumulative score representing node reliability
    uint256 stakedAmount;        // Total amount of tokens locked as collateral
    uint256 autoStakePercentage; // Percentage of earned rewards automatically re-staked
    uint256 lastActive;          // Timestamp of recent network activity
    bool isActive;               // True if node is currently eligible
    uint256 bannedUntil;         // Timestamp when temporary ban expires (0 if not banned)
    uint256 banCount;            // # times node has been banned
}
```

### File Pinning Queue & Auctions

```solidity
struct FilePinRequest {
    bytes32 cid;        // Content Identifier (CID) of file to be pinned
    address requester;  // Address of user or contract requesting pin
    uint256 duration;   // Total time period for which file should remain pinned
    uint256 minReward;  // Minimum token reward required
    uint256 bidAmount;  // Actual amount of $PIN tokens bid for prioritizing request
    uint256 createdAt;  // Timestamp when pin request was submitted to queue
    bool fulfilled;     // True if pin request has been successfully assigned to a node
}
```

### Node Pin Assignments & Rewards

```solidity
struct NodePinAssignment {
    bytes32 cid;            // CID of pinned file
    address node;           // Address of node responsible for pinning
    uint256 pinTimestamp;   // Timestamp when file was initially pinned
    uint256 duration;       // Agreed-upon duration for maintaining pin
    uint256 totalReward;    // Total $PIN tokens allocated for this assignment
    uint256 claimedReward;  // Amount of rewards already claimed through vesting schedule
    bool completed;         // True if pin assignment has been fully fulfilled
}
```

### Purchase Pinning Directly

```solidity
struct DirectPayment {
    address user;           // Address of user paying for pinning
    address node;           // Address of node providing pinning service
    bytes32 cid;            // Content Identifier of file being pinned
    uint256 amount;         // Total amount of $PIN tokens paid for pinning
    uint256 duration;       // Agreed-upon duration for maintaining pin
    uint256 startTimestamp; // Timestamp when direct payment and pinning began
    uint256 claimedAmount;  // Amount of rewards already claimed through vesting schedule
    bool completed;         // True if direct payment has been fully processed
}
```

### Vesting & Escrow

```solidity
struct VestingSchedule {
    address beneficiary;    // Address of node receiving vested tokens
    address payer;          // Address of user who initiated the payment
    bytes32 cid;            // CID associated with this vesting schedule
    uint256 startTimestamp; // Timestamp when vesting schedule begins
    uint256 endTimestamp;   // Timestamp when vesting schedule completes
    uint256 totalAmount;    // Total amount of $PIN tokens to be vested
    uint256 claimedAmount;  // Amount of tokens already claimed by the beneficiary
    bool active;            // Indicates if vesting schedule is currently valid
}
```

## Smart Contract Functions

### Node Registration & Staking

Nodes must register with the PinDAO contract and stake a minimum amount of $PIN tokens to begin receiving requests.

```solidity
function registerNode(string memory ipfsPeerId, uint256 stakeAmount) external;
function setAutoStakePercentage(uint256 percentage) external;
function stakeAdditional(uint256 amount) external;
function unstake(uint256 amount) external;
```

### Submit Pin Request

Auction requests to pin content are submitted to the queue and can come from any address (EOA or contract) who provides a valid CID. Payment (in $PIN) is included with the request, and returned if unfulfilled.

```solidity
function submitPinRequest(bytes32 cid, uint256 duration, uint256 minReward, uint256 bidAmount) external;
```

### Assign Pin Request to Nodes

PinDAO contracts will assign a requested pin to a node, whether decided through an auction or direct payment to a node. This creates a vesting schedule for the payment.

```solidity
function assignPin(bytes32 cid, address node, uint256 reward, uint256 duration) internal;
```

### Verify Pinning

Allows any participant to verify that content is still being pinned. This should be processed through a service that atomically queries IPFS and verifies content backed by PinDAO.

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

## Verifier Services 🕵️‍♀️

The structure of PinDAO provides an economic opportunity for participants called **Verifiers** who (you guessed it) verify the integrity of nodes providing pinning services and maintain network integrity by:

- 🔍 Checking IPFS pins against expected PinDAO nodes
- ✅ Verifying content availability and persistence
- 🚨 Reporting nodes that fail to meet pinning commitments
- 🏆 Earning rewards for maintaining network health

### How Verifier Services Work

1. **Automated Monitoring**: Verifiers continuously scan the network, checking the actual availability of pinned content on IPFS
2. **Reputation Enforcement**: Automatically trigger reputation mechanisms for nodes that fail to maintain pins
3. **Economic Incentives**: Earn a percentage of slashed tokens when reporting misbehaving nodes

Potential use cases include services that monitor PinDAO node performance, open-source verification scripts, and comprehensive pin tracking and reporting systems.

> 💡 This opens up a new market for developers to create value-added services around the PinDAO ecosystem, turning network monitoring into a potential revenue stream.

## Security & Sustainability

- **Stake-Based Participation**: Required staking ensures nodes have skin in the game, reducing Sybil attacks.
- **Linear Vesting Model**: Rewards vest over time, aligning node operators with long-term file persistence.
- **Community Policing**: The 5% bounty for reporting unpinned content creates a decentralized verification network.
- **Smart Contract Verification**: All transactions and reward mechanisms operate transparently on-chain.
- **Economic Penalties**: Slashing mechanisms ensure significant economic consequences for malicious behavior.
- **Progressive Node Banning**: Nodes that repeatedly violate protocol rules face escalating penalties, preventing persistent bad actors from exploiting the network.
- **Decentralized Arbitration**: Disputes over uptime and availability are resolved via community governance.

## Protocol Challenges 🏗️

As with any new protocol, there are several key challenges that will need to be addresses. You can find a breakdown of the ones identified so far, their current status, and proposed solutions [here](CHALLENGES.md).

Key challenge areas include:

- 🖥️ Technical Challenges
- 💸 Economic Challenges
- 🤝 Governance Challenges
- 🔗 Integration Challenges
- 🌐 Competitive Landscape Considerations

[Read the Full Challenges Document →](CHALLENGES.md)

## Community Collaboration 🤝

PinDAO has many opportunities for ongoing research, development, and community collaboration. We welcome contributions from:

- 🔬 Researchers
- 💻 Developers
- 🌐 Community members
