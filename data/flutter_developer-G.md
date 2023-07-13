|      |   issue  | instance |
|------|----------|----------|
|[G-1]|Use selfbalance() instead of address(this).balance|5|
|[G-2]|bytes constants are more eficient than string constans|5|
|[G-3]|State variables should be cached in stack variables rather than re-reading them from storage|3|
|[G-4]|State Variable can be packed into fewer storage slots|1|
|[G-5]|Not using the named return variables when a function returns, wastes deployment gas|1|
|[G-6]|Use constants instead of type(uintx).max|5|
|[G-7]|use Mappings Instead of Arrays|1|
|[G-8]|Use hardcode address instead address(this)|9|
|[G-9]|Use != 0 instead of > 0 for unsigned integer comparison|12|
|[G-10]|With assembly, .call (bool success) transfer can be done gas-optimized|7|
|[G-11]| Do not calculate constants|5|
|[G-12]|Amounts should be checked for 0 before calling a transfer|10|
|[G-13]|internal functions not called by the contract should be removed to save deployment gas|4|
|[G-14]| >= costs less gas than >|17|
|[G-15]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|11|
|[G-16]|Sort Solidity operations using short-circuit mode|1|
|[G-17]|Use nested if statements instead of &&|2|
|[G-18]|Can Make The Variable Outside The Loop To Save Gas |5|
|[G-19]|Use calldata instead of memory|27|




## [G-1] Use selfbalance() instead of address(this).balance
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

## [G-2] bytes constants are more eficient than string constans
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




## [G‑3] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```solidity
File: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
137        auction.amount = msg.value;

138        auction.bidder = payable(msg.sender);

143        auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L137


## [G-4] State Variable can be packed into fewer storage slots



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


## [G-5]Not using the named return variables when a function returns, wastes deployment gas
When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost

```solidity
File: script/DeployDAOV3NewContractsBase.s.sol
105        return timelockV2;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L105



## [G-6] Use constants instead of type(uintx).max


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

## [G-7]use Mappings Instead of Arrays



```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol
56   address[] public erc20TokensToIncludeInQuit;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol#L56


## [G-8] Use hardcode address instead address(this)


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


## [G-9]Use != 0 instead of > 0 for unsigned integer comparison


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


## [G-10] With assembly, .call (bool success) transfer can be done gas-optimized
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


## [G-11] Do not calculate constants
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

## [G-12] Amounts should be checked for 0 before calling a transfer


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


## [G-14] >= costs less gas than >
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


## [G-15] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant



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


## [G-16] Sort Solidity operations using short-circuit mode
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


## [G-17]Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```
contract NestedIfTest {

    //Execution cost: 22334 gas
    function funcBad(uint256 input) public pure returns (string memory) { 
       if (input<10 && input>0 && input!=6){ 
           return "If condition passed";
       } 

   }

    //Execution cost: 22294 gas
    function funcGood(uint256 input) public pure returns (string memory) { 
    if (input<10) { 
        if (input>0){
            if (input!=6){
                return "If condition passed";
            }
        }
    }
   }
}
   
```
```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
381   if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L381

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
227   if (srcRep != dstRep && amount > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L227


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



## [G-19] Use calldata instead of memory
using calldata instead of memory for function arguments in external functions can help to reduce gas costs and improve the performance of your contracts.


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
236   function proposeBySigs(
        ProposerSignature[] memory proposerSignatures,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description
    ) external returns (uint256) {

278   function updateProposal(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description,
        string memory updateMessage
    ) external {


313    function updateProposalTransactions(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory updateMessage
    ) external {

338   function updateProposalBySigs(
        uint256 proposalId,
        ProposerSignature[] memory proposerSignatures,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description,
        string memory updateMessage
    ) external {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L236

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
94   function delegateTo(address callee, bytes memory data) internal {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L94

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol
288    function updateProposal(
        NounsDAOStorageV3.StorageV3 storage ds,
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description,
        string memory updateMessage
    ) external {

321  function updateProposalTransactions(
        NounsDAOStorageV3.StorageV3 storage ds,
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory updateMessage
    ) external {
    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L288

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
206   function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206



