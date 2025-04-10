# SDK Components

## API Key Usage

The API supports retrieving responses in 2 Formats:

1\. **Stream Response**\
2\. **Blob response**

Our SDK also provides an option to incorporate custom context, allowing clients to tailor the response to their specific requirements.\
\
The following cases illustrate how the API key can be utilized effectively.\


### Stream Response

#### **Case 1: Simple General Chat (without Custom Context)**

To retrieve a response using the API's default settings, the following code snippet should be used. The user must input their API key, generated through the API dashboard, in the highlighted section.

```javascript
import { GeneralChat } from '@chaingpt/generalchat';

const generalchat = new GeneralChat({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const stream = await generalchat.createChatStream({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off", // Set to "on" to save chat history; otherwise, keep it "off"
  });
  stream.on('data', (chunk: any)=>console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main();
```



#### Case 2: Use Custom Context From [AI Hub](https://app.chaingpt.org/apidashboard)

Step 1: Create Your API Key (as mentioned in Configurations)

* Obtain your API key from [AI Hub](https://app.chaingpt.org/apidashboard) to access the SDK.

Step 2: Add Company Details from AI Hub/ User Side

* Define your company details, including name, description, website, whitepaper, and chatbot purpose in apiKey (additional information).

Step 3:  Set Custom Context true (Additional information fetched from AI Hub)

* When using CustomContext is true and no context Injection object is provided, the default custom context is fetched based on the API key (additional information).

```javascript
import { GeneralChat } from '@chaingpt/generalchat';
import { AI_TONE, BLOCKCHAIN_NETWORK, PRE_SET_TONES } from "@chaingpt/generalchat";

const generalchat = new GeneralChat({
  apiKey: 'Your ChainGPT API Key',
});

async function main() {
  const stream = await generalchat.createChatStream({
    question: 'Explain quantum computing in simple terms',
    chatHistory: "off", // Set to "on" to save chat history; otherwise, keep it "off"
    useCustomContext: true // When useCustomContext is true and no contextInjection is provided
  });
  stream.on('data', (chunk: any)=>console.log(chunk.toString()));
  stream.on('end', () => console.log("Stream ended"));
}

main()
```



#### Case 3: Use Custom Context From API/ SDK

The _contextInjection_ object is optional, and all its fields are optional. You can provide additional company details for a more tailored chatbot experience . Any chunk of information added here will override the company default details (custom context) that are added from AI Hub.

```javascript
const contextInjection = {
companyName: "Company name for AI chatbot",
companyDescription: "Company description for AI chatbot",
companyWebsiteUrl: "https: //www.example.com",

whitePaperUrl: “https: //www.example.com",

purpose: “Purpose of AI Chatbot for this company.”,
cryptoToken: true,

tokenInformation: {

tokenName: "Token name for AT chatbot"
tokenSymbol: "$TOKEN",
tokenAddress: “Token address for AI chatbot"

tokenSourceCode: "Token source code for AI chatbot

tokenAuditUrl: "https: //www.example.com",

exploreUrl: “https: //www.example.com",

cmcUrl: “https: //www.example.com",

coingeckoUr1
blockchain: [BLOCKCHAIN_NETWORK.ETHEREUM, BLOCKCHAIN_NETWORK.POLYGON_ZKEV!
3

socialMediaUrls: [{ name: "LinkedIn", ur

https: //wmw.example.com",

"https: //abc.com" }],
limitation: false,

aiTone: AI_TONE.PRE_SET_TONE,

selectedTone: PRE_SET_TONES. FRIENDLY,

customTone:

‘Only for custom AT Tone”
};
```



### Example Usage

#### **Example 1:** Using Minimal context/ chunk of context using SDK.

The following examples demonstrate how minimal context injection can be utilized to obtain the desired response. A code snippet can be added via the SDK to override specific information added from the AI Hub while preserving all other data.

```javascript
import { GeneralChat } from '@chaingpt/generalchat';
import { AI_TONE, BLOCKCHAIN NETWORK, PRE_SET_TONES } from “@chaingpt/generalchat;

const generalchat = new Generalchat ({
apikey: "Your ChainGPT API Key’,
});

async function main() {
const stream = await generalchat.createChatStream({
question: ‘Explain quantum computing in simple terms’,
chatHistory: "off",
useCustomContext: true,
contextInjection: {
aiTone: AI_TONE.PRE_SET_TONE,
selectedTone: PRE_SET_TONES.FORMAL,
}
});

stream.on(‘data', (chunk: any) => console. log(chunk.toString()));
stream.on(‘end', () => console. log("Stream ended"));

main();

```

#### **Example 2:** Using extended context injection using sdk.

The following examples demonstrate how detailed/ extended context injection can be utilized to obtain the desired response. A code snippet can be added via the SDK to override all the company information added from the AI Hub.

```javascript
import { GeneralChat } from '@chaingpt/generalchat';
import { AI_TONE, BLOCKCHAIN NETWORK, PRE_SET_TONES } from “@chaingpt/generalchat,

const generalchat = new GeneralChat({
apikey: 'Your ChainGPT API Key",

async function main() {
const stream = await generalchat.createChatStream({
question: "Explain blockchain security best practices",

chatHistory: "on",

context Injection

companyName: “Crypto Solutions",
companyDescription: "A blockchain security firm",
cryptoToken: true,
tokenInformation: {
tokenName: "SecureToken",
tokenSymbol: "$SEC",
blockchain: [BLOCKCHAIN_NETWORK.ETHEREUM]
hb
socialMediaUrls: [{ name: "Twitter", url: "https: //twitter.com/cryptosolutic
aiTone: AI_TONE.CUSTOM_TONE,
customTone: “Provide detailed security explanations.”
3
Ys

stream.on(‘data', (chunk: any) => console. log(chunk.toString()));

stream.on(‘end', () => console. log("Stream ended"));

main();
```

### Additional Parameters Usage

We take inputs for various types of blockchains , Tones and styles , Social media links. The input are taken is following format to enhance the response generated by Chat bot

#### Blockchain Options

We support multiple blockchains to fetch and display crypto token information for chat interactions.

```javascript
blockchain = [ ETHEREUM, BSC, ARBITRUM, BASE, BLAST, AVALANCHE, POLYGON, SCROLL,
OPTIMISM, LINEA, ZKSYNC, POLYGON_ZKEV, GNOSIS, FANTOM, MOONRIVER, MOONBEAM, BOBA,
METIS, LISK, AURORA, SEI, IMMUTABLE_ZK, GRAVITY, TATKO, CRONOS, FRAXTAL, ABSTRACT.
WORLD_CHAIN, MANTLE, BERACHAIN
]
```

#### AI Tones

Our AI supports multiple tone options to customize responses based on user preferences. The following tone selection options are available:

1. DEFAULT\_TONE // if we select DEFAULT\_TONE then no need for customTone and selectedTone
2. CUSTOM\_TONE // if we select CUSTOM\_TONE then customTone text is required
3. PRE\_SET\_TONE // if we select PRE\_SET\_TONE then selectedTone is required

#### Preset AI Tones:

If PRE\_SET\_TONE is selected, users can choose from the following tones: If we select PRE\_SET\_TONE then we select given option:&#x20;

```
1) PROFESSIONAL
2) FRIENDLY
3) INFORMATIVE
4) FORMAL
5) CONVERSATIONAL
6) AUTHORITATIVE
7) PLAYFUL
8) INSPIRATIONAL
9) CONCISE
10) EMPATHETIC
11) ACADEMIC
12) NEUTRAL
13) SARCASTIC_MEME_STYLE
```



#### Social Media Options

```javascript
socialMediaUrls = [
  {name: "linkedIn", url: "https://www.example.com"},
  {name: "telegram", url: "https://www.example.com"},
  {name: "instagram", url: "https://www.example.com"},
  {name: "twitter", url: "https://www.example.com"},
  {name: "youtube", url: "https://www.example.com"},
  {name: "medium", url: "https://www.example.com"}
]
```



### History Feature

Our chatbot SDK also stores the chat history for later usage.  History is enabled/ Disabled within the code using Chat History function. If chat history is “On” it means that history of requests shall be saved and can be retrieved later . If the Chat History function is “Off” then History won’t be saved and can’t be accessed later.



```javascript
import { GeneralChat } from '@chaingpt/generalchat';

const generalChat = new GeneralChat({
    apiKey: 'Your ChainGPT API Key',
});

async function main() {
    const response = await generalChat.createChatBlob({
        question: 'Explain quantum computing in simple terms',
        chatHistory: "on",
    });

    console.log(response.data.bot);
}

main();
```

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXfktrtOWIgAnfquFX6fOdL0wFXSdE6ERG8RnhjO943ye7FFG4PaIvZwdEpm2JT0yfCj3Kh6IJ-JFTiEsqvmUR-oE5JeQEgyzLvYSwvJ70K1Ptz68zSyVo8GghyywHCAP0_yaQyUow?key=BZ2FNEbHjC87wc5CO56IlqCS" alt=""><figcaption></figcaption></figure>

Users can retrieve the history using the following code:

```javascript
import { GeneralChat } from '@chaingpt/generalchat';

const generalChat = new GeneralChat({
    apiKey: 'Your ChainGPT API Key',
});

async function main() {
    const response = await generalChat.getChatHistory({
        limit: 10,
        offset: 0,
        sortBy: "createdAt",
        sortOrder: "ASC"
    });

    console.log(response.data.rows);
}

main();
```

### Pricing/Credits Deduction

In case of both stream and Blob Response , On sending each chat API request, 1 Credit will be deducted from the user account in AI HUB.&#x20;

In addition, if user enables chat history feature, an additional 1 Credit will be deducted\


### Error Handling

When the library is unable to connect to the API, or if the API returns a non-success status code (i.e., 4xx or 5xx response), an error of the  class GeneralChatError  will be thrown:

```javascript
import { Errors } from '@chaingpt/generalchat';

async function main() {
  try {
    const stream = await generalchat.createChatStream({
      question: 'Explain quantum computing in simple terms',
      chatHistory: "off"
    });
    stream.on('data', (chunk: any)=>console.log(chunk.toString()));
    stream.on('end', () => console.log("Stream ended"));
  } catch(error) {
    if(error instanceof Errors.GeneralChatError) {
      console.log(error.message)
    }
  }
}

main();
```

### Language/Framework Compatibility

Our SDK supports JavaScript language and will run on Node application&#x20;



### Security Considerations

1. To ensure the security, SDK is made accessible using an authentication key. User having credits in the web-app and valid API key shall e able to access the SDK
2. Request limitation has been handled to avoid any misuse. Users are allowed to make 200 requests per minute and on each request 1 credit shall be deducted.&#x20;



### Code Documentation

Please check out this link for SDK code documentation\


[https://www.npmjs.com/package/@chaingpt/generalchat\
](https://www.npmjs.com/package/@chaingpt/generalchat)

### Support/Help

If you need further assistance, explore the available support channels provided in our website [https://www.chaingpt.org/contact-us](https://www.chaingpt.org/contact-us)

***

[**Disclaimer**](../../../misc/legal-docs/disclaimer.md)
