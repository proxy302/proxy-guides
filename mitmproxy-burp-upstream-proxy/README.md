# How do I chain mitmproxy or Burp Suite to an upstream proxy with a username and password?

In mitmproxy, start it in upstream mode and pass the credentials as an option: `mitmproxy --mode upstream:http://HOST:PORT --set upstream_auth=USER:PASS`. Your client then talks to mitmproxy on `127.0.0.1:8080`, and mitmproxy forwards everything to the upstream proxy with Basic authentication. In Burp Suite, open Settings, Network, Connections, add a rule under "Upstream proxy servers" with destination host `*`, the proxy's host and port, authentication type Basic, and the username and password; for a SOCKS5 IP, use the "SOCKS proxy" section on the same page instead. Proxy302 maintains this guide and sells these proxies; a static ISP IP costs $0.50 for one day or $7.99 for 30 days. Use these tools only on applications and accounts you own or are authorised to test.

*Checked against the mitmproxy documentation (modes and options reference) and PortSwigger's Burp Suite documentation for the Connections settings, November 2026. Not run locally.*

## Why chain to an upstream proxy

- Your test traffic has to leave from a fixed address that a staging server or API allowlists.
- You want to see your own app's requests as they look from another country, while still inspecting them.
- A company network only allows outbound traffic through its own proxy.

## mitmproxy

```bash
# interactive console
mitmproxy --mode upstream:http://HOST:PORT --set upstream_auth=USER:PASS

# headless, printing a line per request
mitmdump --mode upstream:http://HOST:PORT --set upstream_auth=USER:PASS --listen-port 8080
```

- `upstream_auth` takes `username:password` and adds HTTP Basic authentication to requests sent to the upstream proxy.
- The upstream must be an HTTP proxy. mitmproxy's upstream mode speaks explicit HTTP and HTTPS proxy protocol; its `socks5` mode makes mitmproxy itself a SOCKS server, it does not connect to one. Buy the IP with the HTTP protocol.
- mitmproxy listens on port 8080 unless you set `--listen-port`.
- To keep the password out of your shell history, put it in `~/.mitmproxy/config.yaml` instead:

  ```yaml
  mode:
    - upstream:http://HOST:PORT
  upstream_auth: USER:PASS
  ```

### Trust the mitmproxy certificate

To inspect HTTPS, the client must trust mitmproxy's CA. With mitmproxy running, open `http://mitm.it` through it and follow the steps for your system, or point a command-line client at the file mitmproxy created:

```bash
curl -sS --cacert ~/.mitmproxy/mitmproxy-ca-cert.pem -x http://127.0.0.1:8080 https://ipinfo.io/json
```

The `ip` and `country` in the answer should belong to the upstream proxy, not to your own connection.

## Burp Suite

1. Open **Settings**, then **Network**, then **Connections**.
2. Under **Upstream proxy servers**, click **Add**.
3. **Destination host:** `*` to send all traffic through the proxy. Burp accepts wildcards (`*` for any characters, `?` for any one character except a dot).
4. **Proxy host** and **Proxy port:** `HOST` and `PORT` from your order page.
5. **Authentication type:** Basic. Enter the username and password.
6. Save, then browse through Burp and confirm the exit on `https://ipinfo.io/json`.

Rule notes from PortSwigger's documentation:

- Burp uses the first rule in the table that matches the destination host; hosts with no matching rule connect directly.
- To keep some hosts direct, add a rule for them above the `*` rule and leave its proxy host blank.
- Tick "Override options for this project only" if the upstream should apply to one project and not to every project.

### SOCKS5 IP in Burp

In the **SOCKS proxy** section on the same page, enter host, port, username and password, and tick **Do DNS lookups over SOCKS proxy** so names are resolved through the proxy and not locally.

## Side by side

| | mitmproxy | Burp Suite |
|---|---|---|
| Where | `--mode upstream:` plus `upstream_auth` | Settings, Network, Connections |
| HTTP upstream with username and password | Yes (Basic) | Yes (Basic; NTLM options too) |
| SOCKS5 upstream with username and password | No | Yes, in the SOCKS proxy section |
| Send only some hosts upstream | Run a separate instance, or script it | Per-host rules |

## Check the upstream on its own first

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

If this fails, fix the proxy details before touching mitmproxy or Burp.

## Common errors

| Symptom | Likely cause |
|---|---|
| 407 from upstream in the flow list | `upstream_auth` missing or wrong; in Burp, authentication type not set to Basic or wrong password |
| Every HTTPS site shows a certificate warning | The client does not trust the mitmproxy or Burp CA |
| Exit IP is your own | The client is not using mitmproxy or Burp, or the upstream rule does not match (Burp) |
| Upstream connection refused | Wrong host or port, or a SOCKS5 IP given to mitmproxy's upstream mode |
| Worked yesterday, fails today | The IP's term ended and it was released |

## FAQ

**Can the client send its own proxy credentials through mitmproxy?** You do not need that. The client talks to mitmproxy without credentials, and mitmproxy adds them for the upstream.

**Static or rotating IP for testing?** Static, if a server allowlists your address or you want repeatable results. Rotating residential ($1.50/GB) suits one-off checks from different places.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5 chosen at purchase) and rotating residential traffic. See [pricing](https://proxy302.com/pricing/). Related: [why SSL errors appear behind a proxy](../proxy-ssl-errors/README.md), [a fixed IP for an API allowlist](../fixed-ip-for-api-allowlist/README.md).

*Facts checked 27 November 2026.*
