# [G-01] Use `Do While` of instead `for` in some functions

Using `do while` instead of `for` in some functions may save some gas

## Example

### Before
```
for(uint256 i;i < _tokens.length;++i;) {
    reserves[i] = _tokens[i].balanceOf(address(this));
}
```


### After
```
uint256 i;
do {
    reserves[i] = _tokens[i].balanceOf(address(this));
    unchecked { ++i; }
} while(i < _tokens.length);

```

## Functions Links
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol#L405
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L117claimDuringForkPeriod
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/NounsDAOForkEscrow.sol#L148
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L149
https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/fork/newdao/token/NounsTokenFork.sol#L172