# Gas Optimizations

## Summary

|       | Issue        | Instances    |
| :---: |:-------------|:------------:|
| 1  | State variables should be packed to save storage slots | 3 |
| 2  | Variables inside struct should be packed to save gas | 1 |
| 3  | Using `storage` instead of `memory` for struct/array saves gas | 1 |
| 4  | Emit events outside the for loops | 3 |
| 5  | Use `calldata` instead of `memory` for function parameters type | 6 |
| 6  | Place input check statements at the start of the functions | 3 |
| 7  | `memory` values should be emitted in events instead of `storage` ones | 4 |
| 8  | Initializing state variable to default values cost more gas | 1 |
| 9  | Duplicated input check statements should be refactored to a modifier  |  5 |


## Findings

### 1- State variables should be packed to save storage slots :

State variables inside contract should be packed if possible to save storage slots and thus will save gas cost.

There are 3 instances of this issue:

* Instance 1 :

```
File: newdao/token/NounsTokenFork.sol

49      address public minter;
70      bool public isMinterLocked;
73      bool public isDescriptorLocked;
76      bool public isSeederLocked;
```

The boolean variables `isMinterLocked`, `isDescriptorLocked` and `isSeederLocked` only takes one bit of stotage so their values can be packed with the address variable `minter` for example (which only takes 20 bytes), this will **save 3 storage slots** and thus saves a lot of gas, the code should be modified as follow :

```
/// @notice  An address who has permissions to mint Nouns
address public minter;

/// @notice Whether the minter can be updated
bool public isMinterLocked;

/// @notice Whether the descriptor can be updated
bool public isDescriptorLocked;

/// @notice Whether the seeder can be updated
bool public isSeederLocked;

```

* Instance 2 :

```
File: newdao/NounsAuctionHouseFork.sol

54      address public weth;
63      uint8 public minBidIncrementPercentage;
```

The variables `minBidIncrementPercentage` is only taking 1 byte of storage, so it can be packed with the address variable `weth` for example (which only takes 20 bytes), this will **save 1 storage slots**, the code should be modified as follow :

```
// The address of the WETH contract
address public weth;

// The minimum percentage difference between the last bid amount and the current bid
uint8 public minBidIncrementPercentage;
```

Additionnaly, as both variables `timeBuffer` and `duration` represents period of time (usuallly set in seconds) they can't realistically overflow uint32 (2^32), thus the state variables can be further packed to **save 2 additional storage slots** as follow : 

```
// The address of the WETH contract
address public weth;

// The minimum amount of time left in an auction after a new bid is created
uint32 public timeBuffer;

// The duration of a single auction
uint32 public duration;

// The minimum percentage difference between the last bid amount and the current bid
uint8 public minBidIncrementPercentage;
```

* Instance 3 :

```
File: governance/NounsDAOExecutorV2.sol

90      address public pendingAdmin;
91      uint256 public delay;
```

Because the NounsDAOExecutorV2 is being deployed from fresh (not just upgraded), you can consider using uint64 for the variable `delay`, as this variable has a maximum value `MAXIMUM_DELAY` and a minmum value `MINIMUM_DELAY` it can't overflow 2^64. By using the type `uint64`, the variable can be packed with the address variable `pendingAdmin` for example (which only takes 20 bytes), this will **save 1 storage slots**, the code should be modified as follow :

```
address public pendingAdmin;
uint64 public delay;
```


### 2- Variables inside `struct` should be packed to save gas (saves 4 storage slots) :

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There is 1 instance of this issue:

**File: NounsDAOInterfaces.sol** [Line 653-717](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L653-L717)

```solidity
struct StorageV3 {
    // ================ PROXY ================ //
    /// @notice Administrator for this contract
    address admin;
    /// @notice Pending administrator for this contract
    address pendingAdmin;
    /// @notice Active brains of Governor
    address implementation;
    // ================ V1 ================ //
    /// @notice Vetoer who has the ability to veto any proposal
    address vetoer;
    /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
    uint256 votingDelay;
    /// @notice The duration of voting on a proposal, in blocks
    uint256 votingPeriod;
    /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
    uint256 proposalThresholdBPS;
    /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
    uint256 quorumVotesBPS;
    /// @notice The total number of proposals
    uint256 proposalCount;
    /// @notice The address of the Nouns DAO Executor NounsDAOExecutor
    INounsDAOExecutorV2 timelock;
    /// @notice The address of the Nouns tokens
    NounsTokenLike nouns;
    /// @notice The official record of all proposals ever proposed
    mapping(uint256 => Proposal) _proposals;
    /// @notice The latest proposal for each proposer
    mapping(address => uint256) latestProposalIds;
    // ================ V2 ================ //
    DynamicQuorumParamsCheckpoint[] quorumParamsCheckpoints;
    /// @notice Pending new vetoer
    address pendingVetoer;
    // ================ V3 ================ //
    /// @notice user => sig => isCancelled: signatures that have been cancelled by the signer and are no longer valid
    mapping(address => mapping(bytes32 => bool)) cancelledSigs;
    /// @notice The number of blocks before voting ends during which the objection period can be initiated
    uint32 lastMinuteWindowInBlocks;
    /// @notice Length of the objection period in blocks
    uint32 objectionPeriodDurationInBlocks;
    /// @notice Length of proposal updatable period in block
    uint32 proposalUpdatablePeriodInBlocks;
    /// @notice address of the DAO's fork escrow contract
    INounsDAOForkEscrow forkEscrow;
    /// @notice address of the DAO's fork deployer contract
    IForkDAODeployer forkDAODeployer;
    /// @notice ERC20 tokens to include when sending funds to a deployed fork
    address[] erc20TokensToIncludeInFork;
    /// @notice The treasury contract of the last deployed fork
    address forkDAOTreasury;
    /// @notice The token contract of the last deployed fork
    address forkDAOToken;
    /// @notice Timestamp at which the last fork period ends
    uint256 forkEndTimestamp;
    /// @notice Fork period in seconds
    uint256 forkPeriod;
    /// @notice Threshold defined in basis points (10,000 = 100%) required for forking
    uint256 forkThresholdBPS;
    /// @notice Address of the original timelock
    INounsDAOExecutor timelockV1;
    /// @notice The proposal at which to start using `startBlock` instead of `creationBlock` for vote snapshots
    /// @dev Make sure this stays the last variable in this struct, so we can delete it in the next version
    /// @dev To be zeroed-out and removed in a V3.1 fix version once the switch takes place
    uint256 voteSnapshotBlockSwitchProposalId;
}
```

* As the variables `forkEndTimestamp`, `forkPeriod` represents the number of passed blocks they can't realistically overflow 2^32. And the variables `forkThresholdBPS`represents a threshold defined in basis points hence its max value is 10000, so all those variables should be packed together with the address variable `forkDAOToken` for example (which only takes 20 bytes) inside the struct this will **save 3 storage slots** and thus saves a lot of gas.

* There is also the variables `lastMinuteWindowInBlocks`, `objectionPeriodDurationInBlocks` and `proposalUpdatablePeriodInBlocks` which are of type `uint32` and thus can be packed with the address variable `forkDAOToken` (for example) inside the struct this will **save 1 storage slots**.

```solidity
struct StorageV3 {
    // ================ PROXY ================ //
    /// @notice Administrator for this contract
    address admin;
    /// @notice Pending administrator for this contract
    address pendingAdmin;
    /// @notice Active brains of Governor
    address implementation;
    // ================ V1 ================ //
    /// @notice Vetoer who has the ability to veto any proposal
    address vetoer;
    /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
    uint256 votingDelay;
    /// @notice The duration of voting on a proposal, in blocks
    uint256 votingPeriod;
    /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
    uint256 proposalThresholdBPS;
    /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
    uint256 quorumVotesBPS;
    /// @notice The total number of proposals
    uint256 proposalCount;
    /// @notice The address of the Nouns DAO Executor NounsDAOExecutor
    INounsDAOExecutorV2 timelock;
    /// @notice The address of the Nouns tokens
    NounsTokenLike nouns;
    /// @notice The official record of all proposals ever proposed
    mapping(uint256 => Proposal) _proposals;
    /// @notice The latest proposal for each proposer
    mapping(address => uint256) latestProposalIds;
    // ================ V2 ================ //
    DynamicQuorumParamsCheckpoint[] quorumParamsCheckpoints;
    /// @notice Pending new vetoer
    address pendingVetoer;
    // ================ V3 ================ //
    /// @notice user => sig => isCancelled: signatures that have been cancelled by the signer and are no longer valid
    mapping(address => mapping(bytes32 => bool)) cancelledSigs;
    /// @notice address of the DAO's fork escrow contract
    INounsDAOForkEscrow forkEscrow;
    /// @notice address of the DAO's fork deployer contract
    IForkDAODeployer forkDAODeployer;
    /// @notice ERC20 tokens to include when sending funds to a deployed fork
    address[] erc20TokensToIncludeInFork;
    /// @notice The treasury contract of the last deployed fork
    address forkDAOTreasury;
    /// @notice The number of blocks before voting ends during which the objection period can be initiated
    uint32 lastMinuteWindowInBlocks;
    /// @notice Length of the objection period in blocks
    uint32 objectionPeriodDurationInBlocks;
    /// @notice Length of proposal updatable period in block
    uint32 proposalUpdatablePeriodInBlocks;
    /// @notice The token contract of the last deployed fork
    address forkDAOToken;
    /// @notice Timestamp at which the last fork period ends
    uint32 forkEndTimestamp;
    /// @notice Fork period in seconds
    uint32 forkPeriod;
    /// @notice Threshold defined in basis points (10,000 = 100%) required for forking
    uint32 forkThresholdBPS;
    /// @notice Address of the original timelock
    INounsDAOExecutor timelockV1;
    /// @notice The proposal at which to start using `startBlock` instead of `creationBlock` for vote snapshots
    /// @dev Make sure this stays the last variable in this struct, so we can delete it in the next version
    /// @dev To be zeroed-out and removed in a V3.1 fix version once the switch takes place
    uint256 voteSnapshotBlockSwitchProposalId;
}
```


### 3- Using `storage` instead of `memory` for struct/array saves gas :

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. 

The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

There is 1 instance of this issue :

**File: NounsDAOLogicV1Fork.sol** [Line 208](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L208)
```
address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
```


### 4- Emit events outside the for loops :

As the NounsDAOExecutorV2 contract is being deployed from fresh (not just upgraded), you should consider changing the way the functions `queueTransaction`, `cancelTransaction` and `executeTransaction` are structured.

if we take for example the working of the function `queueTransaction` :

* It is called when the `queue` function is called from the NounsDAOLogicV3 contract (or the DAO fork contracts like NounsDAOLogicV1Fork).

* The `queue` function loops over all the actions in a proposal and call `queueTransaction` for each one of them.

* The `queueTransaction` function implement the queue logic and then emit an event `QueueTransaction` for each queued action.

This stucture cost a lot of gas as the event is emitted at each iteration of the loop and this effect is even more amplified since the protocol will normally have many proposals with many actions to manage. 

The same issue is present in the `cancelTransaction` and `executeTransaction` functions.

Thus before deploying the new NounsDAOExecutorV2 contract and upgrading the DAO logic to NounsDAOLogicV3 (or creating a new DAO forks), you should consider adding a `queueTransactions` function which takes a list of all actions for a given proposal to excute and process them and then at the end emit a final global event `QueueTransactions` for all the actions in a proposal.

This change will require to update the `queue` function in the NounsDAOLogicV3 contract and make it send all the actions in a proposal instead of looping over them.

**I will give an example of how this can be implemented for the `queueTransaction`/`queue` functions but the same can be done for both `cancelTransaction`/`cancel` and `executeTransaction`/`execute` functions.**

The new `queueTransactions` function to replace the old one [NounsDAOExecutorV2.queueTransaction](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L130-L148) :

```solidity
function queueTransactions(
    address[] memory targets,
    uint256[] memory values,
    string[] memory signatures,
    bytes[] memory calldatas
    uint256 eta
) public returns (bytes32[] memory) {
    require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');
    require(
        eta >= getBlockTimestamp() + delay,
        'NounsDAOExecutor::queueTransaction: Estimated execution block must satisfy delay.'
    );

    uint256 length = proposal.targets.length;
    bytes32[] memory hashes = new bytes32[](length);
    for (uint256 i = 0; i < length; i++) {
        bytes32 txHash = keccak256(
            abi.encode(proposal.targets[i], proposal.values[i], proposal.signatures[i], proposal.calldatas[i], eta)
        );
        require(
            !queuedTransactions[txHash],
            'NounsDAO::queueTransactions: identical proposal action already queued at eta'
        );
        queuedTransactions[txHash] = true;
        hashes[i] = txHash;
    }

    emit QueueTransactions(hashes, targets, values, signatures, calldatas, eta);
    return hashes;
}
```

The `queue` and `queueOrRevertInternal` functions from NounsDAOLogicV3 will become : 

```solidity
function queue(NounsDAOStorageV3.StorageV3 storage ds, uint256 proposalId) external {
    require(
        stateInternal(ds, proposalId) == NounsDAOStorageV3.ProposalState.Succeeded,
        'NounsDAO::queue: proposal can only be queued if it is succeeded'
    );
    NounsDAOStorageV3.Proposal storage proposal = ds._proposals[proposalId];
    INounsDAOExecutor timelock = getProposalTimelock(ds, proposal);
    uint256 eta = block.timestamp + timelock.delay();
    queueOrRevertInternal(
        timelock,
        proposal.targets,
        proposal.values,
        proposal.signatures,
        proposal.calldatas,
        eta
    );
    proposal.eta = eta;
    emit ProposalQueued(proposalId, eta);
}
```

```solidity
function queueOrRevertInternal(
    INounsDAOExecutor timelock,
    address[] memory targets,
    uint256[] memory values,
    string[] memory signatures,
    bytes[] memory calldatas
    uint256 eta
) internal {
    timelock.queueTransactions(targets, values, signatures, calldatas, eta);
}
```

**Note : as all the new DAO forks will be managed by a timelock similair to NounsDAOExecutorV2, this feature can also be applied to them.**

### 5- Use `calldata` instead of `memory` for function parameters type :

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

There are 6 instances of this issue:

```
File: governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

206     function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude)
272     function propose(
            address[] memory targets,
            uint256[] memory values,
            string[] memory signatures,
            bytes[] memory calldatas,
            string memory description
        )
File: governance/NounsDAOV3Proposals.sol

220     function proposeBySigs(
            NounsDAOStorageV3.StorageV3 storage ds,
            NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures, 
            ProposalTxs memory txs,
            string memory description
        ) 
288     function updateProposal(
            NounsDAOStorageV3.StorageV3 storage ds,
            uint256 proposalId,
            address[] memory targets,
            uint256[] memory values,
            string[] memory signatures,
            bytes[] memory calldatas,
            string memory description,
            string memory updateMessage
        )
321     function updateProposalTransactions(
            NounsDAOStorageV3.StorageV3 storage ds,
            uint256 proposalId,
            address[] memory targets, 
            uint256[] memory values,
            string[] memory signatures,
            bytes[] memory calldatas,
            string memory updateMessage
        )
383     function updateProposalBySigs(
            NounsDAOStorageV3.StorageV3 storage ds,
            uint256 proposalId,
            NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures, 
            ProposalTxs memory txs,
            string memory description,
            string memory updateMessage
        )
```

### 6- Place input check statements at the start of the functions :

The if or require check statement used for verifying input parameters should be placed at the start of the functions, to save gas in the case the transaction reverts.

There are 3 instances of this issue :

File: NounsDAOV3Proposals.sol [Line 849](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L849)
```
checkNoActiveProp(ds, msg.sender);
```

The check above should be placed at the start of the `verifySignersCanBackThisProposalAndCountTheirVotes` function as it's used to verify the proposer's old proposal status.


File: NounsDAOV3Proposals.sol [Line 989](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L989)
```
if (block.timestamp > proposerSignature.expirationTimestamp) revert SignatureExpired();
```

The check above should be placed at the start of the `verifyProposalSignature` function as it's performing a verification on the input parameter `proposerSignature`.


File: NounsDAOLogicV1Fork.sol [Line 291-298](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L291-L298)
```
require(
    targets.length == values.length &&
        targets.length == signatures.length &&
        targets.length == calldatas.length,
    'NounsDAO::propose: proposal function information arity mismatch'
);
require(targets.length != 0, 'NounsDAO::propose: must provide actions');
require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

All the checks in the code above should be placed at the start of the `propose` function as they performing a verification on the input arrays.



### 7- `memory` values should be emitted in events instead of `storage` ones :

The values emitted in events shouldn’t be read from storage but the existing memory values should be used instead, this will save around **~101 GAS**.

There are 4 instances of this issue :

**File: NounsDAOLogicV1Fork.sol** [Line 340-350](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L340-L350)
```solidity
emit ProposalCreated(
    newProposal.id,
    msg.sender,
    targets,
    values,
    signatures,
    calldatas,
    newProposal.startBlock,
    newProposal.endBlock,
    description
);
```

The memory values `temp.startBlock` & `temp.endBlock` should be emitted as follow :

```solidity
emit ProposalCreated(
    newProposal.id,
    msg.sender,
    targets,
    values,
    signatures,
    calldatas,
    temp.startBlock,
    temp.endBlock,
    description
);
```

**File: NounsDAOLogicV1Fork.sol** [Line 353-365](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L353-L365)
```solidity
emit ProposalCreated(
    newProposal.id,
    msg.sender,
    targets,
    values,
    signatures,
    calldatas,
    newProposal.startBlock,
    newProposal.endBlock,
    description
);
```

The memory values `temp.startBlock`, `temp.endBlock`, `temp.proposalThreshold` and `temp.quorumVotes` should be emitted as follow :

```solidity
emit ProposalCreatedWithRequirements(
    newProposal.id,
    msg.sender,
    targets,
    values,
    signatures,
    calldatas,
    temp.startBlock,
    temp.endBlock,
    temp.proposalThreshold,
    temp.quorumVotes,
    description
);
```

**File: NounsDAOProxy.sol** [Line 85](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L85)
```
emit NewImplementation(oldImplementation, implementation);
```

The memory value `implementation_` should be emitted as follow :

```
emit NewImplementation(oldImplementation, implementation_);
```

**File: NounsDAOV3Admin.sol** [Line 293](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L293)
```
emit NewAdmin(oldAdmin, ds.admin);
```

The memory value `oldPendingAdmin` should be emitted as follow :

```
emit NewAdmin(oldAdmin, oldPendingAdmin);
```


### 8- Initializing state variable to default values cost more gas :

In solidity all state variables are set at the default values when the contract is deployed, so there is no need to initialize them to their default values as this will only cost more due to writing into storage. 

This issue is occuring in the `propose` function of the `NounsDAOLogicV1Fork` contract, as this function will be used frequently by the DAO members, it will cost them more funds to manage the protocol :

**File: NounsDAOLogicV1Fork.sol** [Line 272-368](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L272-L368)

```solidity
function propose(
    address[] memory targets,
    uint256[] memory values,
    string[] memory signatures,
    bytes[] memory calldatas,
    string memory description
) public returns (uint256) {
    // ...

    proposalCount++;
    Proposal storage newProposal = _proposals[proposalCount];

    newProposal.id = proposalCount;
    newProposal.proposer = msg.sender;
    newProposal.proposalThreshold = temp.proposalThreshold;
    newProposal.quorumVotes = bps2Uint(quorumVotesBPS, temp.totalSupply);
    newProposal.eta = 0; // @audit shouldn't be initialized
    newProposal.targets = targets;
    newProposal.values = values;
    newProposal.signatures = signatures;
    newProposal.calldatas = calldatas;
    newProposal.startBlock = temp.startBlock;
    newProposal.endBlock = temp.endBlock;
    newProposal.forVotes = 0; // @audit shouldn't be initialized
    newProposal.againstVotes = 0; // @audit shouldn't be initialized
    newProposal.abstainVotes = 0; // @audit shouldn't be initialized
    newProposal.canceled = false; // @audit shouldn't be initialized
    newProposal.executed = false; // @audit shouldn't be initialized
    newProposal.creationBlock = block.number;

    latestProposalIds[newProposal.proposer] = newProposal.id;

    // ...
}
```


### 9- Duplicated input check statements should be refactored to a modifier :

Input check statements used multiple times inside a contract should be refactored to a modifier for better readability and also to save gas.

There are 5 instances of this issue :

```
File: governance/NounsDAOExecutorV2.sol

137     require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');
157     require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');
172     require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');
214     require(msg.sender == admin, 'NounsDAOExecutor::sendETH: Call must come from admin.');
226     require(msg.sender == admin, 'NounsDAOExecutor::sendERC20: Call must come from admin.');
```

Those checks should be replaced by an `onlyAdmin` modifier as follow :

```solidity
modifier onlyAdmin(){
    require(msg.sender == admin, "NounsDAOExecutor: admin only");
    _;
}
```