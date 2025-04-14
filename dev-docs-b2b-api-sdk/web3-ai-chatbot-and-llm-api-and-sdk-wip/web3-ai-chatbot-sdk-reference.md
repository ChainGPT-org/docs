# Web3 AI Chatbot: SDK Reference

## ChainGPT Web3 LLM SDK (General Chat) Reference

### Introduction

The ChainGPT Web3 LLM SDK (previously known as the “General Chatbot SDK”) is a TypeScript/JavaScript library for integrating ChainGPT’s AI chatbot into Node.js applications. It allows developers to leverage ChainGPT’s large language model (LLM) with Web3 domain knowledge, enabling features like real-time streaming responses, context injection (e.g. blockchain data, company info), and persistent chat history for multi-turn conversations. This SDK communicates with ChainGPT’s General Chat REST API under the hood and abstracts away the HTTP details for convenience.

#### Key features:

* **Easy Integration:** Import the SDK and call its methods to get AI chatbot responses in your Node.js app. (Other language SDKs are planned in the future.)
* **Streaming or JSON Responses:** Choose between stream mode for token-by-token responses or blob mode for the full response in one go.
* **Custom Context Injection:** Tailor the AI’s knowledge base by injecting custom context (company info, token data, etc.) or use preset AI tones and styles.
* **Chat History Management:** Optionally enable the model to remember past questions (contextual chat) on a per-user basis, with an API to retrieve chat transcripts.
* **Credit-Based API:** Usage is metered via credits on your ChainGPT account – each request deducts credits (details in Pricing below).

{% hint style="info" %}
Note: The SDK was previously referred to as “General Chat”. You may see the NPM package named @chaingpt/generalchat – it’s the same Web3 LLM SDK.
{% endhint %}

#### Prerequisites:

* Node.js installed (the SDK runs in a Node environment; both CommonJS and ES modules are supported).
* A ChainGPT account with API access: you’ll need an API key and sufficient credits (CGPTc) on the ChainGPT web app.
* Familiarity with TypeScript/JavaScript for integrating the SDK into your project.

### Installation

**Install the SDK from npm into your Node.js project:**

```
npm install --save @chaingpt/generalchat
# or, if you use Yarn:
yarn add @chaingpt/generalchat
```

This will add the ChainGPT General Chat SDK to your project. \
**Once installed, you can import the SDK in your code:**

```
import { GeneralChat } from '@chaingpt/generalchat';
```

The SDK is written in TypeScript and includes type declarations for easier development. Ensure that your runtime or bundler supports ES2017+ features (since the SDK may use modern JavaScript syntax).

### Authentication

ChainGPT’s API uses API keys for authentication. Before using the SDK, you must generate an API key from the ChainGPT web app and have sufficient credits in your account.

**Steps to obtain and use an API key:**

1. **Generate API Key:** Log in to the [ChainGPT Web App](https://app.chaingpt.org/) and navigate to the API Dashboard (or SDK/API section). Use the interface to create a new _Secret Key_. Copy the generated API key – it will be a long string. (This key gives your application access to the ChainGPT API.)
2. **Add Credits:** Ensure your account has credits (CGPTc). Credits can be obtained via the ChainGPT app (e.g. through purchasing or via membership plans). Each API call will deduct credits from your balance (see Pricing for details).
3. **Secure Storage:** Never hard-code the API key in your source code. Treat it like a password. Store it in an environment variable or a secure configuration, and load it at runtime. For example, you might set CHAINGPT\_API\_KEY in your environment and use process.env.CHAINGPT\_API\_KEY in your code.

**When initializing the SDK, provide your API key in the configuration:**

```
const generalChat = new GeneralChat({
  apiKey: process.env.CHAINGPT_API_KEY  // load from env or config
});
```

The SDK will handle including this key in API requests. If the API key is invalid or missing, requests will fail with an authentication error.

{% hint style="info" %}
Warning: Do not expose your API key in client-side code, public repositories, or logs. Anyone with your key could use up your credits. Rotate your key immediately if you suspect it’s been compromised.
{% endhint %}

### Parameters

Below is a reference of the SDK’s primary classes and methods, along with their parameters and usage. This includes the GeneralChat class configuration and the methods to send chat requests or fetch history, as well as the structure of the contextInjection object for custom context data.

#### GeneralChat Configuration

When you create a new GeneralChat instance, you pass a configuration object. Currently it only requires the API key:

<table><thead><tr><th width="101.1796875">Parameter</th><th width="72.578125">Type</th><th width="83.16796875">Required</th><th>Description</th></tr></thead><tbody><tr><td>apiKey</td><td>string</td><td>Yes</td><td>Your ChainGPT API key for authentication. This key ties the SDK calls to your ChainGPT account and credit balance.</td></tr></tbody></table>

Example:

```
const generalChat = new GeneralChat({ apiKey: 'YOUR_API_KEY_HERE' });
```

_(Future versions of the SDK may support additional config options, but for now apiKey is the essential one.)_

#### generalChat.createChatStream(params)

Generates a chat completion as a stream. This method returns a Node.js readable stream that yields chunks of the AI’s response as they are generated (ideal for real-time updates in chat UIs).

Parameters:

<table><thead><tr><th width="114.02734375">Parameter</th><th width="89.26171875">Type</th><th width="88.30078125">Required</th><th>Description</th></tr></thead><tbody><tr><td>question</td><td>string</td><td>Yes</td><td>The user’s prompt or question for the chatbot.</td></tr><tr><td>chatHistory</td><td>"on" | "off"</td><td>No (Default "off")</td><td>Toggle to enable chat history. "off" (default) means the request is stateless (bot will not consider past chats). "on" means the conversation history is recorded and used for context in future requests (requires a sdkUniqueId if you have multiple users – see History section).</td></tr><tr><td>sdkUniqueId</td><td>string</td><td>No</td><td>A unique identifier for the end-user or session (UUID or similar). Provide this if chatHistory is "on" and you want to isolate this user’s chat history. Using a unique ID ensures that when history is fetched or continued, it’s tied to the correct user session. (If not provided, all chats with chatHistory:"on" under your API key will share a single history.)</td></tr><tr><td>useCustomContext</td><td>boolean</td><td>No (Default false)</td><td>Whether to use a custom context for this query. If true, the AI will incorporate additional context information. This context can come from your profile (AI Hub) or from a provided contextInjection. See Context Injection below.</td></tr><tr><td>contextInjection</td><td>object (see table below)</td><td>No</td><td>An object supplying custom context data (company info, token details, etc.) to inject into the AI’s knowledge for this query. All fields are optional; you can send a partial context. If provided, this will override or augment any default context tied to your API key. (Requires useCustomContext: true to take effect.)</td></tr></tbody></table>

**Returns:** A Node.js Readable stream. You should listen to the stream’s 'data' event to get chunks (each chunk is a Buffer containing part of the response text) and the 'end' event to know when the answer is complete. If an error occurs, it will be emitted as an 'error' event or thrown as a promise rejection (catchable with try/catch on the createChatStream call).

#### generalChat.createChatBlob(params)

Generates a chat completion as a single response (blob). This method returns a promise that resolves to the full response object once the AI has finished generating it. Use this if you prefer to get the entire answer at once (for example, non-interactive scenarios).

**Parameters:** (identical to createChatStream)

<table><thead><tr><th width="116.92578125">Parameter</th><th width="73.015625">Type</th><th width="94.9375">Required</th><th>Description</th></tr></thead><tbody><tr><td>question</td><td>string</td><td>Yes</td><td>The user’s prompt/question to the chatbot.</td></tr><tr><td>chatHistory</td><td>"on" | "off"</td><td>No (Default "off")</td><td>"off" for a standalone query (no memory), "on" to record this query/response in the chat history for context in future queries.</td></tr><tr><td>sdkUniqueId</td><td>string</td><td>No</td><td>Unique session/user ID, required to isolate history if chatHistory: "on". (See notes above.)</td></tr><tr><td>useCustomContext</td><td>boolean</td><td>No (Default false)</td><td>Use custom context (from profile or injection) if true.</td></tr><tr><td>contextInjection</td><td>object</td><td>No</td><td>Custom context data to inject (only used if useCustomContext: true).</td></tr></tbody></table>

**Returns:** A promise that resolves to an Axios response object containing the result. The actual chatbot answer will typically be in response.data.bot (as a string). You can also inspect other fields (e.g., response.data may include metadata, or response.status for the HTTP status).

**Example:** After const response = await generalChat.createChatBlob({...}), you might do console.log(response.data.bot) to print the AI’s answer.

#### generalChat.getChatHistory(params)

Retrieves past chat interactions (history) from the ChainGPT service. This is useful if you have enabled chat history and want to fetch previous Q\&A pairs, either for displaying to the user or for analytics.

**Parameters:**

<table><thead><tr><th width="97.3671875">Parameter</th><th width="95.3359375">Type</th><th width="138.6171875">Required</th><th>Description</th></tr></thead><tbody><tr><td>limit</td><td>number</td><td>No (Default 10)</td><td>Maximum number of chat history records to retrieve. Used for pagination or to limit results.</td></tr><tr><td>offset</td><td>number</td><td>No (Default 0)</td><td>Number of records to skip (for pagination). For example, offset: 0 and limit: 10 retrieves the first 10 entries; offset: 10, limit: 10 would retrieve the next 10.</td></tr><tr><td>sortBy</td><td>string</td><td>No (Default "createdAt")</td><td>Field to sort by. Currently, "createdAt" (timestamp) is the typical field to sort history entries.</td></tr><tr><td>sortOrder</td><td>"ASC" | "DESC"</td><td>No (Default "DESC")</td><td>Sort order: ascending or descending. For example, to get newest entries first, use "DESC" (default).</td></tr><tr><td>sdkUniqueId</td><td>string</td><td>No</td><td>If provided, fetches history only for the given unique user/session ID. Use this to get a specific user’s chat history. If omitted, the history returned will be the combined history for the API key (which could mix multiple users if sdkUniqueId wasn’t used when saving).</td></tr></tbody></table>

**Returns:** A promise that resolves to an Axios response object. On success, response.data will contain the history records. Typically, it includes an array of records (often under response.data.rows). Each record may include the user question, the bot answer, and metadata like timestamps or IDs.

**Note:** No credits are charged for calling getChatHistory – retrieving history is free, provided the history was recorded earlier with chatHistory: "on" (the credit cost was applied at the time of saving the chat).

#### Context Injection Object

The contextInjection parameter allows you to inject custom domain-specific information into the chatbot’s context for a given query. This can greatly enhance the relevance of responses (e.g., the bot can answer questions about your company or token if that info is provided). All fields in contextInjection are optional; provide what you have, and the rest will fall back to defaults or be ignored.

**The object structure and fields are as follows:**

<table><thead><tr><th width="179.12890625">Field</th><th width="130.421875">Type</th><th>Description</th></tr></thead><tbody><tr><td>companyName</td><td>string</td><td>Name of your company or project for the AI to be aware of.</td></tr><tr><td>companyDescription</td><td>string</td><td>A brief description of the company/project. This can include what it does, its mission, etc.</td></tr><tr><td>companyWebsiteUrl</td><td>string</td><td>URL of the company’s website.</td></tr><tr><td>whitePaperUrl</td><td>string</td><td>URL of the project’s whitepaper or technical documentation, if any.</td></tr><tr><td>purpose</td><td>string</td><td>The intended purpose or role of the AI chatbot for your use case. (e.g. <em>“To assist users with DeFi platform questions”</em>.) This helps set the context or persona of the bot.</td></tr><tr><td>cryptoToken</td><td>boolean</td><td>Set to true if the context is about a cryptocurrency/token project and you want to provide token-specific info. This flag indicates that the tokenInformation field is being used.</td></tr><tr><td>tokenInformation</td><td>object</td><td>Details about a cryptocurrency token (use if cryptoToken is true). See Token Information below for subfields.</td></tr><tr><td>socialMediaUrls</td><td>Array&#x3C;object></td><td>List of social media links related to the project. Each item is an object with { name: string, url: string }. The name can be one of a predefined set of platforms (see below). This helps the AI mention or use social links if relevant.</td></tr><tr><td>limitation</td><td>boolean</td><td>If true, it may instruct the AI to limit its answers to only the provided context (avoiding outside knowledge). If false, the AI will use both the provided context and its general knowledge. Default is false (open-ended). Use this to constrain the bot strictly to your data if needed.</td></tr><tr><td>aiTone</td><td>string (enum)</td><td>Controls the tone/style of the AI’s response. Options: "DEFAULT_TONE", "CUSTOM_TONE", or "PRE_SET_TONE". (See AI Tone Options below.) This determines how the next two fields are interpreted.</td></tr><tr><td>selectedTone</td><td>string (enum)</td><td><em>Use only if</em> aiTone is "PRE_SET_TONE". Specifies which predefined tone to use. (See the list of Preset AI Tones below for valid values, e.g. "FRIENDLY", "FORMAL", etc.)</td></tr><tr><td>customTone</td><td>string</td><td><em>Use only if</em> aiTone is "CUSTOM_TONE". Provide a custom style prompt or instructions for the AI’s tone. For example: "Speak in a playful, informal manner using crypto slang."</td></tr></tbody></table>

**Token Information subfields:** If you include tokenInformation, the object can have:

* **tokenName (string):** Name of the token/project (e.g. "Bitcoin").
* **tokenSymbol (string):** Symbol or ticker (e.g. "BTC"), prefixed with $ if desired.
* **tokenAddress (string)**: Contract address of the token (useful if on a specific blockchain).
* **tokenSourceCode (string):** Source code link or snippet reference (if relevant, e.g., GitHub repo or Etherscan link to contract code).
* **tokenAuditUrl (string):** URL to an audit report for the token/project.
* **exploreUrl (string):** Blockchain explorer link for the token or project.
* **cmcUrl (string):** CoinMarketCap page URL for the token.
* **coingeckoUrl (string):** CoinGecko page URL for the token.
* **blockchain (Array\<string>):** List of blockchain networks where this token is deployed or relevant. Use the standardized names from the SDK’s BLOCKCHAIN\_NETWORK enum (see list below).

You do not have to fill every field; only provide what’s relevant. For example, you might only set companyName, companyDescription, and a couple of social links. Or if it’s a crypto project, you might set cryptoToken: true and fill out tokenInformation fields.

**Supported Blockchain Network values**

If providing the blockchain array in tokenInformation, use the supported network identifiers. The SDK defines many options (in the BLOCKCHAIN\_NETWORK enum). Some common values include:

* ETHEREUM
* BSC (Binance Smart Chain)
* ARBITRUM
* BASE
* BLAST
* AVALANCHE
* POLYGON
* SCROLL
* OPTIMISM
* LINEA
* ZKSYNC
* POLYGON\_ZKEVM
* GNOSIS
* FANTOM
* MOONRIVER
* MOONBEAM
* BOBA
* METIS
* LISK
* AURORA
* SEI
* IMMUTABLE\_ZK
* GRAVITY
* TAIKO
* CRONOS
* FRAXTAL
* ABSTRACT
* WORLD\_CHAIN
* MANTLE
* MODE
* CELO
* BERACHAIN

_(These strings are case-sensitive constants in the SDK. Ensure you use the exact values as above. You can import BLOCKCHAIN\_NETWORK from the SDK to get these constants in code.)_

**AI Tone Options**

**The aiTone field controls how the assistant’s response is styled. Choose one of:**

* **DEFAULT\_TONE** – Use the default ChainGPT tone (no special styling). If selected, you do not need to provide selectedTone or customTone.
* **CUSTOM\_TONE –** Use a completely custom tone. When selected, provide a customTone string describing how the AI should respond. (For example, you might instruct the AI to be extremely casual, or to role-play as a tech support agent, etc.)
* **PRE\_SET\_TONE** – Use one of the predefined tone presets. When selected, provide selectedTone with one of the preset values listed below. (Ignore customTone in this case.)

**Preset AI Tones:** If you choose `PRE_SET_TONE`, the `selectedTone` can be one of the following predefined styles:

* PROFESSIONAL
* FRIENDLY
* INFORMATIVE
* FORMAL
* CONVERSATIONAL
* AUTHORITATIVE
* PLAYFUL
* INSPIRATIONAL
* CONCISE
* EMPATHETIC
* ACADEMIC
* NEUTRAL
* SARCASTIC\_MEME\_STYLE

Pick the tone that best suits the response you want. For instance, "FRIENDLY" will make the bot respond in a warm, approachable manner, whereas "ACADEMIC" might produce a more detailed and reference-heavy answer. (These presets are internally defined by ChainGPT.)

**Social Media Options**

If providing socialMediaUrls, each entry should have a name that identifies the platform and the corresponding profile url. Supported names include: LinkedIn, Telegram, Instagram, Twitter, YouTube, Medium. **For example:**

```
socialMediaUrls: [
  { name: "Twitter", url: "https://twitter.com/yourproject" },
  { name: "LinkedIn", url: "https://www.linkedin.com/company/yourcompany" }
]
```

This information can be used by the AI to answer questions like “Where can I follow the project on social media?” or to incorporate into responses if relevant.

***

With the contextInjection mechanism, you can dynamically supply information. Any field you include will override the default context fetched from the AI Hub for your API key. Fields you leave out will use whatever default context is associated with your API key (if useCustomContext is true) or just be omitted.

{% hint style="info" %}
Tip: The simplest way to use context injection is to set useCustomContext: true and include only the fields you want to override. For example, you can just change the aiTone for a single response by sending contextInjection: { aiTone: "PRE\_SET\_TONE", selectedTone: "FORMAL" } – this will use all your default context (company info, etc.) but switch the tone to Formal for that answer.
{% endhint %}

### Request/Response Examples

This section provides concrete examples of how to use the SDK to send a chat prompt and receive a response. We demonstrate both streaming and non-streaming (blob) modes. For each mode, we show how to use the SDK in Node.js and how you could make an equivalent call using a direct HTTP request (via Axios or cURL). These examples assume you have set your API key appropriately.

#### Example 1: Simple Chat (Streaming Response)

Suppose we want to ask the chatbot: _“Explain quantum computing in simple terms.”_ We’ll do this with a streaming response so that we can start receiving the answer as it’s generated.

**Using the SDK (Node.js, streaming):**

```
import { GeneralChat } from '@chaingpt/generalchat';

const generalChat = new GeneralChat({
  apiKey: 'YOUR_CHAINGPT_API_KEY'
});

async function main() {
  const stream = await generalChat.createChatStream({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off"  // no chat history for this simple one-off question
  });
  
  // Output the response as it streams in:
  stream.on('data', (chunk: Buffer) => {
    process.stdout.write(chunk.toString());  // print each chunk immediately
  });
  stream.on('end', () => {
    console.log("\n[Stream ended]");
  });
}

main();
```

When you run this, the program will immediately start printing the answer in chunks. The 'data' event fires multiple times as the AI generates text. Finally, when the answer is complete, it logs "\[Stream ended]". This approach is great for giving users instant feedback (for example, typing out the answer in a chat interface as it comes).

**Using Axios directly (Node.js, streaming):**

If you were not using the SDK, you could use Axios to call the ChainGPT API’s streaming endpoint. The key differences are setting the responseType to 'stream' and handling the data events on the returned stream:

```
import axios from 'axios';

const apiKey = 'YOUR_CHAINGPT_API_KEY';
const prompt = 'Explain quantum computing in simple terms';

// The hypothetical streaming API endpoint URL (for demonstration purposes):
const url = 'https://api.chaingpt.org/v1/chatbot/stream'; 

async function callStreamingAPI() {
  const response = await axios.post(url, 
    { question: prompt, chatHistory: "off" }, 
    { 
      headers: { 'Authorization': `Bearer ${apiKey}`, 'Content-Type': 'application/json' },
      responseType: 'stream'
    }
  );
  
  const stream = response.data; // This is a readable stream
  stream.on('data', (chunk: Buffer) => process.stdout.write(chunk.toString()));
  stream.on('end', () => console.log("\n[Stream ended]"));
}

callStreamingAPI();
```

In the above, we post to an example URL (you would use the actual ChainGPT API endpoint for general chat streaming). We include the API key in an Authorization header (assuming the API expects it there) and send the JSON body with our question. The response’s data is treated as a stream, which we then read from in the same way as with the SDK.

**Using cURL (streaming):**

You can also test the streaming response via a cURL command in the terminal. Using the -N flag with cURL ensures it doesn’t buffer the output, so you see it as it arrives:

```
curl -N -X POST "https://api.chaingpt.org/v1/chatbot/stream" \
  -H "Authorization: Bearer YOUR_CHAINGPT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ 
        "question": "Explain quantum computing in simple terms", 
        "chatHistory": "off" 
      }'
```

This will print the response text chunk by chunk in your terminal. (Replace the URL with the actual endpoint and YOUR\_CHAINGPT\_API\_KEY with your key.) The -N option disables buffering in cURL, allowing you to see partial output.

#### Example 2: Simple Chat (Single JSON Response)

Now, let’s do the same query but get the full response at once (blob mode). This is useful when you don’t need streaming or want to simplify output handling.

**Using the SDK (Node.js, blob):**

```
import { GeneralChat } from '@chaingpt/generalchat';

const generalChat = new GeneralChat({ apiKey: 'YOUR_CHAINGPT_API_KEY' });

async function askQuestion() {
  try {
    const response = await generalChat.createChatBlob({
      question: 'Explain quantum computing in simple terms',
      chatHistory: "on"  // example: turn on history (will cost extra credit and save history)
    });
    // The response object includes the data returned by the API
    console.log("AI says:", response.data.bot);
  } catch (err) {
    console.error("Request failed:", err);
  }
}

askQuestion();
```

In this snippet, response.data.bot contains the answer string from the AI. We also set chatHistory: "on" just to demonstrate usage – this would save this Q\&A to history (assuming we might ask follow-ups). If we don’t need that, we could leave chatHistory off.

**Using Axios directly (Node.js, blob):**

```
import axios from 'axios';

const apiKey = 'YOUR_CHAINGPT_API_KEY';
const prompt = 'Explain quantum computing in simple terms';
const url = 'https://api.chaingpt.org/v1/chatbot';  // hypothetical endpoint for blob response

async function callBlobAPI() {
  try {
    const response = await axios.post(url, 
      { question: prompt, chatHistory: "on" }, 
      { headers: { 'Authorization': `Bearer ${apiKey}`, 'Content-Type': 'application/json' } }
    );
    // Assuming the response JSON looks like: { bot: "answer text", ... }
    console.log("AI says:", response.data.bot);
  } catch (error) {
    console.error("Request failed:", error.response ? error.response.data : error.message);
  }
}

callBlobAPI();
```

Here we post to a non-stream endpoint (for example purposes). The Axios response will contain the full JSON answer. We log response.data.bot which we expect to be the answer text. If the API returns a different structure, adjust accordingly (the SDK examples indicate bot is the field with the assistant’s reply).

**Using cURL (blob):**

```
curl -X POST "https://api.chaingpt.org/v1/chatbot" \
  -H "Authorization: Bearer YOUR_CHAINGPT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ 
        "question": "Explain quantum computing in simple terms", 
        "chatHistory": "on" 
      }'
```

**This will return a JSON response. For example, you might get back something like:**

```
{
  "bot": "Quantum computing is a type of computation that uses the principles of quantum mechanics. In simple terms, it's like using special properties of tiny particles to perform calculations much faster than normal computers for certain tasks...",
  "id": "abc123...",
  "createdAt": "2025-04-13T...Z"
}
```

(The exact format can vary; the important part is the "bot" field containing the answer text.) You can parse the JSON to extract the answer. In cURL, the raw JSON will be printed to your terminal.

***

In both examples above, the SDK abstracts a lot: you didn’t need to worry about the exact URL or headers – just pass parameters to createChatStream or createChatBlob. Using Axios or cURL, you have to include the key and endpoint manually. For production use, the SDK approach is less error-prone, but it’s good to understand what’s happening behind the scenes.

### Error Handling

When calling the ChainGPT SDK, various things can go wrong (network issues, invalid input, insufficient credits, etc.). The SDK is designed to throw exceptions of type GeneralChatError (accessible via Errors.GeneralChatError) for HTTP errors or other API issues.

**How to handle errors:**

Use a try/catch around SDK calls (especially the async createChatBlob or obtaining the stream) to catch errors. Then, check if the error is an instance of GeneralChatError to handle it specifically.

**Example:**

```
import { GeneralChat, Errors } from '@chaingpt/generalchat';

const generalChat = new GeneralChat({ apiKey: 'YOUR_CHAINGPT_API_KEY' });

async function ask() {
  try {
    const stream = await generalChat.createChatStream({
      question: "Explain quantum computing in simple terms",
      chatHistory: "off"
    });
    // If we got here, the request was successful and we have a stream.
    stream.on('data', chunk => console.log(chunk.toString()));
    stream.on('end', () => console.log("Stream ended"));
  } catch (error) {
    if (error instanceof Errors.GeneralChatError) {
      // Known error from ChainGPT API
      console.error("ChainGPT API error:", error.message);
    } else {
      // Some other unexpected error
      console.error("Unexpected error:", error);
    }
  }
}
```

In the above code, if the API returns a non-200 HTTP status (e.g., 401 Unauthorized, 429 Too Many Requests, 500 Internal Server Error, etc.), the SDK will throw a GeneralChatError. The message property of this error will contain a descriptive message. For example, you might get messages like "Unauthorized: Invalid API Key", "Insufficient credits", or "Rate limit exceeded" depending on the scenario.

**Common error scenarios to handle:**

* **Invalid API Key / Unauthorized (401):** Check that your apiKey is correct and active.
* **Insufficient Credits:** If your account doesn’t have enough credits for the request, the API might refuse the request. The error message will indicate this. The solution is to top-up credits (or handle it gracefully by informing the user).
* **Rate Limit Exceeded (429):** If you send too many requests too quickly (see Rate Limits), you may get a rate-limit error. The error message or code will reflect that you should slow down.
* **Bad Request (400):** If some parameter is missing or invalid, the API might return a 400. Double-check required fields or value formats.
* **Internal Server Error (500):** Rare, but it could happen if something is wrong on ChainGPT’s side. In this case, you might implement a retry mechanism (with backoff).

{% hint style="info" %}
Tip: Always wrap your createChatBlob or createChatStream calls in try/catch. For streaming, note that if the error occurs after the stream is returned (for example, connection drop mid-stream), it may emit an 'error' event on the stream. You should listen for 'error' on the stream if you need to handle mid-stream failures.
{% endhint %}

### Advanced Usage

Beyond basic Q\&A, the SDK supports more advanced use cases. This section covers some patterns and tips for getting the most out of the ChainGPT Web3 LLM SDK in complex scenarios.

#### Multi-User Sessions and Persistent Conversations

If you are building a service where multiple end-users are each having their own chat with the AI (for example, a support chatbot on a platform with many users), you’ll want to isolate each user’s conversation. The sdkUniqueId parameter is crucial here. By assigning each user (or each separate chat session) a unique ID, the SDK ensures chat history is stored and retrieved per user.

**How to use sdkUniqueId:**

* When calling createChatStream or createChatBlob, set chatHistory: "on" to enable memory, and provide a unique sdkUniqueId (e.g., a UUID or a user ID from your database). For example:

```
const stream = await generalChat.createChatStream({
  question: "How can I reset my password?",
  chatHistory: "on",
  sdkUniqueId: "user-12345"  // an identifier for the user or session
});

```

* For each subsequent question from the same user, use the same sdkUniqueId and chatHistory: "on". The AI will recall previous interactions for that ID and provide contextual answers (just like a continuous chat).
* If a different user asks a question, use a different sdkUniqueId. Their history will be separate.

This way, you can maintain parallel conversations with the AI, one per user, without overlap.

Support chatbot scenario: Imagine implementing a customer support chatbot. You would enable history so the bot remembers what the user has already asked. By using the user’s unique ID, the chatbot’s context persists between messages, giving a more coherent experience. If the same user returns later (and you use the same ID), you could even retrieve their past chat and continue where they left off.

To retrieve or display the conversation, use `getChatHistory({ sdkUniqueId: "user-12345" })`. This will fetch that user’s past Q\&A. You could use this to preload the conversation in a UI, or to analyze how interactions are going.

{% hint style="info" %}
Warning: If you enable chatHistory: "on" without providing sdkUniqueId in a multi-user environment, all those interactions will be merged into a single history under your API key. This can lead to very confusing behavior, as the AI might mix context from different users. Always set a unique ID for each user or session when using history in multi-user systems.
{% endhint %}

#### Injecting Dynamic Context Data

One powerful feature is the ability to inject dynamic, external data into the AI’s context for a query. This goes beyond static preset context from AI Hub – you can fetch data at runtime and feed it in.

For example, suppose you want the AI to answer questions about the current price of a token or today’s blockchain network status. **You could do something like:**

```
// Pseudo-code for dynamic context injection
const latestPrice = await fetchCurrentPrice("ABC");  // your function to get price
const latestNews = await fetchLatestNewsHeadline();  // maybe get a relevant news piece

const contextInjection = {
  companyName: "ABC Crypto",
  companyDescription: `ABC is a utility token. Current price is $${latestPrice}.`,
  cryptoToken: true,
  tokenInformation: {
    tokenName: "ABCToken",
    tokenSymbol: "$ABC",
    // ... maybe other token info
  },
  aiTone: "CUSTOM_TONE",
  customTone: "Respond as a knowledgeable financial advisor."
};

const response = await generalChat.createChatBlob({
  question: "Should I hold or sell my ABC tokens today?",
  chatHistory: "off",
  useCustomContext: true,
  contextInjection: contextInjection
});
console.log(response.data.bot);
```

In this pseudo-code, we dynamically built the companyDescription to include the latest price. We also fetched a news headline (not actually used in the snippet, but you could insert it into the context or the question). By doing this, the AI’s answer will incorporate the latest price info that normally it wouldn’t know (since LLMs have fixed training data).

You can similarly inject any information your application has: user profile details, database stats, real-time analytics, etc. This makes the AI’s responses highly customized and up-to-date.

Another use case: If your AI is answering questions about a specific blockchain or contract, you might call an API to get the latest block or a particular contract’s state, then put that into the contextInjection or even directly into the question as part of prompt. The contextInjection is useful for background info that should always be considered for that query.

Remember to set useCustomContext: true whenever you provide contextInjection. If you forget, the SDK might ignore the contextInjection and the answer will be generic.

#### Retry Logic and Rate Limit Handling

In production, you should anticipate occasional failures or rate limiting, especially if your app makes a lot of requests or depends on reliable responses.

Retry on transient failures: If you get an error like a network timeout or a 5xx server error, it might be temporary. Implement a retry mechanism with exponential backoff. For example, if a request fails, wait 1 second and try again, if it fails again, wait 2 seconds, then 4, etc., up to a max number of retries. This can be done manually or by using libraries like axios-retry if you use Axios.

**Handling rate limits (429 errors):** The ChainGPT API allows up to 200 requests per minute per account (see Rate Limits). If you exceed this, you’ll get a rate limit error. To handle this:

* Check for the error message or status code 429. In our earlier try/catch, error.message might say something like “Too Many Requests”.
* Implement a cool-off: you might wait for a few seconds before retrying. The API might include a Retry-After header (in seconds) in the response – if so, use that as the delay.
* Ideally, design your usage to stay within the limit (queue requests or batch them if possible). If you foresee higher load, consider contacting ChainGPT for higher rate limits or using multiple accounts/keys (if permitted).

**Example retry snippet:**

```
async function safeAsk(params, retries = 3) {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      return await generalChat.createChatBlob(params);
    } catch (e: any) {
      if (e instanceof Errors.GeneralChatError) {
        // If rate limited, wait and retry
        if (e.message.includes("Too Many Requests") && attempt < retries) {
          const waitTime = attempt * 2; // e.g. 2s, then 4s...
          console.warn(`Rate limit hit, retrying in ${waitTime}s...`);
          await new Promise(res => setTimeout(res, waitTime * 1000));
          continue;
        }
      }
      // If not a rate limit error or out of retries, throw or handle
      throw e;
    }
  }
}
```

In the above logic, if we hit a rate limit, we wait (2s, then 4s, etc.) and try again. If some other error or after max retries, it breaks out.

**Graceful degradation:** If the AI service is down or unreachable, make sure your application can handle it. Perhaps show a fallback message like “Our chatbot is currently unavailable, please try again later.” The user experience shouldn’t dead-end.

#### Combining Stream and History for Live Chat

In an interactive chat web app, a common approach is to use streaming for the response (for immediacy) and also save the interaction to history:

* Call createChatStream with chatHistory: "on" and sdkUniqueId for the user.
* As the stream arrives, show it to the user in real-time.
* Once the stream ends, the entire conversation is automatically stored in history by the backend. If the user refreshes or comes back later, you can call getChatHistory to retrieve the past context and perhaps replay it or summarize it.

**One nuance:** if you plan to use getChatHistory to show the conversation, note that the newly asked question and answer will appear there only after the API call completes. With streaming, the API call technically “completes” at the end of the stream. So the history might not update until the answer is done. This is usually fine (you typically append the user question and then stream the answer live anyway).

#### Additional Tips

* **Context length:** Be mindful of how much data you inject via context. The AI has a context window (limited number of tokens it can pay attention to). If you stuff too much (e.g., a huge whitepaper in companyDescription), the model might get overwhelmed or the API might truncate it. Summarize or stick to key points when injecting custom info.
* **Tone and personality:** Use the aiTone and presets to match your brand’s voice. For example, a DeFi app’s bot might use a friendly or conversational tone, whereas a formal enterprise setting might prefer a professional tone. This can significantly affect user perception.
* **Testing in isolation:** If you’re not sure whether an odd answer is due to your contextInjection or something else, try toggling useCustomContext off and on, or testing the API with minimal inputs. This can help debug if your injected context is being applied correctly.
* **Updates:** Keep an eye on ChainGPT’s announcements. As the SDK evolves (or new language SDKs come out), there may be new features or changes (for example, improved error messages, new tone presets, etc.). Since we avoided mentioning specific version numbers here, always refer to the latest changelog for up-to-date info.

### History

The ChainGPT SDK’s history feature allows the AI to remember previous interactions, enabling a true conversational experience where follow-up questions can reference earlier answers. Here we delve deeper into how to use chat history and retrieve it.

**Enabling Chat History (chatHistory: "on"):**

By default, each request to the API is independent. If you ask two questions separately, the second answer won’t automatically use the first question for context. However, if you set chatHistory: "on" in your request, ChainGPT will link that question/answer into a conversation thread.

* The first time you send chatHistory: "on" with a new sdkUniqueId (or no id), the system will start a new conversation thread.
* The next time you send a request with the same sdkUniqueId and chatHistory: "on", the AI will be given the previous Q\&A from that thread as part of its context. This way, it “remembers” what was said.
* The history grows with each turn. You might cap the number of turns for practicality, but that’s managed by the API’s internal logic.

**For example:**

```
// User 42 asks the first question
await generalChat.createChatBlob({
  question: "Hello, who are you?",
  chatHistory: "on",
  sdkUniqueId: "user-42"
});
// AI might respond: "Hi! I'm an AI assistant ..."

// User 42 asks a follow-up question
await generalChat.createChatBlob({
  question: "Can you help me with investment advice?",
  chatHistory: "on",
  sdkUniqueId: "user-42"
});
// The AI will know the user already greeted and the context of conversation...
```

In the second call, because we used the same ID and kept history on, the AI might respond in a way that builds on the greeting, maintaining a conversational tone or recalling what was discussed.

**Retrieving Chat History (getChatHistory):**

All history data is stored on ChainGPT’s side (so that the AI can use it). You can fetch this data using generalChat.getChatHistory(). **This can be useful for:**

* Displaying the conversation to the user (e.g., showing previous messages when they open a chat window).
* Moderation or review: seeing what questions users asked and what answers were given.
* Debugging: understanding what context the AI had if you get an odd answer.

**Examples:**

* Get the 10 most recent history entries (any user) on your account:

```
const res = await generalChat.getChatHistory({ limit: 10, offset: 0, sortBy: "createdAt", sortOrder: "DESC" });
console.log(res.data.rows);
```

* This might return an array of objects like:

```
[
  { "id": "...", "question": "Can you help me with investment advice?", "answer": "...", "sdkUniqueId": "user-42", "createdAt": "2025-04-13T...Z" },
  { "id": "...", "question": "Hello, who are you?", "answer": "...", "sdkUniqueId": "user-42", "createdAt": "2025-04-13T...Z" },
  ...
]
```

* Each entry can include the unique ID (if one was provided) and timestamps. In this example, we see two entries for user-42’s conversation.
* Get history for a specific user (e.g., user-42):

```
const res = await generalChat.getChatHistory({
  sdkUniqueId: "user-42",
  limit: 5,
  sortBy: "createdAt",
  sortOrder: "ASC"
});
console.log(res.data.rows);
```

* This will list that user’s conversation from the beginning (oldest first, because we used ASC). So you might see the “Hello, who are you?” first, then the next question, etc.

**History data considerations:**

* **Persistence**: The history is stored on the server. It’s not stored client-side unless you save it. So even if your app restarts, you can fetch past conversations as long as you have the sdkUniqueId.
* **Security & Privacy:** If your users are inputting personal data, remember that this info is stored in ChainGPT’s servers as part of the history. Ensure this aligns with your privacy policy. There is currently no explicit “delete history via API” endpoint – you might manage sensitive info by using context injection for one-time data instead of actual user input if needed.
* **Cost:** Enabling history costs extra credits (see Pricing). Each API call with chatHistory: "on" deducts an additional credit (or portion of credit) for the storage/processing. Retrieval (getChatHistory) does not cost credits.
* **Capacity:** The API likely has limits on how much history it will keep or for how long (not documented here). It might also limit how much of the history is sent to the AI model for context (to avoid exceeding token limits). Typically, recent interactions carry more weight.

In summary, use history to create more natural, ongoing conversations. It’s especially useful in chatbots, virtual assistants, or any multi-turn dialogue with the AI.

### Pricing

ChainGPT operates on a credit-based pricing model (credits are denoted as CGPTc in the ChainGPT ecosystem). This means you pre-purchase or earn credits, and each API call consumes a certain number of credits.

**For the Web3 Chatbot SDK (General Chat), the pricing is as follows:**

* **Base cost per request:** _0.5 credits_ per chat request (whether stream or blob). Every time you call createChatStream or createChatBlob, it will deduct 0.5 CGPTc from your balance.
* **Chat history cost:** _+0.5 credits_ (additional) if chatHistory is enabled for that request. This is a surcharge for the added context and storage. So a request with history on will total 1.0 credit (0.5 base + 0.5 history).
  * If history is off, you only pay the base cost.
* **Context injection cost:** Currently, using useCustomContext or providing contextInjection does not incur an extra fee beyond the normal request cost. It’s treated as part of the same request.
* **Getting history:** Calling getChatHistory does not consume credits. (It’s essentially a database lookup on your conversation logs.)

**To illustrate**: if you make 10 requests in a session with no history (all stateless questions), you’ll consume 10 \* 0.5 = 5 credits. If you make 10 requests with history on (assuming same user), that’s 10 \* 1.0 = 10 credits.

Make sure you have enough credits in your account to cover your expected usage. You can purchase credits on the ChainGPT platform, and in some cases subscription plans might offer monthly credit allocations.

{% hint style="info" %}
Note: Credits (CGPTc) might correspond to the ChainGPT utility token or a fixed value per credit. The exact purchase price or token conversion is outside the SDK scope, but effectively each credit represents a fixed amount of API usage.
{% endhint %}

**Credit Monitoring:** You can track your credit balance and usage on the ChainGPT Web App’s dashboard. The dashboard will show how many credits you have left and possibly a log of where they were spent (requests made). It’s good practice to set up alerts on your side if you’re running low (the API might return an error when out of credits). See Monitoring for more.

The credit system ensures you only pay for what you use. There are no separate charges per token or per length of response – it’s a flat rate per request (with the minor addition for history). This simplifies cost calculations.

### Rate Limits

To maintain service quality and prevent abuse, ChainGPT’s API enforces rate limits. For the General Chat API, the limit is:

* 200 requests per minute per API key (account).

This means you can send up to 200 chat requests (stream or blob) within a rolling 60-second window. If you exceed this, additional calls will be rejected with a 429 Too Many Requests error until the rate window resets.

**Important points regarding rate limiting:**

* The 200 requests/min is an aggregate limit. It doesn’t matter if they are streaming or non-streaming requests; both count equally.
* If you have multiple servers or threads using the same API key, their usage is summed together for rate limiting. (If you need to go beyond the limit, you might consider using multiple API keys tied to the same account, if allowed, or request a higher limit from ChainGPT support.)
* The limit resets continuously – it’s not a strict per-minute on the clock, but a sliding window. For example, if you sent 200 requests from 10:00:00 to 10:00:30, you’d have to wait until some of those fall out of the 1-minute window (10:01:00) before new ones are accepted.

**Best practices to handle rate limits:**

* Queue requests: If your application can spike above 200/min (which is about 3.33 requests per second on average), implement a queue or throttle to keep calls within the allowed rate. For instance, you can introduce a small delay between calls if nearing the limit.
* Check headers: The API might include headers like X-RateLimit-Limit and X-RateLimit-Remaining or Retry-After in responses. Check the API documentation. If available, you can use these to dynamically adjust your request rate.
* Backoff on 429: As discussed in Advanced Usage, if you do hit a 429 error, catch it and delay the next attempt. Do not immediately hammer retries as that will likely keep failing until the window clears.
* Optimize queries: Avoid sending multiple requests in parallel when one would do. For instance, instead of asking the AI four separate questions rapidly, consider if those could be combined into one prompt (if context allows). Fewer, more comprehensive requests can reduce total call count.
* If your use case truly requires a higher rate (e.g., a very high-traffic application), reach out to ChainGPT – they may offer higher rate tiers for enterprise or particular partnerships.

Remember that streaming responses occupy a connection for the duration of the stream. However, as far as rate count is concerned, a streaming request counts as one request when initiated. But if you had 200 long-running streams simultaneously, that might be another kind of limit (like connection limits) – typically though, 200 simultaneous streams is also likely too high. Consider the practical concurrency as well.

**In summary:** _200 requests/minute_ is the cap – design your application to stay under that, and implement safeguards to handle the case when it’s exceeded.

### Monitoring

Monitoring your integration is vital for both ensuring reliability and keeping track of usage (and cost). Here are some aspects of monitoring for the ChainGPT Web3 LLM SDK:

* **Credit Usage Monitoring:** The ChainGPT Web App provides an API Dashboard where you can see your credit balance and usage statistics. It’s a good idea to regularly check this or set up alerts if the platform allows. For example, if your credits drop below a threshold, you might want to receive an email or webhook (if ChainGPT supports that) so you can top up before service is interrupted.
  * On the dashboard, each API call may be logged (with timestamp, type, and credit cost). This transparency helps you audit that you’re being charged correctly and to identify any unexpected spikes in usage.
  * Make sure the team members managing the account have access to this dashboard and understand the credit consumption patterns of your application.
* **Application-side Logging:** Implement logging in your application for every request to the SDK. Log at least: timestamp, the sdkUniqueId (if any), question asked, and whether it succeeded or errored (with error message). You might not log the full answer for privacy reasons, but logging questions and errors is useful.
  * This lets you monitor how users are interacting with the bot and catch if something goes wrong. For instance, if suddenly a lot of errors appear in logs, or responses are slow, you can investigate (could be an outage or a new type of input causing issues).
  * If you have a monitoring/observability stack (like Grafana, Datadog, etc.), integrate these logs or counters. You could track metrics like “requests per minute”, “error rate”, “average response time”, etc.
* **Performance Monitoring:** Keep an eye on response times. The SDK call for blob will take some time to return (depending on question complexity and answer length). For streaming, measure the time from request to first byte, and to completion. If these times start increasing significantly, it might indicate the AI service is under heavy load or network issues.
  * You might implement a timeout on your side – e.g., if no response in, say, 30 seconds, you abort and tell the user the service is busy. The SDK itself doesn’t impose a strict timeout by default, but you can use one via wrapping the promise or using Axios configs if needed.
* **Content Monitoring:** Depending on your application, you might want to monitor the content of the AI’s responses. The SDK/ChainGPT presumably has some built-in filtering for inappropriate content, but if your use case is sensitive, you should review the outputs occasionally. You can log responses (with user consent / per privacy policy) or use a moderation model to scan them.
  * For example, if building a chatbot for a financial app, you might want to ensure the AI isn’t giving actual financial advice that could be problematic. Monitoring content can help flag if the AI goes off track.
* **Auditing Context Usage:** When using contextInjection heavily, monitor that it’s working as expected. One way is to occasionally ask the bot something that requires the context and ensure the answer uses it. If it doesn’t, there might be an issue in how context is provided. Essentially, test critical flows in a staging environment with monitoring.
* **Endpoint Health:** The ChainGPT API endpoints should be quite stable, but consider monitoring the endpoint health (e.g., a simple scheduled request that pings the API with a trivial prompt to ensure it’s up). This can alert you of downtime or latency issues proactively.
* **Updates and Changes:** Monitor ChainGPT’s announcements or documentation updates. If pricing changes, rate limits change, or new parameters are introduced, you want to know. Possibly subscribe to a newsletter or RSS of their docs if available. This isn’t “monitoring” in the technical sense, but it helps keep your integration up-to-date and prevents surprises (like a new parameter becoming required).

Finally, integrate monitoring with your DevOps. For instance, if error rate goes above X% or if available credits drop too low, trigger an alert (pager, email, etc.). Proactive monitoring ensures your ChainGPT-powered features remain reliable and you can address issues before they impact users significantly.

***

With comprehensive monitoring and the guidelines above, you’ll be well-equipped to maintain a robust integration with ChainGPT’s Web3 LLM SDK. **Happy coding!**
