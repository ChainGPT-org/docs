# API Reference

## ChainGPT AI NFT Generator – API Reference

This reference describes the RESTful API endpoints for the ChainGPT AI NFT Generator. All endpoints are grouped by functionality and documented with method, path, headers, parameters, and example requests/responses. Use this guide to integrate image generation and NFT minting into your applications.

***

### API Basics

* **Base URL:** All endpoints below are relative to the base URL: `https://api.chaingpt.org`.
*   **Authentication:** Every request must include your API key in the `Authorization` header as a Bearer token. For example:

    ```http
    Authorization: Bearer YOUR_API_KEY
    ```

    Ensure the API key has sufficient credits (see **Pricing & Credits**). Requests without a valid API key will be rejected with `401 Unauthorized`.
* **Content Type:** Endpoints that accept a request body expect `Content-Type: application/json`. Responses are returned as JSON (for image data, the image is encoded in the JSON response).
* **Models:** The AI NFT Generator supports multiple image models. When generating images, you must specify a model. Supported models include **`velogen`**, **`nebula_forge_xl`**, **`VisionaryForge`**, and **`Dale3`** (DALL·E 3). Each model has specific resolution and step constraints (detailed under **Supported Models & Parameters** below).
* **Generate an API Key:** navigate to ChainGPT's API Dashboard ([https://app.chaingpt.org/apidashboard](https://app.chaingpt.org/apidashboard)), and click on "Create New Secret Key", make sure to obtain credits before using your API key.&#x20;

***

### Image Generation Endpoints

These endpoints allow you to generate AI-based images for NFTs. Use the **immediate generation** endpoint for quick results, or the **queued generation** endpoints for asynchronous processing (suitable for high-resolution or longer-running jobs, especially when you intend to mint the result as an NFT).

#### POST `/nft/generate-image`

Generate an image from a text prompt (synchronous). The response will contain the generated image data directly. This endpoint is ideal for quick image generation when you want the result immediately (e.g. for previews or smaller images).

* **Description:** Generates an image based on the given prompt and model. Returns the image data in the response (as a binary buffer or encoded string) . For supported models, resolutions, and steps, see **Supported Models & Parameters**.
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
  * `Content-Type: application/json` (required)
* **Request Body Parameters:** (JSON)
  * **`prompt`** _(string, required)_ – The text prompt describing the image to generate.
  * **`model`** _(string, required)_ – The model to use for generation (e.g. `"velogen"`, `"nebula_forge_xl"`, `"VisionaryForge"`, or `"Dale3"` for DALL·E 3) .
  * **`steps`** _(integer, optional)_ – Number of refinement steps for image generation. Higher steps can yield more detailed images but take longer. Each model supports a different range: e.g. velogen supports 1–4 steps (default 2) and nebula\_forge\_xl up to 50 (default 25). If not provided, a default optimal value for the chosen model is used.
  * **`height`** _(integer, required)_ – Output image height in pixels. Must be within the supported resolution range for the model (see **Supported Models & Parameters**). Common choices are 512 or 768 for base generation, or larger if using enhancement.
  * **`width`** _(integer, required)_ – Output image width in pixels. Follows the same constraints as `height`. (For a square image, use 512×512 or 1024×1024 etc. depending on model capabilities.)
  * **`enhance`** _(string, optional)_ – Image upscaling option to improve quality. Allowed values: `"1x"` for single enhancement, `"2x"` for double enhancement. If omitted or set to `"original"` (no enhancement), the image is generated at base resolution. Using enhancement will produce a higher-resolution image (up to the model’s enhanced max, e.g. 1280px or 1920px) at the cost of additional credits (see **Pricing & Credits**).
*   **Sample Request:**

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
*   **Sample Response:** _(JSON with image data)_

    ```json
    {
      "data": [ 255, 216, 237, ... ] 
    }
    ```

    The `data` field contains the raw image bytes (for example, the above is an array of byte values for a JPEG image). You can convert this buffer into an image file. For instance, in Node.js:

    ```js
    fs.writeFileSync("output.jpg", Buffer.from(response.data.data));
    ```

    The example above writes the returned image buffer to `output.jpg` ). (In some cases, the image data may be provided as a base64-encoded string; decode accordingly.)

#### POST `/nft/generate-nft-queue`

Queue an NFT image generation job (asynchronous). This endpoint is used when you intend to mint the generated image as an NFT. It immediately returns a `collectionId` that identifies the generation task, allowing you to check progress and retrieve results once ready.

* **Description:** Initiates an NFT image generation with the specified prompt and model, associating it with a blockchain and wallet for minting. The image generation is processed asynchronously – the API returns a job ID (`collectionId`) immediately, and you must use the Progress endpoint to poll for completion  . This is ideal for higher resolution or longer generation processes that may take time.
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
  * `Content-Type: application/json` (required)
* **Request Body Parameters:** (JSON)
  * **`walletAddress`** _(string, required)_ – The blockchain wallet address that will own the minted NFT . This should be the public address of the user or application’s wallet where the NFT will be minted.
  * **`prompt`** _(string, required)_ – The text prompt for image generation (same as in **generate-image**).
  * **`model`** _(string, required)_ – The model to use (e.g. `"velogen"`, `"nebula_forge_xl"`, `"VisionaryForge"`, `"Dale3"`). Supports the same models and constraints as **generate-image** .
  * **`steps`** _(integer, optional)_ – Number of generation steps (optional, defaults per model as above). You may use higher values (e.g. 25 or even up to 50 for certain models) to improve image quality.
  * **`height`** _(integer, required)_ – Image height in pixels. Must be within the model’s allowed range (e.g. up to 1024 for base generation on Nebula or VisionaryForge, etc.).
  * **`width`** _(integer, required)_ – Image width in pixels. Must be within allowed range for the model.
  * **`enhance`** _(string, optional)_ – Enhancement/upscaling option (`"1x"` or `"2x"`) for higher resolution output. If provided, the image will be upscaled (up to the model’s enhanced max resolution). If omitted, the image is generated at base resolution. (Using enhancement will incur additional credit cost .)
  * **`chainId`** _(integer, required)_ – Blockchain network identifier where the NFT will be minted . Use the **Get Chains** endpoint to retrieve supported chain IDs. For example, `56` for BSC Mainnet, `97` for BSC Testnet, etc.
*   **Sample Request:**

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
              "chainId": 56
            }'
    ```
*   **Sample Response:** _(JSON with job ID)_

    ```json
    {
      "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
      "status": "queued"
    }
    ```

    The response returns a unique `collectionId` for the generation request. You will use this ID with the Progress endpoint to check the status of the generation. The initial status will typically be `"queued"` or `"processing"`. No image data is returned at this stage.

{% hint style="info" %}
**Note:** Use **POST** `/nft/generate-nft-queue` (with `walletAddress` and `chainId`) if you plan to mint the NFT on-chain. If you only need an image and do not intend to mint it, you can use **POST** `/nft/generate-image` instead. The underlying image generation is similar, but the queued endpoint allows tracking and later minting.
{% endhint %}

#### GET `/nft/progress/{collectionId}`

Retrieve the progress status of a queued NFT generation job.

* **Description:** Checks the status and progress of an NFT generation task identified by `collectionId` . Use this to poll for completion when you have initiated generation via **generate-nft-queue**.
* **Path Parameter:**
  * `{collectionId}` _(string, required)_ – The ID of the NFT generation job (as returned by **generate-nft-queue**).
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
  * _(No request body for GET requests)_
*   **Sample Request:**

    ```bash
    curl -X GET "https://api.chaingpt.org/nft/progress/b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx" \
         -H "Authorization: Bearer YOUR_API_KEY"
    ```
*   **Sample Response (In-Progress):**

    ```json
    {
      "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
      "status": "processing",
      "progress": 45
    }
    ```

    In this example, the job is 45% complete. The response includes a `status` (which could be e.g. `"queued"`, `"processing"`, or `"completed"`) and a numeric `progress` percentage. The presence and format of progress information may vary – e.g. some jobs might return an approximate percentage or just a status flag.
*   **Sample Response (Completed):**

    ```json
    {
      "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
      "status": "completed",
      "data": [ 137, 80, 78, 71, ... ]
    }
    ```

    Once generation is finished (`status: "completed"`), the response may include the generated image data similar to **generate-image**. In the example above, the `data` array contains bytes of a PNG image (indicated by `137 80 78 71` which is the PNG file signature). You can then save or utilize this image data.

{% hint style="info" %}
**Note:** In many cases, instead of retrieving the raw image here, you will proceed to the **Mint NFT** step to get the final metadata and on-chain minting details. The backend may store the image and prepare metadata when the job completes. The **progress** endpoint primarily provides status; final image retrieval (if not done via this response) is handled in the next step.
{% endhint %}

#### POST `/nft/enhancePrompt`

Enhance or improve a text prompt for better image generation results.

* **Description:** Uses ChainGPT’s AI to refine a given prompt, making it more detailed or suitable for image generation. This can help yield higher-quality or more imaginative outputs from the image models. It **does not** generate an image itself, only returns an improved text prompt.
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
  * `Content-Type: application/json` (required)
* **Request Body:** (JSON)
  * **`prompt`** _(string, required)_ – The original prompt text that you want to enhance/refine.
*   **Sample Request:**

    ```bash
    curl -X POST "https://api.chaingpt.org/nft/enhancePrompt" \
         -H "Authorization: Bearer YOUR_API_KEY" \
         -H "Content-Type: application/json" \
         -d '{
              "prompt": "a futuristic city"
            }'
    ```
*   **Sample Response:**

    ```json
    {
      "enhancedPrompt": "A sprawling futuristic cityscape at night, illuminated by neon lights and flying vehicles, with towering skyscrapers and advanced technology."
    }
    ```

    The response returns an `enhancedPrompt` which is a more descriptive version of the input. You can then use this refined prompt in a subsequent image generation call. Each prompt enhancement API call costs a small number of credits (see **Pricing & Credits**).

***

### NFT Minting & Blockchain Endpoints

Once an image is generated (especially via the queued NFT generation), these endpoints help prepare and execute the minting of the image as an NFT on a blockchain. This includes retrieving supported blockchain networks, getting smart contract details, and generating the NFT metadata for minting.

#### GET `/nft/get-chains`

Retrieve the list of supported blockchain networks for NFT creation.

* **Description:** Returns the supported blockchains (networks) on which the AI NFT Generator can mint NFTs. Each chain is identified by an ID used in the generation and minting calls. By default, only mainnet chains are returned; an option is available to include testnets.
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
* **Query Parameters:**
  * **`testNet`** _(boolean, optional)_ – If `true`, include testnet chains in the result. If `false` or omitted, only mainnet chains are listed.
*   **Sample Request:**

    ```bash
    # Fetch only mainnet chains
    curl -X GET "https://api.chaingpt.org/nft/get-chains?testNet=false" \
         -H "Authorization: Bearer YOUR_API_KEY"
    ```
*   **Sample Response:**

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

    Each entry provides at least a `chainId` and human-readable name, and may include whether it’s a testnet. Use the `chainId` values in the **generate-nft-queue** and **mint-nft** calls to specify the target blockchain.

{% hint style="info" %}
**Note:** The ChainGPT NFT Generator supports multiple blockchains to mint your NFT. Ensure you choose a chain where the wallet address (provided during generation) is valid. For example, use chainId `97` with a BSC Testnet address or chainId `1` with an Ethereum address, etc.
{% endhint %}

#### GET `/nft/abi`

Fetch the ABI (Application Binary Interface) of the ChainGPT NFT Mint Factory smart contract.

* **Description:** Returns the ABI for the smart contract used to mint NFTs. The ABI is a JSON array describing the contract’s functions and events. Developers can use this ABI in combination with a web3 library to interact with the NFT contract (for example, to programmatically call the mint function or to decode events).
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
*   **Sample Request:**

    ```bash
    curl -X GET "https://api.chaingpt.org/nft/abi" \
         -H "Authorization: Bearer YOUR_API_KEY"
    ```
*   **Sample Response:**

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
          "outputs": [...]
        },
        {
          "type": "event",
          "name": "NFTMinted",
          "inputs": [ ... ]
        }
        // ...other functions and events...
      ]
    }
    ```

    (Example above is illustrative; the actual ABI JSON will be returned in full.) This ABI corresponds to the NFT minting contract. Typically, the contract has a `mint` function to mint a new NFT to a given address with a token URI (metadata link), and possibly events like `NFTMinted`. Using the ABI, you can integrate with the blockchain directly if needed. For instance, after obtaining metadata via **mint-nft**, you could use the ABI with a web3 library to call the `mint` function on the appropriate chain.

#### POST `/nft/mint-nft`

Retrieve the metadata and configuration needed to mint the generated NFT. This is typically the final step after an image has been generated (via **generate-nft-queue** and the job is completed).

* **Description:** Finalize an NFT creation by generating the metadata (and optionally initiating the mint). You provide the `collectionId` of a completed image generation along with NFT details, and this endpoint returns the information required to mint the NFT. Specifically, it returns the NFT’s metadata (including the image link) and may trigger preparation of the NFT on the backend. Think of this as retrieving the “mint package” for your NFT. _(This call does not itself perform a blockchain transaction from your wallet; rather, it prepares the NFT data. You will use the returned data along with the contract ABI to actually mint on-chain.)_
* **Headers:**
  * `Authorization: Bearer YOUR_API_KEY` (required)
  * `Content-Type: application/json` (required)
* **Request Body:** (JSON)
  * **`collectionId`** _(string, required)_ – The ID of the generated NFT collection/job you want to mint, as returned by **generate-nft-queue**. This should correspond to a completed generation (ensure the progress status is completed before minting).
  * **`name`** _(string, required)_ – The desired name of the NFT. This will be included in the token metadata as the title of the artwork.
  * **`description`** _(string, required)_ – A description for the NFT. This can detail the artwork, the story, or any information about the NFT, and will be stored in the metadata.
  * **`symbol`** _(string, required)_ – A short symbol or ticker for the NFT (or collection). This is typically used like a stock ticker or shorthand for the NFT collection. For example, `"DRAGON"` or `"CGPT"`. The symbol might be used if a new contract is created for the NFT or collection, or simply as part of metadata.
*   **Sample Request:**

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
*   **Sample Response:**

    ```json
    {
      "name": "Neon Skyline City",
      "description": "A futuristic city skyline at night, generated by AI.",
      "image": "ipfs://Qm...abcd", 
      "attributes": [ /* if any attributes were added */ ],
      "collectionId": "b8f86f05-5e8a-4f21-8c4a-xxxxxxxxxxxx",
      "transaction": null
    }
    ```

    The exact response structure may vary. Typically, you will receive the NFT metadata JSON. In the example above, the response includes the `name` and `description` you provided, an `image` field (often an IPFS URI or URL pointing to the generated image file), and possibly additional metadata like `attributes`. The `collectionId` echoes the input for confirmation. If the system automatically initiated a mint transaction, there might be a `transaction` hash or similar (here shown as `null` if not auto-minted).

    **Next Steps:** Once you have this metadata (especially the `image` URL/URI and metadata URI if provided), you can mint the NFT on-chain. If ChainGPT’s service handles the minting for you, the response or a subsequent event may indicate the NFT is minted. In a more manual scenario, you would take the `image`/metadata and use the provided **ABI** and chain information to call the smart contract’s `mint` function. For example, using web3, call the `mint(to, tokenURI)` on the contract (the contract address corresponds to the `chainId` used earlier, obtainable via **get-chains**). The `to` address would be the same `walletAddress` used in generation, and `tokenURI` is a link to a JSON file containing the name, description, and image (the service may have already uploaded this JSON to IPFS or their storage and the URI would be returned).

    This separation allows you to review or edit metadata before the actual on-chain mint. The `mint-nft` endpoint essentially prepares the off-chain metadata needed for the NFT.

***

### Supported Models & Parameters

When calling the generation endpoints, it’s important to choose appropriate model and parameters:

* **Models:**
  * **velogen:** A fast model for smaller images. Supports low step counts and base resolutions up to \~768px. Good for quick generations.
  * **nebula\_forge\_xl:** A high-quality model supporting larger images (up to 1024px) and more steps (up to 50) for refinement.
  * **VisionaryForge:** Another high-quality model similar to Nebula Forge XL (supports up to 1024px base, 50 steps).
  * **Dale3 (DALL·E 3):** An integration of OpenAI’s DALL·E 3 model. Generates 1024×1024 images (no custom steps or upscaling). Use this model for its unique generative capabilities at the fixed resolution of 1024px.
* **Resolution Options:** Each model has recommended base resolutions and maximum enhanced resolutions:

<table><thead><tr><th width="167.15625">Model</th><th width="158.4140625">Base Resolution Range</th><th width="152.7109375">Max Upscaled (Enhanced)</th><th>Min-Max Steps</th></tr></thead><tbody><tr><td><code>velogen</code></td><td>512–768 px (e.g. 512×512, 768×512, 512×768)</td><td>up to 1920×1920 px with 2× enhancement</td><td>1-4 steps </td></tr><tr><td><code>nebula_forge_xl</code></td><td>768–1024 px (e.g. 1024×768, 768×1024)</td><td>up to 1536×1536 px when upscaled</td><td>1-50 steps </td></tr><tr><td><code>VisionaryForge</code></td><td>768–1024 px (e.g. 1024×768, 768×1024)</td><td>up to 1536×1536 px when upscaled</td><td>1-50 steps</td></tr><tr><td><code>Dale3</code> (DALL·E 3)<br>Via External Provider.</td><td>1024×1024 px (fixed)</td><td>N/A (no custom upscale)</td><td>N/A (no step parameter)</td></tr></tbody></table>

**Notes:**

* _Base Resolution_ refers to the image size generated without enhancement. _Upscaled_ refers to using the `enhance` parameter (`1x` or `2x`). For example, velogen can generate a 512px image normally, or up to \~1920px if enhanced twice . Nebula and VisionaryForge can generate at 1024px, or \~1536px enhanced. The aspect ratio can be portrait, landscape, or square within those ranges.
* If you request dimensions beyond a model’s base range without setting `enhance`, the API may automatically apply an enhancement/upscale if available. Ensure to use `enhance` explicitly if you want the highest quality output.
* The **steps** parameter controls image refinement iterations. velogen is limited to a few quick steps (1–4), whereas more powerful models like nebula\_forge\_xl and VisionaryForge can iterate up to 50 steps for finer details. Higher steps can dramatically improve image quality for those models, at the cost of longer generation time. DALL·E 3 does not expose a steps parameter.
* **Prompt Enhancement:** If your initial prompt is short or not yielding the desired result, use **POST** `/nft/enhancePrompt` to get a more detailed prompt suggestion. This can be especially useful before using the more expensive models or higher steps, ensuring you get a good result on the first try.

***

### Error Handling and Status Codes

The API returns standard HTTP status codes to indicate success or failure. Common codes and error scenarios include:

* **200 OK:** The request was successful. For `GET` requests, the response body contains the requested data. For `POST` image generation, a 200 indicates the image (or job) was successfully generated/queued. Even some asynchronous calls (like generate-nft-queue) will return 200 upon successfully queuing the job. The actual outcome (image ready or not) is then determined by progress status.
*   **400 Bad Request:** The request was malformed or missing required parameters. This can occur if you omit a required field (e.g., no prompt or missing walletAddress/chainId for NFT generation), or if a parameter value is out of range (e.g., unsupported image dimensions for the chosen model). The response typically includes a message explaining what was wrong in the input. For example, you might get an error JSON:

    ```json
    { "error": "height and width exceed allowed resolution for model velogen" }
    ```
* **401 Unauthorized:** Authentication failed. This happens if the `Authorization` header is missing, invalid, or the API key has no credits/expired. Ensure you provided the correct Bearer token. If your credits are exhausted, you may also receive a 401 or an error message indicating insufficient credits (in some cases this might also be a 402 Payment Required or a 429; check the error message).
* **429 Too Many Requests:** You have hit a rate limit or usage limit. The API might throttle requests if you call too frequently in a short time span. Additionally, if you attempt operations without credits, the service could respond with a 429-like error indicating you cannot proceed. In general, a 429 means slow down or check your credit balance. The response may include a `Retry-After` header or a message like "Rate limit exceeded" or "Insufficient credits".
* **500 Internal Server Error:** The server encountered an unexpected error. This could be due to an internal issue processing your request (for example, a model service failing). These are not common, but if encountered, you may retry after some time. If the issue persists, it may indicate a problem on the service side that the ChainGPT team needs to address.
* **503 Service Unavailable:** (Less common) Could indicate temporary downtime or maintenance.

When an error occurs (any 4xx or 5xx status), the response body usually contains a JSON with an error message for debugging. For example:

```json
{ "error": "Model name not recognized. Supported models: velogen, nebula_forge_xl, ..." }
```

Using the SDK or client libraries, such errors might be thrown as exceptions (e.g., an `NftError` in the ChainGPT SDK). Always implement error handling to catch these responses.

***

### Example Workflow

To illustrate a full integration workflow using the above endpoints, here’s a brief example:

1. **Generate an NFT image (queued):** Call **POST** `/nft/generate-nft-queue` with the prompt, model, wallet address, etc. Receive a `collectionId` .
2. **Poll for progress:** Periodically call **GET** `/nft/progress/{collectionId}` to check if the image generation is complete . (Optionally, you could also just wait some predetermined time if you expect it to finish without polling.)
3. **Enhance prompt (optional):** If you’re not satisfied with the results or want to try another image, you could use **POST** `/nft/enhancePrompt` on your prompt and repeat generation with a new prompt.
4. **Finalize and get metadata:** Once the image is ready, call **POST** `/nft/mint-nft` with the `collectionId` and your desired NFT name/description . Receive the metadata and image link for the NFT.
5. **Mint on-chain:** Using the data from the previous step, call the NFT contract’s `mint` function. You can get the contract’s ABI from **GET** `/nft/abi` and the chain details from **GET** `/nft/get-chains` to know which contract address to use. The actual mint transaction can be done via a Web3 library or, if ChainGPT provides a service, it may already be minted (check the response or documentation for your specific case).
6. **Result:** The NFT is now minted to the provided wallet address on the chosen chain, containing the AI-generated image and metadata.

This API-centric flow enables developers to programmatically create and mint AI-generated NFTs, integrating ChainGPT’s AI capabilities into websites, dApps, or services with ease.
