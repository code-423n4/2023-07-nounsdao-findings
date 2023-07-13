# Introduction

NounsDAO is one of the pioneers of the NFT-based community-driven platforms on Ethereum. With their amazing slogan 'One Noun, every day, forever' NounsDAO foundation explain that every 24 hours, an ERC721 type non-fungible token is minted and put in a public auction to be claimed by new owners. One of the important features of this protocol is that each unique NFT represents a voting power unit that a token holder can do governance-related tasks.

# Codebase evaluating approach
I firstly dive into external contracts and then focus on the mechansims of the whole architecture. As of my background I put 60% of my time for testing and evaluating the upgradeable contracts.

# Centralization risks
I think the executor contracts have some centralization risks by default. Also, as of passing a malicious transaction in a proposal, the admin of these contracts could be changed.


# Novelty
1 - More fair voting period consideration for new proposals in the recent version
2 - Compound's Governor Bravo Delegator governance mechanism is selected for NounsDAO proxy contracts
3 - UUPS type upgradeable proxy mechanism is adopted for the recent version of the executor contract
4 - Fork mechanism considered for token holders to exit in the case of an upgrade
5 - Sending assets as wrapped ETH to getting out of the Denial-of-Service attack in the case of a non-successful call to a malicious bidder contract.
6 - Considering every peripheral contracts as library inside the NounsDAO logic contracts

# Systemic risks
NounsDAO's recent versions which are considered to be upgradeable, may have some potential risks dealing with the notion of upgradeability. One of these issues may be uninitialized implementation attack vectors which is possible in the executor contract.
The other risk is inside the propose-related functions which don't have any payable modifier. Compared with the Compound's contracts, the NounsDAO contracts lack of such modifier.





# Resources
https://nouns.wtf/
https://nouns.center/


### Time spent:
34 hours