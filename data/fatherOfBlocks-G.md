**script/ProposeTimelockMigrationCleanupMainnet.s.sol**
- L22 - The function run() is public, but it is not used in the whole contract, therefore gas would be saved by making it external.


**script/ProposeDAOV3UpgradeMainnet.s.sol**
- L23 - The function run() is public, but it is not used in the whole contract, therefore gas would be saved by making it external.


**script/DeployDAOV3NewContractsBase.s.sol**
- L43 - The run() function is public, but it is not used in the whole contract, therefore gas would be saved by making it external.


**script/ProposeENSReverseLookupConfigMainnet.s.sol**
- L17 - The run() function is public, but it is not used in the entire contract, therefore gas would be saved by making it external.


**script/ProposeDAOV3UpgradeTestnet.s.sol**
- L31 - The run() function is public, but it is not used in the whole contract, therefore gas would be saved by making it external.


**script/DeployDAOV3DataContractsBase.s.sol**
- L20 - The function run() is public, but it is not used in the whole contract, therefore gas would be saved by making it external.


**contracts/governance/NounsDAOLogicV2.sol**
- L620 - When a bool is being validated in a require, instead of validating like this: require(variable == false) gas is saved by validating like this: require(!variable).

- L652/653/655/670/671/673/690/691/693 - A less expensive way to do these settings is:

	emit VotingDelaySet(votingDelay, newVotingDelay);
	votingDelay = newVotingDelay;

The same can be done the same operation in the 3 functions.


**contracts/governance/NounsDAOProxy.sol**
- L82/83/85 - A less expensive way to do these settings is:

	emit NewImplementation(implementation, implementation_);
	implementation = implementation_;


**contracts/governance/NounsDAOExecutorV2.sol**
- L82 - The constant NAME is created, but it is never used, therefore it is an unnecessary waste of gas.


**contracts/governance/fork/newdao/NounsAuctionHouseFork.sol**
- L48 - The constant NAME is created, but it is never used, therefore it is an unnecessary waste of gas.


**contracts/governance/fork/newdao/token/NounsTokenFork.sol**
- L43 - NoundersCannotBeAddressZero() error is created but never used, therefore should be removed.

- L46 - The constant NAME is created, but it is never used, therefore it is an unnecessary waste of gas.


**contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol**
- L292/293/294/297/298 - The length of targets is consulted multiple times, this generates an unnecessary gas expense that would be avoided if a variable is created in memory.

- L621 - When a bool is being validated in a require, instead of validating like this: require(variable == false) gas is saved by validating like this: require(!variable).


**contracts/governance/NounsDAOLogicV3.sol**
- L79/84/89/94/99/104/109 - Public functions are created that are not used throughout the contract, therefore they could become external and spend less gas.


**contracts/governance/NounsDAOV3Votes.sol**
- L219/282 - When a bool is being validated in a require, instead of validating like this: require(variable == false) gas is saved by validating like this: require(!variable).


**contracts/governance/NounsDAOV3Admin.sol**
- L167/168/170/182/183/185/202/203/205/219/220/202/233/234/236/250/251/253/263/266/269 - A less expensive way to make these settings is:

	emit VotingDelaySet(ds.votingDelay, newVotingDelay);
	ds.votingDelay = newVotingDelay;

The same can be done the same operation in the 3 functions.


**contracts/governance/NounsDAOV3Proposals.sol**
- L968/969/970/972/973 - The length of targets is consulted multiple times, this generates an unnecessary gas expense that would be avoided if a variable was created in memory.
