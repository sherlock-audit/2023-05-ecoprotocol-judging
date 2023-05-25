stopthecap

medium

# STOPTHECAP - Lack of Pausable functionality from in L2ECO, while L1 token is ERC20Pausable

## Summary

ECO fails to follow their circuit breaker by not adding pausable method in their L2 ECO token

## Vulnerability Detail

ECO has a circuit breaker to be prepared for any possible exploit: 

https://docs.eco.org/core-concepts/system-primitives/circuit-breaker

As described by them:

"The Circuit Breaker is comprised of 3 separate, individual pause functions. There is a pause function in the ECO.sol contract, a pause function in the ECOx.sol contract, and a pause function in the CurrencyGovernance.sol contract."

meaning that they rigthly implemented pausable methods in the L1 eco tokens.

You can check the implementation here:

https://etherscan.deth.net/address/0x8dBF9A4c99580fC7Fd4024ee08f3994420035727 

However, ECO fails to add pausable method to their L2 ECO token, causing a miss-match in the circuit breaker. If any unintended issue happens to the ECO ecosystem/contracts, the L2 ECO token would be unprotected as the other ones and it could be still traded. 

## Impact

ECO fails to consistency on their circuit braker, it this circuit is "proven" due to unintended consequences, it will fail to work as intended with the L2 ECO token, being able to be traded still, not like the other L1 ECO tokens that will be paused.

## Code Snippet

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/b440f89234b806f672b9e9ad24cf70c409964db5/op-eco/contracts/token/L2ECO.sol#L1-L303

## Tool used

Manual Review

## Recommendation

Make the L2 ECO token Pausable to follow the specifics of the circuit braker
