# API Reference

## AI News Generator API Reference

### Introduction

ChainGPT’s **AI News Generator API** is a RESTful service that allows developers to integrate AI-generated crypto news into their applications, platforms, or services. It is designed for software engineers and system integrators who need real-time blockchain and crypto news updates, enabling them to build intelligent, up-to-date interfaces with ease. This documentation focuses on the API endpoints (not the SDK or RSS feeds) and covers usage, parameters, and response formats to streamline development.

Before using the API, ensure you have the following prerequisites:

* A development environment (e.g. Node.js, an IDE)
* A ChainGPT API key (obtained from the ChainGPT web app)
* Sufficient API credits in your ChainGPT account (credits are required to make API calls)

***

### Authentication

All requests to the AI News API must be authenticated with an API key. You can generate an API key by logging into the ChainGPT web application and navigating to the **API Dashboard**. Use the "Create Secret Key" feature to create a new key, and copy it for use in your application. Keep your API key secure – do not hard-code it in public code repositories. It’s recommended to store it in an environment variable or a secret management service to avoid exposure.

**Using the API Key:** Include the API key in your request headers as a Bearer token. For example:

```
Authorization: Bearer <YOUR_API_KEY>
```

This header must be sent with each API request. The API expects requests and responses in JSON format, so ensure you set the `Content-Type: application/json` header in requests where you send a body (for GET requests, this is not required but including it does no harm).

***

### Axios Configuration

If you are using Axios (a popular HTTP client for Node.js/JavaScript) to interact with the API, you can set up a reusable Axios instance with the base URL and authorization header configured. This will simplify making multiple requests. For example:

```
import axios from 'axios';

const instance = axios.create({
  baseURL: 'https://api.chaingpt.org',
  timeout: 60000,  // 60 seconds timeout
  headers: {
    Authorization: `Bearer ${process.env.CHAINGPT_API_KEY}`, 
    'Content-Type': 'application/json'
  }
});
```

In the above configuration, replace `process.env.CHAINGPT_API_KEY` with your API key (or an environment variable containing it). The base URL for all AI News API endpoints is `https://api.chaingpt.org`, and we include the `Authorization` header by default on all requests. Using an Axios instance is optional, but it helps by not repeating the base URL and headers for each call.

***

### Available Endpoint(s)

The AI News Generator API currently provides one endpoint:

#### `GET /news`

Retrieves AI-generated news articles. Without any query parameters, this endpoint returns the latest AI news across all categories. Developers can optionally filter the news by various criteria to tailor the results. The supported filters (as query parameters) include:

* **Category** – filter news by high-level topic/category (e.g. NFT, DeFi, Blockchain Gaming, etc.)
* **Sub-Category** – filter news by blockchain ecosystem or network (e.g. Bitcoin, Ethereum, Solana, etc.)
* **Search Query** – search terms to match in the news title or description
* **Token** – filter news related to specific cryptocurrencies or tokens (e.g. Bitcoin, BNB, Ethereum)
* **Date** – fetch news published after a certain date
* **Sorting** – specify sorting by publication date (newest first)

This endpoint is designed to be flexible. You can fetch all recent news or narrow down results using any combination of the above filters. All filters are provided as **query parameters** in the GET request. The next section details each available query parameter.

***

### Parameters

All query parameters for the `GET /news` endpoint are **optional**. If no parameters are specified, the API will return a default set of recent news (by default, the 10 latest articles). You can use one or many parameters to filter the results. Below is a list of all supported query parameters:

<table><thead><tr><th width="151.4609375">Parameter</th><th width="114.5859375">Type</th><th>Description</th></tr></thead><tbody><tr><td><code>categoryId</code></td><td>integer or array of integers</td><td><strong>Optional.</strong> Filter news by one or more <strong>category</strong> IDs. Only news belonging to the specified category(s) will be returned. For example, <code>categoryId=5</code> would retrieve news in the <em>DeFi</em> category. You can specify multiple categories by providing an array of IDs (e.g. <code>categoryId=[5,8]</code> to get news tagged as DeFi or NFT). See the <strong>Category IDs Reference</strong> below for the list of categories and their IDs.</td></tr><tr><td><code>subCategoryId</code></td><td>integer or array of integers</td><td><strong>Optional.</strong> Filter news by one or more <strong>sub-category</strong> IDs. Sub-categories typically correspond to blockchain ecosystems or networks. For example, <code>subCategoryId=14</code> filters news related to <em>Cosmos</em>. Multiple sub-categories can be specified (e.g. <code>subCategoryId=[14,22]</code> for Cosmos or Solana news). See <strong>Sub-Category IDs Reference</strong> below for all sub-category IDs.</td></tr><tr><td><code>tokenId</code></td><td>integer or array of integers</td><td><strong>Optional.</strong> Filter news by specific <strong>token</strong> IDs. This narrows results to news articles that are related to particular cryptocurrencies/tokens (for example, Bitcoin, Ethereum, etc.). You can provide multiple tokens (e.g. <code>tokenId=[79,80]</code> for Bitcoin or Ethereum). Refer to <strong>Token IDs Reference</strong> below for supported token IDs.</td></tr><tr><td><code>searchQuery</code></td><td>string</td><td><strong>Optional.</strong> Keyword search term to filter news by title or description. The API will return only news articles whose title or description contain the provided query string (case-insensitive). For example, <code>searchQuery=upgrade</code> would match news with "upgrade" in the title or body.</td></tr><tr><td><code>fetchAfter</code></td><td>date string</td><td><strong>Optional.</strong> Only return news published <strong>after</strong> the specified date. The date should be in a recognizable format (e.g. <code>YYYY-MM-DD</code> or ISO 8601). For example, <code>fetchAfter=2023-12-20</code> will fetch news published on or after December 20, 2023.</td></tr><tr><td><code>limit</code></td><td>integer</td><td><strong>Optional.</strong> The maximum number of news articles to return. Defaults to <strong>10</strong> if not specified. You can use this to limit the size of the result set (e.g. <code>limit=5</code> for five articles). Note that increasing this beyond 10 will increase the credit cost (see Pricing/Credits).</td></tr><tr><td><code>offset</code></td><td>integer</td><td><strong>Optional.</strong> The number of items to skip (offset) before starting to return results. This is used for pagination along with <code>limit</code>. For example, <code>offset=10&#x26;limit=10</code> would skip the first 10 results and return the next 10.</td></tr><tr><td><code>sortBy</code></td><td>string</td><td><strong>Optional.</strong> Field by which to sort results. Currently, the only supported value is <code>"createdAt"</code>, which sorts news by their creation/publish date. By default, news is sorted by newest first (most recent date).</td></tr></tbody></table>

{% hint style="info" %}
**Notes:** All filters can be combined in a single request. For instance, you could request news in the NFT category about the Ethereum network by setting both `categoryId` and `subCategoryId` accordingly. If multiple filter parameters are provided, the news must satisfy all of them (logical AND). If an array of values is provided for a parameter, the filter will match any of those values (logical OR for the values of that field). For example, `categoryId=[5,8]` returns news that are in _either_ DeFi _or_ NFT categories.
{% endhint %}

***

### Responses

A successful response from the `GET /news` endpoint returns a JSON object containing the news results. The response includes an array of news articles and possibly some pagination info. Each news article in the array has the following structure (fields):

* **id** (integer): A unique identifier for the news article.
* **title** (string): The headline or title of the news article.
* **description** (string): A brief summary or excerpt of the news content. This provides a quick overview of the news.
* **categoryId** (integer): The category ID associated with this news article (e.g., 5 for DeFi, 8 for NFT, etc.). This corresponds to one of the categories listed in the Category IDs reference table below. Use this to determine the high-level topic of the news.
* **subCategoryId** (integer): The sub-category ID for the news, typically indicating a blockchain or ecosystem (e.g., 15 for Ethereum, 22 for Solana). This corresponds to one of the sub-categories listed in the reference table. If a news article is relevant to a specific blockchain or protocol, it will have the respective subCategoryId.
* **tokenId** (integer or null): The token ID if the news is specifically about a certain cryptocurrency/token (e.g., 79 for Bitcoin). This corresponds to one of the token IDs in the reference table. If the news is of general interest and not focused on a particular token, this field may be null or omitted.
* **createdAt** (string timestamp): The date and time when the news article was created/published. This timestamp is typically in ISO 8601 format (e.g., `"2024-01-15T08:30:00Z"`). You can use this to display the publication date or to sort articles by recency. (By default, results are already sorted by this field in descending order.)

The response may also include some metadata for pagination if you requested a non-default limit/offset. For example, it could include fields like **total** (total number of news articles available) or echo back the **limit** and **offset** you used. (Refer to the API's latest specification for exact response format; the core part is the array of news items described above.)

**Example Response:**

```
{
  "data": [
    {
      "id": 10101,
      "title": "DeFi Platform Launches New Yield Farming Program",
      "description": "A major DeFi platform has introduced a new yield farming initiative aimed at ...",
      "categoryId": 5,
      "subCategoryId": 39,
      "tokenId": null,
      "createdAt": "2024-01-10T12:00:00Z"
    },
    {
      "id": 10102,
      "title": "Ethereum Network Upgrade Scheduled for Next Month",
      "description": "The Ethereum foundation announced a network upgrade (hard fork) planned for ...",
      "categoryId": 66,
      "subCategoryId": 15,
      "tokenId": 80,
      "createdAt": "2024-01-09T09:30:00Z"
    }
    // ... more articles ...
  ],
  "limit": 10,
  "offset": 0,
  "total": 42
}
```

In this example, the response `data` is an array of news article objects. Each object contains the fields described above. The first article has `categoryId: 5` which (from the Category reference) means _DeFi_, and `subCategoryId: 39` which corresponds to _Ethereum_ (as a blockchain ecosystem). The second article has `categoryId: 66` (_Smart Contracts_ category) and `subCategoryId: 15` (_Ethereum_ again, but using the other ID in this case; see note on sub-category IDs below). The `tokenId: 80` in the second article indicates it’s specifically about _Ethereum (ETH)_ as a token. The `limit`, `offset`, and `total` in this example are illustrative of possible pagination fields.

***

### Usage Examples

Below are examples of how to use the `GET /news` endpoint with various tools.

#### Using cURL

You can test the API quickly using the command-line tool **cURL**. Make sure to replace `YOUR_API_KEY` with your actual API key. For example, to fetch the latest 5 news articles in the _NFT_ category, you could use:

```
curl -X GET "https://api.chaingpt.org/news?categoryId=8&limit=5" \
     -H "Authorization: Bearer YOUR_API_KEY"
```

This will make a GET request to the `/news` endpoint, filtering results to category ID 8 (which is **NFT**). The `Authorization` header is included with the Bearer token. The response will be a JSON with up to 5 NFT-related news items.

#### Using Axios (JavaScript/Node.js)

In a Node.js environment, using Axios, you can call the API as follows. This example assumes you've configured the Axios instance as shown in the Axios Configuration section above:

```
// Fetch latest news about Ethereum in the NFT category (for example)
async function fetchNews() {
  try {
    const response = await instance.get('/news', {
      params: {
        categoryId: [8],        // NFT category
        subCategoryId: [15],    // Ethereum blockchain
        searchQuery: 'upgrade', // keyword to search in title/description
        limit: 5,               // limit to 5 results
        sortBy: 'createdAt'     // sort by newest
      }
    });
    console.log(response.data);
  } catch (error) {
    console.error(error);
  }
}

fetchNews();
```

In this Axios example, we use an `instance.get` call to `'/news'` with a params object. We filter for news in category ID 8 (NFT) that are related to sub-category ID 15 (Ethereum network), and contain the word "upgrade". We limit to 5 articles and explicitly sort by creation date (newest first). The API key is automatically attached via the Axios instance configuration. The `response.data` will contain the JSON response (as described in the Responses section above), which we then log to the console.

#### Handling Errors

If the API key is missing or invalid, or if your credits are insufficient, the API will return an error (HTTP 4xx status). Make sure to catch errors in your code. For instance, in the Axios example above, if an error occurs, it will be caught in the `catch` block and logged. The error message can help identify issues like authentication failures or invalid parameters. Always ensure your API key is correct and that you have enough credits for the call.

***

### Credits and Pricing

ChainGPT’s AI News API uses a **credit-based** pricing system. You must have credits available in your ChainGPT account to make API calls. Each request to the `/news` endpoint will deduct credits from your balance. By default, **1 credit** is deducted per API call that fetches up to 10 news articles. This means if you use the default `limit` (which is 10), the call will cost 1 credit.

If you increase the `limit` to retrieve more than 10 articles in one call, the credit cost scales proportionally. Specifically, **1 credit is charged per 10 results**. For example:

* `limit=10` (or not specifying a limit, which defaults to 10) – costs 1 credit.
* `limit=20` – costs 2 credits (since 20 results are fetched, equivalent to 2 batches of 10).
* `limit=25` – costs 3 credits (the first 20 results cost 2 credits, and the additional 5 results consume another credit, as any fraction beyond each 10 results counts as a new block of 10).

The credit deduction happens per request, regardless of filters. If you make multiple requests, each will consume credits according to the number of news items requested. Ensure you have a sufficient credit balance before making calls (you can obtain and top-up credits via the ChainGPT web app).

**Note:** Generating an API key requires that you have credits in your account in the first place. If you need to increase your usage beyond the included or purchased credits, consider reviewing ChainGPT’s pricing plans or contacting the ChainGPT team for higher-volume access. Always monitor your credit usage to avoid interruptions.

***

### Category IDs Reference

When filtering news by category (using `categoryId`), use the following IDs which correspond to various high-level news topics/categories:

<table><thead><tr><th width="238.3046875">Category Name</th><th width="125.046875">ID</th></tr></thead><tbody><tr><td><strong>Category Name</strong></td><td><strong>ID</strong></td></tr><tr><td>DAO</td><td>3</td></tr><tr><td>DApps</td><td>4</td></tr><tr><td>DeFi</td><td>5</td></tr><tr><td>Lending</td><td>6</td></tr><tr><td>Metaverse</td><td>7</td></tr><tr><td>NFT</td><td>8</td></tr><tr><td>Stablecoins</td><td>9</td></tr><tr><td>Cryptocurrency</td><td>64</td></tr><tr><td>Decentralized</td><td>65</td></tr><tr><td>Smart Contracts</td><td>66</td></tr><tr><td>Distributed Ledger</td><td>67</td></tr><tr><td>Cryptography</td><td>68</td></tr><tr><td>Digital Assets</td><td>69</td></tr><tr><td>Tokenization</td><td>70</td></tr><tr><td>Consensus Mechanisms</td><td>71</td></tr><tr><td>ICO (Initial Coin Offering)</td><td>72</td></tr><tr><td>Crypto Wallets</td><td>73</td></tr><tr><td>Web3.0</td><td>74</td></tr><tr><td>Interoperability</td><td>75</td></tr><tr><td>Mining</td><td>76</td></tr><tr><td>Cross-Chain Transactions</td><td>77</td></tr><tr><td>Exchange</td><td>78</td></tr></tbody></table>

Use these IDs with the `categoryId` parameter to filter news. For example, `categoryId=8` will retrieve news categorized as **NFT**. If you want multiple categories, you can specify an array of IDs, e.g., `categoryId=[5,8]` for news that are in _DeFi_ or _NFT_ categories.

***

### Sub-Category IDs Reference

Sub-categories usually represent specific blockchain networks or ecosystems. The table below lists all sub-category IDs that can be used with the `subCategoryId` parameter:

<table><thead><tr><th width="257.41015625">Sub-Category (Blockchain)</th><th width="102.18359375">ID</th></tr></thead><tbody><tr><td>Bitcoin</td><td>11</td></tr><tr><td>BNB Chain</td><td>12</td></tr><tr><td>Celo</td><td>13</td></tr><tr><td>Cosmos</td><td>14</td></tr><tr><td>Ethereum</td><td>15</td></tr><tr><td>Fantom</td><td>16</td></tr><tr><td>Flow</td><td>17</td></tr><tr><td>Litecoin</td><td>18</td></tr><tr><td>Monero</td><td>19</td></tr><tr><td>Polygon</td><td>20</td></tr><tr><td>XRP Ledger</td><td>21</td></tr><tr><td>Solana</td><td>22</td></tr><tr><td>Tron</td><td>23</td></tr><tr><td>Terra</td><td>24</td></tr><tr><td>Tezos</td><td>25</td></tr><tr><td>WAX</td><td>26</td></tr><tr><td>algorand</td><td>27</td></tr><tr><td>arbitrum</td><td>28</td></tr><tr><td>astar</td><td>29</td></tr><tr><td>aurora</td><td>30</td></tr><tr><td>avalanche</td><td>31</td></tr><tr><td>base</td><td>32</td></tr><tr><td>binance-smart-chain</td><td>33</td></tr><tr><td>cardano</td><td>34</td></tr><tr><td>celo</td><td>35</td></tr><tr><td>cronos</td><td>36</td></tr><tr><td>moonbeam</td><td>37</td></tr><tr><td>dep</td><td>38</td></tr><tr><td>ethereum</td><td>39</td></tr><tr><td>fantom</td><td>40</td></tr><tr><td>harmony</td><td>41</td></tr><tr><td>oasis</td><td>42</td></tr><tr><td>oasis-sapphire</td><td>43</td></tr><tr><td>ontology</td><td>44</td></tr><tr><td>optimism</td><td>45</td></tr><tr><td>other</td><td>46</td></tr><tr><td>platon</td><td>47</td></tr><tr><td>polygon</td><td>48</td></tr><tr><td>rangers</td><td>49</td></tr><tr><td>ronin</td><td>50</td></tr><tr><td>shiden</td><td>51</td></tr><tr><td>skale</td><td>52</td></tr><tr><td>solana</td><td>53</td></tr><tr><td>stacks</td><td>54</td></tr><tr><td>stargaze</td><td>55</td></tr><tr><td>steem</td><td>56</td></tr><tr><td>sxnetwork</td><td>57</td></tr><tr><td>telos</td><td>58</td></tr><tr><td>telosevm</td><td>59</td></tr><tr><td>tezos</td><td>60</td></tr><tr><td>theta</td><td>61</td></tr><tr><td>thundercore</td><td>62</td></tr></tbody></table>

Use these IDs with `subCategoryId` to filter news for specific blockchains. For example, `subCategoryId=11` will filter news related to **Bitcoin**. You can specify multiple sub-categories; for instance, `subCategoryId=[11,22]` will get news about either _Bitcoin_ or _Solana_.

**Note:** Some blockchain names appear twice with different IDs (for example, _Ethereum_ appears as ID 15 and also as ID 39 in this list, likewise _Celo_ as 13 and 35, etc.). This is due to how the categories are defined in the system. In practice, you should use the ID that appears in the news data you receive. The higher-numbered duplicate entries exist for legacy or internal reasons. For instance, you may see news items tagged with subCategoryId 15 for Ethereum in some cases and 39 in others. Both refer to Ethereum-related news. Similarly, BNB Chain (ID 12) and Binance Smart Chain (ID 33) refer to the same network (Binance Chain), with ID 33 being the older nomenclature. It’s recommended to filter using both IDs if in doubt, or prefer the primary name’s ID.

***

### Token IDs Reference

When filtering by specific tokens or cryptocurrencies using the `tokenId` parameter, use the following token IDs. These represent popular cryptocurrencies by name and symbol:

<table><thead><tr><th width="195.02734375">Token Name</th><th width="126.99609375">Symbol</th><th width="89.51171875">ID</th></tr></thead><tbody><tr><td>Bitcoin</td><td>BTC</td><td>79</td></tr><tr><td>Ethereum</td><td>ETH</td><td>80</td></tr><tr><td>Tether USD₮</td><td>USDT</td><td>81</td></tr><tr><td>BNB (Binance Coin)</td><td>BNB</td><td>82</td></tr><tr><td>XRP</td><td>XRP</td><td>83</td></tr><tr><td>USD Coin</td><td>USDC</td><td>84</td></tr><tr><td>Solana</td><td>SOL</td><td>85</td></tr><tr><td>Cardano</td><td>ADA</td><td>86</td></tr><tr><td>Dogecoin</td><td>DOGE</td><td>87</td></tr><tr><td>TRON</td><td>TRX</td><td>88</td></tr><tr><td>Toncoin</td><td>TON</td><td>89</td></tr><tr><td>Dai</td><td>DAI</td><td>90</td></tr><tr><td>Polygon</td><td>MATIC</td><td>91</td></tr><tr><td>Polkadot</td><td>DOT</td><td>92</td></tr><tr><td>Litecoin</td><td>LTC</td><td>93</td></tr><tr><td>Wrapped Bitcoin</td><td>WBTC</td><td>94</td></tr><tr><td>Bitcoin Cash</td><td>BCH</td><td>95</td></tr><tr><td>Chainlink</td><td>LINK</td><td>96</td></tr><tr><td>Shiba Inu</td><td>SHIB</td><td>97</td></tr><tr><td>UNUS SED LEO</td><td>LEO</td><td>98</td></tr><tr><td>TrueUSD</td><td>TUSD</td><td>99</td></tr><tr><td>Avalanche</td><td>AVAX</td><td>100</td></tr><tr><td>Stellar</td><td>XLM</td><td>101</td></tr><tr><td>Monero</td><td>XMR</td><td>102</td></tr><tr><td>OKB</td><td>OKB</td><td>103</td></tr><tr><td>Cosmos</td><td>ATOM</td><td>104</td></tr><tr><td>Uniswap</td><td>UNI</td><td>105</td></tr><tr><td>Ethereum Classic</td><td>ETC</td><td>106</td></tr><tr><td>BUSD (Binance USD)</td><td>BUSD</td><td>107</td></tr><tr><td>Hedera</td><td>HBAR</td><td>108</td></tr></tbody></table>

Use these token IDs with the `tokenId` query parameter to filter news about a specific cryptocurrency. For example, `tokenId=79` will retrieve news related to **Bitcoin (BTC)**. You can specify multiple tokens as well (e.g., `tokenId=[79,80]` for news about Bitcoin or Ethereum).

Each news article that is specifically about a certain token will include a `tokenId` field in its response data. If an article doesn’t have an associated token (i.e., it’s more general news or related to a blockchain rather than a specific coin), the `tokenId` might be null or omitted. The token IDs above cover many of the top crypto assets.

***

By using the above reference tables in conjunction with the query parameters, developers can precisely query the AI News Generator API for the news content that matters to their use case. The API is designed to be flexible and developer-friendly, following patterns similar to other modern APIs. For more information, support, or to manage your API usage and credits, visit the ChainGPT web app or developer documentation pages. Happy building!
