# Why does a website return 403 Forbidden only when I use a proxy?

First find out who sent the 403. If the proxy sent it, the tunnel was never opened: your tool reports something like `CONNECT tunnel failed, response 403` or `Tunnel connection failed: 403 Forbidden`, and the cause is on the proxy side (a blocked port or destination, or an account restriction). If the website sent it, the tunnel worked and the site chose to refuse requests from that address or that request. Sites do this because of a policy about the IP's network type or country, because the address has a history of abuse, because the request rate is too high, or because the request itself looks incomplete. A proxy does not override a site's access rules; when a site deliberately refuses an address type, the right move is its official API, an allowlisting request, or a different source of the data.

*Proxy302 maintains this guide and sells these proxies. The commands were run with curl 8.7.1 and Python requests 2.32 against a throwaway local proxy in November 2026.*

## Step 1: proxy 403 or site 403?

curl can print both status codes in one line. `%{http_connect}` is the proxy's answer to the tunnel request; `%{http_code}` is the website's answer:

```bash
curl -sS -o /dev/null -w 'proxy=%{http_connect} site=%{http_code}\n' \
  -x http://USER:PASS@HOST:PORT https://example.com/
```

| Output | Who refused | Where to look |
|---|---|---|
| `proxy=403 site=000` plus `CONNECT tunnel failed, response 403` | The proxy | Destination or port not allowed, account limit, wrong product for this use |
| `proxy=200 site=403` | The website | The rest of this page |
| `proxy=407 site=000` | The proxy, for credentials | Username or password ([407 troubleshooting in Chinese](../proxy-407-timeout-troubleshooting-zh/README.md)) |

In Python, the same split shows up as two different outcomes:

```python
import requests

proxy = "http://USER:PASS@HOST:PORT"
try:
    r = requests.get("https://example.com/", proxies={"http": proxy, "https": proxy}, timeout=30)
    print("site answered", r.status_code)          # a 403 here came from the website
except requests.exceptions.ProxyError as e:
    print("proxy refused the tunnel:", e)          # ... Tunnel connection failed: 403 Forbidden
```

## Step 2: if the site sent it, find out why

Compare the same request with and without the proxy, from the same machine:

```bash
curl -sS -o /dev/null -w '%{http_code}\n' https://example.com/
curl -sS -o /dev/null -w '%{http_code}\n' -x http://USER:PASS@HOST:PORT https://example.com/
curl -sS -x http://USER:PASS@HOST:PORT https://ipinfo.io/json
```

The last line shows the exit address, its country and the network (`org`) it belongs to. Then read the 403 page itself (`curl -sS -x ... https://example.com/ | head -50`): many sites name the reason, a firewall vendor, or a reference ID.

| What you find | Likely reason | Reasonable response |
|---|---|---|
| `org` is a hosting or cloud company | The site refuses datacenter networks as a policy | Use the site's API, ask for access, or use an address type the site accepts for your use |
| Exit country differs from where the site serves | The site serves only some countries | Respect it; a proxy is not a way around a region restriction |
| 403 only after many requests | Rate limiting | Slow down, cache, honour `Retry-After` and `robots.txt` |
| 403 even for one request with a browser on the same IP | The address has a bad history on that site | Ask your provider for a replacement IP, or contact the site |
| 403 with curl, 200 with a browser on the same IP | The request is missing what the site expects (cookies, a normal `Accept` header, a login) | Send the same request your app or browser sends; log in properly |

## Step 3: rule out the proxy setup

- **Location mismatch inside your app.** If an account was created from one country and the proxy exits in another, some sites lock the session. Keep one account on one fixed IP in the account's country.
- **Several tools, several exits.** A rotating gateway gives a new address for each new connection. A site that ties a session to an address can refuse the second request. Use a sticky session (the session parameter your provider documents; Proxy302's console generates it for you) or a static IP for multi-step flows.
- **The IP changed.** When a static IP's term ends it is released, and a new purchase gets a new address. Allowlists on the site's side then need updating.

## FAQ

**Will a residential IP fix every 403?** No. It changes the network type the site sees, nothing else. Rate limits, logins, terms of use and country rules apply to residential addresses too.

**The site worked through the proxy yesterday and returns 403 today.** Either the site changed a rule, the exit IP changed (rotating traffic, or an ended static term), or the request volume grew. Re-run Step 1 and Step 2.

**Is a 403 the same as a CAPTCHA page?** No, but both mean the site wants something other than an automated request. Do not try to defeat the challenge; use the site's official access route.

**Can I ask a provider for a different IP?** Yes, describe the site and the result of Step 1. A one-day static IP at Proxy302 costs $0.50 if you need a clean test.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs from $0.50 for one day or $7.99 for 30 days per IP, rotating residential traffic from $1.50/GB, and datacenter IPs for sites that treat all visitors alike. See [pricing](https://proxy302.com/pricing/) and [datacenter vs residential](../datacenter-vs-residential-proxy/README.md).

*Facts checked 20 November 2026.*
