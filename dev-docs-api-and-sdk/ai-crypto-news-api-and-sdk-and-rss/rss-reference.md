# RSS Reference

## ChainGPT AI News Generator RSS Reference

ChainGPT exposes a set of **public, read‑only RSS 2.0 feeds** that surface the latest AI‑driven crypto and blockchain news. Because the feeds are _open to the public_, you can pull them with any HTTP client—no API key, credits, or headers required.

***

### Retention window

Each feed always contains up to **30 days of articles**; older items roll off automatically.

***

### Feed catalogue

| Feed                | URL                                              |
| ------------------- | ------------------------------------------------ |
| **All Crypto**      | `https://app.chaingpt.org/rssfeeds.xml`          |
| **Bitcoin (BTC)**   | `https://app.chaingpt.org/rssfeeds-bitcoin.xml`  |
| **BNB Chain (BNB)** | `https://app.chaingpt.org/rssfeeds-bnb.xml`      |
| **Ethereum (ETH)**  | `https://app.chaingpt.org/rssfeeds-ethereum.xml` |

***

### Quick fetch example

```bash
curl -L https://app.chaingpt.org/rssfeeds.xml
```

The endpoint returns standard RSS 2.0 XML. Parse it with any RSS library or an XML parser in your language of choice.

***

### Best practices

* **Cache intelligently** – the content changes as new articles publish, but the 30‑day cap prevents unbounded growth.
* **Respect intervals** – polling every 5‑10 minutes is plenty for most use‑cases.
* **Filter client‑side** – if you need category or token‑specific filtering beyond the four provided feeds, fetch the general feed and apply your own logic, or switch to the REST API/SDK for granular queries.
