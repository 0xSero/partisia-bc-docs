# How to Get Testnet Gas

When working with smart contracts, the testnet is the best way to upload your smart contracts and estimate their gas cost. To test a smart contract you need gas, this article is here to get you started on getting gas on the testnet. By testing your contract on the testnet and verify its cost on the blockchain you can implement exact costs when doing [contract-to-contract](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/contract-to-contract-gas-estimation.html) or signed transactions to contract transactions.

#### How to see your gas balance <a href="#how-to-see-your-gas-balance" id="how-to-see-your-gas-balance"></a>

We recommend using the testnet the [browser](https://browser.testnet.partisiablockchain.com). To see your testnet gas balance you can log into the [the browser](https://browser.testnet.partisiablockchain.com/) or the [dashboard](https://testnet.partisiablockchain.com/). Under your account you can see how many TEST\_COINs you have. If you are logged into browser you can simply go to your account tab and see your balance.

#### How to get gas on the testnet <a href="#how-to-get-gas-on-the-testnet" id="how-to-get-gas-on-the-testnet"></a>

You can create an account with gas by calling the following command.

```
cargo pbc account create
```

The command create a new account, and print the account address and the private key for the account. The account will also be filled with gas.

To get gas on an account already created, run the following command.

```
cargo partisia-contract account mintgas <account-address>
```

To get even more gas you can do the following:

* Sign in to the [testnet browser](https://browser.testnet.partisiablockchain.com) (Icon in upper right corner).
* Follow this [link](https://browser.testnet.partisiablockchain.com/contracts/02c14c29b2697f3c983ada0ee7fac83f8a937e2ecd) to the [Feed Me contract interaction](https://browser.testnet.partisiablockchain.com/contracts/02c14c29b2697f3c983ada0ee7fac83f8a937e2ecd/feed_me)
* The contract action takes an account address as receiver of our test coin (A [BYOC](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/byoc.html) that only exists on the testnet). You should use your own account address as the receiver.
* By default, the browser sets the transaction cost to 100k gas which is excessive for this transaction. The transaction should take approximately 60000 gas.
* Execute the transaction - the specified account should now have gained approximately 1,000,000,000 gas which has a value of 10,000 TEST\_COIN.

You have now successfully learned how to acquire gas on the testnet and see your account balance. By utilizing the testnet, you can safely experiment with your contracts without incurring any real costs or affecting the main blockchain network. If you need gas on the mainnet you should follow the method described in [BYOC](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/byoc.html).
