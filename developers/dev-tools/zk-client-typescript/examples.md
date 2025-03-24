# Examples

This section demonstrates practical examples of using the `@partisiablockchain/zk-client` library to interact with ZK contracts on the Partisia Blockchain.

### Example 1: On-Chain ZK Input

This example shows how to send an encrypted value to a ZK contract using the on-chain method.

```typescript
import { 
  BitOutput, 
  BlockchainAddress, 
  Client, 
  CryptoUtils, 
  TransactionSender, 
  ZkRpcBuilder 
} from "@partisiablockchain/zk-client";
import BN from "bn.js";

// Connection details
const TESTNET_URL = "https://node1.testnet.partisiablockchain.com";
const SENDER_PRIVATE_KEY = "your_private_key"; // Replace with your private key
const ZK_CONTRACT = BlockchainAddress.fromString("contract_address"); // Replace with contract address
const SECRET_INPUT_INVOCATION = 0x40; // Action identifier in the contract
const SECRET_INPUT_VALUE = 123; // The secret value to send

// Helper function to extract engine public keys from contract state
function getEngineKeys(zkContract) {
  return zkContract.serializedContract.engines.engines.map((e) =>
    BlockchainPublicKey.fromBuffer(Buffer.from(e.publicKey, "base64"))
  );
}

async function sendOnChainZkInput() {
  // Create client connection to testnet
  const client = new Client(TESTNET_URL);
  
  // Get contract state to access its ZK engines
  const zkContract = await client.getContractState(ZK_CONTRACT);
  if (!zkContract) {
    throw new Error("Couldn't get contract data");
  }
  
  // Create transaction sender from private key
  const transactionSender = TransactionSender.create(client, SENDER_PRIVATE_KEY);
  
  // Serialize the secret input value (32-bit signed integer)
  const serializedInput = BitOutput.serializeBits((out) =>
    out.writeSignedNumber(SECRET_INPUT_VALUE, 32)
  );
  
  // Prepare additional RPC data (contains the action identifier)
  const additionalRpc = Buffer.from([SECRET_INPUT_INVOCATION]);
  
  // Get public keys of ZK engines
  const engineKeys = getEngineKeys(zkContract);
  
  // Generate the RPC payload for on-chain ZK input
  const rpc = ZkRpcBuilder.zkInputOnChain(
    transactionSender.getAddress(),
    serializedInput,
    additionalRpc,
    engineKeys
  );
  
  // Send the transaction
  const transactionPointer = await transactionSender.sendAndSign(
    { address: ZK_CONTRACT, rpc },
    new BN(100000) // Gas limit
  );
  
  // Log the transaction hash
  console.log("Sent input in transaction:", 
    transactionPointer.transactionPointer.identifier.toString("hex"));
}

// Execute the function
sendOnChainZkInput().catch(console.error);
```

### Example 2: Off-Chain ZK Input

This example demonstrates how to send a secret value to a ZK contract using the off-chain method.

```typescript
import { 
  BitOutput, 
  BlockchainAddress, 
  Client, 
  RealClient, 
  TransactionSender, 
  ZkRpcBuilder 
} from "@partisiablockchain/zk-client";
import BN from "bn.js";

// Connection details
const TESTNET_URL = "https://node1.testnet.partisiablockchain.com";
const SENDER_PRIVATE_KEY = "your_private_key"; // Replace with your private key
const ZK_CONTRACT = BlockchainAddress.fromString("contract_address"); // Replace with contract address
const SECRET_INPUT_INVOCATION = 0x40; // Action identifier in the contract
const SECRET_INPUT_VALUE = 1000; // The secret value to send

async function sendOffChainZkInput() {
  // Create client connection to testnet
  const client = new Client(TESTNET_URL);
  
  // Get contract state to access its ZK engines
  const zkContract = await client.getContractState(ZK_CONTRACT);
  if (!zkContract) {
    throw new Error("Couldn't get contract data");
  }
  
  // Serialize the secret input value (32-bit signed integer)
  const serializedInput = BitOutput.serializeBits((out) =>
    out.writeSignedNumber(SECRET_INPUT_VALUE, 32)
  );
  
  // Prepare additional RPC data (contains the action identifier)
  const additionalRpc = Buffer.from([SECRET_INPUT_INVOCATION]);
  
  // Generate the RPC payload for off-chain ZK input
  const payload = ZkRpcBuilder.zkInputOffChain(serializedInput, additionalRpc);
  
  // Create transaction sender from private key
  const transactionSender = TransactionSender.create(client, SENDER_PRIVATE_KEY);
  
  // Send transaction with commitments (hashes of blinded shares)
  const sentTransaction = await transactionSender.sendAndSign(
    { address: ZK_CONTRACT, rpc: payload.rpc },
    new BN(100000) // Gas limit
  );
  
  const txIdentifier = sentTransaction.transactionPointer.identifier;
  console.log("Sent input in transaction:", txIdentifier.toString("hex"));
  
  // Create a client for interacting with ZK engines
  const realClient = RealClient.create(zkContract);
  
  // Send the blinded shares directly to the ZK nodes
  const succeeded = await realClient.sendSharesToNodes(
    ZK_CONTRACT,
    transactionSender.getAddress(),
    txIdentifier,
    payload.blindedShares
  );
  
  if (succeeded) {
    console.log("Successfully sent shares to ZK nodes");
  } else {
    console.log("Failed to send shares to ZK nodes");
  }
}

// Execute the function
sendOffChainZkInput().catch(console.error);
```

### Example 3: Reconstructing a Secret Variable

This example shows how to retrieve and reconstruct a secret variable from a ZK contract.

```typescript
import { 
  BinarySecretShares, 
  BitInput, 
  BlockchainAddress, 
  Client, 
  RealClient 
} from "@partisiablockchain/zk-client";

// Connection details
const TESTNET_URL = "https://node1.testnet.partisiablockchain.com";
const SENDER_PRIVATE_KEY = "your_private_key"; // Replace with your private key
const ZK_CONTRACT = BlockchainAddress.fromString("contract_address"); // Replace with contract address
const VARIABLE_ID = 17; // ID of the variable to reconstruct

async function reconstructSecretVariable() {
  // Create client connection to testnet
  const client = new Client(TESTNET_URL);
  
  // Get contract state to access its ZK engines
  const zkContract = await client.getContractState(ZK_CONTRACT);
  if (!zkContract) {
    throw new Error("Couldn't get contract data");
  }
  
  // Create a client for interacting with ZK engines
  const realClient = RealClient.create(zkContract);
  
  // Get shares from all ZK engines
  const secretShares = await realClient.getSharesFromEngines(
    ZK_CONTRACT,
    SENDER_PRIVATE_KEY,
    VARIABLE_ID
  );
  
  // Reconstruct the secret from the shares
  const reconstructedSecret = secretShares.reconstructSecret();
  
  // Convert the binary data to the original format (32-bit signed integer)
  const reconstructedInt = new BitInput(reconstructedSecret.data).readSignedNumber(32);
  
  console.log("Reconstructed secret value:", reconstructedInt);
}

// Execute the function
reconstructSecretVariable().catch(console.error);
```

### Example 4: Working with Different Data Types

This example demonstrates how to encode and decode different data types for ZK inputs.

```typescript
import { 
  BitOutput, 
  BitInput, 
  BinarySecretShares 
} from "@partisiablockchain/zk-client";
import BN from "bn.js";

// Example function to demonstrate encoding and decoding different data types
function dataTypeExample() {
  // Example 1: Encoding and decoding a signed 32-bit integer
  const intValue = 12345;
  const intData = BitOutput.serializeBits((out) => out.writeSignedNumber(intValue, 32));
  const secretShares1 = BinarySecretShares.create(intData);
  const reconstructed1 = secretShares1.reconstructSecret();
  const decodedInt = new BitInput(reconstructed1.data).readSignedNumber(32);
  console.log("Original integer:", intValue);
  console.log("Reconstructed integer:", decodedInt);
  
  // Example 2: Encoding and decoding a big number (64-bit)
  const bnValue = new BN("9223372036854775807"); // Max 64-bit signed integer
  const bnData = BitOutput.serializeBits((out) => out.writeSignedBN(bnValue, 64));
  const secretShares2 = BinarySecretShares.create(bnData);
  const reconstructed2 = secretShares2.reconstructSecret();
  const decodedBN = new BitInput(reconstructed2.data).readSignedBN(64);
  console.log("Original BN:", bnValue.toString());
  console.log("Reconstructed BN:", decodedBN.toString());
  
  // Example 3: Encoding and decoding a boolean array
  const boolArray = [true, false, true, true, false];
  const boolData = BitOutput.serializeBits((out) => {
    for (const value of boolArray) {
      out.writeBoolean(value);
    }
  });
  const secretShares3 = BinarySecretShares.create(boolData);
  const reconstructed3 = secretShares3.reconstructSecret();
  const input = new BitInput(reconstructed3.data);
  const decodedBools = [];
  for (let i = 0; i < boolArray.length; i++) {
    decodedBools.push(input.readBoolean());
  }
  console.log("Original booleans:", boolArray);
  console.log("Reconstructed booleans:", decodedBools);
}

// Execute the function
dataTypeExample();
```

### Example 5: Custom Randomness Generator

This example shows how to use a custom randomness generator for creating shares and blindings.

```typescript
import { 
  BitOutput, 
  BinarySecretShares, 
  ZkRpcBuilder 
} from "@partisiablockchain/zk-client";
import { createHash } from "crypto";

// Create a deterministic random number generator for testing
function createDeterministicRng(seed) {
  let counter = 0;
  const seedBuffer = Buffer.from(seed);
  
  return (length) => {
    const result = Buffer.alloc(length);
    let offset = 0;
    
    while (offset < length) {
      // Create a unique value for each call
      const hash = createHash("sha256")
        .update(seedBuffer)
        .update(Buffer.from([counter++]))
        .digest();
      
      const bytesToCopy = Math.min(hash.length, length - offset);
      hash.copy
```
