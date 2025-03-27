# BlockchainTransactionClient

The `BlockchainTransactionClient` is the central component of the package, providing comprehensive transaction handling capabilities for the Partisia Blockchain.

### Creating a Transaction Client

There are multiple ways to create a transaction client depending on your requirements:

#### Standard Client

```javascript
import { 
  BlockchainTransactionClient, 
  SenderAuthenticationKeyPair 
} from '@partisiablockchain/blockchain-api-transaction-client';

// Create sender authentication from private key
const senderAuthentication = SenderAuthenticationKeyPair.fromString(privateKey);

// Create standard client
const client = BlockchainTransactionClient.create(
  'https://node1.testnet.partisiablockchain.com',
  senderAuthentication
);
```

#### Custom API Client

```javascript
// For custom API implementations
const client = BlockchainTransactionClient.createForCustomApi(
  customTransactionClient,
  senderAuthentication,
  transactionValidityInMillis,  // Optional
  spawnedEventTimeoutInMillis   // Optional
);
```

### Signing Transactions

The signing process prepares a transaction for submission to the blockchain:

```javascript
// Define transaction
const transaction = {
  address: contractAddress,
  rpc: serializedRpcBuffer
};

// Sign transaction (returns SignedTransaction)
const signedTransaction = await client.sign(
  transaction,
  gasCost,
  applicableUntil  // Optional timestamp when transaction becomes invalid
);
```

### Sending Transactions

Once signed, transactions can be sent to the blockchain:

```javascript
// Send previously signed transaction
const sentTransaction = await client.send(signedTransaction);

// Or sign and send in one step
const sentTransaction = await client.signAndSend(transaction, gasCost);
```

### Transaction Lifecycle Management

The client provides methods to track and manage the full lifecycle of transactions:

#### Waiting for Block Inclusion

```javascript
// Wait until transaction is included in a block
const executedTransaction = await client.waitForInclusionInBlock(sentTransaction);
```

#### Waiting for Spawned Events

```javascript
// Wait for all triggered events to complete
const transactionTree = await client.waitForSpawnedEvents(sentTransaction);

// Check if any part of the transaction tree failed
if (transactionTree.hasFailures()) {
  console.error('Transaction or some spawned events failed');
}
```

#### Ensuring Transaction Inclusion

For critical transactions, the client can retry until successful inclusion or timeout:

```javascript
// Try to ensure transaction is included before specified time
const executedTransaction = await client.ensureInclusionInBlock(
  transaction,
  gasCost,
  Date.now() + 60000,  // Try to include within 1 minute
  30000               // Wait for last valid block
);
```

### Configuration Options

The client has several configurable timeouts and durations:

* `DEFAULT_TRANSACTION_VALIDITY_DURATION`: 180,000ms (3 minutes) - How long a signed transaction is valid
* `DEFAULT_SPAWNED_EVENT_TIMEOUT`: 600,000ms (10 minutes) - How long to wait for spawned events
* Custom timeouts can be specified when creating the client or in individual method calls

### Error Handling

For robust applications, always use try/catch blocks when dealing with blockchain operations:

```javascript
try {
  const sentTransaction = await client.signAndSend(transaction, gasCost);
  const executedTransaction = await client.waitForInclusionInBlock(sentTransaction);
  console.log('Transaction successful:', executedTransaction);
} catch (error) {
  console.error('Transaction failed:', error.message);
  // Handle specific error conditions
}
```
