|   No   |   issue  | instance |
|------|----------|----------|
|[G-01]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|10|
|[G-02]|Use selfbalance() instead of address(this).balance|4|
|[G-03]| State Variable can be packed into fewer storage slots|1|
|[G-04]| Use constants instead of type(uintx).max|5|
|[G-05]|State variables should be cached in stack variables rather than re-reading them from storage|4|
|[G-06]|Use != 0 instead of > 0 for unsigned integer comparison|10|
|[G-07]| bytes constants are more eficient than string constans|5|
|[G-08]|Not using the named return variables when a function returns, wastes deployment gas|1|
|[G-09]|Use calldata instead of memory|9|
|[G-10]|Avoid storage keyword during modifiers|1|
|[G-11]|Sort Solidity operations using short-circuit mode|1|
|[G-12]|internal functions not called by the contract should be removed to save deployment gas|4|
|[G-13]|Amounts should be checked for 0 before calling a transfer|10|
|[G-14]|With assembly, .call (bool success) transfer can be done gas-optimized|7|
|[G-15]|use Mappings Instead of Arrays|1|
|[G-16]|Use hardcode address instead address(this)|9|
|[G-17]|Do not calculate constants|5|
|[G-18]|Use assembly for math (add, sub, mul, div)|8|
|[G-19]|Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|3|
|[G-20]|state variables should not used in emit|3|
|[G-21]|Can Make The Variable Outside The Loop To Save Gas |5|
|[G-22]| >= costs less gas than >|17|
|[G-23]|Use nested if statements instead of &&|2|
|[G-24]| Avoid contract existence checks by using low level calls|20|


## [G-01] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant


In Solidity, constant and immutable are both used to declare values that cannot be changed at runtime. However, there is a subtle difference between the two that can have an impact on gas usage.
When you use the constant keyword, the value is computed at compile time and hardcoded into the bytecode. This can be useful for small values, such as integers or strings, but it can lead to bloated bytecode if used excessively. Additionally, the constant keyword has been deprecated in favor of pure and view functions for functions that do not modify state.

```solidity
107   bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L107

```solidity
111   bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L111


```solidity
141    bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L141

```solidity
144    bytes32 public constant PROPOSAL_TYPEHASH =
        keccak256(
            'Proposal(address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'
        );
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L144


```solidity
149    bytes32 public constant UPDATE_PROPOSAL_TYPEHASH =
        keccak256(
            'UpdateProposal(uint256 proposalId,address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'
        );
        
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L149


```solidity
47   bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');
51  bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L47


```solidity
146  bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

150   bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L146

```solidity
70   bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L70

```solidity
74    bytes32 public constant DELEGATION_TYPEHASH =
        keccak256('Delegation(address delegatee,uint256 nonce,uint256 expiry)');                
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L74



## [G-02] Use selfbalance() instead of address(this).balance

In Solidity, selfbalance() and address(this).balance are both used to get the balance of the current contract. However, using selfbalance() can be more gas efficient than using address(this).balance.

```solidity
823        uint256 amount = address(this).balance;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L823

```solidity
1035       uint256 balance = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1035

```solidity
469        uint256 amount = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L469

```solidity
297            uint256 balance = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L297




## [G-03] State Variable can be packed into fewer storage slots


In Solidity, state variables are stored in the contract's storage, which is a key-value store that is organized into 32-byte slots. Each state variable takes up at least one storage slot, regardless of its size.

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




## [G-04] Use constants instead of type(uintx).max

In Solidity, type(uintx).max can be used to get the maximum value of an unsigned integer of a certain number of bits (where x is the number of bits). For example, type(uint256).max returns the maximum value of a 256-bit unsigned integer.

 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.
The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.



```solidity
595        require(n <= type(uint32).max, errorMessage);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L595

```solidity
146        require(n <= type(uint32).max, errorMessage);

151        if (n > type(uint16).max) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L146

```solidity
1079        require(n <= type(uint32).max, errorMessage);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1079


```solidity
1084        if (n > type(uint16).max) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1084




## [G‑05] State variables should be cached in stack variables rather than re-reading them from storage

To reduce the gas cost of reading state variables, you can cache them in stack variables within your function. This means that you read the value from storage once and then store it in a local variable, which you can then use throughout the function. By doing this, you avoid the need to read the value from storage multiple times, which can lead to significant gas savings.

```solidity
137        auction.amount = msg.value;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L137

```solidity
138        auction.bidder = payable(msg.sender);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L138

```solidity
143        auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L143












## [G-06]Use != 0 instead of > 0 for unsigned integer comparison

In Solidity, state variables are stored in the contract's storage, which is a key-value store that is organized into 32-byte slots. Each time you read a state variable from storage, you incur a gas cost, which can add up if you are reading the same variable multiple times within a function.


```solidity
564   if (votes > 0) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L564

```solidity
1026  if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1026


```solidity
484   if (oldVoteSnapshotBlockSwitchProposalId > 0) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L484

```solidity
585   if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L585


```solidity
888  if (proposal.signers.length > 0) revert ProposerCannotUpdateProposalWithSigners();
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L888


```solidity
132   if (votes > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L132


```solidity
255   if (tokensToSend > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L255

```solidity
251    if (_auction.amount > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L151

```solidity
239   if (balancesToSend[i] > 0) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L239

```solidity
281   if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L281


```solidity
227    if (srcRep != dstRep && amount > 0) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L227

```solidity
255    if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {    
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L255






## [G-07] bytes constants are more eficient than string constans

In Solidity, bytes constants are more gas efficient than string constants. This is because bytes are stored in memory as a contiguous array of bytes, while string variables are stored as an array of pointers to the actual string data.

When you declare a string constant, the Solidity compiler stores the string data in memory and generates code to create a pointer to that data. This can be relatively expensive in terms of gas cost, especially if the string is long or if you are declaring multiple string constants.


```solidity
59      string public constant name = 'Nouns DAO';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L59


```solidity
44   string public constant name = 'Nouns DAO';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L44


```solidity
116   string public constant name = 'Nouns DAO';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L116


```solidity
82     string public constant NAME = 'NounsDAOExecutorV2';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L82


```solidity
46   string public constant NAME = 'NounsTokenFork';
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L46




## [G-08]Not using the named return variables when a function returns, wastes deployment gas

```solidity
105        return timelockV2;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L105






## [G-09] Use calldata instead of memory

In Solidity, calldata is a special data location that contains the function arguments passed to a contract. When you declare a function parameter with the calldata keyword, Solidity creates a pointer to the data in the function arguments, rather than creating a new copy of the data in memory. This can be more gas efficient than using the memory keyword, especially for large or complex data structures.


```solidity
236   function proposeBySigs(
        ProposerSignature[] memory proposerSignatures,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description
    ) external returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L236


```solidity
278   function updateProposal(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description,
        string memory updateMessage
    ) external {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L278

```solidity
313    function updateProposalTransactions(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory updateMessage
    ) external {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L313

```soldity
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
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L338

```solidity
94   function delegateTo(address callee, bytes memory data) internal {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L94

```solidity
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
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L288

```solidity
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
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L321

```solidity
206   function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206





## [G-10] Avoid storage keyword during modifiers

The problem with using modifiers with storage variables is that these accesses will likely have to be obtained multiple times, to pass it to the modifier, and to read it into the method code.  

```solidity
150   modifier onlyAdmin(NounsDAOStorageV3.StorageV3 storage ds) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L150



## [G-11] Sort Solidity operations using short-circuit mode

In Solidity, short-circuit mode is a way of optimizing logical operations that involve multiple conditions. When short-circuit mode is enabled, Solidity stops evaluating the conditions as soon as it determines the final outcome of the operation. This can lead to significant gas savings, especially for complex logical operations.

//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
```solidity
171  if (auction.startTime == 0 || auction.settled) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L171






## [G-12] internal functions not called by the contract should be removed to save deployment gas

In Solidity, internal functions are only callable from within the contract in which they are defined, and cannot be accessed externally. If you have internal functions in your contract that are not called by the contract itself, you can consider removing them to save deployment gas.

This is because each function in your contract adds to the overall size of the bytecode, which can increase the gas cost of deploying your contract. By removing unused functions, you can reduce the size of the bytecode and lower the gas cost of deployment.


243   function _authorizeUpgrade(address) internal view override {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L243


```solidity
281    function _authorizeUpgrade(address) internal view override onlyOwner {}
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L281


```solidity
789   function _authorizeUpgrade(address) internal view override {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L789


```solidity
327    function _authorizeUpgrade(address) internal view override onlyOwner {}
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L327





## [G-13] Amounts should be checked for 0 before calling a transfer

In Solidity, when you call the transfer function to send Ether or tokens from one address to another, it's a good practice to check the amount being transferred to ensure that it is not zero. This is because calling transfer with a zero amount can result in an unnecessary transaction, which can increase the gas cost of your contract.
When you call transfer with a zero amount, the transaction is still executed on the blockchain, but no Ether or tokens are actually transferred. This means that the transaction is essentially a no-op, and can be considered a wasted gas cost.

```solidity
228   IERC20(erc20Token).safeTransfer(recipient, tokensToSend);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L228


```solidity
120   nounsToken.transferFrom(address(this), owner, tokenIds[i]);

151    nounsToken.transferFrom(address(this), to, tokenIds[i]);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120

```solidity
84    ds.nouns.safeTransferFrom(msg.sender, address(forkEscrow), tokenIds[i]);

154    ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L84


```solidity
134   _safeTransferETHWithFallback(lastBidder, _auction.amount);

248    nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);

264    IERC20(weth).transfer(to, amount);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L134


```solidity
224    nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L224


```solidity
34    IERC20(token).transferFrom(msg.sender, to, balance);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L34




## [G-14] With assembly, .call (bool success) transfer can be done gas-optimized




```solidity
173   (bool success, bytes memory returnData) = target.call{ value: value }(callData);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L173


```solidity
196    (bool success, bytes memory returnData) = target.call{ value: value }(callData);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L196


```solidity
824    (bool sent, ) = msg.sender.call{ value: amount }('');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L824

```solidity
1043    (bool refundSent, ) = tx.origin.call{ value: refundAmount }('');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1043


```solidity
470   (bool sent, ) = msg.sender.call{ value: amount }('');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L470


```solidity
305   (bool refundSent, ) = msg.sender.call{ value: refundAmount }('');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L305

```solidity
373   (bool success, ) = to.call{ value: value, gas: 30_000 }(new bytes(0));
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L373




## [G-15]use Mappings Instead of Arrays


Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.


```solidity
56   address[] public erc20TokensToIncludeInQuit;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol#L56










## [G-16] Use hardcode address instead address(this)

In Solidity, using a hardcoded address instead of address(this) can lead to gas savings in certain cases. This is because address(this) is a dynamic address that is computed at runtime, while a hardcoded address is a static value that is known at compile time.
When you use address(this) in your contract, Solidity generates code to compute the address at runtime. This can be relatively expensive in terms of gas cost, especially if you are calling the address(this) function multiple times or in a loop.

```solidity
104     require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L104


```solidity
122     msg.sender == address(this),
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L122

```solidity
245     msg.sender == address(this),
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L245




```solidity
595   abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), getChainIdInternal(), address(this))
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L595

```solidity
823   uint256 amount = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L823

```solidity
1035  uint256 balance = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1035


```solidity
120   nounsToken.transferFrom(address(this), owner, tokenIds[i]);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120


```solidity
151    nounsToken.transferFrom(address(this), to, tokenIds[i]);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L151

```solidity
165    return nounsToken.balanceOf(address(this)) - numTokensInEscrow;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L165



## [G-17] Do not calculate constants

In Solidity, calculating constants at runtime can be expensive in terms of gas cost, especially if the calculation is complex or involves large numbers. To save gas, you can calculate constants offline and include the results as hardcoded values in your contract.
When you include hardcoded constants in your contract, Solidity does not need to calculate the values at runtime, as they are already known and included in the bytecode. This can lead to significant gas savings, especially if the constants are used multiple times or in a loop.

```solidity
118   uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L118
```

```solidity
121    uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L121

```solidity
127   uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L127


```solidity
145   uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L145


```solidity
148   uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L118




## [G-18] Use assembly for math (add, sub, mul, div)


Using assembly for math operations such as add, sub, mul, and div can sometimes help to save gas in smart contracts. This is because assembly allows you to perform low-level operations directly on the Ethereum Virtual Machine (EVM), which can be more efficient than using higher-level programming languages.

```solidity
280        uint96 c = a + b;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L280

```solidity
291        return a - b;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L291

```solidity
1010            uint256 center = upper - (upper - lower) / 2;

1067            return (number * bps) / 10000;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1010

```solidity
488        uint256 newVoteSnapshotBlockSwitchProposalId = ds.proposalCount + 1;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L488

```solidity
63        uint256 againstVotesBPS = (10000 * againstVotes) / totalSupply;

64        uint256 quorumAdjustmentBPS = (params.quorumCoefficient * againstVotesBPS) / 1e6;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L63

```solidity
215            uint256 endTime = startTime + duration;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L215



## [G-19] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
91   admin = msg.sender;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L91

```solidity
114  admin = msg.sender;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L114

```solidity
124        if (delegatee == address(0)) delegatee = msg.sender;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L124


## [G-20] state variables should not used in emit

It is generally true that minimizing the use of state variables in emit statements can help reduce the gas cost of smart contract interactions. This is because each time a state variable is accessed or modified, it requires a read or write operation to the contract's storage, which can be expensive in terms of gas.

Instead, it is often better to pass arguments directly to the emit statement, rather than relying on state variables to store those values. This can help reduce the number of storage operations needed during contract execution and can save gas.


```solidity
86        emit NewDelay(delay);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L86

```
94        emit NewAdmin(admin);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L94

```solidity
104        emit NewPendingAdmin(pendingAdmin);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L104


## [G-21] State varaibles only set in the Constructor should be declared Immutable

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

```solidity
76        admin = admin_;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L76
```solidity
77        delay = delay_;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L77






## [G-22] >= costs less gas than >

In Solidity, using the >= operator can be more gas-efficient than using the > operator in certain cases. This is because the >= operator is implemented as a single opcode, while the > operator is implemented as a combination of two opcodes.

```solidity
564   if (votes > 0) {

793   newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND    

797   if (newMaxQuorumVotesBPS > MAX_QUORUM_VOTES_BPS_UPPER_BOUND) {

800   if (newMinQuorumVotesBPS > newMaxQuorumVotesBPS) {

1009  while (upper > lower) {            
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L564


```solidity
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
132   if (votes > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L132


```solidity
255  if (tokensToSend > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L255


```solidity
251    if (_auction.amount > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L251

```solidity
195   while (upper > lower) {

227   if (srcRep != dstRep && amount > 0) { 
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L195




## [G-21] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.



```solidity
829  address signer = proposerSignatures[i].signer;

832  uint256 signerVotes = nouns.getPriorVotes(signer, block.number - 1);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L829


```solidity
254    uint256 tokensToSend = (erc20token.balanceOf(address(timelock)) * tokenCount) / totalSupply;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L254


```solidity
150   uint256 nounId = tokenIds[i];

173   uint256 nounId = tokenIds[i];
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L150



## [G-23]Use nested if statements instead of &&



In Solidity, using nested if statements can be more gas-efficient than using the && operator in certain cases. This is because the && operator is implemented as a combination of two if statements, which can lead to higher gas costs.
```
```solidity
381   if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L381

```solidity
227   if (srcRep != dstRep && amount > 0) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L227


## [G‑24] Avoid contract existence checks by using low level calls


In Solidity, avoiding contract existence checks by using low-level calls can help to save gas costs in certain cases. This is because checking for contract existence using the address.balance property or the address.callcode function can be relatively expensive in terms of gas cost.
When you check for contract existence using address.balance or address.callcode, Solidity generates additional code to perform the check, which can increase the overall gas cost of your contract. However, if you use a low-level call to interact with the contract, Solidity automatically performs a contract existence check, which can save gas costs.

solidity
246   nouns.burn(_auction.nounId);

248   nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);

263   IWETH(weth).deposit{ value: amount }();

264   IERC20(weth).transfer(to, amount);
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/#L246


solidity
104    NounsTokenFork(token).initialize(
        
114      NounsAuctionHouseFork(auction).initialize(
         
126     NounsDAOExecutorV2(payable(treasury)).initialize(governor, originalTimelock.delay());


141     NounsDAOLogicV1Fork(governor).initialize(
          
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol#L104


solidity
120   nounsToken.transferFrom(address(this), owner, tokenIds[i]);

151   nounsToken.transferFrom(address(this), to, tokenIds[i]);

165   return nounsToken.balanceOf(address(this)) - numTokensInEscrow;
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120

solidity
228   IERC20(erc20Token).safeTransfer(recipient, tokensToSend);
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L228

solidity
206    temp.totalSupply = nouns.totalSupply();

211     nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,

367      nouns.getPriorVotes(proposal.proposer, block.number - 1) <= proposal.proposalThreshold,

623     uint96 votes = nouns.getPriorVotes(voter, proposalCreationBlock(proposal));

922    return bps2Uint(proposalThresholdBPS, nouns.totalSupply());

1056    return bps2Uint(getDynamicQuorumParamsAt(block.number).minQuorumVotesBPS, nouns.totalSupply());

1063    return bps2Uint(getDynamicQuorumParamsAt(block.number).maxQuorumVotesBPS, nouns.totalSupply());
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L228


solidity
33    uint256 balance = IERC20(token).balanceOf(msg.sender);

34    IERC20(token).transferFrom(msg.sender, to, balance);
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L33


solidity
100  timelockV1.delay()
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L100


