# QA Report
## Summary
Total 04 Low and 06 Non-Critical
### Low Risk Issues
- [L-01] Proposals cannot contain duplicate transactions.
- [L-02] Be careful implementation of OpenZeppelin's Ownable.sol now requires a direct call to set the owner. 
- [L-03] Storage Write Removal Bug On Conditional Early Termination
- [L-04] Upgrade OpenZeppelin Contract Dependency

## [L-01] Proposals cannot contain duplicate transactions
- When [queueing a proposal](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L305-L313) in the Timelock contract, a [check is done](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L325-L328) for each proposed action which verifies that this action is not being done already in this proposal or with the same eta.
- Although this design appears to be intentional, consider documenting this behavior explicitly. It must be made apparent to future development efforts that any functions which are intended to be called by governance can only be called once with the same parameters per proposal. Developers should understand to design functions such that multiple identical calls are unneeded.

## [L-02] Be careful implementation of OpenZeppelin's Ownable.sol now requires a direct call to set the owner. 
### Description:
[Upgrade Plan](https://github.com/code-423n4/2023-07-nounsdao#upgrade-plan) `viii` and `ix` transferring ownership of NounsToken and descriptor of NounsToken to V2 treasury
- Since [NounsDescriptor.sol](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/NounsDescriptor.sol#L27) , [NounsDescriptorV2.sol](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/NounsDescriptorV2.sol#L29) and [NounsToken.sol](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/NounsToken.sol#L29) use OpenZeppelin library's  `Ownable.sol` .
- The OpenZeppelin library's `Ownable.sol` has recently undergone a significant change. Earlier, the contract owner was automatically designated as the account that deployed the contract. However, the new update requires the contract owner to be specified explicitly as a constructor argument during deployment. An idea of the details of this modification can be found on this back & forth discussion with OZ, found [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/4368).
### Recommendation:
Update all implementations to include the explicit call to the `Ownable` constructor to set the owner.

## [L-03] Storage Write Removal Bug On Conditional Early Termination
See the following for more info:
https://twitter.com/solidity_lang/status/1567953562151579650?s=20&t=fXIo4hRjOiMXl2dqpD5Oyw
https://blog.soliditylang.org/2022/09/08/storage-write-removal-before-conditional-termination/
### Proof Of Concept
```
File: NounsDAOLogicV1.sol
    function getChainIdInternal() internal view returns (uint256) {
        uint256 chainId;
        assembly {
            chainId := chainid()
        }
        return chainId;
    }
```
[NounsDAOLogicV1.sol#L676-L682](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol#L676-L682)
```
File: NounsDAOLogicV2.sol
    function getChainIdInternal() internal view returns (uint256) {
        uint256 chainId;
        assembly {
            chainId := chainid()
        }
        return chainId;
    }
```
[NounsDAOLogicV2.sol#L1070-L1076](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1070-L1076)
### Recommended Mitigation Steps
Upgrade pragma to version 0.8.19

## [L-04] Upgrade OpenZeppelin Contract Dependency
An outdated OZ version is used (which has known vulnerabilities, see: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories). Release: https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0
## Proof Of Concept
Require dependencies to be at least version of 4.9.0 to include critical vulnerability patches.
```
File: nouns-monorepo/packages/nouns-contracts/package.json
32:    "@openzeppelin/contracts": "^4.1.0",
33:    "@openzeppelin/contracts-upgradeable": "^4.1.0",
```
### Recommended Mitigation Steps
Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.9.0 via >=4.9.0



### Non-Critical Issues
- [N-01] No need for `== true` or `== false` checks
- [N-02] Immutables can be named using the same convention
- [N-03] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
- [N-04] Use named parameters for mapping type declarations
- [N-05] Use SMTChecker
- [N-06] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

## [N-01] No need for `== true` or `== false` checks
There is no need to verify that == true or == false when the variable checked upon is a boolean as well.
### Proof Of Concept
```
File: NounsDAOV3Votes.sol
219:        require(receipt.hasVoted == false, 'NounsDAO::castVoteDuringVotingPeriodInternal: voter already voted');
282:        require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');
```
[NounsDAOV3Votes.sol#L219](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L219) , [NounsDAOV3Votes.sol#L282](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L282)
```
File: NounsDAOLogicV1Fork.sol
621:        require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');
```
[NounsDAOLogicV1Fork.sol#L621](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L621)
### Recommended Mitigation Steps
Instead simply check for `variable` or `!variable`

## [N-02] Immutables can be named using the same convention
Immutables should be in uppercase per naming convention. As shown below, some immutables are named using capital letters and underscores while some are not. For a better code quality, please consider naming these immutables using the same naming convention.
### Proof Of Concept
```
File: ForkDAODeployer.sol
35:    address public immutable tokenImpl;
38:    address public immutable auctionImpl;
41:    address public immutable treasuryImpl;
44:    address public immutable governorImpl;
47:    uint256 public immutable delayedGovernanceMaxDuration;
50:    uint256 public immutable initialVotingPeriod;
53:    uint256 public immutable initialVotingDelay;
56:    uint256 public immutable initialProposalThresholdBPS;
59:    uint256 public immutable initialQuorumVotesBPS;
```
[ForkDAODeployer.sol#L35-L59](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol#L35-L59)
```
File: NounsDAOForkEscrow.sol
41:    address public immutable dao;
44:    NounsTokenLike public immutable nounsToken;
```
[NounsDAOForkEscrow.sol#L41-L44](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L41-L44)
```
File: DeployDAOV3DataContractsBase.s.sol
12:    NounsDAOLogicV1 public immutable daoProxy;
13:    address public immutable timelockV2Proxy;
```
[DeployDAOV3DataContractsBase.s.sol#L12-L13](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3DataContractsBase.s.sol#L12-L13)
```
File: DeployDAOV3NewContractsBase.s.sol
20:    uint256 public immutable forkDAOVotingPeriod;
21:    uint256 public immutable forkDAOVotingDelay;
25:    NounsDAOLogicV1 public immutable daoProxy;
26:    INounsDAOExecutor public immutable timelockV1;
27:    bool public immutable deployTimelockV2Harness; // should be true only for testnets
```
[DeployDAOV3NewContractsBase.s.sol#L20-L21](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L20-L21) , [DeployDAOV3NewContractsBase.s.sol#L25-L27](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L25-L27)
```
File: ProposeDAOV3UpgradeTestnet.s.sol
14:    NounsDAOLogicV1 public immutable daoProxyContract;
15:    address public immutable timelockV1;
16:    address public immutable auctionHouseProxy;
17:    address public immutable stETH;
```
[ProposeDAOV3UpgradeTestnet.s.sol#L14-L17](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L14-L17)


## [N-03] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
### All contracts

## [N-04] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18
### All contracts

## [N-05] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [N-06] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
### Description:
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.
#### constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.