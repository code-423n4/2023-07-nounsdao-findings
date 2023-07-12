- Emitting storage values instead of the memory one
    - `NounsDAOExecutor.sol.setDelay()` : use `delay_` 
    - `NounsDAOLogicV1.sol._setVotingDelay()` : use `newVotingDelay`
- State variables only set in the constructor should be declared `immutable`
- Structs can be packed into fewer storage slots
- `NounsTokenFork.sol.claimDuringForkPeriod()` : use cache value on for loop for tokenIds[i]
- Using `storage` instead of `memory` saves gas
  

```solidity
571: function cancel(NounsDAOStorageV3.StorageV3 storage ds, uint256 proposalId) external {
        NounsDAOStorageV3.ProposalState proposalState = stateInternal(ds, proposalId);
        if (
            proposalState == NounsDAOStorageV3.ProposalState.Canceled ||
            proposalState == NounsDAOStorageV3.ProposalState.Defeated ||
            proposalState == NounsDAOStorageV3.ProposalState.Expired ||
            proposalState == NounsDAOStorageV3.ProposalState.Executed ||
            proposalState == NounsDAOStorageV3.ProposalState.Vetoed
        ) {
            revert CantCancelProposalAtFinalState();
        }
....
     
587:        uint256 votes = nouns.getPriorVotes(proposer, block.number - 1);
588:        bool msgSenderIsProposer = proposer == msg.sender;
589:        address[] memory signers = proposal.signers;
590:        for (uint256 i = 0; i < signers.length; ++i) {
591:            msgSenderIsProposer = msgSenderIsProposer || msg.sender == signers[i];
592:            votes += nouns.getPriorVotes(signers[i], block.number - 1);
593:        }

....

612:        emit ProposalCanceled(proposalId);
613:    }
```

### **Emitting storage values instead of the memory one**

Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

- list
    
    `NounsDAOExecutor.sol.setDelay()` : use `delay_`
    
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L86
    
    ```solidity
    File: /contracts/governance/NounsDAOExecutor.sol
    80: function setDelay(uint256 delay_) public {
            require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
            require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
            require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
            delay = delay_;
    
    86:        emit NewDelay(delay);
    87:    }
    ```
    
    ```diff
    function setDelay(uint256 delay_) public {
            require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
            require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
            require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
            delay = delay_;
    
    -        emit NewDelay(delay);
    +        emit NewDelay(delay_);
        }
    ```
    
    `NounsDAOExecutor.sol.acceptAdmin()` : use `msg.sender`
    
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L89C4-L95C6
    
    ```solidity
    89: function acceptAdmin() public {
            require(msg.sender == pendingAdmin, 'NounsDAOExecutor::acceptAdmin: Call must come from pendingAdmin.');
            admin = msg.sender;
            pendingAdmin = address(0);
    
    94:        emit NewAdmin(admin);
    95:    }
    ```
    
    ```diff
    - 94:        emit NewAdmin(admin);
    + 94:        emit NewAdmin(msg.sender);
    ```
    
    `NounsDAOExecutor.sol.setPendingAdmin()` : use `pendingAdmin_`
    
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L104
    
    ```solidity
    File: /contracts/governance/NounsDAOExecutor.sol
    	97: function setPendingAdmin(address pendingAdmin_) public {
    	98:        require(
    	99:            msg.sender == address(this),
    	100:            'NounsDAOExecutor::setPendingAdmin: Call must come from NounsDAOExecutor.'
    	101:        );
    	102:        pendingAdmin = pendingAdmin_;
    	103:
    	104:        emit NewPendingAdmin(pendingAdmin);
    	105:    }
    ```
    
    ```diff
    	97: function setPendingAdmin(address pendingAdmin_) public {
    	98:        require(
    	99:            msg.sender == address(this),
    	100:            'NounsDAOExecutor::setPendingAdmin: Call must come from NounsDAOExecutor.'
    	101:        );
    	102:        pendingAdmin = pendingAdmin_;
    	103:
    -	104:        emit NewPendingAdmin(pendingAdmin);
    +	104:        emit NewPendingAdmin(pendingAdmin_);
    	105:    }
    ```
    
    `NounsDAOProxy.sol._setIMplementation()` : use `implementation_`
    
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L85
    
    ```solidity
    File: /contracts/governance/NounsDAOProxy.sol
    	85: function _setImplementation(address implementation_) public {
    	86:        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
    	87:        require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');
    	88:
    	89:        address oldImplementation = implementation;
    	90:        implementation = implementation_;
    	91:
    	92:        emit NewImplementation(oldImplementation, implementation);
    	93:    }
    ```
    
    ```diff
    - emit NewImplementation(oldImplementation, implementation);
    + emit NewImplementation(oldImplementation, implementation_);
            
    ```
    
    `NounsDAOLogicV1.sol._setVotingDelay()` : use `newVotingDelay`
    
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol#L538
    
    ```diff
    File: contracts/governance/NounsDAOLogicV1.sol
    529: function _setVotingDelay(uint256 newVotingDelay) external {
            require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
            require(
                newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
                'NounsDAO::_setVotingDelay: invalid voting delay'
            );
            uint256 oldVotingDelay = votingDelay;
            votingDelay = newVotingDelay;
    
    - 538:        emit VotingDelaySet(oldVotingDelay, votingDelay);
    + 538:        emit VotingDelaySet(oldVotingDelay, newVotingDelay);
        }
    
    ....
    545: function _setVotingPeriod(uint256 newVotingPeriod) external {
            require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
            require(
                newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
                'NounsDAO::_setVotingPeriod: invalid voting period'
            );
            uint256 oldVotingPeriod = votingPeriod;
            votingPeriod = newVotingPeriod;
    
    - 554:        emit VotingPeriodSet(oldVotingPeriod, votingPeriod);
    + 554:        emit VotingPeriodSet(oldVotingPeriod, newVotingPeriod);
        }
    ....
    562: function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
            require(msg.sender == admin, 'NounsDAO::_setProposalThresholdBPS: admin only');
            require(
                newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
                    newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
                'NounsDAO::_setProposalThreshold: invalid proposal threshold'
            );
            uint256 oldProposalThresholdBPS = proposalThresholdBPS;
            proposalThresholdBPS = newProposalThresholdBPS;
    
    - 572:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS);
    + 572:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, newProposalThresholdBPS);
        }
     ....
    580: function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external {
            require(msg.sender == admin, 'NounsDAO::_setQuorumVotesBPS: admin only');
            require(
                newQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS && newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS,
                'NounsDAO::_setProposalThreshold: invalid proposal threshold'
            );
            uint256 oldQuorumVotesBPS = quorumVotesBPS;
            quorumVotesBPS = newQuorumVotesBPS;
    
    - 589:        emit QuorumVotesBPSSet(oldQuorumVotesBPS, quorumVotesBPS);
    + 589:        emit QuorumVotesBPSSet(oldQuorumVotesBPS, newQuorumVotesBPS);
        }
    ....
    615: function _acceptAdmin() external {
            // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
            require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');
    
            // Save current values for inclusion in log
            address oldAdmin = admin;
            address oldPendingAdmin = pendingAdmin;
    
            // Store admin with value pendingAdmin
            admin = pendingAdmin;
    
            // Clear the pending value
            pendingAdmin = address(0);
    
            emit NewAdmin(oldAdmin, admin);
    -630:        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
    +630:        emit NewPendingAdmin(oldPendingAdmin, address(0));
        }
    ```
    

**State variables only set in the constructor should be declared `immutable`**

**Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a `PUSH32` (3 gas)**

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L76C1-L77C24

```solidity
File: contracts/governance/NounsDAOExecutor.sol
76: admin = admin_;
77: delay = delay_;
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L99C1-L100C24

```solidity
99:   admin = admin_;
100:  delay = delay_;
```

**Structs can be packed into fewer storage slots**

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs) we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.

Total Instances: 12

Estimated Gas Saved:  `12 (slots) * 2000 = 24000`

`Note` : None of these findings were found by bot findings - Gas

- All instances
    
    ```solidity
    File: /contracts/governance/NounsDAOInterfaces.sol
    311: struct Proposal {
            /// @notice Unique id for looking up a proposal
            uint256 id;
            /// @notice Creator of the proposal
            address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
            /// @notice the ordered list of target addresses for calls to be made
            address[] targets;
            /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
            uint256[] values;
            /// @notice The ordered list of function signatures to be called
            string[] signatures;
            /// @notice The ordered list of calldata to be passed to each call
            bytes[] calldatas;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice Receipts of ballots for the entire set of voters
            mapping(address => Receipt) receipts;
        }
    ```
    
    ```diff
    311: struct Proposal {
            /// @notice Unique id for looking up a proposal
            uint256 id;
    -        /// @notice Creator of the proposal
    -        address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
    -        /// @notice the ordered list of target addresses for calls to be made
    -       address[] targets;
            /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
            uint256[] values;
            /// @notice The ordered list of function signatures to be called
            string[] signatures;
    +       /// @notice the ordered list of target addresses for calls to be made
    +       address[] targets;
            /// @notice The ordered list of calldata to be passed to each call
            bytes[] calldatas;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
    +        /// @notice Creator of the proposal
    +        address proposer;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice Receipts of ballots for the entire set of voters
            mapping(address => Receipt) receipts;
        }
    ```
    
    ```solidity
    File: /contracts/governance/NounsDAOInterfaces.sol
    411: struct Proposal {
            /// @notice Unique id for looking up a proposal
            uint256 id;
            /// @notice Creator of the proposal
            address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
            /// @notice the ordered list of target addresses for calls to be made
            address[] targets;
            /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
            uint256[] values;
            /// @notice The ordered list of function signatures to be called
            string[] signatures;
            /// @notice The ordered list of calldata to be passed to each call
            bytes[] calldatas;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice Receipts of ballots for the entire set of voters
            mapping(address => Receipt) receipts;
            /// @notice The total supply at the time of proposal creation
            uint256 totalSupply;
            /// @notice The block at which this proposal was created
            uint256 creationBlock;
        }
    ```
    
    ```diff
    411: struct Proposal {
            /// @notice Unique id for looking up a proposal
            uint256 id;
    -        /// @notice Creator of the proposal
    -        address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
    -        /// @notice the ordered list of target addresses for calls to be made
    -        address[] targets;
            /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
            uint256[] values;
    +        /// @notice the ordered list of target addresses for calls to be made
    +        address[] targets;
            /// @notice The ordered list of function signatures to be called
            string[] signatures;
            /// @notice The ordered list of calldata to be passed to each call
            bytes[] calldatas;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
    +        /// @notice Creator of the proposal
    +        address proposer;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice Receipts of ballots for the entire set of voters
            mapping(address => Receipt) receipts;
            /// @notice The total supply at the time of proposal creation
            uint256 totalSupply;
            /// @notice The block at which this proposal was created
            uint256 creationBlock;
        }
    ```
    
    ```solidity
    File: /contracts/governance/NounsDAOInterfaces.sol
    508: struct Proposal {
            /// @notice Unique id for looking up a proposal
            uint256 id;
            /// @notice Creator of the proposal
            address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
            /// @notice the ordered list of target addresses for calls to be made
            address[] targets;
            /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
            uint256[] values;
            /// @notice The ordered list of function signatures to be called
            string[] signatures;
            /// @notice The ordered list of calldata to be passed to each call
            bytes[] calldatas;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice Receipts of ballots for the entire set of voters
            mapping(address => Receipt) receipts;
        }
    ```
    
    ```diff
    508: struct Proposal {
            /// @notice Unique id for looking up a proposal
            uint256 id;
    -        /// @notice Creator of the proposal
    -        address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
    -        /// @notice the ordered list of target addresses for calls to be made
    -        address[] targets;
            /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
            uint256[] values;
    +        /// @notice the ordered list of target addresses for calls to be made
    +        address[] targets;
            /// @notice The ordered list of function signatures to be called
            string[] signatures;
            /// @notice The ordered list of calldata to be passed to each call
            bytes[] calldatas;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
    +        /// @notice Creator of the proposal
    +        address proposer;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice Receipts of ballots for the entire set of voters
            mapping(address => Receipt) receipts;
        }
    ```
    
    ```solidity
    File: /contracts/governance/NounsDAOInterfaces.sol
    
    653: struct StorageV3 {
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
    
    ```diff
    653: struct StorageV3 {
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
    
    ```solidity
    File: /contracts/governance/NounsDAOInterfaces.sol
    719: struct Proposal {
            /// @notice Unique id for looking up a proposal
            uint256 id;
            /// @notice Creator of the proposal
            address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
            /// @notice the ordered list of target addresses for calls to be made
            address[] targets;
            /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
            uint256[] values;
            /// @notice The ordered list of function signatures to be called
            string[] signatures;
            /// @notice The ordered list of calldata to be passed to each call
            bytes[] calldatas;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice Receipts of ballots for the entire set of voters
            mapping(address => Receipt) receipts;
            /// @notice The total supply at the time of proposal creation
            uint256 totalSupply;
            /// @notice The block at which this proposal was created
            uint64 creationBlock;
            /// @notice The last block which allows updating a proposal's description and transactions
            uint64 updatePeriodEndBlock;
            /// @notice Starts at 0 and is set to the block at which the objection period ends when the objection period is initiated
            uint64 objectionPeriodEndBlock;
            /// @dev unused for now
            uint64 placeholder;
            /// @notice The signers of a proposal, when using proposeBySigs
            address[] signers;
            /// @notice When true, a proposal would be executed on timelockV1 instead of the current timelock
            bool executeOnTimelockV1;
        }
    ```
    
    ```diff
    719: struct Proposal {
            /// @notice Unique id for looking up a proposal
            uint256 id;
    -        /// @notice Creator of the proposal
    -        address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
    -        /// @notice the ordered list of target addresses for calls to be made
    -        address[] targets;
            /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
            uint256[] values;
    +        /// @notice the ordered list of target addresses for calls to be made
    +        address[] targets;
            /// @notice The ordered list of function signatures to be called
            string[] signatures;
            /// @notice The ordered list of calldata to be passed to each call
            bytes[] calldatas;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
    +        /// @notice Creator of the proposal
    +        address proposer;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice Receipts of ballots for the entire set of voters
            mapping(address => Receipt) receipts;
            /// @notice The total supply at the time of proposal creation
            uint256 totalSupply;
            /// @notice The block at which this proposal was created
            uint64 creationBlock;
            /// @notice The last block which allows updating a proposal's description and transactions
            uint64 updatePeriodEndBlock;
            /// @notice Starts at 0 and is set to the block at which the objection period ends when the objection period is initiated
            uint64 objectionPeriodEndBlock;
            /// @dev unused for now
            uint64 placeholder;
            /// @notice The signers of a proposal, when using proposeBySigs
            address[] signers;
            /// @notice When true, a proposal would be executed on timelockV1 instead of the current timelock
            bool executeOnTimelockV1;
        }
    ```
    
    ```solidity
    File: /contracts/governance/NounsDAOInterfaces.sol
    791: struct ProposalCondensed {
            /// @notice Unique id for looking up a proposal
            uint256 id;
            /// @notice Creator of the proposal
            address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The minimum number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
            /// @notice The total supply at the time of proposal creation
            uint256 totalSupply;
            /// @notice The block at which this proposal was created
            uint256 creationBlock;
            /// @notice The signers of a proposal, when using proposeBySigs
            address[] signers;
            /// @notice The last block which allows updating a proposal's description and transactions
            uint256 updatePeriodEndBlock;
            /// @notice Starts at 0 and is set to the block at which the objection period ends when the objection period is initiated
            uint256 objectionPeriodEndBlock;
            /// @notice When true, a proposal would be executed on timelockV1 instead of the current timelock
            bool executeOnTimelockV1;
        }
    ```
    
    ```diff
    791: struct ProposalCondensed {
            /// @notice Unique id for looking up a proposal
            uint256 id;
    -        /// @notice Creator of the proposal
    -        address proposer;
            /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 proposalThreshold;
            /// @notice The minimum number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
            uint256 quorumVotes;
            /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
            uint256 eta;
            /// @notice The block at which voting begins: holders must delegate their votes prior to this block
            uint256 startBlock;
            /// @notice The block at which voting ends: votes must be cast prior to this block
            uint256 endBlock;
            /// @notice Current number of votes in favor of this proposal
            uint256 forVotes;
            /// @notice Current number of votes in opposition to this proposal
            uint256 againstVotes;
            /// @notice Current number of votes for abstaining for this proposal
            uint256 abstainVotes;
    +        /// @notice Creator of the proposal
    +        address proposer;
            /// @notice Flag marking whether the proposal has been canceled
            bool canceled;
            /// @notice Flag marking whether the proposal has been vetoed
            bool vetoed;
            /// @notice Flag marking whether the proposal has been executed
            bool executed;
    +        /// @notice When true, a proposal would be executed on timelockV1 instead of the current timelock
    +        bool executeOnTimelockV1;
            /// @notice The total supply at the time of proposal creation
            uint256 totalSupply;
            /// @notice The block at which this proposal was created
            uint256 creationBlock;
            /// @notice The signers of a proposal, when using proposeBySigs
            address[] signers;
            /// @notice The last block which allows updating a proposal's description and transactions
            uint256 updatePeriodEndBlock;
            /// @notice Starts at 0 and is set to the block at which the objection period ends when the objection period is initiated
            uint256 objectionPeriodEndBlock;
    -        /// @notice When true, a proposal would be executed on timelockV1 instead of the current timelock
    -        bool executeOnTimelockV1;
        }
    ```
    

`**NounsTokenFork.sol.claimDuringForkPeriod()` : use cache value on for loop for tokenIds[i]**

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L176

```solidity
File: /contracts/governance/fork/newdao/token/NounsTokenFork.sol
	166: function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
	167:        uint256 currentNounId = _currentNounId;
	168:	        uint256 maxNounId = 0;
	169:	        if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
	170:	        if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
	171:	
	172:	        for (uint256 i = 0; i < tokenIds.length; i++) {
	173:	            uint256 nounId = tokenIds[i];
	174:	            _mintWithOriginalSeed(to, nounId);
	175:	
	176:	            if (tokenIds[i] > maxNounId) maxNounId = tokenIds[i];
	177:	        }
		
		        // This treats an important case:
		        // During a forking period, people can buy new Nouns on auction, with a higher ID than the auction ID at forking
		        // They can then join the fork with those IDs
		        // If we don't increment currentNounId, unpausing the fork auction house would revert
		        // Since it would attempt to mint a noun with an ID that already exists
	184:	        if (maxNounId >= currentNounId) _currentNounId = maxNounId + 1;
	185:	 }
```

```diff
	166: function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
	167:        uint256 currentNounId = _currentNounId;
	168:	        uint256 maxNounId = 0;
	169:	        if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
	170:	        if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
	171:	
	172:	        for (uint256 i = 0; i < tokenIds.length; i++) {
	173:	            uint256 nounId = tokenIds[i];
	174:	            _mintWithOriginalSeed(to, nounId);
	175:	
-	176:	            if (tokenIds[i] > maxNounId) maxNounId = tokenIds[i];
+ 176:	            if (nounId > maxNounId) maxNounId = nounId;
	177:	        }
		
		        // This treats an important case:
		        // During a forking period, people can buy new Nouns on auction, with a higher ID than the auction ID at forking
		        // They can then join the fork with those IDs
		        // If we don't increment currentNounId, unpausing the fork auction house would revert
		        // Since it would attempt to mint a noun with an ID that already exists
	184:	        if (maxNounId >= currentNounId) _currentNounId = maxNounId + 1;
	185:	 }
```

Using `storage` instead of `memory` saves gas

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another `memory` array/struct

- list
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L401C9-L401C53
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L589
    
    ```solidity
    401: address[] memory signers = proposal.signers;
    
    589: address[] memory signers = proposal.signers;
    ```
    
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L355
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L385
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L412
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L451
    
    ```solidity
    
    ```
    
    - https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol#L412
    
    ```solidity
    
    ```