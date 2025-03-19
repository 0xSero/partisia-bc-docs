# Partisia SDK - Java

## Partisia SDK - Java

### Introduction

The Partisia Java SDK provides a comprehensive toolkit for Java developers to interact with the Partisia Blockchain ecosystem. Designed for enterprise applications, Android development, and server-side implementations, this SDK offers a type-safe and intuitive developer experience for building blockchain-powered applications.

\{% hint style="info" %\} **When to use the Java SDK**

Choose the Java SDK when building enterprise applications, backend services, Android apps, or any Java-based system that needs to interact with Partisia Blockchain. \{% endhint %\}

### Key Features

The Java SDK provides a rich set of features to streamline blockchain development:

#### Account Management

* Generate and import key pairs securely
* Derive blockchain addresses from public keys
* Sign transactions with proper cryptographic security
* Manage account nonces automatically

#### Contract Interaction

* Deploy smart contracts with initialization parameters
* Call contract actions with automatic parameter serialization
* Query contract state with typed deserialization
* Monitor events and state changes
* Support for both public and ZK (zero-knowledge) contracts

#### Transaction Handling

* Create, serialize, and sign transactions
* Calculate gas costs automatically
* Handle transaction monitoring and receipts
* Create transaction batches for complex operations
* Support for callback patterns

#### Developer Tools

* ABI code generation for type-safe contract interactions
* Comprehensive error handling with specific error types
* Automatic serialization/deserialization of complex data structures
* Connection pooling for high-throughput applications

### Getting Started

#### Installation

Add the Partisia Java SDK to your project:

**Maven**

```xml
<dependency>
    <groupId>com.partisiablockchain</groupId>
    <artifactId>partisia-sdk</artifactId>
    <version>{latest-version}</version>
</dependency>
```

**Gradle**

```groovy
implementation 'com.partisiablockchain:partisia-sdk:{latest-version}'
```

#### Quick Start Example

Here's a simple example showing how to connect to Partisia Blockchain and interact with a contract:

```java
import com.partisiablockchain.sdk.PartisiaClient;
import com.partisiablockchain.sdk.Account;
import com.partisiablockchain.sdk.crypto.KeyPair;
import com.partisiablockchain.sdk.model.TransactionResult;

// Initialize the client
PartisiaClient client = PartisiaClient.builder()
    .setReaderNodeUrl("https://reader.testnet.partisiablockchain.com")
    .setWebNodeUrl("https://browser.testnet.partisiablockchain.com")
    .build();

// Create or load an account
KeyPair keyPair = KeyPair.fromPrivateKey(privateKeyBytes);
Account account = new Account(keyPair);

// Call a contract action
TransactionResult result = client.callContractAction(
    account,
    "01a4082d9d560749ecd0ffa1dcaaaee2c2cb25d881", // Contract address
    "increment"                                    // Action name
).get();

System.out.println("Transaction hash: " + result.getTransactionHash());
```

### Core Components

#### PartisiaClient

The central component for all blockchain interactions. Use it to:

* Query blockchain data
* Send transactions
* Deploy contracts
* Subscribe to events

#### Account

Represents a blockchain account with:

* A key pair for cryptographic operations
* Methods for signing transactions
* Address derivation functions

#### TransactionBuilder

Helps construct complex transactions with:

* Multiple actions in a single transaction
* Custom gas parameters
* Expiration timeouts

#### ContractState

Represents the state of a smart contract, supporting:

* Typed deserialization
* State queries
* Historical state access

### Common Workflows

#### Deploying a Smart Contract

```java
// Load contract binary
byte[] contractBinary = Files.readAllBytes(Paths.get("contract.wasm"));
byte[] contractAbi = Files.readAllBytes(Paths.get("contract.abi"));

// Deploy contract
DeployResult deployResult = client.deployContract(
    account,
    contractBinary,
    contractAbi,
    "initialize", 1000, "Example Contract" // Init params
).get();

System.out.println("Contract deployed at: " + deployResult.getContractAddress());
```

#### Interacting with a Contract

```java
// Call contract with generated ABI classes
byte[] transferAction = TokenContract.transfer(
    receiverAddress,
    BigInteger.valueOf(1000)
);

TransactionResult result = client.callContractAction(
    account,
    contractAddress,
    transferAction
).get();

// Check transaction result
if (result.isSuccessful()) {
    System.out.println("Transfer successful: " + result.getTransactionHash());
} else {
    System.out.println("Transfer failed: " + result.getError());
}
```

#### Reading Contract State

```java
// Get and deserialize contract state
ContractStateResponse stateResponse = client.getContractState(contractAddress).get();
TokenContractState state = TokenContractState.fromBytes(stateResponse.getStateBytes());

// Access typed state fields
System.out.println("Total supply: " + state.getTotalSupply());
System.out.println("Your balance: " + state.getBalances().get(account.getAddress()));
```

#### Working with Zero-Knowledge Contracts

```java
// Submit a secret value to a ZK contract
SecretInputResult secretResult = client.submitSecretInput(
    account,
    zkContractAddress,
    "submitBid",
    1000 // This value will be encrypted
).get();

// Retrieve your own secret values
SecretVariable secret = client.getSecret(
    account,
    zkContractAddress,
    secretId
).get();

System.out.println("Secret value: " + secret.getValue());
```

### Error Handling

The SDK uses a comprehensive exception hierarchy for effective error handling:

```java
try {
    TransactionResult result = client.callContractAction(
        account, contractAddress, "transfer", receiverAddress, amount
    ).get();
    
    // Handle success
    System.out.println("Transaction successful: " + result.getTransactionHash());
    
} catch (ExecutionException e) {
    if (e.getCause() instanceof InsufficientGasException) {
        // Handle insufficient gas
        System.err.println("Not enough gas for transaction");
    } else if (e.getCause() instanceof ContractExecutionException) {
        // Handle contract errors
        System.err.println("Contract error: " + e.getCause().getMessage());
    } else {
        // Handle other errors
        System.err.println("Transaction failed: " + e.getMessage());
    }
}
```

### Best Practices

#### Connection Management

* Reuse client instances rather than creating new ones for each operation
* Implement proper connection pooling for high-throughput applications
* Set appropriate timeouts to handle network issues

#### Transaction Batching

* Group related operations into transaction batches when possible
* Use callback patterns for complex workflows
* Implement proper error handling for each transaction in a batch

#### State Management

* Cache contract states when appropriate to reduce network calls
* Implement proper state synchronization in multi-threaded environments
* Use the event subscription mechanism for real-time updates

#### Security Considerations

* Never hard-code private keys in your application
* Use environment variables or secure vaults for sensitive information
* Implement proper key management with hardware security when possible
* Validate all inputs before sending them to the blockchain

### Advanced Topics

#### ABI Code Generation

Generate type-safe Java classes from contract ABIs:

```xml
<!-- In your pom.xml -->
<plugin>
    <groupId>com.partisiablockchain</groupId>
    <artifactId>abi-codegen-maven-plugin</artifactId>
    <version>{latest-version}</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <abiFiles>
                    <abiFile>
                        <path>${project.basedir}/src/main/resources/contract.abi</path>
                        <packageName>com.example.contracts</packageName>
                    </abiFile>
                </abiFiles>
            </configuration>
        </execution>
    </executions>
</plugin>
```

#### Asynchronous Programming

The SDK is designed for asynchronous operations:

```java
// Chain operations together
client.getContractState(contractAddress)
    .thenApply(state -> TokenState.fromBytes(state.getStateBytes()))
    .thenCompose(tokenState -> {
        if (tokenState.getBalance(account.getAddress()).compareTo(amount) >= 0) {
            return client.callContractAction(
                account, contractAddress, "transfer", receiverAddress, amount
            );
        } else {
            throw new InsufficientBalanceException("Not enough tokens");
        }
    })
    .thenAccept(result -> System.out.println("Transaction hash: " + result.getTransactionHash()))
    .exceptionally(e -> {
        System.err.println("Error: " + e.getMessage());
        return null;
    });
```

#### Testing with the SDK

Implement effective tests for your blockchain applications:

```java
// Sample JUnit test
@Test
public void testTransfer() {
    // Use testnet for integration tests
    PartisiaClient testClient = PartisiaClient.builder()
        .setReaderNodeUrl("https://reader.testnet.partisiablockchain.com")
        .build();
    
    // Create test accounts
    Account sender = Account.fromPrivateKey(senderKey);
    Account receiver = Account.fromPrivateKey(receiverKey);
    
    // Get initial balances
    TokenState initialState = getTokenState(testClient, contractAddress);
    BigInteger initialSenderBalance = initialState.getBalance(sender.getAddress());
    BigInteger initialReceiverBalance = initialState.getBalance(receiver.getAddress());
    
    // Execute transfer
    BigInteger transferAmount = BigInteger.valueOf(100);
    try {
        testClient.callContractAction(
            sender, contractAddress, "transfer", receiver.getAddress(), transferAmount
        ).get(30, TimeUnit.SECONDS);
        
        // Wait for transaction to be processed
        Thread.sleep(5000);
        
        // Verify balances changed correctly
        TokenState finalState = getTokenState(testClient, contractAddress);
        assertEquals(
            initialSenderBalance.subtract(transferAmount),
            finalState.getBalance(sender.getAddress())
        );
        assertEquals(
            initialReceiverBalance.add(transferAmount),
            finalState.getBalance(receiver.getAddress())
        );
    } catch (Exception e) {
        fail("Transfer failed: " + e.getMessage());
    }
}
```

### Resources

* [Java SDK Repository](https://gitlab.com/partisiablockchain/core/client)
* [Sample Applications](https://gitlab.com/partisiablockchain/language/example-client)
* Integration Guide for detailed implementation instructions
* [JavaDocs](https://javadoc.io/doc/com.partisiablockchain/partisia-sdk)
* Developer Community for support and collaboration

### Next Steps

* Follow the detailed Integration Guide for more examples
* Explore ZK Smart Contracts to leverage privacy features
* Check out the CLI tools for contract development
* Try the quick start guide for a complete workflow
