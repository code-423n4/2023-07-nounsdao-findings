## GAS-1: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* ERC721CheckpointableUpgradeable.sol (Line: 154, 184, 281, 290)
* NounsAuctionHouseFork.sol (Line: 124, 241)
* NounsDAOExecutor.sol (Line: 73, 74, 82, 83)
* NounsDAOExecutorV2.sol (Line: 96, 97, 105, 106)
* NounsDAOLogicV1Fork.sol (Line: 298, 513, 517, 519, 521, 527, 618)
* NounsDAOLogicV2.sol (Line: 221, 456, 462, 464, 466, 472, 617, 994, 1079)
* NounsDAOV3Admin.sol (Line: 595)
* NounsDAOV3DynamicQuorum.sol (Line: 94, 146)
* NounsDAOV3Fork.sol (Line: 119)
* NounsDAOV3Proposals.sol (Line: 252, 641, 648, 650, 652, 654, 662, 791, 963)
* NounsDAOV3Votes.sol (Line: 216)
* NounsTokenFork.sol (Line: 170, 184)

## GAS-2: Add unchecked{} for subtractions where the operands cannot underflow because of a previous require() or if statement

### Description

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
 if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
 This will stop the check for overflow and underflow so it will save gas.

### Affected file

* ERC721CheckpointableUpgradeable.sol (Line: 291)
* NounsAuctionHouseFork.sol (Line: 141)

## GAS-3: Bytes32 constants are more efficient than string constants

### Description

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

### Affected file

* NounsAuctionHouseFork.sol (Line: 48)
* NounsDAOExecutorV2.sol (Line: 82)
* NounsDAOLogicV1Fork.sol (Line: 116)
* NounsDAOLogicV2.sol (Line: 59)
* NounsDAOV3Votes.sol (Line: 44)
* NounsTokenFork.sol (Line: 46)

## GAS-4: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* ERC721CheckpointableUpgradeable.sol (Line: 124)
* NounsAuctionHouseFork.sol (Line: 214)
* NounsDAOExecutor.sol (Line: 91)
* NounsDAOExecutorV2.sol (Line: 114)
* NounsDAOProxy.sol (Line: 53)
* NounsDAOV3Proposals.sol (Line: 588)

## GAS-5: Change ```public``` state variable visibility to ```private```

### Description

If it is preferred to change the visibility of the ```owner``` state variable to private, this will save significant gas.

### Affected file

* NounsDAOExecutor.sol (Line: 66, 67)
* NounsDAOExecutorV2.sol (Line: 89, 90)
* NounsDAOInterfaces.sol (Line: 265, 268)
* NounsDAOStorageV1Fork.sol (Line: 22, 25)

## GAS-6: Checking msg.sender to not be zero address is redundant

### Description

Unecessary check.

### Affected file

* NounsDAOLogicV1Fork.sol (Line: 733)
* NounsDAOLogicV2.sol (Line: 856)

## GAS-7: Do not calculate constants

### Description

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

### Affected file

* ERC721CheckpointableUpgradeable.sol (Line: 70, 74)
* NounsDAOLogicV1Fork.sol (Line: 146, 150)
* NounsDAOLogicV2.sol (Line: 107, 111)
* NounsDAOV3Admin.sol (Line: 118, 121, 127, 145, 148)
* NounsDAOV3Proposals.sol (Line: 141, 144, 149)
* NounsDAOV3Votes.sol (Line: 47, 51)

## GAS-8: Internal functions not called by the contract should be removed to save deployment gas

### Description

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

### Affected file

* NounsAuctionHouseFork.sol (Line: 281)
* NounsDAOExecutorV2.sol (Line: 243)
* NounsDAOLogicV1Fork.sol (Line: 789)
* NounsTokenFork.sol (Line: 327)

## GAS-9: Keccak256() should only need to be called on a specific string literal once

### Description

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

### Affected file

* ERC721CheckpointableUpgradeable.sol (Line: 146, 147, 149, 150)
* NounsDAOExecutor.sol (Line: 120, 136, 151, 169)
* NounsDAOExecutorV2.sol (Line: 143, 159, 174, 192)
* NounsDAOLogicV1Fork.sol (Line: 595, 596, 598, 599)
* NounsDAOLogicV2.sol (Line: 594, 595, 597, 598)
* NounsDAOV3Proposals.sol (Line: 271, 861, 866, 872, 873, 874, 875, 876, 982, 1006, 1008, 1009)
* NounsDAOV3Votes.sol (Line: 164, 165, 167, 168)

## GAS-10: Not using the named return variables when a function returns, wastes deployment gas

### Description

It is not necessary to have both a named return and a return statement.

### Affected file

* DeployDAOV3NewContractsBase.s.sol (Line: 96)
* NounsDAOLogicV1Fork.sol (Line: 492)
* NounsDAOLogicV2.sol (Line: 435)
* NounsDAOLogicV3.sol (Line: 417, 497)
* NounsDAOV3Proposals.sol (Line: 686)

## GAS-11: Require()/Revert() statements that check input arguments should be at the top of the function

### Description

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

### Affected file

* NounsDAOExecutor.sol (Line: 153, 157, 157)
* NounsDAOExecutorV2.sol (Line: 176, 180, 180)

## GAS-12: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* NounsDAOExecutor.sol (Line: 86, 104)
* NounsTokenFork.sol (Line: 137)

## GAS-13: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* NounsDAOLogicV2.sol (Line: 1079, 1084)
* NounsDAOV3Admin.sol (Line: 595)
* NounsDAOV3DynamicQuorum.sol (Line: 146, 151)
* ProposeDAOV3UpgradeMainnet.s.sol (Line: 116)
* ProposeDAOV3UpgradeTestnet.s.sol (Line: 123)

## GAS-14: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* ERC721CheckpointableUpgradeable.sol (Line: 147)
* NounsAuctionHouseFork.sol (Line: 248)
* NounsDAOExecutor.sol (Line: 81)
* NounsDAOExecutorV2.sol (Line: 104)
* NounsDAOForkEscrow.sol (Line: 120, 151, 165)
* NounsDAOLogicV1Fork.sol (Line: 596)
* NounsDAOLogicV2.sol (Line: 595, 823, 1035)
* NounsDAOV3Admin.sol (Line: 469)
* NounsDAOV3Proposals.sol (Line: 985)
* NounsDAOV3Votes.sol (Line: 165, 297)

## GAS-15: Use nested if and avoid multiple check combinations

### Description

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Affected file

* ERC721CheckpointableUpgradeable.sol (Line: 227, 255)
* NounsDAOLogicV1Fork.sol (Line: 381)
* NounsDAOLogicV2.sol (Line: 1026)
* NounsDAOV3Admin.sol (Line: 585)
* NounsDAOV3Votes.sol (Line: 240)

## GAS-16: Use selfbalance()

### Description

Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.

### Affected file

* NounsDAOLogicV2.sol (Line: 823, 1035)
* NounsDAOV3Admin.sol (Line: 469)
* NounsDAOV3Votes.sol (Line: 297)

## GAS-17: Using > 0 costs more gas than != 0 when used on a uint in a require() statement

### Description

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance.

### Affected file

* ERC721CheckpointableUpgradeable.sol (Line: 227, 255)
* NounsAuctionHouseFork.sol (Line: 251)
* NounsDAOLogicV1Fork.sol (Line: 239, 381)
* NounsDAOLogicV2.sol (Line: 564, 1026)
* NounsDAOV3Admin.sol (Line: 484, 585)
* NounsDAOV3Fork.sol (Line: 255)
* NounsDAOV3Proposals.sol (Line: 888)
* NounsDAOV3Votes.sol (Line: 132)

## GAS-18: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* NounsAuctionHouseFork.sol (Line: 273)
* NounsDAOExecutor.sol (Line: 173)
* NounsDAOExecutorV2.sol (Line: 196)
* NounsDAOLogicV2.sol (Line: 824, 1043)
* NounsDAOProxy.sol (Line: 95, 110)
* NounsDAOV3Admin.sol (Line: 470)
* NounsDAOV3Votes.sol (Line: 305)