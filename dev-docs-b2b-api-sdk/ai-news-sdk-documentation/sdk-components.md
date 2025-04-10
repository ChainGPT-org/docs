# SDK Components

## **API Key Usage**

To use the API key:

1. Execute the specified commands.
2. Paste the generated API key in the placeholder “Your ChainGPT API key”.

```javascript
import { AInews } from '@chaingpt/ainews';

const ainews = new AInews({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const response = await ainews.getNews({});
  console.log(response.data.rows);
}

main();
```

## **3. SDK Components**

### **3.1 Core Libraries**

Our SDK offers TypeScript/JavaScript libraries with support for Node.js. To install:

1. Run the installation command.

```plaintext
npm install --save @chaingpt/ainews
# or
yarn add ainews
```

2. Use the library and your secret key for further operations once installed.

```javascript
import { AInews } from '@chaingpt/ainews';

const ainews = new AInews({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const response = await ainews.getNews({});
  console.log(response.data.rows);
}

main();
```

### **3.2 Advanced Features**

ChainGPT AI News SDK provides several advanced features:

* **Category-Based News Filtering**: Over 20 categories are available, including Blockchain Gaming, DAO, DApps, DeFi, Lending, Metaverse, NFT, Stablecoins, Cryptocurrency, Smart Contracts, etc.
* **Subcategory-Based News Search**: More than 50 subcategories, such as Bitcoin, BNB Chain, Celo, Solana, TRON, etc.
* **Title and Description Search**: Enables news search based on titles and descriptions.
* **Token-Specific News**: Fetch news for various tokens.
* **Date-Based News Retrieval**: Acquire AI news from a specific date, sorted by creation or publication date.
* **ID Access for Categories, Subcategories, and Tokens**: Users can access these using specific IDs.

```javascript
ainews.getNews({
  fetchAfter: new Date(''), // Fetch news after specific date
  categoryId: [1], // Search AI News against specific category i.e NFT, Blockchain, DEFI etc
  subCategoryId: [1], // Search AI News against subcategory i.e BNB chain, CELO, COSMOS etc.
  searchQuery: "search text", // Search for News on the basis of title and description
  limit: 10,
  offset: 0,
  tokenId: [1], // Search AINews against token i.e. BNB, Ethereum, Bitcoin etc
  sortBy: 'createdAt', // sorting News by created date
}).then((res) => {
  console.log(res)
}).catch((err) => {
  if (err instanceof nErrors.AINewsError) {
    console.log(err.message);
  }
});
```



#### **Users can access different categories using the following IDs:**

| Category                    | ID |
| --------------------------- | -- |
| Blockchain Gaming           | 2  |
| DAO                         | 3  |
| DApps                       | 4  |
| DeFi                        | 5  |
| Lending                     | 6  |
| Metaverse                   | 7  |
| NFT                         | 8  |
| Stablecoins                 | 9  |
| Cryptocurrency              | 64 |
| Decentralized               | 65 |
| Smart Contracts             | 66 |
| Distributed Ledger          | 67 |
| Cryptography                | 68 |
| Digital Assets              | 69 |
| Tokenization                | 70 |
| Consensus Mechanisms        | 71 |
| ICO (Initial Coin Offering) | 72 |
| Crypto Wallets              | 73 |
| Web3.0                      | 74 |
| Interoperability            | 75 |
| Mining                      | 76 |
| Cross-Chain Transactions    | 77 |
| Exchange                    | 78 |



#### **Users can access the following sub-categories using the specified IDs:**

| Sub-Category        | ID |
| ------------------- | -- |
| Bitcoin             | 11 |
| BNB Chain           | 12 |
| Celo                | 13 |
| Cosmos              | 14 |
| Ethereum            | 15 |
| Fantom              | 16 |
| Flow                | 17 |
| Litecoin            | 18 |
| Monero              | 19 |
| Polygon             | 20 |
| XRP Ledger          | 21 |
| Solana              | 22 |
| Tron                | 23 |
| Terra               | 24 |
| Tezos               | 25 |
| WAX                 | 26 |
| Algorand            | 27 |
| Arbitrum            | 28 |
| Astar               | 29 |
| Aurora              | 30 |
| Avalanche           | 31 |
| Base                | 32 |
| Binance Smart Chain | 33 |
| Cardano             | 34 |
| Celo                | 35 |
| Cronos              | 36 |
| Moonbeam            | 37 |
| Dep                 | 38 |
| Ethereum            | 39 |
| Fantom              | 40 |
| Harmony             | 41 |
| Oasis               | 42 |
| Oasis Sapphire      | 43 |
| Ontology            | 44 |
| Optimism            | 45 |
| Other               | 46 |
| Platon              | 47 |
| Polygon             | 48 |
| Rangers             | 49 |
| Ronin               | 50 |
| Shiden              | 51 |
| Skale               | 52 |
| Solana              | 53 |
| Stacks              | 54 |
| Stargaze            | 55 |
| Steem               | 56 |
| SXNetwork           | 57 |
| Telos               | 58 |
| TelosEVM            | 59 |
| Tezos               | 60 |
| Theta               | 61 |
| Thundercore         | 62 |



#### Users can access the following tokens using the specified IDs:

| Token                  | ID  |
| ---------------------- | --- |
| Bitcoin - BTC          | 79  |
| Ethereum - ETH         | 80  |
| Tether USDt - USDT     | 81  |
| BNB - BNB              | 82  |
| XRP - XRP              | 83  |
| USDC - USDC            | 84  |
| Solana - SOL           | 85  |
| Cardano - ADA          | 86  |
| Dogecoin - DOGE        | 87  |
| TRON - TRX             | 88  |
| Toncoin - TON          | 89  |
| Dai - DAI              | 90  |
| Polygon - MATIC        | 91  |
| Polkadot - DOT         | 92  |
| Litecoin - LTC         | 93  |
| Wrapped Bitcoin - WBTC | 94  |
| Bitcoin Cash - BCH     | 95  |
| Chainlink - LINK       | 96  |
| Shiba Inu - SHIB       | 97  |
| UNUS SED LEO - LEO     | 98  |
| TrueUSD - TUSD         | 99  |
| Avalanche - AVAX       | 100 |
| Stellar - XLM          | 101 |
| Monero - XMR           | 102 |
| OKB - OKB              | 103 |
| Cosmos - ATOM          | 104 |
| Uniswap - UNI          | 105 |
| Ethereum Classic - ETC | 106 |
| BUSD - BUSD            | 107 |
| Hedera - HBAR          | 108 |

### **3.3 Error Handling**

In case of connection issues to the API or non-success status codes (4xx or 5xx responses), an `AINewsError` a class error will be thrown.

```javascript
import { Errors } from '@chaingpt/ainews';

async function main() {
  try {
    const response = await ainews.getNews({});
    console.log(response.data.rows);
  } catch (error) {
    if (error instanceof Errors.AINewsError) {
      console.log(error.message)
    }
  }
}

main();
```

### **3.4 Language/Framework Compatibility**

Our SDK is compatible with JavaScript and is designed to run on Node.js applications.

### **3.5 Security Considerations**

* **Authentication**: SDK access is secured via an authentication key.
* **Credits System**: Users with credits and a valid API key can access the SDK.
* **Request Limitation**: To prevent misuse, 200 requests per minute are limited, with 1 credit deducted per request.

### **3.6 Release Version**

* **Release History**: We maintain a release history.
* **First Release**: This is the initial release, with plans for future enhancements.

### **3.7 Code Documentation**

For detailed SDK code documentation, visit [npm package documentation](https://www.npmjs.com/package/@chaingpt/ainews?activeTab=readme).

### **3.8 Support/Help**

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)



[**Disclaimer**](../../misc/legal-docs/disclaimer.md)
