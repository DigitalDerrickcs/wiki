This documents version 2 of the Web3 Secret Storage Definition. 

## Definition

The actual encoding and decoding of the file remains largely unchanged from version 1, except that the crypto algorithm is no longer fixed to AES-128-CBC. Most of the meanings/algorithm are similar to version 1, except `mac`, which is given as the SHA3 of the concatenation of the last 16 bytes of the derived key together with the full `ciphertext`.

Secret key files are stored directly in `~/.web3/keys` (for Unix-like systems) and `~/AppData/Web3/keys` (for Windows). They may be named anything, but a good convention is `<uuid>.json`, where `<uuid>` is the 128-bit UUID given to the secret key (a privacy-preserving proxy for the secret key's address).

All such files have an associated password. To derive a given `.json` file's secret key, first derive the file's encryption key; this is done through taking the file's password and passing it through a key derivation function as described by the `kdf` key. KDF-dependent static and dynamic parameters to the KDF function are described in `kdfparams` key.

PBKDF2 must be supported by all minimally-compliant implementations, denoted though:

- `kdf`: `pbkdf2`

For PBKDF2, the `kdfparams` include:

- `prf`: Must be `hmac-sha256` (may be extended in the future);
- `c`: number of iterations;
- `salt`: salt passed to PBKDF;
- `dklen`: length for the derived key (must be 16, may be extended in the future).

One the file's key has been derived, it should be verified through the derivation of the MAC. The MAC should be calculated as the Keccak hash of the byte array formed as the concatenations of the rightmost 16 bytes with the `ciphertext` key's contents.

This value should be compared to the contents of the `mac` key; if they are different, an alternative password should be requested (or the operation cancelled).

After the file's key has been verified, the cipher text (the `ciphertext` key in the file) may be decrypted using the symmetric encryption algorithm specified by the `cipher` key and parameterised through the `cipherparams` key. If the derived key size and the algorithm's key size are mismatched, the zero padded, rightmost bytes of the derived key should be used as the key to the algorithm.

All minimally-compliant implementations must support the AES-128-CBC algorithm, denoted through:

- `cipher`: `aes-128-cbc`

This cipher takes the following parameters, given as keys to the `cipherparams` key:

- `iv`: 128-bit initialisation vector for the cipher.

The creation/encryption of a secret key should be essentially the reverse of these instructions. Make sure the `uuid`, `salt` and `iv` are actually random.

## Test Vector

Test vector using AES-128-CBC and PBKDF2-SHA-256:

File contents of `~/.web3/keys/3198bc9c-6672-5ab3-d9954942343ae5b6.json`:
```json
{
    "crypto" : {
        "cipher" : "aes-128-cbc",
        "cipherparams" : {
            "iv" : "db76cba2e79171364c5d5378190062c8"
        },
        "ciphertext" : "e8ae879315e349d8804a779421b26d95d2d3195fcf6c4039ee5323b858f53ef3",
        "kdf" : "pbkdf2",
        "kdfparams" : {
            "c" : 262144,
            "dklen" : 16,
            "prf" : "hmac-sha256",
            "salt" : "de5742f1f1045c402296422cee5a8a9ecf0ac5bf594deca1170d22aef33a79cf"
        },
        "mac" : "340aed34a9229210bfb7a5731268d78c68dba5e78a5a4eb87ff6e28bf2952735"
    },
    "id" : "3198bc9c-6672-5ab3-d995-4942343ae5b6",
    "version" : 2
}
```

Details:

- Address: `008aeeda4d805471df9b2a5b0f38a0c3bcba786b`
- ICAP: `XE542A5PZHH8PYIZUBEJEO0MFWRAPPIL67`
- UUID: `3198bc9c-6672-5ab3-d9954942343ae5b6`
- Password: `testpassword`
- Secret: `7a28b5ba57c53603b0b07b56bba752f7784bf506fa95edc395f5cf6c7514fe9d`

## Alterations from Version 1

This version fixes several inconsistencies with the version 1 published [here](https://github.com/ethereum/go-ethereum/wiki/Passphrase-protected-key-store-spec). In brief these are:

- Capitalisation is unjustified and inconsistent (`scrypt` lowercase, `Kdf` mixed-case, `MAC` uppercase).
- `Address` unnecessary and compromises privacy.
- `Salt` is intrinsically a parameter of the key derivation function and deserves to be associated with it, not with the crypto in general.
- `SaltLen` unnecessary (just derive it from `Salt`).
- The key derivation function is given, yet the crypto algorithm is hard specified. 
- `Version` is intrinsically numeric yet is a string (structured versioning would be possible with a string, but can be considered out of scope for a rarely changing configuration file format).
- KDF and cipher are notionally sibling concepts yet are organised differently.

Changes have been made to the format to give the following file, functionally equivalent to the example given on the previously linked page:

```json
{
    "crypto": {
        "cipher": "aes-128-cbc",
        "ciphertext": "07533e172414bfa50e99dba4a0ce603f654ebfa1ff46277c3e0c577fdc87f6bb4e4fe16c5a94ce6ce14cfa069821ef9b",
        "cipherparams": {
            "iv": "16d67ba0ce5a339ff2f07951253e6ba8"
        },
        "kdf": "scrypt",
        "kdfparams": {
            "dklen": 32,
            "n": 262144,
            "p": 1,
            "r": 8,
            "salt": "06870e5e6a24e183a5c807bd1c43afd86d573f7db303ff4853d135cd0fd3fe91"
        },
        "mac": "8ccded24da2e99a11d48cda146f9cc8213eb423e2ea0d8427f41c3be414424dd",
        "version": 1
    },
    "id": "0498f19a-59db-4d54-ac95-33901b4f1870",
    "version": 2
}
```