lil.eth

medium

## Uninitialized Oracle Address
## Summary
The StableOracleDAI contract is incorrectly initialized with the ethOracle variable set to the zero address (0x0000000000000000000000000000000000000000), essentially meaning it is not properly configured. This likely represents an unfinished or improperly configured part of the contract.

## Vulnerability Detail
In the constructor of the StableOracleDAI contract, the ethOracle variable is initialized with the zero address. This means that when calling ethOracle.getPriceUSD(), the function call will fail as it attempts to call a function on an uninitialized contract address.

## Impact
The use of the uninitialized contract address for ethOracle will lead to a failure when getPriceUSD() is called, as it attempts to interact with a contract at the zero address. This issue can cause function calls to revert and could potentially halt the operations of the entire contract, leading to a loss of service availability.

## Code Snippet
https://github.com/sherlock-audit/2023-05-USSD/main/ussd-contracts/contracts/oracles/StableOracleDAI.sol#L30
```solidity
    constructor() {
        ...
        // @audit-issue oups TODO to do
        ethOracle = IStableOracle(0x0000000000000000000000000000000000000000); // TODO: WETH oracle price
    }
```
## Tool used
Manual Review

## Recommendation
The ethOracle variable should be properly initialized in the constructor with the correct address of the desired ETH oracle. Be sure to carefully check these crucial parameters during the development and audit processes to prevent such errors.
```solidity
constructor() {
    ...
    ethOracle = IStableOracle(0x5f4ec3df9cbd43714fe2740f5e3616155c5b8419); //ChainLink ETH/USD PriceFeed
}
```