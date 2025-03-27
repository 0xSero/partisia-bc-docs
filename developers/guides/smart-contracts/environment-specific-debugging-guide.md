# Environment-Specific Debugging Guide

This guide provides detailed debugging techniques and tools specific to each development environment when working with Partisia Blockchain smart contracts.

### Local Development Environment

#### Setting Up an Effective Debugging Environment

A well-configured local environment is essential for effective debugging:

```bash
# Essential components
rustup component add rustfmt
rustup component add clippy
rustup target add wasm32-unknown-unknown

# Install PBC toolkit
cargo install cargo-partisia-contract

# Verify installation
cargo pbc --version
```

#### IDE Integration

**VSCode Setup**

For VSCode users, configure these extensions and settings:

1. Install these extensions:
   * rust-analyzer
   * Better TOML
   * CodeLLDB (for Rust debugging)
2. Add these workspace settings (`.vscode/settings.json`):

```json
{
  "rust-analyzer.checkOnSave.command": "clippy",
  "rust-analyzer.cargo.loadOutDirsFromCheck": true,
  "rust-analyzer.procMacro.enable": true,
  "editor.formatOnSave": true
}
```

**JetBrains IntelliJ/Rust Setup**

For IntelliJ IDEA or CLion:

1. Install the Rust plugin
2. Configure External Tools for PBC commands:
   * Go to Settings → Tools → External Tools
   * Add a new tool:
     * Name: `Build PBC Contract`
     * Program: `cargo`
     * Arguments: `pbc build --release`
     * Working directory: `$ProjectFileDir$`

#### Log-Based Debugging

Since traditional debuggers don't work with WASM contracts, use structured logging:

```rust
// Add debug logging in contract code
fn complex_function(param: u32) -> u32 {
    println!("complex_function called with param: {}", param);
    
    let result = calculate_something(param);
    println!("intermediate result: {}", result);
    
    let final_result = apply_transformation(result);
    println!("final result: {}", final_result);
    
    final_result
}
```

For contract test debugging, analyze logs from test execution:

```bash
# Run tests with detailed logging
cd java-test
mvn test -Dtest="YourContractTest" | tee debug.log
```

#### Using Rust Analyzer for Static Analysis

Leverage Rust Analyzer to catch issues before running tests:

1. Hover over code to see type information
2. Use "Go to Definition" to understand macro expansions
3. Check inlay hints for type information
4. Look for warning squiggles that indicate potential issues

#### Java Test Debugging

For Java-based contract tests:

1. Add detailed logging:

```java
@ContractTest
void testContractAction() {
    System.out.println("Deploying contract...");
    // Deploy contract
    
    System.out.println("Sending action...");
    // Send action
    
    System.out.println("Verifying state...");
    ContractState state = ContractState.deserialize(
        blockchain.getContractState(contractAddress)
    );
    System.out.println("Current state: " + state);
}
```

2. Set breakpoints in your Java test code
3. Run tests in debug mode through your IDE
4. Inspect variables and state during test execution

#### Memory and State Inspection

To analyze contract state during testing:

```java
// Inspect contract state at various points
@ContractTest
void debugContractState() {
    // Deploy contract
    byte[] initRpc = Contract.initialize(params);
    Address contractAddress = blockchain.deployContract(
        owner, CONTRACT_BYTES, initRpc
    );
    
    // Execute action that triggers events
    byte[] actionRpc = Contract.actionWithEvents(target, amount);
    blockchain.sendAction(sender, contractAddress, actionRpc);
    
    // Get transaction logs and analyze events
    TransactionReceipt receipt = blockchain.getTransactionReceipt(
        blockchain.getLastTransactionId()
    );
    
    System.out.println("Events generated:");
    for (EventLog event : receipt.getEvents()) {
        System.out.println("  Target: " + event.getTarget());
        System.out.println("  Action: " + event.getAction());
        System.out.println("  Parameters: " + event.getParameters());
        System.out.println("  Callback: " + event.getCallback());
    }
    
    // Check for callback executions
    System.out.println("Callbacks executed:");
    for (CallbackLog callback : receipt.getCallbacks()) {
        System.out.println("  Callback: " + callback.getShortname());
        System.out.println("  Success: " + callback.isSuccess());
        System.out.println("  Return data: " + callback.getReturnData());
    }
}
```

#### Handling Testnet-Specific Issues

Common testnet-specific debugging scenarios:

1. **Transaction stuck pending**:
   * Check network status in the browser dashboard
   * Verify sufficient gas was provided
   * Check if nonce is correct for the account
   * Wait for network congestion to clear
2. **Contract state missing**:
   * Verify contract was successfully deployed
   * Check transaction logs for deployment failures
   * Confirm contract address is correct
   * Wait for state propagation across the network
3. **Gas estimation failures**:
   * Start with higher gas limits initially
   * Monitor actual gas usage in successful transactions
   * Adjust gas limits based on observed consumption
   * Check for infinite loops or inefficient code

### Mainnet Environment

#### Production Debugging Strategies

When debugging on mainnet, caution is paramount:

1. **Simulation first**:
   * Always test full workflows on testnet before mainnet
   * Create a mainnet simulation environment if possible
   * Use the same parameters and inputs expected in production
2. **Progressive deployment**:
   * Start with minimal functionality deployed to mainnet
   * Add features incrementally after verification
   * Monitor each deployment carefully
3. **Failsafe mechanisms**:
   * Implement pause functionality in critical contracts
   * Add admin controls for emergency situations
   * Consider upgrade patterns for contract improvements

#### Mainnet Monitoring

Implement proactive monitoring for mainnet contracts:

```javascript
// Node.js monitoring example
const axios = require('axios');
const contractAddress = 'your_contract_address';

// Monitor contract state changes
async function monitorContract() {
    try {
        const response = await axios.get(
            `https://reader.partisiablockchain.com/shards/Shard0/contract/${contractAddress}/state`
        );
        
        // Process and analyze state
        const currentState = response.data;
        
        // Compare with previous state
        if (hasStateChanged(previousState, currentState)) {
            console.log('State change detected!');
            notifyAdministrators(currentState);
        }
        
        // Store current state for next comparison
        previousState = currentState;
    } catch (error) {
        console.error('Monitoring error:', error);
        notifyAdministrators(error);
    }
}

// Check recent transactions
async function checkRecentTransactions() {
    try {
        const response = await axios.get(
            `https://reader.partisiablockchain.com/shards/Shard0/transactions/contract/${contractAddress}?limit=10`
        );
        
        // Analyze recent transactions
        const transactions = response.data;
        
        for (const tx of transactions) {
            // Check for failures
            if (tx.result && tx.result.statusCode !== 0) {
                console.error(`Transaction ${tx.hash} failed:`, tx.result);
                notifyAdministrators(tx);
            }
        }
    } catch (error) {
        console.error('Transaction check error:', error);
        notifyAdministrators(error);
    }
}

// Run monitoring at regular intervals
setInterval(monitorContract, 60000); // Every minute
setInterval(checkRecentTransactions, 300000); // Every 5 minutes
```

#### Production Investigation Techniques

When issues occur in production:

1. **Transaction forensics**:
   * Analyze transaction logs and receipts
   * Check parameter values and gas usage
   * Compare against successful transactions
   * Look for patterns across failed transactions
2. **State reconstruction**:
   * Reconstruct the state at the time of failure
   * Compare with expected state
   * Identify discrepancies and potential causes
   * Test hypothesis in an isolated environment
3. **User interaction review**:
   * Analyze user interactions leading to failure
   * Check for unexpected input sequences
   * Verify frontend validation is working correctly
   * Look for potential user experience improvements

### ZK Contract Debugging

#### ZK Computation Debugging

Zero Knowledge contracts require special debugging techniques:

1. **ZK computation unit testing**:
   * Add test assertions in zk\_compute functions
   * Use the `test_eq!` macro for validating outputs
   * Create test cases covering edge cases

```rust
// ZK computation with embedded tests
#[zk_compute(shortname = 0x61)]
pub fn my_zk_function(input_id: SecretVarId) -> Sbi32 {
    // Load input
    let input = load_sbi::<Sbi32>(input_id);
    
    // Computation logic
    let result = input * Sbi32::from(2);
    
    result
}

// Test cases using test_eq! macro
test_eq!(my_zk_function(SecretVarId::new(1)), 0, [0i32]);
test_eq!(my_zk_function(SecretVarId::new(1)), 42, [21i32]);
test_eq!(my_zk_function(SecretVarId::new(1)), -10, [-5i32]);
```

2. **ZK flow debugging**:
   * Implement detailed logging in ZK event handlers
   * Track variable inputs, computation triggers, and results
   * Verify expected information flow

```rust
// Add detailed logging in ZK event handlers
#[zk_on_compute_complete(shortname = 0x42)]
fn computation_complete(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
    output_variables: Vec<SecretVarId>
) -> (ContractState, Vec<EventGroup>, Vec<ZkStateChange>) {
    println!("ZK computation complete");
    println!("  Block time: {}", context.block_production_time);
    println!("  Output variables: {:?}", output_variables);
    println!("  Number of variables: {}", output_variables.len());
    
    // Continue with normal processing...
    (
        state,
        vec![],
        vec![ZkStateChange::OpenVariables {
            variables: output_variables,
        }],
    )
}
```

3. **ZK information flow analysis**:
   * Verify secret inputs remain secret
   * Check for inadvertent information leakage
   * Validate that only intended results are opened

#### ZK Variable Management

Debug ZK variable handling effectively:

```rust
// Function to debug ZK variable state
fn debug_zk_variables(zk_state: &ZkState<SecretVarMetadata>, label: &str) {
    println!("=== ZK Variables ({}) ===", label);
    println!("Calculation state: {:?}", zk_state.calculation_state);
    println!("Secret variables count: {}", zk_state.secret_variables.len());
    println!("Pending inputs count: {}", zk_state.pending_inputs.len());
    println!("Attestations count: {}", zk_state.data_attestations.len());
    
    println!("\nVariable details:");
    for (var_id, var) in &zk_state.secret_variables {
        println!("  ID: {}, Owner: {}", var_id.raw_id, var.owner);
        println!("  Metadata: {:?}", var.metadata);
        println!("  Has data: {}", var.data.is_some());
        if let Some(data) = &var.data {
            println!("  Data length: {}", data.len());
        }
    }
}

// Use in ZK handlers
#[zk_on_variable_inputted(shortname = 0x41)]
fn variable_inputted(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
    variable_id: SecretVarId
) -> ContractState {
    debug_zk_variables(&zk_state, "After variable input");
    // Continue normal processing...
    state
}
```

### Wallet Integration Debugging

#### Wallet Connection Issues

Debug wallet connection problems:

```javascript
// Verbose wallet connection
async function connectWalletWithDebug() {
    console.log("Connecting to wallet...");
    
    // Check environment
    console.log("Environment check:");
    console.log("- Window object:", typeof window !== 'undefined');
    console.log("- Wallet extension detected:", 
                typeof window.__onPartisiaConfirmWin === 'function');
    
    try {
        const sdk = new PartisiaSdk();
        console.log("SDK created successfully");
        
        // Add event listeners for debugging
        window.addEventListener('message', (event) => {
            if (event.data && event.data.type === 'partisia_wallet_event') {
                console.log("Wallet event received:", event.data);
            }
        });
        
        // Attempt connection with timeout
        const connectionPromise = sdk.connect({
            permissions: ['sign'],
            dappName: 'Debug App',
            chainId: 'Partisia Blockchain'
        });
        
        // Add timeout for debugging
        const timeoutPromise = new Promise((_, reject) => {
            setTimeout(() => reject(new Error("Connection timeout")), 30000);
        });
        
        const result = await Promise.race([connectionPromise, timeoutPromise]);
        console.log("Connection successful:", result);
        console.log("Connected account:", sdk.connection.account.address);
        
        return sdk;
    } catch (error) {
        console.error("Connection error details:", error);
        console.error("Error name:", error.name);
        console.error("Error message:", error.message);
        console.error("Error stack:", error.stack);
        throw error;
    }
}
```

#### Transaction Signing Debugging

Debug transaction signing issues:

```javascript
// Debug transaction signing
async function signTransactionWithDebug(sdk, contract, payload) {
    console.log("Signing transaction...");
    console.log("Contract:", contract);
    console.log("Payload:", payload);
    
    try {
        // Verify SDK is connected
        if (!sdk.isConnected) {
            console.error("SDK not connected");
            throw new Error("SDK not connected");
        }
        
        console.log("Connection details:", {
            address: sdk.connection.account.address,
            publicKey: sdk.connection.account.pub,
        });
        
        // Prepare signing request
        console.log("Preparing signing request...");
        
        // Sign with timeout for debugging
        const signingPromise = sdk.signMessage({
            contract,
            payload,
            payloadType: 'hex_payload'
        });
        
        const timeoutPromise = new Promise((_, reject) => {
            setTimeout(() => reject(new Error("Signing timeout")), 60000);
        });
        
        const result = await Promise.race([signingPromise, timeoutPromise]);
        
        console.log("Signing successful");
        console.log("Transaction hash:", result.trxHash);
        console.log("Signature:", result.signature);
        
        return result;
    } catch (error) {
        console.error("Signing error details:", error);
        console.error("Error name:", error.name);
        console.error("Error message:", error.message);
        console.error("Error stack:", error.stack);
        throw error;
    }
}
```

### Multi-Environment Testing

#### Cross-Environment Testing Strategies

For comprehensive debugging across environments:

1. **Consistent test scenarios**:
   * Develop standardized test cases
   * Run identical scenarios in each environment
   * Compare results and behavior
2. **Environment progression**:
   * Start with local environment testing
   * Progress to testnet for network testing
   * Finally deploy to mainnet after validation
3. **Regression test automation**:
   * Automate test scenarios for consistent execution
   * Run regression tests after each change
   * Compare results across environments

Example test automation for cross-environment testing:

```javascript
// Cross-environment test runner
async function runCrossEnvironmentTests(config) {
    const environments = [
        { name: 'Local', baseUrl: 'http://localhost:8545' },
        { name: 'Testnet', baseUrl: 'https://reader.testnet.partisiablockchain.com' },
        { name: 'Mainnet', baseUrl: 'https://reader.partisiablockchain.com' }
    ];
    
    const testCases = [
        { name: 'Basic Transfer', func: testBasicTransfer },
        { name: 'Multi-Step Flow', func: testMultiStepFlow },
        { name: 'Error Handling', func: testErrorHandling }
    ];
    
    for (const env of environments) {
        if (!config.environments.includes(env.name)) continue;
        
        console.log(`\n=== Running tests on ${env.name} ===\n`);
        
        // Setup environment-specific clients
        const client = createClient(env.baseUrl);
        
        // Run each test case
        for (const test of testCases) {
            if (!config.tests.includes(test.name)) continue;
            
            console.log(`\n--- Test: ${test.name} ---\n`);
            
            try {
                await test.func(client, env);
                console.log(`✅ ${test.name} passed on ${env.name}`);
            } catch (error) {
                console.error(`❌ ${test.name} failed on ${env.name}:`, error);
            }
        }
    }
}
```

#### Environment-Specific Considerations

Adapt debugging strategies based on environment constraints:

1. **Local environment**:
   * Focus on unit/component testing
   * Use detailed logging and state inspection
   * Test edge cases extensively
2. **Testnet environment**:
   * Focus on integration and system testing
   * Monitor network interactions and gas usage
   * Test user flows and contract interactions
3. **Mainnet environment**:
   * Focus on monitoring and rapid response
   * Implement failsafe mechanisms
   * Prepare rollback strategies

### Debugging Tools Reference

#### Local Development Tools

| Tool                 | Purpose                   | Installation                            |
| -------------------- | ------------------------- | --------------------------------------- |
| Rust Analyzer        | Code analysis and linting | IDE extension                           |
| Clippy               | Advanced linting          | `rustup component add clippy`           |
| Cargo PBC            | Contract build tools      | `cargo install cargo-partisia-contract` |
| Contract Test Runner | Contract testing          | Part of example projects                |

#### Network Debugging Tools

#### Wallet Development Tools

| Tool            | Purpose                       | Installation                                                                                                   |
| --------------- | ----------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Partisia SDK    | Wallet integration library    | `npm install partisia-sdk`                                                                                     |
| Partisia Wallet | Browser extension for testing | [Chrome Extension](https://chrome.google.com/webstore/detail/partisia-wallet/gjkdbeaiifkpoencioahhcilildpjhgh) |
| partisiaCrypto  | Cryptographic utilities       | `npm install partisia-blockchain-applications-crypto`                                                          |

#### Browser Tools for Contract Inspection

The PBC Browser provides several tools for debugging:

1. **Contract state inspection**:
   * Navigate to your contract in the browser
   * Use the "State" tab to view serialized state
   * Examine transaction history
2. **Interaction debugging**:
   * Use the "Interact" tab to call contract actions
   * Test different parameters and observe results
   * Check for errors in transaction logs

Example workflow:

1. Deploy contract
2. Find your contract in the browser
3. Try various interactions through the UI
4. Examine state changes after each action
5. Check transaction logs for any errors

#### REST API Debugging

For programmatic debugging:

```javascript
// Using fetch to query contract state
async function getContractState(contractAddress) {
    const response = await fetch(
        `https://reader.testnet.partisiablockchain.com/shards/Shard0/contract/${contractAddress}/state`
    );
    
    if (!response.ok) {
        console.error(`Error: ${response.status}`);
        return null;
    }
    
    const data = await response.json();
    console.log("Raw contract state:", data);
    return data;
}

// Check transaction status
async function checkTransaction(txHash) {
    const response = await fetch(
        `https://reader.testnet.partisiablockchain.com/shards/Shard0/transaction/${txHash}`
    );
    
    if (!response.ok) {
        console.error(`Error: ${response.status}`);
        return null;
    }
    
    const data = await response.json();
    console.log("Transaction details:", data);
    
    // Check for errors
    if (data.result && data.result.statusCode !== 0) {
        console.error(
            `Transaction failed with code ${data.result.statusCode}:`,
            data.result.message
        );
    }
    
    return data;
}
```

#### Event Logging and Tracing

To trace contract events and callbacks:

1. Deploy contract with specific test logic
2. Execute actions that trigger events
3. Check transaction logs for event execution
4. Verify callback execution and results
