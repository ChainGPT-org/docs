# QuickStart Guide

## Smart-Contracts Auditor (API & SDK) - QuickStart Guide

Get up and running with the ChainGPT Smart Contract Auditor in just a few steps. This guide covers both direct API usage and the Node.js SDK usage side-by-side.

***

### Prerequisites

* **ChainGPT Account & API Key:** Sign up at **ChainGPT** (via the [app.chaingpt.org](https://app.chaingpt.org/) dashboard) to obtain an API key. In your account’s API Dashboard, use “Create Secret Key” to generate a key. _Keep this key secure_ – treat it like a password (store in an environment variable or secrets manager, not in code).
* **API Credits:** Ensure your ChainGPT account has sufficient **credits**. Each audit request costs **1 credit** (with an additional 1 credit if chat history is enabled for the request). If you run out of credits, API calls will fail, so top-up beforehand via the ChainGPT app.
* **Development Environment:**
  * For API usage: any environment that can make HTTPS requests (cURL, Postman, or your programming language of choice).
  * For SDK usage: Node.js installed (the SDK supports Node.js and is written in TypeScript/JavaScript).
* **Smart Contract to Audit:** You should have either the Solidity source code of the contract or a deployed **contract address** on a supported network (with verified source code). Supported languages and networks include Solidity contracts on Ethereum and EVM-compatible chains (BNB Chain, Polygon, Arbitrum, Avalanche, etc.), and experimental support for Solana (Rust) contracts. If providing an address, make sure the contract’s source is publicly verified (e.g., on Etherscan) so the Auditor can retrieve and analyze it. Otherwise, plan to supply the source code directly.

***

### Option #1: Using the REST API

With an API key in hand, you can call the Auditor’s REST endpoint directly to submit a contract for auditing. The base URL for audit requests is `https://api.chaingpt.org/chat/stream`. You will send an HTTP **POST** request to this endpoint with your prompt, and receive a streaming response with the audit results.

#### **Steps:**

1. **Set up Authorization:** Prepare your API key in an HTTP **Authorization** header. The format is:\
   `Authorization: Bearer YOUR_API_KEY`\
   Also ensure the request has `Content-Type: application/json` for a JSON payload.
2. **Craft the Audit Request:** In the JSON body, provide:
   * **`model`** – set this to `"smart_contract_auditor"` (this tells ChainGPT to use the Auditor AI model).
   * **`question`** – your audit prompt. This should include either the contract’s source code or a reference to a contract address. For example, you can literally ask: _"Audit the smart contract deployed at 0xABC... on Ethereum."_ or _"Audit the following contract: **...**"_ followed by the Solidity code.
   * **`chatHistory`** (optional) – `"off"` or `"on"`. Usually you’ll set `"off"` for independent one-shot audit requests. If you plan to have a multi-turn conversation (e.g., asking follow-up questions about the same contract), you can use `"on"` to enable context (note: this consumes an extra credit and you can retrieve the history later).
3. **Send the Request:** Use curl or your preferred tool. For example, here’s a `curl` command that asks the Auditor to analyze a contract by address:

```
curl https://api.chaingpt.org/chat/stream \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "smart_contract_auditor",
    "question": "Audit the smart contract deployed at 0x1234567890ABCDEF1234567890ABCDEF12345678 on Ethereum.",
    "chatHistory": "off"
  }'
```

In this example, replace the API key and the `0x...` address with your own values. The Auditor will fetch the contract at that address on Ethereum, analyze it, and begin streaming back the audit report.

4. **Read the Response:** The response is streamed as it’s generated. If using curl from a terminal, you’ll see the audit content start to print out in chunks. The final output will be a detailed report indicating any issues found. For programmatic access, you can read the HTTP response stream to capture the entire report. For example, in Node.js you might use an HTTP client to gather chunks (see the API Reference for a code snippet). If you prefer not to handle streaming, you can wait for the response to complete – the full audit text will then be available in the response body (under a JSON field, usually `bot`).

{% hint style="info" %}
**Best Practice:** When constructing the `question` for code audits, format it clearly. For instance, begin with a directive like **"Audit the following contract:"** followed by the code. This helps the AI focus on the code provided. If you include source code in the JSON request, be sure to escape newlines and quotes properly, or send the code in a single-line string with  separators.
{% endhint %}

***

### Option #2: Using the Node.js SDK

ChainGPT offers an official Node.js SDK to simplify integration. The SDK wraps the raw API calls, handling details like streaming and authentication for you. This is the easiest way to use the Auditor in a Node.js project.

**Installation:** Install the SDK package via npm or yarn:

```
npm install --save @chaingpt/smartcontractauditor
# or
yarn add @chaingpt/smartcontractauditor
```

**Initialization:** Import and initialize the Auditor client with your API key:

```
import { SmartContractAuditor } from "@chaingpt/smartcontractauditor";

const auditor = new SmartContractAuditor({
  apiKey: process.env.CHAINGPT_API_KEY  // or your API key string
});
```

Now `auditor` is an instance you can use to send audit requests.

**Auditing a Contract (via SDK):** You can audit by providing source code or by providing a contract address in the prompt. The SDK provides two main methods: one for streaming results and one for getting the full result at once. Here’s a quick example using the SDK to audit a contract by source code and stream the results:

```
// Example: Audit a simple Solidity contract via streaming
async function runAudit() {
  try {
    const stream = await auditor.auditSmartContractStream({
      question: `Audit the following contract:
      pragma solidity ^0.8.0;
      contract Counter {
        uint256 private count;
        event CountChanged(uint256 newCount);
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
    // Handle the streaming response
    stream.on('data', chunk => {
      const textChunk = chunk.toString();
      console.log(textChunk);  // print each chunk of the report as it arrives
    });
    stream.on('end', () => {
      console.log("Audit stream completed.");
    });
  } catch (err) {
    console.error("Audit failed:", err);
  }
}

runAudit();
```

In this snippet, we call `auditSmartContractStream` with a prompt containing the contract source. The result is a Node.js stream. We attach listeners to output data chunks as they come in, and to note when the stream ends. The final printed output is the complete audit report for the `Counter` contract. (In practice, you might accumulate the chunks into a single string or write them to a file instead of just logging.)

Alternatively, if you prefer to get the entire result without streaming, you can use `auditSmartContractBlob`. This returns a promise for the full response once the audit is done:

```
// Example: Audit by contract address via blob (non-streaming)
async function auditByAddress() {
  try {
    const response = await auditor.auditSmartContractBlob({
      question: "Audit the smart contract at 0x1234567890ABCDEF1234567890ABCDEF12345678 on BNB Chain.",
      chatHistory: "off"
    });
    const reportText = response.data.bot;
    console.log("Audit Report:\n", reportText);
  } catch (err) {
    console.error("Error during audit:", err);
  }
}

auditByAddress();
```

Here we asked the auditor to analyze a deployed contract by address (on BNB Chain). The `auditSmartContractBlob` call waits for the audit to finish and returns a response object. The actual audit text can be obtained as `response.data.bot` – the SDK has packaged the result for us. We then log the report. In a real integration, you might parse this text or forward it to your app’s UI.

{% hint style="info" %}
**Notes:** In the SDK usage, the `model: "smart_contract_auditor"` parameter is set internally, so you only need to provide the prompt (`question`). The `chatHistory` option works the same as in the API – typically keep it `"off"` for one-off queries. If `"on"`, the SDK will retain conversation context (and you can later retrieve the history with the SDK as shown below). Ensure your `apiKey` is correct; if not, the SDK will throw an authorization error when you call the methods.
{% endhint %}

***

### Best Practices for Auditing

* **Provide Complete Context:** For accurate results, give the auditor the full contract code (including all relevant pieces). If your contract spans multiple files or uses imports, consolidate the code in the prompt, or provide an address where the full verified source can be fetched. The AI analysis is only as good as the input it sees.
* **Use Clear Prompts:** Always explicitly state what you want audited. For source code, prefix with a clear instruction like _"Audit the following contract:"_. For addresses, specify the network (e.g., Ethereum, BSC, Polygon) so the system knows where to fetch the code. Clarity helps the AI focus and avoids confusion (for example, if you only provide an address without context, it might not know which chain to use).
* **Handle Large Contracts Appropriately:** If auditing a very large smart contract (or multiple contracts at once), consider that the request size has limits. You may need to audit parts separately if you hit the API’s maximum input size. In most cases, typical contracts fit within limits, but be mindful of extremely large codebases.
* **Inspect and Store Results:** The Auditor’s output is rich in detail. It’s a good practice to save the audit report (e.g., in a file or database) for later reference, especially if you plan to fix issues and re-audit. You can programmatically scan the text for keywords like "Critical" or "High" to quickly flag serious issues in your pipeline. However, always review the full report to understand the context and recommendations.
* **Security of API Key:** As with any secret, never hardcode your API key in client-side code or public repos. In production, load it from a secure config or environment variable. If you suspect your key is compromised, revoke it in the ChainGPT dashboard and generate a new one.
* **Error Handling:** Anticipate possible errors. For instance, an invalid API key or insufficient credits will result in an error response (HTTP 401 Unauthorized or a message about credits). Network issues or server errors might throw exceptions (the SDK throws a `SmartContractAuditorError` for any 4xx/5xx API response). Wrap API calls in try/catch and handle errors gracefully (e.g., log them, alert the user, or retry if appropriate).

By following these practices, you can seamlessly integrate the ChainGPT Auditor and get the most out of its capabilities in your development cycle.
