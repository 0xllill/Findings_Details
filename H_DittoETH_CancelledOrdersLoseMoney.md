cancelled Orders owners via cancelOrderFarFromOracle lose their money

### Severity
Medium Risk

### Summary
DittoEth has a mechanism to handle the possibility that orderId has hit limit, it is used to deter attackers, which is good. This mechanism is the function in OrdersFacets.sol#cancelOrderFarFromOracle(), and the only check for this function is that orderId has to be bigger than 65000. So if an attacker creates a lot of orders or during big usage of the protocol lot orders are created and orderId becomes bigger than 65000 any order with orderId > 65000 at the TAIL can be cancelled (attackers and normal users orders), the problem is this mechanisms does not include reimbursements of money to place these orders. If an attacker creates 65000 orders and then a normal user create the 65001 , attacker can immediately cancel it's order and normal user money is losen.

### Vulnerability Details
Here is the code of OrdersFacet.sol#cancelOrderFarFromOracle() :
```solidity
    //@dev public function to handle when orderId has hit limit. Used to deter attackers
    function cancelOrderFarFromOracle(
        address asset,
        O orderType,
        uint16 lastOrderId,
        uint16 numOrdersToCancel
    ) external onlyValidAsset(asset) nonReentrant {
        if (s.asset[asset].orderId < 65000) {
            revert Errors.OrderIdCountTooLow();
        }

        if (numOrdersToCancel > 1000) {
            revert Errors.CannotCancelMoreThan1000Orders();
        }

        if (msg.sender == LibDiamond.diamondStorage().contractOwner) {
            if (
                orderType == O.LimitBid
                    && s.bids[asset][lastOrderId].nextId == Constants.TAIL
            ) {
                s.bids.cancelManyOrders(asset, lastOrderId, numOrdersToCancel);
            } else if (
                orderType == O.LimitAsk
                    && s.asks[asset][lastOrderId].nextId == Constants.TAIL
            ) {
                s.asks.cancelManyOrders(asset, lastOrderId, numOrdersToCancel);
            } else if (
                orderType == O.LimitShort
                    && s.shorts[asset][lastOrderId].nextId == Constants.TAIL
            ) {
                s.shorts.cancelManyOrders(asset, lastOrderId, numOrdersToCancel);
            } else {
                revert Errors.NotLastOrder();
            }
        } else {
            //@dev if address is not DAO, you can only cancel last order of a side
            if (
                orderType == O.LimitBid
                    && s.bids[asset][lastOrderId].nextId == Constants.TAIL
            ) {
                s.bids.cancelOrder(asset, lastOrderId);
            } else if (
                orderType == O.LimitAsk
                    && s.asks[asset][lastOrderId].nextId == Constants.TAIL
            ) {
                s.asks.cancelOrder(asset, lastOrderId);
            } else if (
                orderType == O.LimitShort
                    && s.shorts[asset][lastOrderId].nextId == Constants.TAIL
            ) {
                s.shorts.cancelOrder(asset, lastOrderId);
            } else {
                revert Errors.NotLastOrder();
            }
        }
    }
```

and here is the code of libOrders.sol#cancelOrder() :

```solidity
    function cancelOrder(
        mapping(address => mapping(uint16 => STypes.Order)) storage orders,
        address asset,
        uint16 id
    ) internal {
        // save this since it may be replaced
        uint16 prevHEAD = orders[asset][Constants.HEAD].prevId;

        // remove the links of ID in the market
        // @dev (ID) is exiting, [ID] is inserted
        // BEFORE: PREV <-> (ID) <-> NEXT
        // AFTER : PREV <----------> NEXT
        orders[asset][orders[asset][id].nextId].prevId = orders[asset][id].prevId;
        orders[asset][orders[asset][id].prevId].nextId = orders[asset][id].nextId;

        // create the links using the other side of the HEAD
        emit Events.CancelOrder(asset, id, orders[asset][id].orderType);
        _reuseOrderIds(orders, asset, id, prevHEAD, O.Cancelled);
    }

    // shared function for both canceling and order and matching an order
    function _reuseOrderIds(
        mapping(address => mapping(uint16 => STypes.Order)) storage orders,
        address asset,
        uint16 id,
        uint16 prevHEAD,
        O cancelledOrMatched
    ) private {
        // matching ID1 and ID2
        // BEFORE: HEAD <- <---------------- HEAD <-> (ID1) <-> (ID2) <-> (ID3) <-> NEXT
        // AFTER1: HEAD <- [ID1] <---------- HEAD <-----------> (ID2) <-> (ID3) <-> NEXT
        // AFTER2: HEAD <- [ID1] <- [ID2] <- HEAD <---------------------> (ID3) <-> NEXT

        // @dev mark as cancelled instead of deleting the order itself
        orders[asset][id].orderType = cancelledOrMatched;
        orders[asset][Constants.HEAD].prevId = id;
        // Move the cancelled ID behind HEAD to re-use it
        // note: C_IDs (cancelled ids) only need to point back (set prevId, can retain nextId)
        // BEFORE: .. C_ID2 <- C_ID1 <--------- HEAD <-> ... [ID]
        // AFTER1: .. C_ID2 <- C_ID1 <- [ID] <- HEAD <-> ...
        if (prevHEAD != Constants.HEAD) {
            orders[asset][id].prevId = prevHEAD;
        } else {
            // if this is the first ID cancelled
            // HEAD.prevId needs to be HEAD
            // and one of the cancelled id.prevID should point to HEAD
            // BEFORE: HEAD <--------- HEAD <-> ... [ID]
            // AFTER1: HEAD <- [ID] <- HEAD <-> ...
            orders[asset][id].prevId = Constants.HEAD;
        }
    }
```

As you can see nowhere the order owner of the canceled order is refunded but orders[asset][id].orderType is set to O.Cancelled which prevent any user from getting back it's money.

The process is different when a user intentionally cancel it's own order , here is an example of an Ask cancellation :

```solidity
    function cancelAsk(address asset, uint16 id)
        external
        onlyValidAsset(asset)
        nonReentrant
    {
        STypes.Order storage ask = s.asks[asset][id];
        if (msg.sender != ask.addr) revert Errors.NotOwner();
        O orderType = ask.orderType;
        if (orderType == O.Cancelled || orderType == O.Matched) {
            revert Errors.NotActiveOrder();
        }

        s.assetUser[asset][msg.sender].ercEscrowed += ask.ercAmount; //E @audit user refunded
 
        s.asks.cancelOrder(asset, id);
    }
```
As you can see in this case user is refunded

### Impact
As there is no possibility to see if an orderId > 65000 correspond to an attacker or a normal user, OrdersFacets.sol#cancelOrderFarFromOracle() can be either used to deter an attacker but also to punish a normal user , the result is the same : cancelled order owner lose it's money.

### Tools Used
VSCode

### Recommendations
Refund cancelled order owner the same way an order owner is refunded when he/she intentionally cancel it's own order in the cancelOrderFarFromOracle() function. I think the protocol doesn't need this kind of function as user threshold for an order is 240 but if you want to deter attacker, lower this threshold or increase the minEthbid/ask/short threshold.