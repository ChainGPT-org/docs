# Getting Started

## **Installation**

To install the General Chatbot SDK, follow these steps:

1. **Create a Node Application**: Begin by setting up an application in Node.js.
2. **Open Terminal or Command Prompt**:
   * Use the terminal for Linux-based operating systems.
   * Use the command prompt for Windows.
3. **Run Installation Commands**: Execute the specific commands provided to install the SDK.

```
pm install --save @chaingpt/generalchator
#or
yarn add generalchat
```

## **Configuration**

### **Credits System**

* **Initial Setup**: Start by acquiring credits in the web application [https://app.chaingpt.org/](https://app.chaingpt.org/). These credits are necessary to create an API key and access the SDK functionalities.
* **Usage and Deduction**: Credits will be deducted with each request using the SDK.

### **Setup API Key**

* **Generating API Key**: Utilize the ChainGPT API SDK for authentication. In the web application, navigate to the API Dashboard and use the 'Create Secret Key' feature to generate and copy your API key.

<figure><img src="https://lh7-us.googleusercontent.com/DkCb0arDB_K2KasqeW3cfDJ0GX7oRFCrGmT7lU8YcCeaOEPJtBPpOiMY7TE4lM319xnutqHI2LRsBqoUiHJ9laROTzD2b-l3ook0AZ4ujkVC8qYE4xyYipT_6H_dPVeSf_8wIInPD8K5Ju07YtubCFY" alt="" width="563"><figcaption></figcaption></figure>

* **Securing API Key**:
  * It is crucial to secure your API keys to control access effectively.
  * Avoid exposing the API keys in your code or public repositories.
  * Store them in a secure location, and use environment variables or a secret management service for exposure to your application.
  * This method ensures you do not need to hard-code them in your codebase, enhancing security.

## Pricing

The General Chatbot SDK from ChainGPT operates on a credit-based system. Each request made through the SDK deducts one credit from the user's account. This model facilitates a pay-per-use structure, ensuring that users only pay for what they need, allowing for both flexibility and scalability.&#x20;

1. **Stream Response**: Each chat API request costs 1 Credit. Enabling chat history will result in an additional credit deduction.
2. **BLOB Response**: Similar to Stream Response, each request deducts 1 Credit, with chat history also incurring an extra credit cost.

Users can monitor and manage their credit usage via the [Webapp](https://app.chaingpt.org/), aligning with ChainGPT's user-centric approach to accessibility and transparency in service provision.&#x20;

For additional details or to manage your credits, users should visit the ChainGPT [Webapp](https://app.chaingpt.org/).

## Support

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)

[**Disclaimer**](../../../misc/legal-docs/disclaimer.md)
