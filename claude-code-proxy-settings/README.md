# How do I make Claude Code use an HTTP proxy?

Set `HTTPS_PROXY` to the proxy's full URL, such as `http://USER:PASS@HOST:PORT`, before you start `claude`, or put it in the `env` block of `~/.claude/settings.json`. Claude Code reads the standard variables `HTTPS_PROXY`, `HTTP_PROXY` and `NO_PROXY` (lowercase forms too), accepts basic authentication inside the URL, and does not support SOCKS proxies. Confirm it with `/status`, which has a Proxy row. Source: Anthropic's [Enterprise network configuration](https://code.claude.com/docs/en/network-config) page.

*Proxy302 maintains this guide and sells these proxies.*

This is ordinary network setup: a company proxy, a firewall that only lets traffic out through one gateway, or a fixed egress IP that an admin can allowlist. A proxy changes the network path only; it does not change Anthropic's [Usage Policy](https://www.anthropic.com/legal/aup) or the list of [supported regions](https://www.anthropic.com/legal/supported-regions).

## What Claude Code reads

| Setting | Behaviour, per Anthropic's docs |
|---|---|
| `https_proxy`, `HTTPS_PROXY`, `http_proxy`, `HTTP_PROXY` | The first one set, in that order, is used |
| `NO_PROXY` | Hosts to reach directly, space- or comma-separated; `*` bypasses the proxy for everything |
| `user:password@` in the URL | Basic authentication. For NTLM or Kerberos the docs point to an LLM gateway |
| SOCKS URLs | Not supported |
| `NODE_EXTRA_CA_CERTS` | Extra CA file, for proxies that inspect TLS |
| `CLAUDE_CODE_CERT_STORE` | `bundled`, `system`, or both (default `bundled,system`) |
| `CLAUDE_CODE_CLIENT_CERT`, `CLAUDE_CODE_CLIENT_KEY`, `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | Client certificate for mTLS |

## Setup

**1. Check the proxy on its own.** The printed address is your egress IP, the one an admin would add to an allowlist:

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/ip
```

**2. Set it for one terminal session.** macOS or Linux:

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
claude
```

Windows PowerShell:

```powershell
$env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
claude
```

Claude Code reads shell variables once at startup, so restart it after any change.

**3. Make it permanent.** Add the variable to `~/.claude/settings.json`:

```json
{
  "env": {
    "HTTPS_PROXY": "http://USER:PASS@HOST:PORT",
    "NO_PROXY": "localhost,127.0.0.1"
  }
}
```

Anthropic recommends this route for background agents (`claude agents`, `--bg`, `/background`). They run under a shared supervisor process that inherits the environment of whichever shell started it first, so a shell export may or may not reach them; the settings file always does. The file stores the password in plain text, so keep it private and out of shared repositories.

**4. Verify.** Inside Claude Code, run `/status`. The Proxy row shows the active URL and marks a value it cannot parse as invalid and ignored. For more detail, start with `claude --debug`; the log goes to `~/.claude/debug/<session-id>.txt`.

## Certificates and allowed hosts

A forwarding proxy that only tunnels HTTPS needs no certificate settings. A company proxy that inspects TLS presents its own root certificate. Claude Code trusts its bundled Mozilla CA set and the operating system's store by default (npm installs need Node 22.15 or later to read the OS store), so installing the root in the OS store usually works; otherwise point `NODE_EXTRA_CA_CERTS` at a PEM file.

If a firewall sits behind the proxy, allow at least `api.anthropic.com`, `claude.ai`, `claude.com` and `platform.claude.com`. The docs list every other host and what needs it.

## Common errors

| Symptom | Cause and fix |
|---|---|
| Launch stops with an error naming the proxy variable | The URL can't be parsed, usually because `http://` is missing |
| `/status` shows the proxy as invalid | Same, or the password has `@`, `:`, `#` or `/` that was not percent-encoded |
| Requests fail with 407 | Wrong username or password, or a proxy that needs NTLM or Kerberos |
| Nothing connects with a `socks5://` URL | SOCKS is not supported; use an HTTP proxy |
| Certificate verification errors | TLS-inspecting proxy; install its root in the OS store or set `NODE_EXTRA_CA_CERTS` |
| Background agents skip the proxy | The variable was only exported in a shell; move it to `settings.json` |

## FAQ

**Do I need both `HTTPS_PROXY` and `HTTP_PROXY`?** No. Claude Code uses the first one it finds, so one is enough.

**Can the proxy URL itself start with `https://`?** Anthropic's examples show both `http://` and `https://` proxy URLs; use the scheme your proxy speaks. For a proxy sold as an HTTP proxy, that is `http://`.

**Does this work in the Code tab of Claude Desktop?** Anthropic documents that local Code tab sessions signed in through claude.ai can take the proxy variables from settings files. See the [Chinese guide on Claude Desktop](../claude-desktop-proxy-zh/README.md) and the official docs.

**Which IP do I need for an allowlist?** A static one, so the address on the list stays valid. Rotating addresses do not suit allowlists. Your own organisation's admin adds the address; Proxy302 does not register IPs with Anthropic, its IPs are not on any allowlist, and an allowlisted IP does not exempt an account from identity or phone verification.

## Where Proxy302 fits

Proxy302 sells static ISP IPs at the same price in every country: $0.50 for one day, $7.99 for 30 days, up to $23.97 for 90 days. Choose HTTP at purchase for Claude Code; each IP has one protocol. Static IPs of 30 days or longer are dedicated to the buyer. See [pricing](https://proxy302.com/pricing/) and [proxy302.com](https://proxy302.com).

*Black Friday, 24 November to 4 December 2026 (UTC+8): 60-day static ISP IPs are US$14.00 and 90-day IPs US$18.00 (1 to 30 days unchanged). Prices elsewhere on this page are the regular prices; [pricing](https://proxy302.com/pricing/) shows what applies on the day.*<!-- bf2026 -->

*Facts checked 17 November 2026.*
