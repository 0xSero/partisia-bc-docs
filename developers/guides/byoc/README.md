# BYOC

## YOC <a href="#byoc" id="byoc"></a>

In this article we will explain what BYOC is, and how to bring your own coins to the mainnet and to the testnet.

#### What is BYOC and gas <a href="#what-is-byoc-and-gas" id="what-is-byoc-and-gas"></a>

BYOC means bring your own coin. The concept is that you can bring other cryptocurrencies onto PBC. Any coin you bring onto PBC stays as a reference to the coin you brought in. You can convert your BYOC [to gas](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/transaction-gas-prices.html). This is done automatically when using gas to pay for transactions.

BYOC is needed to cover the payment of the node operators which provide the services that enable the transactions to take place. You pay for interactions with gas that is converted from coins you have brought onto the chain. You can see the explained [gas fees here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/transaction-gas-prices.html).

#### How does it work <a href="#how-does-it-work" id="how-does-it-work"></a>

To be able to spend and transfer your coins, you will need to transfer your own cryptocurrency using [the PBC Token Bridge](https://browser.partisiablockchain.com/bridge). This will move your external coins to a contract and mint a PBC version of the coin you brought. This new PBC based version of your coin can interact with the payment scheme of PBC. In essence, you can deposit, withdraw and transfer ETH or other cryptocurrencies with your MetaMask wallet. You can see what bridges are available on our page: [Bridging BYOC by sending transactions](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/bridging-byoc-by-sending-transactions.html).

**BYOC fee (tentative):**\
Bridging has a small fee, since transferring coins between chains requires the nodes to sign the transfers.

The cost of bringing your own coin is 0,1% of the value transferred with a threshold of 25 USD as minimum transfer (equal to a minimum BYOC fee of 2,5 USD cents).

#### Converting MPC token to BYOC <a href="#converting-mpc-token-to-byoc" id="converting-mpc-token-to-byoc"></a>

The native token on PBC, MPC token, can be used as BYOC. To do this, it must first be _wrapped_, using the system smart contract [mpc-wrap](https://browser.partisiablockchain.com/contracts/017d9dacdd01f0b2bd4de40da37f545e89b7faa149), into its BYOC equivalent, WMPC (Wrapped MPC). Alternatively, a shortcut to wrapping your MPC tokens can be found [here](https://browser.partisiablockchain.com/assets) by clicking the _Wrap to WMPC_ button in the top right corner. WMPC can then be used similar to other BYOC, e.g. as gas, or to bridge out to an external chain. Using the same contract, WMPC can be _unwrapped_ back into MPC token.

#### Requirements <a href="#requirements" id="requirements"></a>

To be able to bridge your coins you will need:

* An account on PBC.
* An account on MetaMask holding the coins you want to transfer into PBC. The bridge only supports the MetaMask wallet extension for interacting with other chains.

#### How to deposit using the Bridge <a href="#how-to-deposit-using-the-bridge" id="how-to-deposit-using-the-bridge"></a>

To make a deposit, you can use the [PBC Token Bridge](https://browser.partisiablockchain.com/bridge). Depending on whether the coin you want to transfer is a native coin (e.g. ETH or BNB) or a token based coin (e.g. WMPC or USDC) the steps to transfer differ. This is because tokens needs to be approved, such that the external BYOC contract is able to transfer the tokens for you.

In practice, you can follow these steps:

| Deposit native coin                                                                                                                                                                         | Deposit token based coin                                                                                                                                                         |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Go to the [BYOC page](https://browser.partisiablockchain.com/bridge) and login to your PBC account                                                                                       | 1. Go to the [BYOC page](https://browser.partisiablockchain.com/bridge) and login to your PBC account                                                                            |
| 2. Click _deposit_, either directly or through the three dots menu, on the coin you want to transfer                                                                                        | 2. Click approve, through the three dots menu, on the coin you want to transfer                                                                                                  |
| 3. Click _Connect MetaMask_ to connect your external account                                                                                                                                | 3. Click _Connect MetaMask_ to connect your external account                                                                                                                     |
| 4. Enter the amount you wish to transfer in the dialog                                                                                                                                      | 4. Enter the amount you wish to approve in the dialog                                                                                                                            |
| 5. Wait for the nodes to sign the deposit (Pending deposits on the connected chain can be seen in the top right notification menu, which only appears when you have a pending notification) | 5. Wait for the transaction on the external chain                                                                                                                                |
|                                                                                                                                                                                             | 6. Click _deposit_ on the coin you want to transfer                                                                                                                              |
|                                                                                                                                                                                             | 7. Enter the amount you wish to deposit in the dialog (_Note_ that you cannot deposit more than you have approved)                                                               |
|                                                                                                                                                                                             | 8. Wait for the nodes to sign the deposit (Pending deposits on the connected chain can be seen in the top right corner, which only appears when you have a pending notification) |

When the nodes have signed the deposit, the coins will be available for you on PBC, and can be seen on the [Bridge page](https://browser.partisiablockchain.com/bridge). After the confirmed transfer your gas balance in the wallet should be positive. This means that you can deploy or interact with smart contracts on the blockchain and pay for those interactions with the coins that you have bridged. Read more about smart contracts [here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/what-is-a-smart-contract.html).

#### How to withdraw using the Bridge <a href="#how-to-withdraw-using-the-bridge" id="how-to-withdraw-using-the-bridge"></a>

To withdraw a coin from PBC to your external account, you can use the [PBC Token Bridge](https://browser.partisiablockchain.com/bridge). The withdrawal process has two steps:

1. First, you have to make a pending withdrawal on the PBC chain. When the nodes have seen and signed this pending withdrawal, the coins will be unlocked on the external withdrawal contract.
2. Second, after the signed withdrawal the coins can be transferred to your external account by withdrawing on the withdrawal contract. Both steps are performed in the [Browser](https://browser.partisiablockchain.com/bridge).

In practice, you should follow these steps:

1. Go to the [BYOC page](https://browser.partisiablockchain.com/bridge) and login to your PBC account
2. Click _withdraw_, through the three dots menu, on the coin you want to transfer
3. Click _Connect MetaMask_ to connect your external account
4. Enter the amount you wish to withdraw in the dialog
5. Wait for the nodes to sign the pending withdrawal (Pending withdrawals on the connected chain can be seen in the top right notification menu, this menu only shows if you have pending transactions)
6. Click the _withdraw button_ in the notification menu

Once the withdrawal transaction has gone through on the external chain, the coins will be available for you in the MetaMask wallet.

#### How to test BYOC? <a href="#how-to-test-byoc" id="how-to-test-byoc"></a>

To test BYOC you need to:

1. Get ETH Sepolia into your MetaMask account.
   * You need to change the network to the Sepolia test network in MetaMask to see it.
2. [Deposit](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/byoc.html#how-to-deposit-using-the-bridge) ETH Sepolia from your MetaMask account to your testnet account.
   * We recommend using the [PBC testnet Token Bridge](https://browser.partisiablockchain.com/bridge)
3. After successfully depositing ETH Sepolia on your PBC account you now have access to gas on the testnet through the coins value that was deposited.
