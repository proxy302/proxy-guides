# How do I send Postman requests through a proxy with a username and password?

In the Postman desktop app, open **Settings**, go to the **Proxy** tab, turn on **Use custom proxy configuration**, tick the request types (HTTP, HTTPS or both) that should use it, enter the proxy host and port, then switch on **Proxy auth** and fill in the username and password. Every request you send from the app then leaves through the proxy until you turn it off. Send `GET https://ipinfo.io/json` to confirm the exit address. For Newman runs in CI, set the `HTTPS_PROXY` and `HTTP_PROXY` environment variables instead.

*Proxy302 maintains this guide and sells these proxies. Checked against Postman's proxy documentation and the Newman README in November 2026, not run.*

## Settings, field by field

Postman moves labels around between versions, so match these by meaning if your screen differs.

| Setting | What to enter |
|---|---|
| Use custom proxy configuration | On |
| Proxy type ("use proxy for") | HTTP and HTTPS, both ticked, unless you want only one kind of URL proxied |
| Proxy server | `HOST` and `PORT` from your order |
| Proxy auth | On; `USER` and `PASS` |
| Proxy bypass | Hosts that should go direct, comma-separated, for example `localhost,127.0.0.1,*.internal.example` |

The proxy server type list in recent versions includes HTTP and SOCKS options (SOCKS4, SOCKS4A, SOCKS5, SOCKS5H). Postman's documentation does not say whether a SOCKS login (username and password) is supported, so if your proxy needs a password, use an HTTP proxy. At Proxy302, HTTP or SOCKS5 is chosen per IP at purchase.

The same tab also has **Use system proxy**, with an option to respect the `HTTP_PROXY`, `HTTPS_PROXY` and `NO_PROXY` environment variables. Use one approach at a time: a system proxy and a custom proxy both switched on make it hard to tell which one carried a request.

Do not confuse these settings with Postman's capture proxy (the one that records traffic from a browser or phone into a collection). That proxy receives requests; this guide is about sending them.

## Check that it works

1. Create a request: `GET https://ipinfo.io/json`.
2. Send it. The `ip` and `country` in the response should be the proxy's, not yours.
3. Open the Postman Console (bottom bar) to see each request's details if something looks wrong.

To compare with and without the proxy, switch **Use custom proxy configuration** off and send again.

## Newman and CI

Newman, Postman's open-source collection runner, does not read the app's settings. Its README documents the standard variables:

```bash
export HTTPS_PROXY=http://USER:PASS@HOST:PORT
export HTTP_PROXY=http://USER:PASS@HOST:PORT
export NO_PROXY=localhost,127.0.0.1
newman run collection.json -e environment.json
```

Percent-encode special characters in the password (`p@ss` becomes `p%40ss`). In CI, store the full proxy URL as a secret rather than in the pipeline file.

## Errors

| Response or error | Cause |
|---|---|
| `407 Proxy Authentication Required` | Proxy auth is off, or the username or password is wrong |
| `Error: tunneling socket could not be established` | Postman could not open the CONNECT tunnel: wrong host or port, a SOCKS port used as HTTP, or the proxy refused the target |
| `Error: connect ECONNREFUSED` | Nothing is listening on that host and port |
| Response shows your own IP | The custom proxy is off, the request type (HTTP or HTTPS) is not ticked, or the host is in the bypass list |
| Certificate errors only with the proxy on | Something on your network intercepts TLS; see [proxy SSL errors](../proxy-ssl-errors/README.md) |

## FAQ

**Can different requests use different proxies?** Not from the settings screen; the custom proxy applies app-wide. Run separate Newman jobs with different environment variables if you need that.

**Why test an API through a proxy at all?** Typical reasons: the API allowlists a fixed address, it returns country-specific results, or you need to reproduce what a customer in another country sees. For allowlists see [a fixed IP for an API allowlist](../fixed-ip-for-api-allowlist/README.md).

**Will each request get a new IP on a rotating gateway?** Postman may reuse connections, so consecutive requests can leave through the same exit. For a fixed exit across a whole test run, use a static IP.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs as HTTP or SOCKS5, at $0.50 for one day or $7.99 for 30 days per IP, and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/).

*Facts checked 20 November 2026.*
