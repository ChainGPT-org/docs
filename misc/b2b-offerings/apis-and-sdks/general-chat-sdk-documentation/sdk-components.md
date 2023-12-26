# SDK Components

## **API Key Usage**

To use the API key:

1. Execute the specified commands.
2. Paste the generated API key in the placeholder “Your ChainGPT API key”.

## **3. SDK Components**

### **3.1 Core Libraries**

Our General Chatbot SDK offers TypeScript/JavaScript libraries compatible with Node.js. To install:

1. Run the installation command.
2. Use the library along with your secret key to execute further operations.

### **3.2 Advanced Features**

ChainGPT General Chatbot SDK provides the following features:

#### **Stream Response**

* **Functionality**: Retrieve a chat response as a stream.
* **Credit Deduction**: Each chat API request deducts 1 credit from the user's account in the WebApp. An additional credit is deducted if the chat history feature is enabled.

#### **Blob Response**

* **Functionality**: Retrieve the chat response in the form of a Blob.
* **Credit Deduction**: Same as Stream Response.

#### **Get Chat History**

* **Functionality**: The SDK stores chat history for later use, retrievable by specific code.
* **Credit Deduction**: No additional credits are deducted if the chat history feature was enabled during Stream/Blob response API calls.

### **3.3 Error Handling**

If there is a failure in connecting to the API, or if the API returns a non-success status code (4xx or 5xx), a `GeneralChatError` class error will be thrown.

### **3.4 Language/Framework Compatibility**

Our SDK is compatible with JavaScript and is designed for Node.js applications.

### **3.5 Security Considerations**

* **Authentication**: Access to the SDK is secured via an authentication key.
* **Credits System**: Users with credits and a valid API key can access the SDK.
* **Request Limitation**: To prevent misuse, there is a limit of 200 requests per minute, with 1 credit deducted per request.

### **3.6 Release Version**

* **Release History**: We maintain a detailed release history.
* **First Release**: This is the initial release, with future updates planned for additional features.

### **3.7 Code Documentation**

For detailed SDK code documentation, visit [npm package documentation](https://www.npmjs.com/package/@chaingpt/generalchat).

### **3.8 Support/Help**

For additional assistance, refer to the support channels on our website [https://www.chaingpt.org/](https://www.chaingpt.org/).
