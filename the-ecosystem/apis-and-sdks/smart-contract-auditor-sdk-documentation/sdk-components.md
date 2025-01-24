# SDK Components

## **API Key Usage**

To use the API key:

1. Execute the specified commands.
2. Paste the generated API key in the “Your ChainGPT API key” placeholder.

```javascript
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({
    apiKey: 'Your ChainGPT API Key',
});

async function main() {
    const stream = await smartcontractgenerator.createSmartContractStream({
        question: 'Write a smart contract that counts. It will have two functions.',
        chatHistory: "off"
    });
    stream.on('data', (chunk: any) => console.log(chunk.toString()));
    stream.on('end', () => console.log("Stream ended"));
}

main();

```

##

## Core Libraries

We provide TypeScript/ JavaScript libraries with support for Node.js. Install it by running:

```
npm install --save @chaingpt/smartcontractauditor
# or
yarn add smartcontractgenerator
```



Once installed, you can use the library and your secret key to run the following:

```javascript
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const stream = await smartcontractauditor.auditSmartContractStream({
    question:
      `Audit the following contract:
      pragma solidity ^0.8.0;
      contract Counter {
        uint256 private count; // This variable will hold the count
        constructor() {
          count = 0; // Initialize count to 0
        }
        function increment() public {
          count += 1;
          emit CountChanged(count); // Emit an event whenever the count changes
        }
      }`,
    chatHistory: "on"
  });

  stream.on('data', (chunk: any) => console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();
```



***

## Advanced Features&#x20;

ChainGPT Smart Contract Auditor SDK provides the following features:&#x20;

### Stream Response

Retrieve a chat response as a stream:

```javascript
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const stream = await smartcontractauditor.auditSmartContractStream({
    question: `Audit the following contract:
      pragma solidity ^0.8.0;
      contract Counter {
        uint256 private count; // This variable will hold the count
        constructor() {
          count = 0; // Initialize count to 0
        }
        function increment() public {
          count += 1;
          emit CountChanged(count); // Emit an event whenever the count changes
        }
      }`,
    chatHistory: "on"
  });

  stream.on('data', (chunk: any) => console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();
```

### &#x20;Blob Response

Retrieve the chat Response in the form of Blob&#x20;

```javascript
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const response = await smartcontractauditor.auditSmartContractBlob({
    question: `Audit the following contract:
      pragma solidity ^0.8.0;
      contract Counter {
        uint256 private count; // This variable will hold the count
        constructor() {
          count = 0; // Initialize count to 0
        }
        function increment() public {
          count += 1;
          emit CountChanged(count); // Emit an event whenever the count changes
        }
      }`,
    chatHistory: "off"
  });

  console.log(response.data.bot);
}

main();
```

### Get Chat history:

Our Smart Contract Auditor SDK also stores the chat history for later usage and users can retrieve chat history by following the code:

```javascript
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const response = await smartcontractauditor.getChatHistory({
    limit: 10,
    offset: 0,
    sortBy: "createdAt",
    sortOrder: "DESC"
  });

  console.log(response.data.rows);
}

main();
```

## Error Handling

When the library is unable to connect to the API, or if the API returns a non-success status code (i.e., 4xx or 5xx response), an error of the  class SmartContractAuditorError  will be thrown:

```javascript
import { Errors } from "@chaingpt/smartcontractauditor";

async function main() {
  try {
    const stream = await smartcontractauditor.auditSmartContractStream({
      question: `Audit the following contract:
      pragma solidity ^0.8.0;
      contract Counter {
        uint256 private count; // This variable will hold the count
        constructor() {
          count = 0; // Initialize count to 0
        }
        function increment() public {
          count += 1;
          emit CountChanged(count); // Emit an event whenever the count changes
        }
      }`,
      chatHistory: "on"
    });

    stream.on('data', (chunk: any) => console.log(chunk.toString()));
    stream.on('end', () => console.log("Stream ended"));
  } catch (error) {
    if(error instanceof Errors.SmartContractAuditorError) {
      console.log(error.message)
    }
  }
}

main();
```



***

## Language/ Framework Compatibility.

Our SDK supports Javascript language and will run on Node applications.&#x20;

## Security Considerations

1. To ensure security, the SDK is accessible using an authentication key. Users with credits in the web app and a valid API key can access the SDK.
2. Request limitations have been handled to avoid misuse. Users can make 200 requests per minute, and 1 credit will be deducted for each request.&#x20;

## Release Version&#x20;

Release history is maintained. However, this is the first release; in the future, more features will be added, and the latest version will be released for users.&#x20;

## Code Documentation&#x20;

Please check out this link for SDK code documentation&#x20;

https://www.npmjs.com/package/@chaingpt/smartcontractauditor

\
Support/ Help&#x20;
-------------------

If you need further assistance, explore the available support channels provided on our website [https://www.chaingpt.org/](https://www.chaingpt.org/) or check our [Discord](https://discord.com/invite/sv2NfqSgVW).&#x20;

\
\


\
