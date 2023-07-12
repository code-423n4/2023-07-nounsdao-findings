## Gas Optimizations

| Number                                                                    | Issue                                                             | Instances |
| ------------------------------------------------------------------------- | :---------------------------------------------------------------- | :-------: |
| [G-01](#redundant-zero-address-check)                                     | Redundant zero address check                                      |     3     |
| [G-02](<#replace-`type(uint256).max`-with-hexadecimal-representation>)    | Replace `type(uint256).max` with hexadecimal representation       |     7     |
| [G-03](cheaper-input-valdiations-should-come-before-expensive-operations) | Cheaper input valdiations should come before expensive operations |     4     |
| [G-04](can-use-`unchecked`-in-`bps2Uint`-function)                        | Can use `unchecked` in `bps2Uint` function                        |     4     |
| [G-05](Replace-`i++`-with-`++i`)                                          | Replace `i++` with `++i`                                          |     9     |

## [G-01] Redundant zero address check

Remove useless zero address check.

_Gas savings: Avg. 22 per_

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L279

```diff
        require(
-            msg.sender == ds.pendingAdmin && msg.sender != address(0),
+            msg.sender == ds.pendingAdmin,
            'NounsDAO::_acceptAdmin: pending admin only'
        );
```

Total Instances: `3`

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L856

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L733

## [G-02] Replace `type(uint256).max` with hexadecimal representation

Replace `type(uint256).max` with its hexadecimal representation as it is cheaper
same goes for `type(uint16).max` and `type(uint32).max`

_Gas savings: Avg. 22 per_

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L116

```diff
        i++;
        targets[i] = STETH_MAINNET;
        values[i] = 0;
        signatures[i] = 'approve(address,uint256)';
-        calldatas[i] = abi.encode(erc20Transferer, type(uint256).max);
+        calldatas[i] = abi.encode(erc20Transferer, 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
```

Total Instances: `7`

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1084
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1079
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L595
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L151
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L146
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeTestnet.s.sol#L123

## [G-03] Cheaper input valdiations should come before expensive operations

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L220

```diff
+       require(targets.length != 0, 'NounsDAO::propose: must provide actions');
        require(
            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
            'NounsDAO::propose: proposer votes below proposal threshold'
        );
        require(
            targets.length == values.length &&
                targets.length == signatures.length &&
                targets.length == calldatas.length,
            'NounsDAO::propose: proposal function information arity mismatch'
        );
-        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
```

Total Instances: `4`

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L972
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L989C4-L989C4
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L297

## [G-04] Can use `unchecked` in `bps2Uint` function

_Gas savings: Avg. 400 per_

### Proof of concept:

The maximum possible `bps` argument can be `10_000` or in other words 100% as 1 basis point means 0.01%

The maximum `number` argument that we can use without overflowing is somewhere near `2**251` which is proven from the following formula:

-   `(number * 10_000) <= 2**256-1` - the number multiplied by the biggest possible bps should be under the uint256.max limit
-   Dividing both sides by 10_000 we get: `number <= (2^256 - 1) / 10_000`
-   Now we can easily calculate the value which number should be less or equal: `1157920892373161954235709850086879078532699846656405640394575` or `log2(n) ~= 251.9999....` _(close to `2**251`)_

The function `bps2Uint` is used in the following way `bps2Uint(bps, totalSupply()` where `totalSupply()` is the total amount of minted Nouns Tokens which based on the current implementation should be **impossible** to go over the number of `2**251`. **In other words this optimization is safe**.

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1066

```diff
function bps2Uint(uint256 bps, uint256 number) internal pure returns (uint256) {
+        unchecked {
            return (number * bps) / 10000;
+        }
    }
```

Total Instances: `4`

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L1015-L1017
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L785-L787
-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3DynamicQuorum.sol#L161-L163

## [G-05] Replace `i++` with `++i`

```diff
-       i++;
+       ++i;
        targets[i] = address(daoProxy);
        values[i] = 0;
        signatures[i] = '_setImplementation(address)';
        calldatas[i] = abi.encode(daoV3Implementation);
```

Total Instances: `9`

-   https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/script/ProposeDAOV3UpgradeMainnet.s.sol#L82-L139
