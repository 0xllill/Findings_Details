## Summary
The UniV3SwapInput() function in the USSD contract permits a user to perform a token swap on Uniswap V3 with no minimum amount for the output token. This could lead to undesirable financial consequences for users if the swap executed has an excessive slippage, which can occur when large orders are filled or during times of high market volatility.

## Vulnerability Detail
The UniV3SwapInput() function allows a swap to be performed via Uniswap V3 by specifying the swap path (_path) and the amount of the input token (_sellAmount). The issue arises from the amountOutMinimum parameter being hardcoded as 0, which essentially implies that there is no lower bound on the amount of the output token that will be received in return for the _sellAmount of the input token. This might potentially lead to massive losses as there is no mechanism to prevent trades that would result in receiving an exceedingly low amount of the output token.

Typically, a amountOutMinimum parameter is used to prevent excessive slippage. This parameter sets a floor for the amount of the output token that will be received. If the trade cannot be executed with at least this amount of the output token, the trade will revert. However, with amountOutMinimum set to 0, a user could potentially swap a large number of tokens and receive nearly nothing in return.

This issue becomes even more significant when considering that this function is called by other rebalancing mechanisms to maintain the USSD stablecoin pegged to $1. Therefore, the stability and value of the USSD token are at risk due to this issue.

## Impact
This vulnerability is of high severity. If exploited or simply by calling the rebalancing functions of USSDRebalancer.sol, it lead to significant losses for individual users and destabilize the overall system, threatening the value and stability of the USSD token.

## Code Snippet
https://github.com/sherlock-audit/2023-05-USSD/main/ussd-contracts/contracts/USSD.sol#L237
```solidity
    function UniV3SwapInput( //E used in USSDRebalancer.sol
        bytes memory _path,
        uint256 _sellAmount
    ) public override onlyBalancer {
        
        IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
            .ExactInputParams({
                path: _path, //E path to swap tokens
                recipient: address(this),
                //deadline: block.timestamp,
                amountIn: _sellAmount,
                amountOutMinimum: 0 // @audit-issue slippage
            });
        uniRouter.exactInput(params); //E performs the swap operation as specified by the params.
    }
```
## Tool used
Manual Review

## Recommendation
To fix this vulnerability, it's recommended to add an input parameter for amountOutMinimum to the UniV3SwapInput() function. This parameter should be calculated based on a permitted slippage rate, which can be defined as a contract-wide constant or set by users on a per-call basis.
```solidity
function UniV3SwapInput(
    bytes memory _path,
    uint256 _sellAmount,
    uint256 _amountOutMinimum
) public override onlyBalancer {
    IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
        .ExactInputParams({
            path: _path,
            recipient: address(this),
            amountIn: _sellAmount,
            amountOutMinimum: _amountOutMinimum
        });
    uniRouter.exactInput(params);
}
```
In addition, careful calculations should be made when calling this function from other functions to ensure that the amount of the output token will not be excessively low. It's important to consider market volatility, trade size, and the liquidity of the involved tokens in these calculations.