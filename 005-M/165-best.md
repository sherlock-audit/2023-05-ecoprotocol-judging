branch_indigo

medium

# L2 Transfer Likely Reverts Due to Insufficient Approved Allowance During Rebase

## Summary
In L2ECO, the accounting system used for allowance and balances is different. This likely causes a break between the two accounting during rebase.
## Vulnerability Detail
In L2ECO, token allowance approved for spenders is accounted in scaled token balances (after inflationMultiplier is applied). Whereas, all token balances for users are accounted in unscaled token balances in storage. This is fine when inflationMultiplier stays the same. However, when inflationMultiplier changes, users might approve the sender an amounted denominated in scaled token balances ( scaled with the old inflation multiplier). And the spender would transfer the users balance through `transferFrom` with the same scaled token amount input as user approval transaction, but under the hood, `transferFrom` checks required user balance with unscaled token amount using the new inflation multiplier. 

When inflation multiplier increases, the calculated unscaled token amount in `transferFrom` is larger than previous unscaled token amount using the old inflation multiplier. This is likely causing token transfer to revert. And this also open up potential opportunity for malicious centralized sequencer attack. When the sequencer is decentralized, MEV attack on L2 is possible as well.

[Here's a test.](https://gist.github.com/bzpassersby/20fed702d2498bf3c5b9a60e8da9b6f8)
## Impact
When the inflation multiplier increases, user transfer is likely reverted due to insufficient balance, this potentially give exploiters opportunity to perform MEV attack for profits in an AMM context.
## Code Snippet
[https://github.com/eco-association/op-eco/blob/39f205fb46eea3df770f0119a890ffdc1ac8f382/contracts/token/ERC20Upgradeable.sol#L242](https://github.com/eco-association/op-eco/blob/39f205fb46eea3df770f0119a890ffdc1ac8f382/contracts/token/ERC20Upgradeable.sol#L242)
## Tool used

Manual Review

## Recommendation
Use the same accounting for allowance as well. 