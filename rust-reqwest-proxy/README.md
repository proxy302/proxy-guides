# How do I use a proxy with a username and password in Rust reqwest?

Create a `reqwest::Proxy` with `Proxy::all("http://HOST:PORT")`, add the login with `.basic_auth("USER", "PASS")`, and hand it to `Client::builder().proxy(...)`. That sends every `http://` and `https://` request through the proxy, with HTTPS going through a CONNECT tunnel. For SOCKS5, turn on reqwest's `socks` feature and use a `socks5h://USER:PASS@HOST:PORT` URL so DNS is resolved by the proxy. Without any `.proxy(...)` call, reqwest uses the `HTTP_PROXY`, `HTTPS_PROXY` and `ALL_PROXY` environment variables (and on macOS and Windows the system proxy); adding your own proxy switches that automatic behaviour off.

*Proxy302 maintains this guide and sells these proxies. Checked against the reqwest 0.13.5 documentation and source in November 2026, not run (no Rust toolchain on the test machine).*

## Cargo.toml

```toml
[dependencies]
reqwest = { version = "0.13", features = ["socks"] }   # drop "socks" if you only use HTTP proxies
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
```

reqwest 0.13 uses rustls for TLS by default. Add `"blocking"` to the feature list if you want the synchronous client.

## HTTP proxy with a login (async)

```rust
use std::{env, time::Duration};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let host = env::var("PROXY_HOST")?;
    let port = env::var("PROXY_PORT")?;

    let proxy = reqwest::Proxy::all(format!("http://{host}:{port}"))?
        .basic_auth(&env::var("PROXY_USER")?, &env::var("PROXY_PASS")?);

    let client = reqwest::Client::builder()
        .proxy(proxy)
        .connect_timeout(Duration::from_secs(15))
        .timeout(Duration::from_secs(30))
        .build()?;

    let body = client
        .get("https://ipinfo.io/json")
        .send()
        .await?
        .error_for_status()?
        .text()
        .await?;
    println!("{body}");
    Ok(())
}
```

```bash
PROXY_HOST=HOST PROXY_PORT=PORT PROXY_USER=USER PROXY_PASS=PASS cargo run
```

The JSON should show the proxy's exit address and country. `.basic_auth()` sets the `Proxy-Authorization` header, so the password does not need any escaping here.

## SOCKS5

With the `socks` feature enabled, put the credentials in the URL:

```rust
let proxy = reqwest::Proxy::all("socks5h://USER:PASS@HOST:PORT")?;
```

| Scheme | Who resolves the site's host name |
|---|---|
| `socks5://` | Your machine, then the proxy gets an IP address |
| `socks5h://` | The proxy |

Use `socks5h://` unless you have a reason not to; it keeps DNS lookups on the proxy's side and avoids failures when the name only resolves correctly there. If the password contains `@`, `:` or `/`, percent-encode it in the URL (`p@ss` becomes `p%40ss`).

## Blocking client

```rust
// reqwest = { version = "0.13", features = ["blocking"] }
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let proxy = reqwest::Proxy::all("http://HOST:PORT")?.basic_auth("USER", "PASS");
    let client = reqwest::blocking::Client::builder().proxy(proxy).build()?;
    println!("{}", client.get("https://ipinfo.io/ip").send()?.text()?);
    Ok(())
}
```

## Environment variables and how to turn them off

| Code | What reqwest does |
|---|---|
| `Client::new()` or a builder without `.proxy()` | Reads `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY` (either case), plus the macOS or Windows system proxy |
| `.proxy(p)` | Uses `p` only; the environment is ignored |
| `.no_proxy()` | No proxy at all, even if the environment sets one |
| `Proxy::http(...)` / `Proxy::https(...)` | Proxy only `http://` or only `https://` URLs |

This makes a quick test easy: `HTTPS_PROXY=http://USER:PASS@HOST:PORT cargo run` with an unchanged `Client::new()`.

## Errors

Print the whole chain with `{:?}` rather than `{}`. reqwest wraps the real cause, and the top line alone often reads only `error sending request`.

| What you see in the chain | Likely cause |
|---|---|
| Proxy authentication / status 407 | Wrong username or password |
| Connection refused, or a connect timeout | Wrong host or port, or a firewall |
| Failure right after connecting, with an `http://` proxy URL | The port speaks SOCKS5; switch the scheme |
| A `socks5h://` proxy is refused as an unknown or unsupported scheme | The `socks` feature is not enabled in `Cargo.toml` |

## FAQ

**Does a rotating gateway give a new IP per request?** Per new connection. One `Client` keeps idle connections open and reuses them, so consecutive requests usually leave through the same exit. Build a client with `.pool_max_idle_per_host(0)` if every request must open a new connection, or use the session parameter your provider documents (Proxy302's console generates it for you) for a deliberate sticky exit.

**Can I set different proxies for different hosts?** Use `Proxy::custom(|url| ...)` and return a proxy URL or `None` per request.

**HTTP or SOCKS5 for reqwest?** Both work. HTTP needs no extra feature. At Proxy302 the protocol is chosen per IP at purchase.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs as HTTP or SOCKS5, from $0.50 for one day or $7.99 for 30 days per IP, and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/).

*Facts checked 20 November 2026.*
