# API Reference

This section provides detailed documentation of the key classes and methods in the `@partisiablockchain/zk-client` library.

### BinarySecretShares

The `BinarySecretShares` class handles creation, encryption, blinding, and reconstruction of secret shares.

#### Constructor

```typescript
constructor(secretShares: Share[])
```

Constructs secret shares from a list of shares, with one share for each sending/receiving party.

#### Static Methods

**`create`**

```typescript
static create(variableData: CompactBitArray, rng?: (ln: number) => Buffer): BinarySecretShares
```

Creates binary secret shares from binary secret data:

* `variableData`: The binary secret variable data
* `rng`: (Optional) Randomness generator for polynomial creation, defaults to `randomBytes`

**`read`**

```typescript
static read(rawShares: Buffer[]): BinarySecretShares
```

Creates binary secret shares from bytes read from ZK nodes.

#### Instance Methods

**`encrypt`**

```typescript
encrypt(ephemeralKey: Elliptic.KeyPair, sender: BlockchainAddress, engineKeys: BlockchainPublicKey[]): EncryptedShares
```

Encrypts the secret shares for sending on-chain:

* `ephemeralKey`: Temporary key used for encryption
* `sender`: Address of sender of the secret input
* `engineKeys`: Public keys of the ZK engines that will receive the shares

**`applyBlinding`**

```typescript
applyBlinding(rng: (ln: number) => Buffer): BlindedShares
```

Applies blinding to shares for off-chain input:

* `rng`: Randomness generator for creating blindings

**`reconstructSecret`**

```typescript
reconstructSecret(): CompactBitArray
```

Reconstructs the original secret variable data from the shares.

**`noOfShares`**

```typescript
noOfShares(): number
```

Returns the number of shares.

### ZkRpcBuilder

The `ZkRpcBuilder` provides utility functions for creating RPCs for ZK inputs.

#### Static Methods

**`zkInputOnChain`**

```typescript
static zkInputOnChain(
  sender: BlockchainAddress,
  secretInput: CompactBitArray,
  additionalRpc: Buffer,
  engineKeys: BlockchainPublicKey[],
  encryptionKey?: Elliptic.KeyPair,
  rng?: (ln: number) => Buffer
): Buffer
```

Builds the RPC for inputting secret binary data on-chain to a ZK contract:

* `sender`: The sender of the input
* `secretInput`: The binary input data
* `additionalRpc`: Additional RPC to send (must include ZK input shortname)
* `engineKeys`: Public keys of the ZK engines
* `encryptionKey`: (Optional) Key pair for encryption, generated if not provided
* `rng`: (Optional) Randomness generator, defaults to `randomBytes`

**`zkInputOffChain`**

```typescript
static zkInputOffChain(
  secretInput: CompactBitArray,
  additionalRpc: Buffer,
  rng?: (ln: number) => Buffer
): OffChainInput
```

Builds the RPC for inputting secret binary data off-chain to a ZK contract:

* `secretInput`: The binary input data
* `additionalRpc`: Additional RPC to send (must include ZK input shortname)
* `rng`: (Optional) Randomness generator, defaults to `randomBytes`

Returns an `OffChainInput` object with:

* `blindedShares`: The blinded shares to send to ZK nodes
* `rpc`: The RPC to include in the blockchain transaction

### RealClient

The `RealClient` class facilitates interaction with ZK nodes for retrieving shares and sending blinded shares.

#### Static Methods

**`create`**

```typescript
static create(zkContract: ZkContract): RealClient
```

Creates a `RealClient` based on engines found in a ZK contract.

#### Instance Methods

**`getSharesFromEngines`**

```typescript
getSharesFromEngines(
  contractAddress: BlockchainAddress,
  privateKey: string,
  variableId: number
): Promise<BinarySecretShares>
```

Gets the shares of a secret contract variable:

* `contractAddress`: The contract address containing the secret variable
* `privateKey`: The owner of the secret variable
* `variableId`: The ID of the variable

**`sendSharesToNodes`**

```typescript
sendSharesToNodes(
  contract: BlockchainAddress,
  account: BlockchainAddress,
  txIdentifier: Buffer,
  blindedShares: BlindedShares
): Promise<boolean>
```

Sends secret shares to ZK nodes off-chain:

* `contract`: The contract address
* `account`: The owner of the secret variable
* `txIdentifier`: The transaction hash containing the commitments
* `blindedShares`: The blinded secret shares

### Client

The `Client` class provides methods for interacting with contracts and accounts on the Partisia Blockchain.

#### Constructor

```typescript
constructor(host: string)
```

Creates a new client for the given host URL.

#### Instance Methods

**`getContractState`**

```typescript
getContractState(address: BlockchainAddress): Promise<ZkContract | undefined>
```

Retrieves a contract state from the blockchain.

**`getAccountState`**

```typescript
getAccountState(address: BlockchainAddress): Promise<AccountState | undefined>
```

Retrieves an account state from the blockchain.

**`getChainId`**

```typescript
getChainId(): Promise<ChainIdData | undefined>
```

Gets the chain ID for the blockchain.

**`putTransaction`**

```typescript
putTransaction(
  fromAddress: BlockchainAddress,
  signedTransaction: SignedTransaction
): Promise<TransactionPointer>
```

Sends a signed transaction to the blockchain.

### TransactionSender

The `TransactionSender` class helps with signing and sending transactions.

#### Static Methods

**`create`**

```typescript
static create(
  client: Client,
  privateKey: string,
  transactionValidityDuration: BN = new BN(180000)
): TransactionSender
```

Creates a new transaction sender:

* `client`: The blockchain client
* `privateKey`: The sender's private key
* `transactionValidityDuration`: How long the transaction is valid (milliseconds)

#### Instance Methods

**`sign`**

```typescript
sign(transaction: Transaction, gasCost: BN): Promise<SignedTransaction | undefined>
```

Signs a transaction for sending to the blockchain.

**`send`**

```typescript
send(signedTransaction: SignedTransaction): Promise<SentTransaction>
```

Sends a signed transaction to the blockchain.

**`sendAndSign`**

```typescript
sendAndSign(transaction: Transaction, gasCost: BN): Promise<SentTransaction>
```

Signs and sends a transaction in one operation.

**`getAddress`**

```typescript
getAddress(): BlockchainAddress
```

Returns the blockchain address of the sender.

### BlockchainAddress

Represents an address on the Partisia Blockchain.

#### Static Methods

**`fromString`**

```typescript
static fromString(val: string): BlockchainAddress
```

Creates an address from a hex string.

**`fromBuffer`**

```typescript
static fromBuffer(val: Buffer): BlockchainAddress
```

Creates an address from bytes.

#### Instance Methods

**`asString`**

```typescript
asString(): string
```

Returns the address as a hex string.

**`asBuffer`**

```typescript
asBuffer(): Buffer
```

Returns the address as bytes.

### BlockchainPublicKey

Represents a public key on the Partisia Blockchain.

#### Static Methods

**`fromString`**

```typescript
static fromString(val: string): BlockchainPublicKey
```

Creates a public key from a hex string.

**`fromBuffer`**

```typescript
static fromBuffer(val: Buffer): BlockchainPublicKey
```

Creates a public key from bytes.

#### Instance Methods

**`asString`**

```typescript
asString(): string
```

Returns the public key as a hex string.

**`asBuffer`**

```typescript
asBuffer(): Buffer
```

Returns the public key as bytes.

### CryptoUtils

Utility functions for cryptographic operations.

#### Static Methods

**`generateKeyPair`**

```typescript
static generateKeyPair(): Elliptic.KeyPair
```

Generates a new EC key pair.

**`createSharedKey`**

```typescript
static createSharedKey(keyPair: Elliptic.KeyPair, publicKey: Buffer): Buffer
```

Creates a shared secret from a private and a public key.

**`createAesForParty`**

```typescript
static createAesForParty(keyPair: Elliptic.KeyPair, publicKey: Buffer): Cipher
```

Creates an AES cipher for a party using ECDH key exchange.

**`hashBuffer`**

```typescript
static hashBuffer(buffer: Buffer): Buffer
```

Hashes a buffer using SHA-256.

**`hashBuffers`**

```typescript
static hashBuffers(buffers: Buffer[]): Buffer
```

Hashes multiple buffers using SHA-256.

**`keyPairToAccountAddress`**

```typescript
static keyPairToAccountAddress(keyPair: Elliptic.KeyPair): string
```

Computes the account address based on a key pair.

**`privateKeyToAccountAddress`**

```typescript
static privateKeyToAccountAddress(privateKey: string): string
```

Computes the account address based on a private key.

### Utility Types

#### `Transaction`

```typescript
interface Transaction {
  address: BlockchainAddress;
  rpc: Buffer;
}
```

#### `SignedTransaction`

Represents a transaction that has been signed and is ready to be sent.

#### `SentTransaction`

```typescript
interface SentTransaction {
  signedTransaction: SignedTransaction;
  transactionPointer: TransactionPointer;
}
```

#### `TransactionPointer`

```typescript
interface TransactionPointer {
  identifier: Buffer;
  destinationShard: string;
}
```

#### `ZkContract`

```typescript
type ZkContract = { 
  serializedContract: { 
    engines: { 
      engines: Engine[] 
    } 
  } 
};
```

#### `Engine`

```typescript
interface Engine {
  identity: string;
  publicKey: string;
  restInterface: string;
}
```

#### `OffChainInput`

```typescript
interface OffChainInput {
  blindedShares: BlindedShares;
  rpc: Buffer;
}
```

#### `BlindedShares`

A class containing blinded shares of binary data for off-chain input.

#### `EncryptedShares`

A class containing encrypted shares of binary data for on-chain input
