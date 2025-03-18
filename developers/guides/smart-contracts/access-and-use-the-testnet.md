# Access and use the testnet

The [testnet](https://browser.testnet.partisiablockchain.com/transactions) is a cost free version of Partisia Blockchain mainnet. The governing principles of the testnet are the same as for mainnet. The only differences is that the gas on the testnet is paid with a TEST\_COIN [that you can follow this article to get some of](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/how-to-get-testnet-gas.html).

### How do I use the testnet for smart contract development <a href="#how-do-i-use-the-testnet-for-smart-contract-development" id="how-do-i-use-the-testnet-for-smart-contract-development"></a>

You can deploy and test both public and private smart contracts on the testnet for free. All you need to get started is the following:

* A PBC [account](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/create-an-account.html), you can create an account with the [PBC wallet extension](https://chrome.google.com/webstore/detail/partisia-wallet/gjkdbeaiifkpoencioahhcilildpjhgh).
* Get [testnet gas](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/how-to-get-testnet-gas.html)
* [Download the PBC example smart contracts](https://gitlab.com/partisiablockchain/language/example-contracts) and [try to deploy a contract on the testnet](https://partisiablockchain.gitlab.io/documentation/smart-contracts/compile-and-deploy-contracts.html).

### How to deploy contracts on the testnet <a href="#how-to-deploy-contracts-on-the-testnet" id="how-to-deploy-contracts-on-the-testnet"></a>

* Use your private key to sign in (click icon in upper right corner)
* open the dashboard [wallet](https://testnet.partisiablockchain.com/wallet/upload_wasm) or [the browser](https://browser.partisiablockchain.com/contracts/deploy)
* For public smart contracts you upload a WASM-file as contract file and an ABI-file, but for private contract using ZK computation you upload a ZKWA-file as contract file and an ABI-file. Keep in mind that you need to add enough gas for your contract to run. You need to compile your rust contract to get access to these files, [you can see here how to do that](https://partisiablockchain.gitlab.io/documentation/smart-contracts/compile-and-deploy-contracts.html)
