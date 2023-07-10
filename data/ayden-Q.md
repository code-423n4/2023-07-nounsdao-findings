1.proposal can be canceled mutile times maybe lead to dos attack.
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L359#L380

```solidity
function testQueue() public {
  vm.startPrank(proposer);
  daoProxy.cancel(proposalId);
  daoProxy.cancel(proposalId);
}

```