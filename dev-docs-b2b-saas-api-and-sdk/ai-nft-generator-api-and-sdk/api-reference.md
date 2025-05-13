# API Reference

## ChainGPT AI NFT Generator API Reference

### Introduction

This reference describes the RESTful API endpoints for the ChainGPT AI NFT Generator. It covers endpoints for generating AI-based images and NFTs, along with supporting endpoints for prompt enhancement, chain information, and contract ABI. Each endpoint is documented with its method, path, required headers, parameters, and example requests/responses. Use this guide to integrate AI image generation and NFT minting into your applications.

***

### Authentication & API Basics

* **Base URL:** All endpoints are relative to the base URL `https://api.chaingpt.org`.
*   **API Key (Authentication):** Every request must include your API key in the `Authorization` header as a Bearer token. For example:

    ```
    Authorization: Bearer YOUR_API_KEY
    ```

    You can obtain an API key from the ChainGPT web app’s API dashboard (after acquiring credits). Ensure the API key has sufficient credits in your account (see Pricing & Credits). Requests without a valid key will be rejected with **401 Unauthorized**.
* **Content Type:** Use `Content-Type: application/json` for request bodies, and expect JSON responses. Image data is returned encoded in JSON (e.g. as an array of bytes or base64 string).
* **Models:** The AI NFT Generator supports multiple image models. When generating images, you must specify one of the supported models: `"velogen"`, `"nebula_forge_xl"`, `"VisionaryForge"`, or `"Dale3"` (which uses OpenAI’s DALL·E 3). Each model has specific constraints on image resolution and steps (see Supported Models & Parameters). Choose the model appropriate for your needs (e.g. _VeloGen_ for quick small images, _NebulaForge XL_ or _VisionaryForge_ for high quality larger images, or _DALL·E 3_ for its unique generative capabilities).

***

### Text-to-Image Generation (Synchronous)

Use this endpoint to **generate an image from a text prompt** and get the result immediately. The API will return the image data (as a binary buffer or encoded string) in the response synchronously. This is ideal for quick image generation (e.g. previews or single images) when you don't need NFT minting.

#### `POST /nft/generate-image`

**Description:** Generates an image based on the given prompt and model. The image bytes are returned directly in the response (no need to poll for results). For supported models, resolutions, and steps, see the Supported Models & Parameters section.

**Headers:**

* `Authorization: Bearer YOUR_API_KEY` (required)
* `Content-Type: application/json` (required)

**Request Body Parameters:**

<table data-header-hidden><thead><tr><th width="113.9140625"></th><th width="158.32421875"></th><th></th></tr></thead><tbody><tr><td><strong>Parameter</strong></td><td><strong>Type</strong></td><td><strong>Description</strong></td></tr><tr><td><code>prompt</code></td><td>string (required)</td><td>The text prompt describing the image to generate.</td></tr><tr><td><code>model</code></td><td>string (required)</td><td>Identifier of the model to use (e.g. <code>"velogen"</code>, <code>"nebula_forge_xl"</code>, <code>"VisionaryForge"</code>, or <code>"Dale3"</code> for DALL·E 3).</td></tr><tr><td><code>steps</code></td><td>integer (optional)</td><td>Number of refinement passes for image generation. Higher values can yield more detailed images at the cost of longer generation time. Each model supports a range: for example, <em>velogen</em> supports 1–4 steps (default ~2), while <em>NebulaForge XL</em> and <em>VisionaryForge</em> support up to 50 (default ~25). If not provided, a model-specific default is used.</td></tr><tr><td><code>height</code></td><td>integer (required)</td><td>Output image height in pixels. Must be within the model’s supported base resolution range (see Supported Models &#x26; Parameters). Common values are 512 or 768 for base generation, or larger if using enhancement.</td></tr><tr><td><code>width</code></td><td>integer (required)</td><td>Output image width in pixels. Must meet the same constraints as <code>height</code>. (For a square image, use equal width and height, e.g. 512×512 or 1024×1024.)</td></tr><tr><td><code>enhance</code></td><td>string (optional)</td><td>Upscaling option to improve image quality. Allowed values: <code>"original"</code> (no enhancement), <code>"1x"</code> (single enhancement), or <code>"2x"</code> (double enhancement). If omitted or set to <code>"original"</code>, the image is generated at base resolution. Using <code>"1x"</code> or <code>"2x"</code> will upscale the image (up to the model’s enhanced max resolution, e.g. ~1280px or 1920px) at additional credit cost (see Pricing &#x26; Credits).</td></tr><tr><td>style</td><td>string (optional)</td><td>If you include a style, the image will be generated according to the specified style. Available styles are listed below. (<a href="https://docs.chaingpt.org/dev-docs-b2b-saas-api-and-sdk/ai-nft-generator-api-and-sdk/quickstart-guide#supported-styles">Listed Styles</a>)</td></tr><tr><td>traits</td><td>Array (optional)</td><td>Traits are an optional array of objects. If provided, images will be generated based on the specified trait ratios. For example, if the array includes two traits—<code>Background</code> with values <code>Heaven</code> (ratio: 20) and <code>Hell</code> (ratio: 60)—and you request 5 images, approximately 1 will use <code>Heaven</code> and 3 will use <code>Hell</code> background.You can adjust traits and ratios to guide image generation accordingly.</td></tr></tbody></table>

**Sample Request:**

```bash
 curl -X POST https://api.chaingpt.org/nft/generate-image \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "dragon",
    "model": "velogen",
    "enhance": "original",
    "steps": 3,
    "height": 512,
    "width": 512,
    "style": "cinematic",
    "traits": [
      {
        "trait_type": "Background",
        "value": [
          { "value": "Heaven", "ratio": 20 },
          { "value": "Hell", "ratio": 60 },
          { "value": "garden", "ratio": 20 }
        ]
      },
      {
        "trait_type": "contrast",
        "value": [
          { "value": "dark", "ratio": 20 },
          { "value": "light", "ratio": 80 }
        ]
      }
    ]
  }'
```

**Sample Response:**

```json
{
  "data": [255, 216, 237, ...]
}
```

The response’s `data` field contains the raw image bytes. For example, the above byte array begins with `255, 216, 237, ...`, which corresponds to the JPEG file header. To use the image, you can write these bytes to a file or decode them in your application. For instance, in Node.js you could do:

```
fs.writeFileSync("output.jpg", Buffer.from(response.data.data));
```

This will save the returned image buffer to an `output.jpg` file. (If the API returns image data as a base64-encoded string instead, you would decode it accordingly.)

***

### Text-to-NFT Generation (Asynchronous)

Use these endpoints to **generate an image intended for NFT minting**. The generation is handled asynchronously: first you queue a generation job, then poll for its progress, and finally retrieve the NFT metadata for minting. This flow is suited for higher resolution images or cases where you plan to mint the result as an NFT on a blockchain.

#### `POST /nft/generate-nft-queue`

**Description:** Initiates an NFT image generation job with the specified parameters, associating it with a blockchain and wallet for eventual minting. Unlike the immediate image endpoint, this returns quickly with a job identifier (`collectionId`) while the image is generated in the background. You should subsequently use the progress endpoint to check when the image is ready. This queued approach is recommended for longer-running jobs or when an on-chain mint will follow.

**Headers:**

* `Authorization: Bearer YOUR_API_KEY` (required)
* `Content-Type: application/json` (required)

**Request Body Parameters:**

<table data-header-hidden><thead><tr><th width="152.6015625"></th><th width="152.33203125"></th><th></th></tr></thead><tbody><tr><td><strong>Parameter</strong></td><td><strong>Type</strong></td><td><strong>Description</strong></td></tr><tr><td><code>walletAddress</code></td><td>string (required)</td><td>The blockchain wallet address that will own the minted NFT. This should be a valid address on the target chain (e.g. the user’s wallet).</td></tr><tr><td><code>prompt</code></td><td>string (required)</td><td>The text prompt for image generation (same usage as in the direct image endpoint).</td></tr><tr><td><code>model</code></td><td>string (required)</td><td>The model to use (e.g. <code>"velogen"</code>, <code>"nebula_forge_xl"</code>, <code>"VisionaryForge"</code>, <code>"Dale3"</code>). Supports the same options and constraints as the direct image generation endpoint.</td></tr><tr><td><code>steps</code></td><td>integer (optional)</td><td>Number of generation steps (refinement passes). Defaults to the model’s optimal value if not provided. You may use higher values (up to the model’s max, e.g. 50 for NebulaForge XL/VisionaryForge) to improve quality.</td></tr><tr><td><code>height</code></td><td>integer (required)</td><td>Desired image height in pixels. Must be within the model’s allowed range (e.g. up to 1024 for base generation on NebulaForge XL).</td></tr><tr><td><code>width</code></td><td>integer (required)</td><td>Desired image width in pixels. Must be within the allowed range for the model (same constraints as height).</td></tr><tr><td><code>enhance</code></td><td>string (optional)</td><td>Enhancement/upscaling option (<code>"original"</code> (default),  <code>"1x"</code> or <code>"2x"</code>) for higher resolution output. If provided, the image will be upscaled up to the model’s enhanced max resolution. If omitted, the image is generated at base resolution. (Using enhancement will consume additional credits).</td></tr><tr><td><code>chainId</code></td><td>integer (required)</td><td>Blockchain network ID where the NFT will be minted. Use the <code>/nft/get-chains</code> endpoint to see supported chains and their IDs. For example: <code>1</code> for Ethereum, <code>56</code> for BSC Mainnet, <code>97</code> for BSC Testnet, etc..</td></tr><tr><td>style</td><td>string (optional)</td><td>If you include a style, the image will be generated according to the specified style. Available styles are listed below. (<a href="https://docs.chaingpt.org/dev-docs-b2b-saas-api-and-sdk/ai-nft-generator-api-and-sdk/quickstart-guide#supported-styles">Listed Styles</a>)</td></tr><tr><td>traits</td><td>array (optional)</td><td>TTraits are an optional array of objects. If provided, images will be generated based on the specified trait ratios. For example, if the array includes two traits—<code>Background</code> with values <code>Heaven</code> (ratio: 20) and <code>Hell</code> (ratio: 60)—and you request 5 images, approximately 1 will use <code>Heaven</code> and 3 will use <code>Hell</code> background.You can adjust traits and ratios to guide image generation accordingly.</td></tr></tbody></table>

**Sample Request:**

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
           "chainId": 56,
             "style": "cinematic",
            "traits": [
              {
                "trait_type": "Background",
                "value": [
                  { "value": "Heaven", "ratio": 20 },
                  { "value": "Hell", "ratio": 60 },
                  { "value": "garden", "ratio": 20 }
                ]
              }
              ]
         }'
```

**Sample Response (Queued):**

```json
{
  "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
  "status": "queued"
}
```

On success, the API returns a unique `collectionId` for the requested generation, along with an initial `status` (typically `"queued"` or `"processing"`). No image data is returned at this stage. You must use the `collectionId` with the Progress endpoint (described next) to poll for completion and retrieve the image or metadata once the generation is finished.

{% hint style="info" %}
**Note:** Use the NFT queue endpoint (with `walletAddress` and `chainId`) if you plan to **mint the result as an NFT on-chain**. If you only need an image and do not intend to mint it, use the direct `POST /nft/generate-image` endpoint instead. The underlying image generation is similar, but the queued flow enables tracking progress and later minting.
{% endhint %}

**Track Generation Progress - GET /progress/{collectionId}**

**Description:** Checks the status and progress of a queued NFT generation job identified by its `collectionId`. Use this endpoint to poll for completion when you have initiated a job via `generate-nft-queue`. It returns whether the job is still processing or completed (and may include the result when done).

**Path Parameter:h**

* `{collectionId}` (string, required) – The ID of the NFT generation job, as returned by the `generate-nft-queue` call.

**Headers:**

* `Authorization: Bearer YOUR_API_KEY` (required)
* _(No request body, this is a GET request.)_

**Sample Request:**

```bash
curl -X GET "https://api.chaingpt.org/nft/progress/b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx" \
     -H "Authorization: Bearer YOUR_API_KEY"
```

**Sample Response (In Progress):**

```json
{
  "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
  "status": "processing",
  "progress": 45
}
```

In the above example, the generation job is 45% complete (`"progress": 45`). The `status` could be `"queued"`, `"processing"`, or `"completed"`, etc., and `progress` may be a percentage or other indicator of completion. Not all jobs will report a numeric progress – some may only update the status.

**Sample Response (Completed):**

```json
{
  "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
  "status": "completed",
  "data": [137, 80, 78, 71, ...]
}
```

Once the job status is `"completed"`, the response may include the generated image data (similar format as the direct generate-image endpoint). In the example above, the `data` array contains bytes of a PNG image (`137 80 78 71` ... are the signature bytes of a PNG file). You can then retrieve or save this image data as needed.

{% hint style="info" %}
**Note:** In many cases, you might not need to manually grab the raw image from the progress response. Instead, you would proceed to the minting step to get the final NFT metadata. The backend typically stores the generated image and prepares metadata when the job completes. The Progress endpoint is primarily for status updates; the final image and metadata are obtained via the minting endpoint next.
{% endhint %}

**NFT Mint Configuration/ metadata - POST /mint-nft**

**Description:** Finalizes the NFT creation by generating the metadata and preparing for minting. You call this after an image generation job is completed to get the NFT’s metadata (name, description, image URI, etc.) needed for on-chain minting. Think of this as retrieving the “mint package” for your NFT. **Note:** This endpoint does **not** perform the actual blockchain mint transaction itself; it returns the data that you or the ChainGPT service will use to mint the NFT on-chain.

**Headers:**

* `Authorization: Bearer YOUR_API_KEY` (required)
* `Content-Type: application/json` (required)

**Request Body Parameters:**

<table data-header-hidden><thead><tr><th width="148.484375"></th><th width="153.08984375"></th><th></th></tr></thead><tbody><tr><td><strong>Parameter</strong></td><td><strong>Type</strong></td><td><strong>Description</strong></td></tr><tr><td><code>collectionId</code></td><td>string (required)</td><td>The ID of the completed generation job you want to mint, as returned by <code>generate-nft-queue</code>. Ensure that the generation status is <code>"completed"</code> before calling this (you can check via the progress endpoint).</td></tr><tr><td><code>name</code></td><td>string (required)</td><td>The desired name of the NFT. This will be stored in the token metadata as the title of the artwork.</td></tr><tr><td><code>description</code></td><td>string (required)</td><td>A description for the NFT. This text can describe the artwork or provide context, and will be included in the metadata.</td></tr><tr><td><code>symbol</code></td><td>string (required)</td><td>A short symbol or ticker for the NFT (or the collection). For example, <code>"DRAGON"</code> or <code>"CGPT"</code>. This may be used as an identifier for the NFT collection or simply stored as part of the metadata.</td></tr></tbody></table>

**Sample Request:**

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

**Sample Response:**

```json
{
  "name": "Neon Skyline City",
  "description": "A futuristic city skyline at night, generated by AI.",
  "image": "ipfs://Qm...abcd",
  "attributes": [ /* ... if any attributes ... */ ],
  "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
  "transaction": null
}
```

The response typically returns a JSON object representing the NFT metadata. In the example above, the response echoes the `name` and `description` you provided, an `image` field (in this case an IPFS URI pointing to the generated image), and possibly an `attributes` array if any attributes were added. The `collectionId` is included again for reference. A `transaction` field may also appear – for example, if the system automatically initiated an on-chain mint, this might contain a transaction hash; if not (as in this example), it may be `null`.

**Next Steps:** Once you have the NFT metadata (especially the `image` URI and the metadata JSON URI if provided), you can proceed to mint the NFT on the blockchain. If ChainGPT’s service handles the minting for you, it might provide a transaction or update to indicate the NFT was minted. Otherwise, you can use the returned data along with the provided smart contract ABI and chain information to perform the mint. Typically, you would use a web3 library and call the contract’s `mint` function, providing the target wallet address (the same `walletAddress` used earlier) and a tokenURI that points to the metadata JSON. The `mint-nft` endpoint’s purpose is to prepare the off-chain metadata needed for the NFT, allowing you to review or edit it before performing the actual on-chain mint.

***

### Prompt Enhancement (`POST /nft/enhancePrompt`)

This optional endpoint uses AI to **enhance or refine a text prompt** for better image generation results. It does _not_ generate an image itself, but returns an improved prompt that you can then use with the image generation endpoints.

**Description:** Takes a given prompt and returns a more detailed or refined version of that prompt. This can help yield higher-quality or more specific images from the models by ensuring the prompt is well-crafted. (For example, a short prompt _"a futuristic city"_ might be expanded into a more vivid description.)

**Headers:**

* `Authorization: Bearer YOUR_API_KEY` (required)
* `Content-Type: application/json` (required)

**Request Body:**

* `prompt` (string, required) – The original prompt text that you want to enhance/refine.

**Sample Request:**

```bash
curl -X POST "https://api.chaingpt.org/nft/enhancePrompt" \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
           "prompt": "a futuristic city"
         }'
```

**Sample Response:**

```json
{
  "enhancedPrompt": "A sprawling futuristic cityscape at night, illuminated by neon lights and flying vehicles, with towering skyscrapers and advanced technology."
}
```

The response contains an `enhancedPrompt` field with a more descriptive version of the input prompt. You can use this refined prompt in a subsequent image generation call to potentially get better results. Each call to prompt enhancement consumes a small amount of credits (see Pricing & Credits, typically 0.5 credits per enhancement).

***

### Supported Chains (`GET /nft/get-chains`)

Use this endpoint to retrieve the list of blockchain networks supported by the AI NFT Generator for minting NFTs.

**Description:** Returns an array of supported blockchains (networks) on which you can mint the generated NFTs. Each network entry includes its `chainId` and name. By default, only mainnet networks are returned; you can optionally include test networks.

**Headers:**

* `Authorization: Bearer YOUR_API_KEY` (required)

**Query Parameters:**

* `testNet` (boolean, optional) – If `true`, include testnet chains in the results. If omitted or `false`, only mainnet chains are listed.

**Sample Request:**

```bash
# Fetch only mainnet chains
curl -X GET "https://api.chaingpt.org/nft/get-chains?testNet=false" \
     -H "Authorization: Bearer YOUR_API_KEY"
```

**Sample Response:**

```json
{
  "chains": [
    {
      "chainId": 56,
      "name": "BSC Mainnet",
      "network": "mainnet"
    },
    {
      "chainId": 1,
      "name": "Ethereum",
      "network": "mainnet"
    },
    {
      "chainId": 97,
      "name": "BSC Testnet",
      "network": "testnet"
    },
    {
      "chainId": 5,
      "name": "Goerli (Ethereum Testnet)",
      "network": "testnet"
    }
    // ...other supported chains
  ]
}
```

Each entry in the `chains` list provides a blockchain’s ID and a human-readable name, and often a `network` type (mainnet or testnet). Use the `chainId` values when calling the generation and minting endpoints (`generate-nft-queue` and `mint-nft`) to specify the target blockchain. For example, use `chainId: 97` with a BSC Testnet wallet address, or `chainId: 1` with an Ethereum address, etc..

_Note:_ Ensure the `walletAddress` you provide in the generation call is valid on the chosen chain. The ChainGPT NFT Generator supports multiple blockchains for minting, so choosing the correct chain ID and corresponding wallet is important.

***

### Contract ABI (`GET /nft/abi`)

This endpoint provides the **ABI (Application Binary Interface)** of the ChainGPT NFT smart contract used for minting. Developers can use the ABI to interact with the NFT contract via web3 libraries.

**Description:** Returns the ABI for the ChainGPT NFT Mint Factory smart contract. The ABI is a JSON array describing the contract’s functions and events. With this, you can programmatically call contract functions (like the mint function) or decode events when integrating on-chain.

**Headers:**

* `Authorization: Bearer YOUR_API_KEY` (required)

**Sample Request:**

```bash
curl -X GET "https://api.chaingpt.org/nft/abi" \
     -H "Authorization: Bearer YOUR_API_KEY"
```

**Sample Response (truncated):**

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
      "outputs": [ /* ... */ ]
    },
    {
      "type": "event",
      "name": "NFTMinted",
      "inputs": [ /* ... */ ]
    }
    // ...other functions and events...
  ]
}
```

The actual ABI JSON will include all the contract’s functions and events. The snippet above illustrates the general structure (e.g., a `mint` function that takes a recipient address and token URI, and an `NFTMinted` event). Using this ABI, you can integrate with the blockchain directly. For instance, after you obtain the NFT metadata via `mint-nft`, you could use the ABI with a web3 library (and the appropriate contract address for the selected `chainId`) to call the `mint` function on the contract, passing in the `walletAddress` (as the `to` address) and the token URI that points to the NFT’s metadata. This is useful for custom on-chain interactions if not handled automatically by the ChainGPT service.

***

### Supported Models & Parameters

When generating images, choose the appropriate model and parameters for your use case. The table below summarizes the supported models and their capabilities:

| **Model**          | **Base Resolution Range**          | **Max Upscaled Resolution**          | **Max Steps**             |
| ------------------ | ---------------------------------- | ------------------------------------ | ------------------------- |
| `velogen`          | 512–768 px (e.g. 512×512, 768×512) | Up to \~1920×1920 px with 2× enhance | 1–4 steps                 |
| `nebula_forge_xl`  | 768–1024 px (e.g. 1024×768)        | Up to \~1536×1536 px when upscaled   | 1–50 steps                |
| `VisionaryForge`   | 768–1024 px (similar to above)     | Up to \~1536×1536 px when upscaled   | 1–50 steps                |
| `Dale3` (DALL·E 3) | 1024×1024 px (fixed)               | _N/A_ (no custom upscale)            | _N/A_ (no step parameter) |

{% hint style="info" %}
**Notes:**

* _Base Resolution_ is the native size the model can generate without enhancement. _Max Upscaled_ is the approximate maximum size using the `enhance` parameter (`"1x"` or `"2x"`). For example, VeloGen can generate up to \~768px natively, or up to \~1920px when enhanced twice. NebulaForge XL and VisionaryForge can handle up to 1024px natively, or \~1536px with upscaling. Keep the aspect ratio in mind; you can request landscape, portrait, or square dimensions as long as they fall within these ranges.
* If you request an image size beyond a model’s base range **without** setting `enhance`, the API may automatically apply an enhancement/upscale if possible. However, it’s recommended to explicitly use the `enhance` parameter if you want a higher resolution output, to be clear about the intent.
* The `steps` parameter controls the number of diffusion/refinement iterations. _VeloGen_ is optimized for speed with only a few steps (up to 4). More powerful models like _NebulaForge XL_ and _VisionaryForge_ can run up to 50 steps for finer detail. Higher step counts can significantly improve image quality for those models, at the cost of longer generation time. (The DALL·E 3 model does not use a custom steps setting – it generates at a fixed quality per prompt).
* **Prompt Enhancement:** If your initial prompt is short or not yielding the desired results, consider using the prompt enhancement endpoint before generation. The `POST /nft/enhancePrompt` call can provide a more detailed prompt, which can help ensure you get a good result on the first generation attempt. This can be especially useful when using the more expensive models or high step counts.
{% endhint %}

***

### Pricing & Credits

The ChainGPT NFT Generator API uses a credit-based system. Each API call deducts a certain number of credits from your account. The cost depends on the model and options used. Below is a brief overview of credit costs per image (see the [**Pricing & Credits**](pricing-and-credits.md) page for full details):

* **VeloGen, NebulaForge XL, VisionaryForge:** **1 credit** per image for base resolution generation. If you use the enhancement (`enhance = "1x"` or `"2x"` for higher resolution), it costs an additional 1 credit, for a total of **2 credits** per image with upscale. _(E.g. generating a simple image = 1 credit; generating with HD enhancement = 2 credits.)_
* **Prompt Enhancement:** \~**0.5 credits** per call. Enhancing a prompt via `POST /nft/enhancePrompt` deducts a small fraction of a credit (since it does not generate an image, just text).
* **Other Endpoints:** Non-generative endpoints such as checking progress, retrieving supported chains, or fetching the contract ABI do not consume credits. The `mint-nft` endpoint for metadata preparation also does not charge credits (it assumes the generation was already paid for), aside from any separate blockchain transaction costs which are outside the scope of the API.
* **Extras:** DALL·E 3 (Dale3) model: Higher cost due to external API usage. For a 1024×1024 image at standard quality (no upscale), 4.75 credits\~\~\~\~ are charged. Enabling HD upscale ("2x") for a 1024px image doubles this to \~9.5 credits. For other resolutions, standard generation is \~9.5 credits and HD upscaled is \~14.25 credits per image. (In summary, DALL·E generations range roughly from 4.75 up to 14.25 credits each depending on size/quality.)

{% hint style="info" %}
**Note:** You must have sufficient credits in your ChainGPT account before calling these APIs. If your credits are exhausted or the API key has no credits, calls to generate images (or enhance prompts) will fail (typically with a 401 Unauthorized or an error message indicating insufficient credits). Be sure to top up credits via the ChainGPT web app as needed. Full pricing details and credit packages are available on the ChainGPT Pricing & Membership Plans page.
{% endhint %}
