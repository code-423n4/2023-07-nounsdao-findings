## [N-01] UNDERSCORES CAN BE ADDED FOR NUMBERS
> It is a common practice to separate each 3 digits in a number by an underscore to improve code readability. Unlike the following variable Name.
```
    address public constant NOUNS_TIMELOCK_V1_MAINNET = 0x0BC3807Ec262cB779b38D65b38158acC3bfedE10;
    address public constant NOUNS_TOKEN_MAINNET = 0x9C8fF314C9Bc7F6e59A9d9225Fb22946427eDC03;
    address public constant AUCTION_HOUSE_PROXY_ADMIN_MAINNET = 0xC1C119932d78aB9080862C5fcb964029f086401e;
    address public constant LILNOUNS_MAINNET = 0x4b10701Bfd7BFEdc47d50562b76b436fbB5BdB3B;
    address public constant TOKEN_BUYER_MAINNET = 0x4f2aCdc74f6941390d9b1804faBc3E780388cfe5;
    address public constant PAYER_MAINNET = 0xd97Bcd9f47cEe35c0a9ec1dc40C1269afc9E8E1D;
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeTimelockMigrationCleanupMainnet.s.sol#L15C3-L20C88

==All state varable should fixed in all contest.==

## [N-2] Use underscores for number literals
```
    uint256 public constant FORK_DAO_VOTING_PERIOD = 36000; // 5 days
    uint256 public constant FORK_DAO_VOTING_DELAY = 36000; // 5 days
```

Like this
```
    uint256 public constant FORK_DAO_VOTING_PERIOD = 36_000; // 5 days
    uint256 public constant FORK_DAO_VOTING_DELAY = 36_000; // 5 days
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsMainnet.s.sol#L10C1-L11C69

```
- uint256 public constant FORK_DAO_QUORUM_VOTES_BPS = 1000; // 10%

+ uint256 public constant FORK_DAO_QUORUM_VOTES_BPS = 1_000; // 10%
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L23

## [N-03] Assembly Codes Specific – Should Have Comments
>Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.
>This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.
>Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```
assembly {
            chainId := chainid()
        }
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1072C9-L1074C10

```
assembly {
            if eq(success, 0) {
                revert(add(returnData, 0x20), returndatasize())
}
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L96C9-L99C14

and also this one
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L112C9-L122C14

## [N-04]  function name should be Camel case
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L79C1-L112C1

## [N-5] Empty blocks should be removed or Emit something
` receive() external payable {}`
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L1035C4-L1035C34


## [N-06] NatSpec comments should be increased in contracts
> ### Contest
>>All contest 

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## [N-07] Function writing that does not comply with the Solidity Style Guide
> ### Contest
>>All contest 

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## [G-00] Use a more recent version of solidity
In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.
31: `pragma solidity ^0.8.6;`
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L31
