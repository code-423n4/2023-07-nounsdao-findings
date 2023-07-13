## About Nouns DAO
NounsDAO is a community driven NFT project, where one NFT is auctioned off a day, forever.  Each NFT is completely unique, making them all equally rare. With this, the token holders are able to vote on projects of their choosing to delegate a percentage of the Treasury.  This is useful for users outside of the DAO, because they are able to create a pitch for a project of theirs to make them eligible to compete in these voting contests.  If the user were to win, then they receive funding for their project.

The amount of votes a token holder is allowed is a direct 1:1 correlation with the amount of Nouns NFTs they hold.  

## Key Takeaways

The DAO offers the ability for token holders to make proposals, as well as vote on those proposals.  The determining factor on whether or not a proposal passes and is able to be queued for execution is based on whether or not on if the proposal is defeated.  A proposal becomes defeated if the `forVotes` are less than or equal to the `againstVotes` OR if the `forVotes` are less than the `quorumVotes` for a given proposalId.  This allows for the community to fairly govern the DAO.

Furthermore, with the upgrade of Nouns from V2 to V3, if 20% of the members of the DAO choose to fork away and create their own DAO, then they are able to do so.  The process involves 1. Escrowing their Nouns NFT(s).  What this means is that they transfer their NFT from their wallet to an escrow contract.  If the fork threshold is met, then the owners of the escrowed NFTs are able to successfully fork.  The owners are also able to recover their NFT back if the fork has not yet been executed.

1. Once the execution of the fork is successful, then owners of the escrowed NFTs can go to NounsTokenFork.sol and can ‘claim’ their NFT back.  What this really means is that they ‘re-mint’ their NFT with the same exact traits.  Members of the original DAO who want to join this fork but did not escrow are given a window where they are able to join the forked DAO.  Once the window has passed, the ability to join is revoked.
2. The old NFT that remains in the escrowed contract is no longer able to be withdrawn by the old owner, and can only be withdrawn by the admin, either to the timelock contract or to any arbitrary address, keeping in mind that the latter will increase the tokenSupply.

Above all, the upgrades to the DAO seem to accomplish the goals of the DAO accurately and securely.

Through thoroughly reading this codebase, I learned many interesting things that I will apply to my own projects in the future.  The two biggest learnings are the 1. use of a storage contract rather than storing the data directly on the logic contract, and 2. the use of a logic contract to hold all of the functions and making those functions call to libraries where the actual functionality exists.

## Approach
Prior to the contest starting, I researched the Nouns project and watched Youtube videos, explaining exactly what Nouns were and the goals they were trying to accomplish.  Upon the start time of the contest, I immediately dove in and begun reading the codebase, starting with `NounsDAOLogicV3` because this was the contract that all users were going to be interacting with.  For the first few days, I did a broad read-through, following the function calls to their respective libraries.  After a few days of still not really understanding how everything worked, I created a diagram on Miro, which helped put the forking process and proposal process into place.  After I fully understand how everything worked under the hood, I began running Foundry tests, testing my theories that resulted in many false-positives.  However, these tests led to other theories and ultimately a few findings.

## Time spent

I tried to allocate 8 hours a day to this contest, however probably averaged closer to 6.5 hours, totaling in approximately 65 hours spent on this audit.

### Time spent:
65 hours