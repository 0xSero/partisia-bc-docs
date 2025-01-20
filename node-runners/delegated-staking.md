# Delegated Staking

Delegation of MPC tokens to a node operator is a way to stake tokens and earn [rewards](https://gitlab.com/partisiablockchain/node-operators-rewards/-/tree/main?ref_type=heads) without running a [node](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/dictionary.html#node) yourself. You can see an example of how rewards are calculated [here](https://partisiablockchain.gitlab.io/documentation/node-operations/node-payment-rewards-and-risks.html#rewards-for-delegated-tokens).

Delegated staking begins with the delegation of MPC tokens to the account of a node operator. If the node operator accepts the tokens, they have control over the tokens. This means that the node operator can associate the delegated tokens to a [node service](https://partisiablockchain.gitlab.io/documentation/node-operations/start-running-a-node.html#which-node-should-you-run). You can only retract your tokens when the node operator disassociates the tokens from node service.

As a delegator, it is your responsibility to communicate with the node operator using the tokens, if you want them to disassociate your tokens from node service.

Info

Delegation is a long-term commitment. Tokens delegated to a node operator can be locked to node services. See [restrictions on tokens and rules of retrieval](https://partisiablockchain.gitlab.io/documentation/node-operations/node-payment-rewards-and-risks.html)

#### How to delegate MPC tokens <a href="#how-to-delegate-mpc-tokens" id="how-to-delegate-mpc-tokens"></a>

Before delegating any tokens you should visit the [Staking Marketplace](https://discord.com/channels/819902335567265792/1075334307821920337), where you can find node operators interested in receiving delegated stakes.

Step by step:

1. Go to the [Delegations](https://browser.partisiablockchain.com/delegations) menu
2. Sign in
3. Locate the node operator you'd like to delegate to
4. Select the row of to your chosen node operator
5. Click the DELEGATE button on the top box in the right-hand panel
6. Enter the amount of MPC tokens you wish to delegate and click CONFIRM

Success

Your tokens are now **available** for the node operator to use. However, to reap any rewards the node operator needs to accept and associate your tokens to a job. You might need to contact them to make sure your tokens are being used.

#### How to accept delegated tokens <a href="#how-to-accept-delegated-tokens" id="how-to-accept-delegated-tokens"></a>

As a node operator, you can choose to accept the full amount or only a partial amount of a delegation offer.

Step by step:

1. Go to [Node operation](https://browser.partisiablockchain.com/node-operation)
2. Sign in
3. Click "SEE MORE" to unfold the collapsed delegation table located below the **Delegated from others** section
4. In the **Amount** column of the offer you wish to respond to, either click on the check mark icon to accept the full amount,
5. or click on the minus icon to accept a partial amount

#### How to retract delegated MPC tokens <a href="#how-to-retract-delegated-mpc-tokens" id="how-to-retract-delegated-mpc-tokens"></a>

Step by step:

1. Go to the [Delegations](https://browser.partisiablockchain.com/delegations) menu
2. Sign in
3. Locate the node operator you'd like to delegate to
4. Select the row of to your chosen node operator
5. Click the RETRACT button on the top box in the right-hand panel
6. Enter the amount of MPC tokens you wish to retract and click CONFIRM

Success

Your tokens should now be back into to your account. If this does not happen, the tokens are in use by a node service. To retract your tokens follow the steps in the [next section](https://partisiablockchain.gitlab.io/documentation/node-operations/delegated-staking.html#how-to-retract-delegated-mpc-tokens-locked-to-a-node-service)

#### How to retract delegated MPC tokens locked to a node service <a href="#how-to-retract-delegated-mpc-tokens-locked-to-a-node-service" id="how-to-retract-delegated-mpc-tokens-locked-to-a-node-service"></a>

1. Contact the node operator and ask them to disassociate your tokens from node service
2. Wait for the [pending time](https://partisiablockchain.gitlab.io/documentation/node-operations/node-payment-rewards-and-risks.html) to be over
3. Follow the steps in the previous section: [How to retract delegated MPC tokens](https://partisiablockchain.gitlab.io/documentation/node-operations/delegated-staking.html#how-to-retract-delegated-mpc-tokens)
