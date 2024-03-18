---
title: Understanding ERC-1271 - Standard Signature Validation
slug: understanding-erc-1271-standard-signature-validation
date: 2024-03-18T08:58:46+06:00
description: Standard way to verify a signature when the account is a smart contract
draft: false
tags:
  - erc
  - account-abstraction
ShowToc: true
cover:
  image:
---

Externally Owned Accounts (EOA) can sign messages or transactions with their associated private keys but currently contracts cannot. This ERC implements a standard way for the contracts to verify whether a signature on a behalf of a contract is valid this allows contracts to sign messages with the implementation of `isValidSignature(hash, signature)` function on the signing contract.

## Motivation

There are many contracts that want to utilize signed messages for validation of rights-to-move assets or other purposes. In order for these contracts to be able to support non EOA (smart contract addresses), this serves as a standard mechanism. A proper use case of this standard is ERC-4337 Smart Contract Wallets, DAOs, multisig wallets)

## Specification

This function should be implemented by contracts which desire to sign messages. Application wanting to support contract signatures should call this method if the signer is a contract.

```solidity
pragma solidity 0.8.25;

contract ERC1271 {
	// bytes4(keccak256("isValidSignature(bytes32,bytes)"))
  bytes4 constant internal MAGICVALUE = 0x1626ba7e;

  /**
   * @dev Should return whether the signature provided is valid for the provided hash
   * @param _hash      Hash of the data to be signed
   * @param _signature Signature byte array associated with _hash
   *
   * MUST return the bytes4 magic value 0x1626ba7e when function passes.
   * MUST NOT modify state (using STATICCALL for solc < 0.5, view modifier for solc > 0.5)
   * MUST allow external calls
   */
  function isValidSignature(
	  bytes32 _hash,
	  bytes memory _signature)
	  public view
	  returns (bytes4 magicValue);
}
```

## Security Considerations

Since there are no gas-limit expected for calling the `isValidSIgnature()` function, it is possible that some implementation will consume a large amount of gas. It is therefore important to not hardcode an amount of gas sent when calling this method on an external contract as it could prevent the validation of certain signatures.

Each contract implementing this method is responsible to ensure that the signature passes is indeed valid, otherwise catastrophic outcomes are to be expected.

## Reference Implementation

[EIP-1271 Reference Implementation](https://eips.ethereum.org/EIPS/eip-1271#reference-implementation)

```solidity
/**
 * @notice Verifies that the signer is the owner of the signing contract.
 */
function isValidSignature(
	  bytes32 _hash,
	  bytes calldata _signature
) external override view returns (bytes4) {
		// Validate signatures
	  if (recoverSigner(_hash, _signature) == owner) {
		    return 0x1626ba7e;
	  } else {
		    return 0xffffffff;
	  }
}

/**
 * @notice Recover the signer of hash, assuming it's an EOA account
 * @dev Only for EthSign signatures
 * @param _hash       Hash of message that was signed
 * @param _signature  Signature encoded as (bytes32 r, bytes32 s, uint8 v)
 */
function recoverSigner(
	  bytes32 _hash,
	  bytes memory _signature
) internal pure returns (address signer) {
	  require(_signature.length == 65, "SignatureValidator#recoverSigner: invalid signature length");

	  // Variables are not scoped in Solidity.
	  uint8 v = uint8(_signature[64]);
	  bytes32 r = _signature.readBytes32(0);
	  bytes32 s = _signature.readBytes32(32);

	  // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
	  // unique. Appendix F in the Ethereum Yellow paper (https://ethereum.github.io/yellowpaper/paper.pdf), defines
	  // the valid range for s in (281): 0 < s < secp256k1n ÷ 2 + 1, and for v in (282): v ∈ {27, 28}. Most
	  // signatures from current libraries generate a unique signature with an s-value in the lower half order.
	  //
	  // If your library generates malleable signatures, such as s-values in the upper range, calculate a new s-value
	  // with 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141 - s1 and flip v from 27 to 28 or
	  // vice versa. If your library also generates signatures with 0/1 for v instead 27/28, add 27 to v to accept
	  // these malleable signatures as well.
	  //
	  // Source OpenZeppelin
	  // https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/cryptography/ECDSA.sol

	  if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
	    revert("SignatureValidator#recoverSigner: invalid signature 's' value");
	  }

	  if (v != 27 && v != 28) {
	    revert("SignatureValidator#recoverSigner: invalid signature 'v' value");
	  }

	  // Recover ECDSA signer
	  signer = ecrecover(_hash, v, r, s);

	  // Prevent signer from being 0x0
	  require(
	    signer != address(0x0),
	    "SignatureValidator#recoverSigner: INVALID_SIGNER"
	  );

	  return signer;
}
```

Example implementation of a contract calling the `isValidSignature()` function on an external signing contract -

```solidity
function callERC1271isValidSignature(
	  address _addr,
	  bytes32 _hash,
	  bytes calldata _signature
) external view {
	  bytes4 result = IERC1271Wallet(_addr).isValidSignature(_hash, _signature);
	  require(result == 0x1626ba7e, "INVALID_SIGNATURE");
}
```

## Summary

- ERC 1271 provides a way for application to verify signatures related to wallet contracts / accounts:
  - Proof of account ownership
  - Off-chain messages
- Most if not all Account Abstraction implements ERC 1271
- ERC 1271 allows AA wallets to work with multitude of signature shemes
- ERC 1271 signature are valid at a certain point in time.
  - Not sure if any wallet providers offers “expiring signatures”
