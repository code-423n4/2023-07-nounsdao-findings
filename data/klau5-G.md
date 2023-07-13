# Gas

## [G-01] Do status update first before doing any work

If `remainingTokensToClaim -= tokenIds.length;` underflows, it is better to try `remainingTokensToClaim -= tokenIds.length;` before doing any other works. It reverts faster and saves gas. Additionally, updating the state first is better as it follows the check-effect-interaction pattern.

```solidity
for (uint256 i = 0; i < tokenIds.length; i++) {
    uint256 nounId = tokenIds[i];
    if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();

    _mintWithOriginalSeed(msg.sender, nounId);
}

remainingTokensToClaim -= tokenIds.length;
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L149-L156](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L149-L156)

Same issue happens at `numTokensInEscrow` function.

```solidity
function returnTokensToOwner(address owner, uint256[] calldata tokenIds) external onlyDAO {
    for (uint256 i = 0; i < tokenIds.length; i++) {
        if (currentOwnerOf(tokenIds[i]) != owner) revert NotOwner();

        nounsToken.transferFrom(address(this), owner, tokenIds[i]);
        escrowedTokensByForkId[forkId][tokenIds[i]] = address(0);
    }

    numTokensInEscrow -= tokenIds.length;
}
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L116-L125](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L116-L125)

### **Recommended Mitigation**

Do `remainingTokensToClaim -= tokenIds.length;` and `numTokensInEscrow -= tokenIds.length;` first before for-loop.

## [G-02] Use cached stack variable instead of fetch array element again

Cached the array element in `nounId` but did not use it.

```solidity
for (uint256 i = 0; i < tokenIds.length; i++) {
      uint256 nounId = tokenIds[i];
      _mintWithOriginalSeed(to, nounId);

      if (tokenIds[i] > maxNounId) maxNounId = tokenIds[i];
  }
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L176](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L176)

### **Recommended Mitigation**

Use `nounId` instead of `tokenIds[i]` 

## [G-03] Reduce deployment gas by removing unnecessary and complex view functions using public variables

It is better to declare the storage variable `contractURI` as public and directly store the `ipfs://` URL, rather than using `_contractURIHash` separately and generating the URL each times on `contractURI` function. As a result, the code becomes shorter and the gas required for deployment will be reduced.

```solidity
function contractURI() public view returns (string memory) {
      return string(abi.encodePacked('ipfs://', _contractURIHash));
}

function setContractURIHash(string memory newContractURIHash) external onlyOwner {
    _contractURIHash = newContractURIHash;
}
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L190-L200](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L190-L200)

### **Recommended Mitigation**

Change the `_contractURIHash` storage variable to a public storage variable named `contractURI` and save the URL starts with `ipfs://` .

Chanage `setContractURIHash` function to `setContractURI` function.

## [G-04] Delete delegate data when cancel delegate

If the user did not delegate vote, `_delegates[delegator]` is `address(0)`. So if `_delegates[delegator]` is `address(0)`, it is regarded that the delegate feature is not being used.

```solidity
function delegates(address delegator) public view returns (address) {
    address current = _delegates[delegator];
    return current == address(0) ? delegator : current;
}
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L99-L102](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L99-L102)

If user delegate vote once and then cancel it, `_delegates[delegator]` should be set to `delegator`.

If you delete storage, you can get a gas refund. So it's better to store `address(0)` to delete storage when canceling the delegate.

```solidity
function _delegate(address delegator, address delegatee) internal {
    /// @notice differs from `_delegate()` in `Comp.sol` to use `delegates` override method to simulate auto-delegation
    address currentDelegate = delegates(delegator);

    _delegates[delegator] = delegatee;

    emit DelegateChanged(delegator, currentDelegate, delegatee);

    uint96 amount = votesToDelegate(delegator);

    _moveDelegates(currentDelegate, delegatee, amount);
}
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L209-L220](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L209-L220)

### **Recommended Mitigation**

`_delegates[delegator] = delegatee;` ⇒ `_delegates[delegator] = delegatee != delegator ? delegatee : address(0);`

## [G-05] Use unchecked, ++i, and stack variable cache in nested loop

The `checkForDuplicates` function contains an unoptimized 2-depth for-loop. It is recommended to optimize it as the optimization effect is maximized as the depth of the loop increases.

 Caching frequently used variables such as `erc20tokens.length` and `erc20tokens[i]` in a stack variable can save gas used in array access. In addition, using `++i` and `++j` instead of `i++` and `j++`, and removing the overflow check by using `unchecked`, can reduce gas consumption.

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

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L793-L801](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L793-L801)

### **Recommended Mitigation**

```solidity
function checkForDuplicates(address[] calldata erc20tokens) internal pure {
    uint256 length = erc20tokens.length;

    if (length == 0) return;

    for (uint256 i = 0; i < length - 1; ) {
        address erc20token_i = erc20tokens[i];

        for (uint256 j = i + 1; j < length; ) {
            if (erc20token_i == erc20tokens[j]) revert DuplicateTokenAddress();
            unchecked { ++j; }
        }

        unchecked { ++i; }
    }
}
```