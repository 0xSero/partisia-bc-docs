# Key Concepts

## Key Concepts

Understanding these fundamental concepts will help you develop effectively on Partisia Blockchain.

### Blockchain Architecture

### Sharding

Partisia Blockchain uses sharding to scale transaction throughput. Each shard processes transactions independently, with cross-shard communication handled automatically by the protocol.

### FastTrack Consensus

Our unique consensus protocol delivers speed-of-light finalization while maintaining security. FastTrack achieves this through optimistic execution with built-in failure recovery mechanisms.

### Account Model

Partisia uses an account-based model where each account has:

* An **address** (derived from a public key)
* A **nonce** (for transaction ordering)
* **Balances** for MPC tokens and BYOC currencies
* Optional **staked tokens** for node operations

## Smart Contracts

### Contract Types

Partisia Blockchain supports three types of smart contracts:

1. [**Public Contracts**:](../guides/gas/contract-to-contract-estimates.md) Standard smart contracts with public execution
2. [**Zero-Knowledge Contracts**:](../guides/zk-smart-contracts/) Privacy-preserving contracts using MPC
3. [**System Contracts**:](https://browser.testnet.partisiablockchain.com/contracts?tab=system) Core blockchain functionality contracts

### Contract Lifecycle

Smart contracts follow a lifecycle of:

1. **Deployment**: Contract is uploaded to the blockchain
2. **Initialization**: Initial state is set
3. **Execution**: Contract processes actions
4. **Expiration**: Contract reaches end of life (optional)

### ZK Computation Model

Zero-knowledge contracts operate differently from traditional smart contracts:

* **Secret Sharing**: Inputs are split into shares distributed to multiple nodes
* **Computation Nodes**: A subset of nodes performs private computation
* **Public Verification**: Results can be verified without revealing inputs

## Token Economics

### Dual Token Model

Partisia Blockchain employs a decoupled token economy:

1. [**MPC Token**:](broken-reference) Native token used for staking and node operations
2. [**BYOC (Bring Your Own Coin)**:](../guides/byoc/) External tokens used for transaction fees

### Gas Model

All blockchain operations consume gas, which represents the computational resources used. Gas is paid in BYOC tokens, with prices fixed in fiat value rather than token value.

## Privacy Technologies

### Multi-Party Computation (MPC)

MPC allows computation on encrypted data without revealing the inputs:

* **Input Privacy**: Data remains encrypted during computation
* **Output Control**: Results can be selectively revealed
* **Verifiability**: Computations produce cryptographic proofs

### Secret Sharing

Secret sharing splits sensitive data into meaningless "shares" distributed across nodes:

* Reconstructing the original data requires a threshold of shares
* No single node can access the complete data
* Computations are performed on shares directly

## Cross-Chain Interoperability

### Bridges

Partisia Blockchain connects to other blockchains through [bridge contracts](../guides/byoc/bridging-byoc-by-sending-transactions.md):

* **Token Bridges**: Allow tokens to move between chains
* **Oracle Integration**: External data feeds through trusted price oracles
* **Second-Layer Solutions**: Partisia can serve as a privacy layer for other chains

### Next Steps

Now that you understand the fundamental concepts of Partisia Blockchain, proceed to Your First Application to start building a practical application.
