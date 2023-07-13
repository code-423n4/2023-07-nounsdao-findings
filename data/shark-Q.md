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
## Wrong adoption of block time
The following voting period constants are assuming 9.6 instead of 12 seconds per block. Depending on the sensitivity of lower and upper ranges desired, these may limit or shift the intended settable voting periods. For instance, using the supposed 12 second per block convention, the minimum and maximum settable voting periods should respectively be 7_200 and 100_800. 

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L67-L71

```solidity
    /// @notice The minimum setable voting period
    uint256 public constant MIN_VOTING_PERIOD = 5_760; // About 24 hours

    /// @notice The max setable voting period
    uint256 public constant MAX_VOTING_PERIOD = 80_640; // About 2 weeks
```
## function initialize can be initialized more than once
There is no `initializer` visibility in the following two functions that separately have a require string message denoted as saying `NounsDAO::initialize: can only initialize once`.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L135-L144

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
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L165-L176

```solidity
    function initialize(
        address timelock_,
        address nouns_,
        uint256 votingPeriod_,
        uint256 votingDelay_,
        uint256 proposalThresholdBPS_,
        uint256 quorumVotesBPS_,
        address[] memory erc20TokensToIncludeInQuit_,
        uint256 delayedGovernanceExpirationTimestamp_
    ) public virtual {
        __ReentrancyGuard_init_unchained();
        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
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
