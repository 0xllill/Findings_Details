## Lines of code
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/StabilityPod/StabilizerNode.sol#L178-L182
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/StabilityPod/StabilizerNode.sol#L294-L298
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/StabilityPod/StabilizerNode.sol#L188

## Vulnerability details
## Impact 
priceTarget is inconsistent in StabilizerNode.stabilize so stabilize can do auction instead of selling malt and vice versa.

## Proof of Concept
In StabilizerNode.stabilize, there is an early check using _shouldAdjustSupply function.

    if (!_shouldAdjustSupply(exchangeRate, stabilizeToPeg)) {
      lastStabilize = block.timestamp;
      impliedCollateralService.syncGlobalCollateral();
      return;
    }
In _shouldAdjustSupply, priceTarget is calculated by stabilizeToPeg and then check if exchangeRate is outside of some margin of priceTarget.

    if (stabilizeToPeg) {
      priceTarget = maltDataLab.priceTarget();
    } else {
      priceTarget = maltDataLab.getActualPriceTarget();
    }
But in stabilize, priceTarget is always actual price target of maltDataLab regardless of stabilizeToPeg.
And it decides selling malt or doing auction by the priceTarget. So when stabilizeToPeg is true, priceTarget (= actual price target) can be different from maltDataLab.priceTarget() in most cases, and it can cause wrong decision of selling or starting auction after that.

    uint256 priceTarget = maltDataLab.getActualPriceTarget();
So when stabilizeToPeg is true, stabilize can do auction instead of selling malt, or vice versa.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Use same logic as _shouldAdjustSupply for priceTarget. priceTarget should be maltDataLab.priceTarget() in stabilize when stabilizeToPeg is true.