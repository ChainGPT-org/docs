# QuickStart Guide

## ChainGPT Smart Contract Generator – QuickStart Guide

This quickstart will help you integrate ChainGPT’s **Smart Contract Generator** either via the direct **HTTP API** or using the official **Node.js SDK**. It covers setup steps, authentication, making your first request, handling streaming vs. complete responses, using chat history, and important best practices. Follow the guide below to get up and running quickly and confidently.

***

### API QuickStart

The **Smart Contract Generator API** allows you to access ChainGPT’s contract generation engine over HTTP, without installing any SDK. This is ideal for using the service from any backend environment or language that can make HTTP requests.

#### Prerequisites

Before calling the API, ensure you have the following prerequisites ready:

* **ChainGPT Account with Credits:** A ChainGPT account on the [ChainGPT WebApp](https://app.chaingpt.org/) with sufficient credits. _Each API call consumes credits, so make sure you have credit balance to avoid interruptions_.
* **API Key:** A secret API key associated with your account. You can generate one from the WebApp’s API dashboard. (See **Getting an API Key** below.)
* **Environment to make requests:** For example, a command-line tool like `curl` or an API client (Postman), or a programming environment (e.g. Node.js with Axios, Python with `requests`, etc.) to send HTTP requests.
* **Operating System:** Any OS with internet access (Windows, Linux, macOS, etc.).

#### Getting an API Key

To obtain your API key for authentication:

1. **Log in to ChainGPT WebApp:** Go to [**app.chaingpt.org**](https://app.chaingpt.org/) and sign in to your account (create an account if you haven’t already).
2. **Navigate to API Dashboard:** Find the **API Dashboard** or **API Keys** section in the web application.
3. **Create a Secret Key:** Click on “Create Secret Key” (or similar) to generate a new API key. Give it an identifiable name if prompted.
4. **Copy the API Key:** Once generated, copy the API key. **Store it securely** – you’ll need it for API calls.

{% hint style="info" %}
⚠️ **Important:** Treat your API key like a password. **Do not expose it** in client-side code, public repos, or anywhere insecure. It’s recommended to store the key in an environment variable or secure secrets manager, and load it in your application at runtime rather than hard-coding it in code.
{% endhint %}

Ensure your ChainGPT account has credits before proceeding. The API uses a credit system: **1 credit is deducted per API request**, plus an **additional 1 credit** if you enable chat history for that request. If you have no credits, calls to the API will fail. (You can purchase or earn credits in the WebApp as needed.)

#### Authentication and Endpoint

The Smart Contract Generator API uses simple **Bearer token** authentication with your API key. All requests should include an HTTP header `Authorization: Bearer YOUR_API_KEY`.

The base endpoint for generating smart contracts via chat is:

```
POST https://api.chaingpt.org/chat/stream
```

**Request Headers:**

* `Authorization: Bearer YOUR_API_KEY` (required) – Use the API key obtained above. (Always prefix it with the word “Bearer ” and a space).
* `Content-Type: application/json` – Ensure you send JSON if using raw HTTP (most HTTP libraries set this automatically when sending a JSON body).

**Request Body Parameters:** (JSON)

* `model` (string, **required**): The model name to use. For this API, use `"smart_contract_generator"`.
* `question` (string, **required**): The user’s prompt or question. For example, _“Write a Solidity smart contract for a simple counter.”_
* `chatHistory` (string, optional): `"on"` or `"off"`. Set to `"on"` to enable chat history (conversation memory) for this request; `"off"` for a single-turn query. _(See **Chat History** below for details.)_
* `sdkUniqueId` (string, optional): A unique user or session identifier, used when chat history is on. This ensures the chat history is tied to a specific user or session for continuity. If you’re just doing a one-off call, you can omit this.

**Example Request:** If you use `curl`, a basic example might look like (replace `<API_KEY>` with your key):

```bash
curl -X POST "https://api.chaingpt.org/chat/stream" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"model": "smart_contract_generator", "question": "Generate a smart contract for a sum of two numbers."}'
```

This will initiate a request to generate a smart contract based on your prompt.

#### Making Your First API Call

Let’s walk through making a request using Node.js for illustration. You can adapt the logic to your preferred language or tool.

First, set up an HTTP client. In Node.js, you might use **Axios** for convenience:

```javascript
const axios = require('axios');

// Load configuration from environment (for security, avoid hard‑coding)
const API_URL = 'https://api.chaingpt.org/chat/stream';
const API_KEY = process.env.CHAINGPT_API_KEY;   // set your key in an env var

// Create an Axios instance with the base URL, auth header, and streaming mode
const apiClient = axios.create({
  baseURL: API_URL,
  timeout: 60000,              // 60‑second timeout
  headers: { Authorization: `Bearer ${API_KEY}` },
  responseType: 'stream'       // ⭐️ this makes response.data a readable stream
});
```

With the client ready, you can **send a POST request** to the endpoint. Below, we send a prompt and handle the response as a stream:

```javascript
try {
  const response = await apiClient.post('/', {
    model: 'smart_contract_generator',
    question: 'Write a smart contract that counts. It will have two functions: one to increment, and another to decrement.'
  });
  
  // The response is streamed chunk by chunk
  response.data.on('data', (chunk) => {
    console.log(chunk.toString());  // print each piece of the contract as it arrives
  });
  response.data.on('end', () => {
    console.log("Stream ended");
  });
} catch (error) {
  console.error("API request failed:", error);
}
```

In this example, the API will start **streaming** the generated smart contract code back. We attach handlers to process each `data` chunk as it arrives, and to know when the stream has ended. This streaming approach is useful for showing partial results to users in real-time or handling very large outputs without loading everything in memory.

**Getting the complete response at once:** If you prefer to wait for the full result rather than streaming, you can adjust the request. For instance, remove the streaming response handling and let the request complete normally. In Axios, you could omit the `responseType: "stream"` configuration (or use the default). For example:

```javascript
// Using the same apiClient instance, but force a JSON (non‑stream) response
const fullResponse = await apiClient.post(
  '/',
  {
    model: 'smart_contract_generator',
    question: 'Write a smart contract that counts with increment and decrement functions.'
  },
  { responseType: 'json' }   // <- override the client default
);

console.log(fullResponse.data);
```

This will wait until the smart contract generation is finished and then log the entire response body at once. The response data typically includes the generated contract text (and any metadata) in one complete payload. This mode is handy if you just want the final result and don’t need to stream intermediate output.

{% hint style="info" %}
Note: Alternatively, create a separate Axios instance without the `responseType: 'stream'` configuration for blob‑style requests.
{% endhint %}

#### Chat History and Session Management

One powerful feature of the Smart Contract Generator is the ability to maintain context across multiple interactions. By enabling **chat history**, you allow the model to remember previous questions and answers, which can be useful for follow-up queries or iterative development.

* **Enabling Chat History:** To turn on conversation memory, set `"chatHistory": "on"` in your request body. When enabled, the system will log the conversation so that subsequent requests can include prior context.
* Using `sdkUniqueId` : When chat history is on, it’s recommended to provide an `sdkUniqueId` – a unique identifier for the user or session. This ensures that the history is tied to that ID. For example, you might use a user’s UUID or session ID from your application. That way, if multiple users or separate sessions use the API, their histories won’t mix together.

**Example (API request with chat history):**

```javascript
await apiClient.post('/', {
  model: 'smart_contract_generator',
  question: 'Now add a reset function to the counter contract.',
  chatHistory: 'on',
  sdkUniqueId: 'user-12345'  // unique identifier for the user or session
});
```

If this call is made after the previous prompt (and uses the same `sdkUniqueId`), the API will remember the earlier contract it generated and the new request will be answered in that context (e.g. adding to the same contract code).

When `chatHistory` is `"on"`, each API call will **store the Q\&A** internally. You can retrieve the saved conversations if needed:

*   **Retrieve Chat History (API):** There is an endpoint to fetch past chat entries: `GET https://api.chaingpt.org/chat/chatHistory`. It supports query parameters like `limit`, `offset`, `sortBy`, and `sortOrder` to page through history. For example, you could do:

    ```javascript
    // Assuming API_URL is set to https://api.chaingpt.org/chat/chatHistory for this request
    const historyClient = axios.create({
      baseURL: 'https://api.chaingpt.org/chat/chatHistory',
      headers: { Authorization: `Bearer ${API_KEY}` }
    });
    const historyResponse = await historyClient.get('/', {
      limit: 10,
      offset: 0,
      sortBy: 'createdAt',
      sortOrder: 'desc'
    });
    console.log(historyResponse.data.data.rows);
    ```

    This would fetch the 10 most recent chat history records (including prompts and responses).

**Note:** When using chat history, remember that **each request with `chatHistory: "on"` costs an extra credit** (since the service is storing and maintaining context). However, fetching the history via the `chatHistory` endpoint does **not** consume extra credits (you already paid when saving the history).

If you do not need conversation context or follow-ups, you can keep `chatHistory: "off"` (or omit the field). This will treat each request independently.

#### Credit Usage and Rate Limits

ChainGPT’s API is credit-based and has usage limits to ensure fair use:

* **Credit Consumption:** Every call to the smart contract generator API **deducts 1 credit** from your account. If you enable chat history (`"chatHistory": "on"`), an **additional 1 credit** is deducted for that request (so 2 credits total for that call).
* **Rate Limits:** To prevent abuse, there is a rate limit of **200 requests per minute** per account. If you exceed this, subsequent requests may be throttled or rejected. Design your application to stay within this limit (e.g., by queueing or spacing out calls if needed).
* **Error Handling for Limits:** If you hit the rate limit or run out of credits, the API will return an error (HTTP 4xx/5xx). Make sure to handle such responses in your code (see **Error Handling** below).

Keep an eye on your credit balance via the WebApp dashboard. Credits may be purchased or obtained through ChainGPT’s platform. **💡 Note:** The credit policy (cost per request) may be updated over time, so always refer to the latest information on the ChainGPT WebApp for any changes.

#### Error Handling and Best Practices

When integrating the API, follow these best practices to build a robust and secure application:

* **Use Environment Variables for Secrets:** As mentioned, never hard-code your API key in your codebase. Use environment variables (e.g., `process.env.CHAINGPT_API_KEY`) or a secret management system to inject the key at runtime. This prevents accidental exposure of the key in version control or client-side code.
*   **Wrap API Calls in Error Handling:** Network issues or API errors (like invalid parameters or insufficient credits) can occur. Always use try/catch (for async calls) or handle promise rejections when calling the API. For example, in Node/Axios:

    ```javascript
    try {
      const response = await apiClient.post(...);
      // process response
    } catch (err) {
      console.error("API call failed:", err.response ? err.response.data : err.message);
    }
    ```

    Check the error response for details (status code, message) to implement retries or user messaging as appropriate.
* **Security (Server-Side Calls):** Ideally, call the ChainGPT API from your backend server, not directly from client-side applications. This keeps your API key hidden from end-users. If you must call from a client (not recommended), consider proxying through your server or using other secure delegation methods.
* **Validate Inputs:** Ensure that the prompts (`question` field) sent to the API are validated or sanitized as needed by your application context, especially if they come from end-users, to avoid misuse or unintended content requests.
* **Monitor Usage:** Implement logging around your API calls. Log remaining credits (if the API returns that info via headers or response) or count calls to anticipate when you need to top-up credits. Also handle the scenario where the API key is revoked or becomes invalid (the API would return an unauthorized error).

By following these practices, you’ll maintain secure access and handle common issues gracefully.

***

### SDK QuickStart

If you are using a Node.js environment, the **ChainGPT Smart Contract Generator SDK** provides a convenient wrapper around the API. The SDK handles HTTP calls under the hood and provides a cleaner interface, plus built-in error types. This section will guide you through installing and using the SDK in a project.

#### Prerequisites

Before using the SDK, make sure you have:

* **Node.js runtime:** The SDK supports JavaScript/TypeScript in a Node.js environment. Ensure Node.js is installed on your system (SDK is not intended for use in browsers).
* **NPM or Yarn:** You’ll use a package manager to install the SDK package.
* **ChainGPT Account, API Key, and Credits:** As with the API, you need a ChainGPT account with an API key (secret key) and available credits. Generate a key via the WebApp’s API Dashboard if you haven’t already (refer back to **Getting an API Key** in the API section). Keep the API key ready for use in your code, and ensure your account has credits (each SDK call will consume credits just like the API).

#### Installation

Install the ChainGPT Smart Contract Generator SDK via npm or yarn:

```javascript
npm install --save @chaingpt/smartcontractgenerator
# or
yarn add @chaingpt/smartcontractgenerator
```

This will add the SDK library to your Node.js project. The SDK is published on NPM as **`@chaingpt/smartcontractgenerator`**.

#### Initialization and Configuration

After installing, import the SDK and initialize it with your API key. For example:

```javascript
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({
  apiKey: process.env.CHAINGPT_API_KEY  // Your ChainGPT API Key (use env var for security)
});
```

This creates a `smartcontractgenerator` client instance configured with your API key. The `apiKey` is required for authentication. _(As a best practice, load it from an environment variable rather than embedding the literal key.)_

Behind the scenes, the SDK uses this key to authenticate to the ChainGPT API on your behalf. Make sure the key is valid and has credits associated with it.

#### Making Your First SDK Request

With the SDK client ready, you can now generate smart contracts. The SDK provides two main methods to create a smart contract answer from a prompt:

* **`createSmartContractStream(options)`** – for streaming responses (chunk-by-chunk).
* **`createSmartContractBlob(options)`** – for a complete response (all at once, as a “blob” of data).

Both methods take an `options` object where you specify at least a `question` (the prompt). Other options include `chatHistory` and `sdkUniqueId` (just like the API parameters).

**Example 1: Quick one-off generation (Blob response)** – This will get the full response at once:

```javascript
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({ apiKey: process.env.CHAINGPT_API_KEY });

async function generateContract() {
  try {
    const result = await smartcontractgenerator.createSmartContractBlob({
      question: "Write a smart contract that counts. It will have two functions: increment and decrement.",
      chatHistory: "off"  // no chat history for this single request
    });
    // The result contains the AI's answer including the contract code
    console.log(result.data.bot);
  } catch (error) {
    console.error("Error generating contract:", error);
  }
}

generateContract();
```

In this snippet, we call `createSmartContractBlob` with a prompt. The response (`result`) contains a `data` object, and the generated contract text is in `result.data.bot`. We log it to verify the output. If there’s an error (e.g., no credits or network issue), it gets caught and logged.

**Example 2: Streaming response** – If you want to receive the response incrementally (useful for progress updates or interactive apps):

```javascript
async function generateContractStream() {
  try {
    const stream = await smartcontractgenerator.createSmartContractStream({
      question: "Write a smart contract that counts with increment and decrement functions.",
      chatHistory: "off"
    });
    stream.on('data', (chunk) => {
      console.log(chunk.toString());  // print each chunk of the contract as it streams in
    });
    stream.on('end', () => {
      console.log("Stream ended");
    });
  } catch (error) {
    console.error("Streaming request failed:", error);
  }
}

generateContractStream();
```

Here we use `createSmartContractStream`. It returns a Node.js stream. We attach listeners to handle the incoming `data` events (converting each chunk to string and printing it) and an `end` event to know when the stream is complete. This way, you can start processing or displaying parts of the contract before the entire output is ready.

Both methods (`createSmartContractBlob` and `...Stream`) ultimately achieve the same result – generating a contract from a prompt – but streaming can improve responsiveness for large outputs. Under the hood, the SDK takes care of calling the appropriate API endpoint and handling the HTTP details.

#### Using Chat History in the SDK

The SDK also supports multi-turn interactions via chat history, similar to the raw API:

* To enable conversation memory, pass `chatHistory: "on"` in the options.
* Provide a unique `sdkUniqueId` in the options to identify the user/session whose history should be used.

**Example:** Continuation with chat history.

```javascript
const userId = "907208eb-0929-42c3-a372-c21934fbf44f";  // unique ID for the user or session

async function askFollowUp() {
  try {
    const followUp = await smartcontractgenerator.createSmartContractBlob({
      question: "Now add a reset function to the counter contract.",
      chatHistory: "on",
      sdkUniqueId: userId
    });
    console.log(followUp.data.bot);
  } catch (error) {
    console.error("Error with follow-up question:", error);
  }
}

askFollowUp();
```

In this example, we use the same `userId` for `sdkUniqueId` as was used in a previous call (not shown) where `chatHistory` was on. The SDK (and underlying API) will retrieve the prior conversation for that ID and use it to inform the answer to the follow-up question. The result is that the new output will be consistent with or build upon the earlier contract.

You can also retrieve the stored chat history via the SDK:

```javascript
async function fetchHistory() {
  try {
    const history = await smartcontractgenerator.getChatHistory({
      limit: 10,
      offset: 0,
      sortBy: "createdAt",
      sortOrder: "DESC"
    });
    console.log(history.data.rows);
  } catch (error) {
    console.error("Failed to fetch history:", error);
  }
}

fetchHistory();
```

This uses the `getChatHistory` method to fetch recent chat sessions. The parameters `limit`, `offset`, `sortBy`, `sortOrder` work as in the API, and the response (`history.data.rows`) will contain an array of past interactions (prompts and generated answers) stored for your account.

**Credit usage:** The credit costs for using the SDK are the same as the direct API calls – each generation call uses credits (1 or 2 if history is on). Fetching chat history (`getChatHistory`) does not consume extra credits (assuming the history was recorded in earlier calls where you already paid for it).

#### Error Handling in SDK

The SDK is designed to throw errors if something goes wrong (e.g., network issues, invalid API key, not enough credits, etc.), so you should handle these exceptions. Errors from the SDK are instances of the class `SmartContractGeneratorError` (available under `SmartContractGenerator.Errors` if needed).

For example, you can catch errors as follows:

```javascript
import { Errors } from "@chaingpt/smartcontractgenerator";

try {
  const stream = await smartcontractgenerator.createSmartContractStream({ /*...*/ });
  // use stream...
} catch (error) {
  if (error instanceof Errors.SmartContractGeneratorError) {
    console.error("ChainGPT SDK Error:", error.message);
  } else {
    console.error("Unexpected Error:", error);
  }
}
```

In the above, we specifically check if the error is a `SmartContractGeneratorError` (which indicates an issue with the request or API response) and handle it accordingly. You might inspect `error.message` or other properties to decide next steps (for example, prompt the user to check credits or retry later). Always wrap your SDK calls in try/catch since they are asynchronous.

#### Security Considerations

Using the SDK securely involves similar practices to the API:

* **Keep API Keys Secret:** Do not commit your API key in code. Leverage environment variables or configuration files that are not exposed. When instantiating `SmartContractGenerator`, pass the key securely (as shown earlier).
* **Server-side Usage:** Use the SDK on the server side of your application. Avoid including your API key in any client-side (browser) code. The SDK is meant for Node.js, so it typically wouldn’t run in a browser context anyway.
* **Rate Limiting:** The SDK will not magically bypass rate limits – the same **200 requests/minute** limit applies. If you plan to send a high volume of requests, implement checks or throttling in your application logic to stay within limits.
* **Resource Management:** Streaming responses are Node streams – be mindful to consume them to completion or handle backpressure if needed. In the examples above, we simply log the chunks; in a real app, you might append them to a file or aggregate them. Make sure to handle the `end` event to know when a stream is fully consumed.
* **Input Sanitization:** Just as with direct API calls, ensure that any prompt coming from users is validated or sanitized if necessary.

By following these security guidelines, you help protect your API key and maintain the reliability of your integration.

***

#### Additional Resources

Congratulations on getting started with ChainGPT’s Smart Contract Generator! Here are a few additional resources and next steps to enhance your development:

* **ChainGPT WebApp Dashboard:** Use the [WebApp Dashboard to m](https://app.chaingpt.org/)onitor your API usage, manage credits, and create/revoke API keys. It’s a good practice to rotate keys if you suspect any compromise.
* **SDK Documentation and Source:** Check out the [ChainGPT SmartContractGenerator SDK on NPM for ](https://www.npmjs.com/package/@chaingpt/smartcontractgenerator)the README and documentation. It may contain more examples and details on available methods (e.g., TypeScript types, advanced usage).
* **Support and Community:** If you need help or have questions, visit the ChainGPT website’s support section or community channels. The official site [chaingpt.org prov](https://www.chaingpt.org/)ides links to support resources and contact information. Engaging with the developer community can be a great way to troubleshoot issues or learn best practices.
* **Future Updates:** ChainGPT is continually improving. Stay tuned for updates or new releases of the SDK (you can check version release notes on NPM), and new features in the API. Keeping your SDK version up-to-date will ensure you have the latest capabilities and fixes.

With the information in this guide, you should be able to integrate the Smart Contract Generator into your project. Happy coding, and we look forward to seeing what you build with ChainGPT!
