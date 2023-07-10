# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use `selfbalance()` instead of `address(this).balance` | 2 |
| [GAS-2](#GAS-2) | Use assembly to check for `address(0)` | 14 |
| [GAS-3](#GAS-3) | Using bools for storage incurs overhead | 7 |
| [GAS-4](#GAS-4) | Cache array length outside of loop | 36 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 18 |
| [GAS-6](#GAS-6) | Use Custom Errors | 4 |
| [GAS-7](#GAS-7) | Don't initialize variables with default value | 42 |
| [GAS-8](#GAS-8) | Long revert strings | 95 |
| [GAS-9](#GAS-9) | Functions guaranteed to revert when called by normal users can be marked `payable` | 30 |
| [GAS-10](#GAS-10) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 38 |
| [GAS-11](#GAS-11) | Using `private` rather than `public` for constants, saves gas | 79 |
| [GAS-12](#GAS-12) | Use shift Right/Left instead of division/multiplication if possible | 3 |
| [GAS-13](#GAS-13) | Splitting require() statements that use && saves gas | 3 |
| [GAS-14](#GAS-14) | Use != 0 instead of > 0 for unsigned integer comparison | 16 |
### <a name="GAS-1"></a>[GAS-1] Use `selfbalance()` instead of `address(this).balance`
Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas.
Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

*Saves 15 gas when checking internal balance, 6 for external*

*Instances (2)*:
```solidity
File: NounsDAOLogicV2.sol

838:         require(msg.sender == admin, 'NounsDAO::_setPendingAdmin: admin only');

1048:     function min(uint256 a, uint256 b) internal pure returns (uint256) {

```

### <a name="GAS-2"></a>[GAS-2] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (14)*:
```solidity
File: NounsDAOLogicV1.sol

131:             votingDelay_ >= MIN_VOTING_DELAY && votingDelay_ <= MAX_VOTING_DELAY,

135:             proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,

136:             'NounsDAO::initialize: invalid proposal threshold'

376:                 proposal.calldatas[i],

498:         uint256 proposalId,

637:     function _setVetoer(address newVetoer) public {

```

```solidity
File: NounsDAOLogicV2.sol

155:             votingDelay_ >= MIN_VOTING_DELAY && votingDelay_ <= MAX_VOTING_DELAY,

159:             proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,

160:             'NounsDAO::initialize: invalid proposal threshold bps'

410:                 proposal.calldatas[i],

613:         uint256 proposalId,

873:      * @notice Begins transition of vetoer. The newPendingVetoer must call _acceptVetoer to finalize the transfer.

```

```solidity
File: NounsDAOProxy.sol

94:     function delegateTo(address callee, bytes memory data) internal {

```

```solidity
File: NounsDAOProxyV2.sol

97:     function delegateTo(address callee, bytes memory data) internal {

```

### <a name="GAS-3"></a>[GAS-3] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (7)*:
```solidity
File: NounsDAOExecutor.sol

70:     mapping(bytes32 => bool) public queuedTransactions;

```

```solidity
File: NounsDAOExecutorV2.sol

93:     mapping(bytes32 => bool) public queuedTransactions;

```

```solidity
File: NounsDAOInterfaces.sol

688:         mapping(address => mapping(bytes32 => bool)) cancelledSigs;

```

```solidity
File: data/NounsDAOData.sol

103:     mapping(address => mapping(bytes32 => bool)) public propCandidates;

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

70:     bool public isMinterLocked;

73:     bool public isDescriptorLocked;

76:     bool public isSeederLocked;

```

### <a name="GAS-4"></a>[GAS-4] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (36)*:
```solidity
File: NounsDAOLogicV1.sol

281:         for (uint256 i = 0; i < proposal.targets.length; i++) {

319:         for (uint256 i = 0; i < proposal.targets.length; i++) {

346:         for (uint256 i = 0; i < proposal.targets.length; i++) {

371:         for (uint256 i = 0; i < proposal.targets.length; i++) {

```

```solidity
File: NounsDAOLogicV2.sol

305:         for (uint256 i = 0; i < proposal.targets.length; i++) {

343:         for (uint256 i = 0; i < proposal.targets.length; i++) {

372:         for (uint256 i = 0; i < proposal.targets.length; i++) {

405:         for (uint256 i = 0; i < proposal.targets.length; i++) {

```

```solidity
File: NounsDAOV3Admin.sol

602:         for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

603:             for (uint256 j = i + 1; j < erc20tokens.length; j++) {

```

```solidity
File: NounsDAOV3Proposals.sol

409:         for (uint256 i = 0; i < proposerSignatures.length; ++i) {

446:         for (uint256 i = 0; i < proposal.targets.length; i++) {

508:         for (uint256 i = 0; i < proposal.targets.length; i++) {

553:         for (uint256 i = 0; i < proposal.targets.length; i++) {

590:         for (uint256 i = 0; i < signers.length; ++i) {

602:         for (uint256 i = 0; i < proposal.targets.length; i++) {

826:         for (uint256 i = 0; i < proposerSignatures.length; ++i) {

860:         for (uint256 i = 0; i < txs.signatures.length; ++i) {

865:         for (uint256 i = 0; i < txs.calldatas.length; ++i) {

```

```solidity
File: fork/NounsDAOForkEscrow.sol

117:         for (uint256 i = 0; i < tokenIds.length; i++) {

148:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: fork/NounsDAOV3Fork.sol

83:         for (uint256 i = 0; i < tokenIds.length; i++) {

151:         sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, adjustedTotalSupply(ds));

153:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

209:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

223:         for (uint256 i = 0; i < tokenIds.length; i++) {

231:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

238:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

248:         for (uint256 i = 0; i < addresses.length; i++) {

397:         for (uint256 i = 0; i < proposal.targets.length; i++) {

435:         for (uint256 i = 0; i < proposal.targets.length; i++) {

462:         for (uint256 i = 0; i < proposal.targets.length; i++) {

796:         for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

797:             for (uint256 j = i + 1; j < erc20tokens.length; j++) {

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

149:         for (uint256 i = 0; i < tokenIds.length; i++) {

172:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (18)*:
```solidity
File: NounsDAOExecutor.sol

123:         emit QueueTransaction(txHash, target, value, signature, data, eta);

125:     }

148:     ) public returns (bytes memory) {

149:         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');

157:         require(

158:             getBlockTimestamp() <= eta + GRACE_PERIOD,

```

```solidity
File: NounsDAOLogicV1.sol

192:             targets.length == values.length &&

192:             targets.length == values.length &&

193:                 targets.length == signatures.length &&

194:                 targets.length == calldatas.length,

194:                 targets.length == calldatas.length,

```

```solidity
File: NounsDAOLogicV2.sol

215:             targets.length == values.length &&

215:             targets.length == values.length &&

216:                 targets.length == signatures.length &&

217:                 targets.length == calldatas.length,

217:                 targets.length == calldatas.length,

976:      * @dev The checkpoints array must not be empty, and the block number must be higher than or equal to

```

```solidity
File: NounsDAOProxyV2.sol

72:         _setImplementation(implementation_);

```

### <a name="GAS-6"></a>[GAS-6] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (4)*:
```solidity
File: NounsDAOExecutor.sol

152:         require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");

```

```solidity
File: NounsDAOExecutorV2.sol

175:         require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");

```

```solidity
File: fork/newdao/NounsAuctionHouseFork.sol

239:         require(_auction.startTime != 0, "Auction hasn't begun");

241:         require(block.timestamp >= _auction.endTime, "Auction hasn't completed");

```

### <a name="GAS-7"></a>[GAS-7] Don't initialize variables with default value

*Instances (42)*:
```solidity
File: NounsDAOLogicV1.sol

281:         for (uint256 i = 0; i < proposal.targets.length; i++) {

319:         for (uint256 i = 0; i < proposal.targets.length; i++) {

346:         for (uint256 i = 0; i < proposal.targets.length; i++) {

371:         for (uint256 i = 0; i < proposal.targets.length; i++) {

```

```solidity
File: NounsDAOLogicV2.sol

305:         for (uint256 i = 0; i < proposal.targets.length; i++) {

343:         for (uint256 i = 0; i < proposal.targets.length; i++) {

372:         for (uint256 i = 0; i < proposal.targets.length; i++) {

405:         for (uint256 i = 0; i < proposal.targets.length; i++) {

1007:         uint256 lower = 0;

```

```solidity
File: NounsDAOV3Admin.sol

602:         for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

```

```solidity
File: NounsDAOV3DynamicQuorum.sol

107:         uint256 lower = 0;

```

```solidity
File: NounsDAOV3Proposals.sol

409:         for (uint256 i = 0; i < proposerSignatures.length; ++i) {

446:         for (uint256 i = 0; i < proposal.targets.length; i++) {

508:         for (uint256 i = 0; i < proposal.targets.length; i++) {

553:         for (uint256 i = 0; i < proposal.targets.length; i++) {

590:         for (uint256 i = 0; i < signers.length; ++i) {

602:         for (uint256 i = 0; i < proposal.targets.length; i++) {

825:         uint256 numSigners = 0;

826:         for (uint256 i = 0; i < proposerSignatures.length; ++i) {

860:         for (uint256 i = 0; i < txs.signatures.length; ++i) {

865:         for (uint256 i = 0; i < txs.calldatas.length; ++i) {

```

```solidity
File: NounsDAOV3Votes.sol

224:         bool isForVoteInLastMinuteWindow = false;

229:         bool isDefeatedBefore = false;

```

```solidity
File: fork/NounsDAOForkEscrow.sol

117:         for (uint256 i = 0; i < tokenIds.length; i++) {

148:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: fork/NounsDAOV3Fork.sol

83:         for (uint256 i = 0; i < tokenIds.length; i++) {

153:         for (uint256 i = 0; i < tokenIds.length; i++) {

252:         for (uint256 i = 0; i < erc20Count; ++i) {

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

209:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

223:         for (uint256 i = 0; i < tokenIds.length; i++) {

231:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

238:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

248:         for (uint256 i = 0; i < addresses.length; i++) {

397:         for (uint256 i = 0; i < proposal.targets.length; i++) {

435:         for (uint256 i = 0; i < proposal.targets.length; i++) {

462:         for (uint256 i = 0; i < proposal.targets.length; i++) {

796:         for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

149:         for (uint256 i = 0; i < tokenIds.length; i++) {

168:         uint256 maxNounId = 0;

172:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

52:     uint8 public constant decimals = 0;

193:         uint32 lower = 0;

```

### <a name="GAS-8"></a>[GAS-8] Long revert strings

*Instances (95)*:
```solidity
File: NounsDAOExecutor.sol

73:         require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');

74:         require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');

81:         require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');

82:         require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');

83:         require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');

90:         require(msg.sender == pendingAdmin, 'NounsDAOExecutor::acceptAdmin: Call must come from pendingAdmin.');

114:         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');

134:         require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');

149:         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');

152:         require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");

174:         require(success, 'NounsDAOExecutor::executeTransaction: Transaction execution reverted.');

```

```solidity
File: NounsDAOExecutorV2.sol

96:         require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');

97:         require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');

104:         require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');

105:         require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');

106:         require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');

113:         require(msg.sender == pendingAdmin, 'NounsDAOExecutor::acceptAdmin: Call must come from pendingAdmin.');

137:         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');

157:         require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');

172:         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');

175:         require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");

197:         require(success, 'NounsDAOExecutor::executeTransaction: Transaction execution reverted.');

214:         require(msg.sender == admin, 'NounsDAOExecutor::sendETH: Call must come from admin.');

226:         require(msg.sender == admin, 'NounsDAOExecutor::sendERC20: Call must come from admin.');

```

```solidity
File: NounsDAOLogicV1.sol

122:         require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');

124:         require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');

125:         require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');

197:         require(targets.length != 0, 'NounsDAO::propose: must provide actions');

198:         require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');

336:         require(state(proposalId) != ProposalState.Executed, 'NounsDAO::cancel: cannot cancel executed proposal');

364:         require(vetoer != address(0), 'NounsDAO::veto: veto power burned');

366:         require(state(proposalId) != ProposalState.Executed, 'NounsDAO::veto: cannot veto executed proposal');

422:         require(proposalCount >= proposalId, 'NounsDAO::state: invalid proposal id');

485:         require(signatory != address(0), 'NounsDAO::castVoteBySig: invalid signature');

501:         require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');

502:         require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');

505:         require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');

530:         require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');

546:         require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');

563:         require(msg.sender == admin, 'NounsDAO::_setProposalThresholdBPS: admin only');

581:         require(msg.sender == admin, 'NounsDAO::_setQuorumVotesBPS: admin only');

599:         require(msg.sender == admin, 'NounsDAO::_setPendingAdmin: admin only');

617:         require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

638:         require(msg.sender == vetoer, 'NounsDAO::_setVetoer: vetoer only');

651:         require(msg.sender == vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

```

```solidity
File: NounsDAOLogicV2.sol

144:         require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');

148:         require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');

149:         require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');

220:         require(targets.length != 0, 'NounsDAO::propose: must provide actions');

221:         require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');

456:         require(proposalCount >= proposalId, 'NounsDAO::state: invalid proposal id');

600:         require(signatory != address(0), 'NounsDAO::castVoteBySig: invalid signature');

616:         require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');

617:         require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');

620:         require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');

838:         require(msg.sender == admin, 'NounsDAO::_setPendingAdmin: admin only');

856:         require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

906:         require(msg.sender == vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

```

```solidity
File: NounsDAOProxy.sol

79:         require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');

80:         require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');

```

```solidity
File: NounsDAOProxyV2.sol

82:         require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');

83:         require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');

```

```solidity
File: NounsDAOProxyV3.sol

82:         require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');

83:         require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');

```

```solidity
File: NounsDAOV3Admin.sol

334:         require(msg.sender == ds.vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

```

```solidity
File: NounsDAOV3Proposals.sol

641:         require(ds.proposalCount >= proposalId, 'NounsDAO::state: invalid proposal id');

```

```solidity
File: NounsDAOV3Votes.sol

170:         require(signatory != address(0), 'NounsDAO::castVoteBySig: invalid signature');

216:         require(support <= 2, 'NounsDAO::castVoteDuringVotingPeriodInternal: invalid vote type');

219:         require(receipt.hasVoted == false, 'NounsDAO::castVoteDuringVotingPeriodInternal: voter already voted');

282:         require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

176:         require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');

177:         require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');

178:         require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');

297:         require(targets.length != 0, 'NounsDAO::propose: must provide actions');

298:         require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');

452:         require(state(proposalId) != ProposalState.Executed, 'NounsDAO::cancel: cannot cancel executed proposal');

513:         require(proposalCount >= proposalId, 'NounsDAO::state: invalid proposal id');

601:         require(signatory != address(0), 'NounsDAO::castVoteBySig: invalid signature');

617:         require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');

618:         require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');

621:         require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');

646:         require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');

662:         require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');

679:         require(msg.sender == admin, 'NounsDAO::_setProposalThresholdBPS: admin only');

697:         require(msg.sender == admin, 'NounsDAO::_setQuorumVotesBPS: admin only');

715:         require(msg.sender == admin, 'NounsDAO::_setPendingAdmin: admin only');

733:         require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

790:         require(msg.sender == admin, 'NounsDAO::_authorizeUpgrade: admin only');

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

223:         require(_exists(tokenId), 'NounsToken: URI query for nonexistent token');

232:         require(_exists(tokenId), 'NounsToken: URI query for nonexistent token');

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

145:         require(delegatee != address(0), 'ERC721Checkpointable::delegateBySig: delegatee cannot be zero address');

152:         require(signatory != address(0), 'ERC721Checkpointable::delegateBySig: invalid signature');

153:         require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');

154:         require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');

176:         require(blockNumber < block.number, 'ERC721Checkpointable::getPriorVotes: not yet determined');

```

### <a name="GAS-9"></a>[GAS-9] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (30)*:
```solidity
File: NounsDAOV3Admin.sol

162:     function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external onlyAdmin(ds) {

177:     function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external onlyAdmin(ds) {

261:     function _setPendingAdmin(NounsDAOStorageV3.StorageV3 storage ds, address newPendingAdmin) external onlyAdmin(ds) {

468:     function _withdraw(NounsDAOStorageV3.StorageV3 storage ds) external onlyAdmin(ds) returns (uint256, bool) {

482:     function _setVoteSnapshotBlockSwitchProposalId(NounsDAOStorageV3.StorageV3 storage ds) external onlyAdmin(ds) {

527:     function _setForkEscrow(NounsDAOStorageV3.StorageV3 storage ds, address newForkEscrow) external onlyAdmin(ds) {

533:     function _setForkPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newForkPeriod) external onlyAdmin(ds) {

```

```solidity
File: data/NounsDAOData.sol

294:     function setCreateCandidateCost(uint256 newCreateCandidateCost) external onlyOwner {

301:     function setUpdateCandidateCost(uint256 newUpdateCandidateCost) external onlyOwner {

313:     function withdrawETH(address to, uint256 amount) external onlyOwner {

344:     function _authorizeUpgrade(address) internal view override onlyOwner {}

```

```solidity
File: fork/NounsDAOForkEscrow.sol

116:     function returnTokensToOwner(address owner, uint256[] calldata tokenIds) external onlyDAO {

133:     function closeEscrow() external onlyDAO returns (uint32 closedForkId) {

147:     function withdrawTokens(uint256[] calldata tokenIds, address to) external onlyDAO {

```

```solidity
File: fork/newdao/NounsAuctionHouseFork.sol

159:     function pause() external override onlyOwner {

168:     function unpause() external override onlyOwner {

180:     function setTimeBuffer(uint256 _timeBuffer) external override onlyOwner {

190:     function setReservePrice(uint256 _reservePrice) external override onlyOwner {

200:     function setMinBidIncrementPercentage(uint8 _minBidIncrementPercentage) external override onlyOwner {

281:     function _authorizeUpgrade(address) internal view override onlyOwner {}

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

198:     function setContractURIHash(string memory newContractURIHash) external onlyOwner {

206:     function mint() public override onlyMinter returns (uint256) {

213:     function burn(uint256 nounId) public override onlyMinter {

240:     function setMinter(address _minter) external override onlyOwner whenMinterNotLocked {

250:     function lockMinter() external override onlyOwner whenMinterNotLocked {

260:     function setDescriptor(INounsDescriptorMinimal _descriptor) external override onlyOwner whenDescriptorNotLocked {

270:     function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {

280:     function setSeeder(INounsSeeder _seeder) external override onlyOwner whenSeederNotLocked {

290:     function lockSeeder() external override onlyOwner whenSeederNotLocked {

327:     function _authorizeUpgrade(address) internal view override onlyOwner {}

```

### <a name="GAS-10"></a>[GAS-10] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (38)*:
```solidity
File: NounsDAOLogicV1.sol

216:         proposalCount++;

281:         for (uint256 i = 0; i < proposal.targets.length; i++) {

319:         for (uint256 i = 0; i < proposal.targets.length; i++) {

346:         for (uint256 i = 0; i < proposal.targets.length; i++) {

371:         for (uint256 i = 0; i < proposal.targets.length; i++) {

```

```solidity
File: NounsDAOLogicV2.sol

239:         proposalCount++;

305:         for (uint256 i = 0; i < proposal.targets.length; i++) {

343:         for (uint256 i = 0; i < proposal.targets.length; i++) {

372:         for (uint256 i = 0; i < proposal.targets.length; i++) {

405:         for (uint256 i = 0; i < proposal.targets.length; i++) {

```

```solidity
File: NounsDAOV3Admin.sol

602:         for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

603:             for (uint256 j = i + 1; j < erc20tokens.length; j++) {

```

```solidity
File: NounsDAOV3Proposals.sol

446:         for (uint256 i = 0; i < proposal.targets.length; i++) {

508:         for (uint256 i = 0; i < proposal.targets.length; i++) {

553:         for (uint256 i = 0; i < proposal.targets.length; i++) {

602:         for (uint256 i = 0; i < proposal.targets.length; i++) {

837:             signers[numSigners++] = signer;

```

```solidity
File: fork/NounsDAOForkEscrow.sol

105:         numTokensInEscrow++;

117:         for (uint256 i = 0; i < tokenIds.length; i++) {

138:         forkId++;

148:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: fork/NounsDAOV3Fork.sol

83:         for (uint256 i = 0; i < tokenIds.length; i++) {

153:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

209:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

223:         for (uint256 i = 0; i < tokenIds.length; i++) {

231:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

238:         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {

248:         for (uint256 i = 0; i < addresses.length; i++) {

316:         proposalCount++;

397:         for (uint256 i = 0; i < proposal.targets.length; i++) {

435:         for (uint256 i = 0; i < proposal.targets.length; i++) {

462:         for (uint256 i = 0; i < proposal.targets.length; i++) {

796:         for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

797:             for (uint256 j = i + 1; j < erc20tokens.length; j++) {

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

149:         for (uint256 i = 0; i < tokenIds.length; i++) {

172:         for (uint256 i = 0; i < tokenIds.length; i++) {

207:         return _mintTo(minter, _currentNounId++);

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

153:         require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');

```

### <a name="GAS-11"></a>[GAS-11] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (79)*:
```solidity
File: NounsDAOExecutor.sol

62:     uint256 public constant GRACE_PERIOD = 14 days;

63:     uint256 public constant MINIMUM_DELAY = 2 days;

64:     uint256 public constant MAXIMUM_DELAY = 30 days;

```

```solidity
File: NounsDAOExecutorV2.sol

82:     string public constant NAME = 'NounsDAOExecutorV2';

85:     uint256 public constant GRACE_PERIOD = 21 days;

86:     uint256 public constant MINIMUM_DELAY = 2 days;

87:     uint256 public constant MAXIMUM_DELAY = 30 days;

```

```solidity
File: NounsDAOLogicV1.sol

67:     string public constant name = 'Nouns DAO';

70:     uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%

73:     uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%

76:     uint256 public constant MIN_VOTING_PERIOD = 5_760; // About 24 hours

79:     uint256 public constant MAX_VOTING_PERIOD = 80_640; // About 2 weeks

82:     uint256 public constant MIN_VOTING_DELAY = 1;

85:     uint256 public constant MAX_VOTING_DELAY = 40_320; // About 1 week

88:     uint256 public constant MIN_QUORUM_VOTES_BPS = 200; // 200 basis points or 2%

91:     uint256 public constant MAX_QUORUM_VOTES_BPS = 2_000; // 2,000 basis points or 20%

94:     uint256 public constant proposalMaxOperations = 10; // 10 actions

97:     bytes32 public constant DOMAIN_TYPEHASH =

101:     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

```solidity
File: NounsDAOLogicV2.sol

59:     string public constant name = 'Nouns DAO';

62:     uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%

65:     uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%

68:     uint256 public constant MIN_VOTING_PERIOD = 5_760; // About 24 hours

71:     uint256 public constant MAX_VOTING_PERIOD = 80_640; // About 2 weeks

74:     uint256 public constant MIN_VOTING_DELAY = 1;

77:     uint256 public constant MAX_VOTING_DELAY = 40_320; // About 1 week

80:     uint256 public constant MIN_QUORUM_VOTES_BPS_LOWER_BOUND = 200; // 200 basis points or 2%

83:     uint256 public constant MIN_QUORUM_VOTES_BPS_UPPER_BOUND = 2_000; // 2,000 basis points or 20%

86:     uint256 public constant MAX_QUORUM_VOTES_BPS_UPPER_BOUND = 6_000; // 4,000 basis points or 60%

89:     uint256 public constant MAX_QUORUM_VOTES_BPS = 2_000; // 2,000 basis points or 20%

92:     uint256 public constant proposalMaxOperations = 10; // 10 actions

95:     uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;

98:     uint256 public constant REFUND_BASE_GAS = 36000;

101:     uint256 public constant MAX_REFUND_GAS_USED = 200_000;

104:     uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;

107:     bytes32 public constant DOMAIN_TYPEHASH =

111:     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

```solidity
File: NounsDAOV3Admin.sol

112:     uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%

115:     uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%

118:     uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;

121:     uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;

124:     uint256 public constant MIN_VOTING_DELAY_BLOCKS = 1;

127:     uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;

130:     uint256 public constant MIN_QUORUM_VOTES_BPS_LOWER_BOUND = 200; // 200 basis points or 2%

133:     uint256 public constant MIN_QUORUM_VOTES_BPS_UPPER_BOUND = 2_000; // 2,000 basis points or 20%

136:     uint256 public constant MAX_QUORUM_VOTES_BPS_UPPER_BOUND = 6_000; // 6,000 basis points or 60%

139:     uint256 public constant MAX_FORK_PERIOD = 14 days;

142:     uint256 public constant MIN_FORK_PERIOD = 2 days;

145:     uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;

148:     uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;

```

```solidity
File: NounsDAOV3Proposals.sol

139:     uint256 public constant PROPOSAL_MAX_OPERATIONS = 10; // 10 actions

141:     bytes32 public constant DOMAIN_TYPEHASH =

144:     bytes32 public constant PROPOSAL_TYPEHASH =

149:     bytes32 public constant UPDATE_PROPOSAL_TYPEHASH =

```

```solidity
File: NounsDAOV3Votes.sol

44:     string public constant name = 'Nouns DAO';

47:     bytes32 public constant DOMAIN_TYPEHASH =

51:     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

54:     uint256 public constant MAX_REFUND_PRIORITY_FEE = 2 gwei;

57:     uint256 public constant REFUND_BASE_GAS = 36000;

60:     uint256 public constant MAX_REFUND_GAS_USED = 200_000;

63:     uint256 public constant MAX_REFUND_BASE_FEE = 200 gwei;

```

```solidity
File: data/NounsDAOData.sol

92:     uint256 public constant PRIOR_VOTES_BLOCKS_AGO = 1;

```

```solidity
File: fork/newdao/NounsAuctionHouseFork.sol

48:     string public constant NAME = 'NounsAuctionHouseFork';

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

116:     string public constant name = 'Nouns DAO';

119:     uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%

122:     uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%

125:     uint256 public constant MIN_VOTING_PERIOD = 7_200; // 24 hours

128:     uint256 public constant MAX_VOTING_PERIOD = 100_800; // 2 weeks

131:     uint256 public constant MIN_VOTING_DELAY = 1;

134:     uint256 public constant MAX_VOTING_DELAY = 100_800; // 2 weeks

137:     uint256 public constant MIN_QUORUM_VOTES_BPS = 200; // 200 basis points or 2%

140:     uint256 public constant MAX_QUORUM_VOTES_BPS = 2_000; // 2,000 basis points or 20%

143:     uint256 public constant proposalMaxOperations = 10; // 10 actions

146:     bytes32 public constant DOMAIN_TYPEHASH =

150:     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

46:     string public constant NAME = 'NounsTokenFork';

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

52:     uint8 public constant decimals = 0;

70:     bytes32 public constant DOMAIN_TYPEHASH =

74:     bytes32 public constant DELEGATION_TYPEHASH =

```

### <a name="GAS-12"></a>[GAS-12] Use shift Right/Left instead of division/multiplication if possible

*Instances (3)*:
```solidity
File: NounsDAOLogicV2.sol

1010:             uint256 center = upper - (upper - lower) / 2;

```

```solidity
File: NounsDAOV3DynamicQuorum.sol

110:             uint256 center = upper - (upper - lower) / 2;

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

196:             uint32 center = upper - (upper - lower) / 2; // ceil, avoiding overflow

```

### <a name="GAS-13"></a>[GAS-13] Splitting require() statements that use && saves gas

*Instances (3)*:
```solidity
File: NounsDAOLogicV1.sol

617:         require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

```

```solidity
File: NounsDAOLogicV2.sol

856:         require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

733:         require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

```

### <a name="GAS-14"></a>[GAS-14] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (16)*:
```solidity
File: NounsDAOLogicV2.sol

564:         if (votes > 0) {

1026:         if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```

```solidity
File: NounsDAOV3Admin.sol

484:         if (oldVoteSnapshotBlockSwitchProposalId > 0) {

585:         if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```

```solidity
File: NounsDAOV3Proposals.sol

888:         if (proposal.signers.length > 0) revert ProposerCannotUpdateProposalWithSigners();

```

```solidity
File: NounsDAOV3Votes.sol

132:         if (votes > 0) {

```

```solidity
File: data/NounsDAOData.sol

331:         return nounsToken.getPriorVotes(account, block.number - PRIOR_VOTES_BLOCKS_AGO) > 0;

```

```solidity
File: fork/NounsDAOV3Fork.sol

255:             if (tokensToSend > 0) {

```

```solidity
File: fork/newdao/NounsAuctionHouseFork.sol

251:         if (_auction.amount > 0) {

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

239:             if (balancesToSend[i] > 0) {

381:         if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

165:         return nCheckpoints > 0 ? checkpoints[account][nCheckpoints - 1].votes : 0;

227:         if (srcRep != dstRep && amount > 0) {

230:                 uint96 srcRepOld = srcRepNum > 0 ? checkpoints[srcRep][srcRepNum - 1].votes : 0;

237:                 uint96 dstRepOld = dstRepNum > 0 ? checkpoints[dstRep][dstRepNum - 1].votes : 0;

255:         if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {

```

