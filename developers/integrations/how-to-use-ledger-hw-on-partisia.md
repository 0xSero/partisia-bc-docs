# How to Use Ledger HW on Partisia

Ledger is a hardware wallet that is considered one of the most secure ways to store your digital assets. Ledger uses an offline, or cold storage, method of generating private keys. Ledger is integrated with our block explorer [(Browser)](https://browser.partisiablockchain.com/account).

Install the Partisia Blockchain app on your Ledger device to sign transactions and manage MPC tokens with the [Partisia Blockchain Browser](https://browser.partisiablockchain.com/account). The Partisia Blockchain app is developed and supported by the [Partisia Blockchain Foundation](https://partisiablockchain.com/).

Requirements: What's needed before starting

* You've initialized your Ledger hardware wallet.
* The latest firmware is installed.
* Ledger Live is ready to use.

### Install Partisia Blockchain App on Ledger device <a href="#install-partisia-blockchain-app-on-ledger-device" id="install-partisia-blockchain-app-on-ledger-device"></a>

1. Open My Ledger in Ledger Live.
2. Connect and unlock your Ledger device.
3. If asked, allow My Ledger to access your device.
4. Find Partisia Blockchain in the app catalog.
5. Click the Install button of the app.
6. Follow the onscreen instructions on the Ledger device.

### How to connect the Ledger device with Partisia Blockchain Browser? <a href="#how-to-connect-the-ledger-device-with-partisia-blockchain-browser" id="how-to-connect-the-ledger-device-with-partisia-blockchain-browser"></a>

To connect your device with the Browser you need to have gone through all steps in the above guide.

1.  Enter the pin on your Ledger device&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/login(1)-enter-pin.png" alt=""><figcaption></figcaption></figure>
2.  Choose Partisia Blockchain in the Choose app setting

    ![login(2)-choose-app](https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/login\(2\)-choose-app.png)
3. App is now ready on the Ledger and Ledger can now login using [the Browser](https://browser.partisiablockchain.com)
4.  In the top right corner of the Browser you can click _Sign In_ this gives you a menu where you can click _Sign in using Ledger_&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/login(4)-login%20menu.png" alt=""><figcaption></figcaption></figure>
5.  You can now see a quick loading screen coming up where it signs you into your Partisia Blockchain Browser with the Ledger device.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/login(5)-logging%20in.png" alt=""><figcaption></figcaption></figure>

In the top right corner of browser you can copy and see the [PBC address](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#address) of your Ledger. You can also visit the [account page](https://browser.partisiablockchain.com/account) to get more detailed viewing of the accounts balance.

### How to receive crypto assets using Ledger <a href="#how-to-receive-crypto-assets-using-ledger" id="how-to-receive-crypto-assets-using-ledger"></a>

To receive crypto assets you need to save the address of the ledger. This needs to be the receiving end of a MPC transfer for Ledger to get assets on the blockchain.

1. Connect your ledger account with [Partisia Blockchain Browser](https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/how-to-use-ledger-on-partisia-blockchain.html#how-to-connect-the-ledger-device-with-partisia-blockchain-browser).
2.  In [your account](https://browser.partisiablockchain.com/account) find the PBC address of your ledger.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/browserPBCAddress.png" alt=""><figcaption></figcaption></figure>
3. Share your PBC address to receive crypto assets on your ledger device.

### How to send crypto assets using Ledger <a href="#how-to-send-crypto-assets-using-ledger" id="how-to-send-crypto-assets-using-ledger"></a>

After [connecting with browser](https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/how-to-use-ledger-on-partisia-blockchain.html#how-to-connect-the-ledger-device-with-partisia-blockchain-browser) you are now ready to start moving assets around using the Ledger.

1.  Go to the [Your account](https://browser.testnet.partisiablockchain.com/account) where you can press the transfer button to send MPC tokens. You need to fill out the receiving [address](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html) and the amount of MPC tokens you want to send.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(1)-mpc%20token%20transfer.png" alt=""><figcaption></figcaption></figure>
2.  After sending the transaction to transfer MPC tokens, you can see that the Browser waits for approval on the Ledger&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(2)-waiting%20for%20approval.png" alt=""><figcaption></figcaption></figure>
3.  Review the MPC transfer on the Ledger device by pressing the _right button_ on your Ledger Device and review the different transaction details.&#x20;

    The review consists of:

    *   Are you using the correct chain? Ledger will show which chain you are using: "Partisia Blockchain mainnet" or "Partisia Blockchain testnet".&#x20;

        <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/Interact(4)-Chain.png" alt=""><figcaption></figcaption></figure>
    *   Are you using the correct arguments? Its important that you verify the receiving addresses on the device as to ensure you are doing the transaction you wanted with the amount you wanted to transfer.&#x20;

        <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/Interact(5)-Arguments.png" alt=""><figcaption></figcaption></figure>
    *   If you cannot verify the address of the amount you can reject the transaction on the Ledger.&#x20;

        <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(8)-reject.jpg" alt=""><figcaption></figcaption></figure>

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(3)-review.jpg" alt=""><figcaption></figcaption></figure>
4.  Accept the amount of [gas](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#gas) the transaction costs.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(6)-fee.jpg" alt=""><figcaption></figcaption></figure>
5.  If you want to approve after the review you should click on _Approve_ on the device. Alternatively you should click _Reject_.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(7)-approve.jpg" alt=""><figcaption></figcaption></figure>
6.  After approval, you can see that the Browser finishes the transaction and the transfer is complete.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(9)-sending%20transaction.png" alt=""><figcaption></figcaption></figure>

### How to send blind signed transactions? <a href="#how-to-send-blind-signed-transactions" id="how-to-send-blind-signed-transactions"></a>

If you want to use Ledger to sign all possible transactions on Partisia Blockchain, you need to enable blind signing on your device. This guide will teach you how to enable blind signing on your ledger device and how to do an ETH token transfer to another address on PBC with a blind signed transaction.

1.  Go to settings on your Ledger device&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/enabling-blindsigning(1)-settings.png" alt=""><figcaption></figcaption></figure>
2.  Default for Ledger is that blind signing is disabled.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/enabling-blindsigning(2)-default-not-enabled.png" alt=""><figcaption></figcaption></figure>
3.  To enable blind signing click on the right button for enabling blind signing on the Ledger device.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/enabling-blindsigning(3)-enabled.jpg.png" alt=""><figcaption></figcaption></figure>

This setting update changes how Ledger signs transactions on Partisia Blockchain. The following steps shows how a blind signed transaction looks like.

1.  After signing into [browser with the Ledger device](https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/how-to-use-ledger-on-partisia-blockchain.html#how-to-connect-the-ledger-device-with-partisia-blockchain-browser), we go to the [ETH token contract](https://browser.partisiablockchain.com/contracts/014a6d0fd09fe2e6853a76caedcb46646ab7ee69d6) where you can [transfer ETH tokens](https://browser.partisiablockchain.com/contracts/014a6d0fd09fe2e6853a76caedcb46646ab7ee69d6/transfer). You need to fill out the receiving [address](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html) and the amount of ETH tokens you want to send to another address on Partisia Blockchain.

    Remember the amount of ETH tokens needs to have [18 decimals](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/byoc/bridging-byoc-by-sending-transactions.html#bridgeable-coins-on-mainnet) behind, e.g. if you want to transfer 10 ETH BYOC Tokens you would type: 10000000000000000000 in the transfer action.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/blindsign(1)-transfer-eth.png" alt=""><figcaption></figcaption></figure>
2.  After sending the transaction to transfer tokens, you can see that the Browser waits for approval on the Ledger&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(2)-waiting%20for%20approval.png" alt=""><figcaption></figcaption></figure>
3.  We now need to review the transaction on the Ledger&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/blindsign(2)-transaction.png" alt=""><figcaption></figcaption></figure>
4.  Ledger gives a warning on the device because of the blind signing. Ledger asks if you trust the content of the RPC sent on chain.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/blindsign(3)-warning.png" alt=""><figcaption></figcaption></figure>
5.  In this review stage of using the ledger we need to verify that it is the right contract we are using. Its always important to rigorously review that the available information is correct when blind signing a transaction with Ledger.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/blindsign(4)-contract.png" alt=""><figcaption></figcaption></figure>
6.  We need to accept the fee payment&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/blindsign(5)-fee.png" alt=""><figcaption></figcaption></figure>
7.  If you want to approve after the review you should click on _Approve_ on the device. Alternatively you should click _Reject_.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(7)-approve.jpg" alt=""><figcaption></figcaption></figure>
8.  After approval, you can see that the Browser finishes the transaction and the transfer is complete.&#x20;

    <figure><img src="https://partisiablockchain.gitlab.io/documentation/smart-contracts/integration/ledger/interact(9)-sending%20transaction.png" alt=""><figcaption></figcaption></figure>

If you need help in any of the above explained steps, you should go to the [community Discord](https://partisiablockchain.gitlab.io/documentation/get-support-from-pbc-community.html) where you are able to create support tickets and get help from the Partisia Blockchain community.

If you want to integrate Ledger with your dApp we urge you to reach out in the dev-chat on [Discord](https://partisiablockchain.gitlab.io/documentation/get-support-from-pbc-community.html).
