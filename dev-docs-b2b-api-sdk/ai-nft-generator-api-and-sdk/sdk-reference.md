# SDK Reference

## ChainGPT AI NFT Generator – SDK Reference

This reference guide covers the ChainGPT AI NFT Generator **JavaScript/TypeScript SDK**, detailing its installation, configuration, and all major functions. It focuses on using the SDK in Node.js or web applications (via bundlers) to generate AI-driven images and mint them as NFTs. (For a high-level overview of the AI NFT Generator’s capabilities, see the [Overview section](./); for HTTP endpoints, see the REST API Reference.)&#x20;

The SDK methods documented here provide a convenient wrapper around the ChainGPT AI NFT Generator API, handling lower-level details like HTTP requests and file handling. Each function is explained with parameters, return values, and code examples, including both educational snippets (with commentary) and production-ready snippets for quick copy-paste usage.

***

### Installation & Setup

**Install the SDK Package:** The ChainGPT NFT SDK is available as an npm package `@chaingpt/nft`. Use npm or Yarn to add it to your project:

```bash
npm install --save @chaingpt/nft
# or
yarn add @chaingpt/nft
```

This will install the SDK and its TypeScript type declarations.

**Import and Initialize the SDK:** The SDK exports an `Nft` class. Import this class and instantiate it with your API key (obtained from the ChainGPT developer dashboard):

```js
// Import the Nft class from the SDK
const { Nft } = require('@chaingpt/nft');  // CommonJS
// import { Nft } from '@chaingpt/nft';    // ES Module / TypeScript

// Initialize the SDK with your API key
const nft = new Nft({
  apiKey: 'YOUR_API_KEY_HERE',  // ChainGPT API Key for authentication
  // ... (other config options if any)
});
```

**Configuration:** Only an `apiKey` is required to start. The SDK handles authentication using this key for all calls. By default, the SDK will use ChainGPT’s hosted API endpoints. No additional setup is needed beyond providing a valid API key.

{% hint style="info" %}
**Note:** Ensure you have sufficient API credits or access as per your ChainGPT subscription to perform NFT generations. The SDK will throw errors (of type `NftError`) if requests fail due to invalid keys or quota issues (see **Error Handling** below).
{% endhint %}

***

### Generating AI Images (Without Minting)

The SDK allows you to generate AI art images from text prompts without immediately minting them as NFTs, using the `generateImage()` method. This is useful if you want to preview or use the AI-generated image first (e.g. for user approval) before committing to an on-chain mint.

#### `generateImage(options)`

**Description:** Generates an image from a text prompt using ChainGPT’s AI model, returning the image data. No blockchain interaction occurs in this step – it strictly performs image generation based on the given prompt and parameters. (If you want to directly prepare an NFT for minting, see `generateNft()` below.)

**Parameters:** _(all passed in a single **`options`** object)_

* **`prompt`** _(string, required)_ – The text prompt describing the desired image. This can be any creative description of the image you want.
* **`model`** _(string, required)_ – The name of the AI model to use for generation. Supported models include **`velogen`**, **`nebula_forge_xl`**, **`VisionaryForge`**, and **`Dale3`**. Each model has a different style or aesthetic (e.g. photorealistic, anime, 3D, etc.).
* **`enhance`** _(string, optional)_ – Image enhancement/upscaling level. Use **`"1x"`** for single enhancement, **`"2x"`** for double enhancement, or omit/empty for no enhancement. Enhancement can improve image resolution/quality, but is only supported by certain models (velogen, nebula\_forge\_xl, VisionaryForge).
* **`steps`** _(number, optional)_ – The number of diffusion/refinement steps for image generation. More steps can yield more detailed images at the cost of time. Each model supports a range: e.g. velogen supports 1–4 steps (default **2**), nebula\_forge\_xl up to 50 (default **25**). If not provided, the model’s default is used.
* **`width`** _(number, optional)_ – The width of the resulting image in pixels.
* **`height`** _(number, optional)_ – The height of the resulting image in pixels.

{% hint style="info" %}
_ℹ️ **Size & Aspect Ratio:** Supported dimensions depend on the model and enhancement. For example, **velogen** supports base resolutions like 512×512 (square), 768×512 (landscape), 512×768 (portrait), and with 2× enhancement up to 1920×1920, etc. Ensure the **`height`**×**`width`** you request matches one of the model’s supported aspect ratios to get the best results. (See **Image Generation Settings** below for more details on supported resolutions.) If not specified, a model-specific default (usually a square format) is used._
{% endhint %}

**Returns:** A **Promise** that resolves to an object containing the generated image data. The image data is typically provided as a binary buffer or byte array for the image (in **JPEG format** by default, which you can save as a `.jpg` file). You may access the raw bytes and save or manipulate the image as needed.

**Example – Generate an image and save to a file (with explanations):**

```js
const fs = require('fs');

// Define the image generation parameters
const params = {
  prompt: "A futuristic city skyline at sunset, neon pink and orange hues",  // Text description for the image
  model: "velogen",            // Choose the AI model (e.g., velogen for realistic visuals)
  enhance: "1x",               // Single enhancement for higher resolution output
  steps: 2,                    // Use 2 refinement steps (velogen default)
  width: 1024, height: 1024    // Generate a 1024x1024 image (square)
};

const result = await nft.generateImage(params);  
// The result contains the image data. Now, save it to disk:
fs.writeFileSync(
  "generated-image.jpg",                      // Output file path (using .jpg extension)
  Buffer.from(result.data.data)              // The image binary data buffer
);
console.log("Image saved to generated-image.jpg");
```

In this example, we generate a 1024×1024 image using the `velogen` model with a given prompt. The result’s `data` property holds the image bytes, which we write to a JPEG file. After running this code, the file `generated-image.jpg` will contain the AI-generated artwork.

**Example – Production-ready snippet:**

```js
// Simple image generation (production use)
const imgRes = await nft.generateImage({
  prompt: "A serene mountain landscape with a river",
  model: "nebula_forge_xl",
  height: 768, width: 1024  // produce a landscape-oriented image
});
// Save or utilize imgRes.data as needed (e.g., send to client or save to cloud storage)
```

This minimal snippet shows a straightforward use of `generateImage` to get an image for further use. In a real application, you might directly send `imgRes.data.data` as a response (in a web server) or display it in a frontend app (after encoding to Base64 or creating an object URL, etc.).

***

### Generating and Minting NFTs (End-to-End Workflow)

The core purpose of the ChainGPT NFT SDK is to generate images _and_ mint them as NFTs on a blockchain. The SDK provides two approaches:

* **Synchronous NFT generation** – generate the image (wait until complete) then mint it, suitable for one-off NFTs or immediate results.
* **Asynchronous (Queued) NFT generation** – start the generation and return immediately, allowing you to handle many generation jobs in parallel or off-load long-running tasks, then mint when ready.

In both cases, the minting step is performed via the `mintNft()` method once the image is generated. Below are the relevant SDK functions and how to use them.

#### `generateNft(options)` – Synchronous Generation

**Description:** Generates an NFT image from a prompt _with the intention to mint it_. This call will handle the AI image creation and prepare the NFT metadata, returning when the image is ready. You will typically call `mintNft()` afterwards to actually mint the NFT on-chain. Use this method for single NFT generation flows where the user or app can wait for the image to finish (typically 30–60 seconds). Under the hood, this wraps an API call that combines image generation and NFT metadata creation in one step.

**Parameters:** _(passed in an **`options`** object)_

* \*\*All image generation parameters from \*\***`generateImage()`** – `prompt`, `model`, `enhance`, `steps`, `height`, `width` are used in the same way as in `generateImage()` (see above) to control the AI generation.
* **`walletAddress`** _(string, required)_ – The public wallet address that will own the minted NFT. This is the address to which the NFT will be sent after minting. It should be a valid address on the target blockchain network.
* **`chainId`** _(number, required)_ – The blockchain network ID where the NFT will be minted. ChainGPT supports **25+ chains** including Ethereum, BNB Chain, Polygon, Arbitrum, Avalanche, Tron, etc. (see Overview for the full list). For example: 1 for Ethereum Mainnet, 56 for BSC Mainnet, 137 for Polygon, etc. You can use the `getChains()` method (documented below) to retrieve a list of supported chain IDs at runtime.
* _(No explicit **`name`**/**`description`**\*\*\*\*\*\*\*\*\*\* here – those will be provided in the **`mintNft()`** call instead.)_

**Returns:** A **Promise** resolving to an object with the generated NFT data. This includes the image data (similar to `generateImage` result) and a **`collectionId`** identifier for the generated NFT content. The `collectionId` is a unique ID referencing this generation task / NFT draft on the ChainGPT platform. You will use this ID to mint or check status later. The promise resolves only after the image generation is complete (synchronous behavior).

{% hint style="info" %}
**Note:** At this stage, the NFT is **not yet minted on-chain**. The `generateNft()` call prepares everything (image + metadata). You must call `mintNft()` with the `collectionId` (and desired NFT name/description) to actually mint the token.
{% endhint %}

**Example – Generate an NFT then mint it (with explanations):**

```js
// 1. Generate the NFT image and metadata
const genResult = await nft.generateNft({
  prompt: "A cute cartoon cat astronaut, sticker illustration",
  model: "VisionaryForge",
  enhance: "",            // no enhancement
  steps: 25,              // use 25 steps refinement
  height: 1024, width: 1024,
  walletAddress: "0xABC123...DEF",  // recipient wallet for the NFT
  chainId: 97             // BSC Testnet (chainId 97 in this example)
});
console.log("NFT generation completed. Collection ID:", genResult.data.collectionId);

// 2. Mint the generated NFT on-chain using the returned collectionId
const mintResult = await nft.mintNft({
  collectionId: genResult.data.collectionId,   // ID from generateNft step
  name: "Space Cat Sticker",                   // Name of the NFT (metadata)
  description: "An AI-generated cat astronaut sticker, created via ChainGPT",
  symbol: "CAT"                                // Symbol for the NFT (if a new collection/contract is created)
});
console.log("NFT minted! Transaction/result:", mintResult);
```

In this example, we first call `generateNft()` with the prompt and required parameters. Once the promise resolves, we have an image ready and a `collectionId` that identifies this generated NFT data. We then call `mintNft()` with that ID and provide a human-friendly name, description, and symbol for the NFT. The SDK handles the on-chain minting transaction: the NFT will be minted on **BSC Testnet** (chain 97) to the specified wallet address. The `mintResult` may include details of the mint operation (such as transaction hash or token ID, depending on the implementation).

**Example – Production snippet (generate then mint):**

```js
// Generate NFT (synchronously) and then mint it
const { data: nftData } = await nft.generateNft({ 
  prompt: "Epic dragon portrait in medieval style", model: "velogen",
  walletAddress: "0x...userWallet", chainId: 56 
});
await nft.mintNft({
  collectionId: nftData.collectionId,
  name: "Epic Dragon", description: "AI-generated medieval dragon art", symbol: "DRGN"
});
```

This snippet omits commentary for brevity. It generates an NFT on BNB Chain (chainId 56) for the given wallet, then immediately mints it with a name and symbol. In a real app, you may want to handle errors around these calls or update your UI state between generation and minting steps.

#### `generateNftWithQueue(options)` – Asynchronous/Queued Generation

**Description:** Initiates an NFT generation task asynchronously (non-blocking). Instead of waiting for the image to finish generating, this function returns quickly with a `collectionId` that represents the queued job. The actual image generation happens in the background on ChainGPT’s servers. You can then poll the status of the job using `getNftProgress()` and mint the NFT when ready. This approach is ideal for **batch generation** or handling high volumes, as you can queue up multiple NFTs in parallel without halting your code flow.

**Parameters:** Same as `generateNft()` above – you must provide `prompt`, `model`, any optional generation settings (`enhance`, `steps`, `height`, `width`), plus the target `walletAddress` and `chainId` for the eventual mint.

**Returns:** A **Promise** that resolves to an object containing at least a **`collectionId`** for the queued generation request. The image will **not** be available immediately. You will use the `collectionId` to track progress and later mint the NFT. The function returns as soon as the generation task is successfully queued on the server.

**Example – Queue an NFT generation and check progress (with explanations):**

```js
// 1. Start the NFT generation job (async queue)
const queueResult = await nft.generateNftWithQueue({
  prompt: "Abstract colorful swirl painting, high resolution",
  model: "nebula_forge_xl",
  walletAddress: "0xDEF456...789",
  chainId: 1   // Ethereum Mainnet
});
const jobId = queueResult.data.collectionId;
console.log("Generation queued. Collection ID:", jobId);

// 2. Periodically check the generation progress
let progressInfo;
do {
  progressInfo = await nft.getNftProgress({ collectionId: jobId });
  const percent = progressInfo.data.progress;   // e.g., progress percentage
  console.log(`Generation progress: ${percent}%`);
  await new Promise(res => setTimeout(res, 5000));  // wait 5 seconds before checking again
} while(progressInfo.data.status !== "completed");
// (Loop until the status indicates completion; progress may go from 0 to 100%)

console.log("Generation complete. Now minting the NFT...");
// 3. Mint the NFT now that generation is done
await nft.mintNft({
  collectionId: jobId,
  name: "Color Swirl Art",
  description: "AI-generated abstract painting",
  symbol: "ART"
});
console.log("NFT minted on chain.");
```

In this example, we fire off an NFT generation using `generateNftWithQueue()`. We immediately get a `collectionId` for the job and then enter a loop, using `getNftProgress()` to poll the job’s status. Once the progress status indicates completion (100% or `"completed"`), we call `mintNft()` with the same `collectionId` to mint the NFT on-chain. The polling interval here is 5 seconds, but in a real application you might use a longer interval or a webhook/callback mechanism if provided. You can also queue many jobs and poll each in turn or in parallel, enabling scalable generation of large NFT collections.

**Example – Production snippet (queue generation):**

```js
// Queue NFT generation (non-blocking)
const { data: queued } = await nft.generateNftWithQueue({ 
  prompt: "Portrait of an AI robot", model: "velogen", walletAddress: "0x...addr", chainId: 137 
});
// Immediately get an ID for tracking (no waiting for image)
console.log("Queued job ID:", queued.collectionId);
```

The above snippet demonstrates the basic queued call. After this, you would typically use `getNftProgress()` later and then `mintNft()`. For brevity, those parts are omitted here (see the detailed example above for the full flow).

#### `getNftProgress(options)` – Check Generation Progress

**Description:** Checks the status of an ongoing NFT generation job that was started with `generateNftWithQueue()`. It returns the progress or completion status of the image generation. This is typically used in a loop or interval to poll for completion if you are not using a callback system.

**Parameters:**

* **`collectionId`** _(string, required)_ – The unique ID of the NFT generation job to query. This is the ID returned by `generateNftWithQueue()` (or `generateNft()` as well, if you choose to poll even for synchronous calls).

**Returns:** A **Promise** resolving to an object containing progress information. The exact fields might include a percentage or status indicator. For example, you may receive `{ progress: 100, status: "completed", ... }` when done. Before completion, it might show a lower percent or a status like `"processing"`. Once the generation is finished, you can proceed to mint the NFT. If an invalid ID is provided or the job failed, an error or failure status will be returned.

**Example – Check progress (basic usage):**

```js
const progressRes = await nft.getNftProgress({ collectionId: jobId });
if (progressRes.data.progress === 100) {
  console.log("Job completed!");
}
```

**Production snippet:**

```js
// Polling example (single check)
const status = await nft.getNftProgress({ collectionId: "your-collection-id" });
console.log("Current progress:", status.data.progress, "%");
```

Typically, you will call this repeatedly until completion, as shown in the more detailed example under `generateNftWithQueue()`. Use this method to inform users about wait times or to trigger the next step once the image is ready.

#### `mintNft(options)` – Minting the Generated NFT

**Description:** Mints an NFT on the blockchain for a previously generated image/metadata (identified by a `collectionId`). This completes the NFT creation process by writing the token to the blockchain. You should call this after generating the NFT via either `generateNft()` or `generateNftWithQueue()`. The method also allows you to specify the NFT’s metadata details like name, description, and symbol.

**Parameters:**

* **`collectionId`** _(string, required)_ – The ID of the generated NFT content to mint. This comes from a prior call to `generateNft`/`generateNftWithQueue`.
* **`name`** _(string, required)_ – The name of the NFT to mint. This could be the title of the artwork or a unique name for this token.
* **`description`** _(string, optional)_ – A description for the NFT’s metadata (e.g. details about the image, project, etc.). This will be included in the token’s metadata JSON.
* **`symbol`** _(string, optional)_ – The symbol or shorthand for the NFT collection. If the minting involves deploying a new NFT collection (contract), this symbol will be used for the collection (similar to a ticker symbol). For one-off mints it can be a short identifier.

**Returns:** A **Promise** resolving to an object containing the result of the mint operation. On success, this might include transaction details, the minted token ID, or a confirmation that the NFT has been minted. If the SDK handles the blockchain transaction internally, the promise will only resolve when the transaction is confirmed or at least submitted. (Refer to ChainGPT’s docs for specifics on whether this auto-deploys a new contract or mints on an existing one – the SDK abstracts those details).

After calling `mintNft()`, the image generated by the given `collectionId` is now an actual NFT on the specified blockchain, owned by the provided wallet address. The NFT’s metadata (name, description) is set as given.

**Example – Minting an NFT (immediate usage):**

```js
await nft.mintNft({
  collectionId: "abcd1234-ef56-...-xyz",  // (example ID from prior generation)
  name: "My AI NFT",
  description: "This NFT was generated by AI via ChainGPT",
  symbol: "AINFT"
});
console.log("Mint successful!");
```

**Production snippet:**

```js
// Finalize NFT mint
const result = await nft.mintNft({ collectionId, name: "Artwork #1", symbol: "ART" });
```

In practice, you would incorporate error handling to catch any issues during minting (network errors, insufficient funds if any gas needed, etc.). See **Error Handling** for how to catch `NftError` exceptions from this call.

***

### Prompt Enhancement (Improve Prompt Quality)

ChainGPT’s NFT Generator provides a **Prompt Enhancement** feature to help users get better image generation results. The SDK exposes this via the `enhancePrompt()` method, which takes a raw prompt and returns a more detailed or refined version of it. This can be useful if you want to assist users in crafting prompts or to programmatically improve prompt inputs before generating images.

#### `enhancePrompt(options)`

**Description:** Enhances or refines a given text prompt by adding more details, context, or creative wording. The result is a new prompt that is likely to produce a higher-quality or more imaginative image from the AI. (The platform may use its own AI logic to do this enhancement.)

**Parameters:**

* **`prompt`** _(string, required)_ – The original prompt text to enhance. This should be a short description of an image or concept.

**Returns:** A **Promise** resolving to an object containing the enhanced prompt. The returned prompt may include additional adjectives, context, or formatting. You can directly use the enhanced prompt with `generateImage` or `generateNft` to potentially get better results.

**Example – Enhance a prompt (with explanations):**

```js
const userPrompt = "a small cottage by the lake";
console.log("Original prompt:", userPrompt);

const enhancedRes = await nft.enhancePrompt({ prompt: userPrompt });
const improvedPrompt = enhancedRes.data.prompt;  // assuming .prompt contains the enhanced text
console.log("Enhanced prompt:", improvedPrompt);

// You can now use improvedPrompt for image generation:
await nft.generateImage({ prompt: improvedPrompt, model: "velogen", width:512, height:512 });
```

In this example, a simple prompt _"a small cottage by the lake"_ might be automatically expanded to something like _"a **cozy** small cottage by the **misty** lake at sunrise, **photorealistic**, high detail"_ (the actual output will vary). Using the enhanced prompt in `generateImage` or `generateNft` could yield more vivid results.

**Production snippet:**

```js
// Quickly get an enhanced prompt for user input
const { data: enhanced } = await nft.enhancePrompt({ prompt: "portrait of a dragon" });
console.log("Suggestion:", enhanced.prompt);
```

This shows how you might call `enhancePrompt` in production – for example, to implement a "Suggest improvements" button for user-provided prompts. The output prompt (accessible as `enhanced.prompt` or a similar field) can then be shown to the user or used directly.

_(The SDK may also offer a “Surprise Me” or random prompt generator feature as hinted in the overview, but the primary prompt utility in the SDK is **`enhancePrompt`**.)_

***

### Retrieving Blockchain Chain Info

If you need to know which blockchains are supported or get their identifiers (especially to let users choose a network for minting), the SDK provides a method to fetch the list of supported chains.

#### `getChains(includeTestnet)`

**Description:** Returns a list of blockchain networks (chain IDs) supported by the NFT generator. You can choose to include test networks or not. This is useful for populating network selection dropdowns or validating a `chainId` before attempting a mint.

**Parameters:**

* **`includeTestnet`** _(boolean, optional)_ – If `false` (default), returns only mainnet chains. If `true`, test networks are included as well. Use `true` if you need testnet IDs for development.

**Returns:** A **Promise** resolving to an array of chain information. Each entry will typically include at least a **chain ID** (number) and possibly a **name** or network identifier. For example, an entry might be `{ chainId: 1, name: "Ethereum Mainnet" }`, `{ chainId: 56, name: "BSC Mainnet" }`, or `{ chainId: 97, name: "BSC Testnet" }`, etc. The exact structure can be determined by inspecting the returned object. All networks supported by ChainGPT’s NFT Generator will be listed.

**Example – Get supported chains:**

```js
const chains = await nft.getChains(false);  // fetch mainnet chains
chains.data.forEach(chain => {
  console.log(chain.chainId, chain.name);
});
```

This might output a list such as `1 Ethereum, 56 BNB Chain, 137 Polygon, ...` etc., covering all mainnet networks available. If you call `getChains(true)`, you will see testnets (e.g., Ropsten, Goerli for Ethereum if supported, Mumbai for Polygon, etc.) in the list as well.

**Production snippet:**

```js
// Fetch all supported networks (including testnets)
const { data: allChains } = await nft.getChains(true);
const chainIds = allChains.map(c => c.chainId);
console.log("Supported chain IDs:", chainIds);
```

This quickly retrieves the full list of supported chain IDs (including test networks). You might use this to validate user choices or to know if a particular chainId is currently supported by ChainGPT’s service.

***

### Accessing the NFT Contract ABI

ChainGPT’s NFT Generator mints tokens using its own smart contract (often a factory or template contract). If you need to interact with the NFT contract directly (for example, to call the mint function manually or to integrate with web3 libraries for events), you can retrieve the contract’s **ABI (Application Binary Interface)** via the SDK.

#### `abi()`

**Description:** Fetches the ABI for the ChainGPT NFT Mint Factory contract. The ABI is a JSON specification of the contract’s functions and events, which is required to create a web3 contract instance (e.g., with ethers.js or web3.js) to call contract methods or decode logs.

**Parameters:** None.

**Returns:** A **Promise** that resolves to the ABI data (likely an array of JSON fragments describing each function/event). This ABI corresponds to the smart contract that the SDK uses under the hood to mint NFTs. With this, developers can instantiate the contract in their own web3 context if needed.

**Example – Retrieve contract ABI:**

```js
const abiData = await nft.abi();
console.log("ABI length:", abiData.data.length, "ABI entries");
```

This will output the number of entries in the ABI (functions, events). You could then do something like:

```js
// (Using ethers.js for example)
const { ethers } = require("ethers");
const provider = new ethers.JsonRpcProvider(YOUR_RPC_URL);
const contractAddress = "0x...";  // address of the NFT contract (if known or obtained from mint result)
const contract = new ethers.Contract(contractAddress, abiData.data, provider);
```

Now `contract` is a read-only instance (or if you have a signer, read-write) of the NFT contract, and you can call functions or listen to events as defined by the ABI.

**Production snippet:**

```js
// Get the NFT factory contract ABI
const { data: contractAbi } = await nft.abi();
// You can now integrate this ABI with web3 libraries as needed
```

Typically, you might not need to call `abi()` unless you are doing advanced integrations. The SDK’s own methods (`generateNft`, `mintNft`, etc.) will handle contract interactions for you. But the option is there for completeness or custom use cases.

***

### Image Generation Settings Reference

_(This section provides reference details on the supported models, enhancements, and image formats in the ChainGPT NFT Generator SDK.)_

*   **Supported AI Models:** The following model identifiers can be used for the `model` parameter when generating images or NFTs:

    * **`velogen`** – for realistic or general-purpose image generation.
    * **`nebula_forge_xl`** – for high-resolution, detailed imagery.
    * **`VisionaryForge`** – for creative/artistic styles (e.g., concept art).
    * **`Dale3`** – (integration of OpenAI’s DALL·E 3 or similar) for varied art generation.

    Each model may have different strengths or styles (e.g., _Dale3_ might excel at certain creative compositions). You can experiment with different models to achieve the desired visual style.
* **Image Formats:** Generated images are returned in a binary format that can be saved as common image types. By default, images are in **JPEG** format (as indicated by the `.jpg` usage in examples) when using the provided API endpoints. The SDK does not currently provide an option to choose the file format (such as PNG) via parameters, so you can assume JPEG output. If a different format is needed, you may convert the image bytes using an image processing library after retrieval.
*   **Enhancement (`enhance`\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* parameter):** As noted, you can request upscaling enhancements:

    * `"1x"` – Single enhancement (upscale once).
    * `"2x"` – Double enhancement (upscale twice for even higher resolution).

    Only certain models support enhancement (currently **velogen**, **nebula\_forge\_xl**, **VisionaryForge**). If you use `enhance` on an unsupported model, the enhancement may be ignored. Using enhancement will yield larger image dimensions (roughly doubling the resolution in each step).
*   **Resolution and Aspect Ratios:** Each model has preset supported resolutions:

    * _Velogen:_ Base outputs: **512×512** (square), **768×512** (landscape), **512×768** (portrait). With 2× enhancement: **1920×1920**, **1920×1280**, **1280×1920**.
    * _Nebula Forge XL:_ Base outputs: **1024×1024**, **1024×768**, **768×1024**. With upscaling: **1536×1536**, **1536×1024**, **1024×1536**.
    * _VisionaryForge:_ Base outputs: **1024×1024**, **1024×768**, **768×1024**. With upscaling: **1536×1536**, **1536×1024**, **10**
    * _Dale3:_ (Not explicitly listed in the SDK docs, but it likely supports standard square sizes such as 1024×1024. You may refer to ChainGPT’s documentation for any specifics.)**24×1536**.

    When specifying `height` and `width`, choose one of the supported combinations above for the model you use. If you choose a non-supported resolution, the API may return an error or adjust to the nearest valid size. For simplicity, common squares (512 or 1024) are safe across models.
* **Default Values:** If certain optional parameters are omitted:
  * `enhance`: no enhancement (equivalent to not upscaling).
  * `steps`: uses model default (e.g., 2 for velogen, 25 for nebula\_forge\_xl).
  * `height`/`width`: likely defaults to the model’s base square resolution (e.g., 512×512 or 1024×1024 depending on model). It’s recommended to specify them explicitly to control the aspect ratio.

Use these references to fine-tune your image generation. For instance, if you want a large portrait image, using `model: "nebula_forge_xl", enhance: "2x", width:1024, height:1536` would produce an upscaled portrait orientation result. If unsure, start with a base model and resolution (e.g., 512×512 velogen) and then adjust parameters for higher quality.

***

### Error Handling

The ChainGPT NFT SDK provides a structured way to handle errors. Network issues, invalid requests, or API errors will result in the SDK throwing an instance of `Errors.NftError`. You should catch these errors in your async functions to handle them gracefully.

**Error type:** `Errors.NftError` – This error object (a class instance) includes a message describing the issue (e.g., an HTTP error status or a message from the API). It may also include additional data or codes from the response.

**How to catch:** Use a `try...catch` around SDK calls. In the `catch`, check if `error instanceof Errors.NftError` to confirm it’s an SDK error and not a different exception.

**Example – Error handling pattern:**

```js
import { Errors } from '@chaingpt/nft';

try {
  const response = await nft.generateNft({ /* ...parameters... */ });
  // Use response...
} catch (error) {
  if (error instanceof Errors.NftError) {
    console.error("ChainGPT SDK error:", error.message);
    // Optionally inspect error.response or error.code if available
  } else {
    // Some other error (programming error, etc.)
    console.error("Unexpected error:", error);
  }
}
```

In this snippet, if the `generateNft` call fails (for example, due to an invalid API key, network timeout, or a 400/500 HTTP error from the API), the catch block will execute. The error message (`error.message`) may contain a human-readable explanation of what went wrong. Always handle errors especially for network calls like generation or minting, to ensure your application can inform the user or retry as needed.

**Common error scenarios:**

* _Authentication failure:_ If your API key is wrong or expired, you’ll get an `NftError` (likely with a 401/403 status message).
* _Invalid parameters:_ If you request an unsupported image size or missing required fields, the API may return an error (400 Bad Request), resulting in an `NftError`.
* _Generation failure:_ In rare cases the AI might fail to generate an image; the API could return an error or a status indicating failure for that job.
* _Network issues:_ If the SDK cannot reach the server, it will throw an `NftError` (with a message like network error or timeout).

By catching `NftError`, you can distinguish errors coming from ChainGPT’s API versus other exceptions. You can also use the error information to decide retries or user messages. The SDK’s error handling mechanism ensures you don’t have to manually parse HTTP responses – it consistently throws `NftError` for anything that isn’t a normal successful response.

***

### TypeScript Support

The ChainGPT NFT SDK is written in TypeScript and comes with full type declarations. This means you get autocompletion and type checking out-of-the-box when using the SDK in a TypeScript project or any editor with TypeScript definitions.

* All methods (`generateImage`, `generateNft`, etc.) have typed function signatures. The expected input object shape and return types are defined, which helps catch mistakes (for example, forgetting a required field or using the wrong type for a parameter).
* The `Nft` class constructor options (`apiKey` etc.) are typed as well. You can import types if needed for your own definitions.

**Using ESM/TypeScript import:**

```ts
import { Nft, Errors } from '@chaingpt/nft';

const nft = new Nft({ apiKey: 'YOUR_API_KEY' });
// The type of nft is Nft, which has methods like generateImage, generateNft, etc., all typed.

// Example: auto-complete will show the required fields for generateNft when you start typing.
const result = await nft.generateNft({
  prompt: "TS check example", 
  model: "velogen",
  walletAddress: "0x123...ABC",
  chainId: 5,
  // If you omit a required field or use wrong type, TypeScript will error at compile time.
});
```

In the above TypeScript example, your IDE will guide you with the correct parameter names and types for `generateNft`. The return type `result` will also be known (you can inspect its type, likely something like `NftResponse` with a `.data` field).

**Typing for Errors:** The `Errors.NftError` class is also available for type checking. You can do `catch (error: any) { if (error instanceof Errors.NftError) { ... } }` – TypeScript will then infer `error` as `NftError` inside that block, allowing you to access any specific properties it has.

Because of the included d.ts files, you **do not** need to install any additional `@types` packages for this SDK. Everything is bundled in `@chaingpt/nft` by default.

***

{% hint style="info" %}
With the above reference, you should be well-equipped to integrate ChainGPT’s AI NFT Generator via the SDK. By following the examples and utilizing the SDK’s functions, developers can programmatically generate unique AI-driven images, refine prompts, and mint NFTs across multiple blockchains with minimal effort. For further details on the underlying API or advanced use-cases, refer to the full documentation and the Overview section.&#x20;
{% endhint %}

**Happy building with ChainGPT’s NFT SDK!**
