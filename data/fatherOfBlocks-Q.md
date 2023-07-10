**script/ProposeTimelockMigrationCleanupMainnet.s.sol**
- L38 - The console.log should be removed as it reduces the readability of the code.


**script/ProposeDAOV3UpgradeMainnet.s.sol**
- L6/7 - Two contracts are imported, but they are never used, therefore they generate more gas expense in the deploy and also generate a low understanding in the blockchain explorer.

- L49 - The console.log should be removed as it reduces the readability of the code.


**script/DeployDAOV3NewContractsBase.s.sol**
- L29 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.


**script/ProposeENSReverseLookupConfigMainnet.s.sol**
- L24 - The console.log should be removed as it reduces the readability of the code.


**script/ProposeDAOV3UpgradeTestnet.s.sol**
- L6/7 - Two contracts are imported, but they are never used, therefore they generate more gas expense in the deploy and also generate a low understanding in the blockchain explorer.

- L19 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.

- L57 - The console.log should be removed as it reduces the readability of the code.


**script/DeployDAOV3DataContractsBase.s.sol**
- L15 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.


**contracts/governance/NounsDAOLogicV2.sol**
- L55/77 - The entire NounsDAOInterfaces.sol file is imported, being that only NounsDAOStorageV2 and NounsDAOEventsV2 are used, the simplest and easiest to understand is to directly import the contracts that are used.

- L55/57 - The file NounsDAOInterfaces.sol contains multiple contracts that are not interfaces, this can cause confusion as it does not respect standards.

- L150/154/158/170/171/172/644/662/681 - The initialize() function duplicates validation logic that exists in setter functions for the variables: votingPeriod, votingDelay and proposalThresholdBPS. The most beneficial thing would be to use the setters directly, inside the initialize().

- L598 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/governance/NounsDAOProxy.sol**
- L38/40 - The entire NounsDAOInterfaces.sol file is imported, being that only NounsDAOStorageV2 and NounsDAOEventsV2 are used, the simplest and easiest to understand is to directly import the contracts that are used.


**contracts/governance/fork/ForkDAODeployer.sol**
- L21/25 - The NounsDAOStorageV3 and NounsDAOProxy contracts are imported, but they are never used, therefore they generate more gas expense in the deploy and also generate a low understanding in the blockchain explorer.
 
- L61 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.


**contracts/governance/fork/NounsDAOV3Fork.sol**
- L247/254 - A division is performed by an input and it is never validated, therefore it should be validated beforehand that it is != 0, so that it does not generate an unhandled exception.


**contracts/governance/fork/NounsDAOForkEscrow.sol**
- L63 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.


**contracts/governance/fork/newdao/governance/NounsDAOEventsFork.sol**
- The contract only has events, but it is only used in the NounsDAOLogicV1Fork.sol contract, so it doesn't make much sense to unify the events in one contract, this would make sense if the contract is used in multiple files.


**contracts/governance/fork/newdao/governance/NounsDAOLogicV1Fork.sol**
- L599 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/governance/fork/newdao/token/base/ERC721CheckpointableUpgradeable.sol**
- L150 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/governance/NounsDAOLogicV3.sol**
- L58/65 - The entire NounsDAOInterfaces.sol file is imported, being that only NounsDAOStorageV2 and NounsDAOEventsV2 are used, the simplest and easiest to understand is to directly import the contracts that are used.


**contracts/governance/NounsDAOV3Votes.sol**
- L168 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/governance/NounsDAOInterfaces.sol**
- The file uses the word interfaces, but within the code it does not have interfaces, only contracts, therefore the name of the file should be modified or the contracts should be changed to interfaces.


**contracts/governance/NounsDAOV3Proposals.sol**
- L872/873/874/875/876/1006 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/governance/NounsDAOV3DynamicQuorum.sol**
- L60/63 - A division is performed by an input of the function, this should include a prior validation that the input != 0, so as not to generate an unhandled exception.
