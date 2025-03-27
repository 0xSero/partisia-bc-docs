# Authentication and Transaction Structure

This document explains the authentication mechanisms and transaction structure when working with the Partisia Blockchain Transaction API client.

### Authentication

The package uses the `SenderAuthentication` interface for transaction signing. The primary implementation is the `SenderAuthenticationKeyPair` class.

#### SenderAuthenticationKeyPair

This implementation uses a cryptographic key pair (public and private key) for authentication and signing:

```javascript
import { SenderAuthenticationKeyPair } from '@partisiablockchain/blockchain-api-transaction-client';

// Create from a private key string (hexadecimal format)
const authentication = SenderAuthenticationKeyPair.fromString('your-private-key-in-hex');

// The address is derived from the public key
const address = authentication.getAddress();
```

#### Integration with Wallets

While the package includes key-pair based authentication, it can be extended to work with various wallet implementations:

* [Partisia Wallet](https://chromewebstore.google.com/detail/parti-wallet/gjkdbeaiifkpoencioahhcilildpjhgh)
* [Metamask Snap](../../guides/integrations/partisia-metamask-snap.md)
* Custom wallet implementations that conform to the `SenderAuthentication` interface

### Transaction Structure

#### Basic Transaction

The basic transaction object has a simple structure:

```javascript
const transaction = {
  address: '01a4082d9d560749ecd0ffa1dcaaaee2c2cb25d881', // Contract address
  rpc: serializedRpcBuffer // Binary payload for the contract call
};
```

#### SignedTransaction

When a transaction is signed, it produces a `SignedTransaction` object with additional metadata:

```javascript
const signedTransaction = await client.sign(transaction, gasCost);

// Properties of SignedTransaction
const nonce = signedTransaction.getNonce();         // Account nonce when signed
const validToTime = signedTransaction.getValidToTime(); // Expiration timestamp
const gasCost = signedTransaction.getGasCost();     // Allocated gas
const inner = signedTransaction.getInner();         // Original transaction
```

The `SignedTransaction` also includes methods to:

* Serialize the transaction for transmission
* Calculate the transaction identifier
* Access transaction parameters

#### SentTransaction

After sending, you receive a `SentTransaction` object that includes:

```javascript
// Structure of a sent transaction
const sentTransaction = {
  signedTransaction: signedTransaction,
  transactionPointer: {
    identifier: "11d09178b39c10520aec717200a4a5cd229e948bc15c4a87e65d682008f86db5",
    destinationShardId: "Shard0"
  }
};
```

#### ExecutedTransaction

When a transaction is executed on the blockchain, its status is represented by an `ExecutedTransaction` object:

```javascript
// Example structure (simplified)
const executedTransaction = {
  blockHeight: 12345,
  executionStatus: {
    success: true,
    events: [], // Spawned events
    finalized: true,
    gasUsed: 42000
  },
  transaction: { /* Transaction details */ }
};
```

#### TransactionTree

For transactions that spawn events, the `TransactionTree` provides a complete view of the transaction and all its spawned events:

```javascript
// Get the transaction tree
const tree = await client.waitForSpawnedEvents(sentTransaction);

// Check if any part of the tree failed
const hasFailures = tree.hasFailures();

// Access the main transaction
const mainTransaction = tree.transaction;

// Access all spawned events
const events = tree.events;
```

### Error Handling

The transaction client uses the `ConditionWaiter` utility to handle retries and timeouts when waiting for blockchain operations to complete. [Common error scenarios](../../guides/smart-contracts/debugging-guide.md) include:

* Network connectivity issues
* Transaction rejection due to invalid parameters
* Nonce conflicts
* Insufficient gas
* Contract execution failures

When these errors occur, the client throws appropriate exceptions that should be caught and handled in your application logic.
