# Creating, Signing, and Sending Transactions

This guide explains how to create, sign, and send transactions on the Partisia Blockchain[ using various client libraries.](../../dev-tools/)

#### Using TypeScript

The TypeScript client provides a streamlined interface for transaction creation and signing:

{% code title="Tuypescript" %}
```typescript
import { PartisiaClient } from '@partisiablockchain/client';
import { TransactionBuilder } from '@partisiablockchain/transactions';

// Initialize the client
const client = new PartisiaClient({
  baseUrl: 'https://node1.testnet.partisiablockchain.com'
});

// Create a transaction
const transaction = TransactionBuilder.create()
  .withSender(senderAddress)
  .withNonce(nonce)
  .withAction('transfer', {
    recipient: recipientAddress,
    amount: '1000'
  })
  .withGas(100000)
  .build();

// Sign the transaction
const signedTransaction = await transaction.sign(privateKey);

// Send the transaction
const result = await client.sendTransaction(signedTransaction);
console.log(`Transaction hash: ${result.hash}`);
```
{% endcode %}

#### Using Java

For Java applications, the process follows a similar pattern:

{% code title="Java" overflow="wrap" %}
```java
import com.partisiablockchain.sdk.PartisiaClient;
import com.partisiablockchain.sdk.transaction.Transaction;

// Initialize client
PartisiaClient client = PartisiaClient.builder()
    .setReaderNodeUrl("https://reader.testnet.partisiablockchain.com")
    .build();

// Build transaction
Transaction tx = Transaction.builder()
    .setSender(sender)
    .setNonce(nonce)
    .setAction("transfer", recipient, amount)
    .setGas(100000)
    .build();

// Sign transaction
byte[] signature = signer.sign(tx.getSerializedBytes());

// Send transaction
String txHash = client.sendTransaction(tx, signature).get();
System.out.println("Transaction hash: " + txHash);
```
{% endcode %}

#### Transaction Lifecycle

1. **Creation**: Build a transaction with sender, nonce, action, and gas limit
2. **Signing**: Cryptographically sign the transaction using the sender's private key
3. **Submission**: Send the signed transaction to a blockchain node
4. **Validation**: The node validates the transaction (signature, nonce, gas)
5. **Execution**: If valid, the transaction is executed and included in a block
6. **Confirmation**: After sufficient block confirmations, the transaction is considered final

#### Best Practices

* Always increment the nonce for consecutive transactions from the same account
* Set appropriate gas limits based on the complexity of the operation
* Include a reasonable expiration time for transactions (usually a few minutes to an hour)
* Verify transaction success by checking the receipt or querying the resulting state
* Implement proper error handling for transaction failures

#### Security Considerations

* Private keys should be handled securely and never exposed in client-side code
* Consider using a secure wallet integration for signing operations instead of directly managing private keys
* Remember that while the secret values themselves are protected, metadata may reveal information about the secret
* Always validate secret types before usage to avoid unexpected type conversion issues

#### Related Resources

* [ZK Contract Development Guide](../zk-smart-contracts/)
* [ZK Client Libraries](../../dev-tools/)
