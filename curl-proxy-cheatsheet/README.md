# curl proxy cheat sheet: HTTP, HTTPS, SOCKS5, auth, checking the exit IP

Use `-x` (long form `--proxy`) with a full proxy URL: `curl -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json` for an HTTP proxy, or the same with `socks5h://` for SOCKS5 with DNS resolved at the proxy. Add `-U USER:PASS` instead if you'd rather keep credentials out of the URL. Fetching `https://ipinfo.io/json` through the proxy shows the exit IP, its country and the network it belongs to, which is the quickest check that the proxy works.

*Proxy302 maintains this guide and sells these proxies. Every command below was run with curl 8.7 against an authenticating test proxy.*

## The commands

```bash
# HTTP proxy, credentials in the URL
curl -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json

# HTTP proxy, credentials kept separate (-U is short for --proxy-user)
curl -x "http://HOST:PORT" -U "USER:PASS" https://ipinfo.io/json

# SOCKS5, the proxy resolves the hostname
curl -x "socks5h://USER:PASS@HOST:PORT" https://ipinfo.io/json
curl --socks5-hostname "HOST:PORT" -U "USER:PASS" https://ipinfo.io/json

# SOCKS5, your machine resolves the hostname
curl -x "socks5://USER:PASS@HOST:PORT" https://ipinfo.io/json

# Exit IP only
curl -s -x "http://USER:PASS@HOST:PORT" https://api.ipify.org; echo

# Proxy status, site status and timings in one line
curl -s -o /dev/null -x "http://USER:PASS@HOST:PORT" \
  -w "proxy=%{http_connect} site=%{http_code} connect=%{time_connect}s tls=%{time_appconnect}s total=%{time_total}s\n" \
  https://example.com

# Watch the CONNECT handshake
curl -v -o /dev/null -x "http://USER:PASS@HOST:PORT" https://example.com

# Bytes downloaded by one request
curl -s -o /dev/null -x "http://USER:PASS@HOST:PORT" -w "%{size_download}\n" https://example.com
```

`%{http_connect}` is the proxy's own answer to CONNECT: 200 when it accepted you, 407 when it did not. `%{http_code}` is the website's answer. Keeping them apart tells you which side failed.

The `-v` output includes a `Proxy-Authorization` header, which is your username and password in base64. Don't paste it into tickets or chats.

## Environment variables

```bash
export http_proxy="http://USER:PASS@HOST:PORT"   # lowercase only for http://
export https_proxy="http://USER:PASS@HOST:PORT"  # HTTPS_PROXY also works
export no_proxy="localhost,127.0.0.1"
curl https://ipinfo.io/json            # uses https_proxy
curl --noproxy "*" https://ipinfo.io/ip  # skips the proxy once
```

curl ignores uppercase `HTTP_PROXY` on purpose; we confirmed it connects directly. `ALL_PROXY` (or `all_proxy`) applies to every scheme and accepts `socks5h://` URLs.

## Proxy URL schemes

| Scheme | Proxy type | DNS lookup |
|---|---|---|
| `http://` | HTTP proxy; HTTPS runs through a CONNECT tunnel | At the proxy |
| `https://` | HTTPS proxy: TLS to the proxy itself (rare; only if your provider offers it) | At the proxy |
| `socks5://` | SOCKS5 | On your machine |
| `socks5h://` | SOCKS5 | At the proxy |
| `socks4://` / `socks4a://` | SOCKS4, no password support | Local / at the proxy |

## Common errors

| curl says | Meaning |
|---|---|
| `(56) CONNECT tunnel failed, response 407` | Wrong username or password |
| `(97) User was rejected by the SOCKS5 server (1 1).` | SOCKS5 credentials refused |
| `(7) Failed to connect to HOST port PORT ... Couldn't connect to server` | Wrong port, firewall, or proxy down |
| `(5) Could not resolve proxy: HOST` | Typo in the host name |
| `(5) Unsupported proxy syntax ... Port number was not a decimal number` | Placeholder left in, or a bad port |
| `(56) Proxy CONNECT aborted` | `http://` used on a SOCKS5-only port |
| Hangs until `(28)` timeout, or a SOCKS or TLS error | `socks5h://` or `https://` used on a port that speaks something else |

Exit codes follow the number in brackets, so scripts can branch on `$?`.

## FAQ

**Is `HTTPS_PROXY` an HTTPS proxy?** No. It names the proxy used for `https://` URLs, and its value usually starts with `http://`.

**Where can I set a proxy permanently?** Add a line such as `proxy = "http://USER:PASS@HOST:PORT"` to `~/.curlrc`, and remember every curl command will use it.

**What about Windows?** In Windows PowerShell, `curl` can be an alias for `Invoke-WebRequest`; type `curl.exe` to get real curl with the same flags.

**Why does the city differ between lookup sites?** IP databases disagree on cities. Judge by country.

## Where Proxy302 fits

Each Proxy302 static residential (ISP) IP speaks either HTTP or SOCKS5, chosen at purchase, so use the matching scheme from the table. A one-day IP is $0.50; the [pricing page](https://proxy302.com/pricing/) lists the other terms and the per-GB rotating lines.

*Facts checked 17 November 2026.*
