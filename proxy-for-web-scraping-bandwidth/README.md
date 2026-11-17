# How do I cut bandwidth (and cost) when scraping through residential proxies?

Rotating residential traffic is billed per GB, so the cheapest byte is one you never download. Block images, fonts and media in the browser, call the site's JSON endpoints instead of rendering whole pages, accept compressed responses, reuse connections, cache pages that have not changed, and retry only on 403 and 429 with a cap. At Proxy302's budget rate of $1.50/GB, every gigabyte you avoid saves $1.50 (or $1.124 to $1.42 on prepaid packages).

*Proxy302 maintains this guide and sells these proxies.*

## Block heavy resources in Playwright and measure the result

```python
from playwright.sync_api import sync_playwright

PROXY = {"server": "http://HOST:PORT", "username": "USER", "password": "PASS"}
BLOCK = {"image", "media", "font"}

def block_heavy(route):
    if route.request.resource_type in BLOCK:
        route.abort()
    else:
        route.continue_()

with sync_playwright() as p:
    browser = p.chromium.launch(proxy=PROXY)
    page = browser.new_page()
    page.route("**/*", block_heavy)

    received = []
    page.on("requestfinished", lambda req: received.append(req.sizes()))

    page.goto("https://en.wikipedia.org/wiki/Proxy_server", wait_until="load")
    total = sum(s["responseBodySize"] + s["responseHeadersSize"] for s in received)
    print(f"{len(received)} responses, {total / 1024:.0f} KB received")
    browser.close()
```

Run it once with `BLOCK = set()` and once as shown; the difference is your saving per page. Keep scripts and stylesheets unless the data survives without them.

## Use HTTP clients well

```python
import time
import requests

PROXY = "http://USER:PASS@HOST:PORT"
session = requests.Session()  # reused connections skip repeat TLS handshakes
session.proxies.update({"http": PROXY, "https": PROXY})
etags = {}

def fetch(url, max_retries=2):
    headers = {"If-None-Match": etags[url]} if url in etags else {}
    for attempt in range(max_retries + 1):
        r = session.get(url, headers=headers, timeout=30)
        if r.status_code not in (403, 429) or attempt == max_retries:
            break
        wait = r.headers.get("Retry-After", "")
        time.sleep(int(wait) if wait.isdigit() else 2 ** attempt)
    if r.status_code == 304:
        return None  # unchanged since last fetch: no body was sent
    if "ETag" in r.headers:
        etags[url] = r.headers["ETag"]
    return r
```

- **Compression.** `requests` already asks for gzip and deflate, plus brotli and zstd when their decoders are installed. Check that your own client does too.
- **Conditional requests.** When a server sends an `ETag`, asking with `If-None-Match` returns `304 Not Modified` with no body if nothing changed.
- **Retries.** Every retry downloads the page again. Retry 403 and 429 a couple of times with a pause, honour `Retry-After`, and stop there. A site that keeps answering 403 is refusing you; check its terms instead of paying for more attempts.
- **JSON endpoints.** In the browser's developer tools, filter the Network tab to Fetch/XHR and find the request that carries the data. Calling it directly usually costs a fraction of a rendered page, where the site's terms allow it.
- **Browser background traffic.** Everything a browser fetches is proxy traffic, including the requests branded Chrome makes on its own.

## A worked example

Illustrative page sizes for 100,000 pages a month; measure your own with the script above. (1 GB = 1,000 MB here.)

| How you fetch | Per page | Traffic | Pay as you go at $1.50/GB |
|---|---|---|---|
| Full page, nothing blocked | 2 MB | 200 GB | $300.00 |
| Images, fonts and media blocked | 0.5 MB | 50 GB | $75.00, or $71.00 as the 50 GB prepaid package |
| The site's JSON endpoint | 30 KB | 3 GB | $4.50 |

Retries add to each line: at one retry per ten pages, the full-page row grows by 20 GB, another $30. If you need 200 GB every month, the 300 GB prepaid package at $1.20/GB brings that month to $240, and the remaining 100 GB carries over because prepaid traffic does not expire.

## Common errors

| Symptom | Likely cause |
|---|---|
| Blocking images barely changed the bill | Most bytes are scripts, JSON or video; check sizes by resource type |
| The data vanished after blocking | You blocked scripts or stylesheets the page needs |
| Traffic used is higher than your page measurements | Retries, redirects, TLS handshakes and background requests |
| `304` never comes back | The server does not send `ETag`; try `Last-Modified` with `If-Modified-Since` |

## FAQ

**Is datacenter traffic cheaper?** Yes: Proxy302's rotating datacenter traffic is $0.40/GB. Many sites treat server addresses differently from home connections, though, so try your target with a small amount first.

**Does compression lower the bill?** Yes. Per-GB billing counts the bytes passing through the proxy, and a compressed response is smaller on the wire.

**When should I prepay?** After one measured month shows your real volume. Prepaid traffic does not expire, so buying ahead loses nothing.

## Where Proxy302 fits

Proxy302's budget residential line is $1.50/GB, with prepaid rates of $1.42 (50 GB), $1.35 (100 GB), $1.20 (300 GB) and $1.124/GB (500 GB); the premium line is $5/GB. See [pricing](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
