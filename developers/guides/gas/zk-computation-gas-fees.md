# ZK Computation Gas Fees

Zero-knowledge (ZK) computation involves a number of gas fees to ensure that a contract's associated ZK nodes have enough gas to execute the ZK actions. You can find the overview on [our gas price table here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/gas-price-table-overview.html). In this article we will dive into the specifics of gas fees when doing ZK actions on Partisia Blockchain.

## ZK computation, MPC tokens and gas <a href="#zk-computation-mpc-tokens-and-gas" id="zk-computation-mpc-tokens-and-gas"></a>

A deployed ZK contract has a number of ZK computation nodes associated with it.

These nodes each lock an amount of their MPC tokens as collateral for the computation. If any malicious activity by a ZK node is detected the collateral can be taken to punish the malicious ZK node. A user deploying a ZK contract decides the amount of MPC tokens that must be locked as collateral. In the following, these tokens are called locked stakes.

### Network fees <a href="#network-fees" id="network-fees"></a>

{% hint style="info" %}
**Network gas price**

NETWORK\_FEE = NETWORK\_BYTES \* 5,000 / 1,000
{% endhint %}

When sending transactions to a ZK contract, a network fee is paid by the calling user in the same way as for regular transactions in public contracts.

### WASM execution fees <a href="#wasm-execution-fees" id="wasm-execution-fees"></a>

{% hint style="info" %}
**WASM execution gas price**

WASM\_EXECUTION\_FEE = NO\_OF\_INSTRUCTIONS / 1,000
{% endhint %}

For regular actions, gas is paid by the calling user, in the same way as for public contracts.

Special ZK specific actions which are called when ZK nodes complete some work are paid by the contract.

### Staking fees <a href="#staking-fees" id="staking-fees"></a>

{% hint style="info" %}
**Staking gas price**

STAKING\_FEE = (LOCKED\_STAKES(Minimum 2,000 MPC tokens) / 100) \* 40,000
{% endhint %}

A ZK contract needs to pay 1% of the total locked stakes per month.

Currently, the standard option for a ZK contract ensures that it is on the blockchain for 28 days, you can prolong this by up to half a year at a time after deploying the contract.

The staking fee depends on the locked stakes which are determined by the user deploying the contract. The locked stakes are a minimum of 2,000 MPC tokens. To convert to gas the staking fee is multiplied by 40,000.

### Secret input fees <a href="#secret-input-fees" id="secret-input-fees"></a>

{% hint style="info" %}
**Secret input gas fees**

SECRET\_INPUT\_FEE = 4 \* (700 + 10 \_ (BIT\_LENGTH\_OF\_INPUT + 5)) + 5000
{% endhint %}

A ZK contract needs to pay for the transactions that ZK nodes must send when some variable is inputted.

The input fee is calculated taking into account the bit size of the variable inputted. Since the secret must be inputted to four nodes, this value must be multiplied by 4 to cover participating nodes. This covers the gas costs incurred by the nodes by processing this transaction. In addition, 5,000 is added as payment to the nodes.

### ZK computation fees <a href="#zk-computation-fees" id="zk-computation-fees"></a>

{% hint style="info" %}
Z**K computation gas fees**

ZK\_COMPUTATION\_FEE = 2 \* BASE\_SERVICE\_FEES + 5 \* NO\_OF\_MULTIPLICATIONS
{% endhint %}

A ZK contract needs to pay for the transactions that ZK nodes must send when a ZK computation is executed as well as for the multiplications in the computation.

During a computation transactions are sent from each computation node. For an optimistic computation each node sends 1 transaction to the binder. If the optimistic attempt fails, a pessimistic computation must be executed which entails each node sending 1 additional transaction to the binder. The cost of each of these transactions is covered using BASE\_SERVICE\_FEES, which is set to 25,000.

Besides this, the multiplications done during the computation must also be paid for. Each multiplication cost 5 gas or 5 USD cents for 1000 multiplications.

### ZK preprocessing fees <a href="#zk-preprocessing-fees" id="zk-preprocessing-fees"></a>

{% hint style="info" %}
**ZK preprocessing gas fees**

ZK\_PREPROCESSING\_FEE = 2 \* BASE\_SERVICE\_FEES + 5 \* NO\_OF\_PREPROCESSING\_MATERIAL
{% endhint %}

A ZK contract needs to pay for the preprocessing triples which the preprocessing nodes generate at various points during ZK computation. Preprocessing material is either multiplication triples used during computation to execute multiplications or input masks used during the generation of secret input variables.

When requesting preprocessing material, each node sends two transactions to the preprocessing contract. The cost of each of these transactions is covered using BASE\_SERVICE\_FEES.

The fee for preprocessing material is 5 USD cent per 1000 preprocessing triples.

Preprocessing material is requested in batches of 100,000 triples for multiplication triples and batches of 1,000 triples for input masks. This means the material for each batch of multiplication triples costs

`100,000 / 1,000 * 5 = 5,000 USD cents`

while the material for each input mask batch costs

`1,000 / 1,000 * 5 = 5 USD cents`

which is multiplied by 1,000 to get the price in gas.

### Opening secret variables fees <a href="#opening-secret-variables-fees" id="opening-secret-variables-fees"></a>

{% hint style="info" %}
**Opening secret variables gas fees**

OPEN\_SECRET\_VARIABLES\_FEE = SECRET\_INPUT\_FEE
{% endhint %}

Fees need to be paid for transactions that ZK nodes must send when secret variables are opened.

The fee calculation follows the same rules as those applied to the [secret input fee](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/zk-computation-gas-fees.html#secret-input-fees).

### Attestation fees <a href="#attestation-fees" id="attestation-fees"></a>

{% hint style="info" %}
A**ttesting data gas fees**

ATTESTATION\_FEE = BASE\_SERVICE\_FEES
{% endhint %}

A ZK contract needs to pay for the transactions that ZK nodes must send when some data needs to be attested.

The attestation fee is part of the basic services and is currently hardcoded to use BASE\_SERVICE\_FEES.

### EVM oracle fees <a href="#evm-oracle-fees" id="evm-oracle-fees"></a>

{% hint style="info" %}
**Receiving events from external EVM chain**

EVM\_ORACLE\_FEE = BASE\_SERVICE\_FEES
{% endhint %}

A ZK contract needs to pay for the transactions that the ZK nodes must send when an event from an external EVM contract is moved to the ZK contract.

The EVM oracle fee is part of the basic services and is currently hardcoded to use BASE\_SERVICE\_FEES.
