# QuickStart Guide

### ChainGPT Web3 AI Chatbot & LLM – QuickStart Guide (Tech‑Team‑Verified)

This short guide shows you how to call ChainGPT’s Web3 LLM in **under five minutes**—either with plain HTTPS or the official JavaScript SDK. Everything here mirrors the **internal Tech‑Team reference**, not the older public docs.

***

### 1. Prerequisites ✔︎

| What you need                  | Notes                                                                                         |
| ------------------------------ | --------------------------------------------------------------------------------------------- |
| **ChainGPT account + API key** | Create a key in **AI Hub → API Dashboard** and copy it once.                                  |
| **Credits**                    | Each request = **0.5 credits**.Turning `chatHistory` **on** adds **+0.5 credits** per call    |
| **HTTP client**                | cURL, Postman, `fetch`, `axios`, etc.                                                         |
| **(If using SDK)** Node ≥ 14   | Install from npm.                                                                             |
| **Secure key storage**         | Put the key in an env var, secret manager, or server config—**never** ship it to the browser. |

{% hint style="info" %}
**Tip**  export `CHAINGPT_API_KEY="sk‑***"` so the examples just work.
{% endhint %}

***

### 2. Integration Options

| Option             | When to choose               | How it works                                                                             |
| ------------------ | ---------------------------- | ---------------------------------------------------------------------------------------- |
| **REST API**       | Any language / server        | `POST https://api.chaingpt.org/chat/stream` (single endpoint for blob **and** streaming) |
| **JavaScript SDK** | Node or a build‑step web app | `npm install @chaingpt/generalchat` → call `createChatBlob()` or `createChatStream()`    |

***

### 3. QuickStart via REST API

#### **3.1 Authentication & Endpoint**

```bash
Authorization: Bearer $CHAINGPT_API_KEY
Content-Type: application/json
POST https://api.chaingpt.org/chat/stream
```

{% hint style="info" %}
There is **no separate `/chat` endpoint**—all chat traffic goes to `/chat/stream`.
{% endhint %}

#### **3.2 Single‑Shot (JSON “blob”) response**

```bash
curl -X POST https://api.chaingpt.org/chat/stream \
  -H "Authorization: Bearer $CHAINGPT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "model":"general_assistant",
        "question":"Hello, ChainGPT! Who are you?",
        "chatHistory":"off"
      }'
```

If you don’t treat the response as a stream, cURL (or your HTTP client) waits until the LLM finishes, then returns one JSON:

```json
{ "status":true,
  "data": { "bot": "Hello! I’m ChainGPT, a Web3‑savvy AI assistant…" } }
```

#### **3.3 Streaming in real time**

```bash
curl -N -X POST https://api.chaingpt.org/chat/stream \
  -H "Authorization: Bearer $CHAINGPT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "model":"general_assistant",
        "question":"Give me the latest ETH stats.",
        "chatHistory":"off"
      }'
```

`-N` turns off buffering so you see partial chunks instantly. Concatenate chunks on arrival until the connection closes.

#### **3.4 Conversation Memory (chat history)**

```bash
curl -X POST https://api.chaingpt.org/chat/stream \
  -H "Authorization: Bearer $CHAINGPT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "model":"general_assistant",
        "question":"Remember my token ABC and what it does.",
        "chatHistory":"on",
        "sdkUniqueId":"user42"
      }'
```

Subsequent calls with the **same** `sdkUniqueId` and `chatHistory:"on"` include the prior Q\&A so the bot answers in context.

_Cost impact:_ `chatHistory:"on"` consumes **+0.5 credits** per request.

#### **3.5 Custom Context & Tone**

```bash
{
  "question":"Explain our project.",
  "useCustomContext":true,
  "contextInjection":{
     "companyName":"ABC Crypto",
     "companyDescription":"A DeFi yield platform",
     "aiTone":"PRE_SET_TONE",
     "selectedTone":"FRIENDLY"
  }
}
```

Set `useCustomContext:true`; include only the fields you need. Omit `contextInjection` to fall back to the default context configured in AI Hub.

***

### 4. QuickStart via JavaScript SDK

#### **4.1 Install & init**

```bash
npm install @chaingpt/generalchat

# or 
yarn add @chaingpt/generalchat
```

```js
import { GeneralChat } from "@chaingpt/generalchat";
const chat = new GeneralChat({ apiKey: process.env.CHAINGPT_API_KEY });
```

#### **4.2 Blob response**

```js
const res = await chat.createChatBlob({
  question: "Hi, what is ChainGPT?",
  chatHistory: "off"
});
console.log(res.data.bot);
```

`createChatBlob()` buffers the `/chat/stream` response for you.

#### **4.3 Streaming response**

```js
const stream = await chat.createChatStream({
  question: "Summarise the last BTC block.",
  chatHistory: "off"
});
for await (const chunk of stream) process.stdout.write(chunk);
```

#### **4.4 Chat history & context**

```js
await chat.createChatBlob({
  question: "Track ABC token price.",
  chatHistory: "on",
  sdkUniqueId: "user42",
  useCustomContext: true
});
```

SDK parameters match the REST fields 1‑for‑1.

***

### 5. Error Codes — Quick Reference

| Code          | Meaning                  | Fix                            |
| ------------- | ------------------------ | ------------------------------ |
| **401**       | Missing / bad API key    | Check `Authorization` header.  |
| **402 / 403** | Out of credits           | Top‑up in AI Hub.              |
| **400**       | Bad JSON / missing field | Verify `model` and `question`. |
| **5xx**       | Server error             | Retry after brief delay.       |

***

### 6. Best Practices

* **Hide the key.** Never embed it in client JavaScript—proxy via your backend.
* **Watch credits.** One call = **0.5 credits**; history = +0.5. Set alerts.
* **Use unique `sdkUniqueId`.** One per user/session keeps histories clean.
* **Handle stream ‘end’ & ‘error’.** Always close or retry gracefully.
* **Stay updated.** Check the npm changelog for new SDK features or model IDs.

***

You’re now ready to integrate ChainGPT using the exact endpoints and parameters defined by the Tech Team. For deeper dives, see the full [**API Reference**](api-reference.md) or [**SDK docs**](sdk-reference.md). Happy building!
