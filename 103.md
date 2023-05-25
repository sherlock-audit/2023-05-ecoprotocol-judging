stopthecap

high

# STOPTHECAP - Loss of user funds due to _gasLimit set to zero on L1ECOBridge

## Summary

Loss of user funds due to _gasLimit set to zero on L1ECOBridge

## Vulnerability Detail

In `L1ECOBridge.finalizeERC20Withdrawal`, there exists an issue related to the gas limit setting in case of a withdraw refund. 

The function `L1ECOBridge.finalizeERC20Withdrawal` is designed to finalize an ERC20 token withdrawal from L2 to L1. In the event of a failed transfer, it attempts to create a return transaction to refund the user on L2.

However, in case the transfer reverts or returns false, it will attempt to call `sendCrossDomainMessage`  with `_gasLimit` set to 0. Depending on the size of `_data`, the gas cost on L2 can prevent the refund from happening, leading to potential user loss of funds. 

As the Optimism documentation states ([1](https://community.optimism.io/docs/developers/bedrock/differences/#deposits-from-ethereum-to-optimism), [2](https://community.optimism.io/docs/developers/bridge/messaging/#fees-for-sending-data-between-l1-and-l2), [3](https://community.optimism.io/docs/developers/build/transaction-fees/#sending-transactions)):

> In order to prevent the Optimism network from being DOSed via forced L1 to L2 transactions that bypass the Sequencer, a fee adjustment schedule to all L1→L2 transactions that closely mimics EIP1559 is included with Bedrock. Like in the current network, deposit fees are paid by burning some amount of L1 gas proportional to your deposit's L2 gas limit. Unfortunately, this means that you may have cases where you estimate how much gas an L1→L2 deposit will cost, and deposit fees increase by the time your transaction gets included in a block and executed, causing your deposit to run out of gas and revert. This is why we recommend adding a 50% buffer to your gasLimit to ensure your deposit will not run out of gas.

> The majority of the cost of an L1 to L2 transaction comes from sending a transaction on Ethereum. You send a transaction to the L1 CrossDomainMessenger contract, which then sends a call to the CanonicalTransactionChain. This cost is ultimately determined by gas prices on Ethereum when you're sending the cross-chain transaction. An L1 to L2 message is expected to trigger contract execution on L2, and that contract execution costs gas. The first 1.92 million gas on L2 is free. The vast majority of L1 to L2 calls spend less than the 1.92 million, so nothing further is required.


> The process of sending a transaction on Optimism is identical to the process of sending a transaction on Ethereum. When sending a transaction, you should provide a gas price greater than or equal to the current L2 gas price. Like on Ethereum, you can query this gas price with the eth_gasPrice RPC method. Similarly, you should set your transaction gas limit in the same way that you would set your transaction gas limit on Ethereum (e.g. via eth_estimateGas).

## Impact

User loss of funds in the event a withdraw fails from L2 to L1

## Code Snippet

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L1ECOBridge.sol#L280

## Tool used

Manual Review

## Recommendation

Correctly estimate the gas limit for the refund and add a 50% buffer to the estimated gas limit returned by estimateGas to ensure that your transaction will not run out of gas. 