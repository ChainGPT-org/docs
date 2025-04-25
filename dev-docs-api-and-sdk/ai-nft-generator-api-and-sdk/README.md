# AI NFT Generator (API & SDK)

## ChainGPT AI NFT Generator (API & SDK) Overview

ChainGPT’s **AI NFT Generator** is a full-stack API and SDK for creating, minting, and managing NFTs through AI. It’s the first solution to combine AI-driven image generation with on-chain NFT minting in one platform, so developers can go from an idea to a blockchain asset seamlessly. With a single integration, you can generate unique images from text prompts, convert them into NFT metadata, and mint them on multiple blockchains – all without needing separate AI models or smart contract code​. This empowers developers, product teams, and Web3 projects to launch NFT features faster, using ChainGPT’s infrastructure to handle the heavy lifting.

***

### Key Use Cases

* **Single NFT Generation:** Create one-off AI-generated NFTs on demand. For example, users in a game or app can input a prompt to instantly get a custom NFT avatar or digital collectible. The API returns the generated image and token data, ready to mint or display.
* **Full Collection Creation:** Generate entire NFT collections (up to 10,000 images) in one go​. The AI can produce large sets of art with variation, which you can mint as a series (e.g. generative profile picture collections). This dramatically speeds up launching an NFT project – what used to take weeks of manual design can happen in under a minute.
* **Integrated Minting:** Every image comes with **mint-ready** metadata. You can directly mint the AI-generated art as an NFT on the chosen blockchain via the API/SDK, without writing custom smart contracts. The service handles token creation, storage of the image/metadata, and linking it all on-chain.
* **Metadata Control:** Developers retain control over NFT metadata. By default the system generates standard metadata (e.g. name, description, traits) for each NFT, but you can override or post-edit attributes as needed. This is useful for injecting project-specific traits or ensuring metadata consistency across a collection.
* **Multi-Chain Deployment:** Easily deploy NFTs across **multiple blockchains** from the same API. The AI NFT Generator supports **BNB Chain (BSC)** and **opBNB**, **Ethereum**, **Polygon**, **Arbitrum One**, **Avalanche**, **Cronos**, **Tron**, **Base**, and many more – over 25 networks in total​. You can specify a target chain ID for each mint, enabling cross-chain NFT support without additional tools.

***

### How It Works

<div align="left"><figure><img src="../../.gitbook/assets/image (1).png" alt="" width="403"><figcaption></figcaption></figure></div>

{% hint style="info" %}
_**High-level workflow:** the AI NFT Generator handles everything from prompt to minting._
{% endhint %}

The process starts with a **text prompt** provided by the developer or user, describing the desired artwork. The AI model then generates an **image** based on this prompt (using advanced text-to-image algorithms). Next, the system creates an NFT **metadata** JSON for the image – including details like title, description, and any traits, plus a link to the image file. With one API call, the platform mints a new **NFT token** on the specified blockchain, attaching the generated image and metadata to it. Finally, your NFT collection can be added to your wallet and used as you wish! All these steps are orchestrated behind a single API/SDK interface, so you don’t need to manage them separately.

***

### Integration Options: API vs SDK

ChainGPT offers both a RESTful API and a client SDK, so you can choose the integration path that fits your stack:

* **REST API:** Integrate via HTTP endpoints accessible in any language or platform. This option is ideal if you want direct control or are working in a backend environment outside of Node.js. You send requests with the prompt, parameters, and target chain, and receive the generated image/metadata or transaction details in JSON responses. (If you only need image generation without minting, a lightweight **text-to-image** API endpoint is available as well.) See the **API Reference** for full endpoint details and examples.

{% content-ref url="api-reference.md" %}
[api-reference.md](api-reference.md)
{% endcontent-ref %}

* **JavaScript/TypeScript SDK:** Use ChainGPT’s official SDK package for Node.js projects. The SDK provides convenient methods that wrap the API calls – for example, `generateImage()` or `generateNft()` – abstracting away raw HTTP calls and handling tasks like file I/O. It’s well-suited for frontend web apps, Node backends, or scripts, and includes TypeScript type definitions for safer development. The SDK simplifies common workflows (such as queuing batch jobs or tracking generation progress) with one-line function calls. Refer to the **SDK Reference** for installation and code snippets.

{% content-ref url="sdk-reference.md" %}
[sdk-reference.md](sdk-reference.md)
{% endcontent-ref %}

{% hint style="info" %}
_(Both the API and SDK require an API key for authentication. You can obtain this key from the ChainGPT_ [_developer dashboard_](https://app.chaingpt.org/apidashboard) _– see “Get Started” below.)_
{% endhint %}

***

### Supported Blockchains

One of the AI NFT Generator’s strengths is broad **multichain support**. Out of the box, you can mint NFTs on a wide range of networks without any custom setup. Supported chains include:

* **Ethereum** – Mainnet and testnets
* **Binance Chain (BNB)** – and **opBNB** layer 2
* **Polygon** (Ethereum sidechain)
* **Arbitrum One** and other L2s
* **Avalanche**, **Cronos**, **Hedera**, **Immutable X**
* **Tron**, **Mantle**, **CoreDAO**, **Linea**
* **Base**, **Scroll**, **Skale**, **5ire**, and more...

In total, 25+ blockchain networks are integrated. The API/SDK uses a simple `chainId` parameter to select the target network, so switching the mint destination (or offering a choice to your users) is straightforward. ChainGPT handles the low-level differences between chains, providing a unified NFT minting experience across ecosystems.

***

### Unique Capabilities

ChainGPT’s AI NFT Generator comes with several powerful features that distinguish it from basic image APIs:

* **Prompt Enhancement & Suggestions:** To help improve results, the platform includes an “**Enhance Prompt**” option that refines your input text for a better image output. There’s also a “Surprise Me” feature that suggests random creative prompts, which can inspire users or generate diverse outputs without extra input. These tools lower the barrier for non-artists to get high-quality NFTs.
* **Batch Queueing for Collections:** When generating large collections, you can use the **queue** system to process images asynchronously. Rather than waiting on one NFT at a time, the SDK allows queuing up generation tasks (e.g. `generateNftWithQueue()` in the SDK) so you can request hundreds or thousands of images in parallel​file-leehq5gwmzjy2dqf8wha2q. The API provides methods to check generation progress and retrieve results when ready, enabling scalable NFT creation workflows.
* **Flexible Image Output Settings:** You have control over the size and aspect ratio of generated images. The generator supports multiple aspect ratios (e.g. 1:1 square, 3:2 rectangle, etc.) and custom dimensions up to high resolutions​. It also offers built-in **upscaling** options (1× or 2×) to enhance image resolution after generation. This flexibility means you can tailor the output to your platform’s requirements (thumbnail vs. high-res print, etc.).
* **Multiple AI Art Models:** The service provides a selection of AI models/styles to generate art in different aesthetics. For example, you might choose a _Realistic_ model for photorealistic art, an _Anime_ model for cartoon-style NFTs, a _3D Render_ model for game assets, or others like _Drawing_ or _Pixel Art_. By selecting the model best suited to your use case, you can maintain a consistent style across your NFTs or offer users a choice of artistic styles.
* **Advanced Collection Generation:** Beyond single images, the AI NFT Generator can automate trait assignment and rarity for generative collections. You can supply base assets or let the AI create variations, and the system will combine traits to produce unique NFTs with rarity distribution​. This is particularly useful for profile-picture (PFP) projects that involve layering attributes (hats, glasses, backgrounds, etc.) across thousands of NFTs. The heavy lifting of randomizing traits and ensuring each NFT is one-of-a-kind is handled for you.

***

### Developer-Friendly Design

The AI NFT Generator API & SDK are built with developers in mind, focusing on quick integration and robust performance:

* **Fast & Efficient:** Generation and minting are optimized for low latency. In typical cases, an NFT (image + on-chain mint) can be produced in **30–60 seconds**. Even bulk operations use parallel processing and queueing to finish quickly. This speed lets you integrate on-demand NFT creation in real time applications (like letting a user design an NFT and mint it during a sign-up flow) without long waits.
* **No AI/ML Expertise Needed:** You don’t have to train models or manage machine learning pipelines. ChainGPT’s models are pre-trained and continuously improved by the team, so you simply leverage them via API. There’s no need for you to gather datasets, run GPU servers, or tweak neural network parameters – the service delivers great results out-of-the-box.
* **Customizable Outputs:** While the default settings work for most cases, developers can fine-tune the generation process through parameters. You can adjust the diffusion **steps** (to control image refinement), toggle **enhancements**, select different generator **models**, and set image dimensions. The output NFTs can thus be customized to fit your application’s style and quality requirements.
* **Straightforward Onboarding:** Getting started is simple. Just obtain an API key from ChainGPT’s developer portal and you can begin calling the API or SDK immediately (no complex setup). The SDK can be installed with a single npm/yarn command and comes with clear documentation. Both the REST API and SDK use an intuitive request structure. For example, to generate an NFT you provide the prompt, choose a model, and specify a wallet address & chain – the response returns the image data or a mint transaction. Detailed guides and code examples are available in the docs to accelerate your integration.

***

### Next Steps

* **Quickstart Guide:** Follow the step-by-step Quickstart to create your first AI-generated NFT using the API/SDK. This guide covers setting up your API key, making a test generation call, and minting a sample NFT (ideal for newcomers).&#x20;

{% content-ref url="quickstart-guide.md" %}
[quickstart-guide.md](quickstart-guide.md)
{% endcontent-ref %}

* **API Reference:** Refer to the API reference for a complete list of endpoints, parameters, and response schemas. Each feature (image generation, minting, etc.) is documented with example requests and responses to help you build correctly.

{% content-ref url="api-reference.md" %}
[api-reference.md](api-reference.md)
{% endcontent-ref %}

* **SDK Reference:** Check out the SDK documentation for usage of the JavaScript/TypeScript library. It includes code snippets for all major functions (generate image, mint NFT, queue jobs, etc.) and best practices for integrating the SDK into your project.

{% content-ref url="sdk-reference.md" %}
[sdk-reference.md](sdk-reference.md)
{% endcontent-ref %}

* **Pricing & Credits:** ChainGPT’s AI NFT Generator API uses a simple and transparent credit system. Each API request consumes credits based on the model selected, generation options, and image resolution. Learn more about our pricing.

{% content-ref url="pricing-and-credits.md" %}
[pricing-and-credits.md](pricing-and-credits.md)
{% endcontent-ref %}

{% hint style="info" %}
**Get Started:** To start building with ChainGPT’s AI NFT Generator, sign up for an API key via the ChainGPT web app and review the Quickstart Guide. With just a few lines of code, you can integrate AI-powered NFT creation and minting into your product today. **Enjoy exploring the possibilities and happy coding!**
{% endhint %}
