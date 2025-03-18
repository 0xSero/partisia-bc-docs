# Compile and deploy contracts

In the following sections we focus on the example petition contract included in the example contracts repository. The contract utilizes two functions: _initialize_, and _sign_, which allow you to perform the basic operations needed to create and sign a petition. After deployment, the contract actions can be called from the dashboard. Performing an action changes the contract state. If you inspect the contract you can see the serialized data showing the contract state.

If you want a developer environment without having to install the compiler yourself you can head over to our [dApp playground](https://github.com/partisiablockchain/dapp-playground/) to immediately start building.

### 1) Compile a contract example <a href="#id-1-compile-a-contract-example" id="id-1-compile-a-contract-example"></a>

The petition contract can be found in the [example contracts repository](https://gitlab.com/partisiablockchain/language/example-contracts). After cloning the repository to your machine, the following command will compile the contract and generate an [ABI](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#abi) for it:

```
cd example-contracts/rust/petition
cargo pbc build --release
```

You will find a newly generated .wasm file and an .abi file in your local directory: `/target/wasm32-unknown-unknown/release`.

### 2) Upload the contract to the blockchain <a href="#id-2-upload-the-contract-to-the-blockchain" id="id-2-upload-the-contract-to-the-blockchain"></a>

To deploy a smart contract you need an [account](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/create-an-account.html) with [gas](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/what-is-gas.html) to cover transaction costs. You can deploy a contract through the [Partisia Blockchain browser](https://browser.partisiablockchain.com/contracts/deploy).

Tip

You can also test deploying a contract for free on [the testnet](https://browser.testnet.partisiablockchain.com/contracts/deploy). To get some testnet gas follow [these steps](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/how-to-get-testnet-gas.html).

Navigate to the [Contracts](https://browser.partisiablockchain.com/contracts) menu in the Browser and click on the " Deploy contract" button. Upload the `petition.wasm` and `petition.abi` files. The Browser's main panel will then render a form for the initialization function. If you inspect the `lib.rs` file in your IDE, you will see that this matches the _initialize_ function.

You can give your petition contract any name by filling in the Description field.

It should look like this before deployment:

![compile-and-deploy-contracts-before-deploy](https://partisiablockchain.gitlab.io/documentation/smart-contracts/img/compile-and-deploy-contracts-00.png)

Successful deployment will look like this:

![compile-and-deploy-contracts-after-deploy](https://partisiablockchain.gitlab.io/documentation/smart-contracts/img/compile-and-deploy-contracts-01.png)

You are now ready to interact with the contract. You can click _Interact_ in the browser and start using the sign action.

Congratulations! You have now created an active smart-contract on the Partisia Blockchain.
