# PinDAO: A Decentralized ETH-Based IPFS Pinning Protocol 📌🌐

## Abstract

PinDAO is a decentralized protocol designed to incentivize persistent file storage on the InterPlanetary File System (IPFS) through direct ETH payments on Ethereum and compatible blockchains. By leveraging a decentralized network of storage providers, a fair node rotation queue system, and a reputation-based reward mechanism, PinDAO ensures reliable file availability and contract-level access to IPFS nodes for increased composability.

## Introduction

IPFS has emerged as a powerful distributed file system, but it lacks native economic incentives to ensure long-term file persistence. When a file is uploaded to IPFS, there's no guarantee it will remain accessible unless someone actively "pins" it to their node.

Current solutions fall into three categories:

1. Centralized pinning services (creating single points of failure)
2. Custom blockchain networks like Filecoin and Arweave (requiring migration to separate ecosystems)
3. Self-hosted infrastructure (resource-intensive and technically complex)

PinDAO introduces a simple, Ethereum-native solution where IPFS node operators are directly rewarded in ETH for providing reliable pinning services, while users can prioritize their content through market-based mechanisms, all without requiring a separate token economy.

## Core Mechanisms 🛠️

### Wallet-Linked IPFS Nodes

Each participating IPFS node connects an Ethereum wallet and stake ETH as collateral to participate in the protocol. Nodes receive ETH payments for pinning files from users, with smart contracts tracking node participation, staking, and ensuring service delivery through community verification of pinned files.

### Fair Queue System

Participating nodes are placed in a rotation queue to receive pinning assignments. After completing a job, the node moves to the back of the queue to ensure all participating nodes have fair access to earning opportunities. The base ETH cost per file is calculated based on file size and requested duration.

### Reputation-Based Reward Distribution

Nodes build reputation based on reliability and service quality which determines the percentage of payment received from pin requests:

- Highest reputation nodes receive 99% of payment
- Lower reputation nodes receive proportionally less (down to 50%)

Remaining percentage goes to the DAO treasury, with the DAO taking a base 1% of protocol revenue.

### Dual Pricing Models

PinDAO supports two complementary pricing models:

#### 1. Standard Queue-Based System

Users submit pinning requests with required ETH payment based on file size and duration. The protocol automatically assigns files to the next node(s) in the queue based on the specified replication factor.

#### 2. Direct Node Payments

Users can bypass the queue and pay nodes directly in ETH for long-term pinning, creating a decentralized marketplace where users and nodes negotiate terms. In direct agreements, node reputation does not affect payment percentage.

### Escrow & Protection Mechanism

Funds are held in escrow and are vested linearly for the duration that pinning service is provided for. If a node unpins content prematurely, the remaining escrowed ETH is returned to the payer with 5% of returned funds being awarded to users who report violations, creating community incentives to report bad actors.

### Node Banning Mechanism

Nodes that consistently fail to meet protocol requirements face progressive penalties to reputation and earnings:

🟡 **First offense:** Short-term ban. First offenses do not incur a reputation strike.

🟠 **Repeated offenses:** Increasingly longer ban and reputation hit. The more a node misbehaves, the lower it's reputation gets (down to 50%).

🔴 **Severe violations:** Perma-ban and slashing of staked ETH.

Bans are enforced transparently onchain, and appeals can be made through a decentralized governance process.

✂️ Slashed ETH goes to the DAO treasury

## Verifier Services 🕵️‍♀️

The PinDAO protocol creates economic opportunities for **Verifiers** who maintain network integrity by checking IPFS for content availability promised by nodes and verifying that pinning commitments are being honored.

**Verifiers:**
🚨 Reporting nodes that fail to meet obligations
🏆 Earning ETH rewards (5% of remaining escrow) for maintaining network health

### How Verifier Services Work

1. **Automated Monitoring**: Verifiers scan the network, checking actual availability of pinned content
2. **Reputation Enforcement**: Trigger the protocol's reputation mechanisms for failing nodes
3. **Economic Incentives**: Earn a percentage of returned funds when reporting violations

This creates a market for developers to build services around PinDAO, such as IPFS availability monitors, node reputation tracking dashboards, and analytics tools,.

## DAO Treasury and Fees

PinDAO collects fees that go directly to the DAO treasury:

1. **Queue-Based Assignments**: A percentage of each payment based on node reputation:

   - 1% from highest reputation nodes (99% to node)
   - Up to 50% from lowest reputation nodes (50% to node)

2. **Direct Payments**: Flat 1% fee on all direct agreements between users and nodes

## Comparison with Other Storage Solutions

| Feature                        | PinDAO                       | Filecoin                    | Arweave            | Centralized Pinning |
| ------------------------------ | ---------------------------- | --------------------------- | ------------------ | ------------------- |
| **Native Currency**            | ETH                          | FIL                         | AR                 | Various             |
| **Smart Contract Integration** | Direct                       | Limited                     | Limited            | None                |
| **Barrier to Entry**           | Low (standard IPFS node)     | High (specialized hardware) | Medium             | N/A                 |
| **Payment Model**              | Queue-based + direct payment | Long-term contracts         | One-time perpetual | Subscription        |
| **Node Selection**             | Fair queue rotation          | Auction/manual deals        | Miners compete     | Centralized         |
| **Storage Verification**       | Community-driven             | Proof-of-Spacetime          | Proof-of-Access    | Centralized         |
| **Content Addressing**         | IPFS CIDs                    | IPFS CIDs                   | Transaction-based  | Proprietary         |
| **Node Economics**             | Reputation-based rewards     | Complex deal structure      | Endowment-based    | Fixed pricing       |
| **Censorship Resistance**      | High                         | High                        | Very High          | Low                 |

### Why Choose PinDAO?

1. **Ethereum Native**: Uses ETH directly, no need for token swaps or bridges
2. **Fair Distribution**: Queue-based system ensures all nodes get opportunities
3. **Incentive Alignment**: Reputation affects rewards, not job access
4. **Low Barrier**: Anyone with an IPFS node can participate and earn
5. **Transparent**: All commitments and payments are visible on-chain
6. **Sustainable**: DAO treasury ensures long-term protocol development

## Security & Sustainability

- **Stake-Based Participation**: Required ETH staking ensures nodes have skin in the game
- **Linear Vesting Model**: Payments release over time, aligning incentives for long-term storage
- **Community Policing**: The 5% bounty for reporting violations creates distributed oversight
- **Transparent Queue**: Fair job distribution prevents centralization
- **Reputation Economics**: Better performance leads to higher earnings percentage
- **Progressive Banning**: Repeat offenders face escalating penalties
- **Sustainable Treasury**: Protocol fees fund ongoing development and maintenance

---

## Smart Contract Design

Below is an outline of proposed smart contract design for PinDAO. This code is not tested and should not be used in a production environment.

### Node Registration & Queue Management

Structure of an onchain IPFS node

```js
struct Node {
    address payable wallet;   // Wallet address of node operator
    string ipfsPeerId;        // Unique identifier for IPFS node
    uint256 reputation;       // Score representing node reliability (0-100)
    uint256 stakedAmount;     // Total ETH locked as collateral
    uint256 lastActive;       // Timestamp of most recent activity
    bool isActive;            // Whether node is currently eligible for assignments
    uint256 bannedUntil;      // Timestamp when temporary ban expires (0 if not banned)
    uint256 banCount;         // Number of times node has been banned
    uint256 totalEarned;      // Total ETH earned through the protocol
    mapping(bytes32 => bool) activePins;  // Currently pinned content
}
```

Implementation of the node queue system.

```js
address[] private nodeQueue;
mapping(address => uint256) private nodeQueuePosition;
```

New nodes register with their `peerId` by staking ETH and are placed at the end of the queue.

```js
function registerNode(string calldata ipfsPeerId) external payable {
    require(msg.value >= MINIMUM_STAKE, "Insufficient stake");
    require(nodeQueuePosition[msg.sender] == 0, "Already registered");

    nodes[msg.sender] = Node({
        wallet: payable(msg.sender),
        ipfsPeerId: ipfsPeerId,
        reputation: INITIAL_REPUTATION,
        stakedAmount: msg.value,
        lastActive: block.timestamp,
        isActive: true,
        bannedUntil: 0,
        banCount: 0,
        totalEarned: 0
    });

    // Add node to the end of the queue
    nodeQueue.push(msg.sender);
    nodeQueuePosition[msg.sender] = nodeQueue.length;

    emit NodeRegistered(msg.sender, ipfsPeerId, msg.value);
}
```

Nodes are rotated to the back of the queue after being assigned a pin request.

```js
function _rotateNodeInQueue(address node) internal {
    uint256 pos = nodeQueuePosition[node];
    require(pos > 0, "Node not in queue");

    // Remove node from current position
    for (uint i = pos; i < nodeQueue.length; i++) {
        nodeQueue[i-1] = nodeQueue[i];
        nodeQueuePosition[nodeQueue[i]] = i;
    }

    // Put node at end of queue
    nodeQueue[nodeQueue.length - 1] = node;
    nodeQueuePosition[node] = nodeQueue.length;
}
```

Contract gets the next available nodes from the queue

```js
function _getNextAvailableNodes(uint8 count) internal view returns (address[] memory) {
    address[] memory availableNodes = new address[](count);
    uint256 found = 0;

    for (uint i = 0; i < nodeQueue.length && found < count; i++) {
        address nodeAddr = nodeQueue[i];
        if (nodes[nodeAddr].isActive && nodes[nodeAddr].bannedUntil < block.timestamp) {
            availableNodes[found] = nodeAddr;
            found++;
        }
    }

    require(found == count, "Not enough available nodes");
    return availableNodes;
}
```

### File Pinning Requests & Pricing

Structure of an onchain pin request

```js
struct PinRequest {
    bytes32 cid;                 // Content Identifier (CID) of file
    address payable user;        // Address of requestor
    uint256 duration;            // Time period (in seconds) for pinning
    uint256 fileSize;            // Size of file in bytes
    uint256 payment;             // Total ETH payment
    uint256 createdAt;           // Timestamp when request was submitted
    bool fulfilled;              // Whether request has been assigned to nodes
    uint8 replicationFactor;     // Number of nodes to pin the content
}
```

The price of a pin request is determined based on file size and duration the pin is requested for. DAO fees can be updated by governance proposals but is initialized at 1%.

```js
function calculateBasePrice(uint256 fileSize, uint256 duration) public pure returns (uint256) {
    // Example pricing formula
    // Base price = (fileSize in MB * 0.0001 ETH) * (duration in days / 30)
    uint256 fileSizeInMB = fileSize / (1024 * 1024);
    uint256 durationInDays = duration / 86400;

    return (fileSizeInMB * 1e14 * durationInDays) / 30;
}
```

Pin requests are submitted with a `replicationFactor` that determines how many nodes should pin the file. Payment cost is `(fileSize * duration) * replicationFactor` and paid at the time of submission, after which the payment is split between the number of participating nodes and their respective fees (modified by reputation) are vested to them.

```js
function submitPinRequest(
    bytes32 cid,
    uint256 fileSize,
    uint256 duration,
    uint8 replicationFactor
) external payable {
    // Calculate minimum required payment
    uint256 basePrice = calculateBasePrice(fileSize, duration);
    uint256 totalPrice = basePrice * replicationFactor;

    require(msg.value >= totalPrice, "Insufficient payment");
    require(duration >= MIN_DURATION, "Duration too short");
    require(replicationFactor > 0, "Must request at least one node");

    uint256 requestId = _nextRequestId++;

    pinRequests[requestId] = PinRequest({
        cid: cid,
        user: payable(msg.sender),
        duration: duration,
        fileSize: fileSize,
        payment: msg.value,
        createdAt: block.timestamp,
        fulfilled: false,
        replicationFactor: replicationFactor
    });

    // Process the request immediately
    _processRequest(requestId);

    emit PinRequestSubmitted(requestId, cid, msg.sender, msg.value, duration);
}
```

### Node Assignments & Reputation-Based Payments

Structure of successful pin assignment to a node.

```js
struct PinAssignment {
    bytes32 cid;            // CID of pinned file
    address node;           // Address of assigned node
    address payable user;   // User who requested the pinning
    uint256 startTime;      // When pinning began
    uint256 endTime;        // When pinning should end
    uint256 totalPayment;   // Total ETH allocated for this assignment
    uint256 claimedAmount;  // ETH already claimed through vesting
    bool active;            // Whether assignment is currently active
}
```

Payment percentage is calculated based on node reputation. At max reputation (100) a node receives 99% of fees. At 0 reputation, a node gets 50% of the fees, with a linear scale between. Nodes with 0 reputation who receive another strike are banned from participation.

```js
function _calculatePaymentPercentage(uint256 reputation) internal pure returns (uint256) {
    uint256 basePercentage = 50;
    // Multiply reputation (0-100) by 49 (difference between min & max payment received) to get the variable portion
    uint256 variablePercentage = (reputation * 49) / 100;

    return basePercentage + variablePercentage;
}
```

Requests are processed from the queue, selecting next available nodes and processing their reputation-based payment into a linear vesting schedule.

```js
function _processRequest(uint256 requestId) internal {
    PinRequest storage request = pinRequests[requestId];
    require(!request.fulfilled, "Request already fulfilled");

    // Get the next available nodes
    address[] memory selectedNodes = _getNextAvailableNodes(request.replicationFactor);

    uint256 paymentPerNode = request.payment / request.replicationFactor;
    uint256 daoFee = 0;

    for (uint i = 0; i < selectedNodes.length; i++) {
        address node = selectedNodes[i];

        // Calculate node's payment based on reputation
        uint256 paymentPercentage = _calculatePaymentPercentage(nodes[node].reputation);
        uint256 nodePayment = (paymentPerNode * paymentPercentage) / 100;

        // Track DAO fee (remainder)
        daoFee += paymentPerNode - nodePayment;

        // Create assignment with vesting schedule
        uint256 assignmentId = _nextAssignmentId++;

        pinAssignments[assignmentId] = PinAssignment({
            cid: request.cid,
            node: node,
            user: request.user,
            startTime: block.timestamp,
            endTime: block.timestamp + request.duration,
            totalPayment: nodePayment,
            claimedAmount: 0,
            active: true
        });

        // Update node's active pins
        nodes[node].activePins[request.cid] = true;

        // Rotate node to back of queue
        _rotateNodeInQueue(node);

        emit PinAssigned(assignmentId, request.cid, node, nodePayment, request.duration);
    }

    // Send DAO fee to treasury
    if (daoFee > 0) {
        (bool success, ) = daoTreasury.call{value: daoFee}("");
        require(success, "DAO fee transfer failed");
        emit DAOFeeCollected(requestId, daoFee);
    }

    request.fulfilled = true;
}
```

### Direct Payments

In addition to the queue system, users can select and pay a node directly.

```js
struct DirectPayment {
    bytes32 cid;                // Content Identifier
    address payable node;       // Node providing service
    address payable user;       // User paying for service
    uint256 amount;             // Total ETH paid to node
    // DAO fee initialized at 1% but can be modified by governance proposals
    uint256 daoFee;             // Fee collected by DAO
    uint256 startTime;          // When agreement began
    uint256 endTime;            // When service should end
    uint256 claimedAmount;      // ETH already claimed
    bool active;                // Current status
}
```

Direct payments are negotiated between both parties and bypasses reputation, meaning that nodes paid directly receive the full fee (minus DAO percentage).

```js
function payNodeDirectly(
    address payable node,
    bytes32 cid,
    uint256 duration
) external payable {
    require(msg.value > 0, "Payment required");
    require(nodes[node].isActive, "Node not active");
    require(nodes[node].bannedUntil < block.timestamp, "Node is banned");

    // Calculate basis points for DAO fee (100 = 1%)
    uint256 daoFee = (msg.value * daoFeePercentage) / 10000;
    uint256 nodePayment = msg.value - daoFee;

    uint256 paymentId = _nextDirectPaymentId++;

    directPayments[paymentId] = DirectPayment({
        cid: cid,
        node: node,
        user: payable(msg.sender),
        amount: nodePayment,
        daoFee: daoFee,
        startTime: block.timestamp,
        endTime: block.timestamp + duration,
        claimedAmount: 0,
        active: true
    });

    // Update node's active pins
    nodes[node].activePins[cid] = true;

    // Send DAO fee
    (bool success, ) = daoTreasury.call{value: daoFee}("");
    require(success, "DAO fee transfer failed");

    emit DirectPaymentCreated(paymentId, cid, node, msg.sender, nodePayment, daoFee, duration);
}
```

### Claiming Vested Payments

At any time during an active assignment, a node can claim the ETH currently being vested to them.

```js
function claimVestedPayment(uint256 assignmentId) external {
    PinAssignment storage assignment = pinAssignments[assignmentId];

    require(msg.sender == assignment.node, "Not authorized");
    require(assignment.active, "Assignment not active");

    // Calculate vested amount
    uint256 totalDuration = assignment.endTime - assignment.startTime;
    uint256 elapsed = block.timestamp - assignment.startTime;

    uint256 vestedAmount;
    if (block.timestamp >= assignment.endTime) {
        vestedAmount = assignment.totalPayment;
    } else {
        vestedAmount = (assignment.totalPayment * elapsed) / totalDuration;
    }

    uint256 claimableAmount = vestedAmount - assignment.claimedAmount;
    require(claimableAmount > 0, "No funds to claim");

    // Update claimed amount
    assignment.claimedAmount += claimableAmount;

    // Transfer ETH to node
    (bool success, ) = assignment.node.call{value: claimableAmount}("");
    require(success, "Transfer failed");

    // Update node's total earned
    nodes[assignment.node].totalEarned += claimableAmount;

    emit PaymentClaimed(assignmentId, assignment.node, claimableAmount);
}
```

### Verification & Slashing

If a node is found to have prematurely unpinned an item, or is offline long enough for IPFS garbage collection to remove the pin, **verifiers** can report them to the protocol.

> ⚠️ **NOTE:** The verification procedure needs a lot of work to ensure that it's reliable and doesn't maliciously target nodes who are behaving.

```js
function reportUnpinnedContent(bytes32 cid, address node, uint256 assignmentId) external {
    PinAssignment storage assignment = pinAssignments[assignmentId];

    require(assignment.cid == cid, "CID mismatch");
    require(assignment.node == node, "Node mismatch");
    require(assignment.active, "Assignment not active");

    // Verification logic to confirm content is indeed not pinned
    bool contentUnpinned = _verifyContentIsUnpinned(cid, node);
    require(contentUnpinned, "Content is still pinned");

    // Calculate remaining payment
    uint256 remainingFunds = assignment.totalPayment - assignment.claimedAmount;

    if (remainingFunds > 0) {
        // Return 95% to user
        uint256 userRefund = (remainingFunds * 95) / 100;
        (bool successUser, ) = assignment.user.call{value: userRefund}("");
        require(successUser, "User refund failed");

        // Award 5% to reporter
        uint256 reporterReward = remainingFunds - userRefund;
        (bool successReporter, ) = payable(msg.sender).call{value: reporterReward}("");
        require(successReporter, "Reporter reward failed");

        emit SlashingExecuted(assignmentId, node, userRefund, msg.sender, reporterReward);
    }

    // Mark assignment as inactive
    assignment.active = false;

    // Remove from node's active pins
    nodes[node].activePins[cid] = false;

    // Decrease node reputation
    _decreaseReputation(node, SLASHING_REPUTATION_PENALTY);

    // Check if ban is warranted
    _evaluateBanning(node);
}
```

Ideally, **verifiers** would interact with reputation-altering functions through an oracle or other interface that can atomically check IPFS pin status against contract storage. For the actual implementation, this would check IPFS network.

```js
function _verifyContentIsUnpinned(bytes32 cid, address node) internal returns (bool) {
    //
    // Returns true if content is confirmed to be unpinned
    //
    return true;
}
```

## Challenges & Considerations 🏗️

As with any novel application there are a number of challenges and additional considerations that will need to be worked on as PinDAO is developed. Some of these will have to be addressed before a mainnet launch, while others can be addressed later through community proposals.

You can find a non-exhaustive list of development challenges and considerations [here](CHALLENGES.md).

## Community Collaboration 🤝

The goal is for PinDAO to be run by stakeholders who would utilize the service as content providers, nodes, or verifiers. As such, we welcome contributions from:

- 🔬 Researchers exploring storage incentivization models
- 💻 Developers building on the protocol or creating verification tools
- 🌐 IPFS node operators interested in monetizing their infrastructure
- 🧪 Users with high-value storage needs seeking reliable solutions

## Conclusion

This document outlines the vision for PinDAO, an ETH-based decentralized storage incentivization layer for IPFS. The protocol leverages Ethereum's security, liquidity, and composability to create reliable distributed storage with fair access for all participating nodes.

By providing direct economic incentives for IPFS pinning through a balanced queue system, PinDAO helps bridge the gap between decentralized storage technology and practical applications that require persistence guarantees.

---

### Contact

If you have any questions, open an issue or discussio thread, or reach out on [Warpcast](https://warpcast.com/jonbray.eth).
