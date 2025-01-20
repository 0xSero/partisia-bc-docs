# Starting a Node

This page explains what a node is, how to join the node operator community, and it gives an overview of what type of node you can run with the amount of MPC tokens you have.

### What is a node? <a href="#what-is-a-node" id="what-is-a-node"></a>

Nodes are the computers on the blockchain network. They perform services for blockchain users by facilitating the transactions on the blockchain. The node operator can make revenue from the transaction costs.

### Onboarding <a href="#onboarding" id="onboarding"></a>

To start your journey becoming a node operator on the Partisia Blockchain complete the two onboarding steps:

1. Fill out the [node operator onboarding form](https://forms.monday.com/forms/8de1fb7d3099178333db642c4d1fe640?r=euc1).
2. [Join Discord](https://discord.com/invite/KYjucw3Sad) and submit a node operator support ticket. In the ticket, submit a screenshot of your wallet showing your MPC account balance.

Joining the Discord server and completing the survey gives you the following benefits:

* Support and notification in case we or community members register a problem with your node
* News of updates subject to node operator votes, for example when node operators can support a new coin on the chain ([oracle service](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#small-oracle))

### Which node should you run? <a href="#which-node-should-you-run" id="which-node-should-you-run"></a>

Nodes performing paid services require a [stake](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#stakestaking) of MPC tokens. Higher stake services earn higher revenues. The node operator must complete the [Synaps KYC/KYB](https://partisiablockchain.gitlab.io/documentation/node-operations/complete-synaps-kyb.html) process to gain authorization to perform paid services.

There is an overlap in the set-up of the different kinds of node. All nodes share the set-up related to reader nodes. All staking nodes share the baker node set-up. Therefore, to configure and register your node for higher paying services, you must first set-up and register as a baker node.

If you have completed the reader and baker part of the guide you can perform (given sufficient tokens) any combination of the other services, including multiple oracles on the same node.

| **Required total MPC token balance** | **Available Node service**                                                                                                                      | **Service consist of**                                                           |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 0                                    | [Reader node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-reader-node.html)                                        | Free: Reading the blockchain state                                               |
| 25K                                  | [Baker node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-baker-node.html)                                          | <p>Free: Reader node service<br>25K stake: Signing and producing blocks</p>      |
| 30K                                  | [Price oracle](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-price-oracle-node.html)                                 | <p>25K stake: Baker node service<br>5K stake: Price monitoring</p>               |
| 100K                                 | [ZK node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-zk-node.html)                                                | <p>25K stake: Baker node service<br>75K stake: ZK computations</p>               |
| 275K                                 | [Deposit or withdrawal oracle](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-deposit-or-withdrawal-oracle-node.html) | <p>25K stake: Baker node service<br>250K stake: Moving BYOC on and off chain</p> |

{% hint style="info" %}
**All nodes require a** [**VPS**](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#vps) **with these specs or better**

* 8 vCPU or 8 cores, 10 GB RAM (8 GB allocated Java Virtual Machine), 256 GB SSD, publicly accessible IPv4 with ports9888-9897 open
* Recommended software: Docker, Docker Compose V2, Ubuntu 20.04, nano or other text editors
{% endhint %}
