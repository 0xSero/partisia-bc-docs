# Technical differences between Ethereum and Partisia Blockchain

This segment discusses the challenges that arise when transferring data between Partisia Blockchain and Ethereum, especially when using PBC as second layer. The main issues discussed are the differences in how data is encoded, the hashing algorithm used, how contract/account addresses are derived, and the encoding of signatures. It is placed as the last page to help explain some of the seemingly arbitrary technical details that may slow down development if not understood properly.

### How information is represented on PBC and Ethereum and how to convert between the two <a href="#how-information-is-represented-on-pbc-and-ethereum-and-how-to-convert-between-the-two" id="how-information-is-represented-on-pbc-and-ethereum-and-how-to-convert-between-the-two"></a>

One of the problems with transferring data between PBC and Ethereum is that the two chains take different approaches on how information is represented. Specifically, the differences relevant for using PBC as second layer are:

1. [How user data is encoded](https://partisiablockchain.gitlab.io/documentation/smart-contracts/pbc-as-second-layer/technical-differences-between-eth-and-pbc.html#encoding-data)
2. [The hashing algorithm used](https://partisiablockchain.gitlab.io/documentation/smart-contracts/pbc-as-second-layer/technical-differences-between-eth-and-pbc.html#hashing-data)
3. [How contract or account addresses are derived](https://partisiablockchain.gitlab.io/documentation/smart-contracts/pbc-as-second-layer/technical-differences-between-eth-and-pbc.html#deriving-addresses)
4. [The encoding of signatures](https://partisiablockchain.gitlab.io/documentation/smart-contracts/pbc-as-second-layer/technical-differences-between-eth-and-pbc.html#signatures)

### Encoding data <a href="#encoding-data" id="encoding-data"></a>

Solidity has two methods for encoding data into bytes, `abi.encode()` and `abi.encodePacked()`. The main difference between the two methods is that `abi.encode()` encodes dynamic types in a location separate from the static data, while `abi.encodePacked()` encodes all types in place. For a detailed overview of how they work, see the [solidity documentation](https://docs.soliditylang.org/en/latest/abi-spec.html).

Data received from PBC should be encoded using `abi.encodePacked()`, since this method is closest to the approach PBC uses.

Similarly, PBC also has two ways to encode or serialize data based on what they are used for. The main difference is that data that is stored in state is serialized in little endian order, and data that is serialized for RPC calls is serialized in big endian order. Read [the article about binary formats](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html) for more details.

Data that is meant to be sent to Ethereum must therefore be encoded as an argument for an RPC payload, which can be done by implementing the `WriteRPC` trait which defines the `rpc_write_to<T: Write(&self, writer: &mut T);` method.

To illustrate this, consider the two following code snippets using the code from the [voting example](https://partisiablockchain.gitlab.io/documentation/smart-contracts/pbc-as-second-layer/how-to-create-your-own-second-layer-solution.html).

```rust
#[test]
fn test_encoding() {
    let vote_id: u32 = 7;
    let votes_for: u32 = 51;
    let votes_against: u32 = 43;
    let abstaining: u32 = 4;
    let mut buffer: Vec<u8> = vec![];
    vote_id.rpc_write_to(buffer)?;
    votes_for.rpc_write_to(buffer)?;
    votes_against.rpc_write_to(buffer)?;
    abstaining.rpc_write_to(buffer)?;

    let expected: Vec<u8> = vec![
        0, 0, 0, 7, // vote_id
        0, 0, 0, 51, // votes_for
        0, 0, 0, 43, // votes_against
        0, 0, 0, 4 // abstaining
    ];
    assert_eq!(buffer, expected);
}
```

```rust
function testEncoding() {
    uint32 voteId = 7;
    uint32 votesFor = 51;
    uint32 votesAgainst = 43;
    uint32 missing = 4;
    bytes memory encoded = abi.encodePacked(voteId, votesFor, votesAgainst, missing);
    //                   | voteId|    for|against|missing|
    require(encoded == "0x00000007000000330000002B00000004");
}
```

The above illustrates that the values encoded above results in the same bytes in both contracts.

However, there is one difference between the two that must be considered. In solidity `abi.encodePacked()` encodes dynamic types in place without the data length. Rust also encodes dynamic types in place but prepends the data length.

So if we want to add a short description on the vote data we need to manually add the length in solidity. The following two code snippets illustrates this.

```rust
#[test]
fn test_encoding() {
    let vote_id: u32 = 7;
    let votes_for: u32 = 51;
    let votes_against: u32 = 43;
    let abstaining: u32 = 4;
    let description: String = "cats < dogs".to_string();
    let mut buffer: Vec<u8> = vec![];
    vote_id.rpc_write_to(buffer)?;
    votes_for.rpc_write_to(buffer)?;
    votes_against.rpc_write_to(buffer)?;
    abstaining.rpc_write_to(buffer)?;
    description.rpc_write_to(buffer)?;

    let expected: Vec<u8> = vec![
        0, 0, 0, 7, // vote_id
        0, 0, 0, 51, // votes_for
        0, 0, 0, 43, // votes_against
        0, 0, 0, 4, // abstaining
        0, 0, 0, 11, // description length
        99, 97, 116, 115, // "cats"
        32, 60, 32, // " < "
        100, 111, 103, 115 // "dogs
    ];
    assert_eq!(buffer, expected);
}
```

```rust
function testEncoding() {
    uint32 voteId = 7;
    uint32 votesFor = 51;
    uint32 votesAgainst = 43;
    uint32 missing = 4;
    string memory description = "cats < dogs";
    bytes memory encoded = abi.encodePacked(voteId, votesFor, votesAgainst, missing, description);
    //                   | voteId|    for|against|missing|         "cats < dogs"|
    require(encoded == "0x00000007000000330000002B0000000463617473203C20646F6773");

    uint32 length = bytes(description).length;
    bytes memory encoded = abi.encodePacked(voteId, votesFor, votesAgainst, missing, length, description);
    //                   | voteId|    for|against|missing| length|         "cats < dogs"|
    require(encoded == "0x00000007000000330000002B000000040000001163617473203C20646F6773");
}
```

The limitation described above for strings also applies when encoding other types of dynamic data.

### Hashing data <a href="#hashing-data" id="hashing-data"></a>

As a standard, Ethereum uses the Keccak-256 hashing algorithm, while PBC uses SHA-256 when signing data. This means that the same bytes results in different digests when hashing either on Ethereum or PBC. However, solidity also provides a SHA-256 method, which should be used whenever verifying data sent from PBC.

### Deriving Ethereum addresses <a href="#deriving-ethereum-addresses" id="deriving-ethereum-addresses"></a>

When using PBC as second layer we wish to know that the Ethereum addresses are for the nodes running the ZK computation. Based on the type of contract being written we may also wish to known what the Ethereum address corresponding to any PBC account address is.

There is no function that can convert one address type to the other since both address types are result of a one-way hashing function.

In order to derive the address for either Ethereum or PBC we need to know the public key corresponding to the accounts private key.

As described [here](https://ethereum.org/en/developers/docs/accounts/#account-creation) the address is derived by first generating the public key from the private key. Then the Ethereum address is the last 20 bytes of the Keccak-256 hash of the public key.

Specifically for the computation nodes, the public keys are stored in PBC state, and we can derive their Ethereum addresses from them, however there is a problem.

To understand the problem it is important to understand that both Ethereum and PBC uses [Elliptic Curve Cryptography](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography) (ECC) for providing signatures. This means that public keys are actually a point on an elliptic curve, and the point can be encoded in two different formats: compressed and uncompressed.

The uncompressed encoding is 64 bytes, 32 bytes for each coordinate in the point. As the name implies, the compressed encoding compresses the points, so they can be represented in fewer bytes.

When using PBC, public keys are saved in compressed encoding to optimize storage usage. However, when deriving the address from the public keys, Ethereum expects them to be in uncompressed format. Therefore, before deriving the addresses, the compressed public keys from PBC must be converted to an uncompressed 64-byte encoding. This conversion will ensure that Ethereum can derive the address from the public key.

### Deriving PBC addresses <a href="#deriving-pbc-addresses" id="deriving-pbc-addresses"></a>

Similar to Ethereum, PBC account addresses are calculated as the last 20 bytes of the hash of the public key.

Unlike Ethereum, the hashing algorithm used is SHA-256, and the address is prepended with the byte `0x00`, to signify the address is an account.

In total an address on PBC is 21 bytes long.

### Signatures <a href="#signatures" id="signatures"></a>

Finally, in order to be able to verify data signed by the ZK computation nodes, we need to ensure the signatures are on a format which Ethereum can decode.

Recall that Ethereum and PBC uses ECC and specifically the Elliptic Curve Digital Signature Algorithm (ECDSA), when signing messages.

Signatures outputted by the algorithm consists of three values; _r_ and _s_ which are both 32 bytes, and a _recovery\_id_ which is a single byte.

PBC stores these values as a single 65 bytes entity, on the format `recovery_id || s || r`. However, Ethereum expects that _recovery\_id_ is the last byte instead of the first, so it should be moved to the end of the signature `r || s || recovery_id`.

Additionally, on PBC the _recovery\_id_ can be either 0 or 1, but Ethereum expects it to be either 27 or 28. So we should add 27 to it before we can send to Ethereum.

The final transformation looks like `recovery_id || s || r => s || r || (recovery_id + 27)`.
