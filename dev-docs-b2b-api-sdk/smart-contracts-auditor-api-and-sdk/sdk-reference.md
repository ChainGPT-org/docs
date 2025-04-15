# SDK Reference

## Smart-Contracts Auditor - SDK Reference

The ChainGPT Smart Contract Auditor **SDK** is a Node.js library that provides a convenient interface to the Auditor API. Instead of manually constructing HTTP requests, you can use this SDK to integrate auditing functionality into your JavaScript/TypeScript applications with just a few method calls. This section documents the classes, methods, and usage patterns of the SDK.

***

### Installation & Import

To use the SDK, install the `@chaingpt/smartcontractauditor` package from npm:

```shell
shellCopyEditnpm install @chaingpt/smartcontractauditor
```

_(You may also use `yarn add @chaingpt/smartcontractauditor` if you prefer Yarn.)_

Then import the main class into your Node.js project:

```javascript
javascriptCopyEditimport { SmartContractAuditor } from "@chaingpt/smartcontractauditor";
// If using CommonJS, use: const { SmartContractAuditor } = require("@chaingpt/smartcontractauditor");
```

#### Class: `SmartContractAuditor`

This is the primary class provided by the SDK. It encapsulates the audit functionality.

**Constructor:** `new SmartContractAuditor(options)`

*   **`apiKey`** (string, required) – Your ChainGPT API key. This will be used for all API calls made by this instance. Example:

    ```js
    jsCopyEditconst auditor = new SmartContractAuditor({ apiKey: "YOUR_API_KEY" });
    ```

    Once initialized, the `auditor` instance is ready to make requests. (Under the hood, it’s configured to call the ChainGPT API with the Smart Contract Auditor model.)
* _Note:_ You can instantiate multiple auditors with different keys if needed (for example, if managing multiple user accounts in a server context). Usually one instance is sufficient. Ensure the apiKey is valid and keep it secure (do not expose it on the client side).

#### **Methods:**

**`auditor.auditSmartContractStream(options)`**

Triggers a smart contract audit and returns a **stream** of the results.

* **Parameters:** `options` (object) with:
  * **`question`** (string, required): The audit prompt, i.e. what to audit. This is the same format as the API’s `question` – include the contract source code or an instruction to fetch an address. For example:\
    `"Audit the following contract:\n <SOLIDITY CODE here>"`\
    or\
    `"Audit the smart contract at 0xABCD... on Polygon."`
  * **`chatHistory`** (string, optional): `"on"` or `"off"`. Default is `"off"`. If `"on"`, the SDK will keep the context so you can ask follow-ups with the `auditor` instance (and it will store the history). This corresponds to the API’s chatHistory flag.
* **Returns:** A **Readable Stream** (Node.js `stream.Readable`) that you can listen to for data. The stream will emit `data` events with chunks of the audit report (as `Buffer` or string) and an `end` event when the report is complete. It may also emit an `error` event if something goes wrong during streaming.
*   **Usage:** This method is ideal if you want to start processing or displaying the audit output as it’s being generated (for example, updating a live UI or logging progress). To use it, call it with `await` (since it returns a promise that resolves once the stream is ready), then attach event handlers:

    ```js
    jsCopyEditconst stream = await auditor.auditSmartContractStream({ question: "...", chatHistory: "off" });
    stream.on('data', chunk => { ... });
    stream.on('end', () => { ... });
    stream.on('error', err => { ... });
    ```

    Typically, you’ll concatenate the chunks to build the full response or handle each chunk as it arrives.
* **Example:** See the QuickStart example above for a demonstration. In that example, we audited a `Counter` contract and printed chunks to console.
* **Error Handling:** If the API returns an error immediately (e.g., invalid API key or other request issue), the `auditSmartContractStream` call will throw (reject) instead of returning a stream. You should wrap the call in try/catch. If an error occurs mid-stream, it will be emitted as an `'error'` event on the returned stream. Always handle both cases. The SDK will throw an error of type `Errors.SmartContractAuditorError` (see **Error Handling** below) for API-related issues.

**`auditor.auditSmartContractBlob(options)`**

Triggers a smart contract audit and returns the **full result** once ready, rather than streaming.

* **Parameters:** Same as `auditSmartContractStream` – an `options` object with `question` (string) and optionally `chatHistory` ("on"/"off").
* **Returns:** A **Promise** that resolves to a response object containing the result. Specifically, it returns an Axios response object from the underlying HTTP call. The actual audit text can be accessed via `response.data.bot`. (The SDK does a minimal amount of processing; essentially it wraps the same API call but waits for completion.)
*   **Usage:** Use this method if you prefer a simpler workflow and don’t mind waiting for the entire audit to complete. For example:

    ```js
    jsCopyEdittry {
      const response = await auditor.auditSmartContractBlob({ question: "...", chatHistory: "off" });
      const report = response.data.bot;
      console.log("Audit report:", report);
    } catch (error) {
      // handle error
    }
    ```

    This will only return once the audit is fully done. Keep in mind that an audit can take some seconds (typically under a minute for most contracts). During that time the promise is pending.
* **Example:** In the QuickStart, the `auditByAddress()` function demonstrates using `auditSmartContractBlob` to audit a contract by address. After awaiting the call, it logs `response.data.bot` which contained the report text.
* **Notes:** Because this method buffers the entire response, it internally sets a timeout (e.g., 60 seconds) to avoid hanging indefinitely. Most audits finish well within this. If the contract is extremely large or the server under high load, consider using the stream approach or increasing the timeout in the SDK if supported. For most users, this method is straightforward and works out-of-the-box.

**`auditor.getChatHistory(options)`**

Retrieves the stored chat (audit) history via the SDK.

*   **Parameters:** `options` (object) with optional filtering:

    * `limit` (number, optional) – number of records to fetch.
    * `offset` (number, optional) – starting index for records.
    * `sortBy` (string, optional) – field to sort by, e.g. `"createdAt"`.
    * `sortOrder` (string, optional) – `"ASC"` or `"DESC"` (not case-sensitive).

    If no options are provided, defaults (like latest records) are used. This aligns with the API’s query parameters for history.
* **Returns:** A **Promise** that resolves to a response object containing the history data. You can access the array of history entries via `response.data.rows` (each entry with properties such as `user`, `bot`, etc., as shown in the API reference above).
*   **Usage:** This is a convenience for the GET `/chat/chatHistory` API. For example:

    ```js
    jsCopyEditconst res = await auditor.getChatHistory({ limit: 5, sortOrder: "DESC" });
    const pastChats = res.data.rows;
    pastChats.forEach(entry => {
      console.log(`[${entry.createdAt}] Q: ${entry.user}\nA: ${entry.bot}\n`);
    });
    ```

    This would print the last 5 audit interactions. You could use this in a situation where your application wants to display prior audits or continue a conversation context after a pause.
* **Credits:** No additional credits are charged for calling `getChatHistory`. You pay for the chat history when enabling it on the audit calls. The SDK (and API) do not charge for retrieving the saved data.
*   **Example:** The SDK documentation example:

    ```js
    jsCopyEditconst response = await auditor.getChatHistory({ limit: 10, offset: 0, sortBy: "createdAt", sortOrder: "DESC" });
    console.log(response.data.rows);
    ```

    would output an array of up to 10 most recent chat records.

***

### **Error Handling in SDK**

The SDK provides a unified error class for issues during API calls. If a request fails (due to network issues or a non-200 HTTP response), the SDK will throw an `Errors.SmartContractAuditorError`. You can import this from the package:

```javascript
javascriptCopyEditimport { Errors } from "@chaingpt/smartcontractauditor";

try {
  await auditor.auditSmartContractBlob({ /* ... */ });
} catch (err) {
  if (err instanceof Errors.SmartContractAuditorError) {
    console.error("Auditor API Error:", err.message);
    // You can inspect err.message for details, or err.code if provided
  } else {
    console.error("Unexpected Error:", err);
  }
}
```

For `auditSmartContractStream`, as noted, you should try/catch around the initial call, and also attach an `error` listener to the stream. The error’s `message` will typically contain info like HTTP status or error response from the API (for example, "Unauthorized", "Insufficient credits", etc.).

#### **Common Errors:**

* Invalid API key -> `SmartContractAuditorError` with message about authentication.
* No credits -> error message indicating insufficient funds.
* Request validations (though the SDK should prevent obvious ones like missing `question`).
* Network timeouts or connectivity issues -> could throw a general error or SmartContractAuditorError with a message like "Network Error" or "timeout".

By catching these, you can decide to prompt for a new API key, ask the user to add credits, retry later, etc., depending on the scenario.

***

### Usage Examples

Here are two practical example flows using the SDK, covering both source-code audits and address-based audits:

#### **1. Auditing a Deployed Contract by Address (SDK Example)**

Imagine you want to audit a contract that’s already on-chain. You have its address and you know it’s on a certain network (say Ethereum mainnet). You can simply use that information in the prompt:

```javascript
javascriptCopyEditimport { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const auditor = new SmartContractAuditor({ apiKey: process.env.CHAINGPT_API_KEY });

async function auditDeployedContract() {
  const targetAddress = "0xD3adb33f...";  // example address
  const network = "Ethereum";
  try {
    const response = await auditor.auditSmartContractBlob({
      question: `Audit the smart contract deployed at address ${targetAddress} on ${network}.`,
      chatHistory: "off"
    });
    const auditReport = response.data.bot;
    console.log("Audit completed. Report:");
    console.log(auditReport);
  } catch (error) {
    console.error("Audit failed:", error.message);
  }
}

auditDeployedContract();
```

In this script, we instantiate the auditor with our API key, then call `auditSmartContractBlob` with a prompt that asks the AI to fetch and audit the contract at the given address on Ethereum. The SDK takes care of calling the API. The result is captured in `auditReport` and we print it out. If the contract’s source code was verified and accessible, the report will contain the analysis (if not, the report might note it couldn’t retrieve code or only analyze the bytecode superficially). Typically, for major chains like Ethereum, if the contract is verified on Etherscan, ChainGPT’s auditor will successfully fetch and audit it.

#### **2. Auditing by Source Code (SDK Example)**

Now consider you have the source code locally (perhaps you’re writing the contract and want to audit it pre-deployment). You can pass the code directly:

```javascript
javascriptCopyEditimport { SmartContractAuditor } from "@chaingpt/smartcontractauditor";
const auditor = new SmartContractAuditor({ apiKey: "YOUR_API_KEY" });

const sourceCode = `
pragma solidity ^0.8.4;
contract Token {
    mapping(address => uint256) balances;
    function transfer(address to, uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
`;

async function auditSourceCode() {
  try {
    const stream = await auditor.auditSmartContractStream({
      question: "Audit the following smart contract:\n" + sourceCode,
      chatHistory: "off"
    });
    let fullReport = "";
    stream.on('data', chunk => {
      fullReport += chunk.toString();
    });
    stream.on('end', () => {
      console.log("Audit complete. Full report:\n", fullReport);
    });
  } catch (err) {
    console.error("Error during audit:", err.message);
  }
}

auditSourceCode();
```

Here we used `auditSmartContractStream` to get results as they come in. We build the `fullReport` by concatenating chunks. Once the stream ends, we have the complete report and print it. The contract in this example is a simple token transfer logic. The Auditor might point out that this simplistic token has no initial supply and that anyone could call transfer (which might or might not be an issue depending on context). It might also note potential improvements or confirm that `require` is correctly used.

If you wanted to not stream, you could replace with `auditSmartContractBlob` and simply get `response.data.bot` as before. The content of the report should be identical either way.

***

### SDK Internals and Additional Notes

* **Under the Hood:** The SDK is essentially a wrapper around the HTTP API. It sets the proper base URL (`/chat/stream`) and model, and uses axios (an HTTP client) to make requests. The methods correspond one-to-one with API calls: `auditSmartContractStream` uses a streaming request, `auditSmartContractBlob` waits for the full response, and `getChatHistory` calls the history endpoint. Knowing this, you can trust that any limitations (rate limits, etc.) are the same as described in the API Reference. Using the SDK **does not bypass** any usage limits or costs.
* **Environment & Compatibility:** The SDK is designed for Node.js (JavaScript/TypeScript). It will work in server or backend contexts. It is not intended for browser usage (you wouldn’t want to expose your API key in client-side code anyway). If you need to call the Auditor from a frontend, set up a proxy server or use the API directly from your backend. The SDK requires Node and supports modern Node versions (it uses Promises, async/await, etc.).
* **Streaming and Promises:** Note that `auditSmartContractStream` returns a promise that resolves to a stream. The promise itself rejects on immediate request errors, whereas once you have the stream, you should listen for `'error'` on it. This dual error handling is a common pattern for streaming APIs.
* **Security Considerations:** When using the SDK, treat it as you would any API integration:
  * Do not log sensitive information (like full audit content of private contracts) to insecure places if it contains something sensitive.
  * Handle the API key carefully (we mentioned this multiple times, but it’s worth reiterating).
  * The SDK uses TLS (HTTPS) underneath via the ChainGPT API, so data in transit is encrypted. Still, if you’re auditing highly sensitive contracts that aren’t public, consider the implications of sending the code to an external service.
* **Performance:** Each `auditSmartContractBlob` call is synchronous from the caller perspective (you await it). If you need to audit many contracts concurrently, you can fire off multiple calls (just be mindful of the 200 req/min limit). The SDK doesn’t enforce a client-side rate limit beyond what the server enforces, so it’s on you to throttle if needed.
* **Release Version:** This SDK is in active development. As of this documentation, it’s the initial release (v1.x). Future versions may introduce new features or methods (for example, convenience functions or support for new AI model parameters). Check the npm package page or ChainGPT’s dev docs for update notes and release history.

By using the SDK, you get a developer-friendly way to integrate smart contract audits. Whether you choose the REST API or the SDK will depend on your project’s needs – but both provide the same powerful AI auditing capabilities to help you deploy safer smart contracts.
