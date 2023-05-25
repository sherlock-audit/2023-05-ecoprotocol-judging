stopthecap

medium

# STOPTHECAP  - No Storage Gap in CrossDomainEnabledUpgradeable.sol

## Summary
The `CrossDomainEnabledUpgradeable` contract does miss a storage gap. 

https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/CrossDomainEnabledUpgradeable.sol#L1-L89

## Vulnerability Detail

As an example to show the severity, the `L1ECOBridge` contract inherits `CrossDomainEnabledUpgradeable`, and the `CrossDomainEnabledUpgradeable` contract does not contain any storage gap. If in a future upgrade, an additional variable is added to the `CrossDomainEnabledUpgradeable` contract, that new variable will overwrite the storage slot of the stopped variable in the `L1ECOBridge` contract, causing unintended consequences.

Several past reports correctly judge from past contests, for getting a better vision of the issue.

https://solodit.xyz/issues/3341
https://solodit.xyz/issues/2370
https://solodit.xyz/issues/2504

## Impact

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts.

## Code Snippet
https://github.com/sherlock-audit/2023-05-ecoprotocol/blob/main/op-eco/contracts/bridge/CrossDomainEnabledUpgradeable.sol

## Tool used

Manual Review

## Recommendation
Add a gap in the `CrossDomainEnabledUpgradeable` contract

                            uint256[50] __gap; // gap to reserve storage in the contract for future variable additions