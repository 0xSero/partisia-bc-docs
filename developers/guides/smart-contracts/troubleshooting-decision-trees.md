# Troubleshooting Decision Trees

When developing smart contracts on Partisia Blockchain, you may encounter various issues during development, testing, and deployment. This guide provides systematic decision trees to help you diagnose and resolve common problems.

### Compilation Issues

#### Unable to Compile Contract

```
Is the error related to Rust?
├── Yes → Are you using unsupported Rust features?
│   ├── Yes → Review supported Rust features for contract development
│   └── No → Check Rust compiler version and update if needed
└── No → Is it a PBC toolchain error?
    ├── Yes → Is cargo-pbc installed correctly?
    │   ├── Yes → Check cargo-pbc version compatibility
    │   └── No → Reinstall cargo-pbc: cargo install cargo-partisia-contract
    └── No → Is the error about missing dependencies?
        ├── Yes → Update Cargo.toml with required dependencies
        └── No → Check project structure for correct organization
```

#### ZK Contract Compilation Failures

```
Is the error in ZK Rust code?
├── Yes → Does the code use unsupported ZK features?
│   ├── Yes → Review ZK Rust Language Features documentation
│   └── No → Is the error about invalid secret sharing operations?
│       ├── Yes → Ensure information flow control rules are respected
│       └── No → Check zk_compute function annotations and signatures
└── No → Is the error in public contract code?
    ├── Yes → Follow standard contract compilation troubleshooting
    └── No → Is it related to ZK compiler installation?
        ├── Yes → Ensure Java 17+ is installed
        └── No → Check contract manifest for correct ZK paths
```

#### Contract ABI Generation Failures

```
Is the error about invalid ABI?
├── Yes → Are all state/action/init annotations correct?
│   ├── Yes → Is the error about unsupported types?
│   │   ├── Yes → Replace with supported types
│   │   └── No → Check for proper trait implementations
│   └── No → Fix annotations according to documentation
└── No → Is the error in custom type definitions?
    ├── Yes → Do all custom types implement required traits?
    │   ├── Yes → Is CreateTypeSpec implemented incorrectly?
    │   └── No → Add #[derive(ReadWriteState, ReadWriteRPC)]
    └── No → Check for proper trait implementations
```

### Deployment Issues

#### Contract Deployment Failures

```
Is the error about transaction failure?
├── Yes → Do you have sufficient gas?
│   ├── Yes → Is your contract size too large?
│   │   ├── Yes → Optimize contract size
│   │   └── No → Check transaction parameters
│   └── No → Add more gas to deployment transaction
└── No → Is the error during bytecode validation?
    ├── Yes → Is WASM/ZKWA file correctly generated?
    │   ├── Yes → Check for unsupported contract features
    │   └── No → Rebuild with cargo pbc build --release
    └── No → Is it an ABI compatibility issue?
        ├── Yes → Check ABI version compatibility
        └── No → Verify deployment address and permissions
```

#### ZK Contract Deployment Specific Issues

```
Is the deployment failing with ZK-specific errors?
├── Yes → Is the ZKWA file correctly generated?
│   ├── Yes → Is there sufficient stake for ZK computation?
│   │   ├── Yes → Check ZK compiler version compatibility
│   │   └── No → Increase stake for ZK computation
│   └── No → Rebuild with proper ZK configuration
└── No → Is it a standard deployment issue?
    ├── Yes → Follow standard deployment troubleshooting
    └── No → Check ZK node availability on target network
```

### Runtime Issues

#### Contract Transaction Failures

```
Is the transaction reaching the contract?
├── Yes → Is the error an assertion failure?
│   ├── Yes → Check input parameters against validation rules
│   └── No → Is it an out-of-gas error?
│       ├── Yes → Optimize gas usage or increase gas limit
│       └── No → Check state access patterns
└── No → Is the transaction properly signed?
    ├── Yes → Is the contract address correct?
    │   ├── Yes → Check for correct action shortname
    │   └── No → Update with correct contract address
    └── No → Fix transaction signature
```

#### Contract State Issues

```
Is the contract state incorrect after transaction?
├── Yes → Did the transaction complete successfully?
│   ├── Yes → Check action logic for state update bugs
│   └── No → Fix transaction failure first
└── No → Is the state reading/deserialization correct?
    ├── Yes → Is the contract address or action correct?
    │   ├── Yes → Check transaction sequence
    │   └── No → Update with correct references
    └── No → Fix state deserialization code
```

#### Contract Event/Callback Issues

```
Are events not being processed?
├── Yes → Are EventGroups correctly built?
│   ├── Yes → Is the target contract address correct?
│   │   ├── Yes → Is the action shortname correct?
│   │   └── No → Update with correct action shortname
│   └── No → Fix EventGroup builder usage
└── No → Are callbacks not firing?
    ├── Yes → Is the callback shortname registered?
    │   ├── Yes → Is sufficient gas provided for callback?
    │   └── No → Register callback with correct shortname
    └── No → Check callback context handling
```

### ZK Computation Issues

#### Secret Input Problems

```
Are secret inputs not being accepted?
├── Yes → Is input properly formatted?
│   ├── Yes → Is input within expected bit length?
│   │   ├── Yes → Check ZK input handler for validation errors
│   │   └── No → Adjust input to match expected bit length
│   └── No → Fix input format according to contract requirements
└── No → Are inputs being rejected?
    ├── Yes → Is sender authorized to submit input?
    │   ├── Yes → Check for duplicate input restrictions
    │   └── No → Ensure sender has permission to submit
    └── No → Check zk_on_variable_inputted handler
```

#### ZK Computation Failures

```
Is the ZK computation failing to complete?
├── Yes → Is the computation triggered correctly?
│   ├── Yes → Are there sufficient secret inputs?
│   │   ├── Yes → Check ZK computation code for errors
│   │   └── No → Ensure minimum required inputs are present
│   └── No → Fix computation trigger action
└── No → Are computation results not as expected?
    ├── Yes → Is the computation logic correct?
    │   ├── Yes → Check variable loading and information flow
    │   └── No → Fix ZK computation algorithm
    └── No → Check variable opening and result handling
```

#### Variable Opening Issues

```
Are variables not being opened?
├── Yes → Is the variable specified correctly?
│   ├── Yes → Is the correct ZkStateChange used?
│   │   ├── Yes → Check computation completion status
│   │   └── No → Use ZkStateChange::OpenVariables correctly
│   └── No → Update with correct SecretVarId
└── No → Are opened variables not readable?
    ├── Yes → Is the variable type correct?
    │   ├── Yes → Check data deserialization
    │   └── No → Use correct type for variable reading
    └── No → Check variable data existence
```

### Testing Issues

#### Contract Test Failures

```
Are JUnit contract tests failing?
├── Yes → Is the contract deployment successful?
│   ├── Yes → Are action parameters correct?
│   │   ├── Yes → Check assertion or validation logic
│   │   └── No → Fix RPC parameter serialization
│   └── No → Fix contract deployment parameters
└── No → Is the test setup correct?
    ├── Yes → Is test sequence correct?
    │   ├── Yes → Check expected state against actual
    │   └── No → Update test sequence for correct flow
    └── No → Fix test setup configuration
```

#### Code Coverage Issues

```
Is code coverage incomplete?
├── Yes → Are all contract paths tested?
│   ├── Yes → Are edge cases covered?
│   │   ├── Yes → Check if tests execute correctly
│   │   └── No → Add tests for edge cases
│   └── No → Add tests for untested code paths
└── No → Is coverage reporting configured correctly?
    ├── Yes → Are instrumentation failures occurring?
    │   ├── Yes → Check contract runner generation
    │   └── No → Run with -c -b flags for coverage
    └── No → Configure coverage reporting properly
```

### Gas Optimization Issues

#### Excessive Gas Usage

```
Is contract using too much gas?
├── Yes → Is state serialization expensive?
│   ├── Yes → Use optimized data structures (SortedVecMap/AvlTreeMap)
│   └── No → Are there too many state operations?
│       ├── Yes → Batch state updates
│       └── No → Check for unnecessary computations
└── No → Is deployment gas cost high?
    ├── Yes → Is contract too large?
    │   ├── Yes → Simplify or modularize contract
    │   └── No → Check initialization parameters
    └── No → Profile specific actions for optimization
```

### Testnet to Mainnet Transition Issues

#### Mainnet Deployment Failures

```
Did contract work on testnet but fail on mainnet?
├── Yes → Are there network-specific parameters?
│   ├── Yes → Update parameters for mainnet
│   └── No → Is gas estimation different?
│       ├── Yes → Adjust gas limits for mainnet
│       └── No → Check for mainnet deployment restrictions
└── No → Is it a new issue not seen on testnet?
    ├── Yes → Did you test thoroughly on testnet?
    │   ├── Yes
```
