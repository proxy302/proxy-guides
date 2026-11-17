# How do I use a proxy in Node.js?

For `fetch`, install `undici` and pass `dispatcher: new ProxyAgent('http://USER:PASS@HOST:PORT')` to undici's own `fetch`. On Node 24 (and 22.21+) the built-in `fetch` can instead follow `HTTPS_PROXY` if you set `NODE_USE_ENV_PROXY=1`. For axios, give it an `HttpsProxyAgent` from `https-proxy-agent` as `httpsAgent` and set `proxy: false`; for SOCKS5, use `SocksProxyAgent` from `socks-proxy-agent` with a `socks5h://` URL.

*Proxy302 maintains this guide and sells these proxies. Every snippet ran against an authenticating test proxy on Node 24 in November 2026.*

## fetch with undici

```bash
npm install undici
```

```js
import { fetch, ProxyAgent } from 'undici';

const dispatcher = new ProxyAgent('http://USER:PASS@HOST:PORT');

const res = await fetch('https://ipinfo.io/json', { dispatcher });
console.log(res.status, (await res.json()).ip);
```

Import `fetch` and `ProxyAgent` from the same package. Handing the npm package's agent to Node's built-in `fetch` failed in our test with `invalid onRequestStart method`, because the two undici versions differ.

To proxy every `fetch` in the process from environment variables:

```js
import { setGlobalDispatcher, EnvHttpProxyAgent, fetch } from 'undici';

// Reads HTTP_PROXY, HTTPS_PROXY and NO_PROXY
setGlobalDispatcher(new EnvHttpProxyAgent());
const res = await fetch('https://api.ipify.org');
console.log(await res.text());
```

## Built-in fetch, no packages

```bash
HTTPS_PROXY="http://USER:PASS@HOST:PORT" NODE_USE_ENV_PROXY=1 node app.mjs
```

Without `NODE_USE_ENV_PROXY=1` (or the `--use-env-proxy` flag on Node 24.5+), the built-in `fetch` ignores `HTTPS_PROXY` and connects directly.

## axios

```bash
npm install axios https-proxy-agent
```

```js
import axios from 'axios';
import { HttpsProxyAgent } from 'https-proxy-agent';

const agent = new HttpsProxyAgent('http://USER:PASS@HOST:PORT');

const { data } = await axios.get('https://ipinfo.io/json', {
  httpsAgent: agent,
  proxy: false, // stop axios applying its own proxy option or env vars on top
  timeout: 30000,
});
console.log(data.ip);
```

Recent axios releases also tunnel HTTPS through their own `proxy: { protocol, host, port, auth }` option. The agent route is still handy because switching to SOCKS5 means changing one line.

## SOCKS5

```js
import axios from 'axios';
import { SocksProxyAgent } from 'socks-proxy-agent';

// socks5h: the proxy resolves the hostname
const agent = new SocksProxyAgent('socks5h://USER:PASS@HOST:PORT');

const { data } = await axios.get('https://ipinfo.io/json', {
  httpAgent: agent,
  httpsAgent: agent,
  proxy: false,
});
console.log(data.ip);
```

undici 8 also exports a `Socks5ProxyAgent` for `fetch`; it worked in our test but prints an "experimental" warning.

## Which package for which client

| Client | HTTP proxy | SOCKS5 |
|---|---|---|
| undici `fetch` | `ProxyAgent` | `Socks5ProxyAgent` (experimental) |
| Built-in `fetch` | `NODE_USE_ENV_PROXY=1` + `HTTPS_PROXY` | Use undici instead |
| axios | `https-proxy-agent` or its `proxy` option | `socks-proxy-agent` |
| `https.request()` | `agent: new HttpsProxyAgent(...)` | `agent: new SocksProxyAgent(...)` |

`https-proxy-agent` and `socks-proxy-agent` are ES modules that need Node 20 or newer; undici 8 needs Node 22.19 or newer.

## Common errors

| Error | Cause |
|---|---|
| `TypeError: fetch failed`, cause ending in `Proxy response (407) !== 200 when HTTP Tunneling` | Wrong username or password (undici) |
| `AxiosError: Request failed with status code 407` | Wrong username or password (axios) |
| `fetch failed` with cause `ECONNREFUSED` | Wrong host or port |
| `UND_ERR_INVALID_ARG invalid onRequestStart method` | npm undici agent passed to the built-in `fetch` |
| Requests ignore `HTTPS_PROXY` | Built-in `fetch` without `NODE_USE_ENV_PROXY=1` |

Print `err.cause` to see the real reason; the top-level message is often only "fetch failed".

## FAQ

**Will a rotating gateway give each request a new IP?** Only on a new connection. Agents keep connections alive, so back-to-back requests to one site often share an exit. When you want one exit on purpose, switch to sticky: add the session parameter your provider documents (Proxy302's console generates it for you).

**Special characters in the password?** Percent-encode them with `encodeURIComponent()` before building the URL.

**Which proxy type for API polling?** Rotating residential for many independent requests; a static residential (ISP) IP when the remote side expects one fixed address.

## Where Proxy302 fits

If a Node service polls many public pages, Proxy302's rotating residential line bills $1.50/GB pay as you go. If a partner API expects one known address, a static residential (ISP) IP is $7.99 per 30 days. Either one drops into the agents above as a URL; see the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
