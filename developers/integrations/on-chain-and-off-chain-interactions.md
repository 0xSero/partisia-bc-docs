# On-chain & Off-chain Interactions

The blockchain is not an isolated environment; data can move onto the blockchain from outside clients, and can be extracted from the blockchain for use on traditional server architectures.

This article looks at have to read contract state, and how to write your own specialized client for blockchain interactions. It does not cover how to handle Zero-knowledge state and interactions, which require their own systems.

### State readers (On-chain to off-chain) <a href="#state-readers-on-chain-to-off-chain" id="state-readers-on-chain-to-off-chain"></a>

Blockchains are useful storage mediums for tracking states, but one can often need to use the data stored on-chain for certain tasks off chain.

State reading requires the following steps:

1. Determine contract and type of contract the client should interact with.
2. Download contract ABI from a reader node, which will act as a description of contract state.
3. Download contract state from a reader node, and deserialize it using the ABI.

Depending upon the flexibility of your client, you can do points (1) and (2) manually.

### Clients / Wallets (Off-chain to on-chain) <a href="#clients-wallets-off-chain-to-on-chain" id="clients-wallets-off-chain-to-on-chain"></a>

Most blockchain interactions does not originate on the blockchain, but are rather triggered by outside actions. Clients are the triggers of these transactions, and can either happen directly due to user actions, such as wallets, or automatically due to secondary systems.

Client interaction involves the following steps:

1. Determine the contract and type of contract the client should interact with.
2. Download the contract ABI, which will act as a description of possible interactions and how to read state.
3. Prepare the transaction, and serialize it for the contract using the contract ABI.
4. Sign the transaction, which certifies that the transaction have come from you.
5. Send the signed transaction to a node, which then forwards the transaction to the blockchain itself.

Depending upon the flexibility of your client, you can do points (1) and (2) manually. Point (3) can also be done manually, but it is recommended that you use an ABI client. Point (4) involves advanced cryptology, so a library is needed. Point (5) can be done with most http(s) capable web clients/libraries.

### Libraries & Examples <a href="#libraries-examples" id="libraries-examples"></a>

Libraries:

* [abi-client (TypeScript)](https://gitlab.com/partisiablockchain/language/abi/abi-client-ts): TypeScript ABI client, allowing automatic reading of contract state, and reading/writing invocations.
* [abi-client (Java)](https://gitlab.com/partisiablockchain/language/abi/abi-client): Java ABI client, allowing automatic reading of contract state, and reading/writing invocations.
* [abigen Maven Plugin](https://gitlab.com/partisiablockchain/language/abi/abigen-maven-plugin): Advanced ABI client for Java, which creates data structures and code for reading state, and reading/writing invocations. Better experience when ABI is known beforehand, as it enforces compile-time typing.
* [zk-client](https://gitlab.com/partisiablockchain/language/abi/zk-client): Advanced ABI client for Java with support for Zero-knowledge inputs.

### Examples:

* [partisia-cli](https://gitlab.com/partisiablockchain/language/partisia-cli): General command-line tool for interacting with PBC contracts. Can perform all interaction steps mentioned above. Meant for manual usage, but nothing prevents usage from other scripts. The best example of how to interact with the blockchain.
