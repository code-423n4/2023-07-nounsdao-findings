## QA REPORT

  
|      | Issue                                                                                                                                               |
| ---- |:--------------------------------------------------------------------------------------------------------------------------------------------------- |
| [01] | Array misbehavior in Duplicate Check Function                                                             |
| [02] | Unchecked Gas Refund Leads to Potential Security Risks                                                                                                           |
| [03] | Incomplete error handling in `withdrawal` functions                                                                                                          |
| [04] | Missing version field in the domain separator for signing vote                                                                                                     |

    
## [01] Array misbehavior in Duplicate Check Function

**Context** [NounsDAOV3Admin](https://github.dev/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L599-L607) , [NounsDAOLogicV1Fork](https://github.dev/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L793-L801)

**Impact** 

Potential inconsistency in handling single-element arrays, leading to unexpected outcomes.

**Description**

The `checkForDuplicates` [function](https://github.dev/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L599-L607) skips checking for duplicates if the input array has a single element. This is due to the condition in the first loop, `i < erc20tokens.length - 1`, which is not satisfied for a single-element array, thus preventing the function from proceeding.

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

**Recommendation**

Fix this by modifying the condition in the first loop to `i <= erc20tokens.length - 1` to ensure it also accommodates arrays with a single element.  

  
  
## [02] Unchecked Gas Refund Leads to Potential Security Risks

**Context** [NounsDAOV3Votes](https://github.dev/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L295-L308) , [NounsDAOLogicV2](https://github.dev/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1033-L1046)

**Impact** 

The current implementation allows for gas refunds to be made but does not check if the refund was successful or not. The impact of this potential security vulnerability can vary. In a worst-case scenario, it might cause financial loss to the user if the contract falsely assumes that a gas refund transaction was successful when it actually failed. This failure can go unnoticed, leading to inconsistencies in the contract's balance and potentially in the user's balance.

**Description**

The method `_refundGas()` is designed to refund a certain amount of gas to the sender. A low-level call is made to transfer the refund amount (`msg.sender.call{ value: refundAmount }('')`). However, no check is made to ensure that this call was successful. This is potentially risky, as low-level calls like this can fail for a variety of reasons. The contract currently ignores the boolean `refundSent`, which indicates whether the call was successful or not. Consequently, the contract might continue to operate under the assumption that the refund was successful, even if it was not.

**Recommendation**

To address this vulnerability, a check should be added after the low-level call to ensure it was successful. This can be done using a `require()` statement that checks the `refundSent` boolean. If the boolean is false (indicating the call failed), the contract should revert and throw an error. This will allow the contract to safely handle failed transactions and will prevent any false assumptions about the contract's state.

Here is an example of how this could be implemented:

```solidity
(bool refundSent, ) = msg.sender.call{ value: refundAmount }(''); require(refundSent, "Refund transaction failed");
```

By doing this, the contract becomes safer and more reliable, and ensures that it reflects its true state at all times.

  

## [03] Incomplete error handling in `withdrawal` functions

**Impact**

Misrepresentation of contract state and potential for financial discrepancies due to unverified withdrawals.

**Context**  [NounsDAOLogicV2](https://github.dev/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L818-L829)  ,  [NounsDAOV3Admin](https://github.dev/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L468-L475)


**Description**

The `_withdraw()` function for the admin doesn't halt execution or revert the transaction if a withdrawal fails, leading to possible inaccuracies in balance and misleading return values.

```solidty
function _withdraw() external returns (uint256, bool) {
        if (msg.sender != admin) {
            revert AdminOnly();
        }

        uint256 amount = address(this).balance;
        (bool sent, ) = msg.sender.call{ value: amount }('');
        emit Withdraw(amount, sent);

        return (amount, sent);
    }
```

**Recommendation**

Use a `require()` statement to ensure the transaction's success:

```solidity
(bool sent, ) = msg.sender.call{ value: amount }(''); 
require(sent, "Withdrawal transaction failed");
```

This safeguards the contract's integrity, maintains accuracy of the state, and prevents confusion for the admin user.  

## [04 ] Missing version field in the domain separator for signing vote

  
**Context**   [NounsDAOV3Votes](https://github.dev/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L156-L172)

**Impact**

The lack of the version field in the domain separator can lead to compatibility issues with signatures, resulting in failed transactions or possible security risks when the DApp undergoes major updates.

**Description**

In the 'castVoteBySig' function, the domain separator is created without including the version field:

```solidity
function castVoteBySig(
        NounsDAOStorageV3.StorageV3 storage ds,
        uint256 proposalId,
        uint8 support,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external {
        bytes32 domainSeparator = keccak256(
            abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), block.chainid, address(this))
        );
        bytes32 structHash = keccak256(abi.encode(BALLOT_TYPEHASH, proposalId, support));
        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', domainSeparator, structHash));
        address signatory = ecrecover(digest, v, r, s);
        require(signatory != address(0), 'NounsDAO::castVoteBySig: invalid signature');
        emit VoteCast(signatory, proposalId, support, castVoteInternal(ds, signatory, proposalId, support), '');
    }
```

This absence can cause signature compatibility issues if there are major updates to the contract.

**Recommendation**

Add the version field to the domain separator:

```diff
bytes32 domainSeparator = keccak256(abi.encode(
	DOMAIN_TYPEHASH,
	keccak256(bytes(name)),
+	keccak256(bytes(version)),
	block.chainid,
	address(this))
);
```

This ensures signatures remain compatible across different versions of the contract.