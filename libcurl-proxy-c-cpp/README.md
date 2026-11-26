# How do I use a proxy with a username and password in libcurl (C or C++)?

Set `CURLOPT_PROXY` to the proxy's scheme, host and port (`http://HOST:PORT` for an HTTP proxy, `socks5h://HOST:PORT` for SOCKS5 with DNS through the proxy), and pass the credentials separately with `CURLOPT_PROXYUSERNAME` and `CURLOPT_PROXYPASSWORD`. Setting them separately means a password with `@`, `:` or `#` in it needs no URL-encoding. After a failure, read `CURLINFO_HTTP_CONNECTCODE`: a `407` there means the proxy refused the username or password. Proxy302 maintains this guide and sells these proxies; the example below was compiled and run against libcurl 8.7.1 on macOS on 26 November 2026, through a local test proxy that requires a username and password.

## Complete example

```c
/* proxy_get.c: fetch a URL through an authenticated HTTP or SOCKS5 proxy with libcurl. */
#include <stdio.h>
#include <stdlib.h>
#include <curl/curl.h>

int main(void)
{
    const char *proxy = getenv("PROXY_URL");      /* e.g. http://HOST:PORT or socks5h://HOST:PORT */
    const char *user  = getenv("PROXY_USER");
    const char *pass  = getenv("PROXY_PASS");
    if (!proxy || !user || !pass) {
        fprintf(stderr, "set PROXY_URL, PROXY_USER and PROXY_PASS\n");
        return 2;
    }

    curl_global_init(CURL_GLOBAL_DEFAULT);
    CURL *curl = curl_easy_init();
    if (!curl) return 1;

    curl_easy_setopt(curl, CURLOPT_URL, "https://httpbin.org/ip");
    curl_easy_setopt(curl, CURLOPT_PROXY, proxy);
    curl_easy_setopt(curl, CURLOPT_PROXYUSERNAME, user);   /* no URL-encoding needed here */
    curl_easy_setopt(curl, CURLOPT_PROXYPASSWORD, pass);
    curl_easy_setopt(curl, CURLOPT_CONNECTTIMEOUT, 10L);
    curl_easy_setopt(curl, CURLOPT_TIMEOUT, 30L);

    CURLcode rc = curl_easy_perform(curl);
    if (rc != CURLE_OK) {
        long proxy_code = 0;
        curl_easy_getinfo(curl, CURLINFO_HTTP_CONNECTCODE, &proxy_code);
        fprintf(stderr, "curl error %d: %s (proxy CONNECT status %ld)\n",
                (int)rc, curl_easy_strerror(rc), proxy_code);
    } else {
        long code = 0;
        curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &code);
        fprintf(stderr, "HTTP %ld\n", code);
    }

    curl_easy_cleanup(curl);
    curl_global_cleanup();
    return rc == CURLE_OK ? 0 : 1;
}
```

Build and run:

```bash
cc -Wall -Wextra -o proxy_get proxy_get.c -lcurl
PROXY_URL='http://HOST:PORT' PROXY_USER='USER' PROXY_PASS='PASS' ./proxy_get
```

On Debian or Ubuntu, install the headers first (`libcurl4-openssl-dev`); on macOS the system libcurl is enough. The same calls work unchanged from C++.

What we saw in the test (the origin is replaced by a documentation address):

```text
HTTP 200
{
  "origin": "203.0.113.24"
}
```

and with a wrong password:

```text
curl error 56: Failure when receiving data from the peer (proxy CONNECT status 407)
```

The `origin` is the exit IP the website sees. For a static IP it should be the same on every run.

## The options that matter

| Option | What it does | Notes |
|---|---|---|
| `CURLOPT_PROXY` | Proxy scheme, host and port | `http://`, `https://` (TLS to the proxy itself), `socks5://`, `socks5h://`. No scheme means HTTP |
| `CURLOPT_PROXYUSERNAME` / `CURLOPT_PROXYPASSWORD` | Proxy credentials | Taken as-is, so special characters are fine |
| `CURLOPT_PROXYUSERPWD` | `"user:password"` in one string | Older style; a colon in the username breaks it |
| `CURLOPT_NOPROXY` | Hosts that skip the proxy | Comma-separated, e.g. `"localhost,127.0.0.1,.internal.example"` |
| `CURLOPT_PROXYTYPE` | Proxy type when the URL has no scheme | Usually unnecessary if you write the scheme |
| `CURLINFO_HTTP_CONNECTCODE` | Status the proxy answered to `CONNECT` | 407 = proxy authentication, 403 or 502 = proxy refused the target |

If you prefer one string, `CURLOPT_PROXY` also accepts `http://USER:PASS@HOST:PORT`, but then the username and password must be percent-encoded (`@` becomes `%40`, `:` becomes `%3A`). Use `curl_easy_escape()` for that, or avoid the problem with the two separate options.

## HTTP or SOCKS5?

- **HTTP proxy:** HTTPS requests go through a `CONNECT` tunnel, so the proxy never sees the page content. Plain `http://` URLs are forwarded as normal proxy requests.
- **SOCKS5 proxy:** use `socks5h://` so the hostname is resolved by the proxy, not by your machine. With plain `socks5://`, your local DNS resolver sees every hostname you visit.

With Proxy302 the protocol is chosen when you buy a static IP, and one IP has one protocol. Use the scheme that matches the order page.

## Environment variables

If you do not set `CURLOPT_PROXY` at all, libcurl reads `http_proxy`, `https_proxy`, `ALL_PROXY` and `NO_PROXY` from the environment. That is convenient for small tools but surprising in services, because the proxy then depends on how the program was started. Two ways to stay explicit:

```c
curl_easy_setopt(curl, CURLOPT_PROXY, "http://HOST:PORT"); /* always this proxy */
curl_easy_setopt(curl, CURLOPT_PROXY, "");                 /* never a proxy, even if env vars are set */
```

Note that libcurl only honours the lowercase `http_proxy` for plain HTTP URLs; the uppercase form is ignored there for security reasons.

## Errors and what they mean

| Result | Meaning | What to check |
|---|---|---|
| `CURLE_RECV_ERROR` (56) with connect code 407 | Proxy rejected the credentials | Username and password; trailing spaces from copy and paste |
| `CURLE_OK` but `CURLINFO_RESPONSE_CODE` 407 on an `http://` URL | Same, on a non-tunnelled request | As above |
| `CURLE_COULDNT_CONNECT` (7) | Cannot reach the proxy host and port | Host, port, local firewall |
| `CURLE_PROXY` (97) | Proxy handshake failed | Often a protocol mix-up: `socks5h://` pointed at an HTTP proxy, or the reverse |
| `CURLE_OPERATION_TIMEDOUT` (28) | No answer in time | A firewall dropping traffic, or the same protocol mix-up: in our test, `socks5h://` pointed at an HTTP proxy port simply hung until the timeout |
| `CURLE_PEER_FAILED_VERIFICATION` (60) | TLS certificate problem | A device in the path is inspecting HTTPS; see [SSL errors with a proxy](../proxy-ssl-errors/README.md) |

For more detail while debugging, add `curl_easy_setopt(curl, CURLOPT_VERBOSE, 1L);`. It prints the `CONNECT` request and the proxy's answer. The option reference is on curl's site: [CURLOPT_PROXY](https://curl.se/libcurl/c/CURLOPT_PROXY.html).

## Several requests on one handle

Reuse the same easy handle for a series of requests: libcurl keeps the connection to the proxy open and reuses it. On a static IP the exit address is the same either way; on a rotating proxy, a reused connection usually keeps the same exit, while a new connection may get a new one.

## FAQ

**Does this work with C++ wrappers such as cpr or curlpp?** They call the same libcurl options underneath. Look for their proxy and proxy-authentication settings, or fall back to the raw handle.

**Do I need `CURLOPT_PROXYAUTH`?** Only if the proxy asks for something other than Basic. Basic is the default and is what username-and-password proxies use.

**Is the password sent in clear text?** With an `http://` proxy, the `Proxy-Authorization` header travels unencrypted to the proxy, although the HTTPS content inside the tunnel stays encrypted. If that matters on your network, use an `https://` proxy URL when the provider supports it.

**Which proxy type should I buy for an API client?** If the remote API allowlists source addresses, a static IP. See [which proxy for an allowlisted API](../fixed-ip-for-api-allowlist/README.md).

## Where Proxy302 fits

Proxy302 sells static ISP IPs (HTTP or SOCKS5, chosen at purchase; $0.50 for one day, $7.99 for 30 days, the same in every country) and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/). For the command-line tool, see the [curl proxy cheat sheet](../curl-proxy-cheatsheet/README.md).

*Facts checked 26 November 2026.*
