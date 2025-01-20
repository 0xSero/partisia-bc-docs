# FAQ

### General Rust contracts <a href="#general-rust-contracts" id="general-rust-contracts"></a>

Questions for non-ZK contracts:

* **How does sharding and interactions work?**: PBC does not run in a single memory space, in contrast to EVM-based blockchains. Instead each contract is isolated from every other contract, and can only view it's own internal state. This eliminates common reentrancy issues and allows significant optimizations for scheduling and memory management, but introduces delay when calling other contracts. All contracts are thus inherently asynchronous.
* **What is the latency between each operation?**: The block rate is dynamic, and the current block producer will publish a new block when needed. The expected block rate is at least 1 block/10 seconds.
* **How does callbacks work?**: A callback is an interaction that expects a reply. For example if contract A sends a callback to contract B, it means that contract B must produce a return message, which will be sent back to A. There can be multiple levels of callbacks, similar to a call stack, which the blockchain will keep track of.
* **What is the value of** [**`Context::sender`**](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_common/context/struct.ContractContext.html)**?**: The `sender` field in `Context` contains the address of the caller. The caller will depend upon the current context, though the following guarantees are made:
  1. `#[init]`: Creator of the contract.
  2. `#[action]`: Sender of the transaction. Some contract address if from a contract, user address if from a user.
  3. `#[callback]`: The sender of the interaction that triggered the callback. So if user A sent an interaction to contract B, which then performed a callback to contract C. When the return message is received by contract B, the sender of the interaction will be set to A. Events are explained more in-depth [here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/programmers-guide-to-smart-contracts.html#events)
  4. `#[zk_on_secret_input]`: Sender of the input.
* **Action macros complains about `pbc_lib`**: Ensure that `abi` features are correctly imported. When depending upon other crates, you must never use `features = ["abi"]`, as this will result in malformed contracts. Abi features must only ever be enabled conditionally. Crates must propagate the `abi` feature by placing a `abi = ["crate1/abi", "crate2/abi", "crate3/abi", ...]` statement in the `[features]` toml section.

### Zero Knowledge Rust Contracts <a href="#zero-knowledge-rust-contracts" id="zero-knowledge-rust-contracts"></a>

Questions for ZK contracts:

* **ZkContract: What does `sealing` a ZK variable do?**: Sealing the variable will prevent the owner of the variable from accessing the contents. This is mainly relevant when the contract transfers variables between users.
* **ZkContract: What actions happens off-chain or on-chain?**: All actions in the public part of the contract happens on-chain, including those hooks that are exclusive to ZK contracts, such as `zk_on_compute_complete`, `zk_on_variables_opened`, `zk_on_attestation_complete`. The ZK computation itself happens off-chain on the ZK nodes. Secret shared inputs can either happen off-chain or on-chain, depending upon what the inputter prefers and is entirely transparent to contract developers.
* **ZkContract: What can I store in variable metadata?**: All state serializable data can be stored as variable metadata, but not all types are supported by ZkRust.
* **ZkContract: How large (in bits) can each secret-shared variable be?**: The goal is to support arbitrarily large secret-shared variables. but the largest currently supported bitsize is 127.
* **ZkContract: How does `ZkStateChange::start_computation` work?**: The `start_computation<MetadataT>(metadata: Vec<MetadataT>)` constructor initializes the contract's associated ZK computation. The `metadata` argument must contain the metadata intended for the resulting variables. Thus, if the ZK computation produces three variables, then the `start_computation` must have been provided with three pieces of metadata.
* **ZkContract: Can I access secret variable data outside of the ZK computation?**: Secret variable data access is very restricted, and is only viewable in these cases:
  1. By everyone, when opened by the contract using `ZkState::OpenVariables`. The data will be available as a byte Vec in `ZkClosed::data`. This is not possible for user inputs.
  2. By the owner, when the variable have not been sealed (see question above.)
  3. By the ZK program (but not ZK nodes!), during ZK computation.
* **ZkContract: When can I access secret variable data for user inputs?**: It is never possible for the contract to access the secret variable data of inputs.
* **ZkContract: Is it possible to perform validation of secret user input data in `#[zk_on_secret_input]`?**: No, it is not possible to access the secret variable data in `zk_on_secret_input`. Validation of inputs must be performed in a ZK computation. Generally it is most convenient to validate just before the main computation, and ignoring it if invalid.
* **ZkContract: When are secret variables removed?**: Secret variables can only be removed if the contract deliberately deletes them, using either `DeleteVariable` or `OutputCompute`.
* **ZkContract: When does my ZK computation run?**: The ZK computation runs asynchronously from shortly after returning `ZkStateChange::start_computation` to shortly before the invocation of `#[zk_on_compute_complete]`. Note that the contract is not blocked during ZK execution, and additional inputs may occur. These additional inputs will not be visible to an already started computation.
* **ZkRust: How do I load metadata and inputs?**: Metadata and secret inputs should be loaded using their respective generic functions `load_metadata::<PublicType>(variable_id)` and `load_sbi::<SecretType>(variable_id)`, where `PublicType` must be public, and `SecretType` must be secret.
* **ZkRust: What is the difference between public and secret types?**: Public types are "normal" Rust types like `i32`, where the data is publicly known, for example because it is directly visible in the contract state. Secret types are types like `Sbi32` where the data is secret-shared between ZK nodes. A struct (or tuple) is a public type if all it's fields are public, and a struct is a secret type if all it's fields are secret.
* **ZkRust: Why can I not load or return a Vec?**: The RustZK runtime is currently incapable of working with variable-length data such as strings and vectors. The best workaround at the moment is to restructure your computation to only return a single variable.
* **ZkRust: Ok, but what types&#x20;**_**can**_**&#x20;I return?**: All Secret types and tuples of secret types. Examples: `Sbi128`, `(Sbi32, Sbi32)`, `SecretPoint` (defined as `struct SecretPoint { x: Sbi32, y: Sbi32 }`.) To produce multiple variables, return them as a tuple.
* **ZkRust: Can I use feature X?** : [Check out the currently supported features.](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/zk-language-features.html)
* **ZkRust: Why is important feature X not supported?**: ZkRust is an entire reimplementation of the Rust compiler targeted at Partisia Blockchain, and not all features have been prioritized.
