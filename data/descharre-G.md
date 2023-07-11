# Summary
|ID     | Finding|  Gas saved | Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01      |Emit statements can be done more efficiently|  |  - |
|G-02      |Unnecessary for loop if msg.sender is proposer|  |  - |
# Details
## [G-01] Emit statements can be done more efficiently
A lot of the admin functions emit the old and the new values. This can be done more efficiëntly.

[NounsDAOV3Admin.sol#L162-L171](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L162-L171)
```solidity
    uint256 oldVotingDelay = ds.votingDelay;
    ds.votingDelay = newVotingDelay;

    emit VotingDelaySet(oldVotingDelay, newVotingDelay);
```
Can be written as
```solidity
    emit VotingDelaySet(ds.votingDelay, newVotingDelay);
    ds.votingDelay = newVotingDelay;
```
## [G-02] Unnecessary for loop if msg.sender is proposer
When msg.sender is a proposer it does an unnecessary for loop for all the signers. Since msgSenderIsProposer is true it will always stay true. This is a waste of gas and can easily be mitigated by adding another check.

[NounsDAOV3Proposals.sol#L588-L598](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L588-L598)
```diff
    bool msgSenderIsProposer = proposer == msg.sender;
    address[] memory signers = proposal.signers;
+   if (!msgSenderIsProposer){
    for (uint256 i = 0; i < signers.length; ++i) {
        msgSenderIsProposer = msgSenderIsProposer || msg.sender == signers[i];
        votes += nouns.getPriorVotes(signers[i], block.number - 1);
    }
+   }
    require(
        msgSenderIsProposer || votes <= proposal.proposalThreshold,
        'NounsDAO::cancel: proposer above threshold'
    );
```