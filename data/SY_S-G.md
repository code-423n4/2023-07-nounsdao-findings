## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |Do not calculate constant  | 5 | - |
| [G-02] |Use hardcode address instead of address(this) | 10 | - |
| [G-03] |Use != 0 instead of > 0 for unsigned integer comparison | 5 | - |
| [G-04] | Bytes constants are more efficient than String constants | 5 | - |
| [G-05] |Using fixed bytes is cheaper than using string | 3 | - |
| [G-06] | With assembly, .call (bool success)  transfer can be done gas-optimized | 3 | - |
| [G-07] | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 10 | - |
| [G-08] | Use nested if and, avoid multiple check combinations  | 5 | - |
| [G-09] | Can make the variable outside the loop to save gas  | 3 | - |
| [G-10] | Not using the named return variable when a function returns, wastes deployment gas | 1  | - |
| [G-11] |Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 8 | - |
| [G-12] | >= costs less gas than > | 3 | - |
| [G-13] | Use assembly for math (add, sub, mul, div) | 8 | - |
| [G-14] | Before transfer of  some functions, we should check some variables for possible gas save | 7 | - |
| [G-15] | State Variable can be packed into fewer storage slots | 1 | - |
| [G-16] | Non efficient zero initialization | 30 | - |
| [G-17] | Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it) | 4 | - |
| [G-18] | Use constants instead of type(uintx).max | 5 | - |
| [G-19] |Avoid Storage keyword during modifiers| 1 | - |
| [G-20] |Don't initialize variables with default value | 9 | - |
| [G-21] | State varaibles only set in the Constructor should be declared Immutable | 2 | - |
| [G-22] | Should use arguments instead of state variable | 8 | - |
| [G-23] | Using calldata instead of memory for read-only arguments in external functions saves gas | 3 | - |
| [G-24] | Use selfbalance() instead of address(this).balance| 4 | - |
| [G-25] | State variables should be cached in stack variables rather than re-reading them from storage  | 4 | - |



## Gas Optimizations  

## [G-1]  Do not calculate constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.

```solidity
file: contracts/governance/NounsDAOV3Admin.sol

118    uint256 public constant MIN_VOTING_PERIOD_BLOCKS = 1 days / 12;
    
121    uint256 public constant MAX_VOTING_PERIOD_BLOCKS = 2 weeks / 12;

127    uint256 public constant MAX_VOTING_DELAY_BLOCKS = 2 weeks / 12;

145    uint256 public constant MAX_OBJECTION_PERIOD_BLOCKS = 7 days / 12;

148    uint256 public constant MAX_UPDATABLE_PERIOD_BLOCKS = 7 days / 12;


```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L118





## [G-2]  Use hardcode address instead of address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

 if the contract's address is needed in the code, it's more gas-efficient to hardcode the address as a constant rather than using the address(this) expression. This is because using address(this) requires additional gas consumption to compute the contract's address at runtime.

Here's an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```

```solidity
file: contracts/governance/NounsDAOExecutor.sol

99            msg.sender == address(this),

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L99

```solidity
file: contracts/governance/NounsDAOExecutorV2.sol

104        require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');


```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L104

```solidity
file: contracts/governance/NounsDAOLogicV2.sol

595        abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), getChainIdInternal(), address(this))

823        uint256 amount = address(this).balance;

1035       uint256 balance = address(this).balance;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L595

```solidity
file: contracts/governance/NounsDAOV3Proposals.sol

985        bytes32 digest = sigDigest(typehash, proposalEncodeData, proposerSignature.expirationTimestamp, address(this));

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L985

```solidity
file: contracts/governance/NounsDAOV3Votes.sol

165            abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), block.chainid, address(this))

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L165

```solidity
file: contracts/governance/fork/NounsDAOForkEscrow.sol

120            nounsToken.transferFrom(address(this), owner, tokenIds[i]);

151            nounsToken.transferFrom(address(this), to, tokenIds[i]);

165          return nounsToken.balanceOf(address(this)) - numTokensInEscrow;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120






## [G-3]  Use != 0 instead of > 0 for unsigned integer comparison

it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:
``` solidity
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```

```solidity
file: contracts/governance/NounsDAOV3Admin.sol

585        if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L585

```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

165        return nCheckpoints > 0 ? checkpoints[account][nCheckpoints - 1].votes : 0;

230        uint96 srcRepOld = srcRepNum > 0 ? checkpoints[srcRep][srcRepNum - 1].votes : 0;

237        uint96 dstRepOld = dstRepNum > 0 ? checkpoints[dstRep][dstRepNum - 1].votes : 0;

255        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L165





## [G-4]  Bytes constants are more efficient than String constants

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




## [G-5]  Using fixed bytes is cheaper than using string


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



## [G-6]  With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), 
this storage disappears and gas optimization is provided.

```solidity
file: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

273        (bool success, ) = to.call{ value: value, gas: 30_000 }(new bytes(0));

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L273


```solidity
file: contracts/governance/NounsDAOExecutor.sol

173        (bool success, bytes memory returnData) = target.call{ value: value }(callData);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L173

```solidity
file: contracts/governance/NounsDAOExecutorV2.sol

196        (bool success, bytes memory returnData) = target.call{ value: value }(callData);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L196


## [G-7] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

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




## [G-8] Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.


```solidity
file: contracts/governance/NounsDAOV3Votes.sol

240        if (
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
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L240-L251

```solidity
file: contracts/governance/NounsDAOLogicV2.sol

1026        if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1026

```solidity
file: contracts/governance/NounsDAOV3Admin.sol

585        if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L585

```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

255        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L255

```solidity
file: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

381        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L381


## [G-9] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: contracts/governance/fork/NounsDAOV3Fork.sol

254            uint256 tokensToSend = (erc20token.balanceOf(address(timelock)) * tokenCount) / totalSupply;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L254

```solidity
file: contracts/governance/fork/newdao/token/NounsTokenFork.sol

150            uint256 nounId = tokenIds[i];

173            uint256 nounId = tokenIds[i];

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L150


## [G-10]  Not using the named return variable when a function returns, wastes deployment gas


```solidity
file: script/DeployDAOV3NewContractsBase.s.sol

105        return timelockV2;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L105





## [G-11] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant


While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor.


```solidity
file: contracts/governance/NounsDAOLogicV2.sol

107    bytes32 public constant DOMAIN_TYPEHASH =
108        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


111    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L107-L111


```solidity
file: contracts/governance/NounsDAOV3Proposals.sol

141      bytes32 public constant DOMAIN_TYPEHASH =
142        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');


144      bytes32 public constant PROPOSAL_TYPEHASH =
145        keccak256(
146            'Proposal(address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'


149      bytes32 public constant UPDATE_PROPOSAL_TYPEHASH =
150        keccak256(
151            'UpdateProposal(uint256 proposalId,address proposer,address[] targets,uint256[] values,string[] signatures,bytes[] calldatas,string description,uint256 expiry)'


```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L141-L152

```solidity
file: contracts/governance/NounsDAOV3Votes.sol

47    bytes32 public constant DOMAIN_TYPEHASH =
48        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L47

```solidity
file: contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol

70    bytes32 public constant DOMAIN_TYPEHASH =
71        keccak256('EIP712Domain(string name,uint256 chainId,address verifyingContract)');

74    bytes32 public constant DELEGATION_TYPEHASH =
75        keccak256('Delegation(address delegatee,uint256 nonce,uint256 expiry)');


```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L70





## [G-12] >= costs less gas than >
 
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas \

Reference:  https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde


```solidity
file: contracts/governance/NounsDAOV3Admin.sol

484        if (oldVoteSnapshotBlockSwitchProposalId > 0) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L484

```solidity
file: contracts/governance/NounsDAOV3Proposals.sol

888        if (proposal.signers.length > 0) revert ProposerCannotUpdateProposalWithSigners();

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L888

```solidity
file: contracts/governance/NounsDAOV3Votes.sol

132        if (votes > 0) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L132


## [G-13] Use assembly for math (add, sub, mul, div)

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


## [G-14] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:


```solidity
file: contracts/utils/ERC20Transferer.sol

34        IERC20(token).transferFrom(msg.sender, to, balance);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L34


```solidity
file: contracts/governance/fork/NounsDAOForkEscrow.sol

120            nounsToken.transferFrom(address(this), owner, tokenIds[i]);

151            nounsToken.transferFrom(address(this), to, tokenIds[i]);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L120

```solidity
file: contracts/governance/fork/NounsDAOV3Fork.sol

84             ds.nouns.safeTransferFrom(msg.sender, address(forkEscrow), tokenIds[i]);

154            ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L84

```solidity
file: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

224            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L224

```solidity
file: contracts/governance/NounsDAOExecutorV2.sol

228         IERC20(erc20Token).safeTransfer(recipient, tokensToSend);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L228


## [G-15] State Variable can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas)

```solidity
file: contracts/governance/fork/newdao/token/NounsTokenFork.sol

61    uint32 public forkId;
62
63    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
64    uint256 public remainingTokensToClaim;
65
66    /// @notice The forking period expiration timestamp, after which new tokens cannot be claimed by the original DAO
67    uint256 public forkingPeriodEndTimestamp;
68
69    /// @notice Whether the minter can be updated
70    bool public isMinterLocked;
71
72    /// @notice Whether the descriptor can be updated
73    bool public isDescriptorLocked;
74
75    /// @notice Whether the seeder can be updated
76    bool public isSeederLocked;
77
78    /// @notice The noun seeds
79    mapping(uint256 => INounsSeeder.Seed) public seeds;
80
81    /// @notice The internal noun ID tracker
82    uint256 private _currentNounId;
83
84    /// @notice IPFS content hash of contract-level metadata
85    string private _contractURIHash = 'QmZi1n79FqWt2tTLwCqiy6nLM6xLGRsEPQ5JmReJQKNNzX';

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L61-L85


### Recommended Code

 Sort Plcae of Variables.
```solidity
       /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
       uint256 public remainingTokensToClaim;

       /// @notice The forking period expiration timestamp, after which new tokens cannot be claimed by the original DAO
       uint256 public forkingPeriodEndTimestamp;

       /// @notice The internal noun ID tracker
       uint256 private _currentNounId;

      /// @notice The noun seeds
       mapping(uint256 => INounsSeeder.Seed) public seeds;

      /// @notice IPFS content hash of contract-level metadata
       string private _contractURIHash = 'QmZi1n79FqWt2tTLwCqiy6nLM6xLGRsEPQ5JmReJQKNNzX';

       uint32 public forkId;

       /// @notice Whether the minter can be updated
       bool public isMinterLocked;

       /// @notice Whether the descriptor can be updated
       bool public isDescriptorLocked;

       /// @notice Whether the seeder can be updated
       bool public isSeederLocked;

```

## [G-16] Non efficient zero initialization
 
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





## [G-17] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

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


## [G-18] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: contracts/governance/NounsDAOLogicV2.sol

1079        require(n <= type(uint32).max, errorMessage);

1084        if (n > type(uint16).max) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1079

```solidity
file: contracts/governance/NounsDAOV3Admin.sol

595        require(n <= type(uint32).max, errorMessage);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L595

```solidity
file: contracts/governance/NounsDAOV3DynamicQuorum.sol

146        require(n <= type(uint32).max, errorMessage);

151        if (n > type(uint16).max) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L146



## [G-19] Avoid Storage keyword during modifiers

The problem with using modifiers with storage variables is that these accesses will likely have to be obtained multiple times, to pass it to the modifier, and to read it into the method code.


```solidity
file: contracts/governance/NounsDAOV3Admin.sol

150    modifier onlyAdmin(NounsDAOStorageV3.StorageV3 storage ds) {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L150


## [G-20] Don't initialize variables with default value


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

## [G-21] State varaibles only set in the Constructor should be declared Immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

```solidity
file: contracts/governance/NounsDAOExecutor.sol
///@audit admin and delay  are state variabls

76        admin = admin_;

77        delay = delay_;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L76-L77


## [G-22] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity
file: contracts/governance/NounsDAOExecutor.sol

///@audit ' delay ' is state variable. 

86        emit NewDelay(delay);

///@audit ' admin' is state variable. 

94        emit NewAdmin(admin);

///@audit ' pendingAdmin' is state variable. 

104        emit NewPendingAdmin(pendingAdmin);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L86

## [G-23] Using calldata instead of memory for read-only arguments in external functions saves gas 

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

```solidity
file: contracts/governance/fork/newdao/token/NounsTokenFork.sol

198    function setContractURIHash(string memory newContractURIHash) external onlyOwner {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L198

```solidity
file: contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol

206    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206

```solidity
file: script/ProposeDAOV3UpgradeMainnet.s.sol

33        address[] memory erc20TokensToIncludeInFork = new address[](1);

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L33


## [G-24] Use selfbalance() instead of address(this).balance

```solidity
file: contracts/governance/NounsDAOLogicV2.sol

823        uint256 amount = address(this).balance;

1035       uint256 balance = address(this).balance;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L823

```solidity
file: contracts/governance/NounsDAOV3Admin.sol

469        uint256 amount = address(this).balance;
 
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L469

```solidity
file: contracts/governance/NounsDAOV3Votes.sol

297            uint256 balance = address(this).balance;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L297



## [G-25] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function.
Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read.
Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
Most of the times this if statement will be true and we will save 100 gas at a small possibility of 3 gas loss

```solidity
file: contracts/governance/NounsDAOExecutor.sol#L152
///@audit the ' queuedTransactions ' is state variable , its access second+ in one function.

152        require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");

162        queuedTransactions[txHash] = false;

```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L152


```solidity
file: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
///@audit the ' auction ' is state variable , its access second+ in one function.

137        auction.amount = msg.value;

138        auction.bidder = payable(msg.sender);

143        auction.endTime = _auction.endTime = block.timestamp + timeBuffer;

///@audit the ' timeBuffer ' is state variable , its access second+ in one function.

141        bool extended = _auction.endTime - block.timestamp < timeBuffer;

143        auction.endTime = _auction.endTime = block.timestamp + timeBuffer;

///@audit the ' nouns ' is state variable , its access second+ in one function.

146            nouns.burn(_auction.nounId);

248            nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);


///@audit the ' weth ' is state variable , its access second+ in one function.

263            IWETH(weth).deposit{ value: amount }();

264            IERC20(weth).transfer(to, amount);


```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L246





