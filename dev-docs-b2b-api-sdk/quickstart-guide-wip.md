---
hidden: true
---

# QuickStart Guide \[WIP]

## Quickstart Guide To ChainGPT's API & SDK

This guide will help you start building with ChainGPT’s suite of Web3 AI tools in just a few minutes. You’ll learn how to create an account, generate your API key, make your first API request, and begin integrating ChainGPT’s capabilities into your application.

***

### Step 1: Create an Account and API Key

1. Visit the [ChainGPT API Dashboard](https://app.chaingpt.org).
2. Connect your crypto wallet to sign up or log into your account.
3. Navigate to the API Keys section and click "Create New Secret Key".
4.  Customize your API key based on your company's needs & use-cases (optional)

    <figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
5. Store your API key securely in an environment variable or secret manager. Do not expose it in frontend code.
6. Ensure your account has sufficient credits. Credits can be purchased through the dashboard using Crypto, $CGPT or credit cards (fiat).

***

### Step 2: Install the SDK

ChainGPT provides SDKs for multiple tools. To get started with the Web3 AI Chatbot (General Chat API) using Node.js:

```
npm install @chaingpt/generalchat
```

For other tools such as the Smart Contract Generator, Auditor, or AI NFT Generator, refer to the corresponding SDK installation instructions in their documentation sections.

***

### Step 3: Make Your First API Call

Here is a basic example that sends a question to the Web3 Chatbot and streams the response:

```
import { GeneralChat } from '@chaingpt/generalchat';

const chat = new GeneralChat({ apiKey: 'YOUR_API_KEY' });

async function askChainGPT() {
  const stream = await chat.createChatStream({
    question: "What is DeFi?",
    chatHistory: "off"
  });

  stream.on('data', chunk => process.stdout.write(chunk.toString()));
  stream.on('end', () => console.log("\n[Done]"));
}

askChainGPT();
```

This call uses 0.5 credits by default. You can enable chatHistory or pass additional configuration parameters depending on your use case.

***

### Step 4: Integrate with Your Application

Once your initial request is successful, you can begin integrating ChainGPT’s AI tools into your application. Common integration patterns include:

* AI-powered chat assistants for DeFi, NFT, and on-chain analytics
* Automated smart contract auditing pipelines
* Real-time AI-generated crypto news widgets
* NFT generation tools with natural language prompts
* Internal compliance or support chatbots tailored to your product

Refer to individual tool guides for detailed implementation patterns.

**Full list of use-cases can be found here:**

{% content-ref url="use-cases-and-examples.md" %}
[use-cases-and-examples.md](use-cases-and-examples.md)
{% endcontent-ref %}

***

### Step 5: Monitor and Optimize

* Use the API Dashboard to monitor request history, credit usage, and key performance metrics.
* Implement error handling to manage low-credit conditions or invalid input.
* Adjust parameters such as system prompts, memory mode, or response style to fine-tune output.

***

### Next Steps

Explore each API and SDK in detail:

* [Web3 Chatbot (GeneralChat)](https://docs.chaingpt.org)
* [Smart Contract Generator](https://docs.chaingpt.org)
* [Smart Contract Auditor](https://docs.chaingpt.org)
* [AI NFT Generator](https://docs.chaingpt.org)
* [Crypto News API](https://docs.chaingpt.org)

For integration support or implementation guidance, contact the ChainGPT team or join the developer community.
