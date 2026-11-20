# How do I SSH to a server through an HTTP or SOCKS5 proxy?

OpenSSH has no proxy setting of its own, so you give it a `ProxyCommand`: a small program that opens the connection through the proxy and hands it to `ssh`. For a proxy without a password, the `nc` that ships with macOS and most Linux systems does it: `ProxyCommand nc -X connect -x HOST:PORT %h %p` for an HTTP proxy, or `-X 5` for SOCKS5. For a proxy with a username and password, use Ncat: `ProxyCommand ncat --proxy HOST:PORT --proxy-type socks5 --proxy-auth USER:PASS %h %p` (or `--proxy-type http`). Put it in `~/.ssh/config` under the host you want, and `ssh`, `scp`, `sftp`, `rsync` and Git over SSH will all use it.

*Proxy302 maintains this guide and sells these proxies. The `nc` commands were run with OpenSSH 10.3 and the macOS `nc` against a throwaway local proxy in November 2026; the Ncat options were checked against the Ncat manual, not run.*

## Which command for which proxy

| Proxy | Login | `ProxyCommand` |
|---|---|---|
| HTTP (CONNECT) | none, or your IP is allowlisted | `nc -X connect -x HOST:PORT %h %p` |
| SOCKS5 | none, or your IP is allowlisted | `nc -X 5 -x HOST:PORT %h %p` |
| HTTP (CONNECT) | username and password | `ncat --proxy HOST:PORT --proxy-type http --proxy-auth USER:PASS %h %p` |
| SOCKS5 | username and password | `ncat --proxy HOST:PORT --proxy-type socks5 --proxy-auth USER:PASS %h %p` |

`%h` and `%p` are replaced by `ssh` with the target host and port. The `nc` here is the OpenBSD variant (the default on macOS; the `netcat-openbsd` package on Debian and Ubuntu). It can send a username to an HTTP proxy with `-P USER`, but then asks for the password interactively, and it has no SOCKS5 login at all. That is why the table switches to Ncat for passwords. Ncat comes with Nmap (`apt install ncat`, `dnf install nmap-ncat`, or the Nmap installer on macOS and Windows).

## A config entry

```sshconfig
# ~/.ssh/config
Host build-server
    HostName 203.0.113.10
    User deploy
    ProxyCommand ncat --proxy HOST:PORT --proxy-type socks5 --proxy-auth USER:PASS %h %p
    ServerAliveInterval 30
```

Then `ssh build-server` goes through the proxy. `ServerAliveInterval` sends a small keep-alive every 30 seconds, which stops proxies and NAT devices from closing an idle session.

The password sits in plain text in this file, so lock it down: `chmod 600 ~/.ssh/config`. If several people use the machine, prefer a proxy that authenticates by IP allowlist, so no password is stored at all.

## One-off test without editing the config

```bash
ssh -o ProxyCommand='nc -X connect -x HOST:PORT %h %p' -T git@github.com
```

GitHub is used here only because it runs a public SSH server. A reply of `Permission denied (publickey)` counts as success: GitHub's server can only refuse your key after the tunnel is up and the SSH handshake has finished. Swap in your own server once this works. A proxy problem stops earlier, usually with `Connection closed by UNKNOWN port 65535`, often after a line from `nc` quoting the proxy's error.

## When the proxy only allows port 443

Many HTTP proxies accept CONNECT only to port 443, because that is where HTTPS lives, and refuse port 22. Symptoms: `nc: Proxy error: "HTTP/1.1 403 Forbidden"` followed by `Connection closed by UNKNOWN port 65535`. Options:

- Ask whether the target also listens on 443. GitHub, for example, accepts SSH on `ssh.github.com` port 443:

  ```sshconfig
  Host github.com
      HostName ssh.github.com
      Port 443
      User git
      ProxyCommand nc -X connect -x HOST:PORT %h %p
  ```

- Use a SOCKS5 proxy instead. SOCKS5 forwards any TCP port the proxy allows.

## Common problems

| Symptom | Cause |
|---|---|
| `nc: invalid option -- 'X'` | That `nc` is the traditional or BusyBox version; install the OpenBSD variant or use Ncat |
| Prompt `Proxy password for USER@HOST` | `nc -P` was used; switch to Ncat for unattended logins |
| `nc: Proxy error: "HTTP/1.0 407 ..."`, then `Connection closed by UNKNOWN port 65535` | The proxy wants a login; use Ncat with `--proxy-auth`, or allowlist your IP |
| `Connection closed by UNKNOWN port 65535` with no proxy error | Wrong protocol for the port (HTTP vs SOCKS5), or the proxy hung up |
| Works, then drops after a few idle minutes | Add `ServerAliveInterval 30` |
| Host key warning after switching proxies | The target's key did not change; only the network path did. If the warning persists, verify the key with the server's owner before accepting it |

## FAQ

**Does `ProxyJump` do the same thing?** No. `ProxyJump` hops through another SSH server. A `ProxyCommand` is for HTTP and SOCKS proxies.

**Why would I SSH through a proxy at all?** A common reason is a server firewall that only admits a known source address. A static IP gives your SSH sessions one fixed address to put on that allowlist, and it stays the same for the IP's whole term.

**Will Git over SSH use this?** Yes, Git calls `ssh`, which reads `~/.ssh/config`. For HTTPS remotes see [git proxy](../git-proxy/README.md).

**Can I use a rotating residential gateway?** It works for a single session, but each new connection may leave from a different address, which defeats a firewall allowlist. Use a static IP for that.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs as HTTP or SOCKS5, chosen at purchase, at $0.50 for one day or $7.99 for 30 days per IP. The address stays the same until the term ends; auto-renew can be turned on per IP. See [pricing](https://proxy302.com/pricing/).

*Facts checked 20 November 2026.*
