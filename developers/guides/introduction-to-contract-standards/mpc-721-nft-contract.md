# MPC-721 NFT Contract

A NFT (Non-Fungible Token) is a unique digital asset on the Partisia blockchain, the NFT standard is inspired from the guidelines outlined in the [Ethereum Improvement Proposal (EIP) 721](https://eips.ethereum.org/EIPS/eip-721). Unlike fungible tokens, each NFT possesses distinct characteristics and cannot be exchanged on a one-to-one basis with other tokens. This standard provides basic functionality to track, transfer and approve NFTs.

### Version 1 <a href="#version-1" id="version-1"></a>

A contract is detected as a valid NFT V1 Contract if: The following actions exists where names and types match exactly:

```rust
#[action(shortname=0x03)] transfer_from(from: Address, to:Address, token_id: u128)
#[action(shortname=0x05)] approve(approved: Option<Address>, token_id: u128)
#[action(shortname=0x07)] set_approval_for_all(operator: Address,approved: bool)
```

The root state struct has each of the following state fields itself or in a sub-struct that has a 1-1 composition with the root state struct where names and types match exactly:

```rust
name: String
symbol: String
owners: Map<u128, Address>
token_approvals: Map<u128, Address>
operator_approvals: Vec<OperatorApproval>
```

Where OperatorApproval is a struct with at least the following fields:

```rust
owner: Address
operator: Address
```

#### URI extension <a href="#uri-extension" id="uri-extension"></a>

This extension used to give each NFT a unique URI. The extension is optional.

An NFT contract is detected as a valid NFT Contract with URIs if: The root state struct has each of the following state fields itself or in a sub-struct that has a 1-1 composition with the root state struct where names and types match exactly:

```rust
uri_template: String
token_uri_details: Map<u128, [u8;NN]>
```

Where NN is any legal constant size.

The full URI for an NFT token is found by taking `uri_template` and replacing `{}` with the `token_uri_details` for the token. The `token_uri_details` are stored as a string right-padded with zeros. It is fixed length to ensure the contract state serialization has fixed gas cost.

For example if\
`uri_template` has the value `ipfs://{}` `token_uri_details[id]` has the value `bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi` (plus any zeros needed to fill the fixed size array) The token will have the following full URI `ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi`

### Version 2 <a href="#version-2" id="version-2"></a>

This version makes use of Avl Trees to store NFTs. This allows the contract to store many more NFTs with reduced gas cost.

A contract is detected as a valid NFT V2 Contract if: The following actions exists where names and types match exactly:

```rust
#[action(shortname=0x03)] transfer_from(from: Address, to:Address, token_id: u128)
#[action(shortname=0x05)] approve(approved: Option<Address>, token_id: u128)
#[action(shortname=0x07)] set_approval_for_all(operator: Address,approved: bool)
```

The root state struct has each of the following state fields itself or in a sub-struct that has a 1-1 composition with the root state struct where names and types match exactly:

```rust
name: String
symbol: String
owners: AvlTreeMap<u128, Address>
token_approvals: AvlTreeMap<u128, Address>
operator_approvals: AvlTreeMap<OperatorApproval, Unit>
```

Where Unit is an empty struct and OperatorApproval is a struct with the following fields:

```rust
owner: Address
operator: Address
```

#### URI extension <a href="#uri-extension_1" id="uri-extension_1"></a>

This extension used to give each NFT a unique URI. The extension is optional.

An NFT contract is detected as a valid NFT Contract with URIs if: The root state struct has each of the following state fields itself or in a sub-struct that has a 1-1 composition with the root state struct where names and types match exactly:

```rust
uri_template: String
token_uri_details: AvlTreeMap<u128, String>
```

The full URI for an NFT token is found by taking `uri_template` and replacing `{}` with the `token_uri_details` for the token.

For example if\
`uri_template` has the value `ipfs://{}` `token_uri_details[id]` has the value `bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi` The token will have the following full URI `ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi`

**References:**

[https://eips.ethereum.org/EIPS/eip-721](https://eips.ethereum.org/EIPS/eip-721)

[https://gitlab.com/partisiablockchain/language/contracts/defi/-/tree/main/nft?ref\_type=heads](https://gitlab.com/partisiablockchain/language/contracts/defi/-/tree/main/nft?ref_type=heads)
