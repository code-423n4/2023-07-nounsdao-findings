## [L-01] ```returnTokensToOwner``` should emit and event for ```numTokensInEscrow -= tokenIds.length``` 
#### Description
Missing events for critical arithmetic parameters.
#### Impact
Without emitting an event for ```numTokensInEscrow``` it would be difficult to track ```numTokensInEscrow``` reduced by the number of ```tokenIds``` especially in off-chain procedures, eventually results in wrong calculation in ```numTokensInEscrow```
#### Tools Used 
Encephalon aka Brain
#### Vulnerable Code Link 
[NounsDAOForkEscrow.sol#L116-L125](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L116-L125)
#### Recommended Mitigation Steps
Event should be defined in the contract i.e
``` solidity
event TokensReturnedToOwner(address indexed owner, uint256 numTokens);
```
And should be emit in the function in a way i.e
``` solidity
function returnTokensToOwner(address owner, uint256[] calldata tokenIds) external onlyDAO {
        for (uint256 i = 0; i < tokenIds.length; i++) {
            if (currentOwnerOf(tokenIds[i]) != owner) revert NotOwner();

            nounsToken.transferFrom(address(this), owner, tokenIds[i]);
            escrowedTokensByForkId[forkId][tokenIds[i]] = address(0);
        }

        numTokensInEscrow -= tokenIds.length;
        // Emit that event for tracking changes
        emit TokensReturnedToOwner(owner, tokenIds.length);
    }
```
## [L-02] Using ```block.timestamp``` for making decisions regarding governance could be problematic
#### Description 
usage of ```block.timestamp```. ```block.timestamp``` can be manipulated by miners.
#### Impact 
Miners can manipulate the block timestamp to some extent. Which could Effect ```checkGovernanceActive()``` function
#### Tools Used 
Solidity Visual Developer 
#### Vulnerable Code Link 
[NounsDAOLogicV1Fork.sol#L377-L378](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L377-L378)
[NounsDAOLogicV1Fork.sol#L377-L378](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L381-L382)
#### Recommended Mitigation Step
Avoid relying on block.timestamp.
## [L-03] Nouns will not be able to be ```transferred``` once the ```block.number``` passes ```type(uint32).max```
#### Description 
Considering This Issue From Past Report Of Nouns DAO And Not Sure The Mitigation Is Done So Submitting Again, Feedback if mitigation is implemented because I have seen no changes. 
#### Impact 
While this currently equates to ~1260 years, if there’s a hard fork which makes block times much more frequent (e.g. to compete with Solana), then this limit may be reached much faster than expected, and transfers and delegations will remain stuck at their existing settings
#### Tools Used
Past Report
#### Vulnerable Code Link 
[NounsDAOLogicV2.sol#L984](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L984)


