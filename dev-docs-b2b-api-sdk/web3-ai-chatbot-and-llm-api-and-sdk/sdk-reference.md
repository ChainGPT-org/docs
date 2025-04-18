# SDK Reference

## ChainGPT Web3 AI Chatbot - SDK Reference

The ChainGPT Web3 AI Chatbot SDK is a Node.js library that provides convenient methods to integrate ChainGPT's AI chatbot capabilities into your application. It allows you to retrieve chatbot responses as a real-time stream or as a complete result, with support for custom context injection and other configuration options.

{% hint style="info" %}
**Prerequisites:** Ensure you have Node.js installed and have obtained a ChainGPT API key with available credits from the [ChainGPT AI Hub](https://app.chaingpt.org/). You will need an active ChainGPT account with credits to use the SDK.
{% endhint %}

***

### Quickstart

Get started with a simple chatbot query. The example below shows how to install the package, initialize the SDK with your API key, and make a basic request for a chatbot response:

```
npm install @chaingpt/generalchat

# or 
yarn add @chaingpt/generalchat
```

```
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'YOUR_CHAINGPT_API_KEY', // replace with your actual API key
});

async function main() {
  const response = await generalchat.createChatBlob({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off"  // set to "on" to enable saving chat history (optional)
  });
  console.log(response.data.bot);
}

main();
```

Running this code will print the chatbot's answer to the console. For a streaming response example, see the **Usage Patterns** section.

***

### Table of Contents

* Installation
* Authentication & Credits
* Configuration Options
* Usage Patterns
* Error Handling
* Rate Limits & Pricing
* Language Support & Environment
* Security Considerations
* Further Reading & Support

***

### Installation

To install the ChainGPT Web3 AI Chatbot SDK, use npm (or yarn) in your Node.js project:

```
npm install --save @chaingpt/generalchat
```

This will add the SDK to your project’s dependencies. Once installed, you can import the SDK in your code (ESM syntax is used as shown in the examples).

***

### Authentication & Credits

ChainGPT’s API uses API keys and a credit-based system for authentication. Before using the SDK, make sure you have:

* **API Key:** Generate a secret API key from the ChainGPT AI Hub (go to the API Dashboard on [app.chaingpt.org](https://app.chaingpt.org/) and use the _Create Secret Key_ feature). Copy the generated key – you'll use it to initialize the SDK.
* **Credits:** Ensure your ChainGPT account has sufficient credits. Each API call consumes credits (see **Rate Limits & Pricing** for details).

Initialize the SDK with your API key:

```
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'YOUR_CHAINGPT_API_KEY',
});
```

This API key will be used to authenticate all requests made through the `generalchat` instance.

{% hint style="info" %}
**Important:** Treat your API key as a secret. Avoid hard-coding it in public code or repositories. Instead, store it in an environment variable or secure storage and pass it to the SDK (for example, `apiKey: process.env.CHAINGPT_API_KEY`). This prevents unauthorized use of your key.
{% endhint %}

***

### Configuration Options

The ChainGPT Web3 AI Chatbot SDK offers several configuration options to tailor the chatbot’s behavior and context. These options can be provided when making a chat request (and some are set during SDK initialization):

* **apiKey:** _(string, required)_ Your ChainGPT API key (set when creating a `GeneralChat` instance).
* **question:** _(string, required)_ The user prompt or question for the chatbot (provided in each request via `createChatStream` or `createChatBlob`).
* **chatHistory:** _(string, optional)_ `"on"` or `"off"`. Enables or disables saving chat history for the request. When `"on"`, the conversation is recorded and can be retrieved later; when `"off"`, the conversation is not saved. Default is `"off"`.
* **useCustomContext:** _(boolean, optional)_ If `true`, the chatbot will include custom context information associated with your API key (as configured in the ChainGPT AI Hub) in its response. If `false` or omitted, the chatbot uses only its default knowledge base.
* **contextInjection:** _(object, optional)_ An object providing custom context data to override or supplement the default context from AI Hub. All fields in this object are optional; you can specify as much or as little context as needed. If `contextInjection` is provided (and `useCustomContext` is `true`), the SDK will use these values for this request instead of or in addition to the stored context on AI Hub. The available fields include:
  * `companyName` _(string)_ – Name of the company or project for the chatbot.
  * `companyDescription` _(string)_ – A brief description of the company/project.
  * `companyWebsiteUrl` _(string)_ – URL of the company’s website.
  * `whitePaperUrl` _(string)_ – URL of the project’s whitepaper or documentation.
  * `purpose` _(string)_ – The primary purpose or mission of the AI chatbot (or the company).
  * `cryptoToken` _(boolean)_ – Whether the company/project has an associated cryptocurrency token. If `true`, you can provide token details in `tokenInformation`.
  * `tokenInformation` _(object)_ – Details about the token (if `cryptoToken` is true):
    * `tokenName` _(string)_ – Name of the token.
    * `tokenSymbol` _(string)_ – Symbol or ticker of the token (e.g., `$CGPT`).
    * `tokenAddress` _(string)_ – Blockchain address of the token contract.
    * `tokenSourceCode` _(string)_ – Link or reference to the token’s source code.
    * `tokenAuditUrl` _(string)_ – Link to an audit report for the token.
    * `exploreUrl` _(string)_ – URL of a blockchain explorer for the token.
    * `cmcUrl` _(string)_ – CoinMarketCap URL for the token.
    * `coingeckoUrl` _(string)_ – CoinGecko URL for the token.
    *   `blockchain` _(array)_ – List of blockchain networks where the token is deployed. Use values from the `BLOCKCHAIN_NETWORK` enum. For example:

        ```
        blockchain: [
          BLOCKCHAIN_NETWORK.ETHEREUM,
          BLOCKCHAIN_NETWORK.POLYGON_ZKEVM
        ]
        ```

        **Supported blockchain enum values:** `ETHEREUM`, `BSC`, `ARBITRUM`, `BASE`, `BLAST`, `AVALANCHE`, `POLYGON`, `SCROLL`, `OPTIMISM`, `LINEA`, `ZKSYNC`, `POLYGON_ZKEVM`, `GNOSIS`, `FANTOM`, `MOONRIVER`, `MOONBEAM`, `BOBA`, `MODE`, `METIS`, `LISK`, `AURORA`, `SEI`, `IMMUTABLE_ZK`, `GRAVITY`, `TAIKO`, `CRONOS`, `FRAXTAL`, `ABSTRACT`, `CELO`, `WORLD_CHAIN`, `MANTLE`, `BERACHAIN`.
  *   `socialMediaUrls` _(array)_ – A list of social media links relevant to the company or project. Each entry is an object with `name` and `url` keys, for example:

      ```
      socialMediaUrls: [
        { name: "LinkedIn", url: "https://www.linkedin.com/company/example" },
        { name: "Twitter",  url: "https://twitter.com/example" }
      ]
      ```
  * `limitation` _(boolean)_ – Whether there are any limitations or restrictions for the chatbot. (If `true`, the bot might be instructed to acknowledge or enforce certain limitations in responses. Defaults to `false` if not set.)
  * **AI Tone Configuration:** The SDK allows you to set the tone or style of the AI’s responses:
    * `aiTone` _(enum)_ – Determines how the tone is specified. Possible values from the `AI_TONE` enum:
      * `AI_TONE.DEFAULT_TONE` – Use the default tone (no customTone or selectedTone needed).
      * `AI_TONE.CUSTOM_TONE` – Use a custom tone; if chosen, provide a `customTone` string.
      * `AI_TONE.PRE_SET_TONE` – Use one of the predefined tones; if chosen, provide a `selectedTone` value.
    * `selectedTone` _(enum value)_ – If `aiTone` is `PRE_SET_TONE`, specify which preset tone to use. Options include: `PROFESSIONAL`, `FRIENDLY`, `INFORMATIVE`, `FORMAL`, `CONVERSATIONAL`, `AUTHORITATIVE`, `PLAYFUL`, `INSPIRATIONAL`, `CONCISE`, `EMPATHETIC`, `ACADEMIC`, `NEUTRAL`, `SARCASTIC_MEME_STYLE`. (Use values from the `PRE_SET_TONES` enum.)
    * `customTone` _(string)_ – If `aiTone` is `CUSTOM_TONE`, provide a custom tone description in this string (e.g., `"Provide detailed security explanations."`).

Below is an example of a full `contextInjection` object utilizing many of these fields:

```
const contextInjection = {
  companyName: "Company name for AI chatbot",
  companyDescription: "Company description for AI chatbot",
  companyWebsiteUrl: "https://www.example.com",
  whitePaperUrl: "https://www.example.com/whitepaper.pdf",
  purpose: "Purpose of AI chatbot for this company.",
  cryptoToken: true,
  tokenInformation: {
    tokenName: "ExampleToken",
    tokenSymbol: "$EXM",
    tokenAddress: "0x1234...abcd",
    tokenSourceCode: "Source code reference or repository",
    tokenAuditUrl: "https://www.example.com/audit",
    exploreUrl: "https://explorer.example.com/token/0x1234...abcd",
    cmcUrl: "https://coinmarketcap.com/exampletoken",
    coingeckoUrl: "https://coingecko.com/exampletoken",
    blockchain: [ 
      BLOCKCHAIN_NETWORK.ETHEREUM, 
      BLOCKCHAIN_NETWORK.POLYGON_ZKEVM 
    ]
  },
  socialMediaUrls: [
    { name: "LinkedIn", url: "https://linkedin.com/company/example" }
  ],
  limitation: false,
  aiTone: AI_TONE.PRE_SET_TONE,
  selectedTone: PRE_SET_TONES.FRIENDLY,
  customTone: "Only for custom AI Tone"
};
```

You can pass this `contextInjection` object in a chat request to override or provide additional context. If a field is provided here, it will take precedence over the default context associated with your API key on the AI Hub for that request.

**Using Custom Context:** To use the custom context features, set `useCustomContext: true` in your request options. If you do not supply a `contextInjection` object when `useCustomContext` is true, the SDK will automatically fetch the default context (company details, etc.) associated with your API key from the AI Hub. If you do provide a `contextInjection` object, the SDK will use your provided values (overriding the defaults for those fields) for that request.

***

### Usage Patterns

Once the SDK is installed and configured, you can use it to send questions to the ChainGPT chatbot and receive answers. The SDK supports two modes of getting responses:

* **Streaming responses** – where the answer is received in a stream of data chunks (useful for showing the AI's answer as it is generated, similar to a typing indicator).
* **Single (blob) responses** – where the full answer is returned at once after processing.
* **Chat history management** – optional feature to record and retrieve conversation history.

Below are examples of each usage pattern.

#### Streaming Responses

Use the `createChatStream` method to get a streaming response. This returns a Node.js readable stream. You can listen for `data` events to receive chunks of the answer and an `end` event when the answer is complete.

```
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'YOUR_CHAINGPT_API_KEY',
});

async function main() {
  const stream = await generalchat.createChatStream({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off" // Set to "on" to save chat history; otherwise keep it "off"
  });
  stream.on('data', (chunk) => console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();
```

In this example, the question is sent and the answer will be printed to the console in real-time as the data arrives. The `chatHistory` parameter is set to `"off"`, meaning this conversation is not saved. (Use `"on"` if you want to enable history saving for this request.)

#### Single Response (Blob)

If you prefer to get the entire response at once (instead of streaming), use the `createChatBlob` method. This returns a promise that resolves to the full response data.

```
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'YOUR_CHAINGPT_API_KEY',
});

async function main() {
  const response = await generalchat.createChatBlob({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off"
  });
  console.log(response.data.bot);
}

main();
```

Here, `response` will contain the chatbot’s answer once the promise resolves. The answer text is accessible as `response.data.bot`. This example also disables history saving. You can enable it by setting `chatHistory: "on"` similarly, and providing a unique ID as described below.

#### Chat History

The SDK can store chat history so you can retrieve past conversations. To use this feature, you must turn on history and provide a unique identifier for the conversation:

* Set `chatHistory: "on"` in the request options to enable saving.
* Provide an `sdkUniqueId` parameter (a unique identifier such as a UUID or user ID) to tag the conversation. This ensures the history is associated with a specific user or session.

**Saving history in a chat request:**

```
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'YOUR_CHAINGPT_API_KEY',
});

const uniqueUuid = "907208eb-0929-42c3-a372-c21934fbf44f"; // an example unique identifier

async function main() {
  const stream = await generalchat.createChatStream({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "on",            // enable history saving
    sdkUniqueId: uniqueUuid       // associate this request with a unique ID
  });
  stream.on('data', (chunk) => console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();
```

In the above example, the conversation (question and answer) will be stored on ChainGPT's servers under the provided `sdkUniqueId`. Using the same `sdkUniqueId` for subsequent requests allows the chatbot to continue the conversation context or simply to group all queries from the same user.

**Retrieving chat history:**

You can retrieve saved chat history using the `getChatHistory` method. By default, this will retrieve history entries associated with your API key. You can optionally filter by the `sdkUniqueId` if you want history for a specific user or session.

For example, to retrieve the 10 most recent history entries:

```
const response = await generalchat.getChatHistory({
  limit: 10,
  offset: 0,
  sortBy: "createdAt",
  sortOrder: "ASC"
});
console.log(response.data.rows);
```

This will fetch up to 10 chat history records (in ascending order by creation time). Each record in `response.data.rows` contains details of a past question and answer.

To retrieve history for a specific user/session, include the same `sdkUniqueId` used when saving:

```
const response = await generalchat.getChatHistory({
  sdkUniqueId: uniqueUuid,  // filter by a specific conversation ID
  limit: 10,
  offset: 0,
  sortBy: "createdAt",
  sortOrder: "ASC"
});
```

The above will return history entries only for the conversation identified by that `sdkUniqueId`.

***

### Error Handling

The SDK provides a custom error class to handle API errors gracefully. If the library fails to connect to the API, or if the API returns an error response (e.g., an HTTP 4xx or 5xx status), a `GeneralChatError` will be thrown.

You can catch this error and inspect its message as shown below:

```
import { GeneralChat, Errors } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({ apiKey: 'YOUR_CHAINGPT_API_KEY' });

async function main() {
  try {
    const stream = await generalchat.createChatStream({
      question: 'Explain quantum computing in simple terms',
      chatHistory: "off"
    });
    stream.on('data', (chunk) => console.log(chunk.toString()));
    stream.on('end', () => console.log("Stream ended"));
  } catch (error) {
    if (error instanceof Errors.GeneralChatError) {
      console.error("Chat SDK Error:", error.message);
    } else {
      console.error("Unexpected Error:", error);
    }
  }
}

main();
```

In this snippet, if an error occurs during the API call, the catch block checks if it’s a `GeneralChatError` (accessible via `Errors.GeneralChatError` from the package). You can then handle it accordingly (for example, logging the error message or alerting the user). Non-`GeneralChatError` exceptions are rethrown or handled separately as needed.

***

### Rate Limits & Pricing

Using the ChainGPT Web3 AI Chatbot API via the SDK is subject to credit costs and rate limiting:

* **Credit Deduction:** Each chat request (stream or blob) deducts 0.5\*\* credits\*\* from your ChainGPT account. If you enable the chat history feature (`chatHistory: "on"`), an **additional 0.5 credits** is deducted for that request (total 2 credits for that call). Ensure you have sufficient credits before making requests, or the API will refuse the call.
* **Rate Limits:** To prevent abuse, each API key is limited to **200 requests per minute**. If you exceed this rate (bursting more than 200 requests within a one-minute window), you may receive rate-limit errors or throttled responses. Design your application to respect this limit (for example, by queuing or delaying requests if necessary).

There are no additional fees beyond credit usage. Credits can be obtained or topped up through the ChainGPT platform (see the ChainGPT app for purchasing or earning credits). Always monitor your credit balance, especially if you have an application making a high volume of requests.

***

### Language Support & Environment

The ChainGPT General Chat SDK is written for **JavaScript/TypeScript** and is intended for use in a **Node.js** environment. Key points about compatibility:

* **Runtime:** Node.js (LTS versions are supported). The SDK is not intended to run in a browser environment.
* **Language:** The library is usable in both JavaScript and TypeScript projects. It comes with TypeScript type declarations, so you get type checking and IntelliSense if you use TypeScript or a code editor like VSCode.
* **Module Format:** Distributed as an NPM package. You can import it using ESM `import` syntax (as shown in examples). If using CommonJS, you may use `require('@chaingpt/generalchat')` as needed.

Ensure that your environment can make HTTPS requests (the SDK under the hood communicates with ChainGPT’s REST API) – Node’s built-in `https` module is used by default.

***

### Security Considerations

When integrating the ChainGPT SDK, keep the following security best practices in mind:

* **API Key Protection:** Never expose your ChainGPT API key in client-side code or public repositories. Treat it like a password. Use environment variables or a secure key management system to supply the key to your application at runtime.
* **Authentication Enforcement:** The SDK requires a valid API key with credits, which helps ensure only authorized users (with your key) can use your ChainGPT allowance. If an unauthorized person obtains your key, they could consume your credits, so guard it carefully.
* **Request Limits:** The built-in rate limiting (200 requests/minute) and credit system serve as basic security to mitigate abuse (both for the API service and for your account to not accidentally overspend credits). Design your usage within these limits to avoid triggering protective measures.
* **Data Privacy:** If you enable chat history, conversation data is stored on ChainGPT servers. Consider your application’s privacy requirements – avoid sending sensitive personal data in questions if that data should not be stored. All communication with ChainGPT servers is over HTTPS for encryption in transit.

By following these considerations, you can safely integrate the SDK into your application without compromising sensitive information.

***

### Further Reading & Support

For more information and resources, check out the following:

* **Official NPM Package:** See the [ChainGPT General Chat SDK on npm](https://www.npmjs.com/package/@chaingpt/generalchat) for package details and automatically generated documentation of classes and methods.
* **ChainGPT Documentation:** Visit the [ChainGPT Developer Docs](../introduction-to-chaingpts-developer-tools.md) for comprehensive guides, additional SDKs, and API references (including the [Web3 LLM API Reference](api-reference.md)).
* **Support Channels:** If you need help or have questions, reach out through the official ChainGPT support channels. You can find support via the ChainGPT website (e.g., community Discord, Telegram, or support email listed on [chaingpt.org](https://www.chaingpt.org/)). The ChainGPT team is available to assist with integration issues or further inquiries.
