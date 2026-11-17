# How do I set a proxy in Playwright?

Give Playwright a `proxy` object with `server`, `username` and `password`, either when you launch the browser (every page uses it) or when you create a browser context (each context gets its own proxy). If the proxy needs a password, use its HTTP form: Playwright rejects SOCKS5 with a username and password and throws "Browser does not support socks5 proxy authentication". SOCKS5 works only without credentials, for example on a proxy that authorises your machine's IP.

*Proxy302 maintains this guide and sells these proxies. The same code works with any provider's host, port, username and password.*

## Whole browser (Python)

```python
from playwright.sync_api import sync_playwright

PROXY = {"server": "http://HOST:PORT", "username": "USER", "password": "PASS"}

with sync_playwright() as p:
    browser = p.chromium.launch(proxy=PROXY)
    page = browser.new_page()
    page.goto("https://ipinfo.io/json")
    print(page.inner_text("body"))
    browser.close()
```

## One proxy per context (Python)

Contexts are separate cookie jars inside one browser. Giving each its own proxy lets you run several accounts or regions without starting several browsers.

```python
from playwright.sync_api import sync_playwright

ACCOUNTS = {
    "shop-a": {"server": "http://HOST:PORT", "username": "USER", "password": "PASS"},
    "shop-b": {"server": "http://HOST:PORT", "username": "USER", "password": "PASS"},
}

with sync_playwright() as p:
    browser = p.chromium.launch()
    for name, proxy in ACCOUNTS.items():
        context = browser.new_context(proxy=proxy)
        page = context.new_page()
        page.goto("https://api.ipify.org")
        print(name, page.inner_text("body"))
        context.close()
    browser.close()
```

## Node.js

```js
import { chromium } from 'playwright';

const browser = await chromium.launch();
const context = await browser.newContext({
  proxy: { server: 'http://HOST:PORT', username: 'USER', password: 'PASS' },
});
const page = await context.newPage();
await page.goto('https://ipinfo.io/json');
console.log(await page.textContent('body'));
await browser.close();
```

In Playwright Test, put the same object under `use: { proxy: {...} }` in `playwright.config.ts`. An optional `bypass` field takes a comma-separated list of hosts that should skip the proxy.

## Where to set it

| Set on | Scope | Typical use |
|---|---|---|
| `launch(proxy=...)` | Every context and page | One exit for the whole run |
| `new_context(proxy=...)` | That context only | One IP per account or per country in one browser |
| `use.proxy` in the test config | All tests | Running a suite from a given country |

A context's proxy is fixed when it is created. To change IP, close the context and open a new one.

## SOCKS5 with a password

Chromium cannot send a username and password to a SOCKS5 proxy, and Playwright raised the same error for Firefox in our test. Your options:

1. Use the HTTP version of the proxy. At Proxy302 the protocol of a static IP is chosen at purchase, so pick HTTP for browser automation.
2. Use a SOCKS5 proxy that authorises by IP and pass only `server`.
3. Run a local forwarder without a password that passes traffic on to the SOCKS5 proxy, as shown in the [Selenium guide](../selenium-proxy/README.md).

## Common errors

| Symptom | Likely cause |
|---|---|
| `Browser does not support socks5 proxy authentication` | `socks5://` plus `username`/`password`; switch to HTTP |
| First `goto` hangs until `Timeout ... exceeded` | The proxy rejected the username or password |
| `net::ERR_PROXY_CONNECTION_FAILED` | Wrong host or port, or nothing listening |
| `net::ERR_RESPONSE_HEADERS_TRUNCATED` | `http://` used for a SOCKS5-only port |
| Page shows your own IP | The proxy was set on a different context from the page you opened |

## FAQ

**Can each page have its own proxy?** No. Pages share their context's proxy, so make one context per proxy.

**Will a context keep one exit on a rotating proxy?** Not reliably, because the browser opens new connections as it goes. Give the context a sticky session instead: the session parameter your provider documents, which Proxy302's console generates for you.

**Should a signed-in account use rotating traffic?** No. Give it a static residential (ISP) IP and keep that IP with the context's saved storage state.

**Do I need headless off?** No. The proxy works the same in headless and headed mode.

## Where Proxy302 fits

For Playwright, order Proxy302 static residential (ISP) IPs with the HTTP protocol so the browser can log in to them. Terms run from 1 day at $0.50 to 90 days at $23.97, and logged-out collection can use rotating residential at $1.50/GB. Details on the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
