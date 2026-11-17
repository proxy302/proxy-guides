# How do I use a proxy with Python requests?

Pass a `proxies` dict that maps both `"http"` and `"https"` to the same proxy URL, for example `http://USER:PASS@HOST:PORT`, either on each call or on a `Session`. For SOCKS5, install `requests[socks]` and write `socks5h://` so the proxy resolves hostnames as well. With a rotating residential proxy, every plain `requests.get()` opens a new connection and can get a new exit IP, while a `Session` keeps its connection alive and usually keeps the same IP.

*Proxy302 maintains this guide and sells these proxies. The code works with any provider that gives you a host, port, username and password.*

## HTTP proxy with a username and password

```python
import requests

PROXY = "http://USER:PASS@HOST:PORT"
proxies = {"http": PROXY, "https": PROXY}

r = requests.get("https://ipinfo.io/json", proxies=proxies, timeout=30)
r.raise_for_status()
data = r.json()
print(data["ip"], data["country"], data.get("org"))
```

The `"https"` key decides what happens to `https://` URLs, yet its value still begins with `http://`: the proxy itself speaks plain HTTP and opens a CONNECT tunnel, and TLS runs inside it. Drop the `"https"` key and HTTPS requests quietly go direct, or through `HTTPS_PROXY` if that is set.

If the password contains `@`, `:` or `/`, percent-encode it first with `urllib.parse.quote(password, safe="")`.

## SOCKS5

```bash
pip install "requests[socks]"
```

```python
PROXY = "socks5h://USER:PASS@HOST:PORT"
proxies = {"http": PROXY, "https": PROXY}
```

| Proxy URL | Who looks up the site's hostname | Extra install |
|---|---|---|
| `http://` | The proxy | None |
| `socks5://` | Your machine | `requests[socks]` |
| `socks5h://` | The proxy | `requests[socks]` |

## Environment variables

```bash
export HTTP_PROXY="http://USER:PASS@HOST:PORT"
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
export NO_PROXY="localhost,127.0.0.1"
```

`requests` reads these by default. They also take priority over `Session.proxies`, which surprises many people; set `session.trust_env = False` when a session must use only what you gave it.

## Rotating per request or one Session

```python
import requests

PROXY = "http://USER:PASS@HOST:PORT"
proxies = {"http": PROXY, "https": PROXY}
URL = "https://api.ipify.org"

# requests.get() opens a fresh connection each call
for _ in range(3):
    print("one-off:", requests.get(URL, proxies=proxies, timeout=30).text)

# A Session reuses its connection, so the exit IP usually stays put
with requests.Session() as s:
    s.proxies.update(proxies)
    s.trust_env = False  # otherwise HTTP(S)_PROXY env vars override s.proxies
    for _ in range(3):
        print("session:", s.get(URL, timeout=30).text)
```

Run against a test proxy, the first loop opened three CONNECT tunnels and the session opened one. On a rotating gateway that is the difference between three exits and one.

## Common errors

| Error | Cause and fix |
|---|---|
| `ProxyError ... Tunnel connection failed: 407 Proxy Authentication Required` | Wrong username or password, or special characters not percent-encoded |
| `InvalidSchema: Missing dependencies for SOCKS support.` | Run `pip install "requests[socks]"` |
| SSL error or read timeout on every HTTPS call | The proxy URL starts with `https://` but the proxy speaks plain HTTP; use `http://` |
| `ProxyError ... Connection refused` | Wrong host or port |
| `ConnectionError: ('Connection aborted.', BadStatusLine(...))` | `http://` used for a SOCKS5 IP; switch the scheme |
| A session ignores its proxy | `HTTPS_PROXY` in the environment wins; set `trust_env = False` |
| The exit IP never changes on a rotating proxy | A `Session` is reusing one connection; use one-off calls or a new session |

## FAQ

**Do I really need both keys?** Yes. Each key covers URLs of its own scheme, and most sites redirect to HTTPS.

**How do I hold one IP through a short multi-step flow?** Give a `Session` a sticky session URL, built with the session parameter your provider documents; Proxy302's console generates it for you. See [rotating vs sticky sessions](../rotating-vs-sticky-sessions/README.md).

**Should a logged-in account use rotating traffic?** No. An account you sign in to for days needs one fixed address: a static residential (ISP) IP.

**Is there a free trial?** Not at Proxy302; the smallest paid test is a one-day static IP for $0.50.

## Where Proxy302 fits

Paste the four connection fields from a Proxy302 order into the URLs above. Rotating residential traffic is $1.50/GB pay as you go; a static residential (ISP) IP is $0.50 for a day or $7.99 for 30 days, and each one is sold as either HTTP or SOCKS5. See the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
