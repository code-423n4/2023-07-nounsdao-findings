# Report


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 2 |
| [NC-2](#NC-2) | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 4 |
| [NC-3](#NC-3) | Constants in comparisons should appear on the left side | 38 |
| [NC-4](#NC-4) | delete keyword can be used instead of setting to 0 | 13 |
| [NC-5](#NC-5) | Lines are too long | 3 |
| [NC-6](#NC-6) | Event is missing `indexed` fields | 46 |
| [NC-7](#NC-7) | Functions not used internally could be marked external | 15 |
| [NC-8](#NC-8) | Variable names don't follow the Solidity style guide | 8 |
| [NC-9](#NC-9) | Variables need not be initialized to zero | 1 |
### <a name="NC-1"></a>[NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (2)*:
```solidity
File: NounsDAOExecutor.sol

90:         require(msg.sender == pendingAdmin, 'NounsDAOExecutor::acceptAdmin: Call must come from pendingAdmin.');

120:         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

```

### <a name="NC-2"></a>[NC-2] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant
constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

*Instances (4)*:
```solidity
File: NounsDAOLogicV1.sol

101:     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

```solidity
File: NounsDAOLogicV2.sol

111:     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

```solidity
File: NounsDAOV3Votes.sol

51:     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

150:     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```

### <a name="NC-3"></a>[NC-3] Constants in comparisons should appear on the left side
Constants should appear on the left side of comparisons, to avoid accidental assignment

*Instances (38)*:
```solidity
File: NounsDAOExecutor.sol

166:         if (bytes(signature).length == 0) {

```

```solidity
File: NounsDAOExecutorV2.sol

189:         if (bytes(signature).length == 0) {

```

```solidity
File: NounsDAOLogicV1.sol

434:         } else if (proposal.eta == 0) {

510:         if (support == 0) {

512:         } else if (support == 1) {

514:         } else if (support == 2) {

```

```solidity
File: NounsDAOLogicV2.sol

468:         } else if (proposal.eta == 0) {

625:         if (support == 0) {

627:         } else if (support == 1) {

629:         } else if (support == 2) {

926:         if (proposal.creationBlock == 0) {

938:         if (proposal.totalSupply == 0) {

985:         if (len == 0) {

1036:             if (balance == 0) {

```

```solidity
File: NounsDAOV3Admin.sol

600:         if (erc20tokens.length == 0) return;

```

```solidity
File: NounsDAOV3DynamicQuorum.sol

34:         if (proposal.totalSupply == 0) {

85:         if (len == 0) {

```

```solidity
File: NounsDAOV3Proposals.sol

226:         if (proposerSignatures.length == 0) revert MustProvideSignatures();

251:         if (signers.length == 0) revert MustProvideSignatures();

394:         if (proposerSignatures.length == 0) revert MustProvideSignatures();

658:         } else if (proposal.eta == 0) {

833:             if (signerVotes == 0) {

972:         if (txs.targets.length == 0) revert MustProvideActions();

```

```solidity
File: NounsDAOV3Votes.sol

225:         if (support == 1) {

232:         if (support == 0) {

234:         } else if (support == 1) {

236:         } else if (support == 2) {

248:             proposal.objectionPeriodEndBlock == 0 &&

298:             if (balance == 0) {

326:         if (proposalId < voteSnapshotBlockSwitchProposalId || voteSnapshotBlockSwitchProposalId == 0) {

```

```solidity
File: data/NounsDAOData.sol

283:         if (votes == 0) revert MustBeNouner();

```

```solidity
File: fork/newdao/NounsAuctionHouseFork.sol

171:         if (auction.startTime == 0 || auction.settled) {

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

523:         } else if (proposal.eta == 0) {

626:         if (support == 0) {

628:         } else if (support == 1) {

630:         } else if (support == 2) {

794:         if (erc20tokens.length == 0) return;

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

179:         if (nCheckpoints == 0) {

```

### <a name="NC-4"></a>[NC-4] delete keyword can be used instead of setting to 0
It's clearer and reflects the intention of the programmer

*Instances (13)*:
```solidity
File: NounsDAOLogicV1.sol

223:         newProposal.eta = 0;

230:         newProposal.forVotes = 0;

231:         newProposal.againstVotes = 0;

232:         newProposal.abstainVotes = 0;

```

```solidity
File: NounsDAOLogicV2.sol

244:         newProposal.eta = 0;

251:         newProposal.forVotes = 0;

252:         newProposal.againstVotes = 0;

253:         newProposal.abstainVotes = 0;

```

```solidity
File: NounsDAOV3Votes.sol

289:         receipt.support = 0;

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

323:         newProposal.eta = 0;

330:         newProposal.forVotes = 0;

331:         newProposal.againstVotes = 0;

332:         newProposal.abstainVotes = 0;

```

### <a name="NC-5"></a>[NC-5] Lines are too long
Recommended by solidity docs to keep lines to 120 characters or lesser

*Instances (3)*:
```solidity
File: NounsDAOV3Proposals.sol

146:             'Proposal(address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'

151:             'UpdateProposal(uint256 proposalId,address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

48: import { ERC721EnumerableUpgradeable } from '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';

```

### <a name="NC-6"></a>[NC-6] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (46)*:
```solidity
File: NounsDAOExecutor.sol

62:     uint256 public constant GRACE_PERIOD = 14 days;

67:     address public pendingAdmin;

73:         require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');

```

```solidity
File: NounsDAOInterfaces.sol

59:         uint256 startBlock,

69:     /// @param support Support value for the vote. 0=against, 1=for, 2=abstain

84:     event ProposalVetoed(uint256 id);

89:     /// @notice An event emitted when the voting period is set

92:     /// @notice Emitted when implementation is changed

95:     /// @notice Emitted when proposal threshold basis points is set

96:     event ProposalThresholdBPSSet(uint256 oldProposalThresholdBPS, uint256 newProposalThresholdBPS);

98:     /// @notice Emitted when quorum votes basis points is set

101:     /// @notice Emitted when pendingAdmin is changed

104:     /// @notice Emitted when pendingAdmin is accepted, which means admin is updated

108:     event NewVetoer(address oldVetoer, address newVetoer);

113:     event MinQuorumVotesBPSSet(uint16 oldMinQuorumVotesBPS, uint16 newMinQuorumVotesBPS);

116:     event MaxQuorumVotesBPSSet(uint16 oldMaxQuorumVotesBPS, uint16 newMaxQuorumVotesBPS);

119:     event QuorumCoefficientSet(uint32 oldQuorumCoefficient, uint32 newQuorumCoefficient);

121:     /// @notice Emitted when a voter cast a vote requesting a gas refund.

124:     /// @notice Emitted when admin withdraws the DAO's balance.

128:     event NewPendingVetoer(address oldPendingVetoer, address newPendingVetoer);

132:     /// @notice An event emitted when a new proposal is created, which includes additional information

134:     event ProposalCreatedWithRequirements(

139:         uint256[] values,

143:         uint256 endBlock,

153:     /// @notice Emitted when a proposal is updated

170:         uint256[] values,

176:     /// @notice Emitted when a proposal's description is updated

185:     event ProposalObjectionPeriodSet(uint256 indexed id, uint256 objectionPeriodEndBlock);

192:         uint32 oldObjectionPeriodDurationInBlocks,

197:     event LastMinuteWindowSet(uint32 oldLastMinuteWindowInBlocks, uint32 newLastMinuteWindowInBlocks);

200:     event ProposalUpdatablePeriodSet(

205:     /// @notice Emitted when the proposal id at which vote snapshot block changes is set

211:     /// @notice Emitted when the erc20 tokens to include in a fork are set

214:     /// @notice Emitted when the fork DAO deployer is set

218:     event ForkPeriodSet(uint256 oldForkPeriod, uint256 newForkPeriod);

223:     /// @notice Emitted when the main timelock, timelockV1 and admin are set

226:     /// @notice Emitted when someones adds nouns to the fork escrow

231:         uint256[] proposalIds,

236:     event WithdrawFromForkEscrow(uint32 indexed forkId, address indexed owner, uint256[] tokenIds);

238:     /// @notice Emitted when the fork is executed and the forking period begins

243:         uint256 forkEndTimestamp,

253:         string reason

257:     event DAOWithdrawNounsFromEscrow(uint256[] tokenIds, address to);

264:     /// @notice Administrator for this contract

276:  * @notice For future upgrades, do not change NounsDAOStorageV1. Create a new

278:  * NounsDAOStorageVX.

```

### <a name="NC-7"></a>[NC-7] Functions not used internally could be marked external

*Instances (15)*:
```solidity
File: NounsDAOExecutor.sol

90:         require(msg.sender == pendingAdmin, 'NounsDAOExecutor::acceptAdmin: Call must come from pendingAdmin.');

107:     function queueTransaction(

114:         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');

121:         queuedTransactions[txHash] = true;

145:         string memory signature,

155:             "NounsDAOExecutor::executeTransaction: Transaction hasn't surpassed time lock."

```

```solidity
File: NounsDAOLogicV1.sol

191:         require(

666:      * Differs from `GovernerBravo` which uses fixed amount

676:     function getChainIdInternal() internal view returns (uint256) {

684: 

```

```solidity
File: NounsDAOLogicV2.sol

214:         require(

896:         emit NewPendingVetoer(pendingVetoer, address(0));

921:     function proposalThreshold() public view returns (uint256) {

936:     function quorumVotes(uint256 proposalId) public view returns (uint256) {

1079:         require(n <= type(uint32).max, errorMessage);

```

### <a name="NC-8"></a>[NC-8] Variable names don't follow the Solidity style guide
Constant variable should be all caps  each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

*Instances (8)*:
```solidity
File: NounsDAOLogicV1.sol

67:     string public constant name = 'Nouns DAO';

94:     uint256 public constant proposalMaxOperations = 10; // 10 actions

```

```solidity
File: NounsDAOLogicV2.sol

59:     string public constant name = 'Nouns DAO';

92:     uint256 public constant proposalMaxOperations = 10; // 10 actions

```

```solidity
File: NounsDAOV3Votes.sol

44:     string public constant name = 'Nouns DAO';

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

116:     string public constant name = 'Nouns DAO';

143:     uint256 public constant proposalMaxOperations = 10; // 10 actions

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

52:     uint8 public constant decimals = 0;

```

### <a name="NC-9"></a>[NC-9] Variables need not be initialized to zero
Default value is zero which makes the initialization unnecessary.

*Instances (1)*:
```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

52:     uint8 public constant decimals = 0;

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 10 |
| [L-2](#L-2) | Empty Function Body - Consider commenting why | 11 |
| [L-3](#L-3) | Initializers could be front-run | 22 |
| [L-4](#L-4) | _safeMint() should be used rather than _mint() | 2 |
| [L-5](#L-5) | Unsafe ERC20 operation(s) | 6 |
### <a name="L-1"></a>[L-1]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (10)*:
```solidity
File: NounsDAOLogicV1.sol

483:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

```

```solidity
File: NounsDAOLogicV2.sol

598:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

```

```solidity
File: NounsDAOV3Proposals.sol

872:                 keccak256(abi.encodePacked(txs.targets)),

873:                 keccak256(abi.encodePacked(txs.values)),

874:                 keccak256(abi.encodePacked(signatureHashes)),

875:                 keccak256(abi.encodePacked(calldatasHashes)),

1006:         bytes32 structHash = keccak256(abi.encodePacked(typehash, proposalEncodeData, expirationTimestamp));

```

```solidity
File: NounsDAOV3Votes.sol

168:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

599:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

```

```solidity
File: fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

150:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));

```

### <a name="L-2"></a>[L-2] Empty Function Body - Consider commenting why

*Instances (11)*:
```solidity
File: NounsDAOExecutor.sol

186:     receive() external payable {}

188:     fallback() external payable {}

```

```solidity
File: NounsDAOExecutorProxy.sol

27:     constructor(address logic, bytes memory data) ERC1967Proxy(logic, data) {}

```

```solidity
File: NounsDAOExecutorV2.sol

209:     receive() external payable {}

211:     fallback() external payable {}

```

```solidity
File: NounsDAOLogicV2.sol

1090:     receive() external payable {}

```

```solidity
File: NounsDAOLogicV3.sol

1035:     receive() external payable {}

```

```solidity
File: data/NounsDAOData.sol

344:     function _authorizeUpgrade(address) internal view override onlyOwner {}

```

```solidity
File: data/NounsDAODataProxy.sol

23:     constructor(address _logic, bytes memory _data) payable ERC1967Proxy(_logic, _data) {}

```

```solidity
File: fork/newdao/NounsAuctionHouseFork.sol

281:     function _authorizeUpgrade(address) internal view override onlyOwner {}

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

327:     function _authorizeUpgrade(address) internal view override onlyOwner {}

```

### <a name="L-3"></a>[L-3] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (22)*:
```solidity
File: NounsDAOExecutorV2.sol

95:     function initialize(address admin_, uint256 delay_) public virtual initializer {

95:     function initialize(address admin_, uint256 delay_) public virtual initializer {

```

```solidity
File: NounsDAOLogicV1.sol

113:     function initialize(

```

```solidity
File: NounsDAOLogicV2.sol

135:     function initialize(

```

```solidity
File: NounsDAOLogicV3.sol

141:     function initialize(

```

```solidity
File: NounsDAOProxy.sol

58:                 'initialize(address,address,address,uint256,uint256,uint256,uint256)',

```

```solidity
File: NounsDAOProxyV2.sol

61:                 'initialize(address,address,address,uint256,uint256,uint256,(uint16,uint16,uint32))',

```

```solidity
File: NounsDAOProxyV3.sol

61:                 'initialize(address,address,address,address,address,(uint256,uint256,uint256,uint32,uint32,uint32),(uint16,uint16,uint32))',

```

```solidity
File: data/NounsDAOData.sol

116:     function initialize(

120:     ) external initializer {

```

```solidity
File: fork/ForkDAODeployer.sol

104:         NounsTokenFork(token).initialize(

114:         NounsAuctionHouseFork(auction).initialize(

126:         NounsDAOExecutorV2(payable(treasury)).initialize(governor, originalTimelock.delay());

141:         NounsDAOLogicV1Fork(governor).initialize(

```

```solidity
File: fork/newdao/NounsAuctionHouseFork.sol

76:     function initialize(

84:     ) external initializer {

85:         __Pausable_init();

86:         __ReentrancyGuard_init();

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

165:     function initialize(

```

```solidity
File: fork/newdao/token/NounsTokenFork.sol

119:     function initialize(

127:     ) external initializer {

128:         __ERC721_init('Nouns', 'NOUN');

```

### <a name="L-4"></a>[L-4] _safeMint() should be used rather than _mint()
_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function

*Instances (2)*:
```solidity
File: fork/newdao/token/NounsTokenFork.sol

302:         _mint(to, nounId);

318:         _mint(to, nounId);

```

### <a name="L-5"></a>[L-5] Unsafe ERC20 operation(s)

*Instances (6)*:
```solidity
File: fork/NounsDAOForkEscrow.sol

120:             nounsToken.transferFrom(address(this), owner, tokenIds[i]);

151:             nounsToken.transferFrom(address(this), to, tokenIds[i]);

```

```solidity
File: fork/NounsDAOV3Fork.sol

154:             ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]);

```

```solidity
File: fork/newdao/NounsAuctionHouseFork.sol

248:             nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);

264:             IERC20(weth).transfer(to, amount);

```

```solidity
File: fork/newdao/governance/NounsDAOLogicV1Fork.sol

224:             nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);

```


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners | 18 |
### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (18)*:
```solidity
File: data/NounsDAOData.sol

294:     function setCreateCandidateCost(uint256 newCreateCandidateCost) external onlyOwner {

301:     function setUpdateCandidateCost(uint256 newUpdateCandidateCost) external onlyOwner {

313:     function withdrawETH(address to, uint256 amount) external onlyOwner {

344:     function _authorizeUpgrade(address) internal view override onlyOwner {}

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

240:     function setMinter(address _minter) external override onlyOwner whenMinterNotLocked {

250:     function lockMinter() external override onlyOwner whenMinterNotLocked {

260:     function setDescriptor(INounsDescriptorMinimal _descriptor) external override onlyOwner whenDescriptorNotLocked {

270:     function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {

280:     function setSeeder(INounsSeeder _seeder) external override onlyOwner whenSeederNotLocked {

290:     function lockSeeder() external override onlyOwner whenSeederNotLocked {

327:     function _authorizeUpgrade(address) internal view override onlyOwner {}

```

