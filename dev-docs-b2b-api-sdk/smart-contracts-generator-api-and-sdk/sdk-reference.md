# SDK Reference

## SDK Reference

The ChainGPT Smart-Contracts Generator **SDK** is available as an NPM package (`@chaingpt/smartcontractgenerator`) and provides a convenient wrapper around the API for Node.js environments. In this section, we detail how to use the SDK, its methods, and how it simplifies integration.

The SDK is written in TypeScript, which means you get type definitions and IntelliSense if you use it in a TypeScript project or a modern IDE. It can also be used in plain JavaScript projects.

***

### Installation

Install the SDK via npm or yarn (as shown in the QuickStart):

```
npm install @chaingpt/smartcontractgenerator
# or
yarn add @chaingpt/smartcontractgenerator
```

This adds the SDK to your project’s dependencies.

{% hint style="info" %}
_Note: Ensure you have Node.js installed. The SDK runs in Node and is not intended for direct use in browsers, as it would expose your API key. You’d typically call the SDK from your server-side code._
{% endhint %}

***

### Initialization and Authentication

To start using the SDK, import the `SmartContractGenerator` class from the package and initialize it with your API key:

```javascript
javascriptCopyEditimport { SmartContractGenerator } from "@chaingpt/smartcontractgenerator";

const smartcontractGenerator = new SmartContractGenerator({
  apiKey: "YOUR_API_KEY_HERE"
});
```

Replace `"YOUR_API_KEY_HERE"` with your actual key. As noted earlier, it’s best to not hard-code it; instead, load from an environment variable or config file:

```javascript
javascriptCopyEditconst smartcontractGenerator = new SmartContractGenerator({
  apiKey: process.env.CHAINGPT_API_KEY
});
```

Once initialized, `smartcontractGenerator` becomes your main interface to call the generator methods. You generally only need to initialize once (perhaps at app startup) and reuse this instance.

Under the hood, the SDK is storing the API key and will attach it to all requests made to ChainGPT’s servers. It also sets the appropriate base URL and other config. You don’t need to specify the model name (`"smart_contract_generator"`) on each call; the SDK handles that for you.

***

### Core Methods

The SmartContractGenerator SDK instance provides several methods. The key ones are:

* `createSmartContractStream(options)` – Generate a smart contract and get the result as a stream (useful for real-time data).
* `createSmartContractBlob(options)` – Generate a smart contract and get the full result as a resolved promise (blob meaning the whole output).
* `getChatHistory(options)` – Retrieve past chat history (if any) from previous calls.

All these methods return promises (the first returns a promise that resolves to a stream, the second resolves to a response object, and the third to a history object). Let’s look at each in detail.

#### `createSmartContractStream(options)`

**Description:** Sends a prompt to the AI and returns a Node.js readable stream that will yield the response chunks. Use this if you want to start processing or displaying the AI’s answer incrementally (e.g., show typing effect or progress). This corresponds to the streaming API endpoint under the hood.

**Options Parameter:** an object with the following fields:

* `question` (string, required) – The prompt or request describing the contract you want (same as the API’s question parameter).
* `chatHistory` (string, optional) – `"on"` or `"off"`. Set to `"on"` to enable conversation mode (the AI will consider prior context and the conversation will be recorded in history). Default is `"off"`.
* `sdkUniqueId` (string, optional) – If you want to associate this request with a specific user or session for the sake of chat history isolation, provide a unique ID. If not provided, and you have chatHistory on, history will be stored in a general context tied to your API key.

Example usage:

```javascript
javascriptCopyEditconst stream = await smartcontractGenerator.createSmartContractStream({
  question: "Give me a Solidity contract for a simple decentralized voting system.",
  chatHistory: "on",
  sdkUniqueId: "session-42"
});
stream.on("data", chunk => {
  process.stdout.write(chunk.toString());
});
stream.on("end", () => {
  console.log("\n[Generation complete]");
});
```

In this example, we ask for a voting system contract. We turned `chatHistory` on, meaning this Q\&A will be stored and the context can be used for follow-ups. We provided a `sdkUniqueId` (“session-42”) to label this conversation (so if another user is concurrently using the generator with a different ID, their history stays separate).

**Return Value:** A Promise that resolves to a Readable Stream (`stream.Readable`). You handle it by attaching event listeners as shown. Each `data` event provides a `Buffer` or string chunk; convert to string to get the text. An `end` event signifies no more data. You can also listen for an `error` event on the stream in case something goes wrong during the streaming.

_(Advanced: If you prefer using async iterators, you can also `for await` on the stream in modern Node, treating it like an async iterable.)_

#### `createSmartContractBlob(options)`

**Description:** Sends a prompt and returns the full response once ready. This is convenient if you just want the final result without dealing with streaming. Internally, the SDK still calls the streaming endpoint but collects the output for you, or it calls a non-streaming variant of the API if available. The end result is given to you in one object.

**Options Parameter:** same fields as `createSmartContractStream` (question, chatHistory, sdkUniqueId).

Example usage:

```javascript
javascriptCopyEditconst response = await smartcontractGenerator.createSmartContractBlob({
  question: "Create a smart contract that manages a simple crowdfunding campaign (contributors can send ETH, goal target, deadline, etc).",
  chatHistory: "off"
});
const contractCode = response.data.bot;
console.log(contractCode);
```

After this call, `response` will hold the answer. Typically it has a structure similar to the API JSON. For instance, `response.data.bot` might contain the contract code string, and `response.data.user` could echo the prompt. If there was an error, this call would throw (to be caught in a try/catch).

**Return Value:** A Promise that resolves to a response object. You can inspect `response.data` to get the actual content. The SDK’s documentation or type definitions can give more details on the exact shape, but usually it’s an object with a `data` property that includes `bot` (the code) and possibly other metadata. If `chatHistory: "on"` was used, the history is stored on the server side but not necessarily visible directly in this response object aside from affecting the content.

Use this method when you want simplicity and have no need to stream output. For example, in a web backend, you might call this to get the code, then send that code back in an API response to your frontend.

#### `getChatHistory(options)`

**Description:** Fetches the stored chat history of past prompts/responses for the given context. This is useful if you want to display or analyze previous interactions.

**Options Parameter:** an object with the following:

* `limit` (number, optional) – How many records to retrieve (default might be 10).
* `offset` (number, optional) – For pagination (default 0).
* `sortBy` (string, optional) – e.g. `"createdAt"`.
* `sortOrder` (string, optional) – `"ASC"` or `"DESC"`.
* `sdkUniqueId` (string, optional) – If you want history for a specific session ID. If you have been using `sdkUniqueId` for generation calls, you should pass the same one here to get that session’s history. If omitted, it will fetch the general history for the API key (or whatever default session the service uses).

Example usage:

```javascript
javascriptCopyEditconst historyResponse = await smartcontractGenerator.getChatHistory({
  limit: 5,
  sortBy: "createdAt",
  sortOrder: "DESC",
  sdkUniqueId: "session-42"
});
for (const record of historyResponse.data.rows) {
  console.log("Q:", record.question);
  console.log("A:", record.answer || record.bot);
  console.log("---");
}
```

This would print the last 5 Q\&A pairs from session-42, newest first.

**Return Value:** A Promise that resolves to a history object. The structure typically has `data.rows` which is an array of records. Each record may include fields like question, answer (or `bot`), and timestamps. It may also include an `id` for the record. Check the SDK types for exact details.

**Note:** If no history is present (or you haven’t enabled chat history in prior calls), `data.rows` may be empty. Also remember, calling this will consume a credit as a standard API call (make sure this is acceptable or keep an eye on usage).

#### Error Handling in SDK

All SDK methods can throw errors if something goes wrong (they return promises that can reject). The SDK provides an `Errors` export with a class `SmartContractGeneratorError`. When a request fails due to an API issue (like a network error, or the API returning an error status), the SDK will likely throw an instance of this error class.

As shown in the quickstart example, you can check `error instanceof Errors.SmartContractGeneratorError` to distinguish between known API errors vs other unexpected errors. The `error.message` will contain a human-readable message (often coming from the API’s error response). For example, if you ran out of credits, the message might say something like “Insufficient credits” etc.

It’s good practice to wrap your SDK calls in try/catch if there is any chance of failure (which is any I/O or external call realistically). This way you can handle scenarios such as:

* No internet connectivity / DNS issues (your server can’t reach ChainGPT API).
* Invalid API key (maybe a misconfiguration – you’d get an auth error).
* Out of credits or quota issues.
* Invalid parameters (though if you pass all required fields correctly, this is less likely).
* Service downtime.

By catching and logging or handling these, your application can fail gracefully (maybe show an error to the user like “Service temporarily unavailable, please try later” or prompt yourself to fix an API key issue).

#### Usage Patterns and Workflows

To integrate the SDK effectively, here are some common patterns:

* **Single Prompt → Code:** For apps where the user just inputs a description and gets a contract (one-shot), you can directly use `createSmartContractBlob`. E.g., user fills a form describing a token, you call the SDK, get code, and display or save it.
* **Interactive Refinement:** If you want a chat-like experience where the user can iteratively refine the contract (e.g., “Now add this feature…”), use `createSmartContractStream` or `...Blob` with `chatHistory: "on"`. Keep track of a `sdkUniqueId` per conversation (for example, you might generate a UUID when a user starts using your tool, and stick with it through their session). This will allow follow-up questions to build on earlier ones. Use `getChatHistory` if you need to display the conversation.
* **Batch Generations:** If you need to generate multiple contracts programmatically (say, a batch job of various contract templates), you can loop and call the SDK multiple times. Just be mindful of the 200 requests/minute limit. If doing large batches, add delays or breaks to not exceed the rate limit.
* **Parallel Usage:** The SDK calls are I/O-bound (network requests). You can initiate multiple calls in parallel if needed (e.g., Promise.all on several `createSmartContractBlob` calls). The 200/minute limit still applies, but a few in parallel is fine. Too many parallel might hit that limit quickly. If using streams in parallel, ensure you manage the asynchronous nature properly (each returns its own stream).
* **Streaming vs Blob decision:** Use streaming when you want to start processing output immediately (maybe for a UI or to start working on partial data). Use blob when you just need the final answer and perhaps your environment is easier to code that way (e.g., a function that returns the whole result). They ultimately achieve the same end result.

#### Managing Chat History & Sessions

As discussed, if your application involves multiple distinct conversations or users, you should utilize `sdkUniqueId` for isolation. A common practice:

* When a user begins a session (for instance logs in or starts a new “project” in your app), generate a unique ID (could be their user ID plus a timestamp or a random GUID).
* Store that in the context (session storage or database associated with the user).
* For every SDK call related to that user’s conversation, pass that ID and set `chatHistory: "on"` if you want continuity.
* If the user starts a completely new unrelated conversation (like a new project), you might use a new ID.
* This prevents the AI from mixing contexts between different users or different projects.

The SDK does not automatically manage multiple sessions for you; it’s stateless between calls. So it’s up to you to provide the identifier each time. If you don’t, and you use chatHistory on, all those calls could be considered one big ongoing chat (which might or might not be what you want).

If you only ever do single-turn calls (`chatHistory: "off"` every time), then you don’t need to worry about any of this – no history is stored, and each call is independent.

#### SDK Internals (Under the Hood)

For those curious: the SDK essentially wraps the HTTP calls we described in the API section. It sets up an axios instance (or similar) with the base URL and your API key in headers​. When you call `createSmartContractStream` or `...Blob`, it sends a POST request to `/chat/stream` with the appropriate body. The difference is in how it handles the response:

* For stream, it returns the axios response as a stream (not consuming it, just passing it through to you).
* For blob, it likely collects the data from the stream and returns once done (or uses a non-stream call if that exists).
* `getChatHistory` performs a GET request to `/chat/chatHistory` and returns the parsed result.

Understanding this can reassure you that using the SDK is not missing any capabilities of the raw API – it’s simply making it easier to call.

***

### Rate Limits & Credits (SDK)

When using the SDK, it’s important to know that **all the same rate limit and credit rules apply** as when using the raw API. The SDK does not abstract or circumvent those; it operates on your behalf. Each method call that triggers an API request will count towards your 200 requests/minute quota and will deduct credits from your account in the same way:

* Each `createSmartContractStream` or `...Blob` call will deduct 1 credit (2 if chatHistory is on)​.
* Each `getChatHistory` call will deduct 1 credit (just like any data retrieval call).
* The SDK itself doesn’t enforce the 200 RPM limit in code (it doesn’t queue or throttle for you), so if you call it in a tight loop more than 200 times in a minute, you will start getting errors from the API. Implement throttling on your side if needed for heavy usage.

Always ensure you have sufficient credits before making a batch of calls via the SDK. The SDK doesn’t provide a built-in method to check your credit balance – you’d do that via the ChainGPT web dashboard. It will, however, throw an error if a call fails due to lack of credits (the error message from the API should indicate that).

***

### Security Considerations (SDK)

Most security considerations for the SDK revolve around **API key handling**, since the SDK is essentially acting as a client to the API:

* Do not expose the SDK (with your key) in any client-side (browser) code. If you are building a web application, use the SDK in your server (Node.js backend) only. Your frontend can make requests to your backend, which then calls the SDK. This keeps the API key hidden.
* If you’re building an Electron app or some desktop app with Node, you can use the SDK directly, but be cautious if the code can be inspected or if the environment is untrusted.
* Regularly update the SDK to get the latest fixes and improvements (run `npm update @chaingpt/smartcontractgenerator` when a new version is announced).
* The SDK itself handles connecting to `https://api.chaingpt.org` with HTTPS, so network traffic is encrypted. Just ensure your environment can verify SSL certificates (most Node installations do this by default).
* Clean up any streams or processes if needed. For instance, if a user cancels an operation, you might call `stream.destroy()` on a stream you got, to stop processing further. This is more of a resource management tip, but relevant if you integrate streaming in an interactive app.

***
