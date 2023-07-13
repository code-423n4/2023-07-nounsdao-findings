# GAS OPTIMIZATIONS

## [G-01]   Call block.timestamp direclty instead of function
Rather than writing a function to return block.timestamp and calling the function whenever you need access block.timestamp you should call block.timestamp directly these saves deployment gas and having to do two JUMP instructions, along with stack setup when calling the function.

### Instances

1. https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L204-#L207
```solidity
function getBlockTimestamp() internal view returns (uint256) {
    // solium-disable-next-line security/no-block-members
    return block.timestamp;
}
```
The function above should be removed from the contract removing it saves 66854 units of deployment gas .
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L177 
```solidity
176:    require(
177:        getBlockTimestamp() >= eta,
178:        "NounsDAOExecutor::executeTransaction: Transaction hasn't surpassed time lock."
179:    );
```
The above line of code should be refactored to the code below this saves 65 gas units:
```solidity
176:    require(
177:        block.timestamp >= eta,
178:        "NounsDAOExecutor::executeTransaction: Transaction hasn't surpassed time lock."
179:    );

```
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L181
```solidity
180:    require(
181:        getBlockTimestamp() <= eta + GRACE_PERIOD,
182:        'NounsDAOExecutor::executeTransaction: Transaction is stale.'
183:    );
```
The above line of code should be refactored to the code below this saves 65 gas units:
```solidity
180:    require(
181:        block.timestamp <= eta + GRACE_PERIOD,
182:        'NounsDAOExecutor::executeTransaction: Transaction is stale.'
183:    );

```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L139
```solidity
138:    require(
139:        eta >= getBlockTimestamp() + delay,
140:        'NounsDAOExecutor::queueTransaction: Estimated execution block must satisfy delay.'
141:    );
```
The above line of code should be refactored to the code below this saves 65 gas units:
```solidity
138:    require(
139:        eta >= block.timestamp + delay,
140:        'NounsDAOExecutor::queueTransaction: Estimated execution block must satisfy delay.'
141:    );

```
2. https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L181-#L184
```solidity
function getBlockTimestamp() internal view returns (uint256) {
    // solium-disable-next-line security/no-block-members
    return block.timestamp;
}
```
The function above should be removed from the contract removing it saves 66854 units of deployment gas .

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L116
```solidity
115:    require(
116:        eta >= getBlockTimestamp() + delay,
117:        'NounsDAOExecutor::queueTransaction: Estimated execution block must satisfy delay.'
118:    );
```
The above line of code should be refactored to the code below this saves 65 gas units:
```solidity
115:    require(
116:        eta >= block.timestamp + delay,
117:        'NounsDAOExecutor::queueTransaction: Estimated execution block must satisfy delay.'
118:    );

```
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L154
```solidity
153:    require(
154:        getBlockTimestamp() >= eta,
155:        "NounsDAOExecutor::executeTransaction: Transaction hasn't surpassed time lock."
156:    );
```
The above line of code should be refactored to the code below this saves 65 gas units:
```solidity
153:    require(
154:        block.timestamp >= eta,
155:        "NounsDAOExecutor::executeTransaction: Transaction hasn't surpassed time lock."
156:    );
```
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L158
```solidity
157:    require(
158:        getBlockTimestamp() <= eta + GRACE_PERIOD,
159:        'NounsDAOExecutor::executeTransaction: Transaction is stale.'
160:    );
```
The above line of code should be refactored to the code below this saves 65 gas units:
```solidity
157:    require(
158:        block.timestamp <= eta + GRACE_PERIOD,
159:        'NounsDAOExecutor::executeTransaction: Transaction is stale.'
160:    );
```
```solidity
Total Gas saved: 134098 units.
```

## [G-02] Unused Imports
Some files were imported and were not used these files costs gas during deployment and generally this is bad coding practice
### 7 Instances

In the file below `NounsDAOStorageV3` was imported in the statement `import { IForkDAODeployer, INounsDAOForkEscrow, NounsDAOStorageV3 } from '../NounsDAOInterfaces.sol';` but was not used consider removing this import.
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol#L21


In the file below `NounsDAOProxy` was imported in the statement `import { NounsDAOProxy } from '../NounsDAOProxy.sol';` but was not used consider removing this import.

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol#L25

In the file below `IERC721` was imported in the statement `import { IERC721 } from '@openzeppelin/contracts/token/ERC721/IERC721.sol';` but was not used consider removing this import.
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L25

In the file below `NounsDAOForkEscrow` was imported in the statement `import { NounsDAOForkEscrow } from '../contracts/governance/fork/NounsDAOForkEscrow.sol';` but was not used consider removing this import.
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L6

In the file below `ForkDAODeployer` was imported in the statement `import { ForkDAODeployer } from '../contracts/governance/fork/ForkDAODeployer.sol';` but was not used consider removing this import.
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L7

In the file below `NounsDAOForkEscrow` was imported in the statement `import { NounsDAOForkEscrow } from '../contracts/governance/fork/NounsDAOForkEscrow.sol';` but was not used consider removing this import.
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L6

In the file below `ForkDAODeployer` was imported in the statement `import { ForkDAODeployer } from '../contracts/governance/fork/ForkDAODeployer.sol';` but was not used consider removing this import.
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L7


## [G-03]  Bytes constants are more efficient than string constants
If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

### 6 Instances
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L59
```solidity
 59:    string public constant name = 'Nouns DAO';
```
The line of code above should be refactored to:
```solidity
59:    bytes32 public constant name = "Nouns DAO";
```
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L44
```solidity
44:    string public constant name = 'Nouns DAO';
```
The line of code above should be refactored to:
```solidity
44:    bytes32 public constant name = "Nouns DAO";
```
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L82

```solidity
82:    string public constant NAME = 'NounsDAOExecutorV2';
```
The line of code above should be refactored to:
```solidity
82:    bytes32 public constant NAME = "NounsDAOExecutorV2";
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L48
```solidity
48:    string public constant NAME = 'NounsAuctionHouseFork';
```
The line of code above should be refactored to:
```solidity
48:    bytes32 public constant NAME = "NounsAuctionHouseFork";
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L116
```solidity
116:    string public constant name = 'Nouns DAO';
```
The line of code above should be refactored to:
```solidity
116:    bytes32 public constant name = "Nouns DAO";
```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L46
```solidity
46:    string public constant NAME = 'NounsTokenFork';
```
The line of code above should be refactored to:
```solidity
46:    bytes32 public constant NAME = "NounsTokenFork";
```


## [G-04] Checking msg.sender to not be zero address is redundant
This check is redundant as no private key is known for this address, hence there can be no transactions coming from the zero address.
### 3 Instances
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L856
```solidity
856:    require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');
```
In the line of code above checking if msg.sender is equal to zero is redundant as no private key is known for this address, hence there can be no transactions coming from the zero address. Performing this check involves the following operations `AND` 3 gas units, `CALLER` 2 gas units, `EQ` 3 gas units and `ISZERO` 3 gas units totalling 11 gas units. I would recommend that this check be removed. The code could be refactored to:
```solidity
856:    require(msg.sender == pendingAdmin, 'NounsDAO::_acceptAdmin: pending admin only');

```
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L279
```solidity
278:    require(
279:        msg.sender == ds.pendingAdmin && msg.sender != address(0),
280:        'NounsDAO::_acceptAdmin: pending admin only'
281:    );
```
In the snippet of code above checking if msg.sender is equal to zero is redundant as no private key is known for this address, hence there can be no transactions coming from the zero address. Performing this check involves the following operations `AND` 3 gas units, `CALLER` 2 gas units, `EQ` 3 gas units and `ISZERO` 3 gas units totalling 11 gas units. I would recommend that this check be removed. The code could be refactored to:
```solidity
278:    require(
279:        msg.sender == ds.pendingAdmin,
280:        'NounsDAO::_acceptAdmin: pending admin only'
281:    );
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L733
```solidity
733:    require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');
```
In the line of code above checking if msg.sender is equal to zero is redundant as no private key is known for this address, hence there can be no transactions coming from the zero address. Performing this check involves the following operations `AND` 3 gas units, `CALLER` 2 gas units, `EQ` 3 gas units and `ISZERO` 3 gas units totalling 11 gas units. I would recommend that this check be removed. The code could be refactored to:
```solidity
733:    require(msg.sender == pendingAdmin, 'NounsDAO::_acceptAdmin: pending admin only');
```
```
Total gas saved: 33 units
```


## [G-06] Unnecessary computation
When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.
### Proof of Concept
Replace:
```
    address oldOwner = _owner;
    _owner = newOwner;
    emit OwnershipTransferred(oldOwner, newOwner)
```
with:
```
    emit OwnershipTransferred(_owner_, newOwner)
    _owner = newOwner;
```
### 19 Instances
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L644-#L656
```solidity
644:    function _setVotingDelay(uint256 newVotingDelay) external {
645:        if (msg.sender != admin) {
646:            revert AdminOnly();
647:        }
648:        require(
649:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
650:            'NounsDAO::_setVotingDelay: invalid voting delay'
651:        );
652:        uint256 oldVotingDelay = votingDelay;
653:        votingDelay = newVotingDelay;
654:
655:        emit VotingDelaySet(oldVotingDelay, votingDelay);
656:    }
```
In the code above declaring the variable `oldVotingDelay` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
644:    function _setVotingDelay(uint256 newVotingDelay) external {
645:        if (msg.sender != admin) {
646:            revert AdminOnly();
647:        }
648:        require(
649:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
650:            'NounsDAO::_setVotingDelay: invalid voting delay'
651:        );
652:
653:        emit VotingDelaySet(votingDelay, newVotingDelay);
654:            votingDelay = newVotingDelay;
655:    }

``` 
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L622-#L674
```solidity
662:    function _setVotingPeriod(uint256 newVotingPeriod) external {
663:        if (msg.sender != admin) {
664:            revert AdminOnly();
665:        }
666:        require(
667:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
668:            'NounsDAO::_setVotingPeriod: invalid voting period'
669:        );
670:        uint256 oldVotingPeriod = votingPeriod;
671:        votingPeriod = newVotingPeriod;
672:
673:        emit VotingPeriodSet(oldVotingPeriod, votingPeriod);
674:    }
```
In the code above declaring the variable `oldVotingPeriod` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
662:    function _setVotingPeriod(uint256 newVotingPeriod) external {
663:        if (msg.sender != admin) {
664:            revert AdminOnly();
665:        }
666:        require(
667:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
668:            'NounsDAO::_setVotingPeriod: invalid voting period'
669:        );
670:
671:        emit VotingPeriodSet(votingPeriod, newVotingPeriod);
672:        votingPeriod = newVotingPeriod;
673:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L681-#L694
```solidity
681:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
682:        if (msg.sender != admin) {
683:            revert AdminOnly();
684:        }
685:        require(
686:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
687:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
688:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
689:        );
690:        uint256 oldProposalThresholdBPS = proposalThresholdBPS;
691:        proposalThresholdBPS = newProposalThresholdBPS;
692:
693:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS);
694:    }
```
In the code above declaring the variable `oldProposalThresholdBPS` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
681:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
682:        if (msg.sender != admin) {
683:            revert AdminOnly();
684:        }
685:        require(
686:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
687:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
688:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
689:        );
690:
691:        emit ProposalThresholdBPSSet(proposalThresholdBPS, newProposalThresholdBPS);
692:        proposalThresholdBPS = newProposalThresholdBPS;
693:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L702-#L724
```solidity
702:    function _setMinQuorumVotesBPS(uint16 newMinQuorumVotesBPS) external {
703:        if (msg.sender != admin) {
704:            revert AdminOnly();
705:        }
706:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
707:
708:        require(
709:            newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
710:                newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
711:            'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
712:        );
713:        require(
714:            newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
715:            'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
716:        );
717:
718:        uint16 oldMinQuorumVotesBPS = params.minQuorumVotesBPS;
719:        params.minQuorumVotesBPS = newMinQuorumVotesBPS;
720:
721:        _writeQuorumParamsCheckpoint(params);
722:
723:        emit MinQuorumVotesBPSSet(oldMinQuorumVotesBPS, newMinQuorumVotesBPS);
724:    }
```
In the code above declaring the variable `oldMinQuorumVotesBPS` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
702:    function _setMinQuorumVotesBPS(uint16 newMinQuorumVotesBPS) external {
703:        if (msg.sender != admin) {
704:            revert AdminOnly();
705:        }
706:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
707:
708:        require(
709:            newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
710:                newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
711:            'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
712:        );
713:        require(
714:            newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
715:            'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
716:        );
717:
718:        _writeQuorumParamsCheckpoint(params);
719:
720:        emit MinQuorumVotesBPSSet(params.minQuorumVotesBPS;, newMinQuorumVotesBPS);
721:        params.minQuorumVotesBPS = newMinQuorumVotesBPS;
722:    }
```
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L732-#L753
```solidity
732:    function _setMaxQuorumVotesBPS(uint16 newMaxQuorumVotesBPS) external {
733:        if (msg.sender != admin) {
734:            revert AdminOnly();
735:        }
736:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
737:
738:        require(
739:            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
740:            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
741:        );
742:        require(
743:            params.minQuorumVotesBPS <= newMaxQuorumVotesBPS,
744:            'NounsDAO::_setMaxQuorumVotesBPS: min quorum votes bps greater than max'
745:        );
746:
747:        uint16 oldMaxQuorumVotesBPS = params.maxQuorumVotesBPS;
748:        params.maxQuorumVotesBPS = newMaxQuorumVotesBPS;
749:
750:        _writeQuorumParamsCheckpoint(params);
751:
752:        emit MaxQuorumVotesBPSSet(oldMaxQuorumVotesBPS, newMaxQuorumVotesBPS);
753:    }
```
In the code above declaring the variable `oldMaxQuorumVotesBPS` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
732:    function _setMaxQuorumVotesBPS(uint16 newMaxQuorumVotesBPS) external {
733:        if (msg.sender != admin) {
734:            revert AdminOnly();
735:        }
736:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
737:
738:        require(
739:            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
740:            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
741:        );
742:        require(
743:            params.minQuorumVotesBPS <= newMaxQuorumVotesBPS,
744:            'NounsDAO::_setMaxQuorumVotesBPS: min quorum votes bps greater than max'
745:        );
746:
747:        _writeQuorumParamsCheckpoint(params);
748:
749:        emit MaxQuorumVotesBPSSet(params.maxQuorumVotesBPS;, newMaxQuorumVotesBPS);
750:        params.maxQuorumVotesBPS = newMaxQuorumVotesBPS;
751:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L759-#L771
```solidity
759:    function _setQuorumCoefficient(uint32 newQuorumCoefficient) external {
760:        if (msg.sender != admin) {
761:            revert AdminOnly();
762:        }
763:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
764:
765:        uint32 oldQuorumCoefficient = params.quorumCoefficient;
766:        params.quorumCoefficient = newQuorumCoefficient;
767:
768:        _writeQuorumParamsCheckpoint(params);
769:
770        emit QuorumCoefficientSet(oldQuorumCoefficient, newQuorumCoefficient);
771:    }
```
In the code above declaring the variable `oldQuorumCoefficient` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
759:    function _setQuorumCoefficient(uint32 newQuorumCoefficient) external {
760:        if (msg.sender != admin) {
761:            revert AdminOnly();
762:        }
763:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
764:
765:        _writeQuorumParamsCheckpoint(params);
766:
767        emit QuorumCoefficientSet(params.quorumCoefficient, newQuorumCoefficient);
768:        params.quorumCoefficient = newQuorumCoefficient;
779:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L78-#L86
```solidity
78:    function _setImplementation(address implementation_) public {
79:        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
80:        require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');
81:
82:        address oldImplementation = implementation;
83:        implementation = implementation_;
84:
85:        emit NewImplementation(oldImplementation, implementation);
86:    }
```
In the code above declaring the variable `oldImplementation` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
78:    function _setImplementation(address implementation_) public {
79:        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
80:        require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');
81:
82:        emit NewImplementation(implementation, implementation_);
83:        implementation = implementation_;
84:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L162-#L171
```solidity
162:    function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external onlyAdmin(ds) {
163:        require(
164:            newVotingDelay >= MIN_VOTING_DELAY_BLOCKS && newVotingDelay <= MAX_VOTING_DELAY_BLOCKS,
165:            'NounsDAO::_setVotingDelay: invalid voting delay'
166:        );
167:        uint256 oldVotingDelay = ds.votingDelay;
168:        ds.votingDelay = newVotingDelay;
169:
170:        emit VotingDelaySet(oldVotingDelay, newVotingDelay);
171:    }
```
In the code above declaring the variable `oldVotingDelay` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
162:    function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external onlyAdmin(ds) {
163:        require(
164:            newVotingDelay >= MIN_VOTING_DELAY_BLOCKS && newVotingDelay <= MAX_VOTING_DELAY_BLOCKS,
165:            'NounsDAO::_setVotingDelay: invalid voting delay'
166:        );
167:
168:        emit VotingDelaySet(ds.votingDelay, newVotingDelay);
169:        ds.votingDelay = newVotingDelay;
170:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L177-#L186
```solidity
177:    function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external onlyAdmin(ds) {
178:        require(
179:            newVotingPeriod >= MIN_VOTING_PERIOD_BLOCKS && newVotingPeriod <= MAX_VOTING_PERIOD_BLOCKS,
180:            'NounsDAO::_setVotingPeriod: invalid voting period'
181:        );
182:        uint256 oldVotingPeriod = ds.votingPeriod;
183:        ds.votingPeriod = newVotingPeriod;
184:
185:        emit VotingPeriodSet(oldVotingPeriod, newVotingPeriod);
186:    }
```
In the code above declaring the variable `oldVotingPeriod` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
177:    function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external onlyAdmin(ds) {
178:        require(
179:            newVotingPeriod >= MIN_VOTING_PERIOD_BLOCKS && newVotingPeriod <= MAX_VOTING_PERIOD_BLOCKS,
180:            'NounsDAO::_setVotingPeriod: invalid voting period'
181:        );
184:
185:        emit VotingPeriodSet(ds.votingPeriod, newVotingPeriod);
183:        ds.votingPeriod = newVotingPeriod;
186:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L193-#L206
```solidity
193:    function _setProposalThresholdBPS(NounsDAOStorageV3.StorageV3 storage ds, uint256 newProposalThresholdBPS)
194:        external
195:        onlyAdmin(ds)
196:    {
197:        require(
198:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
199:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
200:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
201:        );
202:        uint256 oldProposalThresholdBPS = ds.proposalThresholdBPS;
203:        ds.proposalThresholdBPS = newProposalThresholdBPS;
204:
205:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, newProposalThresholdBPS);
206:    }
```
In the code above declaring the variable `oldProposalThresholdBPS` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 

```solidity
193:    function _setProposalThresholdBPS(NounsDAOStorageV3.StorageV3 storage ds, uint256 newProposalThresholdBPS)
194:        external
195:        onlyAdmin(ds)
196:    {
197:        require(
198:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
199:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
200:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
201:        );
202:
203:        emit ProposalThresholdBPSSet(ds.proposalThresholdBPS, newProposalThresholdBPS);
204:        ds.proposalThresholdBPS = newProposalThresholdBPS;
205:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L212-#L223
```solidity
212:    function _setObjectionPeriodDurationInBlocks(
213:        NounsDAOStorageV3.StorageV3 storage ds,
214:        uint32 newObjectionPeriodDurationInBlocks
215:    ) external onlyAdmin(ds) {
216:        if (newObjectionPeriodDurationInBlocks > MAX_OBJECTION_PERIOD_BLOCKS)
217:            revert InvalidObjectionPeriodDurationInBlocks();
218:
219:        uint32 oldObjectionPeriodDurationInBlocks = ds.objectionPeriodDurationInBlocks;
220:        ds.objectionPeriodDurationInBlocks = newObjectionPeriodDurationInBlocks;
221:
222:        emit ObjectionPeriodDurationSet(oldObjectionPeriodDurationInBlocks, newObjectionPeriodDurationInBlocks);
223:    }
```
In the code above declaring the variable `oldObjectionPeriodDurationInBlocks` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
212:    function _setObjectionPeriodDurationInBlocks(
213:        NounsDAOStorageV3.StorageV3 storage ds,
214:        uint32 newObjectionPeriodDurationInBlocks
215:    ) external onlyAdmin(ds) {
216:        if (newObjectionPeriodDurationInBlocks > MAX_OBJECTION_PERIOD_BLOCKS)
217:            revert InvalidObjectionPeriodDurationInBlocks();
218:
219:        emit ObjectionPeriodDurationSet(ds.objectionPeriodDurationInBlocks, newObjectionPeriodDurationInBlocks);
220:        ds.objectionPeriodDurationInBlocks = newObjectionPeriodDurationInBlocks;
221:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L229-#L237
```solidity
229:    function _setLastMinuteWindowInBlocks(NounsDAOStorageV3.StorageV3 storage ds, uint32 newLastMinuteWindowInBlocks)
230:        external
231:        onlyAdmin(ds)
232:    {
233:        uint32 oldLastMinuteWindowInBlocks = ds.lastMinuteWindowInBlocks;
234:        ds.lastMinuteWindowInBlocks = newLastMinuteWindowInBlocks;
235:
236:        emit LastMinuteWindowSet(oldLastMinuteWindowInBlocks, newLastMinuteWindowInBlocks);
237:    }
```
In the code above declaring the variable `oldLastMinuteWindowInBlocks` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
229:    function _setLastMinuteWindowInBlocks(NounsDAOStorageV3.StorageV3 storage ds, uint32 newLastMinuteWindowInBlocks)
230:        external
231:        onlyAdmin(ds)
232:    {
233:        emit LastMinuteWindowSet(ds.lastMinuteWindowInBlocks, newLastMinuteWindowInBlocks);
234:        ds.lastMinuteWindowInBlocks = newLastMinuteWindowInBlocks;
235:    }
```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L243-#L254
```solidity
243:    function _setProposalUpdatablePeriodInBlocks(
244:        NounsDAOStorageV3.StorageV3 storage ds,
245:        uint32 newProposalUpdatablePeriodInBlocks
246:    ) external onlyAdmin(ds) {
247:        if (newProposalUpdatablePeriodInBlocks > MAX_UPDATABLE_PERIOD_BLOCKS)
248:            revert InvalidProposalUpdatablePeriodInBlocks();
249:
250:        uint32 oldProposalUpdatablePeriodInBlocks = ds.proposalUpdatablePeriodInBlocks;
251:        ds.proposalUpdatablePeriodInBlocks = newProposalUpdatablePeriodInBlocks;
252:
253:        emit ProposalUpdatablePeriodSet(oldProposalUpdatablePeriodInBlocks, newProposalUpdatablePeriodInBlocks);
254:    }
```
In the code above declaring the variable `oldProposalUpdatablePeriodInBlocks` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
243:    function _setProposalUpdatablePeriodInBlocks(
244:        NounsDAOStorageV3.StorageV3 storage ds,
245:        uint32 newProposalUpdatablePeriodInBlocks
246:    ) external onlyAdmin(ds) {
247:        if (newProposalUpdatablePeriodInBlocks > MAX_UPDATABLE_PERIOD_BLOCKS)
248:            revert InvalidProposalUpdatablePeriodInBlocks();
249:
250:        emit ProposalUpdatablePeriodSet(ds.proposalUpdatablePeriodInBlocks, newProposalUpdatablePeriodInBlocks);
251:        ds.proposalUpdatablePeriodInBlocks = newProposalUpdatablePeriodInBlocks;
252:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L261-#L270
```solidity
261:    function _setPendingAdmin(NounsDAOStorageV3.StorageV3 storage ds, address newPendingAdmin) external onlyAdmin(ds) {
262:        // Save current value, if any, for inclusion in log
263:        address oldPendingAdmin = ds.pendingAdmin;
264:
265:        // Store pendingAdmin with value newPendingAdmin
266:        ds.pendingAdmin = newPendingAdmin;
267:
268:        // Emit NewPendingAdmin(oldPendingAdmin, newPendingAdmin)
269:        emit NewPendingAdmin(oldPendingAdmin, newPendingAdmin);
270:    }
```
In the code above declaring the variable `oldPendingAdmin` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
261:    function _setPendingAdmin(NounsDAOStorageV3.StorageV3 storage ds, address newPendingAdmin) external onlyAdmin(ds) {
262:
263:        // Emit NewPendingAdmin(ds.pendingAdmin, newPendingAdmin)
264:        emit NewPendingAdmin(ds.pendingAdmin, newPendingAdmin);
265:
266:        // Store pendingAdmin with value newPendingAdmin
267:        ds.pendingAdmin = newPendingAdmin;
268:    }
```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L351-#L373
```solidity
351:    function _setMinQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMinQuorumVotesBPS)
352:        external
353:        onlyAdmin(ds)
354:    {
355:        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
356:
357:        require(
358:            newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
359:                newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
360:            'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
361:        );
362:        require(
363:            newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
364:            'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
365:        );
366:
367:        uint16 oldMinQuorumVotesBPS = params.minQuorumVotesBPS;
368:        params.minQuorumVotesBPS = newMinQuorumVotesBPS;
369:
370:        _writeQuorumParamsCheckpoint(ds, params);
371:
372:        emit MinQuorumVotesBPSSet(oldMinQuorumVotesBPS, newMinQuorumVotesBPS);
373:    }
```
In the code above declaring the variable `oldMinQuorumVotesBPS` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
351:    function _setMinQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMinQuorumVotesBPS)
352:        external
353:        onlyAdmin(ds)
354:    {
355:        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
356:
357:        require(
358:            newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
359:                newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
360:            'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
361:        );
362:        require(
363:            newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
364:            'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
365:        );
366:
367:        _writeQuorumParamsCheckpoint(ds, params);
368:
369:        emit MinQuorumVotesBPSSet(params.minQuorumVotesBPS, newMinQuorumVotesBPS);
370:        params.minQuorumVotesBPS = newMinQuorumVotesBPS;
371:    }
```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L381-#L402
```solidity
381:    function _setMaxQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMaxQuorumVotesBPS)
382:        external
383:        onlyAdmin(ds)
384:    {
385:        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
386:
387:        require(
388:            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
389:            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
390:        );
391:        require(
392:            params.minQuorumVotesBPS <= newMaxQuorumVotesBPS,
393:            'NounsDAO::_setMaxQuorumVotesBPS: min quorum votes bps greater than max'
394:        );
395:
396:        uint16 oldMaxQuorumVotesBPS = params.maxQuorumVotesBPS;
397:        params.maxQuorumVotesBPS = newMaxQuorumVotesBPS;
398:
399:        _writeQuorumParamsCheckpoint(ds, params);
400:
401:        emit MaxQuorumVotesBPSSet(oldMaxQuorumVotesBPS, newMaxQuorumVotesBPS);
402:    }
```
In the code above declaring the variable `oldMaxQuorumVotesBPS` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
381:    function _setMaxQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMaxQuorumVotesBPS)
382:        external
383:        onlyAdmin(ds)
384:    {
385:        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
386:
387:        require(
388:            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
389:            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
390:        );
391:        require(
392:            params.minQuorumVotesBPS <= newMaxQuorumVotesBPS,
393:            'NounsDAO::_setMaxQuorumVotesBPS: min quorum votes bps greater than max'
394:        );
395:
396:        _writeQuorumParamsCheckpoint(ds, params);
397:
398:        emit MaxQuorumVotesBPSSet(params.maxQuorumVotesBPS, newMaxQuorumVotesBPS);
399:        params.maxQuorumVotesBPS = newMaxQuorumVotesBPS;
400:    }
```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L645-#L655
```solidity
645:    function _setVotingDelay(uint256 newVotingDelay) external {
646:        require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
647:        require(
648:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
649:            'NounsDAO::_setVotingDelay: invalid voting delay'
650:        );
651:        uint256 oldVotingDelay = votingDelay;
652:        votingDelay = newVotingDelay;
653:
654:        emit VotingDelaySet(oldVotingDelay, newVotingDelay);
656:    }
```
In the code above declaring the variable `oldVotingDelay` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
645:    function _setVotingDelay(uint256 newVotingDelay) external {
646:        require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
647:        require(
648:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
649:            'NounsDAO::_setVotingDelay: invalid voting delay'
650:        );
651:
652:        emit VotingDelaySet(votingDelay, newVotingDelay);
653:        votingDelay = newVotingDelay;
654:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L661-#L671
```solidity
661:    function _setVotingPeriod(uint256 newVotingPeriod) external {
662:        require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
663:        require(
664:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
665:            'NounsDAO::_setVotingPeriod: invalid voting period'
666:        );
667:        uint256 oldVotingPeriod = votingPeriod;
668:        votingPeriod = newVotingPeriod;
669:
670:        emit VotingPeriodSet(oldVotingPeriod, newVotingPeriod);
671:    }
```
In the code above declaring the variable `oldVotingPeriod` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
661:    function _setVotingPeriod(uint256 newVotingPeriod) external {
662:        require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
663:        require(
664:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
665:            'NounsDAO::_setVotingPeriod: invalid voting period'
666:        );
667:
668:        emit VotingPeriodSet(votingPeriod;, newVotingPeriod);
669:        votingPeriod = newVotingPeriod;
670:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L678-#L689
```solidity
678:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
679:        require(msg.sender == admin, 'NounsDAO::_setProposalThresholdBPS: admin only');
680:        require(
681:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
682:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
683:            'NounsDAO::_setProposalThreshold: invalid proposal threshold'
684:        );
685:        uint256 oldProposalThresholdBPS = proposalThresholdBPS;
686:        proposalThresholdBPS = newProposalThresholdBPS;
687:
688:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, newProposalThresholdBPS);
689:    }
```
In the code above declaring the variable `oldProposalThresholdBPS` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
678:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
679:        require(msg.sender == admin, 'NounsDAO::_setProposalThresholdBPS: admin only');
680:        require(
681:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
682:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
683:            'NounsDAO::_setProposalThreshold: invalid proposal threshold'
684:        );
687:
688:        emit ProposalThresholdBPSSet(proposalThresholdBPS, newProposalThresholdBPS);
686:        proposalThresholdBPS = newProposalThresholdBPS;
689:    }
```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L696-#L706
```solidity
696:    function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external {
697:        require(msg.sender == admin, 'NounsDAO::_setQuorumVotesBPS: admin only');
698:        require(
699:            newQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS && newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS,
700:            'NounsDAO::_setQuorumVotesBPS: invalid quorum votes basis points'
701:        );
702:        uint256 oldQuorumVotesBPS = quorumVotesBPS;
703:        quorumVotesBPS = newQuorumVotesBPS;
704:
705:        emit QuorumVotesBPSSet(oldQuorumVotesBPS, newQuorumVotesBPS);
706:    }
```
In the code above declaring the variable `oldQuorumVotesBPS` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to: 
```solidity
696:    function _setQuorumVotesBPS(uint256 newQuorumVotesBPS) external {
697:        require(msg.sender == admin, 'NounsDAO::_setQuorumVotesBPS: admin only');
698:        require(
699:            newQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS && newQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS,
700:            'NounsDAO::_setQuorumVotesBPS: invalid quorum votes basis points'
701:        );
702:
703:        emit QuorumVotesBPSSet(quorumVotesBP, newQuorumVotesBPS);
704:        quorumVotesBPS = newQuorumVotesBPS;
705:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L713-#L725
```solidity
713:    function _setPendingAdmin(address newPendingAdmin) external {
714:        // Check caller = admin
715:        require(msg.sender == admin, 'NounsDAO::_setPendingAdmin: admin only');
716:
717:        // Save current value, if any, for inclusion in log
718:        address oldPendingAdmin = pendingAdmin;
719:
720:        // Store pendingAdmin with value newPendingAdmin
721:        pendingAdmin = newPendingAdmin;
722:
723:        // Emit NewPendingAdmin(oldPendingAdmin, newPendingAdmin)
724:        emit NewPendingAdmin(oldPendingAdmin, newPendingAdmin);
725:    }
```
In the code above declaring the variable `oldPendingAdmin` is unneccessary as it cost `MSTORE` 3 gas units and `MLOAD` 3 gas units operations. Therefore 6 gas units could be saved by refactoring the code to:
```solidity
713:    function _setPendingAdmin(address newPendingAdmin) external {
714:        // Check caller = admin
715:        require(msg.sender == admin, 'NounsDAO::_setPendingAdmin: admin only');
716:
717:        // Emit NewPendingAdmin(pendingAdmin, newPendingAdmin)
718:        emit NewPendingAdmin(pendingAdmin, newPendingAdmin);
719
720:        // Store pendingAdmin with value newPendingAdmin
721:        pendingAdmin = newPendingAdmin;
722:    }
```
```
Total gas saved: 6 * 19 = 114 units
```

## [G-07] Costly keccak of parameter
keccak of calldata parameter takes extra 150 gas for no apparent reason. Using assembly to hash the calldata parameter is cheaper.
### 1 Instance
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L271
```solidity
271:    bytes32 sigHash = keccak256(sig);
```
I would recommend using assembly in this scenario as it would save 150 gas units. The code could be refactored to:
```solidity
    bytes32 sigHash;
    assembly {
        let mem := mload(0x40)
        let len := sig.length
        calldatacopy(mem, sig.offset, len)
        sigHash := keccak256(mem, len)
    }
```
```
Total gas saved: 150 units
```

## [G-08]  Move lesser gas costing require checks to the top
Require() or revert() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### 7 Instances
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L135-#L178
```solidity
144:    require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
145:    if (msg.sender != admin) {
146:        revert AdminOnly();
147:    }
148:    require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
149:    require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
150:    require(
151:        votingPeriod_ >= MIN_VOTING_PERIOD && votingPeriod_ <= MAX_VOTING_PERIOD,
152:        'NounsDAO::initialize: invalid voting period'
153:    );
154:    require(
155:        votingDelay_ >= MIN_VOTING_DELAY && votingDelay_ <= MAX_VOTING_DELAY,
156:        'NounsDAO::initialize: invalid voting delay'
157:    );
158:    require(
159:        proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,
160:        'NounsDAO::initialize: invalid proposal threshold bps'
161:    );
```
In the code above the `require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');` and the `require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');` statements are the least gas consuming statements so they should be moved to the top of the function so that in scenarios where their require check fails the transaction can revert without consuming too much gas. The code could be refactored to:
 ```solidity
144:    require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
145:    require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
146:    require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
147:    if (msg.sender != admin) {
148:        revert AdminOnly();
149:    }
150:    require(
151:        votingPeriod_ >= MIN_VOTING_PERIOD && votingPeriod_ <= MAX_VOTING_PERIOD,
152:        'NounsDAO::initialize: invalid voting period'
153:    );
154:    require(
155:        votingDelay_ >= MIN_VOTING_DELAY && votingDelay_ <= MAX_VOTING_DELAY,
156:        'NounsDAO::initialize: invalid voting delay'
157:    );
158:    require(
159:        proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,
160:        'NounsDAO::initialize: invalid proposal threshold bps'
161:    );
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L197-#L292
```solidity
210:    require(
211:        nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
212:        'NounsDAO::propose: proposer votes below proposal threshold'
213:    );
214:    require(
215:        targets.length == values.length &&
216:            targets.length == signatures.length &&
217:            targets.length == calldatas.length,
218:        'NounsDAO::propose: proposal function information arity mismatch'
219:    );
220:    require(targets.length != 0, 'NounsDAO::propose: must provide actions');
221:    require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```
In the above code the `require(targets.length != 0, 'NounsDAO::propose: must provide actions');` statement is the least gas consuming statement so it should be moved to the top of the function so that in scenarios where the require check fails the transaction can revert without consuming too much gas. The code could be refactored to:
```solidity
210:    require(targets.length != 0, 'NounsDAO::propose: must provide actions');
211:    require(
212:        nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
213:        'NounsDAO::propose: proposer votes below proposal threshold'
214:    );
215:    require(
216:        targets.length == values.length &&
217:            targets.length == signatures.length &&
218:            targets.length == calldatas.length,
219:        'NounsDAO::propose: proposal function information arity mismatch'
220:    );
221:    require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L141-#L172
```solidity
150:    if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
151:    if (msg.sender != ds.admin) revert AdminOnly();
152:    if (timelock_ == address(0)) revert InvalidTimelockAddress();
153:    if (nouns_ == address(0)) revert InvalidNounsAddress();
```
In the above code the `if (timelock_ == address(0)) revert InvalidTimelockAddress();` and `153:    if (nouns_ == address(0)) revert InvalidNounsAddress();` statements are the least gas consuming statements so they should be moved to the top of the function so that in scenarios where their if condition fails the transaction can revert without consuming too much gas. The code could be refactored to:
```solidity
150:    if (timelock_ == address(0)) revert InvalidTimelockAddress();
151:    if (nouns_ == address(0)) revert InvalidNounsAddress();
152:    if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
153:    if (msg.sender != ds.admin) revert AdminOnly();
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L142-#L179
```solidity
149:    require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');
150:    bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
151:    require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");
152:    require(
153:        getBlockTimestamp() >= eta,
154:        "NounsDAOExecutor::executeTransaction: Transaction hasn't surpassed time lock."
155:    );
156:    require(
157:        getBlockTimestamp() <= eta + GRACE_PERIOD,
158:        'NounsDAOExecutor::executeTransaction: Transaction is stale.'
159:    );
```
In the above code `require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");` statement is the least gas consuming statement so it should be moved to the top of the function so that in scenarios where the require check fails the transaction can revert without consuming too much gas. The code could be refactored to:
```solidity
149:    require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");
150:    require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');
151:    bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
152:    require(
153:        getBlockTimestamp() >= eta,
154:        "NounsDAOExecutor::executeTransaction: Transaction hasn't surpassed time lock."
155:    );
156:    require(
157:        getBlockTimestamp() <= eta + GRACE_PERIOD,
158:        'NounsDAOExecutor::executeTransaction: Transaction is stale.'
159:    );
```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L165-#L194
```solidity
176:    require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
177:    require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
178:    require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
```
In the code above the `require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');` and the `require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');` statements are the least gas consuming statements so they should be moved to the top of the function so that in scenarios where their require check fails the transaction can revert without consuming too much gas. The code could be refactored to:
```solidity
176:    require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
177:    require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
178:    require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L272-#L368
```solidity
287:    require(
288:        nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
289:        'NounsDAO::propose: proposer votes below proposal threshold'
290:    );
291:    require(
292:        targets.length == values.length &&
293:            targets.length == signatures.length &&
294:            targets.length == calldatas.length,
295:        'NounsDAO::propose: proposal function information arity mismatch'
296:    );
297:    require(targets.length != 0, 'NounsDAO::propose: must provide actions');
298:    require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```
In the above code `require(targets.length != 0, 'NounsDAO::propose: must provide actions');` statement is the least gas consuming statement so it should be moved to the top of the function so that in scenarios where the require check fails the transaction can revert without consuming too much gas. The code could be refactored to:
```solidity
287:    require(targets.length != 0, 'NounsDAO::propose: must provide actions');
288:    require(
289:        nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
290:        'NounsDAO::propose: proposer votes below proposal threshold'
291:    );
292:    require(
293:        targets.length == values.length &&
294:            targets.length == signatures.length &&
295:            targets.length == calldatas.length,
296:        'NounsDAO::propose: proposal function information arity mismatch'
297:    );
298:    require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L612-#L639
```solidity
617:    require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
618:    require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
```
In the above code `require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');` statement is the least gas consuming statement so it should be moved to the top of the function so that in scenarios where the require check fails the transaction can revert without consuming too much gas. The code could be refactored to:
```solidity
617:    require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
618:    require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
```


## [G-09] Avoid using state variable in emit
In scenarios where you have a memory, calldata variable or parameter with the same value as the state variable you should use the memory, calldata variable or parameter in the emit statement rather than state variable.
### 3 Instances

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L86
```solidity
80:    function setDelay(uint256 delay_) public {
81:        require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
82:        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
83:        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
84:        delay = delay_;
85:
86:        emit NewDelay(delay);
87:    }
```
In the above code rather reading from state which would cost 2100 gas units (cold access) you should emit the `delay_` variable. The code should be refactored to:
```solidity
80:    function setDelay(uint256 delay_) public {
81:        require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
82:        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
83:        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
84:        delay = delay_;
85:
86:        emit NewDelay(delay_);
87:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L94
```solidity
89:    function acceptAdmin() public {
90:        require(msg.sender == pendingAdmin, 'NounsDAOExecutor::acceptAdmin: Call must come from pendingAdmin.');
91:        admin = msg.sender;
92:        pendingAdmin = address(0);
93:
94:        emit NewAdmin(admin);
95:    }
```
In the above code rather reading from state which would cost 2100 gas units (cold access) you should emit the `msg.sender` global variable. The code should be refactored to:
```solidity
89:    function acceptAdmin() public {
90:        require(msg.sender == pendingAdmin, 'NounsDAOExecutor::acceptAdmin: Call must come from pendingAdmin.');
91:        admin = msg.sender;
92:        pendingAdmin = address(0);
93:
94:        emit NewAdmin(msg.sender);
95:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L104
```solidity
97:    function setPendingAdmin(address pendingAdmin_) public {
98:        require(
99:            msg.sender == address(this),
100:           'NounsDAOExecutor::setPendingAdmin: Call must come from NounsDAOExecutor.'
101:       );
102:        pendingAdmin = pendingAdmin_;
103:
104:        emit NewPendingAdmin(pendingAdmin);
105:   }
```
In the above code rather reading from state which would cost 2100 gas units (cold access) you should emit the `pendingAdmin_` global variable. The code should be refactored to:
```solidity
97:    function setPendingAdmin(address pendingAdmin_) public {
98:        require(
99:            msg.sender == address(this),
100:           'NounsDAOExecutor::setPendingAdmin: Call must come from NounsDAOExecutor.'
101:       );
102:        pendingAdmin = pendingAdmin_;
103:
104:        emit NewPendingAdmin(pendingAdmin_);
105:   }
```
```
Total gas saved: 3 * 2100 = 6300 units
```


## [G-10] Use assembly to get contracts eth balance
Use assembly when getting a contract’s balance of ETH.
You can use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas. 
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L823
```solidity
823:    uint256 amount = address(this).balance;
```
15 gas units could be saved if assembly is used to get a contracts ETH balance. The code could be refactored to:
```solidity
    uint256 amount;

    assembly {
        amount := selfbalance()
        mstore(0x00, amount)
        return(0x00, 0x20)
    }

```
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1035
```solidity
1035:    uint256 balance = address(this).balance;
```
15 gas units could be saved if assembly is used to get a contracts ETH balance. The code could be refactored to:
```solidity
    uint256 amount;

    assembly {
        amount := selfbalance()
        mstore(0x00, amount)
        return(0x00, 0x20)
    }

```


- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L469
```solidity
469:    uint256 amount = address(this).balance;
```
15 gas units could be saved if assembly is used to get a contracts ETH balance. The code could be refactored to:
```solidity
    uint256 amount;

    assembly {
        amount := selfbalance()
        mstore(0x00, amount)
        return(0x00, 0x20)
    }

```
```
Total gas saved: 3 * 15 = 45 units
```


## [G-11] Declaring redundant variables
Some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.
### 2 Instances

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L217
```solidity
201:    function _delegate(address delegator, address delegatee) internal {
202:        /// @notice differs from `_delegate()` in `Comp.sol` to use `delegates` override method to simulate auto-delegation
203:        address currentDelegate = delegates(delegator);
204:
205:        _delegates[delegator] = delegatee;
206:
207:        emit DelegateChanged(delegator, currentDelegate, delegatee);
208:
209:        uint96 amount = votesToDelegate(delegator);
210:
211:        _moveDelegates(currentDelegate, delegatee, amount);
212:    }
```
In the function above the `amount` variable is declared and is only used once. You could invoke the `votesToDelegate()` function directly in the `_moveDelegates();` function rather than assigning its value to a memory variable which would cost mstore (3 gas) and mload (3 gas) operations.
The code could be refactored to:
```solidity
201:    function _delegate(address delegator, address delegatee) internal {
202:        /// @notice differs from `_delegate()` in `Comp.sol` to use `delegates` override method to simulate auto-delegation
203:        address currentDelegate = delegates(delegator);
204:
205:        _delegates[delegator] = delegatee;
206:
207:        emit DelegateChanged(delegator, currentDelegate, delegatee);
208:
209:        _moveDelegates(currentDelegate, delegatee, votesToDelegate(delegator));
210:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L53
```solidity
43:    function run()
44:        public
45:        returns (
46:            NounsDAOForkEscrow forkEscrow,
47:            ForkDAODeployer forkDeployer,
48:            NounsDAOLogicV3 daoV3Impl,
49:            NounsDAOExecutorV2 timelockV2,
50:            ERC20Transferer erc20Transferer
51:        )
52:    {
53:        uint256 deployerKey = vm.envUint('DEPLOYER_PRIVATE_KEY');
54:
55:        vm.startBroadcast(deployerKey);
56:
57:        (forkEscrow, forkDeployer, daoV3Impl, timelockV2, erc20Transferer) = deployNewContracts();
58:
59:        vm.stopBroadcast();
60:    }
```
In the function above the `deployerKey` variable is declared and is only used once. You could invoke the `vm.envUint()` function directly in the `vm.startBroadcast()` function rather than assigning its value to a memory variable which would cost mstore (3 gas) and mload (3 gas) operations.
The code could be refactored to:
```solidity
43:    function run()
44:        public
45:        returns (
46:            NounsDAOForkEscrow forkEscrow,
47:            ForkDAODeployer forkDeployer,
48:            NounsDAOLogicV3 daoV3Impl,
49:            NounsDAOExecutorV2 timelockV2,
50:            ERC20Transferer erc20Transferer
51:        )
52:    {
53:
54:        vm.startBroadcast(vm.envUint('DEPLOYER_PRIVATE_KEY'));
55:
56:        (forkEscrow, forkDeployer, daoV3Impl, timelockV2, erc20Transferer) = deployNewContracts();
57:
58:        vm.stopBroadcast();
59:    }
```


## [G-12]  use nested if and avoid multiple check combinations
Rather performing multiple check combinations on if conditionals the conditions could be converted into nested if statements thereby saving gas by avoiding computation for the && operator. This also enhances code readability.
### 4 Instances

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1026
```solidity
1026:    if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
1027:        quorumParamsCheckpoints[pos - 1].params = params;
1028:    } else {
1029:        quorumParamsCheckpoints.push(DynamicQuorumParamsCheckpoint({ fromBlock: blockNumber, params: params }));
1030:    }
```
8 gas units can be saved by refactoring the code as shown below:
```solidity
1026:    if (pos > 0 ) { 
1027:       if(quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
1028:           quorumParamsCheckpoints[pos - 1].params = params;
            }
1028:    } else {
1029:        quorumParamsCheckpoints.push(DynamicQuorumParamsCheckpoint({ fromBlock: blockNumber, params: params }));
1030:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L585
```solidity
585:    if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
586:        ds.quorumParamsCheckpoints[pos - 1].params = params;
587:    } else {
589:        ds.quorumParamsCheckpoints.push(
570:            NounsDAOStorageV3.DynamicQuorumParamsCheckpoint({ fromBlock: blockNumber, params: params })
571:        );
572:    }
```
8 gas units can be saved by refactoring the code as shown below:
```solidity
585:    if (pos > 0) { 
586:        if (ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
587:            ds.quorumParamsCheckpoints[pos - 1].params = params;
588:        }
589:    } else {
590:        ds.quorumParamsCheckpoints.push(
591:            NounsDAOStorageV3.DynamicQuorumParamsCheckpoint({ fromBlock: blockNumber, params: params })
592:        );
593:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L381
```solidity
381:    if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
382:        revert WaitingForTokensToClaimOrExpiration();
383:    }
```
8 gas units can be saved by refactoring the code as shown below:
```solidity
381:    if (block.timestamp < delayedGovernanceExpirationTimestamp) { 
382:        if (nouns.remainingTokensToClaim() > 0) {
383:            revert WaitingForTokensToClaimOrExpiration();
384:        }
385:    }
```

- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L255
```solidity
255:    if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
256:        checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
257:    } else {
258:        checkpoints[delegatee][nCheckpoints] = Checkpoint(blockNumber, newVotes);
259:        numCheckpoints[delegatee] = nCheckpoints + 1;
260:    }
```
8 gas units can be saved by refactoring the code as shown below:
```solidity
255:    if (nCheckpoints > 0) { 
256:        if(checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
257:            checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
258:        }
259:    } else {
260:        checkpoints[delegatee][nCheckpoints] = Checkpoint(blockNumber, newVotes);
261:        numCheckpoints[delegatee] = nCheckpoints + 1;
262:    }
```
```
Total gas saved: 8 * 4 = 32 units
```


## [G-13]  Using calldata for read-only arguments in external functions
Using calldata instead of memory for read-only arguments in external functions saves gas.
### 27 Instances
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L197-#L203
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeTimelockMigrationCleanupMainnet.s.sol#L43-#L50
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L318-#L330
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L556-#L567
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L962-#L971
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L189-#L197
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L210-#L222
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L236-#L250
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L278-#L288
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L313-#L322
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L338-#L355
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L486-#L488
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L130-#L148
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L150-#L163
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L165-#L203
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L220-#L259
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L288-#L310
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L321-#L333
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L383-#L432
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L58-#L68
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L107-#125
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L127-#L140
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L142-#L179
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L165-#L194
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206-216
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L272-#L368
- https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L198-#L200
