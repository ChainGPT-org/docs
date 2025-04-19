# SDK Reference

## ChainGPT AI News Generator SDK Reference

The ChainGPT AI News Generator SDK provides a convenient way for developers to fetch the latest AI-related news articles in their Node.js applications. Using this SDK, you can quickly integrate up-to-date AI, crypto, and Web3 news into your platform without dealing with web scraping or data processing yourself. Key features include filtering news by category (e.g. DeFi, NFT), sub-category (specific blockchains like Ethereum, Solana), associated tokens (like ETH, BTC), keyword search, and date range. All data is delivered in real-time with minimal duplication, allowing you to stay current with relevant news while avoiding bias or noise.

***

### Installation

Install the SDK via npm or yarn. Ensure you have Node.js installed in your development environment.

```
# npm
npm install --save @chaingpt/ainews

# yarn
yarn add @chaingpt/ainews
```

This package includes TypeScript type declarations for type-safe integration. It supports both CommonJS and ESM projects.

***

### Initialization and API Key Configuration

Before using the SDK, you need to obtain an API key from ChainGPT and have sufficient credits in your account (each API call consumes credits, explained later). Follow these steps to get set up:

1. **Get an API Key:** Log in to the [ChainGPT web app](https://app.chaingpt.org/) and navigate to the **API Dashboard**. Use the "Create Secret Key" feature to generate a new API key. Copy the generated key (it will be shown only once).
2. **Add Credits:** Ensure your ChainGPT account has enough credits. Each news fetch call deducts credits from your balance, 1 credit per 10 results, and 1 credit per request (e.g. 25 items → 3 credits | 3 items -> 1 credit).”
3. **Secure the Key:** Store the API key securely, for example in an environment variable. **Do not** hard-code it in public code or commit it to source control.

With the API key in hand, initialize the SDK in your Node.js project:

```
import { AINews } from '@chaingpt/ainews';

const ainews = new AINews({
  apiKey: process.env.CHAINGPT_API_KEY  // or 'YOUR_API_KEY'
});
```

This creates a new `AINews` client instance configured with your API key. You should reuse this instance for all calls. Now you're ready to fetch news articles.

***

### Using the `getNews` Method

The primary method provided by the SDK is `ainews.getNews()`, which retrieves AI news articles according to the criteria you specify. If called with no parameters (or an empty object), it will return the latest news articles across all categories.

**Basic usage example:**

```
import { AINews } from '@chaingpt/ainews';

const ainews = new AINews({ apiKey: 'YOUR_API_KEY' });

async function main() {
  // Fetch the latest 10 AI news articles (default behavior)
  const response = await ainews.getNews({});
  console.log(response.data);
}

main();
```

In the above example, `response.data` will contain an array of news articles (by default, up to 10 articles). Each article in the array is an object with fields like title, description, etc. (see **Response Structure** below for details). The SDK returns a promise, so you can also use `.then()`/.catch() instead of `async/await` if preferred.

#### Filtering & Search Options

You can pass an object to `getNews()` with various optional parameters to filter the news results. All filters are **optional**; omit them to get a general feed. The available filters are:

* **`categoryId`** (`number[]`, optional) – An array of Category IDs to filter news by broad category or topic (e.g. DeFi, NFT, Metaverse). Only articles belonging to the specified category(s) will be returned. For example, `categoryId: [8]` will fetch news in the _NFT_ category. You can provide multiple IDs to include several categories. (See the Category IDs reference table for the list of categories and their IDs.)
* **`subCategoryId`** (`number[]`, optional) – An array of Sub-Category IDs to filter news by specific blockchain ecosystem or network (e.g. Bitcoin, Ethereum, Solana, etc.). For example, `subCategoryId: [15]` will fetch news related to _Ethereum_. This lets you narrow results to a particular chain or protocol. You can include multiple IDs for multiple sub-categories. (See Sub-Category IDs for the full list.)
* **`tokenId`** (`number[]`, optional) – An array of Token IDs to filter news by a specific cryptocurrency or token. For example, `tokenId: [80]` will fetch news associated with _Ethereum (ETH)_. Use this to get news about particular coins or tokens. Multiple token IDs can be provided. (See Token IDs for reference.)
* **`searchQuery`** (`string`, optional) – A text keyword or phrase to search for in the news title and description. This is a free-text search filter. For example, `searchQuery: "regulation"` will return news articles whose title or description contain "regulation". Use this to find news on specific topics or keywords.
* **`fetchAfter`** (`Date` or date string, optional) – Only return news published after the specified date/time. Provide a JavaScript `Date` object or a date string (which will be interpreted in UTC by the API). For example, `fetchAfter: new Date('2024-01-01')` will fetch news items published from January 1, 2024 onward. This is useful for getting news within a certain time range (e.g., only recent news).

You can combine multiple filters in one request. For instance, you could query for news in the NFT category about Ethereum by providing both a `categoryId` and `subCategoryId`, or include a `searchQuery` to find specific content within those filtered results.

#### Pagination & Sorting

By default, `getNews` returns the first page of results (up to 10 articles, sorted by newest first). You can control pagination and sorting with these parameters:

* **`limit`** (`number`, optional) – The number of news articles to return (page size). Default is **10** if not specified. You can increase this to retrieve more articles in one call (e.g. `limit: 20` for 20 articles). Note that higher limits will consume additional credits (see **Rate Limits & Credits**). If you only want a small number of the latest articles, you can set a smaller limit as well.
* **`offset`** (`number`, optional) – The number of items to skip (offset) before starting to return results. Default is **0**. This is used for pagination. For example, to get the second page of results when using a `limit` of 10, you would set `offset: 10` (skip the first 10 articles, return the next set). Similarly, `offset: 20` would fetch the third page (items 21–30), and so on.
* **`sortBy`** (`string`, optional) – The field by which to sort the news. Currently, the only supported sort key is `'createdAt'`, which corresponds to the article's publication time. By default, results are sorted by newest (most recent) first. If not provided, the SDK will sort by `createdAt` descending. (At this time, no other sort fields are supported.)

#### Example: Filtering and Pagination

The following example shows a more advanced usage of `getNews()` combining multiple filters and pagination options. It fetches up to 10 news articles in the **DeFi** category (Category ID 5) that are related to **Ethereum** (Sub-Category ID 15), including only those published after Jan 1, 2024, and then prints out the titles of the returned articles:

```
async function main() {
  try {
    const response = await ainews.getNews({
      categoryId: [5],              // DeFi category
      subCategoryId: [15],          // Ethereum sub-category
      fetchAfter: new Date('2024-01-01'),
      searchQuery: "funding",       // search keyword in title/description
      limit: 10,                    // up to 10 results
      offset: 0,                    // start from the first result
      sortBy: 'createdAt'           // sort by newest
    });
    for (const article of response.data) {
      console.log(article.title);
    }
  } catch (error) {
    console.error("Error fetching news:", error);
  }
}

main();
```

In this example, we combined a category filter (DeFi) with a sub-category (Ethereum) and a search term "funding". We also specified `fetchAfter` to get only recent news, and used the default `limit` of 10. The loop simply logs the title of each news article returned. Adjust `offset` to paginate (e.g., set `offset: 10` to get the next page of results beyond the first 10).

***

### Response Structure

The `ainews.getNews()` method returns a Promise that resolves to an object containing the data (and some metadata). The important part of the response for developers is the `data` field, which is an array of news article objects. Each news object has the following structure:

* **`id`** (`number`) – Unique identifier for the news article (assigned by ChainGPT).
* **`title`** (`string`) – The title or headline of the news article.
* **`description`** (`string`) – A short summary or excerpt of the article content. This gives a quick overview of the news.
* `sourceUrl` (`string`) – The URL of the original full news article on its source website. You can use this link to direct users to read the full article.
* **`categoryId`** (`number`) – The category ID indicating the broad category of the news (e.g., DeFi, NFT, etc.). This corresponds to one of the categories listed in the Category IDs reference table.
* **`subCategoryId`** (`number`) – The sub-category ID indicating the specific blockchain or ecosystem the news is related to (if applicable). This corresponds to one of the IDs in the Sub-Category IDs table. If an article is not specific to a particular blockchain, this may be `null` or not present.
* **`tokenId`** (`number`) – The token ID of a related cryptocurrency if the article is specifically about a certain coin or token. For example, an article about Bitcoin would have `tokenId` for BTC. This corresponds to one of the Token IDs. If the news isn't about a specific token, this field may be omitted or null.
* **`createdAt`** (`string`) – The timestamp when the news article was published (in ISO 8601 format, e.g. `"2024-01-15T09:30:00Z"`). This is used for sorting and can be treated as the publication date/time of the article.

**Example:** After calling `getNews`, you might get data like the following:

```
console.log(response.data);
```

```
[
  {
    "id": 101,
    "title": "AI-Powered Crypto Trading Platform Raises $50M in Funding",
    "description": "A new AI-driven DeFi trading platform has secured $50 million in a Series B round led by ...",
    "url": "https://www.example.com/news/ai-defi-platform-funding",
    "categoryId": 5,
    "subCategoryId": 15,
    "tokenId": 80,
    "createdAt": "2024-01-10T15:30:00Z"
  },
  {
    "id": 102,
    "title": "Ethereum Merge Drives Interest in Sustainable Mining Solutions",
    "description": "Post-Merge Ethereum is prompting miners to explore eco-friendly alternatives as ...",
    "url": "https://www.example.com/news/ethereum-merge-sustainable-mining",
    "categoryId": 5,
    "subCategoryId": 15,
    "tokenId": 80,
    "createdAt": "2024-01-09T08:20:00Z"
  },
  // ...more articles
]
```

Each object in the array represents one news article with the fields described above. Using these fields, you can display the news content in your application (title, description, etc.) and provide a link to the original source (`url`). The categoryId, subCategoryId, and tokenId values can be mapped to human-readable names using the reference tables below if needed.

***

### Error Handling

The SDK is designed to throw a consistent error type when something goes wrong. If the underlying API request fails (e.g. network issue) or returns an error status (HTTP 4xx or 5xx), the SDK will throw an `AINewsError` (a custom error class). You should catch this error to handle issues such as invalid API keys, insufficient credits, or rate-limit violations.

Here's how you can implement error handling with `AINewsError`:

```
import { AINews, Errors } from '@chaingpt/ainews';

const ainews = new AINews({ apiKey: 'YOUR_API_KEY' });

async function main() {
  try {
    const response = await ainews.getNews({ /* your params */ });
    console.log(response.data);
  } catch (error) {
    if (error instanceof Errors.AINewsError) {
      // AINewsError provides a human-readable message
      console.error("AINewsError:", error.message);
    } else {
      // Other unexpected errors
      console.error("Unexpected Error:", error);
    }
  }
}

main();
```

In the example above, we import the `Errors` from the SDK to access `Errors.AINewsError`. We then wrap the `getNews` call in a try/catch. If an `AINewsError` is caught, we log the error message (which typically describes the issue). Common reasons for errors include:

* **Authentication issues:** e.g. invalid or missing API key (would result in a 401 Unauthorized).
* **Insufficient credits:** if your account has no credits, the API may refuse the request.
* **Rate limit exceeded:** if you make too many requests too quickly (see rate limits below, e.g. more than 200 per minute), the API might return a 429 Too Many Requests.
* **Server errors:** a 500 Internal Server Error or other server-side issue.

By checking `instanceof Errors.AINewsError`, you ensure you're only catching errors thrown by the SDK. You can then use `error.message` or other properties (if provided) to inform the user or take remedial action. Always ensure to handle errors to avoid unhandled promise rejections in your application.

***

### Supported Environments

The ChainGPT AI News SDK is supported in **Node.js** environments. You can use it in server-side applications, Node.js scripts, or backend services. It is written in TypeScript and transpiled to JavaScript for compatibility. Both CommonJS (`require`) and ESM (`import`) usage are supported.

At this time, the SDK is not intended for direct use in web browsers, mainly because it requires a secret API key (which should not be exposed publicly) and makes network calls to the ChainGPT API. Always keep your API key on the server side or a secure environment.

**Platform Requirements:** Any modern Node.js version (14.x, 16.x, 18.x, or later) should work. Ensure that `node` and `npm/yarn` are properly installed.

***

### Rate Limits & Credits

When using the AI News SDK, be mindful of rate limits and credit consumption:

* **Rate Limits:** The ChainGPT API allows up to **200 requests per minute** for the AI News endpoint. This means your application should not exceed roughly 3-4 requests per second on average. If you exceed this rate, you will likely receive a rate-limit error (`AINewsError` with a message indicating too many requests or an HTTP 429 status). It's good practice to implement throttling or request queuing if you plan to make a high volume of calls.
* **Credit Consumption:** Every call to `ainews.getNews()` deducts credits from your ChainGPT account. By default, **1 credit** is consumed per request (which returns up to 10 news articles). If you increase the `limit` to fetch more than 10 articles in one call, the credit cost will increase proportionally. Roughly, **1 credit is used per 10 results**. For example:
  * Requesting 10 or fewer articles (the default) costs 1 credit.
  * Requesting up to 20 articles (`limit: 20`) costs 2 credits.
  * Requesting 25 articles would cost 3 credits (rounded up, since 25/10 = 2.5). The SDK itself does not enforce or warn about credit usage, but the API will deduct the appropriate amount. Ensure your account has enough credits to cover the calls your application is making. You can check your remaining credits on the ChainGPT web app dashboard.

Keep these limits in mind to avoid unexpected failures. If you hit the rate limit frequently, consider caching results or reducing the frequency of calls. If you run low on credits, top up via the ChainGPT platform to maintain service continuity.

***

### Reference: Category, Sub-Category, and Token IDs

When using filtering options, you will need to know the IDs for categories, sub-categories, and tokens. Below are tables listing the available categories, sub-categories (blockchains), and tokens along with their corresponding IDs. Use these IDs in the `categoryId`, `subCategoryId`, and `tokenId` parameters to filter news.

#### Category IDs

<table><thead><tr><th width="252.515625">Category</th><th width="102.48828125">ID</th></tr></thead><tbody><tr><td>Blockchain Gaming</td><td>2</td></tr><tr><td>DAO</td><td>3</td></tr><tr><td>DApps</td><td>4</td></tr><tr><td>DeFi</td><td>5</td></tr><tr><td>Lending</td><td>6</td></tr><tr><td>Metaverse</td><td>7</td></tr><tr><td>NFT</td><td>8</td></tr><tr><td>Stablecoins</td><td>9</td></tr><tr><td>Cryptocurrency</td><td>64</td></tr><tr><td>Decentralized</td><td>65</td></tr><tr><td>Smart Contracts</td><td>66</td></tr><tr><td>Distributed Ledger</td><td>67</td></tr><tr><td>Cryptography</td><td>68</td></tr><tr><td>Digital Assets</td><td>69</td></tr><tr><td>Tokenization</td><td>70</td></tr><tr><td>Consensus Mechanisms</td><td>71</td></tr><tr><td>ICO (Initial Coin Offering)</td><td>72</td></tr><tr><td>Crypto Wallets</td><td>73</td></tr><tr><td>Web3.0</td><td>74</td></tr><tr><td>Interoperability</td><td>75</td></tr><tr><td>Mining</td><td>76</td></tr><tr><td>Cross-Chain Transactions</td><td>77</td></tr><tr><td>Exchange</td><td>78</td></tr></tbody></table>

These are the broad news categories you can filter by. For example, use `categoryId: [8]` for NFT-related news, or `categoryId: [5]` for DeFi news. You can combine multiple category IDs in an array if needed.

#### Sub-Category IDs

The following sub-category IDs correspond to specific blockchain ecosystems or networks:

<table><thead><tr><th width="131.359375">Sub-Category</th><th width="76.51171875">ID</th><th width="123.4921875">Sub-Category</th><th width="70.87890625">ID</th></tr></thead><tbody><tr><td>Bitcoin</td><td>11</td><td>Algorand</td><td>27</td></tr><tr><td>BNB Chain</td><td>12</td><td>Arbitrum</td><td>28</td></tr><tr><td>Celo</td><td>13</td><td>Astar</td><td>29</td></tr><tr><td>Cosmos</td><td>14</td><td>Aurora</td><td>30</td></tr><tr><td>Ethereum</td><td>15</td><td>Avalanche</td><td>31</td></tr><tr><td>Fantom</td><td>16</td><td>Base</td><td>32</td></tr><tr><td>Flow</td><td>17</td><td>Binance Smart Chain</td><td>33</td></tr><tr><td>Litecoin</td><td>18</td><td>Cardano</td><td>34</td></tr><tr><td>Monero</td><td>19</td><td>Cronos</td><td>36</td></tr><tr><td>Polygon</td><td>20</td><td>Moonbeam</td><td>37</td></tr><tr><td>XRP Ledger</td><td>21</td><td><strong>DEP</strong> (Deeper Protocol)</td><td>38</td></tr><tr><td>Solana</td><td>22</td><td>Harmony</td><td>41</td></tr><tr><td>TRON</td><td>23</td><td>Oasis</td><td>42</td></tr><tr><td>Terra</td><td>24</td><td>Oasis Sapphire</td><td>43</td></tr><tr><td>Tezos</td><td>25</td><td>Ontology</td><td>44</td></tr><tr><td>WAX</td><td>26</td><td>Optimism</td><td>45</td></tr><tr><td><em>Other</em> (Miscellaneous)</td><td>46</td><td>PlatON</td><td>47</td></tr><tr><td>Steem</td><td>56</td><td>Rangers</td><td>49</td></tr><tr><td>SX Network</td><td>57</td><td>Ronin</td><td>50</td></tr><tr><td>Telos</td><td>58</td><td>Shiden</td><td>51</td></tr><tr><td>Telos EVM</td><td>59</td><td>SKALE</td><td>52</td></tr><tr><td>Theta</td><td>61</td><td>Stacks</td><td>54</td></tr><tr><td>ThunderCore</td><td>62</td><td>Stargaze</td><td>55</td></tr></tbody></table>

_(Entries with blank "Sub-Category" in the table continue the list in the next column.)_

These IDs represent specific blockchain networks or ecosystems that a news article can be associated with. For example:

* Use `subCategoryId: [15]` for **Ethereum**-related news.
* Use `subCategoryId: [11]` for **Bitcoin** news.
* Use `subCategoryId: [46]` for articles that are categorized as "Other" (not specific to a major chain).

You can provide multiple sub-category IDs if an article might fall under multiple chains, but typically one sub-category is used per article. If you are unsure which sub-category an article might be in, you can omit this filter.

#### Token IDs

The following token IDs correspond to specific cryptocurrencies or tokens:

<table><thead><tr><th width="188.81640625">Token (Symbol)</th><th width="78.80859375">ID</th><th width="223.28515625">Token (Symbol)</th><th width="107.59375">ID</th></tr></thead><tbody><tr><td>Bitcoin (BTC)</td><td>79</td><td>Polygon (MATIC)</td><td>91</td></tr><tr><td>Ethereum (ETH)</td><td>80</td><td>Polkadot (DOT)</td><td>92</td></tr><tr><td>Tether USD₮ (USDT)</td><td>81</td><td>Litecoin (LTC)</td><td>93</td></tr><tr><td>BNB (BNB)</td><td>82</td><td>Wrapped Bitcoin (WBTC)</td><td>94</td></tr><tr><td>XRP (XRP)</td><td>83</td><td>Bitcoin Cash (BCH)</td><td>95</td></tr><tr><td>USD Coin (USDC)</td><td>84</td><td>Chainlink (LINK)</td><td>96</td></tr><tr><td>Solana (SOL)</td><td>85</td><td>Shiba Inu (SHIB)</td><td>97</td></tr><tr><td>Cardano (ADA)</td><td>86</td><td>UNUS SED LEO (LEO)</td><td>98</td></tr><tr><td>Dogecoin (DOGE)</td><td>87</td><td>TrueUSD (TUSD)</td><td>99</td></tr><tr><td>TRON (TRX)</td><td>88</td><td>Avalanche (AVAX)</td><td>100</td></tr><tr><td>Toncoin (TON)</td><td>89</td><td>Stellar (XLM)</td><td>101</td></tr><tr><td>Dai (DAI)</td><td>90</td><td>Monero (XMR)</td><td>102</td></tr><tr><td>Uniswap (UNI)</td><td>105</td><td>OKB (OKB)</td><td>103</td></tr><tr><td>Ethereum Classic (ETC)</td><td>106</td><td>Cosmos (ATOM)</td><td>104</td></tr><tr><td>BUSD (BUSD)</td><td>107</td><td>Hedera (HBAR)</td><td>108</td></tr></tbody></table>

Use these token IDs to filter news by specific cryptocurrency. For example, `tokenId: [79]` will retrieve news about **Bitcoin**, and `tokenId: [84]` for **USDC**. You can combine multiple token IDs in one request if you want news related to several different tokens.

**Note:** Many news articles may not be tied to a specific token (especially general news or broad topics). The token filter is most useful if you're interested in news specifically about a particular coin or project.

***

### Additional Resources

* **ChainGPT AI News SDK NPM Package:** You can find the package and additional documentation on [NPM (@chaingpt/ainews)](https://www.npmjs.com/package/@chaingpt/ainews). The README contains basic usage info and is kept up-to-date with releases.
* **ChainGPT Web App & API Dashboard:** Manage your API keys and credits on the [ChainGPT Web App](https://app.chaingpt.org/). This is also where you can monitor usage and top up credits for the AI News API.
* **ChainGPT Developer Documentation:** For more information on ChainGPT’s other APIs and SDKs, visit the ChainGPT Docs site. You can find QuickStart guides, API references, and SDK references for other ChainGPT tools.
* **Support:** If you need help or have questions, please visit the ChainGPT Help Center or reach out via the official ChainGPT community channels (such as Telegram or Discord linked on the ChainGPT website). The support team and community can assist with integration issues or answer any questions about the AI News service.

By leveraging the ChainGPT AI News Generator SDK, developers can effortlessly integrate AI-curated news feeds into their applications, keeping users informed with real-time, relevant information. We hope this reference helps you get started quickly and build something amazing with AI-powered news. Happy coding!
