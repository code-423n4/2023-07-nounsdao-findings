
# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 115 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

                while (length > 0) {


Found in line 236 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

        if (_auction.amount > 0) {


Found in line 354 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            return left > 0 ? ErrorCode.ERR_CONSTRUCT : ErrorCode.ERR_NONE;


Found in line 153 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        return nCheckpoints > 0 ? checkpoints[account][nCheckpoints - 1].votes : 0;


Found in line 215 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        if (srcRep != dstRep && amount > 0) {


Found in line 218 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

                uint96 srcRepOld = srcRepNum > 0 ? checkpoints[srcRep][srcRepNum - 1].votes : 0;


Found in line 225 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

                uint96 dstRepOld = dstRepNum > 0 ? checkpoints[dstRep][dstRepNum - 1].votes : 0;


Found in line 243 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {


Found in line 564 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        if (votes > 0) {


Found in line 1026 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        if (pos > 0 && quorumParamsCheckpoints[pos - 1].fromBlock == blockNumber) {

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] Use shift Right/Left instead of division/multiplication if possible
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 191 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        Draw[] memory draws = new Draw[]((image.length - 5) / 2);


Found in line 111 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

            msg.value >= _auction.amount + ((_auction.amount * minBidIncrementPercentage) / 100),


Found in line 151 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        Rect[] memory rects = new Rect[]((image.length - 5) / 2);


Found in line 184 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

            uint32 center = upper - (upper - lower) / 2; // ceil, avoiding overflow


Found in line 1010 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

            uint256 center = upper - (upper - lower) / 2;


Found in line 1067 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        return (number * bps) / 10000;


Found in line 673 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        return (number * bps) / 10000;

------------------------------------------------------------------------ 

### Mitigation 

Use shift Right/Left instead of division/multiplication if possible.










# VULN 3 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 151 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

            _mintTo(noundersDAO, _currentNounId++);


Found in line 153 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

        return _mintTo(minter, _currentNounId++);


Found in line 100 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        for (uint8 p = 0; p < params.parts.length; p++) {


Found in line 111 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            for (uint256 i = 0; i < image.draws.length; i++) {


Found in line 133 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

                        currentY++;


Found in line 194 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            cursor++;


Found in line 131 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        for (uint256 i = 0; i < _backgrounds.length; i++) {


Found in line 432 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        for (uint256 i = 0; i < len; i++) {


Found in line 110 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < newColors.length; i++) {


Found in line 120 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _backgrounds.length; i++) {


Found in line 130 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _bodies.length; i++) {


Found in line 140 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _accessories.length; i++) {


Found in line 150 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _heads.length; i++) {


Found in line 160 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _glasses.length; i++) {


Found in line 22 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsTokenHarness.sol:

        _mintTo(to, currentNounId++);


Found in line 26 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsTokenHarness.sol:

        for (uint256 i = 0; i < amount; i++) {


Found in line 47 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsTokenHarness.sol:

        _mint(owner(), to, currentNounId++);


Found in line 26 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

        for (uint256 i = 0; i < calls.length; i++) {


Found in line 78 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

        for (uint256 i = 0; i < calls.length; i++) {


Found in line 81 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        for (uint8 p = 0; p < params.parts.length; p++) {


Found in line 90 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            for (uint256 i = 0; i < image.rects.length; i++) {


Found in line 109 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

                    currentY++;


Found in line 154 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            cursor++;


Found in line 89 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                val |= uint256(uint8(s.input[s.incnt++])) << s.bitcnt;


Found in line 117 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            len = uint256(uint8(s.input[s.incnt++]));


Found in line 118 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            len |= uint256(uint8(s.input[s.incnt++])) << 8;


Found in line 120 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            if (uint8(s.input[s.incnt++]) != (~len & 0xFF) || uint8(s.input[s.incnt++]) != ((~len >> 8) & 0xFF)) {


Found in line 137 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                s.output[s.outcnt++] = s.input[s.incnt++];


Found in line 349 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                    h.symbols[offs[lengths[start + symbol]]++] = symbol;


Found in line 715 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                    lengths[index++] = symbol;


Found in line 758 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                        lengths[index++] = len;


Found in line 239 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        proposalCount++;


Found in line 305 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 343 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 372 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 405 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 216 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        proposalCount++;


Found in line 281 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 319 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 346 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 371 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 4 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 100 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        for (uint8 p = 0; p < params.parts.length; p++) {


Found in line 111 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            for (uint256 i = 0; i < image.draws.length; i++) {


Found in line 168 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        for (uint256 i = 0; i < cursor; i += 4) {


Found in line 131 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        for (uint256 i = 0; i < _backgrounds.length; i++) {


Found in line 431 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint256 pageFirstImageIndex = 0;


Found in line 432 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        for (uint256 i = 0; i < len; i++) {


Found in line 110 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < newColors.length; i++) {


Found in line 120 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _backgrounds.length; i++) {


Found in line 130 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _bodies.length; i++) {


Found in line 140 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _accessories.length; i++) {


Found in line 150 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _heads.length; i++) {


Found in line 160 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _glasses.length; i++) {


Found in line 26 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsTokenHarness.sol:

        for (uint256 i = 0; i < amount; i++) {


Found in line 26 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

        for (uint256 i = 0; i < calls.length; i++) {


Found in line 78 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

        for (uint256 i = 0; i < calls.length; i++) {


Found in line 81 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        for (uint8 p = 0; p < params.parts.length; p++) {


Found in line 90 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            for (uint256 i = 0; i < image.rects.length; i++) {


Found in line 127 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        for (uint256 i = 0; i < cursor; i += 4) {


Found in line 150 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            uint256 code = 0;


Found in line 152 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            uint256 first = 0;


Found in line 156 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            uint256 index = 0;


Found in line 181 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint32 lower = 0;


Found in line 305 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 343 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 372 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 405 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 1007 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint256 lower = 0;


Found in line 281 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 319 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 346 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 371 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 5 

## [GAS] Use Custom Errors
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 224 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

        require(_auction.startTime != 0, "Auction hasn't begun");


Found in line 226 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

        require(block.timestamp >= _auction.endTime, "Auction hasn't completed");


Found in line 152 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");

------------------------------------------------------------------------ 

### Mitigation 

Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.










# VULN 6 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 128 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function setContractURIHash(string memory newContractURIHash) external onlyOwner {


Found in line 32 at nouns-monorepo/packages/nouns-contracts/contracts/Inflator.sol:

    function puff(bytes memory source, uint256 destlen) public pure returns (Inflate.ErrorCode, bytes memory) {


Found in line 86 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

    function _generateSVGRects(SVGParams memory params)


Found in line 166 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

    function _getChunk(uint256 cursor, string[16] memory buffer) private pure returns (string memory) {


Found in line 182 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

    function _decodeRLEImage(bytes memory image) private pure returns (DecodedImage memory) {


Found in line 218 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

    function _toHexString(bytes memory b) private pure returns (string memory) {


Found in line 401 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function tokenURI(uint256 tokenId, INounsSeeder.Seed memory seed) external view override returns (string memory) {


Found in line 411 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function dataURI(uint256 tokenId, INounsSeeder.Seed memory seed) public view override returns (string memory) {


Found in line 439 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function generateSVGImage(INounsSeeder.Seed memory seed) external view override returns (string memory) {


Found in line 450 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function getPartsForSeed(INounsSeeder.Seed memory seed) public view returns (ISVGRenderer.Part[] memory) {


Found in line 467 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function _getPalette(bytes memory part) private view returns (bytes memory) {


Found in line 252 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function tokenURI(uint256 tokenId, INounsSeeder.Seed memory seed) external view override returns (string memory) {


Found in line 262 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function dataURI(uint256 tokenId, INounsSeeder.Seed memory seed) public view override returns (string memory) {


Found in line 290 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function generateSVGImage(INounsSeeder.Seed memory seed) external view override returns (string memory) {


Found in line 343 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function _getPartsForSeed(INounsSeeder.Seed memory seed) internal view returns (bytes[] memory) {


Found in line 23 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

    function aggregate(Call[] memory calls) public returns (uint256 blockNumber, bytes[] memory returnData) {


Found in line 33 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

    function blockAndAggregate(Call[] memory calls)


Found in line 76 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

    function tryAggregate(bool requireSuccess, Call[] memory calls) public returns (Result[] memory returnData) {


Found in line 89 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

    function tryBlockAndAggregate(bool requireSuccess, Call[] memory calls)


Found in line 130 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

    function tokenURI(uint256 tokenId, INounsSeeder.Seed memory seed) external view override returns (string memory);


Found in line 132 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

    function dataURI(uint256 tokenId, INounsSeeder.Seed memory seed) external view override returns (string memory);


Found in line 140 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

    function generateSVGImage(INounsSeeder.Seed memory seed) external view returns (string memory);


Found in line 88 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptor.sol:

    function tokenURI(uint256 tokenId, INounsSeeder.Seed memory seed) external view override returns (string memory);


Found in line 90 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptor.sol:

    function dataURI(uint256 tokenId, INounsSeeder.Seed memory seed) external view override returns (string memory);


Found in line 98 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptor.sol:

    function generateSVGImage(INounsSeeder.Seed memory seed) external view returns (string memory);


Found in line 27 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorMinimal.sol:

    function tokenURI(uint256 tokenId, INounsSeeder.Seed memory seed) external view returns (string memory);


Found in line 29 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorMinimal.sol:

    function dataURI(uint256 tokenId, INounsSeeder.Seed memory seed) external view returns (string memory);


Found in line 23 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/IInflator.sol:

    function puff(bytes memory source, uint256 destlen) external pure returns (Inflate.ErrorCode, bytes memory);


Found in line 31 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/ISVGRenderer.sol:

    function generateSVG(SVGParams memory params) external view returns (string memory svg);


Found in line 33 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/ISVGRenderer.sol:

    function generateSVGPart(Part memory part) external view returns (string memory partialSVG);


Found in line 35 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/ISVGRenderer.sol:

    function generateSVGParts(Part[] memory parts) external view returns (string memory partialSVG);


Found in line 48 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

    function generateSVG(SVGParams memory params, mapping(uint8 => string[]) storage palettes)


Found in line 68 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

    function _generateSVGRects(SVGParams memory params, mapping(uint8 => string[]) storage palettes)


Found in line 125 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

    function _getChunk(uint256 cursor, string[16] memory buffer) private pure returns (string memory) {


Found in line 141 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

    function _decodeRLEImage(bytes memory image) private pure returns (DecodedImage memory) {


Found in line 75 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function bits(State memory s, uint256 need) private pure returns (ErrorCode, uint256) {


Found in line 103 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function _stored(State memory s) private pure returns (ErrorCode) {


Found in line 145 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function _decode(State memory s, Huffman memory h) private pure returns (ErrorCode, uint256) {


Found in line 572 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function _build_fixed(State memory s) private pure returns (ErrorCode) {


Found in line 606 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function _fixed(State memory s) private pure returns (ErrorCode) {


Found in line 613 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function _build_dynamic_lengths(State memory s) private pure returns (ErrorCode, uint256[] memory) {


Found in line 646 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function _build_dynamic(State memory s)


Found in line 796 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function _dynamic(State memory s) private pure returns (ErrorCode) {


Found in line 814 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    function puff(bytes memory source, uint256 destlen) internal pure returns (ErrorCode, bytes memory) {


Found in line 15 at nouns-monorepo/packages/nouns-contracts/contracts/libs/SSTORE2.sol:

    function write(bytes memory data) internal returns (address pointer) {


Found in line 34 at nouns-monorepo/packages/nouns-contracts/contracts/libs/NFTDescriptorV2.sol:

    function constructTokenURI(ISVGRenderer renderer, TokenURIParams memory params)


Found in line 60 at nouns-monorepo/packages/nouns-contracts/contracts/libs/NFTDescriptorV2.sol:

    function generateSVGImage(ISVGRenderer renderer, ISVGRenderer.SVGParams memory params)


Found in line 34 at nouns-monorepo/packages/nouns-contracts/contracts/libs/NFTDescriptor.sol:

    function constructTokenURI(TokenURIParams memory params, mapping(uint8 => string[]) storage palettes)


Found in line 60 at nouns-monorepo/packages/nouns-contracts/contracts/libs/NFTDescriptor.sol:

    function generateSVGImage(MultiPartRLEToSVG.SVGParams memory params, mapping(uint8 => string[]) storage palettes)


Found in line 253 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

    function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {


Found in line 258 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

    function safe96(uint256 n, string memory errorMessage) internal pure returns (uint96) {


Found in line 94 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol:

    function delegateTo(address callee, bytes memory data) internal {


Found in line 1023 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function _writeQuorumParamsCheckpoint(DynamicQuorumParams memory params) internal {


Found in line 1078 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {


Found in line 97 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxyV2.sol:

    function delegateTo(address callee, bytes memory data) internal {

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 7 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 100 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        for (uint8 p = 0; p < params.parts.length; p++) {


Found in line 111 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            for (uint256 i = 0; i < image.draws.length; i++) {


Found in line 192 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        for (uint256 i = 5; i < image.length; i += 2) {


Found in line 131 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        for (uint256 i = 0; i < _backgrounds.length; i++) {


Found in line 110 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < newColors.length; i++) {


Found in line 120 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _backgrounds.length; i++) {


Found in line 130 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _bodies.length; i++) {


Found in line 140 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _accessories.length; i++) {


Found in line 150 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _heads.length; i++) {


Found in line 160 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        for (uint256 i = 0; i < _glasses.length; i++) {


Found in line 26 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

        for (uint256 i = 0; i < calls.length; i++) {


Found in line 78 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

        for (uint256 i = 0; i < calls.length; i++) {


Found in line 81 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        for (uint8 p = 0; p < params.parts.length; p++) {


Found in line 90 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            for (uint256 i = 0; i < image.rects.length; i++) {


Found in line 152 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        for (uint256 i = 5; i < image.length; i += 2) {


Found in line 305 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 343 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 372 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 405 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 281 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 319 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 346 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {


Found in line 371 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        for (uint256 i = 0; i < proposal.targets.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 8 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 230 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

        if (_auction.bidder == address(0)) {


Found in line 370 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        if (pointer == address(0)) {


Found in line 34 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOImmutable.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 18 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOLogicV1Harness.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 18 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOLogicV2Harness.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 103 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

        if (from == address(0)) {


Found in line 108 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

        if (to == address(0)) {


Found in line 90 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        return current == address(0) ? delegator : current;


Found in line 113 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        if (delegatee == address(0)) delegatee = msg.sender;


Found in line 144 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 390 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        if (vetoer == address(0)) {


Found in line 122 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 9 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 31 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 top;


Found in line 32 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 right;


Found in line 33 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 bottom;


Found in line 34 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 left;


Found in line 38 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 length;


Found in line 39 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 colorIndex;


Found in line 100 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        for (uint8 p = 0; p < params.parts.length; p++) {


Found in line 114 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

                uint8 length = _getRectLength(currentX, draw.length, image.bounds.right);


Found in line 155 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 drawLength,


Found in line 156 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 rightBound


Found in line 157 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

    ) private pure returns (uint8) {


Found in line 158 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint8 remainingPixelsInLine = rightBound - uint8(currentX);


Found in line 184 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            top: uint8(image[1]),


Found in line 185 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            right: uint8(image[2]),


Found in line 186 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            bottom: uint8(image[3]),


Found in line 187 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            left: uint8(image[4])


Found in line 193 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

            draws[cursor] = Draw({ length: uint8(image[i]), colorIndex: uint8(image[i + 1]) });


Found in line 219 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        uint24 value = uint24(bytes3(b));


Found in line 160 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setPalette(uint8 paletteIndex, bytes calldata palette) external override onlyOwner whenPartsNotLocked {


Found in line 175 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        uint16 imageCount


Found in line 191 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        uint16 imageCount


Found in line 207 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        uint16 imageCount


Found in line 223 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        uint16 imageCount


Found in line 237 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setPalettePointer(uint8 paletteIndex, address pointer) external override onlyOwner whenPartsNotLocked {


Found in line 253 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        uint16 imageCount


Found in line 270 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        uint16 imageCount


Found in line 287 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        uint16 imageCount


Found in line 304 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        uint16 imageCount


Found in line 359 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function palettes(uint8 index) public view override returns (bytes memory) {


Found in line 468 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        return art.palettes(uint8(part[0]));


Found in line 49 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

    uint8 public minBidIncrementPercentage;


Found in line 67 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

        uint8 _minBidIncrementPercentage,


Found in line 185 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

    function setMinBidIncrementPercentage(uint8 _minBidIncrementPercentage) external override onlyOwner {


Found in line 35 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    mapping(uint8 => address) public palettesPointers;


Found in line 155 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function setPalette(uint8 paletteIndex, bytes calldata palette) external override onlyDescriptor {


Found in line 178 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 196 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 214 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 232 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 248 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function setPalettePointer(uint8 paletteIndex, address pointer) external override onlyDescriptor {


Found in line 266 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 285 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 304 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 323 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 368 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function palettes(uint8 paletteIndex) public view override returns (bytes memory) {


Found in line 384 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 397 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        uint16 imageCount


Found in line 44 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    mapping(uint8 => string[]) public override palettes;


Found in line 108 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addManyColorsToPalette(uint8 paletteIndex, string[] calldata newColors) external override onlyOwner {


Found in line 169 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addColorToPalette(uint8 _paletteIndex, string calldata _color) external override onlyOwner {


Found in line 301 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function _addColorToPalette(uint8 _paletteIndex, string calldata _color) internal {


Found in line 10 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

    uint8 public decimals = 18;


Found in line 10 at nouns-monorepo/packages/nouns-contracts/contracts/test/MaliciousVoter.sol:

    uint8 public support;


Found in line 16 at nouns-monorepo/packages/nouns-contracts/contracts/test/MaliciousVoter.sol:

        uint8 support_,


Found in line 13 at nouns-monorepo/packages/nouns-contracts/contracts/test/Voter.sol:

    uint8 public support;


Found in line 19 at nouns-monorepo/packages/nouns-contracts/contracts/test/Voter.sol:

        uint8 support_,


Found in line 46 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

    function palettes(uint8 paletteIndex) external view returns (bytes memory);


Found in line 72 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

    function setPalette(uint8 paletteIndex, bytes calldata palette) external;


Found in line 77 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

        uint16 imageCount


Found in line 83 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

        uint16 imageCount


Found in line 89 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

        uint16 imageCount


Found in line 95 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

        uint16 imageCount


Found in line 98 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

    function setPalettePointer(uint8 paletteIndex, address pointer) external;


Found in line 103 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

        uint16 imageCount


Found in line 109 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

        uint16 imageCount


Found in line 115 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

        uint16 imageCount


Found in line 121 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptorV2.sol:

        uint16 imageCount


Found in line 36 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptor.sol:

    function palettes(uint8 paletteIndex, uint256 colorIndex) external view returns (string memory);


Found in line 58 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptor.sol:

    function addManyColorsToPalette(uint8 paletteIndex, string[] calldata newColors) external;


Found in line 70 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsDescriptor.sol:

    function addColorToPalette(uint8 paletteIndex, string calldata color) external;


Found in line 46 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

    event PaletteSet(uint8 paletteIndex);


Found in line 48 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

    event BodiesAdded(uint16 count);


Found in line 50 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

    event AccessoriesAdded(uint16 count);


Found in line 52 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

    event HeadsAdded(uint16 count);


Found in line 54 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

    event GlassesAdded(uint16 count);


Found in line 57 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount;


Found in line 79 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

    function palettes(uint8 paletteIndex) external view returns (bytes memory);


Found in line 81 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

    function setPalette(uint8 paletteIndex, bytes calldata palette) external;


Found in line 86 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount


Found in line 92 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount


Found in line 98 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount


Found in line 104 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount


Found in line 110 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount


Found in line 113 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

    function setPalettePointer(uint8 paletteIndex, address pointer) external;


Found in line 118 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount


Found in line 124 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount


Found in line 130 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsArt.sol:

        uint16 imageCount


Found in line 64 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsAuctionHouse.sol:

    function setMinBidIncrementPercentage(uint8 minBidIncrementPercentage) external;


Found in line 28 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        uint8 top;


Found in line 29 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        uint8 right;


Found in line 30 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        uint8 bottom;


Found in line 31 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        uint8 left;


Found in line 35 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        uint8 length;


Found in line 36 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        uint8 colorIndex;


Found in line 40 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        uint8 paletteIndex;


Found in line 48 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

    function generateSVG(SVGParams memory params, mapping(uint8 => string[]) storage palettes)


Found in line 68 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

    function _generateSVGRects(SVGParams memory params, mapping(uint8 => string[]) storage palettes)


Found in line 81 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        for (uint8 p = 0; p < params.parts.length; p++) {


Found in line 142 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        uint8 paletteIndex = uint8(image[0]);


Found in line 144 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            top: uint8(image[1]),


Found in line 145 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            right: uint8(image[2]),


Found in line 146 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            bottom: uint8(image[3]),


Found in line 147 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            left: uint8(image[4])


Found in line 153 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

            rects[cursor] = Rect({ length: uint8(image[i]), colorIndex: uint8(image[i + 1]) });


Found in line 89 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                val |= uint256(uint8(s.input[s.incnt++])) << s.bitcnt;


Found in line 117 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            len = uint256(uint8(s.input[s.incnt++]));


Found in line 118 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            len |= uint256(uint8(s.input[s.incnt++])) << 8;


Found in line 120 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            if (uint8(s.input[s.incnt++]) != (~len & 0xFF) || uint8(s.input[s.incnt++]) != ((~len >> 8) & 0xFF)) {


Found in line 372 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            uint16[29] memory lens = [


Found in line 404 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            uint8[29] memory lext = [


Found in line 436 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            uint16[30] memory dists = [


Found in line 469 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            uint8[30] memory dext = [


Found in line 518 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                    s.output[s.outcnt] = bytes1(uint8(symbol));


Found in line 623 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            uint8[19] memory order = [16, 17, 18, 0, 8, 7, 9, 6, 10, 5, 11, 4, 12, 3, 13, 2, 14, 1, 15];


Found in line 34 at nouns-monorepo/packages/nouns-contracts/contracts/libs/NFTDescriptor.sol:

    function constructTokenURI(TokenURIParams memory params, mapping(uint8 => string[]) storage palettes)


Found in line 60 at nouns-monorepo/packages/nouns-contracts/contracts/libs/NFTDescriptor.sol:

    function generateSVGImage(MultiPartRLEToSVG.SVGParams memory params, mapping(uint8 => string[]) storage palettes)


Found in line 41 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

    uint8 public constant decimals = 0;


Found in line 48 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint32 fromBlock;


Found in line 53 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

    mapping(address => mapping(uint32 => Checkpoint)) public checkpoints;


Found in line 56 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

    mapping(address => uint32) public numCheckpoints;


Found in line 130 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint8 v,


Found in line 152 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint32 nCheckpoints = numCheckpoints[account];


Found in line 166 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint32 nCheckpoints = numCheckpoints[account];


Found in line 181 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint32 lower = 0;


Found in line 182 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint32 upper = nCheckpoints - 1;


Found in line 184 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

            uint32 center = upper - (upper - lower) / 2; // ceil, avoiding overflow


Found in line 217 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

                uint32 srcRepNum = numCheckpoints[srcRep];


Found in line 224 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

                uint32 dstRepNum = numCheckpoints[dstRep];


Found in line 234 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint32 nCheckpoints,


Found in line 238 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        uint32 blockNumber = safe32(


Found in line 253 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

    function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {


Found in line 255 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        return uint32(n);


Found in line 111 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');


Found in line 512 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function castVote(uint256 proposalId, uint8 support) external {


Found in line 526 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function castRefundableVote(uint256 proposalId, uint8 support) external {


Found in line 543 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint8 support,


Found in line 558 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint8 support,


Found in line 577 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint8 support,


Found in line 589 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint8 support,


Found in line 590 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint8 v,


Found in line 614 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint8 support


Found in line 702 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function _setMinQuorumVotesBPS(uint16 newMinQuorumVotesBPS) external {


Found in line 718 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint16 oldMinQuorumVotesBPS = params.minQuorumVotesBPS;


Found in line 732 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function _setMaxQuorumVotesBPS(uint16 newMaxQuorumVotesBPS) external {


Found in line 747 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint16 oldMaxQuorumVotesBPS = params.maxQuorumVotesBPS;


Found in line 759 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function _setQuorumCoefficient(uint32 newQuorumCoefficient) external {


Found in line 765 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint32 oldQuorumCoefficient = params.quorumCoefficient;


Found in line 784 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint16 newMinQuorumVotesBPS,


Found in line 785 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint16 newMaxQuorumVotesBPS,


Found in line 786 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint32 newQuorumCoefficient


Found in line 982 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint32 blockNumber = safe32(blockNumber_, 'NounsDAO::getDynamicQuorumParamsAt: block number exceeds 32 bits');


Found in line 1024 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint32 blockNumber = safe32(block.number, 'block number exceeds 32 bits');


Found in line 1078 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {


Found in line 1079 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(n <= type(uint32).max, errorMessage);


Found in line 1080 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        return uint32(n);


Found in line 1083 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    function safe16(uint256 n) internal pure returns (uint16) {


Found in line 1084 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        if (n > type(uint16).max) {


Found in line 1087 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        return uint16(n);


Found in line 101 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    bytes32 public constant BALLOT_TYPEHASH = keccak256('Ballot(uint256 proposalId,uint8 support)');


Found in line 450 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    function castVote(uint256 proposalId, uint8 support) external {


Found in line 462 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        uint8 support,


Found in line 474 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        uint8 support,


Found in line 475 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        uint8 v,


Found in line 499 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        uint8 support


Found in line 61 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxyV2.sol:

                'initialize(address,address,address,uint256,uint256,uint256,(uint16,uint16,uint32))',


Found in line 70 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

    event VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 votes, string reason);


Found in line 111 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

    event MinQuorumVotesBPSSet(uint16 oldMinQuorumVotesBPS, uint16 newMinQuorumVotesBPS);


Found in line 114 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

    event MaxQuorumVotesBPSSet(uint16 oldMaxQuorumVotesBPS, uint16 newMaxQuorumVotesBPS);


Found in line 117 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

    event QuorumCoefficientSet(uint32 oldQuorumCoefficient, uint32 newQuorumCoefficient);


Found in line 221 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

        uint8 support;


Found in line 325 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

        uint8 support;


Found in line 358 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

        uint16 minQuorumVotesBPS;


Found in line 360 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

        uint16 maxQuorumVotesBPS;


Found in line 363 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

        uint32 quorumCoefficient;


Found in line 369 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOInterfaces.sol:

        uint32 fromBlock;

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 10 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 135 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function isApprovedForAll(address owner, address operator) public view override(IERC721, ERC721) returns (bool) {


Found in line 149 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function mint() public override onlyMinter returns (uint256) {


Found in line 159 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function burn(uint256 nounId) public override onlyMinter {


Found in line 168 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function tokenURI(uint256 tokenId) public view override returns (string memory) {


Found in line 177 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function dataURI(uint256 tokenId) public view override returns (string memory) {


Found in line 314 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function backgrounds(uint256 index) public view override returns (string memory) {


Found in line 323 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function heads(uint256 index) public view override returns (bytes memory) {


Found in line 332 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function bodies(uint256 index) public view override returns (bytes memory) {


Found in line 341 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function accessories(uint256 index) public view override returns (bytes memory) {


Found in line 350 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function glasses(uint256 index) public view override returns (bytes memory) {


Found in line 359 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function palettes(uint8 index) public view override returns (bytes memory) {


Found in line 411 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function dataURI(uint256 tokenId, INounsSeeder.Seed memory seed) public view override returns (string memory) {


Found in line 333 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function backgroundsCount() public view override returns (uint256) {


Found in line 340 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function heads(uint256 index) public view override returns (bytes memory) {


Found in line 347 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function bodies(uint256 index) public view override returns (bytes memory) {


Found in line 354 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function accessories(uint256 index) public view override returns (bytes memory) {


Found in line 361 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function glasses(uint256 index) public view override returns (bytes memory) {


Found in line 368 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    function palettes(uint8 paletteIndex) public view override returns (bytes memory) {


Found in line 262 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function dataURI(uint256 tokenId, INounsSeeder.Seed memory seed) public view override returns (string memory) {


Found in line 54 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

    function supportsInterface(bytes4 interfaceId) public view virtual override(IERC165, ERC721) returns (bool) {


Found in line 61 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

    function tokenOfOwnerByIndex(address owner, uint256 index) public view virtual override returns (uint256) {


Found in line 69 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

    function totalSupply() public view virtual override returns (uint256) {


Found in line 76 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

    function tokenByIndex(uint256 index) public view virtual override returns (uint256) {


Found in line 81 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC165, IERC165) returns (bool) {


Found in line 91 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function balanceOf(address owner) public view virtual override returns (uint256) {


Found in line 99 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function ownerOf(uint256 tokenId) public view virtual override returns (address) {


Found in line 108 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function name() public view virtual override returns (string memory) {


Found in line 115 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function symbol() public view virtual override returns (string memory) {


Found in line 122 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {


Found in line 141 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function approve(address to, uint256 tokenId) public virtual override {


Found in line 156 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function getApproved(uint256 tokenId) public view virtual override returns (address) {


Found in line 165 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function setApprovalForAll(address operator, bool approved) public virtual override {


Found in line 175 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function isApprovedForAll(address owner, address operator) public view virtual override returns (bool) {

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 11 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 122 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

                        cursor += 4;


Found in line 130 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

                    currentX += length;


Found in line 168 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        for (uint256 i = 0; i < cursor; i += 4) {


Found in line 192 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

        for (uint256 i = 5; i < image.length; i += 2) {


Found in line 408 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

        trait.storedImagesCount += imageCount;


Found in line 439 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

            pageFirstImageIndex += page.imageCount;


Found in line 29 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

        balanceOf[msg.sender] += msg.value;


Found in line 67 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

        balanceOf[dst] += wad;


Found in line 98 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

                    cursor += 4;


Found in line 106 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

                currentX += rect.length;


Found in line 127 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        for (uint256 i = 0; i < cursor; i += 4) {


Found in line 152 at nouns-monorepo/packages/nouns-contracts/contracts/libs/MultiPartRLEToSVG.sol:

        for (uint256 i = 5; i < image.length; i += 2) {


Found in line 90 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                s.bitcnt += 8;


Found in line 161 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            for (len = 1; len <= MAXBITS; len += 5) {


Found in line 175 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                index += count;


Found in line 176 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                first += count;


Found in line 193 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                index += count;


Found in line 194 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                first += count;


Found in line 211 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                index += count;


Found in line 212 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                first += count;


Found in line 229 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                index += count;


Found in line 230 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                first += count;


Found in line 247 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                index += count;


Found in line 248 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                first += count;


Found in line 293 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            for (len = 1; len <= MAXBITS; len += 5) {


Found in line 563 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

                    s.outcnt += len;


Found in line 629 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            ncode += 4;


Found in line 675 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            nlen += 257;


Found in line 680 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            ndist += 1;


Found in line 56 at nouns-monorepo/packages/nouns-contracts/contracts/libs/SSTORE2.sol:

        start += DATA_OFFSET;


Found in line 66 at nouns-monorepo/packages/nouns-contracts/contracts/libs/SSTORE2.sol:

        start += DATA_OFFSET;


Found in line 67 at nouns-monorepo/packages/nouns-contracts/contracts/libs/SSTORE2.sol:

        end += DATA_OFFSET;


Found in line 331 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        _balances[to] += 1;


Found in line 387 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        _balances[to] += 1;

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 12 

## [GAS] Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 17 and 18 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

    mapping(address => uint256) public balanceOf;

    mapping(address => mapping(address => uint256)) public allowance;

------------------------------------------------------------------------ 

### Mitigation 

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.










# VULN 13 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 233 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

            nouns.transferFrom(address(this), _auction.bidder, _auction.nounId);


Found in line 41 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

        return address(this).balance;


Found in line 135 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

            abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name())), getChainId(), address(this))


Found in line 595 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

            abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), getChainIdInternal(), address(this))


Found in line 823 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        uint256 amount = address(this).balance;


Found in line 1035 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

            uint256 balance = address(this).balance;


Found in line 480 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

            abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), getChainIdInternal(), address(this))


Found in line 81 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(msg.sender == address(this), 'NounsDAOExecutor::setDelay: Call must come from NounsDAOExecutor.');


Found in line 99 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

            msg.sender == address(this),

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 14 

## [GAS] Functions guaranteed to revert when called by normal users can be marked payable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 128 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function setContractURIHash(string memory newContractURIHash) external onlyOwner {


Found in line 196 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function setMinter(address _minter) external override onlyOwner whenMinterNotLocked {


Found in line 206 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function lockMinter() external override onlyOwner whenMinterNotLocked {


Found in line 216 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function setDescriptor(INounsDescriptorMinimal _descriptor) external override onlyOwner whenDescriptorNotLocked {


Found in line 226 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function lockDescriptor() external override onlyOwner whenDescriptorNotLocked {


Found in line 236 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function setSeeder(INounsSeeder _seeder) external override onlyOwner whenSeederNotLocked {


Found in line 246 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function lockSeeder() external override onlyOwner whenSeederNotLocked {


Found in line 68 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setArt(INounsArt _art) external onlyOwner whenPartsNotLocked {


Found in line 78 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setRenderer(ISVGRenderer _renderer) external onlyOwner {


Found in line 89 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setArtDescriptor(address descriptor) external onlyOwner {


Found in line 98 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setArtInflator(IInflator inflator) external onlyOwner {


Found in line 141 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function addManyBackgrounds(string[] calldata _backgrounds) external override onlyOwner whenPartsNotLocked {


Found in line 149 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function addBackground(string calldata _background) external override onlyOwner whenPartsNotLocked {


Found in line 160 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setPalette(uint8 paletteIndex, bytes calldata palette) external override onlyOwner whenPartsNotLocked {


Found in line 237 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setPalettePointer(uint8 paletteIndex, address pointer) external override onlyOwner whenPartsNotLocked {


Found in line 367 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function lockParts() external override onlyOwner whenPartsNotLocked {


Found in line 378 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function toggleDataURIEnabled() external override onlyOwner {


Found in line 391 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    function setBaseURI(string calldata _baseURI) external override onlyOwner {


Found in line 144 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

    function pause() external override onlyOwner {


Found in line 153 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

    function unpause() external override onlyOwner {


Found in line 165 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

    function setTimeBuffer(uint256 _timeBuffer) external override onlyOwner {


Found in line 175 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

    function setReservePrice(uint256 _reservePrice) external override onlyOwner {


Found in line 185 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

    function setMinBidIncrementPercentage(uint8 _minBidIncrementPercentage) external override onlyOwner {


Found in line 108 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addManyColorsToPalette(uint8 paletteIndex, string[] calldata newColors) external override onlyOwner {


Found in line 119 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addManyBackgrounds(string[] calldata _backgrounds) external override onlyOwner whenPartsNotLocked {


Found in line 129 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addManyBodies(bytes[] calldata _bodies) external override onlyOwner whenPartsNotLocked {


Found in line 139 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addManyAccessories(bytes[] calldata _accessories) external override onlyOwner whenPartsNotLocked {


Found in line 149 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addManyHeads(bytes[] calldata _heads) external override onlyOwner whenPartsNotLocked {


Found in line 159 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addManyGlasses(bytes[] calldata _glasses) external override onlyOwner whenPartsNotLocked {


Found in line 169 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addColorToPalette(uint8 _paletteIndex, string calldata _color) external override onlyOwner {


Found in line 178 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addBackground(string calldata _background) external override onlyOwner whenPartsNotLocked {


Found in line 186 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addBody(bytes calldata _body) external override onlyOwner whenPartsNotLocked {


Found in line 194 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addAccessory(bytes calldata _accessory) external override onlyOwner whenPartsNotLocked {


Found in line 202 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addHead(bytes calldata _head) external override onlyOwner whenPartsNotLocked {


Found in line 210 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function addGlasses(bytes calldata _glasses) external override onlyOwner whenPartsNotLocked {


Found in line 218 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function lockParts() external override onlyOwner whenPartsNotLocked {


Found in line 229 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function toggleDataURIEnabled() external override onlyOwner {


Found in line 242 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    function setBaseURI(string calldata _baseURI) external override onlyOwner {

------------------------------------------------------------------------ 

### Mitigation 

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.










# VULN 15 

## [GAS] Do not calculate constants
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 27 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

    string private constant _SVG_START_TAG = '<svg width="320" height="320" viewBox="0 0 320 320" xmlns="http://www.w3.org/2000/svg" shape-rendering="crispEdges">';


Found in line 28 at nouns-monorepo/packages/nouns-contracts/contracts/SVGRenderer.sol:

    string private constant _SVG_END_TAG = '</svg>';


Found in line 16 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

    uint256 constant MAXCODES = (MAXLCODES + MAXDCODES);


Found in line 9 at nouns-monorepo/packages/nouns-contracts/contracts/libs/SSTORE2.sol:

    uint256 internal constant DATA_OFFSET = 1; // We skip the first byte as it's a STOP opcode to ensure the contract can't be called.


Found in line 62 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%


Found in line 65 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%


Found in line 68 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MIN_VOTING_PERIOD = 5_760; // About 24 hours


Found in line 71 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MAX_VOTING_PERIOD = 80_640; // About 2 weeks


Found in line 77 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MAX_VOTING_DELAY = 40_320; // About 1 week


Found in line 80 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MIN_QUORUM_VOTES_BPS_LOWER_BOUND = 200; // 200 basis points or 2%


Found in line 83 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MIN_QUORUM_VOTES_BPS_UPPER_BOUND = 2_000; // 2,000 basis points or 20%


Found in line 86 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MAX_QUORUM_VOTES_BPS_UPPER_BOUND = 6_000; // 4,000 basis points or 60%


Found in line 89 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant MAX_QUORUM_VOTES_BPS = 2_000; // 2,000 basis points or 20%


Found in line 92 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    uint256 public constant proposalMaxOperations = 10; // 10 actions


Found in line 70 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    uint256 public constant MIN_PROPOSAL_THRESHOLD_BPS = 1; // 1 basis point or 0.01%


Found in line 73 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    uint256 public constant MAX_PROPOSAL_THRESHOLD_BPS = 1_000; // 1,000 basis points or 10%


Found in line 76 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    uint256 public constant MIN_VOTING_PERIOD = 5_760; // About 24 hours


Found in line 79 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    uint256 public constant MAX_VOTING_PERIOD = 80_640; // About 2 weeks


Found in line 85 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    uint256 public constant MAX_VOTING_DELAY = 40_320; // About 1 week


Found in line 88 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    uint256 public constant MIN_QUORUM_VOTES_BPS = 200; // 200 basis points or 2%


Found in line 91 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    uint256 public constant MAX_QUORUM_VOTES_BPS = 2_000; // 2,000 basis points or 20%


Found in line 94 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    uint256 public constant proposalMaxOperations = 10; // 10 actions

------------------------------------------------------------------------ 

### Mitigation 

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.










# VULN 16 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 354 at nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol:

            return left > 0 ? ErrorCode.ERR_CONSTRUCT : ErrorCode.ERR_NONE;


Found in line 153 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        return nCheckpoints > 0 ? checkpoints[account][nCheckpoints - 1].votes : 0;


Found in line 126 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : '';


Found in line 1049 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        return a < b ? a : b;

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 17 

## [GAS] Change public state variable visibility to private
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 26 at nouns-monorepo/packages/nouns-contracts/contracts/NounsArt.sol:

    address public override descriptor;

------------------------------------------------------------------------ 

### Mitigation 

It is preferred to change the visibility of the state variables to private, this will save significant gas.










# VULN 18 

## [GAS] With assembly, .call (bool success) transfer can be done gas-optimized
------------------------------------------------------------------------ 

### Proof of concept 

Found in nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol (function _safeTransferETH(address to, uint256 value) internal returns (bool) {):

    /**

     * @notice Transfer ETH and return the success status.

     * @dev This function only forwards 30,000 gas to the callee.

     */

    function _safeTransferETH(address to, uint256 value) internal returns (bool) {

        (bool success, ) = to.call{ value: value, gas: 30_000 }(new bytes(0));

        return success;

    }

}



Found in nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol (function aggregate(Call[] memory calls) public returns (uint256 blockNumber, bytes[] memory returnData) {):



    function aggregate(Call[] memory calls) public returns (uint256 blockNumber, bytes[] memory returnData) {

        blockNumber = block.number;

        returnData = new bytes[](calls.length);

        for (uint256 i = 0; i < calls.length; i++) {

            (bool success, bytes memory ret) = calls[i].target.call(calls[i].callData);

            require(success, 'Multicall aggregate: call failed');

            returnData[i] = ret;

        }

    }





Found in nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol (function tryAggregate(bool requireSuccess, Call[] memory calls) public returns (Result[] memory returnData) {):

    }



    function tryAggregate(bool requireSuccess, Call[] memory calls) public returns (Result[] memory returnData) {

        returnData = new Result[](calls.length);

        for (uint256 i = 0; i < calls.length; i++) {

            (bool success, bytes memory ret) = calls[i].target.call(calls[i].callData);



            if (requireSuccess) {

                require(success, 'Multicall2 aggregate: call failed');

            }





Found in nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol (function delegateTo(address callee, bytes memory data) internal {):

     * @dev It returns to the external caller whatever the implementation returns or forwards reverts

     * @param callee The contract to delegatecall

     * @param data The raw data to delegatecall

     */

    function delegateTo(address callee, bytes memory data) internal {

        (bool success, bytes memory returnData) = callee.delegatecall(data);

        assembly {

            if eq(success, 0) {

                revert(add(returnData, 0x20), returndatasize())

            }

        }



Found in nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol (function _fallback() internal {):

     * It returns to the external caller whatever the implementation returns

     * or forwards reverts.

     */

    function _fallback() internal {

        // delegate all other functions to current implementation

        (bool success, ) = implementation.delegatecall(msg.data);



        assembly {

            let free_mem_ptr := mload(0x40)

            returndatacopy(free_mem_ptr, 0, returndatasize())





Found in nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxyV2.sol (function delegateTo(address callee, bytes memory data) internal {):

     * @dev It returns to the external caller whatever the implementation returns or forwards reverts

     * @param callee The contract to delegatecall

     * @param data The raw data to delegatecall

     */

    function delegateTo(address callee, bytes memory data) internal {

        (bool success, bytes memory returnData) = callee.delegatecall(data);

        assembly {

            if eq(success, 0) {

                revert(add(returnData, 0x20), returndatasize())

            }

        }



Found in nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxyV2.sol (function _fallback() internal {):

     * It returns to the external caller whatever the implementation returns

     * or forwards reverts.

     */

    function _fallback() internal {

        // delegate all other functions to current implementation

        (bool success, ) = implementation.delegatecall(msg.data);



        assembly {

            let free_mem_ptr := mload(0x40)

            returndatacopy(free_mem_ptr, 0, returndatasize())





Found in nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol (function executeTransaction():

        } else {

            callData = abi.encodePacked(bytes4(keccak256(bytes(signature))), data);

        }



        // solium-disable-next-line security/no-call-value

        (bool success, bytes memory returnData) = target.call{ value: value }(callData);

        require(success, 'NounsDAOExecutor::executeTransaction: Transaction execution reverted.');



        emit ExecuteTransaction(txHash, target, value, signature, data, eta);



        return returnData;


------------------------------------------------------------------------ 

### Mitigation 

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.










# VULN 19 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 27 at nouns-monorepo/packages/nouns-contracts/contracts/proxies/NounsAuctionHouseProxy.sol:

    ) TransparentUpgradeableProxy(logic, admin, data) {}


Found in line 23 at nouns-monorepo/packages/nouns-contracts/contracts/proxies/NounsAuctionHouseProxyAdmin.sol:

contract NounsAuctionHouseProxyAdmin is ProxyAdmin {}


Found in line 12 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOExecutorHarness.sol:

    constructor(address admin_, uint256 delay_) NounsDAOExecutor(admin_, delay_) {}


Found in line 19 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsTokenHarness.sol:

    ) NounsToken(noundersDAO, minter, descriptor, seeder, proxyRegistry) {}


Found in line 454 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    ) internal virtual {}


Found in line 1090 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    receive() external payable {}


Found in line 186 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

    receive() external payable {}


Found in line 188 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

    fallback() external payable {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).










# VULN 20 

## [GAS] Use double require instead of using &&
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 856 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');



Found in line 617 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(msg.sender == pendingAdmin && msg.sender != address(0), 'NounsDAO::_acceptAdmin: pending admin only');


------------------------------------------------------------------------ 

### Mitigation 

Using double require instead of operator && can save more gas. When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.










# VULN 21 

## [GAS] bytes constants are more eficient than string constans
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 49 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

    string public override baseURI;


Found in line 41 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

    string public override baseURI;


Found in line 8 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

    string public name = 'Wrapped Ether';


Found in line 9 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

    string public symbol = 'WETH';


Found in line 59 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

    string public constant name = 'Nouns DAO';


Found in line 67 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

    string public constant name = 'Nouns DAO';

------------------------------------------------------------------------ 

### Mitigation 

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.










# VULN 22 

## [GAS] Use a more recent version of solidity
------------------------------------------------------------------------ 

### Proof of concept 

Found in nouns-monorepo/packages/nouns-contracts/contracts/libs/Inflate.sol at line 2: pragma solidity >=0.8.0 <0.9.0;
------------------------------------------------------------------------ 

### Mitigation 

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath * Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining * Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads * Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings * Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value * In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller. * In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.










# VULN 23 

## [GAS] Use assembly to write address storage values
------------------------------------------------------------------------ 

### Proof of concept 

Found in lines 103 to 109 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    constructor(
        address _noundersDAO,
        address _minter,
        INounsDescriptorMinimal _descriptor,
        INounsSeeder _seeder,
        IProxyRegistry _proxyRegistry
    ) ERC721('Nouns', 'NOUN') {


Found in lines 23 to 27 at nouns-monorepo/packages/nouns-contracts/contracts/proxies/NounsAuctionHouseProxy.sol:

    constructor(
        address logic,
        address admin,
        bytes memory data
    ) TransparentUpgradeableProxy(logic, admin, data) {}


Found in lines 8 to 17 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOImmutable.sol:

    constructor(
        address timelock_,
        address nouns_,
        address admin_,
        address vetoer_,
        uint256 votingPeriod_,
        uint256 votingDelay_,
        uint256 proposalThresholdBPS_,
        uint256 quorumVotesBPS_
    ) {


Found in lines 13 to 19 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsTokenHarness.sol:

    constructor(
        address noundersDAO,
        address minter,
        INounsDescriptorMinimal descriptor,
        INounsSeeder seeder,
        IProxyRegistry proxyRegistry
    ) NounsToken(noundersDAO, minter, descriptor, seeder, proxyRegistry) {}


Found in lines 41 to 51 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol:

    constructor(
        address timelock_,
        address nouns_,
        address vetoer_,
        address admin_,
        address implementation_,
        uint256 votingPeriod_,
        uint256 votingDelay_,
        uint256 proposalThresholdBPS_,
        uint256 quorumVotesBPS_
    ) {


Found in lines 44 to 54 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxyV2.sol:

    constructor(
        address timelock_,
        address nouns_,
        address vetoer_,
        address admin_,
        address implementation_,
        uint256 votingPeriod_,
        uint256 votingDelay_,
        uint256 proposalThresholdBPS_,
        DynamicQuorumParams memory dynamicQuorumParams_
    ) {

------------------------------------------------------------------------ 

### Mitigation 

Use assembly to write address storage values. Here are a few reasons: * Reduced opcode usage: When using assembly, you can directly manipulate storage values using lower-level instructions like sstore (storage store) instead of relying on higher-level Solidity storage assignments. These direct operations typically result in fewer opcode executions, reducing gas costs. * Avoiding unnecessary checks: Solidity storage assignments often involve additional checks and operations, such as enforcing security modifiers or triggering events. By using assembly, you can bypass these additional checks and perform the necessary storage operations directly, resulting in gas savings. * Optimized packing: Assembly provides greater flexibility in packing and unpacking data structures. By carefully arranging and manipulating the storage layout in assembly, you can achieve more efficient storage utilization and minimize wasted storage space. * Fine-grained control: Assembly allows for precise control over gas-consuming operations. You can optimize gas usage by selecting specific instructions and minimizing unnecessary operations or data copying.










# VULN 24 

## [GAS] Use ERC721A instead ERC721
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 29 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

contract NounsToken is INounsToken, Ownable, ERC721Checkpointable {


Found in line 109 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    ) ERC721('Nouns', 'NOUN') {


Found in line 135 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

    function isApprovedForAll(address owner, address operator) public view override(IERC721, ERC721) returns (bool) {


Found in line 24 at nouns-monorepo/packages/nouns-contracts/contracts/interfaces/INounsToken.sol:

interface INounsToken is IERC721 {


Found in line 38 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

abstract contract ERC721Enumerable is ERC721, IERC721Enumerable {


Found in line 54 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

    function supportsInterface(bytes4 interfaceId) public view virtual override(IERC165, ERC721) returns (bool) {


Found in line 55 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

        return interfaceId == type(IERC721Enumerable).interfaceId || super.supportsInterface(interfaceId);


Found in line 62 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

        require(index < ERC721.balanceOf(owner), 'ERC721Enumerable: owner index out of bounds');


Found in line 77 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

        require(index < ERC721Enumerable.totalSupply(), 'ERC721Enumerable: global index out of bounds');


Found in line 121 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

        uint256 length = ERC721.balanceOf(to);


Found in line 147 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Enumerable.sol:

        uint256 lastTokenIndex = ERC721.balanceOf(from) - 1;


Found in line 39 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

abstract contract ERC721Checkpointable is ERC721Enumerable {


Found in line 80 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        return safe96(balanceOf(delegator), 'ERC721Checkpointable::votesToDelegate: amount exceeds 96 bits');


Found in line 140 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        require(signatory != address(0), 'ERC721Checkpointable::delegateBySig: invalid signature');


Found in line 141 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');


Found in line 142 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');


Found in line 164 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        require(blockNumber < block.number, 'ERC721Checkpointable::getPriorVotes: not yet determined');


Found in line 219 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

                uint96 srcRepNew = sub96(srcRepOld, amount, 'ERC721Checkpointable::_moveDelegates: amount underflows');


Found in line 226 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

                uint96 dstRepNew = add96(dstRepOld, amount, 'ERC721Checkpointable::_moveDelegates: amount overflows');


Found in line 240 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

            'ERC721Checkpointable::_writeCheckpoint: block number exceeds 32 bits'


Found in line 48 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

contract ERC721 is Context, ERC165, IERC721, IERC721Metadata {


Found in line 83 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

            interfaceId == type(IERC721).interfaceId ||


Found in line 84 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

            interfaceId == type(IERC721Metadata).interfaceId ||


Found in line 92 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(owner != address(0), 'ERC721: balance query for the zero address');


Found in line 101 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(owner != address(0), 'ERC721: owner query for nonexistent token');


Found in line 123 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_exists(tokenId), 'ERC721Metadata: URI query for nonexistent token');


Found in line 142 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        address owner = ERC721.ownerOf(tokenId);


Found in line 143 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(to != owner, 'ERC721: approval to current owner');


Found in line 147 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

            'ERC721: approve caller is not owner nor approved for all'


Found in line 157 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_exists(tokenId), 'ERC721: approved query for nonexistent token');


Found in line 166 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(operator != _msgSender(), 'ERC721: approve to caller');


Found in line 188 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_isApprovedOrOwner(_msgSender(), tokenId), 'ERC721: transfer caller is not owner nor approved');


Found in line 213 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_isApprovedOrOwner(_msgSender(), tokenId), 'ERC721: transfer caller is not owner nor approved');


Found in line 242 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_checkOnERC721Received(from, to, tokenId, _data), 'ERC721: transfer to non ERC721Receiver implementer');


Found in line 265 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_exists(tokenId), 'ERC721: operator query for nonexistent token');


Found in line 266 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        address owner = ERC721.ownerOf(tokenId);


Found in line 302 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

            _checkOnERC721Received(address(0), to, tokenId, _data),


Found in line 303 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

            'ERC721: transfer to non ERC721Receiver implementer'


Found in line 326 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(to != address(0), 'ERC721: mint to the zero address');


Found in line 327 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(!_exists(tokenId), 'ERC721: token already minted');


Found in line 349 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        address owner = ERC721.ownerOf(tokenId);


Found in line 378 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(ERC721.ownerOf(tokenId) == from, 'ERC721: transfer of token that is not own');


Found in line 379 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(to != address(0), 'ERC721: transfer to the zero address');


Found in line 400 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        emit Approval(ERC721.ownerOf(tokenId), to, tokenId);


Found in line 413 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

    function _checkOnERC721Received(


Found in line 420 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

            try IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, _data) returns (bytes4 retval) {


Found in line 421 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

                return retval == IERC721Receiver(to).onERC721Received.selector;


Found in line 424 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

                    revert('ERC721: transfer to non ERC721Receiver implementer');

------------------------------------------------------------------------ 

### Mitigation 

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee. Reference: https://nextrope.com/erc721-vs-erc721a-2/










# VULN 25 

## [GAS] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 67 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

        require(!isMinterLocked, 'Minter is locked');


Found in line 75 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

        require(!isDescriptorLocked, 'Descriptor is locked');


Found in line 83 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

        require(!isSeederLocked, 'Seeder is locked');


Found in line 91 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

        require(msg.sender == noundersDAO, 'Sender is not the nounders DAO');


Found in line 99 at nouns-monorepo/packages/nouns-contracts/contracts/NounsToken.sol:

        require(msg.sender == minter, 'Sender is not the minter');


Found in line 55 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptorV2.sol:

        require(!arePartsLocked, 'Parts are locked');


Found in line 110 at nouns-monorepo/packages/nouns-contracts/contracts/NounsAuctionHouse.sol:

        require(


Found in line 65 at nouns-monorepo/packages/nouns-contracts/contracts/NounsDescriptor.sol:

        require(!arePartsLocked, 'Parts are locked');


Found in line 33 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOImmutable.sol:

        require(msg.sender == admin, 'NounsDAO::initialize: admin only');


Found in line 34 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOImmutable.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 17 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOLogicV1Harness.sol:

        require(msg.sender == admin, 'NounsDAO::initialize: admin only');


Found in line 18 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOLogicV1Harness.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 62 at nouns-monorepo/packages/nouns-contracts/contracts/test/WETH.sol:

            require(allowance[src][msg.sender] >= wad);


Found in line 82 at nouns-monorepo/packages/nouns-contracts/contracts/test/Multicall2.sol:

                require(success, 'Multicall2 aggregate: call failed');


Found in line 17 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOLogicV2Harness.sol:

        require(msg.sender == admin, 'NounsDAO::initialize: admin only');


Found in line 18 at nouns-monorepo/packages/nouns-contracts/contracts/test/NounsDAOLogicV2Harness.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 44 at nouns-monorepo/packages/nouns-contracts/contracts/libs/SSTORE2.sol:

        require(pointer != address(0), 'DEPLOYMENT_FAILED');


Found in line 69 at nouns-monorepo/packages/nouns-contracts/contracts/libs/SSTORE2.sol:

        require(pointer.code.length >= end, 'OUT_OF_BOUNDS');


Found in line 140 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        require(signatory != address(0), 'ERC721Checkpointable::delegateBySig: invalid signature');


Found in line 141 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        require(nonce == nonces[signatory]++, 'ERC721Checkpointable::delegateBySig: invalid nonce');


Found in line 142 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        require(block.timestamp <= expiry, 'ERC721Checkpointable::delegateBySig: signature expired');


Found in line 269 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol:

        require(c >= a, errorMessage);


Found in line 188 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_isApprovedOrOwner(_msgSender(), tokenId), 'ERC721: transfer caller is not owner nor approved');


Found in line 213 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_isApprovedOrOwner(_msgSender(), tokenId), 'ERC721: transfer caller is not owner nor approved');


Found in line 242 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(_checkOnERC721Received(from, to, tokenId, _data), 'ERC721: transfer to non ERC721Receiver implementer');


Found in line 301 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(


Found in line 327 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(!_exists(tokenId), 'ERC721: token already minted');


Found in line 379 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

        require(to != address(0), 'ERC721: transfer to the zero address');


Found in line 424 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

                    revert('ERC721: transfer to non ERC721Receiver implementer');


Found in line 427 at nouns-monorepo/packages/nouns-contracts/contracts/base/ERC721.sol:

                        revert(add(32, reason), mload(reason))


Found in line 118 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxy.sol:

                revert(free_mem_ptr, returndatasize())


Found in line 144 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 148 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');


Found in line 149 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');


Found in line 150 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 154 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 158 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 210 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 214 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 226 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

            require(


Found in line 230 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

            require(


Found in line 325 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 365 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 600 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(signatory != address(0), 'NounsDAO::castVoteBySig: invalid signature');


Found in line 617 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');


Found in line 620 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');


Found in line 708 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 713 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 738 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 742 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV2.sol:

        require(


Found in line 122 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(address(timelock) == address(0), 'NounsDAO::initialize: can only initialize once');


Found in line 123 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(msg.sender == admin, 'NounsDAO::initialize: admin only');


Found in line 124 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(timelock_ != address(0), 'NounsDAO::initialize: invalid timelock address');


Found in line 125 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(nouns_ != address(0), 'NounsDAO::initialize: invalid nouns address');


Found in line 126 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(


Found in line 130 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(


Found in line 134 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(


Found in line 138 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(


Found in line 187 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(


Found in line 191 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(


Found in line 203 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

            require(


Found in line 207 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

            require(


Found in line 301 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(


Found in line 485 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(signatory != address(0), 'NounsDAO::castVoteBySig: invalid signature');


Found in line 502 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(support <= 2, 'NounsDAO::castVoteInternal: invalid vote type');


Found in line 505 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOLogicV1.sol:

        require(receipt.hasVoted == false, 'NounsDAO::castVoteInternal: voter already voted');


Found in line 121 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOProxyV2.sol:

                revert(free_mem_ptr, returndatasize())


Found in line 73 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(delay_ >= MINIMUM_DELAY, 'NounsDAOExecutor::constructor: Delay must exceed minimum delay.');


Found in line 74 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(delay_ <= MAXIMUM_DELAY, 'NounsDAOExecutor::setDelay: Delay must not exceed maximum delay.');


Found in line 114 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(msg.sender == admin, 'NounsDAOExecutor::queueTransaction: Call must come from admin.');


Found in line 115 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(


Found in line 134 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(msg.sender == admin, 'NounsDAOExecutor::cancelTransaction: Call must come from admin.');


Found in line 149 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(msg.sender == admin, 'NounsDAOExecutor::executeTransaction: Call must come from admin.');


Found in line 152 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(queuedTransactions[txHash], "NounsDAOExecutor::executeTransaction: Transaction hasn't been queued.");


Found in line 153 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(


Found in line 157 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(


Found in line 174 at nouns-monorepo/packages/nouns-contracts/contracts/governance/NounsDAOExecutor.sol:

        require(success, 'NounsDAOExecutor::executeTransaction: Transaction execution reverted.');

------------------------------------------------------------------------ 

### Mitigation 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.
