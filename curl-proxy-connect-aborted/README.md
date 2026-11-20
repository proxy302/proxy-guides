# What does curl's "Proxy CONNECT aborted" error mean?

`curl: (56) Proxy CONNECT aborted` means curl reached the proxy, sent its `CONNECT` request for an `https://` address, and the proxy hung up before it finished answering. There is no status code because none arrived, which is what separates it from `CONNECT tunnel failed, response 407` or `response 502`. In practice the cause is one of four things: the URL says `http://` but the port speaks SOCKS5, the proxy refuses your connection outright (wrong login mode, or an IP whose term has ended), the proxy could not reach the target and closed instead of returning an error page, or a device in between is cutting the connection. The variant `Proxy CONNECT aborted due to timeout` (exit code 28) means the proxy accepted the connection but never replied in time.

*Proxy302 maintains this guide and sells these proxies. The messages below were reproduced with curl 8.7.1 against a throwaway local proxy in November 2026, and the wording was checked against the curl 8.23 source (`lib/cf-h1-proxy.c`).*

## What curl shows

Run the failing command with `-v`. The last lines tell you how far the handshake got:

```text
* Establish HTTP proxy tunnel to example.com:443
> CONNECT example.com:443 HTTP/1.1
> Host: example.com:443
> Proxy-Authorization: Basic VVNFUjpQQVNT
> User-Agent: curl/8.7.1
> Proxy-Connection: Keep-Alive
>
* Proxy CONNECT aborted
curl: (56) Proxy CONNECT aborted
```

The request went out complete, with credentials. Nothing came back. That rules out a typo in the host name or port (those fail earlier, at `Trying...`), and points at the proxy's side of the conversation.

## Related messages and what each one tells you

| curl message | Exit code | What happened |
|---|---|---|
| `Proxy CONNECT aborted` | 56 | The proxy closed the connection before a full CONNECT reply |
| `Proxy CONNECT aborted due to timeout` | 28 | No reply within `--connect-timeout` or `-m` |
| `CONNECT tunnel failed, response 407` (or 403, 502...) | 56 | The proxy did answer, with an error status you can act on |
| `Recv failure: Connection reset by peer` | 56 | The connection was reset (RST), by the proxy or by something on the path |
| `connection to proxy closed` | 97 | A SOCKS5 handshake was cut off, often a `socks5h://` URL pointed at an HTTP port |
| `Failed to connect to HOST port PORT` | 7 | Nothing accepted the connection: wrong port, or a firewall |

## Find the cause in five checks

1. **Check the protocol against the port.** Try both schemes against the same host and port:

   ```bash
   curl -sS -x http://USER:PASS@HOST:PORT https://ipinfo.io/ip
   curl -sS -x socks5h://USER:PASS@HOST:PORT https://ipinfo.io/ip
   ```

   If only the `socks5h://` line works, the port is SOCKS5 and every tool needs a SOCKS5 setting. At Proxy302, HTTP or SOCKS5 is chosen when the IP is bought and each IP has one protocol, so the order page tells you which one to use.

2. **Try a plain `http://` target.** `curl -sS -x http://USER:PASS@HOST:PORT http://ipinfo.io/ip` skips the tunnel. If this also fails, usually with `(52) Empty reply from server`, the proxy is refusing you as a client, not struggling with the website.

3. **Try a second website.** If `https://ipinfo.io` works and one particular site aborts, the proxy could not open a connection to that site (DNS failure, the site refusing the proxy's address, a port the proxy does not forward) and closed instead of sending a 502.

4. **Check how the proxy expects you to log in.** Some proxies authenticate by username and password, others by an allowlist of your own public IP addresses. If you use an allowlist and your office or home connection got a new public address, the proxy may drop you without any message. Add the new address, or switch to password login.

5. **Check that the IP is still in its term.** When a static IP's term ends it is released, and a new purchase gets a new address. Old credentials then fail in ways that look like network faults. Auto-renew can be turned on per IP to avoid this.

If all five pass, run the same command from another network (a phone hotspot is enough). Corporate firewalls and some antivirus products inspect outbound proxy traffic and reset it.

## The timeout variant

`Proxy CONNECT aborted due to timeout` appears when the proxy accepted the TCP connection and then said nothing. Two common reasons:

- The proxy is trying to reach a target that does not answer (a firewalled address, or an IP that drops packets) and is still waiting when curl's timer runs out.
- The proxy is overloaded or the path to it is very lossy.

Set an explicit limit so scripts fail quickly instead of hanging: `--connect-timeout 15 -m 60`. Then repeat against `https://ipinfo.io/ip`; if that is fast, the slow part is the target, not the proxy.

## FAQ

**Is "Proxy CONNECT aborted" an authentication error?** Not directly. A proxy that rejects a password normally answers `407`, and curl reports `response 407`. An abort means the proxy closed without answering, which some proxies do when the client is not allowed at all.

**Why does `http://` work but `https://` aborts?** Plain HTTP requests are forwarded without a tunnel. If only tunnels fail, the proxy may not permit CONNECT to that port, or the target is unreachable from the proxy.

**Does `-k` / `--insecure` help?** No. The failure happens before TLS starts, so certificate options change nothing. See [proxy SSL errors](../proxy-ssl-errors/README.md) for errors that do come from TLS.

**I get it only now and then, in a long-running script.** Intermittent aborts usually mean idle connections being closed by the proxy or a NAT device. Add retries with a short back-off: `curl --retry 3 --retry-all-errors`.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs, each as HTTP or SOCKS5 as chosen at purchase, from $0.50 for one day or $7.99 for 30 days per IP, and rotating residential traffic from $1.50/GB. More curl options are in the [curl proxy cheat sheet](../curl-proxy-cheatsheet/README.md). Prices: [proxy302.com/pricing](https://proxy302.com/pricing/).

*Facts checked 20 November 2026.*
