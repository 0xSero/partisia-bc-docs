# Install the smart contract compiler

### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

To develop and compile contracts for the Partisia Blockchain, you need to install Rust. To install Rust for you platform follow the instructions on [https://rustup.rs/](https://rustup.rs/).

Instruct rust to use the latest stable version by running:

```
rustup install stable
rustup default stable
```

The target needs to be added manually, you add the target by running:

```
rustup target add wasm32-unknown-unknown
```

To compile contracts you will also need to install [Git](https://git-scm.com/downloads).

If you need to develop zero-knowledge contracts then you will also need to install [Java 17](https://openjdk.org/) to run the zk-compiler.

If Working from a Windows machine you must [get Visual Studio with C++ build tools](https://visualstudio.microsoft.com/downloads/) - In Visual Studio Installer choose _Desktop development with C++_.

* We recommend you make sure these optionals are marked:
  * MSVC x64/x86 build tools
  * Windows 11 SDK
  * C++ CMake tools for Windows
  * Testing tools core features
  * Build Tools
  * C++ AddressSanitizer

#### 2) Installing the cargo `partisia-contract`/`pbc` command <a href="#id-2-installing-the-cargo-partisia-contractpbc-command" id="id-2-installing-the-cargo-partisia-contractpbc-command"></a>

The [partisia-contract tool](https://crates.io/crates/cargo-partisia-contract) is a small application that helps you compile a contract. To compile it and install it using cargo run:

```
cargo install cargo-partisia-contract
```

The `cargo partisia-contract` and `cargo pbc` are interchangeable commands, the `cargo pbc` is just an alias. To verify the tool you can locally execute: `cargo partisia-contract --version` and `cargo pbc --version`. This command prints the version of the tool. If you want to learn more about the partisia contract tool you can [visit our tooling page](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#command-line-tools).

#### 3) Download Example Contracts <a href="#id-3-download-example-contracts" id="id-3-download-example-contracts"></a>

We supply a small archive with example contracts which can be compiled using the tooling from above. The example contracts are [available here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-examples.html).
