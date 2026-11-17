# How do I use rotating residential proxies with Scrapy?

Put the gateway URL in `request.meta["proxy"]`, for example `http://USER:PASS@HOST:PORT`, or set it once in a small downloader middleware so every request carries it. Scrapy's built-in `HttpProxyMiddleware` turns the username and password into a `Proxy-Authorization` header. Add 403 and 429 to `RETRY_HTTP_CODES` with a low `RETRY_TIMES`, and know that Scrapy keeps its tunnel to each site open, so retries often leave from the same exit IP unless you close the connection.

*Proxy302 maintains this guide and sells these proxies. Tested with Scrapy 2.20.*

## Per request: `meta["proxy"]`

```python
import scrapy

PROXY = "http://USER:PASS@HOST:PORT"

class IpSpider(scrapy.Spider):
    name = "ip"

    async def start(self):
        yield scrapy.Request("https://api.ipify.org?format=json", meta={"proxy": PROXY})

    def parse(self, response):
        yield {"ip": response.json()["ip"]}
```

`async def start()` is the Scrapy 2.13+ form; on older versions name it `start_requests` and drop `async`.

## For every request: a middleware

```python
# myproject/middlewares.py
class ProxyMiddleware:
    """Send each request through the proxy in settings unless it already has one."""

    def __init__(self, proxy_url, fresh_connection):
        self.proxy_url = proxy_url
        self.fresh_connection = fresh_connection

    @classmethod
    def from_crawler(cls, crawler):
        s = crawler.settings
        return cls(s.get("PROXY_URL"), s.getbool("PROXY_FRESH_CONNECTION"))

    def process_request(self, request):
        request.meta.setdefault("proxy", self.proxy_url)
        if self.fresh_connection:
            request.headers["Connection"] = "close"
```

`process_request(self, request)` without a `spider` argument is the Scrapy 2.14+ signature; older versions pass `(request, spider)`.

```python
# myproject/settings.py
PROXY_URL = "http://USER:PASS@HOST:PORT"
PROXY_FRESH_CONNECTION = False  # True = new tunnel, and so a new exit, per request

DOWNLOADER_MIDDLEWARES = {
    "myproject.middlewares.ProxyMiddleware": 350,  # before HttpProxyMiddleware (750)
}

RETRY_HTTP_CODES = [403, 429, 500, 502, 503, 504, 522, 524, 408]
RETRY_TIMES = 2
AUTOTHROTTLE_ENABLED = True
CONCURRENT_REQUESTS_PER_DOMAIN = 4
ROBOTSTXT_OBEY = True
```

## Why retries can keep the same IP

In a test against a logging proxy, three requests to one site opened a single CONNECT tunnel, and a URL that kept returning 429 was retried twice inside that same tunnel. On a rotating gateway the exit belongs to the tunnel, so those retries left from one IP. With `Connection: close` on each request, every request and every retry opened its own tunnel. That costs an extra TLS handshake per request, so turn it on only when you need it.

| Setting | Effect |
|---|---|
| `meta["proxy"]` | Proxy for that request; a `None` value means no proxy |
| Middleware priority below 750 | Runs before the built-in `HttpProxyMiddleware` |
| `RETRY_HTTP_CODES` with 403, 429 | Default list has 429 but not 403 |
| `RETRY_TIMES = 2` | Caps the extra traffic each failing URL can cost |
| `Connection: close` | New tunnel per request, at the cost of more handshakes |
| `AUTOTHROTTLE_ENABLED` | Slows down when the site slows down |

If a site keeps answering 403 after retries, treat it as a refusal: slow down and check the site's terms rather than adding more retries.

## Common errors

| Log line | Cause |
|---|---|
| `TunnelError: Could not open CONNECT tunnel with proxy HOST:PORT [{'status': 407, ...}]` | Wrong username or password |
| `ScrapyDeprecationWarning: ... process_request() requires a spider argument` | Old middleware signature on Scrapy 2.14+; drop `spider` |
| `Gave up retrying <GET ...> (failed 3 times): 403 Forbidden` | The site refused every attempt |
| Requests go out without the proxy | Middleware priority above 750, or `meta["proxy"]` set to `None` |
| Every item shows the same IP | Reused tunnel; see `PROXY_FRESH_CONNECTION` |

## FAQ

**Can I use the `http_proxy` environment variable instead?** Yes. `HttpProxyMiddleware` reads `http_proxy` and `https_proxy` when a request has no `meta["proxy"]`.

**How do I keep one IP for a pagination flow?** Build a sticky session URL with the session parameter your provider documents (Proxy302's console generates it for you) and put it in `meta["proxy"]` for just those requests.

**How much traffic did my crawl use?** Scrapy's end-of-run stats include `downloader/response_bytes`. Divide by 10^9 for GB.

## Where Proxy302 fits

Proxy302's rotating residential traffic costs $1.50/GB on the budget line, dropping to $1.20/GB with 300 GB prepaid and $1.124/GB with 500 GB; prepaid traffic does not expire. See [pricing](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
