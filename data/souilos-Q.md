
# VULN 1 

## [LOW] Keccak Constant values should used to immutable rather than constant
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 111 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');


Found in line 101 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

------------------------------------------------------------------------ 

### Mitigation 

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.










# VULN 2 

## [LOW] Empty receive()/payable fallback() function does not authenticate requests
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 12 at nouns-monorepo/packages/nouns-contracts/contracts/test/MaliciousBidder.sol:

    receive() external payable {


Found in line 24 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

    receive() external payable {


Found in line 33 at nouns-monorepo/packages/nouns-contracts/contracts/test/MaliciousVoter.sol:

    receive() external payable {


Found in line 138 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol:

    receive() external payable {


Found in line 1090 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    receive() external payable {}


Found in line 141 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxyV2.sol:

    receive() external payable {


Found in line 186 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

    receive() external payable {}

------------------------------------------------------------------------ 

### Mitigation 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))).Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas. Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.










# VULN 3 

## [LOW] Use Ownable2Step rather than Ownable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 35 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

contract NounsAuctionHouse is INounsAuctionHouse, PausableUpgradeable, ReentrancyGuardUpgradeable, OwnableUpgradeable {

------------------------------------------------------------------------ 

### Mitigation 

Ownable2Step and Ownable2StepUpgradeable prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.










# VULN 4 

## [LOW] Keccak Constant values should used to immutable rather than constant
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 111 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');


Found in line 101 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');

------------------------------------------------------------------------ 

### Mitigation 

By using immutable variables, the Keccak256 hash is computed at contract deployment time rather than at compilation time. This means that the value can be updated if the algorithm changes in a future compiler version, without breaking backward compatibility. Additionally, it provides better gas optimization, as the Keccak256 hash is computed only once at contract deployment instead of every time the variable is accessed during execution.










# VULN 5 

## [LOW] Avoid using tx.origin
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 1043 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

            (bool refundSent, ) = tx.origin.call{ value: refundAmount }('');


Found in line 1044 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

            emit RefundableVote(tx.origin, refundAmount, refundSent);

------------------------------------------------------------------------ 

### Mitigation 

tx.origin is a global variable in Solidity that returns the address of the account that sent the transaction. Using the variable could make a contract vulnerable if an authorized account calls a malicious contract. You can impersonate a user using a third party contract.










# VULN 6 

## [LOW] Use .call instead of .transfer to send ether
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 249 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

            IERC20(weth).transfer(to, amount);


Found in line 36 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

        payable(msg.sender).transfer(wad);

------------------------------------------------------------------------ 

### Mitigation 

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues.Replace .transfer with .call. Note that the result of .call need to be checked.










# VULN 7 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 258 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

        _mint(owner(), to, nounId);


Found in line 47 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsTokenHarness.sol:

        _mint(owner(), to, currentNounId++);


Found in line 300 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        _mint(creator, to, tokenId);


Found in line 321 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function _mint(

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().










# VULN 8 

## [LOW] Open TODOs
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 370 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            // TODO Solidity doesn't support constant arrays, but these are fixed at compile-time


Found in line 575 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            // TODO this is all a compile-time constant

------------------------------------------------------------------------ 

### Mitigation 

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment.










# VULN 9 

## [LOW] Add parameter to event-emit
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 26 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

    event PartsLocked();


Found in line 33 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsToken.sol:

    event MinterLocked();


Found in line 37 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsToken.sol:

    event DescriptorLocked();


Found in line 41 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsToken.sol:

    event SeederLocked();


Found in line 24 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptor.sol:

    event PartsLocked();

------------------------------------------------------------------------ 

### Mitigation 

Some event-emit description doesn’t have parameter. Add parameter for front-end website or client app, they can has that something has happened on the blockchain.










# VULN 10 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 61 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    IProxyRegistry public immutable proxyRegistry;

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.
