# Unprotected Initializer

## Description

One or more logic contracts do not protect their initializers. An attacker can call the initializer and assume ownership of the logic contract, whereby she can perform privileged operations that trick unsuspecting users into believing that she is the owner of the upgradeable contract. 

- File: contracts/governance/fork/newdao/token/NounsTokenFork.sol
```solidity=119
   function initialize(
        address _owner,
        address _minter,
        INounsDAOForkEscrow _escrow,
        uint32 _forkId,
        uint256 startNounId,
        uint256 tokensToClaim,
        uint256 _forkingPeriodEndTimestamp
    ) external initializer {
        __ERC721_init('Nouns', 'NOUN');
        _transferOwnership(_owner);
        minter = _minter;
        escrow = _escrow;
        forkId = _forkId;
        _currentNounId = startNounId;
        remainingTokensToClaim = tokensToClaim;
        forkingPeriodEndTimestamp = _forkingPeriodEndTimestamp;

        NounsTokenFork originalToken = NounsTokenFork(address(escrow.nounsToken()));
        descriptor = originalToken.descriptor();
        seeder = originalToken.seeder();
    }
```

---

- File: contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
```solidity=76
    function initialize(
        address _owner,
        INounsToken _nouns,
        address _weth,
        uint256 _timeBuffer,
        uint256 _reservePrice,
        uint8 _minBidIncrementPercentage,
        uint256 _duration
    ) external initializer {
        __Pausable_init();
        __ReentrancyGuard_init();
        _transferOwnership(_owner);

        _pause();

        nouns = _nouns;
        weth = _weth;
        timeBuffer = _timeBuffer;
        reservePrice = _reservePrice;
        minBidIncrementPercentage = _minBidIncrementPercentage;
        duration = _duration;
    }
```

---

- File: contracts/governance/NounsDAOExecutorV2.sol
```solidity=95
    function initialize(address admin_, uint256 delay_) public virtual initializer {
        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');
        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');

        admin = admin_;
        delay = delay_;
    }
```

## Recommendation
We advise calling `_disableInitializers` in the constructor or giving the constructor the `initializer` modifier to prevent the intializer from being called on the logic contract.

Reference: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializing_the_implementation_contract

# Missing Zero Address Validation

## Description
1. `https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L138`
2. `https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L836`
3. `https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L876`
4. `https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L568`
5. `https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L120`
6. `https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L121`

Addresses should be checked before assignment or external call to make sure they are not zero addresses.

## Recommenation
We advise adding a zero-check for the passed-in address value to prevent unexpected errors.

# Potential Cast Ineffective Vote

## Description
In the function `castVoteInternal()` of `NounsDAOLogicV2` contract, it is allowed to cast a zero vote for proposals. 

```solidity=611
    function castVoteInternal(
        address voter,
        uint256 proposalId,
        uint8 support
    ) internal returns (uint96) {
        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
        require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
        Proposal storage proposal = _proposals[proposalId];
        Receipt storage receipt = proposal.receipts[voter];
        require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');

        /// @notice: Unlike GovernerBravo, votes are considered from the block the proposal was created in order to normalize quorumVotes and proposalThreshold metrics
        uint96 votes = nouns.getPriorVotes(voter, proposalCreationBlock(proposal));

        if (support == 0) {
            proposal.againstVotes = proposal.againstVotes + votes;
        } else if (support == 1) {
            proposal.forVotes = proposal.forVotes + votes;
        } else if (support == 2) {
            proposal.abstainVotes = proposal.abstainVotes + votes;
        }

        receipt.hasVoted = true;
        receipt.support = support;
        receipt.votes = votes;

        return votes;
    }
```

If a voter has a zero vote power in the block of proposal creation, the function will allow them to cast a vote, but it will not have any effect on the overall vote count since zero votes are added to the proposal's vote tally.

The same issue also exists in the functions `castVoteDuringVotingPeriodInternal()` and `castObjectionInternal()` of library `NounsDAOV3Votes`.

## Proof of concept
```solidity
    function test_castZeroVote() public {
        address someone = makeAddr("someone");
        console2.log("Before voting");
        printProposal(proposalId);
        NounsDAOStorageV3.ProposalCondensed memory proposal = dao.proposalsV3(proposalId);
        vm.roll(proposal.startBlock);
        console2.log("someone's vote power at creationBlock is:");
        printPriorVotes(someone, proposal.creationBlock);
        console2.log("Starting voting");
        vm.roll(block.number + 1);
        vm.prank(someone);
        dao.castVote(proposalId, 1);
        console2.log("After voting");
        printProposal(proposalId);
    }
```

```log
[PASS] test_castZeroVote() (gas: 168783)
Logs:
  Before voting
  ----------------Proposal #1----------------
  proposer=0x6bEf539e8319dACba4C2DaD055006E79682C0f32, proposalThreshold=0, quorumVotes=0
  eta=0, startBlock=15, endBlock=7215
  forVotes=0, againstVotes=0, abstainVotes=0
  canceled=false, vetoed=false, executed=false
  totalSupply=4, creationBlock=4
  ----------------Proposal #1 End----------------
  someone's vote power at creationBlock is:
  ---0x69979820B003b34127eAdBA93BD51CAaC2f768dB in block#15----
  Prior vote is 0
  block=0, votes=0
  Starting voting
  After voting
  ----------------Proposal #1----------------
  proposer=0x6bEf539e8319dACba4C2DaD055006E79682C0f32, proposalThreshold=0, quorumVotes=0
  eta=0, startBlock=15, endBlock=7215
  forVotes=0, againstVotes=0, abstainVotes=0
  canceled=false, vetoed=false, executed=false
  totalSupply=4, creationBlock=4
  ----------------Proposal #1 End----------------

Traces:
  [168783] NounsDAOLogicV3VotesTest::test_castZeroVote() 
    ├─ [0] VM::addr(<pk>) 
    │   └─ ← someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB]
    ├─ [0] VM::label(someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB], someone) 
    │   └─ ← ()
    ├─ [0] console::log(Before voting) [staticcall]
    │   └─ ← ()
    ├─ [52573] NounsDAOProxyV3::proposalsV3(1) [staticcall]
    │   ├─ [47137] NounsDAOLogicV3::proposalsV3(1) [delegatecall]
    │   │   ├─ [40853] NounsDAOV3Proposals::100e0d6d(00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001) [delegatecall]
    │   │   │   └─ ← 0x000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000010000000000000000000000006bef539e8319dacba4c2dad055006e79682c0f32000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000f0000000000000000000000000000000000000000000000000000000000001c2f000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000260000000000000000000000000000000000000000000000000000000000000000e000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
    │   │   └─ ← (1, 0x6bEf539e8319dACba4C2DaD055006E79682C0f32, 0, 0, 0, 15, 7215, 0, 0, 0, false, false, false, 4, 4, [], 14, 0, false)
    │   └─ ← (1, 0x6bEf539e8319dACba4C2DaD055006E79682C0f32, 0, 0, 0, 15, 7215, 0, 0, 0, false, false, false, 4, 4, [], 14, 0, false)
    ├─ [0] console::log(----------------Proposal #%d----------------, 1) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(proposer=%s, proposalThreshold=%d, quorumVotes=%d, proposer: [0x6bEf539e8319dACba4C2DaD055006E79682C0f32], 0, 0) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(eta=%d, startBlock=%d, endBlock=%d, 0, 15, 7215) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(forVotes=%d, againstVotes=%d, abstainVotes=%d, 0, 0, 0) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(canceled=%s, vetoed=%s, executed=%s, false, false, false) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(totalSupply=%d, creationBlock=%d, 4, 4) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(----------------Proposal #%d End----------------, 1) [staticcall]
    │   └─ ← ()
    ├─ [11573] NounsDAOProxyV3::proposalsV3(1) [staticcall]
    │   ├─ [10637] NounsDAOLogicV3::proposalsV3(1) [delegatecall]
    │   │   ├─ [6853] NounsDAOV3Proposals::100e0d6d(00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001) [delegatecall]
    │   │   │   └─ ← 0x000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000010000000000000000000000006bef539e8319dacba4c2dad055006e79682c0f32000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000f0000000000000000000000000000000000000000000000000000000000001c2f000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000260000000000000000000000000000000000000000000000000000000000000000e000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
    │   │   └─ ← (1, 0x6bEf539e8319dACba4C2DaD055006E79682C0f32, 0, 0, 0, 15, 7215, 0, 0, 0, false, false, false, 4, 4, [], 14, 0, false)
    │   └─ ← (1, 0x6bEf539e8319dACba4C2DaD055006E79682C0f32, 0, 0, 0, 15, 7215, 0, 0, 0, false, false, false, 4, 4, [], 14, 0, false)
    ├─ [0] VM::roll(15) 
    │   └─ ← ()
    ├─ [0] console::log(someone's vote power at creationBlock is:) [staticcall]
    │   └─ ← ()
    ├─ [2807] NounsToken::getPriorVotes(someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB], 4) [staticcall]
    │   └─ ← 0
    ├─ [0] console::log(---%s in block#%d----, someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB], 15) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(Prior vote is %d, 0) [staticcall]
    │   └─ ← ()
    ├─ [574] NounsToken::numCheckpoints(someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB]) [staticcall]
    │   └─ ← 0
    ├─ [2828] NounsToken::checkpoints(someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB], 0) [staticcall]
    │   └─ ← 0, 0
    ├─ [0] console::log(block=%d, votes=%d, 0, 0) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(Starting voting) [staticcall]
    │   └─ ← ()
    ├─ [0] VM::roll(16) 
    │   └─ ← ()
    ├─ [0] VM::prank(someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB]) 
    │   └─ ← ()
    ├─ [41225] NounsDAOProxyV3::castVote(1, 1) 
    │   ├─ [40600] NounsDAOLogicV3::castVote(1, 1) [delegatecall]
    │   │   ├─ [37285] NounsDAOV3Votes::46742c7e(000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000001) [delegatecall]
    │   │   │   ├─ [807] NounsToken::getPriorVotes(someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB], 4) [staticcall]
    │   │   │   │   └─ ← 0
    │   │   │   ├─ emit VoteCast(voter: someone: [0x69979820B003b34127eAdBA93BD51CAaC2f768dB], proposalId: 1, support: 1, votes: 0, reason: )
    │   │   │   └─ ← ()
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [0] console::log(After voting) [staticcall]
    │   └─ ← ()
    ├─ [11573] NounsDAOProxyV3::proposalsV3(1) [staticcall]
    │   ├─ [10637] NounsDAOLogicV3::proposalsV3(1) [delegatecall]
    │   │   ├─ [6853] NounsDAOV3Proposals::100e0d6d(00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001) [delegatecall]
    │   │   │   └─ ← 0x000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000010000000000000000000000006bef539e8319dacba4c2dad055006e79682c0f32000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000f0000000000000000000000000000000000000000000000000000000000001c2f000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000260000000000000000000000000000000000000000000000000000000000000000e000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
    │   │   └─ ← (1, 0x6bEf539e8319dACba4C2DaD055006E79682C0f32, 0, 0, 0, 15, 7215, 0, 0, 0, false, false, false, 4, 4, [], 14, 0, false)
    │   └─ ← (1, 0x6bEf539e8319dACba4C2DaD055006E79682C0f32, 0, 0, 0, 15, 7215, 0, 0, 0, false, false, false, 4, 4, [], 14, 0, false)
    ├─ [0] console::log(----------------Proposal #%d----------------, 1) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(proposer=%s, proposalThreshold=%d, quorumVotes=%d, proposer: [0x6bEf539e8319dACba4C2DaD055006E79682C0f32], 0, 0) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(eta=%d, startBlock=%d, endBlock=%d, 0, 15, 7215) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(forVotes=%d, againstVotes=%d, abstainVotes=%d, 0, 0, 0) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(canceled=%s, vetoed=%s, executed=%s, false, false, false) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(totalSupply=%d, creationBlock=%d, 4, 4) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(----------------Proposal #%d End----------------, 1) [staticcall]
    │   └─ ← ()
    └─ ← ()

Test result: ok. 1 passed; 0 failed; finished in 27.89ms

```

## Recommendation
It is recommended to modify the function to explicitly check whether the voter has any tokens before allowing them to cast a vote. This can be done by adding a check for a non-zero vote count before executing the rest of the code in the function.

# No Upper Limit

## Description
There is no upper limit for `_minBidIncrementPercentage`. It is possible to set the percentage to an arbitrary value.

## Recommendation
It is recommended to add a reasonable boundary for the `_minBidIncrementPercentage`.

# Susceptible to Signature Replay

## Description
- File: contracts/governance/NounsDAOV3Proposals.sol

The `calcProposalEncodeData()` function calculates the encoded data for a proposal using the `abi.encode()` function. However, it does not include a `nonce` as part of the signature, which can cause signature replay attacks.

A signature replay attack occurs when a valid signature is reused to execute the same transaction multiple times. If the encoded data for a proposal does not include a nonce, the proposer can create the same proposal after the previous proposal was finished. This maybe unexpected. 

## Recommendation
To prevent signature replay attacks, it is recommended to include a unique `nonce` as part of the signature. The `nonce` should be different for each proposal and should be included in the encoded data. It can be a simple counter that is incremented for each proposal, or it can be a more complex value that incorporates additional variables or randomness to make it more difficult to predict.

# Check-Effects-Interactions Pattern Violation

## Description
File: contracts/governance/fork/NounsDAOForkEscrow.sol

In the `NounsDAOForkEscrow` contract, function `returnTokensToOwner()` violates the Check-Effects-Interactions pattern. This pattern is a best practice for writing secure smart contracts that involves performing all state changes before making any external function calls.

### External call(s)
```solidity=120
            nounsToken.transferFrom(address(this), owner, tokenIds[i]);
```
### State variables written after the call(s)
```solidity=121
            escrowedTokensByForkId[forkId][tokenIds[i]] = address(0);
```

An external function call to the `nounsToken.transferFrom()` function is made before the state changes are made.

To prevent such potential issue, it is important to ensure that all external function calls are made after the state changes have been made.

## Recommendation
It is recommended to follow the [Checks-Effects-Interactions Pattern](https://docs.soliditylang.org/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) to avoid the risk of calling unknown contracts or applying OpenZeppelin [ReentrancyGuard](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol) library - `nonReentrant` modifier for the aforementioned functions to prevent reentrancy attacks.

# Use ABI Code of `NounsDAOExecutorV2` to Wrap Original Timelock
## Description
File: contracts/governance/fork/ForkDAODeployer.sol

The `getOriginalTimelock()` function is used to return the original time lock contract instance. However, it uses the `NounsDAOExecutorV2` contract to wrap the ABI code of the `originalToken.owner()`. The `NounsDAOExecutorV2` contract is the current time lock, and it provides an interface for interacting with the original time lock contract.

If the sender call a function which exists in current time lock but not exists in original, it will revert. 

## Recommdation
It's recommended to use `NounsDAOExecutor` to wrap the ABI code of original time lock.

# Potential Divide by Zero
## Description
Performing division by zero would raise an error and revert the transaction. The `totalSupply` should potentially be zero, in this case, the function should directly return zero. 

```solidity
        uint256 againstVotesBPS = (10000 * againstVotes) / totalSupply;
```

## Recommendation
It is recommended to either reformulate the divisor expression, or to use conditionals or require statements to rule out the possibility of a divide-by-zero.

# The majority can follow the fork of the minority and force them to fork again[]
## Description
File: contracts/governance/fork/NounsDAOV3Fork.sol

In the current design of the `Nouns Fork`, if the minority initiates a forked `Nouns DAO`, the majority of the original `Nouns DAO` can choose to join the forked `Nouns DAO` and potentially force the minority to fork again if they disagree with the direction of the new DAO. This is because the majority still holds a significant amount of voting power and can use it to influence the decisions of the new DAO. However, this scenario is not unique to the `Nouns Fork` and can occur in any DAO where the majority holds a significant amount of power. 

## recommendation 
It highlights the importance of having clear governance mechanisms in place to ensure that the decisions made are fair and representative of the interests of all members, regardless of their voting power.





























