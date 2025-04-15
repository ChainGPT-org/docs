# API Reference

## API Reference

This section documents the REST API for ChainGPT’s Smart-Contracts Generator. If you’re integrating via HTTP calls (without the SDK), this is your go-to reference. The API allows you to generate contracts and manage chat history through simple HTTP endpoints.

All API endpoints are served over HTTPS and require authentication via your API key. The base path for the Smart-Contracts Generator endpoints is **`https://api.chaingpt.org/chat`**.

***

### Authentication & Base URL

**Base URL:** `https://api.chaingpt.org/chat`

All requests should be made to this base URL with the appropriate path and HTTP method as documented below.

**Authentication:** The API uses API keys for auth. Include your API key in the `Authorization` header for every request, using the Bearer token scheme. For example:

```
Authorization: Bearer YOUR_API_KEY_HERE
```

If the API key is missing or invalid, the API will respond with an HTTP 401 Unauthorized error. Always secure your API key; do not expose it in client-side code. (For testing, you can use it in a tool like Postman under the Authorization header.)

**Content Type:** The API expects and returns JSON. Make sure to send `Content-Type: application/json` in your requests when appropriate. Responses will typically be JSON as well, except when streaming data.

***

### Endpoints Overview

The Smart-Contracts Generator API currently offers two main endpoints:

1. **Generate Smart Contract (Chat Completion)** – **`POST /chat/stream`** – This is the primary endpoint to generate a smart contract based on a prompt. It returns a streamed response (chunks of data) by default, which can be handled as server-sent events or a streaming HTTP response.
2. **Retrieve Chat History** – **`GET /chat/chatHistory`** – This endpoint allows you to fetch past prompt/response pairs (chat history) associated with your API key (and optionally a specific `sdkUniqueId` if used). Use this to retrieve conversations if you have been using the `chatHistory` feature.

Below we detail each endpoint, including required parameters and example usage.

#### 1. Generate Smart Contract – `POST /chat/stream`

Use this endpoint to generate a new smart contract given a user prompt/question. The API will process the request and return the AI-generated answer which includes the Solidity contract code (and possibly explanatory text). This response is returned as a stream, meaning the data comes in chunks that you can read progressively. You can also treat it as a single response by reading to the end, depending on your HTTP client’s capabilities.

* **URL:** `https://api.chaingpt.org/chat/stream`
* **Method:** `POST`
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
  * `Content-Type: application/json` (required when sending JSON body)

**Request Body Parameters**

All parameters should be sent in the JSON body of the POST request:

* **`model`** (string, required): The model name to use. For the Smart-Contracts Generator, this **must** be `"smart_contract_generator"`. This ensures your request is routed to the correct AI model.
* **`question`** (string, required): The prompt or instruction describing what smart contract you want. This can be a single sentence or a detailed description. E.g., `"Create an ERC-20 token contract with a burn function"`.
* **`chatHistory`** (string, optional): Set to `"on"` to enable conversation mode, or `"off"` for a single-turn response. If `"on"`, the AI will consider previous prompts and responses in the session (see `sdkUniqueId`). Defaults to `"off"` if not provided.
* **`sdkUniqueId`** (string, optional): A unique identifier for the conversation or end-user. Use this if you want to maintain separate chat histories for different users or sessions. It can be any string (like a UUID, username, or session ID). If not provided, the history may be shared across requests made with the same API key (if `chatHistory` is on). It’s good practice to provide this in multi-user scenarios.

_(Note: The term **`sdkUniqueId`** is named for consistency with the SDK parameter, but it equally applies to API usage. It simply tags the request with a particular session ID for history tracking.)_

**Response Structure**

If the request is successful, the HTTP status will be **200 OK**. The response will be streamed in chunks. If you are using a tool or HTTP client that automatically handles streaming, you will get the data as it arrives. Each chunk is typically a piece of the answer text (part of the Solidity code or explanation). Once the stream is complete, you’ll have the full response.

For example, if you collect all the chunks, you might end up with a JSON object like:

```
{
  "status": "success",
  "data": {
    "user": "Create an ERC-20 token contract with a burn function",
    "bot": "pragma solidity ^0.8.0;\n\ncontract MyToken {\n    string public name = \"MyToken\";\n    string public symbol = \"MTK\";\n    uint8 public decimals = 18;\n    uint256 public totalSupply;\n    address public owner;\n\n    mapping(address => uint256) balances;\n\n    constructor() {\n        owner = msg.sender;\n    }\n\n    function mint(address to, uint256 amount) public {\n        require(msg.sender == owner, \"Only owner can mint\");\n        totalSupply += amount;\n        balances[to] += amount;\n    }\n\n    function burn(uint256 amount) public {\n        require(balances[msg.sender] >= amount, \"Insufficient balance\");\n        totalSupply -= amount;\n        balances[msg.sender] -= amount;\n    }\n\n    function balanceOf(address account) public view returns (uint256) {\n        return balances[account];\n    }\n}\n"
  }
}
```

Here, the `"bot"` field contains the Solidity code as a string (with newline characters, etc.), and `"user"` might echo your prompt. The exact format could vary, but the **core content – the contract code – will be present** (likely under a field called `"bot"` or within `"data"`).

If `chatHistory: "on"` was used, the service has internally logged this Q\&A into the chat history for future reference. The response content itself will still just be the answer to the prompt.

**Example Request (cURL)**

To illustrate, here’s a cURL command example (assuming you have stored your API key in `$API_KEY` environment variable):

```
curl -X POST "https://api.chaingpt.org/chat/stream" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "smart_contract_generator",
    "question": "Create an ERC-721 NFT smart contract with a base URI and a mint function.",
    "chatHistory": "off"
  }'
```

This will initiate a request to generate an NFT contract. Because the endpoint is a streaming one, you will see partial results coming in. If you want to force cURL to wait and show the full response, you might need to tweak settings or use a different approach (since cURL by default will stream chunked response as it comes). Typically, when integrating programmatically, you will use a streaming HTTP client or simply handle the final assembled response.

**Error Responses**

If something is wrong with the request or environment, you may get an error response:

* **401 Unauthorized:** API key missing or invalid. Double-check your `Authorization` header.
* **400 Bad Request:** Missing required fields or invalid values in the JSON (e.g., forgetting to include `model` or `question`). The response body will likely contain a message explaining the issue.
* **402 Payment Required / 403 Forbidden:** If your account has no credits, the API may respond with a status indicating you cannot proceed (the exact status might be 402 or a 4xx code with an error message like “Insufficient credits”). Top-up your credits and try again.
* **429 Too Many Requests:** You’ve hit the rate limit (more than 200 requests per minute). You should wait a bit and ensure you throttle your requests. The response may include a Retry-After header.
* **500 Internal Server Error or 503 Service Unavailable:** General server error or maintenance. Rare, but could happen if the service is down or undergoing upgrades. If this occurs, it’s usually transient – you can try again after some time.

The response body for errors will typically contain a JSON with an `"error"` or `"message"` field describing what went wrong. For instance, a 400 might return `{"error": "Missing 'question' in request body"}`.

_(Tip: when using the SDK, many of these errors will surface as **`SmartContractGeneratorError`** exceptions with a message, so you can handle them in code rather than manually parsing HTTP responses.)_

#### 2. Retrieve Chat History – `GET /chat/chatHistory`

If you have used `chatHistory: "on"` in your requests and want to retrieve the stored conversation logs, use this endpoint. It returns a list of past prompts and responses associated with your API key (and optionally filtered by `sdkUniqueId` if provided).

* **URL:** `https://api.chaingpt.org/chat/chatHistory`
* **Method:** `GET`
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)

This is a standard GET request. The history is tied to your API key and optionally a session ID.

**Query Parameters**

You can provide the following query parameters in the URL to control what portion of history you retrieve:

* **`limit`** (integer, optional): The number of history records to retrieve. Defaults to 10 if not specified. (Each record typically represents one question-answer pair.)
* **`offset`** (integer, optional): The offset for pagination. For example, `offset=0&limit=10` gets the first 10 records; `offset=10&limit=10` would get the next 10, and so forth.
* **`sortBy`** (string, optional): Field by which to sort the history. Likely options are `"createdAt"` (timestamp of the request). Default is `"createdAt"`.
* **`sortOrder`** (string, optional): Sort direction, either `"ASC"` or `"DESC"`. For example, to get the latest entries first, use `sortOrder=DESC`. The default might be descending (newest first).
* **`sdkUniqueId`** (string, optional): If you want to fetch history for a specific user/session (and you have been tagging requests with `sdkUniqueId`), provide the same ID here. This will return only the history for that particular identifier. If not provided, it returns the history associated with the API key in general (or the default session).

**Response Structure**

On success (200 OK), the response will contain a list of history items in JSON format. For example:

```
{
  "status": "success",
  "data": {
    "rows": [
      {
        "id": "12345",
        "question": "Create an ERC-20 token contract with a burn function.",
        "answer": "pragma solidity ... (contract code) ...",
        "createdAt": "2025-03-01T12:34:56Z"
      },
      {
        "id": "12346",
        "question": "Now add a mint function to the contract.",
        "answer": "function mint(...) { ... }",
        "createdAt": "2025-03-01T12:36:10Z"
      }
    ],
    "count": 2
  }
}
```

The exact field names might differ (e.g., the answer might be under `"bot"` rather than `"answer"`), but the concept is that you get an array of past interactions. Each entry could include the prompt, the response, timestamps, and possibly an ID. The `"count"` might indicate total records (useful for pagination).

If no history is found (or `chatHistory` was never used), `"rows"` could be an empty array.

**Example Request (cURL)**

```
curl -X GET "https://api.chaingpt.org/chat/chatHistory?limit=5&offset=0&sortBy=createdAt&sortOrder=DESC" \
  -H "Authorization: Bearer $API_KEY"
```

This fetches the latest 5 history records for your account.

**Error Responses**

Similar to the generate endpoint, you could encounter:

* **401 Unauthorized:** If the API key is missing/invalid.
* **400 Bad Request:** If you provide an invalid query parameter (e.g., non-numeric `limit`).
* **403 Forbidden:** If for some reason history access is not allowed (not typical unless some account issue).
* **429 Too Many Requests:** If you call this too frequently (though the same rate limit of 200/min applies across all endpoints).
* **500/503 Server errors:** if something unexpected goes wrong on the server side.

Usually, retrieving chat history is straightforward and less likely to hit issues if your API key is correct.

_(Note: As mentioned earlier, retrieving chat history does not incur an extra credit cost **if** the history was recorded as part of a generation request with **`chatHistory: "on"`**. In other words, the “cost” of recording history was paid at generation time. Simply fetching existing history is treated as a normal API call in terms of rate limit and does consume a very small amount of your bandwidth, but it should not deduct an extra credit.)_

***

### Rate Limits & Credit Usage

ChainGPT’s API enforces some limits to ensure fair use and system stability. It also operates on a credit system for billing.

* **Rate Limit:** Each API key is allowed up to **200 requests per minute**. This is a generous limit to accommodate high-throughput applications. Exceeding this limit will result in HTTP 429 errors. If you anticipate needing more than this, consider spreading requests across multiple API keys or contact ChainGPT support for higher rate plans.
* **Credit Consumption:** To call the Smart-Contracts Generator API, your account must have credits (each credit typically corresponds to a fixed cost, e.g., a small USD amount). The usage is as follows:
  * **–1 credit per generation request.** Every time you call the `POST /chat/stream` endpoint (via API or SDK), **1 credit** is deducted from your balance.
  * **–Additional 1 credit if chat history is on.** If you set `chatHistory: "on"` for that request, an extra credit is deducted (so 2 credits total for that call). This covers the additional computational cost of maintaining and managing conversation context.
  * **–History retrieval calls consume 1 credit.** Fetching chat history (`GET /chat/chatHistory`) will typically use 1 credit as well (treated like a normal API call). However, if you already paid extra for enabling history, the act of retrieval doesn’t charge extra beyond a standard call. (In practice, assume any API call might deduct 1 credit, unless it’s explicitly part of another operation.)
* **Credit Errors:** If you have **0 credits** and attempt an API call, the request will be refused (likely with an error message about insufficient credits). Always ensure you have a positive credit balance before making calls. You can check your credits on the ChainGPT app dashboard.
* **Credit Replenishment:** Credits can be purchased or obtained through the ChainGPT web app. The pricing is designed to be very low per contract generation (as noted earlier, typically under $1 per call, though exact pricing may vary). This allows for cost-effective development and testing.
* **Monitoring Usage:** It’s a good practice to monitor your usage. Keep track of how many requests you’re making and your remaining credits. If you’re integrating for a business-critical application, consider implementing a warning when credits drop below a certain threshold, so you can top-up and avoid downtime.

{% hint style="info" %}
_💡 **Note:** Credit prices or policies might be updated by ChainGPT over time. Always refer to the official **API Pricing** page or announcements for the latest details. The information above is accurate as of this writing but staying informed ensures you won’t be caught off guard by changes._
{% endhint %}

***

### Security Considerations

When using the Smart-Contracts Generator API, keep the following security best practices in mind:

* **Protect Your API Key:** We cannot stress this enough – do not expose your API key. If you are calling the API from a backend server, that’s ideal (the key remains on your server). If you need to call from a client-side context (like a web app), consider proxying through your own server or using the SDK on a server. Exposing the key in client-side code (browser or mobile app) could lead to misuse by others.
* **Use HTTPS:** All endpoints are HTTPS, ensure you **always** use `https://api.chaingpt.org` (which the provided base URL already does). This encrypts the traffic so your prompts and generated code aren’t intercepted or read in transit.
* **Validate Inputs:** Although the AI will attempt to handle a variety of prompts, always sanitize and validate any user input that you might pass to the API (especially if building an open-ended chatbot or tool for end-users). This is more of a general caution to avoid misuse (like someone trying to get the AI to produce disallowed content via your integration).
* **Check Generated Code:** The AI-generated smart contracts should be reviewed before deployment. While the AI aims to produce secure code, it’s good security hygiene to audit the output. This is outside the API’s direct scope, but from a developer perspective, you might integrate ChainGPT’s **Smart-Contracts Auditor** (if available) to automatically audit the generated code, or manually inspect for any business logic errors or security issues.
* **Privacy:** The prompts you send and the code generated might be stored as part of chat history. If you are dealing with sensitive project ideas, know that this data is going to ChainGPT’s servers. ChainGPT likely uses it to maintain context for chat history and possibly for monitoring or improving the service, under their privacy policy. Avoid sending highly sensitive data in prompts. Focus on generic descriptions (which is usually sufficient for code generation).
* **Abuse Prevention:** The 200 requests/minute limit and credit system act as safeguards against abuse (like someone spamming requests). If you find you need higher limits, it’s possible to discuss with ChainGPT, but be mindful that running a large number of AI generations quickly can be heavy. Ensure your usage is intentional and within acceptable use.

For any security concerns or questions, reach out to ChainGPT’s support. They can provide guidelines if, for example, you want to embed this in a user-facing product securely.
