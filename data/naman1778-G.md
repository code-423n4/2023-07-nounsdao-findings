## [G-01] Division by two should use bit shifting

<x> / 2 is the same as <x> >> 1. While the compiler uses the SHR opcode to accomplish both, the version that uses division incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two.        

    File: contracts/governance/NounsDAOLogicV2.sol	

    1010: uint256 center = upper - (upper - lower) / 2;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

## [G-02] Use nested *if* and, avoid multiple check combinations

Using nesting is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

    File: contracts/governance/NounsDAOLogicV2.sol	

    1026: if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

    File: contracts/governance/NounsDAOV3Votes.sol	

    240: if (
    241:    // only for votes can trigger an objection period
    242:    // we're in the last minute window
    243:    isForVoteInLastMinuteWindow &&
    244:    // first part of the vote flip check
    245:    // separated from the second part to optimize gas
    246:    isDefeatedBefore &&
    247:    // haven't turn on objection yet
    248:    proposal.objectionPeriodEndBlock == 0 &&
    249:    // second part of the vote flip check
    250:    !ds.isDefeated(proposal)
    251: ) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol

    File: contracts/governance/NounsDAOV3Admin.sol

    585: if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

## [G-03] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

    File: contracts/governance/NounsDAOLogicV2.sol	
 
    305: for (uint256 i = 0; i < proposal.targets.length; i++) {

    343: for (uint256 i = 0; i < proposal.targets.length; i++) {

    372: for (uint256 i = 0; i < proposal.targets.length; i++) {

    405: for (uint256 i = 0; i < proposal.targets.length; i++) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

    File: contracts/governance/NounsDAOV3Admin.sol

    602: for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

    603: for (uint256 j = i + 1; j < erc20tokens.length; j++) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

## [G-04] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

    File: contracts/governance/NounsDAOLogicV3.sol	

    236: function proposeBySigs(
    237:     ProposerSignature[] memory proposerSignatures,
    238:     address[] memory targets,
    239:     uint256[] memory values,
    240:     string[] memory signatures,
    241:     bytes[] memory calldatas,
    242:     string memory description
    243: ) external returns (uint256) {

    278: function updateProposal(
    279:     uint256 proposalId,
    280:     address[] memory targets,
    281:     uint256[] memory values,
    282:     string[] memory signatures,
    283:     bytes[] memory calldatas,
    284:     string memory description,
    285:     string memory updateMessage
    286: ) external {

    313: function updateProposalTransactions(
    314:     uint256 proposalId,
    315:     address[] memory targets,
    316:     uint256[] memory values,
    317:     string[] memory signatures,
    318:     bytes[] memory calldatas,
    319:     string memory updateMessage
    320: ) external {

    338: function updateProposalBySigs(
    339:     uint256 proposalId,
    340:     ProposerSignature[] memory proposerSignatures,
    341:     address[] memory targets,
    342:     uint256[] memory values,
    343:     string[] memory signatures,
    344:     bytes[] memory calldatas,
    345:     string memory description,
    346:     string memory updateMessage
    347: ) external {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol