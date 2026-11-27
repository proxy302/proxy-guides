# How do I run Squid as a local forwarder to a proxy that needs a username and password?

Add two lines to Squid's configuration: `cache_peer HOST parent PORT 0 no-query no-digest default login=USER:PASS` and `never_direct allow all`. Squid then sends every request, HTTPS `CONNECT` tunnels included, to the upstream proxy with Basic authentication, and your programs point at Squid (for example `127.0.0.1:3128`) with no credentials at all. That solves the common case of software that only has fields for an address and a port, and lets several machines on your own network use one authenticated upstream without each holding the password. Proxy302 maintains this guide and sells these proxies; a static ISP IP costs $0.50 for one day or $7.99 for 30 days. The upstream must be an HTTP proxy, because Squid cannot use a SOCKS5 proxy as a parent.

*Checked against the Squid `cache_peer` reference and the Squid wiki's cache hierarchy page, November 2026. Not run locally.*

## Step 1: install Squid

| System | Install | Config file |
|---|---|---|
| Debian, Ubuntu | `sudo apt install squid` | `/etc/squid/squid.conf` |
| RHEL, Rocky, AlmaLinux, Fedora | `sudo dnf install squid` | `/etc/squid/squid.conf` |
| macOS with Homebrew | `brew install squid` | `$(brew --prefix)/etc/squid.conf` |

## Step 2: edit the configuration

Keep the file your package installed (it already defines the usual safe-port rules). Replace its existing `http_port 3128` line, since two lines on the same port will not start, and add the rest:

```text
# Listen only where you need it: this machine only...
http_port 127.0.0.1:3128
# ...or also your LAN address, if other machines will use it:
# http_port 192.168.1.10:3128

# Send everything to the upstream proxy with its username and password
cache_peer HOST parent PORT 0 no-query no-digest default login=USER:PASS
never_direct allow all

# A forwarder does not need a cache
cache deny all

# Do not pass your client addresses on in plain-HTTP requests
forwarded_for delete
via off
```

What each piece does:

| Line | Meaning |
|---|---|
| `cache_peer HOST parent PORT 0` | The upstream proxy is a parent; `0` means no ICP port |
| `no-query no-digest` | Do not try cache-to-cache protocols with it |
| `default` | Use this parent as the last-resort route |
| `login=USER:PASS` | Add Basic `Proxy-Authorization` with these credentials to requests sent to the parent |
| `never_direct allow all` | Never connect to websites directly; if the parent is down, requests fail instead of leaking out from your own address |
| `cache deny all` | Do not store responses |
| `forwarded_for delete`, `via off` | Remove the `X-Forwarded-For` and `Via` headers Squid would otherwise add to plain-HTTP requests |

**Special characters in the password:** the Squid reference says the `login=` string can contain URL escapes (`%20` for a space), and that this means a literal `%` has to be written as `%%`.

**Who may connect:** the stock configuration allows `localhost` and, depending on the package, the private ranges in `localnet`. If other machines on your LAN should use it, make sure `http_access allow localnet` is active and `localnet` matches your network. Never listen on a public address without access rules; an open forwarder lets anyone use your paid IP.

## Step 3: check and restart

```bash
sudo squid -k parse            # syntax check, prints errors and warnings
sudo systemctl restart squid   # Linux
brew services restart squid    # macOS with Homebrew
```

## Step 4: test through Squid

```bash
curl -sS --max-time 20 -x http://127.0.0.1:3128 https://ipinfo.io/json
```

The `ip` and `country` should be the upstream proxy's. In Squid's `access.log` the request should show a parent-type hierarchy code (such as `DEFAULT_PARENT` or `FIRSTUP_PARENT`) followed by the upstream host, not `HIER_DIRECT`.

Now point your program at `127.0.0.1:3128` (or the LAN address) as an HTTP proxy with no username or password.

## Before you start: test the upstream on its own

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

If this fails, Squid will fail too.

## Common errors

| Symptom | Likely cause |
|---|---|
| `407` coming back through Squid | Wrong `login=` value, or a `%` in the password not doubled |
| `403 Forbidden` from Squid itself (`TCP_DENIED`) | Your client's address is not allowed by `http_access` |
| `503` or "unable to forward this request" | Upstream host or port wrong, upstream unreachable, or the IP was bought as SOCKS5 |
| Exit IP is your own | `never_direct allow all` missing, so Squid went direct |
| Worked yesterday, fails today | The upstream IP's term ended and it was released |

## FAQ

**Can I add a second upstream for failover?** Yes, add another `cache_peer` line. Squid's wiki notes that with only one parent and `never_direct`, every request fails if that parent is unreachable.

**Is Squid the only way?** No. Any local forwarder works. Squid is packaged on every major Linux distribution and Homebrew, which makes it the least work on a server.

**Static or rotating upstream?** Static if anything downstream depends on one fixed exit address; rotating residential ($1.50/GB) if it does not.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5 chosen at purchase) and rotating residential traffic. For a Squid parent, buy the HTTP protocol. See [pricing](https://proxy302.com/pricing/). Related: [proxy formats in different programs](../proxy-format-zh/README.md), [SOCKS5 vs HTTP](../socks5-vs-http-proxy/README.md).

*Facts checked 27 November 2026.*
