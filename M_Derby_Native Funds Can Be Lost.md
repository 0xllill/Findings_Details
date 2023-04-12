## Summary
When used with native funds XProvider#xTransferToController() doesn't check that amountToSendXChain argument from MainVault#rebalanceXChain() is equivalent to msg.value actually linked to the call.

## Vulnerability Detail
This can lead either to bloating or to underpaying of the cross-chain transfer depending on the mechanics that will be used to call MainVault#rebalanceXChain(). I.e. as two values can differ, and only one can be correct, the difference is a missing funds either to the first chain that call rebalanceXChain() or for the second one receiving funds.

## Impact
Net impact will be missing funds proportional to the difference between amountToSendXChain and msg.value.

## Code Snippet
MainVault.sol where msg.value is sent
https://github.com/sherlock-audit/blob/main/derby-yield-optimiser/contracts/MainVault.sol#L255
```solidity
function rebalanceXChain(uint256 _slippage, uint256 _relayerFee) external payable {
   .....
    vaultCurrency.safeIncreaseAllowance(xProvider, amountToSendXChain);
    IXProvider(xProvider).xTransferToController{value: msg.value}( 
      vaultNumber,
      amountToSendXChain,
      address(vaultCurrency),
      _slippage,
      _relayerFee
    );
   .....
  }
```

XProvider.sol where msg.value should be checked
https://github.com/sherlock-audit/2023-01-derby-nasri136/blob/main/derby-yield-optimiser/contracts/XProvider.sol#L321-L335
```solidity
  function xTransferToController(uint256 _vaultNumber,uint256 _amount,address _asset,uint256 _slippage,uint256 _relayerFee
  ) external payable onlyVaults {
    if (homeChain == xControllerChain) {
      IERC20(_asset).transferFrom(msg.sender, xController, _amount);
      IXChainController(xController).upFundsReceived(_vaultNumber);
    } else {
      xTransfer(_asset, _amount, xController, xControllerChain, _slippage, _relayerFee);
      pushFeedbackToXController(_vaultNumber, _relayerFee);
    }
  }
``` 
**Same happen for
MainVault#pushTotalUnderlyingToController() --> IXProvider(xProvider).pushTotalUnderlying{value: msg.value}(vaultNumber,    homeChain,underlying,totalSupply(),totalWithdrawalRequests);
MainVault#sendRewardsToGame() --> IXProvider(xProvider).pushRewardsToGame{value: msg.value}(vaultNumber, homeChain, rewards);

## Tool used
Manual Review
VSCode

## Recommendation
In order to maintain the uniform approach consider requiring that amount does exactly correspond to msg.value :
```solidity
XProvider.sol
  function xTransferToController(uint256 _vaultNumber,uint256 _amount,address _asset,uint256 _slippage,uint256 _relayerFee
  ) external payable onlyVaults {
+ require(amount == msg.value,"wrong amount");
    if (homeChain == xControllerChain) {
      IERC20(_asset).transferFrom(msg.sender, xController, _amount);
      IXChainController(xController).upFundsReceived(_vaultNumber);
    } else {
      xTransfer(_asset, _amount, xController, xControllerChain, _slippage, _relayerFee);
      pushFeedbackToXController(_vaultNumber, _relayerFee);
    }
  }
``` 
