# Create an Account

## Create an account <a href="#create-an-account" id="create-an-account"></a>

Create an account through the [wallet extension](https://chrome.google.com/webstore/detail/partisia-wallet/gjkdbeaiifkpoencioahhcilildpjhgh) or when you buy [MPC tokens](https://kyc.partisiablockchain.com/) for staking on a node.

Every account has an individual private key used for signing transactions, this key has a public counterpart called a public key, the short form of the public key is called the account address.

A Partisia Blockchain account holds the information necessary to enabling the user to perform transactions. An account hold the user's information such as your balance of MPC tokens and [BYOC](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/byoc.html). Only [system contracts](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/governance-system-smart-contracts-overview.html) can change the state of an account i.e. user deployed smart contracts cannot change account balances. PBC has an open account structure, meaning that any private key of a valid format can create a new account. Accounts and contracts reside on a specific [shard](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/sharding.html). You can find the values of any account attribute of a specific account by looking up the account address in the [browser](https://browser.partisiablockchain.com/accounts).

Every account has the attributes defined below:

* Address (a unique identity derived from the [public key](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#public-key-cryptography))
* Balance (the balance of [BYOC](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/byoc.html))
* MPC tokens (the balance of[MPC Tokens](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/mpc-token-model-and-account-elements.html))
* [Nonce (number used only once)](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#nonce) (the account nonce is incremented when transactions are signed)

The account state itself holds a single piece of information: The nonce. This is a number that is incremented each time a transaction signed by an account is executed. The rest of the account information resides in the account plugin.

![Account\_plugin](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/img/create-an-account-00.png)

Accounts are used when sending transactions to any contract on the blockchain. Since the account nonce is part of the signature it can be used only once. This means that an account holder can only execute one transaction for each block.
