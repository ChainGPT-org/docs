# SDK Components

### API Key Usage&#x20;

To use the API key:

1. Execute the specified commands.
2. Paste the generated API key in the placeholder “Your ChainGPT API key”.

```javascript
import { SmartContractGenerator } from '@chaingpt/smartcontractgenerator';

const smartcontractgenerator = new SmartContractGenerator({
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

## Advanced Features&#x20;

ChainGPT smart contract generator SDK provides the following features&#x20;

### Stream Response

Retrieve a chat response as a stream:

```javascript
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const stream = await smartcontractgenerator.createSmartContractStream({
    question: 'Write a smart contract that counts. It will have two functions. One for incrementing. Other for decrementing.',
    chatHistory: "off"
  });
  stream.on('data', (chunk: any) => console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();
```

### Blob Response

Retrieve the chat Response in the form of Blob:

```javascript
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const response = await smartcontractgenerator.createSmartContractBlob({
    question: 'Write a smart contract that counts. It will have two functions. One for incrementing. Other for decrementing.',
    chatHistory: "off"
  })
  console.log(response.data.bot);
}

main();
```



### Get Chat History:

Our Smart Contract Generator SDK also stores the chat history for later usage and users can retrieve chat history by following code:

```javascript
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const response = await smartcontractgenerator.getChatHistory({
    limit: 10,
    offset: 0,
    sortBy: "createdAt",
    sortOrder: "DESC"
  })
  console.log(response.data.rows);
}

main();
```

### &#x20;Error Handling

When the library is unable to connect to the API, or if the API returns a non-success status code (i.e., 4xx or 5xx response), an error of the  class SmartContractGeneratorError will be thrown:

```javascript
import { Errors } from '@chaingpt/smartcontractgenerator';

async function main() {
  try {
    const stream = await smartcontractgenerator.createSmartContractStream({
      question: 'Write a smart contract that counts. It will have two functions. One for incrementing. Other for decrementing.',
      chatHistory: "on"
    });
    stream.on('data', (chunk: any) => console.log(chunk.toString()));
    stream.on('end', () => console.log("Stream ended"));
  } catch(error) {
    if(error instanceof Errors.SmartContractGeneratorError) {
      console.log(error.message)
    }
  }
}

main();
```

### Language/ Framework Compatibility.

Our SDK supports Javascript language and will run on Node application/



### Security Considerations

1. To ensure security, the SDK is accessible using an authentication key. Users with credits in the web app and a valid API key can access the SDK.
2. Request limitations have been handled to avoid misuse. Users can make 200 requests per minute, and 1 credit will be deducted for each request.&#x20;

\
Release Version&#x20;
---------------------

Release history is maintained. However, this is the first release; in the future, more features will be added, and the latest version will be released for users.&#x20;

### &#x20; Code Documentation&#x20;

Please check out this link for SDK code documentation.&#x20;

[https://www.npmjs.com/package/@chaingpt/smartcontractgenerator](https://www.npmjs.com/package/@chaingpt/smartcontractgenerator)

###

### **Support/Help**

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)



[**Disclaimer**](../../../misc/legal-docs/disclaimer.md)\




\


\
&#x20;
