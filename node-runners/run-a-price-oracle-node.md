# Run a Price Oracle Node

This page teaches you to register your baker node for the additional service of price oracle, and how to deregister from the service in case you want to use the tokens for something else.

Price oracles monitor and report prices of the [BYOC](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/byoc.html). Any node with sufficient tokens associated to the large oracle contract can register as a price oracle.\
You can learn more about the price oracles [here](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#price-oracle).

You must complete these requirements before you can continue

1. [Run a baker node](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-baker-node.html)
2. [Stake 5 K MPC tokens](https://browser.partisiablockchain.com/node-operation). You need a total staking balance of 30K for both price oracle and baker node.

### Register as a price oracle <a href="#register-as-a-price-oracle" id="register-as-a-price-oracle"></a>

Becoming a price oracle requires a stake of 5000 MPC associated to the large oracle contract and a _Register_ transaction at one of the price oracles:

1. Find the [large oracle contract](https://browser.partisiablockchain.com/contracts/04f1ab744630e57fb9cfcd42e6ccbf386977680014/associateTokensToContract) with the address `04f1ab744630e57fb9cfcd42e6ccbf386977680014`
2. Sign in (upper right corner)
3. Invoke the contract action [AssociateTokensToContract](https://browser.partisiablockchain.com/contracts/04f1ab744630e57fb9cfcd42e6ccbf386977680014/associateTokensToContract) with a minimum amount of 5000 MPC tokens. You need 5000 tokens per price oracle you register for.
4. Go to [ETH Price Oracle contract](https://browser.partisiablockchain.com/contracts/0485010babcdb7aa56a0da57a840d81e2ea5f5705d/register), [BNB Price Oracle contract](https://browser.partisiablockchain.com/contracts/049abfc6e763e8115e886fd1f7811944f43b533c39/register) or [MATIC Price Oracle contract](https://browser.partisiablockchain.com/contracts/042a9dcb0c96b9875f529e3a51ddc02473c1a78d33/register) and invoke the action _Register_

### Steps to deregister as a price oracle <a href="#steps-to-deregister-as-a-price-oracle" id="steps-to-deregister-as-a-price-oracle"></a>

The first step to deregister is to verify which price oracle the node serves. If known, skip to [deregistration](https://partisiablockchain.gitlab.io/documentation/node-operations/run-a-price-oracle-node.html#deregister-from-a-price-oracle).

#### Verify which price oracle your node serves <a href="#verify-which-price-oracle-your-node-serves" id="verify-which-price-oracle-your-node-serves"></a>

1. Open the [large oracle contract state](https://browser.partisiablockchain.com/contracts/04f1ab744630e57fb9cfcd42e6ccbf386977680014?tab=state)
2. Open the map `stakedTokens`
3. Search for your blockchain address `CTRL+f`
4. Open the struct next to your blockchain address
5. Open the map `lockedToOracle`
6. You can see the addresses of the oracles your node serve and MPC tokens staked to them
7. Copy the address of the oracle you want to leave (you can distinguish the price oracles from deposit and withdrawal oracles by looking at the amount of MPC tokes they have "locked", price oracles have 5000, deposit and withdrawal oracles 250000)
8. Paste the address to the search field of the browser, to navigate to the contract

#### Deregister from a price oracle <a href="#deregister-from-a-price-oracle" id="deregister-from-a-price-oracle"></a>

To leave a price oracle see the step-by-step guide below.

1. Sign in to the browser
2. Opt out of notifying price updates by invoking the contract action _Opt out_ on the price oracle your node serves:
   * [ETH Price Oracle Contract](https://browser.partisiablockchain.com/contracts/0485010babcdb7aa56a0da57a840d81e2ea5f5705d?tab=state)
   * [BNB Price Oracle Contract](https://browser.partisiablockchain.com/contracts/049abfc6e763e8115e886fd1f7811944f43b533c39?tab=state)
   * [MATIC Price Oracle contract](https://browser.partisiablockchain.com/contracts/042a9dcb0c96b9875f529e3a51ddc02473c1a78d33?tab=state)
3. Wait an hour to ensure any ongoing round is finished.
4. Invoke the contract action _Deregister_ on the contract.

If your node has already reported a price in the current round, your invocation of _Deregister_ will result in an error saying:

`Cannot deregister an oracle node that has notified a price update on an ongoing round`

This means that you did not wait long enough (step 3), _or_ that you did not opt out of notifying price updates for future rounds (step 2). Please make sure that you have opted out, and that you have waited an hour from when you opted out.
