

## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |  Avoid contract existence checks by using low level calls   | 22 | - |
| [G-02] |  Use selfbalance() instead of address(this).balance   | 4 | - |
| [G-03] |  bytes constants are more eficient than string constans   | 5 | - |
| [G-04] |  State variables should be cached in stack variables rather than re-reading them from storage   | 3 | - |
| [G-05] |  State Variable can be packed into fewer storage slots   | 1 | - |
| [G-06] |  Not using the named return variables when a function returns, wastes deployment gas   | 1 | - |
| [G-07] |  Use constants instead of type(uintx).max   | 5 | - |
| [G-08] |  use Mappings Instead of Arrays   | 1 | - |
| [G-09] |  Use hardcode address instead address(this)   | 9 | - |
| [G-10] |  Use != 0 instead of > 0 for unsigned integer comparison  | 12 | - |
| [G-11] |  With assembly, .call (bool success) transfer can be done gas-optimized   | 7 | - |
| [G-12] |  Do not calculate constants   | 5 | - |
| [G-13] |  internal functions not called by the contract should be removed to save deployment gas    | 4 | - |
| [G-14] |  Amounts should be checked for 0 before calling a transfer   | 10 | - |
| [G-15] |  >= costs less gas than >   | 17 | - |
| [G-16] |  Expressions for constant values such as a call to keccak256(), should use immutable rather than constant   | 11 | - |
| [G-17] |  Sort Solidity operations using short-circuit mode   | 1 | - |
| [G-18] |  Can Make The Variable Outside The Loop To Save Gas    | 5 | - |
| [G-19] |  Use calldata instead of memory for function arguments that do not get mutated   | 10 | - |
| [G-20] |  Avoid emitting storage values   | 8 | - |
| [G-21] |  Splitting if() statements that use && saves gas - (saves 8 gas per &&)   | 4 | - |
| [G-22] |  State variables only set in the constructor should be declared immutable  | 18 | - |
| [G-23] |  Using fixed bytes is cheaper than using string  | 3 | - |
| [G-24] |  Pre-increments and pre-decrements are cheaper than post-increments and post-decrements  | 10 | - |
| [G-25] |  Not using the named return variable when a function returns, wastes deployment gas   | 1 | - |
| [G-26] |  Bytes constants are more efficient than String constants  | 5 | - |
| [G-27] |  Avoid Storage keyword during modifiers  | 1 | - |
| [G-28] |  Don't initialize variables with default value  | 9 | - |
| [G-29] |  Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)  | 4 | - |
| [G-30] |  Use assembly for math (add, sub, mul, div)  | 8 | - |
| [G-31] |  Non efficient zero initialization  | 30 | - |




## [G‑01] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.
```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
246   nouns.burn(_auction.nounId);

248   nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);

263   IWETH(weth).deposit{ value: amount }();

264   IERC20(weth).transfer(to, amount);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/#L246


```solidity
File: packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol
104    NounsTokenFork(token).initialize(
        
114      NounsAuctionHouseFork(auction).initialize(
         
126     NounsDAOExecutorV2(payable(treasury)).initialize(governor, originalTimelock.delay());


141     NounsDAOLogicV1Fork(governor).initialize(
          
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol#L104


```solidity
File: packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol
120   nounsToken.transferFrom(address(this), owner, tokenIds[i]);

151   nounsToken.transferFrom(address(this), to, tokenIds[i]);

165   return nounsToken.balanceOf(address(this)) - numTokensInEscrow;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
228   IERC20(erc20Token).safeTransfer(recipient, tokensToSend);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L228

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
206    temp.totalSupply = nouns.totalSupply();

211     nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,

367      nouns.getPriorVotes(proposal.proposer, block.number - 1) <= proposal.proposalThreshold,

623     uint96 votes = nouns.getPriorVotes(voter, proposalCreationBlock(proposal));

922    return bps2Uint(proposalThresholdBPS, nouns.totalSupply());

1056    return bps2Uint(getDynamicQuorumParamsAt(block.number).minQuorumVotesBPS, nouns.totalSupply());

1063    return bps2Uint(getDynamicQuorumParamsAt(block.number).maxQuorumVotesBPS, nouns.totalSupply());
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L228


```solidity
File: packages/nouns-contracts/contracts/utils/ERC20Transferer.sol
33    uint256 balance = IERC20(token).balanceOf(msg.sender);

34    IERC20(token).transferFrom(msg.sender, to, balance);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L33


```solidity
File: packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol
100  timelockV1.delay()
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L100

## [G-02] Use selfbalance() instead of address(this).balance
it's recommended to use the selfbalance() function instead of address(this).balance. The selfbalance() function is a built-in Solidity function that returns the balance of the current contract in Wei and is considered more gas-efficient and secure.

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
823        uint256 amount = address(this).balance;

1035       uint256 balance = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L823

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
469        uint256 amount = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L469

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
297            uint256 balance = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L297

## [G-03] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
59      string public constant name = 'Nouns DAO';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L59


```solidity
File:  packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
44   string public constant name = 'Nouns DAO';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L44


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
116   string public constant name = 'Nouns DAO';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L116


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
82     string public constant NAME = 'NounsDAOExecutorV2';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L82


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
46   string public constant NAME = 'NounsTokenFork';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L46

## [G‑04] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```solidity
File: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
137        auction.amount = msg.value;

138        auction.bidder = payable(msg.sender);

143        auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L137


## [G-05] State Variable can be packed into fewer storage slots


```solidity
File: contracts/governance/fork/newdao/token/NounsTokenFork.sol

  string public constant NAME = 'NounsTokenFork';

    /// @notice  An address who has permissions to mint Nouns
    address public minter;

    /// @notice The Nouns token URI descriptor
    INounsDescriptorMinimal public descriptor;

    /// @notice The Nouns token seeder
    INounsSeeder public seeder;

    /// @notice The escrow contract used to verify ownership of the original Nouns in the post-fork claiming process
    INounsDAOForkEscrow public escrow;

    /// @notice The fork ID, used when querying the escrow for token ownership
    uint32 public forkId;

    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
    uint256 public remainingTokensToClaim;

    /// @notice The forking period expiration timestamp, after which new tokens cannot be claimed by the original DAO
    uint256 public forkingPeriodEndTimestamp;

    /// @notice Whether the minter can be updated
    bool public isMinterLocked;

    /// @notice Whether the descriptor can be updated
    bool public isDescriptorLocked;

    /// @notice Whether the seeder can be updated
    bool public isSeederLocked;

    /// @notice The noun seeds
    mapping(uint256 => INounsSeeder.Seed) public seeds;

    /// @notice The internal noun ID tracker
    uint256 private _currentNounId;

    /// @notice IPFS content hash of contract-level metadata
    string private _contractURIHash = 'QmZi1n79FqWt2tTLwCqiy6nLM6xLGRsEPQ5JmReJQKNNzX';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L46-L85


## [G-06] Not using the named return variables when a function returns, wastes deployment gas

When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost

```solidity
File: script/DeployDAOV3NewContractsBase.s.sol
105        return timelockV2;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L105

## [G-07] Use constants instead of type(uintx).max


```solidity
File: contracts/governance/NounsDAOV3Admin.sol
595        require(n <= type(uint32).max, errorMessage);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L595

```solidity
File: contracts/governance/NounsDAOV3DynamicQuorum.sol
146        require(n <= type(uint32).max, errorMessage);

151        if (n > type(uint16).max) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L146

```solidity
File: contracts/governance/NounsDAOLogicV2.sol
1079        require(n <= type(uint32).max, errorMessage);

1084        if (n > type(uint16).max) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1079


## [G-08] use Mappings Instead of Arrays


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol
56   address[] public erc20TokensToIncludeInQuit;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol#L56

## [G-09] Use hardcode address instead address(this)


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
104     require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');

122     msg.sender == address(this),

245     msg.sender == address(this),
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L104




```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
595   abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), getChainIdInternal(), address(this))

823   uint256 amount = address(this).balance;

1035  uint256 balance = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L595


```solidity
File: packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol
120   nounsToken.transferFrom(address(this), owner, tokenIds[i]);

151    nounsToken.transferFrom(address(this), to, tokenIds[i]);

165    return nounsToken.balanceOf(address(this)) - numTokensInEscrow;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120


## [G-10] Use != 0 instead of > 0 for unsigned integer comparison


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
564   if (votes > 0) {

1026  if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L564


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
484   if (oldVoteSnapshotBlockSwitchProposalId > 0) {

585   if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L484


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol
888  if (proposal.signers.length > 0) revert ProposerCannotUpdateProposalWithSigners();
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L888


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
132   if (votes > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L132


```solidity
File: packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
255   if (tokensToSend > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L255

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
251    if (_auction.amount > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L151

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
239   if (balancesToSend[i] > 0) {

281   if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L239


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
227    if (srcRep != dstRep && amount > 0) {

255    if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L227

## [G-11] With assembly, .call (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
-   (bool success,) = dest.call{value:amount}("");
 bool success; 
 assembly {  
            success := call(gas(), dest, amount, 0, 0)
     }  



```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
173   (bool success, bytes memory returnData) = target.call{ value: value }(callData);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L173


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
196    (bool success, bytes memory returnData) = target.call{ value: value }(callData);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L196


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
824    (bool sent, ) = msg.sender.call{ value: amount }('');

1043    (bool refundSent, ) = tx.origin.call{ value: refundAmount }('');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L824


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
470   (bool sent, ) = msg.sender.call{ value: amount }('');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L470


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
305   (bool refundSent, ) = msg.sender.call{ value: refundAmount }('');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L305

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
373   (bool success, ) = to.call{ value: value, gas: 30_000 }(new bytes(0));
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L373

## [G-12] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
118   uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;

121    uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;

127   uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;

145   uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;

148   uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L118

## [G-13] internal functions not called by the contract should be removed to save deployment gas


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
243   function _authorizeUpgrade(address) internal view override {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L243


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
281    function _authorizeUpgrade(address) internal view override onlyOwner {}
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L281


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
789   function _authorizeUpgrade(address) internal view override {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L789


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
327    function _authorizeUpgrade(address) internal view override onlyOwner {}
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L327

## [G-14] Amounts should be checked for 0 before calling a transfer


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
228   IERC20(erc20Token).safeTransfer(recipient, tokensToSend);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L228


```solidity
File: packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol
120   nounsToken.transferFrom(address(this), owner, tokenIds[i]);

151    nounsToken.transferFrom(address(this), to, tokenIds[i]);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120

```solidity
File: packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
84    ds.nouns.safeTransferFrom(msg.sender, address(forkEscrow), tokenIds[i]);

154    ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L84


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
134   _safeTransferETHWithFallback(lastBidder, _auction.amount);

248    nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);

264    IERC20(weth).transfer(to, amount);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L134


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
224    nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L224


```solidity
File: packages/nouns-contracts/contracts/utils/ERC20Transferer.sol
34    IERC20(token).transferFrom(msg.sender, to, balance);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L34

## [G-15] >= costs less gas than >

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol 
564   if (votes > 0) {

793   newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND    

797   if (newMaxQuorumVotesBPS > MAX_QUORUM_VOTES_BPS_UPPER_BOUND) {

800   if (newMinQuorumVotesBPS > newMaxQuorumVotesBPS) {

1009  while (upper > lower) {            
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L564


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol
216    if (newObjectionPeriodDurationInBlocks > MAX_OBJECTION_PERIOD_BLOCKS)

247     if (newProposalUpdatablePeriodInBlocks > MAX_UPDATABLE_PERIOD_BLOCKS)

440     newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND

444    if (newMaxQuorumVotesBPS > MAX_QUORUM_VOTES_BPS_UPPER_BOUND) {

447     if (newMinQuorumVotesBPS > newMaxQuorumVotesBPS) {

484    if (oldVoteSnapshotBlockSwitchProposalId > 0) {

534    if (newForkPeriod > MAX_FORK_PERIOD) {                
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L216


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
132   if (votes > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L132


```solidity
File: packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
255  if (tokensToSend > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L255


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
251    if (_auction.amount > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L251

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
195   while (upper > lower) {

227   if (srcRep != dstRep && amount > 0) { 
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L195

## [G-16] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
107   bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

111   bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L107


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol
141    bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

144    bytes32 public constant PROPOSAL_TYPEHASH =
        keccak256(
            'Proposal(address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'
        );

149    bytes32 public constant UPDATE_PROPOSAL_TYPEHASH =
        keccak256(
            'UpdateProposal(uint256 proposalId,address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'
        );
        
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L141


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
47   bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');
51  bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L47


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
146  bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

150   bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L146

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
70   bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

74    bytes32 public constant DELEGATION_TYPEHASH =
        keccak256('Delegation(address delegatee,uint256 nonce,uint256 expiry)');                
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L70

## [G-17] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. 

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
171  if (auction.startTime == 0 || auction.settled) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L171

## [G-18] Can Make The Variable Outside The Loop To Save Gas 


```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol
829  address signer = proposerSignatures[i].signer;

832  uint256 signerVotes = nouns.getPriorVotes(signer, block.number - 1);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L829


```solidity
File: packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
254    uint256 tokensToSend = (erc20token.balanceOf(address(timelock)) * tokenCount) / totalSupply;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L254


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
150   uint256 nounId = tokenIds[i];

173   uint256 nounId = tokenIds[i];
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L150


## [G-19] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.


```solidity
file:       governance/NounsDAOLogicV3.sol

236         function proposeBySigs(
            ProposerSignature[] memory proposerSignatures,
            address[] memory targets,
            uint256[] memory values,
            string[] memory signatures,
            bytes[] memory calldatas,
            string memory description
          ) external returns (uint256)

278           function updateProposal(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description,
        string memory updateMessage
    ) external          

313         function updateProposalTransactions(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory updateMessage
    ) external 

338         function updateProposalBySigs(
        uint256 proposalId,
        ProposerSignature[] memory proposerSignatures,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description,
        string memory updateMessage
    ) external

```
https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L236-L243



```solidity
file:      governance/NounsDAOLogicV3.sol

236          function proposeBySigs(
        ProposerSignature[] memory proposerSignatures,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description
    ) external returns (uint256)

278        function updateProposal(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description,
        string memory updateMessage
    ) external  

313        function updateProposalTransactions(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory updateMessage
    ) external

338       function updateProposalBySigs(
        uint256 proposalId,
        ProposerSignature[] memory proposerSignatures,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description,
        string memory updateMessage
    ) external     

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L236-L244

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
206   function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
94   function delegateTo(address callee, bytes memory data) internal {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L94

## [G-20] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

Cache delay, admin, pendingAdmin  and emit cached value instead of reading from storage

```solidity
file:    governance/NounsDAOExecutor.sol

86      emit NewDelay(delay);

94      emit NewAdmin(admin);

104     emit NewPendingAdmin(pendingAdmin);

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L86

Cache proposal.id,newProposal.id,newProposal.startBlock,newProposal.endBlock,  newProposal.updatePeriodEndBlock,newProposal.proposalThreshold,  and emit cached value instead of reading from storage
```solidity
file:    governance/NounsDAOV3Proposals.sol

517     emit ProposalExecuted(proposal.id);

925             emit ProposalCreated(
            newProposal.id,
            msg.sender,
            txs.targets,
            txs.values,
            txs.signatures,
            txs.calldatas,
            newProposal.startBlock,
            newProposal.endBlock,
            description
        );

940             emit ProposalCreatedWithRequirements(
            newProposal.id,
            msg.sender,
            signers,
            txs.targets,
            txs.values,
            txs.signatures,
            txs.calldatas,
            newProposal.startBlock,
            newProposal.endBlock,
            newProposal.updatePeriodEndBlock,
            newProposal.proposalThreshold,
            minQuorumVotes,
            description
        );

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L517


Cache proposal.objectionPeriodEndBlock  and emit cached value instead of reading from storage

```solidity

file:   governance/NounsDAOV3Votes.sol

256     emit ProposalObjectionPeriodSet(proposal.id, proposal.objectionPeriodEndBlock);

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L256


## [G-21] Splitting if() statements that use && saves gas - (saves 8 gas per &&)

Instead of using the && operator in a single if statement to check multiple conditions,using multiple if statements with 1 condition per if statement will save 8 GAS per && The gas difference would only be realized if the revert condition is realized(met). The Gas saved could be higher as evident from the first instance due to refactoring which condition is checked first.
```solidity
file:       governance/NounsDAOLogicV2.sol

1026      if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber)

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1026

```solidity
file:     governance/NounsDAOV3Admin.sol

585      if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber)

```
https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#585

```solidity
file:   governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
 
227     if (srcRep != dstRep && amount > 0)

255     if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber)


```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L227


## [G-22] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

```solidity
file:

63     uint256 public constant MINIMUM_DELAY = 2 days;

64     uint256 public constant MAXIMUM_DELAY = 30 days;

66    address public admin;

68    uint256 public delay;

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L63

```solidity
file:

8       address public constant NOUNS_DAO_PROXY_SEPOLIA = 0x35d2670d7C8931AACdd37C89Ddcb0638c3c44A57;
9     address public constant NOUNS_TIMELOCK_V2_PROXY_SEPOLIA = 0x07e5D6a1550aD5E597A9b0698A474AA080A2fB28;

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3DataContractsSepolia.s.sol#L8

```solidity
file:   script/DeployDAOV3NewContractsMainnet.s.sol
 
8       address public constant NOUNS_DAO_PROXY_MAINNET = 0x6f3E6272A167e8AcCb32072d08E0957F9c79223d;
9       address public constant NOUNS_TIMELOCK_V1_MAINNET = 0x0BC3807Ec262cB779b38D65b38158acC3bfedE10;
10      uint256 public constant FORK_DAO_VOTING_PERIOD = 36000; // 5 days
11      uint256 public constant FORK_DAO_VOTING_DELAY = 36000; // 5 days

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsMainnet.s.sol#L8

```solidity
file:       script/DeployDAOV3NewContractsTestnet.s.sol

8             address public constant NOUNS_DAO_PROXY_GOERLI = 0x9e6D4B42b8Dc567AC4aeCAB369Eb9a3156dF095C;
9            address public constant NOUNS_TIMELOCK_V1_GOERLI = 0xADa0F1A73D1df49477fa41C7F8476F9eA5aB115f;
10            uint256 public constant FORK_DAO_VOTING_PERIOD = 40; // 8 minutes
11            uint256 public constant FORK_DAO_VOTING_DELAY = 1;

25          address public constant NOUNS_DAO_PROXY_SEPOLIA = 0x35d2670d7C8931AACdd37C89Ddcb0638c3c44A57;
26          address public constant NOUNS_TIMELOCK_V1_SEPOLIA = 0x332db58b51393f3a6b28d4DD8964234967e1aD33;
27          uint256 public constant FORK_DAO_VOTING_PERIOD = 40; // 8 minutes
28          uint256 public constant FORK_DAO_VOTING_DELAY = 1;

```

https://github.com/nounsDAO/nouns-monorepo/tree/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsTestnet.s.sol#L25


## [G-23] Using fixed bytes is cheaper than using string


As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.


```solidity
file: contracts/governance/NounsDAOLogicV2.sol

59      string public constant name = 'Nouns DAO';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L59


```solidity
File:  contracts/governance/NounsDAOV3Votes.sol

44       string public constant name = 'Nouns DAO';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L44


```solidity
file: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

116     string public constant name = 'Nouns DAO';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L116


## [G-24] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

Saves 5 gas per iteration

```solidity
file: contracts/governance/fork/NounsDAOForkEscrow.sol

105        numTokensInEscrow++;

138        forkId++;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L105


```solidity
file: contracts/governance/NounsDAOLogicV2.sol

239        proposalCount++;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L239

```solidity
file: contracts/governance/NounsDAOV3Proposals.sol

837            signers[numSigners++] = signer;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L837


```solidity
file: script/ProposeDAOV3UpgradeMainnet.s.sol

82         i++;

88         i++;

100        i++;

106        i++;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L82

```solidity
file: contracts/governance/fork/newdao/token/NounsTokenFork.sol

207        return _mintTo(minter, _currentNounId++);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L207


```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

153        require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L153

## [G-25]  Not using the named return variable when a function returns, wastes deployment gas


```solidity
file: script/DeployDAOV3NewContractsBase.s.sol

105        return timelockV2;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L105

## [G-26] Bytes constants are more efficient than String constants

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

```solidity
file: contracts/governance/NounsDAOLogicV2.sol

59    string public constant name = 'Nouns DAO';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L59

```solidity
file: contracts/governance/NounsDAOV3Votes.sol

44    string public constant name = 'Nouns DAO';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L44

```solidity
file: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

48       string public constant NAME = 'NounsAuctionHouseFork';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L48

```solidity
file: contracts/governance/fork/newdao/token/NounsTokenFork.sol

46       string public constant NAME = 'NounsTokenFork';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L46

```solidity
file: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

116       string public constant name = 'Nouns DAO';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L116


## [G-27] Avoid Storage keyword during modifiers

The problem with using modifiers with storage variables is that these accesses will likely have to be obtained multiple times, to pass it to the modifier, and to read it into the method code.


```solidity
file: contracts/governance/NounsDAOV3Admin.sol

150    modifier onlyAdmin(NounsDAOStorageV3.StorageV3 storage ds) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L150


## [G-28] Don't initialize variables with default value


```solidity
file: contracts/governance/NounsDAOV3DynamicQuorum.sol

107        uint256 lower = 0;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L107

```solidity
file: contracts/governance/NounsDAOV3Votes.sol

224        bool isForVoteInLastMinuteWindow = false;

229        bool isDefeatedBefore = false;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L224


```solidity
file: script/ProposeDAOV3UpgradeMainnet.s.sol

75        uint256 i = 0;

84        values[i] = 0;

90        values[i] = 0;

102       values[i] = 0;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L75

```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

52         uint8 public constant decimals = 0;
  
193        uint32 lower = 0;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L52





## [G-29] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
file: contracts/governance/NounsDAOExecutor.sol

91        admin = msg.sender;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L91

```solidity
file: contracts/governance/NounsDAOExecutorV2.sol

114        admin = msg.sender;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L114

```solidity
file: contracts/governance/NounsDAOProxy.sol

53        admin = msg.sender;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L53

```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

124        if (delegatee == address(0)) delegatee = msg.sender;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L124


## [G-30] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow

```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

280        uint96 c = a + b;

291        return a - b;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L280

```solidity
file: contracts/governance/NounsDAOLogicV2.sol

1010            uint256 center = upper - (upper - lower) / 2;

1067            return (number * bps) / 10000;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1010

```solidity
file: contracts/governance/NounsDAOV3Admin.sol

488        uint256 newVoteSnapshotBlockSwitchProposalId = ds.proposalCount + 1;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L488

```solidity
file: contracts/governance/NounsDAOV3DynamicQuorum.sol

63        uint256 againstVotesBPS = (10000 * againstVotes) / totalSupply;

64        uint256 quorumAdjustmentBPS = (params.quorumCoefficient * againstVotesBPS) / 1e6;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L63

```solidity
file: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

215            uint256 endTime = startTime + duration;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L215

## [G-31] Non efficient zero initialization
 
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

```solidity
file: contracts/governance/NounsDAOLogicV2.sol

305        for (uint256 i = 0; i < proposal.targets.length; i++) {

343        for (uint256 i = 0; i < proposal.targets.length; i++) {

372        for (uint256 i = 0; i < proposal.targets.length; i++) {

405        for (uint256 i = 0; i < proposal.targets.length; i++) {            

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L305

```solidity
file: contracts/governance/NounsDAOV3Admin.sol

602        for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L602

```solidity
file: contracts/governance/NounsDAOV3Proposals.sol

409        for (uint256 i = 0; i < proposerSignatures.length; ++i) {

446        for (uint256 i = 0; i < proposal.targets.length; i++) {

508        for (uint256 i = 0; i < proposal.targets.length; i++) {

553        for (uint256 i = 0; i < proposal.targets.length; i++) {

590        for (uint256 i = 0; i < signers.length; ++i) {

602        for (uint256 i = 0; i < proposal.targets.length; i++) {

826        for (uint256 i = 0; i < proposerSignatures.length; ++i) {

860        for (uint256 i = 0; i < txs.signatures.length; ++i) {

865        for (uint256 i = 0; i < txs.calldatas.length; ++i) {                                

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L409

```solidity
file: contracts/governance/fork/NounsDAOForkEscrow.sol

117        for (uint256 i = 0; i < tokenIds.length; i++) {

148        for (uint256 i = 0; i < tokenIds.length; i++) {    

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L117

```solidity
file: contracts/governance/fork/NounsDAOV3Fork.sol

83        for (uint256 i = 0; i < tokenIds.length; i++) {

153        for (uint256 i = 0; i < tokenIds.length; i++) {

252        for (uint256 i = 0; i < erc20Count; ++i) {        

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L83

```solidity
file: contracts/governance/fork/newdao/token/NounsTokenFork.sol

149        for (uint256 i = 0; i < tokenIds.length; i++) {

172        for (uint256 i = 0; i < tokenIds.length; i++) {    

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L149

```solidity
file: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

209        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

223        for (uint256 i = 0; i < tokenIds.length; i++) {

231        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

238        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

248        for (uint256 i = 0; i < addresses.length; i++) {

397        for (uint256 i = 0; i < proposal.targets.length; i++) {

435        for (uint256 i = 0; i < proposal.targets.length; i++) {

452        for (uint256 i = 0; i < proposal.targets.length; i++) {

796        for (uint256 i = 0; i < erc20tokens.length - 1; i++) {                    

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L209

