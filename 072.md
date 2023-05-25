stopthecap

medium

# STOPTHECAP - L2 ECO BRIDGE does not check that msg.sender is not an EOA for withdrawing to L1 leading to loss of funds

## Summary
Eco does not follow their structure in both bridges, meanwhile in L1 they are checking that msg.sender == EOA for `depositERC20` , this is not the case in the function `withdraw` in the L2 bridge which it does facilitate bridging assets to the wrong address in L1, losing the bridged tokens

## Vulnerability Detail

While bridging assets through chains, you have to be extremely careful the address that you use as the receiver in the other network. In this case, imagine smart contract "A" does have ECO token funds in Optimism and wants to bridge them to Mainnet.

The current implementation: 

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L2ECOBridge.sol#L126

does automatically take the address of contract "A" and bridges the funds to the same contract "A" address in mainnet. The issue is that this contract/address is super unlikely that also exists in mainnet, only if the nonce of the creator matched in both chains while deploying, meaning the funds would have been sent to an arbitrary address where you will not be able to rescue them.  

## Impact

By automatically adding the receiver or `to` field as msg.sender, smart contracts that try to bridge through the function, will most likely loss all the bridged funds.

## Code Snippet
https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L1ECOBridge.sol#L196

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L2ECOBridge.sol#L125

## Tool used

Manual Review

## Recommendation

Follow the pattern that you implemented in the L1 bridge to only allow EOA to bridge through the function that automatically adds msg.sender as the receiver:

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/L1ECOBridge.sol#L196
