# Run a Baker Node

Baker nodes sign and produce blocks. This page describes how to change your node configuration from reader to baker. When you have completed the steps below, you have a node that is signing and producing blocks.

You must complete these requirements before you can continue

1. Get a [VPS](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#vps) with and [run a reader node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-reader-node.html)
2. Complete Synaps [KYC](https://partisiablockchain.gitlab.io/documentation/node-operations/complete-synaps-kyb.html#verification-process-for-individuals-kyc) / [KYB](https://partisiablockchain.gitlab.io/documentation/node-operations/complete-synaps-kyb.html#verification-process-for-businesses-kyb)
3. [Stake 25K MPC tokens](https://browser.partisiablockchain.com/node-operation)

#### Stop your reader node <a href="#stop-your-reader-node" id="stop-your-reader-node"></a>

Change to the directory of `docker-compose.yml`:

```
cd ~/pbc
```

Stop the node container:

```
docker compose down
```

#### Change `config.json` to support block production <a href="#change-configjson-to-support-block-production" id="change-configjson-to-support-block-production"></a>

To fill out the config.json for a block producing node you need to add the following information:

* Account private key (the account you've staked MPC with)
* IPv4 address of the server hosting your node (You get this from your VPS service provider)
* Ethereum, BNB and Polygon API endpoints. These are URL addresses pointing to a reader node on the mainnet for the respective blockchains (you should use a source you find trustworthy). [This user made guide](https://docs.google.com/spreadsheets/d/1Eql-c0tGo5hDqUcFNPDx9v-6-rCYHzZGbITz2QKCljs/edit#gid=0) has a provider list and further information about endpoints.

<details>

<summary>Note</summary>

As new external chains become supported, the chain endpoints configuration should be updated to support these. See [here](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#updating-your-byoc-chain-configuration) for information on updating your chain endpoints configuration.

</details>

* The IP, port and network public key of at least one other producer on the format `networkPublicKey:ip:port`, e.g. `02fe8d1eb1bcb3432b1db5833ff5f2226d9cb5e65cee430558c18ed3a3c86ce1af:172.2.3.4:9999`. The location of other known producers should be obtained by reaching out to the community.

You can read more about different fields in the `config.json` in the source code [here](https://gitlab.com/partisiablockchain/main/-/blob/main/src/main/java/com/partisiablockchain/server/CompositeNodeConfigDto.java)

To fill out the needed information we will use the [node-register tool](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#the-node-registersh-tool):

```
./node-register.sh create-config
```

When asked if the node is a block producing node, answer `yes`. The tool validates your inputs, and you will not be able to finish the configuration generation without inputting _all_ the required information.

Warning

Be sure to back up the keys the tool prints at the end. They are written in `config.json` and cannot be recovered if it is deleted.

You can verify the contents of the files are what you expect by opening them with `cat`:

```
sudo cat /opt/pbc-mainnet/conf/config.json
# The config file should be printed here
```

Your file should have similar contents to the one in the example below.

<details>

<summary>Example: Block producing config</summary>

```
```

</details>

### Start your block producing node <a href="#start-your-block-producing-node" id="start-your-block-producing-node"></a>

```
docker compose up -d
```

This pulls the latest image and starts the reader node in the background. If the command was executed successfully it won't print anything. To verify that the node is running:

```
docker logs -f pbc-mainnet
```

This prints a sequence of log statements. The node needs hours to process the existing blocks in the ledger and catch up to the present. All the timestamps are in [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) and can therefore be offset up to 12 hours from your local time.

In the [maintenance section](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html) you can see what the logs mean.

### Register your node <a href="#register-your-node" id="register-your-node"></a>

Registration of the node happens via the [node-register tool](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#the-node-registersh-tool). The registration ensures that your account and tokens are associated with your node. It also creates a profile with public information about your node.

<details>

<summary>Note</summary>

Your node _must_ be up-to-date with the rest of the network and you _must_ have at least 25,000 gas in your account. Otherwise you will not be able to send the register transaction.

</details>

You can check the status by running the node-register tool with the `status` command:

```
./node-register.sh status
```

The node REST server will respond with a code `204 No Content` if it is up-to-date with the network, and the tool will print a message saying `Your node is up to date with the rest of the network`. Otherwise, a message saying `Your node is NOT up to date` will be printed.

You need at least 25,000 gas to send the register transaction. To check your gas balance log in to the [Partisia Blockchain Browser](https://browser.partisiablockchain.com/account?tab=byoc), go to _Your Account_ and then _BYOC_, where your gas balance is shown. You can add gas to your account with the [bridge](https://browser.partisiablockchain.com/bridge).

To send the register transaction you need to log in to your node and go to the `~/pbc` folder, and run the node-register tool with the `register-node` command:

```
cd ~/pbc
./node-register.sh register-node
```

Follow the on-screen instructions until the registration is completed.

<details>

<summary>Note</summary>

You can update your information from the Register Transaction by doing a new registration transaction.

</details>

Verify that the account key you have in the `config.json` file matches the blockchain address you've used in your KYC/KYB.

If it still fails then reach out to the [community](https://partisiablockchain.gitlab.io/documentation/get-support-from-pbc-community.html).

### Keeping your node software up to date <a href="#keeping-your-node-software-up-to-date" id="keeping-your-node-software-up-to-date"></a>

Using out-of-date node software can prevent your node from joining future committees.

If you have already not setup automatic updates you can follow the guide [here](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-reader-node.html#get-automatic-updates).

### Conditions for inclusion <a href="#conditions-for-inclusion" id="conditions-for-inclusion"></a>

Formal conditions for inclusion in the network is stipulated in the Yellow Paper [YP\_0.98 Ch. 2.3.1 pp. 11-12](https://drive.google.com/file/d/1OX7ljrLY4IgEA1O3t3fKNH1qSO60_Qbw/view):

* The public information regarding the node given by the operator must be verified by Synaps
* Sufficient stakes committed
* The transaction fees of _Register_ and _Staking_ transaction have been paid

You now have a baker node running on your VPS. When your node has caught up to the ledger, you should be able to confirm in your docker logs, that your node is signing and producing blocks. It is a good idea to keep a look at your node's performance, because baker node revenue depend on performance. You can learn more about this in the [health and maintenance section](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html) of the guide.

Continue on the following pages to upgrade to an [oracle node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-deposit-or-withdrawal-oracle-node.html) or [ZK node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-zk-node.html).
