# NounsAuctionHouseFork.nouns type is incorrect
[NounsAuctionHouseFork.nouns](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L51) is supposed to be [INounsTokenFork](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/INounsTokenFork.sol#L24C11-L58) instead of `INounsToken`

# NounsDAOV3Proposals.proposeBySigs doesn't check if `proposerSignatures` contains duplicated elements
Function [NounsDAOV3Proposals.proposeBySigs](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L220C14-L225) doesn't check if the `proposerSignatures` contains duplicated elements, even [verifySignersCanBackThisProposalAndCountTheirVotes](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L830) can prevent the duplicated proposerSignatures by calling `checkNoActiveProp`, but if `verifySignersCanBackThisProposalAndCountTheirVotes` changes in future, the function might cause some issue

# Gas griefing/theft is possible on unsafe external call
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L196
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L273
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1043
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L824
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L173

# NounsAuctionHouseFork.minBidIncrementPercentage should add check to make sure `minBidIncrementPercentage` is between 1~100
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L200-L204