# How do I use a proxy with Selenium?

For Chrome or Edge, add `--proxy-server=http://HOST:PORT` to the browser options. That is all you need when the proxy has no password, for example when your provider authorises your machine's IP. Chrome has no option that takes a proxy username and password, so for a password-protected proxy the dependable route today is a small local forwarder that holds the credentials, with Selenium pointed at it. selenium-wire, the old answer, is archived, and Chrome cannot log in to SOCKS5 at all, although the forwarder can.

*Proxy302 maintains this guide and sells these proxies. The forwarder, the BiDi handler and the error cases below were tested on 17 November 2026 with Selenium 4.52 and Chrome 157.*

## Proxy without a password

```python
from selenium import webdriver

options = webdriver.ChromeOptions()
options.add_argument("--proxy-server=http://HOST:PORT")

driver = webdriver.Chrome(options=options)
try:
    driver.get("https://ipinfo.io/json")
    print(driver.find_element("tag name", "body").text)
finally:
    driver.quit()
```

## Proxy with a username and password: local forwarder

Start a forwarder on your own machine. It listens on `127.0.0.1:8000` without a password and passes everything on to your proxy, adding the credentials. This one uses the `proxy-chain` npm package (Node 20.11 or newer):

```bash
npm install proxy-chain
```

```js
// forward.mjs  (run: node forward.mjs, leave it running)
import { anonymizeProxy } from 'proxy-chain';

const local = await anonymizeProxy({ url: 'http://USER:PASS@HOST:PORT', port: 8000 });
console.log('Point the browser at', local);
```

Then point Selenium at the forwarder:

```python
from selenium import webdriver

options = webdriver.ChromeOptions()
options.add_argument("--proxy-server=http://127.0.0.1:8000")

driver = webdriver.Chrome(options=options)
try:
    driver.get("https://ipinfo.io/json")
    print(driver.find_element("tag name", "body").text)
finally:
    driver.quit()
```

For a SOCKS5 proxy with a password, change the forwarder URL to `socks5://USER:PASS@HOST:PORT`; Chrome still talks plain HTTP to `127.0.0.1:8000`. HTTPS stays encrypted end to end, because the forwarder only relays the CONNECT tunnel.

## Options compared

| Option | Proxy with a password? | Status |
|---|---|---|
| `--proxy-server` with an IP-authorised proxy | Not needed | Works; simplest when your provider offers it |
| Local forwarder (`proxy-chain`) | Yes, for an HTTP or SOCKS5 proxy | Worked in our test; one extra process |
| `driver.network.add_auth_handler()` (WebDriver BiDi) | In principle | In our test it timed out on the proxy's 407 |
| Extension that answers the login | Fragile | Branded Chrome 137+ no longer loads extensions from the `--load-extension` switch |
| selenium-wire | Yes | Archived by its maintainer; no fixes for new browsers |
| `USER:PASS@` inside `--proxy-server` | No | Chrome refuses it |

## Common errors

| Symptom | Cause |
|---|---|
| `ERR_NO_SUPPORTED_PROXIES` on the page | Credentials were put inside `--proxy-server` |
| Blank page, nothing loads | The proxy wants a password and nothing answered it |
| `unknown error: net::ERR_PROXY_CONNECTION_FAILED` | The forwarder is not running, or the port is wrong |
| `Timed out waiting for response to BiDi command` | `add_auth_handler()` waiting on a proxy login |
| `EADDRINUSE` when starting the forwarder | Port 8000 is taken; pick another and update `--proxy-server` |

## FAQ

**Can each driver use a different IP?** Yes. Run one forwarder per proxy on different ports (8000, 8001, ...) and give each driver its own `--proxy-server`.

**Does the forwarder slow things down?** It adds a hop on your own machine, which is small next to the trip to the proxy.

**Firefox?** Point Firefox's `network.proxy.*` preferences at the same forwarder, and it never needs to show a login prompt.

**Which product for logged-in sessions?** A static residential (ISP) IP per account, kept for the life of that account. Rotating traffic suits logged-out page collection.

## Where Proxy302 fits

Any Proxy302 proxy can sit behind the forwarder above: a static residential (ISP) IP (one day for $0.50, 60 days for $15.98) or rotating residential traffic on the $1.50/GB budget line or the $5/GB premium line. Prices are on the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
