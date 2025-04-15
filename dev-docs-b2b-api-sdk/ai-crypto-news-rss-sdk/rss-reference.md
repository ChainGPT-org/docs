# RSS Reference

## RSS Reference

For convenience and open integration, ChainGPT provides a set of publicly accessible RSS feeds that mirror the AI-generated news content. This section lists the available RSS endpoints and provides guidance on how to use them in various scenarios.

***

### Available RSS Feeds and Endpoints

Currently, the following RSS feeds are available (in XML RSS 2.0 format):

* **General News Feed:** All AI-generated news articles, regardless of category.\
  **URL:** `https://app.chaingpt.org/news/rss/general` (public, no auth required)
* **Bitcoin News Feed:** AI-generated news specifically about Bitcoin.\
  **URL:** `https://app.chaingpt.org/news/rss/bitcoin`
* **Ethereum News Feed:** AI-generated news specifically about Ethereum.\
  **URL:** `https://app.chaingpt.org/news/rss/ethereum`
* **BNB Chain News Feed:** AI-generated news about BNB Chain (Binance Chain/BSC).\
  **URL:** `https://app.chaingpt.org/news/rss/bnb`

{% hint style="info" %}
_If these URLs change, refer to the official ChainGPT documentation or interface for updated links. Sometimes feeds might also be available under an **`api.chaingpt.org`** domain or similar._
{% endhint %}

Each feed URL will return an XML document that conforms to the RSS 2.0 standard. You can access these URLs via a web browser, curl, or any RSS reader to see the content.

***

### Consuming RSS Feeds in Different Environments

**Web Browser / RSS Readers:** The simplest way to use the feed is to plug the URL into any RSS reader application or even your browser (modern browsers might require an RSS extension). The reader will display a list of recent news items with their titles and summaries. This is more for end-users and manual checking.

**Front-end Integration (JavaScript):** If you want to integrate the feed into a website, you have a couple of options:

* Use client-side JavaScript to fetch the RSS URL (via `fetch` or AJAX) and then parse the XML. Keep in mind that the RSS feed might not set CORS headers to allow client-side JS on a different domain to fetch it. If you encounter CORS issues, you might need to use a proxy or your own server to fetch the data.
* Use a JavaScript RSS parsing library. There are libraries that can take an RSS URL and handle the fetching and parsing (for example, the npm package `rss-parser` can be used in Node or possibly bundled for frontend).
* Once parsed, you’ll have a JSON object containing items (each with title, link, etc.). You can then render this in your app’s UI (perhaps as a news ticker or list).

**Node.js / Backend:** On a Node server, consuming RSS is straightforward. You can use packages like `feedparser` or `rss-parser`. For example:

```
const Parser = require('rss-parser');
const parser = new Parser();
const feed = await parser.parseURL('https://app.chaingpt.org/news/rss/bitcoin');
feed.items.forEach(item => {
  console.log(item.title, item.link);
});
```

This will give you the parsed feed for Bitcoin news. You can then serve this data through your own API to clients or use it in server-side rendered pages. Doing this on the backend also allows you to cache responses and avoid hitting the ChainGPT feed too often.

**Telegram/Discord Bots via RSS:** If you want automated posting of news to messaging platforms:

* **Zapier/IFTTT:** These platforms have RSS-to-others integrations. For example, Zapier has a “New RSS item -> Send Telegram message” zap. You could plug the ChainGPT RSS URL into such a workflow. Each new item would trigger and be sent to your specified Telegram chat or Discord channel.
* **Custom Bot:** Write a small script that periodically checks the RSS feed (stores the last seen GUID or publish time to know what’s new) and posts new items to a chat. For Telegram, you could use the Telegram Bot API to send messages. For Discord, you could use webhooks or a bot account to post messages.

**Mobile Apps:** Some mobile crypto apps might want to show a news feed. Many mobile dev environments (Swift, Kotlin, React Native, Flutter) have libraries for parsing RSS or can use HTTP+XML parsing directly. The approach is similar: fetch the URL and parse XML into native objects, then display in a list. Because the RSS is open, no special auth is needed, making it easy to integrate on mobile.

***

### Example RSS Feed Item

Here is a more detailed look at an example item you’d find in the RSS feed (general format):

```
<item>
  <title>Ethereum Upgrade Reduces Gas Fees by 20%</title>
  <link>https://app.chaingpt.org/news/12345</link>
  <guid isPermaLink="false">cgpt-news-12345</guid>
  <pubDate>Mon, 14 Apr 2025 18:30:00 GMT</pubDate>
  <category>Ethereum</category>
  <description><![CDATA[
    <b>[Ethereum]</b> The latest Ethereum network upgrade has successfully reduced average gas fees by ~20%. 
    ChainGPT's AI reports that early data shows improved transaction throughput and lower costs for users. 
    This upgrade, dubbed "Zenith", was activated at block 17000000. Major exchanges briefly paused withdrawals 
    during the update, but have since resumed normal operations.
  ]]></description>
</item>
```

A few notes on this structure:

* `<title>` provides a concise headline.
* `<link>` might either link to the original source or to a ChainGPT-hosted page for the news. In the example, it’s a ChainGPT app link with an internal ID.
* `<guid>` is a unique identifier for the item. The feed uses this to identify items (so readers don’t treat the same news as new if it appears again). It’s not necessarily a URL.
* `<pubDate>` is the publish time in RFC-822 format.
* `<category>` in this context likely denotes the category or subcategory (here “Ethereum” as a subcategory). There might be multiple category tags if applicable.
* `<description>` contains the HTML or text summary. In this case, the summary includes a bold tag with “\[Ethereum]” to denote category, followed by the AI-generated summary of the news. It’s enclosed in CDATA because it contains HTML markup.

When rendering, an RSS reader might strip the HTML or might render it, depending on the client. If you parse manually, you may get the inner text or HTML which you can then display.

***

### Best Practices for RSS Integration

* **Polling:** As mentioned, set a reasonable interval. The ChainGPT AI News feed is updated in real-time as news comes, but news itself doesn’t break every second. Polling every 5-10 minutes is typically sufficient for most use cases. If you need more instantaneous updates, you could poll more frequently (e.g. every minute), but be mindful of server load.
* **Caching:** If you have many users or clients accessing your service that in turn fetches the ChainGPT RSS, consider having your server fetch the RSS and then distribute to clients to reduce redundant traffic. The RSS content likely updates with new items rather than changing existing ones, so caching the last response and only fetching new items (using the Last-Modified header or checking the latest GUID) can be efficient.
* **Choosing RSS vs SDK:** Use RSS when you want a **lightweight, read-only** integration. It’s perfect for embedding news or doing one-way syncs (like into a channel or widget). Use the SDK (or API) when you need **interactive or filtered queries** (for example, searching past news, combining multiple filters, building a custom analytics on top of news data, etc.). In fact, you might use both: RSS for a simple “latest news” widget, and SDK for more complex features in the same product.

ChainGPT's Crypto AI News RSS feeds offer a straightforward and efficient way to incorporate real-time, AI-generated crypto news into your applications. With no authentication required, these RSS feeds make integration quick and simple, allowing you to effortlessly deliver relevant crypto updates to your users. Follow the best practices outlined in this guide to ensure optimal performance and reliability. Happy integrating!
