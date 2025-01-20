# Transaction Binary Format

This is the specification for transactions, which includes signed transactions from users and executable events, which are spawned by the blockchain.

You can learn more about how interactions work on the blockchain [here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-interactions-on-the-blockchain.html#simple-interaction-model).

### Signed Transaction <a href="#signed-transaction" id="signed-transaction"></a>

A signed transaction is an instruction from a user containing information used to change the state of the blockchain. After constructing a binary signed transaction it can be delivered to any baker node in the network through their [REST API](https://partisiablockchain.gitlab.io/documentation/rest).

The following is the specification of the binary format of signed transactions.

The easiest way of creating a binary signed transaction is by using one of the available [client libraries](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#client). This specification can help you if you want to make your own implementation, for instance if you are targeting another programming language.

[**SignedTransaction**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#signedtransaction)

::= {

signature: [Signature](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#signature)\
transaction: [Transaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#transaction)

}

\


[**Signature**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#signature)

::= {

recoveryId: 0xnn

valueR: 0xnn×32

(big-endian)

valueS: 0xnn×32

(big-endian)

}

The [Signature](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#signature) includes:

* A recovery id between 0 and 3 used to recover the public key when verifying the signature
* The r value of the ECDSA signature
* The s value of the ECDSA signature

[**Transaction**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#transaction)

::= {

nonce: 0xnn×8

(big-endian)

validToTime: 0xnn×8

(big-endian)

gasCost: 0xnn×8

(big-endian)

address: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
rpc: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

The [Transaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#transactionouter) includes:

* The signer's [nonce](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#nonce)
* A unix time that the transaction is valid to
* The amount of [gas](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/transaction-gas-prices.html) allocated to executing the transaction
* The address of the smart contract that is the target of the transaction
* The rpc payload of the transaction. See [Smart Contract Binary Formats](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html) for a way to build the rpc payload.

#### Creating the signature <a href="#creating-the-signature" id="creating-the-signature"></a>

The signature is an ECDSA signature, using secp256k1 as the curve, on a sha256 hash of the transaction and the chain ID of the blockchain.

[**ToBeHashed**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#tobehashed)

::= transaction:[Transaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#transaction) chainId:[ChainId](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainid)

[**ChainId**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainid)

::= [String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)

The `chain id` is a unique identifier for the blockchain. For example, the chain id for Partisia Blockchain mainnet is `Partisia Blockchain` and the chain id for the testnet is `Partisia Blockchain Testnet`.

### Executable Event Binary Format <a href="#executable-event-binary-format" id="executable-event-binary-format"></a>

An executable event is spawned by the blockchain, and not by a user. It can be created from other events or when the blockchain receives a signed transaction. The executable event contains the information about which action to perform to a smart contract.

The following is the specification of the binary format of executable events.

[**ExecutableEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#executableevent)

::= {

originShard: [Option](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#optiont)<[String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)>\
transaction: [EventTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#eventtransaction)

}

The originShard is an [Option](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#optiont)<[String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)>, the originating shard of the event.

[**EventTransaction**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#eventtransaction)

::= {

originatingTransaction: [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash)\
inner: [InnerEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innerevent)\
shardRoute: [ShardRoute](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#shardroute)\
committeeId: [Long](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#long)\
governanceVersion: [Long](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#long)\
height: [Byte](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#byte)\
returnEnvelope: [Option](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#optiont)<[ReturnEnvelope](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#returnenvelope)>

}

The transaction, [EventTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#eventtransaction), includes:

* The originating transaction: the [SignedTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#signedtransaction) initiating the tree of events that this event is a part of.
* The event type, [InnerEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innerevent).
* The [ShardRoute](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#shardroute) describes which shard the event should be executed on.
* The id of the [committee](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#committee) that produced the block where the event was spawned. The committee id is used to find the correct committee for validating a finalization proof sent from one shard to another. The committee id of an event must be larger than the latest committee id.
* The [governance](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/governance-system-smart-contracts-overview.html) version when this event was produced. The governance version is incremented when the global state is changes, e.g. when updating a plugin. The governance version of the event must be up-to-date with the local governance version.
* The height of the event tree, which is increased for each event spawned after a signed transaction.
* If there is a callback registered to the event, the result of the event is returned in a [ReturnEnvelope](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#returnenvelope).

[**ShardRoute**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#shardroute)

::= {

targetShard: [Option](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#optiont)<[String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)>\
nonce: [Long](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#long)

}

A [ShardRoute](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#shardroute), the unique routing to a shard, contains the id of a target shard and a nonce.

**Event Types**

[**InnerEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innerevent)

::=

0x00 [InnerTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innertransaction)

\| 0x01 [CallbackToContract](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#callbacktocontract)

\| 0x02 [InnerSystemEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innersystemevent)

\| 0x03 [SyncEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#syncevent)

The [InnerEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innerevent) of an [EventTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#eventtransaction) is divided into four types of events: [InnerTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innertransaction), [CallbackToContract](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#callbacktocontract), [InnerSystemEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innersystemevent) and [SyncEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#syncevent).

#### Inner Transaction <a href="#inner-transaction" id="inner-transaction"></a>

A transaction is sent by the user or the contract, meaning that it represents the actions the user can activate. Transactions can either deploy contracts or interact with them. Each transaction also carries an associated sender and an associated cost.

[**InnerTransaction**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innertransaction)

::= {

from: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
cost: [Long](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#long)\
transaction: [Transaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#transaction)

}

The [InnerTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innertransaction) contains an associated sender and an associated cost, as well as the transaction.

[**Transaction**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#transaction)

::=

0x00 => [CreateContractTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createcontracttransaction)

\| 0x01 => [InteractWithContractTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#interactwithcontracttransaction)

A [Transaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#transaction) is either a [CreateContractTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createcontracttransaction) used to deploy contracts, or an [InteractWithContractTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#interactwithcontracttransaction) used to interact with contracts.

**Create Contract Transaction**

[**CreateContractTransaction**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createcontracttransaction)

::= {

address: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
binderJar: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)\
contractJar: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)\
abi: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)\
rpc: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

A [CreateContractTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createcontracttransaction) contains the [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address) where the contract should be deployed, and the binding jar, the contract jar, the contract ABI and the RPC of the create-invocation.

**Interact with Contract Transaction**

[**InteractWithContractTransaction**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#interactwithcontracttransaction)

::= {

contractId: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
payload: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

An [InteractWithContractTransaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#interactwithcontracttransaction) contains the [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address) of the contract to interact with, as well as a payload (RPC) defining the interaction.

#### Callback to Contract <a href="#callback-to-contract" id="callback-to-contract"></a>

Callback transactions call the callback methods in contracts.

[**CallbackToContract**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#callbacktocontract)

::= {

address: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
callbackIdentifier: [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash)\
from: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
cost: [Long](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#long)\
callbackRpc: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

A [CallbackToContract](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#callbacktocontract) contains the [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address) of the contract to target, as well as information about the callback: The event transactions that has been awaited and is completed, the sender, the allocated cost for the callback event, and the callback RPC.

#### Inner System Event <a href="#inner-system-event" id="inner-system-event"></a>

System events can manipulate the system state of the blockchain. These are primarily sent by system/governance contracts.

[**InnerSystemEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innersystemevent)

::= {

systemEventType: [SystemEventType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#systemeventtype)

}

An [InnerSystemEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#innersystemevent) has a [SystemEventType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#systemeventtype).

[**SystemEventType**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#systemeventtype)

::=

0x00 [CreateAccountEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createaccountevent)

\| 0x01 [CheckExistenceEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#checkexistenceevent)

\| 0x02 [SetFeatureEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#setfeatureevent)\


\| 0x03 [UpdateLocalPluginStateEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatelocalpluginstateevent)\


\| 0x04 [UpdateGlobalPluginStateEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updateglobalpluginstateevent)\


\| 0x05 [UpdatePluginEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatepluginevent)\


\| 0x06 [CallbackEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#callbackevent)\


\| 0x07 [CreateShardEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createshardevent)\


\| 0x08 [RemoveShardEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#removeshardevent)\


\| 0x09 [UpdateContextFreePluginState](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatecontextfreepluginstate)\


\| 0x0A [UpgradeSystemContractEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#upgradesystemcontractevent)\


\| 0x0B [RemoveContract](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#removecontract)

**Create Account Event**

[**CreateAccountEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createaccountevent)

::= {

toCreate: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)

}

A [CreateAccountEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createaccountevent) contains the address of the account to create.

**Check Existence Event**

[**CheckExistenceEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#checkexistenceevent)

::= {

contractOrAccountAddress: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)

}

A [CheckExistenceEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#checkexistenceevent) is an event for checking the existence of a contract or account address.

**Set Feature Event**

[**SetFeatureEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#setfeatureevent)

::= {

key: [String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)\
value: [Option](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#optiont)<[String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)>

}

A [SetFeatureEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#setfeatureevent) is an event for setting a feature in the state. The key indicates the feature to set, and the value is the value to set on the feature.

**Update Local Plugin State Event**

[**UpdateLocalPluginStateEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatelocalpluginstateevent)

::= {

type: [ChainPluginType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype)\
update: [LocalPluginStateUpdate](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#localpluginstateupdate)

}

A [UpdateLocalPluginStateEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatelocalpluginstateevent) updates the local state of the plugin, and its fields are a plugin type [ChainPluginType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype) and a plugin state [LocalPluginStateUpdate](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#localpluginstateupdate).

[**ChainPluginType**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype)

::=

0x00 => Account

\| 0x01 => Consensus

\| 0x02 => Routing

\| 0x03 => SharedObjectStore

A [ChainPluginType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype) can have the following types:

* The **Account** plugin controls additional information about account and fees.
* The **Consensus** plugin validates block proposals and finalizes correct blocks.
* The **Routing plugin** selects the right shard for any transaction.
* The **SharedObjectStore** plugin stores binary objects across all shards.

[**LocalPluginStateUpdate**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#localpluginstateupdate)

::= {

context: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
rpc: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

A [LocalPluginStateUpdate](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#localpluginstateupdate) contains the [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address) of the contract where the state should be updated, as well as RPC with the update.

**Update Global Plugin State Event**

[**UpdateGlobalPluginStateEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updateglobalpluginstateevent)

::= {

type: [ChainPluginType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype)\
update: [GlobalPluginStateUpdate](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#globalpluginstateupdate)

}

A [UpdateGlobalPluginStateEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updateglobalpluginstateevent) contains a chain plugin type [ChainPluginType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype) and a state update [GlobalPluginStateUpdate](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#globalpluginstateupdate).

[**GlobalPluginStateUpdate**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#globalpluginstateupdate)

::= {

rpc: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

The [GlobalPluginStateUpdate](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#globalpluginstateupdate) contains the RPC of the update to execute globally.

**Update Plugin Event**

[**UpdatePluginEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatepluginevent)

::= {

type: [ChainPluginType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype)\
jar: [Option](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#optiont)<[DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)>\
rpc: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

An [UpdatePluginEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatepluginevent) constructs a new event for updating an active plugin. It takes the type of the chain plugin to update, the jar to use as plugin, and RPC of global state migration.

**Callback Event**

[**CallbackEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#callbackevent)

::= {

returnEnvelope: [ReturnEnvelope](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#returnenvelope)\
completedTransaction: [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash)\
success: [Boolean](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#boolean)\
returnValue: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

A [CallbackEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#callbackevent) is the callback sent when a transaction has been executed. It contains the following fields:

* A [ReturnEnvelope](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#returnenvelope) with the destination for the callback.
* A [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash) of the transaction which has been executed.
* A [Boolean](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#boolean) indicating whether the transaction was successful.
* The return value of the callback event.

**Create Shard Event**

[**CreateShardEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createshardevent)

::= {

shardId: [String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)

}

A [CreateShardEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#createshardevent) is an event for creating a shard. The shardId is the id of the shard to create.

**Remove Shard Event**

[**RemoveShardEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#removeshardevent)

::= {

shardId: [String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)

}

A [RemoveShardEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#removeshardevent) is an event for removing a shard. The shardId is the id of the shard to remove.

**Update Context Free Plugin State**

[**UpdateContextFreePluginState**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatecontextfreepluginstate)

::= {

type: [ChainPluginType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype)\
rpc: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

An [UpdateContextFreePluginState](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#updatecontextfreepluginstate) updates the local state of a plugin without context on a named shard. Its fields are the type of plugin to update, i.e. [ChainPluginType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#chainplugintype), and the local state migration RPC.

**Upgrade System Contract Event**

[**UpgradeSystemContractEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#upgradesystemcontractevent)

::= {

contractAddress: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
binderJar: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)\
contractJar: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)\
abi: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)\
rpc: [DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

}

An [UpgradeSystemContractEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#upgradesystemcontractevent) upgrades a contract. It contains the contract address, the binder jar, the contract jar, the contract ABI, and the upgrade RPC.

**Remove Contract**

[**RemoveContract**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#removecontract)

::= {

contractAddress: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)

}

The [RemoveContract](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#removecontract) event removes an existing contract from the state. The contract to remove is specified by the [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address) of the contract.

#### Sync Event <a href="#sync-event" id="sync-event"></a>

A sync event is used for moving information from one shard to another when changing the shard configuration. That is, when adding or removing shards or when changing routing logic.

[**SyncEvent**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#syncevent)

::= {

accounts: [List](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#listt)<[AccountTransfer](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#accounttransfer)>\
contracts: [List](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#listt)<[ContractTransfer](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#contracttransfer)>\
stateStorage: [List](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#listt)<[DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)>

}

The [SyncEvent](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#syncevent) contains a lists of account transfer objects, contract transfer objects and serialized data of stored states.

[**AccountTransfer**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#accounttransfer)

::= {

address: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
accountStateHash: [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash)\
pluginStateHash: [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash)

}

An [AccountTransfer](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#accounttransfer) consists of an account [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address), a [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash) of the account state and a [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash) of the plugin state.

[**ContractTransfer**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#contracttransfer)

::= {

address: [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\
contractStateHash: [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash)\
pluginStateHash: [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash)

}

An [ContractTransfer](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#contracttransfer) consists of a contract [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address), a [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash) of the contract state and a [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash) of the plugin state.

### Common Types <a href="#common-types" id="common-types"></a>

**Address Type**

An [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address) is encoded as 21 bytes. The first byte is the [AddressType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#addresstype), and the remaining 20 bytes are the address identifier.

[**Address**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)

::= addressType:[AddressType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#addresstype) identifier:0xnn×20

(identifier is big-endian)

An [AddressType](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#addresstype) can either be an account address, a system address, public address, zk address or a government address.

[**AddressType**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#addresstype)

::=

0x00 => Account

\| 0x01 => System

\| 0x02 => Public

\| 0x03 => Zk

\| 0x04 => Gov

**Boolean Type**

A [Boolean](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#boolean) is encoded as a single byte, which is either 0 (false) or non-0 (true).

[**Boolean**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#boolean)

::= b:0xnn

(false if b==0, true otherwise)

**Byte Type**

A [Byte](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#byte) is encoded as a byte corresponding to its value.

[**Byte**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#byte)

::= 0xnn\


**Dynamic Bytes**

[DynamicBytes](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes) are encoded as a length, len, of 4 bytes, and a payload of len bytes. Both len and payload are encoded in big-endian order.

[**DynamicBytes**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#dynamicbytes)

::= len:0xnn×4 payload:0xnn×len

(big-endian)

**Hash Type**

A [Hash](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash) is encoded as 32 bytes in big-endian order.

[**Hash**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#hash)

::= 0xnn×32

(big-endian)

**List Type**

A [List\<T>](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#listt) of len elements of type **T** are encoded as the len (4 bytes, big-endian order), and the len elements are encoded according to their type **T**.

[**List\<T>**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#listt)

::= len:0xnn×4 elems:T×len

(len is big-endian)

**Long Type**

A [Long](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#long) is encoded as 8 bytes in big-endian order.

[**Long**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#long)

::= 0xnn×8

(big-endian)

**Option Type**

An [Option\<T>](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#optiont) is encoded as either one 0-byte if **T** is None, or as a non-zero byte and the encoding of **T** according to its type.

[**Option\<T>**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#optiont)

::=

0x00 => None

\| b:0xnn t:T => Some(t)

(b != 0)

**Return Envelope**

A [ReturnEnvelope](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#returnenvelope) is encoded as an [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address).

[**ReturnEnvelope**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#returnenvelope)

::= [Address](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#address)\


**String Type**

A [String](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string) is encoded as its length, len (4 bytes), and the UTF-8 of len bytes. Both are encoded in big-endian order.

[**String**](https://partisiablockchain.gitlab.io/documentation/smart-contracts/transaction-binary-format.html#string)

::= len:0xnn×4 uft8:0xnn×len

(big-endian)
