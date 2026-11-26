# How do I connect to a WebSocket through an authenticated proxy in Python?

With the `websockets` library (version 15.0 or later), pass `proxy="http://USER:PASS@HOST:PORT"` to `connect()`, or set `wss_proxy` (or `HTTPS_PROXY`) and leave the default `proxy=True`. With `aiohttp`, pass `proxy="http://HOST:PORT"` and `proxy_auth=aiohttp.BasicAuth(USER, PASS)` to `session.ws_connect()`; note that its environment lookup uses `wss_proxy`, not `HTTPS_PROXY`, for `wss://` URLs. Both open a `CONNECT` tunnel through an HTTP proxy, so the WebSocket stays end-to-end encrypted. For SOCKS5, `websockets` needs the extra `python-socks` package and `aiohttp` needs a third-party connector. Proxy302 maintains this guide and sells these proxies; the `aiohttp` code below was run with aiohttp 3.13.5 on Python 3.9 against `wss://echo.websocket.org/` through a local test proxy with a username and password on 26 November 2026, and the `websockets` code follows that library's documentation.

## websockets

```python
import asyncio

from websockets.asyncio.client import connect

PROXY = "http://USER:PASS@HOST:PORT"   # percent-encode special characters in PASS


async def main():
    async with connect("wss://echo.websocket.org/", proxy=PROXY) as ws:
        await ws.send("hello through the proxy")
        async for message in ws:
            print("received:", message)
            if message == "hello through the proxy":
                break


asyncio.run(main())
```

What the library does with `proxy`, according to its [proxy documentation](https://websockets.readthedocs.io/en/stable/topics/proxies.html):

| `proxy=` | Behaviour |
|---|---|
| `True` (default) | Use the proxy configured in the environment or the operating system |
| `"http://USER:PASS@HOST:PORT"` | Use this proxy, with Basic authentication from the URL |
| `"socks5h://USER:PASS@HOST:PORT"` | SOCKS5 with DNS through the proxy; needs `pip install "python-socks[asyncio]"` |
| `None` | No proxy, even if one is configured |

With `proxy=True`, `websockets` looks in this order: `wss_proxy` (or `ws_proxy` for `ws://`), a SOCKS proxy set in the operating system, then an HTTP proxy from the system or `https_proxy`, and for `ws://` URLs also `http_proxy`. `no_proxy` wins over all of them. Lowercase names are recommended.

The threading client, `websockets.sync.client.connect`, takes the same `proxy` argument.

**Versions.** Proxy support arrived in websockets 15.0 (February 2025), and 16.0 added HTTP/1.0 proxies. Recent releases need a recent Python (17.0 requires Python 3.11 or newer), so check the release notes against your interpreter before you pin a version.

**Errors.** A proxy that refuses the connection (for example with 407 for a wrong password) raises `websockets.exceptions.InvalidProxyStatus`; a malformed proxy answer raises `InvalidProxyMessage`; both are subclasses of `ProxyError`. A proxy URL that cannot be parsed raises `InvalidProxy`.

## aiohttp

```python
import asyncio
import os

import aiohttp

PROXY = f"http://{os.environ['PROXY_HOST']}:{os.environ['PROXY_PORT']}"
AUTH = aiohttp.BasicAuth(os.environ["PROXY_USER"], os.environ["PROXY_PASS"])


async def main():
    async with aiohttp.ClientSession() as session:
        async with session.ws_connect(
            "wss://echo.websocket.org/",
            proxy=PROXY,
            proxy_auth=AUTH,
            heartbeat=30,
        ) as ws:
            await ws.send_str("hello through the proxy")
            async for msg in ws:
                if msg.type == aiohttp.WSMsgType.TEXT:
                    print("received:", msg.data)
                    if msg.data == "hello through the proxy":
                        break


asyncio.run(main())
```

```bash
PROXY_HOST=HOST PROXY_PORT=PORT PROXY_USER=USER PROXY_PASS=PASS python3 ws_aiohttp.py
```

Output from our run (the echo server sends its own greeting first):

```text
received: Request served by 4d896d95b55478
received: hello through the proxy
```

With a wrong password the same script stopped with:

```text
aiohttp.client_exceptions.ClientHttpProxyError: 407, message='Proxy Authentication Required', url='http://HOST:PORT'
```

`BasicAuth` takes the password as-is, so special characters need no encoding. Putting `http://USER:PASS@HOST:PORT` into `proxy=` also works, but then the password must be percent-encoded.

**Environment variables with aiohttp.** With `aiohttp.ClientSession(trust_env=True)`, aiohttp looks the proxy up by the URL's scheme. In our test a `wss://` connection went through the proxy when `wss_proxy` was set, and went direct when only `HTTPS_PROXY` (or `https_proxy`, or `HTTP_PROXY`) was set. That catches people out, because for ordinary `https://` requests `HTTPS_PROXY` is exactly the variable aiohttp uses.

## Which variable does each library read?

| Variable | websockets (`proxy=True`) | aiohttp (`trust_env=True`) |
|---|---|---|
| `wss_proxy` | Yes, checked first for `wss://` | Yes, for `wss://` (tested) |
| `ws_proxy` | Yes, checked first for `ws://` | Yes, for `ws://` (tested) |
| `HTTPS_PROXY` | Yes, when no `wss_proxy` / `ws_proxy` is set | Not used for `wss://` (tested) |
| `HTTP_PROXY` | Only for `ws://` | Not used for `wss://` (tested) |
| `NO_PROXY` | Yes | Yes |

The one variable both libraries read for `wss://` is `wss_proxy` (lowercase). If you want no surprises at all, pass the proxy explicitly in code, as in the examples above.

## Check that it really goes through the proxy

A WebSocket server rarely tells you your IP, so test the proxy with an ordinary request first:

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

Then compare with what the WebSocket server logs on its side if you control it. A static IP gives the same exit for every connection, which also keeps a long-running socket's reconnections on one address.

## Keeping long connections alive

Proxies and the networks behind them close idle TCP connections. Keep pings on: `websockets` sends one every 20 seconds by default (`ping_interval`), and `aiohttp` does when you pass `heartbeat=`. Reconnect with a short back-off when the socket closes.

## Troubleshooting

| Error | Meaning | Fix |
|---|---|---|
| `InvalidProxyStatus` / `ClientHttpProxyError: 407` | Proxy rejected the username or password | Copy them again; encode special characters if they are in the URL |
| `InvalidProxy` | The proxy URL could not be parsed | Check the scheme (`http://`, `socks5h://`) and the port |
| SOCKS proxy ignored or import error | `python-socks` missing | `pip install "python-socks[asyncio]"`, or use an HTTP IP |
| aiohttp connects directly although `HTTPS_PROXY` is set | aiohttp looks up `wss_proxy` for `wss://` URLs | Set `wss_proxy`, or pass `proxy=` and `proxy_auth=` |
| Handshake hangs, then times out | Wrong protocol: an HTTP URL pointed at a SOCKS5 port, or the reverse | Match the scheme to the protocol you bought |

## FAQ

**Does the proxy see my WebSocket messages?** Not for `wss://`. The proxy only relays the encrypted tunnel after `CONNECT`.

**Do I need a special "WebSocket proxy"?** No. Any HTTP proxy that allows `CONNECT` to the server's port works. Port 443 is the usual case.

**Static or rotating for a long-lived socket?** Static. A rotating proxy may hand each reconnection a new exit.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (HTTP or SOCKS5, chosen at purchase; $0.50 for one day, $7.99 for 30 days, the same in every country) and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/). For plain HTTP requests with these libraries, see [httpx and aiohttp with a proxy](../httpx-aiohttp-proxy/README.md).

*Facts checked 26 November 2026.*
