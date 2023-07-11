# Summary

| ID            | Title                                                   | Severity         |
| ------------- | ------------------------------------------------------- | ---------------- |
| [G-01](#g-01) | Use != 0 instead of > 0 for unsigned integer comparison | Gas Optimization |
| [G-02](#g-02) | Redundant zero address check                            | Gas Optimization |
| [G-03](#g-03) | Reducing search range by excluding the edges            | Gas Optimization |
| [G-04](#g-04) | Add equality check to prevent entering the while loop   | Gas Optimization |

## [G-01] Use != 0 instead of > 0 for unsigned integer comparison 

## Details

Using != 0 instead of > 0 for unsigned integer comparison saves more gas per instance.

## Proof of Concept

Instances: 15

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
Line 165: return nCheckpoints > 0 ? checkpoints[account][nCheckpoints - 1].votes : 0;
Line 227: if (srcRep != dstRep && amount > 0) {
Line 230: uint96 srcRepOld = srcRepNum > 0 ? checkpoints[srcRep][srcRepNum - 1].votes : 0;
Line 237: uint96 dstRepOld = dstRepNum > 0 ? checkpoints[dstRep][dstRepNum - 1].votes : 0;
Line 255: if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
Line 564: if (votes > 0) {
Line 1026: if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
Line 484: if (oldVoteSnapshotBlockSwitchProposalId > 0) {
Line 585: if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol
Line 888: if (proposal.signers.length > 0) revert ProposerCannotUpdateProposalWithSigners();
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
Line 132: if (votes > 0) {
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
Line 255: if (tokensToSend > 0) {
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
Line 251: if (_auction.amount > 0) {
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/fork/newdao/governance
/NounsDAOLogicV1Fork.sol
Line 239: if (balancesToSend[i] > 0) {
Line 381: if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
```

## [G-02] Redundant zero address check 

## Details

Redundant check for msg.sender != address(0),since address(0) will never be `msg.sender`.

## Proof of Concept

Instances: 1

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
Line 733: require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');
```

## [G-03] Reducing search range by excluding the edges 

## Details

- In `NounsDAOLogicV2.sol`/`getDynamicQuorumParamsAt` function: while loop iterations can be reduced by excluding the `quorumParamsCheckpoints` array edges :`quorumParamsCheckpoints[0]` & `quorumParamsCheckpoints[len-1]` since they have been checked before.
- This can be achieved by updating the initial `lower` & `upper`searching indexes into:  
  `lower=1` & `upper=len-2`.
- Same issue in `NounsDAOV3DynamicQuorum.sol`/`getDynamicQuorumParamsAt` as well.

## Proof of Concept

Instances: 2

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
Line 1007: uint256 lower = 0;
Line 1008: uint256 upper = len - 1;
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol
Line 107: uint256 lower = 0;
Line 108: uint256 upper = len - 1;
```

## [G-04] Add equality check to prevent entering the while loop 

## Details

- In `NounsDAOLogicV2.sol `/`getDynamicQuorumParamsAt`function: while loop could be avoided to check for`blockNumber`when`blockNumber==quorumParamsCheckpoints[0].fromBlock`.

- This can be achieved by updating the if statement to check for equality as well:  
  `if (quorumParamsCheckpoints[0].fromBlock > blockNumber)`.

- Same issue in `NounsDAOV3DynamicQuorum.sol`/`getDynamicQuorumParamsAt` as well.

## Proof of Concept

Instances: 2

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
Line 998: if (quorumParamsCheckpoints[0].fromBlock > blockNumber) {
```

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol
Line 98:  if (ds.quorumParamsCheckpoints[0].fromBlock > blockNumber) {
```
