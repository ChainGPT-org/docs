# API Reference

## ChainGPT Web3 LLM - API Reference

### Introduction

ChainGPT’s Web3 LLM API (formerly _General Chat_ API) is a RESTful interface that allows developers to integrate a Web3-aware AI chatbot into their applications. This API provides endpoints for generating conversational responses (either as a complete message or via streaming partial results) and managing chat history. Using these endpoints, you can build intelligent conversational interfaces that leverage blockchain and cryptocurrency context.

#### Key features include:

* **Real-time Chat Completions:** Submit a user question or prompt and receive an AI-generated response. You can choose between a single response payload or a streamed response for real-time data delivery.
* **Contextual Customization:** Optionally provide custom context (such as company info, token details, or desired tone of response) to tailor the assistant’s behavior and output. This can be done through pre-configured context tied to your API key, or by injecting context data on each request.
* **Conversation Memory:** Enable chat history to have the AI remember previous interactions for a given session. The API can store and retrieve conversation history, allowing the assistant to maintain context over multiple exchanges.
* **Web3 Domain Knowledge:** The assistant is designed with blockchain terminology and crypto token data in mind. You can supply blockchain metadata (e.g. token contract info, supported chains) so that responses include relevant on-chain information.
* **Flexible Tone Control:** Adjust the style of the AI’s responses by selecting from preset tones (professional, friendly, etc.) or defining a custom tone, enabling you to fine-tune how the assistant communicates.

This reference guide covers all available endpoints, request parameters, response formats, and usage examples for the ChainGPT Web3 LLM API. It is intended for developers and engineers looking to integrate ChainGPT’s AI assistant into their own platforms.

***

### Authentication and API Keys

All API requests require an API key for authentication. You must include this key in the request headers as a Bearer token. If you have not generated an API key yet, follow these steps:

1. **Sign Up / Log In:** Create an account or log in to the ChainGPT web app (the AI Hub) at [app.chaingpt.org](https://app.chaingpt.org).
2. **Navigate to API Dashboard:** Access the API Dashboard section of the web app. [https://app.chaingpt.org/apidashboard](https://app.chaingpt.org/apidashboard)&#x20;
3. **Generate an API Key:** Click on the "Create New Secret Key" button to generate a new API key. You may give it an identifiable name for your own records (e.g. “My Web3 Chatbot Key”).
4. **Customize Your API Key:** You can fully customize each API key to match your company, crypto token, or specific use case by configuring it in the AI Hub. This includes setting your project name, website, token details, preferred response tone, and even uploading a custom knowledge base (e.g. FAQs, docs). Once saved, this context is automatically applied to all requests made with that key—no need to resend it every time.
5. **Securely Store the Key:** Once generated, copy your API key and store it in a secure location. Do not share or expose this key publicly. Treat it like a password – anyone with your key can make requests consuming your credits.

{% hint style="info" %}
Tip: For security, never hard-code your API key in your application code or front-end. Instead, use environment variables or a secure secrets manager to inject the key at runtime. This prevents accidental exposure of the key in your source code repositories.
{% endhint %}

**When making requests, include the following HTTP header:**

```
Authorization: Bearer YOUR_API_KEY
```

Replace YOUR\_API\_KEY with the value of the key you obtained. The API uses this for authenticating your requests and for tracking usage against your account.

**Base URL:** All endpoints described below are accessed via the base URL https://api.chaingpt.org. The endpoints are versioned implicitly (current version v1); if breaking changes are introduced in the future, they will be released under a new version or path. For now, no version prefix is required in the URL.

***

### Usage Overview

**The Web3 LLM API supports two modes of retrieving chat responses:**

* **Non-Streaming (Blob) Response:** The API returns the entire response message in a single JSON payload. This is a standard HTTP POST request to the /chat endpoint. Use this mode if you prefer simplicity or your application does not need token-level streaming.
* **Streaming Response:** The API returns the response incrementally as a stream of chunks. This is accessed via a POST request to the /chat/stream endpoint. Streaming allows your application to begin processing or displaying the answer as it’s being generated (useful for longer responses or interactive feel).

In both modes, you send a JSON request body containing at minimum a model and a question (prompt). The primary model available is "general\_assistant", which is ChainGPT’s general-purpose Web3 chatbot. Additional models might be introduced in the future, but currently this should be used for all requests.

**You can also include optional fields to customize the behavior:**

* **chatHistory –** Set to "on" to enable conversation memory, or "off" to treat the request as standalone. If chat history is on, the system will remember this interaction and use it in future responses (see Chat History below). By default, if not provided, chat history is off (no memory).
* **sdkUniqueId –** A unique identifier (string/UUID) for isolating chat history per end-user or session. Use this if you have multiple users or separate chat sessions under one API key. By providing a different sdkUniqueId for each user/session (especially when chatHistory is on), you ensure each conversation thread is kept separate. If you don’t provide an sdkUniqueId, all history-enabled requests under the same API key will share a single history.
* **useCustomContext –** Boolean flag to enable custom context. When false (default), the assistant behaves with a generic context. When true, the assistant will incorporate additional context about your project or organization. This context can come from two sources:
  * **Pre-configured (AI Hub) context:** If useCustomContext is true _and_ you do not send a contextInjection object in the request, the API will automatically fetch the default context associated with your API key from the ChainGPT AI Hub. (In the API Dashboard, you can fill in details like your company name, description, website, etc. These details become the default custom context for your key.)
  * **On-the-fly context injection**: If useCustomContext is true _and_ you do provide a contextInjection object in the request, the fields in that object will override or supplement the default context for this request. This allows you to dynamically inject or change contextual information per request.
* **contextInjection –** An object allowing you to specify custom context data directly in the API call (effective only if useCustomContext is true). Through this, you can inform the chatbot about your company/project, provide relevant links, set the desired tone/style of responses, and more. See Context Injection Fields for detailed sub-parameters.

By leveraging useCustomContext and contextInjection, you can personalize the AI’s responses. For example, you might configure the assistant to respond as a representative of your company, or to have a specific friendly or formal tone, or to have knowledge of a particular cryptocurrency token that your project has.

Finally, the API offers a chat history endpoint to retrieve past conversations. This is useful for auditing, displaying conversation transcripts to users, or continuing conversations after a pause. The chat history can be filtered by the sdkUniqueId to get history for a particular user or session.

{% hint style="info" %}
Note: Each API request consumes credits from your ChainGPT account. The cost is 0.5 credits per chat request, plus an additional 0.5 credits if chat history is enabled for that request (since storing and managing conversation context incurs extra processing). Ensure your account has sufficient credits to cover your usage. If your credits are exhausted, API calls will fail with an error indicating insufficient credits.
{% endhint %}

***

### Endpoints

This section describes each API endpoint in detail, including request and response formats and example usage in both cURL and Axios (Node.js). All endpoints expect the Authorization: Bearer header with your API key.

#### POST /chat – Chat Completion (Non-Streaming)

Use this endpoint to submit a prompt or question to the AI and receive the full response in one JSON response. This is a blocking call that will return once the AI has generated the complete answer.

* **URL:** https://api.chaingpt.org/chat
* **Method:** POST
* **Headers:** Authorization: Bearer YOUR\_API\_KEY (required), plus Content-Type: application/json for the request body.
* **Request Body:** JSON object with the following fields:

<table><thead><tr><th width="117.55859375">Parameter</th><th width="105.3515625">Type</th><th width="83.41796875">Required</th><th>Description</th></tr></thead><tbody><tr><td>model</td><td>string</td><td>Yes</td><td>ID of the model to use for generation. Use "general_assistant" for the default ChainGPT chatbot model.</td></tr><tr><td>question</td><td>string</td><td>Yes</td><td>The user’s prompt or query for the chatbot. This is the message you want the AI to respond to.</td></tr><tr><td>chatHistory</td><td>string ("on"/"off")</td><td>No (default "off")</td><td>Whether to enable conversation memory for this request. If "on", this question and its answer will be stored and used as context for future requests. If "off", the AI will not consider any past interactions (and this interaction will not be saved).</td></tr><tr><td>sdkUniqueId</td><td>string (UUID or similar)</td><td>No</td><td>A unique session identifier to isolate chat history. Provide this if you want to maintain separate conversation threads under the same API key (e.g., per end-user). When chatHistory is "on", using a consistent sdkUniqueId ensures the AI only uses history from that specific session. If omitted, all history-enabled requests under your API key share one history.</td></tr><tr><td>useCustomContext</td><td>boolean</td><td>No (default false)</td><td>If set to true, the assistant will incorporate custom context. Without a contextInjection object, it uses the default context tied to your API key (configured in the AI Hub dashboard). With a contextInjection provided, it uses the provided context data (overriding any default context) for this response.</td></tr><tr><td>contextInjection</td><td>object</td><td>No</td><td>An object defining custom context details to inject. Requires useCustomContext: true to take effect. This object’s fields (company info, tone, token details, etc.) will influence the assistant’s response. See Context Injection Fields below for all available sub-fields.</td></tr></tbody></table>

All request body parameters should be encapsulated in a single JSON object. For example, a minimal request might include just model and question, while a more complex request could toggle chatHistory or supply contextInjection fields.

* **Response:** On success, the server will return HTTP 200 with a JSON body containing the AI’s answer. The typical response structure is:

```
{
  "status": true,
  "message": "Chat response generated successfully.",
  "data": {
    "bot": "<assistant's answer text>"
  }
}
```

The bot field inside the data object holds the assistant’s answer to your question. (The property is named “bot” to indicate the chatbot’s reply.) Additional metadata may also be included, such as an identifier or timestamp for the interaction, but the primary content of interest is the bot string.

* **Errors:** If the request is malformed or missing required fields, you will receive a 400 Bad Request with an error message. An invalid or missing API key will yield a 401 Unauthorized. Other error codes: 403/402 if you have insufficient credits, 429 Too Many Requests if you exceed rate limits, or 5xx for server errors. The error response body typically contains a JSON with status: false and an error message describing the issue.

**Example – Single Chat Completion (Axios):**

```
import axios from 'axios';

async function getAnswer() {
  const API_URL = 'https://api.chaingpt.org/chat';
  const API_KEY = process.env.CHAINGPT_API_KEY;  // Your ChainGPT API key

  try {
    const response = await axios.post(API_URL, {
      model: "general_assistant",
      question: "Explain the concept of AI",
      chatHistory: "off"
    }, {
      headers: { Authorization: `Bearer ${API_KEY}` }
    });
    console.log("Assistant answer:", response.data.data.bot);
  } catch (error) {
    console.error("API request failed:", error.response?.data || error.message);
  }
}

getAnswer();
```

In this example, we send a prompt to explain AI. We explicitly set chatHistory: "off" to ensure a standalone answer (no history). The assistant’s full answer will be printed to the console.

**Example – Single Chat Completion (cURL):**

```
curl -X POST "https://api.chaingpt.org/chat" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "model": "general_assistant",
        "question": "Explain the concept of AI",
        "chatHistory": "off"
      }'
```

This cURL command sends the same request. The response will be a JSON object containing the answer. For example:

```
{
  "status": true,
  "message": "Chat response generated successfully.",
  "data": {
    "bot": "Artificial Intelligence (AI) is a field of computer science focused on creating systems capable of performing tasks that typically require human intelligence. These tasks include learning from data, reasoning, problem-solving, perception, and language understanding. AI systems can range from simple algorithms to complex neural networks, and they power applications like virtual assistants, recommendation engines, and autonomous vehicles."
  }
}
```

#### POST /chat/stream – Chat Completion (Streaming)

Use this endpoint to receive the AI response as a stream of data, rather than waiting for the full completion. This is ideal for applications where you want to display the answer in real-time as it’s being generated (similar to how other General LLM streams its answers).

* **URL:** https://api.chaingpt.org/chat/stream
* **Method:** POST
* **Headers:** Authorization: Bearer YOUR\_API\_KEY, Content-Type: application/json
*   **Request Body:** JSON object with the same fields as the /chat endpoint (see the parameter table above). All parameters and behavior (model, question, chatHistory, sdkUniqueId, useCustomContext, contextInjection) are used in the same way for streaming.

    Note: The request format is identical to the non-streaming endpoint; the only difference is the URL path used, which dictates that the response will be sent as a stream.
*   **Response:** On success, the server will start sending back chunks of data over the HTTP connection. Each chunk represents a part of the assistant’s answer (e.g., a sentence or portion of text). The exact streaming implementation is via an HTTP chunked response. You should keep the connection open to receive all chunks until the stream ends.

    If you are using Node.js or another backend environment, you can handle the streaming response by listening to the response stream. For example, with Node’s axios or fetch, you would set the response type to “stream” and then handle the data events. In a browser environment, you could use the Fetch API with ReadableStream or SSE (if the API uses server-sent events format) to process chunks as they arrive.

    Once the entire answer has been sent, the server will terminate the stream (end of response). At that point, you can finalize output to the user.
* **Errors:** If there is an authentication or request error, the server will usually terminate the connection immediately with the appropriate HTTP status code (and possibly an error JSON before closing). Ensure to handle error events on the stream. If the stream is interrupted or the connection drops, you may not receive a complete answer (your application should handle such cases gracefully, perhaps by retrying or informing the user).

**Example – Streaming Chat Completion (Axios):**

```
import axios from 'axios';
import { createWriteStream } from 'fs';  // optional, for demonstration

async function streamAnswer() {
  const API_URL = 'https://api.chaingpt.org/chat/stream';
  const API_KEY = process.env.CHAINGPT_API_KEY;

  // Create an Axios instance with streaming response support
  const apiClient = axios.create({
    baseURL: API_URL,
    timeout: 60000,
    headers: { Authorization: `Bearer ${API_KEY}` },
    responseType: "stream"   // important for streaming responses
  });

  try {
    const response = await apiClient.post("/", {
      model: "general_assistant",
      question: "Give me the latest news about Ethereum.",
      chatHistory: "off"
    });
    const stream = response.data;  // this is a Readable stream
    stream.on("data", (chunk) => {
      const text = chunk.toString();
      process.stdout.write(text);  // output chunk to console (could append to a variable or UI element)
    });
    stream.on("end", () => {
      console.log("\n[Stream ended]");
    });
  } catch (error) {
    console.error("Error during streaming request:", error.response?.data || error.message);
  }
}

streamAnswer();
```

In this Node.js example, we send a question and handle the returned stream. As chunks of the answer arrive, we convert them to string and immediately print them (this could be real-time update in a UI). When the stream ends, we log a message. The result is that the answer text is printed gradually, as if the AI is “typing” it out.

**Example – Streaming Chat Completion (cURL):**

You can also invoke streaming via cURL. The response will stream to your terminal. For example:

```
curl -X POST "https://api.chaingpt.org/chat/stream" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "model": "general_assistant",
        "question": "Give me the latest news about Ethereum.",
        "chatHistory": "off"
      }'
```

This will start outputting the answer text to the console as it is generated. You may see partial sentences appear over time until the full answer is delivered and the command exits. (If nothing appears for a while, the AI might be processing; ensure your terminal is not buffering the output. You can add --no-buffer option in curl to see data as soon as it’s received.)

#### GET /chat/chatHistory – Retrieve Chat History

Use this endpoint to fetch past chat interactions associated with your API key, including questions and answers. This is particularly useful if you have enabled chatHistory for your requests and you want to display or utilize the conversation log. You can retrieve all history or filter to a specific session (via sdkUniqueId) and control pagination.

* **URL:** https://api.chaingpt.org/chat/chatHistory
* **Method:** GET
* **Headers:** Authorization: Bearer YOUR\_API\_KEY
* **Query Parameters:** Provide the following as URL query parameters or as an axios params object:

<table><thead><tr><th width="105.42578125">Parameter</th><th width="94.90234375">Type</th><th width="129.98828125">Required</th><th>Description</th></tr></thead><tbody><tr><td>limit</td><td>integer</td><td>No (default depends on server, e.g. 10)</td><td>The number of history records to retrieve per request. Use this for pagination. For example, limit=10 will fetch up to 10 question-answer entries.</td></tr><tr><td>offset</td><td>integer</td><td>No (default 0)</td><td>The number of records to skip from the start, used for pagination. For example, offset=0 starts from the latest (or earliest) record depending on sort order, offset=10 would skip the first 10 records.</td></tr><tr><td>sortBy</td><td>string</td><td>No (default "createdAt")</td><td>The field by which to sort the history records. Currently, the typical field is "createdAt" (timestamp of the interaction).</td></tr><tr><td>sortOrder</td><td>string ("asc"/"desc")</td><td>No (default "desc")</td><td>Sort order for the results. "desc" for descending (e.g., newest first), or "asc" for ascending order.</td></tr><tr><td>sdkUniqueId</td><td>string</td><td>No</td><td>If provided, filters the history to only include interactions that were associated with the given sdkUniqueId (i.e., history of a specific user session). Use this to retrieve conversation logs per end-user or session in a multi-user application. If omitted, the history will include all stored interactions under your API key.</td></tr></tbody></table>

* **Response:** On success (HTTP 200), the response will include the chat history records in JSON format. The structure is:

```
{
  "status": true,
  "message": "Chat history retrieved successfully.",
  "data": {
    "rows": [
      {
        "id": "<record-id-1>",
        "question": "<user question text>",
        "bot": "<assistant answer text>",
        "createdAt": "2025-04-13T21:48:00.123Z",
        "sdkUniqueId": "<session-id-if-used>"
      },
      {
        "id": "<record-id-2>",
        "question": "<next user question>",
        "bot": "<assistant answer>",
        "createdAt": "2025-04-13T21:49:30.456Z",
        "sdkUniqueId": "<session-id-if-used>"
      }
      // ... more records up to the limit
    ],
    "count": 10
  }
}
```

Each object in the rows array represents one question-answer pair. The exact fields may include an internal id, the original question, the bot response, a timestamp (createdAt), and the sdkUniqueId if the record was tied to a specific session. The count field indicates the number of records returned (or possibly the total available records depending on implementation).

If no history is found (e.g., you haven’t enabled chatHistory for any requests yet, or the sdkUniqueId filter has no matches), the rows array will be empty.

* **Errors:** If the query parameters are invalid (e.g., non-numeric where numeric expected), you may get a 400 Bad Request. Unauthorized requests will get 401. Always ensure your API key is correct and that you have proper access.

**Example – Retrieve All Chat History (Axios):**

```
import axios from 'axios';

async function fetchHistory() {
  const HISTORY_URL = 'https://api.chaingpt.org/chat/chatHistory';
  const API_KEY = process.env.CHAINGPT_API_KEY;
  
  try {
    const response = await axios.get(HISTORY_URL, {
      headers: { Authorization: `Bearer ${API_KEY}` },
      params: {
        limit: 5,
        offset: 0,
        sortBy: "createdAt",
        sortOrder: "desc"
      }
    });
    console.log("Recent history entries:", response.data.data.rows);
  } catch (error) {
    console.error("Failed to fetch chat history:", error.response?.data || error.message);
  }
}

fetchHistory();
```

This example fetches the 5 most recent chat history records (since we sorted by createdAt descending). The results are logged to the console as an array of objects.

**Example – Retrieve History for a Specific Session (Axios):**

```
async function fetchUserHistory(userId) {
  const HISTORY_URL = 'https://api.chaingpt.org/chat/chatHistory';
  const API_KEY = process.env.CHAINGPT_API_KEY;
  try {
    const response = await axios.get(HISTORY_URL, {
      headers: { Authorization: `Bearer ${API_KEY}` },
      params: {
        limit: 10,
        offset: 0,
        sortBy: "createdAt",
        sortOrder: "asc",
        sdkUniqueId: userId    // filter by a specific session's ID
      }
    });
    console.log(`History for user ${userId}:`, response.data.data.rows);
  } catch (error) {
    console.error("Failed to fetch user chat history:", error.response?.data || error.message);
  }
}

// Example usage:
fetchUserHistory("907208eb-0929-42c3-a372-c21934fbf44f");
```

In this snippet, userId could be a UUID or unique string you assigned to an end-user’s session. The call will return that user’s conversation with the AI (assuming sdkUniqueId was provided in the original chat requests). We request up to 10 entries, sorted oldest to newest (sortOrder: "asc").

**Example – Retrieve Chat History (cURL):**

```
curl -G "https://api.chaingpt.org/chat/chatHistory" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  --data-urlencode "limit=5" \
  --data-urlencode "offset=0" \
  --data-urlencode "sortBy=createdAt" \
  --data-urlencode "sortOrder=desc"
```

Using -G with curl converts it to a GET and allows --data-urlencode to append query parameters. The above will fetch the 5 latest history records. You can add sdkUniqueId=\<your\_id> similarly to filter by session. The response will be a JSON similar to the example shown earlier, containing the array of history entries.

{% hint style="info" %}
Tip: Remember that only interactions made with chatHistory: "on" are recorded. If you call the chat endpoints with history off, those exchanges will not appear in the chat history log. Also, if you want to isolate history per user in a multi-user system, you _must_ use sdkUniqueId consistently when creating chat completions. Otherwise, all saved history will be mixed together.
{% endhint %}

***

### Context Customization and Injection

One of the powerful features of the ChainGPT Web3 LLM API is the ability to customize the assistant’s context and tone. By providing additional information, you can influence the responses to be more relevant to your project or adopt a certain style. There are two ways to leverage this:

* **Default Custom Context via AI Hub:** You can configure background information for your API key in the ChainGPT AI Hub (web dashboard). This might include your company name, description, website, whitepaper link, and the intended purpose of your chatbot. When you set useCustomContext: true without any further context details in the request, the API will automatically pull in this pre-set context for the AI. This means the assistant will answer as if it’s representing your organization or has knowledge of the provided details.
* **On-Demand Context Injection:** For dynamic or additional context, or if you prefer to supply the info directly with the API call, you can use the contextInjection object in your request. Any fields you include here will override or add to the default context from AI Hub for that request. This is useful if you want to tweak the context on a per-request basis or provide information that changes frequently (for example, a daily updated statistic, or user-specific data).

When using context injection, you can provide a range of details such as branding information, links, whether the chatbot should incorporate knowledge of a specific crypto token, and even instruct the AI to use a particular tone or style.

#### Context Injection Fields

Below is a reference for all fields that can be included in the contextInjection object. All fields are optional – you can include only those relevant to the context you want to set. Any field omitted will either remain as the default (if defined in AI Hub for your key) or simply not be used in the context.

<table><thead><tr><th width="123.04296875">Field</th><th width="86.5625">Type</th><th>Description</th></tr></thead><tbody><tr><td>companyName</td><td>string</td><td>The name of your company or project that the AI is representing. This helps the chatbot refer to the company by name and speak as if it’s associated with it.</td></tr><tr><td>companyDescription</td><td>string</td><td>A description of the company/project. You can include the domain or industry, what the project does, mission, etc. The assistant can draw on this description to answer questions about the project.</td></tr><tr><td>companyWebsiteUrl</td><td>string (URL)</td><td>The URL of the company or project website. This can be provided in responses or used as context. <em>(Make sure to include the full URL with protocol, e.g., “https://…”)</em>.</td></tr><tr><td>whitePaperUrl</td><td>string (URL)</td><td>URL to the project’s whitepaper, documentation, or any technical paper. If provided, the AI might reference it or direct users to it for more detailed info.</td></tr><tr><td>purpose</td><td>string</td><td>The purpose or role of the AI chatbot for this company. For example, “To assist users with information about our DeFi platform and guide them in using our services.” This field lets you clarify why the chatbot exists, which can influence how it responds (e.g., more support-oriented, more marketing-oriented, etc.).</td></tr><tr><td>cryptoToken</td><td>boolean</td><td>Set this to true if your project has a cryptocurrency token and you want the chatbot to be able to provide token-related info. When enabled, you should provide details in the tokenInformation field. If false or omitted, the assistant will not specifically inject token data unless it comes up generally.</td></tr><tr><td>tokenInformation</td><td>object</td><td>An object containing information about your crypto token (if cryptoToken is true). This allows the AI to answer questions about the token’s specifics. Fields for this object are detailed below.</td></tr><tr><td>socialMediaUrls</td><td>array of objects</td><td>A list of social or community links for the project. Each element is an object with { "name": &#x3C;platform>, "url": &#x3C;link> }. For example, you can include links to your LinkedIn, Twitter, Telegram, Instagram, YouTube, Medium, etc. (Only include those relevant to your project.) The assistant might use these to answer questions about where to find more information or community updates.</td></tr><tr><td>limitation</td><td>boolean</td><td>A flag that might indicate if the project has limitations it wants the AI to observe. By default this can be false. (This field is for any constraints or content limitations you want to enforce in answers. Its exact behavior depends on how the backend uses it. For instance, if limitation: true, the assistant might avoid certain topics or be more cautious. If false, no special limitation is applied beyond the general AI content controls.)</td></tr><tr><td>aiTone</td><td>string</td><td>The tone mode for the AI’s responses. This controls the style/personality of the chatbot. Accepted values: "DEFAULT_TONE", "CUSTOM_TONE", or "PRE_SET_TONE". See Tone Customization below for details.</td></tr><tr><td>selectedTone</td><td>string</td><td>If aiTone is set to "PRE_SET_TONE", use this field to choose a specific preset tone. For example "FRIENDLY" or "FORMAL". (If aiTone is not "PRE_SET_TONE", this field is ignored.) See the list of supported preset tone options below.</td></tr><tr><td>customTone</td><td>string</td><td>If aiTone is "CUSTOM_TONE", provide a brief description of the custom tone or style you want the AI to adopt. For example, "Speak in a playful, casual manner using crypto slang.". This text will guide the AI’s style. (Ignored for other aiTone values.)</td></tr></tbody></table>

**Token Information Fields:** If you include the tokenInformation object (and have cryptoToken: true), you can provide the following subfields inside it:

* **tokenName (string):** The name of your token (e.g., “ChainGPT Token”).
* **tokenSymbol (string):** The token’s ticker or symbol (e.g., "CGPT"), including any prefix like $ if desired.
* **tokenAddress (string):** The contract address of the token (for the primary blockchain it’s on). Providing this can help the AI reference the token’s smart contract or verify its existence.
* **tokenSourceCode (string):** The source code of the token’s contract, or a URL to it, if applicable. (If the AI is asked about the token’s contract details, having the source could be useful.)
* **tokenAuditUrl (string, URL):** A link to a security audit report for the token’s smart contract, if available. This lends credibility and detail if users inquire about security.
* **exploreUrl (string, URL):** A link to a block explorer page for the token (e.g., Etherscan link for the contract). This allows the assistant to direct users to see on-chain details.
* **cmcUrl (string, URL):** A link to the token’s page on CoinMarketCap (if applicable).
* **coingeckoUrl (string, URL):** A link to the token’s page on CoinGecko.
*   **blockchain (array of strings):** One or more blockchain networks where this token is deployed or relevant. This helps the AI contextualize the token (e.g., whether it’s an ERC-20 on Ethereum, BEP-20 on BSC, etc.). You can list multiple if the token is multichain. Supported values include:

    * "ETHEREUM"
    * "BSC" (Binance Smart Chain)
    * "ARBITRUM"
    * "BASE"
    * "BLAST"
    * "AVALANCHE"
    * "POLYGON"
    * "SCROLL"
    * "OPTIMISM"
    * "LINEA"
    * "ZKSYNC"
    * "POLYGON\_ZKEVM" (Polygon zkEVM)
    * "GNOSIS"
    * "FANTOM"
    * "MOONRIVER"
    * "MOONBEAM"
    * "BOBA"
    * "METIS"
    * "LISK"
    * "AURORA"
    * "SEI"
    * "IMMUTABLE\_ZK" (Immutable X / zkEVM)
    * "GRAVITY"
    * "TAIKO"
    * "CRONOS"
    * "FRAXTAL"
    * "ABSTRACT"
    * "WORLD\_CHAIN"
    * "MANTLE"
    * "MODE"
    * "CELO"
    * "BERACHAIN"

    Include the networks relevant to your token in the array. For example, a token primarily on Ethereum and Polygon might have "blockchain": \["ETHEREUM", "POLYGON"]. The assistant can use this to clarify which chain’s data it is referring to if asked about token specifics.

Given all these fields, you can mix and match what context to provide. You might only set a companyName and purpose, or you might fill in everything if you want a very detailed persona for the AI.

{% hint style="info" %}
Tip: Context injection is powerful for tailoring the AI, but use it thoughtfully. Overloading the assistant with excessive or irrelevant context might confuse its responses. Generally, provide the key pieces of information that the AI should know or adhere to, rather than lengthy text. The model will incorporate this info as background when formulating its answer.
{% endhint %}

**Example – Using Custom Context (AI Hub default):**

If you have configured your project’s details in the ChainGPT web dashboard (AI Hub), you can simply send requests with useCustomContext: true to have the assistant automatically act with that context. For instance:

```
{
  "model": "general_assistant",
  "question": "What is your project about?",
  "useCustomContext": true,
  "chatHistory": "off"
}
```

If your API key’s default context includes companyName: "Acme DeFi", companyDescription: "Acme DeFi is a lending platform on Ethereum...", etc., then the assistant’s answer to _“What is your project about?”_ will incorporate those details (responding as “Acme DeFi” and explaining the lending platform). You did not need to provide contextInjection explicitly here, since the info is pulled from your stored profile.

**Example – Using Minimal Context Injection:**

You can slightly influence the tone or a particular detail without changing everything. Suppose you mostly want the generic behavior, but just want the AI to sound friendly. You could do:

```
await axios.post(API_URL, {
  model: "general_assistant",
  question: "Hello! Can you tell me about this service?",
  useCustomContext: true,
  chatHistory: "off",
  contextInjection: {
    aiTone: "PRE_SET_TONE",
    selectedTone: "FRIENDLY"
  }
}, { headers: { Authorization: `Bearer ${API_KEY}` } });
```

Here we set aiTone to a preset and choose the “FRIENDLY” tone. We didn’t specify any company info, so if there’s default context in AI Hub it would use that; if not, it just uses friendly tone with no extra project knowledge. The response will be in a noticeably friendly style.

**Example – Using Full Context Injection:**

For a comprehensive example, imagine we want the assistant to represent _Crypto Solutions_, a blockchain security firm that has a token. We want a formal tone. We haven’t set up the AI Hub profile, so we will inject everything in the request:

```
await axios.post(API_URL, {
  model: "general_assistant",
  question: "What services do you offer?",
  useCustomContext: true,
  chatHistory: "off",
  contextInjection: {
    companyName: "Crypto Solutions",
    companyDescription: "Crypto Solutions is a blockchain security firm specializing in smart contract audits and security consultations.",
    companyWebsiteUrl: "https://www.cryptosolutions.example",
    purpose: "Assist users with information about Crypto Solutions’ offerings and guide them on security best practices.",
    cryptoToken: true,
    tokenInformation: {
      tokenName: "SecureToken",
      tokenSymbol: "$SEC",
      tokenAddress: "0x1234...abcd",
      tokenAuditUrl: "https://audit.example/SecureToken.pdf",
      coingeckoUrl: "https://coingecko.com/en/coins/securetoken",
      blockchain: ["ETHEREUM", "POLYGON"]
    },
    socialMediaUrls: [
      { name: "Twitter", url: "https://twitter.com/CryptoSolutions" },
      { name: "LinkedIn", url: "https://linkedin.com/company/crypto-solutions" }
    ],
    limitation: false,
    aiTone: "PRE_SET_TONE",
    selectedTone: "FORMAL"
  }
});
```

This request provides a rich context: the AI knows it is “Crypto Solutions”, what the company does, it has a token with given details, and it should respond in a formal tone. A user asking “What services do you offer?” would get an answer that explicitly mentions Crypto Solutions’ services (likely audit and consultation), possibly references their token if relevant, and does so in a formal manner.

This method overrides any default context – even if the AI Hub had some info, our provided contextInjection will take precedence for this call.

#### Tone Customization

Tone control is a feature that allows you to modify _how_ the AI speaks and presents information, without changing the factual content. The API provides two ways to control tone:

* **Preset Tones:** A selection of predefined styles like Friendly, Professional, Playful, etc.
* **Custom Tone:** A free-form instruction describing the style you want.

By default, if you do not specify any tone, the assistant will respond in a neutral, context-appropriate manner. If you want to enforce a specific style, use the aiTone field in context injection.

**Using Preset Tones:**

To use a preset tone, set aiTone: "PRE\_SET\_TONE" and pick one of the options for selectedTone. The supported preset tone options are:

* **PROFESSIONAL** – The AI responds in a very professional, corporate tone.
* **FRIENDLY** – The AI is warm, approachable, and casual.
* **INFORMATIVE** – The AI focuses on being factual and detailed.
* **FORMAL** – The AI uses formal language and etiquette.
* **CONVERSATIONAL** – The AI is informal and chatty, like talking to a friend.
* **AUTHORITATIVE** – The AI speaks confidently and assertively as an authority.
* **PLAYFUL** – The AI may use humor or a light-hearted style.
* **INSPIRATIONAL** – The AI motivates and encourages, with positive tone.
* **CONCISE** – The AI gives brief and to-the-point answers.
* **EMPATHETIC** – The AI responds with understanding and empathy.
* **ACADEMIC** – The AI uses a scholarly tone, as if writing an academic paper.
* **NEUTRAL** – The AI keeps a neutral, impartial tone.
* **SARCASTIC\_MEME\_STYLE** – The AI responds with a sarcastic or meme-like flair (use with caution in professional contexts).

**For example, to have the AI respond in a playful tone, you would include:**

```
{
  "aiTone": "PRE_SET_TONE",
  "selectedTone": "PLAYFUL"
}
```

in your contextInjection.

**Using a Custom Tone:**

If the presets don’t cover the style you need, you can define a custom tone. Set aiTone: "CUSTOM\_TONE" and provide a customTone string describing what you want. Think of this as telling the AI the persona or style: e.g., _“Speak as a pirate,”_ or _“Respond in a casual tone with emojis,”_ or _“Use technical jargon and assume the reader has a computer science background.”_

**For example:**

```
{
  "aiTone": "CUSTOM_TONE",
  "customTone": "Use a lot of crypto slang and be very casual, as if chatting with a fellow trader."
}
```

The AI will attempt to adapt to that style for the response.

Important: If you choose DEFAULT\_TONE (which is the behavior when aiTone is omitted or explicitly set to "DEFAULT\_TONE"), the selectedTone and customTone fields are not needed and will be ignored. The assistant will use its normal tone (which might still be influenced somewhat by any company context, but not in a strong stylistic way).

#### Incorporating Blockchain and Token Data

The Web3 LLM API is particularly useful for blockchain-related projects. By providing your token and blockchain information via contextInjection.cryptoToken and tokenInformation, you enable the assistant to give more accurate and detailed answers about your token or platform.

For instance, if a user asks _“What is the current supply of $TOKEN?”_ or _“Where can I trade $TOKEN?”_, the assistant, with token context provided, can answer in context (it might know where to find that info or at least respond with something like “You can find $TOKEN on Ethereum and Polygon; the contract addresses are X and Y. Check the block explorer links for details.”). Without that context, the assistant might give a generic “I’m not aware of that token” type answer.

Similarly, by specifying blockchain networks, if a user asks something like _“Does your project operate on BSC?”_ the bot can directly know (from the context) which chains are relevant.

**Dynamic Data:** Note that while you can provide static info such as links and names, the API does not automatically fetch real-time data (like latest price or supply numbers) as part of the conversation. If you want real-time data in answers, you have two options:

1. **Update context for each query –** Your application can fetch the data from an API (e.g., a price feed) and then include it in the contextInjection (perhaps in the companyDescription or a custom field) before sending to ChainGPT. This way, the assistant will have the latest data in its context when forming a response.
2. **Ask the assistant –** Simply ask in the prompt (if the model has been designed to fetch or has access – however, generally these models do not have browsing ability, so method 1 is preferred for dynamic data).

**For example, you could do:**

```
// Pseudo-code: inject dynamic info
const price = await getTokenPriceFromAPI();
await axios.post(API_URL, {
  model: "general_assistant",
  question: `What is the current price of our token?`,
  useCustomContext: true,
  contextInjection: {
    companyName: "MyProject",
    cryptoToken: true,
    tokenInformation: { tokenName: "MyToken", tokenSymbol: "$MYT" },
    // inject latest price into description for context
    companyDescription: `MyProject's token (MYT) is currently trading at $${price} per token.`,
    aiTone: "PRE_SET_TONE",
    selectedTone: "INFORMATIVE"
  }
});
```

The above would prime the assistant with the latest price so it can answer accurately. This is a form of dynamic context injection.

***

### Advanced Usage and Tips

#### Session Management with sdkUniqueId

If you have multiple end-users or want to support multiple independent conversations simultaneously, use the sdkUniqueId field to differentiate their sessions. The sdkUniqueId can be any unique string (a UUID is a good choice) that your application assigns per user or per conversation.

How it works: When you call /chat or /chat/stream with chatHistory: "on" and a given sdkUniqueId, the backend ties the stored history to that ID. If you later call the chat endpoint again with the same sdkUniqueId, the assistant will remember the previous exchanges for that ID only. Other sessions with different IDs will not mix into this conversation.

This is crucial for building chat experiences where each user has their own ongoing conversation with the AI.

**Example – Managing multiple user sessions:**

```
const userSessionId = "907208eb-0929-42c3-a372-c21934fbf44f";  // unique per user
await axios.post("https://api.chaingpt.org/chat", {
  model: "general_assistant",
  question: "Hello, who are you?",
  chatHistory: "on",
  sdkUniqueId: userSessionId
}, { headers: { Authorization: `Bearer ${API_KEY}` } });

// Later, another question from the same user/session:
await axios.post("https://api.chaingpt.org/chat", {
  model: "general_assistant",
  question: "Can you help me with DeFi lending?",
  chatHistory: "on",
  sdkUniqueId: userSessionId
}, { headers: { Authorization: `Bearer ${API_KEY}` } });
```

The AI will remember that the user greeted it and asked “who are you?” previously, which may influence how it responds to the second question (it might not re-introduce itself, for example). Meanwhile, another user with a different sdkUniqueId starting a conversation will not have any of this context.

When retrieving history via /chat/chatHistory, you can specify sdkUniqueId to get only that user’s conversation, as shown earlier. This is how you might display a chat transcript to the user or resume a conversation after some time (you could fetch their history and feed relevant parts back into context if needed, but the system should already remember if using history on).

#### Error Handling and Retries

When integrating the API, be prepared to handle error responses and network issues. Some best practices:

* **Check HTTP Status Codes:** A non-200 status means the request did not succeed. Your code should detect this (in Axios, for example, it would throw an exception you can catch). The error response typically contains a message. Common codes:
  * **400 Bad Request –** Your request had an issue (missing field, invalid JSON, etc.). The message will usually tell you what’s wrong; fix the request and try again.
  * **401 Unauthorized –** Authentication failed. This could mean your API key is missing, malformed, or incorrect.
  * **402 Payment Required / 403 Forbidden –** This likely indicates you’ve run out of credits or are not allowed to access the resource. Check your credit balance on the ChainGPT dashboard.
  * **429 Too Many Requests –** You’ve hit a rate limit. The API will temporarily reject requests if you send too many in a short time. If this occurs, implement a backoff strategy: wait a few seconds (or the time indicated in any Retry-After header if provided) before retrying.
  * **500/502/503 Server Errors –** These indicate an issue on ChainGPT’s side (or a network intermediary). These are usually transient. You can retry after a brief delay. If they persist, there may be maintenance or an outage.
* **Network Timeouts:** Sometimes a request might time out due to network issues. Using streaming mode, for example, you might want to set a generous timeout (60 seconds as in the examples) to allow the model to respond. If your environment or HTTP client has shorter timeouts, be aware that complex queries might take several seconds. Adjust timeouts as appropriate but also inform users if things are taking long.
* **Implement Retries Carefully:** For idempotent operations like retrieving history, retrying on failure is usually fine. For creating chat completions, you can also retry, but be mindful that each attempt will consume credits if it reaches the API and the model starts processing. If you didn’t get any response (e.g., network failure), a retry is reasonable. If you got a definitive error (like 401 or 400), fix the issue instead of retrying immediately. In case of 429 or 5xx, wait briefly before retrying to avoid flooding a possibly stressed service.\


**Example – Handling errors in code (Axios):**

```
try {
  const response = await axios.post(API_URL, requestData, { headers: { Authorization: `Bearer ${API_KEY}` } });
  // use response.data...
} catch (err) {
  if (err.response) {
    // The request was made and the server responded with a non-2xx status
    console.error("API Error:", err.response.status, err.response.data.message);
    if (err.response.status === 429) {
      // Too many requests - maybe implement a retry after delay
    }
  } else if (err.request) {
    // No response received (e.g., network error or timeout)
    console.error("Network error or no response from API.");
    // Consider retrying or informing the user
  } else {
    // Something else triggered an error
    console.error("Error:", err.message);
  }
}
```

#### Rate Limits

ChainGPT’s API enforces rate limits to ensure fair usage and stability. While exact thresholds are not publicly documented here (they may change over time), you should design your integration with reasonable pacing. If you expect to make a high volume of requests, implement client-side rate limiting or batching. If you hit a 429 Too Many Requests error, slow down your request rate.

A typical strategy is to allow perhaps a few requests per second. If you need to scale beyond that, consider contacting ChainGPT for higher rate limit options or enterprise plans.

#### API Versioning and Updates

This API is subject to improvement and expansion. Backwards-compatible changes (like adding new optional parameters or new preset tones) may happen without a new version. Breaking changes will be avoided in the same endpoint; instead, a version bump or new endpoint would be introduced.

As of this documentation, the Web3 LLM API is in v1. You can keep an eye on the official ChainGPT documentation site or release notes for announcements of new features or versions. When a new version is released, the documentation will outline how it differs and how to migrate if necessary. Always test your integration when upgrading any ChainGPT API components.

***

By following this reference, you should be able to integrate ChainGPT’s Web3 AI assistant into your application effectively. For additional support, refer to ChainGPT’s developer support channels or community forums.&#x20;

**Happy building with ChainGPT’s Web3 LLM API!**
