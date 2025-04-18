# API Reference

## ChainGPT Web3 LLM API – REST API Reference

**ChainGPT’s Web3 LLM API** is a RESTful interface that lets developers integrate a Web3-aware AI assistant into their applications. This single endpoint can return the AI’s answer either as a **complete JSON response** or as a **streaming response** (token by token), making it suitable for chatbot UIs, text analysis, content generation, and more. The API supports custom context (to tailor the AI’s persona or knowledge), conversation memory, and tone control for flexible integration into various use cases.

{% hint style="info" %}
⚠️ **Single Endpoint Design:** The ChainGPT Web3 LLM API uses **one endpoint** for chat completions. There is **no separate `/chat` endpoint** for non-streaming responses. All requests should be sent to `POST https://api.chaingpt.org/chat/stream`. By default this returns a full JSON response. If you prefer a streamed answer, you simply adjust how you call the endpoint (e.g. using an HTTP stream or setting `responseType: "stream"` in your client). **Do not** send requests to `/chat` (doing so will result in a 404 Not Found).
{% endhint %}

***

### Authentication

All requests require an API key for authentication. Obtain your API key from the ChainGPT web app (AI Hub) – go to the **API Dashboard**, create a new secret key, and customize your key’s context (company info, token details, tone, etc.) if desired. **Keep this key secure.** Include the API key in your request headers:

```
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

Make sure to replace `YOUR_API_KEY` with the key you generated. Each API call deducts credits from your ChainGPT account (see **Credit Usage** below).

***

### Endpoint: `POST /chat/stream` – Chat Completion

Use this endpoint to submit a prompt or question to the AI and get a response. The behavior of the response depends on how you make the request:

* **Standard (Buffered) Response:** A normal POST request will wait for the AI to generate the complete answer, then return an HTTP 200 with the full JSON result.
* **Streaming Response:** If you prefer to receive the answer incrementally (for real-time display), you can open the request as a stream. The server will send back chunks of the answer text as they are generated, and terminate the stream when complete.

Regardless of response mode, you use the **same endpoint and request format**. The difference is purely client-side: streaming versus waiting for completion.

#### Request Body Parameters

All parameters are sent as a JSON object in the POST request body. Required fields must be present, while optional fields can be omitted to use their default behavior.

* **`model`** (`string`, **required**): ID of the model to use. For the general ChainGPT assistant, use `"general_assistant"`. _(At present this is the primary model available for this endpoint.)_
* **`question`** (`string`, **required**): The user's prompt or query for the AI. This is the message or instruction you want the assistant to respond to.
* **`chatHistory`** (`string`, optional, default `"off"`): Controls conversation memory. Set to `"on"` to enable the AI to remember this question and answer for future requests (allowing multi-turn conversations). If `"on"`, the API will store this Q\&A pair and include relevant history in subsequent answers. If `"off"`, no history is used or stored – the assistant treats each request independently.
* **`sdkUniqueId`** (`string`, optional): A unique session identifier for grouping chat history. If you have multiple distinct conversations or users, provide a consistent `sdkUniqueId` for each thread or user. When `chatHistory` is `"on"`, this ensures the assistant only uses history from the specified session. If omitted, all history-enabled requests under your API key share a single combined history context.
* **`useCustomContext`** (`boolean`, optional, default `false`): Whether to apply custom context to this request. If `false`, the assistant responds with its default behavior (or any default context configured for your API key in the AI Hub). If `true`, the assistant will incorporate additional context:
  * If you **do not** provide a `contextInjection` object, the server will use the **default context tied to your API key** (e.g. your saved company info and tone preferences in the AI Hub).
  * If you **do** provide a `contextInjection` object in the request, that data will **override** any default context for this response (the custom context applies only to this request).
* **`contextInjection`** (`object`, optional): An object containing custom context details to inject into the conversation. This allows you to dynamically override or supplement the AI’s knowledge and behavior. To use `contextInjection`, you **must** set `useCustomContext: true`. Any fields you include here will influence the assistant’s reply (fields omitted will use your API key’s defaults, or be ignored if no default exists). See **Context Injection Fields** below for all supported sub-fields.

A minimal request might include just the required `model` and `question`. More complex requests can turn on `chatHistory` or supply context/tone parameters as needed. **All fields should be in a single JSON object**. For example:

```
{
  "model": "general_assistant",
  "question": "How do Ethereum smart contracts work?",
  "chatHistory": "off"
}
```

This asks the AI a question without any custom context or history. In contrast:

```
{
  "model": "general_assistant",
  "question": "Tell me about our project.",
  "chatHistory": "on",
  "sdkUniqueId": "user-12345",
  "useCustomContext": true,
  "contextInjection": {
    "companyName": "Acme DeFi",
    "companyDescription": "Acme DeFi is a decentralized finance platform offering yield farming.",
    "cryptoToken": true,
    "tokenInformation": {
      "tokenName": "AcmeToken",
      "tokenSymbol": "ACME",
      "blockchain": ["ETHEREUM", "POLYGON"]
    },
    "aiTone": "PRE_SET_TONE",
    "selectedTone": "FRIENDLY"
  }
}
```

In this example, we enabled chat history (so the AI can recall prior Q\&A for `user-12345`), and we injected custom context about _Acme DeFi_ (company name/description and token info) and set the tone to a friendly preset. The assistant will incorporate all of that into its answer.

**Context Injection Fields**

When using `contextInjection`, you can provide any of the following fields to tailor the AI’s persona, knowledge base, or response style. All fields are optional – include only what’s relevant to your use case. (If a field is omitted, the system will fall back to the default context configured for your API key, if any, or simply not use that aspect.)

* **`companyName`** (`string`): Name of your company, project, or organization. The assistant will refer to this name when appropriate, and “speak” as a representative of this entity.
* **`companyDescription`** (`string`): A brief description of the company or project. This can include what the project does, its mission, domain, or any key details. The assistant can draw on this description to answer questions about the project’s purpose or background.
* **`companyWebsiteUrl`** (`string` – URL): The URL of the company/project’s website. The assistant might use this in answers (for example, providing the link to users asking for more info). Include the full URL with `https://`.
* **`whitePaperUrl`** (`string` – URL): URL to the project’s whitepaper or documentation. If provided, the assistant may reference it or suggest reading it for detailed technical info.
* **`purpose`** (`string`): The role or purpose of the AI chatbot from your perspective. For example, _“To assist users with support questions about our DeFi platform”_ or _“To educate users on our blockchain’s features.”_ This helps the AI understand its intended role, which can influence how it frames responses.
* **`cryptoToken`** (`boolean`): Set to `true` if your project has a cryptocurrency token and you want the AI to be able to provide information about it. When true, you should also provide the relevant details in the `tokenInformation` field. If `false` (or omitted), the assistant won’t proactively inject token details (though it may still answer general token questions).
* **`tokenInformation`** (`object`): Details about your crypto token (relevant if `cryptoToken: true`). This object can include:
  * **`tokenName`** (`string`): The name of the token (e.g. `"ChainGPT Token"`).
  * **`tokenSymbol`** (`string`): The token’s symbol or ticker (e.g. `"CGPT"`). You can include a `$` or other prefix if commonly used.
  * **`tokenAddress`** (`string`): The blockchain contract address of the token (on its primary network). This helps the assistant confirm the token’s existence or reference the contract.
  * **`tokenSourceCode`** (`string`): The source code of the token’s smart contract, or a URL to the source code repository. This could be used if technical details about the contract are asked.
  * **`tokenAuditUrl`** (`string` – URL): A link to any security audit report for the token’s contract (if available). This can be cited to reassure users about security.
  * **`explorerUrl`** (`string` – URL): A link to a block explorer page for the token’s contract (for example, an Etherscan URL). Useful if users want to verify on-chain information.
  * **`cmcUrl`** (`string` – URL): Link to the token’s page on CoinMarketCap, if applicable.
  * **`coingeckoUrl`** (`string` – URL): Link to the token’s page on CoinGecko, if applicable.
  * **`blockchain`** (`array[string]`): A list of blockchain networks relevant to the token. For example, if the token is deployed on Ethereum and Polygon, you might set `"blockchain": ["ETHEREUM", "POLYGON"]`. This helps the AI contextualize which chain’s data or standards to consider. _Supported network values include:_ `"ETHEREUM"`, `"BSC"` (Binance Smart Chain), `"ARBITRUM"`, `"BASE"`, `"BLAST"`, `"AVALANCHE"`, `"POLYGON"`, `"SCROLL"`, `"OPTIMISM"`, `"LINEA"`, `"ZKSYNC"`, `"POLYGON_ZKEVM"`, `"GNOSIS"`, `"FANTOM"`, `"MOONRIVER"`, `"MOONBEAM"`, `"BOBA"`, `"METIS"`, `"LISK"`, `"AURORA"`, `"SEI"`, `"IMMUTABLE_ZK"`, `"GRAVITY"`, `"TAIKO"`, `"CRONOS"`, `"FRAXTAL"`, `"ABSTRACT"`, `"WORLD_CHAIN"`, `"MANTLE"`, `"MODE"`, `"CELO"`, `"BERACHAIN"`. (Use all that apply; unrecognized values will be ignored.)
*   **`socialMediaUrls`** (`array<object>`): A list of social media or community links for your project. Each element in the array should be an object with a **`name`** and a **`url`**. For example:

    ```
    "socialMediaUrls": [
        { "name": "twitter",   "url": "https://twitter.com/YourProject" },
        { "name": "telegram",  "url": "https://t.me/YourProjectChannel" },
        { "name": "website",   "url": "https://yourproject.org/blog" }
    ]
    ```

    Include the platforms relevant to your users (e.g. Twitter, Telegram, LinkedIn, YouTube, Medium, etc.). The assistant might use these to answer questions like “Where can I follow your project updates?” by providing these links.
* **`limitation`** (`boolean`): A flag to enforce any special content limitations or strictness. By default this can be `false`. If `true`, it might indicate the assistant should be more cautious or avoid certain types of content (depending on how the backend interprets it). If `false`, the assistant just follows the general content guidelines and your provided context.
* **`aiTone`** (`string`): Controls the tone or style of the AI’s responses. Supported values are:
  * `"DEFAULT_TONE"` – Use the default tone (if you have a default set in your API key’s configuration, or the system’s default neutral tone). In this mode, you **do not** need to specify `selectedTone` or `customTone`.
  * `"CUSTOM_TONE"` – Define a completely custom tone in your own words. If you choose this, provide the desired style in the `customTone` field.
  * `"PRE_SET_TONE"` – Use one of the predefined tone profiles. If you choose this, specify which preset tone with the `selectedTone` field.
*   **`selectedTone`** (`string`): If `aiTone` is `"PRE_SET_TONE"`, use this field to pick the preset style you want. The ChainGPT assistant supports a range of preset tones. The value should be one of the following options:

    * **PROFESSIONAL**
    * **FRIENDLY**
    * **INFORMATIVE**
    * **FORMAL**
    * **CONVERSATIONAL**
    * **AUTHORITATIVE**
    * **PLAYFUL**
    * **INSPIRATIONAL**
    * **CONCISE**
    * **EMPATHETIC**
    * **ACADEMIC**
    * **NEUTRAL**
    * **SARCASTIC\_MEME\_STYLE**

    Choose the tone that best fits how you want the AI to sound. _(If **`aiTone`** is not **`"PRE_SET_TONE"`**, this field is ignored.)_
* **`customTone`** (`string`): If `aiTone` is `"CUSTOM_TONE"`, provide a brief description of the style or persona you want the AI to adopt. For example: _“Speak in a playful, casual manner using crypto slang”_ or _“Respond as a strict professor who uses technical language.”_ The assistant will adjust its writing style according to this description. _(Ignored if **`aiTone`** is not **`"CUSTOM_TONE"`**.)_

Using context injection can powerfully customize the AI’s behavior and knowledge base on a per-request basis. However, you generally only need to include fields that differ from your default API key settings or that are relevant to the current query. Overloading the request with too much extraneous context might confuse the model, so provide **concise, relevant details** for the best results.

#### Response Format

If the request is successful, the API will return an HTTP 200 response. The structure of the response depends on whether you requested a full JSON or a stream:

*   **Buffered (JSON) Response:** You will receive a JSON object once the AI has finished generating the answer. The typical response structure is:

    ```
    {
      "status": true,
      "message": "Chat response generated successfully.",
      "data": {
        "bot": "<assistant's answer text>"
      }
    }
    ```

    The `data.bot` field contains the assistant’s answer as a single string. (The property is named "bot" to denote the chatbot’s reply.) There may be additional metadata in the response (such as an internal `id` or timestamp for the interaction), but the main content of interest is the text under `data.bot`.
* **Streaming Response:** The server will begin sending back the answer in chunks over the open HTTP connection. Each chunk is a piece of the answer text (e.g. a few words or a sentence). The response uses standard HTTP chunked transfer encoding. You should keep the connection open and read from it until the stream ends. There is no enclosing JSON object in this mode; instead, you’ll receive raw text data progressively. Once the full answer has been sent, the server will terminate the stream (signaling the end of response).

In both cases, if an error occurs, you will receive an error status code and a JSON error message (except in some streaming error cases where the connection might drop). Common error responses:

* **400 Bad Request:** Missing or invalid fields in your request JSON.
* **401 Unauthorized:** API key missing or invalid.
* **402 Payment Required / 403 Forbidden:** Insufficient credits in your account (payment needed) or you are not allowed to access this resource.
* **404 Not Found:** Incorrect endpoint (e.g. using a non-existent route like `/chat` instead of `/chat/stream`).
* **429 Too Many Requests:** You’ve hit a rate limit.
* **5xx Server Error:** An issue on the server side (rare).

Always check the response status and the JSON body. On errors, the JSON will typically look like:

```
{
  "status": false,
  "message": "Error description here."
}
```

No `data` field will be present in error responses.

#### Credit Usage

Every API call consumes credits from your ChainGPT account:

* A basic request (no history) costs 0.5 **credits**.
* If `chatHistory` is `"on"`, the request costs 0.5 **credits** (0.5 base credit + 0.5 extra for the history storage/retrieval overhead).

Ensure your account has sufficient credits; otherwise, you will get an error (HTTP 402/403 indicating insufficient credits). You can top up and track credits in the ChainGPT web app dashboard. **Note:** Enabling conversation memory (history) incurs the extra credit charge because the system stores the conversation and uses it for context in future calls.

***

### Examples

Below are code examples demonstrating how to call the ChainGPT chat completion endpoint in different scenarios. These examples show both the default (buffered) usage and streaming usage, using Axios (Node.js) and cURL. You can adapt these to other HTTP clients or languages as needed.

#### Using Axios (Node.js) – Full JSON Response

In this example, we send a question and get the complete answer in one response. We’ll use Axios to make the POST request and simply log the returned answer. This is suitable for scenarios where you don't need real-time streaming updates.

```
import axios from 'axios';

async function getAnswer() {
  const API_URL = 'https://api.chaingpt.org/chat/stream';
  const API_KEY = process.env.CHAINGPT_API_KEY;  // Set your API key in an environment variable

  try {
    const response = await axios.post(API_URL, 
      {
        model: "general_assistant",
        question: "Explain the concept of AI",
        chatHistory: "off"
      },
      {
        headers: { Authorization: `Bearer ${API_KEY}` }
      }
    );
    // The full response is now in response.data
    console.log("Assistant answer:", response.data.data.bot);
  } catch (error) {
    console.error("API request failed:", error.response?.data || error.message);
  }
}

getAnswer();
```

**Explanation:** We set `chatHistory: "off"` to treat this as a single-turn query. The assistant’s answer is logged from `response.data.data.bot`. In a real application, you might handle the JSON further (e.g. displaying the answer in a UI).

#### Using Axios (Node.js) – Streaming Response

This example shows how to receive the answer as a stream using Axios. We configure Axios to expect a stream and then read chunks of data as they arrive, printing them out immediately. This creates an effect of the AI “typing” the answer in real-time.

```
import axios from 'axios';

async function streamAnswer() {
  const API_URL = 'https://api.chaingpt.org/chat/stream';
  const API_KEY = process.env.CHAINGPT_API_KEY;

  // Create an Axios instance configured for streaming
  const apiClient = axios.create({
    baseURL: API_URL,
    timeout: 60000,
    headers: { Authorization: `Bearer ${API_KEY}` },
    responseType: "stream"
  });

  try {
    const response = await apiClient.post("/", {
      model: "general_assistant",
      question: "Give me the latest news about Ethereum.",
      chatHistory: "off"
    });
    const stream = response.data;  // This is a Node.js readable stream
    stream.on("data", (chunk) => {
      const textChunk = chunk.toString();
      process.stdout.write(textChunk);  // Output the chunk (could also append to a variable or UI element)
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

**Explanation:** Here we create a dedicated Axios instance with `responseType: "stream"`. The POST request is made to the baseURL `/` (which corresponds to the full `https://api.chaingpt.org/chat/stream`). As chunks of the answer come in, we convert them to string and write to standard output (this could be replaced with updating a web UI in a real app). When the stream ends, we log a message. This way, the answer is displayed gradually, as if the AI is responding in real time.

#### Using cURL – Full Response (JSON)

You can test the API quickly with cURL. The following example makes a request for a complete response (non-streaming). cURL will wait for the response and then output it all at once:

```
curl -X POST "https://api.chaingpt.org/chat/stream" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "model": "general_assistant",
        "question": "Explain the concept of AI",
        "chatHistory": "off"
      }'
```

Make sure to insert your actual API key. This request asks the AI to explain AI, with no history. The response will be a JSON object containing the answer (under `data.bot`). For example, you might get:

```
{
  "status": true,
  "message": "Chat response generated successfully.",
  "data": {
    "bot": "Artificial Intelligence (AI) is a field of computer science focused on creating systems capable of performing tasks that typically require human intelligence. These tasks include learning from data, reasoning, problem-solving, perception, and understanding natural language. AI systems range from simple algorithms to complex neural networks and power applications like virtual assistants, recommendation engines, and autonomous vehicles."
  }
}
```

#### Using cURL – Streaming Response

To see streaming in action via cURL, you can use the same endpoint. The response will stream to your terminal, printing partial results as the AI generates them:

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

As the request runs, you should see the answer text start to appear in your terminal gradually. It may come in segments, and will complete when the curl command exits. If your terminal buffers the output and you don't see anything until the end, try running cURL with the `-N` / `--no-buffer` option to disable output buffering:

```
curl -N -X POST "https://api.chaingpt.org/chat/stream" ... (rest of command)
```

This ensures you see each chunk as soon as it arrives.

***

Using this single REST endpoint, developers can integrate ChainGPT’s Web3-savvy AI into many environments – from interactive chat interfaces that benefit from streaming, to batch scripts and backends that just need a quick JSON answer. The combination of **chat history** and **context injection** features allows you to build anything from persistent conversational agents to specialized Q\&A bots (for your documentation, DeFi app, NFT project, etc.), or even use the model for tasks like text summarization or classification with the proper prompt.&#x20;

By following this reference and using the provided examples as a starting point, you can seamlessly bring ChainGPT's capabilities into your application. Enjoy building with ChainGPT Web3 LLM, and refer to this guide whenever you need details on request format or features!
