# SDK Reference

## ChainGPT Smart Contract Auditor SDK Reference

### Introduction

#### Purpose and Scope

The **ChainGPT Smart Contract Auditor SDK** is a software development kit that allows developers to integrate ChainGPT's smart contract auditing capabilities directly into their own applications, platforms, or services. It provides a convenient library for programmatically sending smart contracts for audit and receiving AI-generated audit reports. This reference documentation covers the SDK's features, usage, and integration guidelines in detail, enabling you to quickly adopt the SDK in your development workflow.

#### Intended Audience

This documentation is intended for developers, software engineers, and system integrators who plan to use or are already using the Smart Contract Auditor SDK. It assumes a working knowledge of JavaScript/TypeScript and Node.js. Readers should be comfortable with installing npm packages and handling asynchronous operations (Promises/`async`/`await`) in Node.js.

#### Prerequisites

Before using the Smart Contract Auditor SDK, ensure you have the following prerequisites:

* **Node.js environment:** A supported operating system (Windows, macOS, Linux) with Node.js installed (JavaScript/TypeScript runtime).
* **Development tools:** A project or IDE setup for Node.js development (e.g., npm or Yarn for package management).
* **ChainGPT API access:** An active ChainGPT account with access to the Smart Contract Auditor service.
* **API Key:** A ChainGPT API key associated with your account (obtained from the ChainGPT web application).
* **Credits on ChainGPT:** Sufficient credits in your ChainGPT account (via the ChainGPT web app at [app.chaingpt.org](https://app.chaingpt.org/)). The SDK consumes credits for each audit request (details on credit usage are explained below).

Having these in place will ensure a smooth integration with the SDK.

***

### Getting Started

#### Installation

To install the ChainGPT Smart Contract Auditor SDK in your Node.js project, use either npm or Yarn:

```bash
npm install --save @chaingpt/smartcontractauditor

# or for yarn use:
yarn add @chaingpt/smartcontractauditor
```

This will add the SDK package to your project dependencies. The library includes TypeScript type declarations, so it can be used in TypeScript or JavaScript projects seamlessly.

#### Configuration

Before making any audit requests, you need to configure authentication and ensure your ChainGPT account has credits:

* **Credits System:** The Smart Contract Auditor SDK operates on a credit-based system. Each audit request will deduct credits from your ChainGPT account. Make sure you have purchased or earned adequate credits on the ChainGPT web application. _(Each audit call consumes 1 credit; enabling additional features like chat history will consume an extra credit per call, as described later.)_
* **Obtain API Key:** The SDK uses API keys for authenticating requests. Generate a secret API key from the ChainGPT web dashboard and use it in your application. To get an API key:
  1. Log in to the ChainGPT web application and navigate to the **API Dashboard**.
  2. Use the **Create Secret Key** feature to generate a new API key.
  3. Copy the generated API key (it will be shown only once).\
     &#xNAN;_&#x4E;ote:_ Treat this key like a password; **do not share or expose it publicly**.
* **Secure Storage of API Key:** For security, **avoid hard-coding the API key** in your source code or committing it to a repository. Instead, store it in a secure location such as an environment variable or a secrets management service. For example, you might set an environment variable `CGPT_API_KEY` and retrieve it in your Node.js application, so the key isn't directly visible in code.

With credits available and an API key in hand, you can proceed to initialize the SDK in your project.

#### API Key Setup and Initialization

To use the SDK, import the `SmartContractAuditor` class from the package and initialize it with your API key. This will create an SDK client instance that you will use to call audit methods. For example:

```typescript
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

// Initialize the SDK client with your ChainGPT API key
const smartcontractauditor = new SmartContractAuditor({
  apiKey: process.env.CGPT_API_KEY || 'YOUR-CHAINGPT-API-KEY',
});
```

Once initialized, you can call the SDK methods to audit smart contracts. Here's a quick example of auditing a simple Solidity contract using the SDK, with the result streamed back:

```typescript
async function main() {
  const stream = await smartcontractauditor.auditSmartContractStream({
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
    chatHistory: "on"  // enable chat history to preserve context (optional)
  });

  // Handle streaming response
  stream.on('data', (chunk) => {
    console.log(chunk.toString());  // output audit result chunk
  });
  stream.on('end', () => {
    console.log("Audit stream ended.");
  });
}

main().catch(console.error);
```

In this example, we ask the auditor to **audit a simple Counter contract**. We set `chatHistory: "on"` to save this query in the conversation history (more on this in the **Chat History** section). The SDK returns a readable stream, which we listen to for `data` events to get incremental chunks of the audit report, and an `end` event when the streaming is complete.

{% hint style="info" %}
**Note:** The above code uses `auditSmartContractStream` for demonstration. The SDK also provides methods to get the full response without streaming. We will cover both usage patterns in the SDK Components section.
{% endhint %}

***

### SDK Components

The ChainGPT Smart Contract Auditor SDK provides a set of methods and features for auditing smart contracts and managing the audit sessions. All functionality is accessed via the `SmartContractAuditor` instance that you created with your API key. This section details the core methods, advanced features, chat history management, and error handling in the SDK.

#### Core Methods

The core functionality of the SDK revolves around sending a smart contract (or any prompt) to ChainGPT's auditor and receiving an audit report. The primary methods to accomplish this are:

**`auditSmartContractBlob(options)`**

Audits a smart contract and returns the **full result** as a single response object (a "blob" of data). Use this method when you want to get the complete audit report in one go (for example, if you plan to display or process the entire report after the analysis is finished).

* **Parameters:** This method accepts an options object with the following properties:
  * `question` (string): The prompt for the auditor. Typically, this should include the smart contract code and any specific questions or instructions. For example, you can provide a prompt like: _"Audit the following contract: \[contract code]"_. The contract code can be included directly in the string (as shown in examples above).
  * `chatHistory` (string, optional): Set to `"on"` to enable chat history for this request, or `"off"` to treat it as a standalone query. If `"on"`, the conversation context will be preserved (and can be retrieved later), allowing for follow-up questions referencing this audit. The default is `"off"` if not provided.
  * `sdkUniqueId` (string, optional): A unique identifier for the end-user or session. This is used to segregate chat history per user in your application. If you have multiple users or sessions using the same API key, you can supply a unique UUID or identifier here to keep their audit conversations separate. (See **Chat History** below for more details.)
* **Return Value:** This method returns a **Promise** that resolves to a response object containing the audit result. On success, you can expect the response to include the auditor's findings. For example, the resolved object may have a structure like: `{ data: { bot: "…audit report text…" } }`. You can then extract the audit text (e.g., `response.data.bot`) and use it in your application.
*   **Usage Example:** Auditing a contract and logging the full report:

    ```typescript
    const response = await smartcontractauditor.auditSmartContractBlob({
      question: `Audit the following contract:
      pragma solidity ^0.8.0;
      contract Counter { 
        /* contract code */ 
      }`,
      chatHistory: "off"  // no chat context needed for a one-off query
    });
    console.log("Audit Report:", response.data.bot);
    ```

    In this example, `auditSmartContractBlob` is called with a Solidity contract in the question. We set `chatHistory: "off"` since we only want an isolated analysis. The result is then printed from `response.data.bot`. (In a real application, you might format or display this result in a UI.)

**`auditSmartContractStream(options)`**

Audits a smart contract and returns a **streaming response**, allowing you to receive the audit report progressively as it is generated. This method is useful for providing real-time feedback (for example, streaming the audit output to a console or UI as it comes in) or for handling very large responses without waiting for the entire result.

* **Parameters:** It accepts the same options as `auditSmartContractBlob`:
  * `question` (string): The audit prompt including contract code.
  * `chatHistory` (string, optional): `"on"` or `"off"` (default). When enabled, the request and its response will be logged in the chat history for retrieval or context.
  * `sdkUniqueId` (string, optional): Unique session/user identifier for segregating history (if needed).
* **Return Value:** This method returns a **Node.js Readable Stream**. You should attach event listeners to handle the stream:
  * Listen for `'data'` events to receive chunks of the audit output. Each chunk is typically a `Buffer` or string segment of the report. You may need to convert it to string (as in the examples) to handle text.
  * Listen for the `'end'` event to know when the audit report is fully received (the stream has finished).
  * Optionally, listen for `'error'` events to catch any errors emitted during streaming.
*   **Usage Example:** Auditing a contract with streaming output:

    ```typescript
    const stream = await smartcontractauditor.auditSmartContractStream({
      question: `Audit the following contract:
      pragma solidity ^0.8.0;
      contract Counter { 
        /* contract code */ 
      }`,
      chatHistory: "on"
    });
    stream.on('data', (chunk) => {
      process.stdout.write(chunk.toString());  // stream chunks to stdout or handle them as needed
    });
    stream.on('end', () => {
      console.log("\n[Audit complete]");
    });
    ```

    Here, we enabled `chatHistory: "on"` to save the interaction. As the ChainGPT auditor generates the report, data chunks are streamed and immediately written to standard output. When the stream ends, we log a completion message. This allows the user to start seeing audit results without waiting for the entire analysis to finish.

**Credit Usage:** Each call to `auditSmartContractBlob` or `auditSmartContractStream` will deduct **1 credit** from your ChainGPT account for the audit operation. If you enable chat history (`chatHistory: "on"`), the call will consume **1 additional credit** (so 2 credits total for that request) to cover the cost of storing and managing the conversation context. Please ensure your account has enough credits before making requests. If a request is made without sufficient credits or with an invalid API key, it will fail with an error.

#### Advanced Features

Beyond the basic request/response workflow, the Smart Contract Auditor SDK offers several advanced features to enhance integration:

* **Streamed Responses:** The SDK supports real-time streaming of responses (via `auditSmartContractStream`). This is useful for providing immediate feedback to users or handling large responses efficiently. You can process chunks of the audit output as they arrive, improving the responsiveness of your application.
* **Full Response (Blob) Mode:** If streaming is not needed, you can use the blob mode (`auditSmartContractBlob`) to get the complete response after processing. This mode is straightforward and behaves like a typical API call, encapsulating the entire result in one response object.
* **Chat History Toggle:** The SDK allows you to turn on conversation memory for audit requests. By setting `chatHistory: "on"`, each query and its response are recorded. This enables context to be maintained between subsequent calls (you can ask follow-up questions referencing previous answers) and allows you to retrieve past audits. If you don't need this, keep `chatHistory: "off"` to treat each request independently.
* **User-Specific Sessions:** For applications with multiple end-users, you can isolate each user's audit history. Using the `sdkUniqueId` parameter (a unique user or session ID) with your requests ensures that each user's chat history is kept separate on ChainGPT's side. This prevents cross-contamination of context between different users and lets you retrieve or manage each user's history individually.
* **Credit and Rate Limits:** The SDK is integrated with ChainGPT's credit system and usage policies. As noted, every audit request costs credits (1 or 2 depending on history usage). Additionally, to ensure fair usage and system stability, there is a rate limit of **200 requests per minute** per API key. If you exceed this rate, subsequent requests may be rejected or throttled. Design your application to spread out requests or queue them if you anticipate a high volume of audits.

By leveraging these features appropriately, you can build robust and user-friendly tools on top of the ChainGPT auditor. For example, you might use streaming to show live progress in a UI, enable chat history for an interactive auditing session with follow-up queries, and assign unique IDs for each project or user being audited.

#### Chat History

One of the powerful features of the Smart Contract Auditor SDK is the ability to maintain and retrieve a history of audit conversations. When chat history is enabled, the SDK will store the interactions, allowing for context continuity and later review. This section explains how to use the chat history feature.

**Enabling Chat History:** To use chat history, simply set `chatHistory: "on"` in your audit request (either stream or blob). When enabled:

* The current question and the auditor's answer will be logged on ChainGPT's servers.
* Future audit requests with `chatHistory: "on"` (using the same API key and not specifying a new unique ID) may include previous context, allowing the auditor to reference earlier discussions or findings.
* If you disable history (`chatHistory: "off"`), each request is handled in isolation with no memory of prior interactions.

**User-Specific History with `sdkUniqueId`:** If your application serves multiple users or if you want to separate history by session or project, use the `sdkUniqueId` field:

* Provide a unique identifier (for example, a UUID or user ID string) with each audit request for a given user/session. This tells ChainGPT to store the history under that ID.
* When a different `sdkUniqueId` is used, a separate history thread is maintained. This ensures user A's audits and chat context are distinct from user B's.
* If no `sdkUniqueId` is provided, the history is associated with your API key globally (meaning all your requests could be in one conversation thread). It's recommended to use an ID if you have more than one context to manage.

For example, to audit a contract with chat history enabled for a specific user session:

```typescript
const userId = "907208eb-0929-42c3-a372-c21934fbf44f"; // unique ID for a user or session
const response = await smartcontractauditor.auditSmartContractBlob({
  question: "Audit the following contract:\npragma solidity ^0.8.0; contract Counter { ... }",
  chatHistory: "on",
  sdkUniqueId: userId
});
// This call's context is now tied to the provided userId
```

In the above snippet, the audit is performed with `chatHistory` on, and the conversation is tagged with a specific `userId`. If you later make another call with the same `userId` and `chatHistory: "on"`, the auditor will have the context of the previous contract audit for that user.

**Retrieving Chat History:** The SDK provides a method `getChatHistory` to fetch the stored conversation history. This can be useful to display past audits to the user or to review previous questions and answers.

**`getChatHistory(options)`**

Fetches the audit/chat history that has been recorded for your API key (and for a specific `sdkUniqueId` if you used one in your requests).

*   **Parameters:** An object with the following fields to filter and paginate the history:

    * `limit` (number, optional): The maximum number of history records to retrieve. Example: `10` to get the 10 most recent entries.
    * `offset` (number, optional): The offset (starting index) for records retrieval, useful for pagination. For example, `0` to start from the most recent record, `10` to skip the first 10 records.
    * `sortBy` (string, optional): The field by which to sort the history entries. Currently, `"createdAt"` (the timestamp of the record) is used.
    * `sortOrder` (string, optional): Sort direction, either `"ASC"` for ascending or `"DESC"` for descending order. Typically, you'd use `"DESC"` to get the latest records first.

    _Note:_ If you have used `sdkUniqueId` in your audit requests to separate histories, the returned history will correspond to the context of the API key and unique ID that the SDK instance is aware of. Ensure that if you're managing multiple user histories, you call `getChatHistory` accordingly (for example, by using separate `SmartContractAuditor` instances or configuring the `sdkUniqueId` consistently before retrieval).
* **Return Value:** A Promise that resolves to an object containing the history entries. The structure is typically `{ data: { rows: [ ... ] } }`, where each item in `rows` represents a past question/answer or audit record. You can inspect these records to see the content of questions asked and answers given.
*   **Usage Example:** Retrieving the last 10 audit history entries:

    ```typescript
    const history = await smartcontractauditor.getChatHistory({
      limit: 10,
      offset: 0,
      sortBy: "createdAt",
      sortOrder: "DESC"
    });
    console.log("Recent Audit History:", history.data.rows);
    ```

    This will output an array of up to 10 history records, sorted by creation time in descending order (newest first). Each entry in `history.data.rows` may include details such as the question prompt, the audit result, and timestamps. You can use this data to display a history log in your application.

**Credit Consideration for History:** Retrieving chat history via `getChatHistory` **does not consume additional credits** beyond what was already spent when initially enabling and storing that history. In other words, you pay the extra credit cost once when you call an audit with `chatHistory: "on"`, but fetching the saved history is free of charge. (If chat history was not enabled for any requests, there will be no records to retrieve.)

#### Error Handling

When using the SDK, you should be prepared to handle errors, such as network issues or invalid requests. The Smart Contract Auditor SDK will throw a specialized error class for API-related issues.

If the SDK fails to connect to the ChainGPT service, or if the API returns an error status (e.g., a 4xx client error or 5xx server error), it will throw an error of type `SmartContractAuditorError`. You can import this error class via the `Errors` object from the SDK and use it to differentiate SDK errors from other exceptions.

Here is how you might implement error handling for an audit request:

```typescript
import { SmartContractAuditor, Errors } from "@chaingpt/smartcontractauditor";

const smartcontractauditor = new SmartContractAuditor({ apiKey: 'YOUR-API-KEY' });

async function runAudit() {
  try {
    const stream = await smartcontractauditor.auditSmartContractStream({
      question: "Audit the following contract:\npragma solidity ^0.8.0; contract Counter { ... }",
      chatHistory: "on"
    });
    // If successful, use the stream as usual:
    stream.on('data', (chunk) => { /* handle data */ });
    stream.on('end', () => { /* handle completion */ });
  } catch (error) {
    if (error instanceof Errors.SmartContractAuditorError) {
      // An error occurred with the audit request
      console.error("Audit SDK Error:", error.message);
    } else {
      // Some other error (programmer error, etc.)
      console.error("Unexpected Error:", error);
    }
  }
}

runAudit();
```

In the example above, we wrap the audit call in a `try...catch` block. If an error is thrown, we check if it is an instance of `Errors.SmartContractAuditorError`. This indicates a failure related to the API request (for example, invalid API key, rate limit exceeded, insufficient credits, network timeout, etc.). We then handle it appropriately (logging the error message or alerting the user). Other types of errors (for instance, a coding bug in your application) can be handled in the `else` branch.

Always implement proper error handling around SDK calls to ensure your application can gracefully handle scenarios such as lack of credits, lost connectivity, or unexpected responses.

***

### Security Considerations

When integrating the ChainGPT Smart Contract Auditor SDK, it’s important to follow best practices to keep your application and data secure:

* **API Key Security:** Your ChainGPT API key is the credential that grants access to the auditor service. **Never expose this key in client-side code or public repositories.** Treat it like a password. As mentioned, load it from a secure source (environment variable or secret manager) rather than hard-coding. Rotate the key if you suspect it has been compromised.
* **Authentication Required:** The SDK will only function with a valid API key that has sufficient credits. Unauthorized requests will fail. This ensures that only authenticated and credited users can access the audit service.
* **Credit Management:** Monitor your ChainGPT credit balance and usage. If your application might make many audit requests, implement checks or alerts for low credit status to avoid service interruptions. Remember that enabling chat history doubles the credit cost per request, so use it only when needed.
* **Rate Limiting:** As a safeguard against abuse or accidental overuse, the ChainGPT API enforces a rate limit of **200 requests per minute** per API key. Design your integration to respect this limit. If you anticipate bursts of requests (for example, a batch of contracts to audit), consider adding delays or spreading the requests over time. Hitting the rate limit will result in errors (likely `SmartContractAuditorError` with a message about too many requests).
* **Data Privacy:** The smart contracts and prompts you send to ChainGPT are processed by an AI model on ChainGPT's servers. While this is necessary for the service to function, be mindful of **sensitive information**. Avoid sending any private keys, passwords, or truly sensitive data in your audit prompts. Only send the code and information needed for the audit. Check ChainGPT’s privacy and data handling policies if this is a concern for your use case.

By following these security considerations, you can safely integrate the SDK into your application while protecting your credentials and respecting usage limits.

***

### Compatibility

The Smart Contract Auditor SDK is designed for **Node.js** environments and is distributed via npm. It supports both JavaScript and TypeScript:

* **Language:** You can use the SDK in JavaScript or TypeScript. The package is written in TypeScript and provides built-in type declarations, which means you get type hints and compile-time checks if you use TypeScript.
* **Runtime:** The SDK should run on any recent Node.js version that supports ES modules or CommonJS (the package can be imported using modern `import` syntax as shown, or via `require` in CommonJS if needed). Ensure Node is installed on your system.
* **Platform:** It is OS-agnostic – you can use it on Windows, Linux, or macOS. Essentially, any platform that can run Node.js can utilize the SDK.
* **Environment:** The SDK is intended for server-side or backend use (Node.js). Using it directly in a browser is not officially supported, as it requires a secure API key and interacts with a backend service. If you need to call it from a frontend context, you should route the requests through a backend server to keep the API key hidden.

There are no known framework-specific requirements; you can integrate it into any Node.js application, whether it's a plain Node script, an Express backend, a Next.js API route, etc.

***

### Release Information

This SDK is under active development, and version updates will be published as new features or improvements are added. Key release information:

* **Current Version:** – This is the initial release of the ChainGPT Smart Contract Auditor SDK. It includes core auditing functionality, streaming support, chat history management, and error handling features as described above.
* **Release History:** As this is the first release, no previous versions are available. Going forward, version changes and release notes will be documented (e.g., in a changelog on the repository or npm package page).
* **Future Updates:** Planned enhancements may include expanded methods, improved performance, and additional auditing features. Always refer to the official documentation or the npm package page for information on the latest version and its changelog before upgrading.

To ensure compatibility, pin your dependency to a specific major version or use semantic version ranges as appropriate, and test your application after upgrading the SDK.

***

### Code Documentation and Support

For more detailed reference on classes, methods, and types, you can refer to the SDK's code documentation and resources:

* **SDK Documentation on npm:** The [ChainGPT Smart Contract Auditor SDK npm page](https://www.npmjs.com/package/@chaingpt/smartcontractauditor) contains a README with usage examples (similar to those in this guide) and may be updated with additional details for each release. It also lists the package’s dependencies and version information.
* **Source Repository:** (If available) Check if ChainGPT provides access to the SDK’s source code or a GitHub repository. The repository might contain a README, Wiki, or inline documentation for deeper insight into the SDK's implementation.
* **Type Definitions:** Since the SDK includes TypeScript definitions, your IDE can provide intellisense. You can inspect the types of functions and objects (for instance, the shape of the response objects, error classes, etc.) by hovering in your code editor or looking up the `.d.ts` files after installation.

**Support:** If you encounter issues or have questions while using the SDK, the ChainGPT team provides support through multiple channels:

* Visit the official **ChainGPT website** [chaingpt.org](https://www.chaingpt.org/) for links to support resources. There may be a dedicated support portal, community forum, or contact email for developer support.
* **Community & Forums:** ChainGPT might have community support on platforms such as Discord or a forum where you can ask questions and share knowledge with other developers.
* **Documentation Portal:** The ChainGPT documentation site (for example, docs.chaingpt.org) could have additional guides, FAQs, and troubleshooting tips for the Smart Contract Auditor and other ChainGPT tools.
* **Contacting Support:** For urgent or specific issues, reach out to ChainGPT’s support team through the channels provided on their website. Be ready to provide details such as your API key (if requested safely), SDK version, and any error messages or logs to expedite the assistance process.

By leveraging the above resources, you should be able to resolve most integration challenges and stay updated on the SDK’s development. Happy auditing with ChainGPT’s Smart Contract Auditor SDK!
