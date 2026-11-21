# Which proxy should I use for price monitoring?

For retail sites and marketplaces that show prices by country or treat server traffic differently, use rotating residential proxies billed per GB, with a new exit for each independent page and a sticky session only for multi-step flows. Use datacenter proxies when the target answers server addresses the same way, when you call an official API or feed, or when a data partner allowlists one fixed IP. Proxy302 maintains this guide and sells these proxies: rotating residential is $1.50/GB, falling to $1.124/GB with 500 GB prepaid (prepaid traffic does not expire); rotating datacenter is $0.40/GB; a static datacenter IP is $3.99 for 30 days. Whatever you buy, follow each site's terms, keep request rates modest, and prefer an official API or feed where the site offers one.

## Decide by target, not by habit

| What you monitor | Proxy | Why | Proxy302 price |
|---|---|---|---|
| Public product pages that differ by country | Rotating residential | Pages as local visitors see them; new exit per request | $1.50/GB; prepaid 50 GB $1.42, 100 GB $1.35, 300 GB $1.20, 500 GB $1.124 |
| Same, when you need a specific ISP | Rotating residential, premium line | ISP choice is offered on the premium line | $5/GB |
| Sites and APIs that treat every visitor alike | Rotating datacenter | Cheapest per GB | $0.40/GB |
| A partner feed or API that allowlists your address | Static datacenter IP | One fixed address for the allowlist | $0.25 per day, $3.99 per 30 days |
| Prices shown on mobile carrier networks | Rotating mobile | Carrier exits | $8/GB |

If you are not sure, run the same 50 URLs through datacenter and residential for a day and compare error rates and prices returned. Buy volume only after that.

## Estimate the traffic before you pick a package

Per-GB billing means page weight sets the bill. An illustrative estimate, with assumptions you should replace with your own:

| Approach | Assumed size per page | 2,000 pages a day for 30 days | At $1.50/GB |
|---|---|---|---|
| Plain HTTP request, HTML only, compressed | 150 KB | about 9 GB | about $13.50 |
| Headless browser loading every image and script | 2 MB | about 120 GB | about $180, or $1.20/GB with 300 GB prepaid |

Fetching the product's JSON endpoint where the site publishes one, blocking images and fonts in a browser, and caching pages that do not change cut this a lot. See [how to cut bandwidth when scraping through residential proxies](../proxy-for-web-scraping-bandwidth/README.md).

## Rotating or sticky?

- **Rotating** (new exit per connection) for independent pages: one product, one request.
- **Sticky** for flows that must stay on one address for a few minutes, such as choosing a delivery area and then reading prices for it. Use the session parameter your provider documents; Proxy302's console generates it for you.
- **A static IP** only for allowlisted feeds, not for crawling.

## A minimal Python loop

```python
import random
import time

import requests

PROXY = "http://USER:PASS@HOST:PORT"   # placeholders
PROXIES = {"http": PROXY, "https": PROXY}

def fetch(url, tries=3):
    for attempt in range(tries):
        # requests.get opens a new connection each call, so a rotating
        # proxy gives each attempt a fresh exit
        r = requests.get(url, proxies=PROXIES, timeout=30)
        if r.status_code in (403, 429):
            time.sleep(2 ** attempt + random.random())
            continue
        r.raise_for_status()
        return r.text
    return None

for url in ["https://example.com/product/1", "https://example.com/product/2"]:
    html = fetch(url)
    time.sleep(1 + random.random())   # keep the rate modest
```

For SOCKS5, install `requests[socks]` and use `socks5h://USER:PASS@HOST:PORT` so DNS is resolved through the proxy. To keep one exit across several requests, use a `requests.Session()` with the sticky session credentials: the session reuses its connection.

Check where a request leaves from:

```bash
curl -sS -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

## Getting prices right

- **Country.** Confirm the exit country with ipinfo before blaming the site.
- **Currency and region.** Where the site has its own country, currency or delivery-area setting, use it, and record which setting each price came from.
- **Rendering.** If the price is missing from the HTML, it is loaded by script; look for the request that returns it, or use a headless browser for those pages only.
- **CAPTCHA pages.** Treat them as a signal to slow down and reduce concurrency. Do not use CAPTCHA-solving services.

## Being a good citizen

Read the site's terms and robots.txt, collect only the public data you need, do not collect personal data, and space requests out. If a retailer offers an affiliate feed, a partner API or a data export, it is usually more accurate and cheaper than crawling.

## Common errors

| Symptom | Likely cause | Fix |
|---|---|---|
| 407 from the proxy | Wrong proxy username or password | Copy the credentials again |
| 403 or 429 from the site | Too many requests, or the site refuses that address type | Slow down, retry with a cap, try residential |
| Wrong currency | Exit country or the site's own setting | Check the exit; set the site's country |
| Bill higher than expected | Full-page loads in a browser | Fetch HTML or JSON only, block media |
| Same price for every country | The site prices one way everywhere | Datacenter is then enough |

## FAQ

**Does prepaid traffic expire?** Not at Proxy302. Prepaid rotating residential traffic does not expire.

**Do I need city targeting?** Only if prices differ by city or delivery area. Most stores price by country.

**Can I monitor prices behind a login?** Use the site's official API or your own account's export. Signing in to many accounts to crawl usually breaks the site's terms.

**Is there a free trial?** No. The smallest rotating purchase is 1 GB at $1.50.

## Where Proxy302 fits

Proxy302 sells rotating residential (budget and premium lines), rotating datacenter, rotating mobile and static datacenter IPs. See [rotating residential](https://proxy302.com/proxies/rotating-residential/), [datacenter](https://proxy302.com/proxies/datacenter/), [market research use cases](https://proxy302.com/use-cases/market-research/) and [pricing](https://proxy302.com/pricing/). Related: [rotating vs sticky sessions](../rotating-vs-sticky-sessions/README.md) and [datacenter vs residential](../datacenter-vs-residential-proxy/README.md).

*Facts checked 21 November 2026.*
