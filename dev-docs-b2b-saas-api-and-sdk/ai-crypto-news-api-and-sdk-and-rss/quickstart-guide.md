# QuickStart Guide

## AI Crypto News QuickStart Guide

ChainGPT’s **AI News Generator** delivers curated crypto & blockchain‑AI news through a REST API, JavaScript/TypeScript SDK, and public RSS feeds. Integrate once, fetch fresh stories in seconds, and build features like personalized feeds, dashboards, or alerts.

***

### Prerequisites

* ChainGPT account with credits and an API key
* Node ≥ 18 (for SDK examples)
* `curl` or any HTTP client
* Ability to set the environment variable `CHGPT_API_KEY`

***

### Install the SDK

```bash
# npm
npm install --save @chaingpt/ainews

# yarn
yarn add @chaingpt/ainews
```

***

### Authenticate

#### REST

```bash
curl https://api.chaingpt.org/news \
  -H "Authorization: Bearer $CHGPT_API_KEY"
```

#### SDK

```ts
import { AINews } from '@chaingpt/ainews';

const ainews = new AINews({ apiKey: process.env.CHGPT_API_KEY! });
```

The key is charged **1 credit per 10 records** returned.

***

### Generate your first article

#### REST (minimal)

```bash
curl https://api.chaingpt.org/news \
  -H "Authorization: Bearer $CHGPT_API_KEY" \
  -G --data-urlencode "limit=1"
```

Returns JSON with an array of articles sorted by `createdAt`.

#### SDK (equivalent)

```ts
const { data } = await ainews.getNews({ limit: 1 });
console.log(data[0].title);
```

Optional filters: `categoryId`, `subCategoryId`, `tokenId`, `fetchAfter`, `searchQuery`, `sortBy` (`createdAt` or `publishedAt`).

***

### Consume via RSS

Need a zero‑code option? Subscribe to the public feeds:

* General (All Crypto) feed `https://app.chaingpt.org/rssfeeds.xml`
* Bitcoin feed `https://app.chaingpt.org/rssfeeds-bitcoin.xml`
* BNB feed `https://app.chaingpt.org/rssfeeds-bnb.xml`
* Ethereum feed `https://app.chaingpt.org/rssfeeds-ethereum.xml`

{% hint style="info" %}
Note:

* More categories via RSS feed may be available by request.&#x20;
* Via RSS feed you can views up to the last 30 days.
{% endhint %}

```xml
        <item>
            <title><![CDATA[Critical Security Vulnerability in China ESP32 Chip Raises Concerns for Bitcoin Wallets]]></title>
            <description><![CDATA[A critical security flaw in the China ESP32 chip is causing alarm in the crypto community, especially impacting Bitcoin wallets. This vulnerability poses a serious threat to traders as it could lead to the theft of private keys and put millions of dollars in digital assets at risk. The ESP32 chip, manufactured by Espressif Systems, is widely used in hardware wallets due to its cost-effectiveness and adaptability. The vulnerability, known as CVE-2025-27840, allows hackers to bypass security measures and access private keys, potentially exposing seed phrases and enabling unauthorized transactions. This poses a significant risk to users, with experts warning of possible financial losses. The discovery of this flaw has sparked discussions about the security of Chinese-manufactured components in financial infrastructure, emphasizing the need for transparency from manufacturers to protect users.

Read more AI-generated news on: https://app.chaingpt.org/news]]></description>
            <link>https://app.chaingpt.org/news/16546/critical-security-vulnerability-in-china-esp32-chip-raises-concerns-for-bitcoin-wallets</link>
            <guid isPermaLink="false">16546</guid>
            <category><![CDATA[Security breaches]]></category>
            <category><![CDATA[Bitcoin]]></category>
            <category><![CDATA[Bitcoin - BTC]]></category>
            <dc:creator><![CDATA[Nova, ChainGPT's AI Agent]]></dc:creator>
            <pubDate>Thu, 17 Apr 2025 20:00:00 GMT</pubDate>
            <media:content url="https://d2qsg582zx9qac.cloudfront.net/document/394e5895-f2df-3db9-b963-95faead5f809.jpg" type="image/jpg"/>
        </item>
```

***

### Error handling & rate limits

* REST returns standard HTTP codes; non‑2xx responses include a JSON `message`.
* SDK throws `AINewsError`; catch and inspect `error.message`.
* Limit: **200 requests / minute** per key; burst above rate is throttled.

```ts
import { Errors } from '@chaingpt/ainews';

try {
  await ainews.getNews({});
} catch (err) {
  if (err instanceof Errors.AINewsError) {
    console.error(err.message);
  }
}
```

***

### Next steps

* **Pagination** – use `limit` & `offset` to page through results.
* **Advanced filtering** – full category, sub‑category, and token ID lists.
* **Production hardening** – implement caching and exponential back‑off.

***

### Resources

* [REST API reference](api-reference.md)
* [SDK reference](sdk-reference.md)
* [RSS feed specification](rss-reference.md)

Happy building!
