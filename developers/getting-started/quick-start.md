# Quick Start

## Quick Start Guide

This guide will help you deploy your first smart contract on Partisia Blockchain in just a few minutes.

### Prerequisites

* Basic knowledge of Rust programming
* [Rust installed](https://www.rust-lang.org/tools/install) (version 1.60 or later)
* A Partisia Blockchain account with testnet gas
* A code editor

### Step 1: Set Up Your Development Environment

Install the Partisia Blockchain CLI:

```bash
cargo install cargo-partisia-contract
```

**Verify the installation:**

<pre class="language-rust" data-title="Bash"><code class="lang-rust"><strong>cargo pbc --version
</strong></code></pre>

Add the [WebAssembly](https://webassembly.org/) target:

{% code title="Bash" %}
```rust
rustup target add wasm32-unknown-unknown 
```
{% endcode %}

***

### Step 2: Create a Simple Contract

Clone our example repository to start with a ready example:

{% code title="Bash" %}
```git
git clone https://gitlab.com/partisiablockchain/language/example-contracts.git

```
{% endcode %}

Or run <mark style="color:blue;">`cargo init`</mark> and create a new contract in <mark style="color:blue;">`src/lib.rs`</mark><mark style="color:blue;">:</mark>

The counter contract is a simple example that stores a number and allows you to increment it.

{% code title="src/lib.rs" %}
```rust
use pbc_contract_common::context::ContractContext;
use pbc_contract_common::events::EventGroup;
use pbc_contract_codegen::{state, init, action};

#[state]
pub struct CounterState {
    counter: u64,
}

#[init]
pub fn initialize(_ctx: ContractContext, initial_value: u64) -> CounterState {
    CounterState {
        counter: initial_value,
    }
}

#[action]
pub fn increment(_ctx: ContractContext, state: CounterState) -> CounterState {
    CounterState {
        counter: state.counter + 1,
    }
}

#[action]
pub fn decrement(_ctx: ContractContext, state: CounterState) -> CounterState {
    CounterState {
        counter: state.counter - 1,
    }
}

```
{% endcode %}

{% code title="cargo.toml" %}
```toml
[package]
name = "partisia"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
pbc_contract_common = { git = "https://gitlab.com/partisiablockchain/language/contract-sdk.git" }
pbc_contract_codegen = { git = "https://gitlab.com/partisiablockchain/language/contract-sdk.git" }
pbc_traits = { git = "https://gitlab.com/partisiablockchain/language/contract-sdk.git" }
pbc_lib = { git = "https://gitlab.com/partisiablockchain/language/contract-sdk.git" }
read_write_rpc_derive = { git = "https://gitlab.com/partisiablockchain/language/contract-sdk.git" }
read_write_state_derive = { git = "https://gitlab.com/partisiablockchain/language/contract-sdk.git" }
create_type_spec_derive = { git = "https://gitlab.com/partisiablockchain/language/contract-sdk.git" }

[features]
abi = ["pbc_contract_common/abi", "pbc_contract_codegen/abi", "pbc_traits/abi", "create_type_spec_derive/abi", "pbc_lib/abi"]
default = ["abi"]

```
{% endcode %}

***

### Step 3: Compile the Contract

Compile the contract with:

{% code title="Bash" %}
```rust
cargo pbc build --release
```
{% endcode %}

This will generate the [WebAssembly ](https://webassembly.org/)binary and ABI files in `target/wasm32-unknown-unknown/release/`.

***

### Step 4: Get Testnet Gas

Before deploying your contract, you'll need gas on the testnet:

#### Creating a New Account with Gas

You can create a new account with gas by running:

{% code title="bash" %}
```rust
cargo pbc account create
```
{% endcode %}

This command creates a new account, prints the account address and private key, and fills it with gas.

#### Adding Gas to an Existing Account

To add gas to an existing account, run:

{% code title="Bash" %}
```rust
cargo pbc account mintgas <your-account-address>
```
{% endcode %}

#### Getting More Gas

For additional gas:

1. Sign in to the [testnet browser](https://browser.testnet.partisiablockchain.com/) (icon in upper right corner)
2. Go to the [Feed Me contract interaction](https://browser.testnet.partisiablockchain.com/contracts/02c14c29b2697f3c983ada0ee7fac83f8a937e2ecd/feed_me)
3. Enter your account address as the receiver
4. The transaction cost is set to 100k gas by default, which can be reduced to approximately 60000 gas
5. Execute the transaction - your account will gain approximately 1,000,000,000 gas (10,000 TEST\_COIN)

#### Checking Your Balance

To verify your gas balance:

* In the browser, go to your account tab after signing in
* Or check the dashboard under your account information

***

### Step 5: Deploy to Testnet

{% hint style="info" %}
Ensure you have a Partisia Wallet, learn more: [Key Concepts](key-concepts.md)
{% endhint %}

1. Visit the [Partisia Blockchain Testnet Browser](https://browser.testnet.partisiablockchain.com/contracts/deploy)
2. Connect your wallet (create one if needed)
3. Click "Deploy Contract"
4. Upload the generated .wasm and .abi files
5. Set an initial value for the counter (e.g., 0)
6.  Submit the deployment\


    <figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

***

### Step 6: Interact with Your Contract

Once deployed, you can interact with your contract:

1. Find your contract in the browser's "Contracts" section
2. Click on your contract address
3. Select the "Interact" tab
4. Click the "increment" action
5. Submit the transaction
6.  View the updated state to confirm the counter increased\
    \


    <figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### What's Next?

Congratulations! You've deployed your first smart contract on Partisia Blockchain. Next steps:

* Build your first Fullstack Partisia app
* [Learn about privacy-preserving contracts](../guides/zk-smart-contracts/)
* [Explore the full SDK capabilities](../dev-tools/partisia-sdk-java/)
