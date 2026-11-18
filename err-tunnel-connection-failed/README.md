# What does ERR_TUNNEL_CONNECTION_FAILED mean, and how do I fix it?

`ERR_TUNNEL_CONNECTION_FAILED` means Chrome reached your proxy, asked it to open a tunnel to an HTTPS site, and the proxy said no. For every `https://` page, Chrome sends the proxy a `CONNECT site:443` request; any reply other than `200` ends in this error, most often `403 Forbidden`, `502 Bad Gateway` or `503 Service Unavailable`. The proxy address itself is therefore fine. The fix is to find out which status the proxy returned and why: whether the IP is still active, whether the site is reachable through it, and whether a second proxy or a company network is in the way.

*Proxy302 maintains this guide and sells these proxies. In our test on Chrome 157, a local proxy answering CONNECT with 403, 502 or 503 produced exactly this error each time (November 2026).*

## What the error page looks like

Chrome titles the page "This site can't be reached" and says the page "might be temporarily down or it may have moved permanently to a new web address", with `ERR_TUNNEL_CONNECTION_FAILED` underneath. That text is generic; the proxy's real answer is hidden. Chromium's own source describes the code in one line: "A tunnel connection through the proxy could not be established."

Automation tools report the same thing as `net::ERR_TUNNEL_CONNECTION_FAILED` in Puppeteer, Playwright and Selenium.

## Step 1: see what the proxy actually said

curl prints the status the proxy gave to the tunnel request:

```bash
curl -sS -o /dev/null -w "proxy=%{http_connect} site=%{http_code}\n" \
  -x "http://USER:PASS@HOST:PORT" https://www.example.com/
```

A working proxy prints `proxy=200 site=200`. A refused tunnel ends with a line such as `curl: (56) CONNECT tunnel failed, response 502`. Run it twice: once with the site that fails in Chrome, once with a neutral site such as `https://www.example.com/`.

## Step 2: read the status

| Proxy answer | What it usually means | What to do |
|---|---|---|
| `403` | The proxy will not open a tunnel for you or to that destination | Check the provider's rules for that site and port, and that you are using this IP's own credentials |
| `407` | Login needed or refused | A desktop Chrome shows a sign-in box instead of this error; in automation it surfaces as `ERR_INVALID_AUTH_CREDENTIALS` or a 407. Re-enter USER and PASS |
| `502` | The proxy could not connect onward to the site | Check the site loads at all; retry; try another site to see whether the problem follows the IP |
| `503` | The proxy can't serve the request at the moment | Check that the IP's term hasn't ended and, on per-GB plans, that traffic is left; then retry |
| `504` | The proxy gave up waiting for the site | Retry later, or test whether the site is slow without the proxy too |

## Step 3: decide where the problem is

- **Neutral site works, one site fails:** the proxy is working. That one site, or the route from the proxy to it, is the issue. Settings in Chrome won't change it.
- **Every site fails with the same status:** the problem is on the proxy side. Check the IP's status and expiry on your order page, and send the `curl -v` output to your provider's support, with the password masked.
- **curl works but Chrome fails:** something between Chrome and the proxy differs. Look for a second proxy: a system proxy, a proxy extension, another global network tool, or a company network that intercepts traffic. Turn the extras off and test again.

After fixing a proxy, Chrome may keep avoiding a proxy it has marked as bad. Open `chrome://net-internals/#proxy` and click "Clear bad proxies", or restart Chrome.

## Related proxy errors in Chrome

| Code | Where it failed |
|---|---|
| `ERR_PROXY_CONNECTION_FAILED` | Chrome could not connect to the proxy at all: wrong host or port, or a blocked network |
| `ERR_TUNNEL_CONNECTION_FAILED` | Connected to the proxy; the proxy refused or failed the tunnel |
| `ERR_INVALID_AUTH_CREDENTIALS` | The proxy wanted a login that Chrome could not supply |
| `ERR_SOCKS_CONNECTION_FAILED` | A SOCKS5 proxy failed, often because it needs a password, which Chrome cannot send |
| `ERR_NO_SUPPORTED_PROXIES` | The proxy setting itself is invalid, for example credentials written into `--proxy-server` |
| `ERR_EMPTY_RESPONSE` | The proxy closed the connection without answering, for example an `http://` setting aimed at a SOCKS5-only port |

## FAQ

**Is this a problem with my internet connection?** Usually not. Chrome got as far as the proxy, which needs a working connection. If your connection were down, you would see a different error.

**Why do some sites load and others show this error?** The tunnel is opened per site. A proxy can reach one destination and fail on another, which is why Step 1 tests two sites.

**Will clearing cookies or the cache help?** No. The failure happens before any page data is exchanged.

**Should I change proxy type?** Only if it doesn't match the IP. An IP sold as HTTP goes in as HTTP; Chrome cannot log in to SOCKS5.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs, HTTP or SOCKS5 chosen at purchase, at $0.50 for one day or $7.99 for 30 days per IP, the same in every country. When a term ends the IP is released, so an expired IP stops working; auto-renew can be turned on per IP to avoid that. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
