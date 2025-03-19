# Gas

## Understanding Gas on Partisia Blockchain

### What is Gas?

Gas is the unit that measures computational effort on Partisia Blockchain. Every operation—from simple transactions to complex smart contract executions—requires a specific amount of gas, which represents the computational resources consumed by the network to process your request.

{% hint style="info" %}
**Key Difference:** Unlike many other blockchains, Partisia's gas prices are pegged to USD value rather than token value, making transaction costs more predictable regardless of token market fluctuations.&#x20;
{% endhint %}

### Gas Components

Gas costs on Partisia Blockchain are determined by three main factors:

#### 1. Network Fees

**Network Components:**

| Component            | Cost               | Description                                              |
| -------------------- | ------------------ | -------------------------------------------------------- |
| Network Transmission | **5 USD cents/KB** | Cost for transmitting data across the blockchain network |

Network fees cover the bandwidth and resources required to propagate your transaction data across the blockchain. Larger transactions with more data incur higher network fees.

**Formula:** `NETWORK_FEE = NETWORK_BYTES * 5,000 / 1,000`

#### 2. CPU Fees

**CPU Components:**

| Component           | Cost                                      | Description                                                         |
| ------------------- | ----------------------------------------- | ------------------------------------------------------------------- |
| WASM Execution      | **1 USD cent per 1,000,000 instructions** | Cost for executing WebAssembly code in smart contracts              |
| AVL Tree Operations | **1 USD cent per 10 instructions**        | Cost for operations on AVL trees (used for efficient state storage) |

CPU fees account for the computational resources required to execute your transaction. These are typically the most significant portion of gas costs for complex contract operations.

**Formulas:**

* `WASM_EXECUTION_FEE = NO_OF_INSTRUCTIONS / 1,000,000`
* `AVL_TREE_FEE = NO_OF_AVL_INSTRUCTIONS / 10`

#### 3. Storage Fees

**Storage Components:**

| Component     | Cost                       | Description                                 |
| ------------- | -------------------------- | ------------------------------------------- |
| State Storage | **1 USD cent/KB per year** | Cost for maintaining data on the blockchain |

Storage fees are unique as they represent an ongoing cost rather than a one-time fee. Contracts must maintain a positive gas balance to cover these recurring storage costs.

**Important:** If a contract's gas balance becomes negative, the contract will be deleted from the blockchain.

### Gas Pricing Model

Partisia Blockchain uses a fixed gas pricing model:

* **Gas Unit Conversion**: 100,000 gas units = 1 USD
* **Deterministic Costs**: Gas costs are predictable and consistent
* **USD Pegging**: Costs remain stable regardless of token price fluctuations

### Gas Management

#### Transaction Gas

When sending a transaction, you must specify enough gas to cover all operations:

```java
// Java SDK example with gas specification
client.callContractAction(
    account,
    contractAddress,
    "complexAction",
    new GasParameters(500000), // Specify 500,000 gas units
    param1, param2
).get();
```

```typescript
// TypeScript SDK example
sdk.signMessage({
    contract: contractAddress,
    payload: transactionPayload,
    gas: 500000, // Specify 500,000 gas units
});
```

#### Contract Gas Balance

Smart contracts maintain their own gas balance to cover ongoing storage costs:

* **Initial Gas**: When deploying a contract, excess gas from the deployment transaction goes to the contract's balance
* **Refueling**: You can add more gas to a contract using the `refuelgas` operation
* **Contract Balance Check**: Monitor your contract's gas balance to prevent deletion

```bash
# CLI example to add gas to a contract
cargo pbc contract refuelgas --pk=./mykey.pk <CONTRACT_ADDRESS> 100000
```

#### ZK Contract Gas

Zero-knowledge contracts have additional gas considerations:

* **ZK Computation Gas**: Gas for secure multiparty computation
* **Secret Input Gas**: Gas for processing encrypted inputs
* **Attestation Gas**: Gas for verifying and attesting computation results

For detailed information on ZK gas costs, refer to the ZK Computation Gas Fees guide.

### Gas Optimization Strategies

#### 1. Optimize State Structure

* Use fixed-size elements for efficient serialization/deserialization
* Structure data to minimize deserialization effort
* Consider using `AvlTreeMap` for large collections

```rust
// Example of optimized struct
#[repr(C)]
pub struct OptimizedTransfer {
    to: Address,            // offset 0, size 21
    from: Address,          // offset 21, size 21
    padding: [u8; 6],       // offset 42, size 6
    amount: u128,           // offset 48, size 16
    timestamp: i64,         // offset 64, size 8
    // Total size: 72 bytes
}
```

#### 2. Minimize Contract State

* Store only essential data on-chain
* Consider using events for historical data
* Use proper data structures to minimize storage costs

#### 3. Batch Operations

* Group related operations to reduce network fees
* Use transaction batching for multiple actions
* Consider contract-to-contract calls for complex workflows

#### 4. Test and Estimate

* Always test transactions on testnet first
* Measure actual gas consumption for operations
* Build in margin for gas estimation in production applications

### Gas Estimation in Practice

#### Transaction Gas Estimation

To estimate the gas needed for a transaction:

1. Test the transaction on testnet
2. Note the actual gas consumed
3. Add a safety margin (typically 20-30%)
4. Use this estimate in your production application

#### Contract-to-Contract Gas Estimation

When a contract calls another contract, gas must cover the entire chain of operations:

```
Total Gas = (Tx → Contract A) + (Tx → Contract B) + (Any callback operations)
```

For detailed information on contract-to-contract gas estimation, see the Contract-to-Contract Estimates guide.

### Troubleshooting Gas Issues

#### Common Problems and Solutions

**Common Problems and Solutions:**

| Problem                             | Possible Cause                             | Solution                                       |
| ----------------------------------- | ------------------------------------------ | ---------------------------------------------- |
| Transaction fails with "out of gas" | Insufficient gas provided for transaction  | Increase gas limit, optimize contract code     |
| Contract deleted unexpectedly       | Contract ran out of gas for storage costs  | Monitor contract gas balance, refuel regularly |
| High transaction costs              | Inefficient contract design, large state   | Optimize state structure, minimize storage     |
| Inconsistent gas costs              | Variable input data or contract state size | Test with maximum expected inputs, add margin  |

### Getting Testnet Gas

For development and testing, you can get free gas on the testnet:

1.  **Create an Account with Gas**:

    ```bash
    cargo pbc account create
    ```
2.  **Add Gas to an Existing Account**:

    ```bash
    cargo pbc account mintgas <your-account-address>
    ```
3. **Use the Feed Me Contract**: Visit the [Feed Me contract interaction](https://browser.testnet.partisiablockchain.com/contracts/02c14c29b2697f3c983ada0ee7fac83f8a937e2ecd/feed_me) page on the testnet browser.

For detailed instructions, see the How to Get Testnet Gas guide.

### Further Reading

* Efficient Gas Practices - In-depth optimization techniques
* ZK Computation Gas Fees - Special considerations for ZK contracts
* Contract-to-Contract Estimates - Gas for complex interaction patterns
* Transaction Gas Prices - Detailed breakdown of transaction gas pricing
* Storage Gas Prices - Understanding ongoing storage costs

***

{% hint style="info" %}
**Best Practice:** Always [test your contracts on the testnet](../smart-contracts/compile-and-deploy-contracts.md) first to identify potential gas issues and estimate accurate gas costs for production deployments.
{% endhint %}
