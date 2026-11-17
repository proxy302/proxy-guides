# SOCKS5 vs HTTP proxy: which should I use?

For web work (browsers, Playwright, Puppeteer, Selenium, scrapers and HTTP libraries) use an HTTP proxy: every tool supports it, HTTPS passes through it in an encrypted CONNECT tunnel, and Chrome-based browsers can log in to it. Choose SOCKS5 when the program doesn't speak HTTP (mail, chat, games, SSH, custom TCP) or only offers SOCKS5, and use `socks5h` so DNS lookups happen at the proxy. For HTTPS pages, neither type can read the content.

*Proxy302 maintains this guide and sells these proxies, in both protocols.*

## How each one works

An **HTTP proxy** understands HTTP. For an `http://` page it receives the whole request and fetches it for you. For an `https://` page your client sends `CONNECT site:443`, the proxy opens a TCP connection, and from then on it only relays encrypted bytes.

A **SOCKS5 proxy** does not care what protocol you run. Your client asks it to connect to a host and port, and it relays bytes both ways. The SOCKS5 standard also defines UDP relaying, but whether a given proxy offers it varies, so don't assume it.

## Side by side

| | HTTP proxy | SOCKS5 proxy |
|---|---|---|
| Carries | HTTP; HTTPS and other TCP through CONNECT | Any TCP; UDP only if the proxy supports it |
| Username and password in Chrome, Edge, Playwright, Puppeteer | Supported | Not supported |
| Username and password in curl, requests, antidetect browsers | Supported | Supported |
| Who resolves the site's hostname | The proxy | The proxy with `socks5h`, your machine with `socks5` |
| What the proxy sees of an HTTPS page | Hostname and port | Hostname (or IP) and port |
| Plain `http://` pages | Readable by the proxy | Readable by the proxy |
| Typical URL | `http://USER:PASS@HOST:PORT` | `socks5h://USER:PASS@HOST:PORT` |

The browser row is the one that decides most cases: a password-protected SOCKS5 proxy does not work in Chromium-based automation without a local forwarder.

## Test which protocol a proxy speaks

```bash
# Works only if HOST:PORT is an HTTP proxy
curl -s --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/ip; echo

# Works only if HOST:PORT is a SOCKS5 proxy
curl -s --max-time 20 -x "socks5h://USER:PASS@HOST:PORT" https://ipinfo.io/ip; echo
```

The same check in Python (`pip install "requests[socks]"`):

```python
import requests

for scheme in ("http", "socks5h"):
    url = f"{scheme}://USER:PASS@HOST:PORT"
    try:
        ip = requests.get("https://ipinfo.io/ip", proxies={"https": url}, timeout=20).text
        print(scheme, "works, exit IP", ip.strip())
    except requests.RequestException as e:
        print(scheme, "failed:", type(e).__name__)
```

Exactly one of the two should succeed. If both fail, check the credentials before the protocol.

## Quick decision

1. A browser or browser automation with a password? **HTTP.**
2. A scraper or HTTP library? **HTTP**, or SOCKS5 if you prefer; both work.
3. Mail, chat, a game, SSH or other non-HTTP traffic? **SOCKS5.**
4. The app's proxy field lists only one type? Use that one.

## Common errors

| Symptom | Cause |
|---|---|
| Timeout or "Proxy CONNECT aborted" with `http://` | The port speaks SOCKS5 |
| SOCKS handshake error or timeout with `socks5h://` | The port speaks HTTP |
| Browser: "Browser does not support socks5 proxy authentication" or `ERR_NO_SUPPORTED_PROXIES` | SOCKS5 with a password in Chromium; use HTTP |
| Site sees the right IP but DNS goes to your local resolver | `socks5://` instead of `socks5h://` |

## FAQ

**Is SOCKS5 faster?** The protocol overhead of either is small. Distance to the proxy and the quality of the exit network matter far more.

**Is SOCKS5 more private?** Websites see the same exit IP either way, and HTTPS content is encrypted inside both. The real difference is DNS: use `socks5h`.

**Can one IP do both?** At Proxy302 a static IP carries one protocol, chosen at purchase. If you are unsure, pick HTTP for browser work.

**Do antidetect browsers care?** They accept both; match the type to the IP you bought.

## Where Proxy302 fits

Proxy302 static residential (ISP) IPs cost $0.50 for one day, $3.50 for 7 days and $7.99 for 30 days, the same in every country, with HTTP or SOCKS5 picked per IP. See [pricing](https://proxy302.com/pricing/) and the [curl cheat sheet](../curl-proxy-cheatsheet/README.md).

*Facts checked 17 November 2026.*
