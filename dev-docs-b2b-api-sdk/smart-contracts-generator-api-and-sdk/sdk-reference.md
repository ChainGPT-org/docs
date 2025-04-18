# SDK Reference

## ChainGPT Smart-Contracts Generator SDK Reference

### Introduction

The **ChainGPT Smart-Contracts Generator SDK** is a Node.js library that allows developers to programmatically generate Solidity smart contract code using natural language prompts. It provides convenient access to ChainGPT’s AI-powered Smart Contract Generator service via simple JavaScript/TypeScript methods, without needing to handle raw HTTP requests. Using this SDK, you can integrate automated smart contract creation into your applications, tools, or workflows with minimal setup.

**Key features:**

* **Ease of Use:** Initialize the SDK with your API key and call a method with a plain English prompt to receive a ready-to-use smart contract. No deep AI or blockchain expertise is required.
* **Two Generation Modes:** Choose between receiving the full contract output in one response (a “blob” result), or streaming the contract code progressively as it’s generated in real-time.
* **Chat History Support:** Optionally enable chat history to allow multi-turn interactions (iteratively refining contracts) and retrieve past generation sessions.
* **Secure & Scalable:** Authentication is handled via API keys, with support for up to 200 requests per minute per key. The service uses a credit-based system for pricing (typically 1 credit per generation request, see **Credit Usage** below), ensuring predictable costs.

**Prerequisites:** To use the SDK, you need a Node.js environment (JavaScript or TypeScript) and an active ChainGPT account with API access. Make sure you have: Node.js installed, a ChainGPT API Key, and available credits in your ChainGPT account.

***

### Installation

Install the SDK package via npm or Yarn:

```
npm install --save @chaingpt/smartcontractgenerator
# or
yarn add @chaingpt/smartcontractgenerator
```

This will add the ChainGPT Smart-Contract Generator SDK to your project. The library is distributed with TypeScript type declarations, so it can be used in both JavaScript and TypeScript projects. The SDK supports **Node.js** environments (it is not intended for direct use in browsers).

***

### Setup and Authentication

Before using the SDK, you must obtain a **ChainGPT API Key** and ensure your account has sufficient credits:

* **Obtain API Key:** Log in to the ChainGPT web app ([**app.chaingpt.org**](https://app.chaingpt.org/)) and navigate to the API Dashboard. Use the interface to create a new _Secret Key_ (API key) and copy it. Each key is a secret token that authenticates your requests to the ChainGPT API.
* **Credits:** ChainGPT uses a credit-based system. Ensure your account has credits available (you can purchase or earn credits on the ChainGPT platform). Each API call via the SDK will deduct credits from your account (see **Credit Usage** below).

**Authentication:** Once you have your API key, provide it when initializing the SDK. For security, **do not hard-code** the key in your codebase. Instead, store it in an environment variable or secure config and pass it in at runtime. For example:

```
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

// Load API key from environment for security
const apiKey = process.env.CHAINGPT_API_KEY as string; 

// Initialize the SmartContractGenerator SDK client
const smartcontractgenerator = new SmartContractGenerator({
  apiKey: apiKey,  // your ChainGPT API Key
});
```

_Never expose your API Key publicly._ Treat it like a password: commit it to a secure store or environment variable rather than hardcoding. The SDK will use this key to authenticate all requests.

***

### Usage Overview

The ChainGPT Smart-Contracts Generator SDK provides three primary methods to interact with the service:

* **`createSmartContractBlob`** – Generate a smart contract and get the entire result in one response (a “blob” of data).
* **`createSmartContractStream`** – Generate a smart contract and receive the output as a stream of data chunks, allowing you to handle partial results in real-time (useful for progress updates or large contracts).
* **`getChatHistory`** – Retrieve the history of past smart contract generation requests and responses (if chat history was enabled for those requests).

All methods are available on the `smartcontractgenerator` instance created above. In the sections below, each method is detailed with its parameters, return format, example usage, and notes on credit consumption.

#### `createSmartContractBlob(options)`

Generates a new smart contract based on a natural-language prompt, returning the full contract code in a single response object. Use this method when you prefer to receive the complete result at once (for example, in a server-side script or when the final output size is manageable).

**Parameters:** (pass as an object to the method)

* **`question`** (`string`, required): The prompt or query describing the desired smart contract. This can be any natural-language instruction, e.g. _"Create an ERC-20 token contract with a mint function"_.
* **`chatHistory`** (`"on"` or `"off"` , required): Set to `"on"` to enable chat history for this request, or `"off"` to treat it as a one-off request. When `"on"`, the conversation history will be stored and used to inform the response (allowing follow-up questions). When `"off"`, the AI will not reference any prior context, and no history will be saved.
* **`sdkUniqueId`** (`string`, optional): A unique identifier for the user or session. Only needed if `chatHistory` is `"on"`. By providing a stable unique ID (such as a user ID or UUID) for each user/session, you ensure the chat history is kept separate per user. Use the **same** `sdkUniqueId` for subsequent calls to continue the conversation thread of a particular user (see **Chat History & Session Management** below). If not provided, the history will be tracked globally per API key (not separated by user).

**Returns:** A **Promise** that resolves to a response object containing the generated contract. On success, you can access the contract code string in `response.data.bot`. The response may include additional metadata (such as IDs or timestamps) as part of the data. On failure, the promise will reject with an error (see **Error Handling**).

**Example – Blob Response:**

```
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({ apiKey: "<YOUR_API_KEY>" });

async function main() {
  try {
    const response = await smartcontractgenerator.createSmartContractBlob({
      question: "Write a smart contract that counts. It will have two functions: one for incrementing, other for decrementing.",
      chatHistory: "off"
    });
    console.log("Generated contract code:\n", response.data.bot);
  } catch (error) {
    console.error("Error generating contract:", error);
  }
}

main();
```

In this example, `createSmartContractBlob` sends the prompt to ChainGPT and waits for the full contract code to be returned. We then log the `response.data.bot` field, which contains the Solidity code generated by the AI. Because `chatHistory` is set to `"off"`, this request will not be associated with any persistent history and only **1 credit** will be consumed for the API call.

**Credit Usage:** Each call to `createSmartContractBlob` deducts **1 credit** from your ChainGPT account. If you enable chat history for the request (`chatHistory: "on"`), an **additional 1 credit** will be deducted to store and maintain the conversation history. (See **Chat History & Session Management** for details on when to use this.)

#### `createSmartContractStream(options)`

Generates a new smart contract based on a prompt, returning the result as a streaming response. This method is useful for getting real-time feedback or updating a UI as the contract is being generated. The AI’s output will be sent in chunks through a Node.js stream interface.

**Parameters:** _Identical to `createSmartContractBlob`:_

* **`question`** (`string`, required): The natural-language description of the smart contract you want.
* **`chatHistory`** (`"on"` or `"off"`, required): Enable or disable using/storing chat history for this generation (same behavior as described above).
* **`sdkUniqueId`** (`string`, optional): Unique session/user ID (only needed if `chatHistory` is `"on"`, to isolate this conversation’s history).

**Returns:** A **Promise** that resolves to a Node.js **Readable stream**. The stream will emit data events with chunks of the contract code as they are generated. You can listen to the stream’s `'data'` event to process output incrementally, and a `'end'` event to know when generation is complete. If the request fails initially, the promise will reject with an error instead of returning a stream.

**Example – Streaming Response:**

```
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({ apiKey: "<YOUR_API_KEY>" });

async function main() {
  try {
    const stream = await smartcontractgenerator.createSmartContractStream({
      question: "Write a smart contract that counts. It will have two functions: one for incrementing, other for decrementing.",
      chatHistory: "off"
    });
    // Handle the stream events
    stream.on('data', (chunk: any) => {
      const partialOutput = chunk.toString();
      console.log("Received chunk:", partialOutput);
    });
    stream.on('end', () => {
      console.log("Stream ended - contract generation complete.");
    });
  } catch (error) {
    console.error("Error starting stream:", error);
  }
}

main();
```

In this example, `createSmartContractStream` returns a stream, and we set up listeners for `'data'` and `'end'` events. As the contract is generated, chunks of text (Solidity code) will be printed out in real-time. This is useful for providing progress feedback to users. We have used `chatHistory: "off"` here for a single-turn generation (consuming 1 credit). You could set it to `"on"` (with a corresponding `sdkUniqueId` if needed) to enable context continuity, at the cost of an extra credit for history.

**Credit Usage:** Each call to `createSmartContractStream` also costs **1 credit** (plus **1 additional credit** if `chatHistory: "on"` is used). Streaming a response or getting a blob both incur the same credit cost for the request; the difference is only in how the data is delivered.

#### `getChatHistory(options)`

Retrieves the history of past smart-contract generation interactions (prompts and responses) associated with your API key, especially those where chat history was enabled. This allows you to review or display previous contracts generated, or to fetch the context for follow-up questions in an ongoing session.

**Parameters:** (pass as an object)

* **`limit`** (`number`, optional): The maximum number of history entries to retrieve. For example, `10` to get the 10 most recent entries. (Defaults may apply if not specified, e.g. 10 or 20 by default.)
* **`offset`** (`number`, optional): The number of entries to skip (useful for pagination). For instance, an offset of `0` starts from the most recent entry, `10` would skip the first ten entries, etc.
* **`sortBy`** (`string`, optional): Field by which to sort the history entries. The default (and typical) field is `"createdAt"` (the timestamp of creation).
* **`sortOrder`** (`string`, optional): Sort direction, either `"ASC"` for ascending or `"DESC"` for descending. For example, to get newest first, use `"DESC"` (which is likely the default if not specified).

**Returns:** A **Promise** that resolves to a response object containing the history records. On success, the history entries are typically found in `response.data.rows` (as an array). Each entry may include information such as the prompt/question, the generated contract (or a reference to it), timestamps, and possibly the `sdkUniqueId` if one was used. If no history is available (e.g., if you never enabled `chatHistory` in prior calls), the returned list may be empty. If the request fails (e.g., due to invalid parameters or network issues), the promise will reject with an error.

**Example – Retrieving History:**

```
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({ apiKey: "<YOUR_API_KEY>" });

async function main() {
  try {
    const historyResponse = await smartcontractgenerator.getChatHistory({
      limit: 10,
      offset: 0,
      sortBy: "createdAt",
      sortOrder: "DESC"
    });
    const historyEntries = historyResponse.data.rows;
    console.log("Retrieved history entries:", historyEntries);
  } catch (error) {
    console.error("Error fetching chat history:", error);
  }
}

main();
```

This example fetches up to 10 previous smart contract generations (most recent first). The result `historyEntries` would contain the stored history records. You can iterate over these to inspect past prompts and results. Typically, you would call `getChatHistory` to support features like showing a user their past generated contracts or to retrieve a conversation’s context for further processing.

**Credit Usage:** **No additional credits** are deducted when retrieving chat history. The history entries are stored as a result of prior calls that had `chatHistory` enabled, which is when the extra credit was charged. Therefore, calling `getChatHistory` itself does not consume credits beyond those already spent to record the history. (If none of your previous calls had `chatHistory` turned on, you will have no history to retrieve.)

***

### Chat History & Session Management

Enabling **chat history** allows the ChainGPT Smart-Contracts Generator to remember previous interactions, enabling context-aware conversations. This can be useful if you want to ask follow-up questions or iterative refinements to a contract. For example, you might first ask for a basic contract, then ask the AI to add a feature to the last contract. When `chatHistory` is `"on"`, the model can utilize previous prompts and responses for better results.

**Using `chatHistory`:** To turn on history for a generation request, set `chatHistory: "on"` in the options for `createSmartContractBlob` or `createSmartContractStream`. This will store the prompt and response from that request in the ChainGPT backend. Subsequent calls with history enabled can then reference prior context. If `chatHistory` is `"off"`, each request is handled independently with no memory of earlier prompts.

**Session isolation with `sdkUniqueId`:** When building applications with multiple users or multiple concurrent sessions, it’s important to keep each conversation’s history separate. The SDK provides the `sdkUniqueId` parameter for this purpose. You should assign a unique identifier (e.g., a user ID or a randomly generated UUID) for each user or session, and pass that same `sdkUniqueId` value in every request where `chatHistory` is `"on"` for that user. This ensures the chat history stored for one user/session is not mixed with another. In other words, `sdkUniqueId` defines a scope for the conversation history.

{% hint style="info" %}
**Note:** If you omit `sdkUniqueId` while using `chatHistory: "on"`, all history-enabled requests under your API key will be lumped into a single default history. This may not be desirable if multiple distinct conversations are happening. It’s recommended to always use a unique ID for each independent history you want to maintain.
{% endhint %}

**Example – Managing Chat History with a Unique ID:**

```
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({ apiKey: "<YOUR_API_KEY>" });

// Suppose you have a unique identifier for the user or session:
const userSessionId = "907208eb-0929-42c3-a372-c21934fbf44f";  // example UUID

async function main() {
  try {
    const response = await smartcontractgenerator.createSmartContractBlob({
      question: "Now add a reset function that sets the count back to zero.", 
      chatHistory: "on",
      sdkUniqueId: userSessionId
    });
    console.log("Refined contract code:\n", response.data.bot);
  } catch (error) {
    console.error("Error generating contract with history:", error);
  }
}

main();
```

In this snippet, we continue a conversation by asking the AI to "add a reset function" to the contract. We use `chatHistory: "on"` and provide the same `sdkUniqueId` (e.g., the user’s UUID) that was used in previous requests for this user. The ChainGPT service will lookup the prior context associated with that `sdkUniqueId` and include it when generating the new response. The result is a refined contract that builds upon the earlier output. Each history-enabled call costs an extra credit (to store the interaction) as noted in **Credit Usage**.

You can retrieve the entire conversation later with `getChatHistory`, which will include all prompts and responses for that `sdkUniqueId` (and also global history entries if any). This allows for persistent session logs or auditing of AI-generated content.

***

### Error Handling

The SDK is designed to throw informative errors when something goes wrong – for example, network issues, invalid inputs, or API errors (like authentication failure or rate limit exceeded). All such errors are thrown as instances of the class `SmartContractGeneratorError`. You can import this error class via the `Errors` namespace provided by the SDK and use it to distinguish ChainGPT SDK errors from other exceptions.

**Example – Handling errors with try/catch:**

```
import { SmartContractGenerator, Errors } from "@chaingpt/smartcontractgenerator";

const smartcontractgenerator = new SmartContractGenerator({ apiKey: "<YOUR_API_KEY>" });

async function main() {
  try {
    // Attempt a streaming call (this example might fail if, say, the API key is invalid or network is down)
    const stream = await smartcontractgenerator.createSmartContractStream({
      question: "Write a smart contract that counts...",
      chatHistory: "on"
      // (sdkUniqueId omitted for brevity; in a real app include it if using history)
    });
    stream.on('data', chunk => console.log(chunk.toString()));
    stream.on('end', () => console.log("Stream ended"));
  } catch (error) {
    if (error instanceof Errors.SmartContractGeneratorError) {
      // Handle known SDK error (e.g., API returned an error or connection failed)
      console.error("ChainGPT SDK error:", error.message);
    } else {
      // Handle other unexpected errors
      console.error("Unexpected error:", error);
    }
  }
}

main();
```

In the above example, we wrap the call in a `try/catch` block. If an error occurs during the `createSmartContractStream` request (for instance, a 401 Unauthorized due to a bad API key, or a network timeout), the SDK will throw an `Errors.SmartContractGeneratorError`. We check for that specifically, and log the error message. If the error was not from the ChainGPT SDK (which is unlikely in this context, but good practice), it falls to the generic case. The `error.message` typically contains a human-readable description of what went wrong (for example, "Invalid API key" or "Rate limit exceeded"), which you can use for debugging or user feedback.

**Common error scenarios to handle:**

* _Authentication errors:_ If your API key is invalid or expired, the SDK will throw an error indicating authentication failed. Obtain a valid key and ensure it’s correctly configured.
* _Insufficient credits:_ If your account lacks credits, calls to the API will fail. The error message will inform you if you need to top up credits.
* _Rate limit exceeded:_ Exceeding the allowed request rate (200 requests per minute) will result in errors. You may need to throttle your requests if you hit this limit.
* _Network issues:_ Temporary network problems or server unavailability can cause request failures. These should be transient – you can catch and retry the request after a delay as needed.

By handling `SmartContractGeneratorError`, you can implement robust error recovery or user messaging in your application.

***

### Language and Environment Support

The Smart-Contracts Generator SDK is currently supported for **Node.js** environments and is written in TypeScript. This means you can use it in any JavaScript or TypeScript project running on Node.js (backend services, scripts, etc.). The package is distributed with type definitions, so you get IntelliSense and compile-time type checking if you use TypeScript.

* **Languages:** JavaScript (ES6+) and TypeScript are fully supported. Because it’s an npm package, you can also use it with bundlers or frameworks as long as it runs server-side.
* **Platform:** Node.js (LTS versions recommended). The SDK uses Node-compatible modules for HTTP requests. It is not tested for browser use, and using it in front-end web applications is not advised (both for security of the API key and CORS reasons).
* **Compatibility:** No specific framework is required – you can integrate it in a Node environment of your choice (Express, Next.js server-side, scripts, etc.). Ensure that your Node version is relatively up-to-date to support modern JS features.

In summary, if you have Node.js and an internet connection, the SDK should work out-of-the-box on Windows, Linux, or macOS.

***

### Security Best Practices & Rate Limits

Using the ChainGPT SDK in a production application requires mindful handling of credentials and adherence to usage policies:

* **API Key Security:** As emphasized, keep your ChainGPT API key secret. Do not embed it in client-side code or public repositories. Use environment variables or a secrets management service to inject the key into your app at runtime. This prevents malicious actors from stealing your key and consuming your credits.
* **Least Privilege:** Only generate and use API keys when needed. You might rotate keys periodically or revoke any that are compromised. Monitor your credit usage in the ChainGPT dashboard for any unexpected activity.
* **Request Rate Limiting:** The ChainGPT API (and by extension the SDK) enforces a rate limit of **200 requests per minute** per API key. If you exceed this, further requests may be rejected until the rate window resets. Design your application to stay within this limit – e.g., by queuing or spacing out non-urgent requests, or by distributing load across multiple API keys if absolutely necessary (and allowed by ChainGPT’s terms).
* **Credit Usage:** Every generation request costs credits (usually 1 credit, plus 1 more if using history). Make sure your account has enough credits for the workload you expect. The SDK does not currently provide a built-in way to check your credit balance, so you might need to track usage or periodically check the ChainGPT web dashboard. If a request fails due to insufficient credits, you’ll need to top up and retry.
* **Data Privacy:** The prompts you send and the contracts generated may be sensitive. According to ChainGPT’s policies, your data is processed to generate results but you should still avoid sending highly sensitive information in prompts. Also, store any returned contract code securely if it’s not meant to be public.

By following these practices, you can ensure that your integration with ChainGPT is secure and reliable. The SDK’s design (using API keys and credit limits) helps protect the service and users; adhering to these guidelines protects your own application from misuse or unexpected costs.

***

### Additional Resources and Support

For more information, code examples, and the latest updates, you can refer to the following resources:

* **NPM Package Documentation:** The [ChainGPT SmartContractGenerator SDK on npm](https://www.npmjs.com/package/@chaingpt/smartcontractgenerator) contains a README with usage examples (similar to those above) and may be updated with new releases. It also includes links to the package’s source repository and change log.
* **ChainGPT Developer Docs:** Visit the ChainGPT documentation portal for guides and references on other tools and APIs in the ChainGPT ecosystem. (You are likely reading these docs on the portal already – ensure you check out the QuickStart guides and other relevant sections for broader context.)
* **Release Notes:** As new versions of the SDK are released, refer to the release notes or change log (often found in the repository or npm page) to learn about new features or breaking changes. (As of this writing, the Smart-Contracts Generator SDK is in its initial release, v0.x, and future enhancements are planned to expand its capabilities.)

**Support:** If you encounter issues, have questions, or need help with the ChainGPT SDK, please use the official support channels. The ChainGPT team and community are active and ready to assist:

* Join the **ChainGPT Discord** community to ask questions and get help from developers and support staff (invite link: **discord.gg/chaingpt**).
* Visit the **ChainGPT Support** page on the official website for FAQs or to contact the team directly.
* For account or billing issues (such as API key or credit problems), you may reach out through the ChainGPT web app’s support or contact email as provided on the website.

By leveraging these resources, you can get the most out of the ChainGPT Smart-Contracts Generator SDK and quickly resolve any issues that arise. Happy building with ChainGPT!
