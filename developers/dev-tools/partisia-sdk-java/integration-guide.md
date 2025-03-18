# Integration Guide

## Partisia Blockchain Java SDK Development Guide

This comprehensive guide provides detailed implementation instructions for the Java SDK, covering installation, configuration, and common development patterns.

### Installation

#### Maven

Add the following to your `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>com.partisiablockchain</groupId>
        <artifactId>partisia-sdk</artifactId>
        <version>{latest-version}</version>
    </dependency>
</dependencies>
```

#### Gradle

Add the following to your `build.gradle`:

```groovy
dependencies {
    implementation 'com.partisiablockchain:partisia-sdk:{latest-version}'
}
```

### Client Initialization

Create a client instance connected to a Partisia Blockchain node:

```java
// For testnet
PartisiaClient client = PartisiaClient.builder()
    .setReaderNodeUrl("https://reader.testnet.partisiablockchain.com")
    .setWebNodeUrl("https://browser.testnet.partisiablockchain.com")
    .build();

// For mainnet
PartisiaClient mainnetClient = PartisiaClient.builder()
    .setReaderNodeUrl("https://reader.partisiablockchain.com")
    .setWebNodeUrl("https://browser.partisiablockchain.com")
    .build();
```

### Account Management

```java
// Generate a new account with random key pair
Account newAccount = Account.generate();

// Load an existing private key
byte[] privateKeyBytes = Files.readAllBytes(Paths.get("path/to/private.key"));
Account existingAccount = Account.fromPrivateKey(privateKeyBytes);

// Get account address
Address accountAddress = existingAccount.getAddress();
System.out.println("Account address: " + accountAddress.toString());

// Save private key
Files.write(Paths.get("new-private.key"), newAccount.getPrivateKeyBytes());
```

### Contract Deployment

```java
// Load compiled contract binary
byte[] contractBinary = Files.readAllBytes(Paths.get("path/to/contract.pbc"));

// Deploy with initialization parameters
CompletableFuture<DeployContractResult> deployFuture = client.deployContract(
    account,
    contractBinary,
    "Initialize", 1000, "Some parameter", address
);

// Wait for deployment to complete
DeployContractResult result = deployFuture.get(2, TimeUnit.MINUTES);
String contractAddress = result.getContractAddress();
System.out.println("Contract deployed at: " + contractAddress);
```

### Contract Interaction

```java
// Call contract action with parameters
CompletableFuture<TransactionResult> txFuture = client.callContractAction(
    account,
    contractAddress,
    "transfer",
    recipientAddress, 100
);

// Wait for transaction to be processed
TransactionResult txResult = txFuture.get();
System.out.println("Transaction hash: " + txResult.getTransactionHash());

// Check transaction success
if (txResult.isSuccessful()) {
    System.out.println("Transaction executed successfully");
} else {
    System.out.println("Transaction failed: " + txResult.getErrorMessage());
}
```

### State Querying

```java
// Get raw contract state
ContractState state = client.getContractState(contractAddress).get();

// Using generated bindings to decode state
TokenContractState tokenState = TokenContractState.fromBytes(state.getContractState());

// Access state properties
BigInteger totalSupply = tokenState.getTotalSupply();
Map<Address, BigInteger> balances = tokenState.getBalances();
BigInteger myBalance = balances.getOrDefault(account.getAddress(), BigInteger.ZERO);

System.out.println("Total supply: " + totalSupply);
System.out.println("My balance: " + myBalance);
```

### ABI Code Generation

Add the Maven plugin to your `pom.xml`:

```xml
<build>
    <plugins>
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
                                <path>${project.basedir}/src/main/resources/token.abi</path>
                                <packageName>com.example.contracts</packageName>
                            </abiFile>
                        </abiFiles>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

This generates Java classes for your contract, providing type-safe interaction:

```java
// Import generated classes
import com.example.contracts.TokenActions;
import com.example.contracts.TokenState;

// Create action payload
byte[] transferPayload = TokenActions.transfer(recipientAddress, 100);

// Call the action
client.callContractAction(
    account,
    contractAddress,
    transferPayload
).get();

// Decode state
ContractState rawState = client.getContractState(contractAddress).get();
TokenState state = TokenState.fromBytes(rawState.getContractState());
```

### Working with ZK Contracts

For Zero-Knowledge contracts, special handling is required:

```java
// List secret variables owned by this account
List<SecretDescription> secrets = client.listSecrets(account, zkContractAddress).get();

// Get a specific secret (automatically decrypted for the account)
Secret<Integer> secret = client.getSecret(
    account, 
    zkContractAddress, 
    secretId, 
    Integer.class
).get();

// Use the secret value
int secretValue = secret.getValue();
System.out.println("Secret value: " + secretValue);

// Submit a secret value
client.submitSecretInput(
    account,
    zkContractAddress,
    "submitBid",
    bidAmount  // This will be encrypted
).get();
```

### Error Handling

Implement robust error handling for blockchain operations:

```java
try {
    TransactionResult result = client.callContractAction(
        account,
        contractAddress,
        "transfer",
        recipientAddress, amount
    ).get();
    
    // Process successful result
    System.out.println("Transaction successful: " + result.getTransactionHash());
    
} catch (ExecutionException e) {
    if (e.getCause() instanceof InsufficientFundsException) {
        // Handle insufficient funds
        System.err.println("Not enough funds to complete transaction");
    } else if (e.getCause() instanceof ContractExecutionException) {
        // Handle contract-specific errors
        ContractExecutionException cee = (ContractExecutionException) e.getCause();
        System.err.println("Contract error: " + cee.getMessage());
    } else {
        // Handle other errors
        System.err.println("Transaction failed: " + e.getMessage());
    }
} catch (InterruptedException e) {
    // Handle interruption
    System.err.println("Operation was interrupted");
    Thread.currentThread().interrupt();
}
```

### Advanced SDK Features

#### Transaction Batching

Execute multiple actions in a single transaction:

```java
BatchBuilder batch = client.createBatchBuilder(account);

batch.addAction(
    tokenContract,
    "approve",
    spenderAddress, amount
);

batch.addAction(
    exchangeContract,
    "swap",
    tokenAddress, amount, minReturn
);

List<TransactionResult> results = batch.execute().get();
System.out.println("Batch executed with " + results.size() + " transactions");
```

#### Gas Management

Specify custom gas limits for complex operations:

```java
client.callContractAction(
    account,
    contractAddress,
    "complexAction",
    new GasParameters(500000), // Custom gas limit
    param1, param2
).get();
```

#### Event Subscription

Subscribe to contract events:

```java
EventSubscription subscription = client.subscribeToContractEvents(
    contractAddress,
    event -> {
        if ("Transfer".equals(event.getType())) {
            Map<String, Object> data = event.getData();
            System.out.printf("Transfer from %s to %s: %s%n",
                data.get("from"), data.get("to"), data.get("amount"));
        }
    }
).get();

// Unsubscribe when done
subscription.unsubscribe();
```

#### Working with File Storage

Store and retrieve files on-chain:

```java
byte[] fileContent = Files.readAllBytes(Paths.get("document.pdf"));
String cid = client.uploadFile(account, fileContent).get();

// Later retrieve the file
byte[] retrievedContent = client.downloadFile(cid).get();
Files.write(Paths.get("retrieved-document.pdf"), retrievedContent);
```

### Best Practices

#### Connection Management

* **Reuse Clients**: Create a single client instance and reuse it throughout your application
* **Connection Pooling**: Implement proper connection pooling for high-throughput applications
* **Timeout Handling**: Set appropriate timeouts for blockchain operations

```java
PartisiaClient client = PartisiaClient.builder()
    .setReaderNodeUrl("https://reader.testnet.partisiablockchain.com")
    .setWebNodeUrl("https://browser.testnet.partisiablockchain.com")
    .setTimeoutMillis(30000) // 30 seconds
    .setMaxRetries(3)
    .setBackoffMultiplier(1.5)
    .build();
```

#### Error Handling and Retries

Implement robust error handling with retry logic:

```java
public <T> CompletableFuture<T> executeWithRetry(
    Supplier<CompletableFuture<T>> operation,
    int maxRetries
) {
    CompletableFuture<T> resultFuture = new CompletableFuture<>();
    executeWithRetryInternal(operation, maxRetries, 0, resultFuture, null);
    return resultFuture;
}

private <T> void executeWithRetryInternal(
    Supplier<CompletableFuture<T>> operation,
    int maxRetries,
    int attempt,
    CompletableFuture<T> resultFuture,
    Throwable lastError
) {
    if (attempt >= maxRetries) {
        resultFuture.completeExceptionally(lastError != null
            ? lastError
            : new RuntimeException("Max retries exceeded"));
        return;
    }
    
    operation.get().whenComplete((result, error) -> {
        if (error == null) {
            resultFuture.complete(result);
        } else if (!(error instanceof NetworkException) || attempt == maxRetries - 1) {
            resultFuture.completeExceptionally(error);
        } else {
            long delay = (long) Math.pow(2, attempt) * 1000; // Exponential backoff
            System.out.printf("Attempt %d failed, retrying in %dms%n", attempt + 1, delay);
            
            scheduler.schedule(
                () -> executeWithRetryInternal(operation, maxRetries, attempt + 1, resultFuture, error),
                delay,
                TimeUnit.MILLISECONDS
            );
        }
    });
}

// Usage
CompletableFuture<TransactionResult> resultFuture = executeWithRetry(
    () -> client.callContractAction(
        account,
        contractAddress,
        "transfer",
        recipientAddress, amount
    ),
    3
);
```

#### Security Best Practices

* **Key Management**: Never hard-code private keys in your application
* **Secure Storage**: Use appropriate secure storage mechanisms
* **Environment Variables**: Use environment variables or secure vaults for sensitive information
* **Hardware Security**: Consider hardware security modules for production applications

```java
// Using system properties
String privateKeyHex = System.getProperty("app.privateKey");
byte[] privateKey = hexStringToByteArray(privateKeyHex);
Account account = Account.fromPrivateKey(privateKey);
```

#### Testing Strategies

* **Local Development Node**: Use a local development node for testing
* **Mock Clients**: Create mock blockchain clients for unit tests
* **Integration Tests**: Implement integration tests against testnet

```java
public class MockPartisiaClient {
    private final Map<String, byte[]> states = new HashMap<>();
    
    public CompletableFuture<TransactionResult> callContractAction(
        Account account,
        String contractAddress,
        String action,
        Object... parameters
    ) {
        // Simulate contract logic for testing
        if ("transfer".equals(action)) {
            Address recipient = (Address) parameters[0];
            BigInteger amount = (BigInteger) parameters[1];
            // Simulate transfer logic
            
            TransactionResult result = new TransactionResult(
                randomTransactionHash(),
                true,
                null
            );
            
            return CompletableFuture.completedFuture(result);
        }
        
        return CompletableFuture.failedFuture(
            new IllegalArgumentException("Unhandled action: " + action)
        );
    }
    
    // Additional mock methods
}
```

### Complete Example - Auction Bidding

```java
import com.partisiablockchain.sdk.PartisiaClient;
import com.partisiablockchain.sdk.Account;
import com.partisiablockchain.sdk.crypto.Secret;
import com.partisiablockchain.sdk.model.TransactionResult;
import com.partisiablockchain.sdk.model.ContractState;
import com.example.contracts.AuctionState;
import com.example.contracts.AuctionActions;

import java.nio.file.Files;
import java.nio.file.Paths;
import java.math.BigInteger;
import java.util.concurrent.CompletableFuture;

public class AuctionBidder {
    private final PartisiaClient client;
    private final Account account;
    private final String auctionContractAddress;
    
    public AuctionBidder(String privateKeyPath, String contractAddress) throws Exception {
        byte[] privateKey = Files.readAllBytes(Paths.get(privateKeyPath));
        this.account = Account.fromPrivateKey(privateKey);
        this.auctionContractAddress = contractAddress;
        
        this.client = PartisiaClient.builder()
            .setReaderNodeUrl("https://reader.testnet.partisiablockchain.com")
            .setWebNodeUrl("https://browser.testnet.partisiablockchain.com")
            .build();
    }
    
    public CompletableFuture<TransactionResult> placeBid(BigInteger bidAmount) {
        // For ZK auctions, submit as secret input
        return client.submitSecretInput(
            account,
            auctionContractAddress,
            "submitBid",
            bidAmount
        );
    }
    
    public CompletableFuture<AuctionState> getAuctionState() {
        return client.getContractState(auctionContractAddress)
            .thenApply(state -> AuctionState.fromBytes(state.getContractState()));
    }
    
    public CompletableFuture<TransactionResult> finalizeAuction() {
        return client.callContractAction(
            account,
            auctionContractAddress,
            "finalizeAuction"
        );
    }
    
    public static void main(String[] args) throws Exception {
        if (args.length != 3) {
            System.err.println("Usage: AuctionBidder <privateKeyPath> <contractAddress> <bidAmount>");
            System.exit(1);
        }
        
        String privateKeyPath = args[0];
        String contractAddress = args[1];
        BigInteger bidAmount = new BigInteger(args[2]);
        
        AuctionBidder bidder = new AuctionBidder(privateKeyPath, contractAddress);
        
        // Check if auction is still open
        AuctionState auctionState = bidder.getAuctionState().get();
        if (auctionState.isFinalized()) {
            System.err.println("Auction is already finalized");
            System.exit(1);
        }
        
        System.out.println("Placing bid of " + bidAmount + " on auction " + contractAddress);
        
        try {
            TransactionResult result = bidder.placeBid(bidAmount).get();
            System.out.println("Bid placed successfully: " + result.getTransactionHash());
            
            // Wait for a while and check updated state
            Thread.sleep(5000); // Wait 5 seconds
            
            AuctionState updatedState = bidder.getAuctionState().get();
            System.out.println("Current number of bids: " + updatedState.getBidCount());
            System.out.println("Auction end time: " + new java.util.Date(updatedState.getEndTime()));
            
            long timeRemaining = updatedState.getEndTime() - System.currentTimeMillis();
            if (timeRemaining > 0) {
                System.out.println("Time remaining: " + (timeRemaining / 1000) + " seconds");
            } else {
                System.out.println("Auction time has ended. Finalizing auction...");
                TransactionResult finalizeResult = bidder.finalizeAuction().get();
                System.out.println("Auction finalized: " + finalizeResult.getTransactionHash());
                
                // Check final results
                AuctionState finalState = bidder.getAuctionState().get();
                if (finalState.getWinner().equals(bidder.account.getAddress().toString())) {
                    System.out.println("You won the auction with a bid of " + finalState.getWinningBid());
                } else {
                    System.out.println("You did not win the auction. Winner: " + finalState.getWinner());
                    System.out.println("Winning bid: " + finalState.getWinningBid());
                }
            }
        } catch (Exception e) {
            System.err.println("Error during auction bidding: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### Token Transfer Example

```java
import com.partisiablockchain.sdk.PartisiaClient;
import com.partisiablockchain.sdk.Account;
import com.partisiablockchain.sdk.model.TransactionResult;
import com.partisiablockchain.sdk.model.ContractState;
import com.example.contracts.TokenState;
import com.example.contracts.TokenActions;

import java.nio.file.Files;
import java.nio.file.Paths;
import java.math.BigInteger;
import java.util.concurrent.CompletableFuture;

public class TokenTransfer {
    private final PartisiaClient client;
    private final Account account;
    private final String tokenContractAddress;
    
    public TokenTransfer(String privateKeyPath, String contractAddress) throws Exception {
        byte[] privateKey = Files.readAllBytes(Paths.get(privateKeyPath));
        this.account = Account.fromPrivateKey(privateKey);
        this.tokenContractAddress = contractAddress;
        
        this.client = PartisiaClient.builder()
            .setReaderNodeUrl("https://reader.testnet.partisiablockchain.com")
            .setWebNodeUrl("https://browser.testnet.partisiablockchain.com")
            .build();
    }
    
    public CompletableFuture<TokenState> getTokenState() {
        return client.getContractState(tokenContractAddress)
            .thenApply(state -> TokenState.fromBytes(state.getContractState()));
    }
    
    public CompletableFuture<TransactionResult> transferTokens(
        String recipientAddress, 
        BigInteger amount
    ) {
        // Create transfer payload
        byte[] transferPayload = TokenActions.transfer(recipientAddress, amount);
        
        // Execute transfer
        return client.callContractAction(
            account,
            tokenContractAddress,
            transferPayload
        );
    }
    
    public static void main(String[] args) throws Exception {
        if (args.length != 3) {
            System.err.println("Usage: TokenTransfer <privateKeyPath> <contractAddress> <recipient> <amount>");
            System.exit(1);
        }
        
        String privateKeyPath = args[0];
        String contractAddress = args[1];
        String recipient = args[2];
        BigInteger amount = new BigInteger(args[3]);
        
        TokenTransfer transfer = new TokenTransfer(privateKeyPath, contractAddress);
        
        // Check balance before transfer
        TokenState initialState = transfer.getTokenState().get();
        BigInteger initialBalance = initialState.getBalances()
            .getOrDefault(transfer.account.getAddress().toString(), BigInteger.ZERO);
        
        System.out.println("Initial balance: " + initialBalance);
        
        if (initialBalance.compareTo(amount) < 0) {
            System.err.println("Insufficient balance for transfer");
            System.exit(1);
        }
        
        // Execute transfer
        try {
            TransactionResult result = transfer.transferTokens(recipient, amount).get();
            System.out.println("Transfer successful: " + result.getTransactionHash());
            
            // Wait for a while and check updated state
            Thread.sleep(5000); // Wait 5 seconds
            
            TokenState updatedState = transfer.getTokenState().get();
            BigInteger updatedBalance = updatedState.getBalances()
                .getOrDefault(transfer.account.getAddress().toString(), BigInteger.ZERO);
            
            System.out.println("Updated balance: " + updatedBalance);
            System.out.println("Amount transferred: " + initialBalance.subtract(updatedBalance));
        } catch (Exception e) {
            System.err.println("Error during token transfer: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### Performance Optimization

#### Parallelizing Operations

For improved performance, execute operations in parallel:

```java
public CompletableFuture<List<ContractState>> loadMultipleContracts(List<String> addresses) {
    // Execute state queries in parallel
    List<CompletableFuture<ContractState>> futures = addresses.stream()
        .map(address -> client.getContractState(address))
        .collect(Collectors.toList());
    
    // Combine all futures into one
    return CompletableFuture.allOf(
        futures.toArray(new CompletableFuture[0])
    ).thenApply(v -> 
        futures.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList())
    );
}
```

#### Connection Pooling

For high-throughput applications, implement connection pooling:

```java
public class PartisiaClientPool {
    private final List<PartisiaClient> clients;
    private final AtomicInteger index = new AtomicInteger(0);
    
    public PartisiaClientPool(int size, String readerNodeUrl, String webNodeUrl) {
        clients = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            clients.add(PartisiaClient.builder()
                .setReaderNodeUrl(readerNodeUrl)
                .setWebNodeUrl(webNodeUrl)
                .build());
        }
    }
    
    public PartisiaClient getClient() {
        // Simple round-robin selection
        int current = index.getAndIncrement() % clients.size();
        return clients.get(current);
    }
}

// Usage
PartisiaClientPool clientPool = new PartisiaClientPool(5,
    "https://reader.testnet.partisiablockchain.com",
    "https://browser.testnet.partisiablockchain.com");

public void executeTransaction() {
    PartisiaClient client = clientPool.getClient();
    // Use client for transaction
}
```

### SDK Version Compatibility

When working with Partisia Blockchain Java SDK, maintain compatibility between:

1. The SDK version
2. The blockchain node version
3. The contract SDK version used in smart contracts

The following table outlines version compatibility:

| SDK Version | Compatible Node Version | Compatible Contract SDK Version |
| ----------- | ----------------------- | ------------------------------- |
| 0.9.x       | >= 0.16.0               | >= 0.13.0                       |
| 1.0.x       | >= 0.17.0               | >= 0.14.0                       |
| 1.1.x       | >= 0.18.0               | >= 0.14.5                       |
| 1.2.x       | >= 0.19.0               | >= 0.15.0                       |

#### SDK Version Verification

The SDK provides mechanisms to verify compatibility:

```java
import com.partisiablockchain.sdk.PartisiaClient;
import com.partisiablockchain.sdk.compatibility.VersionCompatibility;

boolean isCompatible = VersionCompatibility.checkNodeCompatibility(
    "https://reader.testnet.partisiablockchain.com"
).get();

if (!isCompatible) {
    System.out.println("Warning: SDK version may not be compatible with the node");
}
```

### Conclusion

This guide has covered comprehensive details for working with the Java SDK for Partisia Blockchain. By following the practices and patterns outlined here, developers can build robust, efficient, and secure Java applications on Partisia Blockchain.

For further information and updates, refer to the official documentation and repositories:

* [Java SDK Repository](https://gitlab.com/partisiablockchain/core/client)
* [Partisia Blockchain Explorer](https://browser.partisiablockchain.com)
* [Join our community](../../../welcome-to-partisia/resources.md)
