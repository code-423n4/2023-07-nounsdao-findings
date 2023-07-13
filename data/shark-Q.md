## Reserve price not fully taken care of
It is possible for an active auction to close at a price lower than the newly increased reserve price. This is undesirable especially preventing a Noun auctioned off at the lower than expected price could be out of control in a bear market. Consider adding a check alleging that the contract balance needing to exceed the reserve price. Else, the last bidder will be refunded prior to having the Noun burned. Here's a refactored code logic that will take care of the suggestion.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L236-L256

```solidity
/**
 * @notice Settle an auction, finalizing the bid and paying out to the owner.
 * @dev If there are no bids, the Noun is burned.
 */
function _settleAuction() internal {
    INounsAuctionHouse.Auction memory _auction = auction;

    require(_auction.startTime != 0, "Auction hasn't begun");
    require(!_auction.settled, 'Auction has already been settled');
    require(block.timestamp >= _auction.endTime, "Auction hasn't completed");

    auction.settled = true;

    // Check if contract balance is greater than reserve price
    if (address(this).balance < reservePrice) {
        // If contract balance is less than reserve price, refund to the last bidder
        if (_auction.bidder != address(0)) {
            _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
        }

        // And then burn the Noun
        nouns.burn(_auction.nounId);
    } else {
        if (_auction.bidder == address(0)) {
            nouns.burn(_auction.nounId);
        } else {
            nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);
        }

        if (_auction.amount > 0) {
            _safeTransferETHWithFallback(owner(), _auction.amount);
        }
    }

    emit AuctionSettled(_auction.nounId, _auction.bidder, _auction.amount);
}
```
## Spec document contradicts function logic
The spec is noted as saying,

https://hackmd.io/@el4d/nouns-dao-v3-spec
 
```
In this new version a proposal can also be cancelled if:

1. One of the accounts that signed on the proposal chooses to cancel, or
2. The sum of votes of all proposers no longer exceeds threshold.
```
However, it is implemented otherwise in the require statement of `NounsDAOV3Proposals.cancel()` below where signer can only cancel the proposal when the sum of votes of all proposers no longer exceeds threshold. Consider updating the document to correctly reflect what has been implemented.  

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L595-L598

```solidity
        require(
            msgSenderIsProposer || votes <= proposal.proposalThreshold,
            'NounsDAO::cancel: proposer above threshold'
        );
```
## Code and comment mismatch
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L86

```solidity
    @ audit 4,000 should be changed to 6,000
    uint256 public constant MAX_QUORUM_VOTES_BPS_UPPER_BOUND = 6_000; // 4,000 basis points or 60%
```
## Spelling errors
There are numerous instances throughout the codebase in different contracts. Here's just one of the specific instances:  

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L61

```solidity
    @ audit setable should be changed to settable
    /// @notice The minimum setable proposal threshold
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L218

```solidity
            @ audit arity should be changed to parity
            'NounsDAO::propose: proposal function information arity mismatch'
```
There are numerous instances throughout the codebase in different contracts. Here's just one of the specific instances:  

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L901

```solidity
     @ audit priviledges should be changed to privileges
     * @notice Burns veto priviledges
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L19

```solidity
@ audit NounsDAOLogicV2.sol should be changed to NounsDAOLogicV3.sol
// NounsDAOLogicV2.sol is a modified version of Compound Lab's GovernorBravoDelegate.sol:
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L217

```solidity
    @ audit the during of should be omitted
    /// @notice Emitted when the during of the forking period is set
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L80-L84

```solidity
    @ audit thats should be changed to that's
    /// @notice An event thats emitted when an account changes its delegate
    event DelegateChanged(address indexed delegator, address indexed fromDelegate, address indexed toDelegate);

    @ audit thats should be changed to that's
    /// @notice An event thats emitted when a delegate account's vote balance changes
    event DelegateVotesChanged(address indexed delegate, uint256 previousBalance, uint256 newBalance);
```
## Wrong adoption of block time
The following voting period constants are assuming 9.6 instead of 12 seconds per block. Depending on the sensitivity of lower and upper ranges desired, these may limit or shift the intended settable voting periods. For instance, using the supposed 12 second per block convention, the minimum and maximum settable voting periods should respectively be 7_200 and 100_800. 

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L67-L71

```solidity
    /// @notice The minimum setable voting period
    uint256 public constant MIN_VOTING_PERIOD = 5_760; // About 24 hours

    /// @notice The max setable voting period
    uint256 public constant MAX_VOTING_PERIOD = 80_640; // About 2 weeks
```
## MIN_PROPOSAL_THRESHOLD_BPS is too low a value
In NounsDAOLogicV2.sol, NounsDAOV3Admin.sol, and NounsDAOLogicV1Fork.sol, the minimum proposal threshold can be set as low as 1 basis point.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L61-L62
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L111-L112
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L118-L119

```solidity
    /// @notice The minimum setable proposal threshold
    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%
```
This could pose a precision issue even if the total supply of Nouns tokens is already in its three digits. Apparently, a proposal threshold determined via the following two functions could return zero, e.g. (1 * 720) / 10000 yields zero due to truncation, i.e. numerator smaller than denominator.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L921-L923

```solidity
    function proposalThreshold() public view returns (uint256) {
        return bps2Uint(proposalThresholdBPS, nouns.totalSupply());
    }
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1066-L1068

```solidity
    function bps2Uint(uint256 bps, uint256 number) internal pure returns (uint256) {
        return (number * bps) / 10000;
    }
```
## V3 upgrade could depreciate token value due to oversupply
The forking feature, though rarely happened as documented, could potentially affect the Nouns token value. This is because:
- a new fork will have its own DAO tokens daily generated and auctioned off through a new Auction House
- new Nouns tokens claimed through escrow and during forking period will not have the original Nouns tokens burned. They are transferred to the original treasury or elsewhere as the DAO deems fit.

With a new fork competing with the original DAO for the daily auction, it will likely diverge the amount of ETH intended to go into bidding the daily new NFTs from both ends. The situation could be worse in the far future if more forks were to transpire.

## Nouns fork may not efficiently remedy a bad situation
The 20% threshold, for instance, 800 * 0.2 = 160 of Nouns tokens, is not a small number comparatively in terms of the market cap. This translates to 160 * 30 ETH * $2,000 almost equivalent to 10 million worth of USDC. For members wishing to dodge bad/undesirable proposals that aren't going to be vetoed, it's likely this will not materialize where the proposals get executed long before the threshold could be met to initiate a fork.

Consider conditionally reducing the threshold given that ragequit is going to happen regardless of the size of the fork. It is forking group that could share the same goal and direction in a new DAO that matters.        