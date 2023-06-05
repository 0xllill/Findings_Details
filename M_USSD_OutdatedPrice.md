## Summary
The getPriceUSD() function in the StableOracleWETH contract (and similar contracts stableOracleDAI.sol, stableOracleWBTC.sol ) is vulnerable to stale or outdated price data when fetching from priceFeed.latestRoundData() from chainlink oracles

## Vulnerability Detail
The getPriceUSD() function fetches the price of WETH in USD from the Chainlink oracle via the priceFeed.latestRoundData() call. However, this approach potentially leads to stale or outdated price data. The getPriceUSD() function assumes that the price returned by priceFeed.latestRoundData() is always up-to-date, which might not be the case. This is the same vulnerability as described in the issue #94.

## Impact
Inaccurate price data can lead to incorrect valuations of tokens. An attacker could exploit this to buy tokens at a lower price or sell them at a higher price than the market rate, potentially resulting in significant financial losses for the contract and its users.

## Code Snippet
https://github.com/sherlock-audit/2023-05-USSD/main/ussd-contracts/contracts/oracles/StableOracleDAI.sol#L48
```solidity
   function getPriceUSD() external view override returns (uint256) {
        ...
        (, int256 price, , , ) = priceFeedDAIETH.latestRoundData();
        // @audit-issue no check on the price data date
        return (wethPriceUSD * 1e18) / ((DAIWethPrice + uint256(price) * 1e10) / 2);
    }
```
https://github.com/sherlock-audit/2023-05-USSD/main/ussd-contracts/contracts/oracles/StableOracleWETH.sol#L23
```solidity
   function getPriceUSD() external view override returns (uint256) {
       ...
        (, int256 price, , , ) = priceFeed.latestRoundData();
        // @audit-issue no check on the price data date
        // chainlink price data is 8 decimals for WETH/USD
        return uint256(price) * 1e10;
    }
```
https://github.com/sherlock-audit/2023-05-USSD/main/ussd-contracts/contracts/oracles/StableOracleWBTC.sol#L23
```solidity
    function getPriceUSD() external view override returns (uint256) {
        //(uint80 roundID, int256 price, uint256 startedAt, uint256 timeStamp, uint80 answeredInRound) = priceFeed.latestRoundData();
        (, int256 price, , , ) = priceFeed.latestRoundData();
        // @audit-issue no check on the price data date
        return uint256(price) * 1e10;
    }
```
## Tool used
Manual Review

## Recommendation
Implement a mechanism to check whether the price data from priceFeed.latestRoundData() is stale or outdated. You could potentially do this by checking the timestamp of the last update and comparing it to the current time.
Consider using multiple price feeds and taking the median or mean to get a more accurate price.
Regularly update your price feed oracles to ensure that they are providing the most accurate and up-to-date data.
4; Consider introducing a fail-safe mechanism that halts transactions if the price data is determined to be stale or outdated.
=> Add the below check for returned data
```solidity
function getPriceUSD() external view override returns (uint256) {
      ....

        uint256 maxDelayTime = maxDelayTimes[DAI];
        if (maxDelayTime == 0) revert NO_MAX_DELAY(DAI);

        // chainlink price data is 8 decimals for WETH/USD, so multiply by 10 decimals to get 18 decimal fractional

        (uint80 roundID, int256 price, uint256 timestamp, uint256 updatedAt, ) = priceFeedDAIETH.latestRoundData();
        require(updatedAt >= roundID, "Stale price");
        require(timestamp != 0,"Round not complete");
        require(price > 0,"Chainlink answer reporting 0");
        if (updatedAt < block.timestamp - maxDelayTime) revert PRICE_OUTDATED(_token);
        // @audit-issue no check on the price data date
        return (wethPriceUSD * 1e18) / ((DAIWethPrice + uint256(price) * 1e10) / 2);
    }
```