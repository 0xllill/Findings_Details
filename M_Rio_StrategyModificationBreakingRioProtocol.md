medium

## If thirdPartyTransfersForbidden[strategy] == true, withdrawals will be DoS'd
## Summary
The EigenLayer strategyWhitelister can arbitrarily set thirdPartyTransfersForbidden[strategy] for any strategy. If marked as true, queueWithdrawal will not allow withdrawals where the withdrawer is not the caller, which is the case for every queueWithdrawal call in the system.

## Vulnerability Detail
In the EigenLayer docs, it states that, "The strategyWhitelister can disable third party transfers for a given strategy. If thirdPartyTransfersForbidden[strategy] == true:

[...]
Users cannot withdraw on behalf of someone else. "
In RioLRTOperatorDelegator we call queueWithdrawals in two different circumstances, with both of them setting the withdrawer address as a different contract. In the case that the strategyWhitelister disables third party transfers for a strategy supported by a Rio LRT, withdrawals will be permanently disabled.

## Impact
Permanently disabled withdrawals.

## Code Snippet
https://github.com/sherlock-audit/2024-02-rio-network-core-protocol/blob/main/rio-sherlock-audit/contracts/restaking/RioLRTOperatorDelegator.sol#L213

## Tool used
Manual Review

## Recommendation
Consider making queued withdrawals go to the same contract which they're called from and then transfer them to the intended contract when completed.
