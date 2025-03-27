# ZK-client - Typescript

## @partisiablockchain/zk-client

The `@partisiablockchain/zk-client` library enables developers to interact with Zero-Knowledge (ZK) contracts on the Partisia Blockchain. This library provides functionality for generating Remote Procedure Calls (RPCs) for both On-chain and Off-chain ZK inputs, as well as reconstructing secret variables.

### Overview

Zero-Knowledge contracts on Partisia Blockchain enable privacy-preserving computations where inputs remain secret. This library helps developers interact with these contracts by:

1. **Generating ZK Inputs On-chain**: Send encrypted secret inputs directly through blockchain transactions
2. **Generating ZK Inputs Off-chain**: Send secret inputs through a combination of blockchain transactions and direct node communication
3. **Reconstructing Secret Variables**: Retrieve and decode secret variables stored in ZK contracts

### Installation

Install the library using npm:

```bash
npm install @partisiablockchain/zk-client
```

### Key Concepts

#### Binary Secret Shares

The library uses Shamir's Secret Sharing scheme to split binary secret data into shares. Each share is distributed to a different ZK node (engine) associated with the contract. This ensures that no single node can learn the secret value, but together they can compute results based on it.

#### On-chain vs Off-chain Inputs

* **On-chain Inputs**: Secret data is encrypted for each ZK node and included directly in a blockchain transaction. This is simpler but more expensive for larger inputs.
* **Off-chain Inputs**: Only commitments (hashes) of secret data are included in the blockchain transaction, while the actual blinded shares are sent directly to ZK nodes. This approach is more gas-efficient for larger inputs.

### Core Components

#### BinarySecretShares

The `BinarySecretShares` class is the central component for handling secret data:

{% code title="Typescript" %}
```typescript
import { BinarySecretShares, CompactBitArray } from "@partisiablockchain/zk-client";

// Create binary secret shares from secret data
const secretShares = BinarySecretShares.create(secretData);

// Encrypt shares for on-chain input
const encryptedShares = secretShares.encrypt(ephemeralKey, senderAddress, engineKeys);

// Apply blinding for off-chain input
const blindedShares = secretShares.applyBlinding(randomNumberGenerator);

// Reconstruct a secret from shares
const reconstructedSecret = secretShares.reconstructSecret();
```
{% endcode %}

#### ZkRpcBuilder

The `ZkRpcBuilder` provides utility functions for creating RPCs:

{% code title="Typescript" %}
```typescript
import { ZkRpcBuilder } from "@partisiablockchain/zk-client";

// Generate on-chain input RPC
const onChainRpc = ZkRpcBuilder.zkInputOnChain(
  senderAddress,
  secretInput,
  additionalRpc,
  engineKeys,
  encryptionKey,
  randomNumberGenerator
);

// Generate off-chain input RPC
const offChainInput = ZkRpcBuilder.zkInputOffChain(
  secretInput,
  additionalRpc,
  randomNumberGenerator
);
```
{% endcode %}

### Detailed Usage Guide

#### Creating and Sending On-chain ZK Input

On-chain input sends the encrypted shares directly in the transaction:

{% code title="Typescript" %}
```typescript
import { 
  BitOutput, 
  BlockchainAddress, 
  BlockchainPublicKey,
  ZkRpcBuilder 
} from "@partisiablockchain/zk-client";

// 1. Prepare the secret input (e.g., a number)
const secretValue = 123;
const serializedInput = BitOutput.serializeBits((out) => 
  out.writeSignedNumber(secretValue, 32)
);

// 2. Create additional RPC data (e.g., contract action identifier)
const SECRET_INPUT_INVOCATION = 0x40;
const additionalRpc = Buffer.from([SECRET_INPUT_INVOCATION]);

// 3. Get the ZK contract's engine public keys
const engineKeys = getEngineKeys(zkContract);

// 4. Generate the RPC payload
const senderAddress = transactionSender.getAddress();
const rpc = ZkRpcBuilder.zkInputOnChain(
  senderAddress,
  serializedInput,
  additionalRpc,
  engineKeys
);

// 5. Send the transaction
const transactionPointer = await transactionSender.sendAndSign(
  { address: ZK_CONTRACT, rpc },
  new BN(100000) // Gas limit
);
```
{% endcode %}

#### Creating and Sending Off-chain ZK Input

Off-chain input requires sending the transaction with commitments followed by sending blinded shares directly to nodes:

{% code title="Typescript" %}
```typescript
import { 
  BitOutput, 
  ZkRpcBuilder 
} from "@partisiablockchain/zk-client";

// 1. Prepare the secret input
const secretValue = 1000;
const serializedInput = BitOutput.serializeBits((out) =>
  out.writeSignedNumber(secretValue, 32)
);

// 2. Create additional RPC data
const SECRET_INPUT_INVOCATION = 0x40;
const additionalRpc = Buffer.from([SECRET_INPUT_INVOCATION]);

// 3. Generate the RPC payload and blinded shares
const payload = ZkRpcBuilder.zkInputOffChain(serializedInput, additionalRpc);

// 4. Send the transaction with commitments
const sentTransaction = await transactionSender.sendAndSign(
  { address: ZK_CONTRACT, rpc: payload.rpc },
  new BN(100000) // Gas limit
);

const txIdentifier = sentTransaction.transactionPointer.identifier;

// 5. Send the blinded shares to the nodes
const realClient = RealClient.create(zkContract);
const succeeded = await realClient.sendSharesToNodes(
  ZK_CONTRACT,
  transactionSender.getAddress(),
  txIdentifier,
  payload.blindedShares
);
```
{% endcode %}

#### Reconstructing a Secret Variable

To retrieve and reconstruct a secret variable:

{% code title="Typescript" %}
```typescript
import { 
  BinarySecretShares, 
  BitInput, 
  BlockchainAddress 
} from "@partisiablockchain/zk-client";

// 1. Create a client to interact with the blockchain
const client = new Client(NODE_URL);
const zkContract = await client.getContractState(ZK_CONTRACT);

// 2. Get the secret shares from the ZK contract's engines
const realClient = RealClient.create(zkContract);
const secretShares = await realClient.getSharesFromEngines(
  ZK_CONTRACT,
  SENDER_PRIVATE_KEY,
  VARIABLE_ID
);

// 3. Reconstruct the secret variable
const reconstructedSecret = secretShares.reconstructSecret();

// 4. Convert the binary data to the original format (e.g., number)
const reconstructedInt = new BitInput(reconstructedSecret.data).readSignedNumber(32);
```
{% endcode %}

### Error Handling

The library throws exceptions in various scenarios:

{% code title="Typescript" %}
```typescript
try {
  // Attempt to use the library
  const secretShares = BinarySecretShares.create(secretInput);
  // ...
} catch (error) {
  if (error.message.includes("Number of engines")) {
    // Handle mismatch between number of engines and shares
  } else if (error.message.includes("Failed to sign")) {
    // Handle signing failure
  } else if (error.message.includes("Interpolated polynomial")) {
    // Handle reconstruction error
  } else {
    // Handle other errors
  }
}
```
{% endcode %}

### Security Considerations

1. **Private Key Protection**: The sender's private key must be kept secure as it's used for transaction signing and authenticating with ZK nodes.
2. **Randomness Quality**: Always use cryptographically secure random number generators, as the security of secret sharing depends on the quality of randomness.
3. **Timing Considerations**: After sending an off-chain transaction, ensure blinded shares are sent to nodes promptly, as there's a window where they'll accept the shares.

### Technical Details

#### Secret Sharing Implementation

The library uses a degree-1 polynomial in a binary extension field (GF(2⁴)) for secret sharing:

1. For each bit of the secret, a random polynomial of the form `f(x) = secret_bit + random * x` is created
2. Each share is created by evaluating the polynomial at a different point (alpha)
3. Reconstructing the secret requires interpolating the polynomial using Lagrange interpolation and extracting the constant term

#### Encryption Process

For on-chain inputs, an ephemeral key pair is used to establish a shared secret with each ZK node:

1. A shared key is derived using Elliptic Curve Diffie-Hellman ([ECDH](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie%E2%80%93Hellman)) between the ephemeral private key and each node's public key
2. AES-128-CBC encryption is used with the derived shared key to encrypt the shares
3. The sender's address is also encrypted to prevent spoofing

#### Blinding Process

For off-chain inputs, blinding is applied to the shares before hashing them:

1. Random bytes (32 bytes per share) are generated as blindings
2. Each share is concatenated with its blinding and hashed
3. Only the hashes are included in the transaction, while the actual blinded shares are sent directly to nodes
