### Severity
Medium Risk

### Summary
Future changes on deposit delay on rETH tokens would prevent DittoETH users to use deposit(), withdraw() and unstake() for BridgeReth, which would make its transfering and burning impractical, leading to user funds losses.

### Vulnerability Details
RocketPool rETH tokens has a deposit delay that prevents any user who has recently deposited to transfer or burn tokens. In the past this delay was set to 5760 blocks mined (aprox. 19h, considering one block per 12s). This delay can prevent DittoETH users from transfering if another user staked recently.

File: RocketTokenRETH.sol
```solidity
  // This is called by the base ERC20 contract before all transfer, mint, and burns
    function _beforeTokenTransfer(address from, address, uint256) internal override {
        // Don't run check if this is a mint transaction
        if (from != address(0)) {
            // Check which block the user's last deposit was
            bytes32 key = keccak256(abi.encodePacked("user.deposit.block", from));
            uint256 lastDepositBlock = getUint(key);
            if (lastDepositBlock > 0) {
                // Ensure enough blocks have passed
                uint256 depositDelay = getUint(keccak256(abi.encodePacked(keccak256("dao.protocol.setting.network"), "network.reth.deposit.delay")));
                uint256 blocksPassed = block.number.sub(lastDepositBlock);
                require(blocksPassed > depositDelay, "Not enough time has passed since deposit");
                // Clear the state as it's no longer necessary to check this until another deposit is made
                deleteUint(key);
            }
        }
    }
```
Any future changes made to this delay by the admins could potentially lead to a denial-of-service attack on the BridgeRouterFacet::deposit and BridgeRouterFacet::withdraw mechanism for the rETH bridge.

### Impact
Currently, the delay is set to zero, but if RocketPool admins decide to change this value in the future, it could cause issues. Specifically, protocol users staking actions could prevent other users from unstaking for a few hours. Given that many users call the stake function throughout the day, the delay would constantly reset, making the unstaking mechanism unusable. It's important to note that this only occurs when stake() is used through the rocketDepositPool route. If rETH is obtained from the Uniswap pool, the delay is not affected.
All the ETH swapped for rETH calling `BridgeReth::depositEth`` would become irrecuperable, leading to a user bank run on DittoETH to not be perjudicated of this protocol externalization to all the users that have deposited.

### Tools Used
Manual review.

### Recommendations
Consider modifying Reth bridge to obtain rETH only through the UniswapV3 pool, on average users will get less rETH due to the slippage, but will avoid any future issues with the deposit delay mechanism.