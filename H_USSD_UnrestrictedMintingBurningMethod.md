## Summary
The mintRebalancer() and burnRebalancer() functions are public that allow the minting and burning of USSD tokens. There are no access controls placed on these functions, potentially allowing any address to mint or burn tokens at will.

## Vulnerability Detail
Both the mintRebalancer() and burnRebalancer() functions lack appropriate access control measures that restrict usage. As a result, any external user or contract can call these functions to arbitrarily inflate or deflate the total supply of USSD tokens.

## Impact
This vulnerability could lead to serious consequences, such as undermining the value of the stablecoin, causing price instability, and potentially causing loss of funds or trust for users. Any actor can inflate the token supply, which could lead to rampant inflation and a decline in token value. Similarly, anyone could burn tokens, leading to deflation and potential market manipulation and of course loss of funds for ussd users.

## Code Snippet
https://github.com/sherlock-audit/2023-05-USSD/main/ussd-contracts/contracts/USSD.sol#L204-L210
```solidity
    function mintRebalancer(uint256 amount) public override { //@audit onlyBalancer modifier missing
        _mint(address(this), amount);
    }

    function burnRebalancer(uint256 amount) public override { //@audit onlyBalancer modifier missing
        _burn(address(this), amount);
    }
```
## Tool used
Manual Review

## Recommendation
Add onlyBalancer() modifier to protect these functions from unwanted external call
```solidity
    function mintRebalancer(uint256 amount) public override onlyBalancer {
        _mint(address(this), amount);
    }

    function burnRebalancer(uint256 amount) public override onlyBalancer {
        _burn(address(this), amount);
    }
```