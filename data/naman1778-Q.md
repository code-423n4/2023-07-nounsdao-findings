## [N-01] Typo

    File: contracts/governance/NounsDAOV3Admin.sol
    
    /// @audit threhsold
    105: /// @notice Emitted when the threhsold for forking is set

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

## [N-02] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

    File: contracts/governance/NounsDAOLogicV2.sol	
 
    145: if (msg.sender != admin) {
    146:     revert AdminOnly();
    147: }

    394: if (msg.sender != vetoer) {
    395:     revert VetoerOnly();
    396: }

    645: if (msg.sender != admin) {
    646:     revert AdminOnly();
    647: }

    663: if (msg.sender != admin) {
    664:     revert AdminOnly();
    665: }

    682: if (msg.sender != admin) {
    683:     revert AdminOnly();
    684: }

    703: if (msg.sender != admin) {
    704:     revert AdminOnly();
    705: }

    733: if (msg.sender != admin) {
    734:     revert AdminOnly();
    735: }

    760: if (msg.sender != admin) {
    761:     revert AdminOnly();
    762: }

    788: if (msg.sender != admin) {
    789:     revert AdminOnly();
    790: }

    819: if (msg.sender != admin) {
    820:     revert AdminOnly();
    821: }

    838: require(msg.sender == admin, 'NounsDAO::_setPendingAdmin: admin only');

    856: require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

    877: if (msg.sender != vetoer) {
    878:    revert VetoerOnly();
    879: }

    887: if (msg.sender != pendingVetoer) {
    888:    revert PendingVetoerOnly();
    889: }

    906: require(msg.sender == vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol    

    File: contracts/governance/NounsDAOProxy.sol	

    79: require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol

    File: contracts/governance/NounsDAOLogicV3.sol	

    151: if (msg.sender != ds.admin) revert AdminOnly();

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol

    File: contracts/governance/NounsDAOV3Admin.sol

    278: require(
    279:     msg.sender == ds.pendingAdmin && msg.sender != address(0),
    280:     'NounsDAO::_acceptAdmin: pending admin only'
    281: );

    302: if (msg.sender != ds.vetoer) {
    303:     revert VetoerOnly();
    304: }

    315: if (msg.sender != ds.pendingVetoer) {
    316:     revert PendingVetoerOnly();
    317: }

    334: require(msg.sender == ds.vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

## [N-03] Lack of address(0) checks in the constructor

Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

    File: script/DeployDAOV3NewContractsBase.s.sol

    36: daoProxy = NounsDAOLogicV1(payable(_daoProxy));

    37: timelockV1 = INounsDAOExecutor(_timelockV1);

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3NewContractsBase.s.sol

    File: script/ProposeDAOV3UpgradeTestnet.s.sol

    26: timelockV1 = timelockV1_;

    27: auctionHouseProxy = auctionHouseProxy_;
    
    28: stETH = stETH_;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol

    File: script/DeployDAOV3DataContractsBase.s.sol

    16: daoProxy = NounsDAOLogicV1(payable(_daoProxy));
 
    17: timelockV2Proxy = _timelockV2Proxy;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/DeployDAOV3DataContractsBase.s.sol

    File: contracts/governance/NounsDAOProxy.sol	

    71: admin = admin_;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
 
    File: contracts/governance/fork/ForkDAODeployer.sol	

    72: tokenImpl = tokenImpl_;

    73: auctionImpl = auctionImpl_;
    
    74: governorImpl = governorImpl_;
    
    75: treasuryImpl = treasuryImpl_;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol

## [N-04] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

    File: contracts/governance/NounsDAOLogicV2.sol	
 
    1072: assembly {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/Nouns   

    File: contracts/governance/NounsDAOProxy.sol	

    96: assembly {

    112: assembly {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol

## [N-05] Inconsistent Solidity Versions

Except the below mentioned contracts all other contracts are using 0.8.15 version of solidity.

    File: contracts/utils/ERC20Transferer.sol	

    16: pragma solidity ^0.8.16;

1084710f6a682955e80dcb/packages/nouns-contracts/contracts/utils/ERC20Transferer.sol

    File: contracts/governance/NounsDAOLogicV2.sol	
 
    53: pragma solidity ^0.8.6;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol    

    File: contracts/governance/NounsDAOProxy.sol	

    36: pragma solidity ^0.8.6;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol

    File: contracts/governance/NounsDAOExecutorProxy.sol	

    18: pragma solidity ^0.8.19;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorProxy.sol

    File: contracts/governance/NounsDAOLogicV3.sol	

    56: pragma solidity ^0.8.19;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol

    File: contracts/governance/NounsDAOV3Votes.sol	

    18: pragma solidity ^0.8.19;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol

    File: contracts/governance/NounsDAOV3Admin.sol

    17: pragma solidity ^0.8.19;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

    File: contracts/governance/fork/ForkDAODeployer.sol	

    18: pragma solidity ^0.8.19;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/ForkDAODeployer.sol

## [N-06] Consider addings checks for signature malleability

Use OpenZeppelin’s ECDSA contract rather than calling ecrecover() directly

    File: contracts/governance/NounsDAOLogicV2.sol	
 
    599: address signatory = ecrecover(digest, v, r, s);

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/Nouns

    File: contracts/governance/NounsDAOV3Votes.sol	

    169: address signatory = ecrecover(digest, v, r, s);

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol

## [N-07] Remove code used for testing

    File: script/ProposeTimelockMigrationCleanupMainnet.s.sol	

    38: console.log('Proposed proposalId: %d', proposalId);

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeTimelockMigrationCleanupMainnet.s.sol

    File: script/ProposeDAOV3UpgradeMainnet.s.sol	

    49: console.log('Proposed proposalId: %d', proposalId);

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol

    File: ProposeENSReverseLookupConfigMainnet.s.sol

    24: console.log('Proposed proposalId: %d', proposalId);

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeENSReverseLookupConfigMainnet.s.sol

    File: script/ProposeDAOV3UpgradeTestnet.s.sol	

    57: console.log('Proposed proposalId: %d', proposalId);

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol

## [N-08] Numeric values having to do with time should use time units for readability

There are [units](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#time-units) for seconds, minutes, hours, days, and weeks, and since they’re defined, they should be used

    File: contracts/governance/NounsDAOV3Admin.sol	

    124: uint256 public constant MIN_VOTING_DELAY_BLOCKS = 1;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

## [N-09] Large multiples of ten should use scientific notation (e.g. *1e6*) rather than decimal literals (e.g. *1000000*), for readability

    File: script/ProposeDAOV3UpgradeMainnet.s.sol	

    /// @audit 10000
    14: uint256 public constant ETH_TO_SEND_TO_NEW_TIMELOCK = 10000 ether;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol

    File: contracts/governance/NounsDAOLogicV2.sol	
 
    /// @audit 200_000
    101: uint256 public constant MAX_REFUND_GAS_USED = 200_000;

    /// @audit 10000
    967: uint256 againstVotesBPS = (10000 * againstVotes) / totalSupply;

    /// @audit 10000
    1067: return (number * bps) / 10000;

    /// @audit 200_000
    60: uint256 public constant MAX_REFUND_GAS_USED = 200_000;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/Nouns

## [N-10] Admin Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

-- splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
-- clearly documenting the functions and implementations the owner can change,
-- documenting the risks associated with privileged users and single points of failure, and
-- ensuring that users are aware of all the risks associated with the system.