`NounsDAOLogicV1Fork.initialize()` does not have validation check for `votingPeriod`, `votingDelay` as with `_setVotingDelay()`, `_setVotingPeriod()`

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L188-L189C

```solidity
    function initialize(
        address timelock_,
        address nouns_,
        uint256 votingPeriod_,
        uint256 votingDelay_,
        uint256 proposalThresholdBPS_,
        uint256 quorumVotesBPS_,
        address[] memory erc20TokensToIncludeInQuit_,
        uint256 delayedGovernanceExpirationTimestamp_
    ) public virtual {
        __ReentrancyGuard_init_unchained();
        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
        require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
        require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');

+        require(
+            votingDelay_>= MIN_VOTING_DELAY && votingDelay_<= MAX_VOTING_DELAY,
+            'NounsDAO::_setVotingDelay: invalid voting delay'
+        );
+        require(
+            votingPeriod_>= MIN_VOTING_PERIOD && votingPeriod_<= MAX_VOTING_PERIOD,
+            'NounsDAO::_setVotingPeriod: invalid voting period'
+        );
...
        votingPeriod = votingPeriod_;
        votingDelay = votingDelay_;
```