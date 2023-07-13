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
