# Smart Contract Binary Format

A Partisia Smart Contract utilizes three distinct binary formats, which are described in detail below.

* _RPC Format_: When an _action_ of the smart function is invoked, the payload is sent to the action as binary data. The payload identifies which action is invoked and the values for all parameters to the action.
* _State Format_: The _state_ of a smart contract is stored as binary data in the blockchain state. The state holds the value of all smart contract state variables.
* _ABI Format_: Meta-information about the smart contract is also stored as binary data, The ABI holds the list of available actions and their parameters and information about the different state variables.

### ABI Version changes <a href="#abi-version-changes" id="abi-version-changes"></a>

* Version **5.6** to **5.7**:
  * The list `NamedTypes` and `Hooks` in the `ContractAbi` must be ordered.
* Version **5.5** to **5.6**:
  * Added new type `[T;L]`: Fixed-sized arrays with any content type `T`.
  * Old representation of `[u8;L]` have been deprecated.
* Version **5.4** to **5.5**:
  * Smart contracts now support
* of `FnKind: 0x11`.
* Smart contracts now support
*
  * of `FnKind: 0x13`.
  * `FnKind 0x11` now requires a Shortname.
  * `FnKind 0x13` now requires a Shortname.
* Version **5.3** to **5.4**:
  * Added new `FnKind: 0x18` called `ZkExternalEvent`.
* Version **5.2** to **5.3**:
  * Added new type `AvlTreeMap`, a map whose content is not serialized to the wasm state format, but instead allows for lazy access to its contents.
* Version **5.1** to **5.2**:
  * Added new `FnKind: 0x17` called `ZkSecretInputWithExplicitType`.
  * Added `SecretArgument` field to `FnAbi` to support ZK inputs. Only present when `FnKind` is `ZkSecretInputWithExplicitType`.
  * `FnKind: 0x10` is now deprecated.
* Version **5.0** to **5.1**:
  * Added additional abi types: `U256`, `Hash`, `PublicKey`, `Signature`, `BlsPublicKey`, `BlsSignature`.
* Version **4.1** to **5.0**:
  * Added support for enum with struct items.
  * Changed `StructTypeSpec` to `NamedTypeSpec` which is either an `EnumTypeSpec` or `StructTypeSpec`. This means that there is an additional byte when reading the list of `NamedTypeSpec` in `ContractAbi`.
* Version **3.1** to **4.1**:
  * Added `Kind: FnKind` field to `FnAbi`.
  * Removed `Init` field from `ContractAbi`.
  * Added zero-knowledge related `FnKind`s, for use in Zk contracts.
* Version **2.0** to **3.1**:
  * Shortnames are now encoded as LEB128; `ShortnameLength` field have been removed.
  * Added explicit, easily extensible return result format.

### RPC Binary Format <a href="#rpc-binary-format" id="rpc-binary-format"></a>

The RPC payload identifies which action is invoked and the values for all parameters of the action.

To decode the payload binary format you have to know which argument types each action expects, since the format of the argument values depends on the argument type. The _ABI Format_ holds exactly the meta-data needed for finding types of the arguments for each action.

The RPC payload contains the short name identifying the action being called followed by each of the arguments for the action.

The short name of an action is an u32 integer identifier that uniquely identifies the action within the smart contract. The short name is encoded as [unsigned LEB128 format](https://en.wikipedia.org/wiki/LEB128#Unsigned_LEB128), which means that short names have variable lengths. It is easy to determine how many bytes a LEB128 encoded number contains by examining bit 7 of each byte.

The argument binary format depends on the type of the argument. The argument types for each action is defined by the contract, and can be read from the ABI format.

Only arrays of lengths between (including) 0 and 127 are supported. The high bit in length is reserved for later extensions.

For arguments with variable lengths, such as Vecs or Strings the number of elements is represented as a big endian 32-bit unsigned integer.

### State Binary Format <a href="#state-binary-format" id="state-binary-format"></a>

Integers are stored as little-endian. Signed integers are stored as two's complement. Note that lengths are also stored as little-endian.

Only arrays of lengths between (including) 0 and 127 are supported. The high bit in length is reserved for later extensions.

For arguments with variable lengths, such as Vecs or Strings the number of elements is represented as a little endian 32-bit unsigned integer.

**Note:** An AvlTreeMap is only stored in wasm state as an integer. The actual content of the AvlTreeMap is stored outside the wasm code. The content of the AvlTreeMap is accessed using external calls to the java code. The integer stored is the tree id of the AvlTreeMap and is used as a pointer to find the correct AvlTreeMap. This is done to avoid having to serialize and deserialize the entire content of the Map for every invocation saving gas cost.

#### CopySerializable <a href="#copyserializable" id="copyserializable"></a>

A state type is said to be CopySerializable, if it's serialization is identical to it's in-memory representation, and thus require minimal serialization overhead. PBC have efficient handling for types that are CopySerializable. Internal pointers are the main reason that types are not CopySerializable.

The WellAligned constraint on Struct CopySerializable is to guarantee that struct layouts are identical to serialization.

is WellAligned if following points hold:

1. Annotated with `#[repr(C)]`. Read the [Rust Specification](https://doc.rust-lang.org/reference/type-layout.html#reprc-structs) for details on this representation.
2. No padding:

No wasted bytes when stored in array:

It may be desirable to manually add "padding" fields structs in order to achieve CopySerializable. While this will use extra unneeded bytes for the serialized state, it may significantly improve serialization speed and lower gas costs. A future version of the compiler may automatically add padding fields.

**Examples**

These structs are CopySerializable:

* `Struct E1 { /* empty */ }`
* `Struct E2 { f1: u32 }`
* `Struct E3 { f1: u32, f2: u32 }`

These structs are not CopySerializable:

* `Struct E4 { f1: u8, f2: u16 }` due to padding between f1 and f2.
* `Struct E5 { f1: u16, f2: u8 }` due to alignment not dividing size.
* `Struct E6 { f1: Vec<u8> }` due to non-CopySerializable subfield.

### ABI Binary Format <a href="#abi-binary-format" id="abi-binary-format"></a>

#### Type Specifier binary format <a href="#type-specifier-binary-format" id="type-specifier-binary-format"></a>

**NOTE:** `Map` and `Set` cannot be used as RPC arguments since it's not possible for a caller to check equality and sort order of the elements without running the code.

Only arrays of lengths between (including) 0 and 127 are supported. The high bit in length is reserved for later extensions.

#### ABI File binary format <a href="#abi-file-binary-format" id="abi-file-binary-format"></a>

All `Identifier` names must be [valid Java identifiers](https://docs.oracle.com/javase/specs/jls/se20/html/jls-3.html#jls-3.8); other strings are reserved for future extensions.

Note that a `ContractAbi` is only valid if the `Hooks` list contains a specific number of hooks of each type, as specified in `FnKind`.

Also note that if a function has the deprecated kind `ZkSecretInput`, the default secret argument associated with it is of type i32.

**`NamedTypes` and `Hooks` order**

From version `5.7.0` the `NamedTypes` and `Hooks` list in `ContractAbi` must be ordered for the ABI to be valid.

The types in the `ContractAbi` can be viewed as one or more directed graphs. The roots for graph traversal are the `StateType` and each argument type for functions in the sorted `Hooks` list.

This implicitly means that if `StateType` is a `NamedTypeRef` it must have index 0.

For `NamedTypes` to be considered ordered the following must be true:

1. For each of the roots, traverse the type graph depth-first starting from the left-most type.
2. If the type has not been visited before add it to the `NamedTypes` list.

`Hooks` must be sorted by their `FnKind` identifier. If more than one function with the `FnKind` exists (e.g. `Action`), sort them by their shortname.

### Section format <a href="#section-format" id="section-format"></a>

A section is an indexed chunk of a binary file of dynamic length, which is defined as follows:

The id of a section is a single, leading byte that identifies the section. The section's length then follows and is given as a 32-bit, big-endian, unsigned integer. The byte length of the following dynamically sized data of the section should match this length.

### Wasm contract result format <a href="#wasm-contract-result-format" id="wasm-contract-result-format"></a>

The format used by Wasm contracts to return results is a [section format](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#section-format) defined as follows:

Note that sections must occur in order of increasing ids. Two ids are "well-known" and specially handled by the interpreter:

* `0x01`: Stores event information.
* `0x02`: Stores state.

Section ids `0x00` to `0x0F` are reserved for "well-known" usage. All others are passed through the interpreter without modification.

### ZKWA format <a href="#zkwa-format" id="zkwa-format"></a>

ZK-contracts have their own binary file format with the extension ".zkwa" that contains their compiled WASM code and ZK-circuit byte code. This is a [section format](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#section-format) defined as:

Note that sections must occur in order of increasing ids. The .zkwa format consists of two sections indexed by the following ids:

* `0x02`: The contract's WASM code.
* `0x03`: The contract's ZK-circuit byte code.

### Partisia Blockchain Contract File <a href="#partisia-blockchain-contract-file" id="partisia-blockchain-contract-file"></a>

The file extension of Partisia Blockchain Contract Files is written as ".pbc". This is a [section format](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#section-format) defined as:

Note that sections must occur in order of increasing ids. The .pbc format can consist of up to three sections indexed by the following ids:

* `0x01`: The contract's [ABI](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#abi-binary-format).
* `0x02`: The contract's WASM code.
* `0x03`: The contract's ZK-circuit byte code.
