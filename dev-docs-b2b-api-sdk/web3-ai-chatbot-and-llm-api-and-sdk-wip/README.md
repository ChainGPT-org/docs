# Web3 AI Chatbot & LLM (API & SDK)  \[WIP]

## ChainGPT Web3 AI Chatbot & LLM (API & SDK) Overview

ChainGPT’s Web3 AI Chatbot is a domain-optimized conversational AI built specifically for blockchain and crypto applications. It combines a powerful Large Language Model (LLM) with Web3-specific knowledge, enabling developers and businesses to streamline customer engagement, automate support, and provide real-time insights in their applications . Unlike general-purpose AI, ChainGPT’s LLM is trained on vast blockchain and crypto data, giving it deep expertise in Web3 topics – from smart contracts and DeFi to live market trends and regulatory news . This makes it an essential tool for organizations operating in the blockchain, Web3, and crypto industries , already trusted in production by major platforms (for example, Binance Square for AI-driven news, BNB Chain, TronDAO, and 50+ Web3 communities via chat integrations) .

### Key Differentiators of ChainGPT’s Web3 LLM

**ChainGPT’s Web3 Chatbot & LLM offers unique advantages that set it apart from general AI services:**

* **Web3 Domain Expertise:** Natively understands blockchain terminology, smart contract code, tokenomics, and crypto trends. The model has been fine-tuned on extensive Web3 data, providing precise, industry-specific answers where a generic model might falter or hallucinate . It can explain complex concepts (e.g. _yield farming_, _NFT metadata_) or analyze Solidity code with context that general LLMs lack.
* **Real-Time Blockchain Insights:** ChainGPT’s infrastructure can incorporate live data from the crypto ecosystem. The chatbot can monitor on-chain activity and market data in real-time , enabling use cases like up-to-the-minute price checks, trend analysis, and blockchain explorer queries within chat. This real-time capability means your users get up-to-date information (whereas a static trained model might be outdated).
* **Crypto-Specific Utilities:** Beyond Q\&A, the Web3 Chatbot can perform tasks tailored to crypto users. For example, it can assist with smart contract audits and generation, technical analysis for traders, or even NFT creation via integrated tools. These features are purpose-built for Web3 (e.g. analyzing a smart contract for vulnerabilities), giving your application built-in functionality that would normally require specialized tools.
* **Multi-Channel Deployment:** ChainGPT’s chatbot is flexible in integration . You can embed it in your web or mobile app via the API/SDK, or deploy it on popular community platforms. For instance, projects can launch it on Telegram or Discord (free) to handle community questions and support . This multi-channel support means you can meet your users where they are – whether on your website, in your dApp, or in chat groups – with a consistent AI assistant.
* **Security & Privacy Built-In:** Designed with a privacy-first approach, ChainGPT’s solution ensures that sensitive user data and blockchain queries are handled securely. All API communication is encrypted, and data is stored in secure, isolated infrastructure . For enterprise use, this focus on security and controlled access means you can integrate the AI confidently without exposing confidential information (and unlike some public chatbots, your data isn’t used to train broad models).

### Architecture & Integration Flow

Integrating the ChainGPT Web3 AI Chatbot into your tech stack is straightforward. The high-level architecture involves your application (frontend and/or backend) communicating with ChainGPT’s cloud-hosted LLM service via REST API calls or through ChainGPT’s SDK. Below is an overview of how a typical integration works:

```
[ Client Application ] – (User's query) → – [**ChainGPT SDK** or **API**] – → [**ChainGPT Web3 LLM Service**] – → (AI processes query with Web3 knowledge + custom context) – → [**ChainGPT Response**] – → [ Client Application displays answer to user ]
```

1. **User Interaction:** A user interacts with your interface (website, mobile app, dApp, or chat widget) and asks a question or issues a command (e.g. _“What is the current ETH price?”_ or _“Explain this smart contract code”_).
2. **Application Request:** Your frontend or backend sends this request to ChainGPT’s LLM. This can be done via a direct HTTPS request to the ChainGPT API or by invoking a method from the ChainGPT SDK. The request includes your API key for authentication and can optionally include extra context or parameters (like a specific AI “tone” or additional knowledge, see Customization below).
3. **ChainGPT LLM Processing:** ChainGPT’s cloud service receives the request and routes it to the Web3 AI LLM. The model processes the prompt, leveraging:
   * **Built-in Crypto Knowledge Base:** Comprehensive understanding of blockchain concepts and data sources (it may fetch live data if needed, such as querying market prices or on-chain info).
   * **Custom Context:** Any project-specific context you’ve configured (e.g. details from your AI Hub settings or context injection) is applied to tailor the response.
   * **The LLM then generates a response** (answer, explanation, or action output).
4. **Response Delivery:** The answer is returned back through the API/SDK to your application. If you use the SDK, it will handle the response object or streaming for you; if you use the raw API, you’ll get a JSON response over HTTP. Your application then uses this data to display the answer to the user in the UI.
5. **Continuous Interaction:** The Chatbot can handle multi-turn conversations. Your app can send follow-up user messages along with a conversation ID or the previous context (the SDK makes managing chat history easier). The architecture remains the same for each turn – your app sends the prompt, ChainGPT returns an answer – creating a seamless conversational experience.

**Scalability:** This cloud-based architecture means ChainGPT handles the heavy AI computation on its servers (leveraging GPU infrastructure and optimized models), so you don’t need to run any AI infrastructure yourself. Your app simply makes lightweight requests, which keeps integration scalable and performance optimized on your end.

***

### Integration Options: REST API vs. JavaScript/TypeScript SDK

ChainGPT provides two primary integration methods to suit your development needs: a RESTful Web API and an official JavaScript/TypeScript SDK. Both ultimately give you access to the same Web3 AI capabilities, but they offer different conveniences:

* **💻 REST API:** Ideal for developers in any programming environment. The ChainGPT API is a standard HTTPS interface – you can call it from any language or platform that can send HTTP requests (Python, Java, Go, backend servers, or even directly from front-end if needed). Using the API gives you full control: you’ll manage making requests to endpoints like /v1/chat (for example), including headers and payloads (like the user prompt, API key, etc.), and handle the JSON responses. This is great for integrating into backend services or when you’re not using Node.js. The API is documented with its endpoints, parameters, and response formats in the API Reference (see links below). It also supports advanced features such as streaming responses for token-by-token output (so you can stream chatbot answers to your UI, similar to OpenAI’s streaming API).
* **🧩 ChainGPT JS/TS SDK:** Ideal for Node.js, web, and TypeScript projects. The SDK is a ready-to-use NPM package (@chaingpt/generalchat for the general chat module) that wraps all the API calls and handles a lot of boilerplate for you. By using the SDK, you can simply import and invoke methods on a ChainGPT client object rather than manually constructing HTTP requests. For example, you can call createChatStream() or ask() on the SDK, and it will internally call the API and stream the results back to you as events or promises. The SDK manages authentication (you just configure your API key once), provides TypeScript types for responses, and simplifies features like turning on/off chat history, using streaming, etc. It’s especially convenient if you plan to integrate the chatbot into a Node.js backend or a React/Next.js frontend – giving you a more idiomatic way to interact with the ChainGPT service. Refer to the SDK Reference section for code examples and all available methods.

{% hint style="info" %}
**Choosing API vs SDK:** If you are using a language other than JavaScript/TypeScript or need maximal control, the REST API is the way to go. If you’re building in the JS/TS ecosystem, the SDK will speed up development and reduce potential errors by providing a higher-level interface. In many cases, you can even mix both: use the SDK for your primary app logic, but fall back to direct API calls for other languages or platforms. Regardless of choice, both methods require an API key and follow the same usage quotas and pricing.
{% endhint %}

***

### Customization via AI Hub (Tone & Context)

One of ChainGPT’s most powerful features is the ability to customize the chatbot’s behavior and knowledge to fit your project’s needs. Through the AI Hub – ChainGPT’s developer portal – you can configure your AI agent’s personality and inject custom context or data, without needing to train a new model from scratch.

**AI Hub Custom Settings:** In the AI Hub dashboard (accessible on the ChainGPT platform), you can tweak various parameters of your chatbot instance:

* **Tone & Persona:** Define the AI’s tone to match your brand or use-case. You can choose from preset tones (e.g. friendly, professional, humorous ) or create a custom tone. This setting influences the style of the chatbot’s responses globally (for example, a _friendly_ tone might use more casual language and emojis, whereas a _formal_ tone would be more concise and polite).
* **Company / Project Details:** You can input your project’s specific details such as Company Name, Description, Website URL, Whitepaper link, Token info and more . These details form a custom knowledge base for the AI. Once provided, the ChainGPT LLM will automatically incorporate this information when responding to users. For instance, if a user asks “What is your project about?” or something related to your token, the chatbot will reply with the details you provided (mission, features, tokenomics, etc.) rather than a generic answer.
* **Context Injection:** By default, if you’ve entered your project info in AI Hub, setting useCustomContext: true in the SDK will make the chatbot auto-load that context for each query . Developers can also override or add context dynamically on a per-request basis – the SDK and API allow sending a contextInjection object alongside the user’s question . This can include any ad-hoc information or documents relevant to the query. For example, you might inject the text of a specific blog post or the content of a recent announcement so the bot can reference it in its answer. This on-the-fly injection overrides the default AI Hub context for that response, giving fine-grained control when needed.
* **Knowledge Base Extensions:** In addition to simple text context, ChainGPT is evolving to let you plug in richer knowledge sources. By providing URLs (like documentation or block explorers) or uploading files, the AI can be guided to draw from those resources. The AI Hub acts as the central place to manage these resources. This means your chatbot can behave like an expert on your product – it can answer questions about your platform’s details, or even fetch on-chain data about your token if configured, all in one cohesive conversation.

Using these customization features, you ensure the AI’s output is relevant, accurate, and aligned with your goals. A customer support chatbot, for example, can adopt a polite/helpful tone and have full knowledge of your product FAQs; a community chatbot for a DeFi project could have a chill, meme-friendly tone with knowledge of the project’s roadmap and token stats. All of this is achievable through configuration rather than low-level prompting.

**AI Hub vs. Manual Prompting:** While you could always prepend information to user prompts manually, the AI Hub approach is safer and more manageable – it persists the context securely tied to your API key, and saves you from sending large prompts each time. It’s also updateable on the fly (update your company info in the portal, and all subsequent chats will use the new info).

***

### Getting Started: Onboarding and API Keys

Integrating ChainGPT’s Web3 AI Chatbot is designed to be developer-friendly. Here’s a quick guide to start building with the API/SDK:

1. **Sign Up and Access the AI Hub:** If you haven’t already, create an account on ChainGPT’s platform and log in to the AI Hub (the developer dashboard). This is where you can generate API keys and configure your chatbot. (If you are testing via Telegram/Discord first, you can still use the AI Hub to tweak settings, but an API key is not required for the out-of-box bots.)
2. **Generate an API Key:** In the AI Hub or account dashboard, navigate to the API/SDK section or Developer Settings. Click on “Create Secret Key” (or similar) to generate a new API key . You might be prompted to label it (if you plan to use multiple keys for different apps). The platform will show you the API key string – make sure to copy it somewhere safe. Note: For security, this key will only be shown once. If you lose it, you’d need to generate a new one.
3. **Secure Your Credentials:** Treat your ChainGPT API key like a password. Do not expose it in client-side code or public repos. If you’re calling the API from a web app, route the requests through your backend to keep the key secret. In a Node.js environment, store it in an environment variable or a secure config. The key is used to authenticate your requests and track usage, so keeping it secret prevents others from using your quota or accessing your account.
4. **Choose Your Integration Path:** Install the SDK or plan your API calls:
   * **Using the SDK:** If you’re going with Node/TypeScript, install the package (e.g. npm install @chaingpt/generalchat). Import the relevant classes in your project, and initialize the client with your API key, e.g. const chat = new GeneralChat({ apiKey: "YOUR\_API\_KEY" });. You’re now ready to send queries: const answer = await chat.ask("Your question?") (see SDK docs for details and streaming usage).
   * **Using the API directly:** Find the endpoint documentation in the API Reference. Typically, you’ll POST to an endpoint with your query and parameters. For example, a request might look like:

```
POST https://api.chaingpt.org/v1/chat
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "question": "Explain the difference between ERC-20 and ERC-721",
  "useCustomContext": true
}
```

* The response will contain the chatbot’s answer and any relevant metadata (e.g. tokens used). You can test this out with cURL or Postman first.

4. **Handle Responses in Your App:** Whether using SDK or API, implement the logic to display or utilize the AI’s response. For a chat UI, append the chatbot’s answer to the conversation. If you requested streaming, handle the stream of partial outputs to render the answer in real-time. The SDK provides event handlers for streaming, and with the API you’d read the HTTP response chunk by chunk.
5. **Tune and Iterate:** Using the AI Hub, adjust the chatbot settings (tone, context) to your liking and test how it responds. During development, you can use a test environment or a limited access token to avoid incurring costs. Check the usage dashboard in AI Hub to monitor how many requests you’re making and ensure everything stays within your plan’s limits.
6. **Go Live:** Once satisfied, integrate the chatbot into your production workflow – whether it’s a live web chat on your site, a support bot in your app, or a community assistant in Telegram. Users can now interact with your AI chatbot, and you can continue to refine its behavior through the AI Hub without redeploying code.

**API Key Management:** Remember that you can create multiple API keys for different environments (dev, prod) or services. You can revoke keys via the dashboard if needed. Each key will accumulate usage statistics separately, which is useful for monitoring and budgeting.

**Pricing & Usage:** ChainGPT’s API/SDK access uses a credit-based pricing model (for example, 0.5 credit per API call or per certain number of tokens). Ensure you’ve reviewed the pricing details on the Pricing & Membership Plans page. Community integrations on Telegram/Discord are free to use (with some limits), while heavy or enterprise API usage may require a subscription or pay-as-you-go credits . The AI Hub will show your current credit balance or plan usage. Always secure your API key to prevent unauthorized usage that could consume your credits.

***

### Next Steps and Resources

You now have a high-level understanding of ChainGPT’s Web3 AI Chatbot and how to integrate it. Here are some handy links and references to continue your journey:

* **📕 API Reference:** _Explore the full REST API documentation_, including endpoint definitions, request/response schemas, error codes, and examples in various languages. This is your go-to for in-depth technical specs when calling the API directly.

{% content-ref url="web3-llm-api-reference.md" %}
[web3-llm-api-reference.md](web3-llm-api-reference.md)
{% endcontent-ref %}

* **📗 SDK Documentation:** _Dive into the JS/TS SDK guides_, which include quickstart code snippets, class and method references, and best practices for using the SDK in your project. Learn how to handle streaming responses, manage conversation history, and use advanced features like context injection through code.

{% content-ref url="web3-ai-chatbot-sdk-reference.md" %}
[web3-ai-chatbot-sdk-reference.md](web3-ai-chatbot-sdk-reference.md)
{% endcontent-ref %}

* **💡 Use Cases & Examples:** Check out our documentation on real-world Use Cases for ChainGPT’s chatbot, and example implementations. From DeFi platforms offering on-site AI assistants to NFT marketplaces enhancing user support, these examples will inspire you with what’s possible. _(For instance, see how the chatbot is used for community management and live market analysis in our “AI Web3 Chatbot: Features and Use Cases” page.)_

{% content-ref url="../use-cases-and-examples.md" %}
[use-cases-and-examples.md](../use-cases-and-examples.md)
{% endcontent-ref %}

* **🏆 Case Studies:** Read Case Studies of projects that have successfully integrated ChainGPT. Learn how companies in the Web3 space improved user engagement and support efficiency using the chatbot, and gain insights into integration strategies and ROI. _(For a deep-dive, see the DexCheck case study to learn how an analytics platform leveraged ChainGPT for its users.)_

{% content-ref url="../case-studies.md" %}
[case-studies.md](../case-studies.md)
{% endcontent-ref %}

* **🤖 AI Hub Portal:** When you’re ready to configure or fine-tune your chatbot’s settings, head over to the ChainGPT AI Hub. The portal also provides analytics – such as usage metrics, user feedback, and performance logs – to help you continuously improve your AI agent. [https://app.chaingpt.org/apidashboard](https://app.chaingpt.org/apidashboard)\

* **🛠 Other Tools in the ChainGPT Suite**: ChainGPT offers more than just the chatbot. You might be interested in our AI Smart Contract Auditor or AI NFT Generator for additional functionality in your application. These are accessible via the same API/SDK framework, making it easy to add multiple AI features under one roof. See the relevant sections in our docs for details.

***

With ChainGPT’s Web3 AI Chatbot and LLM, you have a cutting-edge conversational AI solution tailored to the world of crypto. Whether you’re building a customer support bot for a blockchain product, a trading assistant in a DeFi app, or an educational chatbot for your community, ChainGPT delivers the knowledge and flexibility to elevate your user experience. We’re excited to see what you build – and as always, feel free to reach out on our developer Discord or support channels for any help. Happy coding!
