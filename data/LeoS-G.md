## Summary
|        | Issue | Instances | Gas Saved |
|--------|-------|-----------|-----------|
|[G-01]|Use named return|15|35-169|
|[G-02| Ternary over  `if ... else`|3|39|

### [G-01] Use named return
Using the named return whenever possible saves gas by avoiding a return and the double declaration of a variable.

*13 instances*
*Save up to **5-13 gas per call*** (depending on the complexity of the function)

-   [DeployDAOV3NewContractsBase.s.sol#L96-L106](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L96-L106)
-   [ERC20Transferer.sol#L32-L36](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L32-L36)
- [NounsDAOLogicV2.sol#L611-L638](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L611-L638)
- [NounsDAOLogicV2.sol#L818-L829](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L818-L829)
- [NounsDAOV3Votes.sol#L210-L264](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L210-L264)
- [NounsDAOV3Votes.sol#L275-L293](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L275-L293)
- [ForkDAODeployer.sol#L169-L172](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol#L169-L172)
- [NounsDAOV3Proposals.sol#L160-L187](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L160-L187)
- [NounsDAOV3Proposals.sol#L197-L210](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L197-L210)
- [NounsDAOV3Proposals.sol#L220-L259](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L220-L259)
- [NounsAuctionHouseFork.sol#L272-L275](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L272-L275)
- [ERC721CheckpointableUpgradeable.sol#L99-L102](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L99-L102)
- [ERC721CheckpointableUpgradeable.sol#L163-L166](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L163-L166)

For instance, the code block below may be refactored as follows:
-   [ERC20Transferer.sol#L32-L36](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol#L32-L36)
 ``` diff
-   function transferEntireBalance(address token, address to) external returns (uint256) {
+	function transferEntireBalance(address token, address to) external returns (uint256 balance) {
-        uint256 balance = IERC20(token).balanceOf(msg.sender);
+	     balance = IERC20(token).balanceOf(msg.sender);
          IERC20(token).transferFrom(msg.sender, to, balance);
-        return balance;
    }
```

### [G-02] Ternary over  `if ... else`
Replacing an `if-else` statement with the ternary operator can save gas.

For instance, the code block below may be refactored as follows:
-   [DeployDAOV3NewContractsBase.s.sol#L73-L77](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L73-L77)
``` diff
-if (deployTimelockV2Harness) {
-    timelockV2Impl = new NounsDAOExecutorV2Test();
- } else {
-     timelockV2Impl = new NounsDAOExecutorV2();
- }
+ timelockV2Impl = deployTimelockV2Harness ? new NounsDAOExecutorV2Test() : new NounsDAOExecutorV2();
```
*3 instances*
*Save up to **13 gas per call***
-   [DeployDAOV3NewContractsBase.s.sol#L73-L77](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol#L73-L77) (*variable set*)
-   [NounsDAOExecutorV2.sol#L189-L193](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L189-L193) (*variable set*)
-   [NounsDAOV3Proposals.sol#L525-L529](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L525-L529) (*return*)