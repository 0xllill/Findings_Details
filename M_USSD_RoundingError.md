## Summary
The USSDRebalancer contract’s rebalance function, which is designed to adjust the value of the USSD/DAI pool back to a 1-to-1 ratio, may be susceptible to a rounding error vulnerability. This rounding error occurs when calculations with high decimal precision are performed.

## Vulnerability Detail
The rebalance function operates by checking the price of the USSD token in relation to the DAI token. If the USSD token's price deviates beyond a defined threshold, the function will take steps to correct this imbalance. These actions involve selling USSD for buying collateral (DAI) or buying and burning USSD for selling collateral (DAI). During these operations, the contract handles calculations involving large decimal values, which may lead to rounding errors.

The specific lines with potential rounding errors in the rebalance function are:

BuyUSSDSellCollateral((USSDamount - DAIamount / 1e12)/2); // Here
IUSSD(USSD).mintRebalancer(((DAIamount / 1e12 - USSDamount)/2) * 99 / 100);  // And here
In both lines, the contract performs division before multiplication, which could lead to the loss of significant figures.

Examples:

Using simplified numbers for the sake of clarity:

Assume that USSDamount = 100 and DAIamount = 200.

In the first line: (USSDamount - DAIamount / 1e12)/2

The contract first performs the operation DAIamount / 1e12, which is 200 / 1e12 = 0 (due to solidity's integer division).
Next, USSDamount - 0 = 100 and finally 100 / 2 = 50.
The intended operation might be (USSDamount - (DAIamount / 1e12))/2, but the actual operation due to solidity's integer division could lead to an error.

## Impact
The main impact of this vulnerability is potential imbalance between the USSD and DAI tokens in the pool. This could lead to incorrect calculations when buying or selling collateral, causing further price deviations and potential loss of funds.

## Code Snippet
https://github.com/sherlock-audit/2023-05-USSD/main/ussd-contracts/contracts/USSDRebalancer.sol#L92-L107
```solidity
function rebalance() override public {
      uint256 ownval = getOwnValuation(); //E Price of the USSD token in terms of the other token in the pool, likely DAI
      (uint256 USSDamount, uint256 DAIamount) = getSupplyProportion();
      //E if USSD < 0.99 DAI 
      if (ownval < 1e6 - threshold) {
        // peg-down recovery
        BuyUSSDSellCollateral((USSDamount - DAIamount / 1e12)/2); //@audit rounding errors
      } 
      //E if USSD > 1.01 DAI 
      else if (ownval > 1e6 + threshold) {
        // mint and buy collateral
        // never sell too much USSD for DAI so it 'overshoots' (becomes more in quantity than DAI on the pool)
        // otherwise could be arbitraged through mint/redeem
        // the execution difference due to fee should be taken into accounting too
        // take 1% safety margin (estimated as 2 x 0.5% fee)
        IUSSD(USSD).mintRebalancer(((DAIamount / 1e12 - USSDamount)/2) * 99 / 100);  //@audit rounding errors
        // mint ourselves amount till balance recover
        SellUSSDBuyCollateral();
      }
    }
```
## Tool used
Manual Review

## Recommendation
To mitigate this vulnerability, we recommend implementing a safer arithmetic logic for operations that involve high precision. It would be better to perform multiplications before divisions to maintain precision. However, watch out for potential overflow issues when performing multiplication first.

Consider using libraries like OpenZeppelin's SafeMath or even Solidity's native fixed-point libraries, as they are designed to handle these kinds of operations more safely.