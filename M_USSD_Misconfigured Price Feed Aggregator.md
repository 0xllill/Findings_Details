lil.eth

medium

## Misconfigured Price Feed Aggregator
## Summary
The StableOracleWBTC contract is incorrectly configured to use the ETH/USD Chainlink aggregator for fetching price data. Given the naming and the commented references to WBTC in the contract, it seems that the intended configuration was to use the WBTC/USD aggregator.

## Vulnerability Detail
In the StableOracleWBTC contract, the priceFeed aggregator is instantiated with the ETH/USD Chainlink aggregator (0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419) instead of the WBTC/USD aggregator (0xf4030086522a5beea4988f8ca5b36dbc97bee88c), as suggested by the comment. This means that all the prices returned by the getPriceUSD function are in reference to the price of ETH and not WBTC.

## Impact
The use of an incorrect price feed could have significant ramifications depending on how the price data is used within the wider system. This could cause erroneous valuations, mispricing, and improper balance calculations, which could be exploited by an attacker to their advantage or lead to a loss of user trust due to inaccurate price information.

## Code Snippet
https://github.com/sherlock-audit/2023-05-USSD/main/ussd-contracts/contracts/oracles/StableOracleWBTC.sol#L17
```solidity
    constructor() {
        priceFeed = AggregatorV3Interface(
            0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419  //@audit-issue ETH/USD
        ); 
    }
```
## Tool used
Manual Review

## Recommendation
The price feed aggregator in the constructor should be changed to use the appropriate WBTC/USD Chainlink aggregator. Ensure that the contract's code accurately represents the intended function and carefully check such crucial parameters during the development and audit processes.

constructor() {
    priceFeed = AggregatorV3Interface(
        0xf4030086522a5beea4988f8ca5b36dbc97bee88c  // WBTC/USD
    ); 
}