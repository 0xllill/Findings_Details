## Summary
Front Run of addBlackList() function

## Vulnerability Detail
Front running can be done either by sending a tx with a higher gas price (usually tx are ordered in a block by the gas price / total fee), or by paying an additional fee to the validator if they manage to run their tx without reverting (i.e. by sending additional ETH to block.coinbase, hoping validator will notice it).

## Impact
Malicious user could listen the mempool in order to check if he sees a tx of blacklisting for his address , if it happens he could front run this tx by sending a tx with higher gas fee to transfer his funds to prevent them to be removed by removeBlackFunds() function

## Code Snippet
https://github.com/sherlock-audit/2023-02-telcoin/main/telcoin-audit/contracts/stablecoin/Stablecoin.sol#L159

```solidity
  function addBlackList(address holder) external onlyRole(BLACKLISTER_ROLE) override {
    if (blacklisted(holder)) revert AlreadyBlacklisted(holder);
    _blacklist[holder] = true;
    emit AddedBlacklist(holder);
    removeBlackFunds(holder);
  }
``

## Tool used
Manual Review in VSCode

## Recommendation
Use the same mechanism as in StakingModule.sol to prevent user from withdrawing their funds if blacklisted so that front running won't be useful