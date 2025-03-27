# Debugging Guide

Developing smart contracts on Partisia Blockchain requires a systematic approach to testing and debugging. This guide provides structured methods to identify, diagnose, and resolve common issues when working with both public and [Zero Knowledge (ZK) smart contracts.](../zk-smart-contracts/)

### Common Error Types

[Smart contract](./) errors on Partisia Blockchain generally fall into these categories:

1. **Compilation Errors**: Issues that prevent your contract from compiling into WASM or ZKWA
2. **Deployment Errors**: Problems that occur when deploying contracts to the blockchain
3. **Runtime Errors**: Failures that happen during contract execution
4. **ZK-Specific Errors**: Issues specific to Zero Knowledge computations

### Development Environment Setup for Debugging

#### Required Tools

Ensure you have these properly configured before starting:

{% code title="Bash" %}
```bash
# Check Rust version - should be latest stable
rustup show

# Check wasm32 target is installed
rustup target list --installed

# Verify PBC tool installation
cargo pbc --version
```
{% endcode %}

#### Test Environment Configuration

Create a comprehensive local testing setup:

1. **Local Tests**: Use the [testing framework](testing.md) for unit testing
2. **Testnet Deployment**: [Deploy to testnet](access-and-use-the-testnet.md) before mainnet for integration testing

### Debugging by Environment

#### Local Development Environment

When debugging locally:

{% code title="Bash" %}
```bash
# Build in debug mode for better error messages
cargo pbc build

# Run contract tests with detailed logs
cd java-test
mvn test -Dtest="YourContractTest"

# Enable code coverage to verify all paths
./run-java-tests.sh -c -b
```
{% endcode %}

#### Testnet Environment

For testnet debugging:

1. Use the [PBC Browser](https://browser.testnet.partisiablockchain.com/contracts) to monitor contract states
2. Examine transaction logs for any failures
3. Use [testnet REST API](https://reader.partisiablockchain.com/openapi) for programmatic debugging

### Error Decision Trees

#### Compilation Error Decision Tree

If encountering compilation errors:

1. **Macro-related errors?**
   * Verify proper use of `#[state]`, `#[action]`, `#[init]` macros
   * Check that all required attributes are provided
2. **Type-related errors?**
   * Confirm all types implement required traits (`ReadWriteState`, `ReadWriteRPC`)
   * Ensure proper serialization for complex data structures
3. **ZK-specific errors?**
   * Verify ZK Rust code follows [ZK language feature constraints](../zk-smart-contracts/)
   * Check secret variable types and operations

#### Deployment Error Decision Tree

For deployment failures:

1. **Gas issues?**
   * Ensure [sufficient gas](../gas/) for deployment
   * Check state serialization efficiency
2. **Binary format issues?**
   * Verify [WASM/ZKWA](compile-and-deploy-contracts.md) files are properly generated
   * Check [ABI compatibility](../../dev-tools/abi-parser-typescript/parsing-abis-and-reading-contract-data.md)
3. **Permission issues?**
   * Confirm account has proper permissions

### Runtime Error Decision Tree

When contracts fail during execution:

1. **State access issues?**
   * Check for proper state access patterns
   * Verify state consistency
2. **Input validation failures?**
   * Add explicit validation for all inputs
   * Check for edge cases in inputs
3. **Event/callback errors?**
   * Inspect event group formation
   * Verify callback handlers

## Common Error Patterns and Solutions

### 1. State Serialization Gas Costs

**Problem**: High gas costs for state serialization/deserialization.

**Solution**: Use `SortedVecMap<T>` and `SortedVecSet<T>` for efficiently serializable data structures. For larger collections, use `AvlTreeMap`.

```rust
// Inefficient
let expensive_map: HashMap<Address, u128> = HashMap::new();

// Efficient
let optimized_map: SortedVecMap<Address, u128> = SortedVecMap::new();

// For larger datasets
let avl_map: AvlTreeMap<Address, u128> = AvlTreeMap::new();
```

### 2. Memory Management in ZK Contracts

**Problem**: ZK contracts frequently encounter memory usage errors.

**Solution**: Be mindful of variable sizes and bit lengths in ZK computations.

```rust
// Typical bit length declaration for ZK inputs
const BITLENGTH_OF_SECRET_VARIABLES: [u32; 1] = [32];

// Define ZK input with proper metadata and bit length
let input_def = ZkInputDef {
    seal: false,
    metadata: metadata,
    expected_bit_lengths: BITLENGTH_OF_SECRET_VARIABLES.to_vec(),
};
```

### 3. Callback Handling Errors

**Problem**: Missing or improper callback handling.

**Solution**: Ensure callbacks properly check success status and extract data correctly.

```rust
#[callback(shortname = 13)]
pub fn my_callback(
    contract_context: ContractContext,
    callback_context: CallbackContext,
    state: ContractState,
) -> (ContractState, Vec<EventGroup>) {
    // Check success status
    assert!(callback_context.success, "External call failed");
    
    // Safely extract return data
    if callback_context.results.len() > 0 {
        let first_event = &callback_context.results[0];
        let return_data = first_event.get_return_data::<ExpectedType>();
        // Process return data
    }
    
    // Return updated state
    (state, vec![])
}
```

### Testing Best Practices

Comprehensive testing dramatically reduces debugging needs:

1. **Aim for 100% Code Coverage**: Test all possible contract paths
2. **Test State Transitions**: Verify state changes after each action
3. **Simulate Edge Cases**: Test boundary conditions and unusual inputs
4. **Test Contract Interactions**: Verify multi-contract interaction patterns

```java
@ContractTest
void testEdgeCases() {
    // Test with boundary values
    byte[] initRpc = YourContract.initialize(MAX_VALUE, MIN_PARTICIPANTS);
    contractAddress = blockchain.deployContract(owner, CONTRACT_BYTES, initRpc);
    
    // Verify state is as expected
    YourContract.State state = YourContract.State.deserialize(
        blockchain.getContractState(contractAddress)
    );
    
    Assertions.assertThat(state.parameter()).isEqualTo(MAX_VALUE);
}
```

### ZK-Specific Debugging

Zero Knowledge smart contracts require special debugging attention:

1. **ZK Computation Functions**: Test ZK computation functions independently
2. **Secret Variable Management**: Check proper loading and management of secret variables
3. **Information Flow**: Verify information flow control rules are followed (no leaking secrets)

```rust
// ZK unit tests (in ZK computation module)
test_eq!(my_zk_function(SecretVarId::new(1)), 0, [0i32]);
test_eq!(my_zk_function(SecretVarId::new(1)), 42, [42i32]);
```

### Specific Contract Pattern Debugging

Different contract patterns require specific debugging approaches:

#### Multi-Contract Interactions

Check callback success status and properly handle failures in contract-to-contract interactions:

```rust
// Event builder with callback
let mut event_group_builder = EventGroup::builder();
event_group_builder.call(target_address, ACTION_SHORTNAME)
                  .argument(some_argument)
                  .done();

// Add callback to handle response
event_group_builder.with_callback(MY_CALLBACK_SHORTNAME)
                  .with_cost(10000)
                  .done();
```

#### Access Control Patterns

Verify access control permissions are properly checked:

```rust
// Access control validation
assert_eq!(
    context.sender, state.owner,
    "Only the owner can perform this action"
);
```

### Reporting Issues

If you encounter persistent issues:

1. Join the [active community](../../../welcome-to-partisia/resources.md)
2. Provide minimal reproduction steps
3. Include contract code, error messages, and transaction IDs

### Additional Resources

* [Transaction Gas Prices](../gas/)
* [ZK Rust Language Features](../zk-smart-contracts/)
* [Smart Contract Binary Formats](../zk-smart-contracts/zk-rust-reference.md)
