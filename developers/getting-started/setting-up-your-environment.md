# Setting up your Environment

## Setting Up Your Development Environment

This guide will walk you through setting up a complete development environment for building applications on Partisia Blockchain. We'll cover installing all necessary tools, configuring your environment, and understanding the core concepts before you start building.

### Prerequisites

Before diving into Partisia Blockchain development, you should have:

* Basic familiarity with [programming concepts](https://www.coderscampus.com/basic-programming-concepts/)
* Command-line interface experience
* Understanding of [basic blockchain concepts](https://coingeek.com/blockchain101/blockchain-basics-key-things-to-know-as-a-beginner/) (helpful but not required)

### Installing Rust

Rust is the primary programming language for developing smart contracts on Partisia Blockchain. Follow these steps to install Rust:

1.  **Install Rustup** (Rust installer and version manager):

    Visit [https://rustup.rs/](https://rustup.rs/) and follow the instructions, or run:

    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```

    On Windows, download and run [rustup-init.exe](https://win.rustup.rs/x86_64) from the official site.
2.  **Configure Rust to use the stable version**:

    ```bash
    rustup install stable
    rustup default stable
    ```
3.  **Add the WebAssembly target** (required for smart contract compilation):

    ```bash
    rustup target add wasm32-unknown-unknown
    ```
4.  **Verify installation**:

    ```bash
    rustc --version
    cargo --version
    ```

### Installing the Partisia Blockchain CLI

The `cargo-partisia-contract` CLI tool (often used as `cargo pbc`) is essential for compiling and deploying Partisia smart contracts:

```bash
cargo install cargo-partisia-contract
```

Verify installation:

```bash
cargo pbc --version
```

### Required Dependencies

Depending on your operating system, you may need additional dependencies:

#### Linux

```bash
# Ubuntu/Debian
sudo apt-get install build-essential pkg-config libssl-dev

# Fedora
sudo dnf install gcc gcc-c++ openssl-devel
```

#### macOS

```bash
# Using Homebrew
brew install openssl
```

#### Windows

* Install [Visual Studio with C++ build tools](https://visualstudio.microsoft.com/downloads/)
* Preferably set up [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) for an easier setup time&#x20;
* In Visual Studio Installer, select "Desktop development with C++"
* Make sure these components are included:
  * MSVC x64/x86 build tools
  * Windows SDK
  * C++ CMake tools

### Setting Up Your Code Editor

While you can use any code editor, we recommend the following options for the best development experience:

### Setting Up Your Code Editor

**Visual Studio Code (Recommended)**

1. Download and install [Visual Studio Code](https://code.visualstudio.com/)
2. Install these recommended extensions:
   * **rust-analyzer**: Provides intelligent code completion and analysis
   * **WebAssembly**: Support for WebAssembly development
   * **Better TOML**: For working with Cargo.toml files
   * **GitLens**: Enhanced Git capabilities (optional)



**Partisia DApp Playground (For Quick Start)**

For developers who want to skip local environment setup, we offer the [Partisia DApp Playground](https://github.com/partisiablockchain/dapp-playground) - a pre-configured development environment with all necessary tools and dependencies already installed. This option allows you to:\


* Start coding immediately without installation steps
* Experiment with Partisia Blockchain features in a sandbox environment
* Access example templates and boilerplate code
* Test your applications in a controlled environment

To use the DApp Playground:

```bash
# Clone the repository
git clone https://github.com/partisiablockchain/dapp-playground.git
cd dapp-playground

# Follow the setup instructions in the README
```

#### Other Options

* **IntelliJ IDEA / CLion**: With the Rust plugin
* **Vim/NeoVim**: With rust.vim and additional plugins
* **Sublime Text**: With Rust Enhanced package

***

### Introduction to Key Concepts

Before starting development, it's helpful to understand these fundamental concepts:

#### Blockchain Basics

Blockchain technology operates as a distributed ledger system where transactions are grouped into blocks and linked cryptographically. Key aspects include:

* **Decentralization**: No single entity controls the entire network
* **Immutability**: Once recorded, data cannot be altered
* **Consensus**: Agreement on the state of the ledger
* **Smart Contracts**: Self-executing code that runs on the blockchain

For a deeper understanding, see our Key Concepts guide.

#### Zero-Knowledge Computation (ZK)

Partisia Blockchain's unique feature is its native support for Zero-Knowledge computation through Multi-Party Computation (MPC):

* **Privacy Preservation**: Compute on encrypted data without revealing the inputs
* **Confidential Transactions**: Keep sensitive information private while still enabling verification
* **Secret Sharing**: Data is split into "shares" that individually reveal nothing
* **Verifiable Results**: Cryptographic proofs verify computation correctness

To learn more about ZK concepts, see our ZK Smart Contracts Introduction.

#### Partisia Blockchain Architecture

Partisia Blockchain uses a unique architecture:

* **Sharding**: Multiple shards process transactions in parallel
* **FastTrack Consensus**: Optimistic execution model for high throughput
* **BYOC (Bring Your Own Coin)**: Use external tokens for transaction fees
* **MPC Token**: Native token used for staking and governance

For more details on the architecture, see our Key Concepts page.

### Testing Your Installation

Let's verify that everything is working by creating and compiling a simple contract:

1.  **Clone the example contracts repository**:

    ```bash
    git clone https://gitlab.com/partisiablockchain/language/example-contracts.git
    cd example-contracts
    ```
2.  **Build the counter example**:

    ```bash
    cd rust/counter
    cargo pbc build --release
    ```
3.  **Verify output**:

    You should see output indicating a successful build, and find these files in `target/wasm32-unknown-unknown/release/`:

    * `counter.wasm` (WebAssembly binary)
    * `counter.abi` (Contract ABI)

If these steps complete successfully, your development environment is properly set up!

### Getting Testnet Gas

Before deploying contracts, you'll need gas on the testnet:

```bash
# Create a new account with gas
cargo pbc account create

# Or add gas to an existing account
cargo pbc account mintgas <YOUR_ACCOUNT_ADDRESS>
```

For more details, see How to Get Testnet Gas.

### Next Steps

Now that your environment is set up, you can:

1. **Follow the Quick Start Guide** to [deploy your first contract](../guides/smart-contracts/)
2. **Build a Web Application** that interacts with contracts
3. **Explore ZK Smart Contracts** for privacy-preserving applications
4. **Learn about Gas and Optimization** for efficient contracts

### Troubleshooting

#### Common Issues

**Rust Installation Problems**

If you encounter issues with Rust installation:

* Ensure your internet connection is stable
* Check if you have sufficient permissions
* On Windows, ensure you have the right Visual C++ components

**WebAssembly Target Issues**

If you see errors related to the wasm32 target:

```bash
# Try reinstalling the target
rustup target remove wasm32-unknown-unknown
rustup target add wasm32-unknown-unknown
```

**Cargo PBC Command Not Found**

If the `cargo pbc` command is not recognized:

```bash
# Check if it's installed
cargo install --list | grep partisia

# Reinstall if necessary
cargo install cargo-partisia-contract
```

**Build Failures**

If contract builds fail:

```bash
# Ensure your contract dependencies are updated
cargo pbc init

# Try cleaning the build
cargo clean
```

For additional help, visit our Developer Community resources.

### Essential Resources

| Resource                          | Description                        | Link                                                                                  |
| --------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------- |
| Partisia Blockchain Documentation | Complete developer documentation   | [Documentation Home](../dev-tools/)                                                   |
| Example Contracts                 | Reference implementations          | [Example Contracts](https://gitlab.com/partisiablockchain/language/example-contracts) |
| Testnet Browser                   | Deploy and interact with contracts | [Testnet Browser](https://browser.testnet.partisiablockchain.com/)                    |
| Rust Documentation                | Official Rust language docs        | [Rust Docs](https://doc.rust-lang.org/book/)                                          |
| Smart Contract CLI                | Compiler documentation             | Install the Smart Contract CLI                                                        |
| ZK Reference                      | Zero-knowledge language reference  | ZK Rust Reference                                                                     |

### Developer Community

Join our developer community to get help, share knowledge, and stay updated:

* [Discord Community](https://discord.gg/partisia)
* [GitLab Repository](https://gitlab.com/partisiablockchain)
* [Medium Blog](https://medium.com/partisia-blockchain)

***

{% hint style="info" %}
L**ooking for a specific topic?** Browse our comprehensive documentation to find detailed guides on every aspect of Partisia Blockchain development.
{% endhint %}

