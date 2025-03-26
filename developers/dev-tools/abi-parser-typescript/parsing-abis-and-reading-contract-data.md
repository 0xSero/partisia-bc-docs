# Parsing ABIs and Reading Contract Data

### Parsing ABI Files

The first step in working with a Partisia Blockchain smart contract is to parse its ABI (Application Binary Interface). The ABI defines the structure of the contract, including its state, functions, and data types.

#### Basic Parsing

```typescript
import { AbiParser } from "@partisiablockchain/abi-client";

// Assuming you have the ABI buffer from a file or API
const buffer: Buffer = /* ABI buffer */;

// Parse the ABI
const parser = new AbiParser(buffer);
const fileAbi = parser.parseAbi();

// Get the contract ABI
const contractAbi = fileAbi.contract();
```

#### Inspecting Contract Structure

Once you have parsed the ABI, you can inspect the contract structure:

```typescript
// Get contract functions
const functions = contractAbi.contractInvocations;

// Get the state structure
const stateStruct = contractAbi.getStateStruct();

// Find a specific function by name
const myFunction = contractAbi.getFunctionByName("transfer");

// Find a specific type
const myType = contractAbi.getNamedType("TokenBalance");
```

#### Understanding Contract Functions

Each function in a contract is represented by a `ContractInvocation` object:

```typescript
// Print function information
function printFunction(func) {
  console.log(`Function: ${func.name}`);
  console.log(`Kind: ${func.kind.name}`);
  console.log("Arguments:");
  func.arguments.forEach(arg => {
    console.log(`  ${arg.name}: ${TypeIndex[arg.type.typeIndex]}`);
  });
}
```

### Reading Contract Data

The ABI Parser provides tools for reading data from a contract's state or from RPC calls.

#### Reading State Data

To read contract state data, you'll use the `BigEndianReader` or specialized readers:

```typescript
import { BigEndianReader } from "@partisiablockchain/abi-client";

// Assuming you have the state binary data
const stateData: Buffer = /* state data */;

// Create a reader
const reader = new BigEndianReader(stateData, contractAbi);

// Read the state according to the contract's state type
const state = reader.read(contractAbi.stateType);
```

#### Working with ScValue Objects

After reading, you'll get `ScValue` objects that represent the contract's data:

```typescript
// Accessing fields in a struct
function accessStruct(struct: ScValueStruct) {
  // Get a field by name
  const balance = struct.getFieldValue("balance");
  
  // Convert to a specific type
  if (balance) {
    const amount = balance.asNumber(); // If it's a number
    // or
    const amountBN = balance.asBN(); // If it's a big number
  }
}

// Working with vectors
function workWithVector(vector: ScValueVector) {
  const size = vector.size();
  
  // Iterate over elements
  for (let i = 0; i < size; i++) {
    const element = vector.get(i);
    // Process element...
  }
  
  // If it's a vector of u8 (bytes), you can get a Buffer
  if (vector.get(0)?.getType() === TypeIndex.u8) {
    const buffer = vector.vecU8Value();
  }
}

// Working with maps
function workWithMap(map: ScValueMap) {
  // Get value by key
  const key = /* some ScValue */;
  const value = map.get(key);
  
  // Check if empty
  const isEmpty = map.isEmpty();
}
```

#### Reading RPC Data

For reading RPC data (like function call results), use the `RpcReader`:

```typescript
import { RpcReader } from "@partisiablockchain/abi-client";

// Assuming you have RPC response data
const rpcData: Buffer = /* RPC data */;

// Create RPC reader
const reader = RpcReader.create(rpcData, fileAbi, TransactionKind.Action);

// Read the RPC value
const rpcValue = reader.readRpc();

// Access arguments
const args = rpcValue.arguments();
```

### Reading Transactions

For reading transaction data, use the `TransactionReader`:

```typescript
import { TransactionReader } from "@partisiablockchain/abi-client";

// Assuming you have transaction payload data
const txData: Buffer = /* Transaction data */;

// Deserialize transaction payload
const txStruct = TransactionReader.deserializeTransactionPayload(txData);
```

### Practical Example: Reading a Token Balance

Here's a complete example of reading a token balance from a contract state:

```typescript
import { AbiParser, BigEndianReader, ScValueStruct } from "@partisiablockchain/abi-client";

async function getTokenBalance(contractAddress: string, userAddress: string) {
  // 1. Get the contract ABI
  const abiResponse = await fetch(`/api/contract/${contractAddress}/abi`);
  const abiBuffer = Buffer.from(await abiResponse.arrayBuffer());
  const fileAbi = new AbiParser(abiBuffer).parseAbi();
  const contractAbi = fileAbi.contract();
  
  // 2. Get the contract state
  const stateResponse = await fetch(`/api/contract/${contractAddress}/state`);
  const stateBuffer = Buffer.from(await stateResponse.arrayBuffer());
  
  // 3. Read the state
  const reader = new BigEndianReader(stateBuffer, contractAbi);
  const state = reader.read(contractAbi.stateType) as ScValueStruct;
  
  // 4. Access the balances map
  const balances = state.getFieldValue("balances")?.mapValue();
  if (!balances) {
    return 0;
  }
  
  // 5. Create an address key
  const addressKey = new ScValueAddress(Buffer.from(userAddress, 'hex'));
  
  // 6. Get the balance
  const balance = balances.get(addressKey);
  
  // 7. Return the balance as a number or BN depending on the token's precision
  return balance ? balance.asNumber() : 0;
}
```

This example demonstrates the complete flow from parsing an ABI to reading specific data from a contract's state.
