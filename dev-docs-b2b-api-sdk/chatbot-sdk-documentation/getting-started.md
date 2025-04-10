# Getting Started

### 1. INTRODUCTION

#### 1.1    Purpose and Scope of General Chatbot SDK

Our general chatbot SDK (Software Development Kit) serves as a library/ docs that developers can use to build and integrate our general chatbot functionality into their applications, platforms, or services.&#x20;

#### 1.2    Audience of document / SDK

This documentation is intended to provide a brief overview of General Chatbot SDK,  its features and usage guidelines for developers, software engineers, and system integrators who are using or planning to use our SDK.

#### 1.3 Prerequisites

Before using the SDK, ensure that you have the following:

* Supported operating system  ( Windows, Linux etc.  having Node installed )
* Development environment (IDE, Node etc. )
* Access to necessary API keys&#x20;
* Credits in the web application [https://app.chaingpt.org/](https://app.chaingpt.org/)

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

### 2. Installation

To install the SDK, follow the instructions, follow the steps below

1. Create an application in Node
2. Open terminal in linux based operating system and command prompt in windows
3. Run following commands

```shell
npm install --save @chaingpt/generalchat
# or
yarn add generalchat
```

### 3. Configuration

* Credits System&#x20;

User must start by having credits in the web-App  [https://app.chaingpt.org/](https://app.chaingpt.org/). These credits will allow users to create API key and access SDK. On each request made using SDK, credits shall be deducted.

* Setup API Key

The ChainGPT API SDK uses API keys for authentication. In Web app , Go to API Dashboard module and generate an API Key using create secret key feature and copy.  &#x20;

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXfPhftjCj6HSdKNxKiNuiSwPdGruk6kEdFGqtnL3EdsYh98vZn8AKojMBggPoJjth19wMH8QKM4lRcbVVJghhDKzazIlXPFXGcqx8z8gKoMBpmb7MlFGTPI4cnVMLnP2A1-7DwHOw?key=BZ2FNEbHjC87wc5CO56IlqCS" alt=""><figcaption></figcaption></figure>

This is a relatively straightforward way to control access, but you must be vigilant about securing these keys. Avoid exposing the API keys in your code or in public repositories; instead, store them in a secure location. You should expose your keys to your application using environment variables or secret management service, so that you don't need to hard-code them in your codebase.

### **Pricing**

The General Chatbot SDK from ChainGPT operates on a credit-based system. Each request made through the SDK deducts **0.5 CGPTc** from the user's account. This pay-per-use model ensures flexibility and scalability, allowing users to only pay for what they need.

* **Stream Response:** Each chat API request costs **0.5 CGPTc**. If chat history is enabled, an additional **0.5 CGPTc** is deducted per request.
* **BLOB Response:** Similarly, each BLOB request costs **0.5 CGPTc**, with chat history also incurring an extra **0.5 CGPTc** per call.

Users can monitor and manage their credit usage via the ChainGPT Webapp, aligning with ChainGPT’s commitment to transparency and accessibility.

For additional details or to manage your credits, users should visit the [ChainGPT Webapp](https://app.chaingpt.org/).

## Support

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)

[**Disclaimer**](../../../misc/legal-docs/disclaimer.md)
