## QA Report and Low issues list

| Number | Issue Title | Instances |
|--------|-------------|-----------|
| [L-01] | Checks for `block.number` should be removed or re-written to avoid contracts permanently failing | 3 |
| [NC-01] | Same function names are confusing | 1 |
| [NC-02] | Split `if` statements that perform multiple checks using `&&` | 6 |
| [NC-03] | Constant names should consist of all capital letters | 3 |
| [NC-04] | Inconsistent versions of solidity | 11 |


### [L-01] Checks for `block.number` should be removed or re-written to avoid contracts permanently failing

Here:

```solidity
File: NounsDAOLogicV2.sol
1023:     function _writeQuorumParamsCheckpoint(DynamicQuorumParams memory params) internal {
1024:         uint32 blockNumber = safe32(block.number, 'block number exceeds 32 bits');
...
265:     function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {
266:         require(n < 2**32, errorMessage);
267:         return uint32(n);
268:     }
```

`block.number` is checked to be lower than 32bits. However, the current block number is very far from approaching this limit, so it can be removed.

But if `block.number` were to actually to be over this, then any calls to `_writeQuorumParamsCheckpoint` would always fail and revert, permanently (because the block number obviously doesn't decrease), potentially bricking the entire contract and any contracts using the function. This also applies for `_writeCheckpoint`. 

However, it is of low severity as right now the block number is far from the 32 bit limit, but in the future if it does go over, the contracts will break.

Consider removing the check, or assigning the value to `uint64` or larger values, then removing the check, to avoid this issue.

*There are 3 instances of this issue:*
```solidity
File: NounsDAOLogicV2.sol
1023:     function _writeQuorumParamsCheckpoint(DynamicQuorumParams memory params) internal {
1024:         uint32 blockNumber = safe32(block.number, 'block number exceeds 32 bits');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1023-L1024

```solidity
File: NounsDAOV3Admin.sol
579:     function _writeQuorumParamsCheckpoint(
580:         NounsDAOStorageV3.StorageV3 storage ds,
581:         NounsDAOStorageV3.DynamicQuorumParams memory params
582:     ) internal {
583:         uint32 blockNumber = safe32(block.number, 'block number exceeds 32 bits');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L579-L583

```solidity
File: ERC721CheckpointableUpgradeable.sol
244:     function _writeCheckpoint(
245:         address delegatee,
246:         uint32 nCheckpoints,
247:         uint96 oldVotes,
248:         uint96 newVotes
249:     ) internal {
250:         uint32 blockNumber = safe32(
251:             block.number,
252:             'ERC721Checkpointable::_writeCheckpoint: block number exceeds 32 bits'
253:         );
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L244-L253

### [NC-01] Same function names are confusing

The same function name `quorumParamsCheckpoints` is used for 2 different functions. Although this is valid as their function signatures are different, it may be confusing. Therefore it is recommended to use different function names, such as `quorumParamsCheckpointsAll()` or something similar to distinquist between the 2 functions.

*There is 1 instance of this issue:*

```solidity
File: NounsDAOLogicV3.sol
931:     /**
932:      * @notice Get all quorum params checkpoints
933:      */
934:     function quorumParamsCheckpoints() public view returns (DynamicQuorumParamsCheckpoint[] memory) { //@audit same name
935:         return ds.quorumParamsCheckpoints;
936:     }
937: 
938:     /**
939:      * @notice Get a quorum params checkpoint by its index
940:      */
941:     function quorumParamsCheckpoints(uint256 index) public view returns (DynamicQuorumParamsCheckpoint memory) { //@audit same name
942:         return ds.quorumParamsCheckpoints[index];
943:     }
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L931-L943

### [NC-02] Split `if` statements that perform multiple checks using `&&`

Splitting `if` statements that combine multiple checks using `&&` improves readibility. The statements should be nested such as:

```solidity
if (A){
    if (B){
        _;
    }
}
```

instead of

```solidity
if (A && B){
    _;
}
```

*There are 6 instances of this issue:*

```solidity
File: NounsDAOV3Votes.sol
  240         if (
  241             // only for votes can trigger an objection period
  242             // we're in the last minute window
  243             isForVoteInLastMinuteWindow &&
  244             // first part of the vote flip check
  245             // separated from the second part to optimize gas
  246             isDefeatedBefore &&
  247             // haven't turn on objection yet
  248             proposal.objectionPeriodEndBlock == 0 &&
  249             // second part of the vote flip check
  250             !ds.isDefeated(proposal)
  251         ) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L240-L241
```solidity
File: NounsDAOLogicV1Fork.sol
  381         if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1Fork.sol#L381
```solidity
File: ERC721CheckpointableUpgradeable.sol
  227         if (srcRep != dstRep && amount > 0) {
  255         if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L227
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L255

```solidity
File: NounsDAOLogicV2.sol
  1026         if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1026

```solidity
File: NounsDAOV3Admin.sol
  585         if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L585

### [NC-03] Constant names should consist of all capital letters

For consistency, `constant` variable names should consist of all capital letters. The only exception to this can be `name`. However there are some cases listed below where this is not the case:

*There are 3 instances of this issue:*
```solidity
File: NounsDAOLogicV2.sol
91:     /// @notice The maximum number of actions that can be included in a proposal
92:     uint256 public constant proposalMaxOperations = 10; // 10 actions
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L91-L92

```solidity
File: NounsDAOLogicV1Fork.sol
142:     /// @notice The maximum number of actions that can be included in a proposal
143:     uint256 public constant proposalMaxOperations = 10; // 10 actions
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1Fork.sol#L142-L143

```solidity
File: ERC721CheckpointableUpgradeable.sol
51:     /// @notice Defines decimals as per ERC-20 convention to make integrations with 3rd party governance platforms easier
52:     uint8 public constant decimals = 0;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L51-L52


### [NC-04] Inconsistent versions of solidity

The same version of solidity should be used across all contracts. As `^0.8.19` is the latest used and it is started in the info that `^0.8.20` will not be used, all contracts should use `^0.8.19` for consistency.

*There are 11 instances of this issue:*

```solidity
File: ERC20Transferer.sol
  16 pragma solidity ^0.8.16;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L16

```solidity
File: DeployDAOV3DataContractsBase.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3DataContractsBase.s.sol#L2

```solidity
File: DeployDAOV3DataContractsGoerli.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3DataContractsGoerli.s.sol#L2

```solidity
File: DeployDAOV3DataContractsSepolia.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3DataContractsSepolia.s.sol#L2

```solidity
File: DeployDAOV3NewContractsBase.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L2

```solidity
File: DeployDAOV3NewContractsMainnet.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsMainnet.s.sol#L2

```solidity
File: DeployDAOV3NewContractsTestnet.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsTestnet.s.sol#L2

```solidity
File: ProposeDAOV3UpgradeMainnet.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L2

```solidity
File: ProposeDAOV3UpgradeTestnet.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L2

```solidity
File: ProposeENSReverseLookupConfigMainnet.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeENSReverseLookupConfigMainnet.s.sol#L2

```solidity
File: ProposeTimelockMigrationCleanupMainnet.s.sol
  2 pragma solidity ^0.8.15;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeTimelockMigrationCleanupMainnet.s.sol#L2