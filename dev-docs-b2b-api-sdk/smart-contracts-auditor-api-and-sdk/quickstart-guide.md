# QuickStart Guide

## ChainGPT Smart Contract Auditor QuickStart Guide

Welcome to the ChainGPT Smart Contract Auditor QuickStart. This guide will help you set up your account and perform your first smart contract audit using ChainGPT’s Auditor, via either the REST API or the official Node.js SDK. By the end, you’ll be able to send an audit request and receive results, whether through a simple HTTP call or within a Node.js application.

***

### Prerequisites

Before you begin, make sure you have the following in place:

* **ChainGPT Account & API Key:** Sign up for an account on the ChainGPT web app and navigate to the **API Dashboard** to create a new **Secret Key** (API key). Copy this key – you’ll need it for authentication. **Ensure your account has credits** available, as each API call consumes credits (more on this below).
* **Development Environment:**
  * For using the **REST API**: a tool to make HTTP requests (e.g. cURL, Postman, or any HTTP client).
  * For using the **Node.js SDK**: a Node.js environment (Node installed on your system) and a package manager like npm or yarn.
* **Smart Contract to Audit:** This can be the Solidity source code of a smart contract or any prompt describing the audit you want. In our examples, we’ll use a simple Solidity contract.

{% hint style="info" %}
**Important:** Keep your API key **secret**. Do not hard-code it in public code or repositories. Use environment variables or a secure storage mechanism to store the key in production.
{% endhint %}

{% hint style="info" %}
**Note:** ChainGPT uses a credit system for the Auditor. **1 credit** is deducted per audit request. If you enable the chat history feature (explained below), **an additional 1 credit** is deducted per request. Ensure you have sufficient credits to avoid interruptions.
{% endhint %}

***

### Option 1: Using the REST API

The ChainGPT Smart Contract Auditor can be accessed via a RESTful HTTP endpoint. This option is ideal if you want to integrate audits into any environment or language that can make HTTP requests.

#### Endpoint and Authentication

The base endpoint for audit requests is:

```
POST https://api.chaingpt.org/chat/stream
```

All requests must include your API key in the header:

```
Authorization: Bearer <YOUR_API_KEY>
```

This key authenticates you to the Auditor service. In the request **body**, you will send JSON with at least two fields:

* **`model`** – the model name, which **must** be `"smart_contract_auditor"`.
* **`question`** – your prompt or question for the auditor (e.g. the contract code and any specific instructions).

Optional fields:

* **`chatHistory`** – set to `"on"` or `"off"` to enable or disable saving of chat history. (By default, history is off if not provided.)
* **`sdkUniqueId`** – an optional unique identifier for your user or session, used for tracking conversation history per user. This is mainly useful in multi-user applications (more on this in Best Practices).

With these in mind, let's make our first audit request.

#### Auditing a Contract via cURL

One of the simplest ways to test the API is by using **cURL** from the command line. Replace `<YOUR_API_KEY>` with the API key you obtained, and run the following command:

```
curl -X POST "https://api.chaingpt.org/chat/stream" \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "smart_contract_auditor",
    "question": "Audit the following contract:\npragma solidity ^0.8.0;\ncontract Counter {\n    uint256 private count;\n    constructor() {\n        count = 0;\n    }\n    function increment() public {\n        count += 1;\n        emit CountChanged(count);\n    }\n}\n",
    "chatHistory": "off"
  }'
```

This request sends a Solidity contract (a simple counter) to the Auditor. The response will stream back the audit results. You should see the Auditor’s analysis of the contract’s code, including any potential issues or summaries, printed to your terminal.

* The `Authorization` header carries your secret key.
* The JSON body specifies the model and the question. We included the Solidity code in the question prompt by prefixing it with "Audit the following contract:".
* We set `chatHistory` to `"off"` for this basic request, meaning the conversation won’t be stored.

{% hint style="info" %}
**Tip:** You can replace the contract in the `question` field with your own Solidity code or even ask a question about the contract. For example, you might ask the Auditor, _"What vulnerabilities are in the following contract?"_ followed by the code. The Auditor will respond accordingly.
{% endhint %}

#### Streaming Results with Axios (Node.js)

The Auditor endpoint delivers results as a **stream**, meaning the response is sent back in chunks (this is similar to how OpenAI’s streaming responses work). To handle streaming responses programmatically, you can use Node.js with an HTTP client like **Axios**.

In the example below, we use Axios to call the Auditor API and print the streamed chunks as they arrive:

```
const axios = require('axios');

const API_URL = "https://api.chaingpt.org/chat/stream";
const API_KEY = "<YOUR_API_KEY>";

async function main() {
  try {
    const response = await axios.post(
      API_URL,
      {
        model: "smart_contract_auditor",
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
      },
      {
        headers: { Authorization: `Bearer ${API_KEY}` },
        responseType: 'stream'  // ensure the response is returned as a stream
      }
    );

    // Print streamed chunks to console as they arrive
    response.data.on('data', (chunk) => {
      process.stdout.write(chunk);
    });
    response.data.on('end', () => {
      console.log("\n[Audit stream ended]");
    });
  } catch (error) {
    console.error("API request error:", error);
  }
}

main();
```

In this code:

* We post to the same `API_URL` with the required `model` and `question` fields. We’ve set `chatHistory: "on"` to demonstrate enabling the history (this will store the conversation on the server for later retrieval).
* **Axios** is configured with `responseType: 'stream'`, so `response.data` will be a Node.js readable stream. We attach listeners to output each data chunk immediately and log when the stream ends.
* Be sure to replace `API_KEY` with your actual key (or use `process.env.CHAINGPT_API_KEY` and load your environment variables as a best practice).

When you run this script, you should see the audit result gradually printed to your console. This streaming mode is useful for potentially long analyses, allowing you to start reading results without waiting for the entire response.

#### Fetching Chat History via REST API

If you have enabled chat history for your requests, the Auditor service stores recent conversation history. You can retrieve past audits using the **Chat History** endpoint. This is helpful for reviewing past audit results or continuing an interactive session.

The chat history endpoint is:

```
GET https://api.chaingpt.org/chat/chatHistory
```

It supports query parameters to paginate or sort the history:

* `limit` – number of records to fetch (e.g. 10).
* `offset` – starting index for pagination (e.g. 0 to start from the latest).
* `sortBy` – field to sort by (use `"createdAt"` for chronological sorting).
* `sortOrder` – `"asc"` or `"desc"` for ascending/descending order.

For example, to fetch the 10 most recent audit sessions in descending order, you can use cURL:

```
curl "https://api.chaingpt.org/chat/chatHistory?limit=10&offset=0&sortBy=createdAt&sortOrder=desc" \
  -H "Authorization: Bearer <YOUR_API_KEY>"
```

This will return a JSON response containing your chat history. Each entry typically includes information like the question (prompt) you asked and the Auditor’s response, along with timestamps. For instance, the response data structure has a list (e.g. in a field called `rows`) of history records.

{% hint style="info" %}
**Tip:** Only requests made with `chatHistory: "on"` are recorded in history. If you call the history endpoint but had never enabled history in your audit requests, the returned list will be empty or contain no entries. Always turn on chat history for the requests you want to log.
{% endhint %}

{% hint style="info" %}
**Note:** Retrieving chat history does **not** consume additional credits. You pay the credit cost when creating an audit (with history enabled), but querying the saved history is free.
{% endhint %}

***

### Option 2: Using the Node.js SDK

ChainGPT provides an official Node.js SDK for the Smart Contract Auditor, which wraps the API and handles streaming for you. This is the recommended approach if you are building a Node.js application, as it provides a more convenient interface and built-in error handling.

#### Installation

First, install the SDK package via npm or yarn:

```
npm install --save @chaingpt/smartcontractauditor
# or
yarn add @chaingpt/smartcontractauditor
```

This will add the Smart Contract Auditor SDK to your project. (Ensure you have Node.js and npm/yarn set up beforehand.)

#### Initializing the SDK

To use the SDK, import the `SmartContractAuditor` class and initialize it with your API key:

```
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({
  apiKey: "<YOUR_API_KEY>"
});
```

This creates a client instance (`smartcontractauditor`) that is authorized with your API key. (As before, consider loading the API key from an environment variable in practice, instead of hardcoding it.)

#### Auditing a Contract (Stream Mode)

Using the SDK, you can request an audit and receive the result as a stream. The SDK provides an method `auditSmartContractStream()` for this. Here’s an example:

```
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({
  apiKey: "<YOUR_API_KEY>"
});

async function main() {
  // Request an audit stream
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
  
  // Listen for streamed data
  stream.on('data', chunk => {
    console.log(chunk.toString());
  });
  stream.on('end', () => {
    console.log("Stream ended");
  });
}

main();
```

In this code:

* We call `auditSmartContractStream()` with a prompt containing our Solidity contract and set `chatHistory: "on"`. This returns a readable stream (`stream`).
* We attach event listeners to output each chunk of data as it arrives, and log when the stream ends. The Auditor’s response will be streamed chunk by chunk, similar to the raw Axios example earlier, but the SDK abstracts the setup for you.
* The stream delivers the Auditor’s analysis progressively. This is useful for getting immediate feedback on the audit as it runs.

#### Auditing a Contract (Blob Mode)

If you prefer to get the full response in one go (instead of streaming), the SDK offers `auditSmartContractBlob()`. This method waits for the Auditor to finish analyzing and returns the complete result (often referred to as a "blob" of data). Here’s how to use it:

```
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({
  apiKey: "<YOUR_API_KEY>"
});

async function main() {
  // Request an audit and get the full response
  const response = await smartcontractauditor.auditSmartContractBlob({
    question: `Audit the following contract:
pragma solidity ^0.8.0;
contract Counter {
    uint256 private count;
    constructor() {
        count = 0;
    }
    function increment() public {
        count += 1;
        emit CountChanged(count);
    }
}`,
    chatHistory: "off"
  });
  
  // The complete audit result is now available
  console.log(response.data.bot);
}

main();
```

Key points:

* We call `auditSmartContractBlob()` with the contract code. Here we set `chatHistory: "off"` to perform a one-off audit without saving the session.
* The method returns a `response` object. You can access the Auditor’s answer from `response.data.bot`. (The exact structure: the SDK returns an object where `data.bot` contains the auditor’s response text.)
* This approach is simpler if you just want the final output and do not need to stream it live. However, note that you will only see the result after the Auditor has finished processing the contract, which might take some time for complex contracts.

#### Fetching Chat History via SDK

Similar to the REST API, the SDK can retrieve stored chat history using the `getChatHistory()` method. This allows you to fetch past audit sessions that were saved with chat history enabled:

```
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({
  apiKey: "<YOUR_API_KEY>"
});

async function main() {
  const history = await smartcontractauditor.getChatHistory({
    limit: 10,
    offset: 0,
    sortBy: "createdAt",
    sortOrder: "DESC"
  });
  
  console.log(history.data.rows);
}

main();
```

Here, we request the 10 most recent records (`limit: 10` starting at `offset: 0`) sorted by creation time in descending order (`sortOrder: "DESC"`). The returned object’s `data.rows` contains an array of history entries. Each entry represents a past audit (with details such as the question and answer).

You can adjust the parameters as needed:

* Use `sortOrder: "ASC"` for oldest-first ordering.
* Increase `limit` or change `offset` for pagination if you have many records.

Make sure you had `chatHistory: "on"` for the audit requests you want to appear here – otherwise those sessions won’t have been saved.

***

### Common Tasks & Best Practices

Now that you have the basics, let’s cover some common scenarios and best practices to ensure a smooth integration:

* **Secure Your API Key:** Never commit your API key to source control or expose it in client-side code. Treat it like a password. In development, you can use a `.env` file and load it (for example, using `process.env.CHAINGPT_API_KEY` in Node). In production, use environment variables or a secret manager.
* **Environment Configuration:** The API endpoints are fixed as documented. You do not need to change the base URL. If needed, you can store the base URL (`https://api.chaingpt.org/chat/stream`) in a config or environment variable as well, but it’s generally static. Ensure your server can reach this URL (outbound internet access).
* **Credits Management:** Keep an eye on your ChainGPT credits. Each audit call deducts 1 credit, plus 1 more if chat history is enabled. If an API call isn’t working, verify that you have enough credits left in your account. You can check your credit balance on the ChainGPT web app.
* **Chat History Usage:** Enable `chatHistory: "on"` if you plan to ask follow-up questions or retrieve the Q\&A later. When this is on, the service will log the interaction. Use the `getChatHistory` endpoint/SDK method to fetch it as shown above. If you only need one-off answers and want to save credits, keep it off. Remember, fetching the history doesn’t cost credits beyond what was already spent.
* **Using `sdkUniqueId` for Multi-User Apps:** If you are integrating ChainGPT into an application with multiple end-users (for example, auditing contracts for different users from a single backend), use the `sdkUniqueId` field to separate their chat history and sessions. You can generate a unique UUID or use a user ID from your database. Include this `sdkUniqueId` in each audit request’s body along with `chatHistory: "on"`. This ensures that when you call `getChatHistory`, you can retrieve the history specific to that user. (In the SDK, simply pass `sdkUniqueId` in the options object for `auditSmartContractStream` or `auditSmartContractBlob` as needed.)
* **Handling Rate Limits:** The ChainGPT API has a rate limit of **200 requests per minute**. If you send too many audit requests too quickly, you may receive HTTP 429 errors or have requests rejected. Design your integration to stay within this limit. For example, spread out automated batch audits or implement a queue if necessary. If you legitimately need a higher rate, consider reaching out to ChainGPT support.
* **Interpreting Responses:** The Auditor’s response (especially in blob mode) will usually contain a detailed analysis. In JSON form, it may look like: `{ "data": { "bot": "...analysis text..." } }`. If you use the SDK’s blob method, you directly get this object, and as shown, `response.data.bot` holds the text output. If using the streaming method, the text will be delivered in chunks; you’ll need to concatenate or process them in order. Make sure your code handles multi-part responses correctly (as demonstrated with the stream listeners).
*   **Error Handling:** Be prepared to handle errors gracefully. If you provide an invalid API key or your credits are exhausted, the API will return an error (e.g., HTTP 401 for unauthorized or a message indicating insufficient credits). In the Node SDK, such situations throw a `SmartContractAuditorError` (available via the `Errors` export) if the response status is 4xx/5xx. For example, you can catch errors and inspect `error.message` for details. Always use try/catch around SDK calls that hit the API:

    ```
    import { SmartContractAuditor, Errors } from "@chaingpt/smartcontractauditor";
    // ... initialize smartcontractauditor as shown earlier ...
    try {
      const stream = await smartcontractauditor.auditSmartContractStream({ /* ... */ });
      // use stream
    } catch (error) {
      if (error instanceof Errors.SmartContractAuditorError) {
        console.error("Auditor API error:", error.message);
      } else {
        console.error("Unexpected error:", error);
      }
    }
    ```
* **Time Considerations:** Auditing complex smart contracts might take some time. The examples above set a 60-second timeout for Axios requests. If you anticipate longer processing (large contracts or multiple files), you might increase this timeout or handle partial responses via streaming so the connection isn’t cut off prematurely.
* **Keep SDK Updated:** The Smart Contract Auditor SDK may receive updates with new features or improvements. Check the npm package page or ChainGPT documentation for release notes. Using the latest version ensures you have the latest capabilities and bug fixes.

***

### Troubleshooting

Even with the best setup, you might encounter some issues. Here are some common problems and how to address them:

* **No Response / Empty Output:** If you call the API and get no answer or an empty response, first check that you included all required fields (`model` and `question`). The `model` must be exactly `"smart_contract_auditor"` – if this is missing or typoed, the request won’t reach the Auditor model. Also ensure your `Authorization` header is present and correct.
* **Authentication Errors (401 Unauthorized):** This usually means your API key is missing or incorrect. Double-check that:
  * You included the `Authorization: Bearer ...` header with a valid key.
  * The key is active (if you suspect it might have been revoked, generate a new one in the ChainGPT dashboard).
  * You have enough credits – if your credits are zero, the service may refuse requests. Top up credits via the web app if needed.
* **Insufficient Credits or Payment Required (402/403):** If the error message indicates lack of credits, you’ll need to purchase or allocate more credits to your account on the ChainGPT web app before retrying.
* **Rate Limit Exceeded (429 Too Many Requests):** If you send more than 200 requests/minute, the API will start rejecting calls. Implement a backoff or delay and try again later. If you are loop-testing, add a small delay between iterations to stay under the limit.
* **Malformed Request (400 Bad Request):** This can happen if your JSON body is not formatted correctly. For instance, a common mistake is forgetting to stringify JSON in a cURL call or sending an integer where a string is expected. Ensure the `question` and other fields are properly quoted strings in JSON. If using an HTTP client library, pass a JSON object (as shown in our examples) and set the `Content-Type: application/json` header.
* **Stream Not Emitting Data (in Node):** If you use the streaming endpoint with a custom HTTP library and nothing happens, make sure you are handling the response as a stream. In Axios (browser environment) or other environments, streaming might not be supported the same way as in Node. The given Axios example works in Node.js. If implementing in another language, use an HTTP client that can handle server-sent events or chunked responses. For quick tests, cURL with the `-N` option (unbuffered mode) can help display streaming text.
* **SDK Exceptions:** When using the SDK, any exception of type `SmartContractAuditorError` will include a message describing the issue (for example, "Request failed with status code 401"). Logging or printing this message can help diagnose if it’s an auth issue, a missing parameter, etc. The SDK also ensures network errors (like inability to reach the API) are caught as exceptions. Ensure your `apiKey` is correct in the SDK initialization. If you see a syntax-related error, verify that your Node version supports the syntax used (the SDK is written in modern JavaScript/TypeScript).
* **Debugging Tips:** Use logging. For REST API calls, log the request payload and the response status/body for debugging. For the SDK, you can intercept or print the parameters you pass in. If an audit seems to hang, try the same input with chatHistory off (to rule out any history issues), or vice versa. You can also test the input in the ChainGPT web app’s Auditor interface (if available) to see if the issue might be with the service or your integration.

If you encounter an issue not covered above, consider reaching out via ChainGPT’s support channels or community forums. Often, checking the official documentation or GitBook for updates can also provide insights.

***

### Next Steps

You’ve now performed a basic smart contract audit using ChainGPT’s Smart Contract Auditor! 🎉 From here, you may want to explore more advanced capabilities and integrate deeper into your workflows:

* **Explore the Full API Reference:** Review the complete API documentation for all available endpoints, parameters, and examples. (See the [ChainGPT Smart Contract Auditor API Reference](https://docs.chaingpt.org/smart-contract-auditor-api-reference) for details.)
* **Dive into the SDK Documentation:** Learn about all methods, classes, and error types provided by the Node.js SDK. (Visit the [ChainGPT Smart Contract Auditor SDK Reference](https://docs.chaingpt.org/smart-contract-auditor-sdk-reference) for a comprehensive guide.)

By combining this quick start with the full reference materials, you’ll be well-equipped to build powerful applications that automatically audit smart contracts using ChainGPT. Happy coding and secure auditing!
