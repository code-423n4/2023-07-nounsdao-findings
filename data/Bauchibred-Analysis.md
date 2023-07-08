# NounsDAO Analysis Report

## Approach taken in evaluating the codebase

Used a systematic approach that involved the following steps:

- Started with the docs
- Then the past audit reports
- Had a brief overview of other files not in scope of this audit just to have more base of the project itself
- Then the codes in scope line by line (most especially the updated V3 logic)
- Back to the docs to counter some notes I made while going through codebase
- Discussed a few ideas with sponsors to understand their design choice and why I think it's risky or how I can recommend a better method to implement the strategies
- Finally then back to the codebase to sum up my reports/findings

NB: I don't write my reports on the last day, but rather a few each day while working on a contest, so on the last day I must have gotten more context of the project and can then edit a few reports, finalize others or let them be, more of as the situation calls

## Anything that you learned from evaluating the codebase

Multiple things, to list a few:

- Nft borrowing platforms, learnt about this while diving through an attack path since the the snapshot is going to be known publicly way before time.
- Dynamic voting proposals (would have to spend more time to really understand this)

### Refresher on previously acquainted things

Additionally, the evaluation reinforced my understanding of topics I was already acquainted with, such as:

- How a DAO should work, i.e rootly _consensus_, this was also due to a dive to argue the current implementation of the _objectionPeriod_
- How block times being different could affect a protocol, in this case, causing a week to only last like _~ 5.5_ days
- A few basic solidity & governance concepts

## Any comments for the judge to contextualize your findings

I believe I approached the contest first as a learning opportunity and then submit issues I believe should be looked on, I spent around 20-22 hours spread around 3/4 days on the contest, had some breaks inbetween to allow the codebase settle in my mind and also think about attack vectors.
All these asides though, I would be waiting for the comments on my findings to find out what I can improve in my reports or why my thought process is wrong.
Finally, appreciate you and C4 for giving new auditors the opportunity to learn, improve as auditors and earn at the same time.


### Time spent:
21 hours