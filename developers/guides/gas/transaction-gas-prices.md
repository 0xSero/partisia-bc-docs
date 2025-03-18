# Transaction Gas Prices

Here, we will explore the factors that determine the gas cost for deploying or interaction with a contract in the Partisia blockchain. By considering the network, CPU, and storage aspects, developers can effectively estimate and manage the gas costs associated with their smart contract transactions to understand the specifics of gas pricing.

{% hint style="info" %}
<mark style="color:blue;">**BASIC FEES**</mark>\
\
**Network fee:** 5 USD cents/kb.

**CPU WASM fee:** 1 USD cents per 1,000,000 instructions.

**CPU AVL fee:** 1 USD cents per 10 instructions.
{% endhint %}

When a transaction is sent, the transaction needs to have enough gas to cover all resource usage. Otherwise the transaction will fail and the gas will go to waste. When developing a smart contract the developer should keep in mind that the best practice is to have your front-end suggest the needed gas amount for the users of your smart contract. We recommend you to test your smart contract to find the maximum gas cost of your transactions and use your results as the gas price in your front-end.

### The cost for using the blockchain <a href="#the-cost-for-using-the-blockchain" id="the-cost-for-using-the-blockchain"></a>

Gas on Partisia Blockchain is fixed, you pay for what you use. Gas units are pegged to the USD. The fixed ratio is 100,000 gas units to 1 USD.

### CPU Cost in Gas Pricing: Understanding the Three CPU Stages to optimization <a href="#cpu-cost-in-gas-pricing-understanding-the-three-cpu-stages-to-optimization" id="cpu-cost-in-gas-pricing-understanding-the-three-cpu-stages-to-optimization"></a>

The CPU cost reflects the computational resources consumed during the execution of a smart contract transaction. It consists of three primary stages: deserializing the contract state, performing the required work or computations, and serializing the updated contract state.

**1. Deserialize State:** Before executing a smart contract, the contract's current state needs to be deserialized. This process involves converting the serialized contract state into a data structure that can be manipulated and processed by the blockchain's virtual machine. The gas cost associated with this step depends on the size and complexity of the contract state or transaction.

**2. Perform Work:** Once the contract state is deserialized, the required computations or work specified by the smart contract are performed. This work may involve various operations such as calculations, data manipulation, and interactions with other contracts or external systems. The gas cost for this stage depends on the complexity and intensity of the computations involved.

**3. Serialize State:** After the work is completed, the contract state is serialized back into a format suitable for storage on the blockchain. The gas cost associated with serialization depends on the size and complexity of the updated contract state.

It's a good practice to consider your gas consumption during the architecture and design of smart contracts. By implementing these strategies, optimizing gas usage and improving the efficiency of smart contracts can be achieved. You can find more best practice on our article of [efficient gas practices](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/efficient-gas-practices.html).

### Network cost <a href="#network-cost" id="network-cost"></a>

The network cost is associated with the transmission of the transaction across the blockchain network. It takes into account the bandwidth and other networking resources utilized during the transaction's propagation.

By considering the above gas pricing properties, Partisia blockchain empowers developers to estimate and manage the gas costs associated with smart contract transactions. You should visit our article about [efficient gas practices](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/efficient-gas-practices.html) to understand how to optimize contracts gas usage.
