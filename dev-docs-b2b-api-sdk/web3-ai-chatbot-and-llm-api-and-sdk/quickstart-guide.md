# QuickStart Guide

## ChainGPT Web3 AI Chatbot & LLM - QuickStart Guide

**Overview:** This QuickStart guide will help you integrate ChainGPT’s Web3 AI Chatbot (LLM) into your project, either via the REST API or using the official JavaScript SDK. We’ll walk through installation, authentication, sending your first prompt (including streaming responses), managing conversation history, injecting custom context/tone, and best practices for security and usage. The guide is designed for both experienced and new developers, with clear steps and code examples to get you up and running quickly.

***

### Prerequisites

* **ChainGPT Account & API Key:** Sign up for a ChainGPT account and generate an API key from the API Dashboard in the ChainGPT web app. Ensure your account has sufficient credits (CGPTc) since the API is credit-based – each request will deduct credits from your balance. If you run out of credits, API calls will return an error and won’t succeed.
* **Environment:**
  * For **REST API** integration, you’ll need a tool or library to make HTTPS requests (for example, **cURL**, Postman, or an HTTP client in your programming language).
  * For **JavaScript SDK** integration, you need a Node.js environment (Node.js installed). The SDK is available via npm for server-side or backend JS use.

{% hint style="info" %}
**Tip:** Before proceeding, copy your API key and store it securely (e.g., in an environment variable or a secrets vault). You’ll need this key for all API or SDK calls.
{% endhint %}

{% hint style="info" %}
**Warning:** **Never expose your API key publicly.** Don’t hard-code it in client-side code or commit it to a repo. Treat it like a password. If someone obtains your key, they could consume your credits. Always load it from a secure location (environment variables, config files not in source control, etc.).
{% endhint %}

With these in place, you’re ready to integrate ChainGPT. The following sections provide two integration paths – via REST API and via the Node.js SDK – you can choose whichever fits your use case.

***

### QuickStart via REST API

If you prefer direct HTTP calls or are using a language without an official SDK, the REST API is the simplest way to interact with ChainGPT’s LLM. All functionality (prompting the AI, streaming responses, chat history, etc.) is exposed via HTTPS endpoints.

#### 1. Authentication and Endpoint

All ChainGPT API requests require authentication using your API key. Include the key in an HTTP header:

```
Authorization: Bearer YOUR_API_KEY
```

This should be added to every request. The base URL for the API is `https://api.chaingpt.org`, and the primary endpoint for chat is `POST /chat` (for non-streaming responses). We’ll use `curl` in examples, but you can use any HTTP client.

{% hint style="info" %}
**Note:** In examples below, replace `YOUR_API_KEY` with your actual key. Also ensure you set the `Content-Type: application/json` header when sending JSON bodies.
{% endhint %}

#### 2. Sending Your First Prompt (Single-shot Response)

To get an AI-generated answer, you send an HTTP POST request to the `/chat` endpoint with a JSON body containing at least two fields: `model` and `question`. The model ID for ChainGPT’s general Web3 chatbot is `"general_assistant"`. The `question` is your prompt or user query. Here’s a minimal example using curl:

```bash
curl -X POST "https://api.chaingpt.org/chat" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "general_assistant",
    "question": "Hello, ChainGPT! Can you give me a quick intro of yourself?",
    "chatHistory": "off"
  }'
```

In this example:

* **model** is set to `"general_assistant"` (required for now, as this is the primary model).
* **question** is a simple greeting to the chatbot.
* **chatHistory** is set to `"off"` for a stateless single-turn query (more on this later). By default, if you omit this field, it’s treated as `"off"`.

**Response:** The API will return a JSON response with the assistant’s reply. A successful response (HTTP 200) has a structure like:

```json
{
  "status": true,
  "message": "Chat response generated successfully.",
  "data": {
    "bot": "Hello! I’m ChainGPT, a Web3-savvy AI assistant. I can help with crypto and blockchain questions in real-time."
  }
}
```

The answer text is contained in `data.bot`. In our example, you’d get a friendly introduction from ChainGPT. At this point, you’ve made your first successful API call to ChainGPT!

{% hint style="info" %}
**Tip:** If you receive an error, check the HTTP status and message:

* **401 Unauthorized:** Your API key is missing or invalid. Double-check the `Authorization` header.
* **402/403 Payment Required/Forbidden:** You might be out of credits or not have access. Top up your credits in the ChainGPT dashboard.
* **400 Bad Request:** There’s an issue with your request JSON (e.g., missing required fields). Ensure `model` and `question` are present.
{% endhint %}

#### 3. Streaming Responses for Real-Time Output

For a better user experience (especially for longer answers or chat-like UIs), ChainGPT can stream its response token-by-token. This allows your application to start displaying the answer as it’s being generated.

To use streaming, call the `/chat/stream` endpoint with the same parameters. The only difference is the URL. For example:

```bash
curl -N -X POST "https://api.chaingpt.org/chat/stream" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "general_assistant",
    "question": "Give me the latest on Ethereum's market status.",
    "chatHistory": "off"
  }'
```

Here we used `curl -N` (no-buffering) to ensure we see the streamed chunks in real time. The response will come as a series of partial data chunks (likely in a chunked HTTP response format). Each chunk will contain a piece of the `"bot"` message as it’s generated. You should handle this in your client by concatenating or processing chunks as they arrive.

For example, you might see something like:

```
{"status":true,"message":"Streaming response","data":{"bot":"Ethereum's current price is $1,800"}}
{"status":true,"message":"Streaming response","data":{"bot":"Ethereum's current price is $1,800 and its 24h volume has increased"}}
{"status":true,"message":"Streaming response","data":{"bot":"Ethereum's current price is $1,800 and its 24h volume has increased by 10%."}}
```

Each chunk extends the `bot` text. Once the stream ends (indicated by an end-of-response chunk or closing of connection), you have the full answer. In a real implementation, you’d parse these incremental JSON chunks and update your UI or logs accordingly.

{% hint style="info" %}
**Note:** The exact format of stream chunks may vary (it could be raw text or JSON lines). Always refer to the latest API documentation for how to parse streaming responses. ChainGPT’s streaming API is designed to be similar to other AI streaming APIs (like OpenAI’s) for familiarity.
{% endhint %}

#### 4. Maintaining Conversation State (Chat History)

One of the powerful features of ChainGPT is multi-turn conversations: the AI can remember previous questions and answers to provide contextual responses. To enable this, use the `chatHistory` flag and a session identifier:

* Include `"chatHistory": "on"` in your request JSON to tell the system to **remember this interaction for future queries**.
* Provide a `"sdkUniqueId"` field as a unique session or user ID (any string, like a UUID or username) to isolate the conversation thread. If you have multiple users or separate sessions, **use different `sdkUniqueId` values for each**. This ensures each user’s chat history is kept separate.

For example, say we want to have a conversation with the AI:

**First question (start conversation):**

```bash
curl -X POST "https://api.chaingpt.org/chat" \
  -H "Authorization: Bearer YOUR_API_KEY" -H "Content-Type: application/json" \
  -d '{
    "model": "general_assistant",
    "question": "Hi! I have a token called ABC. Can you remember that my project name is ABC and what it does?",
    "chatHistory": "on",
    "sdkUniqueId": "session123"
  }'
```

_(ChainGPT might respond acknowledging the token and project details.)_

**Follow-up question (same session):**

```bash
curl -X POST "https://api.chaingpt.org/chat" \
  -H "Authorization: Bearer YOUR_API_KEY" -H "Content-Type: application/json" \
  -d '{
    "model": "general_assistant",
    "question": "Now, given that context, what’s a good elevator pitch for ABC?",
    "chatHistory": "on",
    "sdkUniqueId": "session123"
  }'
```

On the second call, because we used the same `sdkUniqueId` and left history on, the AI knows you already told it about “ABC” in the prior query. It will use that context to craft a better answer (in this case, an elevator pitch for the ABC project).

**How it works:** When `chatHistory` is `"on"`, the backend stores the Q\&A pair and will include previous Q\&As in the context for the next request (for that same `sdkUniqueId`). This enables a continuous conversation. If `sdkUniqueId` is not provided, all history-enabled requests under your account would share one giant history, which is usually **not** what you want. So always set a unique ID per user or chat session when using history.

{% hint style="info" %}
**Tip:** There’s also an API endpoint to fetch past chat transcripts if needed (for example, to display chat history to the user or to audit conversations). This is beyond the quickstart scope, but know that turning on `chatHistory` not only influences the AI’s memory, but also allows retrieving the conversation later via the API.
{% endhint %}

#### 5. Injecting Custom Context and Setting the AI’s Tone

ChainGPT allows you to inject **custom context or persona** into the AI on a per-request basis, as well as to adjust the **tone/style** of responses. This is useful to tailor the AI to your application (e.g. making it speak as if it’s your company’s assistant, or limiting its knowledge to a certain document).

There are two mechanisms for context:

* **Default Context (AI Hub configuration):** You can configure your API key’s default context in the ChainGPT AI Hub dashboard (e.g. set your company name, project info, or upload a knowledge base). If you’ve done that, you can enable `useCustomContext: true` without sending a `contextInjection` object, and the AI will automatically pull that preset context for the conversation.
* **On-the-fly Context Injection:** You can send a `contextInjection` object in your API request to dynamically provide context or override the defaults. For example, you might include details about a particular token, or a snippet of documentation, or specify the tone.

To use either, set `"useCustomContext": true` in your request. If you include a `contextInjection` object, its fields will **override or supplement** any default context tied to your API key.

**Example – Setting a Friendly Tone and Injecting Project Info:**

```bash
curl -X POST "https://api.chaingpt.org/chat" \
  -H "Authorization: Bearer YOUR_API_KEY" -H "Content-Type: application/json" \
  -d '{
    "model": "general_assistant",
    "question": "What can you tell users about our project?",
    "chatHistory": "off",
    "useCustomContext": true,
    "contextInjection": {
      "companyName": "ABC Crypto",
      "companyDescription": "ABC is a DeFi platform for earning yield on crypto.",
      "purpose": "To assist users with questions about ABC Crypto.",
      "aiTone": "PRE_SET_TONE",
      "selectedTone": "FRIENDLY"
    }
  }'
```

In this API call, we are injecting some custom context:

* We tell the AI our company is “ABC Crypto” and what it does (DeFi platform).
* We specify the **purpose** of the AI (act as an assistant for ABC Crypto users).
* We set the tone to “friendly” by using a pre-defined friendly tone.

The AI will incorporate this context when formulating its answer – it will likely respond as if it is a helpful representative of ABC Crypto, with a friendly tone (e.g. using approachable language). This demonstrates how context injection and tone work together: you can inform the AI about domain-specific facts and control the style of the response.

{% hint style="info" %}
**Note:** The `contextInjection` object has many optional fields (company info, links, token details, etc., as well as `aiTone` settings for style). You do **not** need to fill all of them – provide only what’s relevant. For instance, to use a custom tone not covered by presets, you could set `"aiTone": "CUSTOM_TONE"` and provide your own style instructions in a `customTone` field. Refer to the documentation for the full list of fields and preset tone options.
{% endhint %}

#### 6. API Best Practices

* **Secure Your API Key:** As mentioned earlier, never expose your key on the client side. If you’re calling the ChainGPT API from a web app, route the requests through your backend or use a proxy so the key remains hidden. Rotate the key if you suspect it’s been compromised.
* **Monitor Credit Usage:** Keep an eye on your credit balance. Each chat request costs a certain number of credits (e.g. 0.5 credits per request, plus extra if using history). You can check your usage and top up credits in the ChainGPT dashboard. Build in alerts or checks in your app if you approach your credit limit to avoid downtime.
* **Error Handling & Retries:** Handle API errors gracefully. For example, if you get a network error or a 500 response, you might retry the request after a brief delay. If you get a 402/403 indicating no credits, inform the team or user accordingly rather than retrying indefinitely.
* **Timeouts:** The complexity of the question will affect response time. Set a reasonable timeout for your HTTP requests (the API should handle many requests quickly, but very long analyses might take a bit longer). In streaming mode, you’ll keep the connection open until the server signals completion.
* **Test with Simple Queries:** If you’re not getting the responses you expect, test the API with a simple prompt (like "Hello" or "What is Bitcoin?") without any custom context. This helps isolate whether issues are with context injection or general usage.

***

### QuickStart via JavaScript SDK (Node.js)

If you are building a Node.js application, ChainGPT provides an official JavaScript/TypeScript SDK that wraps the REST API. The SDK simplifies streaming and helps manage chat history behind the scenes. Let’s get you set up with the SDK and make a request.

#### 1. Installation

The SDK is available as an npm package. Install it into your Node.js project using npm or Yarn:

```bash
npm install --save @chaingpt/generalchat

# OR with Yarn:
yarn add @chaingpt/generalchat
```

This will add the ChainGPT General Chat SDK to your project. (Note: The package name is `@chaingpt/generalchat`. It was previously referred to as the “General Chatbot SDK” in some places.)

#### 2. Import and Initialize the SDK

After installing, import the SDK and initialize it with your API key:

```js
import { GeneralChat } from '@chaingpt/generalchat';

const generalChat = new GeneralChat({
  apiKey: process.env.CHAINGPT_API_KEY  // Load your API key from an env variable
});
```

This creates a `generalChat` client instance. Make sure `CHAINGPT_API_KEY` is set in your environment (e.g., in a `.env` file or your server config). The API key is the only required configuration for initialization. The SDK will automatically include this key in all requests for you. If the key is missing or invalid, you'll get an authentication error when you call the SDK methods.

{% hint style="info" %}
**Warning:** Just like with direct API use, never hard-code the key in your code. Use environment variables and do not expose the key in front-end code. The SDK is meant for server-side use.
{% endhint %}

#### 3. Sending a Prompt (Non-Streaming Mode)

The SDK provides a method `createChatBlob()` to get a full response in one go (non-streaming, analogous to the POST `/chat` endpoint). It returns a **Promise** that resolves to the response object once the AI has completed its answer.

Here’s how to send your first prompt using the SDK:

```js
// Example: Ask a question and get the full response
async function askQuestion() {
  try {
    const response = await generalChat.createChatBlob({
      question: "Hello, ChainGPT! Can you provide a quick overview of yourself?",
      chatHistory: "off"  // no history for this single question
    });
    console.log(response.data.bot);
  } catch (err) {
    console.error("Error from ChainGPT:", err);
  }
}

askQuestion();
```

When you run this function, it will print the assistant’s answer to the console. The `response` object structure mirrors the API: `response.data.bot` contains the answer text. For example, it might log something like:

```
Hello! I’m ChainGPT, a blockchain-savvy AI. I specialize in crypto and Web3 questions, pulling in real-time data to help you.
```

Some notes on using `createChatBlob`:

* The `question` field is required. It’s the prompt or user query.
* We passed `chatHistory: "off"` here explicitly for clarity, but by default the SDK will assume no history unless you turn it on.
* The call is `await`ed because it’s asynchronous. Always use `await` or `.then()` to get the result.
* If something goes wrong (bad API key, no credits, etc.), the SDK will throw an error (specifically a `GeneralChatError`). Our try/catch will catch it and log it. In production, handle errors appropriately (perhaps retry or send an error response to your user).

#### 4. Streaming Mode (Real-time Response)

For streaming responses (token-by-token output), the SDK offers `createChatStream()`. This method returns a Node.js **readable stream** that yields chunks of the AI’s answer as they arrive. You can consume this stream to provide real-time feedback in your application.

Here’s an example of using `createChatStream`:

```js
// Example: Streaming a response
async function streamAnswer() {
  try {
    const stream = await generalChat.createChatStream({
      question: "Explain the concept of yield farming in DeFi.",
      chatHistory: "off"
    });
    // The stream is a Node.js readable stream.
    stream.on('data', (chunk) => {
      // Each chunk is a Buffer or string of partial response text
      process.stdout.write(chunk.toString());
    });
    stream.on('end', () => {
      console.log("\n[Stream ended]");
    });
  } catch (err) {
    console.error("Streaming error:", err);
  }
}

streamAnswer();
```

When you run `streamAnswer()`, you should start seeing partial output printed to your console immediately as the answer is being generated (for example, it might print "Yield farming is a process in DeFi where..." then continue). The stream will emit a `'data'` event for each chunk of the answer, and an `'end'` event when the answer is complete (at which point we print a newline and “\[Stream ended]”).

This streaming approach is ideal for building chat interfaces or any UX where you want the answer to appear gradually, giving a smoother experience to the user.

{% hint style="info" %}
**Tip:** If you prefer, you can also use asynchronous iteration (`for await (const chunk of stream) { ... }`) to read from the stream. Under the hood, the SDK’s streaming uses the same `/chat/stream` API endpoint, so you get the same data just in a Node-friendly way.
{% endhint %}

#### 5. Multi-turn Conversations with the SDK (Chat History)

Just like the REST API, the SDK supports conversation history so the AI can remember previous interactions. To use it, you will include the same two fields in your SDK calls: `chatHistory` and `sdkUniqueId`.

When calling `createChatBlob` or `createChatStream` via the SDK:

* Set `chatHistory: "on"` to enable memory for that call.
* Provide a consistent `sdkUniqueId` string to identify the session or user.

For example:

```js
// Continuation of a conversation using the SDK
await generalChat.createChatBlob({
  question: "What's the current price of Bitcoin?",
  chatHistory: "on",
  sdkUniqueId: "user_42"
});
// ... (later)
await generalChat.createChatBlob({
  question: "How about its price last week versus today?",
  chatHistory: "on",
  sdkUniqueId: "user_42"
});
```

On the second call, the AI will remember you asked about Bitcoin’s current price already, and can use that context to compare with last week without you re-specifying the subject. The `sdkUniqueId: "user_42"` ties those two calls together in one ongoing conversation thread.

If you had another user or a separate conversation, you’d use a different ID (e.g. `"user_43"`). This ensures each session’s history is isolated. If you forget to set an `sdkUniqueId` while using history, all your history-enabled calls with that API key would share one mixed conversation, which can lead to confusing AI answers.

The SDK also provides a helper to retrieve chat history (`getChatHistory`) if needed, but for quickstart purposes, simply know that setting `chatHistory: "on"` makes the model stateful, and using a unique ID keeps each conversation separate.

#### 6. Context Injection and Tone via SDK

The SDK fully supports custom context injection and tone control, similar to the API. You’ll use the `useCustomContext` flag and a `contextInjection` object in the parameters.

For instance, using the same example as earlier (injecting company info and friendly tone), but now in the SDK:

```js
const response = await generalChat.createChatBlob({
  question: "What can you tell users about our project?",
  chatHistory: "off",
  useCustomContext: true,
  contextInjection: {
    companyName: "ABC Crypto",
    companyDescription: "ABC is a DeFi platform for earning yield on crypto.",
    purpose: "To assist users with questions about ABC Crypto.",
    aiTone: "PRE_SET_TONE",
    selectedTone: "FRIENDLY"
  }
});
console.log(response.data.bot);
```

The result will be similar – the SDK passes these parameters to the API and you’ll get a response tailored to the provided context and tone. The benefit of the SDK here is purely convenience; the behavior and fields are exactly the same as described in the REST section. Just remember to set `useCustomContext: true` whenever you include a `contextInjection` object, otherwise the context will be ignored.

You can also configure your API key’s default context in the AI Hub and simply use `useCustomContext: true` without providing `contextInjection` each time, if you want all responses to have your predefined context. Decide what works best for your application (pre-configure once vs. send context dynamically per request).

#### 7. Additional SDK Tips and Best Practices

* **Async Behavior:** All SDK methods (`createChatBlob`, `createChatStream`, etc.) are asynchronous. Make sure to use `await` or handle Promises. If you call them without await, you’ll need to manage the Promise or stream events.
* **Error Handling:** The SDK will throw exceptions of type `GeneralChatError` (accessible via `import { Errors } from '@chaingpt/generalchat';`) for HTTP errors or issues like invalid parameters. You can check `error.message` or use the error’s HTTP status to handle specific cases (e.g. 401 for auth issues, 402 for out-of-credit). Wrap calls in try/catch as shown to catch these.
* **Streaming Cleanup:** Always handle the `'end'` and `'error'` events on the stream. In Node, if you’re piping the stream to a writable (like an HTTP response to a client), make sure to properly end the response when the stream ends.
* **Efficiency:** If you have multiple prompts to send in parallel, you can reuse the same `generalChat` instance. The SDK is thread-safe for multiple requests. Just be mindful of your rate limits or credit usage when firing off many requests.
* **Stay Updated:** The SDK may receive updates. Check the ChainGPT documentation or npm for any new features (for example, support for additional model IDs or new parameters in the future).

***

### Next Steps and Recommended Use Cases

Congratulations! You have successfully sent prompts to ChainGPT and received responses, both via direct API calls and the JavaScript SDK. From here, you can integrate this powerful Web3 AI into various applications. Here are some ideas and best-fit use cases:

* **Building a Chatbot or Support Assistant:** ChainGPT can serve as the AI brain behind a chatbot on your website or within a crypto app, answering user questions about your product or the market. Because it’s Web3-aware, it’s great for crypto support bots or community assistants (for example, a Discord bot that answers questions about your DeFi platform). You can integrate via the API on your server or use the SDK if your bot runs in a Node environment. ChainGPT’s ability to handle live data and context means your chatbot could fetch real-time prices, check wallet info, or explain on-chain events when users ask.
* **Web3 Agents and Automation:** Leverage ChainGPT in backend services that monitor blockchain activity or execute automated tasks. For instance, you could create an **autonomous agent** that uses ChainGPT to interpret on-chain events and trigger actions. With ChainGPT’s contextual understanding of blockchain data, an agent could analyze transactions or market changes and make decisions (like alerting you of unusual activities or even formulating blockchain transactions for you to review). The SDK can be particularly handy here, letting your Node.js agent continuously feed information to ChainGPT and get analyses in real-time. ChainGPT is designed to integrate with such agent frameworks, extending its capabilities beyond simple Q\&A.
* **Real-Time Analytics Dashboards:** If you’re building a dashboard (for trading, NFTs, DeFi, etc.), ChainGPT can be embedded to provide explanations and insights on the data shown. For example, on a price dashboard, the AI could answer “Why is token X up today?” by pulling in on-chain and off-chain data to give context. With streaming, you could have it narrate changes in real-time. Because the model can access live data and your injected context, it can act like an on-demand analyst within your application. Use the API to query ChainGPT when users click an “Ask the AI” button next to a chart, for instance.
* **Custom Knowledge Base Q\&A:** Inject your own documents or data (using context injection) to make ChainGPT an expert on your specific domain. This could be internal docs, blockchain research, or support FAQs. The quickstart examples only touched on this, but you can dynamically feed the model info it doesn’t know by default. This is powerful for enterprise use-cases where ChainGPT becomes a personalized assistant for your team, combining general Web3 knowledge with your proprietary data.

{% hint style="info" %}
As you build, keep in mind the best practices we covered: secure your API keys, monitor credits, and take advantage of the customization features to tailor the AI to your needs. For deeper details on advanced features, error codes, or to explore specialized endpoints (like chat history retrieval, credit balance checks, etc.), refer to the official ChainGPT documentation and API reference.
{% endhint %}

We hope this guide has made it easy to get started with ChainGPT. Happy coding, and we look forward to seeing what you build with a Web3-savvy AI at your fingertips!
