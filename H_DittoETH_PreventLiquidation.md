Front running flagger/liquidator by transfering short NFT

### Severity
High Risk

### Relevant GitHub Links
https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ERC721Facet.sol

https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol

https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallSecondaryFacet.sol

### Summary
A short can create an NFT of its short and if this short has a bad ratio and could be liquidated, front-run this liquidator/flagger by transferring the short NFT , the liquidator TX would then revert as the short address would have changed.

### Vulnerability Details
When a shorter wants to create a short order he/she calls libShortRecord.sol#createShortRecord() the process will create a struct in storage :

LibShortRecord.sol#createShortRecord()
if match => `LibOrders.sol#sellMatchAlgo()` else LibOrders.sol#addShort() will be called to add the short in the queue
when short is matched (immediately or when matched with good bid) LibOrders.sol#matchIncomingShort() is called
LibOrders.sol#matchIncomingShort() will call LibShortRecord.sol#createShortRecord()

LibShortRecord.sol#createShortRecord() will create a storage struct
```solidity
    s.shortRecords[asset][shorter][id] = STypes.ShortRecord({
                prevId: Constants.HEAD,
                id: id,
                nextId: nextId,
                status: status,
                collateral: collateral,
                ercDebt: ercAmount,
                ercDebtRate: ercDebtRate,
                zethYieldRate: zethYieldRate,
                flaggerId: 0,
                tokenId: tokenId,
                updatedAt: LibOrders.getOffsetTimeHours()
            });
```
This struct will be specific to shorter address.

DittoEth allow a short to create an NFT representing a short order using ERC721Facet.sol#mintNFT() :
```solidity
function mintNFT(address asset, uint8 shortRecordId)
        external
        isNotFrozen(asset)
        nonReentrant
        onlyValidShortRecord(asset, msg.sender, shortRecordId) 
    {
        if (shortRecordId == Constants.SHORT_MAX_ID) {
            revert Errors.CannotMintLastShortRecord();
        } 
        //E fetch msg.sender shorts
        STypes.ShortRecord storage short = s.shortRecords[asset][msg.sender][shortRecordId];

        if (short.tokenId != 0) revert Errors.AlreadyMinted();

        s.nftMapping[s.tokenIdCounter] = STypes.NFT({
            owner: msg.sender,
            assetId: s.asset[asset].assetId,
            shortRecordId: shortRecordId
        });

        short.tokenId = s.tokenIdCounter;

        //@dev never decreases
        s.tokenIdCounter += 1;
    }
```
DittoEth also allow an NFT short owner to transfer this NFT to another address by using function transferFrom(address from, address to, uint256 tokenId) public also from ERC721Facet.sol.

Process to transfer an NFT is like this : LibShortRecord.transferShortRecord(asset, from, to, uint40(tokenId), nft) :

deleteShortRecord(asset, from, nft.shortRecordId) => set s.shortRecords[asset][oldOwner][id].status to SR.Cancelled
createShortRecord() => create a new struct in storage with new owner : s.shortRecords[asset][newOwner][id]
Regarding liquidation ,there is 2 process to liquidate a short that has a bad ratio :

MarginCallPrimaryFacet.sol#flagShort() and then wait 10 hours to liquidate using MarginCallPrimaryFacet.sol#liquidate()
MarginCallSecondartFacet.sol#liquidateSecondary() to liquidate shorters having a really bad ratio (< 1.5 time collateralized) both these function use s.assetUser[asset][m.shorter].shortRecordId to find which short to liquidate.
Regarding all this process,a malicious user which have a short with bad ratio could look at the mempool for flagShort() or liquidateSecondary() TX and front-run them by transferring the NFT just before so that the flag/liquidate TX would revert because of the checks on these functions : MarginCallPrimaryFacet.sol#flagShort() :
```solidity
function flagShort(address asset, address shorter, uint8 id, uint16 flaggerHint)
        external
        isNotFrozen(asset)
        nonReentrant
        onlyValidShortRecord(asset, shorter, id) //E @audit Revert if cancelled
    { 
        ...
        // code
        ..
    }
 function liquidate(
        address asset, //E asset The market that will be impacted
        address shorter, //E shorter Shorter getting liquidated
        uint8 id, //E id Id of short getting liquidated
        uint16[] memory shortHintArray 
    )
        external
        isNotFrozen(asset)
        nonReentrant
        onlyValidShortRecord(asset, shorter, id) //E @audit Revert if cancelled
    {
        ...
        // code
        ..
    }
```
Both these functions have the onlyValidShortRecord() modifier which is in AppStorage.sol :

```solidity
    function _onlyValidShortRecord(address asset, address shorter, uint8 id)
        internal
        view
    {
        uint8 maxId = s.assetUser[asset][shorter].shortRecordId; //E 
        if (id >= maxId) revert Errors.InvalidShortId();
        if (id < Constants.SHORT_STARTING_ID) revert Errors.InvalidShortId(); 
        if (s.shortRecords[asset][shorter][id].status == SR.Cancelled) {  //E @audit revert if cancelled
            revert Errors.InvalidShortId();
        }
    }   

    modifier onlyValidShortRecord(address asset, address shorter, uint8 id) {
        _onlyValidShortRecord(asset, shorter, id);
        _;
    }
```
And revert if short is canceled (or transferred)

Regarding the second method : MarginCallSecondartFacet.sol#liquidateSecondary()
```solidity
function liquidateSecondary(..)
        STypes.AssetUser storage AssetUser = s.assetUser[asset][msg.sender];
        ...
        //code
        ...
        for (uint256 i; i < batches.length;) {
            // If ineligible, skip to the next shortrecord instead of reverting
            if (
                m.shorter == msg.sender || m.cRatio > secondaryLiquidationCR
                    || m.short.status == SR.Cancelled //E @audit would continue !
                    ||... other check
            ) {
                continue;
            }
            ...
            //code
            ...
        }
```
continue to next short to liquidate if short to liquidate is cancelled.

### Impact
We can see that if a short is cancelled (or transferred) it won't be possible to liquidate it in any way, so a malicious user could indefinitely transfer its bad collaterallized NFT each time someone wants to liquidate it.

### Tools Used
VSCode

### Recommendations
Add a timelock that prevent a transferred short to be re-transferred during 24 hours so that liquidator could liquidate it if necessary