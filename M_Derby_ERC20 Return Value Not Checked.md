ERC20 return values not checked
## Summary
There are multiple ways of handling success or failure during transfer and approvals. While some tokens revert upon failure, others consistently return boolean flags or have mixed behaviors.

## Vulnerability Detail
The ERC20.transfer() and ERC20.transferFrom() functions return a boolean value indicating success. This parameter needs to be checked for success.
Some tokens do not revert if the transfer failed but return false instead.

## Code Snippet
See :
XProvider.sol Line 147 : `IERC20(_token).transferFrom(msg.sender, address(this), _amount);`
XProvider.sol Line 329 : `IERC20(_asset).transferFrom(msg.sender, xController, _amount);`
XProvider.sol Line 372 : `IERC20(_asset).transferFrom(msg.sender, _vault, _amount);`
XProvider.sol Line 575 : `IERC20(_token).transfer(xController, balance);`
XProvider.sol Line 584 : `IERC20(_token).transfer(vault, balance);`

Per example in the case of USDT, the transferFrom function does not return any value.

## Impact
Tokens that don't actually perform the transfer and return false are still counted as a correct transfer and the tokens remain in the contract and could potentially be stolen by someone else.
Furthermore, in function xTransfer tokens are approved before an xcall from Connext and in xTransferToController, tokens are accounted on the controller but in case of a failure these tokens doesn't exist on the Provider.

## Tool used
Manual Review
VS Code

## Recommendation
Use OpenZeppelin’s SafeERC20 versions that you are already using but replace Transfer with the safeTransfer and safeTransferFrom functions that handle the return value check as well as non-standard-compliant tokens.