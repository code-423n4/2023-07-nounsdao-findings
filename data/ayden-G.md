1.External functions can be marked as external to save more gas fees
   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L885
   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L902
   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L917

2.use global variable instead of storage variable or local variable to save more gas.
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L835#L85
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L867#L879

```
    function _acceptAdmin() external {
    // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
    require(
        msg.sender == pendingAdmin && msg.sender != address(0),
        'NounsDAO::_acceptAdmin: pending admin only'
    );

    // Save current values for inclusion in log
-   address oldAdmin = admin;
-   address oldPendingAdmin = pendingAdmin;

+   emit NewAdmin(admin, msg.sender);
    // Store admin with value pendingAdmin
-   admin = pendingAdmin; //@audit use local variable to save more gas.
+   admin = msg.sender;

    // Clear the pending value
+   emit NewPendingAdmin(pendingAdmin, address(0))
    pendingAdmin = address(0);

-   emit NewAdmin(oldAdmin, admin);
-   emit NewPendingAdmin(oldPendingAdmin, pendingAdmin); //@audit is address(0) to reduce storage read.

    }

```

```
    function _acceptVetoer() external {
        if (msg.sender != pendingVetoer) {
            revert PendingVetoerOnly();
        }

        // Update vetoer
-       emit NewVetoer(vetoer, pendingVetoer);
+       emit NewVetoer(vetoer, msg.sender);
        vetoer = pendingVetoer;

        // Clear the pending value
        emit NewPendingVetoer(pendingVetoer, address(0));
        pendingVetoer = address(0);
    }
```

3.delete intermediate variable to save more gas.
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L78#L86

```
    function _setImplementation(address implementation_) public {
        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
        require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');

-       address oldImplementation = implementation;
+       emit NewImplementation(implementation, implementation_);
        implementation = implementation_;

-       emit NewImplementation(oldImplementation, implementation);
    }
```

4.delete intermediate variable to save more gas.
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L162#L170
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L177#L186
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L193#L206
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L212#L223
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L229#L237
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L243#L254
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L261#L270
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L276#L295

```
    function _setVotingDelay(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingDelay) external onlyAdmin(ds) {
        require(
            newVotingDelay >= MIN_VOTING_DELAY_BLOCKS && newVotingDelay <= MAX_VOTING_DELAY_BLOCKS,
            'NounsDAO::_setVotingDelay: invalid voting delay'
        );
-       uint256 oldVotingDelay = ds.votingDelay;
+       emit VotingDelaySet(ds.votingDelay, newVotingDelay);
        ds.votingDelay = newVotingDelay;

-       emit VotingDelaySet(oldVotingDelay, newVotingDelay);
    }

```

```
    function _setVotingPeriod(NounsDAOStorageV3.StorageV3 storage ds, uint256 newVotingPeriod) external onlyAdmin(ds) {
        require(
            newVotingPeriod >= MIN_VOTING_PERIOD_BLOCKS && newVotingPeriod <= MAX_VOTING_PERIOD_BLOCKS,
            'NounsDAO::_setVotingPeriod: invalid voting period'
        );
-       uint256 oldVotingPeriod = ds.votingPeriod;
+       emit VotingPeriodSet(ds.votingPeriod, newVotingPeriod);
        ds.votingPeriod = newVotingPeriod;

-       emit VotingPeriodSet(oldVotingPeriod, newVotingPeriod);
    }
```
