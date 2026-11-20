# Why do I get ECONNRESET in Node.js when I send requests through a proxy?

`ECONNRESET` means the TCP connection was torn down from the other side while Node was still using it. Through a proxy, that other side is usually the proxy itself: it reset a keep-alive connection that Node thought was still open, it rejected the client before answering the CONNECT request, or it lost the connection to the target and passed the reset along. The fix is to work out which of the three it is: compare it against the proxy-specific errors that Node libraries raise, drop idle sockets before the proxy does, and retry idempotent requests once or twice. Rarely is the password wrong; a bad password produces a 407, not a reset.

*Proxy302 maintains this guide and sells these proxies. The code was run on Node.js 24.14 with undici 8.12, axios 1.21 and https-proxy-agent 9.1 against a throwaway local proxy in November 2026.*

## How each library reports a proxy problem

The same fault looks different depending on the client. These are the messages we reproduced by making a test proxy misbehave in three ways:

| What the proxy did | undici `fetch` + `ProxyAgent` (`err.cause`) | axios + `https-proxy-agent` |
|---|---|---|
| Sent a TCP reset | `ECONNRESET` (`read ECONNRESET`) | `ECONNRESET` |
| Closed the connection without replying to CONNECT | `UND_ERR_PRX_CONN` (`Proxy Connection failed`) | `Proxy connection ended before receiving CONNECT response` |
| Replied with an error status, such as 407 | `UND_ERR_ABORTED` (`Proxy response (407) !== 200 when HTTP Tunneling`) | the status, for example `407` |

With undici, the top-level error is always `TypeError: fetch failed`. Log `err.cause` to see the real reason:

```js
try {
  await fetch(url, { dispatcher });
} catch (err) {
  console.error(err.cause?.code, err.cause?.message);
}
```

## The four usual causes

1. **A stale keep-alive socket.** Node keeps tunnels open to reuse them. If the proxy closes an idle tunnel first and Node writes to it a moment later, the write is answered with a reset. This shows up as occasional `ECONNRESET` after a pause in traffic, never on the first request.
2. **The proxy refused the client.** Some proxies drop or reset connections from addresses that are not on their allowlist, or for an IP whose term has ended. These fail every time, from the first request.
3. **Wrong protocol on the port.** Sending an HTTP CONNECT to a SOCKS5 port (or the reverse) makes the proxy hang up. At Proxy302, HTTP or SOCKS5 is chosen at purchase, one protocol per IP. For SOCKS5 in Node use `socks-proxy-agent` (see [the Node.js proxy guide](../nodejs-proxy/README.md)).
4. **The target reset the connection.** The proxy forwards whatever the site does. If the site drops the connection, Node sees a reset inside the tunnel. Test the same URL with another site through the same proxy to tell the two apart.

## Expire idle sockets early, and retry

This undici setup closes idle sockets after 4 seconds, before most proxies would, and retries a failed request twice with a short back-off. `RetryAgent` only retries idempotent methods (GET, HEAD, PUT, DELETE and similar) by default, so a POST is never sent twice by accident.

```js
// npm install undici
import { fetch, ProxyAgent, RetryAgent } from 'undici';

const proxy = new ProxyAgent({
  uri: process.env.PROXY_URL,          // http://USER:PASS@HOST:PORT
  keepAliveTimeout: 4_000,             // drop idle sockets before the proxy does
  keepAliveMaxTimeout: 4_000,
});
const dispatcher = new RetryAgent(proxy, { maxRetries: 2, minTimeout: 500 });

try {
  const res = await fetch('https://ipinfo.io/ip', { dispatcher });
  console.log(res.status, (await res.text()).trim());
} catch (err) {
  console.error('failed:', err.cause?.code ?? err.cause?.message ?? err.message);
}
```

Run it with `PROXY_URL=http://USER:PASS@HOST:PORT node check.mjs`. A healthy proxy prints `200` and the exit address. If the proxy resets every attempt, it prints `failed: ECONNRESET` after the retries, which tells you the problem is cause 2, 3 or 4, not a stale socket.

### With axios

axios does not retry on its own, so wrap the call and retry only on a reset:

```js
// npm install axios https-proxy-agent
import axios from 'axios';
import { HttpsProxyAgent } from 'https-proxy-agent';

const httpsAgent = new HttpsProxyAgent(process.env.PROXY_URL, { keepAlive: true });

async function getWithRetry(url, tries = 3) {
  for (let i = 1; ; i++) {
    try {
      return await axios.get(url, { httpsAgent, proxy: false, timeout: 20_000 });
    } catch (err) {
      if (i >= tries || err.code !== 'ECONNRESET') throw err;
      await new Promise((r) => setTimeout(r, 500 * i));
    }
  }
}

const res = await getWithRetry('https://ipinfo.io/ip');
console.log(res.status, res.data);
```

`proxy: false` stops axios from applying its own proxy handling on top of the agent.

## When retries are the wrong answer

| Pattern | What it points to | What to do |
|---|---|---|
| Fails on the very first request, every time | Client refused (allowlist, ended term) or wrong protocol | Test with `curl -v -x http://USER:PASS@HOST:PORT https://ipinfo.io/ip` |
| Fails only after the script has been idle | Stale keep-alive socket | Shorter `keepAliveTimeout`, plus a retry |
| Fails only for one website | The site or its firewall resets the connection | Check the site's rate limits and terms; slow down |
| Fails under high concurrency | Too many parallel tunnels | Lower concurrency and reuse one dispatcher |

## FAQ

**Is `socket hang up` the same thing?** Close enough to treat the same way. Node's `http` module reports `socket hang up` when the connection closes before a response arrives, and gives that error the code `ECONNRESET` even when the close was a clean one.

**Does disabling keep-alive fix it?** It removes the stale-socket case at the cost of a new tunnel per request. On a rotating gateway that also means a new exit IP per request, which may or may not be what you want.

**Should I retry POST requests?** Only if the API supports idempotency keys or you can check whether the first attempt went through. Otherwise you risk sending the same order or message twice.

**Why does `fetch` say "fetch failed" with no detail?** That is undici's wrapper. The detail is always in `err.cause`.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs as HTTP or SOCKS5, from $0.50 for a day or $7.99 for 30 days per IP, and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/).

*Facts checked 20 November 2026.*
