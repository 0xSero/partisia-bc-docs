# Node Health and Maintenance

The maintenance page takes you through the following node-related topics:

* [The node-register.sh tool](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#the-node-registersh-tool)
* [Is your baker node working?](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#is-your-baker-node-working)
* [Get automatic updates](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#get-automatic-updates)
* [Update your node manually](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#updating-your-node-manually)
* [Check your IP accessibility and peers](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#check-your-connection-to-the-peers-in-the-network-and-your-uptime)
* [How to monitor node performance](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#how-to-monitor-node-performance)
* [Interpret log messages and debugging problems - See if your node is signing blocks](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#logs-and-storage)
* [For ZK and reader nodes: Sorting logs of nginx proxy and acme](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#for-zk-and-reader-nodes-sorting-logs-of-nginx-proxy-and-acme)
* [Confirm that your BYOC endpoints are working](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#confirm-that-your-byoc-endpoints-are-working)
* [How to migrate your node to a different VPS](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#how-to-migrate-your-node-to-a-different-vps)
* [Install Network Time Protocol (NTP) to avoid time drift](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#install-network-time-protocol)
* [Inactivity](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#Inactivity)
* [Voting for quarterly Rewards](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#voting-for-quarterly-rewards)

### The `node-register.sh` tool <a href="#the-node-registersh-tool" id="the-node-registersh-tool"></a>

The `node-register.sh` utility tool can help you set up, run and maintain your node. This includes initializing and updating your node configuration, as well as registering your node for block production and signing. The tool can be used with the following commands:

* `create-config`: Used to set up the initial config file for your node, or to update an existing config file. This command will take you through a step-by-step process for filling out the information needed for your config.
  * Example: `./node-register.sh create-config`
* `register-node`: Used to register your node for producing and signing blocks. This command will take you through a step-by-step process to register for block production and signing.
* `status`: Checks whether your node is up to date with the rest of the network.
* `validate-kyc <session-id>`: Checks whether your Synaps KYC registration, based on the supplied `session-id`, is valid and approved.
  * Example: `./node-register.sh validate-kyc 3bca023e-abc1-12a1-bdab-5fa1c3b7b9b3`
* `report-active`: Used to mark your node as _CONFIRMED_ if it was previously set to [_INACTIVE_](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#Inactivity) on the [Block Producer Orchestration contract](https://browser.partisiablockchain.com/contracts/04203b77743ad0ca831df9430a6be515195733ad91?tab=state).
* `compute-rewards <rewards-quarter>`: Compute quarterly rewards for staking. See [Voting for Quarterly Rewards](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#voting-for-quarterly-rewards).

The commands can be run with `--help` for additional information.

The newest version of `node-register.sh` is located on [GitLab](https://gitlab.com/partisiablockchain/main/-/raw/main/scripts/node-register.sh). To download the tool run:

```
curl https://gitlab.com/partisiablockchain/main/-/raw/main/scripts/node-register.sh --output node-register.sh
```

Once it is downloaded you need to make it executable:

```
chmod +x node-register.sh
```

### Is your baker node working? <a href="#is-your-baker-node-working" id="is-your-baker-node-working"></a>

If your node is unattended for to long it can run into problems. Problems that may affect your node's earning potential and the safety of your stake. Your node has to be up-to-date to participate in the committee. If your node is not updated regularly, it is bound to fall out of committee. Only nodes up-to-date can participate in forming a new committee, so every time a new committee is formed from registered nodes, only nodes with the newest version of Partisia Software can be included. Your node can only perform services and by extension earn rewards when in the committee. After you are included you want to make sure your node is able to continue to participate.\
To optimize your node's earning potential you should implement automatic updates and check up on the node's performance regularly.

**Your baker node is working when:**

* Your node is producing blocks when chosen as producer. At the moment nodes take turns based on their index from the list of [committee members](https://browser.partisiablockchain.com/accounts?tab=node_operators). This can be affirmed in [How to monitor node performance](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#how-to-monitor-node-performance) below
* Your node is signing blocks. Can be checked in the logs as explained below
* Your node is running the newest version of Partisia Software. The easiest way to ensure this is by implementing [automatic updates](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#get-automatic-updates)

You can confirm that your node software is up-to-date with the following command:

```
docker inspect --format='{{.Image}}' YOUR_CONTAINER_NAME
```

The number must match the latest [configuration digest](https://gitlab.com/partisiablockchain/mainnet/container_registry/3175145).

### Get automatic updates <a href="#get-automatic-updates" id="get-automatic-updates"></a>

All nodes, independent of type, should be set up to update their software automatically. To set up automatic updates you will need to install Cron, a time based job scheduler:

```
apt-get install cron
```

Now you are ready to start.

**1. Create the auto update script:**

Go to the directory where `docker-compose.yml` is located.

```
cd ~/pbc
```

Open the file in nano:

```
nano update_docker.sh
```

Paste the following content into the file:

```
#!/usr/bin/env bash

DATETIME=$(date -u)
echo "$DATETIME"

cd ~/pbc

/usr/bin/docker compose pull pbc
/usr/bin/docker compose up -d pbc
```

Save the file by pressing `CTRL+O` and then `ENTER` and then `CTRL+X`.

**2. Make the file executable:**

```
chmod +x update_docker.sh
```

Type `ls -l` and confirm _update\_docker.sh_ has an x in its first group of attributes, that means it is now executable.

**3. Set update frequency to once a day at a random time:**

```
crontab -e
```

This command allows you to add a rule for a scheduled event. You will be asked to choose your preferred text editor to edit the cron rule, if you have not already chosen a preference.

Paste and add the rule to the scheduler. Make sure to have no "#" in front of the rule:

```
m h * * * /home/userNameHere/pbc/update_docker.sh >> /home/userNameHere/pbc/update.log 2>&1
```

For minutes (m) choose a random number between 0 and 59, and for hours (h) choose a random number between 0 and 23. If you are in doubt about what the cron rule means you can use this page: [https://crontab.guru/](https://crontab.guru/) to see the rule expressed in words.

Press `CTRL+X` and then `Y` and then `ENTER`.

This rule will make the script run and thereby check for available updates once every day.

To see if the script is working you can read the update log with the _cat command_:

```
cat update.log
```

You can change the time of the first update if you don't want to wait a day to confirm that it works.

If your version is up-to-date, you should see:

```
YOUR_CONTAINER_NAME is up-to-date
```

If you are currently updating you should see:

```
Pulling YOUR_CONTAINER_NAME ... pulling from privacyblockchain/de...
```

Warning

Never include a shutdown command in your update script, otherwise your node will go offline every time it checks for updates.

### Updating your node manually <a href="#updating-your-node-manually" id="updating-your-node-manually"></a>

You should always have [automatic updates](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#get-automatic-updates) enabled on your node. But there can be situations where you might want to update it manually, for instance if you are experiencing problems with the node.

In the following it is assumed you are using `~/pbc` as directory for your `docker-compose.yml`.

Updating the PBC node is a simple 3-step process:

<details>

<summary>If you are running more than one container in your <code>docker-compose.yml</code> read this</summary>

```
```

</details>

```
cd ~/pbc
```

```
docker compose pull
```

```
docker compose up -d
```

First you change the directory to where you put your `docker-compose.yml`. You then pull the newest image and start it again. You should now be running the newest version of the software.

### Check your connection to the peers in the network and your uptime <a href="#check-your-connection-to-the-peers-in-the-network-and-your-uptime" id="check-your-connection-to-the-peers-in-the-network-and-your-uptime"></a>

Your node can only get registered as a block producer and participate in the committee if your host IP is reachable. Replace the letters in the URL below with the IP of the server hosting your node. This should navigate you to a page showing a JSON, with the following information:

`http://PUBLIC_IP_OF_SERVER_HOSTING_THIS_NODE:9888/status`

```
{
    "versionIdentifier": "Version number of PBC",
    "uptime": 11552567,
    "systemTime": 1700491419888,
    "knownPeersNetworkKeys": ["network addresses (Base64) of connected baker nodes"],
    "networkKey": "Network address(Base64)",
    "blockchainAddress": "account address (Hex)",
    "finalizationKey": "finalization publicKey (base64)",
    "numberOfProcessors": 8,
    "systemLoad": 0.51,
    "freeMemory": 1574211176,
    "totalMemory": 2701131776,
    "maxMemory": 17179869184
}
```

Uptime is measured in milliseconds, and show how long your server has been running uninterrupted.

If you cannot open your status endpoint there is probably a problem with the opening of ports of the [VPS](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#vps). See which ports are allowed through the firewall:

```
sudo ufw status
```

Make sure you have opened for ports 9888-9897. If not consult the instructions [here](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-reader-node.html).

### How to monitor node performance <a href="#how-to-monitor-node-performance" id="how-to-monitor-node-performance"></a>

The node operator community has made several tools that can help you monitor the performance of your node:

Check block production and finalization time with MPC Node Stats:

* [Browser Version](https://mpcnodestats.dreamhosters.com/)
* [iPhone](https://apps.apple.com/gr/app/mpc-node-stats/id1661132518)
* [Android](https://play.google.com/store/apps/details?id=com.mpcnodeio.mpcstats\&pli=1)

Compare your received stake delegations with that of other committee members:

* [Parti Scan](https://partiscan.io/)

### Logs and storage <a href="#logs-and-storage" id="logs-and-storage"></a>

You use the docker logs to see activity on the chain and if your node is signing blocks. The logs of the node are written to the standard output of the container and are therefore managed using the tools provided by Docker. You can read about configuring Docker logs [here](https://docs.docker.com/config/containers/logging/configure/).

The storage of the node is based on RocksDB. It is write-heavy and will increase in size for the foreseeable future. The number and size of reads and writes is entirely dependent on the traffic on the network.

#### Common log messages <a href="#common-log-messages" id="common-log-messages"></a>

**Signing BlockState** - All is well.

**Not signing as shutdown is active** - You may assume all is well. Shutdown happens when chosen producer fails to produce a block, a reset block is made, and then a new node is chosen for the role of producer.

**Not signing** - This is not a good sign, you are not signing blocks. First, check if you are on the list of [current committee members](https://browser.partisiablockchain.com/accounts?tab=node_operators), if you are not, and you have already sent the Register Transaction, then you should search for your PBC account address in the state on the [Block Producer Orchestration Contract](https://browser.partisiablockchain.com/contracts/04203b77743ad0ca831df9430a6be515195733ad91) (BPOC). There is a field for each producer called "status": - after this you will see either "CONFIRMED" or "PENDING". Confirmed means you are registered as a block producer and are formally eligible to participate in the committee. Pending means your public information is still awaiting manual approval from the team cross-checking the information you have given. If you cannot find your address in the BPOC at all you need to resend your registration. Alternatively, if you are on the list of committee members and still get persistent "Not signing" then you almost certainly have some problem in your config.json Probably you have a wrong or no key in one of the fields: networkKey, accountKey or finalizationKey, or you forgot to add the host IP address.

**Got a message with wrong protocol identifier** - This message comes every time a shutdown has occurred (in other words whenever a producer has not produced the block he is supposed to). So, on its own that message does not indicate a problem. But, if the log just repeats and don't change to a new message saying Executing Block... it could suggest you are running an outdated version of our software, a version that does not pull the newest docker image automatically.

**WebApplicationException. Status=404** - You may assume all is well. You may encounter different types of not found errors in the logs. Most of them are not indicative of a problem at your end. They occur when a node in the network has not received what it expected you can in most cases see the address or producer index of the nodes related to the error.

#### Sorting Baker node logs <a href="#sorting-baker-node-logs" id="sorting-baker-node-logs"></a>

**Latest logs:**

```
docker logs -f nameOfDockerContainer
```

This will show the latest logs after they have caught up to present.

**Sorting by time:**

```
docker logs --since 1h  nameOfDockerContainer
```

This will give you the latest hour of logs

**Sorting by number of lines:**

```
docker logs --tail 1000  nameOfDockerContainer
```

This will give you the latest 1000 lines of logs.

```
docker logs --tail 1000 -f  nameOfDockerContainer
```

You can add the `-f` after a command to continue the logs afterwards.

**Sort for specific messages:**

You can use the _grep command_ to get logs containing a specific string.

```
docker logs --since 1h pbc-mainnet | grep "Signing BlockState"
```

This will give you the blocks you have signed the last hour. You might also want to look for blocks you created when you were chosen as producer `| grep "Created Block"`.

#### For ZK and reader nodes: Sorting logs of nginx proxy and acme <a href="#for-zk-and-reader-nodes-sorting-logs-of-nginx-proxy-and-acme" id="for-zk-and-reader-nodes-sorting-logs-of-nginx-proxy-and-acme"></a>

Your nginx reverse proxy and acme certificate renewal are run in docker containers (`pbc-nginx` and `pbc-acme`). Same way you run your baker or reader node service in the container `pbc-mainnet`. You can use the same docker commands to get the nginx and acme container logs. When sorting the logs, its recommended to use relevant keywords: E.g. keywords related to your SSL/TSL certificate.

The docker logs of a node service are stored as one category in the same file, and displayed in the same color when you print the container logs with the `docker logs` command. Whereas nginx stores logs in two separate categories displayed in its own color, access logs (white text) and error logs (red text). The access logs shows client request received by nginx. Error logs shows messages related to the function of nginx including processes started and ended, requests processed or skipped and shutdowns.

The SSL/TSL certificate renewal, done with acme-companion, shows up in the nginx logs because the `pbc-nginx` and `pbc-acme` containers communicates: challenge, proof and certificate on a container with port 80 ([details in proxy server guide](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-zk-node.html#how-nginx-and-acme-run-as-services-in-docker-containers)).

Use the same commands as for baker logs

You can use the same docker commands, but remember to specify the name you used for the nginx docker container in your `docker-compose.yml`. Container name used in our `docker-compose.yml` template is `pbc-nginx`.

Find out if you have downloaded the SSL/TSL certificate (to limit logs to a recent period use `--since 1h` for last hour):

```
docker logs pbc-nginx | grep "Downloading cert."
```

When certificate is installed, it will be checked after one hour for renewal. Thereafter, it will be automatically renewed every 60 days.

Check if your SSL/TSL certificate has been renewed:

```
docker logs pbc-nginx | grep "renewal"
```

Check for the nginx container shutting down and restarting:

```
docker logs pbc-nginx | grep "starting nginx"
```

Note

It is a sign of a problem if restarts of your proxy server happen very frequently (not counting the restarts related to your automatic update schedule defined in your auto-update script). Read the log statements leading up to the shutdown and find out what happened.

### Confirm that your BYOC endpoints are working <a href="#confirm-that-your-byoc-endpoints-are-working" id="confirm-that-your-byoc-endpoints-are-working"></a>

Nodes must have working BYOC REST endpoints to participate in oracle service.

Bad or outdated endpoints cause serious problems

* Can make your price oracle node start a wrongful dispute causing slashing
* Two nodes with bad endpoints in a deposit or withdrawal oracle will crash the bridge

Check if your BYOC endpoints for other chains in config.json are working:

```
curl "ENDPOINT_YOU_WANT_TO_CHECK" \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{"method":"eth_chainId","params":[],"id":1,"jsonrpc":"2.0"}'
```

Alternatively:

```
curl -X POST "ENDPOINT_YOU_WANT_TO_CHECK" \
  -H 'Content-Type: application/json' \
  --data '{"method":"eth_blockNumber", "jsonrpc":"2.0", "params":[],"id":1}'
```

If the block number is way off, or if you don't get anything with either command, there is likely a problem with the endpoint, replace it!

#### Updating your BYOC chain configuration <a href="#updating-your-byoc-chain-configuration" id="updating-your-byoc-chain-configuration"></a>

The endpoints used for communicating with supported external blockchains are defined in the `config.json` file under the tag `chainConfigs`.

Updating your `config.json` should happen when:

* New external chains become supported
* Switching to a different endpoint supplier

Below is an example of a config with external chain endpoints.

<details>

<summary>Example config.json</summary>

```
```

</details>

To update a config, the easiest way is to simply run the node-register tool again:

```
./node-register.sh create-config
```

Alternatively, `config.json` can be edited manually as shown in the example above.

#### How to migrate your node to a different VPS <a href="#how-to-migrate-your-node-to-a-different-vps" id="how-to-migrate-your-node-to-a-different-vps"></a>

When changing VPS there are a few important precautions you take ensuring a problem free migration.

**You may never run two nodes performing baker services at the same time**\
Running two nodes with same config can be interpreted as malicious behavior. You can start a [reader node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-reader-node.html) on the new VPS. Then, when you are ready to change the `config.json` to the BP version, you stop the node from running on the old server:

```
docker stop  nameOfDockerContainer
```

**If you change host IP, you need to correct your `config.json`**\
In `config.json` correct the IPv4:

```
  "host": "PUBLIC_IP_OF_SERVER_HOSTING_THIS_NODE"
```

**You must migrate certain files for your node to participate in voting on a new committee (Large Oracle)**\
From the storage directory `/opt/pbc-mainnet/storage` of your old node host you move the 3 files below to the storage directory of the new server:

`large-oracle-backup-database.db`, `large-oracle-database.db` and `peers.json`

#### Install Network Time Protocol <a href="#install-network-time-protocol" id="install-network-time-protocol"></a>

To avoid time drift use Network Time Protocol (NTP). First install:

```
 sudo apt-get update
```

```
 sudo apt-get install ntp ntpdate
```

Stop NTP service and point to NTP server:

```
sudo service ntp stop
```

```
sudo ntpdate pool.ntp.org
```

Start NTP service and check status:

```
sudo service ntp start
```

```
sudo systemctl status ntp
```

### Keeping your node software up-to-date <a href="#keeping-your-node-software-up-to-date" id="keeping-your-node-software-up-to-date"></a>

New versions of the node software is released periodically. If you are running out-of-date node software you may not be able to join committees. To prevent this make sure to stay up-to-date with new node software versions.

To get automatic updates please follow the guide [here](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#get-automatic-updates).

You can also choose to manually update your node software by following the guide [here](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#updating-your-node-manually).

The node software version of your own node `nodeVersion` and the minimum required node software version to be part of the committee `minimumNodeVersion` can be found in the [Block producer orchestration contract state](https://browser.partisiablockchain.com/contracts/04203b77743ad0ca831df9430a6be515195733ad91?tab=state).

### Inactivity <a href="#inactivity" id="inactivity"></a>

If a confirmed baker node is unresponsive when a new committee is forming, it will be marked as `inactive`. Nodes that are marked as `inactive` will not be considered for future committees.

To mark your node as active again first make sure the node is up-to-date and working correctly, then use the following command in [node-register.sh tool](https://partisiablockchain.gitlab.io/documentation/node-operations/node-health-and-maintenance.html#the-node-registersh-tool):

```
./node-register.sh report-active
```

### Voting for Quarterly Rewards <a href="#voting-for-quarterly-rewards" id="voting-for-quarterly-rewards"></a>

Every quarter rewards are paid out to delegated stakers and node operators based on the amount of tokens staked and node performance.

As a node operator you must participate in the quarterly decentralized calculation and voting for the payout of the rewards to take place.

To calculate rewards for a quarter, and vote for the payout do the following:

1. Ensure you have the newest version of the `node-register.sh` script. You can download the newest version by running

```
curl https://gitlab.com/partisiablockchain/main/-/raw/main/scripts/node-register.sh --output node-register.sh
```

2. Run the reward calculation for the quarter on your own node by executing the following, where `rewards-quarter` is the index of the quarter since mainnet launch. Rewards quarter _1_ started on 1. june 2022 and ended on 31. aug 2022.

```
./node-register.sh compute-rewards <rewards-quarter>
```

3. The rewards calculation will use your nodes local data to calculate the rewards for the quarter. It takes some minutes to run. The command will output all the relevant rewards information into a directory in the storage directory of the node. It also outputs a hash value that is used when voting on-chain.
4. Vote for the payout of the rewards on-chain
   1. Go the the [Rewards Vote Contract](https://browser.partisiablockchain.com/contracts/02722d2a7b82a619303df8fd9e9248355f217513ef/vote)
   2. Sign in with the account that runs your node.
   3. Enter the hash and rewards quarter index and send the transaction by clicking "VOTE". 5000 gas is enough to send your vote.

When enough votes has been cast for the same rewards hash, the rewards will be paid out according to the calculated distribution.

You can consult the calculation method for rewards, and the history of quarterly payouts [here](https://gitlab.com/partisiablockchain/node-operators-rewards/-/blob/main/mainnet/README.md#computing-rewards).
