# gas optimization

# SUMMRY
|      |   ISSUE  | INSTANCE |
|------|----------|----------|
|[G-01]|Use calldata instead of memory|27|
|[G-02]|Can Make The Variable Outside The Loop To Save Gas |5|
|[G-03]|Use nested if statements instead of &&|2|
|[G-04]| Avoid contract existence checks by using low level calls|22|
|[G-05]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|11|
|[G-06]| >= costs less gas than >|17|
|[G-07]|internal functions not called by the contract should be removed to save deployment gas|4|
|[G-08]|Amounts should be checked for 0 before calling a transfer|10|
|[G-09]| Do not calculate constants|5|
|[G-10]|With assembly, .call (bool success) transfer can be done gas-optimized|7|
|[G-11]|Use != 0 instead of > 0 for unsigned integer comparison|12|
|[G-12]|Use hardcode address instead address(this)|9|
|[G-13]|use Mappings Instead of Arrays|1|
|[G-14]|Use constants instead of type(uintx).max|5|
|[G-15]|Not using the named return variables when a function returns, wastes deployment gas|1|
|[G-16]|State Variable can be packed into fewer storage slots|1|
|[G-17]|State variables should be cached in stack variables rather than re-reading them from storage|21|
|[G-18]|bytes constants are more eficient than string constans|5|
|[G-19]|Use selfbalance() instead of address(this).balance|5|
|[G-20]|Instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant|11|
|[G-21]| Avoid storage keyword during modifiers|1|
|[G-22]|Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|4|
|[G-23]|state variables should not used in emit|3|
|[G-24]|State varaibles only set in the Constructor should be declared Immutable|2|
|[G-25]|Use assembly for math (add, sub, mul, div)|8|
|[G-26]|Use assembly to hash instead of solidity|5|
|[G-27]|Sort Solidity operations using short-circuit mode|1|



## [G-01] Use calldata instead of memory
using calldata instead of memory for function arguments in external functions can help to reduce gas costs and improve the performance of your contracts.

When a function is marked as external, its arguments are passed in the calldata section of the transaction, which is a read-only area of memory that contains the input data for the transaction. Using calldata instead of memory for function arguments can be more gas-efficient, especially for functions that take large arguments.

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

## [G-02] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

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

## [G-03]Use nested if statements instead of &&
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


## [G‑04] Avoid contract existence checks by using low level calls
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

## [G-05] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.

In contrast, immutable variables are evaluated at compilation time, and their values are included in the bytecode of the contract as constants. This means that any expensive operations performed as part of the immutable expression are only executed once, when the contract is compiled, and the result is reused every time the contract is deployed. This can result in lower gas costs compared to using constant variables.

Let's consider an example to illustrate this. Suppose we want to store the hash of a string as a constant value in our contract. We could do this using a constant variable, like so:

```
bytes32 constant MY_HASH = keccak256("my string");
```
Alternatively, we could use an immutable variable, like so:
```
bytes32 immutable MY_HASH = keccak256("my string");
```


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


## [G-06] >= costs less gas than >
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

## [G-07] internal functions not called by the contract should be removed to save deployment gas
When you define an internal function in a Solidity contract, Solidity generates code for that function and includes it in the contract bytecode. If the function is not called by the contract itself or any of its external functions, then including the code for that function in the contract bytecode is unnecessary and can lead to higher deployment gas costs.

By removing unused internal functions, you can reduce the size of your contract bytecode and lower the overall deployment gas cost of your contract.

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

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

## [G-08] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.


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


## [G-09] Do not calculate constants
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

## [G-11]Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```

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

## [G-12] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

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

## [G-13]use Mappings Instead of Arrays
Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol
56   address[] public erc20TokensToIncludeInQuit;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol#L56

## [G-14] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.


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

## [G-15]Not using the named return variables when a function returns, wastes deployment gas
When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost

```solidity
File: script/DeployDAOV3NewContractsBase.s.sol
105        return timelockV2;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L105

## [G-16] State Variable can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas)

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


## [G‑17] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.




```solidity
File: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
137        auction.amount = msg.value;

138        auction.bidder = payable(msg.sender);

143        auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L137


```
State Variable: ds
Contract: NounsDAOStorageV3
Type Info:
Struct: StorageV3
Contract: NounsDAOStorageV3

INHERITED (StorageV3) StateVar NounsDAOStorageV3.ds (Declaration: NounsDAOStorageV3#651)
```

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
155    ds._setVotingPeriod(daoParams_.votingPeriod);
        ds._setVotingDelay(daoParams_.votingDelay);
        ds._setProposalThresholdBPS(daoParams_.proposalThresholdBPS);
        ds.timelock = INounsDAOExecutorV2(timelock_);
        ds.nouns = NounsTokenLike(nouns_);
        ds.forkEscrow = INounsDAOForkEscrow(forkEscrow_);
        ds.forkDAODeployer = IForkDAODeployer(forkDAODeployer_);
        ds.vetoer = vetoer_;

169     ds._setLastMinuteWindowInBlocks(daoParams_.lastMinuteWindowInBlocks);
        ds._setObjectionPeriodDurationInBlocks(daoParams_.objectionPeriodDurationInBlocks);
        ds._setProposalUpdatablePeriodInBlocks(daoParams_.proposalUpdatablePeriodInBlocks);

851     ds._setForkEscrow(forkEscrow_);
        ds._setForkDAODeployer(forkDAODeployer_);
        ds._setErc20TokensToIncludeInFork(erc20TokensToIncludeInFork_);
        ds._setForkPeriod(forkPeriod_);
        ds._setForkThresholdBPS(forkThresholdBPS_);
                
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L155


## [G-18] bytes constants are more eficient than string constans
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

## [G-19] Use selfbalance() instead of address(this).balance
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

## [G-20] Instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
107   bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

111   bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L107



```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol
141   bytes32 public constant DOMAIN_TYPEHASH =
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

51   constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L47


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
146    bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

150     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L146


```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
70    bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

74     bytes32 public constant DELEGATION_TYPEHASH =
        keccak256('Delegation(address delegatee,uint256 nonce,uint256 expiry)');
                        
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L70

## [G-21] Avoid storage keyword during modifiers
The problem with using modifiers with storage variables is that these accesses will likely have to be obtained multiple times, to pass it to the modifier, and to read it into the method code.  

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
150   modifier onlyAdmin(NounsDAOStorageV3.StorageV3 storage ds) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L150

## [G-22] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
91   admin = msg.sender;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L91

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
114  admin = msg.sender;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L114


```solidity
File: contracts/governance/NounsDAOProxy.sol
53        admin = 
file: contracts/governance/NounsDAOExecutor.sol

91        admin = msg.sender;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L91


## [G-23] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.
```solidity
File:  packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
86        emit NewDelay(delay);

94        emit NewAdmin(admin);

104        emit NewPendingAdmin(pendingAdmin);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L86

## [G-24] State varaibles only set in the Constructor should be declared Immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
76        admin = admin_;

77        delay = delay_;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L76

## [G-25] Use assembly for math (add, sub, mul, div)
Using assembly for math operations such as add, sub, mul, and div can sometimes help to save gas in smart contracts. This is because assembly allows you to perform low-level operations directly on the Ethereum Virtual Machine (EVM), which can be more efficient than using higher-level programming languages.

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
280        uint96 c = a + b;

291        return a - b;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L280

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
1010            uint256 center = upper - (upper - lower) / 2;

1067            return (number * bps) / 10000;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1010

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
488        uint256 newVoteSnapshotBlockSwitchProposalId = ds.proposalCount + 1;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L488

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol
63        uint256 againstVotesBPS = (10000 * againstVotes) / totalSupply;

64        uint256 quorumAdjustmentBPS = (params.quorumCoefficient * againstVotesBPS) / 1e6;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L63

```solidity
File: packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
215            uint256 endTime = startTime + duration;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L215


## [G-26] Use assembly to hash instead of solidity
```
function solidityHash(uint256 a, uint256 b) public view {
    //unoptimized
    keccak256(abi.encodePacked(a, b));
}
```
Gas: 313
```
function assemblyHash(uint256 a, uint256 b) public view {
    //optimized
    assembly {
        mstore(0x00, a)
        mstore(0x20, b)
        let hashedVal := keccak256(0x00, 0x40)
    }
} 
```
Gas: 231

```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
120   bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

136    bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

151    bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L120


```solidity
File: packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
143    bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

174      bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L143

## [G-27] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

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


