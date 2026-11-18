# Why do I get SSL errors when I use a proxy?

Nearly always for one of two reasons. **First:** the proxy URL starts with `https://`, but the proxy only speaks plain HTTP, so your client tries a TLS handshake with a server that answers in plain text. That shows up as `WRONG_VERSION_NUMBER`, `record layer failure` or `packet length too long`, and the fix is to write `http://HOST:PORT` even when the sites you visit are HTTPS. **Second:** something on the path decrypts and re-signs HTTPS traffic, usually a company network or antivirus product, and your client rejects the certificate it has never seen (`CERTIFICATE_VERIFY_FAILED`, `self-signed certificate`). The fix there is to trust that organisation's CA certificate, not to turn verification off. An ordinary forwarding proxy never touches certificates: TLS runs end to end between you and the site.

*Proxy302 maintains this guide and sells these proxies. The error messages below were produced in November 2026 with Python requests 2.32 / urllib3 2.6, Node.js 24 and curl 8.7 against local test proxies built to fail in each way.*

## Which problem do I have?

| Error text contains | Cause |
|---|---|
| `Your proxy appears to only use HTTP and not HTTPS` (Python, urllib3 2.x) | `https://` proxy URL for a plain HTTP proxy |
| `WRONG_VERSION_NUMBER` or `[SSL] record layer failure` (Python) | Same; the wording depends on the OpenSSL version |
| `ERR_SSL_PACKET_LENGTH_TOO_LONG` (Node.js) | Same |
| `EOF occurred in violation of protocol` while connecting to the proxy | Same, with a proxy that hangs up instead of answering |
| `CERTIFICATE_VERIFY_FAILED ... self signed certificate` (Python) | Someone on the path presents their own certificate |
| `SELF_SIGNED_CERT_IN_CHAIN`, `DEPTH_ZERO_SELF_SIGNED_CERT` or `UNABLE_TO_GET_ISSUER_CERT_LOCALLY` (Node.js, npm) | Same |
| `curl: (60) SSL certificate problem` | Same |
| `curl: (35)` with an SSL message | Often the `https://` mistake; read the message for which |

## Fix 1: use `http://` for the proxy itself

The scheme at the start of a proxy URL says how your program talks *to the proxy*. It has nothing to do with whether the websites are HTTPS. HTTPS sites still travel encrypted, inside a tunnel the proxy opens with `CONNECT`.

```python
import requests

proxy = "http://USER:PASS@HOST:PORT"          # not https://
proxies = {"http": proxy, "https": proxy}     # the https key means "for https:// URLs"

r = requests.get("https://ipinfo.io/json", proxies=proxies, timeout=20)
print(r.json())
```

The same applies to environment variables, where the name causes most of the confusion:

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"   # variable name says https, value says http
```

Write `https://` for a proxy only when the provider documents a TLS-encrypted proxy port. For an IP sold as SOCKS5, the scheme is `socks5h://`.

## Fix 2: find out who is signing the certificate

Look at the certificate your client receives through the proxy. `-k` is used here only so curl shows the details instead of stopping; don't use it in real work.

```bash
curl -sv -k -x "http://USER:PASS@HOST:PORT" https://www.example.com/ -o /dev/null 2>&1 | grep -iE "subject:|issuer:"
```

Through a normal proxy, the issuer is a public certificate authority, the same one you see without the proxy. If the issuer names your employer, a security appliance or your antivirus, that product is intercepting HTTPS.

- **On a managed company network:** ask IT for the CA certificate (a `.pem` or `.crt` file) and point your tools at it, as in the table below.
- **On your own machine:** look for antivirus "HTTPS scanning" or "web shield" options and their documentation for developer tools.
- **If you can't explain the issuer at all:** stop sending passwords over that connection and find out what is on the path before you continue.

| Tool | Trust an extra CA certificate |
|---|---|
| Python requests | `verify="/path/ca-bundle.pem"` or `REQUESTS_CA_BUNDLE`; the file replaces the default list, so include the public CAs too |
| Node.js | `NODE_EXTRA_CA_CERTS=/path/corp-ca.pem`, which adds to the built-in list |
| curl | `--cacert /path/ca-bundle.pem` |
| git | `git config --global http.sslCAInfo /path/ca-bundle.pem` |
| npm | `npm config set cafile /path/ca-bundle.pem` |
| pip | `pip config set global.cert /path/ca-bundle.pem` |

## Why not just turn verification off?

`verify=False`, `-k`, `strict-ssl=false` and `NODE_TLS_REJECT_UNAUTHORIZED=0` make the error vanish by accepting any certificate from anyone. That removes the one check that tells you whether something is reading your traffic, including the passwords and API keys inside it. Fix the trust instead.

## First rule out problems that aren't the proxy

Run the same request without the proxy. If it fails the same way, the proxy is not the cause. Common culprits are a wrong system clock, an expired certificate on the site, or an outdated CA bundle in an old Python environment (`pip install --upgrade certifi`).

## FAQ

**Can the proxy read my HTTPS traffic?** A forwarding proxy sees the hostname and port you connect to, and the amount of data, but not the content. That is only true while certificate checks stay on.

**Why does curl work while Python fails?** They may trust different certificate lists. On macOS, the system curl uses the system keychain, where IT may have installed a company CA; Python's requests uses its own `certifi` bundle, which doesn't include it.

**Does SOCKS5 avoid these errors?** It avoids the `https://` mix-up, since the scheme is `socks5h://`. Interception problems look the same either way.

**Should a Proxy302 static IP be written with `https://`?** No. Each static IP is HTTP or SOCKS5, fixed at purchase, and comes with host, port, username and password, so the URL starts with `http://` or `socks5h://`.

## Where Proxy302 fits

Proxy302 static residential (ISP) IPs are delivered as plain HTTP or SOCKS5, chosen at purchase, so their proxy URL starts with `http://` or `socks5h://`. They cost $0.50 for one day or $7.99 for 30 days each; rotating residential traffic is $1.50/GB on the budget line. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
