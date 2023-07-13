# Low

## [L-01] Storage variable will not be initialized when set i**nitial values in field declarations for upgradeable contract**

### Impact

Cannot fetch contract metadata

### **Proof of Concept**

 Declaring the storage variable and initializing its field at the same time is equivalent to initializing it with the constructor. If using upgradeable, variables other than `constant` should not be initialized at the time of declaration. As a result, `_contractURIHash` will not be initialized. Refer to [openzeppelin docs](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations) for more information. 

```solidity
string private _contractURIHash = 'QmZi1n79FqWt2tTLwCqiy6nLM6xLGRsEPQ5JmReJQKNNzX';
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L85](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L85)

### **Recommended Mitigation Steps**

Initialize storage variable at `initialize` function. 

## [L-02] Check address at lock functions

### Impact

When the minter/descriptor/seeder is locked with the wrong address, the system will be paralyzed and can not be fixed.

### **Proof of Concept**

Since there is no `unlockMinter` , `unlockDescriptor` , `unlockSeeder` function, you cannot undo it once minter/descriptor/seeder locked. Therefore, it is better to confirm address at lock functions.

```solidity
function lockMinter() external override onlyOwner whenMinterNotLocked {
    isMinterLocked = true;

    emit MinterLocked();
}
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L250-L254](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L250-L254)

```solidity
function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {
    isDescriptorLocked = true;

    emit DescriptorLocked();
}
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L270-L274](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L270-L274)

```solidity
function lockSeeder() external override onlyOwner whenSeederNotLocked {
    isSeederLocked = true;

    emit SeederLocked();
}
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L290-L294](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L290-L294)

### **Recommended Mitigation Steps**

Check zero address or supportsInterface at lock functions to make sure.

# Non-Critical

## [N-01] The `initializer` modifier should be added to the `initialize` function when making upgradeable contract

### Impact

Adding `initializer` modifier ensures that the initialization is only done once. While it is protected with `require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');`, it is more appropriate to use `initializer` . It is more accurate to remove `require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');` and add `initializer` modifier.

### **Proof of Concept**

```jsx
function initialize(
        address timelock_,
        address nouns_,
        uint256 votingPeriod_,
        uint256 votingDelay_,
        uint256 proposalThresholdBPS_,
        uint256 quorumVotesBPS_,
        address[] memory erc20TokensToIncludeInQuit_,
        uint256 delayedGovernanceExpirationTimestamp_
) public virtual {
```

[https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L165-L174](https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol#L165-L174)

### **Recommended Mitigation Steps**

Inherit `Initializable` and Add `initializer` modifier at `initialize` function.