# SDK Components

## **API Key Usage**

To use the API key:

1. Execute the specified commands.
2. Paste the generated API key in the placeholder “Your ChainGPT API key”.

## **3. SDK Components**

### **3.1 Core Libraries**

Our SDK offers TypeScript/JavaScript libraries with support for Node.js. To install:

1. Run the installation command.
2. Use the library and your secret key for further operations once installed.

### **3.2 Advanced Features**

ChainGPT AI News SDK provides several advanced features:

* **Category-Based News Filtering**: Over 20 categories are available, including Blockchain Gaming, DAO, DApps, DeFi, Lending, Metaverse, NFT, Stablecoins, Cryptocurrency, Smart Contracts, etc.
* **Subcategory-Based News Search**: More than 50 subcategories, such as Bitcoin, BNB Chain, Celo, Solana, TRON, etc.
* **Title and Description Search**: Enables news search based on titles and descriptions.
* **Token-Specific News**: Fetch news for various tokens.
* **Date-Based News Retrieval**: Acquire AI news from a specific date, sorted by creation or publication date.
* **ID Access for Categories, Subcategories, and Tokens**: Users can access these using specific IDs.

### **3.3 Error Handling**

In case of connection issues to the API or non-success status codes (4xx or 5xx responses), an `AINewsError` class error will be thrown.

### **3.4 Language/Framework Compatibility**

Our SDK is compatible with JavaScript and is designed to run on Node.js applications.

### **3.5 Security Considerations**

* **Authentication**: SDK access is secured via an authentication key.
* **Credits System**: Users with credits and a valid API key can access the SDK.
* **Request Limitation**: To prevent misuse, there is a limit of 200 requests per minute, with 1 credit deducted per request.

### **3.6 Release Version**

* **Release History**: We maintain a release history.
* **First Release**: This is the initial release, with plans for future enhancements.

### **3.7 Code Documentation**

For detailed SDK code documentation, visit [npm package documentation](https://www.npmjs.com/package/@chaingpt/ainews?activeTab=readme).

### **3.8 Support/Help**

For additional assistance, refer to the support channels on our website [https://www.chaingpt.org/](https://www.chaingpt.org/).
