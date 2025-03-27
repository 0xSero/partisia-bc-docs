# Building Transactions and RPC Calls

### Introduction to Builders

The Partisia Blockchain ABI client provides a set of builder classes that help you construct properly formatted transactions and RPC calls to interact with smart contracts. These builders leverage the type information from the ABI to ensure your calls are correctly structured.

### RPC Builders

RPC (Remote Procedure Call) builders help you construct function calls to contracts. The main class for this is `RpcBuilder`.

#### Basic RPC Building

```typescript
import { AbiParser, RpcBuilder } from "@partisiablockchain/abi-client";

// Parse the ABI
const fileAbi = new AbiParser(buffer).parseAbi();
const contractAbi = fileAbi.contract();

// Create an RPC builder for a specific action
const builder = new RpcBuilder(contractAbi, "transfer");

// Add arguments
builder
  .addAddress("recipient_address") // Add an address argument
  .addU64(1000); // Add a u64 argument for the amount

// Get the serialized bytes
const rpcBytes = builder.getBytes();
```

#### Using Contract Calls

For more complex scenarios involving binder invocations with contract calls:

```typescript
// Create a builder for the "invoke" binder
const builder = new RpcBuilder(contractAbi, "invoke");

// Add the contract call and its arguments
const contractCallBuilder = builder.contractCall("transfer");
contractCallBuilder
  .addAddress("recipient_address")
  .addU64(1000);

// Get the serialized bytes
const rpcBytes = builder.getBytes();
```

### Transaction Builders

For building complete blockchain transactions, use the `TransactionBuilder`:

```typescript
import { TransactionBuilder } from "@partisiablockchain/abi-client";

// Create a transaction builder
const txBuilder = TransactionBuilder.createTransactionBuilder();

// Add signature
txBuilder.addSignature(signatureBytes);

// Add inner part
const innerBuilder = txBuilder.addStruct();

// Add core part
const coreBuilder = innerBuilder.addStruct();
coreBuilder
  .addU64(1) // nonce
  .addU64(Date.now() + 3600000) // valid_to_time
  .addU64(100000); // cost

// Add transaction
const txTypeBuilder = coreBuilder.addStruct();

// Add contract ID
txTypeBuilder.addAddress(contractAddressBytes);

// Add payload (RPC bytes from earlier)
txTypeBuilder.addVecU8(rpcBytes);

// Get the final transaction bytes
const transactionBytes = txBuilder.getBytes();
```

### Working with Complex Types

The builders support all the complex types defined in the ABI:

#### Structs

```typescript
const structBuilder = builder.addStruct();
structBuilder
  .addU32(123) // first field
  .addString("hello"); // second field
```

#### Enums

```typescript
const enumBuilder = builder.addEnumVariant(2); // discriminant value
enumBuilder
  .addU32(123) // enum variant field
  .addString("hello"); // enum variant field
```

#### Vectors

```typescript
// Adding a vector of u32
const vecBuilder = builder.addVec();
vecBuilder.addU32(1);
vecBuilder.addU32(2);
vecBuilder.addU32(3);

// Alternative for byte vectors
builder.addVecU8(Buffer.from([1, 2, 3, 4]));
```

#### Maps

```typescript
const mapBuilder = builder.addMap();
// Add key-value pair 1
mapBuilder.addString("key1"); // key
mapBuilder.addU32(123); // value
// Add key-value pair 2
mapBuilder.addString("key2"); // key
mapBuilder.addU32(456); // value
```

#### Options

```typescript
// Some value
const optionBuilder = builder.addOption();
optionBuilder.addU32(123);

// None value
const emptyOptionBuilder = builder.addOption();
// Don't add any value
```

### Practical Example: Token Transfer Transaction

Here's a complete example of building a transaction to transfer tokens:

```typescript
import { 
  AbiParser, 
  RpcBuilder, 
  TransactionBuilder 
} from "@partisiablockchain/abi-client";

async function buildTokenTransferTransaction(
  contractAddress: string,
  recipientAddress: string,
  amount: number,
  signerPrivateKey: Buffer
) {
  // 1. Get the contract ABI
  const abiResponse = await fetch(`/api/contract/${contractAddress}/abi`);
  const abiBuffer = Buffer.from(await abiResponse.arrayBuffer());
  const fileAbi = new AbiParser(abiBuffer).parseAbi();
  const contractAbi = fileAbi.contract();
  
  // 2. Build the RPC call
  const rpcBuilder = new RpcBuilder(contractAbi, "invoke");
  const transferBuilder = rpcBuilder.contractCall("transfer");
  transferBuilder
    .addAddress(recipientAddress)
    .addU64(amount);
  const rpcBytes = rpcBuilder.getBytes();
  
  // 3. Get current account info (nonce)
  const accountResponse = await fetch(`/api/account/${senderAddress}`);
  const accountData = await accountResponse.json();
  const nonce = accountData.nonce + 1;
  
  // 4. Build the transaction
  const txBuilder = TransactionBuilder.createTransactionBuilder();
  
  // 5. Create the inner part (this is what gets signed)
  const innerBuilder = txBuilder.addStruct();
  
  // 6. Add core transaction data
  const coreBuilder = innerBuilder.addStruct();
  coreBuilder
    .addU64(nonce)
    .addU64(Date.now() + 3600000) // Valid for 1 hour
    .addU64(100000); // Gas limit
  
  // 7. Add the contract interaction
  const txTypeBuilder = coreBuilder.addStruct();
  txTypeBuilder
    .addAddress(contractAddress)
    .addVecU8(rpcBytes);
  
  // 8. Get the bytes to sign
  const innerBytes = innerBuilder.getBytes();
  
  // 9. Sign the inner bytes (assuming signData function exists)
  const signature = signData(innerBytes, signerPrivateKey);
  
  // 10. Add the signature to the transaction
  txBuilder.addSignature(signature);
  
  // 11. Get the final transaction bytes
  return txBuilder.getBytes();
}
```

### Tips for Building Transactions

1. **Order Matters**: Add fields in the exact order they appear in the ABI.
2. **Type Checking**: The builders do runtime type checking to ensure the values match the expected types.
3. **Error Handling**: The builders throw descriptive errors if you try to add values that don't match the expected types.
4. **Gas Costs**: Always include reasonable gas limits. Too low and your transaction will fail, too high and you waste resources.
5. **Transaction Validity**: Set an appropriate validity time window (`valid_to_time`). This is typically the current time plus a few minutes or hours.
6. **Nonces**: Always use a nonce value that is one higher than your current account's nonce to ensure proper transaction sequencing.
7. **Signatures**: The transaction signature must be generated after building the inner transaction part (but before finalizing the entire transaction).

### Using Builders with the Blockchain API

Typically, after building a transaction, you would send it to the blockchain:

```typescript
async function sendTransaction(transactionBytes: Buffer) {
  const response = await fetch('/api/transactions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/octet-stream'
    },
    body: transactionBytes
  });
  
  return await response.json();
}
```

By following these patterns, you can build and execute complex transactions on the Partisia Blockchain network while ensuring type safety and correct formatting.
