# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       |initialize() can be called again if timelock is set to 0|1|
|L-02       |Admin can be set directly using `_setTimelocksAndAdmin()`|1|

## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | Typos | 1 |
|N-02       | Missing max value for `lastMinuteWindowInBlocks` | 1 |
|N-03       | Be consistent with using if and require | 1 |
|N-04       | Ethereum gas costs might change in the future | 1 |
# Details
## Low Risk
## [L-01] initialize() can be called again if timelock is set to 0
The [`initialize()`](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L141-L172) function in NounsDAOLogicV3 doesn't have an initializer modifier. Instead it checks `address(ds.timelock) != address(0)`. This would be a good practice to only initialize once. However the admin can set the timelock address in [`_setTimelocksAndAdmin`](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L566-L577). Since there is no 0 address check, the admin can set the timelock to 0 by mistake. When that happens, the NounsDAOLogicV3 can be initialized again.
```solidity
    function initialize(
        address timelock_,
        address nouns_,
        address forkEscrow_,
        address forkDAODeployer_,
        address vetoer_,
        NounsDAOParams calldata daoParams_,
        DynamicQuorumParams calldata dynamicQuorumParams_
    ) public virtual {
        if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
        ...
    }
```
```solidity
    function _setTimelocksAndAdmin(
        NounsDAOStorageV3.StorageV3 storage ds,
        address timelock,
        address timelockV1,
        address admin
    ) external onlyAdmin(ds) {
        ds.timelock = INounsDAOExecutorV2(timelock);
        ds.timelockV1 = INounsDAOExecutor(timelockV1);
        ds.admin = admin;

        emit TimelocksAndAdminSet(timelock, timelockV1, admin);
    }
```
## [L-02] Admin can be set directly using `_setTimelocksAndAdmin()`
The NounsDAOExecutor has a 2 step admin transfer with the functions [`setPendingAdmin()`](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L97-L105) and [`acceptAdmin()`](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L89C14-L95). A 2-step ownership pattern is a good practice. However this can be bypassed when the admin calls [`_setTimelocksAndAdmin()`](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L566-L577). This function allows to set the admin directly, which eliminates the purpose of the other 2 functions. 
```solidity
    function _setTimelocksAndAdmin(
        NounsDAOStorageV3.StorageV3 storage ds,
        address timelock,
        address timelockV1,
        address admin
    ) external onlyAdmin(ds) {
        ds.timelock = INounsDAOExecutorV2(timelock);
        ds.timelockV1 = INounsDAOExecutor(timelockV1);
        ds.admin = admin;

        emit TimelocksAndAdminSet(timelock, timelockV1, admin);
    }
```
Consider removing the option to set an admin directly.
## Non critical
## [N-01] Typos
- [NounsDAOLogicV2.sol](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L19) should be NounsDAOLogicV3.sol
## [N-02] Missing max value for `lastMinuteWindowInBlocks`
When setting the [lastMinuteWindowInBlocks](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L229-L237), there is no min or max settable value. Missing a min value isn't a problem in this case. However missing a max value can lead to really long waiting times if it's set wrong. Consider adding a max value of a few days.
```solidity
    function _setLastMinuteWindowInBlocks(NounsDAOStorageV3.StorageV3 storage ds, uint32 newLastMinuteWindowInBlocks)
        external
        onlyAdmin(ds)
    {
        uint32 oldLastMinuteWindowInBlocks = ds.lastMinuteWindowInBlocks;
        ds.lastMinuteWindowInBlocks = newLastMinuteWindowInBlocks;

        emit LastMinuteWindowSet(oldLastMinuteWindowInBlocks, newLastMinuteWindowInBlocks);
    }
```
## [N-03] Be consistent with using if and require
The NounsDAOV3Admin is not consistent when setting the variables and checking the min and max values. Some functions use if and some require. It can be confusing when you switch them up so it's best to always use the same.
```
L387:
    require(
        newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
        'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
    );
L438:
    if (
        newMinQuorumVotesBPS < MIN_QUORUM_VOTES_BPS_LOWER_BOUND ||
        newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND
    ) {
        revert InvalidMinQuorumVotesBPS();
    }
```

## [N-04] Ethereum gas costs might change in the future
When a vote gets refunded, there is a [`REFUND_BASE_GAS`](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L57) of 36000 gas. This includes 7K for ETH transfer and 29K for general transaction overhead. However gas costs might change in the future. Leading to more or less refunding than necessary. A mitigation is to remove the so the admin can change the value.