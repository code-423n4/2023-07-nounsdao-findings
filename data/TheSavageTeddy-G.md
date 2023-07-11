## Gas optimizations list

| Number  | Issue Title                                          | Number of Instances |
| ------- | ---------------------------------------------------- | ------------------ |
| [G-01]  | Use constants instead of `type(uintx).max`            | 7                  |
| [G-02]  | Do not calculate constants                           | 5                  |
| [G-03]  | Avoid multiple check combinations by using nested `if` statements | 6                  |
| [G-04]  | Expressions for constant values                       | 3                  |
| [G-05]  | Use hardcode address instead of `address(this)`       | 18                 |
| [G-06]  | Change `public` function visibility to `external`     | 64                 |
| [G-07]  | Remove unnecessary use of variable                    | 8                  |


### [G-01] Use constants instead of `type(uintx).max`

Using `type(uintx).max` such as `type(uint128).max` incurs more gas in distribution and for each transaction. Constants (such as `340282366920938463463374607431768211455` for `type(uint128).max`) should be used instead.


*There are 7 instances of this issue*
```solidity
File: NounsDAOLogicV2.sol
  1079         require(n <= type(uint32).max, errorMessage);
  1084         if (n > type(uint16).max) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1079
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1084
```solidity
File: NounsDAOV3Admin.sol
  595         require(n <= type(uint32).max, errorMessage);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L595
```solidity
File: NounsDAOV3DynamicQuorum.sol
  146         require(n <= type(uint32).max, errorMessage);
  151         if (n > type(uint16).max) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L146
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L151
```solidity
File: ProposeDAOV3UpgradeMainnet.s.sol
  116         calldatas[i] = abi.encode(erc20Transferer, type(uint256).max);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L116
```solidity
File: ProposeDAOV3UpgradeTestnet.s.sol
  123         calldatas[i] = abi.encode(erc20Transferer, type(uint256).max);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L123



### [G-02] Do not calculate constants

The implementation of constant variables (replacements at compile-time) leads to the recomputation of an expression assigned to a constant variable each time the variable is used, wasting gas. Therefore constants should not involve calculations, rather the final value. For clarity, use comments to show where that value is being derived from.

*There are 5 instances of this issue*
```solidity
File: NounsDAOV3Admin.sol
117:     /// @notice The minimum setable voting period in blocks
118:     uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;
119: 
120:     /// @notice The max setable voting period in blocks
121:     uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;
126:     /// @notice The max setable voting delay in blocks
127:     uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;
144:     /// @notice Upper bound for objection period duration in blocks.
145:     uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;
146: 
147:     /// @notice Upper bound for proposal updatable period duration in blocks.
148:     uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L117-L121
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L126-L127
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L144-L148



### [G-03] Avoid multiple check combinations by using nested `if` statements

Using nested `if` statements is cheaper than using multiple check combinations with `&&`.

*There are 6 instances of this issue:*

```solidity
File: NounsDAOV3Votes.sol
  240         if (
  241             // only for votes can trigger an objection period
  242             // we're in the last minute window
  243             isForVoteInLastMinuteWindow &&
  244             // first part of the vote flip check
  245             // separated from the second part to optimize gas
  246             isDefeatedBefore &&
  247             // haven't turn on objection yet
  248             proposal.objectionPeriodEndBlock == 0 &&
  249             // second part of the vote flip check
  250             !ds.isDefeated(proposal)
  251         ) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L240-L241
```solidity
File: NounsDAOLogicV1Fork.sol
  381         if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1Fork.sol#L381
```solidity
File: ERC721CheckpointableUpgradeable.sol
  227         if (srcRep != dstRep && amount > 0) {
  255         if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L227
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L255

```solidity
File: NounsDAOLogicV2.sol
  1026         if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1026

```solidity
File: NounsDAOV3Admin.sol
  585         if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L585


### [G-04] Expressions for constant values such as calling `keccak256()` should use `immutable` rather than `constant`

Constant values for function calls result in increased gas costs as `immutable`s are evaluated at deployment time, in contrast to `constants` which are executed at runtime for each invocation.

*There are 3 instances of this issue:*
```solidity
File: NounsDAOLogicV1Fork.sol
  150     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L150
```solidity
File: NounsDAOLogicV2.sol
  111     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L111
```solidity
File: NounsDAOV3Votes.sol
  51     bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L51



### [G-05] Use hardcode address instead of `address(this)`

Instead of address(this), it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's script.sol and solmate ```LibRlp.sol``` contracts can do this.

References:
- https://book.getfoundry.sh/reference/forge-std/compute-create-address
- https://twitter.com/transmissions11/status/1518507047943245824


*There are 18 instances of this issue:*

```solidity
File: NounsDAOLogicV1Fork.sol
  596             abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), block.chainid, address(this))
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L596
```solidity
File: ERC721CheckpointableUpgradeable.sol
  147             abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name())), block.chainid, address(this))
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L147
```solidity
File: NounsAuctionHouseFork.sol
  248             nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L248
```solidity
File: NounsDAOForkEscrow.sol
  120             nounsToken.transferFrom(address(this), owner, tokenIds[i]);
  151             nounsToken.transferFrom(address(this), to, tokenIds[i]);
  165         return nounsToken.balanceOf(address(this)) - numTokensInEscrow;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L151
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L165
```solidity
File: NounsDAOExecutor.sol
  81         require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
  99             msg.sender == address(this),
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L81
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L99
```solidity
File: NounsDAOExecutorV2.sol
  104         require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
  122             msg.sender == address(this),
  245             msg.sender == address(this),
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L104
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L122
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L245
```solidity
File: NounsDAOLogicV2.sol
  595             abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), getChainIdInternal(), address(this))
  823         uint256 amount = address(this).balance;
  1035             uint256 balance = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L595
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L823
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1035
```solidity
File: NounsDAOV3Admin.sol
  469         uint256 amount = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L469
```solidity
File: NounsDAOV3Proposals.sol
  985         bytes32 digest = sigDigest(typehash, proposalEncodeData, proposerSignature.expirationTimestamp, address(this));
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L985
```solidity
File: NounsDAOV3Votes.sol
  165             abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), block.chainid, address(this))
  297             uint256 balance = address(this).balance;
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L165
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L297




### [G-06] Change `public` function visibility to `external` when appropriate

Since these functions are not called from within the contract, their visibility can be made from `public` to `external`, saving gas.

*There are 64 instances of this issue:*

```solidity
File: NounsDAOExecutor.sol
107:     function queueTransaction(
108:         address target,
109:         uint256 value,
110:         string memory signature,
111:         bytes memory data,
112:         uint256 eta
113:     ) public returns (bytes32) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L107-L113

```solidity
File: NounsDAOExecutor.sol
142:     function executeTransaction(
143:         address target,
144:         uint256 value,
145:         string memory signature,
146:         bytes memory data,
147:         uint256 eta
148:     ) public returns (bytes memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L142-L148

```solidity
File: NounsDAOExecutorV2.sol
130:     function queueTransaction(
131:         address target,
132:         uint256 value,
133:         string memory signature,
134:         bytes memory data,
135:         uint256 eta
136:     ) public returns (bytes32) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L130-L136

```solidity
File: NounsDAOExecutorV2.sol
165:     function executeTransaction(
166:         address target,
167:         uint256 value,
168:         string memory signature,
169:         bytes memory data,
170:         uint256 eta
171:     ) public returns (bytes memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L165-L171

```solidity
File: NounsDAOLogicV2.sol
197:     function propose(
198:         address[] memory targets,
199:         uint256[] memory values,
200:         string[] memory signatures,
201:         bytes[] memory calldatas,
202:         string memory description
203:     ) public returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L197-L203

```solidity
File: NounsDAOLogicV2.sol
921:     function proposalThreshold() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L921

```solidity
File: NounsDAOLogicV2.sol
1062:     function maxQuorumVotes() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1062

```solidity
File: NounsDAOLogicV3.sol
79:     function MIN_PROPOSAL_THRESHOLD_BPS() public pure returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L79

```solidity
File: NounsDAOLogicV3.sol
84:     function MAX_PROPOSAL_THRESHOLD_BPS() public pure returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L84

```solidity
File: NounsDAOLogicV3.sol
94:     function MAX_VOTING_PERIOD() public pure returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L94

```solidity
File: NounsDAOLogicV3.sol
94:     function MAX_VOTING_PERIOD() public pure returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L94

```solidity
File: NounsDAOLogicV3.sol
99:     function MIN_VOTING_DELAY() public pure returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L99

```solidity
File: NounsDAOLogicV3.sol
104:     function MAX_VOTING_DELAY() public pure returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L104

```solidity
File: NounsDAOLogicV3.sol
109:     function proposalMaxOperations() public pure returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L109

```solidity
File: NounsDAOLogicV3.sol
189:     function propose(
190:         address[] memory targets,
191:         uint256[] memory values,
192:         string[] memory signatures,
193:         bytes[] memory calldatas,
194:         string memory description
195:     ) public returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L189-L195

```solidity
File: NounsDAOLogicV3.sol
210:     function proposeOnTimelockV1(
211:         address[] memory targets,
212:         uint256[] memory values,
213:         string[] memory signatures,
214:         bytes[] memory calldatas,
215:         string memory description
216:     ) public returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L210-L216

```solidity
File: NounsDAOLogicV3.sol
396:     function state(uint256 proposalId) public view returns (ProposalState) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L396

```solidity
File: NounsDAOLogicV3.sol
456:     function proposalThreshold() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L456

```solidity
File: NounsDAOLogicV3.sol
882:     function quorumVotes(uint256 proposalId) public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L882

```solidity
File: NounsDAOLogicV3.sol
898:     function dynamicQuorumVotes(
899:         uint256 againstVotes,
900:         uint256 adjustedTotalSupply_,
901:         DynamicQuorumParams memory params
902:     ) public pure returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L898-L902

```solidity
File: NounsDAOLogicV3.sol
913:     function getDynamicQuorumParamsAt(uint256 blockNumber_) public view returns (DynamicQuorumParams memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L913

```solidity
File: NounsDAOLogicV3.sol
920:     function minQuorumVotes() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L920

```solidity
File: NounsDAOLogicV3.sol
927:     function maxQuorumVotes() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L927

```solidity
File: NounsDAOLogicV3.sol
934:     function quorumParamsCheckpoints() public view returns (DynamicQuorumParamsCheckpoint[] memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L934

```solidity
File: NounsDAOLogicV3.sol
941:     function quorumParamsCheckpoints(uint256 index) public view returns (DynamicQuorumParamsCheckpoint memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L941

```solidity
File: NounsDAOLogicV3.sol
951:     function vetoer() public view returns (address) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L951

```solidity
File: NounsDAOLogicV3.sol
955:     function pendingVetoer() public view returns (address) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L955

```solidity
File: NounsDAOLogicV3.sol
959:     function votingDelay() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L959

```solidity
File: NounsDAOLogicV3.sol
963:     function votingPeriod() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L963

```solidity
File: NounsDAOLogicV3.sol
967:     function proposalThresholdBPS() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L967

```solidity
File: NounsDAOLogicV3.sol
971:     function quorumVotesBPS() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L971

```solidity
File: NounsDAOLogicV3.sol
975:     function proposalCount() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L975



```solidity
File: NounsDAOLogicV3.sol
979:     function timelock() public view returns (INounsDAOExecutor) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L979

```solidity
File: NounsDAOLogicV3.sol
983:     function nouns() public view returns (NounsTokenLike) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L983

```solidity
File: NounsDAOLogicV3.sol
987:     function latestProposalIds(address account) public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L987

```solidity
File: NounsDAOLogicV3.sol
991:     function lastMinuteWindowInBlocks() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L991

```solidity
File: NounsDAOLogicV3.sol
995:     function objectionPeriodDurationInBlocks() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L995

```solidity
File: NounsDAOLogicV3.sol
999:     function erc20TokensToIncludeInFork() public view returns (address[] memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L999

```solidity
File: NounsDAOLogicV3.sol
999:     function erc20TokensToIncludeInFork() public view returns (address[] memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L999

```solidity
File: NounsDAOLogicV3.sol
1003:     function forkEscrow() public view returns (INounsDAOForkEscrow) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1003

```solidity
File: NounsDAOLogicV3.sol
1007:     function forkDAODeployer() public view returns (IForkDAODeployer) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1007

```solidity
File: NounsDAOLogicV3.sol
1011:     function forkEndTimestamp() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1011

```solidity
File: NounsDAOLogicV3.sol
1015:     function forkPeriod() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1015

```solidity
File: NounsDAOLogicV3.sol
1019:     function forkThresholdBPS() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1019

```solidity
File: NounsDAOLogicV3.sol
1023:     function proposalUpdatablePeriodInBlocks() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1023

```solidity
File: NounsDAOLogicV3.sol
1027:     function timelockV1() public view returns (address) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1027

```solidity
File: NounsDAOLogicV3.sol
1031:     function voteSnapshotBlockSwitchProposalId() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1031

```solidity
File: NounsDAOV3Admin.sol
432:     function _setDynamicQuorumParams(
433:         NounsDAOStorageV3.StorageV3 storage ds,
434:         uint16 newMinQuorumVotesBPS,
435:         uint16 newMaxQuorumVotesBPS,
436:         uint32 newQuorumCoefficient
437:     ) public onlyAdmin(ds) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L432-L437

```solidity
File: NounsDAOForkEscrow.sol
94:     function onERC721Received(
95:         address operator,
96:         address from,
97:         uint256 tokenId,
98:         bytes memory
99:     ) public override returns (bytes4) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L94-L99

```solidity
File: NounsDAOV3Fork.sol
215:     function numTokensInForkEscrow(NounsDAOStorageV3.StorageV3 storage ds) public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L215

```solidity
File: NounsDAOLogicV1Fork.sol
272:     function propose(
273:         address[] memory targets,
274:         uint256[] memory values,
275:         string[] memory signatures,
276:         bytes[] memory calldatas,
277:         string memory description
278:     ) public returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L272-L278

```solidity
File: NounsDAOLogicV1Fork.sol
765:     function proposalThreshold() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L765

```solidity
File: NounsDAOLogicV1Fork.sol
773:     function quorumVotes() public view returns (uint256) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L773

```solidity
File: NounsDAOLogicV1Fork.sol
781:     function erc20TokensToIncludeInQuitArray() public view returns(address[] memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L781

```solidity
File: NounsTokenFork.sol
190:     function contractURI() public view returns (string memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L190

```solidity
File: NounsTokenFork.sol
206:     function mint() public override onlyMinter returns (uint256) {
```
https://github.com/nounsDAO/nouns

-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L206

```solidity
File: NounsTokenFork.sol
222:     function tokenURI(uint256 tokenId) public view override returns (string memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L222

```solidity
File: NounsTokenFork.sol
231:     function dataURI(uint256 tokenId) public view override returns (string memory) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L231

```solidity
File: ERC721CheckpointableUpgradeable.sol
175:     function getPriorVotes(address account, uint256 blockNumber) public view returns (uint96) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L175

```solidity
File: DeployDAOV3DataContractsBase.s.sol
20:     function run() public returns (NounsDAODataProxy dataProxy) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3DataContractsBase.s.sol#L20

```solidity
File: ProposeDAOV3UpgradeMainnet.s.sol
23:     function run() public returns (uint256 proposalId) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L23

```solidity
File: ProposeDAOV3UpgradeTestnet.s.sol
31:     function run() public returns (uint256 proposalId) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L31

```solidity
File: ProposeENSReverseLookupConfigMainnet.s.sol
17:     function run() public returns (uint256 proposalId) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeENSReverseLookupConfigMainnet.s.sol#L17

```solidity
File: ProposeTimelockMigrationCleanupMainnet.s.sol
22:     function run() public returns (uint256 proposalId) {
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeTimelockMigrationCleanupMainnet.s.sol#L22


### [G-07] Remove unnecessary use of variable

Here, variable `i` is not being used as part or a loop or anything, so any references to it can be hardcoded (e.g. `targets[0]`, `targets[1]`) and the variable can be removed to save gas. This also includes `numTxs`.

*There are 8 instance of this issue:*
```solidity
File: ProposeTimelockMigrationCleanupMainnet.s.sol
52:         uint8 numTxs = 6;
53:         address[] memory targets = new address[](numTxs);
54:         uint256[] memory values = new uint256[](numTxs);
55:         string[] memory signatures = new string[](numTxs);
56:         bytes[] memory calldatas = new bytes[](numTxs);
57: 
58:         // Change auction house proxy admin owner
59:         uint256 i = 0;
60:         targets[i] = auctionHouseProxyAdmin;
61:         values[i] = 0;
62:         signatures[i] = 'transferOwnership(address)';
63:         calldatas[i] = abi.encode(timelockV2);
64: 
65:         // Move leftover ETH
66:         i++;
67:         targets[i] = timelockV2;
68:         values[i] = timelockV1.balance;
69:         signatures[i] = '';
70:         calldatas[i] = '';
71: 
72:         // Approve timelockV2 to move LilNouns for V1
73:         i++;
74:         targets[i] = lilNouns;
75:         values[i] = 0;
76:         signatures[i] = 'setApprovalForAll(address,bool)';
77:         calldatas[i] = abi.encode(timelockV2, true);
78: 
79:         // Transfer DAO-owned Nouns to timelockV2
80:         i++;
81:         targets[i] = nounsToken;
82:         values[i] = 0;
83:         signatures[i] = 'transferFrom(address,address,uint256)';
84:         calldatas[i] = abi.encode(timelockV1, timelockV2, 687);
85: 
86:         // Transfer ownership of TokenBuyer
87:         i++;
88:         targets[i] = TOKEN_BUYER_MAINNET;
89:         values[i] = 0;
90:         signatures[i] = 'transferOwnership(address)';
91:         calldatas[i] = abi.encode(timelockV2);
92: 
93:         // Transfer ownership of Payer
94:         i++;
95:         targets[i] = PAYER_MAINNET;
96:         values[i] = 0;
97:         signatures[i] = 'transferOwnership(address)';
98:         calldatas[i] = abi.encode(timelockV2);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeTimelockMigrationCleanupMainnet.s.sol#L52-L98

```solidity
File: ProposeDAOV3UpgradeMainnet.s.sol
066:         uint8 numTxs = 10;
067:         address[] memory targets = new address[](numTxs);
068:         uint256[] memory values = new uint256[](numTxs);
069:         string[] memory signatures = new string[](numTxs);
070:         bytes[] memory calldatas = new bytes[](numTxs);
071: 
072:         // Can't send the entire ETH balance because we can't reference self.balance
073:         // Would also be good to leave some ETH in case of queued proposals
074:         // For both reasons, we will first sent a chunk of ETH, and send the rest in a followup proposal
075:         uint256 i = 0;
076:         targets[i] = timelockV2;
077:         values[i] = ethToSendToNewTimelock;
078:         signatures[i] = '';
079:         calldatas[i] = '';
080: 
081:         // Upgrade to DAO V3
082:         i++;
083:         targets[i] = address(daoProxy);
084:         values[i] = 0;
085:         signatures[i] = '_setImplementation(address)';
086:         calldatas[i] = abi.encode(daoV3Implementation);
087: 
088:         i++;
089:         targets[i] = address(daoProxy);
090:         values[i] = 0;
091:         signatures[i] = '_setForkParams(address,address,address[],uint256,uint256)';
092:         calldatas[i] = abi.encode(
093:             forkEscrow,
094:             forkDeployer,
095:             erc20TokensToIncludeInFork,
096:             FORK_PERIOD,
097:             FORK_THRESHOLD_BPS
098:         );
099: 
100:         i++;
101:         targets[i] = address(daoProxy);
102:         values[i] = 0;
103:         signatures[i] = '_setVoteSnapshotBlockSwitchProposalId()';
104:         calldatas[i] = '';
105: 
106:         i++;
107:         targets[i] = AUCTION_HOUSE_PROXY_MAINNET;
108:         values[i] = 0;
109:         signatures[i] = 'transferOwnership(address)';
110:         calldatas[i] = abi.encode(timelockV2);
111: 
112:         i++;
113:         targets[i] = STETH_MAINNET;
114:         values[i] = 0;
115:         signatures[i] = 'approve(address,uint256)';
116:         calldatas[i] = abi.encode(erc20Transferer, type(uint256).max);
117: 
118:         i++;
119:         targets[i] = erc20Transferer;
120:         values[i] = 0;
121:         signatures[i] = 'transferEntireBalance(address,address)';
122:         calldatas[i] = abi.encode(STETH_MAINNET, timelockV2);
123: 
124:         // Change nouns token owner
125:         i++;
126:         targets[i] = NOUNS_TOKEN_MAINNET;
127:         values[i] = 0;
128:         signatures[i] = 'transferOwnership(address)';
129:         calldatas[i] = abi.encode(timelockV2);
130: 
131:         // Change descriptor owner
132:         i++;
133:         targets[i] = DESCRIPTOR_MAINNET;
134:         values[i] = 0;
135:         signatures[i] = 'transferOwnership(address)';
136:         calldatas[i] = abi.encode(timelockV2);
137: 
138:         // This must be the last transaction, because starting now, execution will be from timelockV2
139:         i++;
140:         targets[i] = address(daoProxy);
141:         values[i] = 0;
142:         signatures[i] = '_setTimelocksAndAdmin(address,address,address)';
143:         calldatas[i] = abi.encode(timelockV2, NOUNS_TIMELOCK_V1_MAINNET, timelockV2);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L66-L143

```solidity
File: ProposeDAOV3UpgradeTestnet.s.sol
073:         uint8 numTxs = 8;
074:         address[] memory targets = new address[](numTxs);
075:         uint256[] memory values = new uint256[](numTxs);
076:         string[] memory signatures = new string[](numTxs);
077:         bytes[] memory calldatas = new bytes[](numTxs);
078: 
079:         // Can't send the entire ETH balance because we can't reference self.balance
080:         // Would also be good to leave some ETH in case of queued proposals
081:         // For both reasons, we will first sent a chunk of ETH, and send the rest in a followup proposal
082:         uint256 i = 0;
083:         targets[i] = timelockV2;
084:         values[i] = ethToSendToNewTimelock;
085:         signatures[i] = '';
086:         calldatas[i] = '';
087: 
088:         // Upgrade to DAO V3
089:         i++;
090:         targets[i] = address(daoProxy);
091:         values[i] = 0;
092:         signatures[i] = '_setImplementation(address)';
093:         calldatas[i] = abi.encode(daoV3Implementation);
094: 
095:         i++;
096:         targets[i] = address(daoProxy);
097:         values[i] = 0;
098:         signatures[i] = '_setForkParams(address,address,address[],uint256,uint256)';
099:         calldatas[i] = abi.encode(
100:             forkEscrow,
101:             forkDeployer,
102:             erc20TokensToIncludeInFork,
103:             FORK_PERIOD,
104:             FORK_THRESHOLD_BPS
105:         );
106: 
107:         i++;
108:         targets[i] = address(daoProxy);
109:         values[i] = 0;
110:         signatures[i] = '_setVoteSnapshotBlockSwitchProposalId()';
111:         calldatas[i] = '';
112: 
113:         i++;
114:         targets[i] = auctionHouseProxy;
115:         values[i] = 0;
116:         signatures[i] = 'transferOwnership(address)';
117:         calldatas[i] = abi.encode(timelockV2);
118: 
119:         i++;
120:         targets[i] = stETH;
121:         values[i] = 0;
122:         signatures[i] = 'approve(address,uint256)';
123:         calldatas[i] = abi.encode(erc20Transferer, type(uint256).max);
124: 
125:         i++;
126:         targets[i] = erc20Transferer;
127:         values[i] = 0;
128:         signatures[i] = 'transferEntireBalance(address,address)';
129:         calldatas[i] = abi.encode(stETH, timelockV2);
130: 
131:         i++;
132:         targets[i] = address(daoProxy);
133:         values[i] = 0;
134:         signatures[i] = '_setTimelocksAndAdmin(address,address,address)';
135:         calldatas[i] = abi.encode(timelockV2, timelockV1, timelockV2);
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L73-L135

```solidity
File: ProposeENSReverseLookupConfigMainnet.s.sol
34:         uint8 numTxs = 1;
35:         address[] memory targets = new address[](numTxs);
36:         uint256[] memory values = new uint256[](numTxs);
37:         string[] memory signatures = new string[](numTxs);
38:         bytes[] memory calldatas = new bytes[](numTxs);
39: 
40:         uint256 i = 0;
41:         targets[i] = reverseRegistrar;
42:         values[i] = 0;
43:         signatures[i] = 'setName(string)';
44:         calldatas[i] = abi.encode('nouns.eth');
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeENSReverseLookupConfigMainnet.s.sol#L34-L44

