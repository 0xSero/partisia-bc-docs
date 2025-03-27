# Contract Development Patterns

This guide documents proven patterns and architectures for developing robust smart contracts on Partisia Blockchain. These patterns will help you design more maintainable, secure, and gas-efficient contracts.

## Foundational Patterns

### State Management Pattern

The state of a contract is central to its functionality. Partisia Blockchain contracts follow a functional paradigm where actions take the current state and return a new state.

```rust
#[state]
pub struct ContractState {
    // Core data that persists between transactions
    owner: Address,
    data: SortedVecMap<Address, DataStruct>,
    configuration: ConfigSettings,
}

#[action]
pub fn update_data(
    ctx: ContractContext,
    state: ContractState,
    new_value: DataStruct
) -> ContractState {
    // Create a modified copy of the state
    let mut new_state = state;
    new_state.data.insert(ctx.sender, new_value);
    new_state
}
```

**Best Practice**: Design your state struct carefully. Group related data into sub-structs to improve readability and maintainability.

### Access Control Pattern

Implement consistent access control mechanisms to protect contract functions:

```rust
impl ContractState {
    fn assert_owner(&self, caller: Address) {
        assert_eq!(
            caller, self.owner,
            "Only the owner can perform this action"
        );
    }
    
    fn assert_authorized(&self, caller: Address) {
        assert!(
            self.authorized_users.contains(&caller),
            "User not authorized"
        );
    }
}

#[action]
pub fn admin_function(ctx: ContractContext, mut state: ContractState) -> ContractState {
    // Verify permission first
    state.assert_owner(ctx.sender);
    
    // Perform privileged operation
    // ...
    
    state
}
```

**Advanced Pattern**: For more complex permission systems, consider implementing a role-based access control system similar to the example in the Access Control contract:

```rust
pub trait SecurityLevel: PartialOrd + Eq {
    const LOWEST_LEVEL: Self;
    const HIGHEST_LEVEL: Self;
}

pub struct AccessControlMap<SecurityLevelT: SecurityLevel> {
    map: SortedVecMap<Address, SecurityLevelT>,
}

impl<SecurityLevelT: SecurityLevel + Clone> AccessControlMap<SecurityLevelT> {
    pub fn get_user_level(&self, user: &Address) -> SecurityLevelT {
        self.map
            .get(user)
            .cloned()
            .unwrap_or(SecurityLevelT::LOWEST_LEVEL)
    }
    
    pub fn update_user_level(
        &mut self,
        sender: &Address,
        user: Address,
        new_level: SecurityLevelT,
    ) {
        // Implementation details...
    }
}
```

### Interacting with Other Contracts

#### Event Group Builder Pattern

Use the EventGroup builder to create structured interactions with other contracts:

```rust
#[action]
pub fn interact_with_token(
    ctx: ContractContext,
    state: ContractState,
    token_address: Address,
    amount: u128
) -> (ContractState, Vec<EventGroup>) {
    // Create an event group for the token contract
    let mut event_group = EventGroup::builder();
    
    // Call the token contract
    event_group.call(token_address, TRANSFER_SHORTNAME)
               .argument(ctx.sender)
               .argument(amount)
               .done();
    
    // Add a callback to be notified of the result
    event_group.with_callback(TRANSFER_CALLBACK)
               .with_cost(5000)
               .done();
    
    // Return state and events
    (state, vec![event_group.build()])
}
```

#### Callback Handling Pattern

Always implement proper callback handling for contract interactions:

```rust
#[callback(shortname = 0x01)]
pub fn transfer_callback(
    ctx: ContractContext,
    callback_ctx: CallbackContext,
    mut state: ContractState,
) -> (ContractState, Vec<EventGroup>) {
    // Check if the interaction succeeded
    if !callback_ctx.success {
        // Handle failure case
        return (state, vec![]);
    }
    
    // Process successful result
    if let Some(result) = callback_ctx.results.get(0) {
        if result.succeeded {
            let return_data = result.get_return_data::<ExpectedType>();
            // Update state based on the result
        }
    }
    
    (state, vec![])
}
```

### Data Structure Patterns

#### Efficient Collection Patterns

Choose the right collection types for your use case:

1. **SortedVecMap/SortedVecSet**: For small to medium-sized collections (dozens to hundreds of entries)
2. **AvlTreeMap**: For larger collections (hundreds to thousands of entries)

```rust
// For moderate number of entries with good serialization performance
pub struct ModerateState {
    users: SortedVecMap<Address, UserData>,
    tokens: SortedVecSet<TokenId>,
}

// For large number of entries
pub struct LargeState {
    users: AvlTreeMap<Address, UserData>,
    tokens: AvlTreeMap<TokenId, Unit>,
}
```

#### Gas-Optimized State Pattern

Design your state for minimum serialization/deserialization cost:

```rust
// Less efficient (many small maps)
pub struct IneffientState {
    user_names: SortedVecMap<Address, String>,
    user_balances: SortedVecMap<Address, u128>,
    user_roles: SortedVecMap<Address, Role>,
}

// More efficient (consolidated structure)
pub struct EfficientState {
    users: SortedVecMap<Address, UserProfile>,
}

pub struct UserProfile {
    name: String,
    balance: u128,
    role: Role,
}
```

### Zero Knowledge Contract Patterns

#### Secret Variable Management

Properly manage secret variables in [ZK contracts:](../zk-smart-contracts/)

```rust
#[zk_on_secret_input(shortname = 0x40)]
fn add_secret_data(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
) -> (
    ContractState,
    Vec<EventGroup>,
    ZkInputDef<SecretVarMetadata, SecretDataType>,
) {
    // Validate input constraints
    assert!(
        !already_inputted(&zk_state, context.sender),
        "Each address is only allowed to send one variable"
    );
    
    // Create input definition with metadata
    let input_def = ZkInputDef::with_metadata(
        Some(CALLBACK_SHORTNAME),
        SecretVarMetadata { /* metadata fields */ },
    );
    
    (state, vec![], input_def)
}
```

#### Zero Knowledge Computation Flow

Structure your ZK computations to maintain a clear flow:

1. **Input Phase**: Collect secret inputs from users
2. **Computation Phase**: Process secret data securely
3. **Output Phase**: Declassify only the minimum necessary results

```rust
// Start ZK computation
#[action(shortname = 0x01, zk = true)]
fn start_computation(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
) -> (ContractState, Vec<EventGroup>, Vec<ZkStateChange>) {
    // Verify preconditions
    assert_eq!(
        zk_state.calculation_state,
        CalculationStatus::Waiting,
        "Computation must start from Waiting state"
    );
    
    // Start the ZK computation
    (
        state,
        vec![],
        vec![zk_compute::computation_start(
            Some(COMPUTE_COMPLETE_SHORTNAME),
            &SecretVarMetadata { /* metadata */ },
        )],
    )
}

// Handle computation completion
#[zk_on_compute_complete(shortname = 0x42)]
fn computation_complete(
    context: ContractContext,
    state: ContractState,
    zk_state: ZkState<SecretVarMetadata>,
    output_variables: Vec<SecretVarId>,
) -> (ContractState, Vec<EventGroup>, Vec<ZkStateChange>) {
    // Open specified variables only
    (
        state,
        vec![],
        vec![ZkStateChange::OpenVariables {
            variables: output_variables,
        }],
    )
}
```

### Contract Standard Implementation Patterns

#### Token Contract Pattern (MPC-20)

Implement token contracts following the[ MPC-20 standard:](../introduction-to-contract-standards/mpc-20-token-contract.md)

```rust
#[action(shortname=0x01)]
pub fn transfer(
    context: ContractContext,
    mut state: TokenState,
    to: Address,
    amount: u128,
) -> TokenState {
    let from = context.sender;
    
    // Check balance
    let from_balance = state.balances.get(&from).unwrap_or(&0);
    assert!(*from_balance >= amount, "Insufficient balance");
    
    // Update balances
    state.balances.insert(from, from_balance - amount);
    let to_balance = state.balances.get(&to).unwrap_or(&0);
    state.balances.insert(to, to_balance + amount);
    
    state
}

#[action(shortname=0x03)]
pub fn transfer_from(/* ... */) -> TokenState {
    // Implementation
}

#[action(shortname=0x05)]
pub fn approve(/* ... */) -> TokenState {
    // Implementation
}
```

#### NFT Contract Pattern (MPC-721)

Structure NFT contracts based on the MPC-721 standard:

```rust
#[action(shortname=0x03)]
pub fn transfer_from(
    context: ContractContext,
    mut state: NftState,
    from: Address,
    to: Address,
    token_id: u128,
) -> NftState {
    // Authorization check
    assert!(
        from == context.sender || 
        state.is_approved_for_all(from, context.sender) ||
        state.get_approved(token_id) == Some(context.sender),
        "Not authorized to transfer"
    );
    
    // Ownership check
    assert_eq!(
        state.owners.get(&token_id),
        Some(&from),
        "From address is not the owner"
    );
    
    // Transfer the token
    state.owners.insert(token_id, to);
    state.token_approvals.remove(&token_id);
    
    state
}
```

### Security Patterns

#### Input Validation Pattern

Always validate inputs to prevent unexpected behavior:

```rust
#[action]
pub fn process_data(
    ctx: ContractContext,
    state: ContractState,
    data: Vec<u8>,
    parameters: Parameters,
) -> ContractState {
    // Validate inputs
    assert!(!data.is_empty(), "Data cannot be empty");
    assert!(parameters.value > 0, "Parameter value must be positive");
    assert!(
        parameters.option < MAX_OPTION,
        "Option exceeds maximum allowed value"
    );
    
    // Process with validated inputs
    // ...
}
```

#### Safe State Transition Pattern

Ensure that state transitions maintain invariants:

```rust
impl ContractState {
    fn validate_state_invariants(&self) -> bool {
        // Check invariants like:
        self.total_supply == self.calculate_actual_total() &&
        self.owner != Address::zero() &&
        // other invariants...
        true
    }
}

#[action]
pub fn complex_action(ctx: ContractContext, mut state: ContractState) -> ContractState {
    // Modify state
    // ...
    
    // Verify invariants haven't been broken
    assert!(
        state.validate_state_invariants(),
        "State invariants violated"
    );
    
    state
}
```

#### Reentrancy Protection Pattern

Although Partisia Blockchain contracts are inherently protected from reentrancy by design (functional execution model), still follow this pattern when making external calls:

1. Update internal state before making external calls
2. Verify state changes in callbacks

```rust
#[action]
pub fn safe_external_call(
    ctx: ContractContext,
    mut state: ContractState,
) -> (ContractState, Vec<EventGroup>) {
    // 1. Update internal state first
    state.in_progress = true;
    state.balances.insert(ctx.sender, new_balance);
    
    // 2. Then make external calls
    let event_group = EventGroup::builder()
        .call(external_contract, ACTION_SHORTNAME)
        .done()
        .build();
    
    (state, vec![event_group])
}
```

### Factory and Registry Patterns

#### Contract Factory Pattern

Create a factory contract to deploy and manage other contracts:

```rust
#[action]
pub fn deploy_new_contract(
    ctx: ContractContext,
    mut state: FactoryState,
    parameters: DeployParameters,
) -> (FactoryState, Vec<EventGroup>) {
    // Prepare deployment
    let mut event_group = EventGroup::builder();
    
    // Call system deploy contract
    event_group.call(DEPLOY_CONTRACT_ADDRESS, DEPLOY_SHORTNAME)
               .argument(contract_wasm)
               .argument(contract_abi)
               .argument(create_init_data(parameters))
               .done();
    
    // Add callback to track newly deployed contract
    event_group.with_callback(DEPLOY_CALLBACK_SHORTNAME)
               .with_cost(10000)
               .done();
    
    (state, vec![event_group.build()])
}

#[callback(shortname = 0x10)]
pub fn deploy_callback(
    ctx: ContractContext,
    callback_ctx: CallbackContext,
    mut state: FactoryState,
) -> (FactoryState, Vec<EventGroup>) {
    if callback_ctx.success {
        // Record deployed contract in the registry
        let contract_address = /* extract from callback */;
        state.deployed_contracts.push(contract_address);
    }
    
    (state, vec![])
}
```

#### DNS/Registry Pattern

Implement a registry to track contracts by domain names:

```rust
#[state]
pub struct DnsState {
    records: AvlTreeMap<String, DnsEntry>,
}

#[derive(CreateTypeSpec, ReadWriteState)]
pub struct DnsEntry {
    address: Address,
    owner: Address,
}

#[action(shortname = 0x01)]
pub fn register_domain(
    ctx: ContractContext,
    mut state: DnsState,
    domain: String,
    address: Address,
) -> DnsState {
    // Validate the domain isn't already taken
    assert!(
        state.records.get(&domain).is_none(),
        "Domain already registered"
    );
    
    // Register the new domain
    state.records.insert(domain, DnsEntry {
        address,
        owner: ctx.sender,
    });
    
    state
}
```

### Advanced Patterns

#### Event-Based Programming Pattern

Structure complex contracts around clear event flows:

```rust
#[action]
pub fn initiate_process(
    ctx: ContractContext,
    mut state: ContractState,
) -> (ContractState, Vec<EventGroup>) {
    // Record process initiation
    state.processes.insert(process_id, ProcessState::Started);
    
    // Create first step events
    let events = create_process_events(process_id);
    
    (state, events)
}

#[callback]
pub fn process_step_callback(
    ctx: ContractContext,
    callback_ctx: CallbackContext,
    mut state: ContractState,
) -> (ContractState, Vec<EventGroup>) {
    // Update process state
    let process_id = extract_process_id(callback_ctx);
    state.processes.insert(process_id, ProcessState::StepComplete);
    
    // Create next step events if needed
    let events = if more_steps_needed(process_id) {
        create_next_step_events(process_id)
    } else {
        state.processes.insert(process_id, ProcessState::Complete);
        vec![]
    };
    
    (state, events)
}
```

#### Proxy/Update Pattern

Build contracts that can be upgraded via proxy pattern:

```rust
#[state]
pub struct ProxyState {
    implementation: Address,
    owner: Address,
    // Other state variables
}

#[action]
pub fn forward_call(
    ctx: ContractContext,
    state: ProxyState,
    call_data: Vec<u8>,
) -> (ProxyState, Vec<EventGroup>) {
    // Forward the call to the implementation contract
    let mut event_group = EventGroup::builder();
    
    event_group.call(state.implementation, EXECUTE_SHORTNAME)
               .argument(call_data)
               .done();
    
    event_group.with_callback(FORWARD_CALLBACK)
               .done();
    
    (state, vec![event_group.build()])
}

#[action]
pub fn update_implementation(
    ctx: ContractContext,
    mut state: ProxyState,
    new_implementation: Address,
) -> ProxyState {
    // Only owner can update implementation
    assert_eq!(ctx.sender, state.owner, "Only owner can update");
    
    // Update implementation address
    state.implementation = new_implementation;
    
    state
}
```

### Optimization Best Practices

#### Minimize State Operations

Every state operation has a gas cost:

```rust
// Inefficient (multiple state lookups)
#[action]
pub fn inefficient_action(ctx: ContractContext, mut state: ContractState) -> ContractState {
    let user = state.users.get(&ctx.sender).unwrap();
    
    // First state access
    state.last_active.insert(ctx.sender, ctx.block_production_time);
    
    // Process something
    let result = process_data(user.data);
    
    // Second state access (could be combined with first)
    state.user_results.insert(ctx.sender, result);
    
    state
}

// Efficient (grouped state operations)
#[action]
pub fn efficient_action(ctx: ContractContext, mut state: ContractState) -> ContractState {
    let user = state.users.get(&ctx.sender).unwrap();
    
    // Process result
    let result = process_data(user.data);
    
    // Single grouped state update
    let user_state = UserState {
        last_active: ctx.block_production_time,
        result: result,
    };
    state.user_states.insert(ctx.sender, user_state);
    
    state
}
```

#### Batch Operations Where Possible

Batch related operations to reduce gas costs:

```rust
// Inefficient (many separate actions)
#[action]
pub fn add_single_item(ctx: ContractContext, mut state: ContractState, item: Item) -> ContractState {
    state.items.push(item);
    state
}

// Efficient (batch operation)
#[action]
pub fn add_multiple_items(
    ctx: ContractContext,
    mut state: ContractState,
    items: Vec<Item>,
) -> ContractState {
    for item in items {
        state.items.push(item);
    }
    state
}
```

### Reference Implementations

Study these reference implementations for common use cases:

1. **Token Contract**: See the [MPC-20 standard implementation](../introduction-to-contract-standards/mpc-20-token-contract.md)
2. **NFT Contract**: Refer to the [MPC-721 standard implementation](../introduction-to-contract-standards/mpc-721-nft-contract.md)
3. **DNS Contract**: A domain name registry system for contracts
4. **ZK Computing**: Sample zero-knowledge computation contracts like [ZK-Average-Salary](https://gitlab.com/partisiablockchain/language/example-contracts/-/tree/main/rust/zk-average-salary)
5. **Access Control**: Sophisticated permissions management
