# Troubleshooting Guide

### CLI Troubleshooting Guide

This guide addresses common issues encountered when using the Partisia Blockchain CLI.

#### Common Error Messages and Solutions

| Error Message                                                     | Possible Cause                 | Solution                                                                                   |
| ----------------------------------------------------------------- | ------------------------------ | ------------------------------------------------------------------------------------------ |
| `cargo-pbc-not-found`                                             | CLI not properly installed     | Run `cargo install cargo-partisia-contract`                                                |
| `Failed to build WASM`                                            | Rust code compilation error    | Check your code for syntax errors and make sure you've imported all necessary dependencies |
| `error: unknown or invalid target triple: wasm32-unknown-unknown` | WebAssembly target not added   | Run `rustup target add wasm32-unknown-unknown`                                             |
| `Transaction failed: insufficient gas`                            | Not enough gas provided        | Increase gas amount with `--gas=<amount>` parameter                                        |
| `Transaction failed: nonce mismatch`                              | Incorrect nonce value          | Check current account nonce with `cargo pbc account show <address>`                        |
| `Failed to connect to network`                                    | Network connectivity issues    | Check your internet connection and verify network configuration                            |
| `Private key file not found`                                      | Incorrect path to private key  | Ensure the private key file exists at the specified path                                   |
| `Failed to deserialize contract state`                            | Contract state format mismatch | Verify you're using the correct ABI and version                                            |

### Debugging Tips

**Build Issues**

If your contract fails to build:

{% code title="Bash" %}
```bash
# Initialize the project dependencies first
cargo pbc init

# Try building with verbose output
cargo pbc build --release -v

# Clean the build directory and try again
cargo clean
cargo pbc build --release
```
{% endcode %}

**Deployment Issues**

If your contract fails to deploy:

{% code title="Bash" %}
```git
# Check if you have enough gas
cargo pbc account show <your-address>

# Try increasing the gas limit
cargo pbc transaction deploy --gas=500000 <contract.wasm> <init_params>

# Verify the WASM file exists and is valid
cargo pbc print-version <contract.wasm>
```
{% endcode %}

**Transaction Issues**

If your transaction fails:

```bash
# Check transaction status
cargo pbc transaction show <tx-hash>

# Verify contract exists
cargo pbc contract show <contract-address>

# Try with higher gas limit
cargo pbc transaction action --gas=200000 <contract-address> <action> <params>

```

#### Getting Help

If you're still experiencing issues:

1. Check the detailed documentation for the specific command with `cargo pbc <command> --help`
2. Visit our [Discord community](https://discord.gg/partisia) for developer support
3. Search for similar issues in our [GitLab repository](https://gitlab.com/partisiablockchain)
4. Run commands with the `--verbose` flag for more detailed error message

