ast3ros

high

# The rebase call to set inflationMultiplier can be front-run malicious users to benefit from the difference in inflationMultiplier between L1 and L2

## Summary

When the bridges are deployed and initialized, there can be a difference in inflationMultiplier between L1ECOBridge and L2ECOBridge contracts. Malicious users can exploit this difference by depositing ECO tokens to the bridge and transferring them to L2 before the rebase call is executed.

## Vulnerability Detail

From the https://eco.org/manual.pdf: 

        Linear supply change — This mechanism allows the quantity of circulating tokens to be adjusted on a pro rata basis that maintains the relative wealth for all holders. This is a rebasing function with a `multiplier coefficient that may be either greater than or less than one`, enabling Eco Trustees to enact both supply-inflationary and deflationary policies. When a linear supply policy is enacted, a new scale factor is applied to individual wallet balances network-wide, effectively ‘minting’ or ‘burning’ new ECO on a per-wallet basis.

This means that depending on the supply policy at the time of bridge deployment, the INITIAL_INFLATION_MULTIPLIER can be greater than or less than one.

The deploy.ts (`op-eco/scripts/deploy.ts`) script deploys and initializes L1ECOBridge and L2ECOBridge. After the deployment and initialization are completed,

- L1ECOBridge: inflationMultiplier is set to current block inflationMultiplier -  `IECO(_l1Eco).getPastLinearInflation(block.number);`

        inflationMultiplier = IECO(_l1Eco).getPastLinearInflation(
            block.number
        );

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L1ECOBridge.sol#L129-L131

- L2ECOBridge: inflationMultiplier is set to the initial value - `l2Eco.INITIAL_INFLATION_MULTIPLIER()`: 1e18.

        inflationMultiplier = l2Eco.INITIAL_INFLATION_MULTIPLIER();

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L2ECOBridge.sol#L114

In this state, 

- case 1: if the current block `inflationMultiplier` in L1 is `less` than `1e18`, the amount of L2ECO received in L2 is less than the amount of ECO deposited in L1. For example: if inflationMutiplier is `0.9e18` and user deposits `1e18` ECO token, the amount received in L2 is 1e18 (amount deposit) * 0.9e18 (inflationMultiplier L1) / 1e18 (inflationMultiplier L2) = `0.9e18`.

- case 2: if the current block `inflationMultiplier` in L1 is `more` than `1e18`, the amount of L2ECO received in L2 is more than the amount of ECO deposited in L1. For example: if inflationMutiplier is `1.1e18` and user deposits `1e18` ECO token, the amount received in L2 is 1e18 (amount deposit) * 1.1e18 (inflationMultiplier L1) / 1e18 (inflationMultiplier L2) = `1.1e18`.

In case 2, a malicious user can front-run the rebase call and deposit ECO tokens to the bridge immediately to gain from the difference between inflationMultipliers. He then can call rebase to sync the inflationMultipliers, wait for other users to deposit ECO tokens into the L1ECOBridge and withdraw them to drain ECO tokens of other users from the bridge.

## Impact

If the current block inflationMultiplier in L1 is more than 1e18, malicious users can mint more L2ECO tokens in L2 than ECO tokens deposited in L1. They then can drain other users’ ECO tokens from the bridge.

If the current block inflationMultiplier in L1 is less than 1e18, users will lose ECO tokens if they deposit them to the bridge too early.

## Code Snippet

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L1ECOBridge.sol#L129-L131
https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L2ECOBridge.sol#L114
https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L1ECOBridge.sol#L335
https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L2ECOBridge.sol#L161

## Tool used

Manual Review

## Recommendation

Use a contract or multicall to deploy the bridges, initialize them and rebase the L2ECOBridge in one transaction.