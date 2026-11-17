# How do I use an authenticated proxy with Puppeteer?

Launch the browser with `--proxy-server=http://HOST:PORT`, keeping the username and password out of that flag, then call `await page.authenticate({ username: 'USER', password: 'PASS' })` on the page before `page.goto()`. Chrome then answers the proxy's 407 challenge with those credentials. This only works with HTTP proxies: Chrome cannot log in to a SOCKS5 proxy, so a password-protected SOCKS5 proxy needs either an HTTP version of the same proxy or a local forwarder.

*Proxy302 maintains this guide and sells these proxies; the code is the same for any provider.*

## One proxy for the whole browser

```js
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch({
  args: ['--proxy-server=http://HOST:PORT'],
});
const page = await browser.newPage();
await page.authenticate({ username: 'USER', password: 'PASS' });
await page.goto('https://ipinfo.io/json', { waitUntil: 'domcontentloaded' });
console.log(await page.evaluate(() => document.body.innerText));
await browser.close();
```

## A different proxy per browser context

`createBrowserContext()` (the name since Puppeteer 22; earlier versions called it `createIncognitoBrowserContext`) takes its own `proxyServer`. Each context keeps separate cookies, so this is how one browser can hold several accounts on several IPs.

```js
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch();
const context = await browser.createBrowserContext({
  proxyServer: 'http://HOST:PORT',
});
const page = await context.newPage();
await page.authenticate({ username: 'USER', password: 'PASS' });
await page.goto('https://api.ipify.org');
console.log(await page.evaluate(() => document.body.innerText));
await context.close();
await browser.close();
```

Both snippets ran against an authenticating test proxy with Puppeteer 25 and current Chrome. `puppeteer-core` has the same API; you only add `executablePath`.

## What goes where

| Piece | Where | Note |
|---|---|---|
| Host and port | `--proxy-server` flag or `proxyServer` option | Scheme `http://`; no `USER:PASS@` here |
| Username and password | `page.authenticate()` | Once per page, before the first navigation |
| New tabs and popups | Call `authenticate()` on them too | Credentials belong to the page, not the browser |
| Hosts that skip the proxy | `--proxy-bypass-list=` flag or `proxyBypassList` option | Semicolon-separated in the flag |

The Puppeteer docs note that `authenticate()` switches on request interception behind the scenes, which might affect performance.

## Common errors

| Error | Cause and fix |
|---|---|
| `net::ERR_NO_SUPPORTED_PROXIES` | Credentials were put inside `--proxy-server`; move them to `authenticate()` |
| `net::ERR_INVALID_AUTH_CREDENTIALS` | `authenticate()` was not called on this page |
| `goto` returns a response with status 407 | Wrong username or password |
| `net::ERR_SOCKS_CONNECTION_FAILED` | A SOCKS5 proxy that requires a password; Chrome cannot send one |
| `net::ERR_PROXY_CONNECTION_FAILED` | Chrome could not reach the proxy at that address |

## Using a SOCKS5 proxy anyway

Run a forwarder on your own machine that has no password and passes traffic to the SOCKS5 proxy with credentials. The `proxy-chain` npm package does this in a few lines:

```js
import puppeteer from 'puppeteer';
import { anonymizeProxy, closeAnonymizedProxy } from 'proxy-chain';

const local = await anonymizeProxy('socks5://USER:PASS@HOST:PORT');
const browser = await puppeteer.launch({ args: [`--proxy-server=${local}`] });
// ... use the browser as usual, no authenticate() needed
await browser.close();
await closeAnonymizedProxy(local, true);
```

## FAQ

**Can I change proxy without restarting Chrome?** Yes, at the context level: close the context and create another with a different `proxyServer`.

**Do all browser requests count as proxy traffic?** Yes. Everything the browser loads goes through the proxy, including background requests branded Chrome makes on its own, which matters when you pay per GB.

**One IP per account?** Yes: one context, one static IP, kept for as long as the account exists.

## Where Proxy302 fits

Pick HTTP when you order a Proxy302 static residential (ISP) IP for Puppeteer, because the protocol is fixed per IP at purchase. A 7-day IP costs $3.50 and a 30-day IP $7.99; rotating residential traffic is $1.50/GB, less when prepaid. See the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
