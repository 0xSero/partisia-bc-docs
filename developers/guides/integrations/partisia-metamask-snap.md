# Partisia Metamask Snap

The Partisia Blockchain MetaMask snap allows users to use MetaMask for signing transactions towards Partisia Blockchain. The Snap feature is available within the latest version of [MetaMask](https://metamask.io/).

### What does the snap do? <a href="#what-does-the-snap-do" id="what-does-the-snap-do"></a>

The snap allows users and developers to use MetaMask to sign transactions. It allows getting the Partisia Blockchain address of the key and to sign transactions.

### How to install and use the snap as a user? <a href="#how-to-install-and-use-the-snap-as-a-user" id="how-to-install-and-use-the-snap-as-a-user"></a>

1. Install the latest version of the [MetaMask extension](https://metamask.io/download/).
2. Create a Wallet in MetaMask. You can reuse the seed phrase from PBC wallet if you want to have the same account address and private key. You cannot import the private key directly into MetaMask. If you want to use multiple wallets with the MetaMask snap you need to follow the MetaMask article about: [How to manage multiple wallets](https://support.metamask.io/hc/en-us/articles/12174759849371#h_01GQ58M3T5NQ19NYWTQ1C1XS2M). The Partisia Blockchain snap can only use your primary MetaMask account when signing transaction on the Partisia Blockchain Browser.
3. Go [to Partisia MetaMask Snap](https://snaps.metamask.io/snap/npm/partisiablockchain/snap/)
4. Click add to MetaMask
5. Sign in with your MetaMask account and accept the terms and conditions for the snap
6. Click "Connect MetaMask Snap" - this installs the PBC Snap (Protocol 3757).
7. You now have access to the MetaMask snap

### Video tutorial

{% embed url="https://www.youtube.com/watch?v=cdMVVQmyASU" %}

To use the newly installed MetaMask snap with Partisia Blockchain

1. Go to [Partisia Blockchain Browser Testnet](https://browser.testnet.partisiablockchain.com/) or [Partisia Blockchain Browser Mainnet](https://browser.partisiablockchain.com/)
2. In the upper right corner click Sign In
3. Click on the "MetaMask" button\
   ![browser login popup](https://partisiablockchain.gitlab.io/documentation/smart-contracts/img/metamask-snap-integration-00.png)\

4. Enter your password and click the blue unlock button
5. You have now successfully signed in to your MetaMask account through Partisia Blockchain Browser tool and can use the MetaMask Partisia Blockchain snap to sign transactions to the blockchain
6. Ensure that your PBC account has gas. If you lack gas, find the account address key inside MetaMask, then get some gas for the account. For gas on the testnet you can visit [our article about getting testnet gas](https://partisiablockchain.gitlab.io/documentation/smart-contracts/access-and-use-the-testnet.html).

We recommend you to try the "Mint 10.000 tokens" action - and examine the transaction in [Partisia Blockchain Browser](https://browser.testnet.partisiablockchain.com/transactions). If it succeeds it will appear with the Action named "Mint" in the browser.

**How to sign a transaction?**

When [signing a transaction](https://partisiablockchain.gitlab.io/documentation/smart-contracts/compile-and-deploy-contracts.html) while logged in with the MetaMask extension you will get prompted with a approval window from the extension with all the details of the transaction.

![MetaMask integration](https://partisiablockchain.gitlab.io/documentation/smart-contracts/img/metamask-snap-integration-01.png)

### Developing dApps with the MetaMask Partisia Blockchain Snap <a href="#developing-dapps-with-the-metamask-partisia-blockchain-snap" id="developing-dapps-with-the-metamask-partisia-blockchain-snap"></a>

When developing dApps or other application it can be beneficial to use the MetaMask Partisia Blockchain Snap as it creates an easy plug-and-play solution to help users sign transactions onto the chain.

As an example on how to integrate and use the snap as part of a dApp we recommend you visit our [web-client example](https://gitlab.com/partisiablockchain/language/example-web-client).

#### How do I install the Snap as a developer? <a href="#how-do-i-install-the-snap-as-a-developer" id="how-do-i-install-the-snap-as-a-developer"></a>

Using MetaMask `wallet_requestSnaps` with the snap identifier `npm:@partisiablockchain/snap`.

<details>

<summary>How to install the snap</summary>

```javascript
try {
    const result = await window.ethereum.request({
        method: 'wallet_requestSnaps',
        params: {
            'npm:@partisiablockchain/snap': {},
        },
    });
    console.log(result);
} catch (error) {
    console.log(error);
}
```

</details>

#### How do I get the Partisia Blockchain address of the user? <a href="#how-do-i-get-the-partisia-blockchain-address-of-the-user" id="how-do-i-get-the-partisia-blockchain-address-of-the-user"></a>

When the snap has been installed the snap invocation `get_address` will return the address of the user.

<details>

<summary>How to get address of the user</summary>

```javascript
window.ethereum.request({
    method: 'wallet_invokeSnap',
    params: {
        snapId: "npm:@partisiablockchain/snap",
        request: {method: 'get_address'}
    },
});
```

</details>

#### How do I sign a transaction? <a href="#how-do-i-sign-a-transaction" id="how-do-i-sign-a-transaction"></a>

Use the snap method `sign_transaction`. The method requires a parameter object with two fields, `chainId` that is the chain id of the chain that the transaction are signed towards and `payload` that should be a hex encoded transaction. The method will return a hex encoded signature.

<details>

<summary>How to create a signature</summary>

```javascript
window.ethereum.request({
    method: 'wallet_invokeSnap',
    params: {
        snapId: "npm:@partisiablockchain/snap",
        request: {
            method: 'sign_transaction',
            params: {
                payload: payload,
                chainId: chainId
            },
        },
    },
});
```

</details>
