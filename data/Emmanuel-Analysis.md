## Approach taken in Evaluating the Codebase
- Visited the [website](https://nouns.wtf/) to learn about examples of proposals.
- Read Nouns C4 contest details
- Dived into codebase
- Checked if bugs found in previous audit were properly fixed
- Conducted a manual review of the codebase

## Anything that you learned from evaluating the codebase
- There was no function to updateProposalDescriptionBySigs
- There were some minor inconsistencies:
    - updateProposals, updateProposalTransactions does not use `ProposalTxs` struct for its input parameters
    - `getBlockTimestamp` was not used in many places
    - return values from some external calls were not checked.(Although aren't exploitable)
    - cancelling of a signature only has effect before proposal creation.
    - NounsAuctionHouseFork#_settleAuction: if auction is paused, and there is no bidder for current auction, calls will revert because there is an attempt to burn tokens when minter has been changed(auction is paused because minter was changed)

## Any comments for the judge to contextualize your findings
None


### Time spent:
34.6 hours