# How do I use npm behind a proxy?

Run `npm config set https-proxy http://USER:PASS@HOST:PORT` and `npm config set proxy http://USER:PASS@HOST:PORT`. npm fetches packages over `https://`, so `https-proxy` is the setting that actually carries the traffic; `proxy` covers plain-HTTP registries. Both are written to your user `.npmrc`, and npm also reads `HTTPS_PROXY` from the environment. pnpm reads the same `.npmrc` keys. Yarn 2 and later ignore all of that and use `httpsProxy` and `httpProxy` in `.yarnrc.yml`.

*Proxy302 maintains this guide and sells these proxies. Every command below ran with npm 11, pnpm 12 and Yarn 4 against local authenticating HTTP and SOCKS5 test proxies in November 2026.*

## npm

```bash
npm config set proxy http://USER:PASS@HOST:PORT
npm config set https-proxy http://USER:PASS@HOST:PORT
npm config set noproxy "localhost,127.0.0.1"
```

These lines end up in `~/.npmrc` (on Windows, `%USERPROFILE%\.npmrc`). Note that the value of `https-proxy` still begins with `http://`: it names the proxy used for HTTPS downloads, not a proxy that speaks TLS.

Check what npm will use. Current npm refuses to print a proxy value with `npm config get` because it may hold a password, so list the config instead, which masks it:

```bash
npm config list
```

You should see `https-proxy = "http://USER:***@HOST:PORT"`. Then fetch something small:

```bash
npm view is-number version
```

A version number means the registry was reached through the proxy.

### Just for one command

```bash
npm install --https-proxy=http://USER:PASS@HOST:PORT
```

or through the environment:

```bash
HTTPS_PROXY=http://USER:PASS@HOST:PORT npm install
```

### SOCKS5

npm accepts a SOCKS5 URL in the same setting:

```bash
npm config set https-proxy socks5h://USER:PASS@HOST:PORT
```

With `socks5h`, the proxy resolves `registry.npmjs.org`; with `socks5`, your machine looks it up first. Both worked in our test.

### Removing it

```bash
npm config delete proxy
npm config delete https-proxy
```

## pnpm

pnpm takes proxy settings from `.npmrc` files, so the `npm config set` lines above already apply. To keep the proxy to one project, put it in that project's `.npmrc`:

```ini
https-proxy=http://USER:PASS@HOST:PORT
proxy=http://USER:PASS@HOST:PORT
```

Do not commit a project `.npmrc` that contains a password.

## Yarn 2, 3 and 4

Modern Yarn ignores `.npmrc` proxy keys. Set its own:

```bash
yarn config set httpsProxy http://USER:PASS@HOST:PORT
yarn config set httpProxy http://USER:PASS@HOST:PORT
```

This writes to the project's `.yarnrc.yml`; add `-H` to write to your home `.yarnrc.yml` instead. Yarn 4 also ignored a plain `HTTPS_PROXY` variable in our test; for a one-off run it reads `YARN_HTTPS_PROXY`. Yarn 1 (classic) still uses `yarn config set proxy` and `yarn config set https-proxy`.

## Settings at a glance

| Tool | HTTPS registry setting | Where it lives |
|---|---|---|
| npm | `https-proxy` | `~/.npmrc` or project `.npmrc` |
| pnpm | `https-proxy` | `.npmrc` |
| Yarn 2+ | `httpsProxy` | `.yarnrc.yml` |
| Yarn 1 | `https-proxy` | `~/.yarnrc` |
| npm or pnpm, temporarily | `HTTPS_PROXY` environment variable | Your shell |
| Yarn 2+, temporarily | `YARN_HTTPS_PROXY` environment variable | Your shell |

## Common errors

| Message | Cause |
|---|---|
| `npm error code E407` / `407 Proxy Authentication Required` | Wrong or missing proxy username or password |
| `request to https://registry.npmjs.org/... failed, reason: connect ECONNREFUSED HOST:PORT` | Wrong host or port, or nothing listening there |
| `Proxy connection ended before receiving CONNECT response` | Something at that address accepted the connection but never answered as an HTTP proxy, for example a SOCKS-only port given an `http://` URL |
| `tunnel error: proxy authorization required` (pnpm) | Credentials missing or refused |
| `YN0001: RequestError: Bad response: 407` (Yarn) | Credentials missing or refused in `.yarnrc.yml` |
| `The https-proxy option is protected, and cannot be retrieved in this way` | Not an error with the proxy: use `npm config list` to see the masked value |
| `SELF_SIGNED_CERT_IN_CHAIN` or `UNABLE_TO_GET_ISSUER_CERT_LOCALLY` | A TLS-inspecting company proxy; point `cafile` at its CA certificate rather than turning off `strict-ssl`. See [SSL errors through a proxy](../proxy-ssl-errors/README.md) |

## FAQ

**What if the password contains `@` or `#`?** Percent-encode those characters, for example `@` as `%40` and `#` as `%23`. An unencoded `@` makes npm read part of the password as the hostname.

**Does `npm config set` affect Node scripts I run?** No. It only configures npm's own downloads. For `fetch` or axios inside your code, see [using a proxy in Node.js](../nodejs-proxy/README.md).

**Why did install use the proxy even though I didn't set anything?** npm and pnpm honour `HTTPS_PROXY` and `HTTP_PROXY` if they are exported in your shell profile. Check with `env | grep -i proxy`.

**Static or rotating for a CI server?** A CI runner that pulls from a private registry with an IP allowlist needs one address that stays the same: a static residential (ISP) IP.

## Where Proxy302 fits

Proxy302 static residential (ISP) IPs come as HTTP or SOCKS5, chosen at purchase, at $0.50 for one day or $7.99 for 30 days per IP, the same in every country. There is no free trial; one IP for one day is the cheapest way to test a setup like this. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
