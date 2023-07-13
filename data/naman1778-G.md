## [G-01] Division by two should use bit shifting

<x> / 2 is the same as <x> >> 1. While the compiler uses the SHR opcode to accomplish both, the version that uses division incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two.        

There is 1 instance of this issue in 1 file:

    File: contracts/governance/NounsDAOLogicV2.sol	

    1010: uint256 center = upper - (upper - lower) / 2;

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

#### Test Code
    
    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    
    contract Contract0 {
        uint256 num = 5;
        uint256 mul;
        function not_optimized() public returns(bool){
            mul = num / 2;
        }
    }
    
    contract Contract1 {
        uint256 num = 5;
        uint256 mul;
        function optimized() public returns(bool){
            mul = num >> 2;
        }
    }
    
#### Gas report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 62999                                     | 240             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24448           | 24448 | 24448  | 24448 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 47581                                     | 161             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24361           | 24361 | 24361  | 24361 | 1       |

## [G-02] Use nested *if* and, avoid multiple check combinations

Using nesting is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

There are 3 instances of this issue in 3 files:

    File: contracts/governance/NounsDAOLogicV2.sol	

    1026: if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

    File: contracts/governance/NounsDAOV3Votes.sol	

    240: if (
    241:    // only for votes can trigger an objection period
    242:    // we're in the last minute window
    243:    isForVoteInLastMinuteWindow &&
    244:    // first part of the vote flip check
    245:    // separated from the second part to optimize gas
    246:    isDefeatedBefore &&
    247:    // haven't turn on objection yet
    248:    proposal.objectionPeriodEndBlock == 0 &&
    249:    // second part of the vote flip check
    250:    !ds.isDefeated(proposal)
    251: ) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Votes.sol

    File: contracts/governance/NounsDAOV3Admin.sol

    585: if (pos > 0 && ds.quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }

    contract Contract0 {

        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }

    }

    contract Contract1 {

        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |

## [G-03] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up.

There are 6 instances of this issue in 2 files:

    File: contracts/governance/NounsDAOLogicV2.sol	
 
    305: for (uint256 i = 0; i < proposal.targets.length; i++) {

    343: for (uint256 i = 0; i < proposal.targets.length; i++) {

    372: for (uint256 i = 0; i < proposal.targets.length; i++) {

    405: for (uint256 i = 0; i < proposal.targets.length; i++) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol

    File: contracts/governance/NounsDAOV3Admin.sol

    602: for (uint256 i = 0; i < erc20tokens.length - 1; i++) {

    603: for (uint256 j = i + 1; j < erc20tokens.length; j++) {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOV3Admin.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-04] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

There are 4 instances of this issue in 1 file:

    File: contracts/governance/NounsDAOLogicV3.sol	

    236: function proposeBySigs(
    237:     ProposerSignature[] memory proposerSignatures,
    238:     address[] memory targets,
    239:     uint256[] memory values,
    240:     string[] memory signatures,
    241:     bytes[] memory calldatas,
    242:     string memory description
    243: ) external returns (uint256) {

    278: function updateProposal(
    279:     uint256 proposalId,
    280:     address[] memory targets,
    281:     uint256[] memory values,
    282:     string[] memory signatures,
    283:     bytes[] memory calldatas,
    284:     string memory description,
    285:     string memory updateMessage
    286: ) external {

    313: function updateProposalTransactions(
    314:     uint256 proposalId,
    315:     address[] memory targets,
    316:     uint256[] memory values,
    317:     string[] memory signatures,
    318:     bytes[] memory calldatas,
    319:     string memory updateMessage
    320: ) external {

    338: function updateProposalBySigs(
    339:     uint256 proposalId,
    340:     ProposerSignature[] memory proposerSignatures,
    341:     address[] memory targets,
    342:     uint256[] memory values,
    343:     string[] memory signatures,
    344:     bytes[] memory calldatas,
    345:     string memory description,
    346:     string memory updateMessage
    347: ) external {

https://github.com/nounsDAO/nouns-monorepo/blob/718211e063d511eeda1084710f6a682955e80dcb/packages/nouns-contracts/contracts/governance/NounsDAOLogicV3.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 100747                                    | 535             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 790             | 790 | 790    | 790 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 66917                                     | 366             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 556             | 556 | 556    | 556 | 1       |
