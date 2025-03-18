# How to create a vote from a Smart Contract

### Case transparency in parliament - Voting record of MPs as a means to strengthen democracy and transparency <a href="#case-transparency-in-parliament-voting-record-of-mps-as-a-means-to-strengthen-democracy-and-transpar" id="case-transparency-in-parliament-voting-record-of-mps-as-a-means-to-strengthen-democracy-and-transpar"></a>

The newly founded republic of Faraway is plagued by corruption. To ensure transparency and public mandate behind the parliamentary process the voting record of the elected Members of Parliament (MPs) is added to the blockchain via smart contracts. Using smart contracts enables the public to see how MPs exercise their mandate. Laws that are passed through the smart contract are added to the immutable record, giving all citizens access to the official legal code.

**The setup of our scenario**

* The parliament has 197 MPs.
* Each MP has a key set. The public key enables the public to follow the MP's voting record on the blockchain. The private key is known only by the individual MP and is used to sign their vote.
* Each time the parliament votes on an issue they do it through a smart contract vote. Laws that passed are therefore also added to the immutable record.

**NB.** No ZK computation is necessary since the both individual votes and voting results are supposed to be public.

### How to program the voting smart contract in Rust <a href="#how-to-program-the-voting-smart-contract-in-rust" id="how-to-program-the-voting-smart-contract-in-rust"></a>

In the following the different parts of a smart contract implementing the voting scenario is explained.

If you need help with any of the rust concepts we recommend you visit [Rust intro book for general understanding](https://doc.rust-lang.org/std/index.html) or if you need help understanding Rust standard macros, or other specific keywords we recommend the [Rust Standard Library](https://doc.rust-lang.org/std/index.html) for specifications of these.

You can see the complete Rust source code of the [voting contract here](https://gitlab.com/partisiablockchain/language/example-contracts/-/blob/main/voting/src/lib.rs)

#### 1) Importing libraries <a href="#id-1-importing-libraries" id="id-1-importing-libraries"></a>

First we need to include a few libraries to get access to the functions and types needed for programming our smart contract. It is not necessary to understand exactly what the library includes here or what they do in order to create your first smart contract.

```rust
#![allow(unused_variables)]

#[macro_use]
extern crate pbc_contract_codegen;
extern crate pbc_contract_common;

use pbc_contract_common::address::Address;
use pbc_contract_common::context::ContractContext;
use pbc_contract_common::sorted_vec_map::SortedVecMap;
```

#### 2) Defining the contract state <a href="#id-2-defining-the-contract-state" id="id-2-defining-the-contract-state"></a>

When programming a smart contract we have to define the _state_ of the contract. We do this in Rust by creating a struct and marking it with [`#[state]`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.state.html).

For our voting contract the contract state has the following parts:

* `proposal_id` A proposal id, so you can identify what proposal the vote is concerned with.
* `mp_addresses` The list of people that are allowed to vote on the proposal. Only people with voting rights in the parliament should be allowed to vote, so the contract also needs a list of MP addresses.
* `votes` The actual votes cast are contained in a map pairing each person with their vote.
* `closed` Finally, when the voting ends people must no longer be able to change their vote. Therefore, the contract the state also includes information about whether the voting is open or closed.

We can also define methods associated with the state struct that read or write to the state. In Rust these methods are defined inside an `impl` block associated with the state struct.

For our voting contract we have defined two state methods:

* `register_vote` When an MP cast her vote, it is recorded in the votes map.
* `close_if_finished_vote` The voting process automatically closes after everybody has voted.

We could have chosen to make closing the vote depend on when the majority was reached, or to make an action where the chairman of the parliament closes the vote after some deadline. We could also add a result to the contract state if we want to make the contract more informative. The idea here was a simple voting record, so what we have now will suffice.

```rust
#[state]
pub struct VotingContractState {
  proposal_id: u64,
  mp_addresses: Vec<Address>,
  votes: SortedVecMap<Address, u8>,
  closed: u8,
}

impl VotingContractState {
  fn register_vote(&mut self, address: Address, vote: u8) {
    self.votes.insert(address, vote);
  }

  fn close_if_finished(&mut self) {
    if self.votes.len() == self.mp_addresses.len() {
      self.closed = 1;
    };
  }
}
```

#### 3) Defining the initialization of the contract <a href="#id-3-defining-the-initialization-of-the-contract" id="id-3-defining-the-initialization-of-the-contract"></a>

When deploying a smart contract on the blockchain the state of the contract has to be initialized properly.

When programming the smart contract in Rust we can define how the initialization takes place by creating a function and marking it with [`#[init]`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.init.html)

To initialize our voting contract the user deploying the contract has to supply the proposal id and the list of people eligible to vote. Our initialization code checks that the supplied input is valid (i.e. the list of people allowed to vote has at least one person and has no duplicates) and creates the initial state object for the contract from the input.

After successful initialization, the contract state becomes live on the blockchain.

```rust
#[init]
pub fn initialize(
  _ctx: ContractContext,
  proposal_id: u64,
  mp_addresses: Vec<Address>,
) -> VotingContractState {
  assert_ne!(mp_addresses.len(), 0, "Cannot start a poll without parliament members");
  let mut address_set = SortedVecSet::new();
  for mp_address in mp_addresses.iter() {
    address_set.insert(*mp_address);
  }
  assert_eq!(mp_addresses.len(), address_set.len(), "Duplicate MP address in input");

  VotingContractState {
    proposal_id,
    mp_addresses,
    votes: SortedVecMap::new(),
    closed: 0,
  }
}
```

#### 4) Defining the actions of the contract <a href="#id-4-defining-the-actions-of-the-contract" id="id-4-defining-the-actions-of-the-contract"></a>

When a smart contract is live on the blockchain, people can interact with the contract by creating a transaction that initiate execution of an _action_ of the smart contract.

The only way of changing the state of a smart contract is through the defined actions. The smart contract actions hereby define the exact conditions and rules for changing the contract state. A smart contract can have multiple defined actions.

In Rust, you define a smart contract action by coding a function and marking it with [`#[action]`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.action.html). The function receives the existing state and the inputs from the user initiating the action, and it can then produce and return a new updated state reflecting the changes.

For our voting contract the only action users can perform is to cast their vote. The code starts by checking that the voting is still open, that the sender is among the people allowed to vote and that the delivered vote is one of the allowed voting options. If the checks succeed the code creates a new state where the vote of the sender is registered in the vote map. Also, if this vote was the last one and everyone has now voted, then voting is closed.

```rust
#[action]
pub fn vote(context: ContractContext, state: VotingContractState, vote: u8) -> VotingContractState {
  assert_eq!(state.closed, 0, "The poll is closed");
  assert!(state.mp_addresses.contains(&context.sender), "Only members of the parliament can vote");
  assert!(vote == 0 || vote == 1, "Only \"yes\" and \"no\" votes are allowed");

  let mut new_state = state;
  new_state.register_vote(context.sender, vote);
  new_state.close_if_finished();
  new_state
}
```

### Building and testing the voting contract <a href="#building-and-testing-the-voting-contract" id="building-and-testing-the-voting-contract"></a>

The process for building and testing the voting contract is the same as any other contract that needs to be deployed, we recommend you follow our general guide [here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/compile-and-deploy-contracts.html) to get your smart contract onto our testnet.
