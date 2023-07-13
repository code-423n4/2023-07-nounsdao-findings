# Findings Summary

| ID     | Title                                                   | Severity |
| ------ | ------------------------------------------------------- | -------- |
| [L-01] | openzeppelin contracts should update to the new version | Low      |
| [L-02] | _refundGas target is different in v2 and v3 contract    | Low      |

# Detailed Findings

# [L-01] openzeppelin contracts should update to the new version

## Description

The current version of openzeppelin contracts has some high risks of vulnerability:
- Initializer reentrancy: https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3006
- ERC165Checker supportsInterface: https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3552
- Signature malleability attack: https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3610

I submit the signature vulnerability in a separate issue, although the other issues were not exploited scenarios in the current codebase, since the contract is upgradable, future changes may introduce attack vectors and need to be upgraded to the new version as soon as possible.

## Recommendations

openzeppelin contracts should update to the new version

# [L-02] _refundGas target is different in v2 and v3 contract

## Description

```solidity
    function _refundGas(uint256 startGas) internal {
        unchecked {
            uint256 balance = address(this).balance;
            if (balance == 0) {
                return;
            }
            uint256 basefee = min(block.basefee, MAX_REFUND_BASE_FEE);
            uint256 gasPrice = min(tx.gasprice, basefee + MAX_REFUND_PRIORITY_FEE);
            uint256 gasUsed = min(startGas - gasleft() + REFUND_BASE_GAS, MAX_REFUND_GAS_USED);
            uint256 refundAmount = min(gasPrice * gasUsed, balance);
            (bool refundSent, ) = tx.origin.call{ value: refundAmount }('');
            emit RefundableVote(tx.origin, refundAmount, refundSent);
        }
    }

    function _refundGas(uint256 startGas) internal {
        unchecked {
            uint256 balance = address(this).balance;
            if (balance == 0) {
                return;
            }
            uint256 basefee = min(block.basefee, MAX_REFUND_BASE_FEE);
            uint256 gasPrice = min(tx.gasprice, basefee + MAX_REFUND_PRIORITY_FEE);
            uint256 gasUsed = min(startGas - gasleft() + REFUND_BASE_GAS, MAX_REFUND_GAS_USED);
            uint256 refundAmount = min(gasPrice * gasUsed, balance);
            (bool refundSent, ) = msg.sender.call{ value: refundAmount }('');
            emit RefundableVote(msg.sender, refundAmount, refundSent);
        }
    }
```

In v2 the gas compensation will be returned to `tx.origin`, but changed to `msg.sender` in v3. Inconsistent targets may exceed user expectations.     
For example, when a user interacts with protocol by a contract in v2, the EOA receives gas compensation normally. When they tried to do so on v3, they found that gas compensation was sent to the contract, which could be a third-party payment or not implemented withdrawal function, preventing the user from receiving the gas compensation.

## Recommendations

The target should be the same as on v2, or specified in the documentation.
