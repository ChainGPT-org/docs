# QuickStart Guide

## QuickStart Guide

Ready to dive in? This quickstart guide will help you set up and run your first smart contract generation via the API or SDK. We’ll cover prerequisites, installation, and some basic usage examples (including streaming responses, blob responses, error handling, and using chat history).

***

### Prerequisites

Before using the Smart-Contracts Generator, make sure you have the following prerequisites in place:

* **ChainGPT Account & Credits:** You need an account on the ChainGPT platform with available credits. Each API/SDK call consumes credits (see “Credit Usage” below). You can obtain credits by logging into [**app.chaingpt.org**](https://app.chaingpt.org) and purchasing or earning credits.
* **API Key:** Generate an API key from the ChainGPT API Dashboard. This key authenticates your requests. (Instructions below on how to get this key.)
* **Development Environment:**
  * For using the **SDK**: Node.js (v14+ recommended) installed on your system, and a project where you can install NPM packages.
  * For using the **API**: Any environment capable of making HTTP requests (this could be a backend server, or you can test with tools like cURL or Postman).
* **Basic Familiarity** with JavaScript/TypeScript (for SDK usage) or HTTP requests. While the generator is designed to be simple to use, understanding how to call an API or use an NPM package will help.

***

### Setup and Installation

**1. Get Your API Key:**\
To use either the API or SDK, you first need a ChainGPT API key:

* Log in to [**app.chaingpt.org**](https://app.chaingpt.org) with your account.
* Navigate to the **API Dashboard** section.
* Use the “**Create Secret Key**” feature to generate a new API key.
* Copy the generated key and store it somewhere safe. **⚠️ Important:** Treat this key like a password. **Do not** commit it to source code or share it publicly. It should be kept in an environment variable or a secure vault in your application (e.g. `CHAINGPT_API_KEY`).

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

**2. Set Up Credits:**\
Ensure your account has credits. Each request will deduct credits from your balance (1 credit per request, plus an extra credit if chat history is enabled on that request, as detailed later). You can view and top-up your credit balance on the ChainGPT web app. Trying to use the API without credits will result in an error, so don’t skip this step.

**3. Using the SDK – Installation:**\
If you plan to use the JavaScript/TypeScript SDK, install the package via your package manager:

```
npm install --save @chaingpt/smartcontractgenerator
# or
yarn add @chaingpt/smartcontractgenerator
```

This will add the Smart-Contracts Generator SDK to your project. The SDK abstracts the raw HTTP calls and provides convenient methods to interact with the generator.

**4. Using the API – Endpoint Access:**\
If you prefer to call the REST API directly (without the SDK), there’s nothing to install. You will be making direct HTTP requests to the ChainGPT API endpoint. The base URL for the Smart-Contracts Generator API is:

```
https://api.chaingpt.org/chat/stream
```

All requests must include your API key in the Authorization header. For example, in cURL you would include `-H "Authorization: Bearer YOUR_API_KEY"`. We’ll demonstrate an example request shortly.

***

### Basic Usage Examples

Let’s walk through a couple of quick examples: one using the SDK in a Node.js environment, and one using direct API calls with a typical HTTP client (axios). These will illustrate generating a smart contract code via streaming and via a single response (“blob”), as well as how to handle errors and optional chat history.

#### Example 1: Generate a Contract with Streaming Response (SDK)

Using the SDK, you can receive the AI’s response in a streaming fashion. Streaming is useful because you can start reading partial output (useful for a live preview or progress indication) as the AI is generating the code, rather than waiting for the entire response.

Below is a simple Node.js script that uses the SDK to stream a smart contract generation. It asks the AI to create a basic counter contract with increment and decrement functions:

```
import { SmartContractGenerator, Errors } from "@chaingpt/smartcontractgenerator";

// Initialize the SDK with your API key
const smartcontractGenerator = new SmartContractGenerator({
  apiKey: process.env.CHAINGPT_API_KEY,  // load your API key from an env variable for safety
});

async function run() {
  try {
    const stream = await smartcontractGenerator.createSmartContractStream({
      question: "Write a Solidity smart contract that implements a simple counter with increment and decrement functions.",
      chatHistory: "off"  // no chat history for this one-off prompt
    });
    
    // Stream comes as a Node.js readable stream
    stream.on("data", (chunk) => {
      const partialOutput = chunk.toString();
      console.log(partialOutput);  // print the streaming chunks (parts of the contract code)
    });
    stream.on("end", () => {
      console.log("Stream ended: full contract received.");
    });
  } catch (error) {
    if (error instanceof Errors.SmartContractGeneratorError) {
      // This error type is thrown for API-related issues (e.g., invalid request, no credits, etc.)
      console.error("API Error:", error.message);
    } else {
      console.error("Unexpected Error:", error);
    }
  }
}

run();
```

**What this does:** We import the SDK, initialize it with the API key, and then call `createSmartContractStream()` with a prompt (`question`). We’ve set `chatHistory: "off"` for simplicity (this means we’re not using conversation memory; more on that later). The SDK returns a stream, which we listen to for `data` events to get chunks of the Solidity code as they are generated. When the stream ends, we know the full output has been received. We also wrap the call in a try/catch and specifically catch `SmartContractGeneratorError` – this SDK error class will be used if the HTTP request fails or returns an error status (for example, due to invalid API key or insufficient credits).

**Expected output:** As the code runs, you’ll see the Solidity code printed in pieces, followed by “Stream ended” at completion. For instance, the output might start like:

```
pragma solidity ^0.8.0;

contract Counter {
    int256 private count;

    constructor() {
        count = 0;
    }

    function increment() public {
        count += 1;
    }

    function decrement() public {
        ...
```

and so on, until the entire contract is output. You can assemble the pieces to get the full contract text.

#### Example 2: Generate a Contract with Single Response (API or SDK Blob)

If you prefer to get the complete response in one go (instead of handling a stream), you can do that as well. One approach is to use the SDK’s “blob” method which waits for the full response, or directly call the API and buffer the result. Here’s an example using the SDK’s blob method:

```
import { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractGenerator = new SmartContractGenerator({
  apiKey: process.env.CHAINGPT_API_KEY,
});

async function run() {
  // This time we'll get the full response at once (not streaming)
  const result = await smartcontractGenerator.createSmartContractBlob({
    question: "Write a Solidity smart contract for a token that has a name, symbol, and an owner-only mint function.",
    chatHistory: "off"
  });
  
  // The result contains the AI response
  const contractCode = result.data.bot;
  console.log("Generated Contract:\n", contractCode);
}

run();
```

In this snippet, `createSmartContractBlob` sends the prompt and waits for the AI to finish generating the contract. The returned `result` object includes a `data.bot` field which contains the Solidity code as a string. We then print the full contract code.

If you wanted to do this **without the SDK**, you can directly call the REST API. For example, using **axios** in Node.js:

```
import axios from "axios";

const API_URL = "https://api.chaingpt.org/chat/stream";
const API_KEY = process.env.CHAINGPT_API_KEY;

async function runDirectApiCall() {
  try {
    const response = await axios.post(API_URL, {
      model: "smart_contract_generator",
      question: "Write a Solidity smart contract for a token that has a name, symbol, and an owner-only mint function.",
      chatHistory: "off"
    }, {
      headers: { Authorization: `Bearer ${API_KEY}` }
      // (Note: by default axios will collect the response; no need for stream here)
    });
    console.log("Generated Contract:\n", response.data.data.bot);
  } catch (err) {
    console.error("API call error:", err.response ? err.response.data : err.message);
  }
}

runDirectApiCall();
```

Here we post to the `API_URL` with the necessary body parameters. We include `model: "smart_contract_generator"` as required by the API, along with our prompt (`question`). The response, if successful, should contain the result. We assume the response JSON has a structure where the generated code is in `response.data.data.bot` (as returned by the API). The exact structure is detailed in the API reference section below, but the SDK abstracts this for you.

***

### Using Chat History (Conversation Context)

One powerful feature of the Smart-Contracts Generator is the ability to maintain **chat history**, meaning the AI can remember previous prompts and responses in the session to provide continuity (much like how ChatGPT remembers earlier messages). By default, as we saw above, we used `chatHistory: "off"` for single-turn prompts. If you set `chatHistory: "on"`, the system will retain context of the conversation so far.

**How to enable chat history:** simply include `chatHistory: "on"` in the request (either SDK or API). For example:

```
// Continuing from previous SDK example:
const result = await smartcontractGenerator.createSmartContractBlob({
  question: "Now add a burn function to the token contract.",
  chatHistory: "on"
});
```

If this call is made after a previous question (in the same runtime and using the same API key or an identifier – see below), the AI will know you’re referring to the token contract from before and attempt to modify/extend that contract code with a burn function.

**Managing user sessions:** In scenarios where you have multiple users or sessions using the API, you’ll want to segregate chat histories. The API/SDK provides a parameter `sdkUniqueId` for this purpose. Think of `sdkUniqueId` as a session or user ID – it ensures that the chat history is kept separate for different users of your application. If you’re just one person using your API key, you might not need this. But if you run a service where many end-users are generating contracts under the hood with your API key, you should provide a unique ID for each user/session.

**For example:**

```
await smartcontractGenerator.createSmartContractBlob({
  question: "Write a basic ERC-20 token contract.",
  chatHistory: "on",
  sdkUniqueId: "user123-session1"
});
```

This will store the history under the ID “user123-session1”. Subsequent calls with the same `sdkUniqueId` will have access to that conversation history. Calls with a different `sdkUniqueId` (or none provided) won’t mix up the contexts.

**Retrieving past chat history:** You can also fetch the conversation history via the SDK’s `getChatHistory` method or the API endpoint for history. This returns a list of past questions and answers. For instance:

```
const history = await smartcontractGenerator.getChatHistory({ limit: 10, offset: 0 });
console.log(history.data.rows);
```

This will fetch up to 10 of the most recent Q\&A pairs you’ve had (if `chatHistory` was enabled for those requests). You can use this in your app to display previous prompts and responses to the user, for example. By default, history will be fetched for the current API key or the provided `sdkUniqueId` context.

{% hint style="info" %}
_**Note**: If you enabled chat history during generation, the credit for maintaining that history was already charged on generation. Retrieving chat history via the API/SDK does **not** consume additional credits beyond a normal request, as long as you’re retrieving existing history._
{% endhint %}

***

### Best Practices & Tips

* **Secure your API Key:** Always load your ChainGPT API key from a secure location (environment variable, secret manager). Never hard-code the key in client-side code or commit it to a repository. Treat it as you would a password. If you suspect your key is compromised, revoke it in the dashboard and generate a new one.
* **Handle Errors Gracefully:** As shown in the examples, be prepared to catch errors from the API/SDK. Common issues include running out of credits, hitting rate limits, or providing invalid parameters. The SDK throws a specific error type you can check, and the API will return meaningful HTTP status codes (documented in the API Reference). Log these errors or surface them to users appropriately (e.g., ask the user to check API key or credit balance).
* **Use Chat History Thoughtfully:** Enabling `chatHistory` can be very powerful for multi-step contract generation (iteratively refining a contract). However, remember it consumes an extra credit per request. If you don’t need context, keep it off to save credits. If you do use it, consider using `sdkUniqueId` to isolate sessions and avoid bleed-over between different users or separate tasks.
* **Optimize Prompt Clarity:** The quality of the generated contract depends on your prompt. Be specific about what you want (e.g., “a token with burn and mint functions and role-based access control for minting” will yield a more precise result than “a token contract”). You can always refine with follow-up prompts (especially with chat history on).
* **Respect Rate Limits:** The service allows up to 200 requests per minute per API key. That’s quite high, but if you plan to scale usage (for example, in a production app), implement client-side throttling or request queuing to avoid exceeding this. If the limit is exceeded, you’ll receive HTTP 429 errors.
* **Stay Updated:** ChainGPT may release updates or improvements to the Smart-Contracts Generator. Keep an eye on the official docs and announcements. If the model improves or new features are added (like support for other programming languages or chains), you might want to update your integration or SDK version.

With these basics covered, you’re ready to start integrating! Next, we provide a detailed reference for the API and SDK, so you can dig deeper into all available endpoints, methods, parameters, and behaviors.
