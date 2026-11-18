# How do I make wget use a proxy with a username and password?

Set `https_proxy=http://HOST:PORT` (plus `http_proxy` for plain-HTTP downloads) and add `--proxy-user=USER --proxy-password=PASS`, or write the credentials into the proxy URL as `http://USER:PASS@HOST:PORT`. For a single command that leaves your shell untouched, pass the setting with `-e https_proxy=http://HOST:PORT`. GNU Wget only talks to HTTP proxies; it cannot use SOCKS5, so for an IP sold as SOCKS5 use curl with `socks5h://`. If your distribution ships Wget2 as `wget`, the password options are named `--http-proxy-user` and `--http-proxy-password`.

*Proxy302 maintains this guide and sells these proxies. The commands were checked against the GNU Wget and Wget2 manuals and the Wget 1.x source in November 2026.*

## One command, nothing saved

```bash
wget -e use_proxy=on -e https_proxy=http://HOST:PORT \
     --proxy-user=USER --proxy-password=PASS \
     -qO- https://ipinfo.io/json
```

`-e` runs a `.wgetrc` setting for this invocation only. The proxy URL starts with `http://` even though the setting is called `https_proxy`: the name says which downloads use it, the scheme says how wget talks to the proxy. `-qO-` prints the page to the terminal, here the exit IP and its country.

## Environment variables for a shell session

```bash
export https_proxy="http://USER:PASS@HOST:PORT"
export http_proxy="$https_proxy"
export no_proxy="localhost,127.0.0.1"
wget -qO- https://ipinfo.io/json
```

Credentials in the URL must be percent-encoded if they contain `@`, `:`, `/` or `#`. Other tools in the same shell (curl, git, pip) read the same variables, which is convenient or surprising depending on what you meant.

## Permanent settings in ~/.wgetrc

```ini
use_proxy = on
http_proxy = http://HOST:PORT
https_proxy = http://HOST:PORT
proxy_user = USER
proxy_password = PASS
no_proxy = localhost,127.0.0.1
```

The file now holds a password, so restrict it with `chmod 600 ~/.wgetrc`. Wget also reads a system-wide `wgetrc`, whose path depends on how it was installed. Settings given on the command line win over both files.

To bypass every proxy setting for one download, add `--no-proxy`.

## Wget2

Fedora and some other distributions now install Wget2 under the name `wget`. Check with `wget --version`; Wget2 identifies itself on the first line. Its proxy options differ:

```bash
wget --https-proxy=http://HOST:PORT \
     --http-proxy-user=USER --http-proxy-password=PASS \
     -qO- https://ipinfo.io/json
```

The Wget2 manual lists `--proxy-user` and `--proxy-password` as not implemented, which is why a command copied from an older tutorial can fail with 407 there. The `http_proxy`, `https_proxy` and `no_proxy` environment variables work in both versions. Wget2 has no SOCKS support either.

## Which method to use

| Situation | Method |
|---|---|
| One download, nothing left behind | `-e https_proxy=...` with `--proxy-user` / `--proxy-password` |
| Several tools in one terminal | `export https_proxy=...` |
| Every wget run on this account | `~/.wgetrc`, mode 600 |
| Multi-user server where others can run `ps` | Avoid passwords on the command line; use the file |
| IP sold as SOCKS5 | Not possible in wget; `curl -x socks5h://USER:PASS@HOST:PORT -O URL` |

## Common errors

| What wget prints | Cause |
|---|---|
| `Proxy tunneling failed: Proxy Authentication Required` then `Unable to establish SSL connection.` | Wrong or missing credentials on an `https://` download |
| `ERROR 407: Proxy Authentication Required.` | The same, on a plain `http://` download |
| `Proxy tunneling failed: Bad Gateway` (or `Service Unavailable`) | The proxy accepted you but could not reach the site; try again or test another URL |
| `Error parsing proxy URL socks5://...: Unsupported scheme.` | A SOCKS address; wget cannot use it |
| `Connecting to HOST ... failed: Connection refused.` | Wrong port, or nothing listening at that address |
| Exit IP is still your own | The variable was set in a different shell, `use_proxy = off` is set somewhere, or the host matches `no_proxy` |

## FAQ

**Does `wget` in Windows PowerShell work the same way?** In Windows PowerShell 5.1, `wget` is an alias for `Invoke-WebRequest`, which uses its own parameters: `Invoke-WebRequest -Uri https://ipinfo.io/json -Proxy http://HOST:PORT -ProxyCredential (Get-Credential)`. PowerShell 7 dropped that alias.

**Why does `--proxy-password` worry security teams?** Command-line arguments show up in process listings and shell history. A `.wgetrc` readable only by you avoids both.

**Does a proxy speed up large downloads?** Usually not; it adds a hop. Use one when you need the download to come from a particular address or country.

**Which IP type for a server that downloads from an allowlisted host?** A static residential (ISP) IP, so the address on the allowlist stays valid for the whole term.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs with HTTP or SOCKS5 chosen at purchase; for wget, choose HTTP. A static IP costs $0.50 for one day, $3.50 for 7 days or $7.99 for 30 days, the same in every country. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
