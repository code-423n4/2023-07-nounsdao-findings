# Gas Optimization

# Summary

| Number | Optimization Details                                                                                                                                      | Context |
| :----: | :-------------------------------------------------------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Avoid contract existence checks by using low level calls                                                                                                  |   10    |
| [G-02] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant                                                  |   11    |
| [G-03] | Using fixed bytes is cheaper than using string                                                                                                            |    6    |
| [G-04] | state variables should be cached in stack variables rather than re-reading them from storage                                                              |    1    |
| [G-05] | stack variable used as a cheaper cache for a state variable is only used once                                                                             |   36    |
| [G-06] | Can make the state variable outside the loop to save gas                                                                                                  |    1    |
| [G-07] | Use calldata instead of memory for function arguments that do not get mutated                                                                             |   53    |
| [G-08] | Before some functions, we should check some variables ro possible gas save                                                                                |    1    |
| [G-09] | Instead of calculating a state variable with keccak256() every time the contract is made pre calculate them before and only give the result to a constant |   11    |
| [G-10] | Avoid storage keyword during modifiers                                                                                                                    |   20    |
| [G-11] | With assembly, .call (bool success)  transfer can be done gas-optimized                                                                                   |    4    |
| [G-12] | Use constants instead of type(uintx).max                                                                                                                  |    6    |
| [G-13] | Use nested if and, avoid multiple check combinations &&                                                                                                   |    6    |
| [G-14] | Caching global variables is more expensive than using the actual variable                                                                                 |    2    |
| [G-15] | Use assembly when getting a contract’s balance                                                                                                            |    4    |
| [G-16] | Using > 0 costs more gas than != 0 when used on a uint                                                                                                    |    9    |
| [G-17] | Do not calculate constants variables                                                                                                                      |    4    |
| [G-18] | Use do while loops instead of for loops                                                                                                                   |   28    |
| [G-19] | Use assembly to hash instead of solidity                                                                                                                  |   18    |
| [G-20] | use mappings instead of arrays                                                                                                                            |   16    |
| [G-21] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4                                                                     |    5    |
| [G-22] | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements |    10    |
| [G-23] | Use assembly for math (add, sub, mul, div)  |    9    |

## [G-01] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

Total gas save = `100 * 10 = 1000`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L33

```solidity
File: utils/ERC20Transferer.sol

33        uint256 balance = IERC20(token).balanceOf(msg.sender);

34        IERC20(token).transferFrom(msg.sender, to, balance);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L228

```solidity
File: governance/NounsDAOExecutorV2.sol

228        IERC20(erc20Token).safeTransfer(recipient, tokensToSend);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol#L104

```solidity
File: governance/fork/ForkDAODeployer.sol

104        NounsTokenFork(token).initialize(


114        NounsAuctionHouseFork(auction).initialize(


126        NounsDAOExecutorV2(payable(treasury)).initialize(governor, originalTimelock.delay());


141        NounsDAOLogicV1Fork(governor).initialize(

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L157

```solidity
File: governance/fork/NounsDAOV3Fork.sol


157        NounsTokenFork(ds.forkDAOToken).claimDuringForkPeriod(msg.sender, tokenIds);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

```solidity
File: governance/fork/newdao/NounsAuctionHouseFork.sol


263            IWETH(weth).deposit{ value: amount }();


264            IERC20(weth).transfer(to, amount);

```

## [G-02] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

By using immutable, you ensure that the value is calculated and stored once during deployment, rather than being re-evaluated each time the contract is accessed. This approach can improve contract efficiency and reduce gas costs compared to using constant.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

```solidity
File: governance/NounsDAOLogicV2.sol

107    bytes32 public constant DOMAIN_TYPEHASH =
108        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


111    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L47

```solidity
File: governance/NounsDAOV3Votes.sol

47    bytes32 public constant DOMAIN_TYPEHASH =
48        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


51    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol

```solidity
File: governance/NounsDAOV3Proposals.sol

141    bytes32 public constant DOMAIN_TYPEHASH =
142        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


144    bytes32 public constant PROPOSAL_TYPEHASH =
145        keccak256(
146            'Proposal(address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'
147        );



149    bytes32 public constant UPDATE_PROPOSAL_TYPEHASH =
150        keccak256(
151            'UpdateProposal(uint256 proposalId,address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'
152        );
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/

```solidity
File: governance/NounsDAOLogicV1Fork.sol

146    bytes32 public constant DOMAIN_TYPEHASH =
147        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');



150    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L70

```solidity
File: governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

70    bytes32 public constant DOMAIN_TYPEHASH =
71        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');



74    bytes32 public constant DELEGATION_TYPEHASH =
75        keccak256('Delegation(address delegatee,uint256 nonce,uint256 expiry)');
```

## [G-03] Using fixed bytes is cheaper than using string

Bytes constants are more efficient than string constants

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is
cheaper in solidity.

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

```solidity
File: governance/NounsDAOLogicV2.sol

59    string public constant name = 'Nouns DAO';

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L44

```solidity
File: governance/NounsDAOV3Votes.sol

44    string public constant name = 'Nouns DAO';

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L48

```solidity
File: governance/fork/newdao/NounsAuctionHouseFork.sol

48    string public constant NAME = 'NounsAuctionHouseFork';

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L116

```solidity
File: governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

116    string public constant name = 'Nouns DAO';

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L46

```solidity
File: governance/fork/newdao/token/NounsTokenFork.sol

46    string public constant NAME = 'NounsTokenFork';


85    string private _contractURIHash = 'QmZi1n79FqWt2tTLwCqiy6nLM6xLGRsEPQ5JmReJQKNNzX';

```

## [G-04] state variables should be cached in stack variables rather than re-reading them from storage

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

```solidity
File: governance/fork/newdao/NounsAuctionHouseFork.sol

// @audit  timeBuffer  read twice from the storage
141        bool extended = _auction.endTime - block.timestamp < timeBuffer;
142        if (extended) {
143            auction.endTime = _auction.endTime = block.timestamp + timeBuffer;
144        }
```

## [G-05] stack variable used as a cheaper cache for a state variable is only used once

when a state variable is only accessed once, it is more cost-effective to directly use the state variable in that particular instance, thus avoiding the unnecessary gas expenditure that would result from assigning it to a stack variable.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

```solidity
File: governance/NounsDAOLogicV2.sol

59    string public constant name = 'Nouns DAO';


92    uint256 public constant proposalMaxOperations = 10; // 10 actions


95    uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;


98    uint256 public constant REFUND_BASE_GAS = 36000;


101    uint256 public constant MAX_REFUND_GAS_USED = 200_000;


104    uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;



107    bytes32 public constant DOMAIN_TYPEHASH =
        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


111    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

```diff --git a/NounsDAOLogicV2.org.sol b/NounsDAOLogicV2.sol
index 64b1043..8638429 100644
--- a/NounsDAOLogicV2.org.sol
+++ b/NounsDAOLogicV2.sol
@@ -55,9 +55,7 @@ pragma solidity ^0.8.6;
 import './NounsDAOInterfaces.sol';

 contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
-    /// @notice The name of this contract
-    string public constant name = 'Nouns DAO';

     /// @notice The minimum setable proposal threshold
     uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%

@@ -88,27 +86,6 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
     /// @notice The maximum setable quorum votes basis points
     uint256 public constant MAX_QUORUM_VOTES_BPS = 2_000; // 2,000 basis points or 20%

-    /// @notice The maximum number of actions that can be included in a proposal
-    uint256 public constant proposalMaxOperations = 10; // 10 actions

-    /// @notice The maximum priority fee used to cap gas refunds in `castRefundableVote`
-    uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;

-    /// @notice The vote refund gas overhead, including 7K for ETH transfer and 29K for general transaction overhead
-    uint256 public constant REFUND_BASE_GAS = 36000;

-    /// @notice The maximum gas units the DAO will refund voters on; supports about 9,190 characters
-    uint256 public constant MAX_REFUND_GAS_USED = 200_000;

-    /// @notice The maximum basefee the DAO will refund voters on
-    uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;

-    /// @notice The EIP-712 typehash for the contract's domain
-    bytes32 public constant DOMAIN_TYPEHASH =
-        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

-    /// @notice The EIP-712 typehash for the ballot struct used by the contract
-    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

     /// @dev Introduced these errors to reduce contract size, to avoid deployment failure
     error AdminOnly();
@@ -201,6 +178,9 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         bytes[] memory calldatas,
         string memory description
     ) public returns (uint256) {
+        /// @notice The maximum number of actions that can be included in a proposal
+        uint256 public constant proposalMaxOperations = 10; // 10 actions

         ProposalTemp memory temp;

         temp.totalSupply = nouns.totalSupply();
@@ -591,6 +571,14 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         bytes32 r,
         bytes32 s
     ) external {
+        /// @notice The name of this contract
+        string public constant name = 'Nouns DAO';

+        /// @notice The EIP-712 typehash for the contract's domain
+        bytes32 public constant DOMAIN_TYPEHASH =
+            keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

+        /// @notice The EIP-712 typehash for the ballot struct used by the contract
+        bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

         bytes32 domainSeparator = keccak256(
             abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), getChainIdInternal(), address(this))
         );
@@ -1036,6 +1024,15 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             if (balance == 0) {
                 return;
             }
+            /// @notice The maximum priority fee used to cap gas refunds in `castRefundableVote`
+            uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;

+            /// @notice The vote refund gas overhead, including 7K for ETH transfer and 29K for general transaction overhead
+            uint256 public constant REFUND_BASE_GAS = 36000;

+            /// @notice The maximum gas units the DAO will refund voters on; supports about 9,190 characters
+            uint256 public constant MAX_REFUND_GAS_USED = 200_000;

+            /// @notice The maximum basefee the DAO will refund voters on
+            uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;
+
             uint256 basefee = min(block.basefee, MAX_REFUND_BASE_FEE);
             uint256 gasPrice = min(tx.gasprice, basefee + MAX_REFUND_PRIORITY_FEE);
             uint256 gasUsed = min(startGas - gasleft() + REFUND_BASE_GAS, MAX_REFUND_GAS_USED);
(END)
         );
@@ -1036,6 +1024,15 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             if (balance == 0) {
                 return;
             }
+            /// @notice The maximum priority fee used to cap gas refunds in `castRefundableVote`
+            uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;

+            /// @notice The vote refund gas overhead, including 7K for ETH transfer and 29K for general transaction overhead
+            uint256 public constant REFUND_BASE_GAS = 36000;

+            /// @notice The maximum gas units the DAO will refund voters on; supports about 9,190 characters
+            uint256 public constant MAX_REFUND_GAS_USED = 200_000;

+            /// @notice The maximum basefee the DAO will refund voters on
+            uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;

             uint256 basefee = min(block.basefee, MAX_REFUND_BASE_FEE);
             uint256 gasPrice = min(tx.gasprice, basefee + MAX_REFUND_PRIORITY_FEE);
             uint256 gasUsed = min(startGas - gasleft() + REFUND_BASE_GAS, MAX_REFUND_GAS_USED);



         );
@@ -1036,6 +1024,15 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             if (balance == 0) {
                 return;
             }
+            /// @notice The maximum priority fee used to cap gas refunds in `castRefundableVote`
+            uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;

+            /// @notice The vote refund gas overhead, including 7K for ETH transfer and 29K for general transaction overhead
+            uint256 public constant REFUND_BASE_GAS = 36000;

+            /// @notice The maximum gas units the DAO will refund voters on; supports about 9,190 characters
+            uint256 public constant MAX_REFUND_GAS_USED = 200_000;

+            /// @notice The maximum basefee the DAO will refund voters on
+            uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;
+
             uint256 basefee = min(block.basefee, MAX_REFUND_BASE_FEE);
             uint256 gasPrice = min(tx.gasprice, basefee + MAX_REFUND_PRIORITY_FEE);
             uint256 gasUsed = min(startGas - gasleft() + REFUND_BASE_GAS, MAX_REFUND_GAS_USED);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol

```solidity
File: governance/NounsDAOV3Votes.sol

44    string public constant name = 'Nouns DAO';


47    bytes32 public constant DOMAIN_TYPEHASH =
48        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


51    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');


54    uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;


57    uint256 public constant REFUND_BASE_GAS = 36000;


60    uint256 public constant MAX_REFUND_GAS_USED = 200_000;


63    uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;

```

```diff
diff --git a/NounsDAOV3Votes.org.sol b/NounsDAOV3Votes.sol
index fc2b211..a755d72 100644
--- a/NounsDAOV3Votes.org.sol
+++ b/NounsDAOV3Votes.sol
@@ -40,27 +40,11 @@ library NounsDAOV3Votes {
     /// @notice Emitted when a proposal is set to have an objection period
     event ProposalObjectionPeriodSet(uint256 indexed id, uint256 objectionPeriodEndBlock);

-    /// @notice The name of this contract
-    string public constant name = 'Nouns DAO';

-    /// @notice The EIP-712 typehash for the contract's domain
-    bytes32 public constant DOMAIN_TYPEHASH =
-        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

-    /// @notice The EIP-712 typehash for the ballot struct used by the contract
-    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

-    /// @notice The maximum priority fee used to cap gas refunds in `castRefundableVote`
-    uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;

-    /// @notice The vote refund gas overhead, including 7K for ETH transfer and 29K for general transaction overhead
-    uint256 public constant REFUND_BASE_GAS = 36000;

-    /// @notice The maximum gas units the DAO will refund voters on; supports about 9,190 characters
-    uint256 public constant MAX_REFUND_GAS_USED = 200_000;

-    /// @notice The maximum basefee the DAO will refund voters on
-    uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;

     /**
      * @notice Cast a vote for a proposal
@@ -161,9 +145,21 @@ library NounsDAOV3Votes {
         bytes32 r,
         bytes32 s
     ) external {
+        /// @notice The name of this contract
+        string public constant name = 'Nouns DAO';

+        /// @notice The EIP-712 typehash for the contract's domain
+        bytes32 public constant DOMAIN_TYPEHASH =
+            keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

         bytes32 domainSeparator = keccak256(
             abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), block.chainid, address(this))
         );

+        /// @notice The EIP-712 typehash for the ballot struct used by the contract
+        bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

         bytes32 structHash = keccak256(abi.encode(BALLOT_TYPEHASH, proposalId, support));
         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));
         address signatory = ecrecover(digest, v, r, s);
@@ -298,6 +294,19 @@ library NounsDAOV3Votes {
             if (balance == 0) {
                 return;
             }
+            /// @notice The maximum priority fee used to cap gas refunds in `castRefundableVote`
+            uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;

+            /// @notice The maximum basefee the DAO will refund voters on
+            uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;


+            /// @notice The vote refund gas overhead, including 7K for ETH transfer and 29K for general transaction overhead
+            uint256 public constant REFUND_BASE_GAS = 36000;


+            /// @notice The maximum gas units the DAO will refund voters on; supports about 9,190 characters
+            uint256 public constant MAX_REFUND_GAS_USED = 200_000;

             uint256 basefee = min(block.basefee, MAX_REFUND_BASE_FEE);
             uint256 gasPrice = min(tx.gasprice, basefee + MAX_REFUND_PRIORITY_FEE);
             uint256 gasUsed = min(startGas - gasleft() + REFUND_BASE_GAS, MAX_REFUND_GAS_USED);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

```solidity
File: governance/NounsDAOV3Admin.sol


112    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%


115    uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%


118    uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;

121    uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;


124    uint256 public constant MIN_VOTING_DELAY_BLOCKS = 1;


127    uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;


139    uint256 public constant MAX_FORK_PERIOD = 14 days;


142    uint256 public constant MIN_FORK_PERIOD = 2 days;


145    uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;


148    uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;

```

```diff
diff --git a/NounsDAOV3Admin.org.sol b/NounsDAOV3Admin.sol
index a861b6d..4b05d8e 100644
--- a/NounsDAOV3Admin.org.sol
+++ b/NounsDAOV3Admin.sol
@@ -108,23 +108,7 @@ library NounsDAOV3Admin {
     /// @notice Emitted when the main timelock, timelockV1 and admin are set
     event TimelocksAndAdminSet(address timelock, address timelockV1, address admin);

-    /// @notice The minimum setable proposal threshold
-    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%

-    /// @notice The maximum setable proposal threshold
-    uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%

-    /// @notice The minimum setable voting period in blocks
-    uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;

-    /// @notice The max setable voting period in blocks
-    uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;

-    /// @notice The min setable voting delay in blocks
-    uint256 public constant MIN_VOTING_DELAY_BLOCKS = 1;

-    /// @notice The max setable voting delay in blocks
-    uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;

     /// @notice The lower bound of minimum quorum votes basis points
     uint256 public constant MIN_QUORUM_VOTES_BPS_LOWER_BOUND = 200; // 200 basis points or 2%
@@ -135,17 +119,6 @@ library NounsDAOV3Admin {
     /// @notice The upper bound of maximum quorum votes basis points
     uint256 public constant MAX_QUORUM_VOTES_BPS_UPPER_BOUND = 6_000; // 6,000 basis points or 60%

-    /// @notice Upper bound for forking period. If forking period is too high it can block proposals for too long.
-    uint256 public constant MAX_FORK_PERIOD = 14 days;

-    /// @notice Lower bound for forking period
-    uint256 public constant MIN_FORK_PERIOD = 2 days;

-    /// @notice Upper bound for objection period duration in blocks.
-    uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;

-    /// @notice Upper bound for proposal updatable period duration in blocks.
-    uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;

     modifier onlyAdmin(NounsDAOStorageV3.StorageV3 storage ds) {
         if (msg.sender != ds.admin) {
@@ -160,6 +133,13 @@ library NounsDAOV3Admin {
      * @param newVotingDelay new voting delay, in blocks
      */
     function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external onlyAdmin(ds) {

+        /// @notice The min setable voting delay in blocks
+        uint256 public constant MIN_VOTING_DELAY_BLOCKS = 1;

+        /// @notice The max setable voting delay in blocks
+        uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;

         require(
             newVotingDelay >= MIN_VOTING_DELAY_BLOCKS && newVotingDelay <= MAX_VOTING_DELAY_BLOCKS,
             'NounsDAO::_setVotingDelay: invalid voting delay'
@@ -175,6 +155,12 @@ library NounsDAOV3Admin {
      * @param newVotingPeriod new voting period, in blocks
      */
     function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external onlyAdmin(ds) {
+        /// @notice The minimum setable voting period in blocks
+        uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;

+        /// @notice The max setable voting period in blocks
+        uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;

         require(
             newVotingPeriod >= MIN_VOTING_PERIOD_BLOCKS && newVotingPeriod <= MAX_VOTING_PERIOD_BLOCKS,
             'NounsDAO::_setVotingPeriod: invalid voting period'
@@ -194,6 +180,15 @@ library NounsDAOV3Admin {
         external
         onlyAdmin(ds)
     {
+        /// @notice The minimum setable proposal threshold
+        uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%

+        /// @notice The maximum setable proposal threshold
+        uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%

         require(
             newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
                 newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
@@ -213,6 +208,9 @@ library NounsDAOV3Admin {
         NounsDAOStorageV3.StorageV3 storage ds,
         uint32 newObjectionPeriodDurationInBlocks
     ) external onlyAdmin(ds) {
+        /// @notice Upper bound for objection period duration in blocks.
+        uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;

         if (newObjectionPeriodDurationInBlocks > MAX_OBJECTION_PERIOD_BLOCKS)
             revert InvalidObjectionPeriodDurationInBlocks();

@@ -244,6 +242,9 @@ library NounsDAOV3Admin {
         NounsDAOStorageV3.StorageV3 storage ds,
         uint32 newProposalUpdatablePeriodInBlocks
     ) external onlyAdmin(ds) {
+        /// @notice Upper bound for proposal updatable period duration in blocks.
+        uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;

         if (newProposalUpdatablePeriodInBlocks > MAX_UPDATABLE_PERIOD_BLOCKS)
             revert InvalidProposalUpdatablePeriodInBlocks();

@@ -531,6 +532,13 @@ library NounsDAOV3Admin {
     }

     function _setForkPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newForkPeriod) external onlyAdmin(ds) {
+        /// @notice Upper bound for forking period. If forking period is too high it can block proposals for too long.
+        uint256 public constant MAX_FORK_PERIOD = 14 days;

+        /// @notice Lower bound for forking period
+        uint256 public constant MIN_FORK_PERIOD = 2 days;

         if (newForkPeriod > MAX_FORK_PERIOD) {
             revert ForkPeriodTooLong();
         }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

```solidity
File: governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

116    string public constant name = 'Nouns DAO';

119    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%


122    uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%


125    uint256 public constant MIN_VOTING_PERIOD = 7_200; // 24 hours


128    uint256 public constant MAX_VOTING_PERIOD = 100_800; // 2 weeks


131    uint256 public constant MIN_VOTING_DELAY = 1;


134    uint256 public constant MAX_VOTING_DELAY = 100_800; // 2 weeks


137    uint256 public constant MIN_QUORUM_VOTES_BPS = 200; // 200 basis points or 2%


140    uint256 public constant MAX_QUORUM_VOTES_BPS = 2_000; // 2,000 basis points or 20%


143    uint256 public constant proposalMaxOperations = 10; // 10 actions


146    bytes32 public constant DOMAIN_TYPEHASH =
147          keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


150    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

## [G-06] Can make the state variable outside the loop to save gas

If a state variable is only used within a loop and its value doesn't need to persist across multiple iterations or be accessed outside of the loop, it is generally more gas-efficient to declare the variable as a local variable inside the loop rather than as a state variable.

This can lead to excessive storage reads To improve gas efficiency, declare separate local variables for both outside the loop, assign their values to these variables, and then use the local variables within the loop.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L148-L152

```solidity
File: governance/fork/NounsDAOForkEscrow.sol

148        for (uint256 i = 0; i < tokenIds.length; i++) {
149            if (currentOwnerOf(tokenIds[i]) != dao) revert NotOwner();
150
151            nounsToken.transferFrom(address(this), to, tokenIds[i]);
152        }
```

```diff
diff --git a/NounsDAOForkEscrow.org.sol b/NounsDAOForkEscrow.sol
index a5440b5..e011aff 100644
--- a/NounsDAOForkEscrow.org.sol
+++ b/NounsDAOForkEscrow.sol
@@ -1,6 +1,8 @@
     function withdrawTokens(uint256[] calldata tokenIds, address to) external onlyDAO {
+        address _dao = dao

         for (uint256 i = 0; i < tokenIds.length; i++) {
-            if (currentOwnerOf(tokenIds[i]) != dao) revert NotOwner();
+            if (currentOwnerOf(tokenIds[i]) != _dao) revert NotOwner();

             nounsToken.transferFrom(address(this), to, tokenIds[i]);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L150


```solidity
file: contracts/governance/fork/newdao/token/NounsTokenFork.sol

150            uint256 nounId = tokenIds[i];

173            uint256 nounId = tokenIds[i];

```

## [G-07] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L198-L201

```solidity
File: governance/NounsDAOLogicV2.sol

197    function propose(
198        address[] memory targets,
199        uint256[] memory values,
200        string[] memory signatures,
201        bytes[] memory calldatas,
202        string memory description
203    ) public returns (uint256) {
```

```diff
diff --git a/NounsDAOLogicV2.org.sol b/NounsDAOLogicV2.sol
index 74f6ee7..4f399de 100644
--- a/NounsDAOLogicV2.org.sol
+++ b/NounsDAOLogicV2.sol
@@ -1,7 +1,7 @@
     function propose(
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory description
     ) public returns (uint256) {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L189-L195

```solidity
File: governance/NounsDAOLogicV3.sol

189    function propose(
190        address[] memory targets,
191        uint256[] memory values,
192        string[] memory signatures,
193        bytes[] memory calldatas,
194        string memory description
195   ) public returns (uint256) {


210    function proposeOnTimelockV1(
211        address[] memory targets,
212        uint256[] memory values,
213        string[] memory signatures,
214        bytes[] memory calldatas,
215        string memory description
216    ) public returns (uint256) {


236    function proposeBySigs(
237        ProposerSignature[] memory proposerSignatures,
238        address[] memory targets,
239        uint256[] memory values,
240        string[] memory signatures,
241        bytes[] memory calldatas,
242        string memory description
243    ) external returns (uint256) {


278    function updateProposal(
279        uint256 proposalId,
280        address[] memory targets,
281        uint256[] memory values,
282        string[] memory signatures,
283        bytes[] memory calldatas,
284        string memory description,
285        string memory updateMessage
286    ) external {


313    function updateProposalTransactions(
314        uint256 proposalId,
315        address[] memory targets,
316        uint256[] memory values,
317        string[] memory signatures,
318        bytes[] memory calldatas,
319        string memory updateMessage
320    ) external {



338    function updateProposalBySigs(
339        uint256 proposalId,
340        ProposerSignature[] memory proposerSignatures,
341        address[] memory targets,
342        uint256[] memory values,
343        string[] memory signatures,
344        bytes[] memory calldatas,
345        string memory description,
346        string memory updateMessage
347    ) external {
```

```diff
diff --git a/NounsDAOLogicV3.org.sol b/NounsDAOLogicV3.sol
index 7168476..da62c4f 100644
--- a/NounsDAOLogicV3.org.sol
+++ b/NounsDAOLogicV3.sol
@@ -187,10 +187,10 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      * @return uint256 Proposal id of new proposal
      */
     function propose(
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory description
     ) public returns (uint256) {
         return ds.propose(NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas), description);
@@ -208,10 +208,10 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      * @return uint256 Proposal id of new proposal
      */
     function proposeOnTimelockV1(
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory description
     ) public returns (uint256) {
         return
@@ -234,11 +234,11 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      * @return uint256 Proposal id of new proposal
      */
     function proposeBySigs(
-        ProposerSignature[] memory proposerSignatures,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        ProposerSignature[] calldata proposerSignatures,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory description
     ) external returns (uint256) {
         return
@@ -277,10 +277,10 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      */
     function updateProposal(
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory description,
         string memory updateMessage
     ) external {
@@ -312,10 +312,10 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      */
     function updateProposalTransactions(
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory updateMessage
     ) external {
         ds.updateProposalTransactions(proposalId, targets, values, signatures, calldatas, updateMessage);
@@ -337,11 +337,11 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      */
     function updateProposalBySigs(
         uint256 proposalId,
-        ProposerSignature[] memory proposerSignatures,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        ProposerSignature[] calldata proposerSignatures,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory description,
         string memory updateMessage
     ) external {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L288-L297

```solidity
File: governance/NounsDAOV3Proposals.sol

222        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures,



288    function updateProposal(
289        NounsDAOStorageV3.StorageV3 storage ds,
290        uint256 proposalId,
291        address[] memory targets,
292        uint256[] memory values,
293        string[] memory signatures,
294        bytes[] memory calldatas,
295        string memory description,
296        string memory updateMessage
297    ) external {


321    function updateProposalTransactions(
322        NounsDAOStorageV3.StorageV3 storage ds,
323        uint256 proposalId,
324        address[] memory targets,
325        uint256[] memory values,
326        string[] memory signatures,
327        string memory updateMessage
328    ) external {


335    function updateProposalTransactionsInternal(
336        NounsDAOStorageV3.StorageV3 storage ds,
337        uint256 proposalId,
338        address[] memory targets,
339        uint256[] memory values,
340        string[] memory signatures,
341        bytes[] memory calldatas
342    ) internal {

386        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures,


816        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures,


919        address[] memory signers,

```

```diff
diff --git a/org.sol b/not.sol
index 4d0d3c4..fabb150 100644
--- a/org.sol
+++ b/not.sol
@@ -219,7 +219,7 @@ library NounsDAOV3Proposals {
      */
     function proposeBySigs(
         NounsDAOStorageV3.StorageV3 storage ds,
-        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures,
+        NounsDAOStorageV3.ProposerSignature[] calldata proposerSignatures,
         ProposalTxs memory txs,
         string memory description
     ) external returns (uint256) {
@@ -288,10 +288,10 @@ library NounsDAOV3Proposals {
     function updateProposal(
         NounsDAOStorageV3.StorageV3 storage ds,
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory description,
         string memory updateMessage
     ) external {
@@ -321,10 +321,10 @@ library NounsDAOV3Proposals {
     function updateProposalTransactions(
         NounsDAOStorageV3.StorageV3 storage ds,
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory updateMessage
     ) external {
         updateProposalTransactionsInternal(ds, proposalId, targets, values, signatures, calldatas);
@@ -335,10 +335,10 @@ library NounsDAOV3Proposals {
     function updateProposalTransactionsInternal(
         NounsDAOStorageV3.StorageV3 storage ds,
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas
     ) internal {
         checkProposalTxs(ProposalTxs(targets, values, signatures, calldatas));

@@ -383,7 +383,7 @@ library NounsDAOV3Proposals {
     function updateProposalBySigs(
         NounsDAOStorageV3.StorageV3 storage ds,
         uint256 proposalId,
-        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures,
+        NounsDAOStorageV3.ProposerSignature[] calldata proposerSignatures,
         ProposalTxs memory txs,
         string memory description,
         string memory updateMessage
@@ -678,10 +678,10 @@ library NounsDAOV3Proposals {


         NounsDAOStorageV3.Proposal storage p = ds._proposals[proposalId];
@@ -813,7 +813,7 @@ library NounsDAOV3Proposals {
      */
     function verifySignersCanBackThisProposalAndCountTheirVotes(
         NounsDAOStorageV3.StorageV3 storage ds,
-        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures,
+        NounsDAOStorageV3.ProposerSignature[] calldata proposerSignatures,
         ProposalTxs memory txs,
         string memory description,
         uint256 proposalId
@@ -916,7 +916,7 @@ library NounsDAOV3Proposals {

     function emitNewPropEvents(
         NounsDAOStorageV3.Proposal storage newProposal,
-        address[] memory signers,
+        address[] calldata signers,
         uint256 minQuorumVotes,
         ProposalTxs memory txs,
         string memory description

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

```solidity
File:  governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

172        address[] memory erc20TokensToIncludeInQuit_,

247    function isAddressIn(address a, address[] memory addresses) internal pure returns (bool) {


272    function propose(
273        address[] memory targets,
274        uint256[] memory values,
275        string[] memory signatures,
276        bytes[] memory calldatas,
277        string memory description
278    ) public returns (uint256) {
```

```diff
diff --git a/NounsDAOLogicV1Fork.org.sol b/NounsDAOLogicV1Fork.sol
index b3e0421..bc76955 100644
--- a/NounsDAOLogicV1Fork.org.sol
+++ b/NounsDAOLogicV1Fork.sol
@@ -169,7 +169,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         uint256 votingDelay_,
         uint256 proposalThresholdBPS_,
         uint256 quorumVotesBPS_,
-        address[] memory erc20TokensToIncludeInQuit_,
+        address[] calldata erc20TokensToIncludeInQuit_,
         uint256 delayedGovernanceExpirationTimestamp_
     ) public virtual {
         __ReentrancyGuard_init_unchained();
@@ -203,7 +203,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         quitInternal(tokenIds, erc20TokensToIncludeInQuit);
     }

-    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
+    function quit(uint256[] calldata tokenIds, address[] calldata erc20TokensToInclude) external nonReentrant {
         // check that erc20TokensToInclude is a subset of `erc20TokensToIncludeInQuit`
         address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
@@ -215,7 +215,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         quitInternal(tokenIds, erc20TokensToInclude);
     }

-    function quitInternal(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) internal {
+    function quitInternal(uint256[] calldata tokenIds, address[] calldata erc20TokensToInclude) internal {
         checkGovernanceActive();

         uint256 totalSupply = adjustedTotalSupply();
@@ -244,7 +244,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         emit Quit(msg.sender, tokenIds);
     }

-    function isAddressIn(address a, address[] memory addresses) internal pure returns (bool) {
+    function isAddressIn(address a, address[] calldata addresses) internal pure returns (bool) {
         for (uint256 i = 0; i < addresses.length; i++) {
             if (addresses[i] == a) return true;
         }
@@ -270,10 +270,10 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
      * @return Proposal id of new proposal
      */
     function propose(
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
         string memory description
     ) public returns (uint256) {
         checkGovernanceActive();
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L62

```solidity
File: script/ProposeDAOV3UpgradeMainnet.s.sol

62        address[] memory erc20TokensToIncludeInFork,

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L70

```solidity
File: script/ProposeDAOV3UpgradeTestnet.s.sol

70        address[] memory erc20TokensToIncludeInFork,

```

## [G-08] Before some functions, we should check some variables ro possible gas save

Before transfer, we should check for balance being 0 so the function doesn't run when its not gonna do anything:

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L34

```solidity
File: utils/ERC20Transferer.sol


34        IERC20(token).transferFrom(msg.sender, to, balance);// @audit check balance

```

```diff

    function transferEntireBalance(address token, address to) external returns (uint256) {
        uint256 balance = IERC20(token).balanceOf(msg.sender);
+       if (balance != 0 ){
        IERC20(token).transferFrom(msg.sender, to, balance);
+       }

```

## [G-09] Instead of calculating a state variable with keccak256() every time the contract is made pre calculate them before and only give the result to a constant

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

```solidity
File: governance/NounsDAOLogicV2.sol

108        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


111    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

```diff
diff --git a/NounsDAOLogicV2.org.sol b/NounsDAOLogicV2.sol
index fb1f6ea..64e4a94 100644
--- a/NounsDAOLogicV2.org.sol
+++ b/NounsDAOLogicV2.sol
@@ -1,6 +1,5 @@
     /// @notice The EIP-712 typehash for the contract's domain
-    bytes32 public constant DOMAIN_TYPEHASH =
-        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');
+    bytes32 public constant DOMAIN_TYPEHASH = 0xabcdef1234567890; // Precomputed value of keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)')

     /// @notice The EIP-712 typehash for the ballot struct used by the contract
-    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
\ No newline at end of file
+    bytes32 public constant BALLOT_TYPEHASH = 0x1234567890abcdef; // Precomputed value of keccak256('Ballot(uint256 proposalId,uint8 support)')

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L48

```solidity
File: governance/NounsDAOV3Votes.sol

48        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


51    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L142

```solidity
File: governance/NounsDAOV3Proposals.sol

141     bytes32 public constant DOMAIN_TYPEHASH =
142        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


144    bytes32 public constant PROPOSAL_TYPEHASH =
146        keccak256(
147            'Proposal(address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'
148        );



149    bytes32 public constant UPDATE_PROPOSAL_TYPEHASH =
150        keccak256(
151            'UpdateProposal(uint256 proposalId,address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'
152        );
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L146-L147

```solidity
File: governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

146    bytes32 public constant DOMAIN_TYPEHASH =
147        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


150    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L70

```solidity
File: governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

70    bytes32 public constant DOMAIN_TYPEHASH =
71        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


74   bytes32 public constant DELEGATION_TYPEHASH =
75        keccak256('Delegation(address delegatee,uint256 nonce,uint256 expiry)');

```

## [G-10] Avoid storage keyword during modifiers

The problem with using modifiers with storage variables is that these accesses will likely have to be obtained multiple times, to pass it to the modifier, and to read it into the method code.

it is recommended to use a method instead of a modifier

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L150-L155

```solidity
File: governance/NounsDAOV3Admin.sol


150    modifier onlyAdmin(NounsDAOStorageV3.StorageV3 storage ds) {
151        if (msg.sender != ds.admin) {
152            revert AdminOnly();
153        }
154        _;
155    }
```

Using a method instead of a modifier to access storage variables multiple times can help improve efficiency and save gas. By encapsulating the access logic within a method, you can avoid redundant storage variable accesses .

In the provided code snippet from the NounsDAOV3Admin.sol contract, the onlyAdmin modifier is defined to check if the msg.sender is the admin address stored in the NounsDAOStorageV3 contract's storage.

the modifier can be replaced with a method that performs the same check. Here's an example of how the modifier can be converted into a method:

```diff
diff --git a/NounsDAOV3Admin.org.sol b/NounsDAOV3Admin.sol
index a861b6d..80bdd1e 100644
--- a/NounsDAOV3Admin.org.sol
+++ b/NounsDAOV3Admin.sol
@@ -147,19 +147,19 @@ library NounsDAOV3Admin {
     /// @notice Upper bound for proposal updatable period duration in blocks.
     uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;

-    modifier onlyAdmin(NounsDAOStorageV3.StorageV3 storage ds) {
-        if (msg.sender != ds.admin) {
-            revert AdminOnly();
-        }
-        _;
-    }
+    function requireAdmin(NounsDAOStorageV3.StorageV3 storage ds) internal view {
+      if (msg.sender != ds.admin) {
+         revert AdminOnly();
+         }
+     }

     /**
      * @notice Admin function for setting the voting delay. Best to set voting delay to at least a few days, to give
      * voters time to make sense of proposals, e.g. 21,600 blocks which should be at least 3 days.
      * @param newVotingDelay new voting delay, in blocks
      */
-    function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external onlyAdmin(ds) {
+    function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external  {
+        requireAdmin(ds);
         require(
             newVotingDelay >= MIN_VOTING_DELAY_BLOCKS && newVotingDelay <= MAX_VOTING_DELAY_BLOCKS,
             'NounsDAO::_setVotingDelay: invalid voting delay'
@@ -174,7 +174,8 @@ library NounsDAOV3Admin {
      * @notice Admin function for setting the voting period
      * @param newVotingPeriod new voting period, in blocks
      */
-    function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external onlyAdmin(ds) {
+    function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external  {
+        requireAdmin(ds);
         require(
             newVotingPeriod >= MIN_VOTING_PERIOD_BLOCKS && newVotingPeriod <= MAX_VOTING_PERIOD_BLOCKS,
             'NounsDAO::_setVotingPeriod: invalid voting period'
@@ -192,8 +193,9 @@ library NounsDAOV3Admin {
      */
     function _setProposalThresholdBPS(NounsDAOStorageV3.StorageV3 storage ds, uint256 newProposalThresholdBPS)
         external
-        onlyAdmin(ds)
+
     {
+        requireAdmin(ds);
         require(
             newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
                 newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
@@ -212,7 +214,8 @@ library NounsDAOV3Admin {
     function _setObjectionPeriodDurationInBlocks(
         NounsDAOStorageV3.StorageV3 storage ds,
         uint32 newObjectionPeriodDurationInBlocks
-    ) external onlyAdmin(ds) {
+    ) external  {
+        requireAdmin(ds);
         if (newObjectionPeriodDurationInBlocks > MAX_OBJECTION_PERIOD_BLOCKS)
             revert InvalidObjectionPeriodDurationInBlocks();

@@ -228,8 +231,8 @@ library NounsDAOV3Admin {
      */
     function _setLastMinuteWindowInBlocks(NounsDAOStorageV3.StorageV3 storage ds, uint32 newLastMinuteWindowInBlocks)
         external
-        onlyAdmin(ds)
     {
+        requireAdmin(ds);
         uint32 oldLastMinuteWindowInBlocks = ds.lastMinuteWindowInBlocks;
         ds.lastMinuteWindowInBlocks = newLastMinuteWindowInBlocks;

@@ -243,7 +246,8 @@ library NounsDAOV3Admin {
     function _setProposalUpdatablePeriodInBlocks(
         NounsDAOStorageV3.StorageV3 storage ds,
         uint32 newProposalUpdatablePeriodInBlocks
-    ) external onlyAdmin(ds) {
+    ) external  {
+        requireAdmin(ds);
         if (newProposalUpdatablePeriodInBlocks > MAX_UPDATABLE_PERIOD_BLOCKS)
             revert InvalidProposalUpdatablePeriodInBlocks();

@@ -258,7 +262,9 @@ library NounsDAOV3Admin {
      * @dev Admin function to begin change of admin. The newPendingAdmin must call `_acceptAdmin` to finalize the transfer.
      * @param newPendingAdmin New pending admin.
      */
-    function _setPendingAdmin(NounsDAOStorageV3.StorageV3 storage ds, address newPendingAdmin) external onlyAdmin(ds) {
+    function _setPendingAdmin(NounsDAOStorageV3.StorageV3 storage ds, address newPendingAdmin) external {

+        requireAdmin(ds);
         // Save current value, if any, for inclusion in log
         address oldPendingAdmin = ds.pendingAdmin;

@@ -350,8 +356,10 @@ library NounsDAOV3Admin {
      */
     function _setMinQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMinQuorumVotesBPS)
         external
-        onlyAdmin(ds)
+
     {
+        requireAdmin(ds);
         NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);

         require(
@@ -380,8 +388,9 @@ library NounsDAOV3Admin {
      */
     function _setMaxQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMaxQuorumVotesBPS)
         external
-        onlyAdmin(ds)
     {
+        requireAdmin(ds);
         NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);

         require(
@@ -407,8 +416,9 @@ library NounsDAOV3Admin {
      */
     function _setQuorumCoefficient(NounsDAOStorageV3.StorageV3 storage ds, uint32 newQuorumCoefficient)
         external
-        onlyAdmin(ds)
     {
+
+        requireAdmin(ds);
         NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);

         uint32 oldQuorumCoefficient = params.quorumCoefficient;
@@ -434,7 +444,8 @@ library NounsDAOV3Admin {
         uint16 newMaxQuorumVotesBPS,
         uint32 newQuorumCoefficient
-    ) public onlyAdmin(ds) {
+    ) public  {
+        requireAdmin(ds);
         if (
             newMinQuorumVotesBPS < MIN_QUORUM_VOTES_BPS_LOWER_BOUND ||
             newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND
@@ -465,7 +476,8 @@ library NounsDAOV3Admin {
     /**
      * @notice Withdraws all the ETH in the contract. This is callable only by the admin (timelock).
      */
-    function _withdraw(NounsDAOStorageV3.StorageV3 storage ds) external onlyAdmin(ds) returns (uint256, bool) {
+    function _withdraw(NounsDAOStorageV3.StorageV3 storage ds) external returns (uint256, bool) {
+        requireAdmin(ds);
         uint256 amount = address(this).balance;
         (bool sent, ) = msg.sender.call{ value: amount }('');

@@ -479,7 +491,8 @@ library NounsDAOV3Admin {
      * instead of the proposal creation block.
      * Sets it to the next proposal id.
      */
-    function _setVoteSnapshotBlockSwitchProposalId(NounsDAOStorageV3.StorageV3 storage ds) external onlyAdmin(ds) {
+    function _setVoteSnapshotBlockSwitchProposalId(NounsDAOStorageV3.StorageV3 storage ds) external{
+        requireAdmin(ds);
         uint256 oldVoteSnapshotBlockSwitchProposalId = ds.voteSnapshotBlockSwitchProposalId;
         if (oldVoteSnapshotBlockSwitchProposalId > 0) {
             revert VoteSnapshotSwitchAlreadySet();
@@ -499,8 +512,8 @@ library NounsDAOV3Admin {
      */
     function _setForkDAODeployer(NounsDAOStorageV3.StorageV3 storage ds, address newForkDAODeployer)
         external
-        onlyAdmin(ds)
     {
+        requireAdmin(ds);
         address oldForkDAODeployer = address(ds.forkDAODeployer);
         ds.forkDAODeployer = IForkDAODeployer(newForkDAODeployer);

@@ -512,8 +525,8 @@ library NounsDAOV3Admin {
      */
     function _setErc20TokensToIncludeInFork(NounsDAOStorageV3.StorageV3 storage ds, address[] calldata erc20tokens)
         external
-        onlyAdmin(ds)
     {
+        requireAdmin(ds);
         checkForDuplicates(erc20tokens);

         emit ERC20TokensToIncludeInForkSet(ds.erc20TokensToIncludeInFork, erc20tokens);
@@ -524,13 +537,15 @@ library NounsDAOV3Admin {
     /**
      * @notice Admin function for setting the fork escrow contract
      */
-    function _setForkEscrow(NounsDAOStorageV3.StorageV3 storage ds, address newForkEscrow) external onlyAdmin(ds) {
+    function _setForkEscrow(NounsDAOStorageV3.StorageV3 storage ds, address newForkEscrow) external {
+        requireAdmin(ds);
         emit ForkEscrowSet(address(ds.forkEscrow), newForkEscrow);

-    function _setForkPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newForkPeriod) external onlyAdmin(ds) {
+    function _setForkPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newForkPeriod) external {
+        requireAdmin(ds);
         if (newForkPeriod > MAX_FORK_PERIOD) {

@@ -550,8 +565,8 @@ library NounsDAOV3Admin {
      */
     function _setForkThresholdBPS(NounsDAOStorageV3.StorageV3 storage ds, uint256 newForkThresholdBPS)
         external
-        onlyAdmin(ds)
     {
+        requireAdmin(ds);
         emit ForkThresholdSet(ds.forkThresholdBPS, newForkThresholdBPS);

@@ -568,7 +583,8 @@ library NounsDAOV3Admin {
         address timelock,
         address timelockV1,
         address admin
-    ) external onlyAdmin(ds) {
+    ) external {
+        requireAdmin(ds);
         ds.timelock = INounsDAOExecutorV2(timelock);

```

## [G-11] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

Did you know that the normal "(bool success, bytes memory returnData) = http://target.call()" automatically copies the return data to memory even if you omit the returnData variable? If a relayer executes transactions with such calls it can lead to a gas-griefing attack.

https://t.co/dHtRfZXEnq

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L824

```solidity
File: governance/NounsDAOLogicV2.sol

824        (bool sent, ) = msg.sender.call{ value: amount }('');


1043            (bool refundSent, ) = tx.origin.call{ value: refundAmount }('');

```

```diff
diff --git a/NounsDAOLogicV2.org.sol b/NounsDAOLogicV2.sol
index 1b13e1d..f8b6df7 100644
--- a/NounsDAOLogicV2.org.sol
+++ b/NounsDAOLogicV2.sol
@@ -1,5 +1,9 @@

         uint256 amount = address(this).balance;
-        (bool sent, ) = msg.sender.call{ value: amount }('');
+        bool success;
+        assembly {
+                success := call(gas(), msg.sender, amount, 0, 0)
+             }
+
        emit Withdraw(amount, sent);

@@ -1,4 +1,9 @@
             uint256 gasUsed = min(startGas - gasleft() + REFUND_BASE_GAS, MAX_REFUND_GAS_USED);
             uint256 refundAmount = min(gasPrice * gasUsed, balance);
-            (bool refundSent, ) = tx.origin.call{ value: refundAmount }('');
-            emit RefundableVote(tx.origin, refundAmount, refundSent);

+            bool refundSent;
+            assembly {
+                refundSent := call(gas(), tx.origin, refundAmount, 0, 0)
+             }
            emit RefundableVote(tx.origin, refundAmount, refundSent);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L305

```solidity
File: contracts/governance/NounsDAOV3Votes.sol

305            (bool refundSent, ) = msg.sender.call{ value: refundAmount }('');

```

```diff
diff --git a/NounsDAOV3Votes.org.sol b/NounsDAOV3Votes.sol
index 5625d1a..cf32942 100644
--- a/NounsDAOV3Votes.org.sol
+++ b/NounsDAOV3Votes.sol
@@ -1,4 +1,7 @@
             uint256 gasUsed = min(startGas - gasleft() + REFUND_BASE_GAS, MAX_REFUND_GAS_USED);
             uint256 refundAmount = min(gasPrice * gasUsed, balance);
-            (bool refundSent, ) = msg.sender.call{ value: refundAmount }('');
+            bool refundSent;
+            assembly {
+                refundSent := call(gas(), msg.sender, refundAmount, 0, 0)
+             }
             emit RefundableVote(msg.sender, refundAmount, refundSent);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L469-L472

```diff
File: contracts/governance/NounsDAOV3Admin.sol

469        uint256 amount = address(this).balance;
-470        (bool sent, ) = msg.sender.call{ value: amount }('');
+            bool sent;
+            assembly {
+                sent := call(gas(), msg.sender, amount, 0, 0)
+             }
472        emit Withdraw(amount, sent);
```

## [G-12] Use constants instead of type(uintx).max

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1079

```solidity
File: contracts/governance/NounsDAOLogicV2.sol

1079        require(n <= type(uint32).max, errorMessage);


1084        if (n > type(uint16).max) {

```

```diff
diff --git a/NounsDAOLogicV2.org.sol b/NounsDAOLogicV2.sol
index 9035eb8..4eec50c 100644
--- a/NounsDAOLogicV2.org.sol
+++ b/NounsDAOLogicV2.sol
@@ -1,12 +1,15 @@
+   uint32 constant MAX_UINT32 = type(uint32).max;
+   uint16 constant MAX_UINT16 = type(uint16).max;


    function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {
-        require(n <= type(uint32).max, errorMessage);
+        require(n <= MAX_UINT32, errorMessage);
         return uint32(n);
     }

     function safe16(uint256 n) internal pure returns (uint16) {
-        if (n > type(uint16).max) {
+        if (n > MAX_UINT16 {
             revert UnsafeUint16Cast();
         }
         return uint16(n);
     }

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L595

```solidity
File: contracts/governance/NounsDAOV3Admin.sol

595        require(n <= type(uint32).max, errorMessage);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L146

```solidity
File: contracts/governance/NounsDAOV3DynamicQuorum.sol

146        require(n <= type(uint32).max, errorMessage);


151        if (n > type(uint16).max) {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L123

```solidity
File:  script/ProposeDAOV3UpgradeTestnet.s.sol

123        calldatas[i] = abi.encode(erc20Transferer, type(uint256).max);

```

## [G-13] Use nested if and, avoid multiple check combinations &&

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1026

```solidity
File: contracts/governance/NounsDAOLogicV2.sol

1026        if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```

```diff

diff --git a/org.sol b/not.sol
index a397d0a..c783d56 100644
--- a/org.sol
+++ b/not.sol
@@ -1,4 +1,6 @@
         uint256 pos = quorumParamsCheckpoints.length;
-        if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
-            quorumParamsCheckpoints[pos - 1].params = params;
        } else {

+        if (pos > 0) {
+           if(quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber){
+                quorumParamsCheckpoints[pos - 1].params = params;
+            }
        } else {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L240-L251

```solidity
File: contracts/governance/NounsDAOV3Votes.sol

240      if (
241            // only for votes can trigger an objection period
242            // we're in the last minute window
243            isForVoteInLastMinuteWindow &&
244            // first part of the vote flip check
245            // separated from the second part to optimize gas
246            isDefeatedBefore &&
247            // haven't turn on objection yet
248            proposal.objectionPeriodEndBlock == 0 &&
249            // second part of the vote flip check
250            !ds.isDefeated(proposal)
251        ) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L585

```diff
File: contracts/governance/NounsDAOV3Admin.sol

-585        if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

+ if (pos > 0) {
+    if (ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
        ...
+    }
+}
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L381

```diff
File: governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

-381        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {

+ if (block.timestamp < delayedGovernanceExpirationTimestamp) {
+    if (nouns.remainingTokensToClaim() > 0) {
+
+    }
+}

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L227-L227C

```solidity
File:  governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

227        if (srcRep != dstRep && amount > 0) {

255        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {

```

## [G-14] Caching global variables is more expensive than using the actual variable

`use msg.sender instead of caching it`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L242-L243

```diff
File: contracts/governance/NounsDAOLogicV2.sol

-242        newProposal.proposer = msg.sender;

+260        latestProposalIds[msg.sender] = newProposal.id;

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L320

```diff
File: governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

-242        newProposal.proposer = msg.sender;

+260        latestProposalIds[msg.sender] = newProposal.id;


```

`use block.timestamp instead of caching it`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L214-L214

```diff
File: governance/fork/newdao/NounsAuctionHouseFork.sol

214            uint256 startTime = block.timestamp;
-215            uint256 endTime = startTime + duration;
+               uint256 endTime = block.timestamp + duration;

217            auction = Auction({
218                nounId: nounId,
219                amount: 0,
-220               startTime: block.timestamp,
+                  startTime: startTime,
221                endTime: endTime,
222                bidder: payable(0),
223                settled: false
224            });
225
-226            emit AuctionCreated(nounId, startTime, endTime);
+               emit AuctionCreated(nounId, block.timestamp, endTime);
```

## [G-15] Use assembly when getting a contract’s balance

You can use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L823

```solidity
File: contracts/governance/NounsDAOLogicV2.sol

823        uint256 amount = address(this).balance;

1035            uint256 balance = address(this).balance;

```

```diff
diff --git a/NounsDAOLogicV2.org.sol b/NounsDAOLogicV2.sol
index 71a7e42..dc7909d 100644
--- a/NounsDAOLogicV2.org.sol
+++ b/NounsDAOLogicV2.sol
@@ -1,14 +1,25 @@
         }

-        uint256 amount = address(this).balance;
+        uint256 amount = assemblyInternalBalance();
         (bool sent, ) = msg.sender.call{ value: amount }('');



        function _refundGas(uint256 startGas) internal {
            unchecked {
-                uint256 balance = address(this).balance;
+                uint256 balance = assemblyInternalBalance();
                if (balance == 0) {
                return;
            }



+        function assemblyInternalBalance() public returns (uint256) {
+           assembly {
+                let c := selfbalance()
+                mstore(0x00, c)
+                return(0x00, 0x20)
+            }
+        }


```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L297

```diff
File: contracts/governance/NounsDAOV3Votes.sol

-297            uint256 balance = address(this).balance;
+               uint256 balance = assemblyInternalBalance();


+        function assemblyInternalBalance() public returns (uint256) {
+           assembly {
+                let c := selfbalance()
+                mstore(0x00, c)
+                return(0x00, 0x20)
+            }
+        }

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L469

```diff
File: governance/NounsDAOV3Admin.sol

-469        uint256 amount = address(this).balance;
+           uint256 amount = assemblyInternalBalance();


+        function assemblyInternalBalance() public returns (uint256) {
+           assembly {
+                let c := selfbalance()
+                mstore(0x00, c)
+                return(0x00, 0x20)
+            }
+        }

```

## [G-16] Using > 0 costs more gas than != 0 when used on a uint

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L564

```solidity
File: governance/NounsDAOLogicV2.sol

564        if (votes > 0) {

1026        if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L585

```solidity
File: governance/NounsDAOV3Admin.sol

484        if (oldVoteSnapshotBlockSwitchProposalId > 0) {

585        if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L255

```solidity
File: governance/fork/NounsDAOV3Fork.sol

255            if (tokensToSend > 0) {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

```solidity
File: governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

227        if (srcRep != dstRep && amount > 0) {

230                uint96 srcRepOld = srcRepNum > 0 ? checkpoints[srcRep][srcRepNum - 1].votes : 0;

237                uint96 dstRepOld = dstRepNum > 0 ? checkpoints[dstRep][dstRepNum - 1].votes : 0;

255        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {

```

## [G-17]  Do not calculate constants variables

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L118

```solidity
File: contracts/governance/NounsDAOV3Admin.sol

118    uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;

121    uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;

127    uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;

145    uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;

```

## [G-18] Use do while loops instead of for loops

https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L305

```solidity
File: governance/NounsDAOLogicV2.sol

305        for (uint256 i = 0; i < proposal.targets.length; i++) {

343        for (uint256 i = 0; i < proposal.targets.length; i++) {

372        for (uint256 i = 0; i < proposal.targets.length; i++) {

405        for (uint256 i = 0; i < proposal.targets.length; i++) {


```

```diff
diff --git a/NounsDAOLogicV2.org.sol b/NounsDAOLogicV2.sol
index b5e1564..1b59e47 100644
--- a/NounsDAOLogicV2.org.sol
+++ b/NounsDAOLogicV2.sol
@@ -1,4 +1,5 @@
-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             queueOrRevertInternal(
                 proposal.targets[i],
                 proposal.values[i],
@@ -6,4 +7,4 @@
                 proposal.calldatas[i],
                 eta
             );
-        }
+        }while(i < proposal.targets.length);


-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
            timelock.executeTransaction(
                proposal.targets[i],
                proposal.values[i],
                proposal.signatures[i],
                proposal.calldatas[i],
                proposal.eta
            );
        }
+        }while(i < proposal.targets.length);
@@ -1,7 +1,8 @@
   );

         proposal.canceled = true;
-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             timelock.cancelTransaction(
                 proposal.targets[i],
                 proposal.values[i],
@@ -9,7 +10,7 @@
                 proposal.calldatas[i],
                 proposal.eta
             );
-        }
+        }while(i < proposal.targets.length);

         emit ProposalCanceled(proposalId);
     }
@@ -34,7 +35,8 @@
         Proposal storage proposal = _proposals[proposalId];

         proposal.vetoed = true;
-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             timelock.cancelTransaction(
                 proposal.targets[i],
                 proposal.values[i],
@@ -42,7 +44,8 @@
                 proposal.calldatas[i],
                 proposal.eta
             );
-        }
+        }while(i < proposal.targets.length);
+

         emit ProposalVetoed(proposalId);
     }
(END)

         proposal.canceled = true;
-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             timelock.cancelTransaction(
                 proposal.targets[i],
                 proposal.values[i],
@@ -9,7 +10,7 @@
                 proposal.calldatas[i],
                 proposal.eta
             );
-        }
+        }while(i < proposal.targets.length);

         emit ProposalCanceled(proposalId);
     }
@@ -34,7 +35,8 @@
         Proposal storage proposal = _proposals[proposalId];

         proposal.vetoed = true;
-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             timelock.cancelTransaction(
                 proposal.targets[i],
                 proposal.values[i],
@@ -42,7 +44,8 @@
                 proposal.calldatas[i],
                 proposal.eta
             );
-        }
+        }while(i < proposal.targets.length);


         emit ProposalVetoed(proposalId);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol

```solidity
File: contracts/governance/NounsDAOV3Proposals.sol

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

```diff
diff --git a/org.sol b/not.sol
index 16f2cfe..e315eec 100644
--- a/org.sol
+++ b/not.sol
@@ -8,13 +8,14 @@
             calcProposalEncodeData(msg.sender, txs, description)
         );

-        for (uint256 i = 0; i < proposerSignatures.length; ++i) {
+        uint256 i;
+        do{
             verifyProposalSignature(ds, proposalEncodeData, proposerSignatures[i], UPDATE_PROPOSAL_TYPEHASH);

             // To avoid the gas cost of having to search signers in proposal.signers, we're assuming the sigs we get
             // use the same amount of signers and the same order.
             if (signers[i] != proposerSignatures[i].signer) revert OnlyProposerCanEdit();
-        }
+        }while(i < proposerSignatures.length);

         proposal.targets = txs.targets;
         proposal.values = txs.values;
@@ -45,7 +46,8 @@
         NounsDAOStorageV3.Proposal storage proposal = ds._proposals[proposalId];
         INounsDAOExecutor timelock = getProposalTimelock(ds, proposal);
         uint256 eta = block.timestamp + timelock.delay();
-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             queueOrRevertInternal(
                 timelock,
                 proposal.targets[i],
@@ -54,7 +56,7 @@
                 proposal.calldatas[i],
                 eta
             );
-        }
+        }while(i < proposal.targets.length);
         proposal.eta = eta;
         emit ProposalQueued(proposalId, eta);
     }
@@ -107,7 +109,8 @@

         proposal.executed = true;

-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             timelock.executeTransaction(
                 proposal.targets[i],
                 proposal.values[i],
@@ -115,7 +118,7 @@
                 proposal.calldatas[i],
                 proposal.eta
             );
-        }
+        }while(i < proposal.targets.length);
         emit ProposalExecuted(proposal.id);
     }

@@ -152,7 +155,8 @@

         proposal.vetoed = true;
         INounsDAOExecutor timelock = getProposalTimelock(ds, proposal);
-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             timelock.cancelTransaction(
                 proposal.targets[i],
                 proposal.values[i],
@@ -160,7 +164,7 @@
                 proposal.calldatas[i],
                 proposal.eta
             );
-        }
+        }while(i < proposal.targets.length);

         emit ProposalVetoed(proposalId);
     }
@@ -189,10 +193,11 @@
         uint256 votes = nouns.getPriorVotes(proposer, block.number - 1);
         bool msgSenderIsProposer = proposer == msg.sender;
         address[] memory signers = proposal.signers;
-        for (uint256 i = 0; i < signers.length; ++i) {
+        uint256 i;
+        do{
             msgSenderIsProposer = msgSenderIsProposer || msg.sender == signers[i];
             votes += nouns.getPriorVotes(signers[i], block.number - 1);
-        }
+        }while(i < signers.length);

         require(
             msgSenderIsProposer || votes <= proposal.proposalThreshold,
@@ -201,7 +206,8 @@

         proposal.canceled = true;
         INounsDAOExecutor timelock = getProposalTimelock(ds, proposal);
-        for (uint256 i = 0; i < proposal.targets.length; i++) {
+        uint256 i;
+        do{
             timelock.cancelTransaction(
                 proposal.targets[i],
                 proposal.values[i],
@@ -209,7 +215,7 @@
                 proposal.calldatas[i],
                 proposal.eta
             );
-        }
+        }while(i < proposal.targets.length);

         emit ProposalCanceled(proposalId);
     }
@@ -425,7 +431,8 @@

         signers = new address[](proposerSignatures.length);
         uint256 numSigners = 0;
-        for (uint256 i = 0; i < proposerSignatures.length; ++i) {
+        uint256 i;
+        do{
             verifyProposalSignature(ds, proposalEncodeData, proposerSignatures[i], PROPOSAL_TYPEHASH);

             address signer = proposerSignatures[i].signer;
@@ -439,7 +446,7 @@
             signers[numSigners++] = signer;
             ds.latestProposalIds[signer] = proposalId;
             votes += signerVotes;
-        }
+        }while(i < proposerSignatures.length);

         if (numSigners < proposerSignatures.length) {
             // this assembly trims the signer array, getting rid of unused cells
@@ -459,14 +466,16 @@
         string memory description
     ) internal pure returns (bytes memory) {
         bytes32[] memory signatureHashes = new bytes32[](txs.signatures.length);
-        for (uint256 i = 0; i < txs.signatures.length; ++i) {
+        uint256 i;
+        do{
             signatureHashes[i] = keccak256(bytes(txs.signatures[i]));
-        }
+        }while(i < txs.signatures.length);
+

         bytes32[] memory calldatasHashes = new bytes32[](txs.calldatas.length);
-        for (uint256 i = 0; i < txs.calldatas.length; ++i) {
+        do{
             calldatasHashes[i] = keccak256(txs.calldatas[i]);
-        }
+        }while(i < txs.calldatas.length);

         return
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol

```solidity
File: governance/fork/NounsDAOV3Fork.sol

83        for (uint256 i = 0; i < tokenIds.length; i++) {

153        for (uint256 i = 0; i < tokenIds.length; i++) {

252        for (uint256 i = 0; i < erc20Count; ++i) {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol

```solidity
File: governance/fork/NounsDAOForkEscrow.sol

117        for (uint256 i = 0; i < tokenIds.length; i++) {

148        for (uint256 i = 0; i < tokenIds.length; i++) {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

```solidity
File: governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

209        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

223        for (uint256 i = 0; i < tokenIds.length; i++) {

231        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

238        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

248        for (uint256 i = 0; i < addresses.length; i++) {

397        for (uint256 i = 0; i < proposal.targets.length; i++) {

435        for (uint256 i = 0; i < proposal.targets.length; i++) {

462        for (uint256 i = 0; i < proposal.targets.length; i++) {

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol

```solidity
File: governance/fork/newdao/token/NounsTokenFork.sol

149        for (uint256 i = 0; i < tokenIds.length; i++) {

172        for (uint256 i = 0; i < tokenIds.length; i++) {

```

## [G-18] Ternary operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L525-L529

```solidity
File: governance/NounsDAOV3Proposals.sol

525        if (proposal.executeOnTimelockV1) {
526            return ds.timelockV1;
527        } else {
528            return ds.timelock;
529        }

```

```diff
diff --git a/NounsDAOV3Proposals.org.sol b/NounsDAOV3Proposals.sol
index 4d72873..2c9d994 100644
--- a/NounsDAOV3Proposals.org.sol
+++ b/NounsDAOV3Proposals.sol
@@ -3,9 +3,5 @@
         view
         returns (INounsDAOExecutor)
     {
-        if (proposal.executeOnTimelockV1) {
-            return ds.timelockV1;
-        } else {
-            return ds.timelock;
-        }
+        return proposal.executeOnTimelockV1 ? ds.timelockV1 : ds.timelock;
     }

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L187-L191

```solidity
File: governance/fork/NounsDAOForkEscrow.sol

187        if (owner == address(0)) {
188            return dao;
189        } else {
190            return owner;
191        }
```

```diff
diff --git a/NounsDAOForkEscrow.org.sol b/NounsDAOForkEscrow.sol
index 94b079c..ed7abdc 100644
--- a/NounsDAOForkEscrow.org.sol
+++ b/NounsDAOForkEscrow.sol
@@ -1,8 +1,4 @@
     function currentOwnerOf(uint256 tokenId) public view returns (address) {
         address owner = escrowedTokensByForkId[forkId][tokenId];
-        if (owner == address(0)) {
-            return dao;
-        } else {
-            return owner;
-        }
+        return owner == address(0) ? dao : owner;
     }
```

## [G-19] Use assembly to hash instead of solidity

for example:

```solidity

function solidityHash(uint256 a, uint256 b) public view {
    //unoptimized
    keccak256(abi.encodePacked(a, b));
}

```

Gas: 313

```solidity

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

https://medium.com/@bloqarl/solidity-gas-optimization-tips-with-assembly-you-havent-heard-yet-1381c77ff078

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L326

```solidity
File: contracts/governance/NounsDAOLogicV2.sol

326            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),

597        bytes32 structHash = keccak256(abi.encode(BALLOT_TYPEHASH, proposalId, support));

598        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));
```

```diff
diff --git a/NounsDAOLogicV2.org.sol b/NounsDAOLogicV2.sol
index 89a4781..12053a8 100644
--- a/NounsDAOLogicV2.org.sol
+++ b/NounsDAOLogicV2.sol
@@ -6,4 +6,11 @@
         uint256 eta
     ) internal {
         require(
-            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),

+        !timelock.queuedTransactions(    assembly {
+        mstore(0x00, target)
+        mstore(0x20, value)
+        mstore(0x40, signature)
+        mstore(0x60, data)
+        mstore(0x80, eta)
+        let hashedVal := keccak256(0x00, 0x80)
+    }),


```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L167

```solidity
File: contracts/governance/NounsDAOV3Votes.sol

167        bytes32 structHash = keccak256(abi.encode(BALLOT_TYPEHASH, proposalId, support));

168        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L143

```solidity
File: governance/NounsDAOExecutorV2.sol

143        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));


159        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));


174        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L469

```solidity
File: governance/NounsDAOV3Proposals.sol


469            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),


1006        bytes32 structHash = keccak256(abi.encodePacked(typehash, proposalEncodeData, expirationTimestamp));

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L120

```solidity
File: governance/NounsDAOExecutor.sol

120        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

136        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

151        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L418

```solidity
File: governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol


418            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),

598        bytes32 structHash = keccak256(abi.encode(BALLOT_TYPEHASH, proposalId, support));

599        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L149

```solidity
File: governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

149        bytes32 structHash = keccak256(abi.encode(DELEGATION_TYPEHASH, delegatee, nonce, expiry));

150        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));
```

## [G-20] use mappings instead of arrays

Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol

```solidity
File: contracts/governance/NounsDAOInterfaces.sol


323        address[] targets;

325        uint256[] values;

327        string[] signatures;

329        bytes[] calldatas;

700        address[] erc20TokensToIncludeInFork;

731        address[] targets;

733        uint256[] values;

735        string[] signatures;

737        bytes[] calldatas;

767        address[] signers;

823        address[] signers;

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol#L56

```solidity
File:  governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol

56    address[] public erc20TokensToIncludeInQuit;

70        address[] targets;

72        uint256[] values;

74        string[] signatures;

76        bytes[] calldatas;

```

## [G-21] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L598

```solidity
File: governance/NounsDAOLogicV2.sol

598        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

```

```diff
-598        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

+           bytes32 digest = keccak256(bytes.concat('\x19\x01', domainSeparator, structHash));
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L167

```diff
File: governance/NounsDAOV3Votes.sol

-168        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

+           bytes32 digest = keccak256(bytes.concat('\x19\x01', domainSeparator, structHash));
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L192

```diff
File: governance/NounsDAOExecutorV2.sol

-192            callData = abi.encodePacked(bytes4(keccak256(bytes(signature))), data);

+                callData = bytes.concat(bytes4(keccak256(bytes(signature))), data);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol

```solidity
File: contracts/governance/NounsDAOV3Proposals.sol


1006        bytes32 structHash = keccak256(abi.encodePacked(typehash, proposalEncodeData, expirationTimestamp));


404        bytes memory proposalEncodeData = abi.encodePacked(
            proposalId,
            calcProposalEncodeData(msg.sender, txs, description)
        );
```

## [G-22] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L105

```solidity
file: contracts/governance/fork/NounsDAOForkEscrow.sol

105        numTokensInEscrow++;
138        forkId++;

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L239


```solidity
file: contracts/governance/NounsDAOLogicV2.sol

239        proposalCount++;

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L837

```solidity
file: contracts/governance/NounsDAOV3Proposals.sol

837            signers[numSigners++] = signer;

```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L82

```solidity
file: script/ProposeDAOV3UpgradeMainnet.s.sol

82         i++;

88         i++;

100        i++;

106        i++;

```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L207


```solidity
file: contracts/governance/fork/newdao/token/NounsTokenFork.sol

207        return _mintTo(minter, _currentNounId++);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L153


```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

153        require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');

```
## [G-23] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

280        uint96 c = a + b;

291        return a - b;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

```solidity
file: contracts/governance/NounsDAOLogicV2.sol

1010            uint256 center = upper - (upper - lower) / 2;

1067            return (number * bps) / 10000;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

```solidity
file: contracts/governance/NounsDAOV3Admin.sol

488        uint256 newVoteSnapshotBlockSwitchProposalId = ds.proposalCount + 1;

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol

```solidity
file: contracts/governance/NounsDAOV3DynamicQuorum.sol

63        uint256 againstVotesBPS = (10000 * againstVotes) / totalSupply;

64        uint256 quorumAdjustmentBPS = (params.quorumCoefficient * againstVotesBPS) / 1e6;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

```solidity
file: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

215            uint256 endTime = startTime + duration;
```
