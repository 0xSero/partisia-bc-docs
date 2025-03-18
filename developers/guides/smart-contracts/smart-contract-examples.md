# Smart Contract examples

The Partisia Blockchain Foundation provides the following reviewed smart contracts, as examples of real world problems with a blockchain solution. This page is created to explain the use cases of our example contracts and the titles reflect the name of the smart contract one-to-one.

We have two sets of open source examples, one is a general introduction to different world problems and the contracts work as standalone solutions. The other suite of open sourced contracts are our DeFi examples which solves decentralized finance problems with a blockchain solution, these are heavily integrated to [the token standard](https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/mpc-20-token-contract.html) and [the NFT standard](https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/mpc-721-nft-contract.html).

[Visit our repository of standalone open source examples](https://gitlab.com/partisiablockchain/language/example-contracts)

[Visit our DeFi suite of contracts to explore the token contract, liquidity swap, NFTS and more](https://gitlab.com/partisiablockchain/language/contracts/defi)

The easiest way to get started coding your own smart contracts is to learn from concrete examples of other smart contracts solving problems similar to the one you need to solve. The examples includes both public contracts and [zero-knowledge smart contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/zk-smart-contracts.html)

### Examples of combinations that you can use in your contract innovation <a href="#examples-of-combinations-that-you-can-use-in-your-contract-innovation" id="examples-of-combinations-that-you-can-use-in-your-contract-innovation"></a>

Partisia has several successful use cases and pilot projects with private deployments of the same infrastructure supporting Partisia Blockchain. The scope of these applications include internet privacy, user control of data, financial privacy, cybersecurity, humanitarian aid and confidential health statistics. You can read more about these pilot projects [here](https://partisiablockchain.com/ecosystem).

By combining the functionality of different types of open sourced smart contracts it is possible to create applications on the public blockchain within the same areas as the successful pilot projects for quick minimum viable products.

### MPC examples <a href="#mpc-examples" id="mpc-examples"></a>

Secure multiparty computation [(MPC)](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#mpc) extend the scope blockchain technology to encompass areas that before required some kind of independent third party, like a trustee, to handle sensitive data. With MPC in your smart contracts, on PBC, you can do arithmetic and statistics. But the original variables, of the problem being solved, are split into randomized parts called secret shares. Inputs are preprocessed such that the data handled by the ZK nodes cannot recreate the original user data. You can read more about how and why in the [MPC Techniques series](https://medium.com/partisia-blockchain/pbcacademy/home). See the average salary contract above as an example of MPC being used.

#### Data collaboration <a href="#data-collaboration" id="data-collaboration"></a>

* [Voting](https://gitlab.com/partisiablockchain/language/example-contracts/-/tree/main/rust/voting)\
  Voting on a public blockchain comes with the inbuilt advantage of getting an accurate result on an unalterable public ledger. However, it used to entail the disadvantage that you could not preserve the privacy of the voters while proving the accuracy of the vote count. Voting on PBC allows you to use MPC to preserve privacy. That enables you to create a vote that produce a provable correct unalterable result, but also allows the individual voters to stay anonymous.
* [Millionaires problem](https://en.wikipedia.org/wiki/Yao's_Millionaires'_problem)\
  The millionaires problem is a famous MPC problem. Whom of two parties is the richest? So, euphemism for the comparing the size of two numbers without revealing the numbers. You can see how some of these issues are handled by visiting our [average salary example](https://gitlab.com/partisiablockchain/language/example-contracts/-/tree/main/rust/zk-average-salary).
* Machine learning\
  You can do decision tree classification, pattern recognition and regression analysis on secret-shared data. There are a myriad of use cases for this on big data sets, where you still want to keep the data private. An example that uses [decision tree classification](https://gitlab.com/partisiablockchain/language/example-contracts/-/tree/main/rust/zk-classification) to predict an individual's income could be used by a bank in their loan approval process.
* Surveys\
  Individual answers are encrypted at the moment of submission and stay encrypted at all times. Only aggregated statistics are decrypted and revealed.
* Confidential statistics\
  MPC allows for combination and statistical analysis of sensitive data in separate registries. The data can be pulled into a virtual MPC sandbox where you do the analysis on secret-shared inputs.
