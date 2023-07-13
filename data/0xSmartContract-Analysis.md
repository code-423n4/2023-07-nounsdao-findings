# 🛠️ Analysis - Nouns DAO Project 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Architectural | Architecture feedback |
|e) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|f) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|g) |Systemic risks | Potential systemic risks in the project |
|h) |Competition analysis| What are similar projects? |
|i) |Security Approach of the Project | Audit approach of the Project |
|j) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|k) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|l) |Project team | Details about the project team and approach |
|m) |Other recommendations | What is unique? How are the existing patterns used? |
|n) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-07-nounsdao#audit-scope

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-07-nounsdao#dev-env-setup)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [Nouns Fork Spec](https://github.com/verbsteam/nouns-fork-spec/blob/main/spec.md) |
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-07-nounsdao#running-tests)|The Slither output did not indicate a direct security risk to us|
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-07-nounsdao#running-tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-07-nounsdao#audit-scope)||
|7|Project Spearbit Audit |[ Audit Report](https://gist.github.com/hrkrshnn/25e13d04fa728309370acb6ca8ccc16b#file-nouns-md)|This reports gives us a clue about the topics that should be considered in the current audit|
|8|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|9|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-07-nounsdao#upgrade-plan)|The specific focus areas indicated by the project team are the most sensitive and the potential logic-mathematical errors are also focused on.|

## b) Analysis of the code base

-Script DeployDAOV3NewContractsMainnet;
This contract is a part of a deployment script for deploying new contracts in the DAOv3 system on the  mainnet.
Distributes with Foundry

-Script ProposeDAOV3UpgradeMainnet;
This contract serves as a proposal script for proposing an upgrade to the DAOv3 system on the Ethereum mainnet. It defines the transactions to be executed as part of the proposal and submits the proposal to the DAO proxy contract.

--Script ProposeTimelockMigrationCleanupMainnet;
This contract serves as a proposal script for proposing additional ownership and funds transfers from the DAOv3 treasury V1 to V2 on the Ethereum mainnet. It defines the transactions to be executed as part of the proposal and submits the proposal to the DAO proxy contract.

-NounsDAOLogicV3.sol;
This contract serves as a deployer contract for deploying a new fork of the Nouns DAO system. It creates upgradable proxies for the token, auction house, governor, and treasury contracts, initializes them with the necessary parameters and values

-NounsDAOV3Votes.sol;
Acquired with BSD-3-Clause license in Compound Lab's GovernorBravoDelegate.sol contract and 2 new states have been added to the state machine

```js
// 2 new states have been added to the proposal state machine: Updatable, ObjectionPeriod
//
// Updated state machine:
// Updatable -> Pending -> Active -> ObjectionPeriod (conditional) -> Succeeded -> Queued -> Executed
// ┖> Defeated
//
```

<img width="596" alt="image" src="https://github.com/code-423n4/2023-07-nounsdao/assets/104318932/ae3bea67-0379-4e5b-8640-09b503c9699e">

## c) Test analysis

What did the project do differently? ;
The project, while designing the tests, modeled it with the Threat Model, wrote the tests and documented it and presented it to the auditors, which increases the auditability and gives us the project team's view of the threat models specifically for the project.

What could they have done better?;
It seems that the scope of the test has not been determined. Failure to calculate test coverage can cause very small details to be overlooked in such large projects



## d) Architectural 

The architecture in this control consists of the following 5 steps;

-Proposal editing: allowing proposers to update their proposal's transactions and text description, during the Updatable period only, which is the state upon proposal creation. Editing also works with signatures, assuming the proposer is able to accumulate signatures from the same signers.

-Propose by signature: allowing Nouners and delegates to pool their voting power towards submitting a proposal, by submitting their signature, instead of the current approach where sponsors must delegate their votes to help a proposer achieve threshold.

-Objection-only Period: a conditional voting period that gets activated upon a last-minute proposal swing from defeated to successful, affording against voters more reaction time. Only against votes are possible during the objection period.

-Votes Snapshot After Voting Delay: moving votes snapshot up, to provide Nouners with reaction time per proposal, to get their votes ready (e.g. some might want to move their delegations around). Currently the vote snapshot block is the proposal creation block.

-Nouns Fork: any token holder can signal to fork (exit) in response to a governance proposal. If a quorum of 20% of tokens signals to exit, the fork will succeed

## e) Documents 
 Documentation of the project for the new audit needs to be increased further, adding infographics and graphic-heavy documents showing the functioning of the functions will make the audit easier.

##  f) Centralization risks 
The owner role has a single point of failure and onlyOwner can use critical a few functions.

It is recommended to use timelock and multisign wallets to reduce the risk of centrality.

```solidity

packages/nouns-contracts/contracts/governance/data/NounsDAOData.sol:
  294:     function setCreateCandidateCost(uint256 newCreateCandidateCost) external onlyOwner {
  301:     function setUpdateCandidateCost(uint256 newUpdateCandidateCost) external onlyOwner {
  313:     function withdrawETH(address to, uint256 amount) external onlyOwner {
  344:     function _authorizeUpgrade(address) internal view override onlyOwner {}

packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol:
  159:     function pause() external override onlyOwner {
  168:     function unpause() external override onlyOwner {
  180:     function setTimeBuffer(uint256 _timeBuffer) external override onlyOwner {
  190:     function setReservePrice(uint256 _reservePrice) external override onlyOwner {
  200:     function setMinBidIncrementPercentage(uint8 _minBidIncrementPercentage) external override onlyOwner {
  281:     function _authorizeUpgrade(address) internal view override onlyOwner {}

packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol:
  198:     function setContractURIHash(string memory newContractURIHash) external onlyOwner {
  240:     function setMinter(address _minter) external override onlyOwner whenMinterNotLocked {
  250:     function lockMinter() external override onlyOwner whenMinterNotLocked {
  260:     function setDescriptor(INounsDescriptorMinimal _descriptor) external override onlyOwner whenDescriptorNotLocked {
  270:     function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {
  280:     function setSeeder(INounsSeeder _seeder) external override onlyOwner whenSeederNotLocked {
  290:     function lockSeeder() external override onlyOwner whenSeederNotLocked {
  327:     function _authorizeUpgrade(address) internal view override onlyOwner {}
```
## g) Systemic risk 
The biggest systemic risk is DAO management in the project, there is currently $54 million in the project's treasury, the management of this amount is carried out entirely on proposals, all proposals and upgrades related to the DAO and the codes to be deployed to the network must be constantly audited.

With this audited upgrade, the Treasury becomes upgradable, the risk will increase due to the nature of the upgrade
https://etherscan.io/address/0x0bc3807ec262cb779b38d65b38158acc3bfede10

To date, there has been no DAO version of forking, this is a serious system risk as it will be the first to do so.


The MEV BOT movements in the [Payer contract](https://etherscan.io/address/0xd97bcd9f47cee35c0a9ec1dc40c1269afc9e8e1d) of the project could not be understood, I could not find anything in the related documents.
https://etherscan.io/tx/0xd8e2efb006207163aab07da8050e6155616da317fe9260cf893ae22dc2f94039

## h) Competition analysis
There is no other nft project managed with Dao and sold by auction every day, the decisions of the treasury determined by proposals.


## i) Security Approach of the Project

Successful current security understanding of the project;
1 - First they did the main audit from a reputable auditing organization like Spearbit  and resolved all the security concerns in the report
2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.

What the project should add in the understanding of Security;
1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)
2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

## j) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://gist.github.com/CloudEllie/deb7d1c9c91b605555cbe604662e58cf


**Other Audit Reports :**
https://gist.github.com/hrkrshnn/25e13d04fa728309370acb6ca8ccc16b#file-nouns-md

## k) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

##  l) Project team

The group that created Nouns NFTs is called the Nounders. These are the 10 builders that initiated the project. Many are well known in the NFT space, most notably the founder of CyrpToadz [@gremplin](https://twitter.com/gremplin), [@punk4156](https://twitter.com/punk4156), and Vine co-founder Dom Hofmann [@dhof](https://twitter.com/dhof).

The full list of Nouns Founders is as follows:

[@cryptoseneca](https://twitter.com/cryptoseneca)
[@gremplin](https://twitter.com/gremplin)
[@punk4156](https://twitter.com/punk4156)
[@eboyarts](https://twitter.com/eBoyArts)
[@punk4464](https://twitter.com/punk4464)
[solimander](https://twitter.com/_solimander_)
[@dhof](https://twitter.com/dhof)
[@devcarrot](https://twitter.com/carrot_init)
[@TimpersHD](https://twitter.com/TimpersHD)
[@lastpunk9999](https://twitter.com/lastpunk9999)

These 10 founders share control of the multi-sig wallet that receives every tenth Noun released for the first five years.
## m) Other recommendations

- It is important for the future of the project that it completely passes to DAO, which eliminates the risk of centralization.

- In the project using an upgradeable format, I recommend doing minor checks on each upgrade and proposal

- NounsDAOLogicV3.sol : `propose()` function used to propose a new proposal , Sender must have delegates above the proposal threshold.This functin is very critical because it builds an important task where DAO proposals are given, however it should be tightly controlled for a recent security concern, 
the following hack is done using exactly this function, each proposal in It may even need to pass a minor inspection.

https://cointelegraph.com/news/attacker-hijacks-tornado-cash-governance-via-malicious-proposal

```solidity
packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol:
  188       */
  189:     function propose(
  190:         address[] memory targets,
  191:         uint256[] memory values,
  192:         string[] memory signatures,
  193:         bytes[] memory calldatas,
  194:         string memory description
  195:     ) public returns (uint256) {
  196:         return ds.propose(NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas), description);
  197:     }
```

## n) New insights and learning from this audit 

- I learned that NFT Projects are not just codes, they can be used in fun and social responsibility projects.

- I discovered how the DAO mechanism of an NFT project could be in a coding architecture

- There is currently $54 million in the project's treasury, the management of this amount is carried out entirely on the proposals, I learned how to code the effective method of such a treasury.

- I learned how such a large NFT project should proceed with an upgrade method, within the scope of the code.

- First time I've seen and learned how to code the DAO version of forking


### Time spent:
25 hours