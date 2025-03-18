# Programmers Guide to smart contracts

This page is a quick overview of the software development kit for Partisia Blockchain.

### Smart Contract Overview <a href="#smart-contract-overview" id="smart-contract-overview"></a>

A smart contract on Partisia Blockchain consists of, on a surface level, some state, and some actions defined to either operate on the state and/or to interact with other contracts.

Consider for example a basic public voting contract:

> **State:**
>
> * What are we voting on? (if applicable)
> * Who are allowed to vote?
> * Deadline (if applicable)
> * Who have voted?
> * Are we done yet?
> * What is the result if we are done?
>
> **Actions:**
>
> * Voters should be able to vote.
> * Anybody should be able to retrieve how many votes have been cast, and whether the vote is complete yet.
> * Anybody should be able to retrieve the result of the vote.
>
> **Initializer:**
>
> * Vote subject, Voters and Deadline are all permanent attributes of the vote, and so should be set in the initializer.

Contracts' state and actions must be declared in an [Contract ABI file](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#abi-binary-format); a concise description of the contract's interface and internal state representation, that must be uploaded together with the contract code when initializing the contract. Without an ABI file, it might be impossible for the dashboard and other contracts to interact with your contract.

The PBC compiler is capable of automatically producing an ABI for your contract, along with state and RPC serialization code for your actions.

### Macros <a href="#macros" id="macros"></a>

Smart contract elements can be declared using these [macros](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/index.html#partisia-blockchain-sdk-macros):

* [`#[state]`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.state.html) declares how the contract represents its state.
* [`#[action]`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.action.html) declares an endpoint that the contract can be interacted with by.
* [`#[init]`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.init.html) declares the code to run when the contract is initialized.
* [`#[callback]`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.callback.html) declares the code to run after a corresponding `action` has been called.

#### `#[state]` <a href="#state" id="state"></a>

Declares that the annotated struct is the top level of the contract state. This macro must occur exactly once in any given contract.

Example:

```rust
#[state]
pub struct VotingContractState {
    proposal_id: u64,
    mp_addresses: Vec<Address>,
    votes: SortedVecMap<Address, u8>,
    closed: u8,
}
```

This macro implicitly derives the [ReadWriteState trait](https://partisiablockchain.gitlab.io/language/contract-sdk/read_write_state_derive/derive.ReadWriteState.html) for the struct. The `ReadWriteState` derive may fail if any of the state struct's fields aren't `impl ReadWriteState`.

Further reading: [state macro documentation](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.state.html)

#### `#[action]` <a href="#action" id="action"></a>

Declares that the annotated function is a contract action that can be called from other contracts and dashboard. Must have a signature of the following format:

```rust
#[action]
pub fn action_internal_name(
  context: ContractContext,
  state: ContractState,
  ... // RPC arguments.
) -> (ContractState, Vec<EventGroup>)
```

The action receives the previous state, along with a context, and the declared arguments, and must return the new state, along with a vector of EventGroup; a list of interactions with other contracts.

Example:

```rust
#[action]
pub fn vote(
    context: ContractContext,
    state: VotingContractState,
    vote: u8,
) -> (VotingContractState, Vec<EventGroup>) {
  // Code...
}
```

Further reading: [action macro documentation](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.action.html)

**A note on functional contracts**

Contracts are _functional_: Each interaction point, whether `init` or `action` take some input, and return some output. Interactions cannot produce side effects, visible or not. The state will thus not be changed should a transaction fail while running the contract code, whether due to panics or insufficient gas.

Of further interest here, is that the entire contract is essentially "reset" after every interaction. Any [`static mut` items](https://doc.rust-lang.org/reference/items/static-items.html) will possess their initial value, once again. The only state your contract can possess is the state returned from interactions.

#### `#[init]` <a href="#init" id="init"></a>

Similar to `#[action]` macro, but declares how the contract can be initialized.

```rust
#[init]
pub fn initialize(
    context: ContractContext,
    ... // Initialization arguments
) -> (VotingContractState, Vec<EventGroup>) {
  // Code...
}
```

Note that there are no previous context when initializing, in contrast to the `action` macro. All arguments will be required during initialization. This macro must occur exactly once in any given contract. If the initializer fails the contract will not be created.

Example:

```rust
#[init]
pub fn initialize(
    context: ContractContext,
    proposal_id: u64,
    mp_addresses: Vec<Address>,
) -> (VotingContractState, Vec<EventGroup>) {
  // Code...
}
```

Further reading: [init macro documentation](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.init.html)

#### `#[callback]` <a href="#callback" id="callback"></a>

Has to be linked with an `#[action]` macro in order to be called. In practice this is done by combining the implicitly created constant of the corresponding callback-annotated function with a call to an event group builder's with\_callback() function.

```rust
#[action]
pub fn call_bob(
    context: ContractContext,
    state: ContractState,
    address: Address
) -> (ContractState, Vec<EventGroup>) {
    // 0x42 being the shortname of the action to call at 'contract' Bob
    let SHORTNAME_CALL_BOB: Shortname = Shortname::from_be_bytes(&[0x42]).unwrap();
    let mut event_group_builder = EventGroup::builder();

    event_group_builder.call(address, SHORTNAME_CALL_BOB)
                       .done();

    // Ask for callback. Notice that the constant provided has not been explicitly created.
    event_group_builder.with_callback(SHORTNAME_CALL_BOB_CALLBACK)
                       .argument(some_argument)
                       .with_cost(10000)
                       .done()

    (state, vec![event_group_builder.build()])
}

#[callback(shortname = 13)]
pub fn call_bob_callback(
  contract_context: ContractContext,
  callback_context: CallbackContext,
  state: ContractState,
  // ... RPC arguments
) -> (ContractState, Vec<EventGroup>) {
  assert!(callback_context.success, "Bob failed to reply");

  // Code to run after executing the external action 0x42...
}
```

Further reading: [callback macro documentation](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.callback.html)

### Traits <a href="#traits" id="traits"></a>

The compiler exposes [traits](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_traits/index.html) that provides serialization methods. These traits are important for the operation of PBC contracts, but should rarely be implemented manually; prefer using the built-in derive methods.

* ReadWriteState: Serialization for [State serialization format](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#state-binary-format).
* ReadWriteRPC: Serialization for [RPC argument serialization format](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#rpc-binary-format).
* CreateTypeSpec: Serialization for [ABI serialization format](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#abi-binary-format).

### Data Structures <a href="#data-structures" id="data-structures"></a>

#### Address <a href="#address" id="address"></a>

[`Address`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_common/address/struct.Address.html) represents an address on the blockchain; it has a subfield indicating the type of the address (account, system contract, public contract or zk contract).

#### ContractContext <a href="#contractcontext" id="contractcontext"></a>

[`ContractContext`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_common/context/index.html#structs) is available from every action, and contains some useful context information for the current transaction:

* `Address` of the contract itself and the caller; the person or contract that caused the interaction.
* The current block number, and time since some epoch.
* Hashes of both the current transaction and the previous transaction.

#### Events <a href="#events" id="events"></a>

Partisia Blockchain's contract interaction model sandboxes each contract, and allows RPC calls as the primary form of interaction. As each transaction is entirely isolated, RPCs can only occur "between" action calls.

For example:

* Zeno calls Alice in transaction 1: Alice determines she needs some information from Bob, and exits while telling the blockchain: "Call Bob for me, I want a reply, and let me pay for the reply"
* Alice calls Bob in transaction 2: Bob performs it's computation, and exists with "Here's the reply to Alice, she said she'd pay for this reply".
* Bob calls Alice in transaction 3: Alice finally got the necessary information to perform her own computation...

To accommodate this model, the compiler requires each `action` annotated function to return a (possibly empty) `Vec` of `EventGroup`s, which represents the "Call X for me" information.

Each `EventGroup` consists of one or more interactions (representing "Call X for me",) with the possibility of callbacks (representing "I want a reply") or return data (representing the optional data to be passed along with a reply) but never both at the same time. All interactions in an `EventGroup` shares gas costs uniformly.

To see how Bob might pass along some return data, see the following example:

```mustache
#[action(shortname=0x42)]
pub fn provide_information_for_alice(
  context: ContractContext,
  state: ContractState
) -> (ContractState, Vec<EventGroup>) {
  let mut event_group_builder = EventGroup::builder();
  event_group_builder.return_data(state.information);

  (state, vec![event_group_builder.build()])
}
```

The information in `state.information` will be serialized and passed along to Alice's `CallbackContext`, where she may retrieve it for whatever use-case she has with it:

```rust
#[callback(shortname = 13)]
pub fn call_bob_callback(
  contract_context: ContractContext,
  callback_context: CallbackContext,
  state: ContractState,
  // ... RPC arguments
) -> (ContractState, Vec<EventGroup>) {
  assert!(callback_context.success, "Bob failed to reply");

  // Index into the information related to the first event
  let first_event = callback_context.results[0];

  // Provide the expected type of the returned data and return it.
  let bobs_information = first_event.get_return_data::<expected_type>();

  // Do something with Bob's information...
}
```

Further reading: [events module documentation](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_common/events/index.html)

#### CallbackContext <a href="#callbackcontext" id="callbackcontext"></a>

The additional context struct that all callback-annotated functions receive as a parameter. It holds information about all events in the `EventGroup` linked to the callback. This information denotes whether _all_ events successfully executed, the individual execution success-status of each event and optional return data linked to the event.

Further reading: [CallbackContext struct documentation](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_common/context/struct.CallbackContext.html)

### State serialization gas considerations <a href="#state-serialization-gas-considerations" id="state-serialization-gas-considerations"></a>

`SortedVecMap<T>` and `SortedVecSet<T>` (specifically for [CopySerializable](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#CopySerializable) `T`) allow for efficiently (de)serializable map and set structures, both in terms of gas and computation time. Remember that (de)serialization gas costs must be paid for _every_ action, even ones that never handle state.

Ensure you [depend upon and link against the `pbc_lib` crate](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_lib/index.html). This _should_ automatically lower gas costs.

#### Cheap memcpy (or _why_ you should definitely link `pbc_lib`) <a href="#cheap-memcpy-or-why-you-should-definitely-link-pbc_lib" id="cheap-memcpy-or-why-you-should-definitely-link-pbc_lib"></a>

The compiler exposes cheaper (in terms of gas) versions of the `memcpy` and `memmove` functions. These functions are commonly used for copying bytes around directly, but are (thankfully) rarely manually used in Rust, though they may still occur in compiled programs due to lower-level libraries and compiler optimizations. Your compiled contracts will be using `memcpy` for (de)serialization, hence why the compiler defines these alternatives.

The `pbc_lib` crate overwrites these functions with versions that directly interact with the PBC WASM Interpreter to trigger the interpreter's built-in support for quickly copying data around.

Further reading: [`pbc_lib` crate documentation](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_lib/index.html)
