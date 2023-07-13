## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | Misleading comment on timelock_ contract address | 1 |
| [L&#x2011;02] | Missing events in NounsDAOExecutorV2.initialize() function | 1 |
| [L&#x2011;03] | Prevent constructor from setting admin address as address(0) | 1 |
| [L&#x2011;04] | Add event for NounsDAOLogicV1Fork.initialize() | 1 |

### [L&#x2011;01]  Misleading comment on timelock_ contract address
In NounsDAOLogicV1Fork.sol, timelock_ address [comment](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L156) is misleading with respect to function used in contract. It says,
```Solidity
156     * @param timelock_ The address of the NounsDAOExecutor
```
However, NounsDAOExecutor do not have sendERC20() and sendETH() functions which are used in NounsDAOLogicV1Fork.sol. Therefore the comment needs to be corrected with below recommendation.

### Recommended Mitigation steps
```Solidity
-     * @param timelock_ The address of the NounsDAOExecutor
+     * @param timelock_ The address of the NounsDAOExecutorV2
```

### [L&#x2011;02]  Missing events in initialize() function
In NounsDAOExecutorV2.sol, initialize() function can only be called once. The state variables are set inside the initialize() must emit event for everyones information and verifications. It should log events. Add event indicating the delay and admin state variables.

There is 1 instance of this issue:
In NounsDAOExecutorV2.sol at [L95](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L95-L101),

```Solidity
File: nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol

    function initialize(address admin_, uint256 delay_) public virtual initializer {
        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');
        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');

        admin = admin_;
        delay = delay_;
    }
```

### [L&#x2011;03]  Prevent constructor from setting admin address as address(0)
admin address can not be address(0). Add zero address check inside constructor.

```Solidity
File: nouns-contracts/contracts/governance/NounsDAOExecutor.sol

    constructor(address admin_, uint256 delay_) {
        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');
        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');

        admin = admin_;
        delay = delay_;
    }
```

### Recommended Mitigation steps

```Solidity
File: nouns-contracts/contracts/governance/NounsDAOExecutor.sol

    constructor(address admin_, uint256 delay_) {
+        require(admin != address(0), "invalid address");
        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');
        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');

        admin = admin_;
        delay = delay_;
    }
```
### [L&#x2011;04]  Add event for NounsDAOLogicV1Fork.initialize()
In NounsDAOLogicV1Fork.initialize(), it doesn not emit the event which must be emitted. Events are the cheapest to store data on blockchain. Recommend to add same in initialize function.

There is 1 instance of this issue:
In NounsDAOLogicV1Fork.initialize() at [L-165](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L165-L194)
