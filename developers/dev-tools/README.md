# Dev Tools

This article introduces tools designed to help developers working with smart contracts on the blockchain. These tools offer a range of capabilities, from testing smart contracts to facilitating command-line interactions with the blockchain, and even integrating the blockchain with your own applications.\
\
**Development Workflow Integration**

The following flowchart illustrates how Partisia Blockchain's various development tools integrate into the typical development workflow:

#### Tool Selection Guide

| Development Phase         | Recommended Tools                                                                | Purpose                                 |
| ------------------------- | -------------------------------------------------------------------------------- | --------------------------------------- |
| **Contract Development**  | <p>- Rust/Cargo<br>- VS Code + rust-analyzer<br>- <code>cargo pbc</code> CLI</p> | Write and compile smart contracts       |
| **Local Testing**         | <p>- <code>cargo pbc</code> test commands<br>- JUnit test framework</p>          | Verify contract logic and functionality |
| **Deployment**            | <p>- <code>cargo pbc transaction deploy</code><br>- Browser interface</p>        | Deploy contracts to testnet/mainnet     |
| **Frontend Integration**  | <p>- ABI codegen<br>- TypeScript SDK<br>- React components</p>                   | Build user interfaces for contracts     |
| **Backend Integration**   | <p>- Java SDK<br>- ABI client</p>                                                | Create server-side integration          |
| **Wallet Integration**    | <p>- Wallet SDK<br>- MetaMask Snap</p>                                           | Add authentication and transactions     |
| **Monitoring & Analysis** | <p>- Browser explorer<br>- REST API</p>                                          | Track contract activity and state       |

### Partisia Blockchain browser <a href="#partisia-blockchain-browser" id="partisia-blockchain-browser"></a>

Partisia blockchain browser is a web-based interface which translates the blockchains data into a user-friendly application. Partisia blockchain browser is essentially a complete UI for Partisia Blockchain.

Partisia blockchain browser can be used to:

* Display details for the entire ledger: blocks and transactions.
* Display details for all accounts and smart contracts.
* Interact with any smart contract.
* Deploy your own new smart contracts.
* Manage your own account, including your MPC tokens and becoming a Node Operator.
* Create local references of contracts and accounts to help you keep track of already deployed contracts.

Partisia Blockchain browser can be used with the Testnet and the Mainnet:

* [Testnet](https://browser.testnet.partisiablockchain.com)
* [Mainnet](https://browser.partisiablockchain.com)

### Command-line tools <a href="#command-line-tools" id="command-line-tools"></a>

[`cargo pbc`](https://crates.io/crates/cargo-partisia-contract) is an umbrella for multiple sub-tools. The tools assist you in interacting with the blockchain and working with smart contracts. These tools are thoroughly documented when using them within `cargo pbc`, enabling you to explore their capabilities inside `cargo pbc`. Below are a short description and use case for each of these sub-tools.

Pre requisite to use any cargo PBC commands

If you want to use any of the command-line tools or below commands you need to install [the smart contract compiler](https://partisiablockchain.gitlab.io/documentation/smart-contracts/install-the-smart-contract-compiler.html).

#### The Compiler `build` <a href="#the-compiler-build" id="the-compiler-build"></a>

This is a primary part of developing smart contracts. The `build` command compiles [rust smart contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/compile-and-deploy-contracts.html) and [ZK Rust smart contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/compile-and-deploy-zk-contract.html). It compiles and returns `.abi` file and a `.wasm` for rust contracts or `.zkwa` for ZK rust contracts.

#### Blockchain interaction <a href="#blockchain-interaction" id="blockchain-interaction"></a>

To interact with the blockchain you can use the command line tool: `cargo pbc`. There are 3 main commands:

* `transaction`
* `account`
* `contract`

It can help you specifically with:

* Sending transactions to smart contracts
* Deploying your own smart contracts
* Showing smart contracts state

To start using the CLI you can try minting some test\_coin with the following command:

```
cargo pbc transaction action 02c14c29b2697f3c983ada0ee7fac83f8a937e2ecd feed_me [PublicAddressYouWantMintToGoTo] --gas 60000 --privatekey=PathToPrivatekeyFile
```

If you want to try an action yourself you can type: `cargo pbc transaction action` and the help messages will help you from there.

Want to explore more possibilities within the CLI? You can go visit the [cli-execution-reference-tests to see example usage of the CLI tool](https://gitlab.com/partisiablockchain/language/partisia-cli/-/tree/main/src/test/resources/cli-execution-reference-tests?ref_type=heads) You can look in the commandline.sh that is placed within each test folder to understand the multitude of applications this tool can have.

#### The ABI tool `abi` <a href="#the-abi-tool-abi" id="the-abi-tool-abi"></a>

This tool is focused on helping you understand the [ABI](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#abi) actions and state of a contract. By using the command `cargo pbc abi show`, you can access information about a compiled contract's state, initialization, actions, and variables. It simplifies the process of identifying shortnames for existing contracts using optional arguments. This tool is using the [abi-client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#abi-client).

Example command:

```
cargo pbc abi show example-contracts/target/wasm32-unknown-unknown/release/auction_contract.abi
```

<details>

<summary>Response from example command</summary>

```
```

</details>

#### The ABI codegen tool `abi codegen` <a href="#the-abi-codegen-tool-abi-codegen" id="the-abi-codegen-tool-abi-codegen"></a>

Codegen produces autogenerated code in both Java & TypeScript to streamline interactions with contract actions and deserializing contracts states. The autogenerated code provides methods to interact with actions based on a smart contracts [abi](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#abi). If you are working with Java, we recommend you follow [the readme here](https://gitlab.com/partisiablockchain/language/abi/abi-client/-/tree/main/maven-plugin?ref_type=heads) to automate your usage of abi codegen.

Abi codegen can also be used manually. Here is an example of how:

```
cargo pbc abi codegen --ts mySmartContract/target/wasm32-unknown-unknown/release/auction_contract.abi mySmartContract/AutogeneratedCode/auction_contract.ts
```

<details>

<summary>Response from example command</summary>

```
```

</details>

### Libraries <a href="#libraries" id="libraries"></a>

Partisia Blockchain offers a set of different libraries to help you interact with smart contracts and work with the states of smart contracts on the chain.

#### client <a href="#client" id="client"></a>

Facilitates communication with a blockchain node. It helps you to interact with the blockchain programmatically. It can get data about blocks, transactions, smart contracts, accounts etc. It has the functionality to create and send transactions.

* [Client library](https://gitlab.com/partisiablockchain/core/client)

#### abi-client <a href="#abi-client" id="abi-client"></a>

Our Smart Contract Binary Interface Client Library. It offers a standard binary interface for deploying contracts and creating transactions, making it ideal for several use cases:

* Creating binary RPC, which calls an action with parameters on a smart contract.
* Decode binary contract states to readable types.
* Decode binary parts of transactions and events.

ABI-Client is available in the following programming languages:

* [Java](https://gitlab.com/partisiablockchain/language/abi/abi-client/-/tree/main?ref_type=heads)
* [TypeScript](https://gitlab.com/partisiablockchain/language/abi/abi-client-ts)

When using the abi-client using codegen can help you as a developer to create a plug-and-play interactions with the blockchain. Abi-client can be used to read from contracts that is not necessarily linked to a specific contract on the blockchain. The strength of [codegen](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#the-abi-codegen-tool-abi-codegen) compared to abi-client is to handle specific contracts you want to work with. Whereas abi-client is more generalistic in its approach to contracts not already know to the system.\
We have created an [example client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#example-client) to showcase how to work with the abi-client.

#### zk-client <a href="#zk-client" id="zk-client"></a>

Sending secret input to ZK Rust smart contracts. The zk-client is a library that can help you interact with the blockchain. Secondly the zk-client can fetch the secret data that you have ownership of from a ZK contract deployed onto the chain. You can visit the tests inside the projects to see how it works and start from there.

zk-client is available in the following programming languages:

* [Java](https://gitlab.com/partisiablockchain/language/abi/zk-client/)
* [TypeScript](https://gitlab.com/partisiablockchain/language/abi/zk-client-ts)

#### Example client <a href="#example-client" id="example-client"></a>

This is a front end and a back end example of how to integrate you application with Partisia Blockchain, specifically it uses the [client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#client) and the [abi-client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#abi-client) to send transactions and read states of the contracts.

Example client is available in the following programming languages:

* [Java](https://gitlab.com/partisiablockchain/language/example-client)
*   [TypeScript](https://gitlab.com/partisiablockchain/language/example-web-client)

    This article introduces tools designed to help developers working with smart contracts on the blockchain. These tools offer a range of capabilities, from testing smart contracts to facilitating command-line interactions with the blockchain, and even integrating the blockchain with your own applications.

    ### Partisia Blockchain browser <a href="#partisia-blockchain-browser" id="partisia-blockchain-browser"></a>

    Partisia blockchain browser is a web-based interface which translates the blockchains data into a user-friendly application. Partisia blockchain browser is essentially a complete UI for Partisia Blockchain.

    Partisia blockchain browser can be used to:

    * Display details for the entire ledger: blocks and transactions.
    * Display details for all accounts and smart contracts.
    * Interact with any smart contract.
    * Deploy your own new smart contracts.
    * Manage your own account, including your MPC tokens and becoming a Node Operator.
    * Create local references of contracts and accounts to help you keep track of already deployed contracts.

    Partisia Blockchain browser can be used with the Testnet and the Mainnet:

    * [Testnet](https://browser.testnet.partisiablockchain.com)
    * [Mainnet](https://browser.partisiablockchain.com)

    ### Command-line tools <a href="#command-line-tools" id="command-line-tools"></a>

    [`cargo pbc`](https://crates.io/crates/cargo-partisia-contract) is an umbrella for multiple sub-tools. The tools assist you in interacting with the blockchain and working with smart contracts. These tools are thoroughly documented when using them within `cargo pbc`, enabling you to explore their capabilities inside `cargo pbc`. Below are a short description and use case for each of these sub-tools.

    Pre requisite to use any cargo PBC commands

    If you want to use any of the command-line tools or below commands you need to install [the smart contract compiler](https://partisiablockchain.gitlab.io/documentation/smart-contracts/install-the-smart-contract-compiler.html).

    #### The Compiler `build` <a href="#the-compiler-build" id="the-compiler-build"></a>

    This is a primary part of developing smart contracts. The `build` command compiles [rust smart contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/compile-and-deploy-contracts.html) and [ZK Rust smart contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/compile-and-deploy-zk-contract.html). It compiles and returns `.abi` file and a `.wasm` for rust contracts or `.zkwa` for ZK rust contracts.

    #### Blockchain interaction <a href="#blockchain-interaction" id="blockchain-interaction"></a>

    To interact with the blockchain you can use the command line tool: `cargo pbc`. There are 3 main commands:

    * `transaction`
    * `account`
    * `contract`

    It can help you specifically with:

    * Sending transactions to smart contracts
    * Deploying your own smart contracts
    * Showing smart contracts state

    To start using the CLI you can try minting some test\_coin with the following command:

    ```
    cargo pbc transaction action 02c14c29b2697f3c983ada0ee7fac83f8a937e2ecd feed_me [PublicAddressYouWantMintToGoTo] --gas 60000 --privatekey=PathToPrivatekeyFile
    ```

    If you want to try an action yourself you can type: `cargo pbc transaction action` and the help messages will help you from there.

    Want to explore more possibilities within the CLI? You can go visit the [cli-execution-reference-tests to see example usage of the CLI tool](https://gitlab.com/partisiablockchain/language/partisia-cli/-/tree/main/src/test/resources/cli-execution-reference-tests?ref_type=heads) You can look in the commandline.sh that is placed within each test folder to understand the multitude of applications this tool can have.

    #### The ABI tool `abi` <a href="#the-abi-tool-abi" id="the-abi-tool-abi"></a>

    This tool is focused on helping you understand the [ABI](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#abi) actions and state of a contract. By using the command `cargo pbc abi show`, you can access information about a compiled contract's state, initialization, actions, and variables. It simplifies the process of identifying shortnames for existing contracts using optional arguments. This tool is using the [abi-client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#abi-client).

    Example command:

    ```
    cargo pbc abi show example-contracts/target/wasm32-unknown-unknown/release/auction_contract.abi
    ```

    #### The ABI codegen tool `abi codegen` <a href="#the-abi-codegen-tool-abi-codegen" id="the-abi-codegen-tool-abi-codegen"></a>

    Codegen produces autogenerated code in both Java & TypeScript to streamline interactions with contract actions and deserializing contracts states. The autogenerated code provides methods to interact with actions based on a smart contracts [abi](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#abi). If you are working with Java, we recommend you follow [the readme here](https://gitlab.com/partisiablockchain/language/abi/abi-client/-/tree/main/maven-plugin?ref_type=heads) to automate your usage of abi codegen.

    Abi codegen can also be used manually. Here is an example of how:

    ```
    cargo pbc abi codegen --ts mySmartContract/target/wasm32-unknown-unknown/release/auction_contract.abi mySmartContract/AutogeneratedCode/auction_contract.ts
    ```

    ### Libraries <a href="#libraries" id="libraries"></a>

    Partisia Blockchain offers a set of different libraries to help you interact with smart contracts and work with the states of smart contracts on the chain.

    #### client <a href="#client" id="client"></a>

    Facilitates communication with a blockchain node. It helps you to interact with the blockchain programmatically. It can get data about blocks, transactions, smart contracts, accounts etc. It has the functionality to create and send transactions.

    * [Client library](https://gitlab.com/partisiablockchain/core/client)

    #### abi-client <a href="#abi-client" id="abi-client"></a>

    Our Smart Contract Binary Interface Client Library. It offers a standard binary interface for deploying contracts and creating transactions, making it ideal for several use cases:

    * Creating binary RPC, which calls an action with parameters on a smart contract.
    * Decode binary contract states to readable types.
    * Decode binary parts of transactions and events.

    ABI-Client is available in the following programming languages:

    * [Java](https://gitlab.com/partisiablockchain/language/abi/abi-client/-/tree/main?ref_type=heads)
    * [TypeScript](https://gitlab.com/partisiablockchain/language/abi/abi-client-ts)

    When using the abi-client using codegen can help you as a developer to create a plug-and-play interactions with the blockchain. Abi-client can be used to read from contracts that is not necessarily linked to a specific contract on the blockchain. The strength of [codegen](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#the-abi-codegen-tool-abi-codegen) compared to abi-client is to handle specific contracts you want to work with. Whereas abi-client is more generalistic in its approach to contracts not already know to the system.\
    We have created an [example client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#example-client) to showcase how to work with the abi-client.

    #### zk-client <a href="#zk-client" id="zk-client"></a>

    Sending secret input to ZK Rust smart contracts. The zk-client is a library that can help you interact with the blockchain. Secondly the zk-client can fetch the secret data that you have ownership of from a ZK contract deployed onto the chain. You can visit the tests inside the projects to see how it works and start from there.

    zk-client is available in the following programming languages:

    * [Java](https://gitlab.com/partisiablockchain/language/abi/zk-client/)
    * [TypeScript](https://gitlab.com/partisiablockchain/language/abi/zk-client-ts)

    #### Example client <a href="#example-client" id="example-client"></a>

    This is a front end and a back end example of how to integrate you application with Partisia Blockchain, specifically it uses the [client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#client) and the [abi-client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#abi-client) to send transactions and read states of the contracts.

    Example client is available in the following programming languages:

    * [Java](https://gitlab.com/partisiablockchain/language/example-client)
    * [TypeScript](https://gitlab.com/partisiablockchain/language/example-web-client)
