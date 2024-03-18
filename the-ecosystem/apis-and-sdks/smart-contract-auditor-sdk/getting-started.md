# Getting Started

## Installation

To install the SDK, follow the instructions, follow the steps below

1. Create an application in Node.
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

**Generating API Key**: Utilize the ChainGPT API SDK for authentication. In the web application, navigate to the API Dashboard and use the 'Create Secret Key' feature to generate and copy your API key.



**Secret key feature and copy.**  &#x20;

<figure><img src="https://lh7-us.googleusercontent.com/JPzAcw4zRXGExj0Kubccsc41yLBVog50sjU_ws0eGcpt5BXq6CCpOe5gtfLqVF67H7BQ8MF1vkvc-VH-oRSN-tjCm5Dhdb6xjISRt0MNScKcsqPcsMFiGwuz2jeG_G8wucDR0urplhTjVAjifU12H-4" alt=""><figcaption></figcaption></figure>

### **Securing API Key**:

* It is crucial to secure your API keys to control access effectively.
* Avoid exposing the API keys in your code or public repositories.
* Store them in a secure location, and use environment variables or a secret management service for exposure to your application.
* This method ensures you do not need to hard-code them in your codebase, enhancing security.

## Pricing

#### Stream Response

1. On sending each chat API request, 5 Credits will be deducted from the user account in WebApp.&#x20;
2. In addition, if the user enables the chat history feature, an additional 1 Credit will be deducted.

#### Blob Response

1. On sending each API request, 5 Credits will be deducted from the user account in WebApp.&#x20;
2. In addition, if the user enables the chat history feature, an additional 1 Credit will be deducted.

#### Get Chat history

1. Since the user has already enabled the chat history feature while calling stream/blob response chat API, no additional credits will be deducted on calling the Get credit history API.

## Support

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)

\




\
\
