# API Reference

## ChainGPT Smart Contract Auditor API Reference

### Introduction

The **ChainGPT Smart Contract Auditor API** allows developers to integrate automated smart contract security auditing into their applications via simple HTTP requests. This API provides direct access to ChainGPT’s smart contract auditing AI, enabling you to submit Solidity code and receive an AI-generated audit report. By using HTTP endpoints (REST API), you can leverage the auditor’s capabilities **without installing any SDK** or additional tools.

**Key features:**

* **Smart Contract Analysis:** Identify vulnerabilities, bugs, and security issues in Ethereum smart contract code by sending the code to the API.
* **Easy Integration:** Use standard HTTP POST/GET requests to interact with the API from any environment (backend server, command-line, etc.).
* **Streaming Responses:** Receive audit results in real-time as they are generated, allowing for interactive or incremental display in your application.
* **Chat History (Optional):** Enable conversation memory to let the AI remember previous audit queries and results, and retrieve those past interactions via an endpoint.
* **Secure and Controlled Access:** Authentication is done with API keys, and usage is metered with credit-based billing and rate limits for fair use (details below).

**Prerequisites:** Before using the API, ensure you have the following:

* A ChainGPT account with access to the Smart Contract Auditor and an active API key (see **Authentication** below).
* Sufficient **credits** in your ChainGPT account (purchased via the ChainGPT web app) to cover API usage.
* An environment for making HTTP requests (e.g. Node.js, Python, Postman, or curl in a terminal) and internet connectivity.

***

### Authentication & API Access

All requests to the Smart Contract Auditor API must include a valid API key in the request headers. Your API key authenticates your requests and tracks your usage (for credits and rate limiting).

**Getting an API Key:**

1. Log in to the ChainGPT web app at [**app.chaingpt.org**](https://app.chaingpt.org) with your account.
2. Navigate to the **API Dashboard** or **API Keys** section.
3. Use the “Create Secret Key” feature to generate a new API key.
4. Copy the API key and **store it securely**. (Once generated, you won’t be able to view the full key again for security reasons.)

{% hint style="info" %}
**Important:** Treat your API key like a password. **Do not expose or hard-code your API key** in public client-side code or repositories. Store it in a secure location (e.g. an environment variable or a secrets manager) and load it in your app backend or secure environment.
{% endhint %}

**Using the API Key:**\
Include the API key in the Authorization header for every API request. Use the Bearer authentication scheme:

```
Authorization: Bearer YOUR_API_KEY
```

For example, if your API key is `abc123...`, set the header as:\
`Authorization: Bearer abc123...`

All API endpoints are served over **HTTPS** (secure HTTP). Always ensure you are calling the `https://api.chaingpt.org` endpoints and include the `Authorization` header. The API expects request and response bodies in JSON format, so you should typically set `Content-Type: application/json` for POST requests.

***

### Credits & Usage Limits

ChainGPT uses a credit-based system to bill for API usage. You must have sufficient credits in your account for the Smart Contract Auditor API calls to succeed. Each API call will deduct credits as follows:

* **Audit Request:** Each call to the audit endpoint (each smart contract audit request) costs **1 Credit**.
* **Chat History Surcharge:** If chat history is enabled for an audit request (see the `chatHistory` parameter), an **additional 1 Credit** is deducted for that request (total 2 credits for that call). This covers the storage and management of the conversation history.
* **Chat History Retrieval:** Retrieving stored chat history via the history endpoint does **not** deduct additional credits, provided that those interactions were already recorded (you paid for them when `chatHistory` was "on" in the audit requests).

Credits will be automatically debited from your ChainGPT account **per request**. If your account’s credit balance is insufficient, audit requests will fail (the API will return an error indicating insufficient credits). Always ensure your account has enough credits to avoid interruptions.

{% hint style="info" %}
**Note:** Credit costs and policies may be updated over time. The above values (1 credit per audit, +1 with history) are current as of writing. Always refer to the ChainGPT web app or pricing documentation for the latest credit pricing and usage details.
{% endhint %}

**Rate Limits:** To prevent abuse, the API imposes a rate limit of **200 requests per minute** per API key. If you exceed this rate, you will receive a **429 Too Many Requests** error. Design your integration to stay within this limit (e.g., by queuing or spacing out calls if necessary). If you consistently need a higher rate limit, consider contacting ChainGPT support for options.

***

### Audit Request – _Smart Contract Audit Endpoint_

This endpoint accepts a smart contract auditing request and returns the audit result. Under the hood, it submits your prompt (typically the contract code and any specific question) to ChainGPT’s AI and streams back the response (the audit analysis).

**Endpoint:** `POST https://api.chaingpt.org/chat/stream`\
**Description:** Submits a smart contract code or audit query to the ChainGPT auditor and returns the AI’s response as a stream of data (allowing you to read the answer progressively).

#### Request

* **Method:** `POST`
* **URL:** `https://api.chaingpt.org/chat/stream`
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
  * `Content-Type: application/json` (required for the JSON body)
* **Body:** JSON object with the following parameters:

<table><thead><tr><th width="124.52734375">Parameter</th><th width="91.5859375">Type</th><th width="88.5546875">Required</th><th>Description</th></tr></thead><tbody><tr><td><strong>model</strong></td><td>string</td><td><strong>Yes</strong></td><td>The ID of the AI model to use. For the Smart Contract Auditor API, this <strong>must</strong> be set to <code>"smart_contract_auditor"</code>.</td></tr><tr><td><strong>question</strong></td><td>string</td><td><strong>Yes</strong></td><td>The user’s prompt or query. In practice, this is where you provide the smart contract code and any specific instructions or questions for the auditor. You can ask the AI to audit a piece of code by including the Solidity code in this string. <em>(There is no separate field for code; include the code as part of the question text.)</em></td></tr><tr><td><strong>chatHistory</strong></td><td>string (<code>"on"</code> or <code>"off"</code>)</td><td>No (default <code>"off"</code>)</td><td>Whether to enable chat history (conversation memory) for this request. If <code>"on"</code>, this request and its response will be stored, and the AI will remember this conversation context in future requests. If <code>"off"</code>, the AI treats this as a single-turn query (no memory of past questions), and no history is saved. Enabling chat history incurs an extra credit cost (see <strong>Credits</strong> above).</td></tr><tr><td><strong>sdkUniqueId</strong></td><td>string</td><td>No</td><td>A unique identifier for the user or session making the request. This is used in conjunction with <code>chatHistory</code>. If you plan to have multiple separate conversations or multiple end-users using your API key, assign a distinct <code>sdkUniqueId</code> for each user or session. This ensures that when <code>chatHistory</code> is <code>"on"</code>, the stored history is tied to that specific <code>sdkUniqueId</code> and not mixed with other sessions. If omitted, all history-enabled requests under your API key will share a single, global history.</td></tr></tbody></table>

**Notes on `chatHistory` and `sdkUniqueId`:** If you enable chat history, the API will remember the conversation context. To maintain separate histories for different users or threads, use a different `sdkUniqueId` for each. For example, if User A and User B both make audit requests with your API key, give them distinct IDs (`sdkUniqueId: "userA"` vs `sdkUniqueId: "userB"`). The auditor will remember past interactions **per ID**. When you make subsequent calls with `chatHistory: "on"` and the same `sdkUniqueId`, the AI will recall previous questions/answers from that session, providing continuity (useful for follow-up questions or iterative audits).

#### Response

On success, this endpoint responds with an **HTTP 200** status and begins streaming the audit result through the HTTP response body. The response is streamed in chunks (using chunked transfer encoding), allowing the client to start receiving and processing the output before the entire audit report is complete. This is ideal for displaying results in real-time to users.

* **Streaming Format:** Each chunk of data is a portion of the auditor’s answer (usually a piece of text). The chunks, when concatenated in order, form the full audit report. The API does not wrap each chunk in JSON; it streams the raw text of the answer. You should keep the connection open and read continuously until the stream ends (the end of the response signals the completion of the audit report).
*   **Final Output:** If you capture or recombine the entire streamed response, the full message typically can be represented as a JSON object with a structure like:

    ```
    {
      "status": true,
      "message": "Chat response generated successfully.",
      "data": {
        "bot": "<complete audit report text>"
      }
    }
    ```

    In streaming mode, this JSON is effectively sent as a continuous text stream (without needing a closing `]` or `}` until the end). The main content of interest is the **`data.bot`** field, which contains the auditor’s answer (the audit findings and explanations). The `status` and `message` provide a high-level confirmation. You may parse the final aggregated text as JSON if needed, or simply use the raw text of the `bot` response.
* **Error Handling:** If your request is unauthorized (e.g., missing or invalid API key), the API will immediately return an HTTP 401 Unauthorized error. If you exceed rate limits, a 429 error will be returned. Insufficient credits or other invalid request issues will result in an error response (often with an error message in JSON). In streaming mode, an error usually causes the connection to close early, possibly with an error JSON before closing. Always implement error handling: check HTTP status codes and be prepared to handle a terminated stream or error object.

#### Example: Audit Request (Streaming)

Below is an example of how to call the audit endpoint using **Node.js (JavaScript)** with the Axios HTTP client, and how to handle the streaming response. In this example, we ask the AI to audit a simple Solidity contract:

```
import axios from 'axios';

// Set up the API endpoint and API key (loaded from environment for security)
const API_URL = 'https://api.chaingpt.org/chat/stream';
const API_KEY = process.env.CHAINGPT_API_KEY;  // ensure you've stored your key in an env variable

// Create an Axios instance with the authorization header and stream response enabled
const apiClient = axios.create({
  baseURL: API_URL,
  timeout: 60000,  // 60 seconds timeout
  headers: { Authorization: `Bearer ${API_KEY}` },
  responseType: 'stream'   // important for streaming responses
});

async function auditContract() {
  try {
    // Make the POST request to the audit endpoint
    const response = await apiClient.post('/', {
      model: "smart_contract_auditor",
      question: `Audit the following smart contract:
      \`\`\`
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
      }
      \`\`\`
      Identify any security issues or improvements.`,
      chatHistory: "on",
      sdkUniqueId: "user12345"  // an example unique ID for this session/user
    });
    
    // `response.data` is a Readable stream since we set responseType: 'stream'
    const stream = response.data;
    stream.on('data', (chunk) => {
      // Process each chunk of the response as it arrives
      const textChunk = chunk.toString();
      process.stdout.write(textChunk);  // here we simply print it out (could accumulate if needed)
    });
    stream.on('end', () => {
      console.log("\n[Audit complete]");
    });
  } catch (error) {
    console.error("Error during audit request:", error.response?.data || error.message);
  }
}

auditContract();
```

In the above example, the contract code is embedded in the `question` string (using a template literal for readability). The response stream is consumed chunk by chunk, and written to standard output. In a real application, you might accumulate the chunks and display them in a UI as they arrive. The `chatHistory: "on"` parameter means this Q/A will be stored, and since we provided `sdkUniqueId: "user12345"`, the context is scoped to that ID for future calls.

**Example (cURL):** Alternatively, you can test the endpoint with a curl command. This will print the response to your terminal as it streams:

```
curl -X POST "https://api.chaingpt.org/chat/stream" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "smart_contract_auditor",
    "question": "Please audit the following Solidity code...\n<YOUR CONTRACT CODE HERE>",
    "chatHistory": "off"
  }'
```

_(Replace `YOUR_API_KEY` and the `question` content with your actual API key and contract/prompt. The `chatHistory` is set to "off" in this curl example for simplicity.)_

***

### Chat History – _Retrieve Past Interactions_

If you have enabled `chatHistory` for your audit requests, the API stores those conversations (prompts and responses). The **Chat History endpoint** allows you to retrieve a log of past audit queries and their results associated with your API key (and optionally filtered by a specific `sdkUniqueId` session).

**Endpoint:** `GET https://api.chaingpt.org/chat/chatHistory`\
**Description:** Fetches stored chat history (audit Q\&A records) for your account. You can use query parameters to paginate results, sort them, or filter by a specific session ID.

#### Request

* **Method:** `GET`
* **URL:** `https://api.chaingpt.org/chat/chatHistory`
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
* **Query Parameters:** (passed as URL query string or using your HTTP client’s query params support)

<table><thead><tr><th width="121.16015625">Parameter</th><th width="89.65234375">Type</th><th width="87.8203125">Required</th><th>Description</th></tr></thead><tbody><tr><td><strong>limit</strong></td><td>integer</td><td>No</td><td>The number of history records to retrieve in this request (page size). If not provided, a default (e.g. 10) is used. Example: <code>limit=10</code> will return up to 10 audit entries.</td></tr><tr><td><strong>offset</strong></td><td>integer</td><td>No</td><td>The number of records to skip (for pagination). For example, <code>offset=0</code> starts from the first record (usually the most recent if using default sort), <code>offset=10</code> would skip the first 10 records. Default is 0.</td></tr><tr><td><strong>sortBy</strong></td><td>string</td><td>No</td><td>Field to sort by. Default is <code>"createdAt"</code> (the timestamp of the interaction). Currently, <code>"createdAt"</code> is the primary field available for sorting.</td></tr><tr><td><strong>sortOrder</strong></td><td>string</td><td>No</td><td>Sort direction: <code>"desc"</code> for descending (newest first) or <code>"asc"</code> for ascending (oldest first). Default is <code>"desc"</code>.</td></tr><tr><td><strong>sdkUniqueId</strong></td><td>string</td><td>No</td><td>If provided, only return history entries that were associated with this <code>sdkUniqueId</code>. Use this to fetch the conversation history for a specific user or session (if you have multiple under one API key). If omitted, the response will include all stored interactions for your API key (across all sessions).</td></tr></tbody></table>

You can combine these parameters as needed. For example, you might call `GET /chat/chatHistory?limit=5&offset=0&sortOrder=asc&sdkUniqueId=user12345` to get the oldest 5 records for session “user12345”. If you don’t need filtering or pagination, you can just call `/chat/chatHistory` with no params (defaults will apply).

#### Response

On success, the server will return **HTTP 200** and a JSON object containing the history records. The response structure is:

```
{
  "status": true,
  "message": "Chat history retrieved successfully.",
  "data": {
    "rows": [
      {
        "id": "<record-id-1>",
        "question": "<the audit question from your request>",
        "bot": "<the auditor's response>",
        "createdAt": "2025-04-13T21:48:00.123Z",
        "sdkUniqueId": "<session-id-if-used>"
      },
      {
        "id": "<record-id-2>",
        "question": "<next audit question>",
        "bot": "<auditor's answer>",
        "createdAt": "2025-04-13T21:49:30.456Z",
        "sdkUniqueId": "<session-id-if-used>"
      }
      // ... more records up to the specified limit
    ],
    "count": 2
  }
}
```

* **`data.rows`**: an array of history entries. Each entry represents one audit Q\&A interaction. Fields include:
  * `id`: an internal identifier for the record.
  * `question`: the exact question/prompt that was sent (e.g. the contract code and query).
  * `bot`: the auditor’s full response to that question.
  * `createdAt`: timestamp when the interaction occurred (ISO 8601 format).
  * `sdkUniqueId`: the session identifier if one was associated with this interaction (absent or null if none was provided).
* **`data.count`**: the number of records returned in this response (e.g. for pagination tracking).
* **`status` / `message`**: indicate the success of the request and a human-readable message.

Using this data, you can display a history of audits to your users or utilize past answers. For example, you might show users a list of their previous contract audits or allow them to review an audit report again.

**Note:** Retrieving chat history does **not** consume credits. You are effectively accessing records you’ve already paid for when the history was created. However, you must have had `chatHistory: "on"` in the original requests for any records to exist (if history was off, that interaction is not stored).

#### Example: Retrieve Chat History

Here’s an example using Axios (Node.js) to fetch the 10 most recent audit interactions:

```
import axios from 'axios';

const HISTORY_URL = 'https://api.chaingpt.org/chat/chatHistory';
const API_KEY = process.env.CHAINGPT_API_KEY;

async function fetchHistory() {
  try {
    const response = await axios.get(HISTORY_URL, {
      headers: { Authorization: `Bearer ${API_KEY}` },
      params: {
        limit: 10,
        offset: 0,
        sortBy: "createdAt",
        sortOrder: "desc"
        // sdkUniqueId: "user12345"  // optionally filter by a specific session
      }
    });
    // The history records are in response.data.data.rows
    console.log("Retrieved records:", response.data.data.rows);
  } catch (error) {
    console.error("Error fetching history:", error.response?.data || error.message);
  }
}

fetchHistory();
```

This will output an array of up to 10 recent audit sessions (including the questions and answers). You can adjust the parameters as needed or add `sdkUniqueId` to filter for a specific user’s history.

**Example (cURL):** Fetch 5 most recent history records using curl:

```
curl -G "https://api.chaingpt.org/chat/chatHistory" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  --data-urlencode "limit=5" \
  --data-urlencode "offset=0" \
  --data-urlencode "sortOrder=desc"
```

(Add `--data-urlencode "sdkUniqueId=yourUserId"` if you want to filter by a session ID.)

***

### Security Considerations

Integrating the ChainGPT Smart Contract Auditor API requires attention to security and best practices:

* **API Key Security:** Keep your API key secret at all times. Do not embed it in client-side code or anywhere it could be exposed publicly. If you suspect your API key is compromised, revoke it in the ChainGPT dashboard and generate a new one. Only send the API key over HTTPS (which the `api.chaingpt.org` endpoints use by default).
* **Authentication Required:** All endpoints demand a valid API key. Without authentication, or with an incorrect key, requests will be rejected (HTTP 401). There is no "public" or unauthorized access mode.
* **Credits and Authorization:** The API will only process requests if your account has sufficient credits. This acts as a security measure to prevent abuse and ensure only authorized (paying) users can use the resource-intensive audit service. Always monitor your credit balance, especially in production environments, to avoid disruptions.
* **Rate Limiting:** As noted, the API enforces a limit of 200 requests per minute per API key. This protects the service from abuse and ensures fair usage. If you hit this limit, back off and retry after a short delay. Do not try to work around the rate limit (e.g., by creating multiple keys for the same app) as this may violate the terms of service and result in suspension.
* **Data Privacy:** The code you send and the audit results are processed by ChainGPT’s AI. ChainGPT does not share your code or responses with unauthorized parties, and your data is used only to provide the service. However, avoid sending any sensitive personal data in the prompts. The focus should be on code analysis. (For more details on data handling, refer to ChainGPT’s privacy policy.)
* **Network Security:** Always use HTTPS to interact with the API. The endpoints provided (`api.chaingpt.org`) are TLS/SSL encrypted. This ensures your API key and data are not exposed in transit.

By following these practices, you help maintain the security of your application and safeguard your access to the ChainGPT API.

***

### Support and Resources

If you need help integrating or using the Smart Contract Auditor API, or you encounter issues not covered in this documentation, please refer to ChainGPT’s support channels and resources:

* **Documentation:** Visit the official ChainGPT docs site for more guides, examples, and concept explanations for the AI tools.
* **Support Channels:** You can reach out through the channels listed on the [ChainGPT website](https://www.chaingpt.org/) – such as community Telegram/Discord, or official support email – for assistance.
* **Web App Dashboard:** The ChainGPT web app’s API Dashboard may provide usage logs, error details, or additional settings that can help debug issues.

For any further assistance, explore the support section on our website or contact the ChainGPT team. We’re here to help ensure your integration is successful!
