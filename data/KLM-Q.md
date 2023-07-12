# Report: Redundant Variable in NounsAuctionHouse.sol

## Overview

This report addresses the presence of a redundant variable named `value` in the `NounsAuctionHouse.sol` contract at line 258. This redundancy could potentially introduce errors when utilizing the `safeTransferETH` method.

## Details

In the `NounsAuctionHouse.sol` contract, at line 258, a redundant variable named `value` is used as follows:
```solidity
function _safeTransferETH(address to, uint256 value) internal returns (bool) {
        (bool success, ) = to.call{ value: value, gas: 30_000 }(new bytes(0));
        return success;
    }
```
## Potential Issue

Note that the `value` in a `call` defines the value that is sent, using it as an argument without modifying its spelling such as `_value` could cause the function of `safeTransferETH` to revert.

## Recommendation

Change the parameter `value`to `amount` to prevent transaction bugs.