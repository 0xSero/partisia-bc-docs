# Commands Reference

## CLI Commands Reference

This reference provides comprehensive documentation for all Partisia Blockchain CLI commands. Each command includes options, subcommands, and examples to help you effectively interact with the Partisia Blockchain ecosystem.

### Command Structure

The CLI uses a hierarchical command structure:

```
cargo pbc [COMMAND] [SUBCOMMAND] [OPTIONS]
```

All commands support the `--help` flag to display usage information.

### Contract Development Commands

These commands focus on creating, building, and managing smart contract code.

#### build

Compiles contracts to WebAssembly and generates ABI files.

```bash
cargo pbc build [OPTIONS] [ADDITIONAL_ARGS]...
```

| Option                    | Description                                            |
| ------------------------- | ------------------------------------------------------ |
| `-r, --release`           | Build artifacts in release mode with optimizations     |
| `-n, --no-abi`            | Skip generating .abi file                              |
| `-q, --quiet`             | No messages printed to stdout                          |
| `-w, --no-wasm-strip`     | Do not remove custom sections from WASM file           |
| `-z, --no-zk`             | Skip compilation of ZK computation                     |
| `--workspace`             | Build all packages in the workspace                    |
| `--coverage`              | Compile an instrumented binary for coverage generation |
| `--manifest-path <PATH>`  | Path to the Cargo.toml file                            |
| `-p, --package <PACKAGE>` | Build only specified packages                          |

**Example:**

```bash
# Build a contract in release mode
cargo pbc build --release
```

#### init

Initializes the contract by retrieving dependencies for build.

```bash
cargo pbc init [OPTIONS]
```

| Option                   | Description                            |
| ------------------------ | -------------------------------------- |
| `--workspace`            | Init all ZK contracts in the workspace |
| `--manifest-path <PATH>` | Path to the Cargo.toml file            |

**Example:**

```bash
# Initialize a contract project
cargo pbc init
```

#### print-version

Prints the client and binder version of the contract.

```bash
cargo pbc print-version [OPTIONS] <WASM_CONTRACT>
```

| Option           | Description                     |
| ---------------- | ------------------------------- |
| `-b, --bashlike` | Print version as bash variables |

**Example:**

```bash
cargo pbc print-version target/wasm32-unknown-unknown/release/my_contract.wasm
```

#### path-of-wasm

Prints the expected WASM file path based on Cargo.toml context.

```bash
cargo pbc path-of-wasm [OPTIONS]
```

| Option                   | Description                                |
| ------------------------ | ------------------------------------------ |
| `-r, --release`          | File is in release folder instead of debug |
| `--manifest-path <PATH>` | Path to the Cargo.toml file                |

**Example:**

```bash
cargo pbc path-of-wasm --release
```

#### path-of-abi

Prints the expected ABI file path based on Cargo.toml context.

```bash
cargo pbc path-of-abi [OPTIONS]
```

| Option                   | Description                                |
| ------------------------ | ------------------------------------------ |
| `-r, --release`          | File is in release folder instead of debug |
| `--manifest-path <PATH>` | Path to the Cargo.toml file                |

**Example:**

```bash
cargo pbc path-of-abi --release
```

#### set-sdk

Updates the SDK used for compiling contracts.

```bash
cargo pbc set-sdk [OPTIONS] <SDK>
```

| Option                    | Description                           |
| ------------------------- | ------------------------------------- |
| `--workspace`             | Set SDK for all packages in workspace |
| `--manifest-path <PATH>`  | Path to the Cargo.toml file           |
| `-p, --package <PACKAGE>` | Set SDK only for specified packages   |

**Example:**

```bash
# Set SDK to specific version
cargo pbc set-sdk "git: https://git@gitlab.com/partisiablockchain/language/contract-sdk.git, tag: 9.1.2"
```

### Blockchain Interaction Commands

These commands allow you to interact with the Partisia Blockchain network.

#### transaction

Sign, send, and interact with transactions.

```bash
cargo pbc transaction [SUBCOMMAND] [OPTIONS]
```

**Subcommands:**

| Subcommand | Description                                     |
| ---------- | ----------------------------------------------- |
| `action`   | Call a specific contract action with parameters |
| `deploy`   | Deploy a new smart contract to the blockchain   |
| `raw`      | Send a transaction with specific RPC bytes      |
| `sign`     | Sign a prebuilt unsigned transaction            |
| `send`     | Send a prebuilt signed transaction              |
| `show`     | Show information about a transaction            |
| `latest`   | Get latest transactions from the blockchain     |
| `upgrade`  | Upgrade a smart contract on the blockchain      |

**Examples:**

```bash
# Deploy a contract
cargo pbc transaction deploy --pk=./mykey.pk my_contract.wasm "Init Param" 123

# Call a contract action
cargo pbc transaction action --pk=./mykey.pk 01abcdef... increment 5

# View transaction details
cargo pbc transaction show 0x7a8b9c...
```

#### account

Create and interact with blockchain accounts.

```bash
cargo pbc account [SUBCOMMAND] [OPTIONS]
```

**Subcommands:**

| Subcommand | Description                                      |
| ---------- | ------------------------------------------------ |
| `create`   | Create a private key and associated address      |
| `show`     | Show information about an account                |
| `mintgas`  | Mint gas for an account (testnet only)           |
| `address`  | Print blockchain address for a given private key |

**Examples:**

```bash
# Create a new account and save the private key
cargo pbc account create --file ./mykey.pk

# Get testnet gas
cargo pbc account mintgas 01abcdef...

# View account information
cargo pbc account show 01abcdef...
```

#### contract

Get information about deployed contracts.

```bash
cargo pbc contract [SUBCOMMAND] [OPTIONS]
```

**Subcommands:**

| Subcommand       | Description                              |
| ---------------- | ---------------------------------------- |
| `show`           | Show information about a contract        |
| `check-standard` | Check if a contract is a valid standard  |
| `secret`         | Interact with secrets from a ZK contract |
| `refuelgas`      | Add more gas to a contract               |

**Examples:**

```bash
# View contract state
cargo pbc contract show 01abcdef... --state

# Check if contract implements a standard
cargo pbc contract check-standard 01abcdef... mpc-20

# List secrets owned by an account
cargo pbc contract --pk=./mykey.pk secret list 01abcdef...
```

#### block

View latest or specific blocks.

```bash
cargo pbc block [SUBCOMMAND] [OPTIONS]
```

**Subcommands:**

| Subcommand | Description                           |
| ---------- | ------------------------------------- |
| `show`     | Show information about a block        |
| `latest`   | Get latest blocks from the blockchain |

**Examples:**

```bash
# View block details
cargo pbc block show 123456

# Get latest blocks
cargo pbc block latest
```

#### wallet

Create wallets for blockchain interaction.

```bash
cargo pbc wallet [SUBCOMMAND] [OPTIONS]
```

**Subcommands:**

| Subcommand | Description                            |
| ---------- | -------------------------------------- |
| `create`   | Create a new wallet and save to a file |

**Example:**

```bash
cargo pbc wallet create --file ./my_wallet.json
```

#### abi

View ABI information and generate client code.

```bash
cargo pbc abi [SUBCOMMAND] [OPTIONS]
```

**Subcommands:**

| Subcommand | Description                               |
| ---------- | ----------------------------------------- |
| `show`     | Show basic information about an ABI       |
| `codegen`  | Generate code to interact with a contract |

**Examples:**

```bash
# View ABI structure
cargo pbc abi show my_contract.abi

# Generate TypeScript client code
cargo pbc abi codegen --ts my_contract.abi output/my_contract.ts

# Generate Java client code
cargo pbc abi codegen --java my_contract.abi output/
```

### Configuration Commands

#### config

Set default values for CLI options.

```bash
cargo pbc config [SUBCOMMAND] [OPTIONS]
```

**Subcommands:**

| Subcommand   | Description                 |
| ------------ | --------------------------- |
| `privatekey` | Set default private key     |
| `net`        | Set default network         |
| `list`       | List current configurations |

**Examples:**

```bash
# Set default private key
cargo pbc config privatekey ./mykey.pk

# Configure for local network
cargo pbc config net local "http://localhost:9432,http://localhost:8300" --default

# List current configuration
cargo pbc config list
```

### Global Options

These options are available for most commands:

| Option            | Description                      |
| ----------------- | -------------------------------- |
| `-q, --quiet`     | No messages printed to stdout    |
| `-v, --verbose`   | Debug messages printed to stdout |
| `-h, --help`      | Print help information           |
| `-V, --version`   | Print version information        |
| `--net=<NETNAME>` | Blockchain network to target     |

### Network Configuration

The CLI supports multiple network configurations:

* `testnet`: The default Partisia Blockchain testnet
* `mainnet`: The Partisia Blockchain mainnet
* `local`: A local development network

You can set a default network using the config command:

```bash
cargo pbc config net local "http://localhost:9432,http://localhost:8300" --default
```

### Next Steps

Now that you're familiar with the CLI commands, check out the [CLI Developer Guide ](developer-guide.md)for practical workflows and examples that combine these commands into effective development patterns.
