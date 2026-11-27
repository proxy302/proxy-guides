# How do I run a GitHub Actions self-hosted runner behind a proxy with a username and password?

Create a file named `.env` in the runner's folder (where you unpacked it) with `https_proxy=http://USER:PASS@HOST:PORT`, `http_proxy=` the same, and `no_proxy=` for hosts that should go direct, then restart the runner service. GitHub's documentation says the runner reads these variables at startup, supports Basic authentication in the URL, prefers the lowercase names on Linux and macOS, and does not accept IP addresses or CIDR ranges in `no_proxy`. If your organisation uses GitHub's IP allow list, add the proxy's address to it, since that is where the runner's traffic now comes from. Proxy302 maintains this guide and sells these proxies; a static ISP IP costs $0.50 for one day or $7.99 for 30 days, which suits an allowlist because the address stays the same for the whole term.

*Checked against GitHub's "Using a proxy server with self-hosted runners" and "Managing allowed IP addresses for your organization" pages, November 2026. Not run locally.*

## When this comes up

- **Company network:** the machine running the runner can only reach the internet through a proxy.
- **IP allow list:** your organisation on GitHub Enterprise Cloud only accepts connections from listed addresses. GitHub's documentation says self-hosted runners must then connect from an address on that list. A runner on a laptop, a home connection or an autoscaling cloud instance has no fixed address of its own; sending it through one static IP gives it one.
- **Deploy targets that allowlist:** jobs that call a database, API or server which only accepts known addresses.

## Step 1: the .env file

In the runner folder (the one that contains `run.sh` or `run.cmd`):

```bash
cat > .env <<'EOF'
https_proxy=http://USER:PASS@HOST:PORT
http_proxy=http://USER:PASS@HOST:PORT
no_proxy=localhost,ghe.internal.example
EOF
chmod 600 .env
```

- One `NAME=value` per line, no `export`, no quotes.
- `no_proxy` is a comma-separated list of host names, optionally with a port (`myserver.local:443`). IP addresses and CIDR ranges are not supported there.
- A password with `@`, `:` or `/` must be percent-encoded (`@` becomes `%40`). A password of letters and digits avoids the question.
- On Linux and macOS, if both `https_proxy` and `HTTPS_PROXY` exist, the runner uses the lowercase one.

## Step 2: configure or restart the runner

The variables are read when the runner starts, so they must be in place before `config.sh` registers it and again whenever it restarts.

**New runner:** export the same variables in your shell, then register:

```bash
export https_proxy="http://USER:PASS@HOST:PORT"
export http_proxy="$https_proxy"
./config.sh --url https://github.com/YOUR-ORG --token YOUR-REGISTRATION-TOKEN
```

**Runner installed as a service (Linux):**

```bash
sudo ./svc.sh stop
sudo ./svc.sh start
```

On macOS the same script runs without `sudo`. On Windows, restart the service whose name starts with `actions.runner.` from Services or with `Restart-Service`.

GitHub's documentation also mentions putting the variables in `/etc/environment` on Linux and macOS so they survive reboots and image rebuilds; on Windows you can set them as system environment variables.

## Step 3: allow the proxy's address

If you use GitHub's IP allow list (a GitHub Enterprise Cloud feature), add the proxy's exit address. Find it from the runner machine:

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/ip
```

A static IP keeps the same address for its whole term; renew before it ends, because an expired IP is released and a new purchase gets a different address that you would have to add again.

## What goes through the proxy

| Traffic | Uses the `.env` proxy? |
|---|---|
| The runner talking to GitHub (job assignment, logs, artifacts) | Yes |
| Tools your workflow steps run (`npm`, `pip`, `docker`, `curl`) | Each tool decides. Many read `https_proxy` and `http_proxy`; some, such as Docker's daemon, need their own setting |
| Hosts listed in `no_proxy` | No |

## Which IP to use

| Need | Proxy | Price at Proxy302 |
|---|---|---|
| Allowlisted runner, long-lived | Static ISP IP, HTTP protocol | $7.99 per 30 days |
| Try the setup | Static ISP IP for 1 day | $0.50 |
| Only bulk downloads, no allowlist | Rotating datacenter | $0.40/GB |

Buy the IP with the HTTP protocol: the runner uses an HTTP proxy URL. The protocol is fixed per IP at purchase.

## Common errors

| Symptom | Likely cause |
|---|---|
| Runner shows offline right after adding `.env` | Not restarted, or a typo in the URL |
| 407 in the runner's `_diag` logs | Wrong username or password, or special characters not encoded |
| Organisation rejects the runner | The proxy's exit address is missing from the IP allow list |
| Internal GitHub Enterprise Server unreachable | It is reached by IP, which `no_proxy` cannot exclude; use its host name |
| Worked yesterday, fails today | The IP's term ended and it was released |

## FAQ

**Can I use the `.env` file on GitHub-hosted runners?** No. It is for self-hosted runners only.

**Does each runner need its own IP?** No. Several runners can use the same proxy if one allowlisted address is what you want.

**Where should the proxy password be stored?** Only in the runner's `.env` (readable by the runner's user) or in the service's environment, never in the repository or workflow file.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5 chosen at purchase), rotating residential and datacenter traffic. See [pricing](https://proxy302.com/pricing/). Related: [which proxy for an API that only accepts allowlisted IPs](../fixed-ip-for-api-allowlist/README.md), [git through a proxy](../git-proxy/README.md).

*Facts checked 27 November 2026.*
