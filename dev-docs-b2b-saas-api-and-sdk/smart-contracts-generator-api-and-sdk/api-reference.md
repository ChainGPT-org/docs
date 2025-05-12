# API Reference

## ChainGPT Smart Contract Generator API QuickStart Guide

### Introduction

ChainGPT's **Smart Contract Generator API** is an AI-powered service that generates secure Solidity smart contract code from natural language prompts. This API allows you to integrate on-demand smart contract creation into your applications with simple HTTP requests. It's built on ChainGPT’s specialized Solidity-trained LLM, so you can get production-ready smart contract code without writing it manually.

***

### Prerequisites & Access

Before using the Smart Contract Generator API, make sure you have the following:

* **ChainGPT Account & Credits:** A ChainGPT account with sufficient credits. Each API call consumes credits (see **Credit Usage** below). You can purchase or earn credits by logging into the ChainGPT app.
* **API Key:** A secret API key obtained from the ChainGPT web app’s API Dashboard. Log in to the ChainGPT app, navigate to the **API Dashboard**, and use **“Create Secret Key”** to generate a new key. Copy the key and store it securely. **⚠️ Important:** Treat your API key like a password – do not expose it in client-side code or public repositories.
* **Development Tool:** An environment for making HTTP requests (e.g. a Node.js runtime, cURL, Postman, or similar). No SDK installation is required to use the API directly.

***

### Authentication

All requests to the API must include your API key for authentication. Provide your key in the Authorization header as a Bearer token. For example:

```
Authorization: Bearer YOUR_API_KEY
```

Ensure you use HTTPS for all requests (the endpoint is HTTPS by default) to keep your key and data secure. When sending JSON in the request body, include the header `Content-Type: application/json`.

***

### Base URL

The base URL for the Smart Contract Generator API is:

```
https://api.chaingpt.org/chat/stream
```

Use the `POST /chat/stream` endpoint to generate a smart contract from a prompt. (There is also a GET endpoint for retrieving chat history, which is covered later, but for contract generation you will use the `/chat/stream` POST endpoint.)

***

### Example Request (Streaming)

Below is an example of calling the API in Node.js using **Axios** to get a streaming response. This example prompt asks for a simple token contract, and we stream the Solidity code as it's generated:

```javascript
import axios from "axios";

const API_URL = "https://api.chaingpt.org/chat/stream";
const API_KEY = "YOUR_API_KEY";  // ideally load from an environment variable

try {
  const response = await axios.post(API_URL, {
    model: "smart_contract_generator",
    question: "Write a Solidity smart contract for a token with a name, symbol, and owner-only mint function.",
    chatHistory: "off"
  }, {
    headers: { Authorization: `Bearer ${API_KEY}` },
    responseType: "stream"
  });

  // Stream the response in chunks as they arrive
  response.data.on("data", (chunk) => {
    console.log(chunk.toString());  // output partial contract code chunk by chunk
  });
} catch (error) {
  console.error("API call error:", error.response ? error.response.data : error.message);
}
```

In this snippet, we send a POST request to the API with our prompt. We set `responseType: "stream"` so that Axios treats the HTTP response as a stream. As the ChainGPT service generates the contract, it will send chunks of the response which we print out immediately. This way, you can start receiving parts of the Solidity code in real-time without waiting for the entire generation to finish. By the end of the stream, you will have the full smart contract code printed to the console.

***

### Request Parameters

When calling the **`POST /chat/stream`** endpoint, include the following JSON fields in the request body:

#### Required Parameters

* **model** (string, **required**): The model name to use. For the Smart Contract Generator API, this **must** be `"smart_contract_generator"` to route your request to the correct AI model.
* **question** (string, **required**): The prompt or instruction describing the smart contract you want. This can be a simple instruction or a detailed specification (e.g. _"Create an ERC-20 token contract with a burn function"_).

#### Optional Parameters

* **chatHistory** (string, optional): `"on"` or `"off"`. Enable this to `"on"` if you want the AI to remember previous prompts and responses (conversation mode). Defaults to `"off"` if not provided. When enabled, the model will consider past Q\&A in this session, allowing iterative refinement of the contract. **Note:** Using chat history will incur an extra credit cost (see **Credit Usage**).
* **sdkUniqueId** (string, optional): A unique identifier for the user session or conversation. This is used to separate chat histories for different sessions or end-users when `chatHistory` is `"on"`. It can be any string (e.g. a user ID or session ID). If not provided, all prompts with history `"on"` under the same API key share the same context. (Despite the name, you can use `sdkUniqueId` in direct API calls as well — it’s essentially a **session ID** for chat context.)

***

### Response Format

On a successful request, the API will return a **200 OK** response containing the generated contract. The response comes in JSON format by default. The important part of the response is the **Solidity code** generated by the AI, which will be included as a string within the JSON structure.

For example, after a complete response, you might receive JSON like this:

```json
{
  "status": "success",
  "data": {
    "user": "Create an ERC-20 token contract with a burn function.",
    "bot": "pragma solidity ^0.8.0;\n\ncontract MyToken {\n    string public name = \"MyToken\";\n    string public symbol = \"MTK\";\n    uint8 public decimals = 18;\n    uint256 public totalSupply;\n    address public owner;\n\n    mapping(address => uint256) balances;\n\n    constructor() {\n        owner = msg.sender;\n    }\n\n    function mint(address to, uint256 amount) public {\n        require(msg.sender == owner, \"Only owner can mint\");\n        totalSupply += amount;\n        balances[to] += amount;\n    }\n\n    function burn(uint256 amount) public {\n        require(balances[msg.sender] >= amount, \"Insufficient balance\");\n        totalSupply -= amount;\n        balances[msg.sender] -= amount;\n    }\n\n    function balanceOf(address account) public view returns (uint256) {\n        return balances[account];\n    }\n}\n"
  }
}
```

Here, the `"bot"` field (inside the `"data"` object) contains the Solidity smart contract code as a single string, complete with newline characters (). The `"user"` field may echo your prompt, and `"status"` indicates success. In your integration, you'll primarily extract the contract code (the content of `"bot"`).

{% hint style="info" %}
**Note:** If you enabled chat history (`"chatHistory": "on"`), the returned content will still just be the current answer (the generated contract for your prompt). The service will have internally logged the Q\&A into the conversation history for future calls, but the response JSON for the generation call remains focused on the prompt and answer at hand.
{% endhint %}

***

### Streaming vs Full Response Modes

By default, the `/chat/stream` endpoint streams its output in chunks, which is great for real-time processing or displaying progress. However, you can also receive the entire response in one go (what we refer to as a "blob" or full response) if you prefer to wait for completion before using the result.

* **Streaming Mode:** (as shown in the example above) – You set your HTTP client to stream the response. You will receive pieces of the contract as they are generated. This is useful for large contracts or giving the user immediate feedback. In Axios (Node.js), this is done by setting `responseType: "stream"` and handling the `data` events on the response. In other environments or languages, you might use streaming APIs or Server-Sent Events to achieve similar behavior.
*   **Full Response Mode (Blob):** If you do **not** enable streaming on the client side, the API will still work – your HTTP client will simply wait until the generation is finished and then give you the complete response. For example, using Axios without the stream option:

    ```javascript
    // Without streaming: get the full response at once
    const res = await axios.post(API_URL, {
      model: "smart_contract_generator",
      question: "Write a Solidity contract for a basic NFT with a mint function.",
      chatHistory: "off"
    }, {
      headers: { Authorization: `Bearer ${API_KEY}` }
      // no responseType: "stream" here
    });
    // Once the request completes, the entire response is available:
    console.log(res.data);  // the complete Solidity contract code
    ```

    In this mode, the call will return only after the AI has finished generating the contract. The `res.data` will contain the full JSON response, and `res.data` holds the complete contract code. This approach is simpler but has a bit more latency (you wait for the whole answer). It can be suitable for server-side operations where streaming isn't necessary.

Choose the mode that fits your use case: **streaming** for responsiveness or progress updates, vs **full response** for simplicity.

**Get Chat history:**

Smart Contract Generator SDK also stores chat history for future use, which users can retrieve using the following code

```javascript
async function fetchData() {
  try {
    const response = await apiclient.get("/", {
      limit: 10,
      offset: 0,
      sortBy: "createdAt",
      sortOrder: "desc",
    });

    console.log("Response data:", response.data.data.rows);
  } catch (error) {
    console.error("Error fetching data:", error);
  }
}

// Execute the function to fetch data
fetchData();

```

***

### Credit Usage

The Smart Contract Generator API uses a credit-based model for billing:

* **1 Credit per request:** Each call to generate a contract (each `POST /chat/stream` request) deducts **1 credit** from your ChainGPT account.
* **+1 Credit with chat history:** If you enable `chatHistory: "on"` for a request, an **additional 1 credit** is deducted (so a total of 2 credits for that call). This covers the extra resources needed to maintain and utilize the conversation context.
* **Chat history retrieval:** (Advanced) If you use the separate history endpoint to fetch past conversations (`GET /chat/chatHistory`), retrieving chat history does **not** consume extra credits (it was already charged when you enabled `chatHistory` during generation).

Make sure your account has enough credits before making requests, otherwise the API will refuse calls due to insufficient balance. You can check and top up your credits on the ChainGPT web app. _(Credit pricing is designed to be low per call — usually on the order of cents per contract — but refer to the official pricing page for the latest details.)_

***

### Security & Rate Limits

When integrating the API, keep these security and usage limits in mind:

* **API Key Security:** Always keep your API key secret. **Do not** embed it in client-side applications (browser or mobile code) where it could be exposed. Instead, make API calls from a secure backend server or other secure environment. If you suspect your key is compromised, revoke it in the dashboard and generate a new one.
* **HTTPS Only:** Always use the HTTPS endpoint (`https://api.chaingpt.org`) for all requests. This ensures all data (your prompts and the generated code) is encrypted in transit.
* **Rate Limits:** Each API key is limited to **200 requests per minute**. This generous limit prevents abuse and ensures stability. If you exceed this rate, the API will return HTTP 429 "Too Many Requests" errors. Plan your usage accordingly (e.g., queue or throttle calls if needed). If you consistently need a higher rate, consider using multiple API keys or contact ChainGPT support for enterprise options.
* **Input Validation:** The API will attempt to handle a wide range of prompts, but it's good practice to sanitize or validate any untrusted user input you send as a prompt, especially if you might use the output in sensitive contexts.

By following these guidelines, you ensure secure and efficient usage of the Smart Contract Generator API.

***

With the above, you should be able to quickly start integrating smart contract generation into your app!
