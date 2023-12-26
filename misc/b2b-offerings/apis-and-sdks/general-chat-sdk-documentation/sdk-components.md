# SDK Components

## **API Key Usage**

To use the API key:

1. Execute the specified commands.
2. Paste the generated API key in the placeholder “Your ChainGPT API key”.

<figure><img src="https://lh7-us.googleusercontent.com/DMRY_K_Y2k9ryPzvu0xEszYimwpOgDX0xY5cAtP88rpC7pIheE6O2tLCpCLHfcB0VKxrXqIhFXEbSYTNi6LKCbVo2Rxg76NR4CVe4BvQ3Lo8FapLX1pIHmQhloCRMVv3JnI-0DW-sjSejT8TmlYRObk" alt="" width="563"><figcaption></figcaption></figure>

## **3. SDK Components**

### **3.1 Core Libraries**

Our General Chatbot SDK offers TypeScript/JavaScript libraries compatible with Node.js. To install:

1. Run the installation command.

<figure><img src="https://lh7-us.googleusercontent.com/Xgo8jTi9sb24C9_bs2MIU80xTDLonxUNyMQJaLTrRh1TX5RAh6Ba3ZzHE7BUAfzz1c19bRppOSnpBTOwTTnFJxpeV4YI6MBjaTFxU3U8dqWw9Yb-wLa4BCm-DfzA7xqvRnOmNcZjJ-U0RfPuE6ZJUHk" alt="" width="563"><figcaption></figcaption></figure>

2. Use the library along with your secret key to execute further operations.

<figure><img src="https://lh7-us.googleusercontent.com/uAoAKPdkw0zfXngTgAp2SBQVPb-C4Mkme326H2mD2UDXqY7fkQAeOrLRHe81v2aouBmhUAv0x4pfSPSaUrsq2ojVNrj6V0NvLRqBFgTdzHiR1SSgGft6i51lFBxxYGQE04TbHWKm3lO-8auWEnYnku0" alt="" width="563"><figcaption></figcaption></figure>

### **3.2 Advanced Features**

ChainGPT General Chatbot SDK provides the following features:

#### **Stream Response**

* **Functionality**: Retrieve a chat response as a stream.

<figure><img src="https://lh7-us.googleusercontent.com/0-u71iPagP2ijcG0EvGS1zRPOPhpyLna6K6Dq1QEmT8ejTVJ6qvQx6YC0rFuIIaTVhdxWhAv4d_lK-61e7NYgV9Ndbx9UFOKVacifr8clhzdiwVoCvZ5jD0LrIy-k1G1XbkMG_5BymwRFKca41b4G5U" alt="" width="563"><figcaption></figcaption></figure>

* **Credit Deduction**: Each chat API request deducts 1 credit from the user's account in the WebApp. An additional credit is deducted if the chat history feature is enabled.

<figure><img src="https://lh7-us.googleusercontent.com/vFx3oM3HmSdyEw_IWKpU8q_UvKgLUsGR53T2dpGQ-ccDsIOhSwkCoPHCs2o7rsrFoYpejeOimn-OmZ4UQQBqiSgtHKqAo5elhmpChvuO3lptfmfIunctdLwjvl70_eYTlECezadaGblh_ToEwXfNcIs" alt="" width="563"><figcaption></figcaption></figure>

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
