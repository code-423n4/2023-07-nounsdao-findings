# **NounsDAO QA Report**

## **Table of Contents**

| Issue | Summary                                                                                                                                                                                                                                                                    |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| L-01  | Potential Front-running Vulnerability in Vote Refund Mechanism. The current vote refund mechanism operates on a first-come, first-served basis, which can be exploited for front-running attacks.                                                                          |
| L-02  | Unreachable 'initialize' Function in NounsDAOLogic Contracts should be removed. The `initialize` function in some contracts is not used due to pre-initialized state variables, rendering it inaccessible.                                                                 |
| L-03  | The 'ProposalCreatedWithRequirement()' event in the `propose()` function should not be emitted using the current `minQuorumVotes()`. The event may provide incorrect information about the required quorum votes if the minimum quorum votes change within the same block. |
| L-04  | Lack of input checks when setting new _quorumCoefficient_ would easily cost an unnecessary proposal process. There is a lack of input checks when setting a new `_quorumCoefficient`, which could result in unnecessary proposal processes.                                |

## L-01 Potential Front-running Vulnerability in Vote Refund Mechanism

### Overview Details

The current vote refund mechanism used by the NounsDAOProxy contract appears to operate on a first-come, first-served basis until the contract's ETH balance is exhausted. While this is likely part of the protocol's design, this approach has the potential to create an environment conducive to front-running attacks and should be given closer examination.

In the existing setup, a user who casts a vote while there is still ETH in the contract balance receives a refund, while a user who votes after the balance has been depleted does not. This creates an incentive for users to monitor the mempool for upcoming votes and to then front-run these transactions with slightly higher gas fees to ensure they receive the refund.

As an example, consider a scenario where Alice notices Bob's voting transaction in the mempool. She sees that the contract's ETH balance is about to deplete and decides to front-run Bob's transaction by paying a slightly higher gas fee. Consequently, Alice's voting transaction is processed first, and she secures a refund before the ETH balance is exhausted. Unfortunately for Bob, by the time his transaction is processed, the contract's balance has depleted, and he receives no refund.

The `_refundGas` function as seen in the NounsDAOLogicV2 contract is the root cause of this issue:

```solidity
function _refundGas(uint256 startGas) internal {
    unchecked {
        uint256 balance = address(this).balance;
        if (balance == 0) {
            return;
        }
        uint256 gasPrice = min(tx.gasprice, block.basefee + MAX_REFUND_PRIORITY_FEE);
        uint256 gasUsed = startGas - gasleft() + REFUND_BASE_GAS;
        uint256 refundAmount = min(gasPrice * gasUsed, balance);
        (bool refundSent, ) = msg.sender.call{ value: refundAmount }('');
        emit RefundableVote(msg.sender, refundAmount, refundSent);
    }
}
```

### Impact

Front-running may discourage users from participating in voting due to the unpredictability of receiving a refund, potentially impacting the decentralization and fairness of the governance process.

### Tools Used

Manual Audit

### Recommended Mitigation Steps

One potential solution to mitigate front-running attacks is the use of flashbots, which keep users' voting transactions away from the mempool and help to ensure transaction ordering based on set criteria rather than gas prices.

## L-02 Unreachable 'initialize' Function in NounsDAOLogic Contracts should be removed

### Overview Details

The `initialize` function are not really used in some instances, say for the NounsDAOLogicV2 contract, this function cannot be invoked due to the timelock state variable being pre-initialized in the NounsDAOLogicV1 contract, which could be said to be intentional by design.

Cause in the practical implementation, Nouns V2 is deployed as an upgrade to V1 and the `initialize` function is never invoked. Instead, and new deployments should utilize NounsDAOProxyV2, which is capable of initializing LogicV2.

The NounsDAOLogicV1 contract pre-initializes the timelock state variable at the following Nouns DAO Executor address `0x0bc3807ec262cb779b38d65b38158acc3bfede10`. Therefore, the NounsDAOLogicV2 'initialize' function cannot be called post-upgrade.

The initialize function in the contract is programmed to only be called when the timelock state variable is empty, as shown in the [following code snippet](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L125-L186):

```solidity
function initialize(
    address timelock_,
    address nouns_,
    address vetoer_,
    uint256 votingPeriod_,
    uint256 votingDelay_,
    uint256 proposalThresholdBPS_,
    DynamicQuorumParams calldata dynamicQuorumParams_
) public virtual {
    require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
    require(msg.sender == admin, 'NounsDAO::initialize: admin only');
    require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
    ..SNIP..
    _setDynamicQuorumParams(
        dynamicQuorumParams_.minQuorumVotesBPS,
        dynamicQuorumParams_.maxQuorumVotesBPS,
        dynamicQuorumParams_.quorumCoefficient
    );
}
```

This design emphasizes the role of NounsDAOProxyV2 in initializing LogicV2 rather than allowing NounsDAOLogicV2 to initialize itself.

### Impact

Inaccessible functions are a flaw in project's design

### Tool Used

Manual Audit

### Recommended Mitigation Steps

If the function is not going to be used it should be removed

## L-03 The 'ProposalCreatedWithRequirement()' event in the `propose()` function should not be emitted using the current ` minQuorumVotes()`

### Overwiew Details

In the NounsDAO, the admin is permitted to change the minimum quorum votes at any block, if this happens in the same block where a proposal is created the evnt could be emiitted with the wrong minimum amount of quorum votes needed since when the proposal was passed a different value was used for the calculation.
Take a look at [propose()](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L181-L197) which calls the below:

```solidity
    /**
     * @notice Function used to propose a new proposal. Sender must have delegates above the proposal threshold
     * @param targets Target addresses for proposal calls
     * @param values Eth values for proposal calls
     * @param signatures Function signatures for proposal calls
     * @param calldatas Calldatas for proposal calls
     * @param description String description of the proposal
     * @return Proposal id of new proposal
     */
    function propose(
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description
    ) public returns (uint256) {
        ProposalTemp memory temp;

        temp.totalSupply = nouns.totalSupply();

        temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

        require(
            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
            'NounsDAO::propose: proposer votes below proposal threshold'
        );
        require(
            targets.length == values.length &&
                targets.length == signatures.length &&
                targets.length == calldatas.length,
            'NounsDAO::propose: proposal function information arity mismatch'
        );
        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');

        temp.latestProposalId = latestProposalIds[msg.sender];
        if (temp.latestProposalId != 0) {
            ProposalState proposersLatestProposalState = state(temp.latestProposalId);
            require(
                proposersLatestProposalState != ProposalState.Active,
                'NounsDAO::propose: one live proposal per proposer, found an already active proposal'
            );
            require(
                proposersLatestProposalState != ProposalState.Pending,
                'NounsDAO::propose: one live proposal per proposer, found an already pending proposal'
            );
        }

        temp.startBlock = block.number + votingDelay;
        temp.endBlock = temp.startBlock + votingPeriod;

        proposalCount++;
        Proposal storage newProposal = _proposals[proposalCount];
        newProposal.id = proposalCount;
        newProposal.proposer = msg.sender;
        newProposal.proposalThreshold = temp.proposalThreshold;
        newProposal.eta = 0;
        newProposal.targets = targets;
        newProposal.values = values;
        newProposal.signatures = signatures;
        newProposal.calldatas = calldatas;
        newProposal.startBlock = temp.startBlock;
        newProposal.endBlock = temp.endBlock;
        newProposal.forVotes = 0;
        newProposal.againstVotes = 0;
        newProposal.abstainVotes = 0;
        newProposal.canceled = false;
        newProposal.executed = false;
        newProposal.vetoed = false;
        newProposal.totalSupply = temp.totalSupply;
        newProposal.creationBlock = block.number;

        latestProposalIds[newProposal.proposer] = newProposal.id;

        /// @notice Maintains backwards compatibility with GovernorBravo events
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

        /// @notice Updated event with `proposalThreshold` and `minQuorumVotes`
        /// @notice `minQuorumVotes` is always zero since V2 introduces dynamic quorum with checkpoints
        //@audit  This event should instead be submittead with the quorum votes from atleast the previous block
        emit ProposalCreatedWithRequirements(
            newProposal.id,
            msg.sender,
            targets,
            values,
            signatures,
            calldatas,
            newProposal.startBlock,
            newProposal.endBlock,
            newProposal.proposalThreshold,
            minQuorumVotes(),
            description
        );

        return newProposal.id;
    }
```

### Impact

Nouners could easily act wrongly due to the data that's being passed in the emitted event, say 100 Nouners exist, the proposal was created at a time where the min quorum votes is 400, but this gets updated to a lower value say 300, now after the votes reach 300, Nouners who shoukld vote for this if they knew the quorum margin have being changed wouldn't vote cause they think there is no need and there is already enough votes in their favor

### Tool Used

Manual Audit

### Recommended Mitigation Steps

To address this, it is advisable to add `require` checks for the `newQuorumCoefficient` parameter in both the `_setQuorumCoefficient` and `_setDynamicQuorumParams` functions to guarantee its validity. However, it's worth deliberating whether such inclusion is essential given the inherent checks and balances provided by the DAO's voting process. If this feature is deemed necessary, it should be implemented in a way that does not result in unnecessary voting processes that could waste time and gas when a simple revert operation could have been sufficient.

## L-04 Lack of input checks when setting new _quorumCoefficien_ would easily cost an unnecessary proposal process

### Overwiew Details

In the NounsDAO, the admin is permitted to set the QuorumCoefficient without any verification on its value in both the `_setQuorumCoefficient` and `_setDynamicQuorumParams` functions. This can lead to a potential misconfiguration, which can considerably impact the voting process, as an incorrect QuorumCoefficient could alter the computation of QuorumVotes, thus affecting the acceptance or rejection of proposals. It's crucial to note that this vulnerability is mitigated to a significant degree by the protocol's internal safeguards. Any modifications to the QuorumCoefficient would need to go through a proposal and voting process. DAO members would likely vote against inappropriate values, preventing its implementation.

In the current design of `_setQuorumCoefficient` function, there are no checks on the `newQuorumCoefficient` parameter:

```solidity
function _setQuorumCoefficient(uint32 newQuorumCoefficient) external {
    require(msg.sender == admin, 'NounsDAO::_setQuorumCoefficient: admin only');
    DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);

    uint32 oldQuorumCoefficient = params.quorumCoefficient;
    params.quorumCoefficient = newQuorumCoefficient;

    _writeQuorumParamsCheckpoint(params);

    emit QuorumCoefficientSet(oldQuorumCoefficient, newQuorumCoefficient);
}
```

Similarly, the `_setDynamicQuorumParams` function contains checks for `newMinQuorumVotesBPS` and `newMaxQuorumVotesBPS` parameters, but none for `newQuorumCoefficient`.

### Impact

An unnecessary need for the change to go through a proposal when a simple check could have prevented an outrageously wrong value to be set for the quorum

### Tool Used

Manual Review

### Recommended Mitigation Steps

To address this, it is advisable to add `require` checks for the `newQuorumCoefficient` parameter in both the `_setQuorumCoefficient` and `_setDynamicQuorumParams` functions to guarantee its validity. However, it's worth deliberating whether such inclusion is essential given the inherent checks and balances provided by the DAO's voting process. If this feature is deemed necessary, it should be implemented in a way that does not result in unnecessary voting processes that could waste time and gas when a simple revert operation could have been sufficient.
