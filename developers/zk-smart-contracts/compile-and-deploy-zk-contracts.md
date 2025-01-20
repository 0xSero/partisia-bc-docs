# Compile and Deploy ZK Contracts

As an example of compiling a zk-contract we will make use of the second-price auction contract. The contract can be found in the example contracts archive at: `contracts/example-zk-second-price-auction`. You can download [our example zip archive here](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-examples.html) and read what a second-price auction is on that page as well.

The zk-contracts consist of two main parts. The contract itself as well as a zero-knowledge computation. To compile a zero-knowledge contract run:

```
cargo pbc build --release
```

Note that this is the same command as for [public contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/compile-and-deploy-contracts.html). The tool looks for a path to the zk-computation in the manifest file. If this is defined under package.metadata.zk then the tool tries to compile the contract as a zk-contract instead of a normal public contract. The tool first compiles the public part of the contract to get a WASM- and ABI-file. Afterwards it fetches the zk-compiler and compiles the zk-computation and links it with the generated WASM-file to get a ZKWA-file.

You can use the [testnet](https://partisiablockchain.gitlab.io/documentation/smart-contracts/access-and-use-the-testnet.html) to try out deployment for free.

#### Zero knowledge assurance <a href="#zero-knowledge-assurance" id="zero-knowledge-assurance"></a>

Another key difference between public smart contracts and the private zero knowledge smart contracts is that subset of high stakes nodes are allocated to carry out the ZK computations of each ZK contract. The contract owner has to specify how much value the ZK computation represents. This amount of the ZK nodes' stake will be locked til the ZK calculation is over in addition to a waiting period. The stakes function as an extra assurance of the confidential values managed by the ZK nodes. It is, however, only the services and/or the users of the service running on top of Partisia Blockchain that truly know the value of the confidential information involved. To accommodate for this asymmetric information problem the users can adjust stakes to meet an appropriate level of insurance ([Yellow Paper pp. 40-41](https://drive.google.com/file/d/1OX7ljrLY4IgEA1O3t3fKNH1qSO60_Qbw/view)).
