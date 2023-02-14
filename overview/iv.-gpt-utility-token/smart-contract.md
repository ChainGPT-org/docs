# Smart-Contract

{% embed url="https://github.com/ChainGPT-org/ChainGPT-token-contracts" %}

Audit: In process&#x20;

### Overview

ChainGPT is a decentralized finance (DeFi) token running on the Ethereum blockchain. The smart contract is designed to provide a range of features including liquidity generation, automated token swaps, and reflection. ChainGPT aims to incentivize holders by providing automatic reflections of their holdings.

### Token Information

* Token Name: ChainGPT
* Token Symbol: GPT
* Decimals: 18
* Total Supply: 100,000,000 GPT

## Features

### Liquidity Generation

ChainGPT includes a built-in liquidity generation feature that enables users to earn rewards by providing liquidity to the token’s liquidity pool. A portion of every transaction is automatically added to the liquidity pool, increasing the value of the pool and the overall liquidity of the token.

### Automated Token Swaps

ChainGPT uses PancakeSwap to enable automated token swaps. PancakeSwap is a decentralized exchange (DEX) that allows users to trade tokens without the need for an intermediary. The smart contract has the PancakeSwap router address hardcoded, so it cannot be changed.

### Reflection

ChainGPT uses a reflection mechanism that automatically distributes tokens to all holders. Every time a transaction occurs, a portion of the transaction fee is distributed to all holders, incentivizing holding of the token.

### Fees

* 1% of each transaction is taken as a fee and added to the liquidity pool.
* 0.5% of each transaction is taken as a fee and distributed to all holders as a reflection.
* 0.5% of each transaction is taken as a fee and added to the administration wallet.
* The maximum total fees combined are limited to 0-25%.

### Wallet Limits

The contract includes wallet limits that restrict the amount of tokens that can be traded in a single transaction.

### Swap Helper

ChainGPT uses a Swap Helper contract that enables users to swap tokens directly for the smart contract. The Swap Helper contract is an intermediary between the user and the smart contract, allowing for a seamless exchange of tokens.

### Contract Addresses

* Contract Address: 0x...
* PancakeSwap Router Address: 0x10ED43C718714eb63d5aA57B78B54704E256024E
* WBNB Address: 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c

### Public Functions

The smart contract includes a range of public functions that enable users to interact with the contract.

* **getFeesTotal()**\
  ****Returns the total fee percentage of the token.
* **getPoolFee()**\
  ****Returns the fee percentage allocated to the liquidity pool.
* **getReflectFee()**\
  ****Returns the fee percentage allocated to the reflection mechanism.
* **getAdminFee()**\
  ****Returns the fee percentage allocated to the administration wallet.
* **getSpecialWalletFee(address target, bool isSender)**\
  ****Returns the fee percentage of a specific wallet.
* **setWalletFee(address target, bool isSender, uint fee)**\
  ****Sets the fee percentage of a specific wallet.
* **setSpecialWalletFee(address target, bool isSender, uint fee)**\
  ****Sets the fee percentage of a specific wallet, and marks it as a special wallet.
* **setFees(uint \_poolFee, uint \_reflectFee, uint \_administrationFee)**\
  ****Sets the total fee percentage of the token.
* **setPausedSwapPool(bool \_pausedSwapPool)**\
  ****Disables or enables the pool fees as token instead of WBNB.
* **setPausedSwapAdmin(bool \_pausedSwapAdmin)**\
  ****Disables or enables the admin fees as token instead of WBNB.
* **setExemptReflect(uint attributes, bool value)**\
  ****Sets the reflection exemption attribute for an address.
* **setExemptFeeSender (uint attributes, bool value)**\
  ****Sets the fee sender exemption attribute for an address. \
  If an address has the fee sender attribute set to true, \
  it will not be charged any transaction fees.
* **Events**\
  ****The following events are emitted by the contract:

### **Transfer**

**\`**event Transfer(address indexed sender, address indexed recipient, uint256 amount)\` \
Emitted when tokens are transferred from one address to another.

* sender: The address tokens are being transferred from.
* recipient: The address tokens are being transferred to.
* amount: The amount of tokens being transferred.

### **Approval**

`event Approval(address indexed owner, address indexed spender, uint256 amount)` Emitted when an address is approved to spend tokens on behalf of another address.

* owner: The address that approves the spending.
* spender: The address that is being approved to spend tokens.
* amount: The amount of tokens being approved for spending.

### **Burn**

`event Burn(address indexed burner, uint256 amount)` Emitted when tokens are burned.

* burner: The address that burned tokens.
* amount: The amount of tokens that were burned.

### **OwnershipTransferred**

`event OwnershipTransferred(address indexed previousOwner, address indexed newOwner)` Emitted when ownership of the contract is transferred from one address to another.

* previousOwner: The address of the previous owner.
* newOwner: The address of the new owner.

### Conclusion

This smart contract provides a basic implementation of an ERC-20 token with additional features such as burning and fee exemption. It can be used as a starting point for creating a custom token with more advanced functionality. Please note that this documentation is not intended to be a comprehensive guide to smart contract development, and it is recommended that you have a solid understanding of Solidity and smart contract development before attempting to modify this contract.
