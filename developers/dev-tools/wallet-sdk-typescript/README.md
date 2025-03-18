# Wallet SDK - Typescript

## Partisia SDK

The Partisia SDK creates a secure end-to-end encrypted channel between decentralized applications (dApps) and the Partisia Wallet, enabling secure authentication and transaction signing.

### Installation

```bash
npm install partisia-sdk
```

### Core Features

* Encrypted communication channel with the Partisia Wallet
* Permission-based authorization model
* Transaction and message signing
* Full contract interaction support

### Getting Started

#### Initialize SDK

```javascript
import PartisiaSdk from 'partisia-sdk'

// Create a new SDK instance
const sdk = new PartisiaSdk()
```

#### Connect to Wallet

```javascript
await sdk.connect({
  permissions: ['sign'],
  dappName: 'My First Partisia dApp',
  chainId: 'Partisia Blockchain'
})

// Verify connection
if (sdk.isConnected) {
  console.log(`Connected to account: ${sdk.connection.account.address}`)
}
```

### Connection Object Structure

After a successful connection, the SDK contains a connection object with the following structure:

```javascript
{
  "connection": {
    "account": {
      "address": "00cb7d60b722e85f7530e67652ef41ad84b9c8cd17",
      "pub": "02156ef2d8e1d406549822aa790d838d6c550f79a3c397d9912129b122203a75a3"
    },
    "popupWindow": {
      "tabId": 237,
      "box": "02408b13a15b07e479f4933f8b006907426238f048957187fd6eb077c25d044358"
    }
  },
  "seed": "62825305f0d64932fee23549d8dbd230d4d4d011ee2596f4f07e6af3c61ac8cb"
}
```

| Property  | Description                                                        |
| --------- | ------------------------------------------------------------------ |
| `address` | On-chain address of the connected account                          |
| `pub`     | Public key of the connected account                                |
| `seed`    | dApp's seed for generating a keypair used for wallet communication |
| `box`     | Wallet's public key used for dApp communication                    |

### Signing Messages

Once connected, you can request signature for arbitrary messages:

```javascript
const result = await sdk.signMessage({
  payload: 'Hello, Partisia Blockchain!',
  payloadType: 'utf8'
})

console.log(`Signature: ${result.signature}`)
console.log(`Transaction Hash: ${result.trxHash}`)
```

#### Message Types

The SDK supports three types of payload formats:

| Type          | Description                                          | Example                                   |
| ------------- | ---------------------------------------------------- | ----------------------------------------- |
| `utf8`        | Plain text messages                                  | `'Hello, Partisia!'`                      |
| `hex`         | Pre-serialized transactions                          | `'0x45a2b1c3d4...'`                       |
| `hex_payload` | Contract method payloads (requires contract address) | `{contract: '01a4...', payload: '0x...'}` |

### Contract Interactions

#### Basic Contract Interaction

```javascript
import { partisiaCrypto } from 'partisia-blockchain-applications-crypto'

// Prepare contract call payload
const abi = partisiaCrypto.abi_system.Payload_StakeTokens
const dataPayload = partisiaCrypto.structs.serializeToBuffer({
  invocationByte: 0,
  amount: '10000'
}, ...abi)

// Sign and broadcast transaction
const result = await sdk.signMessage({
  contract: '01a4082d9d560749ecd0ffa1dcaaaee2c2cb25d881',
  payload: dataPayload.toString('hex'),
  payloadType: 'hex_payload'
})

console.log(`Transaction Hash: ${result.trxHash}`)
```

#### Advanced Transaction Building

For more control over transaction parameters:

```javascript
import { partisiaCrypto } from 'partisia-blockchain-applications-crypto'
import { PartisiaRpc } from 'partisia-rpc'

// Create RPC client
const rpc = PartisiaRpc({
  baseURL: 'https://reader.partisiablockchain.com/shards/Shard0'
})

// Get current nonce
const nonce = await rpc.getNonce(sdk.connection.account.address)

// Prepare contract parameters
const contract = '01a4082d9d560749ecd0ffa1dcaaaee2c2cb25d881'
const payload = {
  invocationByte: 0,
  amount: '1000'
}

// Serialize payload
const abi = partisiaCrypto.abi_system.Payload_StakeTokens
const dataPayload = partisiaCrypto.structs.serializeToBuffer(payload, ...abi)

// Create full transaction with 2-minute validity
const serialized = partisiaCrypto.transaction.serializedTransaction(
  { nonce, validTo: Date.now() + 120 * 1000 },
  { contract },
  dataPayload
)

// Sign and broadcast
const result = await sdk.signMessage({
  payload: serialized.toString('hex'),
  payloadType: 'hex',
  dontBroadcast: false
})
```

### Requesting Private Key (Advanced)

In specific developer scenarios, you might need to request access to the private key:

```javascript
const keyData = await sdk.requestKey()
console.log(`Address: ${keyData.address}`)
console.log(`Public Key: ${keyData.public_key}`)
console.log(`Private Key: ${keyData.private_key}`)
```

\{% hint style="warning" %\} Requesting private keys should be done only in trusted environments and for specific development purposes. Production applications should never request or store user private keys. \{% endhint %\}

### Response Object

The response from signing operations contains these properties:

```javascript
{
  "signature": "304402...",
  "serializedTransaction": "0ab401...",
  "digest": "8773e068...",
  "trxHash": "11d09178b39c10520aec717200a4a5cd229e948bc15c4a87e65d682008f86db5",
  "isFinalOnChain": false
}
```

| Property                | Description                                               |
| ----------------------- | --------------------------------------------------------- |
| `signature`             | The cryptographic signature                               |
| `serializedTransaction` | The complete serialized transaction                       |
| `digest`                | The hash digest that was signed                           |
| `trxHash`               | The transaction hash (can be used to look up transaction) |
| `isFinalOnChain`        | Indicates if transaction has been finalized on-chain      |

### Error Handling

The SDK throws standardized errors that should be handled appropriately:

```javascript
try {
  const result = await sdk.signMessage({
    payload: 'Hello Partisia!',
    payloadType: 'utf8'
  })
  // Process successful result
} catch (error) {
  console.error('Signing error:', error.message)
  // Handle rejection, disconnection, or timeout
}
```

Common errors include:

* Connection not established
* User rejected request
* Invalid payload format
* Network errors
* Session timeout
