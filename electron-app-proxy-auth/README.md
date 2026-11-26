# How do I make an Electron app use a proxy with a username and password?

Call `session.setProxy({ proxyRules: 'http://HOST:PORT' })` on the session your windows use, and answer the proxy's login challenge in the `app` `'login'` event: check `authInfo.isProxy`, call `event.preventDefault()`, then `callback(USER, PASS)`. Electron cannot take the username and password inside the proxy address or the `--proxy-server` switch, and, being Chromium underneath, it does not do SOCKS5 with a username and password, so use an HTTP proxy. Proxy302 maintains this guide and sells these proxies: an HTTP static ISP IP costs $0.50 for one day or $7.99 for 30 days, the same in every country. The code follows Electron's API documentation ([session](https://www.electronjs.org/docs/latest/api/session), [app](https://www.electronjs.org/docs/latest/api/app)) as of November 2026; we did not run an Electron build for this page.

## Minimal main process

```js
// main.js
const { app, BrowserWindow, session } = require('electron')

const PROXY = 'http://HOST:PORT'          // no credentials here
const PROXY_USER = process.env.PROXY_USER // USER
const PROXY_PASS = process.env.PROXY_PASS // PASS

// Answer proxy authentication for windows and utility processes.
app.on('login', (event, webContents, details, authInfo, callback) => {
  if (authInfo.isProxy) {
    event.preventDefault()               // the default is to cancel
    callback(PROXY_USER, PROXY_PASS)
  }
})

app.whenReady().then(async () => {
  await session.defaultSession.setProxy({
    proxyRules: PROXY,
    proxyBypassRules: '<local>'          // localhost, 127.0.0.1 and ::1 go direct
  })

  // Optional: confirm which proxy Chromium will use for a URL.
  console.log(await session.defaultSession.resolveProxy('https://ipinfo.io/json'))

  const win = new BrowserWindow({ width: 900, height: 700 })
  await win.loadURL('https://ipinfo.io/json')   // should show the proxy's IP and country
})
```

Run it with the credentials in the environment rather than in the source:

```bash
PROXY_USER='USER' PROXY_PASS='PASS' npx electron .
```

`resolveProxy` resolves to a string such as `PROXY HOST:PORT` when the rule applies, and `DIRECT` when it does not.

## The pieces, and what each one does

| API | Role | Notes from the Electron docs |
|---|---|---|
| `ses.setProxy(config)` | Sets the proxy for one session | Returns a Promise. Connections already open may keep the old route; call `ses.closeAllConnections()` if you switch proxies at runtime |
| `proxyRules` | Which proxy to use | `http://HOST:PORT` for everything, or per scheme: `http=HOST:PORT;https=HOST:PORT` |
| `proxyBypassRules` | Hosts that go direct | Comma-separated; `<local>` matches localhost, 127.0.0.1 and ::1 |
| `app` event `'login'` | Supplies credentials | Fired for `webContents` and utility processes; `authInfo.isProxy` tells proxy auth apart from website auth |
| `request.on('login')` on `net.request` | Credentials for requests made with the `net` module | `callback(username, password)`; empty credentials cancel the request |
| `--proxy-server` switch | Proxy from the command line | Overrides the system setting; the URL cannot carry a username and password |
| `app.setProxy(config)` | Proxy for requests that have no session | After `ready`; covers `net` requests in the utility process and some runtime-internal requests |

## A different proxy per window

Each session can have its own proxy. Partitions give you separate sessions with separate cookies and storage:

```js
const { BrowserWindow, session } = require('electron')

async function openWindowWithProxy(partition, proxyUrl) {
  const ses = session.fromPartition(partition)   // e.g. 'persist:profile-a'
  await ses.setProxy({ proxyRules: proxyUrl, proxyBypassRules: '<local>' })
  const win = new BrowserWindow({ webPreferences: { session: ses } })
  await win.loadURL('https://ipinfo.io/json')
  return win
}
```

The single `app.on('login')` handler serves every session. If different proxies need different credentials, look at `authInfo.host` and `authInfo.port` and return the matching pair.

## Why not SOCKS5?

Chromium, and therefore Electron, supports `socks5://` in `proxyRules` but not SOCKS5 with a username and password. Your options with an authenticated proxy:

1. **Use an HTTP proxy** (simplest). With Proxy302, the protocol is chosen when you buy a static IP; pick HTTP for Electron.
2. **Use a proxy that authenticates by allowlisted IP** instead of a password, if your provider offers it.
3. **Run a small local forwarder** that accepts unauthenticated connections on `127.0.0.1` and logs in to your SOCKS5 proxy for you, then point `proxyRules` at the forwarder.

## Installing Electron itself behind a proxy

`npm install electron` downloads a large binary with its own downloader (`@electron/get`). For an HTTP proxy, Electron's installation docs say to set `ELECTRON_GET_USE_PROXY` plus the environment variables of the `global-agent` package:

```bash
export ELECTRON_GET_USE_PROXY=1
export GLOBAL_AGENT_HTTPS_PROXY='http://USER:PASS@HOST:PORT'
npm install electron --save-dev
```

`global-agent` accepts only `http://` proxy URLs and reads credentials from them; percent-encode special characters in the password. If `GLOBAL_AGENT_HTTPS_PROXY` is not set, it uses `GLOBAL_AGENT_HTTP_PROXY` for both.

## Troubleshooting

| What you see | Likely cause | Fix |
|---|---|---|
| `ERR_PROXY_CONNECTION_FAILED` | Host or port wrong, or the proxy is SOCKS5 | Check with `curl -x http://USER:PASS@HOST:PORT https://ipinfo.io/json` |
| Pages fail to load as soon as the proxy is set (for HTTPS sites often `ERR_TUNNEL_CONNECTION_FAILED`) | No `'login'` handler, `preventDefault()` missing, or wrong credentials | Add the handler shown above and check the credentials with curl |
| Proxy ignored after changing it at runtime | Connections opened earlier still use the old route | `await ses.closeAllConnections()` after `setProxy` |
| Some requests bypass the proxy | They run in a different session or in the main process with `net` | Set the proxy on that session too, or use `app.setProxy` |
| Works for a while, then fails | The static IP's term ended | Renew before the end date; auto-renew can be set per IP |

More on that first error: [ERR_TUNNEL_CONNECTION_FAILED explained](../err-tunnel-connection-failed/README.md).

## FAQ

**Can I put `http://USER:PASS@HOST:PORT` in `proxyRules`?** No. Chromium does not take credentials from the proxy address. Use the `'login'` event.

**Does `setProxy` affect Node's `https` module in the main process?** No. Electron's session proxy covers Chromium's network stack (windows, `net`). Plain Node `http`/`https` calls need their own proxy agent.

**Static or rotating IP?** If the app signs in to an account, a static IP per profile. Rotating residential fits logged-out fetching across many addresses.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (HTTP or SOCKS5, chosen at purchase; $7.99 per 30 days, the same price in every country) and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/). Related: [Puppeteer with an authenticated proxy](../puppeteer-proxy/README.md).

*Facts checked 26 November 2026.*
