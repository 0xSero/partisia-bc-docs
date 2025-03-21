# CLI

## Partisia Blockchain CLI

### What is the Partisia Blockchain CLI?

The Partisia Blockchain Command Line Interface (CLI) is a powerful developer tool that enables efficient interaction with the Partisia Blockchain ecosystem. Implemented as a Cargo subcommand called <mark style="color:blue;">`cargo-partisia-contract`</mark> (commonly used with the shorthand <mark style="color:blue;">`cargo pbc`</mark>), this tool streamlines the entire development workflow from contract creation and compilation to deployment and blockchain interaction.

### Why Use the CLI?

The CLI serves as the primary gateway for developers to:

* **Develop Smart Contracts**: Create, build, and test smart contracts in Rust with enhanced privacy features
* **Deploy Contracts**: Easily deploy contracts to testnet or mainnet environments
* **Interact with the Blockchain**: Send transactions, query state, and manage accounts
* **Generate Client Code**: Automatically create TypeScript or Java bindings for your contracts

By providing these capabilities through a unified command-line interface, developers can efficiently build and deploy applications on Partisia Blockchain without the complexity of managing multiple tools.

### Key Features

* **Contract Development Acceleration**: Streamlined project setup, compilation, and [ABI generation](../../guides/fundementals/)
* **Zero-Knowledge Support**: Built-in tools for working with privacy-preserving [ZK contracts](../../guides/zk-smart-contracts/)
* **Account Management**: Generate keys, create accounts, and manage blockchain identities
* **Blockchain Interaction**: Query contract state, send transactions, and view transaction results
* **Network Configuration**: Easily switch between testnet, mainnet, or local development environments
* **Code Generation**: Create type-safe client code from contract ABIs for frontend integration

### Installation

Installing the CLI is straightforward using Cargo, Rust's package manager:

```bash
cargo install cargo-partisia-contract
```

After installation, you can verify it's working with:

```bash
cargo pbc --version
```

### Getting Started

New to Partisia Blockchain development? Here's a typical workflow to get started:

1.  **Create a new contract project**:

    ```bash
    cargo pbc new my_contract
    cd my_contract
    ```
2.  **Build your contract**:

    ```bash
    cargo pbc build --release
    ```
3.  **Deploy to testnet**:

    ```bash
    cargo pbc transaction deploy --pk=./mykey.pk target/wasm32-unknown-unknown/release/my_contract.wasm "Init Param" 123
    ```
4.  **Interact with your contract**:

    ```bash
    cargo pbc transaction action --pk=./mykey.pk <contract_address> performAction param1 param2
    ```

For detailed instructions on using the CLI's various commands and capabilities, explore the following sections:

* CLI Commands Reference: Comprehensive documentation of all CLI commands
* CLI Developer Guide: Practical guide with common workflows and examples

### CLI Command Quick Reference

This quick reference table helps you find commonly used commands for different tasks.

| Task                          | Command                                                       | Example                                             |
| ----------------------------- | ------------------------------------------------------------- | --------------------------------------------------- |
| **Build Contract**            | `cargo pbc build [--release]`                                 | `cargo pbc build --release`                         |
| **Deploy Contract**           | `cargo pbc transaction deploy <wasm_file> [params...]`        | `cargo pbc transaction deploy counter.wasm 0`       |
| **Call Contract Action**      | `cargo pbc transaction action <address> <action> [params...]` | `cargo pbc transaction action 01abc... increment`   |
| **Create Account**            | `cargo pbc account create [--file <path>]`                    | `cargo pbc account create --file mykey.pk`          |
| **Get Account Info**          | `cargo pbc account show <address>`                            | `cargo pbc account show 01abc...`                   |
| **Get Testnet Gas**           | `cargo pbc account mintgas <address>`                         | `cargo pbc account mintgas 01abc...`                |
| **View Contract State**       | `cargo pbc contract show <address> --state`                   | `cargo pbc contract show 01abc... --state`          |
| **Generate Client Code**      | `cargo pbc abi codegen [--ts/--java] <abi_file> <output>`     | `cargo pbc abi codegen --ts counter.abi counter.ts` |
| **View Transaction**          | `cargo pbc transaction show <tx_hash>`                        | `cargo pbc transaction show 0xabc...`               |
| **View Block**                | `cargo pbc block show <block_number/hash>`                    | `cargo pbc block show 123456`                       |
| **Configure Default Network** | `cargo pbc config net <network> [--default]`                  | `cargo pbc config net testnet --default`            |
| **Set Default Private Key**   | `cargo pbc config privatekey <path>`                          | `cargo pbc config privatekey ./mykey.pk`            |

### Next Steps

Ready to dive deeper? The CLI Commands Reference provides detailed documentation for each command, while the Developer Guide walks through common workflows with practical examples.
