# Governance system smart contracts overview

Partisia Blockchain is governed by a group of system smart contracts. The system contracts solve different types of tasks. Below the contracts are listed under a category and description of their overall responsibilities. Each contract description links to the contract on chain via the the [Partisia Blockchain Browser](https://browser.partisiablockchain.com/). In the browser you can see the contract state and invoke the actions of the contract with your [wallet](https://snaps.metamask.io/snap/npm/partisiablockchain/snap/).

### Node operation <a href="#node-operation" id="node-operation"></a>

[Node operators](https://partisiablockchain.gitlab.io/documentation/node-operations/start-running-a-node.html) own and operate the nodes (servers) where the blockchain software runs. System contracts are needed for listing and staking for confirmed block producers, as well as for payment to node operators for the services they provide, votes on software updates and registration of new node operators.

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/04203b77743ad0ca831df9430a6be515195733ad91">Block producer orchestration contract</a></summary>

The Block producer orchestration (BPO) contract holds the information about block producers. In the state you can see the current and previous committees, the timestamp of the last committee change and the minimum required node software version. Invoking the contract will change the committee if the current committee is more than a month old. [See contract in browser](https://browser.partisiablockchain.com/contracts/04203b77743ad0ca831df9430a6be515195733ad91)

[Block producer orchestration contract documentation](https://partisiablockchain.gitlab.io/governance/bp-orchestration-contract/com/partisiablockchain/governance/bporchestration/BpOrchestrationContract.html)

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/04fe17d1009372c8ed3ac5b790b32e349359c2c7e9">Fee distribution</a></summary>

The Fee distribution contract takes care of the payment for baker node services. Every node makes a list of how many signatures they have received from every other node. Roughly, the top ⅔ best performers in each list get a vote. If more than ⅓ producers vote for your node, then it will be paid in that [epoch](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#epoch). [See contract in browser](https://browser.partisiablockchain.com/contracts/04fe17d1009372c8ed3ac5b790b32e349359c2c7e9)

[Fee distribution contract documentation](https://partisiablockchain.gitlab.io/governance/fee-distribution-contract/com/partisiablockchain/governance/feedistribution/package-summary.html)

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/04c5f00d7c6d70c3d0919fd7f81c7b9bfe16063620">System update</a></summary>

The system update contract facilitates votes from committee members (confirmed and active block producers) on updates of on chain information, such as updates to governance contracts. There are manual and automated votes. Automated votes are yes votes by default unless actively changed by the individual node operator. [Learn how to change the automated vote from yes to no for a specific vote](https://partisiablockchain.gitlab.io/documentation/node-operations/how-system-updates-and-voting-work.html). [See contract in browser](https://browser.partisiablockchain.com/contracts/04c5f00d7c6d70c3d0919fd7f81c7b9bfe16063620)

[System update contract documentation](https://partisiablockchain.gitlab.io/governance/voting/com/partisiablockchain/governance/voting/VotingContract.html)

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/014aeb24bb43eb1d62c0cebf2a1318e63e35e53f96">Synaps KYC / KYB contract</a></summary>

The KYC contract validates block producer's personal information coming from the KYC provider before they can be confirmed as block producers and listed in the BPO contract. [See contract in browser](https://browser.partisiablockchain.com/contracts/014aeb24bb43eb1d62c0cebf2a1318e63e35e53f96)

[Synaps KYC / KYB contract documentation](https://partisiablockchain.gitlab.io/governance/synaps-kycb-contract/com/partisiablockchain/governance/SynapsKycbContract.html)

</details>

### [BYOC](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/byoc.html) - The system of liquidity <a href="#byoc-the-system-of-liquidity" id="byoc-the-system-of-liquidity"></a>

On Partisia Blockchain you use coins from other chains (BYOC) as payment for transactions. PBC also allows users to transfer BYOC between [accounts](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/create-an-account.html) and contracts. BYOC system contracts controls allocations of nodes handling the transfers, withdrawals, deposits, price monitoring, staking of nodes and the audit of transfers done in each [epoch](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#epoch).

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/04f1ab744630e57fb9cfcd42e6ccbf386977680014">Large oracle contract</a></summary>

Administrates and oversees work done by [small oracles](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#small-oracle). This includes allocation of eligible nodes to serve in the small oracles, the tokens staked on oracle service and voting on all disputes regarding the small oracles' work with BYOC. [See contract in browser](https://browser.partisiablockchain.com/contracts/04f1ab744630e57fb9cfcd42e6ccbf386977680014)

[Large oracle contract documentation](https://partisiablockchain.gitlab.io/governance/large-oracle-contract/com/partisiablockchain/governance/LargeOracleContract.html).

</details>

#### Small oracles <a href="#small-oracles" id="small-oracles"></a>

Small oracles move funds ([Deposit and withdrawal oracles](https://partisiablockchain.gitlab.io/documentation/node-operations/oracles-on-partisia-blockchain.html#what-is-a-small-oracle)) or check prices ([Price Oracles](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#price-oracle)). Each deposit and withdrawal oracle consist of 3 nodes selected by the Large oracle. There is a small oracle contract for each deposit and withdrawal oracle corresponding to the different coins supported on PBC. Price oracles can be supported by a large number of nodes, there is likewise a small oracle contract for each price oracle monitoring a price of a specific BYOC.

<details>

<summary><a href="https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#price-oracle">Price Oracles</a></summary>

Price oracles ensures accurate prices for specific BYOC. The nodes serving in price oracles get price data from [Chainlink nodes](https://docs.chain.link/data-feeds/price-feeds). [Price oracle explanation](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#price-oracle). The contracts responsible for the price oracles are below. [ETH Price Oracle](https://browser.partisiablockchain.com/contracts/0485010babcdb7aa56a0da57a840d81e2ea5f5705d) [BNB Price Oracle](https://browser.partisiablockchain.com/contracts/049abfc6e763e8115e886fd1f7811944f43b533c39)

[Price oracle contract documentation](https://partisiablockchain.gitlab.io/governance/price-oracle-contract/com/partisiablockchain/governance/priceoracle/PriceOracleContract.html).

</details>

<details>

<summary><a href="https://partisiablockchain.gitlab.io/documentation/node-operations/oracles-on-partisia-blockchain.html#what-is-a-small-oracle">Deposit and withdrawal oracles</a></summary>

You see the oracle contracts handling deposits and withdrawals for BYOCs in the [list of bridgeable coins](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/bridging-byoc-by-sending-transactions.html#bridgeable-coins-on-mainnet). These contracts have a transfer limit controlling when the nodes providing the service will be rotated. The contract state shows which nodes serve the oracle and a list of the deposits or withdrawals they have completed.

[Comprehensive list of deposit and withdrawal oracles on PBC and native chains of BYOC](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/bridging-byoc-by-sending-transactions.html#bridgeable-coins-on-mainnet)

[Deposit oracle contract documentation](https://partisiablockchain.gitlab.io/governance/byoc-incoming/com/partisiablockchain/governance/byocincoming/ByocIncomingContract.html).

[Withdrawal oracle contract documentation](https://partisiablockchain.gitlab.io/governance/byoc-outgoing/com/partisiablockchain/governance/byocoutgoing/ByocOutgoingContract.html).

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/0458ff0a290e2fe847b23a364925799d1c53c8b36b">BYOC orchestration</a></summary>

Facilitates the deployment of new price, deposit and withdrawal [oracles](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-deposit-or-withdrawal-oracle-node.html). [See contract in browser](https://browser.partisiablockchain.com/contracts/0458ff0a290e2fe847b23a364925799d1c53c8b36b)

[BP Orchestration contract documentation](https://partisiablockchain.gitlab.io/governance/byoc-orchestrator-contract/com/partisiablockchain/governance/byocorchestrator/ByocOrchestratorContract.html).

</details>

### MPC tokens <a href="#mpc-tokens" id="mpc-tokens"></a>

[MPC tokens](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#mpc-token) are the native tokens of PBC. They are used by node operators and other token holders for staking on a service provided by a node. Tokens can be confiscated in case of malicious behavior and be used for dispute settlement. Staked tokens incentives honest and reliable node services. They can also be exchanged for Wrapped MPC (WMP), the [BYOC](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/governance-system-smart-contracts-overview.html#byoc-the-system-of-liquidity) equivalent of MPC tokens. [Read more about MPC tokens](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#mpc-token)

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/01a4082d9d560749ecd0ffa1dcaaaee2c2cb25d881">MPC token contract</a></summary>

Facilitates the change of state (location, association, staking, vesting) of MPC tokens as well as transfer of MPC tokens between accounts. MPC tokens are always associated with the MPC token balance of a specific account or contract. You can learn more about MPC tokens and their properties in the [MPC token model](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/mpc-token-model-and-account-elements.html).

[MPC token contract documentation](https://partisiablockchain.gitlab.io/governance/mpc-token/com/partisiablockchain/governance/mpctoken/package-summary.html).

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/01615beb1c2bf57e45fcd1c4e67ef35b8735a685b1">MPC MPC20 contract</a></summary>

Facilitates transfers of MPC tokens between accounts _or_ contracts. Note that the "MPC token" contract does not allow transfers to and from contracts. This contract uses [the standard of MPC-20](https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/mpc-20-token-contract.html)

[MPC MPC20 contract documentation](https://partisiablockchain.gitlab.io/governance/mpc-mpc20-contract/com/partisiablockchain/governance/mpcmpc20/package-summary.html)

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/017d9dacdd01f0b2bd4de40da37f545e89b7faa149">MPC wrap contract</a></summary>



</details>

Facilitates the exchange, or _wrapping_, of MPC tokens into their BYOC equivalent, WMPC. It also handles _unwrapping_, which is the process of exchanging WMPC for MPC tokens.

```
[MPC wrap contract documentation](https://partisiablockchain.gitlab.io/governance/mpc-wrap/com/partisiablockchain/governance/mpcwrap/MpcWrapContract.html)
```

### Public smart contracts and Zero Knowledge smart contract <a href="#public-smart-contracts-and-zero-knowledge-smart-contract" id="public-smart-contracts-and-zero-knowledge-smart-contract"></a>

Users can deploy two types of smart contracts, [public contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/what-is-a-smart-contract.html) and [zero knowledge contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/zk-smart-contracts.html). Node operators can [register](https://browser.partisiablockchain.com/contracts/01a2020bb33ef9e0323c7a3210d5cb7fd492aa0d65/registerAsZkNode) their nodes for ZK services. ZK calculations requires the allocation of 4 registered nodes to handle the secret calculation. You have the ability to deploy your own smart contracts with WASM Deploy and ZK Deploy.

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/018bc1ccbb672b87710327713c97d43204905082cb">ZK Deploy</a></summary>

Responsible for the deployment of new ZK smart contracts. [See contract in browser](https://browser.partisiablockchain.com/contracts/018bc1ccbb672b87710327713c97d43204905082cb)

[ZK Deploy contract documentation](https://partisiablockchain.gitlab.io/real/real-deploy/com/partisiablockchain/zk/real/deploy/RealDeployContract.html).

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/01a2020bb33ef9e0323c7a3210d5cb7fd492aa0d65">ZK Node Registry</a></summary>

Handles registration of new ZK nodes, as well as disputes and allocation to specific ZK jobs. [See contract in browser](https://browser.partisiablockchain.com/contracts/01a2020bb33ef9e0323c7a3210d5cb7fd492aa0d65)

[ZK Node Registry contract documentation](https://partisiablockchain.gitlab.io/real/zk-node-registry/com/partisiablockchain/zk/real/ZkNodeRegistryContract.html).

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/01385fedf807390c3dedf42ba51208bc51292e2657">ZK Preprocess</a></summary>

Preprocesses data prior to use for secure multiparty computation by currently allocated ZK nodes. [See contract in browser](https://browser.partisiablockchain.com/contracts/01385fedf807390c3dedf42ba51208bc51292e2657)

[ZK Preprocess contract documentation](https://partisiablockchain.gitlab.io/real/real-preprocess/com/partisiablockchain/zk/real/preprocess/PreProcess.html).

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/0197a0e238e924025bad144aa0c4913e46308f9a4d">WASM Deploy</a></summary>

Facilitates deployment of public smart contracts. [See contract in browser](https://browser.partisiablockchain.com/contracts/0197a0e238e924025bad144aa0c4913e46308f9a4d)

[Public deploy contract documentation](https://partisiablockchain.gitlab.io/governance/pub-deploy/com/partisiablockchain/governance/pubdeploy/PublicDeployContract.html).

</details>

### Foundation <a href="#foundation" id="foundation"></a>

[Partisia Blockchain foundation](https://partisiablockchain.com/foundation) provide incentives for node operators in form of token rewards. The incentives depend on initial time and size of investment as well measured scores of reliability and trust. The foundation raise funds for infrastructure through the sale of tokens. These two tasks are handled by following system contracts. [Read more about PBC foundation](https://partisiablockchain.com/foundation)

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/01ad44bb0277a8df16408006c375a6fa015bb22c97">Foundation Eco-System</a></summary>

Allows foundation members to release remaining tokens from the Eco-system pool to incentivize further growth of the infrastructure and expansion of services. [See contract in browser](https://browser.partisiablockchain.com/contracts/01ad44bb0277a8df16408006c375a6fa015bb22c97)

[Foundation eco-system contract documentation](https://partisiablockchain.gitlab.io/governance/foundation-contract/com/partisiablockchain/governance/foundationcontract/FoundationContract.html)

</details>

<details>

<summary><a href="https://browser.partisiablockchain.com/contracts/012635f1c0a9bffd59853c6496e1c26ebda0e2b4da">Foundation Sale</a></summary>

Facilitates the sale of MPC tokens. [See contract in browser](https://browser.partisiablockchain.com/contracts/012635f1c0a9bffd59853c6496e1c26ebda0e2b4da)

[Foundation sale contract documentation (Same as foundation eco-system)](https://partisiablockchain.gitlab.io/governance/foundation-contract/com/partisiablockchain/governance/foundationcontract/FoundationContract.html)

</details>
