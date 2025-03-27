# API Transaction Client - TS

## Blockchain API Transaction Client

The `@partisiablockchain/blockchain-api-transaction-client` package provides a comprehensive solution for building, signing, and sending transactions to the Partisia Blockchain. This client acts as a higher-level abstraction built on top of the core [Partisia Blockchain APIs.](../zk-client-ts/api-reference.md)

### Key Components

[The package](https://www.npmjs.com/package/@partisiablockchain/blockchain-api-transaction-client) consists of several key components:

* **BlockchainTransactionClient**: A high-level client that supports signing, sending, and waiting for transactions on the blockchain.
* **SignedTransaction**: Represents a transaction that has been signed by a private key and is ready to be sent to the blockchain.
* **SenderAuthenticationKeyPair**: Provides authentication based on a key-pair for signing transactions.
* **TransactionTree**: Represents the status of a transaction and its entire tree of spawned events.
* **ConditionWaiter**: Utility to wait for conditions to be fulfilled with automatic retries.

### Installation

```bash
npm install @partisiablockchain/blockchain-api-transaction-client
```

### Basic Usage

```javascript
import { 
  BlockchainTransactionClient, 
  SenderAuthenticationKeyPair 
} from '@partisiablockchain/blockchain-api-transaction-client';

// Create authentication from private key
const senderAuthentication = SenderAuthenticationKeyPair.fromString('your-private-key');

// Create transaction client
const client = BlockchainTransactionClient.create(
  'https://node1.testnet.partisiablockchain.com',
  senderAuthentication
);

// Define transaction
const transaction = {
  address: 'contract-address',
  rpc: serializedRpcPayload
};

// Sign and send transaction
const sentTransaction = await client.signAndSend(transaction, 100000);

// Wait for transaction to be included in a block
const executedTransaction = await client.waitForInclusionInBlock(sentTransaction);

// Wait for all spawned events
const transactionTree = await client.waitForSpawnedEvents(sentTransaction);
```

### Client Architecture

The package leverages lower-level API clients for interacting with the blockchain:

* Uses `ChainControllerApi` to get information about contracts and accounts
* Uses `ShardControllerApi` to get information about blocks and transactions
* Combines them with cryptographic utilities to build, sign and send transactions

The `BlockchainTransactionClient` is designed to simplify these interactions by providing a unified interface for transaction handling
