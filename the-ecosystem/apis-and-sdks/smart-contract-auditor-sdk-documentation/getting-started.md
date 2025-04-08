# Getting Started

## Installation

To install the SDK, follow the instructions, follow the steps below

1. Create an application in Node.
2. Open terminal in Linux-based operating system and command prompt in Windows.
3. Run the following commands.

```
npm install --save @chaingpt/smartcontractauditor
# or
yarn add smartcontractgenerator
```

## Configuration

### Credits System&#x20;

Users must start by getting credits on the web app, https://app.chaingpt.org/. These credits will allow users to create API keys and access SDK. Credits will be deducted from each request made using SDK.



### Setup API Key&#x20;

**Generating API Key**: Utilize the ChainGPT API SDK for authentication. In the web application, navigate to the API Dashboard and use the 'Create Secret Key' feature to generate and copy your API key.



**Secret key feature and copy.**  &#x20;

<figure><img src="https://lh7-us.googleusercontent.com/JPzAcw4zRXGExj0Kubccsc41yLBVog50sjU_ws0eGcpt5BXq6CCpOe5gtfLqVF67H7BQ8MF1vkvc-VH-oRSN-tjCm5Dhdb6xjISRt0MNScKcsqPcsMFiGwuz2jeG_G8wucDR0urplhTjVAjifU12H-4" alt=""><figcaption></figcaption></figure>

### **Securing API Key**:

* It is crucial to secure your API keys to control access effectively.
* Avoid exposing the API keys in your code or public repositories.
* Store them in a secure location, and use environment variables or a secret management service to expose them to your application.
* This method ensures you do not need to hard-code them in your codebase, enhancing security.

## Pricing

#### **Stream Response**

Each LLM Chatbot request sent via the Smart Contract Auditor SDK deducts 1 CGPTc from the user's account.\
If the chat history feature is enabled, an additional 1 CGPTc is deducted.

**Blob Response**

Similar to the stream response, each request deducts 1 CGPTc from the user’s account.\
Chat history, if enabled, adds 1 CGPTc per request.

**Get Chat History**

No additional credits are deducted for accessing chat history if it was previously enabled during the original stream or blob request.

Users can monitor and manage their credit usage via the Web App, which aligns with ChainGPT's user-centric approach to accessibility and transparency in service provision.

For additional details or to manage your credits, users should visit the ChainGPT Webapp.\


Users can monitor and manage their credit usage via the [Webapp](https://app.chaingpt.org/), aligning with ChainGPT's user-centric approach to accessibility and transparency in service provision.&#x20;

For additional details or to manage your credits, users should visit the [ChainGPT Webapp](https://app.chaingpt.org/).



## Support

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)

\
[**Disclaimer**](../../../misc/legal-docs/disclaimer.md)



\
\
