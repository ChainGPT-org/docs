# API Reference

## Smart-Contracts Auditor - API Reference

The ChainGPT Smart Contract Auditor API allows you to audit contracts via simple HTTP requests. It’s a single endpoint that accepts an audit prompt and returns a comprehensive analysis of the contract. This section details the API usage, endpoints, parameters, and expected responses.

**Base URL:** `https://api.chaingpt.org/chat/stream`\
All requests should be made to this base URL. The API uses HTTPS and expects JSON formatted requests/responses.

**Authentication:** Every API request must include your API key in the **Authorization** header:

```
Authorization: Bearer YOUR_API_KEY
```

You can obtain an API key from your ChainGPT account dashboard. **Keep this key secret.** If the key is missing or invalid, the API will return an HTTP 401 Unauthorized error.

**Credits & Rate Limits:** You must have sufficient credits in your ChainGPT account to use the API. Each audit request consumes **1 credit** (plus an additional 1 credit if you enable chat history for that request). The system enforces a rate limit of **200 requests per minute** per API key. If you exceed this rate, you will receive a 429 Too Many Requests error. Ensure you budget your usage according to your credit balance; if a request is made without enough credits, the API will respond with an error indicating insufficient credits (HTTP 402 or a similar error message).

#### POST `/chat/stream` — Smart Contract Audit

Submit a smart contract audit request. This is the primary endpoint to get an audit report for a given contract (source code or address).

* **URL:** `POST https://api.chaingpt.org/chat/stream`
* **Purpose:** Analyzes the provided smart contract and returns an audit report.
* **Headers:**
  * `Authorization: Bearer <API_KEY>` (required) – Your API key for authentication.
  * `Content-Type: application/json` (required) – The request and response bodies are JSON formatted.
* **Body Parameters:** (JSON object)

<table><thead><tr><th width="132.76953125">Parameter</th><th width="73.76171875">Type</th><th width="87.55859375">Required</th><th>Description</th></tr></thead><tbody><tr><td><code>model</code></td><td>string</td><td><strong>Yes</strong></td><td>Specifies which AI model to use. <strong>Must</strong> be <code>"smart_contract_auditor"</code> for this endpoint.</td></tr><tr><td><code>question</code></td><td>string</td><td><strong>Yes</strong></td><td>The audit prompt. This should include the contract to audit. You can provide the Solidity source code directly here, or a request to fetch an address (e.g., "Audit the contract at 0x... on Polygon"). The prompt may also include any specific questions or focus points for the audit.</td></tr><tr><td><code>chatHistory</code></td><td>string</td><td>No</td><td><code>"off"</code> or <code>"on"</code>. If <code>"on"</code>, the audit will be associated with a chat session, and the AI will remember this exchange for subsequent related queries (allowing follow-up questions). Default is <code>"off"</code>. Enabling this costs an extra credit per request.</td></tr><tr><td><code>sdkUniqueId</code></td><td>string</td><td>No</td><td>A unique identifier for your user or session (if you are managing multiple users). This can be used for tracking purposes. It does not affect the audit content.</td></tr></tbody></table>

*   **Response:** Streamed JSON/ text. The response is sent as an **HTTP chunked stream** (Content-Type: `text/event-stream`). As such, you will receive parts of the audit report as they are generated. If you read the entire response to completion, the final result is a JSON object containing the audit outcome. The most important field in the JSON is typically `bot`, which holds the Auditor’s response (the audit report text). For example, a completed response body (after streaming) might look like:

    ```
    {
      "bot": "### Audit Report\n**Score:** 95%\n**Summary:** The contract is a simple counter with no critical vulnerabilities. \n\n**Issues Found:**\n- Low: The `increment` function emits an event but the event is not defined. This will cause a compile error. Define the event `CountChanged` before using it.\n\n**Recommendations:**\nDefine the missing event or remove the emit line. No other issues found. The contract follows best practices.",
      "user": "Audit the following contract: pragma solidity ^0.8.0; contract Counter { ... }"
    }
    ```

    In the above example, the `bot` field contains a (truncated) audit report in Markdown-like format. It includes a score, summary, and one identified issue with a recommendation. The `user` field echoes the prompt that was sent (in this case the source code). The exact structure may vary, but you can reliably extract the audit text from `bot`. If no vulnerabilities are found, the report will state so and may still provide gas optimization notes or affirm compliance. If multiple issues are found, they will be listed (often under headings or bullet points by severity).
*   **Error Responses:** In case of an error, you will typically receive an HTTP error status and a JSON error message. Some possible error scenarios:

    * **401 Unauthorized:** API key is missing or invalid. The response may be a JSON like `{"error": "Unauthorized"}`.
    * **402 Payment Required / Error Message about credits:** The account lacks enough credits to fulfill the request. Ensure your credit balance is sufficient.
    * **429 Too Many Requests:** Rate limit exceeded (more than 200 requests/min). You should throttle your requests if you hit this.
    * **400 Bad Request:** If required fields like `model` or `question` are missing or malformed, you’ll get a 4xx error with details. For example, if `model` is not `"smart_contract_auditor"`, the request will not be processed.
    * **500 Internal Server Error:** An unexpected server-side issue occurred. These are rare; you may retry after some delay if it happens.

    Errors will often include a descriptive message. For instance, insufficient credits might return a message like "Not enough credits to perform this action." Always check the HTTP status code and the response body on errors, and implement appropriate retry or user feedback logic in your application.

#### **Example – Audit by Source Code (via API):**

Here’s a simplified example using `curl` to send a contract source for auditing:

```
curl https://api.chaingpt.org/chat/stream \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "smart_contract_auditor",
    "question": "Audit the following contract:\npragma solidity ^0.8.0;\ncontract SimpleStorage {\n    uint256 value;\n    function set(uint256 x) public {\n        value = x;\n    }\n    function get() public view returns (uint256) {\n        return value;\n    }\n}",
    "chatHistory": "off"
  }'
```

This request sends a small `SimpleStorage` contract in the prompt. The response (streamed) might start like:

```
{"bot":"### Audit Report\n**Score:** 90%\n**Summary:** The contract is very simple...","user":"Audit the following contract:...}
```

and so on. Once complete, it would detail that no critical issues were found, perhaps warning if the contract lacks an event for the setter (just an example). You would collect the chunks and parse the final JSON to get the full `bot` message.

#### GET `/chat/chatHistory` — Retrieve Audit History (Optional)

If you enabled `chatHistory: "on"` for your audit requests, the system saved those conversation logs. This endpoint allows you to fetch past prompts and responses, which is useful if you want to review previous audit Q\&A or continue a conversation thread programmatically.

* **URL:** `GET https://api.chaingpt.org/chat/chatHistory`
* **Purpose:** Fetches a list of past chat exchanges (prompts and responses) associated with your API key, optionally filtered by parameters.
* **Headers:**
  * `Authorization: Bearer <API_KEY>` (required, same authentication).
*   **Query Parameters:** (all optional)

    * `limit` – integer, how many records to retrieve (default might be 10).
    * `offset` – integer, for pagination (0-indexed offset of records).
    * `sortBy` – string, field to sort by (e.g., `"createdAt"`).
    * `sortOrder` – string, `"asc"` or `"desc"` for ascending/descending order. Default is descending (most recent first).

    If no parameters are given, a default recent history may be returned.
*   **Response:** JSON object containing an array of chat history entries. Each entry typically includes the user prompt and the AI response, along with metadata.

    Example response structure:

    ```
    {
      "rows": [
        {
          "id": "abc123",
          "user": "Audit the smart contract at 0x123... on Ethereum.",
          "bot": "Audit Report:\n... (report content) ...",
          "createdAt": "2025-04-10T12:34:56Z"
        },
        {
          "id": "def456",
          "user": "What are the implications of the High severity issue?",
          "bot": "The High severity issue means ... (follow-up answer)...",
          "createdAt": "2025-04-10T12:35:10Z"
        }
        // ... more entries ...
      ],
      "count": 2
    }
    ```

    In this example, `rows` is an array of the two most recent Q\&A pairs (the initial audit prompt and a follow-up question). Each has an `id` (identifier), the `user` message, the `bot` reply, and a timestamp. The `count` might indicate total records available (if pagination is supported).
* **Notes:** Only chats where `chatHistory` was turned "on" are recorded. If you never use that feature, this endpoint will return an empty list or not be needed. Retrieving history **does not consume extra credits** (as noted in the docs: you already paid the credit when enabling chat history on the original request). You can use this data to display past audit conversations or feed it back into new requests if needed.
* **Example Request:**\
  `GET /chat/chatHistory?limit=5&offset=0&sortBy=createdAt&sortOrder=desc` with the appropriate Authorization header will fetch the 5 most recent stored exchanges.
* **Error Handling:** Similar to the main endpoint, you can get 401 Unauthorized if the API key is wrong, but other errors here are unlikely if parameters are valid. A 400 could occur if you use an invalid query param value. Also note, if you have no history, the `rows` array could be empty.

***

### Additional Information

* **Streaming vs Non-Streaming:** The `/chat/stream` endpoint is designed to stream responses. If you prefer not to handle streaming on the client side, one approach is to wait for the connection to close and collect the full response (which the SDK’s `blob` method does internally). When using raw HTTP, you might set a suitable timeout and read the entire response into memory. Just be aware that very large reports could be a few hundred KB of text. In most scenarios this is manageable.
* **Content Format:** The audit report (`bot` content) is often formatted in Markdown (with headings, bullet points, etc.) for readability. This makes it convenient to display in developer tools or UIs (you can render it as Markdown to get styled output). The content will include the severity labels, so you can parse them if needed (e.g., to count how many "Critical" issues were found).
* **Audit Scope:** The Auditor currently focuses on Solidity smart contracts. If you provide other languages or unsupported formats, the results may be less reliable. Support for more languages is expanding. For best results, use verified Solidity code or EVM-based contract addresses.

By using the above endpoint and parameters, you can fully automate smart contract audits, retrieving both immediate reports and any follow-up context as needed.
