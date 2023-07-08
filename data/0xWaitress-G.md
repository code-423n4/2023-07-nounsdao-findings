[G-1] implement checkForDuplicates using a temporary Enumerable Set can reduce the O(n^2) operation to O(n)

```solidity
    function checkForDuplicates(address[] calldata erc20tokens) internal pure {
        if (erc20tokens.length == 0) return;

        for (uint256 i = 0; i < erc20tokens.length - 1; i++) {
            for (uint256 j = i + 1; j < erc20tokens.length; j++) {
                if (erc20tokens[i] == erc20tokens[j]) revert DuplicateTokenAddress();
            }
        }
    }
```

## Recommendation
using a temporary Openzeppelin Enumerable Set instead of enumerating the list through O(n^2)