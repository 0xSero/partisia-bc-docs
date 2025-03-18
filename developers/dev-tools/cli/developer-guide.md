# Developer Guide

## CLI Developer Guide

This guide provides practical workflows and examples for using the Partisia Blockchain CLI in your development process. Follow these step-by-step instructions to efficiently build, deploy, and interact with smart contracts on the Partisia Blockchain.

### Getting Started

#### Installation

Before you begin, make sure you have installed the Partisia Blockchain CLI:

```bash
cargo install cargo-partisia-contract
```

#### Setting Up Your Environment

Configure your CLI to target the desired network:

```bash
# For testnet (default)
cargo pbc config net testnet

# For local development
cargo pbc config net local "http://localhost:9432,http://localhost:8300" --default
```

For convenience, you can set a default private key to avoid specifying it in every command:

```bash
cargo pbc config privatekey ./mykey.pk
```

### Smart Contract Development Workflow

This section outlines the typical development workflow for Partisia Blockchain smart contracts.

#### 1. Create a New Contract Project

```bash
# Create a new contract project
cargo pbc new my_token_contract
cd my_token_contract
```

This command scaffolds a new Rust project with the proper structure and dependencies for a Partisia smart contract.

#### 2. Implement Your Contract

Edit the generated Rust files to implement your contract logic. For example, a simple token contract might look like:

```rust
#[state]
pub struct TokenState {
    pub total_supply: u128,
    pub name: String,
    pub symbol: String,
    pub balances: Map<Address, u128>,
}

#[init]
pub fn initialize(ctx: ContractContext, name: String, symbol: String, initial_supply: u128) -> TokenState {
    TokenState {
        name,
        symbol,
        total_supply: initial_supply,
        balances: Map::from([(ctx.sender, initial_supply)]),
    }
}

#[action]
pub fn transfer(ctx: ContractContext, state: TokenState, to: Address, amount: u128) -> TokenState {
    let mut balances = state.balances;
    
    let sender_balance = balances.get(&ctx.sender).unwrap_or(&0);
    assert!(sender_balance >= &amount, "Insufficient balance");
    
    let recipient_balance = balances.get(&to).unwrap_or(&0);
    
    balances.insert(ctx.sender, sender_balance - amount);
    balances.insert(to, recipient_balance + amount);
    
    TokenState {
        balances,
        ..state
    }
}
```

#### 3. Build Your Contract

```bash
# Initialize the project dependencies
cargo pbc init

# Build the contract in release mode
cargo pbc build --release
```

This generates:

* A WebAssembly (<mark style="color:blue;">`.wasm`</mark>) file
* An ABI (<mark style="color:purple;">`.abi`</mark>) file
* For ZK contracts, also a ZK WebAssembly (<mark style="color:red;">`.zkwasm`</mark>) file

The compiled files will be located in `target/wasm32-unknown-unknown/release/`.

#### 4. Create a Blockchain Account

If you don't already have an account, create one:

```bash
cargo pbc account create --file ./mykey.pk
```

This generates a private key and prints the corresponding address. On testnet, you can get test gas with:

```bash
cargo pbc account mintgas <YOUR_ADDRESS>
```

#### 5. Deploy Your Contract

Deploy your compiled contract to the blockchain:

```bash
cargo pbc transaction deploy --pk=./mykey.pk \
  target/wasm32-unknown-unknown/release/my_token_contract.wasm \
  "My Token" "MTK" 1000000
```

Upon successful deployment, you'll receive a contract address that you'll use for subsequent interactions.

#### 6. Interact with Your Contract

Call functions on your deployed contract:

```bash
# Transfer tokens to another address
cargo pbc transaction action --pk=./mykey.pk \
  <CONTRACT_ADDRESS> transfer <RECIPIENT_ADDRESS> 100
```

#### 7. Check Contract State

View the current state of your contract:

```bash
cargo pbc contract show <CONTRACT_ADDRESS> --state
```

This returns the contract's state in JSON format, allowing you to verify that your actions had the expected effect.

### Working with ZK Contracts

Zero-Knowledge contracts require additional steps for handling private data.

#### 1. Creating a ZK Contract

A ZK contract includes both public and private computation components. The project structure includes:

* `src/lib.rs` - The public contract interface
* `src/zk_compute.rs` - The private computation logic

#### 2. Building a ZK Contract

Build the contract with both components:

```bash
cargo pbc build --release
```

#### 3. Working with Secret Values

To submit a secret value to a ZK contract:

```bash
# First get information about the required input
cargo pbc abi show my_contract.abi | grep zk_on_secret_input

# Then submit a secret value (in this example, a salary value)
cargo pbc transaction action --pk=./mykey.pk \
  <CONTRACT_ADDRESS> submit_salary 50000
```

#### 4. Retrieving Secret Values

To view a secret value that you own:

```bash
# List secrets owned by your account
cargo pbc contract --pk=./mykey.pk secret list <CONTRACT_ADDRESS>

# View a specific secret (ID 1) as a 32-bit integer
cargo pbc contract --pk=./mykey.pk secret show <CONTRACT_ADDRESS> 1 i32
```

### Client Code Generation

Generate client code to interact with your contracts from applications.

#### TypeScript Client Generation

```bash
cargo pbc abi codegen --ts \
  target/wasm32-unknown-unknown/release/my_token_contract.abi \
  webapp/src/generated/TokenContract.ts
```

This generates TypeScript code that you can use in your web application:

```typescript
import { TokenContract } from './generated/TokenContract';
import { BlockchainAddress } from '@partisiablockchain/abi-client';

// Read contract state
const state = await client.getContractState(contractAddress);
const deserializedState = TokenContract.deserializeState(state);
console.log(`Total supply: ${deserializedState.total_supply}`);

// Create an action RPC
const rpc = TokenContract.transfer(
  BlockchainAddress.fromString(recipientAddress),
  BigInt(100)
);
```

#### Java Client Generation

```bash
cargo pbc abi codegen --java \
  target/wasm32-unknown-unknown/release/my_token_contract.abi \
  app/src/main/java/com/example/token/
```

This generates Java classes for interacting with your contract from Android or server applications.

### Advanced Workflows

#### Contract Upgrade

Upgrade an existing contract while preserving its state:

```bash
cargo pbc transaction upgrade --pk=./mykey.pk \
  <CONTRACT_ADDRESS> \
  target/wasm32-unknown-unknown/release/my_token_contract_v2.wasm
```

#### Gas Management

Specify gas limits for complex transactions:

```bash
cargo pbc transaction action --pk=./mykey.pk --gas=200000 \
  <CONTRACT_ADDRESS> complex_action param1 param2
```

Add more gas to a contract:

```bash
cargo pbc contract refuelgas --pk=./mykey.pk <CONTRACT_ADDRESS> 100000
```

#### Transaction Analysis

Examine a transaction's details and events:

```bash
cargo pbc transaction show <TRANSACTION_HASH>
```

#### Blockchain Monitoring

View the latest blocks and transactions:

```bash
cargo pbc block latest
cargo pbc transaction latest
```

### Common Patterns and Best Practices

#### Development Loop

A typical development loop involves:

1. Edit contract code
2. Build contract
3. Deploy to testnet
4. Interact with contract
5. Check state and debug
6. Repeat from step 1

#### Testing

For thorough testing before mainnet deployment:

1. Write unit tests in your Rust contract
2. Deploy to testnet and perform integration tests
3. Generate client code and build a test frontend
4. Test all contract functions through your frontend

#### Versioning

Track your contract versions:

```bash
# Check the SDK version used by your contract
cargo pbc print-version target/wasm32-unknown-unknown/release/my_contract.wasm

# Update to a newer SDK version
cargo pbc set-sdk "tag: 9.1.2"
```

### Troubleshooting

#### Common Issues

**Insufficient Gas**

```typescript
Error: Transaction failed: not enough gas
```

Solution: Increase gas with `--gas=<AMOUNT>` or add funds to your account.

**Build Errors**

```typescript
Error: failed to run custom build command for `my_contract`
```

Solution: Run `cargo pbc init` to ensure dependencies are properly set up.

**Network Connection Issues**

```typescript
Error: Failed to connect to network
```

Solution: Check your network configuration with `cargo pbc config list` and ensure endpoints are correct.

#### Debugging Tips

1. Use `--verbose` for more detailed output
2. Check contract state after operations
3. Examine transaction details with `transaction show`
4. For ZK contracts, list secrets to verify they're properly stored

### Next Steps

Now that you're familiar with the CLI workflows, explore these advanced topics:

* Creating custom client libraries with the generated code
* Implementing more complex ZK contract patterns
* Building UI components that interact with your contracts
* Setting up CI/CD pipelines for contract deployment

For comprehensive reference information, see the[ CLI Commands Reference.](commands-reference.md)
