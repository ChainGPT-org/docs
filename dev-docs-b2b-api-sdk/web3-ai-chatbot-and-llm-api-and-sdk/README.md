# Web3 AI Chatbot & LLM (API & SDK)

## ChainGPT Web3 AI Chatbot & LLM (API & SDK)  - Overview

### **A Web3-native AI Chatbot built for the crypto world.**

**ChainGPT’s Web3 AI Chatbot & LLM** combines a powerful Large Language Model (LLM) with deep blockchain expertise — enabling seamless integration of crypto-native AI into your platform. It’s trained on smart contracts, DeFi, NFTs, DAOs, and real-time market data, making it ideal for use cases like support, analytics, trading assistance, and community engagement.

Already trusted by leading platforms like **Binance Square**, **BNB Chain**, and **TronDAO**, ChainGPT’s LLM is purpose-built for the next generation of Web3 applications.

{% hint style="info" %}
**Pricing:** each API or SDK request = 0.5 credits / ($0.005).&#x20;
{% endhint %}

***

### Key Differentiators

<table><thead><tr><th width="144.55859375">Title</th><th>What It Does</th></tr></thead><tbody><tr><td><strong>Web3 Domain Expertise</strong></td><td>Understands the Crypto market, smart contracts, DeFi, NFTs, tokenomics. Fine-tuned on blockchain data, unlike general LLMs. Great for explaining and analyzing crypto concepts/code, or market research.</td></tr><tr><td><strong>Real-Time Insights</strong></td><td>Pulls live on-chain + market data. Supports use cases like price checks, explorer queries, trend tracking. Keeps responses always up-to-date.</td></tr><tr><td><strong>Crypto-Specific Utilities</strong></td><td>Built-in tools for smart contract audits, generation, NFT creation, and TA — replacing separate dev tools.</td></tr><tr><td><strong>Multi-Channel Deployment</strong></td><td>Use the bot in websites, dApps, Discord, Telegram, and more — a consistent AI assistant across channels.</td></tr><tr><td><strong>Security &#x26; Privacy First</strong></td><td><p>Data is encrypted and isolated. ChainGPT doesn’t train its models on your data. Enterprise-safe, privacy-respecting, </p><p>production-ready.</p></td></tr></tbody></table>

**Learn more about the unique capabilities of ChainGPT's Web3 LLM:**

{% content-ref url="unique-capabilities.md" %}
[unique-capabilities.md](unique-capabilities.md)
{% endcontent-ref %}

***

### Architecture & Integration Flow

Integrating the ChainGPT Web3 AI Chatbot into your tech stack is straightforward. Your application (frontend and/or backend) communicates with ChainGPT’s cloud-hosted AI service via REST API calls or through ChainGPT’s API. At a high level, the flow looks like this:

<figure><img src="../../.gitbook/assets/image (44).png" alt="" width="375"><figcaption></figcaption></figure>

1. **User question:** A user interacts with your interface (website, mobile app, dApp, chat widget) and asks a question or issues a command (e.g. _“What’s the current ETH price?”_ or \*“Explain this smart contract code").
2. **Application request:** Your app (client or backend) sends the user’s query to ChainGPT’s service, either via a direct HTTPS API call or using the ChainGPT SDK. The request includes your API key for authentication and can optionally include parameters like a specific AI tone or extra context (see **Customization** via AI Hub).
3. **AI processing:** ChainGPT’s cloud servers receive the request and the Web3 LLM processes it. The model uses its built-in crypto knowledge base (and can fetch live on-chain or market data if needed) along with any custom context you’ve provided. It then generates a response – answering the question or performing the requested action.
4. **Response delivery:** The answer is returned back through the API/SDK to your application. If you use the SDK, it will handle parsing the response (and even streaming tokens) for you; if you use the raw API, you’ll receive a JSON response. Your app then displays the answer to the user (e.g. showing the chatbot reply in the UI).
5. **Multi-turn conversation:** The chatbot supports follow-up questions. Your app can send subsequent queries along with a conversation ID or previous chat messages so ChainGPT knows the context (The SDK makes this easy by managing conversation state for you.) This way, the user can have a natural back-and-forth dialogue with the AI.

{% hint style="info" %}
_Note:_ ChainGPT’s cloud handles the heavy AI computation (GPU model inference), so your integration stays lightweight. You don’t need any AI infrastructure on your side – your app just sends requests and receives answers, which keeps it scalable and efficient.
{% endhint %}

***

### Integration Options: REST API vs. JavaScript SDK

You can integrate ChainGPT via a RESTful web API or with the official JavaScript/TypeScript SDK – both provide the same Web3 AI capabilities, but with different developer experience.&#x20;

{% content-ref url="api-reference.md" %}
[api-reference.md](api-reference.md)
{% endcontent-ref %}

{% content-ref url="sdk-reference.md" %}
[sdk-reference.md](sdk-reference.md)
{% endcontent-ref %}

**The table below compares these options:**

<table><thead><tr><th width="98.703125">Aspect</th><th>REST API (HTTP)</th><th>ChainGPT JS/TS SDK</th></tr></thead><tbody><tr><td><strong>Platform</strong></td><td>Works with any environment that can send HTTPS requests (cURL, Python, Java, Go, etc) Great for server-side integration or non-JS languages.</td><td>Designed for JavaScript/TypeScript projects (Node.js, web browsers, React, etc). Distributed via NPM for easy inclusion in Node and front-end apps.</td></tr><tr><td><strong>Setup</strong></td><td>No additional library – simply make HTTP calls to ChainGPT’s API endpoints. You handle HTTP headers (e.g. <code>Authorization: Bearer &#x3C;API_KEY></code>) and parse JSON responses manual.</td><td>Install the SDK package (e.g. <code>npm install @chaingpt/generalchat</code>) and import it. Initialize an instance with your API key, then call provided methods (e.g. <code>chat.ask("...")</code>) instead of manual request. The SDK manages authentication internally.</td></tr><tr><td><strong>Features</strong></td><td>Full control over request/response handling. Supports all features (including streaming responses via HTTP. You’ll manage details like constructing JSON payloads, maintaining conversation IDs, and error handling with HTTP status codes.</td><td>Higher-level abstraction with utilities: built-in streaming support (events or async iterators), automatic conversation context management, and TypeScript type definitions for responses. Simplifies error handling and retries by exposing them as methods or callbacks.</td></tr><tr><td><strong>When to Use</strong></td><td>Use the REST API if you’re working in a non-JS environment or need maximum convinience. It’s ideal for backend services in Python/Java/etc., or any scenario where adding an SDK isn’t feasible. You get language-agnostic flexibility at the cost of a bit more coding.</td><td>Use the SDK if you are building in JavaScript/TypeScript and want to integrate quickly. It eliminates boilerplate and reduces mistakes by handling the API under the hood. This is perfect for Node.js backends or web apps that want a plug-and-play chatbot client.</td></tr></tbody></table>

***

### Customization via AI Hub (Tone & Context)

One of ChainGPT’s most powerful features is the ability to **customize** the chatbot’s behavior and knowledge to fit your project. Through the ChainGPT **AI Hub** (a web portal for developers), you can configure your AI agent’s personality and inject custom context or data, without needing to train a new model from scratch. Key customization options include:

* **Tone & Persona:** Define the AI’s tone and style to match your brand or use-case. You can choose from preset personalities (e.g. friendly, professional, humorous) or create a custom one. This setting influences the style of all responses (for example, a _friendly_ tone might use casual language and emojis, whereas a _formal_ tone would be more concise and polite).
* **Project Knowledge Base:** Provide your project’s specific details such as name, description, website, documentation links, token info, FAQs, etc., via the AI Hub. The ChainGPT LLM will incorporate this information when responding. For instance, if a user asks “What is this project about?”, the chatbot will answer with the details **you** provided (mission, features, tokenomics) instead of a generic response.
* **Context Injection:** Dynamically inject additional context on a per-request basis. Along with your user’s question, you can send contextual data (enabled by setting `useCustomContext: true` or by adding a `contextInjection` object in the API/SDK call). This could be a specific article, a recent announcement, or any relevant text. The injected context will influence the answer for that query, allowing fine-tuned responses (e.g. ask a question about a particular document by injecting its content).
* **Knowledge Extensions:** Extend the AI’s knowledge beyond its base training. In AI Hub you can link to external URLs (like your platform’s docs or blockchain explorers) or upload files for the AI to refer to. This makes the chatbot capable of pulling in information from those sources when needed. For example, you could enable it to answer questions using data from your **own** documentation or even fetch on-chain stats about your token.

Using these features, you can tailor the AI’s output to be highly relevant and aligned with your project without writing long prompts each time. A support chatbot could adopt a polite, helpful tone and have full knowledge of your product’s FAQ, while a community bot for a DeFi project might use a meme-friendly voice and know the project’s latest roadmap update. This configuration-driven approach is safer and more manageable than manual prompt engineering – the context is stored securely in the AI Hub (tied to your API key) and can be updated on the fly for all subsequent calls.

***

### Getting Started: Onboarding and API Keys

Integrating ChainGPT’s Web3 AI Chatbot is designed to be developer-friendly. Here’s a quick step-by-step guide to start building with the API or SDK:

1. **Sign Up & Access AI Hub:** If you haven’t already, create an account on ChainGPT and log in to the **AI Hub** (the developer dashboard). This is where you can generate API keys and configure your chatbot.&#x20;

{% hint style="info" %}
_Note:_ You can use the AI Hub to tweak settings for built-in bots (like the Telegram/Discord chatbot) even without an API key, but to integrate into your own app you’ll need to obtain a key.
{% endhint %}

1. **Generate an API Key:** In the AI Hub, navigate to the API/SDK section or developer settings and click “**Create API Key**” (you may be asked to label it for organization. Copy the key when it’s displayed – for security, you won’t be able to view it again later. (If you lose a key, you can always create a new one.)
2. **Secure the Key:** Treat your ChainGPT API key like a password. **Do not** expose it in client-side code or public-side. If you’re making calls from a web frontend, route them through your backend server to keep the key hidden. In a Node.js or server environment, store the key in an environment variable or secure config file, not in your code / front-end. This prevents others from using your key, which protects your account and usage quota.
3. **Choose Integration Method:** Decide whether you’ll use the JavaScript SDK or call the REST API directly in your project.
   * _**Using the SDK:**_ Install the NPM package (e.g. `npm install @chaingpt/generalchat`) and import it into your project. Initialize the client with your API key, for example: `const chat = new GeneralChat({ apiKey: "YOUR_API_KEY" });` . Now you can send queries with a single method call – e.g. `const answer = await chat.ask("...");` – instead of manually handling HTTP. _(See the SDK docs for details on streaming responses or advanced usage.)_
   *   _**Using the REST API**:_ Make HTTPS requests to the ChainGPT API endpoint from your app. For example, a chat query can be sent with an HTTP **POST** request to the `/v1/chat` endpoint. Include your API key in the header and your question in the JSON format. **For instance:**

       ```http
       POST https://api.chaingpt.org/v1/chat
       Authorization: Bearer YOUR_API_KEY
       Content-Type: application/json

       {
         "question": "Explain the difference between ERC-20 and ERC-721",
         "useCustomContext": true
       }
       ```

{% hint style="info" %}
_**Example API request**_**&#x20;–** The response will contain the chatbot’s answer and any relevant metadata (e.g. token usage). You can test this in a tool like **curl** or Postman before coding it.
{% endhint %}

4. **Handle the Response:** Parse and use the AI’s reply in your app. For a chat interface, you’ll append the chatbot’s answer to the conversation thread. If you requested a streaming response, handle the stream of partial data to update the answer in real-time. (The SDK provides convenience methods/events for streaming; with the raw API you’d read the response in chunks.) Make sure to account for any error messages or rate-limit responses in your code.
5. **Test and Tweak:** During development, experiment with your chatbot and adjust settings as needed. Use the AI Hub to tweak the bot’s tone or inject more context, then test again to see the channels. Monitor your usage in the AI Hub dashboard – you might use a test mode or small scale to ensure everything works without hitting quotas. This iterative process will help fine-tune the user experience.
6. **Go Live:** Integrate the chatbot into your production environment once you’re satisfied. This could mean deploying a chat widget on your site, adding a new support assistant in your app, or inviting the bot to your community channels. As users start interacting with it, you can continue to refine the AI’s behavior via AI Hub settings (which take effect immediately, no code deploy needed). Keep an eye on usage to ensure you have the appropriate plan or credits for your user base.

***

### Pricing, API Key Management & Security

When deploying ChainGPT in a production setting, keep these considerations in mind:

* **API Key Management:** You can generate multiple API keys to separate different environments or applications (e.g. one for development, one for products. This allows you to monitor usage per key and revoke keys individually if needed. Regularly review your keys in the dashboard and **revoke any that are not in use** to minimize security risks. Each key accumulates its own usage stats, which helps with cost tracking and debugging.
* **Security Best Practices:** Never embed your secret API key in client-side code (JavaScript) or any public repositories. If building a web app, have your server relay requests so the key stays hidden. Use environment variables or secure storage for keys in your backend. All requests to ChainGPT’s API are encrypted via HTTPS and processed on secure infrastructure, but it’s up to you to enforce good security on your side. Also, ChainGPT does **not** use your query data to train its models, so your interactions remain private to your account.
* **Pricing & Usage:** ChainGPT uses a **credit-based pricing model** for API/SDK usage. (For example, a certain number of tokens or API calls might cost a fraction of a credit.) Refer to the [Pricing & Membership Plans](https://docs.chaingpt.org/the-ecosystem/ai-tools-and-applications/pricing-and-membership-plans) page for details on free tiers and paid plans. Generally, basic community chatbot usage on Telegram/Discord is free (within limits), whereas higher-volume or commercial API usage will require purchasing credits. The AI Hub dashboard shows your current credit balance and usage in real-time, so you can monitor consumption and top-up or upgrade as needed to avoid service interruption.

***

### Next Steps & Resources

* **API Reference:** Dive into the full REST API documentation for ChainGPT’s Web3 LLM. It covers all available endpoints, request/response schemas, error codes, and examples in various languages. This is your go-to resource when you need detailed info on calling the API and handling responses.

{% content-ref url="api-reference.md" %}
[api-reference.md](api-reference.md)
{% endcontent-ref %}

* **SDK Documentation:** Check out the JavaScript/TypeScript SDK docs, which include quickstart examples, class and method details, and best practices for using the SDK in your product.. Learn how to stream responses, manage conversation history in code, and more.

{% content-ref url="sdk-reference.md" %}
[sdk-reference.md](sdk-reference.md)
{% endcontent-ref %}

* **Use Cases & Examples:** Explore real-world examples and use cases to see ChainGPT in action. From DeFi platforms with on-site AI advisors to NFT marketplaces enhancing user support, these examples will inspire you and provide guidance on design patterns and integration approaches. _(See our “AI Web3 Chatbot: Features and Use Cases” page for a feature-oriented overview.)_

{% content-ref url="../use-cases-and-examples.md" %}
[use-cases-and-examples.md](../use-cases-and-examples.md)
{% endcontent-ref %}

* **Case Studies:** Read in-depth case studies of projects that have successfully integrated ChainGPT. Learn how companies improved user engagement and support efficiency using the chatbot, and understand the ROI and quantitative benefits. (For a detailed example, see how **DexCheck** – an analytics platform – leveraged ChainGPT to assist its users.)

{% content-ref url="../case-studies.md" %}
[case-studies.md](../case-studies.md)
{% endcontent-ref %}

* **ChainGPT AI Hub:** Use the AI Hub portal to manage and refine your chatbot. In the Hub you can adjust the AI’s persona and knowledge base over time, view analytics (usage metrics, chat logs, user feedback), and ensure your bot is performing well. It’s essentially the control center for your AI agent, so make sure to familiarize yourself with its features. \
  &#xNAN;**— Crypto AI Hub:** [https://app.chaingpt.org/](https://app.chaingpt.org/)\
  &#xNAN;**— API Dashboard:** [https://app.chaingpt.org/apidashboard](https://app.chaingpt.org/apidashboard)
* **Other AI Tools:** Remember, ChainGPT’s ecosystem offers more than just the chatbot. Via the same API/SDK, you can access tools like the **Smart Contract Auditor**, **Smart Contract Generator**, **AI NFT Generator,** and more. These can complement your chatbot – for example, you might integrate the auditor to let users _audit a contract via chat_, or use the NFT generator for creative user commands. Check out the relevant docs sections for these tools to expand your application’s AI capabilities.

{% content-ref url="../introduction-to-chaingpts-developer-tools.md" %}
[introduction-to-chaingpts-developer-tools.md](../introduction-to-chaingpts-developer-tools.md)
{% endcontent-ref %}

***

**Ready to build?** Get your API key from the AI Hub and start integrating a Web3-smart chatbot into your project today. With ChainGPT’s specialized LLM, you can offer users a cutting-edge, crypto-savvy assistant that elevates their experience. We’re excited to see what you create – and if you need any help, feel free to reach out on our developer Discord or support channels. _**Happy coding!**_
