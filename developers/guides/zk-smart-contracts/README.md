# ZK Smart Contracts

One of the main features which set Partisia Blockchain(PBC) apart from other blockchains is that PBC supports zero knowledge computations, notably secure multiparty computation (MPC). You can utilize PBC's capacity for ZK computations through zero knowledge smart contracts (ZKSC).

#### What is zero knowledge smart contracts <a href="#what-is-zero-knowledge-smart-contracts" id="what-is-zero-knowledge-smart-contracts"></a>

Zero knowledge smart contracts has all the same functionality as public smart contracts (SC), but in addition to that the ZKSC allocate a subset of qualified nodes to do computations on private versions of input data. A ZKSC stipulate some non-public actions in addition to the public actions. This means that a ZKSC has a private state only present on the ZK nodes in addition to a public state on the chain. When a ZK node is allocated to a specific ZKSC the node's associated stakes will be locked to the contract until the ZK work is finished. If our ZKSC is an auction like below, the public state will contain the winner's ID and the price of the auctioned item, whereas the private state will contain all the non-winning bids. The calculation in the private state are done on [secret shared data](https://medium.com/partisia-blockchain/mpc-techniques-series-part-1-secret-sharing-d8f98324674a). This means that the nodes allocated for the ZK work in the contract does not have access to the user input, i.e. the ZK nodes do not have access to the values of the non-winning bids.

#### Example of a zero knowledge smart contract on PBC - Vickrey Auction <a href="#example-of-a-zero-knowledge-smart-contract-on-pbc-vickrey-auction" id="example-of-a-zero-knowledge-smart-contract-on-pbc-vickrey-auction"></a>

In this example we will use [our contract](https://gitlab.com/partisiablockchain/language/example-contracts/-/tree/main/zk-second-price-auction?ref_type=head) that creates a [Vickrey Auction (second price auction)](https://en.wikipedia.org/wiki/Vickrey_auction), which is a sealed bid auction where the winner is the person with the highest bid (as in a normal auction), you can read more about this type of contract if you visit our example contract overview [here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-examples.html#zk-second-price-auction). The second price auction takes as inputs the bids from the registered participants. The bids are delivered encrypted and secret-shared to the ZK nodes allocated to the contract. When the computation is initiated by the contract owner, the zero knowledge computation nodes reads the collected input and then create a bit vector consisting of prices and the ordering number. The list of bit vectors is now sorted in MPC. The winner is the first entry (the bidder with the highest price-bid), the price is determined by the size of the second-highest bid.

The contract follows these phases:

1. Initialization on the blockchain.
2. Registration of bidders allowed to participate in the auction.
3. Receival of secret bids.
4. Once enough bids have been received, the owner of the contract can initialize computation of the auction result.
5. The ZK nodes derive the winning bid in a secure manner by executing a Secure Multiparty Computation protocol off-chain.
6. Once the ZK computation concludes, the winning bid will be published and the winner will be stored in the state, together with their bid.
7. The ZK nodes sign the result of the auction with a digital signature proving that the nodes in question were responsible for the generating the result of the auction.

Below you can see the Rust implementation of the [zero knowledge smart contract](https://gitlab.com/partisiablockchain/language/example-contracts/-/blob/main/zk-second-price-auction/src/zk_compute.rs?ref_type=heads) for the Vickrey Auction:

The function _zk\_compute_ performs a ZK computation on the secret shared bids (step 5 above), it returns the index of the highest bidder and amount of second-highest bid.

```rust
use crate::zk_lib::{num_secret_variables, sbi32_from, sbi32_input, sbi32_metadata, Sbi32};

pub fn zk_compute() -> (Sbi32, Sbi32) {
    // Initialize state
    let mut highest_bidder: Sbi32 = sbi32_from(sbi32_metadata(1));
    let mut highest_amount: Sbi32 = sbi32_from(0);
    let mut second_highest_amount: Sbi32 = sbi32_from(0);

    // Determine max
    for variable_id in 1..(num_secret_variables() + 1) {
        if sbi32_input(variable_id) > highest_amount {
            second_highest_amount = highest_amount;
            highest_amount = sbi32_input(variable_id);
            highest_bidder = sbi32_from(sbi32_metadata(variable_id));
        } else if sbi32_input(variable_id) > second_highest_amount {
            second_highest_amount = sbi32_input(variable_id);
        }
    }

    // Return highest bidder index, and second highest amount
    (highest_bidder, second_highest_amount)
}
```

[You can see the code handling the remaining contract phases further down the page](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/zk-smart-contracts.html#full-zksc-code-example)

#### Use zero knowledge smart contracts on PBC as a second layer <a href="#use-zero-knowledge-smart-contracts-on-pbc-as-a-second-layer" id="use-zero-knowledge-smart-contracts-on-pbc-as-a-second-layer"></a>

It is possible to use a zero knowledge smart contracts on PBC as a second layer for other blockchains. If we want to do a secret bid second price auction like above, we need to deploy two smart contracts: one zero knowledge smart contract on PBC and a public smart contract on the layer one chain. The public functionality of the contracts will be very similar. But the contract on PBC will privately calculate the result of the auction using zero knowledge computation.

![Diagram1](https://partisiablockchain.gitlab.io/documentation/smart-contracts/img/zk-smart-contracts-00.png)

The contract owner controls the functions on the Zero knowledge smart contract on PBC, but the functions of the layer one public contract are open for all users. Naturally only the winner on PBC can successfully claim the win on the contract located on the layer one BC. You can read more about this on our page on [pbc as a second layer](https://partisiablockchain.gitlab.io/documentation/smart-contracts/pbc-as-second-layer/partisia-blockchain-as-second-layer.html)

#### Full ZKSC code example <a href="#full-zksc-code-example" id="full-zksc-code-example"></a>

```rust
#![allow(unused_variables)]

#[macro_use]
extern crate pbc_contract_codegen;
extern crate pbc_contract_common;
extern crate pbc_lib;

use create_type_spec_derive::CreateTypeSpec;
use pbc_contract_common::address::Address;
use pbc_contract_common::context::ContractContext;
use pbc_contract_common::events::EventGroup;
use pbc_contract_common::zk::{
    AttestationId, CalculationStatus, SecretVarId, ZkInputDef, ZkState, ZkStateChange,
};
use pbc_traits::{ReadWriteRPC, ReadWriteState};
use read_write_rpc_derive::ReadWriteRPC;
use read_write_state_derive::ReadWriteState;

/// Id of a contract bidder.
#[repr(transparent)]
#[derive(PartialEq, ReadWriteRPC, ReadWriteState, Debug, Clone, Copy, CreateTypeSpec)]
#[non_exhaustive]
struct BidderId {
    id: i32,
}

/// Secret variable metadata. Contains unique ID of the bidder.
#[derive(ReadWriteState, ReadWriteRPC, Debug)]
struct SecretVarMetadata {
    bidder_id: BidderId,
}

/// The size of the MPC bid input variables.
const BITLENGTH_OF_SECRET_BID_VARIABLES: [u32; 1] = [32];

/// Number of bids required before starting auction computation.
const MIN_NUM_BIDDERS: u32 = 3;

/// Type of tracking bid amount
type BidAmount = i32;

/// This state of the contract.
#[state]
struct ContractState {
    /// Owner of the contract
    owner: Address,
    /// Registered bidders - only registered bidders are allowed to bid.
    registered_bidders: Vec<RegisteredBidder>,
    /// The auction result
    auction_result: Option<AuctionResult>,
}

#[derive(Clone, ReadWriteState, CreateTypeSpec, ReadWriteRPC)]
struct AuctionResult {
    /// Bidder id of the auction winner
    winner: BidderId,
    /// The winning bid
    second_highest_bid: BidAmount,
}

/// Representation of a registered bidder with an address
#[derive(Clone, ReadWriteState, CreateTypeSpec)]
struct RegisteredBidder {
    bidder_id: BidderId,
    address: Address,
}

/// Initializes contract
///
/// Note that owner is set to whoever initializes the contact.
#[init]
fn initialize(context: ContractContext, zk_state: ZkState<SecretVarMetadata>) -> ContractState {
    ContractState {
        owner: context.sender,
        registered_bidders: Vec::new(),
        auction_result: None,
    }
}

/// Registers a bidder with an address and updates the state accordingly.
////
/// Ensures that only the owner of the contract is able to register bidders.
#[action(shortname = 0x30)]
fn register_bidder(
    context: ContractContext,
    mut state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
    bidder_id: i32,
    address: Address,
) -> ContractState {
    let bidder_id = BidderId { id: bidder_id };

    assert_eq!(
        context.sender, state.owner,
        "Only the owner can register bidders"
    );

    assert!(
        state
            .registered_bidders
            .iter()
            .all(|x| x.address != address),
        "Duplicate bidder address: {:?}",
        address,
    );

    assert!(
        state
            .registered_bidders
            .iter()
            .all(|x| x.bidder_id != bidder_id),
        "Duplicate bidder id: {:?}",
        bidder_id,
    );

    state
        .registered_bidders
        .push(RegisteredBidder { bidder_id, address });

    state
}

/// Adds another bid variable to the ZkState.
///
/// The ZkInputDef encodes that variables should have size [`BITLENGTH_OF_SECRET_BID_VARIABLES`].
#[zk_on_secret_input(shortname = 0x40)]
fn add_bid(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
) -> (
    ContractState,
    Vec<EventGroup>,
    ZkInputDef<SecretVarMetadata>,
) {
    let bidder_info = state
        .registered_bidders
        .iter()
        .find(|x| x.address == context.sender);

    let bidder_info = match bidder_info {
        Some(bidder_info) => bidder_info,
        None => panic!("{:?} is not a registered bidder", context.sender),
    };

    // Assert that only one bid is placed per bidder
    assert!(
        zk_state
            .secret_variables
            .iter()
            .chain(zk_state.pending_inputs.iter())
            .all(|v| v.owner != context.sender),
        "Each bidder is only allowed to send one bid. : {:?}",
        bidder_info.bidder_id,
    );

    let input_def = ZkInputDef {
        seal: false,
        metadata: SecretVarMetadata {
            bidder_id: bidder_info.bidder_id,
        },
        expected_bit_lengths: BITLENGTH_OF_SECRET_BID_VARIABLES.to_vec(),
    };

    (state, vec![], input_def)
}

/// Allows the owner of the contract to start the computation, computing the winner of the auction.
///
/// The second price auction computation is beyond this call, involving several ZK computation steps.
#[action(shortname = 0x01)]
fn compute_winner(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
) -> (ContractState, Vec<EventGroup>, Vec<ZkStateChange>) {
    assert_eq!(
        zk_state.calculation_state,
        CalculationStatus::Waiting,
        "Computation must start from Waiting state, but was {:?}",
        zk_state.calculation_state,
    );
    assert_eq!(
        zk_state.data_attestations.len(),
        0,
        "Auction must have exactly zero data_attestations at this point"
    );

    assert_eq!(
        context.sender, state.owner,
        "Only contract owner can start the auction"
    );
    let amount_of_bidders = zk_state.secret_variables.len() as u32;

    assert!(
        amount_of_bidders >= MIN_NUM_BIDDERS,
        "At least {} bidders must have submitted bids for the auction to start",
        MIN_NUM_BIDDERS
    );

    (
        state,
        vec![],
        vec![ZkStateChange::start_computation(vec![
            SecretVarMetadata {
                bidder_id: BidderId { id: -1 },
            },
            SecretVarMetadata {
                bidder_id: BidderId { id: -1 },
            },
        ])],
    )
}

/// Automatically called when the computation is completed
///
/// The only thing we do is instantly open/declassify the output variables.
#[zk_on_compute_complete]
fn auction_compute_complete(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
    output_variables: Vec<SecretVarId>,
) -> (ContractState, Vec<EventGroup>, Vec<ZkStateChange>) {
    assert_eq!(
        zk_state.data_attestations.len(),
        0,
        "Auction must have exactly zero data_attestations at this point"
    );
    (
        state,
        vec![],
        vec![ZkStateChange::OpenVariables {
            variables: output_variables,
        }],
    )
}

/// Automatically called when the auction result is declassified. Updates state to contain result,
/// and requests attestation from nodes.
#[zk_on_variables_opened]
fn open_auction_variable(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
    opened_variables: Vec<SecretVarId>,
) -> (ContractState, Vec<EventGroup>, Vec<ZkStateChange>) {
    assert_eq!(
        opened_variables.len(),
        2,
        "Unexpected number of output variables"
    );
    assert_eq!(
        zk_state.data_attestations.len(),
        0,
        "Auction must have exactly zero data_attestations at this point"
    );

    let auction_result = AuctionResult {
        winner: read_variable(&zk_state, opened_variables.get(0)),
        second_highest_bid: read_variable(&zk_state, opened_variables.get(1)),
    };

    let attest_request = ZkStateChange::Attest {
        data_to_attest: serialize_as_big_endian(&auction_result),
    };

    (state, vec![], vec![attest_request])
}

/// Automatically called when some data is attested
#[zk_on_attestation_complete]
fn auction_results_attested(
    context: ContractContext,
    mut state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
    attestation_id: AttestationId,
) -> (ContractState, Vec<EventGroup>, Vec<ZkStateChange>) {
    assert_eq!(
        zk_state.data_attestations.len(),
        1,
        "Auction must have exactly one attestation"
    );
    let attestation = zk_state.get_attestation(attestation_id).unwrap();

    assert_eq!(attestation.signatures.len(), 4, "Must have four signatures");

    let auction_result = AuctionResult::rpc_read_from(&mut attestation.data.as_slice());

    state.auction_result = Some(auction_result);

    (state, vec![], vec![ZkStateChange::ContractDone])
}

/// Writes some value as RPC data.
fn serialize_as_big_endian<T: ReadWriteRPC>(it: &T) -> Vec<u8> {
    let mut output: Vec<u8> = vec![];
    it.rpc_write_to(&mut output).expect("Could not serialize");
    output
}

/// Reads a variable's data as some state value
fn read_variable<T: ReadWriteState>(
    zk_state: &ZkState<SecretVarMetadata>,
    variable_id: Option<&SecretVarId>,
) -> T {
    let variable_id = *variable_id.unwrap();
    let variable = zk_state.get_variable(variable_id).unwrap();
    let buffer: Vec<u8> = variable.data.clone().unwrap();
    T::state_read_from(&mut buffer.as_slice())
}
```
