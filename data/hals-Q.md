# Summary

| ID              | Title                                                        | Severity     |
| --------------- | ------------------------------------------------------------ | ------------ |
| [L-01]  | Unequal/unfair opportunities between auction bidders         | Low          |
| [NC-01] | External/public function names should begin with a lowerCase | Non Critical |

# Low

## [L-01] Unequal/unfair opportunities between auction bidders

## Details

In `NounsAuctionHouseFork.sol`: the owner can set `reservePrice` & `minBidIncrementPercentage` bid parameters during an active auction

## Impact

This will result in unequal/unfair biddidng between bidders,as follows:

1. if `minBidIncrementPercentage` has ben increased during an active auction: this will result in bidders adding their bids following a higher percentages.
2. if `minBidIncrementPercentage` has ben decreased during an active auction: this will result in early bidderslosing the chance to win the auction.
3. if `reservePrice` has ben increased during an active auction: bidders are required to provide bids with minimum value larger than the bidders who was on the previous low `reservePrice`.

## Proof of Concept

```solidity
File: nouns-monorepo/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
Line 190:  function setReservePrice(uint256 _reservePrice) external override onlyOwner {
Line 200:  function setMinBidIncrementPercentage(uint8 _minBidIncrementPercentage) external override onlyOwner {
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

`reservePrice` & `minBidIncrementPercentage` must be updated when there's no active auction.

# Non Critical

## [NC-01] External/public function names should begin with a lowerCase

## Details

It was spotted in some contracts that some external/public functions starting with an underscore, which collides with solidity internal functions naming convention.

## Proof of Concept

Instances: 23

```solidity
File:nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
Line 162: function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external onlyAdmin(ds) {
Line 177: function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external onlyAdmin(ds) {
Line 193: function _setProposalThresholdBPS(NounsDAOStorageV3.StorageV3 storage ds, uint256 newProposalThresholdBPS)
Line 212: function _setObjectionPeriodDurationInBlocks
Line 229: function _setLastMinuteWindowInBlocks(NounsDAOStorageV3.StorageV3 storage ds, uint32 newLastMinuteWindowInBlocks)
Line 243: function _setProposalUpdatablePeriodInBlocks(
Line 261: function _setPendingAdmin(NounsDAOStorageV3.StorageV3 storage ds, address newPendingAdmin) external onlyAdmin(ds) {
Line 276: function _acceptAdmin(NounsDAOStorageV3.StorageV3 storage ds) external {
Line 301: function _setPendingVetoer(NounsDAOStorageV3.StorageV3 storage ds, address newPendingVetoer) public {
Line 314: function _acceptVetoer(NounsDAOStorageV3.StorageV3 storage ds) external {
Line 332:  function _burnVetoPower(NounsDAOStorageV3.StorageV3 storage ds) public {
Line 351: function _setMinQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMinQuorumVotesBPS)
Line 381: function _setMaxQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMaxQuorumVotesBPS)
Line 408: function _setQuorumCoefficient(NounsDAOStorageV3.StorageV3 storage ds, uint32 newQuorumCoefficient)
Line 432: function _setDynamicQuorumParams(
Line 468: function _withdraw(NounsDAOStorageV3.StorageV3 storage ds) external onlyAdmin(ds) returns (uint256, bool) {
Line 482: function _setVoteSnapshotBlockSwitchProposalId(NounsDAOStorageV3.StorageV3 storage ds) external only
Line 500: function _setForkDAODeployer(NounsDAOStorageV3.StorageV3 storage ds, address newForkDAODeployer)
Line 513: function _setErc20TokensToIncludeInFork(NounsDAOStorageV3.StorageV3 storage ds, address[] calldata erc20tokens)
Line 527: function _setForkEscrow(NounsDAOStorageV3.StorageV3 storage ds, address newForkEscrow) external onlyAdmin(ds) {
Line 533: function _setForkPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newForkPeriod) external onlyAdmin(ds) {
Line 551: function _setForkThresholdBPS(NounsDAOStorageV3.StorageV3 storage ds, uint256 newForkThresholdBPS)
Line 566: function _setTimelocksAndAdmin(
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

External/public functions should start with a lowerCase not with an underscore.
