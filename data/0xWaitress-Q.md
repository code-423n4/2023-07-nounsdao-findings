[L-1] native ecrecover in ERC721Checkpointable has malleability risk 

```solidity
    function delegateBySig(
        address delegatee,
        uint256 nonce,
        uint256 expiry,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public {
        bytes32 domainSeparator = keccak256(
            abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name())), getChainId(), address(this))
        );
        bytes32 structHash = keccak256(abi.encode(DELEGATION_TYPEHASH, delegatee, nonce, expiry));
        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));
        address signatory = ecrecover(digest, v, r, s);
        require(signatory != address(0), 'ERC721Checkpointable::delegateBySig: invalid signature');
        require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');
        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');
        return _delegate(signatory, delegatee);
    }
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol#L139

## Reccomendation:
use ECDSA ecrecover from Openzeppelin which enforces the lower signature to be used, or revert.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L100-L104

[L-2] quit in NounsDAOLogicV1Fork can be called with empty tokenIds and empty erc20TokensToInclude and still trigger an external call to the caller through Address.sendValue inside `timelock.sendETH`. 

```solidity
    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
        // check that erc20TokensToInclude is a subset of `erc20TokensToIncludeInQuit`
        address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
            if (!isAddressIn(erc20TokensToInclude[i], erc20TokensToIncludeInQuit_)) {
                revert TokensMustBeASubsetOfWhitelistedTokens();
            }
        }

        quitInternal(tokenIds, erc20TokensToInclude);
    }
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206-L216

## Recommendation
Consider adding a non-zero check on `ethToSend`
```solidity
+++ if(ethToSend > 0) {
// Send ETH and ERC20 tokens
        timelock.sendETH(payable(msg.sender), ethToSend);
}
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L237

[L-3] executeFork in NounsDAOV3Fork calls sendProRataTreasury which call address.sendETH to forkTreasury before updating ds.forkEndTimestamp, leading to re-entrency possible for executeFork.

executeFork has a check of `isForkPeriodActive(ds)` which depends if `ds.forkEndTimestamp > block.timestamp`. However it could re-enter itself during sendProRataTreasury which call `timelock.sendETH` which calls `address.sendValue`. This is an external call which could technically re-enter `executeFork`. Since the recipient is the forkTreasury contract so the risk is quite low. 

```solidity
    function executeFork(NounsDAOStorageV3.StorageV3 storage ds)
        external
        returns (address forkTreasury, address forkToken)
    {
        if (isForkPeriodActive(ds)) revert ForkPeriodActive();
...

    function sendProRataTreasury(
        NounsDAOStorageV3.StorageV3 storage ds,
        address newDAOTreasury,
        uint256 tokenCount,
        uint256 totalSupply
    ) internal {
        INounsDAOExecutorV2 timelock = ds.timelock;
        uint256 ethToSend = (address(timelock).balance * tokenCount) / totalSupply;

        timelock.sendETH(newDAOTreasury, ethToSend);

...
    function sendETH(address payable recipient, uint256 ethToSend) external {
        require(msg.sender == admin, 'NounsDAOExecutor::sendETH: Call must come from admin.');

        recipient.sendValue(ethToSend);

        emit ETHSent(recipient, ethToSend);
    }
```

## Recommendation
assign ds before calling sendProRataTreasury
```solidity
...
ds.forkDAOTreasury = forkTreasury;
ds.forkDAOToken = forkToken;
ds.forkEndTimestamp = forkEndTimestamp;
sendProRataTreasury(ds, forkTreasury, tokensInEscrow, adjustedTotalSupply(ds));
...
```

[QA-1] sigDigest in NounsDAOV3Proposal should ensure the expirationTiemstamp is bigger than the block.timestamp

```solidity
    function sigDigest(
        bytes32 typehash,
        bytes memory proposalEncodeData,
        uint256 expirationTimestamp,
        address verifyingContract
    ) internal view returns (bytes32) {
        bytes32 structHash = keccak256(abi.encodePacked(typehash, proposalEncodeData, expirationTimestamp));

        bytes32 domainSeparator = keccak256(
            abi.encode(DOMAIN_TYPEHASH, keccak256(bytes('Nouns DAO')), block.chainid, verifyingContract)
        );

        return ECDSA.toTypedDataHash(domainSeparator, structHash);
    }
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L1000-L1017

## Recommendation
```solidity
+++ require(expirationTimestamp > block.timestamp);
```