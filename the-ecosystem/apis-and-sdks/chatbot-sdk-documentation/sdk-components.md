# SDK Components

## **API Key Usage**

To use the API key:

1. Execute the specified commands.
2. Paste the generated API key in the placeholder “Your ChainGPT API key”.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>Please add your API Key</p></figcaption></figure>

```javascript
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const stream = await generalchat.createChatStream({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off"
  });
  stream.on('data', (chunk: any) => console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();

```

## **3. SDK Components**

### **3.1 Core Libraries**

Our General Chatbot SDK offers TypeScript/JavaScript libraries compatible with Node.js. To install:

1. Run the installation command.

```
npm install --save @chaingpt/generalchat
# or
yarn add generalchat
```

2. Use the library along with your secret key to execute further operations.

```javascript
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const stream = await generalchat.createChatStream({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off"
  });
  stream.on('data', (chunk: any) => console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();
```

### **3.2 Advanced Features**

ChainGPT General Chatbot SDK provides the following features:

#### **Stream Response**

* **Functionality**: Retrieve a chat response as a stream.

```javascript
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const stream = await generalchat.createChatStream({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off"
  });
  stream.on('data', (chunk: any) => console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();
```

* **Credit Deduction**: Each chat API request deducts 1 credit from the user's account in the WebApp. An additional credit is deducted if the chat history feature is enabled.

<figure><img src="https://lh7-us.googleusercontent.com/vFx3oM3HmSdyEw_IWKpU8q_UvKgLUsGR53T2dpGQ-ccDsIOhSwkCoPHCs2o7rsrFoYpejeOimn-OmZ4UQQBqiSgtHKqAo5elhmpChvuO3lptfmfIunctdLwjvl70_eYTlECezadaGblh_ToEwXfNcIs" alt="" width="563"><figcaption></figcaption></figure>

#### **Blob Response**

* **Functionality**: Retrieve the chat response in the form of a Blob.

```javascript
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const response = await generalchat.createChatBlob({
    question: 'Explain quantum computing in simple termsQQQ',
    chatHistory: "on"
  })
  console.log(response.data.bot);
}

main();
```

* **Credit Deduction**: Same as Stream Response.

<figure><img src="https://lh7-us.googleusercontent.com/vFx3oM3HmSdyEw_IWKpU8q_UvKgLUsGR53T2dpGQ-ccDsIOhSwkCoPHCs2o7rsrFoYpejeOimn-OmZ4UQQBqiSgtHKqAo5elhmpChvuO3lptfmfIunctdLwjvl70_eYTlECezadaGblh_ToEwXfNcIs" alt="" width="563"><figcaption></figcaption></figure>

#### **Get Chat History**

* **Functionality**: The SDK stores chat history for later use, retrievable by specific code.

```javascript
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const response = await generalchat.getChatHistory({
    limit: 10,
    offset: 0,
    sortBy: "createdAt",
    sortOrder: "ASC"
  })
  console.log(response.data.rows);
}

main();
```

* **Credit Deduction**: No additional credits are deducted if the chat history feature was enabled during Stream/Blob response API calls.

### **3.3 Error Handling**

If there is a failure in connecting to the API or if the API returns a non-success status code (4xx or 5xx), a `GeneralChatError` class error will be thrown.

```javascript
import { Errors } from "@chaingpt/generalchat";

async function main() {
  try {
    const stream = await generalchat.createChatStream({
      question: 'Explain quantum computing in simple terms',
      chatHistory: "off"
    });
    stream.on('data', (chunk: any) => console.log(chunk.toString()));
    stream.on('end', () => console.log("Stream ended"));
  } catch (error) {
    if (error instanceof Errors.GeneralChatError) {
      console.log(error.message)
    }
  }
}

main();
```

### **3.4 Language/Framework Compatibility**

Our SDK is compatible with JavaScript and is designed for Node.js applications.

### **3.5 Security Considerations**

* **Authentication**: Access to the SDK is secured via an authentication key.
* **Credits System**: Users with credits and a valid API key can access the SDK.
* **Request Limitation**: To prevent misuse, 200 requests per minute are limited, with 1 credit deducted per request.

### **3.6 Release Version**

* **Release History**: We maintain a detailed release history.
* **First Release**: This is the initial release, with future updates for additional features.

### **3.7 Code Documentation**

For detailed SDK code documentation, visit [npm package documentation](https://www.npmjs.com/package/@chaingpt/generalchat).

### **3.8 Support/Help**

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)



[**Disclaimer**](../../../misc/legal-docs/disclaimer.md)
