stopthecap

medium

# STOPTHECAP - missing EIP712Upgradeable.__EIP712_init on L2ECO.initialize

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