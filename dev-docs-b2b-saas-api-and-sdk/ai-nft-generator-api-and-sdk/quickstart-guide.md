# QuickStart Guide

## ChainGPT AI NFT Generator QuickStart Guide

This QuickStart walks you through generating AI images and minting NFTs using ChainGPT’s API and SDK — with real examples, valid parameters, and everything you need to go live fast.

***

### Prerequisites

Before you begin, make sure you have the following in place:

* **ChainGPT API Key:** Sign up on the ChainGPT platform and obtain an API key from the API dashboard. Every request must include this key in the Authorization header (as a Bearer token). Also ensure your account has sufficient **credits** (see _Pricing & Credits_ below) – requests will fail if you run out of credits.
* **Node.js Environment:** Install [Node.js](https://nodejs.org/) (for using the SDK). You’ll also need a package manager like npm or Yarn.
* **API Access Tool:** For making raw API calls, you can use cURL (as shown in examples) or any HTTP client (Postman, etc.). All API endpoints are relative to the base URL `https://api.chaingpt.org`.
* **Secure Key Storage:** Treat your API key like a password – store it securely (for example, in environment variables or a secrets manager), **not** hard-coded in your code or pushed to repositories.

{% hint style="info" %}
**Note:** Make sure your ChainGPT account has sufficient credits or an appropriate plan. NFT generation calls will fail if you run out of credits or if your API key is invalid.
{% endhint %}

***

### Using the REST API (via cURL)

ChainGPT’s AI NFT Generator API allows you to generate AI images from text and optionally mint them as NFTs. Below we demonstrate the workflow using direct REST calls with cURL.

#### 1. Generate an AI Image (Synchronous)

To **generate an image** from a text prompt, use the `POST /nft/generate-image` endpoint. This returns the image data directly in the response, suitable for quick previews or standalone image generation.

**Request:** Include your API key in the header and provide the prompt and generation parameters in JSON. For example:

```bash
curl -X POST "https://api.chaingpt.org/nft/generate-image" \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
           "prompt": "A majestic dragon under a sunset sky",
           "model": "velogen",
           "steps": 3,
           "height": 512,
           "width": 512,
           "enhance": "1x"
         }'
```

In this request:

* `prompt` – The text description of the image to generate.
* `model` – The AI model to use (e.g. `"velogen"`, `"nebula_forge_xl"`, `"VisionaryForge"`, or `"Dale3"` for DALL·E 3).
* `steps` – _(Optional)_ Number of refinement steps. Higher values can yield more detail but take longer. Each model has a range: **velogen** supports 1–4 steps (default 2), **nebula\_forge\_xl** and **VisionaryForge** support up to 50 (default 25), while **Dale3** (DALL·E 3) doesn’t use a steps parameter.
* `height` and `width` – The image dimensions in pixels. Ensure they are within the supported range for the chosen model (see **Model Options** below). Common base sizes are 512 or 768 for quick generation, or 1024 if supported.
* `enhance` – _(Optional)_ Upscaling option for higher resolution. Use `"1x"` for 1× enhancement or `"2x"` for 2× enhancement. If omitted or set to `"original"`, the image is generated at base resolution (no upscale). Using enhancement produces a larger image (up to the model’s max upscale) at additional credit cost.

**Response:** On success, you’ll get a JSON response with the image data. For example:

```json
{
  "data": [ 255, 216, 237, ... ]
}
```

The `data` field is an array of bytes representing the generated image (in this case, a JPEG). You can convert this to an image file. For instance, in Node.js you could do:

```
fs.writeFileSync("output.jpg", Buffer.from(response.data));
```

This will write the bytes to `output.jpg`. (If the API returns a base64 string instead, decode it accordingly.)

**Model Options & Parameters:** The API supports multiple models, each with different capabilities:

* **velogen** – Fast generation for smaller images. Supports base resolutions up to \~768px and 1–4 steps (quick iterations).
* **nebula\_forge\_xl** – High-quality model for larger images (up to 1024px) and more steps (up to 50 for fine detail).
* **VisionaryForge** – Similar to Nebula Forge XL (up to 1024px, 50 steps), with its own style/profile.
* **Dale3** – Integration of OpenAI’s DALL·E 3 model. Fixed 1024×1024 output, no custom steps or enhancements allowed. Use this for DALL·E’s unique output, but note it has a separate pricing structure (see below).

Each model has recommended resolution ranges. For example, **velogen** is optimal at 512–768px (can upscale to \~1920px with `enhance: "2x"`), whereas **Nebula/VisionaryForge** excel at 768–1024px (up to \~1536px with upscale). If you request a resolution beyond a model’s base capability without setting `enhance`, the API may automatically upscale it if possible. It’s best to explicitly use the `enhance` parameter if you want high resolution output.

#### 2. Queue an NFT Image Generation (Asynchronous)

If you intend to **mint the image as an NFT**, it’s recommended to use the queued generation endpoint. This allows the backend to handle longer-running jobs and prepare NFT metadata. Use `POST /nft/generate-nft-queue` to start the generation without waiting for the image immediately.

**Request:** In addition to the prompt and image settings, you must provide:

* `walletAddress` – The blockchain wallet address that will ultimately receive/own the NFT.
* `chainId` – The identifier of the blockchain network where the NFT will be minted. (Use the **Get Chains** endpoint or see below for how to find supported chain IDs.)

Example cURL to queue a generation:

```bash
curl -X POST "https://api.chaingpt.org/nft/generate-nft-queue" \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
           "walletAddress": "0xABC123...DEF",
           "prompt": "Futuristic city skyline at night, neon glow",
           "model": "nebula_forge_xl",
           "steps": 25,
           "height": 1024,
           "width": 1024,
           "enhance": "2x",
           "amount":1,
           "chainId": 56
         }'
```

This request queues an image generation with the given parameters (prompt, model, etc.) on BSC Mainnet (`chainId`: 56). Higher values for `steps` (like 25 or even 50 for Nebula) can yield very detailed images, and here we used `enhance: "2x"` for an upscaled 1024→1536px output.

**Response:** The server immediately returns a job identifier:

```json
{
  "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
  "status": "queued"
}
```

The `collectionId` is a unique UUID for this generation task. The `status` will initially be `"queued"` (or `"processing"` shortly after) – no image is returned yet. You will use this ID to query the generation status and later to mint the NFT.

{% hint style="info" %}
**Note:** If you **only** need an image (no NFT minting), you can skip the queue and use the direct `generate-image` endpoint (as in step 1). The queued endpoint is designed for NFT workflows, where you’ll track progress and eventually call minting endpoints.
{% endhint %}

#### 3. Check Generation Progress

After queueing an NFT generation, you can poll its status using `GET /nft/progress/{collectionId}`. This returns the current status and (if completed) the result of the generation.

For example:

```bash
curl -X GET "https://api.chaingpt.org/nft/progress/b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx" \
     -H "Authorization: Bearer YOUR_API_KEY"
```

**Response:** If the job is still in progress, you’ll see something like:

```json
{
  "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
  "status": "processing",
  "progress": 45
}
```

Here, `status` is `"processing"` and `progress` is \~45%. (Some jobs may not return a percentage, just a status.) When the status becomes `"completed"`, the response will include the image data:

```json
{
  "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
  "status": "completed",
  "data": [137, 80, 78, 71, ...]
}
```

In the above completed example, `data` contains the bytes of the generated image (starting with `137 80 78 71` which is the PNG file signature). You can now save or utilize this image (just as in step 1).

Often, instead of manually retrieving the raw image here, the next step is to call the minting API to get the NFT metadata. The image is stored on the backend once generation completes. For simplicity, you can proceed directly to **Mint the NFT** after you see a `"completed"` status.

> **Tip:** You don’t have to query progress in a tight loop. Polling every few seconds is sufficient. Alternatively, you could implement a webhook/callback if the API provided one, but polling is straightforward for most use cases.

#### 4. (Optional) Enhance a Text Prompt

If you have a short or less-detailed prompt and want to improve it **before** generating an image, you can use `POST /nft/enhancePrompt`. This endpoint uses AI to refine your prompt (making it more descriptive or creative), which can lead to better image outcomes.

For example:

```bash
curl -X POST "https://api.chaingpt.org/nft/enhancePrompt" \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
           "prompt": "a futuristic city"
         }'
```

**Response:**

```json
{
  "enhancedPrompt": "A sprawling futuristic cityscape at night, illuminated by neon lights and flying vehicles, with towering skyscrapers and advanced technology."
}
```

The API returns an `enhancedPrompt` field containing a more detailed version of your input. You can then take this suggestion and use it as the `prompt` in your next `generate-image` or `generate-nft-queue` call. Each prompt enhancement call costs a small number of credits (0.5 credits per call).

_(Using prompt enhancement is optional but recommended if you’re not getting the desired detail in the outputs. It can help maximize the quality of the image, especially for expensive models or high-step generations.)_

#### 5. Retrieve Supported Blockchains (Get Chains)

The AI NFT Generator supports minting on multiple blockchain networks. To see which `chainId` values you can use, call `GET /nft/get-chains`:

```bash
# Fetch mainnet chains only
curl -X GET "https://api.chaingpt.org/nft/get-chains?testNet=false" \
     -H "Authorization: Bearer YOUR_API_KEY"
```

By default, this returns only mainnets. Add `?testNet=true` to include test networks. A sample response:

```json
{
  "chains": [
    { "chainId": 56, "name": "BSC Mainnet", "network": "mainnet" },
    { "chainId": 1,  "name": "Ethereum",   "network": "mainnet" },
    { "chainId": 97, "name": "BSC Testnet","network": "testnet" },
    ...
  ]
}
```

Each entry has a `chainId` and name. Use these IDs in the generation and minting calls. For example, `56` for Binance Smart Chain, `1` for Ethereum, `137` for Polygon, etc. If you’re testing on a testnet (like BSC Testnet or Goerli), be sure to both use the testnet `chainId` **and** a wallet address from that test network.

#### 6. Mint the NFT (Finalize Metadata & Minting)

Once your image is generated (and you have a `collectionId` from the queued generation), the final step is to **mint the NFT**. This involves calling `POST /nft/mint-nft` with the `collectionId` and some NFT metadata. This call will prepare the NFT’s metadata (and potentially trigger the on-chain mint, depending on the setup).

**Request:** Provide:

* `collectionId` – The ID from the completed generation job (ensure the status was “completed” before minting).
* `name` – A name/title for the NFT (e.g. `"Neon Skyline City"`).
* `description` – A description for the NFT, detailing the artwork or any info you want in the token metadata.
* `symbol` – A short symbol or ticker for the NFT/collection (e.g. `"NEON"`).

Example cURL:

```bash
curl -X POST "https://api.chaingpt.org/nft/mint-nft" \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
           "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
           "name": "Neon Skyline City",
           "description": "A futuristic city skyline at night, generated by AI.",
           "symbol": "NEON"
         }'
```

**Response:** If successful, you will receive metadata about the newly created NFT. For example:

```json
{
  "name": "Neon Skyline City",
  "description": "A futuristic city skyline at night, generated by AI.",
  "image": "ipfs://Qm...abcd",
  "attributes": [ /* ... */ ],
  "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
  "transaction": null
}
```

This includes the `name`, `description` you provided, and importantly an `image` URL (often an IPFS link to the image that was uploaded). Additional metadata like `attributes` could appear if relevant. The `transaction` field may contain a transaction hash if the system initiated an on-chain mint for you, or `null` if not applicable.

At this point, the NFT’s data is ready. If the service did not automatically mint the NFT on-chain, you have two options to finalize the minting:

* Use the ChainGPT NFT contract to mint the token to the provided wallet address (you'll need on-chain interaction, see **Contract ABI** below).
* Or, if `transaction` was returned, wait for that transaction to confirm on-chain (or fetch it via a blockchain explorer).

In most cases, you will need to perform the actual blockchain transaction yourself. The `mint-nft` response gives you the metadata (and the image is now pinned e.g. on IPFS). Now you or your backend can call the blockchain smart contract’s `mint` function to create the NFT.

#### 7. (Optional) Get the NFT Contract ABI

If you plan to interact with the NFT smart contract directly (to manually call the `mint` function, or to integrate with web3 tooling), you can fetch the contract’s ABI via `GET /nft/abi`. This returns the ABI (Application Binary Interface) JSON of the ChainGPT NFT Mint Factory contract.

Example:

```bash
curl -X GET "https://api.chaingpt.org/nft/abi" \
     -H "Authorization: Bearer YOUR_API_KEY"
```

**Response:**

```json
{
  "abi": [
    {
      "type": "function",
      "name": "mint",
      "inputs": [
        { "name": "to", "type": "address" },
        { "name": "tokenURI", "type": "string" }
      ],
      "outputs": [ ... ]
    },
    {
      "type": "event",
      "name": "NFTMinted",
      "inputs": [ ... ]
    }
    // ... other functions and events ...
  ]
}
```

This ABI JSON can be used with a web3 library (Ethers.js, Web3.py, etc.) to invoke the contract. Typically, the NFT contract has a `mint(address to, string tokenURI)` function you’ll call, passing the `walletAddress` and a token URI (which the backend has likely already created for you, pointing to the metadata JSON). The ABI also defines events like `NFTMinted` that you could listen for.

Using the ABI from the API ensures you’re interacting with the correct contract and function signature. After calling `mint-nft` (Step 6), you would use this ABI along with the `collectionId`/metadata info to execute an on-chain transaction to actually mint the NFT to the user’s wallet.

***

### Using the Node.js SDK (@chaingpt/nft)

ChainGPT provides an official JavaScript/TypeScript SDK (`@chaingpt/nft` on npm) to simplify integration. This SDK wraps all the above API calls into convenient functions. In this section, we’ll accomplish the same tasks using Node.js.

#### 1. Installation and Setup

First, install the SDK into your Node.js project:

```bash
npm install --save @chaingpt/nft
# or
yarn add @chaingpt/nft
```

Import and initialize the SDK in your code with your API key:

```javascript
const { Nft } = require('@chaingpt/nft');  // import the SDK class
// import * as fs from 'fs';               // Node.js file system, if needed for saving files

const nft = new Nft({
  apiKey: process.env.CHAINGPT_API_KEY  // your ChainGPT API Key
});
```

Here we store the API key in an environment variable for safety (you can replace `process.env.CHAINGPT_API_KEY` with your key string for testing, but avoid hardcoding secrets in production). The `Nft` class instance `nft` will be used to call all NFT API methods.

#### 2. Generate an Image using the SDK

To generate an image, use the `nft.generateImage()` method. This mirrors the `POST /nft/generate-image` endpoint.

```javascript
const fs = require('fs');

async function quickGenerateImage() {
  try {
    const imageResult = await nft.generateImage({
      prompt: "A majestic dragon under a sunset sky",
      model: "velogen",
      steps: 2,
      height: 512,
      width: 512,
      enhance: "1x"
    });
    // The response contains the image data buffer.
    // Save it to a file:
    fs.writeFileSync("generated-image.jpg", Buffer.from(imageResult.data.data));
    console.log("Image saved to generated-image.jpg");
  } catch (error) {
    console.error("Image generation failed:", error);
  }
}

quickGenerateImage();
```

A few notes on the above code:

* The parameters `prompt`, `model`, `steps`, `height`, `width`, `enhance` are the same as described in the REST API section and are passed in as an object. The values here will produce a 512×512 image using the **velogen** model with 2 refinement steps and a 1× enhancement (effectively upscaled from 256px base to 512px output, since velogen’s base resolution is smaller).
* `imageResult` will be an object containing the image data. Typically, the SDK returns a structure similar to the API’s JSON. In this case, `imageResult.data.data` is the byte array of the image (the double `.data` is because the SDK may wrap the response). We convert it to a Buffer and write to a file.
* Error handling: we wrap the call in a try/catch to catch any exceptions (network issues, invalid parameters, etc.).

This method is synchronous from the caller’s perspective (it `await`s the image generation to complete and returns the image). Use this for quick generation or when you need the image immediately.

**Model options and parameters** (as discussed earlier) apply here as well. For example, you could switch `model` to `"nebula_forge_xl"` and increase `steps` to 25 for a higher-quality image (just ensure to adjust `height`/`width` within that model’s range, e.g. 1024×1024, and consider using `enhance: "2x"` for maximum quality). The SDK will call the appropriate API and retrieve the result.

#### 3. Get Supported Chains (Networks)

You can retrieve the list of supported blockchains via the SDK using `nft.getChains()`. By default, `getChains()` returns mainnets; pass `true` to include testnets.

```javascript
async function listChains() {
  const chainsMainnet = await nft.getChains();        // same as getChains(false)
  const chainsIncludingTest = await nft.getChains(true);
  console.log("Mainnet chains:", chainsMainnet.data);
  console.log("All chains (with testnets):", chainsIncludingTest.data);
}
```

This will fetch the same data as the `/nft/get-chains` API. Each call returns an object containing a `chains` array (accessible via `.data` in the SDK response). For example, `chainsMainnet.data.chains` might include entries like `{ chainId: 56, name: 'BSC Mainnet', ... }`. Use these IDs for the `chainId` parameter in generation and minting.

#### 4. Generate an NFT (Image + Metadata) using the SDK

To generate an image _with the intent to mint it as an NFT_, you have two options in the SDK:

* **Immediate generation** with `nft.generateNft()`: This will **wait** for the image generation to complete and return the image data plus a `collectionId`. Use this if your application can afford to wait (e.g. 30-60 seconds) for the generation to finish in one go.
* **Queued generation** with `nft.generateNftWithQueue()`: This will return immediately with a `collectionId` (similar to the REST `generate-nft-queue`), and you can then poll progress with `nft.getNftProgress()` if needed. Use this if you want to handle the waiting or progress feedback in your app manually (for example, show a progress bar).

For simplicity, here’s how to use `generateNft()` for a synchronous flow:

```javascript
async function generateAndPrepareNFT() {
  try {
    const result = await nft.generateNft({
      prompt: "Futuristic city skyline at night, neon glow",
      model: "nebula_forge_xl",
      steps: 25,
      height: 1024,
      width: 1024,
      enhance: "2x",
      walletAddress: "0xABC123...DEF",
      chainId: 97  // BSC Testnet for example
    });
    console.log("NFT generation completed. Collection ID:", result.data.collectionId);
    // The image data is also available in result.data (similar to generateImage).
  } catch (error) {
    console.error("NFT generation failed:", error);
  }
}
```

This call will take some time to resolve (it internally queues the generation and waits for completion). Once it returns, you get an object `result` containing the image data and a `collectionId`. We log the `collectionId` for reference. At this point, the image is generated and stored, and you can proceed to mint it.

{% hint style="info" %}
**Note:** We used `chainId: 97` (BSC Testnet) above for testing. In production, you might use a mainnet chain like 56 for BSC or 1 for Ethereum, etc. Ensure the `walletAddress` is compatible with the chain.
{% endhint %}

If instead you prefer to not block execution, you could use `generateNftWithQueue()`:

```javascript
const queueResult = await nft.generateNftWithQueue({ ...params });
console.log("Queued Collection ID:", queueResult.data.collectionId);
```

This returns immediately with the `collectionId`. You can then call `nft.getNftProgress({ collectionId })` in intervals to check `status` and `progress`, similar to the REST polling in step 3. The usage of `getNftProgress` is straightforward:

```javascript
const progressRes = await nft.getNftProgress({ collectionId: queueResult.data.collectionId });
console.log(progressRes.data);
```

It will output the status (`"queued"`, `"processing"`, or `"completed"`) and progress percent if available. Once completed, you could retrieve the image (though if you plan to immediately mint, you might skip directly retrieving the raw image and just call `mintNft` as shown next).

#### 5. Enhance a Prompt via SDK (Optional)

To use the prompt enhancement feature through the SDK, call `nft.enhancePrompt()`:

```javascript
async function enhanceMyPrompt() {
  const res = await nft.enhancePrompt({ prompt: "a futuristic city" });
  console.log("Enhanced prompt:", res.data.enhancedPrompt);
}
```

This will return an object whose `data.enhancedPrompt` is the improved prompt text (just like the REST response). You can then use that string in `generateImage` or `generateNft` calls to potentially get better results. Each call costs \~0.5 credits.

#### 6. Mint the NFT (using the SDK)

After generating the NFT (and obtaining a `collectionId`), use the SDK’s `nft.mintNft()` method to retrieve the minting details and finalize the NFT.

```javascript
async function mintGeneratedNFT(collectionId) {
  try {
    const mintRes = await nft.mintNft({
      collectionId: collectionId,
      name: "Neon Skyline City",
      description: "A futuristic city skyline at night, generated by AI.",
      symbol: "NEON"
    });
    console.log("Mint response:", mintRes.data);
  } catch (error) {
    console.error("Minting failed:", error);
  }
}
```

In this snippet, replace `collectionId` with the ID from the generation step (e.g. `result.data.collectionId` from `generateNft()` earlier). The `mintNft` call requires the same three metadata fields: `name`, `description`, `symbol`. On success, `mintRes.data` will contain the NFT metadata and possibly a transaction reference, similar to the REST response. You should see the `image` URL (likely an IPFS link) and the `collectionId` echoed back. If the SDK/backend initiated an on-chain mint, a transaction hash might be present in the data; if not, you’ll use the ABI to mint manually.

After this, the NFT’s metadata is on-chain or ready to be minted on-chain. If no auto-mint happened, the next step would be using a blockchain library to call the actual smart contract’s mint function, as described below.

#### 7. Get the NFT Contract ABI (SDK)

You can fetch the NFT contract’s ABI via the SDK as well with `nft.abi()`:

```javascript
async function getContractABI() {
  const abiRes = await nft.abi();
  console.log("Contract ABI:", abiRes.data.abi);
}
```

This returns the same JSON ABI that the REST call would. You can use this output in your web3 interactions. For example, with Ethers.js you might do:

```javascript
const contractAddress = "<ChainGPT_NFT_Contract_Address_For_Selected_Chain>";
const abi = abiRes.data.abi;
const provider = new ethers.providers.JsonRpcProvider(<RPC_URL>);
const signer = new ethers.Wallet(<PRIVATE_KEY>, provider);
const nftContract = new ethers.Contract(contractAddress, abi, signer);
await nftContract.mint(walletAddress, tokenURI);
```

Where `tokenURI` is the metadata URL (e.g. the IPFS link) you got from `mintNft()`. The exact contract address to use would be provided by ChainGPT’s documentation for each chain (or possibly returned in the mint response or available in the `getChains` data if a new contract is deployed per chain).

{% hint style="info" %}
**Note:** _The above blockchain interaction is just an illustration; the QuickStart’s scope is primarily the ChainGPT API/SDK usage. In practice, consult ChainGPT docs on whether a new contract is deployed or a shared factory contract is used across chains, and how the **`tokenURI`** is structured._
{% endhint %}

***

### Pricing & Credit Usage

ChainGPT’s NFT Generator API uses a credit system. Each API call deducts credits from your balance, so you should understand the costs:

* **Standard image generation (ChainGPT models):** Using models like `velogen`, `nebula_forge_xl`, or `VisionaryForge` costs **1 credit per image** at base resolution. If you use enhancement (`"1x"` or `"2x"`), it costs **2 credits per image** (effectively +1 credit for the upscale). For example, generating a 1024×1024 image with Nebula Forge XL will cost 1 credit, and if you set `enhance: "1x"` (HD upscale), it will cost 2 credits.
* **DALL·E 3 model (Dale3):** This third-party model has higher costs. Generating a 1024×1024 image with DALL·E 3 uses about **4.75 credits** (standard quality). Using HD upscale on it roughly doubles the cost to **\~9.5 credits**. For other resolutions (since DALL·E can only output 1024×1024 internally, the service resizes if you ask for non-1024 dimensions), the cost is about **9.5 credits** (standard) or **\~14.25 credits** with HD. In short, DALL·E 3 is significantly more expensive per image.
* **Prompt Enhancement:** Each call to the `/nft/enhancePrompt` endpoint costs **0.5 credits** (since it’s a lighter-weight operation).
* **Minting calls and others:** Calling `mint-nft` or `get-chains` does not typically deduct credits (they mostly prepare data or fetch info). The main costs are for image generation and prompt enhancement.

Make sure you have enough credits for the operations you plan to run. For example, a single `generateNft` with Nebula (no enhance) will use 1 credit, plus a `mintNft` call (0 credits) – total 1 credit. The same with DALL·E 3 might use \~4.75 credits. If you plan to generate many images or use upscale frequently, ensure your credit balance is topped up accordingly.

{% hint style="info" %}
_Refer to the official pricing page for the latest credit costs and any changes to the model pricing structure._
{% endhint %}

***

### Error Handling

Both the REST API and SDK provide error information to help you diagnose issues:

* **HTTP Status Codes:** The REST API returns standard codes. A `200 OK` indicates success (image or job created). **`400 Bad Request`** means something was wrong with your request (e.g., missing required field or invalid parameter value). For instance, if you exceed a model’s resolution limits, you might get a 400 with an error message like `{"error": "height and width exceed allowed resolution for model velogen"}`. **`401 Unauthorized`** means your API key is missing/invalid, or you’ve run out of credits – check that you included the correct Bearer token and have sufficient credits. **`429 Too Many Requests`** indicates you hit a rate limit or tried to use the API with no credits (the service may also treat no-credit attempts as a form of rate limit). **`500 Internal Server Error`** is a rare server-side issue – you can retry after a delay or contact support if it persists. The error responses typically include a JSON body with an `"error"` message explaining what went wrong.
*   **SDK Exceptions:** In the Node.js SDK, errors from the API are thrown as exceptions. Specifically, they will be instances of the `Errors.NftError` class if they originate from a non-200 API response. You should wrap SDK calls in `try/catch`. For example:

    ```javascript
    const { Errors } = require('@chaingpt/nft');
    try {
      const response = await nft.generateImage({ ... });
      // use response
    } catch (err) {
      if (err instanceof Errors.NftError) {
        console.error("ChainGPT API Error:", err.message);
      } else {
        console.error("Unexpected Error:", err);
      }
    }
    ```

    This way you can distinguish ChainGPT API errors (e.g., invalid input, no credits, etc., conveyed in `err.message`) from other runtime errors. The SDK’s error message will usually match the JSON error from the HTTP response.
* **Debugging Tips:** Ensure that all required parameters are provided (the SDK’s TypeScript definitions can help catch missing fields). If you get a 401, double-check the API key. If an image generation is failing consistently, try the enhancePrompt to improve your prompt, or reduce the resolution/steps in case it’s hitting limits. On rate limits (429), implement a backoff/retry after the suggested time.

***

### Supported Styles

Here is a list of supported models:

3d-model

&#x20;analog-film

&#x20;anime

cinematic

comic-book

digital-art

enhance

fantasy-art

isometric

line-art

low-poly

neon-punk

origami

photographic

pixel-art

texture

craft-clay

### Next Steps and Additional Resources

You have now generated AI-driven images and minted an NFT using ChainGPT’s API and SDK. From here, you can integrate these calls into your application’s backend or frontend. For a more in-depth understanding of each endpoint and method, refer to the official **ChainGPT AI NFT Generator API Reference** and **SDK Reference** documentation. These resources provide detailed descriptions of all parameters, more example scenarios, and information on advanced features.

{% content-ref url="api-reference.md" %}
[api-reference.md](api-reference.md)
{% endcontent-ref %}

{% content-ref url="sdk-reference.md" %}
[sdk-reference.md](sdk-reference.md)
{% endcontent-ref %}

For any questions or support, check out the ChainGPT community channels or contact the team as listed on the ChainGPT website. Happy building your NFT application with ChainGPT’s AI NFT Generator!
