# SDK Components

## **API Key Usage**

To use the API key:

1. Execute the specified commands.
2. Paste the generated API key in the “Your ChainGPT API key” placeholder.

```javascript
const { Nft } = require('@chaingpt/nft');

const nftInstance = new Nft({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const generatedImage = await nftInstance.generateNft({
    walletAddress: "", // Public Key of the wallet that will mint NFT
    prompt: "", // Prompt to be used to generate the NFT art
    model: "" // Model to be used to generate the NFT art
  });
}

```

##

## Core Libraries

We provide TypeScript/ JavaScript libraries with support for Node.js. Install it by running:



Once installed, you can use the library and your secret key to run the following:

```javascript
const { Nft } = require('@chaingpt/nft');

const nftInstance = new Nft({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const generatedImage = await nftInstance.generateNft({
    walletAddress: "", // Public Key of the wallet that will mint NFT
    prompt: "", // Prompt to be used to generate the NFT art
    model: "" // Model to be used to generate the NFT art
  });
}

main();

```



***

## Advanced Features&#x20;

ChainGPT NFT Generator SDK provides following features&#x20;

### 1. Generate simple NFT

#### Request Params:

1. User’s wallet Address (String)
2. User’s Prompt (String)
3.  Model name (String)

    1. ChainGPT
    2. RealMimic
    3. Animatrix
    4. PixelMaster
    5. ThreeDImageForge
    6. DrawDreamer
    7. DrawDreamerV2’
    8. AnimeVerse
    9. RealVision
    10. FantasyVerse
    11. VisionaryForge

    Note: You must have to select any of the model above.\
    \


    \


```javascript
const { Nft } = require('@chaingpt/nft');

const nftInstance = new Nft({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const generatedImage = await nftInstance.generateNft({
    walletAddress: "", // Public Key of the wallet that will mint NFT
    prompt: "", // Prompt to be used to generate the NFT art
    model: "" // Model to be used to generate the NFT art
  });
}

main();

```

### &#x20;2. Generate an NFT and queue the generation instead of waiting for it to complete



```javascript
const { Nft } = require('@chaingpt/nft');

const nftInstance = new Nft({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const generatedImageQueue = await nftInstance.generateNftWithQueue({
    walletAddress: "", // Public Key of the wallet that will mint NFT
    prompt: "", // Prompt to be used to generate the NFT art
    model: "" // Model to be used to generate the NFT art
  });
}

main();

```



### 3. Check the creation progress of your NFT Generation

It's useful with the function: generateNftWithQueue(). It allows users to see the progress of their NFT generation. If users generate a huge amount of NFTs, they can always see how many NFTs are generated and remaining.

```javascript
const { Nft } = require('@chaingpt/nft');

const nftInstance = new Nft({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const getProgress = await nftInstance.getNftProgress({
    collectionId: "" // your NFT generation collection ID
  });
}

main();

```

### 4. Retrieving the required data to mint the generated NFT

After generating the NFT, users can now mint the NFT on the chain using the code below. This chunk of code will provide the required data for minting the generated NFT

```javascript
const { Nft } = require('@chaingpt/nft');

const nftInstance = new Nft({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const mint = await nftInstance.mintNft({
    collectionId: "", // Your NFT generation Collection ID
    name: "", // A name for your NFT
    description: "" // A description for your NFT
  });
}

main();

```

### &#x20;5.  Retrieving the NFT Mint Factory Contract ABI to call the Mint function

```javascript
const { Nft } = require('@chaingpt/nft');

const nftInstance = new Nft({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const abi = await nftInstance.abi();
}

main();
```

### &#x20;

## Error Handling

When the library is unable to connect to the API, or if the API returns a non-success status code (i.e., 4xx or 5xx response), an error will be thrown:

```javascript
// Error handler of NFT SDK
const { Nft } = require('@chaingpt/nft');

const nftInstance = new Nft({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  try {
    const mint = await nftInstance.mintNft({
      collectionId: "", // Your NFT generation Collection ID
      name: "", // A name for your NFT
      description: "" // A description for your NFT
    });
  } catch (error) {
    console.log("error", error.message);
  }
}

main();
```



***

## Language/ Framework Compatibility.

Our SDK supports Javascript language and will run on Node applications.&#x20;

## Security Considerations

1. To ensure security, the SDK is made accessible using an authentication key. Users having credits in the web-app and a valid API key shall be able to access the SDK

## Release Version&#x20;

Release history is maintained. However, this is the first release; in the future, more features will be added, and the latest version will be released for users.&#x20;

## Code Documentation&#x20;

Please check out this link for SDK code documentation&#x20;

[https://www.npmjs.com/package/@chaingpt/nft](https://www.npmjs.com/package/@chaingpt/nft)

\
Support/ Help&#x20;
-------------------

If you need further assistance, explore the available support channels provided on our website [https://www.chaingpt.org/](https://www.chaingpt.org/) or check our [Discord](https://discord.com/invite/sv2NfqSgVW).&#x20;

\
\


\
