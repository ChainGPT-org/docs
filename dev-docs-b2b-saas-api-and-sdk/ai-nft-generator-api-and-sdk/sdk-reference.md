# SDK Reference

## ChainGPT AI NFT Generator SDK Reference

### Introduction

The ChainGPT AI NFT Generator SDK provides a JavaScript/TypeScript interface for generating AI-driven images and minting them as NFTs. It wraps the ChainGPT AI NFT Generator REST API, allowing developers to easily integrate image generation and NFT creation into their applications without dealing with raw HTTP requests or file handling. This reference guide covers the SDK's installation, configuration, and major functions, focusing on how to use the SDK in Node.js or web environments to create and mint AI-generated images.

**Key capabilities:**

* Generate AI art images from text prompts using ChainGPT’s proprietary models.
* Optionally enhance (upscale) images for higher resolution outputs.
* Create NFTs by generating an image _and_ preparing it for minting in one flow.
* Mint the generated image as an NFT on various blockchain networks with a single call.
* Check generation progress for queued jobs and retrieve supported blockchain info.
* Enhance text prompts for better image generation results.

_Note:_ This guide is for the **SDK usage**. If you prefer to call the REST API directly, refer to the API Reference. For a step-by-step tutorial on using the NFT Generator (with examples), see the QuickStart Guide. Ensure you have an API key from the ChainGPT platform and sufficient credits (or an appropriate subscription plan) before using the SDK (the SDK will throw an error if your API key is invalid or you run out of credits).

***

### Installation

**Install the SDK Package:** The ChainGPT NFT Generator SDK is available as an npm package `@chaingpt/nft`. Add it to your project using npm or Yarn:

```bash
npm install --save @chaingpt/nft
# or
yarn add @chaingpt/nft
```

This will install the SDK and its TypeScript type declarations.

**Import and initialize the SDK:** The SDK exports an `Nft` class. Import this class and instantiate it with your API key (obtained from the ChainGPT developer dashboard):

```js
// Import the Nft class from the SDK
const { Nft } = require('@chaingpt/nft');  // CommonJS
// import { Nft } from '@chaingpt/nft';    // ES Module / TypeScript

// Initialize the SDK with your API key
const nft = new Nft({
  apiKey: 'YOUR_API_KEY_HERE'  // ChainGPT API Key for authentication
  // ... you can include other config options if supported
});
```

Only the `apiKey` is required to configure the SDK. The SDK uses this key to authenticate your requests to ChainGPT’s API. By default, it will use ChainGPT’s hosted endpoints—no additional setup is needed beyond providing a valid API key.

**Note:** Keep your API key secure (do not hard-code it in client-side code or expose it publicly). Also ensure you have sufficient API credits or an active plan to perform NFT generations. If your key is invalid or you exceed your quota, the SDK will throw an `Errors.NftError` (see **Error Handling** below).

***

### SDK Usage

Below are the primary methods provided by the `Nft` SDK instance. Each method is documented with its purpose, expected parameters, return values, and example usage.

#### `generateImage(options)`

**Description:** Generates an image from a text prompt using one of ChainGPT’s AI models, returning the image data. This function **does not** mint an NFT; it only performs image generation. Use `generateImage` when you want to create an image (for example, to let a user preview it) without immediately putting it on-chain. To generate an image _and_ directly prepare it for minting, use `generateNft()` instead.

**Parameters:** _(passed as properties of the `options` object)_

* `prompt` (string, **required**): The text prompt describing the desired image. This can be any creative description of the image you want (e.g. _"A futuristic city skyline at sunset, neon pink and orange hues"_).
* `model` (string, **required**): The identifier of the AI model to use for generation. Supported models include **`velogen`**, **`nebula_forge_xl`**, **`VisionaryForge`**. Each model has a different style or aesthetic (e.g. photorealistic, anime, 3D render, etc.). _ChainGPT’s proprietary models are `velogen`, `nebula_forge_xl`, and `VisionaryForge`._&#x20;
* `enhance` (string, _optional_): Image enhancement/upscaling level. Use `"1x"` for single enhancement, `"2x"` for double enhancement, or omit / use an empty string for no enhancement. Enhancement will upscale the image (improving resolution/quality) and is only supported by certain models (currently `velogen`, `nebula_forge_xl`, `VisionaryForge`). If an unsupported model is used with this parameter, the enhancement request is ignored.
* `steps` (number, _optional_): The number of diffusion or refinement steps to use for image generation. More steps can produce more detailed or refined images at the cost of longer generation time. Each model has its own supported range (e.g. `velogen` supports 1–4 steps, default is 2; `nebula_forge_xl` & `VisionaryForge` supports up to 50, default is 25). If not provided, the model’s default number of steps is used.
* `width` (number, _optional_): The width of the generated image in pixels.
* `height` (number, _optional_): The height of the generated image in pixels.
* `style` (string, optional): If you include a style, the image will be generated according to the specified style. Available styles are listed below. ([Listed Styles](quickstart-guide.md))
* `traits` (Array , optional): Traits are an optional array of objects. If provided, images will be generated based on the specified trait ratios. For example, if the array includes two traits—`Background` with values `Heaven` (ratio: 20) and `Hell` (ratio: 60)—and you request 5 images, approximately 1 will use `Heaven` and 3 will use `Hell` background.You can adjust traits and ratios to guide image generation accordingly.

**Note on image size:** Supported `width` × `height` combinations (aspect ratios) depend on the model (and whether enhancement is used). For example, **Velogen** supports base resolutions like 512×512 (square), 768×512 (landscape), 512×768 (portrait). With 2× enhancement, it can produce up to \~1920×1920 (square) and corresponding larger rectangles. **Nebula Forge XL** and **VisionaryForge** have their own supported dimensions (see **Supported Models & Settings** below for detailed resolution options). If you request a size that the model doesn’t support, the API may return an error or adjust to the nearest valid size. If `height`/`width` are not specified, the SDK will default to the model’s base default (usually a square format such as 512×512 or 1024×1024 depending on the model).

**Returns:** A Promise that resolves to an object containing the generated image data. On success, you can expect the image bytes (binary data) to be available (typically under a field like `data` or similar). The image is in JPEG format by default. You can take the returned binary data and save it to a file, send it to a client, or further process it as needed.

**Example – Generate an image and save to a file (with explanations):**

```js
const fs = require('fs');

// Define the image generation parameters
const params = {
  prompt: "A futuristic city skyline at sunset, neon pink and orange hues", 
  // ^ Text description of the desired image
  model: "velogen",        // Use the "velogen" model for generation (realistic style)
  enhance: "1x",           // Upscale once (1x enhancement for higher resolution)
  steps: 2,                // Use 2 refinement steps (velogen's default)
  width: 1024, height: 1024,// Generate a 1024x1024 image (square format)
  style: 'cinematic',       //Style of image
   traits: [
      {
        trait_type: 'Background',
        value: [
          { value: 'Heaven', ratio: 20 },
          { value: 'Hell', ratio: 60 },
          { value: 'garden', ratio: 20 },
        ],
      },
      {
        trait_type: 'contrast',
        value: [
          { value: 'dark', ratio: 20 },
          { value: 'light', ratio: 80 },
        ],
      },
    ],
};

const result = await nft.generateImage(params);
// The result contains image data (e.g., result.data.data is a buffer of bytes)

// Save the image data to a file
fs.writeFileSync(
  "generated-image.jpg",            // output file path (JPEG format)
  Buffer.from(result.data.data)     // image binary data from the result
);
console.log("Image saved to generated-image.jpg");
```

In this example, we used the `velogen` model to generate a 1024×1024 image from the given prompt, with a single upscaling enhancement. The resulting image bytes are written to **generated-image.jpg**. After running this code, that file will contain the AI-generated artwork. You can adjust the `prompt`, `model`, and other parameters as needed.

#### Synchronous NFT Generation

#### `generateNft(options)`&#x20;

**Description:** Generates an image _and_ prepares NFT metadata from a text prompt, intended for minting. This call handles the AI image creation similarly to `generateImage()`, but also registers the result for minting (on ChainGPT’s backend) and returns metadata needed to proceed with minting. Typically, after `generateNft` completes, you would call `mintNft()` to actually mint the NFT on-chain. This method is **synchronous** in the sense that it waits for the image generation to finish before returning, so it can take some time (usually 30–60 seconds for the image to be generated). Under the hood, this corresponds to an API call that generates the image and prepares the NFT in one step.

**Parameters:** _(passed in an `options` object; includes all `generateImage` parameters plus NFT-specific fields)_

* **All image generation parameters from** `generateImage()`: You can include `prompt`, `model`, `enhance`, `steps`, `width`, `height` with the same meanings as described above. These control the image generation part of this call.
* `walletAddress` (string, **required**): The public wallet address that will eventually own the NFT. This is the address to which the NFT will be minted (transferred) when you call `mintNft()`. Make sure this is a valid address on the blockchain network you intend to use.
* `chainId` (number, **required**): The blockchain network ID where the NFT will be minted. ChainGPT’s NFT generator supports many networks (Ethereum, BNB Chain, Polygon, Arbitrum, Avalanche, Tron, and more). For example: use `1` for Ethereum Mainnet, `56` for BNB Smart Chain (BSC) Mainnet, `137` for Polygon, etc. You can retrieve the full list of supported networks and their IDs at runtime with the `getChains()` method. Ensure the `chainId` you provide corresponds to the network of the provided wallet address.
* `style` (string, optional): If you include a style, the image will be generated according to the specified style. Available styles are listed below. ([Listed Styles](quickstart-guide.md))
* `traits` (Array , optional): Traits are an optional array of objects. If provided, images will be generated based on the specified trait ratios. For example, if the array includes two traits—`Background` with values `Heaven` (ratio: 20) and `Hell` (ratio: 60)—and you request 5 images, approximately 1 will use `Heaven` and 3 will use `Hell` background.You can adjust traits and ratios to guide image generation accordingly.

_(There are no parameters for NFT name/description in this step – those are provided later to the `mintNft()` call.)_

**Returns:** A Promise that resolves to an object containing the generated NFT data. On success, the result will include:

* The **image data** (similar to what `generateImage` returns, i.e. the binary image bytes).
* A **`collectionId`** (string or UUID) that uniquely identifies this generation job and the associated metadata on ChainGPT’s backend.

The `collectionId` is important: it references the saved AI-generated image and metadata. You will use it to either check the generation status (if queued) or to mint the NFT. In the case of `generateNft()` (synchronous), the promise only resolves **after** the image is fully generated, so at that point the image is ready and `collectionId` can be used directly for minting.

**Note:** After `generateNft()` completes, the NFT is _not yet on the blockchain_. It’s essentially a prepared record (image + metadata) awaiting the minting step. You must call `mintNft()` with the returned `collectionId` to actually mint the NFT on-chain. This two-step process (generate then mint) allows you to, for example, confirm with a user or display the image before incurring blockchain costs.

**Example – Generate an NFT then mint it (with explanations):**

```js
// 1. Generate the NFT image and metadata (synchronously)
const genResult = await nft.generateNft({
  prompt: "A cute cartoon cat astronaut, sticker illustration",
  model: "VisionaryForge",
  enhance: "",             // no enhancement (empty string or omit for none)
  steps: 25,               // use 25 refinement steps (only supported by some models like NebulaForge XL or VisionaryForge)
  height: 1024, width: 1024,
  walletAddress: "0xABC123...DEF",  // the recipient wallet address for the NFT
  chainId: 97,                      // target chain ID (97 = BSC Testnet in this example)
  style: 'cinematic',       //Style of image
  traits: [
      {
        trait_type: 'Background',
        value: [
          { value: 'Heaven', ratio: 20 },
          { value: 'Hell', ratio: 60 },
          { value: 'garden', ratio: 20 },
        ],
      },
      {
        trait_type: 'contrast',
        value: [
          { value: 'dark', ratio: 20 },
          { value: 'light', ratio: 80 },
        ],
      },
    ],
};

const result = await nft.generateImage(params);
// The result contains image data (e.g., result.data.data is a buffer of bytes)

// Save the image data to a file
fs.writeFileSync(
  "generat
});
console.log("Generation completed. Collection ID:", genResult.data.collectionId);

// 2. Mint the generated NFT on-chain using the returned collectionId
const mintResult = await nft.mintNft({
  collectionId: genResult.data.collectionId,  // use the collectionId obtained from generateNft
  name: "Space Cat Sticker",                  // human-readable name for the NFT
  description: "An AI-generated cat astronaut sticker, created via ChainGPT",  // description metadata
  symbol: "CAT"                               // symbol for the NFT collection (if a new collection is created)
});
console.log("NFT minted! Transaction/result:", mintResult);
```

In this example, we first call `generateNft()` with a prompt and parameters including the wallet address and chain ID where we want to mint. When this completes, we get a `collectionId` (printed to console) which references the generated image and metadata. We then call `mintNft()` with that `collectionId` and provide an NFT name, description, and symbol. The SDK then handles the blockchain transaction to mint the NFT to the specified address on BSC Testnet (chainId 97). The `mintResult` would contain information about the minting operation (such as a transaction hash or token ID, depending on the implementation).

**Example – Production usage (generate then mint, minimal code):**

```js
// Generate NFT (blocking until ready) and then mint it on BSC Mainnet
const { data: nftData } = await nft.generateNft({ 
  prompt: "Epic dragon portrait in medieval style", 
  model: "velogen",
  walletAddress: "0x...YOUR_WALLET_ADDRESS", 
  chainId: 56  // BNB Chain Mainnet
});
await nft.mintNft({
  collectionId: nftData.collectionId,
  name: "Epic Dragon", 
  description: "AI-generated medieval dragon art", 
  symbol: "DRGN"
});
```

The above snippet omits comments for brevity. It generates an NFT on BSC (chain 56) for the given wallet address, then immediately mints it with a name, description, and symbol. In a real application, you should include error handling around these calls (e.g., try/catch) and perhaps update your UI to inform the user of progress between generation and minting.

#### `generateNftWithQueue(options)` – Asynchronous/Queued Generation

**Description:** Initiates an NFT generation task asynchronously, without waiting for the image to be fully generated. This function returns quickly with a `collectionId` that represents the queued generation job on ChainGPT’s servers. The actual image creation then happens in the background. You can use the returned `collectionId` to poll the job’s status via `getNftProgress()` and later call `mintNft()` to mint the NFT once the image is ready. This queued approach is useful for handling multiple generations in parallel or offloading longer generation times without blocking your application flow.

**Parameters:** Same as `generateNft(options)` above. You must provide all required fields (`prompt`, `model`, `walletAddress`, `chainId`) and any optional generation settings (`enhance`, `steps`, `height`, `width`) as needed.

**Returns:** A Promise that resolves to an object containing at least a `collectionId` for the queued generation request. The image data will **not** be available immediately in the response, since generation is still in progress. The `collectionId` is the key identifier you'll use to check progress and eventually mint the NFT. Essentially, if this call succeeds, it means the generation job was accepted and is now processing on the server.

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
} while (progressInfo.data.status !== "completed");
// Loop until the status indicates completion (status might go from "processing" to "completed")

console.log("Generation complete! Now minting the NFT...");
// 3. Mint the NFT now that the image generation is done
await nft.mintNft({
  collectionId: jobId,
  name: "Color Swirl Art",
  description: "AI-generated abstract painting",
  symbol: "ART"
});
console.log("NFT minted on chain.");
```

In this example, we queue an NFT generation using `generateNftWithQueue()`. The call returns immediately with a `collectionId` (stored in `jobId`). We then enter a loop to poll the job’s status using `getNftProgress()`, every 5 seconds, logging the progress percentage. Once the `status` becomes `"completed"` (which implies 100% progress), we proceed to call `mintNft()` with the same `collectionId` to mint the NFT on-chain. In a real application, you might use a longer delay between polls or integrate a webhook/callback if available, rather than busy polling. You can also queue multiple jobs in parallel and poll each one, which makes this approach suitable for generating large collections of NFTs asynchronously.

**Example – Production usage (queue generation, minimal):**

```js
// Queue NFT generation (non-blocking call)
const { data: queued } = await nft.generateNftWithQueue({ 
  prompt: "Portrait of an AI robot", 
  model: "velogen", 
  walletAddress: "0x...ADDR", 
  chainId: 137  // Polygon Mainnet
});
console.log("Queued job ID:", queued.collectionId);
// (Later, use getNftProgress() and mintNft() as shown above once the job is done)
```

Here we simply queue the generation and log the job ID. The subsequent steps (polling progress and minting) would be similar to the detailed example above. This pattern allows your code to continue doing other work or handling other requests while images are being generated in the background.

#### `getNftProgress(options)`

**Description:** Checks the status of an NFT generation job that was started with `generateNftWithQueue()` (or even a `generateNft()` call if you want to poll its progress, though `generateNft()` usually waits internally). It returns information about the generation progress, such as a percentage complete or a status indicator.

**Parameters:**

* `collectionId` (string, **required**): The unique identifier of the NFT generation job to query. This is the `collectionId` returned by `generateNft` or `generateNftWithQueue`. Without a valid `collectionId`, the API cannot identify which job’s status to retrieve.

**Returns:** A Promise that resolves to an object containing progress information for the specified job. The exact structure of the data may include fields like `progress` (number) and `status` (string). For example, a successful response might be `{ progress: 100, status: "completed", ... }` when the generation is finished. Before completion, you might see a lower `progress` value (e.g. 0 to 99) and a status like `"processing"` or `"queued"`. If an invalid `collectionId` is provided, the SDK will throw an error or you might get a response indicating the job wasn’t found.

Typically, you will call this method repeatedly (polling) until the `status` is `"completed"` or `progress` reaches 100, then proceed to mint the NFT. The frequency of polling can be adjusted based on how long generations typically take (e.g., polling every few seconds).

**Example – Basic usage:**

```js
const progressRes = await nft.getNftProgress({ collectionId: jobId });
if (progressRes.data.progress === 100) {
  console.log("Job completed!");
}
```

**Example – Production snippet (single progress check):**

```js
// Check the current progress of a generation job
const status = await nft.getNftProgress({ collectionId: "YOUR_COLLECTION_ID" });
console.log("Current progress:", status.data.progress, "%");
```

In practice, you would integrate this into a loop or schedule as shown in the queued generation example. Use `getNftProgress()` to inform users about the generation status (e.g., a loading bar) or to trigger the next step of your workflow when the image is ready.

#### `mintNft(options)`

**Description:** Mints an NFT on the specified blockchain, using a previously generated image/metadata. This is the final step in the NFT creation process, taking the output of `generateNft` (identified by a `collectionId`) and writing the NFT to the blockchain. The method also lets you set the NFT’s name, description, and symbol (metadata) if they weren't already set. Under the hood, this will interact with ChainGPT’s NFT minting smart contract to either create a new NFT in an existing collection or deploy a new collection, then mint the token to the provided wallet address.

**Parameters:**

* `collectionId` (string, **required**): The ID of the generated NFT content to mint. This is obtained from a prior `generateNft` or `generateNftWithQueue` call. It tells the system which image/metadata to use for minting.
* `name` (string, **required**): The name for the NFT. This could be the title of the artwork or any descriptive name. This will appear as the token’s name in metadata.
* `description` (string, _optional_): A text description for the NFT. This is stored in the token’s metadata, providing details or context for the artwork. (If not provided, it might be left blank or a default in the metadata.)
* `symbol` (string, _optional_): The symbol (short identifier) for the NFT collection. If this minting operation creates a new NFT collection (smart contract), this symbol will be used for that collection (similar to a ticker or shorthand for the collection). If the NFT is being minted into an existing collection, this parameter might be ignored. For one-off mints or testing, you can use any short string (e.g. "AI" or "ART").

**Returns:** A Promise that resolves to an object containing the result of the mint operation. On success, this will typically include details such as:

* Confirmation that the NFT was minted (e.g., a success status or the minted token’s ID/URL).
* Transaction details, like a transaction hash, if the mint involved an on-chain transaction.
* Possibly the address of the new NFT collection contract (if one was deployed) or confirmation of the collection used.

The exact fields returned may vary, but the key outcome is that the image identified by `collectionId` is now minted as an NFT owned by `walletAddress` (which was provided during generation). The promise will only resolve once the minting is complete (which may involve waiting for a blockchain transaction).

After calling `mintNft()`, the NFT is live on the blockchain. You can then fetch it from the blockchain or via ChainGPT’s APIs, and it will have the metadata (name, description, etc.) that you provided.

**Example – Mint an NFT directly (assuming image is already generated):**

```js
await nft.mintNft({
  collectionId: "abcd1234-ef56-...-xyz",  // collection ID from a previous generateNft call
  name: "My AI Artwork",
  description: "This NFT image was generated by AI via ChainGPT",
  symbol: "AINFT"
});
console.log("Mint successful!");
```

In this example, we directly call `mintNft()` using a known `collectionId` (from a prior generation step) and provide the desired metadata. If the operation succeeds, the console will log a success message. In a real scenario, you’d want to wrap this in a try/catch to handle errors (like network issues or insufficient permissions) and perhaps confirm the transaction result.

**Example – Production snippet:**

```js
// Finalize NFT minting (production use)
const result = await nft.mintNft({ collectionId, name: "Artwork #1", symbol: "ART" });
// (Add error handling as needed)
```

Typically, you will call `mintNft()` once per generated image to put it on-chain. Remember to catch errors from this call as it involves external blockchain interaction (see **Error Handling** below). For instance, if the user’s wallet is invalid or if there are network fees required that the system cannot cover, you would get an error.

#### Enhance Your Prompt

Update your prompt by setting the enhancement level to receive a response tailored to the enhanced prompt.

```javascript
const { Nft } = require('@chaingpt/nft');
const nftInstance = new Nft({ apiKey: 'Your ChainGPT API Key' });

async function main() {
  const enhancedPrompt = await nftInstance.enhancePrompt({
    prompt: 'lion in jungle',
  });
  console.log(enhancedPrompt);
}
main();
```

***

#### Generate Random Prompt (Surprise Me)

Run this snippet of code to receive a surprise prompt for generating an NFT.

```javascript
const { Nft } = require('@chaingpt/nft');
const nftInstance = new Nft({ apiKey: 'Your ChainGPT API Key' });

async function main() {
  const randomPrompt = await nftInstance.surpriseMe();
  console.log(randomPrompt);
}
main();
```

#### Get Collections with Filters

This example demonstrates how to use the `@chaingpt/nft` SDK to retrieve NFT collections associated with a specific wallet address.

#### Parameters

| Parameter       | Type      | Required | Description                                 |
| --------------- | --------- | -------- | ------------------------------------------- |
| `walletAddress` | `string`  | ✅        | The wallet address to fetch collections for |
| `isPublic`      | `boolean` | ❌        | Filter only public collections              |
| `isDraft`       | `boolean` | ❌        | Filter by draft status                      |
| `isMinted`      | `boolean` | ❌        | Filter by minted status                     |
| `name`          | `string`  | ❌        | Filter by collection name                   |
| `symbol`        | `string`  | ❌        | Filter by token symbol                      |
| `page`          | `number`  | ❌        | Pagination: page number                     |
| `limit`         | `number`  | ❌        | Pagination: number of items per page        |

```javascript
const { Nft } = require('@chaingpt/nft');
const nftInstance = new Nft({ apiKey: 'Your ChainGPT API Key' });

async function main() {
  const collections = await nftInstance.getCollections({
    walletAddress: '<wallet address>', 
    isPublic: true,
     isDraft: false, 
     isMinted: false, 
     name: "<name>", 
     symbol: "<symbol>", 
    page: 1,
    limit: 10,
  });
  console.log(collections);
}
main();
```

#### &#x20;Toggle NFT Visibility

toggle the **visibility status** (e.g., from public to private or vice versa) of an NFT collection using the `@chaingpt/nft` SDK.

```javascript
const { Nft } = require('@chaingpt/nft');
const nftInstance = new Nft({ apiKey: 'Your ChainGPT API Key' });

async function main() {
  const response = await nftInstance.toggleNftVisibility({
    collectionId: '<collection id>',
  });
  console.log(response);
}
main();
```

### Supported Models & Settings

_(This section provides reference details on the supported AI models, image enhancements, and output formats available in the ChainGPT NFT Generator SDK.)_

*   **Supported AI Models:** You can choose from several AI models via the `model` parameter in `generateImage`/`generateNft`:

    * **`velogen`** – ChainGPT’s VeloGen model, suitable for realistic or general-purpose image generation.
    * **`nebula_forge_xl`** – ChainGPT’s high-resolution model, useful for detailed and high-quality imagery.
    * **`VisionaryForge`** – ChainGPT’s creative model for artistic or concept art styles.
    * **`Dale3`** – An integration of OpenAI’s DALL·E 3 model (optional). This model can be used for varied art styles and compositions, though it may not support all the advanced settings (enhancements, etc.) that the ChainGPT proprietary models do.

    Each model has different strengths and styles. We recommend experimenting with the proprietary models (`velogen`, `nebula_forge_xl`, `VisionaryForge`) to see which best suits your use case.&#x20;
* **Image Formats:** Generated images are returned in a binary form (typically a buffer containing JPEG data). By default, the images generated via the API/SDK are in **JPEG** format (as seen by the `.jpg` extension in examples). The SDK itself does not expose a parameter to select the image format (e.g., PNG vs JPEG); it will always return the image data as provided by the API (JPEG). If you require a different format, you can convert the image bytes to another format using an image processing library after you receive the data.
*   **Enhancement (Upscaling) Options:** The `enhance` parameter allows you to request higher-resolution outputs:

    * `"1x"` – Single enhancement (image is upscaled once, roughly doubling the resolution).
    * `"2x"` – Double enhancement (upscaled twice, resulting in an even larger image).

    Only certain models support enhancement/upscaling (currently **velogen**, **nebula\_forge\_xl**, and **VisionaryForge**). Using `"1x"` or `"2x"` with those models will produce larger, more detailed images (as shown in their supported resolutions below). If you attempt to enhance on a model that doesn’t support it (such as `Dale3`), the request will simply generate at the model’s default resolution without upscale. Keep in mind that enhanced images consume more processing and may count as additional credit usage.
*   **Resolution & Aspect Ratios:** Each model supports specific image resolutions, especially when using enhancement:

    * **Velogen:** Base output resolutions include 512×512 (square), 768×512 (landscape), 512×768 (portrait). With 2× enhancement, approximate maximum resolutions are 1920×1920 (square), 1920×1280 (landscape), 1280×1920 (portrait).
    * **Nebula Forge XL:** Base outputs are 1024×1024, 1024×768, 768×1024. With upscaling (enhancement), it can go up to \~1536×1536 (square), 1536×1024 (landscape), 1024×1536 (portrait).
    * **VisionaryForge:** Base outputs are 1024×1024, 1024×768, 768×1024 (similar to Nebula’s base). With upscaling, up to around 1536×1536, 1536×1024, 1024×1536.
    * **Dale3:** (1024x1024, upscaling or enhancement isn't available).&#x20;

    When specifying the `width` and `height` in your requests, it’s best to stick to one of the supported aspect ratio combinations for the chosen model (as listed above) to get optimal results. If you choose a non-standard size, the API might adjust the output size or return an error. For simplicity, using a square resolution (512×512 for smaller images, or 1024×1024 for larger images) is a safe choice across models.
* **Default Values:** If you omit certain optional parameters in the generation calls:
  * `enhance` – Defaults to no enhancement (no upscaling) if not provided.
  * `steps` – Defaults to the model’s built-in default (for example, 2 steps for VeloGen, 25 for Nebula Forge XL) if not specified.
  * `height`/`width` – Typically default to the model’s default resolution (often the base square, e.g., 512×512 for some models or 1024×1024 for others). It’s recommended to explicitly set these for clarity.

***

### Error Handling

The ChainGPT NFT SDK provides a structured error handling mechanism to help you catch and manage errors from any SDK call (image generation, minting, etc.).

When something goes wrong (such as a network issue, an invalid API request, or an error returned from the ChainGPT API), the SDK will throw an error of type **`Errors.NftError`**. This is a custom error class provided by the SDK. It includes a message describing the error, and it may include additional information such as an HTTP status code or an error payload from the API.

**How to handle errors:** Ensure you wrap your SDK calls in a `try...catch` block in your async function. In the `catch`, you can check if the caught error is an instance of `Errors.NftError` to distinguish it from other types of exceptions.

**Example – Error handling pattern:**

```js
import { Errors } from '@chaingpt/nft';

try {
  const response = await nft.generateNft({ /* ...parameters... */ });
  // Use the response normally if successful
} catch (error) {
  if (error instanceof Errors.NftError) {
    console.error("ChainGPT SDK error:", error.message);
    // You could also inspect error.response or error.code if needed
  } else {
    // Some other unexpected error (programming error, etc.)
    console.error("Unexpected error:", error);
  }
}
```

In the above snippet, if `nft.generateNft()` fails (for example, due to an invalid API key, a 400 Bad Request from the API, or a network timeout), the catch block will execute. The error’s message (`error.message`) usually provides a human-readable description of what went wrong (e.g. "Unauthorized" for invalid API key, or a specific validation message). By checking `instanceof Errors.NftError`, we ensure we’re handling errors thrown by the ChainGPT SDK. Any other error would fall to the `else` block (which could be a coding mistake or something unrelated).

**Common error scenarios to handle:**

* _Authentication failures:_ If your API key is missing, incorrect, or expired, the SDK will throw an `NftError` (often indicating a 401 or 403 HTTP error under the hood). You should catch this and prompt for a correct key or notify about invalid credentials.
* _Invalid parameters:_ If you pass values that the API cannot process (e.g., an unsupported image resolution, or missing a required field like `walletAddress`), the API will respond with a 400 error and the SDK will throw an `NftError` with details. Check the message to adjust your request.
* _Generation failures:_ In rare cases, the AI might fail to generate an image or the job might encounter an internal error. This could result in an error or a job status that indicates failure. Handle this by catching the error or checking for a failure status in `getNftProgress()`.
* _Network issues:_ If the SDK cannot reach the ChainGPT API (due to network outage or timeout), it will throw an `NftError` indicating a network error. You might implement retries or inform the user to check their connection.

By consistently catching `Errors.NftError`, you can gracefully handle all these cases. The SDK abstracts away the need to manually parse HTTP responses; any non-2xx HTTP response or other failure will result in a thrown `NftError` that you can catch in one place. This makes it easier to implement robust error-handling logic (such as retries, user notifications, or logging) around SDK calls.

***

### Additional Resources

* **ChainGPT Developer Dashboard:** To obtain your API key or manage your subscription/credits, visit the ChainGPT Developer Dashboard on the ChainGPT website. This is where you can monitor your usage and top up credits if needed.
* **Pricing & Credits:** For details on how the credit system works and the cost of generating images or NFTs, refer to the **Pricing & Membership Plans** page on the ChainGPT documentation site. (Pricing is subject to change and depends on the model and enhancements used, so it’s maintained separately from this SDK reference.)
* **NPM Package Documentation:** You can find the SDK’s package page on [npmjs.com](https://www.npmjs.com/package/@chaingpt/nft), which may include additional usage examples and the latest version information.
* **REST API Reference:** If you need to use the NFT generator via HTTP calls or want to know the underlying endpoints, consult the AI NFT Generator API Reference in the ChainGPT docs.
* **Support & Community:** If you encounter issues or have questions, you can reach out through ChainGPT’s official support channels. Visit the ChainGPT website [ChainGPT.org](https://www.chaingpt.org/) for links to community forums, Discord, or contact information. The development team and community are available to help with integration questions or troubleshooting.

This SDK reference is meant to be a comprehensive guide for developers. For any further help, check out the QuickStart guide for step-by-step examples or contact the ChainGPT support team. Happy building with the ChainGPT AI NFT Generator SDK!
