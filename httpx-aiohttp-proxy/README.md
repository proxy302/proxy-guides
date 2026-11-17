# How do I use a proxy with httpx or aiohttp (async Python)?

In httpx 0.28, give the client a single proxy URL: `httpx.AsyncClient(proxy="http://USER:PASS@HOST:PORT")`. The older `proxies=` argument has been removed. In aiohttp, pass `proxy="http://HOST:PORT"` with `proxy_auth=aiohttp.BasicAuth("USER", "PASS")` on the request or on the `ClientSession`. aiohttp's own proxy support covers HTTP proxies only, so SOCKS5 goes through the `aiohttp-socks` connector, while httpx handles SOCKS5 itself once you install `httpx[socks]`.

*Proxy302 maintains this guide and sells these proxies. Nothing below is specific to us.*

## httpx

```python
import asyncio
import httpx

PROXY = "http://USER:PASS@HOST:PORT"  # socks5://... needs: pip install "httpx[socks]"
URLS = ["https://api.ipify.org?format=json"] * 5

async def main():
    limits = httpx.Limits(max_connections=10)
    async with httpx.AsyncClient(proxy=PROXY, limits=limits, timeout=30) as client:
        responses = await asyncio.gather(*(client.get(u) for u in URLS))
        for r in responses:
            print(r.status_code, r.json()["ip"])

asyncio.run(main())
```

`max_connections` caps how many tunnels are open at once, which also caps the load you put on the target site.

## aiohttp with an HTTP proxy

```python
import asyncio
import aiohttp

PROXY = "http://HOST:PORT"
AUTH = aiohttp.BasicAuth("USER", "PASS")

async def main():
    timeout = aiohttp.ClientTimeout(total=30)
    async with aiohttp.ClientSession(timeout=timeout) as session:
        async with session.get("https://ipinfo.io/json", proxy=PROXY, proxy_auth=AUTH) as r:
            data = await r.json()
            print(r.status, data["ip"])

asyncio.run(main())
```

Current aiohttp also accepts `proxy=` and `proxy_auth=` on `ClientSession(...)`, so you don't repeat them on every call.

## aiohttp with SOCKS5

```bash
pip install aiohttp-socks
```

```python
import asyncio
import aiohttp
from aiohttp_socks import ProxyConnector

async def main():
    connector = ProxyConnector.from_url("socks5://USER:PASS@HOST:PORT")
    async with aiohttp.ClientSession(connector=connector) as session:
        async with session.get("https://ipinfo.io/json") as r:
            print(r.status, (await r.json())["ip"])

asyncio.run(main())
```

`aiohttp-socks` sends the hostname to the proxy by default, so DNS is resolved there. It accepts `socks5://`, not `socks5h://`.

## At a glance

| | httpx 0.28 | aiohttp |
|---|---|---|
| HTTP proxy | `AsyncClient(proxy=...)` | `proxy=` plus `proxy_auth=` |
| SOCKS5 | `pip install "httpx[socks]"`, then `socks5://` or `socks5h://` | `aiohttp-socks` `ProxyConnector` |
| `HTTP_PROXY` / `HTTPS_PROXY` | Read by default (`trust_env=True`) | Read only with `ClientSession(trust_env=True)` |
| Different proxy per host | `mounts={"https://example.com": httpx.AsyncHTTPTransport(proxy=...)}` | Pass `proxy=` per request |

On a rotating residential gateway, each new connection can leave from a different IP. Requests that run in parallel open separate connections; requests that reuse an open keep-alive connection keep its exit.

## Common errors

| Error | Cause and fix |
|---|---|
| `TypeError: ... unexpected keyword argument 'proxies'` | httpx 0.28 removed `proxies=`; use `proxy=` or `mounts=` |
| `ImportError: Using SOCKS proxy, but the 'socksio' package is not installed` | `pip install "httpx[socks]"` |
| `httpx.ProxyError: 407 Proxy Authentication Required` | Wrong username or password in the proxy URL |
| `aiohttp.ClientHttpProxyError: 407` | Missing or wrong `proxy_auth` |
| `ClientResponseError: 400, message="Expected HTTP/..."` from aiohttp | A `socks5://` URL was passed to `proxy=`; use `aiohttp-socks` |
| Every task reports the same exit IP | Reused keep-alive connections; expected, and fine for one-site sessions |

## FAQ

**Which one should I choose?** Both work. httpx has a requests-like API and covers both sync and async code; aiohttp is mature and widely deployed in async crawlers.

**How many concurrent requests are safe?** Start low, watch for 429 responses, and stay within the site's published limits. More concurrency also means more traffic billed per GB.

**Can I keep one IP for a multi-step flow?** Yes: create one client whose proxy URL carries the session parameter your provider documents; Proxy302's console generates it for you. Run the whole flow on that client.

**Do these clients work with a static IP?** Yes. A static residential (ISP) IP is just a proxy URL that keeps the same exit for its term.

## Where Proxy302 fits

Proxy302 sells rotating residential traffic on a budget line at $1.50/GB and a premium line at $5/GB, with lower per-GB prices for prepaid packages that do not expire. Static ISP IPs start at $0.50 for one day. Details on the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
