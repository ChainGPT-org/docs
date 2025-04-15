# QuickStart Guide

## QuickStart Guide

Get started quickly with ChainGPT’s Crypto AI News through either the open RSS feeds or the JavaScript SDK. This guide will walk you through both methods.

***

### Option #1: Using the AI News SDK (JavaScript/Node.js)

**Step 1: Install the SDK.** The SDK is available as an npm package. In your Node.js project, install it via:

```
npm install @chaingpt/ainews
# or with Yarn:
yarn add @chaingpt/ainews
```

_(The SDK is a TypeScript/JavaScript library for Node.js applications.)_

**Step 2: Obtain an API Key and Credits.** The AI News API requires an API key for authentication and consumes credits on each request. Log in to the ChainGPT Crypto AI Hub (the web app) and navigate to the **API Dashboard**. Create a new secret key to use with the AI News SDK, and copy it. Make sure your account has enough credits — each news query costs **1 credit** from your balance. You can top-up credits in the web app if needed.

{% hint style="info" %}
**Security Tip:** Treat your API key like a password. Do **not** hard-code it in public code or expose it client-side. Store it in an environment variable or a secure config so it isn’t leaked.
{% endhint %}

**Step 3: Initialize the SDK.** In your Node.js code, import and initialize the SDK with your API key:

```
import { AINews } from '@chaingpt/ainews';

const ainews = new AINews({
  apiKey: 'YOUR_CHAINGPT_API_KEY',
});
```

This creates an `ainews` client instance. You’re now ready to fetch news!

**Step 4: Fetch the Latest News.** The SDK provides a method `getNews()` to retrieve news articles. If called with no filters, it returns the most recent AI-curated news across all categories. For example:

```
async function main() {
  try {
    const response = await ainews.getNews({});
    console.log(response.data.rows);
  } catch (error) {
    console.error("Error fetching news:", error);
  }
}
main();
```

This will print out an array of news items (`response.data.rows`) containing the latest headlines and summaries. Each news item includes details such as title, description (summary), publication date, source, category, etc. (See **SDK Reference** below for a detailed response schema.)

**Step 5: Apply Filters (Optional).** You can narrow down the news query by specifying filters in the `getNews` parameters. For instance, to fetch only DeFi-related news, or only news about Bitcoin, you can pass parameters like `categoryId`, `subCategoryId`, or `tokenId`. Below are a few examples:

*   _Fetch news about a specific token:_ E.g. Ethereum-related news by token –

    ```
    await ainews.getNews({ tokenId: [80] });
    ```

    This returns news where the primary token is Ethereum (ETH). (In this case, `80` is Ethereum’s token ID; see the token ID table in SDK Reference.)
*   _Fetch news by category:_ E.g. DeFi category –

    ```
    await ainews.getNews({ categoryId: [5] });
    ```

    This returns news categorized as **DeFi** (Decentralized Finance). Category IDs for other topics like NFT, Metaverse, etc. are listed in the reference table.
*   _Filter by blockchain ecosystem:_ E.g. Solana ecosystem news –

    ```
    await ainews.getNews({ subCategoryId: [22] });
    ```

    This fetches news related to the **Solana** blockchain (ID 22 in the subcategories list). For example, this may include Solana network updates or projects on Solana.
*   _Search by keyword:_ You can search article titles and descriptions using `searchQuery`. For example, to find news mentioning "SEC regulation":

    ```
    await ainews.getNews({ searchQuery: "SEC" });
    ```

    The response will include any recent news articles that contain "SEC" in the title or summary text.
*   _Date filtering:_ You can retrieve historical news by specifying `fetchAfter` with a date. For instance, to get news published after 1st January 2025:

    ```
    await ainews.getNews({ fetchAfter: new Date("2025-01-01") });
    ```

    This returns all news from 2025 onward. (The dates/times are based on article publish times.)
*   _Pagination:_ The SDK supports pagination via `limit` and `offset`. For example:

    ```
    await ainews.getNews({ limit: 5, offset: 5 });
    ```

    would skip the first 5 results and fetch the next 5 news items (useful for “page 2” of results). By default, if you don’t specify these, a default page size is used (e.g. 10 items).

You can combine multiple filters in one request. For example, you could get **DeFi news related to Ethereum** by specifying both `categoryId: [5]` (DeFi) and `subCategoryId: [15]` (Ethereum) in the same `getNews` call. In general, an article must satisfy _all_ provided filters to be returned (within each filter type, multiple IDs act as an OR). See the **SDK Reference** for all parameters and their usage.

**Step 6: Handle the Response.** The response from `getNews()` includes a data object with an array of news items (as `rows`). You can iterate through these to display titles, descriptions, etc. For example:

```
const news = await ainews.getNews({ categoryId: [5] });
for (let item of news.data.rows) {
  console.log(item.title, "-", item.description);
}
```

This would log each news headline and its summary. Typically you would integrate this into your UI or pass it to your application logic.

**Step 7: Error Handling.** If something goes wrong (e.g. invalid API key or network issue), the SDK will throw an `AINewsError`. You should catch errors around the `getNews` call:

```
import { Errors } from '@chaingpt/ainews';

try {
  const res = await ainews.getNews({ /* params */ });
  // ... use res.data.rows
} catch (err) {
  if (err instanceof Errors.AINewsError) {
    console.error("AINewsError:", err.message);
  } else {
    console.error("Unexpected error:", err);
  }
}
```

In production, make sure to handle errors gracefully – for example, you might retry after a delay or fall back to a default message if the news service is temporarily unreachable.

That’s it! With these steps, you can start integrating AI-generated crypto news into your application. For more detailed options, continue to the **SDK Reference**.

***

### Option #2: Using the Public RSS Feeds

If you prefer a no-code integration or want to consume the news via standard RSS, ChainGPT offers **public RSS feeds** that update continuously with the latest AI-generated news. This is ideal for quickly adding the news feed to websites, blogs, or using RSS automation tools.

**Available RSS Feeds:**

* **General News Feed:** includes all crypto/Web3 news aggregated by the AI (across all categories).
* **Bitcoin News Feed:** only news specifically about Bitcoin (BTC) and its ecosystem.
* **Ethereum News Feed:** only news related to Ethereum (ETH) and its ecosystem.
* **BNB Chain News Feed:** only news for BNB Chain (formerly Binance Smart Chain) and related topics.

Each feed is accessible via a simple URL (HTTP endpoint). For example, a _general feed URL_ might be:

```
https://app.chaingpt.org/news/rss/general
```

And a _Bitcoin-only feed_ might be:

```
https://app.chaingpt.org/news/rss/bitcoin
```

_(Note: These URLs are indicative; use the exact endpoints provided by ChainGPT’s documentation or interface. The feeds are openly accessible without an API key.)_

**To use an RSS feed, simply plug the URL into any RSS-compatible application:**

* **Web Applications (Frontend):** You can fetch and parse the RSS XML in a client-side app or static site. For instance, you could use AJAX to fetch the URL and parse the XML (with an XML parser or converting to JSON). However, many modern frontend frameworks might block cross-origin RSS requests due to CORS. A common pattern is to have your backend proxy the RSS feed, or use a service worker, to get around CORS restrictions. Once fetched, you’ll have an XML document containing `<item>` entries for each news article (title, link, description, date, etc.) which you can format and display in your UI.
* **Node.js or Backend Server:** Using Node, you can consume the RSS easily with request and XML parsing libraries. For example, you might use the `axios` library to GET the feed URL, then an XML parser (like `xml2js` or `feedparser`) to convert it into JSON. This can then be served to your frontend or used however you need. The advantage of doing it server-side is you won’t run into cross-origin issues, and you can cache the results to reduce load.
* **RSS-to-Telegram/Discord Bots:** If you want to broadcast news to chat applications, there are existing tools and libraries that read RSS feeds and post updates to channels. For instance, you could use a service like IFTTT or Zapier to monitor the RSS feed URL and auto-post new items to a Telegram channel or Discord webhook. Alternatively, a custom bot script can periodically fetch the RSS feed (e.g. every few minutes) and send any new article to your chat group. The open nature of the feed makes integration with such automation straightforward.

**Sample RSS Output:** An example snippet of the RSS XML for a news item might look like:

```
<item>
  <title>Bitcoin ETF Approved by Regulators</title>
  <link>https://original-news-site.com/bitcoin-etf-approved</link>
  <description><![CDATA[Regulators have approved a new Bitcoin ETF, marking a significant milestone...]]></description>
  <pubDate>Mon, 14 Apr 2025 18:00:00 GMT</pubDate>
  <category>Cryptocurrency</category>
</item>
```

Each `<item>` corresponds to one AI-curated news article. The **title** is the headline written by the AI (or sourced from the original title if appropriate), the **description** is the AI-generated summary of the article (often contained in a CDATA section due to HTML content), and the **link** typically points to a source or the ChainGPT news page for that item. There’s also a publication date and possibly category tags.

**Polling Frequency:** It’s recommended to poll the RSS feed periodically (but not too frequently). A good practice is to check for updates every 5 to 15 minutes. Polling more often (like every minute) is usually unnecessary and could be considered excessive load on the server, since the AI agent typically adds news in intervals (near real-time, but it depends on when new information is found). By polling every few minutes, you’ll catch new articles shortly after they appear without hammering the server. Many RSS readers also use conditional requests (HTTP `Last-Modified` or `ETag` headers) to avoid downloading the full feed if nothing has changed, which is a good practice if you implement your own fetcher.

***

### RSS vs SDK – Which One is Right for You?

When choosing between RSS feeds and SDK/API integration, your decision should be guided by your specific use case and desired functionality.

* **RSS feeds:** are ideal for quick integration with minimal setup. They require no coding, no authentication, and seamlessly integrate with numerous existing tools and feed readers. They're perfect if you simply need an easy, straightforward news feed.
* **SDK/API integration:** on the other hand, provides enhanced flexibility and customization. If your application demands specific news topics, advanced filtering, search functionality, or integration into an interactive app environment, the SDK/API is your best choice. It delivers data in JSON format and allows precise, programmatic control over news content.

You can also leverage both methods strategically—for instance, implementing SDK/API integration on your backend for dynamic, interactive experiences while simultaneously offering RSS feeds for end-users who prefer traditional feed readers.

#### **Quick Comparison Table:**

| Feature                 | RSS Feed                       | SDK/API                     |
| ----------------------- | ------------------------------ | --------------------------- |
| **Ease of Use**         | ✅ No coding needed             | ⚙️ Requires coding          |
| **Customization**       | ❌ Limited                      | ✅ Highly customizable       |
| **Authentication**      | ❌ Not required                 | ✅ Typically required        |
| **Integration Options** | ✅ Broad support (feed readers) | ✅ Flexible, app integration |
| **Data Format**         | ✅ XML                          | ✅ JSON                      |
| **Filtering & Search**  | ❌ Not supported                | ✅ Supported                 |
| **Dynamic Queries**     | ❌ Not supported                | ✅ Fully supported           |

{% hint style="info" %}
**Tip**: Choose RSS for simplicity, SDK/API for power, or combine them for the best of both worlds.
{% endhint %}
