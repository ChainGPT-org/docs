# Getting Started

## Installation

To install the SDK, follow the instructions, follow the steps below

1. Create an application in Node.&#x20;
2. Open terminal in Linux-based operating system and command prompt in Windows.
3. Run the following commands.



```
npm install --save @chaingpt/smartcontractgenerator
# or
yarn add smartcontractgenerator
```

## Configuration

### Credits System&#x20;

Users must start by having credits in the web app, https://app.chaingpt.org/. These credits will allow users to create API keys and access SDK. Credits will be deducted from each request made using SDK.

### Setup API Key&#x20;

The ChainGPT API SDK uses API keys for authentication. In the Web app , Go to the API Dashboard module and generate an API Key using the create secret key feature and copy.  &#x20;

<figure><img src="https://lh7-us.googleusercontent.com/c4GnH1fl_AqfXvifI_kOE_-E-JNRAz7XHeMeI9Zw7ryDK5PNKDnx2MbM1qZhMH5e-YkOTNsb20NbkNBkm8dz-bYB7r3AqFVvwOf-yLMIAIWybXXDxL6UBYggi9ZbfkTgz8eLBAaSMZ8pCWck85m9Ecw" alt=""><figcaption></figcaption></figure>

This is a relatively straightforward way to control access, but you must be vigilant about securing these keys. Avoid exposing the API keys in your code or public repositories; instead, store them in a secure location. You should expose your keys to your application using environment variables or a secret management service so you don't need to hard-code them in your codebase.



## Pricing

**Stream Response**

Each LLM Chatbot request made with the Smart Contract Generator SDK deducts 2 CGPTc from the user's account.\
If the chat history feature is enabled, an additional 1 CGPTc is deducted.

**Blob Response**

Each API request also deducts 2 CGPTc, with chat history enabled adding 1 CGPTc to the request total.

**Get Chat History**

No additional credits are deducted when retrieving chat history, as the cost is covered when the history feature is initially enabled during the blob or stream request.

Users can monitor and manage their credit usage via the [Webapp](https://app.chaingpt.org/), aligning with ChainGPT's user-centric approach to accessibility and transparency in service provision.&#x20;

For additional details or to manage your credits, users should visit the [ChainGPT Webapp](https://app.chaingpt.org/).



## Support

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)



[**Disclaimer**](../../../misc/legal-docs/disclaimer.md)\
