# Storage Gas Prices

When data is stored on a Partisia blockchain, it incurs a running gas cost. Storage gas is inherently not tied to any transaction. For contracts on the blockchain, each contract has an associated account balance. This balance is used to cover the storage gas fees.

{% hint style="info" %}
**S**T**ORAGE FEE**

1 USD cent/kb per year
{% endhint %}

#### Negative contract gas balance <a href="#negative-contract-gas-balance" id="negative-contract-gas-balance"></a>

If a contract's gas balance becomes negative the contract is deleted. This is done by one of two possible methods.

1. The account management plugin within the blockchain system will eventually delete the contract if its balance does not stay above a positive value.
2. When a new transaction is received by the contract, the blockchain checks if the gas balance is negative. If it is, the contract will be deleted before the transaction can be processed by the contract.
