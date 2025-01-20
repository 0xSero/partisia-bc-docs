# Testing a Smart Contract

The testing framework allows you to deploy and transact with the contract on a local blockchain. Tests for your smart contract helps you avoid mistakes and exploits in the contract, you are developing. To learn how to write tests for smart contracts, take a look at [our example contracts](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-examples.html). All of those contracts have tests that ensure 100% code coverage.

After reading this page, you should be able to do the following:

1. Write a test, that deploys a contract
2. Use the deployment test as setup for another test.
3. Interact with the contract in the test.

The tests are written in Java, utilizing our testing framework [JUnit-contract-test](https://gitlab.com/partisiablockchain/language/junit-contract-test). During the test we are, provided a blockchain object, which allows us to deploy and interact with multiple different contracts.

#### How to run our example tests <a href="#how-to-run-our-example-tests" id="how-to-run-our-example-tests"></a>

To run the tests, navigate to the root of [example contracts](https://gitlab.com/partisiablockchain/language/example-contracts) and execute the following commands in order.

Compile the contracts:

```git
cargo pbc build --release
```

Go into the test directory, compile the tests and run them.

```
cd java-test
mvn test-compile test
```

You can also run the provided script `run-java-tests.sh`, where a build flag, `-b`, for compiling the contracts before test run, can be given. This would be equivalent to running the above commands in order.

```
./run-java-tests.sh -b
```

To run a single test class, use the maven option `-Dtest="Name of Test Class"`, and run it while standing in the test directory.

```
mvn test -Dtest="VotingTest"
```

#### Run test with code coverage <a href="#run-test-with-code-coverage" id="run-test-with-code-coverage"></a>

To see that the tests hits the different parts of the smart contract code, we can run the tests with coverage enabled. The tests will generate coverage information, which can be compiled into a report. To run the tests with coverage enabled, execute `run-java-tests.sh` with the build flag, `-b`, and the coverage flag, `-c`.

```
./run-java-tests.sh -c -b
```

This will create the instrumented executable, and run the test with coverage enabled. The coverage report will be located in the `java-test/target/coverage/html`, where the `index.html` can be opened in your browser.

### Detailed explanation of the voting test example <a href="#detailed-explanation-of-the-voting-test-example" id="detailed-explanation-of-the-voting-test-example"></a>

The following section is the JUnit test of our [voting smart contract](https://gitlab.com/partisiablockchain/language/example-contracts/-/blob/main/java-test/src/test/java/examples/VotingTest.java?ref_type=heads). This example will be explained in the following sections.

```java
import com.partisiablockchain.BlockchainAddress;
import com.partisiablockchain.language.abicodegen.Voting;
import com.partisiablockchain.language.junit.ContractBytes;
import com.partisiablockchain.language.junit.ContractTest;
import com.partisiablockchain.language.junit.JunitContractTest;
import com.partisiablockchain.language.junit.exceptions.ActionFailureException;

import java.nio.file.Path;
import java.util.List;

import org.assertj.core.api.Assertions;

/** Test suite for the Voting contract. */
public final class VotingTest extends JunitContractTest {
    private static final ContractBytes VOTING_CONTRACT_BYTES =
            ContractBytes.fromPaths(
                    Path.of("../target/wasm32-unknown-unknown/release/voting.wasm"),
                    Path.of("../target/wasm32-unknown-unknown/release/voting.abi"),
                    Path.of("../target/wasm32-unknown-unknown/release/voting_runner"));
    private BlockchainAddress voter1; //Unique address of a voter on Partisia Blockchain
    private BlockchainAddress voter2;
    private BlockchainAddress voter3;
    private BlockchainAddress voting;

    /** Setup for all the other tests. Deploys a voting contract and instantiates accounts. */
    @ContractTest
    void setUp() {
        voter1 = blockchain.newAccount(2);
        voter2 = blockchain.newAccount(3);
        voter3 = blockchain.newAccount(4);

        // Initialize a voting with a proposal ID, a list of voters and a deadline.
        byte[] initRpc = Voting.initialize(10, List.of(voter1, voter2, voter3), 60 * 60 * 1000);
        voting = blockchain.deployContract(voter1, VOTING_CONTRACT_BYTES, initRpc);
    }

    /** An eligible voter can cast a vote. */
    @ContractTest(previous = "setUp")
    public void castVote() {
        byte[] votingRpc = Voting.vote(true);
        // An eligible voter casts a vote
        blockchain.sendAction(voter1, voting, votingRpc);
        // Get the current state of the voting contract
        Voting.VoteState state = Voting.VoteState.deserialize(blockchain.getContractState(voting));

        // One vote should be cast, and it should be "true"
        Assertions.assertThat(state.votes().size()).isEqualTo(1);
        Assertions.assertThat(state.votes().get(voter1)).isTrue();
    }
}
```

#### Abstract Test Class <a href="#abstract-test-class" id="abstract-test-class"></a>

The test extends the JunitContractTest class, this is the extension providing the blockchain used in the tests.

<details>

<summary>JunitContractTest explained:</summary>

```java
public final class VotingTest extends JunitContractTest { // The test class must extend to JunitContractTest, to run Smart Contract Tests.
    // Fields and tests.
}
```

</details>

#### Deploying smart contract bytecode <a href="#deploying-smart-contract-bytecode" id="deploying-smart-contract-bytecode"></a>

In order to test a smart contract, you need to deploy the contract. To deploy a contract, we must specify the bytecode to deploy. We have declared an instance of `ContractBytes`, where we specify the location of the .wasm and .abi file for the [example voting contract](https://gitlab.com/partisiablockchain/language/example-contracts/-/tree/main/voting?ref_type=heads). The three paths declared are the path to the wasm, the abi and the local instrumented executable. The local instrumented executable is used for code coverage, the executable is named runner after the contract name.

<details>

<summary>ContractBytes explained:</summary>

```java
    private static final ContractBytes VOTING_CONTRACT_BYTES =
      ContractBytes.fromPaths(
              Path.of("../target/wasm32-unknown-unknown/release/voting.wasm"),  // The path to the WASM of the contract.
              Path.of("../target/wasm32-unknown-unknown/release/voting.abi"),   // The path to the ABI of the contract.
              Path.of("../target/wasm32-unknown-unknown/release/voting_runner") // The path to the instrumented executable.
      );
```

</details>

#### BlockchainAddress Fields - Values transferred <a href="#blockchainaddress-fields-values-transferred" id="blockchainaddress-fields-values-transferred"></a>

The four fields with type BlockchainAddress, `voter1`, `voter2`, `voter3` and `voting`, these will be populated in the test, where a public account is created for each voter, and when deploying the voting contract, the call returns the address the contract was deployed at.

These fields will be transferred to tests depending on the test, if the field is populated at the end of test execution. This means we only have to instantiate the field once. To create an account, we call the `blockchain.newAccount(1)`, where `1` is the secret key for the Address created.

The address for the contract, is created when it is deployed. To deploy a contract, the address sending the transaction is needed, the bytecode to deploy and the arguments for the initialize call, creating the initial state of the contract.

<details>

<summary>Addresses - By deployment or creation</summary>

```java
voter1 = blockchain.newAccount(2); // Create an account with secret key '2' and return the related BlockchainAddress.
voter2 = blockchain.newAccount(3); // Create an account with secret key '3' and return the related BlockchainAddress.
voter3 = blockchain.newAccount(4); // Create an account with secret key '4' and return the related BlockchainAddress.

byte[] initRpc =
    Voting.initialize(10, List.of(voter1, voter2, voter3), 60 * 60 * 1000); // The rpc for the init function when creating a contract.

voting =
    blockchain.deployContract(voter1, VOTING_CONTRACT_BYTES, initRpc); // Deployment of a contract, returns the address the contract is deployed at.
```

</details>

**Using ABI codegen for RPC**

Calling actions in a contract requires byte streams serialized according to the [RPC binary format](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-binary-formats.html#rpc-binary-format).

In the deployment call, the RPC is serialized using code generated from the ABI of the contract. This can either be generated with [ABI client](https://partisiablockchain.gitlab.io/documentation/smart-contracts/smart-contract-tools-overview.html#the-abi-codegen-tool-abi-codegen), or using the [Maven plugin](https://gitlab.com/partisiablockchain/language/abi/abi-client/-/tree/main/maven-plugin?ref_type=heads).

The generated code provides methods for serializing the RPC for all actions in the contract. The generated code also provides a state deserialization method. The generated code deserializes a state given in bytes, to a record object, which can be used to assert on the state of a contract in a test. An example of the state serialization is in [the second test](https://partisiablockchain.gitlab.io/documentation/smart-contracts/testing-contracts/testing-a-smart-contract.html#second-test-using-the-first-test-as-a-setup).

<details>

<summary>State</summary>

```java
Voting.VoteState state = // The state variable created from the deserialization.
    Voting.VoteState.deserialize(blockchain.getContractState(voting)); // The call to the generated deserialize state method

Assertions.assertThat(state.votes().size()).isEqualTo(1); // Assertion on the amount of votes currently.
Assertions.assertThat(state.votes().get(voter1)).isTrue(); // Assertion on who voted.
```

</details>

#### How to build on top of an already existing test <a href="#how-to-build-on-top-of-an-already-existing-test" id="how-to-build-on-top-of-an-already-existing-test"></a>

Smart contracts are great since the order of operations are clear, from the perspective of behavioral execution. This means that every time we would have to test 2 different actions, meaning two different behaviors, on the same state. Tests would have to perform all the actions up to that point, to then perform one of the actions, and then perform them all again for the other action.

Here the testing framework helps, the methods with the annotation `@ContractTest`, can be used as setup for other tests.

<details>

<summary>@ContractTest and previous</summary>

```java
@ContractTest // The annotation with no previous, means the test is independent of other test, also called 'root-test'.
void setUp() {
// Omitted
}

/** An eligible voter can cast a vote. */
@ContractTest(previous = "setUp") // The previous for the test, is the 'setUp', so for the test to succeed the 'setUp' test must be run before this test.
public void castVote() {
// Omitted
}
```

</details>

#### Sending an Action to a deployed contract. <a href="#sending-an-action-to-a-deployed-contract" id="sending-an-action-to-a-deployed-contract"></a>

Interaction with the contract, is made by sending an action to a deployed contract. The action needs three parameters, the address for the contract the action is targeting, the address of the sender and the RPC, which specifies the arguments for the action call.

<details>

<summary>Send action.</summary>

```java
@ContractTest(previous = "setUp")
public void castVote() {
byte[] votingRpc = Voting.vote(true); // Create the RPC with the generated code.
blockchain.sendAction(voter1, voting, votingRpc); // Send the action, with the sender, the target contract and the RPC.
Voting.VoteState state = Voting.VoteState.deserialize(blockchain.getContractState(voting)); // Get the state of the contract and deserialize.

Assertions.assertThat(state.votes().size()).isEqualTo(1); // Assert the vote is registered.
Assertions.assertThat(state.votes().get(voter1)).isTrue(); // Assert the registered vote is from 'voter1'.

}
```

</details>

#### Second test using the first test as a setup <a href="#second-test-using-the-first-test-as-a-setup" id="second-test-using-the-first-test-as-a-setup"></a>

We can now perform our first interaction with the deployed contract. So we add another test to our test class. Another account must be created, so we add another field, `receiver`, and create the account.

Notice in the annotation `@ContractTest`, we have provided an argument that points to our previous method, where we deployed our contract. In our example, that is `setup`.

In this test, we send an action to our contract, the functionality we are testing is just a normal transfer from `owner` to `receiver`. We call the `.sendAction()`, where we provided the address of the contract, we want to send the action to. The sender of the [action](https://partisiablockchain.gitlab.io/documentation/smart-contracts/programmers-guide-to-smart-contracts.html#action), and the [RPC](https://partisiablockchain.gitlab.io/documentation/smart-contracts/programmers-guide-to-smart-contracts.html#events) for the call. The RPC in this case contains the receiver of the transfer, and the amount to send.

If you want to more examples of testing smart contracts, go to [example contracts](https://gitlab.com/partisiablockchain/language/example-contracts), where there are multiple tests.
