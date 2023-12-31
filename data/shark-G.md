## Redundant code blocks
The first and third if blocks of the function logic below may be grouped together with `||` in their conditional checks. 

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L981-L1005

```solidity
    function getDynamicQuorumParamsAt(uint256 blockNumber_) public view returns (DynamicQuorumParams memory) {
        uint32 blockNumber = safe32(blockNumber_, 'NounsDAO::getDynamicQuorumParamsAt: block number exceeds 32 bits');
        uint256 len = quorumParamsCheckpoints.length;

        if (len == 0) {
            return
                DynamicQuorumParams({
                    minQuorumVotesBPS: safe16(quorumVotesBPS),
                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
                    quorumCoefficient: 0
                });
        }

        if (quorumParamsCheckpoints[len - 1].fromBlock <= blockNumber) {
            return quorumParamsCheckpoints[len - 1].params;
        }

        if (quorumParamsCheckpoints[0].fromBlock > blockNumber) {
            return
                DynamicQuorumParams({
                    minQuorumVotesBPS: safe16(quorumVotesBPS),
                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
                    quorumCoefficient: 0
                });
        }
```
## Unnecessary if block
The functions below can only be called by the vetoer. The first if block is unneeded since a zero address is incapable of calling a function.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L385-L396

```solidity
    /**
     * @notice Vetoes a proposal only if sender is the vetoer and the proposal has not been executed.
     * @param proposalId The id of the proposal to veto
     */
    function veto(uint256 proposalId) external {
        if (vetoer == address(0)) {
            revert VetoerBurned();
        }

        if (msg.sender != vetoer) {
            revert VetoerOnly();
        }
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L536-L543

```solidity
    function veto(NounsDAOStorageV3.StorageV3 storage ds, uint256 proposalId) external {
        if (ds.vetoer == address(0)) {
            revert VetoerBurned();
        }

        if (msg.sender != ds.vetoer) {
            revert VetoerOnly();
        }
```
## Contradictory unused constant
The constant `MAX_QUORUM_VOTES_BPS` has been declared in NounsDAOLogicV2.sol but never in use. But even if it were to be used, it would contradict the upper bound of maximum quorum votes basis points, `MAX_QUORUM_VOTES_BPS_UPPER_BOUND`.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L85-L89

```solidity
    /// @notice The upper bound of maximum quorum votes basis points
    uint256 public constant MAX_QUORUM_VOTES_BPS_UPPER_BOUND = 6_000; // 4,000 basis points or 60%

    /// @notice The maximum setable quorum votes basis points
    uint256 public constant MAX_QUORUM_VOTES_BPS = 2_000; // 2,000 basis points or 20%
```
## Irrational if block
The if block in the function below returns `proposal.startBlock - votingDelay` if `proposal.creationBlock == 0`.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L925-L930

```solidity
    function proposalCreationBlock(Proposal storage proposal) internal view returns (uint256) {
        if (proposal.creationBlock == 0) {
            return proposal.startBlock - votingDelay;
        }
        return proposal.creationBlock;
    }
```
This adoption is irrational and unneeded considering `proposal.creationBlock` is always assigned `block.timestamp` whenever a new proposal is proposed.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L258

```solidity
        newProposal.creationBlock = block.number;
```
## Unnecessary for loops
When cancelling or vetoing a proposal, the following for loop is unneeded.  

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L372-L380
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L405-L413
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L553-L561
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L602-L610

```solidity
        for (uint256 i = 0; i < proposal.targets.length; i++) {
            timelock.cancelTransaction(
                proposal.targets[i],
                proposal.values[i],
                proposal.signatures[i],
                proposal.calldatas[i],
                proposal.eta
            );
        }
```
This is because it is only iteratively making `queuedTransactions[txHash]` false.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L150-L163
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L127-L140

```solidity
    function cancelTransaction(
        address target,
        uint256 value,
        string memory signature,
        bytes memory data,
        uint256 eta
    ) public {
        require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');

        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
        queuedTransactions[txHash] = false;

        emit CancelTransaction(txHash, target, value, signature, data, eta);
    }
```
This double measure is indeed unnecessary because the proposal state so updated to `Cancelled` or `Vetoed` will deny all future [queue](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L300)/[execute](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L338) processes after all. 

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L371
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L600

```solidity
        proposal.canceled = true;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L404
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L551

```solidity
        proposal.vetoed = true;
```
## Boolean expressions to boolean literals comparison
Comparing a boolean value to a boolean literal incurs the `ISZERO` opcode operation, costing more gas than using a boolean expression. Avoid this adoption if possible.

There are numerous instances throughout the codebase in different contracts. Here's just one of the specific instances:  

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L219

```solidity
        require(receipt.hasVoted == false, 'NounsDAO::castVoteDuringVotingPeriodInternal: voter already voted');
```
## Direct utilization of global variables
The following function is unneeded since `block.timestamp` is a global variable that can readily be used directly.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L204-L207

```solidity
    function getBlockTimestamp() internal view returns (uint256) {
        // solium-disable-next-line security/no-block-members
        return block.timestamp;
    }
```
## Unnecessary zero address check
`msg.sender != address(0)` in the require statement below is not needed since a zero address will not be able to act as `msg.sender` to make a function call.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L276-L281

```solidity
    function _acceptAdmin(NounsDAOStorageV3.StorageV3 storage ds) external {
        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
        require(
            msg.sender == ds.pendingAdmin && msg.sender != address(0),
            'NounsDAO::_acceptAdmin: pending admin only'
        );
```
## Redundant boolean assignment
Self assigning of `msgSenderIsProposer` is implemented in the for loop below. Consider replacing `msgSenderIsProposer = msgSenderIsProposer` with `msgSenderIsProposer`. 

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L587-L593

```solidity
        uint256 votes = nouns.getPriorVotes(proposer, block.number - 1);
        bool msgSenderIsProposer = proposer == msg.sender;
        address[] memory signers = proposal.signers;
        for (uint256 i = 0; i < signers.length; ++i) {
@ audit            msgSenderIsProposer = msgSenderIsProposer || msg.sender == signers[i];
            votes += nouns.getPriorVotes(signers[i], block.number - 1);
        }
```
## Unnecessary input parameter and first require check
The input parameter as well as the first require statement in `NounsAuctionHouseFork.sol.createBid()` below are unneeded. 

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L115-L124

```solidity
    /**
     * @notice Create a bid for a Noun, with a given amount.
     * @dev This contract only accepts payment in ETH.
     */
    function createBid(uint256 nounId) external payable override nonReentrant {
        INounsAuctionHouse.Auction memory _auction = auction;

        require(_auction.nounId == nounId, 'Noun not up for auction');
        require(block.timestamp < _auction.endTime, 'Auction expired');
        require(msg.value >= reservePrice, 'Must send at least reservePrice');
```
This is because `_auction.nounId` has already been defaulted to the returned value of `nounId` when `_createAuction()` attempted to mint the next Nouns token.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L212-L230

```solidity
    function _createAuction() internal {
        try nouns.mint() returns (uint256 nounId) {
            uint256 startTime = block.timestamp;
            uint256 endTime = startTime + duration;

            auction = Auction({
                nounId: nounId,
                amount: 0,
                startTime: startTime,
                endTime: endTime,
                bidder: payable(0),
                settled: false
            });

            emit AuctionCreated(nounId, startTime, endTime);
        } catch Error(string memory) {
            _pause();
        }
    }
```