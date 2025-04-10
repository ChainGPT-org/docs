# Getting Started

## I**nstallation**

To install the AI News SDK, please follow these steps:

1. **Create a Node Application**: Initiate a new application in your Node.js environment.
2. **Open Terminal or Command Prompt**:
   * For Linux-based operating systems, use the terminal.
   * For Windows, use the command prompt.
3. **Run Installation Commands**: Execute the provided commands in your terminal or command prompt to install the SDK.

{% code fullWidth="false" %}
```plaintext
npm install --save @chaingpt/ainews
# or
yarn add ainews
```
{% endcode %}

## **Configuration**

### **Credits System**

* **Initial Setup**: Start by acquiring credits in the web application [https://app.chaingpt.org/](https://app.chaingpt.org/). These credits are necessary to create an API key and access the SDK functionalities.
* **Usage and Deduction**: Credits will be deducted with each request using the SDK.

### **Setup API Key**

* **Generating API Key**: Utilize the ChainGPT API SDK for authentication. In the web application, navigate to the API Dashboard and use the 'Create Secret Key' feature to generate and copy your API key.

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>



### **Securing API Key**:

* It is crucial to secure your API keys to control access effectively.
* Avoid exposing the API keys in your code or public repositories.
* Store them in a secure location, and use environment variables or a secret management service for exposure to your application.
* This method ensures you do not need to hard-code them in your codebase, enhancing security.



## Pricing

The AI News SDK operates on a credit-based pricing model. Each request made through the SDK deducts 1 CGPTc (ChainGPT credit) from the user’s account.

This model facilitates a pay-per-use structure, ensuring users only pay for what they consume. The approach balances flexibility, transparency, and scalability for all users — from individuals to developers.

Users can monitor and manage their credit usage via the Webapp, aligning with ChainGPT’s user-centric approach to service clarity.

For additional details or to manage your credits, users should visit the  [ChainGPT Webapp](https://app.chaingpt.org/).



## Support

For additional assistance, refer to Discord - [https://discord.gg/chaingpt](https://discord.gg/chaingpt)



[**Disclaimer**](../../misc/legal-docs/disclaimer.md)
