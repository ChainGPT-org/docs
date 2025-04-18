# Web3 AI Chatbot & LLM (API & SDK)

## ChainGPT Web3 AI Chatbot & LLM (API & SDK) - Overview and Architecture

**ChainGPT Web3 AI Chatbot & LLM** is a Web3-native AI assistant built specifically for the crypto world. It combines a powerful large language model with deep blockchain expertise, enabling seamless integration of crypto-aware AI into your applications. The model is trained on blockchain data (smart contracts, DeFi protocols, NFTs, DAOs) and real-time market information, making it ideal for use cases like customer support, on-chain analytics, trading assistance, and community engagement.

Already trusted by leading platforms like Binance Square, BNB Chain, and TronDAO, ChainGPT’s Web3 LLM is purpose-built for the next generation of Web3 applications.

**Pricing:** The API is credit-based – each API or SDK request consumes **0.5 credits** (approximately $0.005 USD), and enabling chat history for a request consumes an **additional 0.5 credits**. Ensure your ChainGPT account has sufficient credit balance; if you run out, API calls will return an error (HTTP 402/403 for insufficient credits).

***

### **Key Differentiators:**&#x20;

ChainGPT’s Web3 AI stands out from general-purpose LLMs in several ways:

<table><thead><tr><th width="158.46484375">Title</th><th>What It Does</th></tr></thead><tbody><tr><td><strong>Web3 Domain Expertise</strong></td><td>Fine-tuned on crypto and blockchain content (smart contracts, DeFi, NFTs, tokenomics). This domain-specific training means it excels at explaining or analyzing crypto concepts and code, unlike generic AI models.</td></tr><tr><td><strong>Real-Time Insights</strong></td><td>Able to pull live on-chain data and market information on the fly. This supports use cases like real-time price checks, blockchain explorer queries, trend analysis, and more, ensuring answers are always up-to-date.</td></tr><tr><td><strong>Crypto-Specific Utilities</strong></td><td>Includes built-in capabilities for tasks like smart contract auditing, code generation, NFT creation, and technical analysis, which traditionally require separate developer tools.</td></tr><tr><td><strong>Multi-Channel Deployment</strong></td><td>Can be deployed across websites, dApps, Discord, Telegram, and other platforms for a consistent AI assistant experience. A single ChainGPT-powered chatbot can serve users across multiple channels.</td></tr><tr><td><strong>Security &#x26; Privacy First</strong></td><td>All data is encrypted in transit and stored in isolated environments. ChainGPT does <strong>not</strong> use your data to train its models. This makes it enterprise-safe and privacy-conscious for production use.</td></tr></tbody></table>

**Learn more about the unique capabilities of ChainGPT's Web3 LLM:**

{% content-ref url="unique-capabilities.md" %}
[unique-capabilities.md](unique-capabilities.md)
{% endcontent-ref %}

***

### Architecture & Integration Flow

Integrating the ChainGPT Web3 AI Chatbot into your tech stack is straightforward. Your application (front-end or back-end) communicates with ChainGPT’s cloud-hosted AI service via REST API calls or through the provided SDK.

<div align="left"><figure><img src="../../.gitbook/assets/image (46).png" alt="" width="375"><figcaption></figcaption></figure></div>

At a high level, the interaction flow works as follows:

1. **User Question:** A user interacts with your interface (website, mobile app, dApp, chat widget, etc.) and asks a question or issues a command. For example: _“What’s the current ETH price?”_ or _“Explain this smart contract code.”_
2. **Application Request:** Your application (client-side or server) sends the user’s query to the ChainGPT service, either by making a direct HTTPS request to the API or by calling the ChainGPT SDK. The request includes your API key for authentication, and can optionally include parameters like a desired response tone or additional context data.
3. **AI Processing in Cloud:** ChainGPT’s backend receives the request and the Web3 LLM processes it. The model uses its built-in crypto knowledge base and, if needed, fetches live on-chain or off-chain data to answer the query. If you’ve supplied custom context (see **Customization via AI Hub** below), that information is incorporated as well. The AI then generates a response—either answering the question or performing the requested analysis.
4. **Response Delivery:** The answer is returned to your application through the API/SDK. If you used the SDK, it will handle parsing the response (and streaming it, if applicable) for you. If you used the raw REST API, you’ll receive a JSON response containing the result. Your application can then display or utilize the answer (e.g. rendering the chatbot’s reply in your UI).
5. **Multi-Turn Conversations:** ChainGPT supports follow-up questions and contextual conversations. Your app can send subsequent queries along with a conversation identifier or the prior chat history so that ChainGPT knows the context (previous Q\&A) and can respond accordingly. The SDK simplifies this by managing conversation state for you, while the REST API provides parameters to handle it (more on this in the sections below). This allows users to have natural back-and-forth dialogues with the AI.

{% hint style="info" %}
**Note:** All heavy AI computation (GPU model inference, data fetching, etc.) is handled on ChainGPT’s cloud servers. Your integration remains lightweight—your app sends a request and renders the AI’s reply. There’s no need to run any AI model locally, which keeps your deployment scalable and efficient.
{% endhint %}

***

### Integration Options: REST API vs JavaScript SDK

**You can integrate ChainGPT in two ways:** via a RESTful HTTP API, or via the official JavaScript/TypeScript SDK. Both methods provide the same core Web3 AI capabilities, but offer different developer experiences. The table below compares these options:

<table><thead><tr><th width="95.77734375">Aspect</th><th>REST API (HTTP calls)</th><th>JavaScript SDK (Node/Browser)</th></tr></thead><tbody><tr><td><strong>Platform</strong></td><td>Language-agnostic – works with any environment that can send HTTPS requests (cURL, Python, Java, Go, etc.). Ideal for server-side integration or non-JS languages.</td><td>Designed for JavaScript/TypeScript projects (Node.js backends, browser apps with bundlers, React, etc.). Distributed via NPM for easy inclusion.</td></tr><tr><td><strong>Setup</strong></td><td>No additional library required. You manually make HTTP requests to ChainGPT’s endpoints, adding headers (e.g. <code>Authorization: Bearer &#x3C;API_KEY></code>) and parsing JSON responses yourself.</td><td>Install the SDK package (<code>npm install @chaingpt/generalchat</code>). Import it and initialize with your API key. The SDK handles authentication headers internally, and provides convenient methods (e.g. <code>createChatBlob</code>, <code>createChatStream</code>) instead of manual HTTP calls.</td></tr><tr><td><strong>Features</strong></td><td>Full low-level control over requests and responses. Supports all features (including streaming) via direct HTTP. You manage details like JSON payload construction, maintaining conversation IDs, and interpreting HTTP status codes or errors.</td><td>Higher-level abstraction with built-in utilities: streaming handled via Node.js streams or async iterators, automatic conversation context management, and TypeScript type definitions. Simplifies error handling (throws exceptions or emits events instead of raw HTTP codes) and reduces boilerplate.</td></tr><tr><td><strong>When to Use</strong></td><td>Use the REST API if you’re working in a non-JS environment or prefer to avoid external libraries. It’s ideal for backends in Python, Java, etc., or any scenario where adding an SDK isn’t feasible. This approach offers language flexibility at the cost of a bit more manual coding.</td><td>Use the SDK if you are building in JavaScript/TypeScript and want to integrate quickly and reliably. It eliminates a lot of boilerplate and potential mistakes by handling the HTTP calls under the hood. Perfect for Node.js services or web apps that want a plug-and-play chatbot client.</td></tr></tbody></table>

***

### Customization via AI Hub (Tone & Context)

One of ChainGPT’s most powerful features is the ability to customize the chatbot’s behavior and knowledge **without** writing complex prompts or training a model from scratch. Through the ChainGPT **AI Hub** (the web portal for configuring your AI agent), you can tailor the assistant’s personality and context to fit your project. Key customization options include:

* **Tone & Persona:** Define the AI’s tone and style to match your brand or use-case. You can choose from preset personas (e.g. friendly, professional, humorous) or define a custom tone. This setting influences the style of all responses. For example, a friendly tone might use casual language and emojis, whereas a formal tone would be more concise and polite.
* **Project Knowledge Base:** Provide your project’s specific details—such as the project name, description, website, documentation links, token information, FAQs—via the AI Hub. The ChainGPT LLM will incorporate this information when answering questions. For instance, if a user asks “What is this project about?”, the chatbot will respond with the details you provided (mission, features, tokenomics, etc.) instead of a generic answer.
* **Context Injection:** Dynamically inject additional context on a per-request basis. Along with a user’s question, your application can send extra information (by setting `useCustomContext: true` and including a `contextInjection` object in the API/SDK call). This could be a specific document, a recent announcement, or any relevant text snippet. The injected context will influence the answer for that query, allowing fine-tuned responses. For example, you might ask a question about a particular whitepaper and inject that whitepaper text as context, so the AI can draw on it.
* **Knowledge Extensions:** Extend the AI’s knowledge beyond its base training data. In the AI Hub, you can link external resources (like your platform’s docs or blockchain explorers) or upload files for the AI to reference. This makes the chatbot capable of pulling in information from those sources when needed. For example, you could enable it to answer questions using your product’s documentation or to fetch on-chain statistics about your token.

Using these customization features, you can tailor the AI’s output to be highly relevant and aligned with your project, **without** having to prepend every prompt with a long description. Once configured, the context and tone settings are stored securely in the AI Hub and automatically applied to all requests made with your API key (unless you override them on a specific request). This approach is more manageable and consistent than manual prompt engineering for each query.

***

### Getting Started: Onboarding & API Keys

Integrating ChainGPT is designed to be fast and friction‑free. Follow these steps to get up and running:

1. **Sign In & Access the** [**API Dashboard**](https://app.chaingpt.org/apidashboard) **on ChainGPT's AI Hub**
2. **Generate Your API Key**
   * In AI Hub’s **API Dashboard**, click **"Create New Secret Key"**.
   * Label it for your project, then copy the key when it appears (you won’t be able to view it again).
   * If you ever lose it, simply revoke and generate a new one.
3. **Customize your API key’s context by providing (optional step):**&#x20;
   * Company and product details.
   * Crypto‑token metadata (if applicable).
   * Social media links.
   * Uploaded knowledge‑base documents so the AI has complete, up‑to‑date information about your organization.
4. **Secure Your Key**
   * Treat your API key like a password.
   * **Never** hard‑code it in client‑side code or commit it to a public repo.
   * In server environments (Node.js, Python, etc.), store it in environment variables or a secrets manager.
   * If calling from a browser, proxy all requests through your backend to keep the key hidden.
5. **Choose Your Integration Method**
   * **REST API:** Make HTTPS requests directly to `https://api.chaingpt.org` (e.g. `POST /chat/stream`).
   * **JavaScript SDK:** Install with `npm install @chaingpt/generalchat` and call methods like `createChatBlob()` or `createChatStream()`.
6. **Test, Tweak & Go Live**
   * In development, experiment with your prompts, chat history settings, and context injection in AI Hub.
   * Monitor usage and credits in the AI Hub dashboard to avoid interruptions.
   * When ready, deploy your integration to production—updates to tone or context in AI Hub take effect immediately, no code changes required.

#### Security & API Key Management

* **Multiple Keys:** Create separate keys for dev, staging, and production to isolate usage and simplify revocation.
* **Least Privilege:** Only grant each key the permissions it needs. Revoke unused keys promptly.
* **Encryption:** All API calls occur over HTTPS; ChainGPT does **not** train on your data, ensuring privacy.
* **Credit Monitoring:** Each chat request costs **0.5 credits**, and enabling chat history costs **an additional 0.5 credits**. Keep an eye on your balance in AI Hub to avoid 402/403 errors.

***

### Next Steps & Resources

* **QuickStart Guide:** Follow the step‑by‑step REST API and SDK examples in the QuickStart section.

{% content-ref url="quickstart-guide.md" %}
[quickstart-guide.md](quickstart-guide.md)
{% endcontent-ref %}

* **API Reference:** Dive into every endpoint, parameter, and response schema in the full API Reference.

{% content-ref url="api-reference.md" %}
[api-reference.md](api-reference.md)
{% endcontent-ref %}

* **SDK Docs:** Explore detailed method descriptions and code samples in the JavaScript/TypeScript SDK Reference.

{% content-ref url="sdk-reference.md" %}
[sdk-reference.md](sdk-reference.md)
{% endcontent-ref %}

* **Use Cases & Examples:** See real‑world patterns for support bots, analytics agents, DeFi advisors, and more on our Use Cases page.

{% content-ref url="../use-cases-and-examples.md" %}
[use-cases-and-examples.md](../use-cases-and-examples.md)
{% endcontent-ref %}

* **Case Studies:** Learn how leading projects like DexCheck leveraged ChainGPT to boost engagement and efficiency.

{% content-ref url="../case-studies.md" %}
[case-studies.md](../case-studies.md)
{% endcontent-ref %}

* **ChainGPT AI Hub:** Return to [https://app.chaingpt.org/](https://app.chaingpt.org/) to refine context, tone, knowledge bases, and monitor usage metrics (if needed).
* **Support & Community:** Join our developer Discord or reach out via the support channels listed in AI Hub if you have questions.

{% hint style="info" %}
**Ready to build?** Grab your API key and start integrating a Web3‑smart chatbot that elevates your user experience—happy coding!
{% endhint %}
