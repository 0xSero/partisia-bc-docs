# Integrations

We have collected articles that help you integrate your application with the rest of Partisia Blockchain in this section. There are different ways of integrating with the Partisia Blockchain and figuring out what tool to use is explained in the next sets of articles. If you cant find what you need, feel free to engage with the dev-community [on our discord](https://partisiablockchain.gitlab.io/documentation/get-support-from-pbc-community.html).

### Rest endpoint examples and libraries <a href="#rest-endpoint-examples-and-libraries" id="rest-endpoint-examples-and-libraries"></a>

You can get information about blocks, transactions, contract state and more through the REST API. The REST API is also be used for sending signed transactions to the blockchain.

The REST API is available on any reader node.

If you need access to a reader node for integration purposes we recommend you to [run your own node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-reader-node.html).\
If you don't want to run your own, you can ask for help in the [community Discord](https://partisiablockchain.gitlab.io/documentation/get-support-from-pbc-community.html).

If you just want to test the API or only need occasional access, then a good approach is to use the public reader node. Notice, that the public reader is rate limited.

The public reader node is available at

* [https://reader.partisiablockchain.com](https://reader.partisiablockchain.com) (mainnet)
* [https://node1.testnet.partisiablockchain.com](https://node1.testnet.partisiablockchain.com) (testnet)

We also publish a number of [libraries](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#libraries) that help create transactions and using the REST API.

### REST API endpoints <a href="#rest-api-endpoints" id="rest-api-endpoints"></a>

Below is a collection of commonly used rest endpoints.

In our [rest server source repository](https://gitlab.com/partisiablockchain/core/server), you can find all the different endpoints and all returned data types.

### Open API <a href="#open-api" id="open-api"></a>

The reader nodes and baker nodes expose an API that is defined by an OpenAPI spec. You can view the endpoints available here: [OpenAPI UI](https://gitlab.com/partisiablockchain/core/server/-/blob/main/java/server/src/main/resources/openapi.json).

#### Transaction information <a href="#transaction-information" id="transaction-information"></a>

[https://reader.partisiablockchain.com/shards/Shard1/blockchain/transaction/11d09178b39c10520aec717200a4a5cd229e948bc15c4a87e65d682008f86db5](https://reader.partisiablockchain.com/shards/Shard1/blockchain/transaction/11d09178b39c10520aec717200a4a5cd229e948bc15c4a87e65d682008f86db5)

Where

* `Shard1` is the shard
* `11d09178b39c10520aec717200a4a5cd229e948bc15c4a87e65d682008f86db5` is the transaction identifier

#### List of transactions from point in time <a href="#list-of-transactions-from-point-in-time" id="list-of-transactions-from-point-in-time"></a>

[https://reader.partisiablockchain.com/shards/Shard0/blockchain/transaction/latest/10/2018112](https://reader.partisiablockchain.com/shards/Shard0/blockchain/transaction/latest/10/2018112)

Where

* `Shard0` is the shard
* `10`is the number of transactions to return
* `2018112` is the block number on the shard

#### Block Information <a href="#block-information" id="block-information"></a>

[https://reader.partisiablockchain.com/shards/Shard0/blockchain/blocks/6132deae77e2f2576e450b82cc681a363e064637443b736f841f2b99256f5926](https://reader.partisiablockchain.com/shards/Shard0/blockchain/blocks/6132deae77e2f2576e450b82cc681a363e064637443b736f841f2b99256f5926)

Where

* `Shard0` is the shard
* `6132deae77e2f2576e450b82cc681a363e064637443b736f841f2b99256f5926` is the block identifier

#### Smart Contract Information <a href="#smart-contract-information" id="smart-contract-information"></a>

[https://reader.partisiablockchain.com/chain/contracts/02c63dc725bfce5abc5b019d14ac84b4aaf8747e9f](https://reader.partisiablockchain.com/chain/contracts/02c63dc725bfce5abc5b019d14ac84b4aaf8747e9f)

Where

* `02c63dc725bfce5abc5b019d14ac84b4aaf8747e9f` is the smart contract address

#### Account Information <a href="#account-information" id="account-information"></a>

[https://reader.partisiablockchain.com/chain/accounts/00803c974202d4dd8db22bd3833b8d123f89ea199b](https://reader.partisiablockchain.com/chain/accounts/00803c974202d4dd8db22bd3833b8d123f89ea199b)

Where

* `00803c974202d4dd8db22bd3833b8d123f89ea199b` is the address of the account

#### Blockchain global information <a href="#blockchain-global-information" id="blockchain-global-information"></a>

[https://reader.partisiablockchain.com/chain](https://reader.partisiablockchain.com/chain)
