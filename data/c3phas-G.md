# Codebase Optimization Report

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Table Of Contents
- [Codebase Optimization Report](#codebase-optimization-report)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Table Of Contents](#table-of-contents)
  - [Note on Gas estimates.](#note-on-gas-estimates)
  - [Tighly pack storage variables/optimize the order of variable declaration](#tighly-pack-storage-variablesoptimize-the-order-of-variable-declaration)
    - [We can pack `admin and delay` by reducing the size of delay to uint8, or any other type less than uint96(Save 1 SLOT: 2100 GAS)](#we-can-pack-admin-and-delay-by-reducing-the-size-of-delay-to-uint8-or-any-other-type-less-than-uint96save-1-slot-2100-gas)
    - [Pack admin and delay together by reducing size of delay(Save 1 SLOT: 2.1K Gas)](#pack-admin-and-delay-together-by-reducing-size-of-delaysave-1-slot-21k-gas)
    - [Both duration and timeBuffer can be changed to uint48(Save 3 SLOTS: 6300 Gas )](#both-duration-and-timebuffer-can-be-changed-to-uint48save-3-slots-6300-gas-)
    - [Pack minter with isMinterLocked, descriptor with isDescriptorLocked, seeder with isSeederLocked, escrow with both forkId and forkingPeriodEndTimestamp : From 10 slots to 8 slots(Save 2 SLOTS)](#pack-minter-with-isminterlocked-descriptor-with-isdescriptorlocked-seeder-with-isseederlocked-escrow-with-both-forkid-and-forkingperiodendtimestamp--from-10-slots-to-8-slotssave-2-slots)
    - [Due to how variables are constrained during allocation, their sizes can be reduced to uint48(save 4 Slots ) From 10 slots to 6 slots](#due-to-how-variables-are-constrained-during-allocation-their-sizes-can-be-reduced-to-uint48save-4-slots--from-10-slots-to-6-slots)
  - [Pack structs by putting data types that can fit together next to each other](#pack-structs-by-putting-data-types-that-can-fit-together-next-to-each-other)
    - [We have some uint32 that can be be packed with an address(Save 1 SLOT: 2.1K gas)](#we-have-some-uint32-that-can-be-be-packed-with-an-addresssave-1-slot-21k-gas)
  - [Use calldata instead of memory for function parameters](#use-calldata-instead-of-memory-for-function-parameters)
    - [NounsDAOLogicV3.sol.updateProposal(): use calldata for `targets,values,signatures,calldatas,description` (Saves 2251 gas on average)](#nounsdaologicv3solupdateproposal-use-calldata-for-targetsvaluessignaturescalldatasdescription-saves-2251-gas-on-average)
    - [NounsDAOLogicV3.sol.updateProposalTransactions(): use calldata for `targets,values,signatures,calldatas,updateMessage` (Saves 2301 gas on average)](#nounsdaologicv3solupdateproposaltransactions-use-calldata-for-targetsvaluessignaturescalldatasupdatemessage-saves-2301-gas-on-average)
    - [NounsDAOLogicV3.sol.updateProposalBySigs(): use calldata for `proposerSignatures,targets,values` (Saves 2197 gas on average)](#nounsdaologicv3solupdateproposalbysigs-use-calldata-for-proposersignaturestargetsvalues-saves-2197-gas-on-average)
    - [NounsDAOLogicV3.sol.proposeBySigs(): use calldata for `proposerSignatures,targets,values,signatures` (Saves 1565 gas on average)](#nounsdaologicv3solproposebysigs-use-calldata-for-proposersignaturestargetsvaluessignatures-saves-1565-gas-on-average)
    - [NounsDAOLogicV3.sol.proposeOnTimelockV1():  Change to external and use calldata for `targets,values,signatures,calldatas,description` (Saves 541 gas on average)](#nounsdaologicv3solproposeontimelockv1--change-to-external-and-use-calldata-for-targetsvaluessignaturescalldatasdescription-saves-541-gas-on-average)
    - [NounsDAOLogicV1Fork.sol.quit(): use calldata for `erc20TokensToInclude` (Saves 105 gas on average)](#nounsdaologicv1forksolquit-use-calldata-for-erc20tokenstoinclude-saves-105-gas-on-average)
    - [Change to external and use calldata on signature and data (Save 520 gas on average)](#change-to-external-and-use-calldata-on-signature-and-data-save-520-gas-on-average)
    - [Change to external and use calldata on signature and data (Save 462 gas on average)](#change-to-external-and-use-calldata-on-signature-and-data-save-462-gas-on-average)
    - [Change to external and use calldata on signature and data (Save 412 gas on average)](#change-to-external-and-use-calldata-on-signature-and-data-save-412-gas-on-average)
    - [Change to external and use calldata on signature and data (Save 786 gas on average)](#change-to-external-and-use-calldata-on-signature-and-data-save-786-gas-on-average)
    - [Change to external and use calldata on signature and data (Save 404 gas on average)](#change-to-external-and-use-calldata-on-signature-and-data-save-404-gas-on-average)
    - [Change to external and use calldata on signature and data (Save 785 gas on average)](#change-to-external-and-use-calldata-on-signature-and-data-save-785-gas-on-average)
  - [Expensive operation inside a for loop](#expensive-operation-inside-a-for-loop)
    - [Function `quitInternal()` does a lot of inefficient operation mainly inside it's for loops. I've optimized it as a whole but avoided some common optimizations](#function-quitinternal-does-a-lot-of-inefficient-operation-mainly-inside-its-for-loops-ive-optimized-it-as-a-whole-but-avoided-some-common-optimizations)
    - [Don't read state inside loops,`escrow and forkId` should be cached outside the loop(Save 199 Gas on average)](#dont-read-state-inside-loopsescrow-and-forkid-should-be-cached-outside-the-loopsave-199-gas-on-average)
  - [Cache storage values in memory to minimize SLOADs](#cache-storage-values-in-memory-to-minimize-sloads)
    - [NounsDAOLogicV2.sol.propose(): `nouns` should be cached(saves 110 gas on average)](#nounsdaologicv2solpropose-nouns-should-be-cachedsaves-110-gas-on-average)
    - [queueOrRevertInternal(): `timelock` should be cached(save 1 SLOAD: 200 gas)](#queueorrevertinternal-timelock-should-be-cachedsave-1-sload-200-gas)
    - [NounsDAOLogicV2.sol.veto(): `vetoer` should be cached(saves 92 gas on average)](#nounsdaologicv2solveto-vetoer-should-be-cachedsaves-92-gas-on-average)
    - [NounsDAOLogicV2.sol.getDynamicQuorumParamsAt(): `quorumVotesBPS` should be cached(saves 2 SLOADs: 200 gas)](#nounsdaologicv2solgetdynamicquorumparamsat-quorumvotesbps-should-be-cachedsaves-2-sloads-200-gas)
    - [NounsAuctionHouseFork.sol.\_safeTransferETHWithFallback(): `weth` should be cached(save 1 SLOAD: ~97 gas)](#nounsauctionhouseforksol_safetransferethwithfallback-weth-should-be-cachedsave-1-sload-97-gas)
    - [NounsDAOLogicV1Fork.sol.adjustedTotalSupply(): `nouns` should be cached(Save 237 gas on average)](#nounsdaologicv1forksoladjustedtotalsupply-nouns-should-be-cachedsave-237-gas-on-average)
  - [Use the existing Local variable/global variable when equal to a state variable to avoid reading from state](#use-the-existing-local-variableglobal-variable-when-equal-to-a-state-variable-to-avoid-reading-from-state)
    - [Local variable `_escrow` should be used instead of reading `escrow`](#local-variable-_escrow-should-be-used-instead-of-reading-escrow)
    - [Global variable `msg.sender` should be used instead of reading state(Save 128 gas on average)](#global-variable-msgsender-should-be-used-instead-of-reading-statesave-128-gas-on-average)
  - [Emitting storage values instead of the memory one.](#emitting-storage-values-instead-of-the-memory-one)
    - [NounsDAOLogicV2.sol.\_setVotingDelay(): Emit `newVotingDelay` instead of `votingDelay`(save 1 SLOAD: 100 gas)](#nounsdaologicv2sol_setvotingdelay-emit-newvotingdelay-instead-of-votingdelaysave-1-sload-100-gas)
    - [NounsDAOLogicV2.sol.\_setVotingPeriod(): Emit `newVotingPeriod` instead of `votingPeriod`(save 1 SLOAD: 100 gas)](#nounsdaologicv2sol_setvotingperiod-emit-newvotingperiod-instead-of-votingperiodsave-1-sload-100-gas)
    - [NounsDAOLogicV2.sol.\_setProposalThresholdBPS(): Emit `newProposalThresholdBPS` instead of `proposalThresholdBPS`(save 1 SLOAD: 100 gas)](#nounsdaologicv2sol_setproposalthresholdbps-emit-newproposalthresholdbps-instead-of-proposalthresholdbpssave-1-sload-100-gas)
    - [Use the cheaper global variable(msg.sender) instead of reading from state when emitting(Save 1 SLOAD: ~100 gas)](#use-the-cheaper-global-variablemsgsender-instead-of-reading-from-state-when-emittingsave-1-sload-100-gas)
    - [NounsDAOProxy.sol.\_setImplementation(): Emit `implementation_` instead of `implementation`(save 1 SLOAD: 100 gas)](#nounsdaoproxysol_setimplementation-emit-implementation_-instead-of-implementationsave-1-sload-100-gas)
    - [NounsDAOExecutor.sol.setDelay(): Emit `delay_` instead of `delay`(save 1 SLOAD: 100 gas)](#nounsdaoexecutorsolsetdelay-emit-delay_-instead-of-delaysave-1-sload-100-gas)
    - [NounsDAOExecutor.sol.acceptAdmin(): Emit `msg.sender` instead of `admin`(save 1 SLOAD: 100 gas)](#nounsdaoexecutorsolacceptadmin-emit-msgsender-instead-of-adminsave-1-sload-100-gas)
    - [NounsDAOExecutor.sol.setPendingAdmin(): Emit `pendingAdmin_` instead of `pendingAdmin`(save 1 SLOAD: 100 gas)](#nounsdaoexecutorsolsetpendingadmin-emit-pendingadmin_-instead-of-pendingadminsave-1-sload-100-gas)
  - [Optimizing check order for cost efficient function execution](#optimizing-check-order-for-cost-efficient-function-execution)
    - [Cheaper require statements should be performed first](#cheaper-require-statements-should-be-performed-first)
    - [External calls + state reads are very expensive. Validate local/functional parameters first](#external-calls--state-reads-are-very-expensive-validate-localfunctional-parameters-first)
    - [Validate function parameter first before reading from state](#validate-function-parameter-first-before-reading-from-state)
    - [Avoid validating state variables before validating function parameters](#avoid-validating-state-variables-before-validating-function-parameters)
    - [Validate function parameters first before reading any state variables](#validate-function-parameters-first-before-reading-any-state-variables)
    - [Validate Function parameter  before validating state variables](#validate-function-parameter--before-validating-state-variables)
    - [Reorder the checks here to have cheaper checks first](#reorder-the-checks-here-to-have-cheaper-checks-first)
    - [Function parameters should be validated first](#function-parameters-should-be-validated-first)
    - [Validate all function parameters first before reading any state variables](#validate-all-function-parameters-first-before-reading-any-state-variables)
    - [Validate function parameters before validating state variables](#validate-function-parameters-before-validating-state-variables)
    - [Function parameters should be checked first before reading any state variables](#function-parameters-should-be-checked-first-before-reading-any-state-variables)
    - [Validate parameters before making external function calls](#validate-parameters-before-making-external-function-calls)
    - [Avoid making external calls before validating function parameters](#avoid-making-external-calls-before-validating-function-parameters)
    - [Validate function parameters before validating state variables](#validate-function-parameters-before-validating-state-variables-1)
    - [Validate all function parameters first before doing other operations](#validate-all-function-parameters-first-before-doing-other-operations)
    - [Reorder the checks to have the cheaper checks first](#reorder-the-checks-to-have-the-cheaper-checks-first)
    - [Validations that involve state reads should not be done at the beginning if we have other cheaper checks](#validations-that-involve-state-reads-should-not-be-done-at-the-beginning-if-we-have-other-cheaper-checks)
    - [Reorder the checks to have cheaper checks first](#reorder-the-checks-to-have-cheaper-checks-first)
    - [Any variable checks should be done first](#any-variable-checks-should-be-done-first)
    - [Reorder the checks here to avoid wasting gas on later reverts](#reorder-the-checks-here-to-avoid-wasting-gas-on-later-reverts)
  - [The following functions can benefit from some optimizations](#the-following-functions-can-benefit-from-some-optimizations)
  - [We can  optimize the function `_acceptAdmin`(Save 4 SLOADS: ~400 gas)](#we-can--optimize-the-function-_acceptadminsave-4-sloads-400-gas)
    - [We can optimize the function `_acceptVetoer()` Save 3 SLOADS: ~300 gas](#we-can-optimize-the-function-_acceptvetoer-save-3-sloads-300-gas)
    - [We can optimize the function `_acceptAdmin()` Save 3 SLOADS: ~300 gas](#we-can-optimize-the-function-_acceptadmin-save-3-sloads-300-gas)
    - [`_acceptVetoer()` can be optimized(Save 3 SLOADs: ~300 gas )](#_acceptvetoer-can-be-optimizedsave-3-sloads-300-gas-)
    - [Cheaper to read global variable compared to state variable(Save 1 SLOAD: ~100 gas)](#cheaper-to-read-global-variable-compared-to-state-variablesave-1-sload-100-gas)
    - [We can optimize the function `_acceptAdmin()` (Save 4 SLOADs: ~400 Gas)](#we-can-optimize-the-function-_acceptadmin-save-4-sloads-400-gas)
  - [Nested if is cheaper than single statement](#nested-if-is-cheaper-than-single-statement)
  - [Caching a variable that is used once just wastes Gas](#caching-a-variable-that-is-used-once-just-wastes-gas)
    - [No need to cache `ds.forkEscrow` as it's being used once](#no-need-to-cache-dsforkescrow-as-its-being-used-once)
    - [NounsTokenFork.sol.claimDuringForkPeriod(): `_currentNounId` should not be cached](#nounstokenforksolclaimduringforkperiod-_currentnounid-should-not-be-cached)
  - [Importing an entire library while only using one function isn't necessary](#importing-an-entire-library-while-only-using-one-function-isnt-necessary)
  - [Conclusion](#conclusion)

## Note on Gas estimates.
I've tried to give the exact amount of gas being saved from running the included tests. Whenever the function is within the test coverage, the average gas before and after will be included, and often a diff of the code will also accompany this.

Some functions are not covered by the test cases or are internal/private functions. In this case, the gas can be estimated by looking at the opcodes involved. For some benchmarks are based on the function that calls this internal functions.

## Tighly pack storage variables/optimize the order of variable declaration
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.
Here, the storage variables can be tightly packed from:

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L89-L91
### We can pack `admin and delay` by reducing the size of delay to uint8, or any other type less than uint96(Save 1 SLOT: 2100 GAS)
**This is possible because we constrain the value of `delay` to 0 - 30** 
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
89:    address public admin;
90:    address public pendingAdmin;
91:    uint256 public delay;
```

We can see why it's safe to reduce the size of the variable delay. When setting delay we are checking that the value set should be between **MINIMUM_DELAY and MAXIMUM_DELAY** which are constant variables with values 0 and 30 respectively 
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L103-L110

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
103:    function setDelay(uint256 delay_) public {

105:        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
106:        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
107:        delay = delay_;
```

**The refactored code would be like this, note that we pack with admin as they are accessed in one transaction**
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
index f4f85883..ab900fed 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
@@ -87,8 +87,9 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     uint256 public constant MAXIMUM_DELAY = 30 days;

     address public admin;
+    uint8 public delay;
     address public pendingAdmin;
-    uint256 public delay;
```
Some other parts of the code will pass a uint256 thus we can type cast those parts eg 
```diff
     function setDelay(uint256 delay_) public {
         require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
         require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
         require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
-        delay = delay_;
+        delay = uint8(delay_);

         emit NewDelay(delay_);
     }
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L66-L68
### Pack admin and delay together by reducing size of delay(Save 1 SLOT: 2.1K Gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
66:    address public admin;
67:    address public pendingAdmin;
68:    uint256 public delay;
```
When setting delay, we have a conditional check that ensures the value of delay is between  2 and 30
```solidity
        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');
        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
```
As such we know delay will always be less than 30. We can therefore reduce the size of the data type to uint8 and pack the variable with `admin`

```diff
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -64,9 +64,9 @@ contract NounsDAOExecutor {
     uint256 public constant MAXIMUM_DELAY = 30 days;

     address public admin;
+    uint8 public delay;
     address public pendingAdmin;
-    uint256 public delay;
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L50-L69
### Both duration and timeBuffer can be changed to uint48(Save 3 SLOTS: 6300 Gas )
**We pack nouns with duration, weth with timeBuffer and minBidIncrementPercentage**
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

50:    // The Nouns ERC721 token contract
51:    INounsToken public nouns;

53:    // The address of the WETH contract
54:    address public weth;

56:    // The minimum amount of time left in an auction after a new bid is created
57:    uint256 public timeBuffer;

59:    // The minimum price accepted in an auction
60:    uint256 public reservePrice;

62:    // The minimum percentage difference between the last bid amount and the current bid
63:    uint8 public minBidIncrementPercentage;

65:    // The duration of a single auction
66:    uint256 public duration;

68:    // The active auction
69:    INounsAuctionHouse.Auction public auction;
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
index 0bd594b6..2746e3d6 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
@@ -49,21 +49,21 @@ contract NounsAuctionHouseFork is

     // The Nouns ERC721 token contract
     INounsToken public nouns;
+    // The duration of a single auction
+    uint48 public duration;

     // The address of the WETH contract
     address public weth;

     // The minimum amount of time left in an auction after a new bid is created
-    uint256 public timeBuffer;
-
-    // The minimum price accepted in an auction
-    uint256 public reservePrice;
+    uint48 public timeBuffer;

     // The minimum percentage difference between the last bid amount and the current bid
     uint8 public minBidIncrementPercentage;

-    // The duration of a single auction
-    uint256 public duration;
+    // The minimum price accepted in an auction
+    uint256 public reservePrice;
+

     // The active auction
     INounsAuctionHouse.Auction public auction;
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L48-L85
### Pack minter with isMinterLocked, descriptor with isDescriptorLocked, seeder with isSeederLocked, escrow with both forkId and forkingPeriodEndTimestamp : From 10 slots to 8 slots(Save 2 SLOTS)
**1 SLOT = 2k Gas**
 **Total gas saved: 4K gas**

*For forkingPeriodEndTimestamp we can reduce the size to uint48 as it should be safe for timestamp*
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol

49:    address public minter;

52:    INounsDescriptorMinimal public descriptor;

55:    INounsSeeder public seeder;

58:    INounsDAOForkEscrow public escrow;

61:    uint32 public forkId;

64:    uint256 public remainingTokensToClaim;

67:    uint256 public forkingPeriodEndTimestamp;

70:    bool public isMinterLocked;

73:    bool public isDescriptorLocked;

76:    bool public isSeederLocked;

79:    mapping(uint256 => INounsSeeder.Seed) public seeds;

82:    uint256 private _currentNounId;

85:    string private _contractURIHash = 'QmZi1n79FqWt2tTLwCqiy6nLM6xLGRsEPQ5JmReJQKNNzX';
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
index a1f9d6d3..ba8947db 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -48,11 +48,20 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /// @notice  An address who has permissions to mint Nouns
     address public minter;

+    /// @notice Whether the minter can be updated
+    bool public isMinterLocked;
+
     /// @notice The Nouns token URI descriptor
     INounsDescriptorMinimal public descriptor;

+    /// @notice Whether the descriptor can be updated
+    bool public isDescriptorLocked;
+
     /// @notice The Nouns token seeder
     INounsSeeder public seeder;
+
+    /// @notice Whether the seeder can be updated
+    bool public isSeederLocked;

     /// @notice The escrow contract used to verify ownership of the original Nouns in the post-fork claiming process
     INounsDAOForkEscrow public escrow;
@@ -60,20 +69,11 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /// @notice The fork ID, used when querying the escrow for token ownership
     uint32 public forkId;

-    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
-    uint256 public remainingTokensToClaim;
-
     /// @notice The forking period expiration timestamp, after which new tokens cannot be claimed by the original DAO
-    uint256 public forkingPeriodEndTimestamp;
+    uint48 public forkingPeriodEndTimestamp;

-    /// @notice Whether the minter can be updated
-    bool public isMinterLocked;
-
-    /// @notice Whether the descriptor can be updated
-    bool public isDescriptorLocked;
-
-    /// @notice Whether the seeder can be updated
-    bool public isSeederLocked;
+    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
+    uint256 public remainingTokensToClaim;

     /// @notice The noun seeds
     mapping(uint256 => INounsSeeder.Seed) public seeds;
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L380-L403
### Due to how variables are constrained during allocation, their sizes can be reduced to uint48(save 4 Slots ) From 10 slots to 6 slots
Gas saved: `4 SLots * 2.1K gas` = 8.4K gas
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {
382:    address public vetoer;

385:    uint256 public votingDelay;

388:    uint256 public votingPeriod;

391:    uint256 public proposalThresholdBPS;

394:    uint256 public quorumVotesBPS;

397:    uint256 public proposalCount;

400:    INounsDAOExecutor public timelock;

403:    NounsTokenLike public nouns;
```

**Whenever votingDelay, votingPeriod, proposalThresholdBPS are being assigned a value, we ensure the value has a limit** See below
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L150-L161
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
150:        require(
151:            votingPeriod_ >= MIN_VOTING_PERIOD && votingPeriod_ <= MAX_VOTING_PERIOD,
152:            'NounsDAO::initialize: invalid voting period'
153:        );
154:        require(
155:            votingDelay_ >= MIN_VOTING_DELAY && votingDelay_ <= MAX_VOTING_DELAY,
156:            'NounsDAO::initialize: invalid voting delay'
157:        );
158:        require(
159:            proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,
160:            'NounsDAO::initialize: invalid proposal threshold bps'
161:        );
```
The above checks are always made whenever we need to set a value to our storage values. As the checks ensure our values don't exceed a certain number(constant numbers defined) we can inspect the value of the constants and make sure our chosen data type would fit the number. 
For all constants defined in our contracts, for the selected variables here, uint48 would fit without loosing any precision.
We can therefore pack as follows
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol b/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
index 8fb0b4d3..3fed7bc2 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
@@ -382,17 +382,11 @@ contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {
     address public vetoer;

     /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
-    uint256 public votingDelay;
+    uint48 public votingDelay;

     /// @notice The duration of voting on a proposal, in blocks
-    uint256 public votingPeriod;
-
-    /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
-    uint256 public proposalThresholdBPS;
-
-    /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
-    uint256 public quorumVotesBPS;
-
+    uint48 public votingPeriod;
+
     /// @notice The total number of proposals
     uint256 public proposalCount;

@@ -401,6 +395,10 @@ contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {

     /// @notice The address of the Nouns tokens
     NounsTokenLike public nouns;
+        /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
+    uint48 public proposalThresholdBPS;
+    /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
+    uint48 public quorumVotesBPS;

     /// @notice The official record of all proposals ever proposed
     mapping(uint256 => Proposal) internal _proposals;
```

## Pack structs by putting data types that can fit together next to each other
As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L653-L717
### We have some uint32 that can be be packed with an address(Save 1 SLOT: 2.1K gas)
**Pack lastMinuteWindowInBlocks,objectionPeriodDurationInBlocks,proposalUpdatablePeriodInBlocks with forkDAOTreasury** 
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
    struct StorageV3 {
		<Truncated>
690:        uint32 lastMinuteWindowInBlocks;
691:        /// @notice Length of the objection period in blocks
692:        uint32 objectionPeriodDurationInBlocks;
693:        /// @notice Length of proposal updatable period in block
694:        uint32 proposalUpdatablePeriodInBlocks;
695:        /// @notice address of the DAO's fork escrow contract
696:        INounsDAOForkEscrow forkEscrow;
697:        /// @notice address of the DAO's fork deployer contract
698:        IForkDAODeployer forkDAODeployer;
699:        /// @notice ERC20 tokens to include when sending funds to a deployed fork
700:        address[] erc20TokensToIncludeInFork;
701:        /// @notice The treasury contract of the last deployed fork
702:        address forkDAOTreasury;
    <Truncated>
       
    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol b/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
index 8fb0b4d3..ba0a251f 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
@@ -686,12 +686,6 @@ contract NounsDAOStorageV3 {
         // ================ V3 ================ //
         /// @notice user => sig => isCancelled: signatures that have been cancelled by the signer and are no longer valid
         mapping(address => mapping(bytes32 => bool)) cancelledSigs;
-        /// @notice The number of blocks before voting ends during which the objection period can be initiated
-        uint32 lastMinuteWindowInBlocks;
-        /// @notice Length of the objection period in blocks
-        uint32 objectionPeriodDurationInBlocks;
-        /// @notice Length of proposal updatable period in block
-        uint32 proposalUpdatablePeriodInBlocks;
         /// @notice address of the DAO's fork escrow contract
         INounsDAOForkEscrow forkEscrow;
         /// @notice address of the DAO's fork deployer contract
@@ -700,6 +694,12 @@ contract NounsDAOStorageV3 {
         address[] erc20TokensToIncludeInFork;
         /// @notice The treasury contract of the last deployed fork
         address forkDAOTreasury;
+        /// @notice The number of blocks before voting ends during which the objection period can be initiated
+        uint32 lastMinuteWindowInBlocks;
+        /// @notice Length of the objection period in blocks
+        uint32 objectionPeriodDurationInBlocks;
+        /// @notice Length of proposal updatable period in block
+        uint32 proposalUpdatablePeriodInBlocks;
         /// @notice The token contract of the last deployed fork
         address forkDAOToken;
```

## Use calldata instead of memory for function parameters
If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I've also flagged instances where the function is public but can be marked as external since it's not called by the contract

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L278-L288
### NounsDAOLogicV3.sol.updateProposal(): use calldata for `targets,values,signatures,calldatas,description` (Saves 2251 gas on average)
**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 10622    | 20773   | 14500 | 92416 |
| After  | 9612    | 18522   | 12580 | 90342 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
278:    function updateProposal(
279:        uint256 proposalId,
280:        address[] memory targets,
281:        uint256[] memory values,
282:        string[] memory signatures,
283:        bytes[] memory calldatas,
284:        string memory description,
285:        string memory updateMessage
286:    ) external {
287:        ds.updateProposal(proposalId, targets, values, signatures, calldatas, description, updateMessage);
288:    }
```
Last variable has not been modified to avoid the stack too deep error
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
index 00c5ccdc..4e8281cb 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
@@ -277,11 +277,11 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      */
     function updateProposal(
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
-        string memory description,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
+        string calldata description,
         string memory updateMessage
     ) external {
         ds.updateProposal(proposalId, targets, values, signatures, calldatas, description, updateMessage);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L313-L322
### NounsDAOLogicV3.sol.updateProposalTransactions(): use calldata for `targets,values,signatures,calldatas,updateMessage` (Saves 2301 gas on average)
**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 9566    | 19643   | 13349 | 90201 |
| After  | 8515    | 17342   | 11461 | 88076 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
313:    function updateProposalTransactions(
314:        uint256 proposalId,
315:        address[] memory targets,
316:        uint256[] memory values,
317:        string[] memory signatures,
318:        bytes[] memory calldatas,
319:        string memory updateMessage
320:    ) external {
321:        ds.updateProposalTransactions(proposalId, targets, values, signatures, calldatas, updateMessage);
322:    }
```

```diff
     function updateProposalTransactions(
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
-        string memory updateMessage
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
+        string calldata updateMessage
     ) external {
         ds.updateProposalTransactions(proposalId, targets, values, signatures, calldatas, updateMessage);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L338-L355
### NounsDAOLogicV3.sol.updateProposalBySigs(): use calldata for `proposerSignatures,targets,values` (Saves 2197 gas on average)
**Gas benchmarks**

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 15239  | 38005   | 33310  | 122687 |
| After  | 14700 | 35808   | 31945  | 120500 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
338:    function updateProposalBySigs(
339:        uint256 proposalId,
340:        ProposerSignature[] memory proposerSignatures,
341:        address[] memory targets,
342:        uint256[] memory values,
343:        string[] memory signatures,
344:        bytes[] memory calldatas,
345:        string memory description,
346:        string memory updateMessage
347:    ) external {
348:        ds.updateProposalBySigs(
349:            proposalId,
350:            proposerSignatures,
351:            NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas),
352:            description,
353:            updateMessage
354:        );
355:    }
```
We cannot refactor all variables to avoid stack too deep error
```diff
     function updateProposalBySigs(
         uint256 proposalId,
-        ProposerSignature[] memory proposerSignatures,
-        address[] memory targets,
-        uint256[] memory values,
+        ProposerSignature[] calldata proposerSignatures,
+        address[] calldata targets,
+        uint256[] calldata values,
         string[] memory signatures,
         bytes[] memory calldatas,
         string memory description,
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L236-L250
### NounsDAOLogicV3.sol.proposeBySigs(): use calldata for `proposerSignatures,targets,values,signatures` (Saves 1565 gas on average)
**Gas benchmarks**

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 13725  | 431650   | 431722  | 535759 |
| After  | 13336 | 430085   | 430511  | 533718 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
236:    function proposeBySigs(
237:        ProposerSignature[] memory proposerSignatures,
238:        address[] memory targets,
239:        uint256[] memory values,
240:        string[] memory signatures,
241:        bytes[] memory calldatas,
242:        string memory description
243:    ) external returns (uint256) {
244:        return
245:            ds.proposeBySigs(
246:                proposerSignatures,
247:                NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas),
248:                description
249:            );
250:    }
```
Note: not all variables are changed, this is to avoid the stack too deep error

```diff
     function proposeBySigs(
-        ProposerSignature[] memory proposerSignatures,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
+        ProposerSignature[] calldata proposerSignatures,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
         bytes[] memory calldatas,
         string memory description
     ) external returns (uint256) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L210-L222
### NounsDAOLogicV3.sol.proposeOnTimelockV1():  Change to external and use calldata for `targets,values,signatures,calldatas,description` (Saves 541 gas on average)
**Gas benchmarks**

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 341736  | 683821   | 683821  | 1025906 |
| After  | 341705 | 683280   | 683280  | 1024855 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
210:    function proposeOnTimelockV1(
211:        address[] memory targets,
212:        uint256[] memory values,
213:        string[] memory signatures,
214:        bytes[] memory calldatas,
215:        string memory description
216:    ) public returns (uint256) {
217:        return
218:            ds.proposeOnTimelockV1(
219:                NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas),
220:                description
221:            );
222:    }
```

```diff
     function proposeOnTimelockV1(
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
-        string memory description
-    ) public returns (uint256) {
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
+        string calldata description
+    ) external returns (uint256) {
         return
             ds.proposeOnTimelockV1(
                 NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas),
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206-L216
### NounsDAOLogicV1Fork.sol.quit(): use calldata for `erc20TokensToInclude` (Saves 105 gas on average)
**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 14309    | 165989   | 151098 | 347452 |
| After  | 165884    | 165884   | 151011 | 347390 |

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
206:    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {

208:        address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
209:        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
210:            if (!isAddressIn(erc20TokensToInclude[i], erc20TokensToIncludeInQuit_)) {
211:                revert TokensMustBeASubsetOfWhitelistedTokens();
212:            }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..beaff3ab 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -203,7 +203,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         quitInternal(tokenIds, erc20TokensToIncludeInQuit);
     }

-    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
+    function quit(uint256[] calldata tokenIds, address[] calldata erc20TokensToInclude) external nonReentrant {
         // check that erc20TokensToInclude is a subset of `erc20TokensToIncludeInQuit`
         address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L130-L148
### Change to external and use calldata on signature and data (Save 520 gas on average)
`Gas benchmarks`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 26117    | 27113   | 26117 | 29751 |
| After  | 25652    | 26593   | 25652 | 28749 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
130:    function queueTransaction(
131:        address target,
132:        uint256 value,
133:        string memory signature,
134:        bytes memory data,
135:        uint256 eta
136:    ) public returns (bytes32) {

143:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
        
146:        emit QueueTransaction(txHash, target, value, signature, data, eta);
147:        return txHash;
148:    }
```

```diff
@@ -130,10 +130,10 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function queueTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public returns (bytes32) {
+    ) external returns (bytes32) {
         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');
         require(
             eta >= getBlockTimestamp() + delay,
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L150-L163
### Change to external and use calldata on signature and data (Save 462 gas on average)
`Gas benchmarks`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 10017    | 10068   | 10017 | 10426 |
| After  | 9552    | 9606   | 9552 | 9882 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
150:    function cancelTransaction(
151:        address target,
152:        uint256 value,
153:        string memory signature,
154:        bytes memory data,
155:        uint256 eta
156:    ) public {

159:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

162:        emit CancelTransaction(txHash, target, value, signature, data, eta);
163:    }
```


```diff
@@ -150,10 +150,10 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function cancelTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public {
+    ) external {
         require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');

         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L165-L202
### Change to external and use calldata on signature and data (Save 412 gas on average)
`Gas benchmarks`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2437    | 53364   | 29527 | 339600 |
| After  | 2048    | 52952   | 28893 | 339006 |


```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
165:    function executeTransaction(
166:        address target,
167:        uint256 value,
168:        string memory signature,
169:        bytes memory data,
170:        uint256 eta
171:    ) public returns (bytes memory) {

174:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

189:        if (bytes(signature).length == 0) {
190:            callData = data;
191:        } else {
192:            callData = abi.encodePacked(bytes4(keccak256(bytes(signature))), data);
193:        }

199:        emit ExecuteTransaction(txHash, target, value, signature, data, eta);
```

```diff
@@ -165,10 +165,10 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function executeTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public returns (bytes memory) {
+    ) external returns (bytes memory) {
         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');

         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L107-L125
### Change to external and use calldata on signature and data (Save 786 gas on average)
`Gas benchmarks`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 26107    | 28172   | 28107 | 29783 |
| After  | 25691    | 27386   | 27691 | 28240 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
107:    function queueTransaction(
108:        address target,
109:        uint256 value,
110:        string memory signature,
111:        bytes memory data,
112:        uint256 eta
113:    ) public returns (bytes32) {

120:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

123:        emit QueueTransaction(txHash, target, value, signature, data, eta);
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..d705c94d 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -107,10 +107,10 @@ contract NounsDAOExecutor {
     function queueTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public returns (bytes32) {
+    ) external returns (bytes32) {
         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');
         require(
             eta >= getBlockTimestamp() + delay,
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L127-L140
### Change to external and use calldata on signature and data (Save 404 gas on average)
`Gas benchmarks`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4788    | 8671   | 9985 | 9985 |
| After  | 4456    | 8267   | 9569 | 9569 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
127:    function cancelTransaction(
128:        address target,
129:        uint256 value,
130:        string memory signature,
131:        bytes memory data,
132:        uint256 eta
133:    ) public {

136:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

139:        emit CancelTransaction(txHash, target, value, signature, data, eta);
140:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..25956a16 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -127,10 +127,10 @@ contract NounsDAOExecutor {
     function cancelTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public {
+    ) external {
         require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');

         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L142-L179
### Change to external and use calldata on signature and data (Save 785 gas on average)
`Gas benchmarks`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2360    | 42009   | 30881 | 142555 |
| After  | 2020    | 41224   | 30668 | 140491 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
142:    function executeTransaction(
143:        address target,
144:        uint256 value,
145:        string memory signature,
146:        bytes memory data,
147:        uint256 eta
148:    ) public returns (bytes memory) {

151:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

166:        if (bytes(signature).length == 0) {
167:            callData = data;
168:        } else {
169:            callData = abi.encodePacked(bytes4(keccak256(bytes(signature))), data);
170:        }

172:        // solium-disable-next-line security/no-call-value
173:        (bool success, bytes memory returnData) = target.call{ value: value }(callData);


176:        emit ExecuteTransaction(txHash, target, value, signature, data, eta);
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..d3d4b135 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -142,10 +142,10 @@ contract NounsDAOExecutor {
     function executeTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public returns (bytes memory) {
+    ) external returns (bytes memory) {
         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');

         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```

## Expensive operation inside a for loop
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L218-L245
### Function `quitInternal()` does a lot of inefficient operation mainly inside it's for loops. I've optimized it as a whole but avoided some common optimizations 
**As it's an internal one, the gas changes can be seen from the two function `quit`**
`Benchmarks for quit(uint256[])`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 743    | 245981   | 202066 | 501600 |
| After  | 743    | 245388   | 201653 | 500096 |

`Benchmarks for quit(uint256[],address[])`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 14309    | 165989   | 151098 | 347452 |
| After  | 14309    | 165727   | 150891 | 346817 |

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
218:    function quitInternal(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) internal {

223:        for (uint256 i = 0; i < tokenIds.length; i++) {
224:            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
225:        }

231:        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
232:            IERC20 erc20token = IERC20(erc20TokensToInclude[i]);
233:            balancesToSend[i] = (erc20token.balanceOf(address(timelock)) * tokenIds.length) / totalSupply;
234:        }

237:        timelock.sendETH(payable(msg.sender), ethToSend);
238:        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
239:            if (balancesToSend[i] > 0) {
240:                timelock.sendERC20(msg.sender, erc20TokensToInclude[i], balancesToSend[i]);
241:            }
242:        }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..49d60515 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -219,25 +219,26 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         checkGovernanceActive();

         uint256 totalSupply = adjustedTotalSupply();
-
+        NounsDAOExecutorV2  _timelock = timelock;
+        INounsTokenForkLike _nouns = nouns;
         for (uint256 i = 0; i < tokenIds.length; i++) {
-            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
+            _nouns.transferFrom(msg.sender, address(_timelock), tokenIds[i]);
         }

         uint256[] memory balancesToSend = new uint256[](erc20TokensToInclude.length);

         // Capture balances to send before actually sending them, to avoid the risk of external calls changing balances.
-        uint256 ethToSend = (address(timelock).balance * tokenIds.length) / totalSupply;
+        uint256 ethToSend = (address(_timelock).balance * tokenIds.length) / totalSupply;
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
             IERC20 erc20token = IERC20(erc20TokensToInclude[i]);
-            balancesToSend[i] = (erc20token.balanceOf(address(timelock)) * tokenIds.length) / totalSupply;
+            balancesToSend[i] = (erc20token.balanceOf(address(_timelock)) * tokenIds.length) / totalSupply;
         }

         // Send ETH and ERC20 tokens
-        timelock.sendETH(payable(msg.sender), ethToSend);
+        _timelock.sendETH(payable(msg.sender), ethToSend);
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
             if (balancesToSend[i] > 0) {
-                timelock.sendERC20(msg.sender, erc20TokensToInclude[i], balancesToSend[i]);
+                _timelock.sendERC20(msg.sender, erc20TokensToInclude[i], balancesToSend[i]);
             }
         }

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L148-L157
### Don't read state inside loops,`escrow and forkId` should be cached outside the loop(Save 199 Gas on average)
**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 6721    | 387833   | 217402 | 3683287 |
| After  | 6754    | 387714   | 217439 | 3680633 |
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
149:        for (uint256 i = 0; i < tokenIds.length; i++) {
150:            uint256 nounId = tokenIds[i];
151:            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();

156:            _mintWithOriginalSeed(msg.sender, nounId);
157:        }
```

```diff
     function claimFromEscrow(uint256[] calldata tokenIds) external {
+        INounsDAOForkEscrow _escrow =  escrow;
+        uint32 _forkId = forkId;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             uint256 nounId = tokenIds[i];
-            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();
+            if (_escrow.ownerOfEscrowedToken(_forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();

             _mintWithOriginalSeed(msg.sender, nounId);
         }
```

## Cache storage values in memory to minimize SLOADs
The code can be optimized by minimizing the number of SLOADs.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L206-L213
### NounsDAOLogicV2.sol.propose(): `nouns` should be cached(saves 110 gas on average)

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 13546    | 439476   | 380456 | 947962 |
| After  | 13434    | 439366   | 380346 | 947852 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
206:        temp.totalSupply = nouns.totalSupply();

210:        require(
211:            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
212:            'NounsDAO::propose: proposer votes below proposal threshold'
213:        );
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..88a92be0 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -203,12 +203,14 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
     ) public returns (uint256) {
         ProposalTemp memory temp;

-        temp.totalSupply = nouns.totalSupply();
+         NounsTokenLike _nouns = nouns;
+
+        temp.totalSupply = _nouns.totalSupply();

         temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

         require(
-            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
+            _nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
             'NounsDAO::propose: proposer votes below proposal threshold'
         );
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L325-L329
### queueOrRevertInternal(): `timelock` should be cached(save 1 SLOAD: 200 gas)

**Gas benchmarks** based on the function `queue` that calls our function of interest `queueOrRevertInternal()`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 1039    | 101175   | 68684 | 173646 |
| After  | 1039    | 100975   | 68586 | 173254 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
325:        require(
326:            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
327:            'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
328:        );
329:        timelock.queueTransaction(target, value, signature, data, eta);
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..8b880214 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -322,11 +322,12 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         bytes memory data,
         uint256 eta
     ) internal {
+        INounsDAOExecutor _timelock = timelock;
         require(
-            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
+            !_timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
             'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
         );
-        timelock.queueTransaction(target, value, signature, data, eta);
+        _timelock.queueTransaction(target, value, signature, data, eta);
     }
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L389-L396
### NounsDAOLogicV2.sol.veto(): `vetoer` should be cached(saves 92 gas on average)
**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 488    | 25736   | 30324 | 45345 |
| After  | 491    | 25644   | 30239 | 45239 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
389:    function veto(uint256 proposalId) external {
390:        if (vetoer == address(0)) {
391:            revert VetoerBurned();
392:        }

394:        if (msg.sender != vetoer) {
395:            revert VetoerOnly();
396:        }
```

```diff
     function veto(uint256 proposalId) external {
-        if (vetoer == address(0)) {
+        address _vetoer = vetoer;
+        if (_vetoer == address(0)) {
             revert VetoerBurned();
         }

-        if (msg.sender != vetoer) {
+        if (msg.sender != _vetoer) {
             revert VetoerOnly();
         }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L981-L1005
### NounsDAOLogicV2.sol.getDynamicQuorumParamsAt(): `quorumVotesBPS` should be cached(saves 2 SLOADs: 200 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
981:    function getDynamicQuorumParamsAt(uint256 blockNumber_) public view returns (DynamicQuorumParams memory) {

985:        if (len == 0) {
986:            return
987:                DynamicQuorumParams({
988:                    minQuorumVotesBPS: safe16(quorumVotesBPS),
989:                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
990:                    quorumCoefficient: 0
991:                });
992:        }

998:        if (quorumParamsCheckpoints[0].fromBlock > blockNumber) {
999:            return
1000:                DynamicQuorumParams({
1001:                    minQuorumVotesBPS: safe16(quorumVotesBPS),
1002:                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
1003:                    quorumCoefficient: 0
1004:                });
1005:        }
```


```diff
@@ -981,12 +981,14 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
     function getDynamicQuorumParamsAt(uint256 blockNumber_) public view returns (DynamicQuorumParams memory) {
         uint32 blockNumber = safe32(blockNumber_, 'NounsDAO::getDynamicQuorumParamsAt: block number exceeds 32 bits');
         uint256 len = quorumParamsCheckpoints.length;
+

         if (len == 0) {
+            uint16 _quorumVotesBPS  = safe16(quorumVotesBPS);
             return
                 DynamicQuorumParams({
-                    minQuorumVotesBPS: safe16(quorumVotesBPS),
-                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
+                    minQuorumVotesBPS: _quorumVotesBPS,
+                    maxQuorumVotesBPS: _quorumVotesBPS,
                     quorumCoefficient: 0
                 });
         }
@@ -996,10 +998,11 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         }

         if (quorumParamsCheckpoints[0].fromBlock > blockNumber) {
+            uint16 _quorumVotesBPS  = safe16(quorumVotesBPS);
             return
                 DynamicQuorumParams({
-                    minQuorumVotesBPS: safe16(quorumVotesBPS),
-                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
+                    minQuorumVotesBPS: _quorumVotesBPS,
+                    maxQuorumVotesBPS: _quorumVotesBPS,
                     quorumCoefficient: 0
                 });
         }
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L261-L265
### NounsAuctionHouseFork.sol.\_safeTransferETHWithFallback(): `weth` should be cached(save 1 SLOAD: ~97 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
261:    function _safeTransferETHWithFallback(address to, uint256 amount) internal {
262:        if (!_safeTransferETH(to, amount)) {
263:            IWETH(weth).deposit{ value: amount }();
264:            IERC20(weth).transfer(to, amount);
265:        }
```

```diff
     function _safeTransferETHWithFallback(address to, uint256 amount) internal {
         if (!_safeTransferETH(to, amount)) {
-            IWETH(weth).deposit{ value: amount }();
-            IERC20(weth).transfer(to, amount);
+            address _weth = weth;
+            IWETH(_weth).deposit{ value: amount }();
+            IERC20(_weth).transfer(to, amount);
         }
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L777-L779
### NounsDAOLogicV1Fork.sol.adjustedTotalSupply(): `nouns` should be cached(Save 237 gas on average)
**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4709    | 8709   | 8709 | 12709 |
| After  | 4472    | 8472   | 8472 | 12472 |
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
777:    function adjustedTotalSupply() public view returns (uint256) {
778:        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim();
779:    }
```

```diff

     function adjustedTotalSupply() public view returns (uint256) {
-        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim();
+        INounsTokenForkLike _nouns =  nouns;
+        return _nouns.totalSupply() - _nouns.balanceOf(address(timelock)) + _nouns.remainingTokensToClaim();
     }

```

## Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L131-L137
### Local variable `_escrow` should be used instead of reading `escrow`
**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 211357    | 224893   | 211357 | 255180 |
| After  | 211339    | 224875   | 211339 | 255162 |
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
131:        escrow = _escrow;
        
137:        NounsTokenFork originalToken = NounsTokenFork(address(escrow.nounsToken()));
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsT
okenFork.sol
index a1f9d6d3..fea6fad8 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -134,7 +134,7 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
         remainingTokensToClaim = tokensToClaim;
         forkingPeriodEndTimestamp = _forkingPeriodEndTimestamp;

-        NounsTokenFork originalToken = NounsTokenFork(address(escrow.nounsToken()));
+        NounsTokenFork originalToken = NounsTokenFork(address(_escrow.nounsToken()));
         descriptor = originalToken.descriptor();
         seeder = originalToken.seeder();
     }
```

### Global variable `msg.sender` should be used instead of reading state(Save 128 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L240-L260

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 13546    | 439476   | 380456 | 947962 |
| After  | 13456    | 439348   | 380323 | 947829 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
240:        Proposal storage newProposal = _proposals[proposalCount];

242:        newProposal.proposer = msg.sender;

260:        latestProposalIds[newProposal.proposer] = newProposal.id;
```
We are setting `newProposal.proposer` to be equal to `msg.sender`. As `newProposal.proposer` is a state variable, it's a bit expensive to read, we can instead read `msg.sender` which is a global variable , thus more cheaper to read
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..9328d9ce 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -257,7 +257,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         newProposal.totalSupply = temp.totalSupply;
         newProposal.creationBlock = block.number;

-        latestProposalIds[newProposal.proposer] = newProposal.id;
+        latestProposalIds[msg.sender] = newProposal.id;

```


## Emitting storage values instead of the memory one.
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L644-L656
### NounsDAOLogicV2.sol.\_setVotingDelay(): Emit `newVotingDelay` instead of `votingDelay`(save 1 SLOAD: 100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
644:    function _setVotingDelay(uint256 newVotingDelay) external {

652:        uint256 oldVotingDelay = votingDelay;
653:        votingDelay = newVotingDelay;

655:        emit VotingDelaySet(oldVotingDelay, votingDelay);
656:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..c0b85b3f 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -652,7 +652,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldVotingDelay = votingDelay;
         votingDelay = newVotingDelay;

-        emit VotingDelaySet(oldVotingDelay, votingDelay);
+        emit VotingDelaySet(oldVotingDelay, newVotingDelay);
     }

```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L662-L674
### NounsDAOLogicV2.sol.\_setVotingPeriod(): Emit `newVotingPeriod` instead of `votingPeriod`(save 1 SLOAD: 100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
662:    function _setVotingPeriod(uint256 newVotingPeriod) external {

671:        votingPeriod = newVotingPeriod;

673:        emit VotingPeriodSet(oldVotingPeriod, votingPeriod);
674:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..96f8682a 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -670,7 +670,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldVotingPeriod = votingPeriod;
         votingPeriod = newVotingPeriod;

-        emit VotingPeriodSet(oldVotingPeriod, votingPeriod);
+        emit VotingPeriodSet(oldVotingPeriod, newVotingPeriod);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L681-L694
### NounsDAOLogicV2.sol.\_setProposalThresholdBPS(): Emit `newProposalThresholdBPS` instead of `proposalThresholdBPS`(save 1 SLOAD: 100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
681:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {

691:        proposalThresholdBPS = newProposalThresholdBPS;

693:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS);
694:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..0c6ad8cf 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -690,7 +690,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldProposalThresholdBPS = proposalThresholdBPS;
         proposalThresholdBPS = newProposalThresholdBPS;

-        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS);
+        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, newProposalThresholdBPS);
     }
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L904-L909
### Use the cheaper global variable(msg.sender) instead of reading from state when emitting(Save 1 SLOAD: ~100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
904:    function _burnVetoPower() public {
905:        // Check caller is vetoer
906:        require(msg.sender == vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

908:        // Update vetoer to 0x0
909:        emit NewVetoer(vetoer, address(0));
```

```diff
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -906,7 +906,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         require(msg.sender == vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

         // Update vetoer to 0x0
-        emit NewVetoer(vetoer, address(0));
+        emit NewVetoer(msg.sender, address(0));
         vetoer = address(0);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L83-L85
### NounsDAOProxy.sol.\_setImplementation(): Emit `implementation_` instead of `implementation`(save 1 SLOAD: 100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
83:        implementation = implementation_;

85:        emit NewImplementation(oldImplementation, implementation);
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol b/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
index f42e3bf6..e95391cf 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
@@ -82,7 +82,7 @@ contract NounsDAOProxy is NounsDAOProxyStorage, NounsDAOEvents {
         address oldImplementation = implementation;
         implementation = implementation_;

-        emit NewImplementation(oldImplementation, implementation);
+        emit NewImplementation(oldImplementation, implementation_);
     }
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L80-L87
### NounsDAOExecutor.sol.setDelay(): Emit `delay_` instead of `delay`(save 1 SLOAD: 100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
80:    function setDelay(uint256 delay_) public {

84:        delay = delay_;
        
86:        emit NewDelay(delay);
87:    }
```

```diff
         require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
         delay = delay_;

-        emit NewDelay(delay);
+        emit NewDelay(delay_);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L89-L95
### NounsDAOExecutor.sol.acceptAdmin(): Emit `msg.sender` instead of `admin`(save 1 SLOAD: 100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
89:    function acceptAdmin() public {

91:        admin = msg.sender;
     
94:        emit NewAdmin(admin);
```

```diff
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -91,7 +91,7 @@ contract NounsDAOExecutor {
         admin = msg.sender;
         pendingAdmin = address(0);

-        emit NewAdmin(admin);
+        emit NewAdmin(msg.sender);

```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L97-L105
### NounsDAOExecutor.sol.setPendingAdmin(): Emit `pendingAdmin_` instead of `pendingAdmin`(save 1 SLOAD: 100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
97:    function setPendingAdmin(address pendingAdmin_) public {

102:        pendingAdmin = pendingAdmin_;

104:        emit NewPendingAdmin(pendingAdmin);
105:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..1f23d7c4 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -101,7 +101,7 @@ contract NounsDAOExecutor {
         );
         pendingAdmin = pendingAdmin_;

-        emit NewPendingAdmin(pendingAdmin);
+        emit NewPendingAdmin(pendingAdmin_);
```


## Optimizing check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L143-L161
### Cheaper require statements should be performed first
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
143:    ) public virtual {
144:        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
145:        if (msg.sender != admin) {
146:            revert AdminOnly();
147:        }
148:        require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
149:        require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
150:        require(
151:            votingPeriod_ >= MIN_VOTING_PERIOD && votingPeriod_ <= MAX_VOTING_PERIOD,
152:            'NounsDAO::initialize: invalid voting period'
153:        );
154:        require(
155:            votingDelay_ >= MIN_VOTING_DELAY && votingDelay_ <= MAX_VOTING_DELAY,
156:            'NounsDAO::initialize: invalid voting delay'
157:        );
158:        require(
159:            proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,
160:            'NounsDAO::initialize: invalid proposal threshold bps'
161:        );
```
In the above function, the first sanity checks(require statements) validates some state variables ie Involves reading from state which is quite expensive(2100 Cold ,100 warm). In this case we read `timelock` and `admin` . In case of a revert on other cheaper checks eg function parameter checks, we would end up wasting the gas spent validating the state variables ~4000 gas. To minimize this cost, we can first validate the variables that are cheaper to check. Reorder the checks as follows.
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..1f02f8dd 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -141,10 +141,6 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 proposalThresholdBPS_,
         DynamicQuorumParams calldata dynamicQuorumParams_
     ) public virtual {
-        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
         require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
         require(
@@ -159,6 +155,10 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,
             'NounsDAO::initialize: invalid proposal threshold bps'
         );
+        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }

```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L204-L221
### External calls + state reads are very expensive. Validate local/functional parameters first
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
204:        ProposalTemp memory temp;

206:        temp.totalSupply = nouns.totalSupply();

208:        temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

210:        require(
211:            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
212:            'NounsDAO::propose: proposer votes below proposal threshold'
213:        );
214:        require(
215:            targets.length == values.length &&
216:                targets.length == signatures.length &&
217:                targets.length == calldatas.length,
218:            'NounsDAO::propose: proposal function information arity mismatch'
219:        );
220:        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
221:        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

In our function above, we start by making some external function calls `nouns.totalSupply()` and reading from state `proposalThresholdBPS`. We then do some validation for some function parameters. As reading function parameters is cheaper, we should validate them first , so that if they don't pass our check, we can revert early and without too much gas
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..15ecc0c6 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -201,6 +201,15 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         bytes[] memory calldatas,
         string memory description
     ) public returns (uint256) {
+        require(
+            targets.length == values.length &&
+                targets.length == signatures.length &&
+                targets.length == calldatas.length,
+            'NounsDAO::propose: proposal function information arity mismatch'
+        );
+        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
+        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
+
         ProposalTemp memory temp;

         temp.totalSupply = nouns.totalSupply();
@@ -211,14 +220,6 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
             'NounsDAO::propose: proposer votes below proposal threshold'
         );
-        require(
-            targets.length == values.length &&
-                targets.length == signatures.length &&
-                targets.length == calldatas.length,
-            'NounsDAO::propose: proposal function information arity mismatch'
-        );
-        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
-        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L615-L617
### Validate function parameter first before reading from state
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
615:    ) internal returns (uint96) {
616:        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
617:        require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
```
As `support` is a function parameter, it would be cheaper to validate it first before reading the state variable `ProposalState` 
In case of a revert on the check `support <= 2` we would not end up wasting too much gas validating the state variable
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..2c5034df 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -613,8 +613,8 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 proposalId,
         uint8 support
     ) internal returns (uint96) {
-        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
         require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
+        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L644-L651
### Avoid validating state variables before validating function parameters
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
644:    function _setVotingDelay(uint256 newVotingDelay) external {
645:        if (msg.sender != admin) {
646:            revert AdminOnly();
647:        }
648:        require(
649:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
650:            'NounsDAO::_setVotingDelay: invalid voting delay'
651:        );
```
In the above, the second check reads function parameters and constants. The first check reads from state. As it it more gas efficient to read function parameters and constants, we should validate it first.
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..52a3bb63 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -642,13 +642,13 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
      * @param newVotingDelay new voting delay, in blocks
      */
     function _setVotingDelay(uint256 newVotingDelay) external {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         require(
             newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
             'NounsDAO::_setVotingDelay: invalid voting delay'
         );
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L663-L669
### Validate function parameters first before reading any state variables
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
663:        if (msg.sender != admin) {
664:            revert AdminOnly();
665:        }
666:        require(
667:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
668:            'NounsDAO::_setVotingPeriod: invalid voting period'
669:        );
```
Reorder the checks to validate function parameters first as they are cheaper to read compared to state variables
```diff
     function _setVotingPeriod(uint256 newVotingPeriod) external {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         require(
             newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
             'NounsDAO::_setVotingPeriod: invalid voting period'
         );
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L681-L689
### Validate Function parameter  before validating state variables
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
681:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
682:        if (msg.sender != admin) {
683:            revert AdminOnly();
684:        }
685:        require(
686:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
687:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
688:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
689:        );
```
In case of a revert on the function parameter validation, we might save some good amount of gas (SLOAD) that would have been used in reading `admin` which is a state variable
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..2a9aca20 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -679,14 +679,14 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
      * @param newProposalThresholdBPS new proposal threshold
      */
     function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         require(
             newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
                 newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
             'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
         );
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L702-L711
### Reorder the checks here to have cheaper checks first
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
702:    function _setMinQuorumVotesBPS(uint16 newMinQuorumVotesBPS) external {
703:        if (msg.sender != admin) {
704:            revert AdminOnly();
705:        }
706:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);

708:        require(
709:            newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
710:                newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
711:            'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
```
We have a check for function parameters against some constants variables. As this is a cheaper check, it should be done before any other check or any other operations.
Move the check to the beginning
```diff
     function _setMinQuorumVotesBPS(uint16 newMinQuorumVotesBPS) external {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
-        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
-
         require(
             newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
                 newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
             'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
         );
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
+        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
+
         require(
             newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
             'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L732-L741
### Function parameters should be validated first
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
732:    function _setMaxQuorumVotesBPS(uint16 newMaxQuorumVotesBPS) external {
733:        if (msg.sender != admin) {
734:            revert AdminOnly();
735:        }
736:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);

738:        require(
739:            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
740:            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
741:        );
```
Consider validating the function parameter `newMaxQuorumVotesBPS` before reading from state . The first check `msg.sender != admin` involves reading a state variable `admin`. Reading from state is expensive. As it is, in case of a revert on the require statement on line 738, the gas consumed reading from state would be wasted
```diff
     function _setMaxQuorumVotesBPS(uint16 newMaxQuorumVotesBPS) external {
+        require(
+            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
+            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
+        );
         if (msg.sender != admin) {
             revert AdminOnly();
         }
         DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);

-        require(
-            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
-            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
-        );
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L783-L802
### Validate all function parameters first before reading any state variables
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
783:    function _setDynamicQuorumParams(
784:        uint16 newMinQuorumVotesBPS,
785:        uint16 newMaxQuorumVotesBPS,
786:        uint32 newQuorumCoefficient
787:    ) public {
788:        if (msg.sender != admin) {
789:            revert AdminOnly();
790:        }
791:        if (
792:            newMinQuorumVotesBPS < MIN_QUORUM_VOTES_BPS_LOWER_BOUND ||
793:            newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND
794:        ) {
795:            revert InvalidMinQuorumVotesBPS();
796:        }
797:        if (newMaxQuorumVotesBPS > MAX_QUORUM_VOTES_BPS_UPPER_BOUND) {
798:            revert InvalidMaxQuorumVotesBPS();
799:        }
800:        if (newMinQuorumVotesBPS > newMaxQuorumVotesBPS) {
801:            revert MinQuorumBPSGreaterThanMaxQuorumBPS();
802:        }
```

```diff
@@ -785,9 +785,6 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint16 newMaxQuorumVotesBPS,
         uint32 newQuorumCoefficient
     ) public {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         if (
             newMinQuorumVotesBPS < MIN_QUORUM_VOTES_BPS_LOWER_BOUND ||
             newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND
@@ -800,7 +797,10 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         if (newMinQuorumVotesBPS > newMaxQuorumVotesBPS) {
             revert MinQuorumBPSGreaterThanMaxQuorumBPS();
         }
-
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
+
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L78-L80
### Validate function parameters before validating state variables
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
78:    function _setImplementation(address implementation_) public {
79:        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
80:        require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');
```
The first check, `msg.sender == admin` involves reading from state `admin` which is a bit expensive. The second check, however validates that a function parameter `implementation_` != `address(0)`. As the second check is cheaper , it should be done first so that in case it reverts , no gas would be wasted reading from state
```diff
     function _setImplementation(address implementation_) public {
-        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
         require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');
-
+        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
+
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L150-L153
### Function parameters should be checked first before reading any state variables
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
150:        if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
151:        if (msg.sender != ds.admin) revert AdminOnly();
152:        if (timelock_ == address(0)) revert InvalidTimelockAddress();
153:        if (nouns_ == address(0)) revert InvalidNounsAddress();
```
The first two checks involves reading some state variables ie `ds.timelock` and `ds.admin`. The next two simply checks some local variables(function parameters). Reading function parameters is cheaper than state variables and in case of a revert on the function parameter check, we would end up wasting too much gas on reading state. We should reorder the checks to have cheaper checks first
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
index 00c5ccdc..fee29faa 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
@@ -147,10 +147,10 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
         NounsDAOParams calldata daoParams_,
         DynamicQuorumParams calldata dynamicQuorumParams_
     ) public virtual {
-        if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
-        if (msg.sender != ds.admin) revert AdminOnly();
         if (timelock_ == address(0)) revert InvalidTimelockAddress();
         if (nouns_ == address(0)) revert InvalidNounsAddress();
+        if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
+        if (msg.sender != ds.admin) revert AdminOnly();

```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L351-L365
### Validate parameters before making external function calls
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
351:    function _setMinQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMinQuorumVotesBPS)
352:        external
353:        onlyAdmin(ds)
354:    {
355:        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);

357:        require(
358:            newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
359:                newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
360:            'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
361:        );
362:        require(
363:            newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
364:            'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
365:        );
```
We make an external call `getDynamicQuorumParamsAt(block.number)` then we make some checks  for the function parameters. If the parameters don't meet our requirements, we would end up reverting. In case of a revert, it would mean that the gas spent making the external function call was wasted. We should reorder this checks to validate the parameters first before doing the external calls. This way, a revert on the parameter check would not waste gas doing the external call
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..c2218691 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
@@ -352,13 +352,13 @@ library NounsDAOV3Admin {
         external
         onlyAdmin(ds)
     {
-        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
-
         require(
             newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
                 newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
             'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
         );
+        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
+
         require(
             newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
             'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L381-L390
### Avoid making external calls before validating function parameters
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
381:    function _setMaxQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMaxQuorumVotesBPS)
382:        external
383:        onlyAdmin(ds)
384:    {
385:        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);

387:        require(
388:            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
389:            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
390:        );
```
In the above function, we are making an external `getDynamicQuorumParamsAt(block.number)` then we do some validity checks. In case of a revert when validating the function parameter `newMaxQuorumVotesBPS`  the gas used making the external call would be wasted. Perform all validations first before making any external calls
```diff
-        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
-
         require(
             newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
             'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
         );
+        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
+
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L176-L178
### Validate function parameters before validating state variables
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
176:        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
177:        require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
178:        require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
```
If we end up reverting on the checks for function parameters(`timelock_` and `nouns_`) the gas spent reading the state variable `timelock` would be wasted. As SLOAD are quite expensive, it would be wise to first validate the cheaper variables
```diff
         __ReentrancyGuard_init_unchained();
-        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
         require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
         require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
+        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L279-L298
### Validate all function parameters first before doing other operations 
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
279:        checkGovernanceActive();

281:        ProposalTemp memory temp;

283:        temp.totalSupply = adjustedTotalSupply();

285:        temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

287:        require(
288:            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
289:            'NounsDAO::propose: proposer votes below proposal threshold'
290:        );
291:        require(
292:            targets.length == values.length &&
293:                targets.length == signatures.length &&
294:                targets.length == calldatas.length,
295:            'NounsDAO::propose: proposal function information arity mismatch'
296:        );
297:        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
298:        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

```diff
+        require(
+            targets.length == values.length &&
+                targets.length == signatures.length &&
+                targets.length == calldatas.length,
+            'NounsDAO::propose: proposal function information arity mismatch'
+        );
+        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
+        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
+
         checkGovernanceActive();

         ProposalTemp memory temp;
@@ -288,14 +297,6 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
             nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
             'NounsDAO::propose: proposer votes below proposal threshold'
         );
-        require(
-            targets.length == values.length &&
-                targets.length == signatures.length &&
-                targets.length == calldatas.length,
-            'NounsDAO::propose: proposal function information arity mismatch'
-        );
-        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
-        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L617-L618
### Reorder the checks to have the cheaper checks first
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
617:        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
618:        require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
```
The first check involves reading a state variable while the second check just reads a function parameter. To minimize gas consumed in case of a revert on the second check, we should do it first as it's cheaper
```diff
-        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
         require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
+        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L646-L650
### Validations that involve state reads should not be done at the beginning if we have other cheaper checks
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
646:        require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
647:        require(
648:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
649:            'NounsDAO::_setVotingDelay: invalid voting delay'
650:        );
```
The second check is cheaper as it just reads a function parameter and a constant variable.
```diff
     function _setVotingDelay(uint256 newVotingDelay) external {
-        require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
         require(
             newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
             'NounsDAO::_setVotingDelay: invalid voting delay'
         );
+        require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
         uint256 oldVotingDelay = votingDelay;
         votingDelay = newVotingDelay;
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L662-L666
### Reorder the checks to have cheaper checks first
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
662:        require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
663:        require(
664:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
665:            'NounsDAO::_setVotingPeriod: invalid voting period'
666:        );
```

```diff
     function _setVotingPeriod(uint256 newVotingPeriod) external {
-        require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
         require(
             newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
             'NounsDAO::_setVotingPeriod: invalid voting period'
         );
+        require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
         uint256 oldVotingPeriod = votingPeriod;
         votingPeriod = newVotingPeriod;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L166-L185
### Any variable checks should be done first
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
166:    function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
167:        uint256 currentNounId = _currentNounId;
168:        uint256 maxNounId = 0;
169:        if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
170:        if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
```
Instead of reading from states and performing some other variable initialization, we should do the if checks first. This way we can revert early and cheaply if our conditions are not met
```diff
     function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
-        uint256 currentNounId = _currentNounId;
-        uint256 maxNounId = 0;
         if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
         if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
+        uint256 currentNounId = _currentNounId;
+        uint256 maxNounId = 0;

         for (uint256 i = 0; i < tokenIds.length; i++) {
             uint256 nounId = tokenIds[i];
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L145-L154
### Reorder the checks here to avoid wasting gas on later reverts
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
145:        require(delegatee != address(0), 'ERC721Checkpointable::delegateBySig: delegatee cannot be zero address');

154:        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');
```
The require check on line 154 basically validates a function parameter which is not that gas intensive. Before we hit the check, we are performing some other operations which are gas intensive. If we do revert on the check on line 154, the gas spent doing other operations would just be wasted. We can reorder the operations to perform the parameter checks earlier.
```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
index 47aabf7c..b2d79195 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
@@ -143,6 +143,7 @@ abstract contract ERC721CheckpointableUpgradeable is ERC721EnumerableUpgradeable
         bytes32 s
     ) public {
         require(delegatee != address(0), 'ERC721Checkpointable::delegateBySig: delegatee cannot be zero address');
+        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');
         bytes32 domainSeparator = keccak256(
             abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name())), block.chainid, address(this))
         );
@@ -151,7 +152,6 @@ abstract contract ERC721CheckpointableUpgradeable is ERC721EnumerableUpgradeable
         address signatory = ecrecover(digest, v, r, s);
         require(signatory != address(0), 'ERC721Checkpointable::delegateBySig: invalid signature');
         require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');
-        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');
         return _delegate(signatory, delegatee);
     }
```

## The following functions can benefit from some optimizations

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L854-L870
## We can  optimize the function `_acceptAdmin`(Save 4 SLOADS: ~400 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
854:    function _acceptAdmin() external {
855:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
856:        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

858:        // Save current values for inclusion in log
859:        address oldAdmin = admin;
860:        address oldPendingAdmin = pendingAdmin;

862:        // Store admin with value pendingAdmin
863:        admin = pendingAdmin;

865:        // Clear the pending value
866:        pendingAdmin = address(0);

868:        emit NewAdmin(oldAdmin, admin);
869:        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
870:    }
```
The first thing the function does is validate that `msg.sender== pendingAdmin` which means that to proceed with execution the state variable `pendingAdmin` should be equal to `msg.sender`. As such instead of reading  the state variable `pendingAdmin` on the next operations, we can replace it's occurrence with the cheaper global variable `msg.sender`

On line 866, we set `pendingAdmin = address(0)` , we then proceed to emit an event that has the state variable `pendingAdmin`, we can refactor this emit to emit the `address(0)` instead as we already know that's the value of the state variable.
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..1cd99181 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -857,16 +857,16 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {

         // Save current values for inclusion in log
         address oldAdmin = admin;
-        address oldPendingAdmin = pendingAdmin;
+        address oldPendingAdmin = msg.sender;

         // Store admin with value pendingAdmin
-        admin = pendingAdmin;
+        admin = msg.sender;

         // Clear the pending value
         pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, admin);
-        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
+        emit NewAdmin(oldAdmin, msg.sender);
+        emit NewPendingAdmin(oldPendingAdmin, address(0));
     }

```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L886-L898
### We can optimize the function `_acceptVetoer()` Save 3 SLOADS: ~300 gas
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
886:    function _acceptVetoer() external {
887:        if (msg.sender != pendingVetoer) {
888:            revert PendingVetoerOnly();
889:        }

891:        // Update vetoer
892:        emit NewVetoer(vetoer, pendingVetoer);
893:        vetoer = pendingVetoer;

895:        // Clear the pending value
896:        emit NewPendingVetoer(pendingVetoer, address(0));
897:        pendingVetoer = address(0);
898:    }
```

The first check ensures that `msg.sender` is equal to `pendingVetoer` and reverts if not. Thus executing other operations on this function is guaranteed that the `msg.sender` is same as `pendingVetoer`. As `pendingVetoer` is a state variable, we can replace it's occurrence with `msg.sender` which is a global variable thus more cheaper to read.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..7be11cf8 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -889,11 +889,11 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         }

         // Update vetoer
-        emit NewVetoer(vetoer, pendingVetoer);
-        vetoer = pendingVetoer;
+        emit NewVetoer(vetoer, msg.sender);
+        vetoer = msg.sender;

         // Clear the pending value
-        emit NewPendingVetoer(pendingVetoer, address(0));
+        emit NewPendingVetoer(msg.sender, address(0));
         pendingVetoer = address(0);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L276-L295
### We can optimize the function `_acceptAdmin()` Save 3 SLOADS: ~300 gas
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
276:    function _acceptAdmin(NounsDAOStorageV3.StorageV3 storage ds) external {
277:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
278:        require(
279:            msg.sender == ds.pendingAdmin && msg.sender != address(0),
280:            'NounsDAO::_acceptAdmin: pending admin only'
281:        );

283:        // Save current values for inclusion in log
284:        address oldAdmin = ds.admin;
285:        address oldPendingAdmin = ds.pendingAdmin;

287:        // Store admin with value pendingAdmin
288:        ds.admin = ds.pendingAdmin;

290:        // Clear the pending value
291:        ds.pendingAdmin = address(0);

293:        emit NewAdmin(oldAdmin, ds.admin);
294:        emit NewPendingAdmin(oldPendingAdmin, address(0));
295:    }
```
We are checking that `msg.sender == ds.pendingAdmin` and reverting if not. Instead of reading `ds.pendingAdmin` which is  a state variable in next operations, we can instead read `msg.sender` which is a global variable hence more cheaper in terms of gas
We then set `ds.admin` equal to `ds.pendingAdmin`(which we've established is equal to `msg.sender`). On the emit block, instead of emitting `ds.admin` we might as well just emit `msg.sender`

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..452e89b4 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
@@ -282,15 +282,15 @@ library NounsDAOV3Admin {

         // Save current values for inclusion in log
         address oldAdmin = ds.admin;
-        address oldPendingAdmin = ds.pendingAdmin;
+        address oldPendingAdmin = msg.sender;

         // Store admin with value pendingAdmin
-        ds.admin = ds.pendingAdmin;
+        ds.admin = msg.sender;

         // Clear the pending value
         ds.pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, ds.admin);
+        emit NewAdmin(oldAdmin, msg.sender);
         emit NewPendingAdmin(oldPendingAdmin, address(0));
     }
```



https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L314-L326
### `_acceptVetoer()` can be optimized(Save 3 SLOADs: ~300 gas )
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
314:    function _acceptVetoer(NounsDAOStorageV3.StorageV3 storage ds) external {
315:        if (msg.sender != ds.pendingVetoer) {
316:            revert PendingVetoerOnly();
317:        }

319:        // Update vetoer
320:        emit NewVetoer(ds.vetoer, ds.pendingVetoer);
321:        ds.vetoer = ds.pendingVetoer;

323:        // Clear the pending value
324:        emit NewPendingVetoer(ds.pendingVetoer, address(0));
325:        ds.pendingVetoer = address(0);
326:    }
```
The first check ensures that `msg.sender`  is same as `ds.pendingVetoer` and reverts the call if not.  We can therefore save some gas by replacing the `ds.pendingVetoer` variable access with `msg.sender`
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..f7fe6acd 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
@@ -317,11 +317,11 @@ library NounsDAOV3Admin {
         }

         // Update vetoer
-        emit NewVetoer(ds.vetoer, ds.pendingVetoer);
-        ds.vetoer = ds.pendingVetoer;
+        emit NewVetoer(ds.vetoer, msg.sender);
+        ds.vetoer = msg.sender;

         // Clear the pending value
-        emit NewPendingVetoer(ds.pendingVetoer, address(0));
+        emit NewPendingVetoer(msg.sender, address(0));
         ds.pendingVetoer = address(0);
     }
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L332-L337
### Cheaper to read global variable compared to state variable(Save 1 SLOAD: ~100 gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
332:    function _burnVetoPower(NounsDAOStorageV3.StorageV3 storage ds) public {
333:        // Check caller is vetoer
334:        require(msg.sender == ds.vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

336:        // Update vetoer to 0x0
337:        emit NewVetoer(ds.vetoer, address(0));
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..1f747a58 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
@@ -334,7 +334,7 @@ library NounsDAOV3Admin {
         require(msg.sender == ds.vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

         // Update vetoer to 0x0
-        emit NewVetoer(ds.vetoer, address(0));
+        emit NewVetoer(msg.sender, address(0));
         ds.vetoer = address(0);
```


https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L731-L747
### We can optimize the function `_acceptAdmin()` (Save 4 SLOADs: ~400 Gas)
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
731:    function _acceptAdmin() external {
732:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
733:        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

735:        // Save current values for inclusion in log
736:        address oldAdmin = admin;
737:        address oldPendingAdmin = pendingAdmin;

739:        // Store admin with value pendingAdmin
740:        admin = pendingAdmin;

742:        // Clear the pending value
743:        pendingAdmin = address(0);

745:        emit NewAdmin(oldAdmin, admin);
746:        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
747:    }
```


```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..7351d0d9 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -734,16 +734,16 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou

         // Save current values for inclusion in log
         address oldAdmin = admin;
-        address oldPendingAdmin = pendingAdmin;
+        address oldPendingAdmin = msg.sender;

         // Store admin with value pendingAdmin
-        admin = pendingAdmin;
+        admin = msg.sender;

         // Clear the pending value
         pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, admin);
-        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
+        emit NewAdmin(oldAdmin, msg.sender);
+        emit NewPendingAdmin(oldPendingAdmin, address(0));
     }
```


## Nested if is cheaper than single statement
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol#L243-L245
```solidity
File: /packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol
243:        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
244:            checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
245        } else {          }
```

```diff

-        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
-            checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
+        if (nCheckpoints > 0) {
+            if( checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
+                 checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
+            }
         } else {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L381-L383
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
381:        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
382:            revert WaitingForTokensToClaimOrExpiration();
383:        }
```

```diff
-        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
-            revert WaitingForTokensToClaimOrExpiration();
+        if (block.timestamp < delayedGovernanceExpirationTimestamp) {
+            if ( nouns.remainingTokensToClaim() > 0) {
+                revert WaitingForTokensToClaimOrExpiration();
+            }
```

## Caching a variable that is used once just wastes Gas
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L141-L160
### No need to cache `ds.forkEscrow` as it's being used once
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
141:    function joinFork(
142:        NounsDAOStorageV3.StorageV3 storage ds,
143:        uint256[] calldata tokenIds,
144:        uint256[] calldata proposalIds,
145:        string calldata reason
146:    ) external {
147:        if (!isForkPeriodActive(ds)) revert ForkPeriodNotActive();

149:        INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
150:        address timelock = address(ds.timelock);
151:        sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, adjustedTotalSupply(ds));

153:        for (uint256 i = 0; i < tokenIds.length; i++) {
154:            ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]);
155:        }

157:        NounsTokenFork(ds.forkDAOToken).claimDuringForkPeriod(msg.sender, tokenIds);

159:        emit JoinFork(forkEscrow.forkId() - 1, msg.sender, tokenIds, proposalIds, reason);
160:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol b/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
index d87ffc70..4051da05 100644
--- a/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
@@ -146,7 +146,6 @@ library NounsDAOV3Fork {
     ) external {
         if (!isForkPeriodActive(ds)) revert ForkPeriodNotActive();

-        INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
         address timelock = address(ds.timelock);
         sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, adjustedTotalSupply(ds));

@@ -156,7 +155,7 @@ library NounsDAOV3Fork {

         NounsTokenFork(ds.forkDAOToken).claimDuringForkPeriod(msg.sender, tokenIds);

-        emit JoinFork(forkEscrow.forkId() - 1, msg.sender, tokenIds, proposalIds, reason);
+        emit JoinFork(ds.forkEscrow.forkId() - 1, msg.sender, tokenIds, proposalIds, reason);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L166-L185
### NounsTokenFork.sol.claimDuringForkPeriod(): `_currentNounId` should not be cached
The variable `currentNounId` is being once, as such no need to cache
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
166:    function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
167:        uint256 currentNounId = _currentNounId;

184:        if (maxNounId >= currentNounId) _currentNounId = maxNounId + 1;
```

## Importing an entire library while only using one function isn't necessary
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L22
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
22:import { SafeCast } from '@openzeppelin/contracts/utils/math/SafeCast.sol';

252:            proposal.objectionPeriodEndBlock = SafeCast.toUint64(
253:                proposal.endBlock + ds.objectionPeriodDurationInBlocks
254:            );
```

We import the entire library `SafeCast` yet we only need to utilize one function from it ie `toUint64()`. Peeking into it's implementation from Openzeppelin we have the following
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7ccea54dc15856d0e6c3b61b829b85d9e52195cd/contracts/utils/math/SafeCast.sol#L441-L446
```solidity
File: /contracts/utils/math/SafeCast.sol
441:    function toUint64(uint256 value) internal pure returns (uint64) {
442:        if (value > type(uint64).max) {
443:            revert SafeCastOverflowedUintDowncast(64, value);
444:        }
445:        return uint64(value);
446:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
index 3743132b..782a2cf2 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
@@ -19,13 +19,14 @@ pragma solidity ^0.8.19;

 import './NounsDAOInterfaces.sol';
 import { NounsDAOV3Proposals } from './NounsDAOV3Proposals.sol';
-import { SafeCast } from '@openzeppelin/contracts/utils/math/SafeCast.sol';

 library NounsDAOV3Votes {
     using NounsDAOV3Proposals for NounsDAOStorageV3.StorageV3;

     error CanOnlyVoteAgainstDuringObjectionPeriod();

+    error SafeCastOverflowedUintDowncast();
+
     /// @notice An event emitted when a vote has been cast on a proposal
     /// @param voter The address which casted a vote
     /// @param proposalId The proposal id which was voted on
@@ -249,9 +250,11 @@ library NounsDAOV3Votes {
             // second part of the vote flip check
             !ds.isDefeated(proposal)
         ) {
-            proposal.objectionPeriodEndBlock = SafeCast.toUint64(
-                proposal.endBlock + ds.objectionPeriodDurationInBlocks
-            );
+
+            if (proposal.endBlock + ds.objectionPeriodDurationInBlocks  > type(uint64).max) {
+                        revert SafeCastOverflowedUintDowncast();
+            }
+            proposal.objectionPeriodEndBlock = uint64(proposal.endBlock + ds.objectionPeriodDurationInBlocks);

             emit ProposalObjectionPeriodSet(proposal.id, proposal.objectionPeriodEndBlock);
         }
```

Alternatively we can implement our own internal function to do the safeCast


## Conclusion
It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
