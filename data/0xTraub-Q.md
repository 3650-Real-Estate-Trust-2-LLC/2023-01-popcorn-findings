1. Pool2SingleAssetCompounder.sol -> Adapter may be ruled incompatible for an adapter in which a single-hop trade router does not exist even
when a 2-step route does. This limits the applicability of adapters that would fail to initialize because no such hop exists. Consider
using an external contract for trade routers or allowing a user to submit their own

      tradePath[0] = rewardTokens[i];

      uint256[] memory amountsOut = IUniswapRouterV2(router).getAmountsOut(ERC20(asset).decimals() ** 10, tradePath);
      if (amountsOut[amountsOut.length] == 0) revert NoValidTradePath();


2. By default all tokens will be allowed until otherwise denied. This is bad opsec and should be limited at first and then more tokens
allowed later.

function _verifyToken(address token) internal view {
    if (
      (
        permissionRegistry.endorsed(address(0))
          ? !permissionRegistry.endorsed(token)
          : permissionRegistry.rejected(token)
      


3. MultiRewardEscrow.sol -> if fee amount is zero due to rounding the 1e18 in the calculation then an error NoFee should be thrown
 if (feePerc > 0) {
      uint256 fee = Math.mulDiv(amount, feePerc, 1e18);

      amount -= fee;
      token.safeTransfer(feeRecipient, fee);
    }

Similarly, Error NoFee(IERC20) is never used. If accidental then function setFees() should check that each fee is nonzero when executing a new fee, otherwise remove dead-code.



4. MultiRewardEscrow.sol -> block.timestamp can be manipulated by validators. Consider using block.number instead, with
duration equaling number_of_slots * seconds_per_slot (12)

5. AdapterBase.sol -> `withdraw()` and `redeem()` not tagged nonReentrant. While vault does, the adapter makes calls to external contracts
not owned by the protocol/user and so extra care should be applied









