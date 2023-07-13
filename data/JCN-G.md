# Summary

A majority of the optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.19`, `optimizer on`, and `200 runs`. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes. The following command was used to run the tests for the benchmarks below: `forge test --ffi --gas-report`.

*Notes*: 

- Only optimizations to state-mutating functions and view/pure functions invoked by state-mutating functions are highlighted below.
- Only runtime gas is highlighted below, as it will inevitably outweight deployment gas costs throughout the lifetime of the protocol.
- Some code snippets may be truncated to save space. Code snippets may also be accompanied by @audit tags in comments to aid in explaining the issue.

## Gas Optimizations
| Number |Issue|Instances|Estimated Gas Saved|
|-|:-|:-:|:-:| 
| [G-01](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 39 | 3900 |
| [G-02](#read-memory-values-instead-of-reading-from-storage) | Read `memory` values instead of reading from storage | 12 | 1200 |
| [G-03](#state-variables-can-be-packed-into-fewer-storage-slots) | State variables can be packed into fewer storage slots | 17 | 32000 |
| [G-04](#structs-can-be-packed-into-fewer-storage-slots) | Structs can be packed into fewer storage slots | 2 | 16000 |
| [G-05](#cache-state-variables-outside-of-loop-to-avoid-readingwriting-storage-on-every-iteration) | Cache state variables outside of loop to avoid reading/writing storage on every iteration | 11 | 2280 |
| [G-06](#avoid-emitting-storage-variables) | Avoid emitting storage variables | 8 | 800 |
| [G-07](#refactor-functions-to-avoid-unnecessary-sloads-and-sstores) | Refactor functions to avoid unnecessary SLOADs and SSTOREs | 6 | 1812 |
| [G-08](#refactor-functions-to-avoid-redundant-external-calls) | Refactor functions to avoid redundant External Calls | 3 | 5230 |
| [G-09](#use-calldata-instead-of-memory-for-function-parameters) | Use calldata instead of memory for function parameters | 12 | 12361 |

**Total gas saved across all listed functions: 75583**

## State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read.

Total Instances: `39`

Estimated Gas Saved: `39 * 100 = 3900`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L316-L367

### Cache  `proposalCount + 1` to save 6 SLOADs for the following function: `NounsDAOLogicV1Fork.proopose`
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
316:        proposalCount++; // @audit: 1st sload
317:        Proposal storage newProposal = _proposals[proposalCount]; // @audit: 2nd sload
318:
319:        newProposal.id = proposalCount; // @audit: 3rd sload
...
337:        latestProposalIds[newProposal.proposer] = newProposal.id; // @audit: 4th sload `newProposal.id == proposalCount`
338:
339:        /// @notice Maintains backwards compatibility with GovernorBravo events
340:        emit ProposalCreated(
341:            newProposal.id, // @audit: 5th sload `newProposal.id == proposalCount`
...
352:        /// @notice Updated event with `proposalThreshold` and `quorumVotes`
353:        emit ProposalCreatedWithRequirements(
354:            newProposal.id, // @audit: 6th sload `newProposal.id == proposalCount`
...
367:        return newProposal.id; // @audit: 7th sload `newProposal.id == proposalCount`
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..cd8357ed 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -312,11 +312,12 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou

         temp.startBlock = block.number + votingDelay;
         temp.endBlock = temp.startBlock + votingPeriod;
+
+        uint256 newProposalCount = proposalCount + 1;
+        proposalCount = newProposalCount;
+        Proposal storage newProposal = _proposals[newProposalCount];

-        proposalCount++;
-        Proposal storage newProposal = _proposals[proposalCount];
-
-        newProposal.id = proposalCount;
+        newProposal.id = newProposalCount;
         newProposal.proposer = msg.sender;
         newProposal.proposalThreshold = temp.proposalThreshold;
         newProposal.quorumVotes = bps2Uint(quorumVotesBPS, temp.totalSupply);
@@ -334,11 +335,11 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         newProposal.executed = false;
         newProposal.creationBlock = block.number;

-        latestProposalIds[newProposal.proposer] = newProposal.id;
+        latestProposalIds[newProposal.proposer] = newProposalCount;

         /// @notice Maintains backwards compatibility with GovernorBravo events
         emit ProposalCreated(
-            newProposal.id,
+            newProposalCount,
             msg.sender,
             targets,
             values,
@@ -351,7 +352,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou

         /// @notice Updated event with `proposalThreshold` and `quorumVotes`
         emit ProposalCreatedWithRequirements(
-            newProposal.id,
+            newProposalCount,
             msg.sender,
             targets,
             values,
@@ -364,7 +365,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
             description
         );

-        return newProposal.id;
+        return newProposalCount;
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L239-L291

### Cache  `proposalCount + 1` to save 6 SLOADs
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
239:        proposalCount++; // @audit: 1st sload
240:        Proposal storage newProposal = _proposals[proposalCount]; // @audit: 2nd sload
241:        newProposal.id = proposalCount; // @audit: 3rd sload
...
260:        latestProposalIds[newProposal.proposer] = newProposal.id; // @audit: 4th sload `newProposal.id == proposalCount`
261:
262:        /// @notice Maintains backwards compatibility with GovernorBravo events
263:        emit ProposalCreated(
264:            newProposal.id, // @audit: 5th sload `newProposal.id == proposalCount`
...
275:        /// @notice Updated event with `proposalThreshold` and `minQuorumVotes`
276:        /// @notice `minQuorumVotes` is always zero since V2 introduces dynamic quorum with checkpoints
277:        emit ProposalCreatedWithRequirements(
278:            newProposal.id, // @audit: 6th sload `newProposal.id == proposalCount`
...
291:        return newProposal.id; // @audit: 7th sload `newProposal.id == proposalCount`
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..bdc9df7e 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -235,10 +235,11 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {

         temp.startBlock = block.number + votingDelay;
         temp.endBlock = temp.startBlock + votingPeriod;
-
-        proposalCount++;
-        Proposal storage newProposal = _proposals[proposalCount];
-        newProposal.id = proposalCount;
+
+        uint256 newProposalCount = proposalCount + 1;
+        proposalCount = newProposalCount;
+        Proposal storage newProposal = _proposals[newProposalCount];
+        newProposal.id = newProposalCount;
         newProposal.proposer = msg.sender;
         newProposal.proposalThreshold = temp.proposalThreshold;
         newProposal.eta = 0;
@@ -257,11 +258,11 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         newProposal.totalSupply = temp.totalSupply;
         newProposal.creationBlock = block.number;

-        latestProposalIds[newProposal.proposer] = newProposal.id;
+        latestProposalIds[newProposal.proposer] = newProposalCount;

         /// @notice Maintains backwards compatibility with GovernorBravo events
         emit ProposalCreated(
-            newProposal.id,
+            newProposalCount,
             msg.sender,
             targets,
             values,
@@ -275,7 +276,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         /// @notice Updated event with `proposalThreshold` and `minQuorumVotes`
         /// @notice `minQuorumVotes` is always zero since V2 introduces dynamic quorum with checkpoints
         emit ProposalCreatedWithRequirements(
-            newProposal.id,
+            newProposalCount,
             msg.sender,
             targets,
             values,
@@ -288,7 +289,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             description
         );

-        return newProposal.id;
+        return newProposalCount;
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L318-L330

### Cache `timelock` to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
318:    function queueOrRevertInternal(
319:        address target,
320:        uint256 value,
321:        string memory signature,
322:        bytes memory data,
323:        uint256 eta
324:    ) internal {
325:        require(
326:            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))), // @audit: 1st sload
327:            'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
328:        );
329:        timelock.queueTransaction(target, value, signature, data, eta); // @audit: 2nd sload
330:    }
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..91605bce 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
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

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L854-L863

### Cache `pendingAdmin` to save 2 SLOADs
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
854:    function _acceptAdmin() external {
855:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
856:        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only'); // @audit: 1st sload
857:
858:        // Save current values for inclusion in log
859:        address oldAdmin = admin;
860:        address oldPendingAdmin = pendingAdmin; // @audit: 2nd sload
861:
862:        // Store admin with value pendingAdmin
863:        admin = pendingAdmin; // @audit: 3rd sload
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..f8e47d90 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -853,14 +853,15 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
      */
     function _acceptAdmin() external {
         // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
-        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');
+        address _pendingAdmin = pendingAdmin;
+        require(msg.sender == _pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

         // Save current values for inclusion in log
         address oldAdmin = admin;
-        address oldPendingAdmin = pendingAdmin;
+        address oldPendingAdmin = _pendingAdmin;

         // Store admin with value pendingAdmin
-        admin = pendingAdmin;
+        admin = _pendingAdmin;

         // Clear the pending value
         pendingAdmin = address(0);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L886-L898

### Cache `pendingVetoer` to save 3 SLOADs
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
886:    function _acceptVetoer() external {
887:        if (msg.sender != pendingVetoer) { // @audit: 1st sload
888:            revert PendingVetoerOnly();
889:        }
890:
891:        // Update vetoer
892:        emit NewVetoer(vetoer, pendingVetoer); // @audit: 2nd sload
893:        vetoer = pendingVetoer; // @audit: 3rd sload
894:
895:        // Clear the pending value
896:        emit NewPendingVetoer(pendingVetoer, address(0)); // @audit: 4th sload
897:        pendingVetoer = address(0);
898:    }
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..2d69a3d4 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -884,16 +884,17 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
     }

     function _acceptVetoer() external {
-        if (msg.sender != pendingVetoer) {
+        address _pendingVetoer = pendingVetoer;
+        if (msg.sender != _pendingVetoer) {
             revert PendingVetoerOnly();
         }

         // Update vetoer
-        emit NewVetoer(vetoer, pendingVetoer);
-        vetoer = pendingVetoer;
+        emit NewVetoer(vetoer, _pendingVetoer);
+        vetoer = _pendingVetoer;

         // Clear the pending value
-        emit NewPendingVetoer(pendingVetoer, address(0));
+        emit NewPendingVetoer(_pendingVetoer, address(0));
         pendingVetoer = address(0);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L210-L257

### Cache new value assigned to `proposal.objectionPeriodEndBlock` to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOV3Votes.sol
210:    function castVoteDuringVotingPeriodInternal(
211:        NounsDAOStorageV3.StorageV3 storage ds,
212:        uint256 proposalId,
213:        address voter,
214:        uint8 support
215:    ) internal returns (uint96) {
...
240:        if (
241:            // only for votes can trigger an objection period
242:            // we're in the last minute window
243:            isForVoteInLastMinuteWindow &&
244:            // first part of the vote flip check
245:            // separated from the second part to optimize gas
246:            isDefeatedBefore &&
247:            // haven't turn on objection yet
248:            proposal.objectionPeriodEndBlock == 0 && // @audit: 1st sload
249:            // second part of the vote flip check
250:            !ds.isDefeated(proposal)
251:        ) {
252:            proposal.objectionPeriodEndBlock = SafeCast.toUint64( // @audit: cache new value assigned to `proposal.objectionPeriodEndBlock`
253:                proposal.endBlock + ds.objectionPeriodDurationInBlocks
254:            );
255:
256:            emit ProposalObjectionPeriodSet(proposal.id, proposal.objectionPeriodEndBlock); // @audit: 2nd sload, can use cached new value (after refactoring)
257:        }
```
```diff
diff --git a/contracts/governance/NounsDAOV3Votes.sol b/contracts/governance/NounsDAOV3Votes.sol
index 3743132b..c3022e5d 100644
--- a/contracts/governance/NounsDAOV3Votes.sol
+++ b/contracts/governance/NounsDAOV3Votes.sol
@@ -249,11 +249,13 @@ library NounsDAOV3Votes {
             // second part of the vote flip check
             !ds.isDefeated(proposal)
         ) {
-            proposal.objectionPeriodEndBlock = SafeCast.toUint64(
+            uint64 newObjectionPeriodEndBlock = SafeCast.toUint64(
                 proposal.endBlock + ds.objectionPeriodDurationInBlocks
             );
+
+            proposal.objectionPeriodEndBlock = newObjectionPeriodEndBlock;

-            emit ProposalObjectionPeriodSet(proposal.id, proposal.objectionPeriodEndBlock);
+            emit ProposalObjectionPeriodSet(proposal.id, newObjectionPeriodEndBlock);
         }

         receipt.hasVoted = true;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L276-L293

### Cache `ds.pendingAdmin` and emit `oldPendingAdmin` to save 3 SLOADs
```solidity
File: contracts/governance/NounsDAOV3Admin.sol
276:    function _acceptAdmin(NounsDAOStorageV3.StorageV3 storage ds) external {
277:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
278:        require(
279:            msg.sender == ds.pendingAdmin && msg.sender != address(0), // @audit: 1st sload
280:            'NounsDAO::_acceptAdmin: pending admin only'
281:        );
282:
283:        // Save current values for inclusion in log
284:        address oldAdmin = ds.admin;
285:        address oldPendingAdmin = ds.pendingAdmin; // @audit: 2nd sload
286:
287:        // Store admin with value pendingAdmin
288:        ds.admin = ds.pendingAdmin; // @audit 3rd sload | `ds.admin == ds.pendingAdmin == oldPendingAdmin`
289:
290:        // Clear the pending value
291:        ds.pendingAdmin = address(0);
292:
293:        emit NewAdmin(oldAdmin, ds.admin); // @audit: unnecessary sload, emit `oldPendingAdmin`
```
```diff
diff --git a/contracts/governance/NounsDAOV3Admin.sol b/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..cf9c3468 100644
--- a/contracts/governance/NounsDAOV3Admin.sol
+++ b/contracts/governance/NounsDAOV3Admin.sol
@@ -275,22 +275,23 @@ library NounsDAOV3Admin {
      */
     function _acceptAdmin(NounsDAOStorageV3.StorageV3 storage ds) external {
         // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
+        address _pendingAdmin = ds.pendingAdmin;
         require(
-            msg.sender == ds.pendingAdmin && msg.sender != address(0),
+            msg.sender == _pendingAdmin && msg.sender != address(0),
             'NounsDAO::_acceptAdmin: pending admin only'
         );

         // Save current values for inclusion in log
         address oldAdmin = ds.admin;
-        address oldPendingAdmin = ds.pendingAdmin;
+        address oldPendingAdmin = _pendingAdmin;

         // Store admin with value pendingAdmin
-        ds.admin = ds.pendingAdmin;
+        ds.admin = _pendingAdmin;

         // Clear the pending value
         ds.pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, ds.admin);
+        emit NewAdmin(oldAdmin, oldPendingAdmin);
         emit NewPendingAdmin(oldPendingAdmin, address(0));
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L314-L326

### Cache `ds.pendingVetoer` to save 3 SLOADs
```solidity
File: contracts/governance/NounsDAOV3Admin.sol
314:    function _acceptVetoer(NounsDAOStorageV3.StorageV3 storage ds) external {
315:        if (msg.sender != ds.pendingVetoer) { // @audit: 1st sload
316:            revert PendingVetoerOnly();
317:        }
318:
319:        // Update vetoer
320:        emit NewVetoer(ds.vetoer, ds.pendingVetoer); // @audit: 2nd sload
321:        ds.vetoer = ds.pendingVetoer; // @audit: 3rd sload
322:
323:        // Clear the pending value
324:        emit NewPendingVetoer(ds.pendingVetoer, address(0)); // @audit: 4th sload
325:        ds.pendingVetoer = address(0);
326:    }
```
```diff
diff --git a/contracts/governance/NounsDAOV3Admin.sol b/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..8986b598 100644
--- a/contracts/governance/NounsDAOV3Admin.sol
+++ b/contracts/governance/NounsDAOV3Admin.sol
@@ -312,16 +312,17 @@ library NounsDAOV3Admin {
      * @notice Called by the pendingVetoer to accept role and update vetoer
      */
     function _acceptVetoer(NounsDAOStorageV3.StorageV3 storage ds) external {
-        if (msg.sender != ds.pendingVetoer) {
+        address _pendingVetoer = ds.pendingVetoer;
+        if (msg.sender != _pendingVetoer) {
             revert PendingVetoerOnly();
         }

         // Update vetoer
-        emit NewVetoer(ds.vetoer, ds.pendingVetoer);
-        ds.vetoer = ds.pendingVetoer;
+        emit NewVetoer(ds.vetoer, _pendingVetoer);
+        ds.vetoer = _pendingVetoer;

         // Clear the pending value
-        emit NewPendingVetoer(ds.pendingVetoer, address(0));
+        emit NewPendingVetoer(_pendingVetoer, address(0));
         ds.pendingVetoer = address(0);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L332-L338

### Cache `ds.vetoer` to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOV3Admin.sol
332:    function _burnVetoPower(NounsDAOStorageV3.StorageV3 storage ds) public {
333:        // Check caller is vetoer
334:        require(msg.sender == ds.vetoer, 'NounsDAO::_burnVetoPower: vetoer only'); // @audit: 1st sload
335:
336:        // Update vetoer to 0x0
337:        emit NewVetoer(ds.vetoer, address(0)); // @audit: 2nd sload
338:        ds.vetoer = address(0);
```
```diff
diff --git a/contracts/governance/NounsDAOV3Admin.sol b/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..52c5e55a 100644
--- a/contracts/governance/NounsDAOV3Admin.sol
+++ b/contracts/governance/NounsDAOV3Admin.sol
@@ -331,10 +331,11 @@ library NounsDAOV3Admin {
      */
     function _burnVetoPower(NounsDAOStorageV3.StorageV3 storage ds) public {
         // Check caller is vetoer
-        require(msg.sender == ds.vetoer, 'NounsDAO::_burnVetoPower: vetoer only');
+        address _vetoer = ds.vetoer;
+        require(msg.sender == _vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

         // Update vetoer to 0x0
-        emit NewVetoer(ds.vetoer, address(0));
+        emit NewVetoer(_vetoer, address(0));
         ds.vetoer = address(0);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L224-L226

### Cache `ds.nouns` to save 1 SLOAD
```solidity
File: contracts/governance/fork/NounsDAOV3Fork.sol
224:    function adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds) internal view returns (uint256) {
225:        return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - ds.forkEscrow.numTokensOwnedByDAO(); // @audit: 2 sloads for `ds.nouns`
226:    }
```
```diff
diff --git a/contracts/governance/fork/NounsDAOV3Fork.sol b/contracts/governance/fork/NounsDAOV3Fork.sol
index d87ffc70..dd1d1c29 100644
--- a/contracts/governance/fork/NounsDAOV3Fork.sol
+++ b/contracts/governance/fork/NounsDAOV3Fork.sol
@@ -222,7 +222,8 @@ library NounsDAOV3Fork {
      * This is used when calculating proposal threshold, quorum, fork threshold & treasury split.
      */
     function adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds) internal view returns (uint256) {
-        return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - ds.forkEscrow.numTokensOwnedByDAO();
+        NounsTokenLike nouns = ds.nouns;
+        return nouns.totalSupply() - nouns.balanceOf(address(ds.timelock)) - ds.forkEscrow.numTokensOwnedByDAO();
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L495-L518

### Cache `proposal.id` to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOV3Proposals.sol
495:    function executeInternal(
496:        NounsDAOStorageV3.StorageV3 storage ds,
497:        NounsDAOStorageV3.Proposal storage proposal,
498:        INounsDAOExecutor timelock
499:    ) internal {
500:        require(
501:            stateInternal(ds, proposal.id) == NounsDAOStorageV3.ProposalState.Queued, // @audit: 1st sload
502:            'NounsDAO::execute: proposal can only be executed if it is queued'
503:        );
...
517:        emit ProposalExecuted(proposal.id); // @audit: 2nd sload
518:    }
```
```diff
diff --git a/contracts/governance/NounsDAOV3Proposals.sol b/contracts/governance/NounsDAOV3Proposals.sol
index 2685dc20..142fb5e8 100644
--- a/contracts/governance/NounsDAOV3Proposals.sol
+++ b/contracts/governance/NounsDAOV3Proposals.sol
@@ -497,8 +497,9 @@ library NounsDAOV3Proposals {
         NounsDAOStorageV3.Proposal storage proposal,
         INounsDAOExecutor timelock
     ) internal {
+        uint256 _id = proposal.id;
         require(
-            stateInternal(ds, proposal.id) == NounsDAOStorageV3.ProposalState.Queued,
+            stateInternal(ds, _id) == NounsDAOStorageV3.ProposalState.Queued,
             'NounsDAO::execute: proposal can only be executed if it is queued'
         );
         if (ds.isForkPeriodActive()) revert CannotExecuteDuringForkingPeriod();
@@ -514,7 +515,7 @@ library NounsDAOV3Proposals {
                 proposal.eta
             );
         }
-        emit ProposalExecuted(proposal.id);
+        emit ProposalExecuted(_id);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L536-L543

### Cache `ds.vetoer` to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOV3Proposals.sol
536:    function veto(NounsDAOStorageV3.StorageV3 storage ds, uint256 proposalId) external {
537:        if (ds.vetoer == address(0)) { // @audit: 1st sload
538:            revert VetoerBurned();
539:        }
540:
541:        if (msg.sender != ds.vetoer) { // @audit: 2nd sload
542:            revert VetoerOnly();
543:        }
```
```diff
diff --git a/contracts/governance/NounsDAOV3Proposals.sol b/contracts/governance/NounsDAOV3Proposals.sol
index 2685dc20..ebdd2744 100644
--- a/contracts/governance/NounsDAOV3Proposals.sol
+++ b/contracts/governance/NounsDAOV3Proposals.sol
@@ -534,11 +534,12 @@ library NounsDAOV3Proposals {
      * @param proposalId The id of the proposal to veto
      */
     function veto(NounsDAOStorageV3.StorageV3 storage ds, uint256 proposalId) external {
-        if (ds.vetoer == address(0)) {
+        address _vetoer = ds.vetoer;
+        if (_vetoer == address(0)) {
             revert VetoerBurned();
         }

-        if (msg.sender != ds.vetoer) {
+        if (msg.sender != _vetoer) {
             revert VetoerOnly();
         }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L133-L139

### Use already cached `closedForkId` instead of re-reading from storage to save 1 SLOAD
```solidity
File: contracts/governance/fork/NounsDAOForkEscrow.sol
133:    function closeEscrow() external onlyDAO returns (uint32 closedForkId) {
134:        numTokensInEscrow = 0;
135:
136:        closedForkId = forkId; // @audit: 1st sload
137:
138:        forkId++; // @audit: 2nd sload
139:    }
```
```diff
diff --git a/contracts/governance/fork/NounsDAOForkEscrow.sol b/contracts/governance/fork/NounsDAOForkEscrow.sol
index 53c54c9e..0ea08c8f 100644
--- a/contracts/governance/fork/NounsDAOForkEscrow.sol
+++ b/contracts/governance/fork/NounsDAOForkEscrow.sol
@@ -135,7 +135,7 @@ contract NounsDAOForkEscrow is IERC721Receiver {

         closedForkId = forkId;

-        forkId++;
+        forkId = closedForkId + 1;
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L261-L265

### Cache `weth` to save 1 SLOAD
```solidity
File: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
261:    function _safeTransferETHWithFallback(address to, uint256 amount) internal {
262:        if (!_safeTransferETH(to, amount)) {
263:            IWETH(weth).deposit{ value: amount }(); // @audit: 1st sload
264:            IERC20(weth).transfer(to, amount); // @audit; 2nd sload
265:        }
```
```diff
diff --git a/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol b/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
index 0bd594b6..edfa8e25 100644
--- a/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
+++ b/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
@@ -260,8 +260,9 @@ contract NounsAuctionHouseFork is
      */
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

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L376-L384

### Cache `nouns` to save 1 SLOAD for the following functions: `NounsDAOLogicV1Fork.propose` and `NounsDAOLogicV1Fork.quit (both variations)`
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
376:    function checkGovernanceActive() internal view {
377:        if (block.timestamp < nouns.forkingPeriodEndTimestamp()) { // @audit: 1st sload
378:            revert GovernanceBlockedDuringForkingPeriod();
379:        }
380:
381:        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) { // @audit: 2nd sload
382:            revert WaitingForTokensToClaimOrExpiration();
383:        }
384:    }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..171da42b 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -374,11 +374,12 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
      * 2. The delayed governance expiration timestamp is reached
      */
     function checkGovernanceActive() internal view {
-        if (block.timestamp < nouns.forkingPeriodEndTimestamp()) {
+        INounsTokenForkLike _nouns = nouns;
+        if (block.timestamp < _nouns.forkingPeriodEndTimestamp()) {
             revert GovernanceBlockedDuringForkingPeriod();
         }

-        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
+        if (block.timestamp < delayedGovernanceExpirationTimestamp && _nouns.remainingTokensToClaim() > 0) {
             revert WaitingForTokensToClaimOrExpiration();
         }
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L777-L779

### Cache `nouns` to save 2 SLOADs for the following functions: `NounsDAOLogicV1Fork.propose` and `NounsDAOLogicV1Fork.quit (both variations)`
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
777:    function adjustedTotalSupply() public view returns (uint256) {
778:        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim(); // @audit: 3 sloads for `nouns`
779:    }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..3553aab0 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -775,7 +775,8 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
     }

     function adjustedTotalSupply() public view returns (uint256) {
-        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim();
+        INounsTokenForkLike _nouns = nouns;
+        return _nouns.totalSupply() - _nouns.balanceOf(address(timelock)) + _nouns.remainingTokensToClaim();
     }

     function erc20TokensToIncludeInQuitArray() public view returns(address[] memory) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L410-L422

### Cache `timelock` to save 1 SLOAD for the following function: `NounsDAOLogicV1Fork.queue`
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
410:    function queueOrRevertInternal( // @audit: invoked by `NounsDAOLogicV1Fork.queue`s
411:        address target,
412:        uint256 value,
413:        string memory signature,
414:        bytes memory data,
415:        uint256 eta
416:    ) internal {
417:        require(
418:            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))), // @audit: 1st sload
419:            'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
420:        );
421:        timelock.queueTransaction(target, value, signature, data, eta); // @audit: 2nd sload
422:    }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..7b105ce6 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -414,11 +414,12 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         bytes memory data,
         uint256 eta
     ) internal {
+        NounsDAOExecutorV2 _timelock = timelock;
         require(
-            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
+            !_timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
             'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
         );
-        timelock.queueTransaction(target, value, signature, data, eta);
+        _timelock.queueTransaction(target, value, signature, data, eta);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L731-L747

### Cache `pendingAdmin`, use new value for `admin`, and use the new value for `pendingAdmin` to save 4 SLOADs for the following function: `NounsDAOLogicV1Fork._acceptAdmin`
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
731:    function _acceptAdmin() external {
732:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
733:        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only'); // @audit: 1st sload
734:
735:        // Save current values for inclusion in log
736:        address oldAdmin = admin; // @audit: 1st sload
737:        address oldPendingAdmin = pendingAdmin; // @audit: 2nd sload
738:
739:        // Store admin with value pendingAdmin
740:        admin = pendingAdmin; // @audit 3rd sload
741:
742:        // Clear the pending value
743:        pendingAdmin = address(0);
744:
745:        emit NewAdmin(oldAdmin, admin); // @audit: 2nd sload, use `oldPendingAdmin`
746:        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin); // @audit: 4th sload, use `address(0)`
747:    }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..a65e9c42 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -730,20 +730,21 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
      */
     function _acceptAdmin() external {
         // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
-        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');
+        address _pendingAdmin = pendingAdmin;
+        require(msg.sender == _pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

         // Save current values for inclusion in log
         address oldAdmin = admin;
-        address oldPendingAdmin = pendingAdmin;
+        address oldPendingAdmin = _pendingAdmin;

         // Store admin with value pendingAdmin
-        admin = pendingAdmin;
+        admin = _pendingAdmin;

         // Clear the pending value
         pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, admin);
-        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
+        emit NewAdmin(oldAdmin, oldPendingAdmin);
+        emit NewPendingAdmin(oldPendingAdmin, address(0));
     }
```

## Read `memory` values instead of reading from storage
Reading from storage is expensive as it uses the SLOAD opcode (`Gcoldsload (2100 gas)` for a cold address and `Gwarmaccess (100 gas)` for a warm access). If we set new values for storage variables and need to re-use those storage variables multiple times afterwards we should use the new value assigned instead of re-reading the state variable.

Total Instances: `12`

Estimated Gas Saved: `1200`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L320-L365

### Instead of reading from storage, use the memory values (`msg.sender`, `temp.proposalThreshold`, `temp.startBlock`, and `temp.endBlock`) to save 6 SLOADs
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
320:        newProposal.proposer = msg.sender; // @audit: `newProposal.proposer == msg.sender`
321:        newProposal.proposalThreshold = temp.proposalThreshold; // @audit: `newProposal.proposalThreshold == temp.proposalThreshold`
322:        newProposal.quorumVotes = bps2Uint(quorumVotesBPS, temp.totalSupply);
323:        newProposal.eta = 0;
324:        newProposal.targets = targets;
325:        newProposal.values = values;
326:        newProposal.signatures = signatures;
327:        newProposal.calldatas = calldatas;
328:        newProposal.startBlock = temp.startBlock; // @audit: `newProposal.startBlock == temp.startBlock`
329:        newProposal.endBlock = temp.endBlock; // @audit: `newProposal.endBlock == temp.endBlock`
...
337:        latestProposalIds[newProposal.proposer] = newProposal.id; // @audit: use `msg.sender`
338:
339:        /// @notice Maintains backwards compatibility with GovernorBravo events
340:        emit ProposalCreated(
341:            newProposal.id,
342:            msg.sender,
343:            targets,
344:            values,
345:            signatures,
346:            calldatas,
347:            newProposal.startBlock, // @audit: use `temp.startBlock`
348:            newProposal.endBlock, // @audit: use `temp.endBlock`
349:            description
350:        );
351:
352:        /// @notice Updated event with `proposalThreshold` and `quorumVotes`
353:        emit ProposalCreatedWithRequirements(
354:            newProposal.id,
355:            msg.sender,
356:            targets,
357:            values,
358:            signatures,
359:            calldatas,
360:            newProposal.startBlock, // @audit: use `temp.startBlock`
361:            newProposal.endBlock, // @audit: use `temp.endBlock`
362:            newProposal.proposalThreshold, // @audit: `temp.proposalThreshold`
363:            newProposal.quorumVotes,
364:            description
365:        );
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..bbeca336 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -334,7 +334,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         newProposal.executed = false;
         newProposal.creationBlock = block.number;

-        latestProposalIds[newProposal.proposer] = newProposal.id;
+        latestProposalIds[msg.sender] = newProposal.id;

         /// @notice Maintains backwards compatibility with GovernorBravo events
         emit ProposalCreated(
@@ -344,8 +344,8 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
             values,
             signatures,
             calldatas,
-            newProposal.startBlock,
-            newProposal.endBlock,
+            temp.startBlock,
+            temp.endBlock,
             description
         );

@@ -357,9 +357,9 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
             values,
             signatures,
             calldatas,
-            newProposal.startBlock,
-            newProposal.endBlock,
-            newProposal.proposalThreshold,
+            temp.startBlock,
+            temp.endBlock,
+            temp.proposalThreshold,
             newProposal.quorumVotes,
             description
         );
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L242-L286

### Instead of reading from storage, use the memory values (`msg.sender`, `temp.proposalThreshold`, `temp.startBlock`, and `temp.endBlock`) to save 6 SLOADs
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
242:        newProposal.proposer = msg.sender; // @audit: `newProposal.proposer == msg.sender`
243:        newProposal.proposalThreshold = temp.proposalThreshold; // @audit: `newProposal.proposalThreshold == temp.proposalThreshold`
244:        newProposal.eta = 0;
245:        newProposal.targets = targets;
246:        newProposal.values = values;
247:        newProposal.signatures = signatures;
248:        newProposal.calldatas = calldatas;
249:        newProposal.startBlock = temp.startBlock; // @audit: `newProposal.startBlock == temp.startBlock`
250:        newProposal.endBlock = temp.endBlock; // @audit: `newProposal.endBlock == temp.endBlock`
...
260:        latestProposalIds[newProposal.proposer] = newProposal.id; // @audit: use `msg.sender`
261:
262:        /// @notice Maintains backwards compatibility with GovernorBravo events
263:        emit ProposalCreated(
264:            newProposal.id,
265:            msg.sender,
266:            targets,
267:            values,
268:            signatures,
269:            calldatas,
270:            newProposal.startBlock, // @audit: use `temp.startBlock`
271:            newProposal.endBlock, // @audit: use `temp.endBlock`
272:            description
273:        );
274:
275:        /// @notice Updated event with `proposalThreshold` and `minQuorumVotes`
276:        /// @notice `minQuorumVotes` is always zero since V2 introduces dynamic quorum with checkpoints
277:        emit ProposalCreatedWithRequirements(
278:            newProposal.id,
279:            msg.sender,
280:            targets,
281:            values,
282:            signatures,
283:            calldatas,
284:            newProposal.startBlock, // @audit: use `temp.startBlock`
285:            newProposal.endBlock, // @audit: use `temp.endBlock`
286:            newProposal.proposalThreshold, // @audit: use `temp.proposalThreshold`
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..71680e6a 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -257,7 +257,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         newProposal.totalSupply = temp.totalSupply;
         newProposal.creationBlock = block.number;

-        latestProposalIds[newProposal.proposer] = newProposal.id;
+        latestProposalIds[msg.sender] = newProposal.id;

         /// @notice Maintains backwards compatibility with GovernorBravo events
         emit ProposalCreated(
@@ -267,8 +267,8 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             values,
             signatures,
             calldatas,
-            newProposal.startBlock,
-            newProposal.endBlock,
+            temp.startBlock,
+            temp.endBlock,
             description
         );

@@ -281,9 +281,9 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             values,
             signatures,
             calldatas,
-            newProposal.startBlock,
-            newProposal.endBlock,
-            newProposal.proposalThreshold,
+            temp.startBlock,
+            temp.endBlock,
+            temp.proposalThreshold,
             minQuorumVotes(),
             description
         );
```

## State variables can be packed into fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.

Total Instances: `7`

Estimated Gas Saved: `16 (slots) * 2000 = 32000`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L653-L717

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L112-L142

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L162-L206

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L162-L206

### Reduce `uint` types for `votingDelay`, `votingPeriod`, `proposalThresholdBPS`, `forkEndTimestamp`, and `forkPeriod` and pack these values into one storage slot to save 4 SLOTs (~8000 gas)

**Note: 1. `NounsDAOV3Admin` is a library used to set the state variables for `NounsDAOStorageV3`. The `NounsDAOLogicV3` contract inherits from `NounsDAOStorageV3`. 2. `votingDelay`, `votingPeriod`, `proposalThresholdBPS`, and `forkPeriod` all have enforced upper bounds and therefore their `uint` type can be reduced to a type which can hold that upper bound. `forkEndTimestamp` is a timestamp and therefore its uint type can be reduced to `>= uint(32)`.*
```solidity
File: contracts/governance/NounsDAOInterfaces.sol
653:    struct StorageV3 {
654:        // ================ PROXY ================ //
655:        /// @notice Administrator for this contract
656:        address admin;
657:        /// @notice Pending administrator for this contract
658:        address pendingAdmin;
659:        /// @notice Active brains of Governor
660:        address implementation;
661:        // ================ V1 ================ //
662:        /// @notice Vetoer who has the ability to veto any proposal
663:        address vetoer;
664:        /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
665:        uint256 votingDelay; // @audit: has max upper bound, can be `uint type >= uint(24)`
666:        /// @notice The duration of voting on a proposal, in blocks
667:        uint256 votingPeriod; // @audit: has max upper bound, can be `uint type >= uint(24)`
668:        /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
669:        uint256 proposalThresholdBPS; // @audit: has max upper bound, can be `uint type >= uint(16)`
...
705:        /// @notice Timestamp at which the last fork period ends
706:        uint256 forkEndTimestamp; // @audit: timestamp, can be `uint type >= uint(32)`
707:        /// @notice Fork period in seconds
708:        uint256 forkPeriod; // @audit: has a max upper bound, can be `uint type >= uint(24)`
709:        /// @notice Threshold defined in basis points (10,000 = 100%) required for forking
710:        uint256 forkThresholdBPS;
711:        /// @notice Address of the original timelock
712:        INounsDAOExecutor timelockV1;
713:        /// @notice The proposal at which to start using `startBlock` instead of `creationBlock` for vote snapshots
714:        /// @dev Make sure this stays the last variable in this struct, so we can delete it in the next version
715:        /// @dev To be zeroed-out and removed in a V3.1 fix version once the switch takes place
716:        uint256 voteSnapshotBlockSwitchProposalId;
717:    }
```

Below are the bounds for the above storage variables:

```solidity
File: contracts/governance/NounsDAOV3Admin.sol
112:    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%
113:
114:    /// @notice The maximum setable proposal threshold
115:    uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10% // @audit: `uint16` is sufficient
116:
117:    /// @notice The minimum setable voting period in blocks
118:    uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12; // @audit: 7200 seconds
119:
120:    /// @notice The max setable voting period in blocks
121:    uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12; // @audit: 100800 seconds, `uint24` is sufficient
122:
123:    /// @notice The min setable voting delay in blocks
124:    uint256 public constant MIN_VOTING_DELAY_BLOCKS = 1;
125:
126:    /// @notice The max setable voting delay in blocks
127:    uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12; // @audit: 100800 seconds, `uint24` is sufficient
...
138:    /// @notice Upper bound for forking period. If forking period is too high it can block proposals for too long.
139:    uint256 public constant MAX_FORK_PERIOD = 14 days; // @audit: 1209600 seconds, `uint24` is sufficient
140:
141:    /// @notice Lower bound for forking period
142:    uint256 public constant MIN_FORK_PERIOD = 2 days; // @audit: 172800 seconds
```

Below are the functions that enforce the above bounds:

```solidity
File: contracts/governance/NounsDAOV3Admin.sol
162:    function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external onlyAdmin(ds) {
163:        require(
164:            newVotingDelay >= MIN_VOTING_DELAY_BLOCKS && newVotingDelay <= MAX_VOTING_DELAY_BLOCKS,
165:            'NounsDAO::_setVotingDelay: invalid voting delay'
166:        );
167:        uint256 oldVotingDelay = ds.votingDelay;
168:        ds.votingDelay = newVotingDelay;
169:
170:        emit VotingDelaySet(oldVotingDelay, newVotingDelay);
171:    }
172:
173:    /**
174:     * @notice Admin function for setting the voting period
175:     * @param newVotingPeriod new voting period, in blocks
176:     */
177:    function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external onlyAdmin(ds) {
178:        require(
179:            newVotingPeriod >= MIN_VOTING_PERIOD_BLOCKS && newVotingPeriod <= MAX_VOTING_PERIOD_BLOCKS,
180:            'NounsDAO::_setVotingPeriod: invalid voting period'
181:        );
182:        uint256 oldVotingPeriod = ds.votingPeriod;
183:        ds.votingPeriod = newVotingPeriod;
184:
185:        emit VotingPeriodSet(oldVotingPeriod, newVotingPeriod);
186:    }
187:
188:    /**
189:     * @notice Admin function for setting the proposal threshold basis points
190:     * @dev newProposalThresholdBPS must be in [`MIN_PROPOSAL_THRESHOLD_BPS`,`MAX_PROPOSAL_THRESHOLD_BPS`]
191:     * @param newProposalThresholdBPS new proposal threshold
192:     */
193:    function _setProposalThresholdBPS(NounsDAOStorageV3.StorageV3 storage ds, uint256 newProposalThresholdBPS)
194:        external
195:        onlyAdmin(ds)
196:    {
197:        require(
198:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
199:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
200:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
201:        );
202:        uint256 oldProposalThresholdBPS = ds.proposalThresholdBPS;
203:        ds.proposalThresholdBPS = newProposalThresholdBPS;
204:
205:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, newProposalThresholdBPS);
206:    }

533:    function _setForkPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newForkPeriod) external onlyAdmin(ds) {
534:        if (newForkPeriod > MAX_FORK_PERIOD) {
535:            revert ForkPeriodTooLong();
536:        }
537:
538:        if (newForkPeriod < MIN_FORK_PERIOD) {
539:            revert ForkPeriodTooShort();
540:        }
541:
542:        emit ForkPeriodSet(ds.forkPeriod, newForkPeriod);
543:
544:        ds.forkPeriod = newForkPeriod;
545:    }
```

```diff
diff --git a/contracts/governance/NounsDAOInterfaces.sol b/contracts/governance/NounsDAOInterfaces.sol
index 8fb0b4d3..838fb3eb 100644
--- a/contracts/governance/NounsDAOInterfaces.sol
+++ b/contracts/governance/NounsDAOInterfaces.sol
@@ -661,12 +661,6 @@ contract NounsDAOStorageV3 {
         // ================ V1 ================ //
         /// @notice Vetoer who has the ability to veto any proposal
         address vetoer;
-        /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
-        uint256 votingDelay;
-        /// @notice The duration of voting on a proposal, in blocks
-        uint256 votingPeriod;
-        /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
-        uint256 proposalThresholdBPS;
         /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
         uint256 quorumVotesBPS;
         /// @notice The total number of proposals
@@ -702,12 +696,18 @@ contract NounsDAOStorageV3 {
         address forkDAOTreasury;
         /// @notice The token contract of the last deployed fork
         address forkDAOToken;
-        /// @notice Timestamp at which the last fork period ends
-        uint256 forkEndTimestamp;
-        /// @notice Fork period in seconds
-        uint256 forkPeriod;
         /// @notice Threshold defined in basis points (10,000 = 100%) required for forking
         uint256 forkThresholdBPS;
+        /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
+        uint24 votingDelay;
+        /// @notice The duration of voting on a proposal, in blocks
+        uint24 votingPeriod;
+        /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
+        uint16 proposalThresholdBPS;
+        /// @notice Timestamp at which the last fork period ends
+        uint32 forkEndTimestamp;
+        /// @notice Fork period in seconds
+        uint24 forkPeriod;
         /// @notice Address of the original timelock
         INounsDAOExecutor timelockV1;
         /// @notice The proposal at which to start using `startBlock` instead of `creationBlock` for vote snapshots
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol#L27-L54

### Reduce the `uint` types for `votingDelay`, `votingPeriod`, `proposalThresholdBPS`, and `delayedGovernanceExpirationTimestamp` and pack these values with `nouns` to save 4 SLOTs (~8000 gas)

**Note: 1. `NounsDAOLogicV1Fork` will benefit from this optimization as it inherits from `NounsDAOStorageV1Fork`. 2. `votingDelay`, `votingPeriod`, and `proposalThresholdBPS` all have enforced upper bounds similar to the **instance above** and therefore their `uint` type can be reduced to a type which can hold that upper bound. `delayedGovernanceExpirationTimestamp` is a timestamp and therefore its uint type can be reduced to `>= uint(32)`.*
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol
27:    /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
28:    uint256 public votingDelay; // @audit: has a max upper bounds, can be `uint24`
29:
30:    /// @notice The duration of voting on a proposal, in blocks
31:    uint256 public votingPeriod; // @audit: has a max upper bounds, can be `uint24`
32:
33:    /// @notice The basis point number of votes to exceed in order for a voter to become a proposer. *DIFFERS from GovernerBravo
34:    uint256 public proposalThresholdBPS; // @audit: has a max upper bounds, can be `uint16`
35:
36:    /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
37:    uint256 public quorumVotesBPS;
38:
39:    /// @notice The total number of proposals
40:    uint256 public proposalCount;
41:
42:    /// @notice The address of the Nouns DAO Executor NounsDAOExecutor
43:    NounsDAOExecutorV2 public timelock;
44:
45:    /// @notice The address of the Nouns tokens
46:    INounsTokenForkLike public nouns;
47:
48:    /// @notice The official record of all proposals ever proposed
49:    mapping(uint256 => Proposal) public _proposals;
50:
51:    /// @notice The latest proposal for each proposer
52:    mapping(address => uint256) public latestProposalIds;
53:
54:    uint256 public delayedGovernanceExpirationTimestamp; // @audit: timestamp, can be `>= uint(32)`
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol
index 8d18f510..fbe5d1ab 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOStorageV1Fork.sol
@@ -24,14 +24,6 @@ contract NounsDAOStorageV1Fork {
     /// @notice Pending administrator for this contract
     address public pendingAdmin;

-    /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
-    uint256 public votingDelay;
-
-    /// @notice The duration of voting on a proposal, in blocks
-    uint256 public votingPeriod;
-
-    /// @notice The basis point number of votes to exceed in order for a voter to become a proposer. *DIFFERS from GovernerBravo
-    uint256 public proposalThresholdBPS;

     /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
     uint256 public quorumVotesBPS;
@@ -45,14 +37,23 @@ contract NounsDAOStorageV1Fork {
     /// @notice The address of the Nouns tokens
     INounsTokenForkLike public nouns;

+    /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
+    uint24 public votingDelay;
+
+    /// @notice The duration of voting on a proposal, in blocks
+    uint24 public votingPeriod;
+
+    /// @notice The basis point number of votes to exceed in order for a voter to become a proposer. *DIFFERS from GovernerBravo
+    uint16 public proposalThresholdBPS;
+
+    uint32 public delayedGovernanceExpirationTimestamp;
+
     /// @notice The official record of all proposals ever proposed
     mapping(uint256 => Proposal) public _proposals;

     /// @notice The latest proposal for each proposer
     mapping(address => uint256) public latestProposalIds;

-    uint256 public delayedGovernanceExpirationTimestamp;
-
     address[] public erc20TokensToIncludeInQuit;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L380-L391

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L62-L77

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L644-L694

### Reduce `uint` types for `votingDelay`, `votingPeriod`, and `proposalThresholdBPS`, and pack these values with `vetoer` into one storage slot to save 3 SLOTs (~6000 gas)

**Note: 1. `NounsDAOLogicV2` inherits from `NounsDAOStorageV2`, which inherits from `NounsDAOStorageV1Adjusted`. 2. `votingDelay`, `votingPeriod`, and `proposalThresholdBPS` all have enforced upper bounds and therefore their `uint` type can be reduced to a type which can hold that upper bound.*
```solidity
File: contracts/governance/NounsDAOInterfaces.sol
380: contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {
381:    /// @notice Vetoer who has the ability to veto any proposal
382:    address public vetoer;
383:
384:    /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
385:    uint256 public votingDelay;
386:
387:    /// @notice The duration of voting on a proposal, in blocks
388:    uint256 public votingPeriod;
389:
390:    /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
391:    uint256 public proposalThresholdBPS;
```

Below are the bounds for the above storage variables:

```solidity
File: contracts/governance/NounsDAOLogicV2.sol
62:    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%
63:
64:    /// @notice The maximum setable proposal threshold
65:    uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%
66:
67:    /// @notice The minimum setable voting period
68:    uint256 public constant MIN_VOTING_PERIOD = 5_760; // About 24 hours
69:
70:    /// @notice The max setable voting period
71:    uint256 public constant MAX_VOTING_PERIOD = 80_640; // About 2 weeks
72:
73:    /// @notice The min setable voting delay
74:    uint256 public constant MIN_VOTING_DELAY = 1;
75:
76:    /// @notice The max setable voting delay
77:    uint256 public constant MAX_VOTING_DELAY = 40_320; // About 1 week
```

Below are the functions that enforce the above bounds:

```solidity
File: contracts/governance/NounsDAOLogicV2.sol
644:    function _setVotingDelay(uint256 newVotingDelay) external {
645:        if (msg.sender != admin) {
646:            revert AdminOnly();
647:        }
648:        require(
649:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
650:            'NounsDAO::_setVotingDelay: invalid voting delay'
651:        );
652:        uint256 oldVotingDelay = votingDelay;
653:        votingDelay = newVotingDelay;
654:
655:        emit VotingDelaySet(oldVotingDelay, votingDelay);
656:    }
...
662:    function _setVotingPeriod(uint256 newVotingPeriod) external {
663:        if (msg.sender != admin) {
664:            revert AdminOnly();
665:        }
666:        require(
667:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
668:            'NounsDAO::_setVotingPeriod: invalid voting period'
669:        );
670:        uint256 oldVotingPeriod = votingPeriod;
671:        votingPeriod = newVotingPeriod;
672:
673:        emit VotingPeriodSet(oldVotingPeriod, votingPeriod);
674:    }
...
681:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
682:        if (msg.sender != admin) {
683:            revert AdminOnly();
684:        }
685:        require(
686:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
687:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
688:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
689:        );
690:        uint256 oldProposalThresholdBPS = proposalThresholdBPS;
691:        proposalThresholdBPS = newProposalThresholdBPS;
692:
693:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS);
694:    }
```

```diff
diff --git a/contracts/governance/NounsDAOInterfaces.sol b/contracts/governance/NounsDAOInterfaces.sol
index 8fb0b4d3..513f2678 100644
--- a/contracts/governance/NounsDAOInterfaces.sol
+++ b/contracts/governance/NounsDAOInterfaces.sol
@@ -382,13 +382,13 @@ contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {
     address public vetoer;

     /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
-    uint256 public votingDelay;
+    uint16 public votingDelay;

     /// @notice The duration of voting on a proposal, in blocks
-    uint256 public votingPeriod;
+    uint24 public votingPeriod;

     /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
-    uint256 public proposalThresholdBPS;
+    uint16 public proposalThresholdBPS;

     /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
     uint256 public quorumVotesBPS;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L89-L91

### Reduce the `uint` type for `delay` and pack it into one storage slot with `admin` to save 1 SLOT (~2000 gas)

*Note: `delay` has an enforced upper bounds and therefore its `uint` type can be reduced to a type which can hold that upper bound.*
```solidity
File: contracts/governance/NounsDAOExecutorV2.sol
89:    address public admin;
90:    address public pendingAdmin;
91:    uint256 public delay;
```

Below are the bounds for `delay`:

```solidity
File: contracts/governance/NounsDAOExecutorV2.sol
87:    uint256 public constant MINIMUM_DELAY = 2 days; // @audit: 172800 seconds
88:    uint256 public constant MAXIMUM_DELAY = 30 days; // @audit: 2592000 seconds, `uint24` is sufficient
```

Below is the function that enforced the bounds for `delay`:

```solidity
File: contracts/governance/NounsDAOExecutorV2.sol
103:    function setDelay(uint256 delay_) public {
104:        require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
105:        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
106:        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
107:        delay = delay_;
108:
109:        emit NewDelay(delay_);
110:    }
```

```diff
diff --git a/contracts/governance/NounsDAOExecutorV2.sol b/contracts/governance/NounsDAOExecutorV2.sol
index f4f85883..693d4034 100644
--- a/contracts/governance/NounsDAOExecutorV2.sol
+++ b/contracts/governance/NounsDAOExecutorV2.sol
@@ -88,7 +88,7 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {

     address public admin;
     address public pendingAdmin;
-    uint256 public delay;
+    uint24 public delay;

     mapping(bytes32 => bool) public queuedTransactions;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L63-L87

### Reduce the `uint` type for `delay` and pack it into one storage slot with `admin` to save 1 SLOT (~2000 gas)

*Note: `delay` has an enforced upper bounds and therefore its `uint` type can be reduced to a type which can hold that upper bound.*
```solidity
File: contracts/governance/NounsDAOExecutor.sol
66:    address public admin;
67:    address public pendingAdmin;
68:    uint256 public delay;
```

Below are the bounds for `delay`:

```solidity
File: contracts/governance/NounsDAOExecutor.sol
63:    uint256 public constant MINIMUM_DELAY = 2 days; // @audit: 172800 seconds
64:    uint256 public constant MAXIMUM_DELAY = 30 days; // @audit: 2592000 seconds, `uint24` is sufficient
```

Below is the function that enforced the bounds for `delay`:

```solidity
File: contracts/governance/NounsDAOExecutor.sol
80:    function setDelay(uint256 delay_) public {
81:        require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
82:        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
83:        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
84:        delay = delay_;
85:
86:        emit NewDelay(delay_);
87:    }
```

```diff
diff --git a/contracts/governance/NounsDAOExecutor.sol b/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..ff83c6c2 100644
--- a/contracts/governance/NounsDAOExecutor.sol
+++ b/contracts/governance/NounsDAOExecutor.sol
@@ -65,7 +65,7 @@ contract NounsDAOExecutor {

     address public admin;
     address public pendingAdmin;
-    uint256 public delay;
+    uint24 public delay;

     mapping(bytes32 => bool) public queuedTransactions;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L51-L63

### Rearrange state variables to pack `minBidIncrementPercentage` and `weth` into one storage slot to save 1 SLOT (~2000 gas)
```solidity
File: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
51:    INounsToken public nouns;
52:
53:    // The address of the WETH contract
54:    address public weth;
55:
56:    // The minimum amount of time left in an auction after a new bid is created
57:    uint256 public timeBuffer;
58:
59:    // The minimum price accepted in an auction
60:    uint256 public reservePrice;
61:
62:    // The minimum percentage difference between the last bid amount and the current bid
63:    uint8 public minBidIncrementPercentage;
```
```diff
diff --git a/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol b/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
index 0bd594b6..f852f24d 100644
--- a/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
+++ b/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
@@ -53,15 +53,15 @@ contract NounsAuctionHouseFork is
     // The address of the WETH contract
     address public weth;

+    // The minimum percentage difference between the last bid amount and the current bid
+    uint8 public minBidIncrementPercentage;
+
     // The minimum amount of time left in an auction after a new bid is created
     uint256 public timeBuffer;

     // The minimum price accepted in an auction
     uint256 public reservePrice;

-    // The minimum percentage difference between the last bid amount and the current bid
-    uint8 public minBidIncrementPercentage;
-
     // The duration of a single auction
     uint256 public duration;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L57-L76

### Reduce the `uint` type for `forkingPeriodEndTimestamp` and pack `escrow`, `forkId`, `forkingPeriodEndTimestamp`, `isMinterLocked`, `isDescriptorLocked`, and `isSeederLocked` into one storage slot to save 2 SLOTs (~4000 gas)

*Note: `forkingPeriodEndTimestamp` holds a timestamp and therefore its `uint` type can be `>= uint(32)`.*
```solidity
File: contracts/governance/fork/newdao/token/NounsTokenFork.sol
57:    /// @notice The escrow contract used to verify ownership of the original Nouns in the post-fork claiming process
58:    INounsDAOForkEscrow public escrow;
59:
60:    /// @notice The fork ID, used when querying the escrow for token ownership
61:    uint32 public forkId;
62:
63:    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
64:    uint256 public remainingTokensToClaim;
65:
66:    /// @notice The forking period expiration timestamp, after which new tokens cannot be claimed by the original DAO
67:    uint256 public forkingPeriodEndTimestamp; // @audit: timestamp, can be `>= uint(32)`
68:
69:    /// @notice Whether the minter can be updated
70:    bool public isMinterLocked;
71:
72:    /// @notice Whether the descriptor can be updated
73:    bool public isDescriptorLocked;
74:
75:    /// @notice Whether the seeder can be updated
76:    bool public isSeederLocked;
```
```diff
diff --git a/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
index a1f9d6d3..573beaca 100644
--- a/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -60,11 +60,8 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /// @notice The fork ID, used when querying the escrow for token ownership
     uint32 public forkId;

-    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
-    uint256 public remainingTokensToClaim;
-
     /// @notice The forking period expiration timestamp, after which new tokens cannot be claimed by the original DAO
-    uint256 public forkingPeriodEndTimestamp;
+    uint32 public forkingPeriodEndTimestamp;

     /// @notice Whether the minter can be updated
     bool public isMinterLocked;
@@ -74,6 +71,8 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint

     /// @notice Whether the seeder can be updated
     bool public isSeederLocked;
+    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
+    uint256 public remainingTokensToClaim;

     /// @notice The noun seeds
     mapping(uint256 => INounsSeeder.Seed) public seeds;
```

## Structs can be packed into fewer storage slots

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L411-L452

### Pack `eta`, `startBlock`, `endBlock`, `canceled`, `vetoed`, `executed, and `creationBlock` into one storage slot to save 4 SLOTs (~8000 gas)

*Note: 1. The `NounsDAOLogicV2` contract will benefit from this optimization as it inherits from `NounsDAOStorageV2` which inherits from `NounsDAOStorageV1Adjusted` (where the struct below is defined). 2. With the exception of the `bools` all the storage variables above hold timestamps/block numbers and therefore can be a `uint type >= uint(32)`.*
```solidity
File: contracts/governance/NounsDAOInterfaces.sol
411:    struct Proposal {
412:        /// @notice Unique id for looking up a proposal
413:        uint256 id;
414:        /// @notice Creator of the proposal
415:        address proposer;
416:        /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
417;        uint256 proposalThreshold;
418:        /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
419:        uint256 quorumVotes;
420:        /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
421;        uint256 eta; // @audit: timestamp, can be `uint type >= uint(32)`
422:        /// @notice the ordered list of target addresses for calls to be made
423:        address[] targets;
424:        /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
425:        uint256[] values;
426:        /// @notice The ordered list of function signatures to be called
427:        string[] signatures;
428:        /// @notice The ordered list of calldata to be passed to each call
429:        bytes[] calldatas;
430:        /// @notice The block at which voting begins: holders must delegate their votes prior to this block
431:        uint256 startBlock; // @audit: block number, can be `uint type >= uint(32)`
432:        /// @notice The block at which voting ends: votes must be cast prior to this block
433:        uint256 endBlock; // @audit: block number, can be `uint type >= uint(32)`
434:        /// @notice Current number of votes in favor of this proposal
435:        uint256 forVotes;
436:        /// @notice Current number of votes in opposition to this proposal
437:        uint256 againstVotes;
438:        /// @notice Current number of votes for abstaining for this proposal
439:        uint256 abstainVotes;
440:        /// @notice Flag marking whether the proposal has been canceled
441:        bool canceled;
442:        /// @notice Flag marking whether the proposal has been vetoed
443:        bool vetoed;
444:        /// @notice Flag marking whether the proposal has been executed
445:        bool executed;
446:        /// @notice Receipts of ballots for the entire set of voters
447:        mapping(address => Receipt) receipts;
448:        /// @notice The total supply at the time of proposal creation
449:        uint256 totalSupply;
450:        /// @notice The block at which this proposal was created
451:        uint256 creationBlock; // @audit: block number, can be `uint type >= uint(32)`
452:    }
```
```diff
diff --git a/contracts/governance/NounsDAOInterfaces.sol b/contracts/governance/NounsDAOInterfaces.sol
index 8fb0b4d3..24579a7e 100644
--- a/contracts/governance/NounsDAOInterfaces.sol
+++ b/contracts/governance/NounsDAOInterfaces.sol
@@ -417,8 +417,6 @@ contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {
         uint256 proposalThreshold;
         /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
         uint256 quorumVotes;
-        /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
-        uint256 eta;
         /// @notice the ordered list of target addresses for calls to be made
         address[] targets;
         /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
@@ -427,28 +425,30 @@ contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {
         string[] signatures;
         /// @notice The ordered list of calldata to be passed to each call
         bytes[] calldatas;
-        /// @notice The block at which voting begins: holders must delegate their votes prior to this block
-        uint256 startBlock;
-        /// @notice The block at which voting ends: votes must be cast prior to this block
-        uint256 endBlock;
         /// @notice Current number of votes in favor of this proposal
         uint256 forVotes;
         /// @notice Current number of votes in opposition to this proposal
         uint256 againstVotes;
         /// @notice Current number of votes for abstaining for this proposal
         uint256 abstainVotes;
+        /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
+        uint32 eta;
+        /// @notice The block at which voting begins: holders must delegate their votes prior to this block
+        uint64 startBlock;
+        /// @notice The block at which voting ends: votes must be cast prior to this block
+        uint64 endBlock;
         /// @notice Flag marking whether the proposal has been canceled
         bool canceled;
         /// @notice Flag marking whether the proposal has been vetoed
         bool vetoed;
         /// @notice Flag marking whether the proposal has been executed
         bool executed;
+        /// @notice The block at which this proposal was created
+        uint64 creationBlock;
         /// @notice Receipts of ballots for the entire set of voters
         mapping(address => Receipt) receipts;
         /// @notice The total supply at the time of proposal creation
         uint256 totalSupply;
-        /// @notice The block at which this proposal was created
-        uint256 creationBlock;
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L719-L770

### Pack `eta`, `startBlock`, `endBlock`, `canceled`, `vetoed`, `executed, and `executeOnTimelockV1` into one storage slot to save 4 SLOTs (~8000 gas)

*Note: 1. The `NounsDAOLogicV3` contract will benefit from this optimization as it inherits from `NounsDAOStorageV3` (where the struct below is defined). 2. With the exception of the `bools` all the storage variables above hold timestamps/block numbers and therefore can be a `uint type >= uint(32)`.*
```solidity
File: contracts/governance/NounsDAOInterfaces.sol
719:    struct Proposal {
720:        /// @notice Unique id for looking up a proposal
721:        uint256 id;
722:        /// @notice Creator of the proposal
723:        address proposer;
724:        /// @notice The number of votes needed to create a proposal at the time of proposal creation. *DIFFERS from GovernerBravo
725:        uint256 proposalThreshold;
726:        /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
727:        uint256 quorumVotes;
728:        /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
729:        uint256 eta; // @audit: timestamp, can be `uint type >= uint(32)`
730:        /// @notice the ordered list of target addresses for calls to be made
...
738:        /// @notice The block at which voting begins: holders must delegate their votes prior to this block
739:        uint256 startBlock; // @audit: block number, can be `uint type >= uint(32)`
740:        /// @notice The block at which voting ends: votes must be cast prior to this block
741:        uint256 endBlock; // @audit: block number, can be `uint type >= uint(32)`
742:        /// @notice Current number of votes in favor of this proposal
743:        uint256 forVotes;
744:        /// @notice Current number of votes in opposition to this proposal
745:        uint256 againstVotes;
746:        /// @notice Current number of votes for abstaining for this proposal
747:        uint256 abstainVotes;
748:        /// @notice Flag marking whether the proposal has been canceled
749:        bool canceled;
750:        /// @notice Flag marking whether the proposal has been vetoed
751:        bool vetoed;
752:        /// @notice Flag marking whether the proposal has been executed
753:        bool executed;
754:        /// @notice Receipts of ballots for the entire set of voters
755:        mapping(address => Receipt) receipts;
...
768:        /// @notice When true, a proposal would be executed on timelockV1 instead of the current timelock
769:        bool executeOnTimelockV1;
    }
```
```diff
diff --git a/contracts/governance/NounsDAOInterfaces.sol b/contracts/governance/NounsDAOInterfaces.sol
index 8fb0b4d3..a88ca430 100644
--- a/contracts/governance/NounsDAOInterfaces.sol
+++ b/contracts/governance/NounsDAOInterfaces.sol
@@ -725,8 +725,6 @@ contract NounsDAOStorageV3 {
         uint256 proposalThreshold;
         /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed at the time of proposal creation. *DIFFERS from GovernerBravo
         uint256 quorumVotes;
-        /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
-        uint256 eta;
         /// @notice the ordered list of target addresses for calls to be made
         address[] targets;
         /// @notice The ordered list of values (i.e. msg.value) to be passed to the calls to be made
@@ -735,22 +733,26 @@ contract NounsDAOStorageV3 {
         string[] signatures;
         /// @notice The ordered list of calldata to be passed to each call
         bytes[] calldatas;
-        /// @notice The block at which voting begins: holders must delegate their votes prior to this block
-        uint256 startBlock;
-        /// @notice The block at which voting ends: votes must be cast prior to this block
-        uint256 endBlock;
         /// @notice Current number of votes in favor of this proposal
         uint256 forVotes;
         /// @notice Current number of votes in opposition to this proposal
         uint256 againstVotes;
         /// @notice Current number of votes for abstaining for this proposal
         uint256 abstainVotes;
+        /// @notice The timestamp that the proposal will be available for execution, set once the vote succeeds
+        uint32 eta;
+        /// @notice The block at which voting begins: holders must delegate their votes prior to this block
+        uint64 startBlock;
+        /// @notice The block at which voting ends: votes must be cast prior to this block
+        uint64 endBlock;
         /// @notice Flag marking whether the proposal has been canceled
         bool canceled;
         /// @notice Flag marking whether the proposal has been vetoed
         bool vetoed;
         /// @notice Flag marking whether the proposal has been executed
         bool executed;
+        /// @notice When true, a proposal would be executed on timelockV1 instead of the current timelock
+        bool executeOnTimelockV1;
         /// @notice Receipts of ballots for the entire set of voters
         mapping(address => Receipt) receipts;
         /// @notice The total supply at the time of proposal creation
@@ -765,8 +767,6 @@ contract NounsDAOStorageV3 {
         uint64 placeholder;
         /// @notice The signers of a proposal, when using proposeBySigs
         address[] signers;
-        /// @notice When true, a proposal would be executed on timelockV1 instead of the current timelock
-        bool executeOnTimelockV1;
     }
```

## Cache state variables outside of loop to avoid reading/writing storage on every iteration
Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a `Gwarmaccess (100 gas)` per loop iteration.

Total Instances: `11`

Estimated Gas Saved: `2280` (This will be more if there are > 1 loop iterations for each instance)

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L336-L351

*Gas Savings for `NounsDAOLogicV2.execute`, obtained via protocol's tests: Avg 230 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  74613   |  221026  |  121065 |    18    |
| After  |  74624   |  220499  |  120835 |    18    |

### Cache `timelock` and `proposal.eta` to save 2 SLOAD per iteration
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
336:    function execute(uint256 proposalId) external {
337:        require(
338:            state(proposalId) == ProposalState.Queued,
339:            'NounsDAO::execute: proposal can only be executed if it is queued'
340:        );
341:        Proposal storage proposal = _proposals[proposalId];
342:        proposal.executed = true;
343:        for (uint256 i = 0; i < proposal.targets.length; i++) {
344:            timelock.executeTransaction( // @audit: `timelock` read on every iteration
345:                proposal.targets[i],
346:                proposal.values[i],
347:                proposal.signatures[i],
348:                proposal.calldatas[i],
349:                proposal.eta // @audit: `proposal.eta` read on every iteration
350:            );
351:        }
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..d98b307f 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -340,13 +340,15 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         );
         Proposal storage proposal = _proposals[proposalId];
         proposal.executed = true;
+        INounsDAOExecutor _timelock = timelock;
+        uint256 _eta = proposal.eta;
         for (uint256 i = 0; i < proposal.targets.length; i++) {
-            timelock.executeTransaction(
+            _timelock.executeTransaction(
                 proposal.targets[i],
                 proposal.values[i],
                 proposal.signatures[i],
                 proposal.calldatas[i],
-                proposal.eta
+                _eta
             );
         }
         emit ProposalExecuted(proposalId);
```

*Gas Savings for `NounsDAOLogicV2.queue`, obtained via protocol's tests: Avg 431 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  68684   |  173646  |  101175 |    21    |
| After  |  68476   |  172799  |  100744 |    21    |

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L298-L330

*The `queueOrRevertInternal` internal function reads the `timelock` state variable twice on each iteration. We can cache `timelock` outside of the loop and pass the cached value into `queueOrRevertInternal` to save 2 SLOADs per iteration. See Diff below for necessary refactoring.*
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
298:    function queue(uint256 proposalId) external {
299:        require(
300:            state(proposalId) == ProposalState.Succeeded,
301:            'NounsDAO::queue: proposal can only be queued if it is succeeded'
302:        );
303:        Proposal storage proposal = _proposals[proposalId];
304:        uint256 eta = block.timestamp + timelock.delay(); // @audit: 1st sload
305:        for (uint256 i = 0; i < proposal.targets.length; i++) {
306:            queueOrRevertInternal( // @audit: reads `timelock` twice on every iteration
307:                proposal.targets[i],
308:                proposal.values[i],
309:                proposal.signatures[i],
310:                proposal.calldatas[i],
311:                eta
312:            );
313:        }
314:        proposal.eta = eta;
315:        emit ProposalQueued(proposalId, eta);
316:    }
317:
318:    function queueOrRevertInternal(
319:        address target,
320:        uint256 value,
321:        string memory signature,
322:        bytes memory data,
323:        uint256 eta
324:    ) internal {
325:        require(
326:            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))), // @audit: 2nd sload
327:            'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
328:        );
329:        timelock.queueTransaction(target, value, signature, data, eta); // @audit: 3rd sload
330:    }
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..57b3a391 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -301,9 +301,11 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             'NounsDAO::queue: proposal can only be queued if it is succeeded'
         );
         Proposal storage proposal = _proposals[proposalId];
-        uint256 eta = block.timestamp + timelock.delay();
+        INounsDAOExecutor _timelock = timelock;
+        uint256 eta = block.timestamp + _timelock.delay();
         for (uint256 i = 0; i < proposal.targets.length; i++) {
             queueOrRevertInternal(
+                _timelock,
                 proposal.targets[i],
                 proposal.values[i],
                 proposal.signatures[i],
@@ -316,6 +318,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
     }

     function queueOrRevertInternal(
+        INounsDAOExecutor _timelock,
         address target,
         uint256 value,
         string memory signature,
@@ -323,10 +326,10 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 eta
     ) internal {
         require(
-            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
+            !_timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
             'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
         );
-        timelock.queueTransaction(target, value, signature, data, eta);
+        _timelock.queueTransaction(target, value, signature, data, eta);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L74-L85

*Gas Savings for `NounsDAOLogicV3.escrowToFork`, obtained via protocol's tests: Avg 210 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  154228  |  3597341 |  288475 |    107   |
| After  |  154242  |  3592588 |  288265 |    107   |

### Cache `ds.nouns` outside the loop to save 1 SLOAD per iteration
```solidity
File: contracts/governance/fork/NounsDAOV3Fork.sol
74:    function escrowToFork(
75:        NounsDAOStorageV3.StorageV3 storage ds,
76:        uint256[] calldata tokenIds,
77:        uint256[] calldata proposalIds,
78:        string calldata reason
79:    ) external {
80:        if (isForkPeriodActive(ds)) revert ForkPeriodActive();
81:        INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
82:
83:        for (uint256 i = 0; i < tokenIds.length; i++) {
84:            ds.nouns.safeTransferFrom(msg.sender, address(forkEscrow), tokenIds[i]); // @audit: `ds.nouns` read on each iteration
85:        }
```
```diff
diff --git a/contracts/governance/fork/NounsDAOV3Fork.sol b/contracts/governance/fork/NounsDAOV3Fork.sol
index d87ffc70..3867fde2 100644
--- a/contracts/governance/fork/NounsDAOV3Fork.sol
+++ b/contracts/governance/fork/NounsDAOV3Fork.sol
@@ -17,7 +17,7 @@

 pragma solidity ^0.8.19;

-import { NounsDAOStorageV3, INounsDAOForkEscrow, INounsDAOExecutorV2 } from '../NounsDAOInterfaces.sol';
+import { NounsDAOStorageV3, INounsDAOForkEscrow, INounsDAOExecutorV2, NounsTokenLike } from '../NounsDAOInterfaces.sol';
 import { IERC20 } from '@openzeppelin/contracts/token/ERC20/IERC20.sol';
 import { NounsTokenFork } from './newdao/token/NounsTokenFork.sol';

@@ -79,9 +79,10 @@ library NounsDAOV3Fork {
     ) external {
         if (isForkPeriodActive(ds)) revert ForkPeriodActive();
         INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
-
+
+        NounsTokenLike nouns = ds.nouns;
         for (uint256 i = 0; i < tokenIds.length; i++) {
-            ds.nouns.safeTransferFrom(msg.sender, address(forkEscrow), tokenIds[i]);
+            nouns.safeTransferFrom(msg.sender, address(forkEscrow), tokenIds[i]);
         }

         emit EscrowedToFork(forkEscrow.forkId(), msg.sender, tokenIds, proposalIds, reason);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L141-L155

*Gas Savings for `NounsDAOLogicV3.joinFork`, obtained via protocol's tests: Avg 31 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  53456   |  546057  |  188263 |    10    |
| After  |  53467   |  545974  |  188232 |    10    |

### Cache `ds.nouns` outside the loop to save 1 SLOAD per iteration
```solidity
File: contracts/governance/fork/NounsDAOV3Fork.sol
141:    function joinFork(
142:        NounsDAOStorageV3.StorageV3 storage ds,
143:        uint256[] calldata tokenIds,
144:        uint256[] calldata proposalIds,
145:        string calldata reason
146:    ) external {
...
153:        for (uint256 i = 0; i < tokenIds.length; i++) {
154:            ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]); // @audit: `ds.nouns` read on every iteration
155:        }
```
```diff
diff --git a/contracts/governance/fork/NounsDAOV3Fork.sol b/contracts/governance/fork/NounsDAOV3Fork.sol
index d87ffc70..9f0ddb70 100644
--- a/contracts/governance/fork/NounsDAOV3Fork.sol
+++ b/contracts/governance/fork/NounsDAOV3Fork.sol
@@ -17,7 +17,7 @@

 pragma solidity ^0.8.19;

-import { NounsDAOStorageV3, INounsDAOForkEscrow, INounsDAOExecutorV2 } from '../NounsDAOInterfaces.sol';
+import { NounsDAOStorageV3, INounsDAOForkEscrow, INounsDAOExecutorV2, NounsTokenLike } from '../NounsDAOInterfaces.sol';
 import { IERC20 } from '@openzeppelin/contracts/token/ERC20/IERC20.sol';
 import { NounsTokenFork } from './newdao/token/NounsTokenFork.sol';

@@ -149,9 +149,10 @@ library NounsDAOV3Fork {
         INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
         address timelock = address(ds.timelock);
         sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, adjustedTotalSupply(ds));
-
+
+        NounsTokenLike nouns = ds.nouns;
         for (uint256 i = 0; i < tokenIds.length; i++) {
-            ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]);
+            nouns.transferFrom(msg.sender, timelock, tokenIds[i]);
         }

         NounsTokenFork(ds.forkDAOToken).claimDuringForkPeriod(msg.sender, tokenIds);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L116-L122

*Gas Savings for `NounsDAOForkEscrow.returnTokensToOwner`, obtained via protocol's tests: Avg 108 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  30662   |  229510  |  48036  |    46    |
| After  |  30519   |  229153  |  47928  |    46    |

*Note In this instance we are also taking advantage of the fact that we can cache the storage variable inside of a storage pointer for our the `escrowedTokensByForkId` nested mapping.*

```solidity
File: contracts/governance/fork/NounsDAOForkEscrow.sol
116:    function returnTokensToOwner(address owner, uint256[] calldata tokenIds) external onlyDAO {
117:        for (uint256 i = 0; i < tokenIds.length; i++) {
118:            if (currentOwnerOf(tokenIds[i]) != owner) revert NotOwner();
119:
120:            nounsToken.transferFrom(address(this), owner, tokenIds[i]);
121:            escrowedTokensByForkId[forkId][tokenIds[i]] = address(0); // @audit: `forkId` read on every iteration
122:        }
```
```diff
diff --git a/contracts/governance/fork/NounsDAOForkEscrow.sol b/contracts/governance/fork/NounsDAOForkEscrow.sol
index 53c54c9e..4d9052ff 100644
--- a/contracts/governance/fork/NounsDAOForkEscrow.sol
+++ b/contracts/governance/fork/NounsDAOForkEscrow.sol
@@ -114,11 +114,12 @@ contract NounsDAOForkEscrow is IERC721Receiver {
      * @param tokenIds The ids of the tokens being unescrowed
      */
     function returnTokensToOwner(address owner, uint256[] calldata tokenIds) external onlyDAO {
+        mapping(uint256 => address) storage _escrowedTokensByForkId = escrowedTokensByForkId[forkId];
         for (uint256 i = 0; i < tokenIds.length; i++) {
             if (currentOwnerOf(tokenIds[i]) != owner) revert NotOwner();

             nounsToken.transferFrom(address(this), owner, tokenIds[i]);
-            escrowedTokensByForkId[forkId][tokenIds[i]] = address(0);
+            _escrowedTokensByForkId[tokenIds[i]] = address(0);
         }

         numTokensInEscrow -= tokenIds.length;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L148-L154

*Gas Savings for `NounsTokenFork.claimFromEscrow`, obtained via protocol's tests: Avg 119 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  217402  |  3683287 |  387833 |    24    |
| After  |  217439  |  3680633 |  387714 |    24    |

### Cache `escrow` and `forkId` outside the loop to save 2 SLOADs per iteration
```solidity
File: contracts/governance/fork/newdao/token/NounsTokenFork.sol
148:    function claimFromEscrow(uint256[] calldata tokenIds) external {
149:        for (uint256 i = 0; i < tokenIds.length; i++) {
150:            uint256 nounId = tokenIds[i];
151:            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim(); // @audit: `escrow` and `forkId` read on every iteration
152:
153:            _mintWithOriginalSeed(msg.sender, nounId);
154:        }
```
```diff
diff --git a/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
index a1f9d6d3..1f532025 100644
--- a/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -146,9 +146,11 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
      * @param tokenIds The token IDs to claim
      */
     function claimFromEscrow(uint256[] calldata tokenIds) external {
+        INounsDAOForkEscrow _escrow = escrow;
+        uint32 _forkId = forkId;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             uint256 nounId = tokenIds[i];
-            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();
+            if (_escrow.ownerOfEscrowedToken(_forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();

             _mintWithOriginalSeed(msg.sender, nounId);
         }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L218-L225

*Gas Savings for `NounsDAOLogicV1Fork.quit(uint256[])`, obtained via protocol's tests: Avg 141 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  202066  |  501600  |  245981 |    16    |
| After  |  202001  |  501005  |  245840 |    16    |

### Cache `nouns` outside of loop to save 1 SLOAD per iteration
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
218:    function quitInternal(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) internal {
219:        checkGovernanceActive();
220:
221:        uint256 totalSupply = adjustedTotalSupply();
222:
223:        for (uint256 i = 0; i < tokenIds.length; i++) {
224:            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]); // @audit: `nouns` read on every iteration
225:        }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..52718cab 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -219,9 +219,10 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         checkGovernanceActive();

         uint256 totalSupply = adjustedTotalSupply();
-
+
+        INounsTokenForkLike _nouns = nouns;
         for (uint256 i = 0; i < tokenIds.length; i++) {
-            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
+            _nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
         }

         uint256[] memory balancesToSend = new uint256[](erc20TokensToInclude.length);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L218-L242

*Gas Savings for `NounsDAOLogicV1Fork.quit(uint256[])`, obtained via protocol's tests: Avg 453 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  202066  |  501600  |  245981 |    16    |
| After  |  201720  |  500679  |  245528 |    16    |

### Cache `timelock` to save at least 5 SLOADs. `timelock` is read in 3 loops in the function below.
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
218:    function quitInternal(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) internal {
219:        checkGovernanceActive();
220:
221:        uint256 totalSupply = adjustedTotalSupply();
222:
223:        for (uint256 i = 0; i < tokenIds.length; i++) {
224:            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]); // @audit: 1st sload for `timelock` + on every iteration
225:        }
226:
227:        uint256[] memory balancesToSend = new uint256[](erc20TokensToInclude.length);
228:
229:        // Capture balances to send before actually sending them, to avoid the risk of external calls changing balances.
230:        uint256 ethToSend = (address(timelock).balance * tokenIds.length) / totalSupply; // @audit: 2nd sload
231:        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
232:            IERC20 erc20token = IERC20(erc20TokensToInclude[i]);
233:            balancesToSend[i] = (erc20token.balanceOf(address(timelock)) * tokenIds.length) / totalSupply; // @audit: 3rd sload + on every iteration
234:        }
235:
236:        // Send ETH and ERC20 tokens
237:        timelock.sendETH(payable(msg.sender), ethToSend); // @audit: 4th sload
238:        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
239:            if (balancesToSend[i] > 0) {
240:                timelock.sendERC20(msg.sender, erc20TokensToInclude[i], balancesToSend[i]); // @audit: 5th sload + on every iteration
241:            }
242:        }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..ee5ea5b1 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -219,25 +219,25 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         checkGovernanceActive();

         uint256 totalSupply = adjustedTotalSupply();
-
+        NounsDAOExecutorV2 _timelock = timelock;
         for (uint256 i = 0; i < tokenIds.length; i++) {
-            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
+            nouns.transferFrom(msg.sender, address(_timelock), tokenIds[i]);
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

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L390-L422

*Gas Savings for `NounsDAOLogicV1Fork.queue`, obtained via protocol's tests: Avg 157 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  66570   |  71232   |  53392  |    10    |
| After  |  66374   |  71036   |  53235  |    10    |

### Cache `timelock` outside of the loop and pass cached value into `queueOrRevertInternal` to save 2 SLOADs per iteration
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
390:    function queue(uint256 proposalId) external {
391:        require(
392:            state(proposalId) == ProposalState.Succeeded,
393:            'NounsDAO::queue: proposal can only be queued if it is succeeded'
394:        );
395:        Proposal storage proposal = _proposals[proposalId];
396:        uint256 eta = block.timestamp + timelock.delay(); // @audit: 1st sload
397:        for (uint256 i = 0; i < proposal.targets.length; i++) {
398:            queueOrRevertInternal( // @audit: reads `timelock` twice on every iteration
399:                proposal.targets[i],
400:                proposal.values[i],
401:                proposal.signatures[i],
402:                proposal.calldatas[i],
403:                eta
404:            );
405:        }
406:        proposal.eta = eta;
407:        emit ProposalQueued(proposalId, eta);
408:    }
409:
410:    function queueOrRevertInternal(
411:        address target,
412:        uint256 value,
413:        string memory signature,
414:        bytes memory data,
415:        uint256 eta
416:    ) internal {
417:        require(
418:            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))), // @audit: 2nd sload + on every iteration
419:            'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
420:        );
421:        timelock.queueTransaction(target, value, signature, data, eta); // @audit: 3rd sload + on every iteration
422:    }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..f0df10e7 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -393,14 +393,16 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
             'NounsDAO::queue: proposal can only be queued if it is succeeded'
         );
         Proposal storage proposal = _proposals[proposalId];
-        uint256 eta = block.timestamp + timelock.delay();
+        NounsDAOExecutorV2 _timelock = timelock;
+        uint256 eta = block.timestamp + _timelock.delay();
         for (uint256 i = 0; i < proposal.targets.length; i++) {
             queueOrRevertInternal(
                 proposal.targets[i],
                 proposal.values[i],
                 proposal.signatures[i],
                 proposal.calldatas[i],
-                eta
+                eta,
+                _timelock
             );
         }
         proposal.eta = eta;
@@ -412,13 +414,14 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         uint256 value,
         string memory signature,
         bytes memory data,
-        uint256 eta
+        uint256 eta,
+        NounsDAOExecutorV2 _timelock
     ) internal {
         require(
-            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
+            !_timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
             'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
         );
-        timelock.queueTransaction(target, value, signature, data, eta);
+        _timelock.queueTransaction(target, value, signature, data, eta);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L428-L443

### Cache `timelock` and `proposal.eta` outside of the loop to save 2 SLOADs per iteration
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
428:    function execute(uint256 proposalId) external {
429:        require(
430:            state(proposalId) == ProposalState.Queued,
431:            'NounsDAO::execute: proposal can only be executed if it is queued'
432:        );
433:        Proposal storage proposal = _proposals[proposalId];
434:        proposal.executed = true;
435:        for (uint256 i = 0; i < proposal.targets.length; i++) {
436:            timelock.executeTransaction( // @audit: `timelock` read on every iteration
437:                proposal.targets[i],
438:                proposal.values[i],
439:                proposal.signatures[i],
440:                proposal.calldatas[i],
441:                proposal.eta // @audit: `proposal.eta` read on every iteration
442:            );
443:        }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..92268937 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -432,13 +432,15 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         );
         Proposal storage proposal = _proposals[proposalId];
         proposal.executed = true;
+        NounsDAOExecutorV2 _timelock = timelock;
+        uint256 _eta = proposal.eta;
         for (uint256 i = 0; i < proposal.targets.length; i++) {
-            timelock.executeTransaction(
+            _timelock.executeTransaction(
                 proposal.targets[i],
                 proposal.values[i],
                 proposal.signatures[i],
                 proposal.calldatas[i],
-                proposal.eta
+                _eta
             );
         }
         emit ProposalExecuted(proposalId);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L451-L470

### Cache `timelock` and `proposal.eta` outside of the loop to save 2 SLOADs per iteration
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
451:    function cancel(uint256 proposalId) external {
452:        require(state(proposalId) != ProposalState.Executed, 'NounsDAO::cancel: cannot cancel executed proposal');
453:
454:        Proposal storage proposal = _proposals[proposalId];
455:        require(
456:            msg.sender == proposal.proposer ||
457:                nouns.getPriorVotes(proposal.proposer, block.number - 1) <= proposal.proposalThreshold,
458:            'NounsDAO::cancel: proposer above threshold'
459:        );
460:
461:        proposal.canceled = true;
462:        for (uint256 i = 0; i < proposal.targets.length; i++) {
463:            timelock.cancelTransaction( // @audit: `timelock` read on every iteration
464:                proposal.targets[i],
465:                proposal.values[i],
466:                proposal.signatures[i],
467:                proposal.calldatas[i],
468:                proposal.eta // @audit: `proposal.eta` read on every iteration
469:            );
470:        }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..92268937 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -459,13 +461,16 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         );

         proposal.canceled = true;
+
+        NounsDAOExecutorV2 _timelock = timelock;
+        uint256 _eta = proposal.eta;
         for (uint256 i = 0; i < proposal.targets.length; i++) {
-            timelock.cancelTransaction(
+            _timelock.cancelTransaction(
                 proposal.targets[i],
                 proposal.values[i],
                 proposal.signatures[i],
                 proposal.calldatas[i],
-                proposal.eta
+                _eta
             );
         }
```

## Avoid emitting storage variables
In the instances below we can emit calldata/stack values instead of emitting storage values. This will result in using a cheap `CALLDATALOAD/DUP` instead of an expensive `SLOAD`.

Total Instances: `8`

Estimated Gas Saved: `8 * 100 = 800`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L644-L656

### Emit `newVotingDelay` instead of re-reading from storage to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
644:    function _setVotingDelay(uint256 newVotingDelay) external {
645:        if (msg.sender != admin) {
646:            revert AdminOnly();
647:        }
648:        require(
649:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
650:            'NounsDAO::_setVotingDelay: invalid voting delay'
651:        );
652:        uint256 oldVotingDelay = votingDelay;
653:        votingDelay = newVotingDelay;
654:
655:        emit VotingDelaySet(oldVotingDelay, votingDelay); // @audit: unnecessary sload, emit calldata
656:    }
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..1e44b729 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -652,7 +652,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldVotingDelay = votingDelay;
         votingDelay = newVotingDelay;

-        emit VotingDelaySet(oldVotingDelay, votingDelay);
+        emit VotingDelaySet(oldVotingDelay, newVotingDelay);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L662-L674

### Emit `newVotingPeriod` instead of re-reading from storage to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
662:    function _setVotingPeriod(uint256 newVotingPeriod) external {
663:        if (msg.sender != admin) {
664:            revert AdminOnly();
665:        }
666:        require(
667:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
668:            'NounsDAO::_setVotingPeriod: invalid voting period'
669:        );
670:        uint256 oldVotingPeriod = votingPeriod;
671:        votingPeriod = newVotingPeriod;
672:
673:        emit VotingPeriodSet(oldVotingPeriod, votingPeriod); // @audit: unnecessary sload, emit calldata
674:    }
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..724e09cd 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -670,7 +670,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldVotingPeriod = votingPeriod;
         votingPeriod = newVotingPeriod;

-        emit VotingPeriodSet(oldVotingPeriod, votingPeriod);
+        emit VotingPeriodSet(oldVotingPeriod, newVotingPeriod);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L681-L694

### Emit `newProposalThresholdBPS` instead of re-reading from storage to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
681:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
682:        if (msg.sender != admin) {
683:            revert AdminOnly();
684:        }
685:        require(
686:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
687:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
688:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
689:        );
690:        uint256 oldProposalThresholdBPS = proposalThresholdBPS;
691:        proposalThresholdBPS = newProposalThresholdBPS;
692:
693:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS); // @audit: unnecessary sload, emit calldata
694:    }
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..18e0c01c 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -690,7 +690,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldProposalThresholdBPS = proposalThresholdBPS;
         proposalThresholdBPS = newProposalThresholdBPS;

-        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS);
+        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, newProposalThresholdBPS);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L854-L870

### Emit `oldPendingAdmin` and `address(0)` instead of reading from storage to save 2 SLOADs
```solidity
File: contracts/governance/NounsDAOLogicV2.sol
854:    function _acceptAdmin() external {
855:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
856:        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');
857:
859:        // Save current values for inclusion in log
860:        address oldAdmin = admin;
861:        address oldPendingAdmin = pendingAdmin;
862:
863:        // Store admin with value pendingAdmin
864:        admin = pendingAdmin;
865:
866:        // Clear the pending value
867:        pendingAdmin = address(0);
868:
869:        emit NewAdmin(oldAdmin, admin); // @audit: unnecessary sload, `admin == pendingAdmin == oldPendingAdmin`
870:        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin); // @audit: unnecessary sload, `pendingAdmin == address(0)`
871:    }
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..ed25f239 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -865,8 +865,8 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         // Clear the pending value
         pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, admin);
-        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
+        emit NewAdmin(oldAdmin, oldPendingAdmin);
+        emit NewPendingAdmin(oldPendingAdmin, address(0));
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L78-L86

### Emit `implementation_` to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOProxy.sol
78:    function _setImplementation(address implementation_) public {
79:        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
80:        require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');
81:
82:        address oldImplementation = implementation;
83:        implementation = implementation_;
84:
85:        emit NewImplementation(oldImplementation, implementation); // @audit: unnecessary sload, emit calldata
86:    }
```
```diff
diff --git a/contracts/governance/NounsDAOProxy.sol b/contracts/governance/NounsDAOProxy.sol
index f42e3bf6..94621495 100644
--- a/contracts/governance/NounsDAOProxy.sol
+++ b/contracts/governance/NounsDAOProxy.sol
@@ -82,7 +82,7 @@ contract NounsDAOProxy is NounsDAOProxyStorage, NounsDAOEvents {
         address oldImplementation = implementation;
         implementation = implementation_;

-        emit NewImplementation(oldImplementation, implementation);
+        emit NewImplementation(oldImplementation, implementation_);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L89-L95

### Emit `msg.sender` to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOExecutor.sol
89:    function acceptAdmin() public {
90:        require(msg.sender == pendingAdmin, 'NounsDAOExecutor::acceptAdmin: Call must come from pendingAdmin.');
91:        admin = msg.sender;
92:        pendingAdmin = address(0);
93:
94:        emit NewAdmin(admin); // @audit: unnecessary sload, emit `msg.sender`
95:    }
```
```diff
diff --git a/contracts/governance/NounsDAOExecutor.sol b/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..9d48d4c3 100644
--- a/contracts/governance/NounsDAOExecutor.sol
+++ b/contracts/governance/NounsDAOExecutor.sol
@@ -91,7 +91,7 @@ contract NounsDAOExecutor {
         admin = msg.sender;
         pendingAdmin = address(0);

-        emit NewAdmin(admin);
+        emit NewAdmin(msg.sender);
     }

     function setPendingAdmin(address pendingAdmin_) public {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L97-L105

### Emit `pendingAdmin_` to save 1 SLOAD
```solidity
File: contracts/governance/NounsDAOExecutor.sol
97:     function setPendingAdmin(address pendingAdmin_) public {
98:         require(
99:             msg.sender == address(this),
100:            'NounsDAOExecutor::setPendingAdmin: Call must come from NounsDAOExecutor.'
101:        );
102:        pendingAdmin = pendingAdmin_;
103:
104:        emit NewPendingAdmin(pendingAdmin); // @audit: unnecessary sload, emit calldata
105:    }
```
```diff
diff --git a/contracts/governance/NounsDAOExecutor.sol b/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..453da1eb 100644
--- a/contracts/governance/NounsDAOExecutor.sol
+++ b/contracts/governance/NounsDAOExecutor.sol
@@ -101,7 +101,7 @@ contract NounsDAOExecutor {
         );
         pendingAdmin = pendingAdmin_;

-        emit NewPendingAdmin(pendingAdmin);
+        emit NewPendingAdmin(pendingAdmin_);
     }

     function queueTransaction(
```

## Refactor functions to avoid unnecessary SLOADs and SSTOREs
The functions below read storage slots that are previously read in the functions that invoke them. We can refactor the functions so we could pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions. Two methods are used when refactoring these instances: In some instances the logic from invoked internal/public functions are inlined so as to more accurately illustrate the optimization. In other instances the invoked internal/public functions are refactored to take in the cached state variables.

Total Instances: `6`

Estimated Gas Saved: `1812`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L218-L225

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L376-L384

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L777-L779

*Gas Savings for `NounsDAOLogicV1Fork.quit(uint256[])`, obtained via protocol's tests: Avg 589 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  202066  |  501600  |  245981 |    16    |
| After  |  201486  |  500489  |  245392 |    16    |

### Cache `nouns` and pass cached value into `checkGovernanceActive` and `adjustedTotalSupply` to save 5 SLOADs
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
218:    function quitInternal(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) internal {
219:        checkGovernanceActive(); // @audit: reads `nouns` twice
220:
221:        uint256 totalSupply = adjustedTotalSupply(); // @audit: reads `nouns` 3 times
222:
223:        for (uint256 i = 0; i < tokenIds.length; i++) {
224:            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]); // @audit: `nouns` read on every iteration
225:        }

376:    function checkGovernanceActive() internal view {
377:        if (block.timestamp < nouns.forkingPeriodEndTimestamp()) { // @audit: 1st sload
378:            revert GovernanceBlockedDuringForkingPeriod();
379:        }
380:
381:        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) { // @audit: 2nd sload
382:            revert WaitingForTokensToClaimOrExpiration();
383:        }
384:    }

777:    function adjustedTotalSupply() public view returns (uint256) {
778:        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim(); // @audit: 3rd, 4th, and 5th sloads
779:    }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..8ea63694 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -216,12 +216,13 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
     }

     function quitInternal(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) internal {
-        checkGovernanceActive();
+        INounsTokenForkLike _nouns = nouns;
+        checkGovernanceActive(_nouns);

-        uint256 totalSupply = adjustedTotalSupply();
+        uint256 totalSupply = _adjustedTotalSupply(_nouns);

         for (uint256 i = 0; i < tokenIds.length; i++) {
-            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
+            _nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
         }

         uint256[] memory balancesToSend = new uint256[](erc20TokensToInclude.length);
@@ -373,12 +375,12 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
      * 1. All tokens are claimed
      * 2. The delayed governance expiration timestamp is reached
      */
-    function checkGovernanceActive() internal view {
-        if (block.timestamp < nouns.forkingPeriodEndTimestamp()) {
+    function checkGovernanceActive(INounsTokenForkLike _nouns) internal view {
+        if (block.timestamp < _nouns.forkingPeriodEndTimestamp()) {
             revert GovernanceBlockedDuringForkingPeriod();
         }

-        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
+        if (block.timestamp < delayedGovernanceExpirationTimestamp && _nouns.remainingTokensToClaim() > 0) {
             revert WaitingForTokensToClaimOrExpiration();
         }
     }
@@ -775,7 +777,12 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
     }

     function adjustedTotalSupply() public view returns (uint256) {
-        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim();
+        return _adjustedTotalSupply(nouns);
+    }
+
+    // @audit: for benchmarking purposes and to stay compatible with interfaces
+    function _adjustedTotalSupply(INounsTokenForkLike _nouns) internal view returns (uint256) {
+        return _nouns.totalSupply() - _nouns.balanceOf(address(timelock)) + _nouns.remainingTokensToClaim();
     }

     function erc20TokensToIncludeInQuitArray() public view returns(address[] memory) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L272-L290

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L376-L384

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L777-L779

*Gas Savings for `NounsDAOLogicV1Fork.propose`, obtained via protocol's tests: Avg 496 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  367050  |  417023  |  295642 |    34    |
| After  |  366436  |  416409  |  295146 |    34    |

### Cache `nouns` and pass cached value into `checkGovernanceActive` and `adjustedTotalSupply` to save 5 SLOADs
```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
272:    function propose(
273:        address[] memory targets,
274:        uint256[] memory values,
275:        string[] memory signatures,
276:        bytes[] memory calldatas,
277:        string memory description
278:    ) public returns (uint256) {
279:        checkGovernanceActive(); // @audit: reads `nouns` twice
280:
281:        ProposalTemp memory temp;
282:
283:
284:        temp.totalSupply = adjustedTotalSupply(); // @audit: reads `nouns` 3 times
285:
286:        temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);
287:
288:        require(
289:            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold, // @audit: 6th sload
290:            'NounsDAO::propose: proposer votes below proposal threshold'
291:        );

376:    function checkGovernanceActive() internal view {
377:        if (block.timestamp < nouns.forkingPeriodEndTimestamp()) { // @audit: 1st sload
378:            revert GovernanceBlockedDuringForkingPeriod();
379:        }
380:
381:        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) { // @audit: 2nd sload
382:            revert WaitingForTokensToClaimOrExpiration();
383:        }
384:    }

777:    function adjustedTotalSupply() public view returns (uint256) {
778:        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim(); // @audit: 3rd, 4th, and 5th sloads
779:    }
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..8ea63694 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -276,16 +277,17 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         bytes[] memory calldatas,
         string memory description
     ) public returns (uint256) {
-        checkGovernanceActive();
+        INounsTokenForkLike _nouns = nouns;
+        checkGovernanceActive(_nouns);

         ProposalTemp memory temp;

-        temp.totalSupply = adjustedTotalSupply();
+        temp.totalSupply = _adjustedTotalSupply(_nouns);

         temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

         require(
-            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
+            _nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
             'NounsDAO::propose: proposer votes below proposal threshold'
         );
         require(
@@ -373,12 +375,12 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
      * 1. All tokens are claimed
      * 2. The delayed governance expiration timestamp is reached
      */
-    function checkGovernanceActive() internal view {
-        if (block.timestamp < nouns.forkingPeriodEndTimestamp()) {
+    function checkGovernanceActive(INounsTokenForkLike _nouns) internal view {
+        if (block.timestamp < _nouns.forkingPeriodEndTimestamp()) {
             revert GovernanceBlockedDuringForkingPeriod();
         }

-        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
+        if (block.timestamp < delayedGovernanceExpirationTimestamp && _nouns.remainingTokensToClaim() > 0) {
             revert WaitingForTokensToClaimOrExpiration();
         }
     }
@@ -775,7 +777,12 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
     }

     function adjustedTotalSupply() public view returns (uint256) {
-        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim();
+        return _adjustedTotalSupply(nouns);
+    }
+
+    // @audit: for benchmarking purposes and to stay compatible with interfaces
+    function _adjustedTotalSupply(INounsTokenForkLike _nouns) internal view returns (uint256) {
+        return _nouns.totalSupply() - _nouns.balanceOf(address(timelock)) + _nouns.remainingTokensToClaim();
     }

     function erc20TokensToIncludeInQuitArray() public view returns(address[] memory) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L111-L124

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L224-L226

*Gas Savings for `NounsDAOLogicV3.executeFork`, obtained via protocol's tests: Avg 158 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  174736  |  1438654 |  336014 |    39    |
| After  |  174537  |  1438455 |  335856 |    39    |

### Pass cached variable (`forkEscrow`) into `adjustedTotalSupply` to save 1 SLOAD
```solidity
File: contracts/governance/fork/NounsDAOV3Fork.sol
111:    function executeFork(NounsDAOStorageV3.StorageV3 storage ds)
112:        external
113:        returns (address forkTreasury, address forkToken)
114:    {
115:        if (isForkPeriodActive(ds)) revert ForkPeriodActive();
116:        INounsDAOForkEscrow forkEscrow = ds.forkEscrow; // @audit: 1st sload
117:
118:        uint256 tokensInEscrow = forkEscrow.numTokensInEscrow();
119:        if (tokensInEscrow <= forkThreshold(ds)) revert ForkThresholdNotMet();
120:
121:        uint256 forkEndTimestamp = block.timestamp + ds.forkPeriod;
122:
123:        (forkTreasury, forkToken) = ds.forkDAODeployer.deployForkDAO(forkEndTimestamp, forkEscrow);
124:        sendProRataTreasury(ds, forkTreasury, tokensInEscrow, adjustedTotalSupply(ds)); // @audit: reads `ds.forkEscrow` again

224:    function adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds) internal view returns (uint256) {
225:        return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - ds.forkEscrow.numTokensOwnedByDAO(); // @audit: 2nd sload
226:    }
```
```diff
diff --git a/contracts/governance/fork/NounsDAOV3Fork.sol b/contracts/governance/fork/NounsDAOV3Fork.sol
index d87ffc70..93c6123f 100644
--- a/contracts/governance/fork/NounsDAOV3Fork.sol
+++ b/contracts/governance/fork/NounsDAOV3Fork.sol
@@ -121,7 +121,7 @@ library NounsDAOV3Fork {
         uint256 forkEndTimestamp = block.timestamp + ds.forkPeriod;

         (forkTreasury, forkToken) = ds.forkDAODeployer.deployForkDAO(forkEndTimestamp, forkEscrow);
-        sendProRataTreasury(ds, forkTreasury, tokensInEscrow, adjustedTotalSupply(ds));
+        sendProRataTreasury(ds, forkTreasury, tokensInEscrow, _adjustedTotalSupply(ds, forkEscrow));
         uint32 forkId = forkEscrow.closeEscrow();

         ds.forkDAOTreasury = forkTreasury;
@@ -224,6 +224,11 @@ library NounsDAOV3Fork {
     function adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds) internal view returns (uint256) {
         return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - ds.forkEscrow.numTokensOwnedByDAO();
     }
+
+    // @audit: to simplify benchmarking. To implement this optimization all functions (in all contracts) that call `adjustedTotalSupply` need to be refactored as well.
+    function _adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds, INounsDAOForkEscrow forkEscrow) internal view returns (uint256) {
+        return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - forkEscrow.numTokensOwnedByDAO();
+    }

     /**
      * @notice Returns true if noun holders can currently join a fork
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L141-L151

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L224-L226

*Gas Savings for `NounsDAOLogicV3.joinFork`, obtained via protocol's tests: Avg 100 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  53456   |  546057  |  188263 |    10    |
| After  |  53356   |  545858  |  188163 |    10    |

### Pass cached variable (`forkEscrow`) into `adjustedTotalSupply` to save 1 SLOAD
```solidity
File: contracts/governance/fork/NounsDAOV3Fork.sol
141:    function joinFork(
142:        NounsDAOStorageV3.StorageV3 storage ds,
143:        uint256[] calldata tokenIds,
144:        uint256[] calldata proposalIds,
145:        string calldata reason
146:    ) external {
147:        if (!isForkPeriodActive(ds)) revert ForkPeriodNotActive();
148:
149:        INounsDAOForkEscrow forkEscrow = ds.forkEscrow; // @audit: 1st sload
150:        address timelock = address(ds.timelock);
151:        sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, adjustedTotalSupply(ds)); // @audit: reads `ds.forkEscrow` again

224:    function adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds) internal view returns (uint256) {
225:        return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - ds.forkEscrow.numTokensOwnedByDAO(); // @audit: 2nd sload
226:    }
```
```diff
diff --git a/contracts/governance/fork/NounsDAOV3Fork.sol b/contracts/governance/fork/NounsDAOV3Fork.sol
index d87ffc70..93c6123f 100644
--- a/contracts/governance/fork/NounsDAOV3Fork.sol
+++ b/contracts/governance/fork/NounsDAOV3Fork.sol
@@ -148,7 +148,7 @@ library NounsDAOV3Fork {

         INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
         address timelock = address(ds.timelock);
-        sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, adjustedTotalSupply(ds));
+        sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, _adjustedTotalSupply(ds, forkEscrow));

         for (uint256 i = 0; i < tokenIds.length; i++) {
             ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]);
@@ -224,6 +224,11 @@ library NounsDAOV3Fork {
     function adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds) internal view returns (uint256) {
         return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - ds.forkEscrow.numTokensOwnedByDAO();
     }
+
+    // @audit: to simplify benchmarking. To implement this optimization all functions (in all contracts) that call `adjustedTotalSupply` need to be refactored as well.
+    function _adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds, INounsDAOForkEscrow forkEscrow) internal view returns (uint256) {
+        return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - forkEscrow.numTokensOwnedByDAO();
+    }

     /**
      * @notice Returns true if noun holders can currently join a fork
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L148-L154

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L311-L314

*Gas Savings for `NounsTokenFork.claimFromEscrow`, obtained via protocol's tests: Max 264 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  217402  |  3683287 |  387833 |    24    |
| After  |  217391  |  3683023 |  387813 |    24    |

### Cache `escrow` and pass cached value into `_mintWithOriginalSeed` to save 1 SLOAD per loop iteration
```solidity
File: contracts/governance/fork/newdao/token/NounsTokenFork.sol
148:    function claimFromEscrow(uint256[] calldata tokenIds) external {
149:        for (uint256 i = 0; i < tokenIds.length; i++) {
150:            uint256 nounId = tokenIds[i];
151:            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim(); // @audit: 1st sload for `escrow`
152:
153:            _mintWithOriginalSeed(msg.sender, nounId); // @audit: reads `escrow` again
154:        }

311:    function _mintWithOriginalSeed(address to, uint256 nounId) internal {
312:        (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(
313:            address(escrow.nounsToken()) // @audit: 2nd sload
314:        ).seeds(nounId);
```
```diff
diff --git a/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
index a1f9d6d3..8aa7d412 100644
--- a/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -147,10 +147,11 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
      */
     function claimFromEscrow(uint256[] calldata tokenIds) external {
         for (uint256 i = 0; i < tokenIds.length; i++) {
+            INounsDAOForkEscrow _escrow = escrow;
             uint256 nounId = tokenIds[i];
-            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();
+            if (_escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();

-            _mintWithOriginalSeed(msg.sender, nounId);
+            _mintWithOriginalSeed(msg.sender, nounId, _escrow);
         }

         remainingTokensToClaim -= tokenIds.length;
@@ -308,9 +310,9 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /**
      * @notice Mint a new token using the original Nouns seed.
      */
-    function _mintWithOriginalSeed(address to, uint256 nounId) internal {
+    function _mintWithOriginalSeed(address to, uint256 nounId, INounsDAOForkEscrow _escrow) internal {
         (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(
-            address(escrow.nounsToken())
+            address(_escrow.nounsToken())
         ).seeds(nounId);
         INounsSeeder.Seed memory seed = INounsSeeder.Seed(background, body, accessory, head, glasses);
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L166-L174

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L311-L314

*Gas Savings for `NounsTokenFork.claimDuringForkPeriod`, obtained via protocol's tests: Avg 205 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  354329  |  538953  |  272940 |    6     |
| After  |  354054  |  538535  |  272735 |    6     |

### Cache `escrow` and pass cached value into `_mintWithOriginalSeed` to save 1 SLOAD per loop iteration
```solidity
File: contracts/governance/fork/newdao/token/NounsTokenFork.sol
166:    function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
167:        uint256 currentNounId = _currentNounId;
168:        uint256 maxNounId = 0;
169:        if (msg.sender != escrow.dao()) revert OnlyOriginalDAO(); // @audit: 1st sload for `escrow`
170:        if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
171:
172:        for (uint256 i = 0; i < tokenIds.length; i++) {
173:            uint256 nounId = tokenIds[i];
174:            _mintWithOriginalSeed(to, nounId); // @audit: reads `escrow` again

311:    function _mintWithOriginalSeed(address to, uint256 nounId) internal {
312:        (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(
313:            address(escrow.nounsToken()) // @audit: 2nd sload
314:        ).seeds(nounId);
```
```diff
diff --git a/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
index a1f9d6d3..8aa7d412 100644
--- a/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -166,12 +167,13 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
         uint256 currentNounId = _currentNounId;
         uint256 maxNounId = 0;
-        if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
+        INounsDAOForkEscrow _escrow = escrow;
+        if (msg.sender != _escrow.dao()) revert OnlyOriginalDAO();
         if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();

         for (uint256 i = 0; i < tokenIds.length; i++) {
             uint256 nounId = tokenIds[i];
-            _mintWithOriginalSeed(to, nounId);
+            _mintWithOriginalSeed(to, nounId, _escrow);

             if (tokenIds[i] > maxNounId) maxNounId = tokenIds[i];
         }
@@ -308,9 +310,9 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /**
      * @notice Mint a new token using the original Nouns seed.
      */
-    function _mintWithOriginalSeed(address to, uint256 nounId) internal {
+    function _mintWithOriginalSeed(address to, uint256 nounId, INounsDAOForkEscrow _escrow) internal {
         (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(
-            address(escrow.nounsToken())
+            address(_escrow.nounsToken())
         ).seeds(nounId);
         INounsSeeder.Seed memory seed = INounsSeeder.Seed(background, body, accessory, head, glasses);
```

## Refactor functions to avoid redundant External Calls
We can refactor the functions in the instances below to take in cached values from external calls. This will allow us to avoid redundant external calls that would otherwise take place in the invoked functions.

Total Instances: `3`

Estimated Gas Saved: `5230`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L111-L124

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L208-L210

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L224-L226

*Gas Savings for `NounsDAOLogicV3.executeFork`, obtained via protocol's tests: Avg 3481 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  174736  |  1438654 |  336014 |    39    |
| After  |  170354  |  1434226 |  332533 |    39    |

### Cache return value from `adjustedTotalSupply` and pass cached value into `forkThreshold` to save 3 SLOADs and 3 External Calls
```solidity
File: contracts/governance/fork/NounsDAOV3Fork.sol
111:    function executeFork(NounsDAOStorageV3.StorageV3 storage ds)
112:        external
113:        returns (address forkTreasury, address forkToken)
114:    {
115:        if (isForkPeriodActive(ds)) revert ForkPeriodActive();
116:        INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
117:
118:        uint256 tokensInEscrow = forkEscrow.numTokensInEscrow();
119:        if (tokensInEscrow <= forkThreshold(ds)) revert ForkThresholdNotMet(); // @audit: invokes `adjustedTotalSupply`, which performs 3 SLOADs and 3 External Calls
120:
121:        uint256 forkEndTimestamp = block.timestamp + ds.forkPeriod;
122:
123:        (forkTreasury, forkToken) = ds.forkDAODeployer.deployForkDAO(forkEndTimestamp, forkEscrow);
124:        sendProRataTreasury(ds, forkTreasury, tokensInEscrow, adjustedTotalSupply(ds)); // @audit: 3 more SLOADs and External Calls are performed

208:    function forkThreshold(NounsDAOStorageV3.StorageV3 storage ds) public view returns (uint256) {
209:        return (adjustedTotalSupply(ds) * ds.forkThresholdBPS) / 10_000; // @audit: `adjustedTotalSupply` performs SLOADs + External Calls
210:    }

224:    function adjustedTotalSupply(NounsDAOStorageV3.StorageV3 storage ds) internal view returns (uint256) {
225:        return ds.nouns.totalSupply() - ds.nouns.balanceOf(address(ds.timelock)) - ds.forkEscrow.numTokensOwnedByDAO(); // @audit: 3 SLOADs + 3 External Calls
226:    }
```
```diff
diff --git a/contracts/governance/fork/NounsDAOV3Fork.sol b/contracts/governance/fork/NounsDAOV3Fork.sol
index d87ffc70..f25a76b0 100644
--- a/contracts/governance/fork/NounsDAOV3Fork.sol
+++ b/contracts/governance/fork/NounsDAOV3Fork.sol
@@ -116,12 +116,13 @@ library NounsDAOV3Fork {
         INounsDAOForkEscrow forkEscrow = ds.forkEscrow;

         uint256 tokensInEscrow = forkEscrow.numTokensInEscrow();
-        if (tokensInEscrow <= forkThreshold(ds)) revert ForkThresholdNotMet();
+        uint256 _adjustedTotalSupply = adjustedTotalSupply(ds);
+        if (tokensInEscrow <= _forkThreshold(ds, _adjustedTotalSupply)) revert ForkThresholdNotMet();

         uint256 forkEndTimestamp = block.timestamp + ds.forkPeriod;

         (forkTreasury, forkToken) = ds.forkDAODeployer.deployForkDAO(forkEndTimestamp, forkEscrow);
-        sendProRataTreasury(ds, forkTreasury, tokensInEscrow, adjustedTotalSupply(ds));
+        sendProRataTreasury(ds, forkTreasury, tokensInEscrow, _adjustedTotalSupply);
         uint32 forkId = forkEscrow.closeEscrow();

         ds.forkDAOTreasury = forkTreasury;
@@ -206,7 +207,11 @@ library NounsDAOV3Fork {
      * @notice Returns the required number of tokens to escrow to trigger a fork
      */
     function forkThreshold(NounsDAOStorageV3.StorageV3 storage ds) public view returns (uint256) {
-        return (adjustedTotalSupply(ds) * ds.forkThresholdBPS) / 10_000;
+        return _forkThreshold(ds, adjustedTotalSupply(ds));
+    }
+
+    function _forkThreshold(NounsDAOStorageV3.StorageV3 storage ds, uint256 _adjustedTotalSupply) internal view returns (uint256) {
+        return (_adjustedTotalSupply * ds.forkThresholdBPS) / 10_000;
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L148-L154

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L311-L314

*Gas Savings for `NounsTokenFork.claimFromEscrow`, obtained via protocol's tests: Avg 1012 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  217402  |  3683287 |  387833 |    24    |
| After  |  217399  |  3664085 |  386821 |    24    |

### Cache `escrow.nounsToken()` call outside the loop and pass it into `_mintWithOriginalSeed` to save 1 External Call per iteration
```solidity
File: contracts/governance/fork/newdao/token/NounsTokenFork.sol
148:    function claimFromEscrow(uint256[] calldata tokenIds) external {
149:        for (uint256 i = 0; i < tokenIds.length; i++) {
150:            uint256 nounId = tokenIds[i];
151:            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim(); 
152:
153:            _mintWithOriginalSeed(msg.sender, nounId); // @audit: performs `escrow.nounsToken` call every iteration
154:        }

311:    function _mintWithOriginalSeed(address to, uint256 nounId) internal {
312:        (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(
313:            address(escrow.nounsToken()) // @audit: external call
314:        ).seeds(nounId);
```
```diff
diff --git a/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
index a1f9d6d3..2446f75c 100644
--- a/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -146,11 +146,12 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
      * @param tokenIds The token IDs to claim
      */
     function claimFromEscrow(uint256[] calldata tokenIds) external {
+        address _nounsToken = address(escrow.nounsToken());
         for (uint256 i = 0; i < tokenIds.length; i++) {
             uint256 nounId = tokenIds[i];
             if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();

-            _mintWithOriginalSeed(msg.sender, nounId);
+            _mintWithOriginalSeed(msg.sender, nounId, _nounsToken);
         }

         remainingTokensToClaim -= tokenIds.length;
@@ -308,10 +310,8 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /**
      * @notice Mint a new token using the original Nouns seed.
      */
-    function _mintWithOriginalSeed(address to, uint256 nounId) internal {
-        (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(
-            address(escrow.nounsToken())
-        ).seeds(nounId);
+    function _mintWithOriginalSeed(address to, uint256 nounId, address _nounsToken) internal {
+        (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(_nounsToken).seeds(nounId);
         INounsSeeder.Seed memory seed = INounsSeeder.Seed(background, body, accessory, head, glasses);

         seeds[nounId] = seed;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L166-L174

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L311-L314

*Gas Savings for `NounsTokenFork.claimDuringForkPeriod`, obtained via protocol's tests: Avg 737 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  354329  |  538953  |  272940 |    6     |
| After  |  353493  |  537042  |  272203 |    6     |

### Cache `escrow.nounsToken()` call outside the loop and pass it into `_mintWithOriginalSeed` to save 1 External Call per iteration
```solidity
File: contracts/governance/fork/newdao/token/NounsTokenFork.sol
166:    function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
167:        uint256 currentNounId = _currentNounId;
168:        uint256 maxNounId = 0;
169:        if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
170:        if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
171:
172:        for (uint256 i = 0; i < tokenIds.length; i++) {
173:            uint256 nounId = tokenIds[i];
174:            _mintWithOriginalSeed(to, nounId); // @audit: performs `escrow.nounsToken` call every iteration

311:    function _mintWithOriginalSeed(address to, uint256 nounId) internal {
312:        (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(
313:            address(escrow.nounsToken()) // @audit: external call
314:        ).seeds(nounId);
```
```diff
diff --git a/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
index a1f9d6d3..2446f75c 100644
--- a/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -168,10 +169,11 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
         uint256 maxNounId = 0;
         if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
         if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
-
+
+        address _nounsToken = address(escrow.nounsToken());
         for (uint256 i = 0; i < tokenIds.length; i++) {
             uint256 nounId = tokenIds[i];
-            _mintWithOriginalSeed(to, nounId);
+            _mintWithOriginalSeed(to, nounId, _nounsToken);

             if (tokenIds[i] > maxNounId) maxNounId = tokenIds[i];
         }
@@ -308,10 +310,8 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /**
      * @notice Mint a new token using the original Nouns seed.
      */
-    function _mintWithOriginalSeed(address to, uint256 nounId) internal {
-        (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(
-            address(escrow.nounsToken())
-        ).seeds(nounId);
+    function _mintWithOriginalSeed(address to, uint256 nounId, address _nounsToken) internal {
+        (uint48 background, uint48 body, uint48 accessory, uint48 head, uint48 glasses) = NounsTokenFork(_nounsToken).seeds(nounId);
         INounsSeeder.Seed memory seed = INounsSeeder.Seed(background, body, accessory, head, glasses);

         seeds[nounId] = seed;
```

## Use calldata instead of memory for function parameters
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

Total Instances: `12`

Estimated Gas Saved: `12361`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L197-L203

*Gas Savings for `NounsDAOLogicV2.propose`, obtained via protocol's tests: Avg 830 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  380456  |  947962  |  439476 |    57    |
| After  |  380000  |  947479  |  438646 |    57    |

*Note: We are able to change the data location for the `description` paremeter without causing a `stack too deep` error. Changing the locations for other function parameters resulted in a `stack too deep` error.*

```solidity
File: contracts/governance/NounsDAOLogicV2.sol
197:    function propose(
198:        address[] memory targets,
199:        uint256[] memory values,
200:        string[] memory signatures,
201:        bytes[] memory calldatas,
202:        string memory description // @audit: can change to `calldata`
203:    ) public returns (uint256) {
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV2.sol b/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..6d2e89e8 100644
--- a/contracts/governance/NounsDAOLogicV2.sol
+++ b/contracts/governance/NounsDAOLogicV2.sol
@@ -199,7 +199,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256[] memory values,
         string[] memory signatures,
         bytes[] memory calldatas,
-        string memory description
+        string calldata description
     ) public returns (uint256) {
         ProposalTemp memory temp;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L165-L171

*Gas Savings for `NounsDAOExecutorV2.executeTransaction`, obtained via protocol's tests: Avg 412 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  29527   |  339600  |  53364  |    19    |
| After  |  28893   |  339006  |  52952  |    19    |

```solidity
File: contracts/governance/NounsDAOExecutorV2.sol
165:    function executeTransaction(
166:        address target,
167:        uint256 value,
168:        string memory signature,
169:        bytes memory data,
170:        uint256 eta
171:    ) public returns (bytes memory) {
```
```diff
diff --git a/contracts/governance/NounsDAOExecutorV2.sol b/contracts/governance/NounsDAOExecutorV2.sol
index f4f85883..408d2dcf 100644
--- a/contracts/governance/NounsDAOExecutorV2.sol
+++ b/contracts/governance/NounsDAOExecutorV2.sol
@@ -165,8 +165,8 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function executeTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
     ) public returns (bytes memory) {
         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L150-L156

*Gas Savings for `NounsDAOExecutorV2.cancelTransaction`, obtained via protocol's tests: Avg 462 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  10017   |  10426   |  10068  |    34    |
| After  |  9552    |  9882    |  9606   |    34    |

```solidity
File: contracts/governance/NounsDAOExecutorV2.sol
150:    function cancelTransaction(
151:        address target,
152:        uint256 value,
153:        string memory signature,
154:        bytes memory data,
155:        uint256 eta
156:    ) public {
```
```diff
diff --git a/contracts/governance/NounsDAOExecutorV2.sol b/contracts/governance/NounsDAOExecutorV2.sol
index f4f85883..130eeb58 100644
--- a/contracts/governance/NounsDAOExecutorV2.sol
+++ b/contracts/governance/NounsDAOExecutorV2.sol
@@ -150,8 +150,8 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function cancelTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
     ) public {
         require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L130-L136

*Gas Savings for `NounsDAOExecutorV2.queueTransaction`, obtained via protocol's tests: Avg 520 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  26117   |  29751   |  27113  |    29    |
| After  |  25652   |  28749   |  26593  |    29    |

```solidity
File: contracts/governance/NounsDAOExecutorV2.sol
130:    function queueTransaction(
131:        address target,
132:        uint256 value,
133:        string memory signature,
134:        bytes memory data,
135:        uint256 eta
136:    ) public returns (bytes32) {
```
```diff
diff --git a/contracts/governance/NounsDAOExecutorV2.sol b/contracts/governance/NounsDAOExecutorV2.sol
index f4f85883..5a6ed111 100644
--- a/contracts/governance/NounsDAOExecutorV2.sol
+++ b/contracts/governance/NounsDAOExecutorV2.sol
@@ -130,8 +130,8 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function queueTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
     ) public returns (bytes32) {
         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L288-L297

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L278-L286

*Gas Savings for `NounsDAOLogicV3.updateProposal`, obtained via protocol's tests: Avg 1395 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  14500   |  92416   |  20773  |    17    |
| After  |  13224   |  90743   |  19378  |    17    |

```solidity
File: contracts/governance/NounsDAOV3Proposals.sol
288:    function updateProposal(
289:        NounsDAOStorageV3.StorageV3 storage ds,
290:        uint256 proposalId,
291:        address[] memory targets,
292:        uint256[] memory values,
293:        string[] memory signatures,
294:        bytes[] memory calldatas,
295:        string memory description,
296:        string memory updateMessage
297:    ) external {
```
```solidity
File: contracts/governance/NounsDAOLogicV3.sol
278:    function updateProposal(
279:        uint256 proposalId,
280:        address[] memory targets,
281:        uint256[] memory values,
282:        string[] memory signatures,
283:        bytes[] memory calldatas,
284:        string memory description,
285:        string memory updateMessage
286:    ) external {
```
```diff
diff --git a/contracts/governance/NounsDAOV3Proposals.sol b/contracts/governance/NounsDAOV3Proposals.sol
index 2685dc20..558ffb1d 100644
--- a/contracts/governance/NounsDAOV3Proposals.sol
+++ b/contracts/governance/NounsDAOV3Proposals.sol
@@ -291,9 +291,9 @@ library NounsDAOV3Proposals {
         address[] memory targets,
         uint256[] memory values,
         string[] memory signatures,
-        bytes[] memory calldatas,
-        string memory description,
-        string memory updateMessage
+        bytes[] calldata calldatas,
+        string calldata description,
+        string calldata updateMessage
     ) external {
         updateProposalTransactionsInternal(ds, proposalId, targets, values, signatures, calldatas);
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV3.sol b/contracts/governance/NounsDAOLogicV3.sol
index 00c5ccdc..967bc4cb 100644
--- a/contracts/governance/NounsDAOLogicV3.sol
+++ b/contracts/governance/NounsDAOLogicV3.sol
@@ -280,9 +280,9 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
         address[] memory targets,
         uint256[] memory values,
         string[] memory signatures,
-        bytes[] memory calldatas,
-        string memory description,
-        string memory updateMessage
+        bytes[] calldata calldatas,
+        string calldata description,
+        string calldata updateMessage
     ) external {
         ds.updateProposal(proposalId, targets, values, signatures, calldatas, description, updateMessage);
     }
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L321-L329

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L313-L320

*Gas Savings for `NounsDAOLogicV3.updateProposalTransactions`, obtained via protocol's tests: Avg 2514 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  13349   |  90201   |  19643  |    17    |
| After  |  11366   |  87843   |  17129  |    17    |

```solidity
File: contracts/governance/NounsDAOV3Proposals.sol
321:    function updateProposalTransactions(
322:        NounsDAOStorageV3.StorageV3 storage ds,
323:        uint256 proposalId,
324:        address[] memory targets,
325:        uint256[] memory values,
326:        string[] memory signatures,
327:        bytes[] memory calldatas,
328:        string memory updateMessage
329:    ) external {
```
```solidity
File: contracts/governance/NounsDAOLogicV3.sol
313:    function updateProposalTransactions(
314:        uint256 proposalId,
315:        address[] memory targets,
316:        uint256[] memory values,
317:        string[] memory signatures,
318:        bytes[] memory calldatas,
319:        string memory updateMessage
320:    ) external {
```
```diff
diff --git a/contracts/governance/NounsDAOV3Proposals.sol b/contracts/governance/NounsDAOV3Proposals.sol
index 2685dc20..bab5770e 100644
--- a/contracts/governance/NounsDAOV3Proposals.sol
+++ b/contracts/governance/NounsDAOV3Proposals.sol
@@ -321,11 +321,11 @@ library NounsDAOV3Proposals {
     function updateProposalTransactions(
         NounsDAOStorageV3.StorageV3 storage ds,
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
         updateProposalTransactionsInternal(ds, proposalId, targets, values, signatures, calldatas);
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV3.sol b/contracts/governance/NounsDAOLogicV3.sol
index 00c5ccdc..fffe97e9 100644
--- a/contracts/governance/NounsDAOLogicV3.sol
+++ b/contracts/governance/NounsDAOLogicV3.sol
@@ -312,11 +312,11 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      */
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

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L383-L390

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L338-L347

*Gas Savings for `NounsDAOLogicV3.updateProposalBySigs`, obtained via protocol's tests: Avg 3826 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  33310   |  122687  |  38005  |    33    |
| After  |  30398   |  120176  |  34179  |    33    |

```solidity
File: contracts/governance/NounsDAOV3Proposals.sol
383:    function updateProposalBySigs(
384:        NounsDAOStorageV3.StorageV3 storage ds,
385:        uint256 proposalId,
386:        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures,
387:        ProposalTxs memory txs,
388:        string memory description,
389:        string memory updateMessage
390:    ) external {
```
```solidity
File: contracts/governance/NounsDAOLogicV3.sol
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
```
```diff
diff --git a/contracts/governance/NounsDAOV3Proposals.sol b/contracts/governance/NounsDAOV3Proposals.sol
index 2685dc20..2d0965e3 100644
--- a/contracts/governance/NounsDAOV3Proposals.sol
+++ b/contracts/governance/NounsDAOV3Proposals.sol
@@ -383,10 +383,10 @@ library NounsDAOV3Proposals {
     function updateProposalBySigs(
         NounsDAOStorageV3.StorageV3 storage ds,
         uint256 proposalId,
-        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures,
+        NounsDAOStorageV3.ProposerSignature[] calldata proposerSignatures,
         ProposalTxs memory txs,
-        string memory description,
-        string memory updateMessage
+        string calldata description,
+        string calldata updateMessage
     ) external {
         checkProposalTxs(txs);
         // without this check it's possible to run through this function and update a proposal without signatures
```
```diff
diff --git a/contracts/governance/NounsDAOLogicV3.sol b/contracts/governance/NounsDAOLogicV3.sol
index 00c5ccdc..62ce6217 100644
--- a/contracts/governance/NounsDAOLogicV3.sol
+++ b/contracts/governance/NounsDAOLogicV3.sol
@@ -337,13 +337,13 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      */
     function updateProposalBySigs(
         uint256 proposalId,
-        ProposerSignature[] memory proposerSignatures,
+        ProposerSignature[] calldata proposerSignatures,
         address[] memory targets,
         uint256[] memory values,
         string[] memory signatures,
         bytes[] memory calldatas,
-        string memory description,
-        string memory updateMessage
+        string calldata description,
+        string calldata updateMessage
     ) external {
         ds.updateProposalBySigs(
             proposalId,
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L107-L113

*Gas Savings for `NounsDAOExecutor.queueTransaction`, obtained via protocol's tests: Avg 786 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  28107   |  29783   |  28172  |    66    |
| After  |  27691   |  28240   |  27386  |    66    |

```solidity
File: contracts/governance/NounsDAOExecutor.sol
107:    function queueTransaction(
108:        address target,
109:        uint256 value,
110:        string memory signature,
111:        bytes memory data,
112:        uint256 eta
113:    ) public returns (bytes32) {
```
```diff
diff --git a/contracts/governance/NounsDAOExecutor.sol b/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..86585ff5 100644
--- a/contracts/governance/NounsDAOExecutor.sol
+++ b/contracts/governance/NounsDAOExecutor.sol
@@ -107,8 +107,8 @@ contract NounsDAOExecutor {
     function queueTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
     ) public returns (bytes32) {
         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L127-L133

*Gas Savings for `NounsDAOExecutor.cancelTransaction`, obtained via protocol's tests: Avg 404 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  9985    |  9985    |  8671   |    28    |
| After  |  9569    |  9569    |  8267   |    28    |

```solidity
File: contracts/governance/NounsDAOExecutor.sol
127:    function cancelTransaction(
128:        address target,
129:        uint256 value,
130:        string memory signature,
131:        bytes memory data,
132:        uint256 eta
133:    ) public {
```
```diff
diff --git a/contracts/governance/NounsDAOExecutor.sol b/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..b6854192 100644
--- a/contracts/governance/NounsDAOExecutor.sol
+++ b/contracts/governance/NounsDAOExecutor.sol1
@@ -127,8 +127,8 @@ contract NounsDAOExecutor {
     function cancelTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
     ) public {
         require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L142-L148

*Gas Savings for `NounsDAOExecutor.executeTransaction`, obtained via protocol's tests: Avg 785 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  30881   |  142555  |  42009  |    59    |
| After  |  30668   |  140491  |  41224  |    59    |

```solidity
File: contracts/governance/NounsDAOExecutor.sol
142:    function executeTransaction(
143:        address target,
144:        uint256 value,
145:        string memory signature,
146:        bytes memory data,
147:        uint256 eta
148:    ) public returns (bytes memory) {
```
```diff
diff --git a/contracts/governance/NounsDAOExecutor.sol b/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..c55906c1 100644
--- a/contracts/governance/NounsDAOExecutor.sol
+++ b/contracts/governance/NounsDAOExecutor.sol
@@ -142,8 +142,8 @@ contract NounsDAOExecutor {
     function executeTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
     ) public returns (bytes memory) {
         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206

*Gas Savings for `NounsDAOLogicV1Fork.quit(uint256[],address[])`, obtained via protocol's tests: Avg 105 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  151098  |  347452  |  165989 |    4     |
| After  |  151011  |  347390  |  165884 |    4     |

```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
206:    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..14006186 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -203,7 +203,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         quitInternal(tokenIds, erc20TokensToIncludeInQuit);
     }

-    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
+    function quit(uint256[] calldata tokenIds, address[] calldata erc20TokensToInclude) external nonReentrant {
         // check that erc20TokensToInclude is a subset of `erc20TokensToIncludeInQuit`
         address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L272-L278

*Gas Savings for `NounsDAOLogicV1Fork.propose`, obtained via protocol's tests: Avg 322 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  367050  |  417023  |  295642 |    34    |
| After  |  366692  |  416665  |  295320 |    34    |

```solidity
File: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
272:    function propose(
273:        address[] memory targets,
274:        uint256[] memory values,
275:        string[] memory signatures,
276:        bytes[] memory calldatas,
277:        string memory description
278:    ) public returns (uint256) {
```
```diff
diff --git a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..1abe01bb 100644
--- a/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -274,7 +274,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         uint256[] memory values,
         string[] memory signatures,
         bytes[] memory calldatas,
-        string memory description
+        string calldata description
     ) public returns (uint256) {
         checkGovernanceActive();
```