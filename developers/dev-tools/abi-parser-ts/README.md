# ABI Parser - Typescript

## Partisia Blockchain ABI Parser

### Introduction

The Partisia Blockchain ABI Parser is a TypeScript library for interacting with Partisia Blockchain smart contracts. It provides tools for parsing Application Binary Interface (ABI) files, which define the structure of smart contracts, including their state, functions, and argument types.

This library is essential for developers building applications that need to interact with Partisia Blockchain smart contracts, as it provides the necessary tooling to read and write data in the format expected by the blockchain.

### Key Components

The ABI Parser toolkit consists of several key components that work together:

#### 1. ABI Parser

The `AbiParser` class is the core component responsible for parsing binary ABI files into structured JavaScript objects. It reads the binary representation of a contract's interface and converts it into a more developer-friendly format.

```typescript
import { AbiParser } from "@partisiablockchain/abi-client";

// Parse ABI from a binary buffer
const fileAbi = new AbiParser(buffer).parseAbi();
```

#### 2. Contract Interface Objects

After parsing, you get access to contract interface objects like:

* `ContractAbi`: Represents the full contract interface
* `StructTypeSpec`: Defines structures used in the contract
* `EnumTypeSpec`: Defines enumerations used in the contract
* `ContractInvocation`: Represents contract functions
* `BinderInvocation`: Represents binder functions (low-level interactions)

#### 3. Value Representation

The library includes classes for representing all the value types supported by Partisia Blockchain:

* Basic types: `ScValueBool`, `ScValueNumber`, `ScValueString`, etc.
* Complex types: `ScValueStruct`, `ScValueEnum`, `ScValueVector`, etc.
* Blockchain-specific types: `ScValueAddress`, `ScValueHash`, `ScValuePublicKey`, etc.

#### 4. Readers

Various reader classes help deserialize binary data:

* `BigEndianReader`: Handles standard binary data reading
* `RpcReader`: Specializes in reading RPC (Remote Procedure Call) data
* `TransactionReader`: Reads transaction data

#### 5. Builders

Builder classes help construct binary data for interacting with contracts:

* `RpcBuilder`: Constructs RPC calls
* `TransactionBuilder`: Builds transactions
* `AbstractBuilder`: Base class for all builders

### Architecture

The architecture of the ABI Parser follows these general principles:

1. **Binary Data Handling**: The library is built around the concept of converting between binary data (used by the blockchain) and TypeScript objects (used by developers).
2. **Type Safety**: The library makes extensive use of TypeScript's type system to provide compile-time safety when working with smart contracts.
3. **Modular Design**: Components are separated into distinct modules with clear responsibilities.
4. **Builder Pattern**: Many components use the builder pattern to make constructing complex objects more intuitive.

### Version Compatibility

The ABI Parser supports different versions of the Partisia Blockchain ABI format, with configuration options to handle various versions. The `Configuration` class manages these options:

```typescript
const config = Configuration.fromClientVersion(abiVersion, shortnameLength);
```

### Next Steps

In the following sections, we'll explore:

1. How to parse ABI files and inspect contract structures
2. Reading and writing contract state data
3. Building and executing transactions
4. Working with different value types

This documentation aims to provide both a high-level understanding of the library's architecture and specific examples for common tasks.
