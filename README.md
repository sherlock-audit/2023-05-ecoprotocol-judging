# Issue M-1: STOPTHECAP 

Source: https://github.com/sherlock-audit/2023-05-ecoprotocol-judging/issues/28 

## Found by 
Dug, sashik\_eth, stopthecap
## Summary

The ECO token on L2 fails to initialize the contract EIP712Upgradeable leaving the name and version parameters of the TYPEHASH un-initialized.

## Vulnerability Detail

The ECO token on L2 it does inherit from the contract EIP712Upgradeable.

The EIP712Upgradeable must be initialized for it to be used correctly.

Specially, because it does set the name and version which is used to return the domain separator to avoid signature replay attacks on the signature recovery in the eip712 standard.

Missing to initialize the EIP712Upgradeable contract, leaves those values as default, returning a wrong typehash.

```solidity
function __EIP712_init(string memory name, string memory version) internal onlyInitializing {
        __EIP712_init_unchained(name, version);
    }

    function __EIP712_init_unchained(string memory name, string memory version) internal onlyInitializing {
        bytes32 hashedName = keccak256(bytes(name));
        bytes32 hashedVersion = keccak256(bytes(version));
        _HASHED_NAME = hashedName;
        _HASHED_VERSION = hashedVersion;
    }

```


## Impact
Using default values for name and version in the TYPEHASH does increase the chances of having a situation of signature replay.

## Code Snippet

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/token/L2ECO.sol#L26
https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/token/L2ECO.sol#L149

## Tool used

Manual Review

## Recommendation

Call the `__EIP712_init` function from inside the init function in L2ECO



## Discussion

**albertnbrown**

EIP712 is not implemented in any way in our contract so there is no security vulnerability here. It was included because we had in mind an inclusion of Permit later on.

**ctf-sec**

Can be a valid low

**0xffff11**

This is the current message from sherlocks website: 
<img width="412" alt="image" src="https://github.com/sherlock-audit/2023-05-ecoprotocol-judging/assets/123578292/65647d82-390a-483f-91be-8c55707001ff">

As it was flagged there, I include it as an issue. In this case if you would like to include Permit after but having the EIP712Upgradeable  un-init would leave name and version as default increasing the chances  of a reply attack on the permit.



**albertnbrown**

Yeah, we can submit the addition of Permit to confirm that EIP712 is initialized correctly.

**ogreckoner**

A fix to this issue is in this PR.
https://github.com/eco-association/op-eco/pull/37

**0xffff11**

Confirmed issue fixed in: https://github.com/eco-association/op-eco/pull/37

# Issue M-2: STOPTHECAP 

Source: https://github.com/sherlock-audit/2023-05-ecoprotocol-judging/issues/103 

## Found by 
Bahurum, Bauer, Dug, J4de, Kose, T1MOH, blackhole, nobody2018, sashik\_eth, scaraven, stopthecap, toshii
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



## Discussion

**ctf-sec**

It is difficult to set accurate gas limit on l2 for sure

but l2gas should not be set to 0

given that the L1 finalizeERC20Withdrawal normal transfer only revert in edge case such as when token is paused and

```solidity
sendCrossDomainMessage(l2TokenBridge, 0, message);
```

only triggers in edge case

a medium severity is well deserved!

**albertnbrown**

Yeah, I was chatting a bit with our Optimism liaison on best ways to calculate this. This was a weird one because it came from mirroring a code pattern from pre-bedrock contracts, but this functionality worked on Goerli. This is definitely a real issue, though.

**ogreckoner**

This issue has been fixed in this PR:
https://github.com/eco-association/op-eco/pull/39


