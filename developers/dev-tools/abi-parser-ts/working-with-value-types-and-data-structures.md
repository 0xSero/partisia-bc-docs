# Working with Value Types and Data Structures

### Introduction to Value Types

The Partisia Blockchain ABI client uses a hierarchy of `ScValue` classes to represent different value types from smart contracts. Understanding these types is crucial for correctly reading and writing contract data.

### Basic Value Types

#### Numbers

Numbers are represented by the `ScValueNumber` class:

```typescript
import { ScValueNumber, TypeIndex } from "@partisiablockchain/abi-client";

// Creating a small number
const u8Value = new ScValueNumber(TypeIndex.u8, 42);

// Creating a large number
import BN from "bn.js";
const u128Value = new ScValueNumber(TypeIndex.u128, new BN("123456789012345678901234567890"));

// Converting to JavaScript types
const number = u8Value.asNumber(); // Returns 42
const bigNum = u128Value.asBN();   // Returns BN instance
```

#### Boolean

Boolean values use `ScValueBool`:

```typescript
import { ScValueBool } from "@partisiablockchain/abi-client";

const boolValue = new ScValueBool(true);
const value = boolValue.boolValue(); // Returns true
```

#### String

Strings use `ScValueString`:

```typescript
import { ScValueString } from "@partisiablockchain/abi-client";

const stringValue = new ScValueString("Hello, world!");
const value = stringValue.stringValue(); // Returns "Hello, world!"
```

### Blockchain-Specific Types

The library supports blockchain-specific types like addresses, hashes, and keys:

#### Address

```typescript
import { ScValueAddress } from "@partisiablockchain/abi-client";

// Create from a 21-byte buffer
const addressValue = new ScValueAddress(addressBuffer);

// Get the buffer
const buffer = addressValue.value;

// Convert to hex string
const hexString = buffer.toString("hex");
```

#### Hash

```typescript
import { ScValueHash } from "@partisiablockchain/abi-client";

// Create from a 32-byte buffer
const hashValue = new ScValueHash(hashBuffer);
```

#### Public Key and Signature

```typescript
import { ScValuePublicKey, ScValueSignature } from "@partisiablockchain/abi-client";

// Create from buffers
const publicKeyValue = new ScValuePublicKey(publicKeyBuffer); // 33 bytes
const signatureValue = new ScValueSignature(signatureBuffer);  // 65 bytes
```

### Complex Data Structures

#### Structures

Structs are represented by `ScValueStruct`:

```typescript
import { ScValueStruct, ScValueNumber, ScValueString, TypeIndex } from "@partisiablockchain/abi-client";

// Create a map of field values
const fields = new Map();
fields.set("id", new ScValueNumber(TypeIndex.u32, 123));
fields.set("name", new ScValueString("John Doe"));

// Create the struct
const personStruct = new ScValueStruct("Person", fields);

// Access fields
const id = personStruct.getFieldValue("id")?.asNumber();
const name = personStruct.getFieldValue("name")?.stringValue();
```

#### Enumerations

Enums are represented by `ScValueEnum`:

```typescript
import { ScValueEnum, ScValueStruct } from "@partisiablockchain/abi-client";

// First create the variant struct
const variantFields = new Map();
variantFields.set("amount", new ScValueNumber(TypeIndex.u64, 1000));
const transferVariant = new ScValueStruct("Transfer", variantFields);

// Create the enum
const actionEnum = new ScValueEnum("Action", transferVariant);

// Access the variant
const variant = actionEnum.item;
const variantName = variant.name; // "Transfer"
const amount = variant.getFieldValue("amount")?.asNumber();
```

#### Vectors

Vectors (arrays) use `ScValueVector`:

```typescript
import { ScValueVector, ScValueNumber, TypeIndex } from "@partisiablockchain/abi-client";

// Create a vector of numbers
const numbers = [
  new ScValueNumber(TypeIndex.u32, 1),
  new ScValueNumber(TypeIndex.u32, 2),
  new ScValueNumber(TypeIndex.u32, 3)
];
const vector = new ScValueVector(numbers);

// Access elements
const size = vector.size();
const element = vector.get(1); // Second element
const allValues = vector.values();

// Special case: byte arrays
const byteVector = ScValueVector.fromBytes(Buffer.from([1, 2, 3, 4]));
const buffer = byteVector.vecU8Value(); // Convert back to Buffer
```

#### Maps

Maps are represented using `ScValueMap` and the `HashMap` class:

```typescript
import { ScValueMap, HashMap, ScValueString, ScValueNumber, TypeIndex } from "@partisiablockchain/abi-client";

// Create key-value pairs
const entries = [
  [new ScValueString("key1"), new ScValueNumber(TypeIndex.u32, 100)],
  [new ScValueString("key2"), new ScValueNumber(TypeIndex.u32, 200)]
];

// Create a HashMap
const hashMap = new HashMap(entries);

// Create a map value
const mapValue = new ScValueMap(hashMap);

// Access values
const value = mapValue.get(new ScValueString("key1"))?.asNumber();
const size = mapValue.size();
const isEmpty = mapValue.isEmpty();
```

#### Options

Options (nullable values) use `ScValueOption`:

```typescript
import { ScValueOption, ScValueNumber, TypeIndex } from "@partisiablockchain/abi-client";

// Some value
const someOption = new ScValueOption(new ScValueNumber(TypeIndex.u32, 42));

// None value
const noneOption = new ScValueOption(null);

// Check if has value
const hasSome = someOption.isSome(); // true
const hasNone = noneOption.isSome(); // false

// Access value (safely)
someOption.valueOrUndefined(value => value.asNumber()); // 42
noneOption.valueOrUndefined(value => value.asNumber()); // undefined

// Direct access (be careful)
const innerValue = someOption.innerValue; // ScValueNumber or null
```

### Type Conversion and Validation

When working with values, it's important to check types and convert correctly:

```typescript
function processValue(value: ScValue) {
  // Check the type
  const typeIndex = value.getType();
  
  if (typeIndex === TypeIndex.u32) {
    // It's a u32 number
    return value.asNumber();
  } else if (typeIndex === TypeIndex.String) {
    // It's a string
    return value.stringValue();
  } else if (typeIndex === TypeIndex.Named) {
    // It could be a struct or enum
    if (value instanceof ScValueStruct) {
      // It's a struct
      return processStruct(value);
    } else if (value instanceof ScValueEnum) {
      // It's an enum
      return processEnum(value);
    }
  }
  
  throw new Error(`Unsupported type: ${TypeIndex[typeIndex]}`);
}
```

### JSON Conversion

The ABI client provides `JsonValueConverter` to convert between `ScValue` objects and JSON:

```typescript
import { JsonValueConverter } from "@partisiablockchain/abi-client";

// Convert ScValue to JSON
const jsonValue = JsonValueConverter.toJson(scValue);

// Convert ScValue to JSON string
const jsonString = JsonValueConverter.toJsonString(scValue);
```

This is particularly useful for debugging or displaying contract data in user interfaces.

### Working with AvlTreeMap

The `ScValueAvlTreeMap` represents on-chain AVL tree maps:

```typescript
import { ScValueAvlTreeMap, HashMap } from "@partisiablockchain/abi-client";

// Create an AVL tree map with a tree ID and a HashMap
const treeId = 1;
const hashMap = new HashMap(entries);
const avlTreeMap = new ScValueAvlTreeMap(treeId, hashMap);

// Or without pre-loaded data
const emptyAvlTreeMap = new ScValueAvlTreeMap(treeId, null);

// Access the tree ID
const id = avlTreeMap.treeId;

// Map to a different format
const mappedEntries = avlTreeMap.mapKeysValues(
  key => key.stringValue(),
  value => value.asNumber()
);
```

### Reading a State Struct Example

Here's a complete example of reading a complex state struct from a contract:

```typescript
import { 
  AbiParser, 
  BigEndianReader, 
  ScValueStruct,
  ScValueMap,
  ScValueVector,
  ScValueAddress,
  TypeIndex
} from "@partisiablockchain/abi-client";

async function readTokenContract(contractAddress: string) {
  // 1. Get the ABI
  const abiResponse = await fetch(`/api/contract/${contractAddress}/abi`);
  const abiBuffer = Buffer.from(await abiResponse.arrayBuffer());
  const fileAbi = new AbiParser(abiBuffer).parseAbi();
  const contractAbi = fileAbi.contract();
  
  // 2. Get the state
  const stateResponse = await fetch(`/api/contract/${contractAddress}/state`);
  const stateBuffer = Buffer.from(await stateResponse.arrayBuffer());
  
  // 3. Read the state
  const reader = new BigEndianReader(stateBuffer, contractAbi);
  const state = reader.read(contractAbi.stateType) as ScValueStruct;
  
  // 4. Process the token contract state
  const tokenInfo = {
    name: state.getFieldValue("name")?.stringValue() || "",
    symbol: state.getFieldValue("symbol")?.stringValue() || "",
    decimals: state.getFieldValue("decimals")?.asNumber() || 0,
    totalSupply: state.getFieldValue("total_supply")?.asBN().toString() || "0"
  };
  
  // 5. Get balances (assuming it's a map of address -> amount)
  const balances = state.getFieldValue("balances")?.mapValue();
  const balanceMap = new Map<string, string>();
  
  if (balances && balances.map) {
    balances.map.forEach((value, key) => {
      if (key.getType() === TypeIndex.Address) {
        const addressHex = (key as ScValueAddress).value.toString("hex");
        const amount = value.asBN().toString();
        balanceMap.set(addressHex, amount);
      }
    });
  }
  
  return {
    tokenInfo,
    balances: Object.fromEntries(balanceMap)
  };
}
```

### Best Practices

1. **Always Check Types**: Before attempting to convert an `ScValue` to a specific type, check its type or use the appropriate instance check.
2. **Handle Nulls**: When working with optional values or map lookups, always check for null/undefined values.
3. **Use BN for Large Numbers**: For u64, i64, u128, i128, and u256 types, always use the `BN` class rather than JavaScript numbers to avoid precision loss.
4. **Validate Inputs**: When creating blockchain-specific types like addresses or hashes, ensure the input buffers have the correct length.
5. **Prefer High-Level Methods**: Use methods like `valueOrUndefined` for options and `mapKeysValues` for maps rather than direct access where possible.

By understanding these value types and their operations, you can effectively work with complex data structures in Partisia Blockchain smart contracts.
