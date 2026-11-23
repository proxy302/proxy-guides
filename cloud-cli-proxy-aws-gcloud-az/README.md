# How do I run the AWS CLI, gcloud and Azure CLI through a proxy with a username and password?

Set `HTTPS_PROXY=http://USER:PASS@HOST:PORT` (and `HTTP_PROXY` to the same value) before running `aws` or `az`; both send the username and password with HTTP Basic authentication. For `gcloud`, either use the same variables or set its own properties: `gcloud config set proxy/type http`, `proxy/address`, `proxy/port`, plus `CLOUDSDK_PROXY_USERNAME` and `CLOUDSDK_PROXY_PASSWORD` in the environment so the password is not saved to disk. On a cloud VM, add the metadata address `169.254.169.254` to `NO_PROXY`. Proxy302 maintains this guide and sells these proxies; a static ISP IP is the one to use when an IAM or sign-in policy only allows a fixed source address.

## Why a fixed IP is useful here

All three clouds let an administrator restrict access by source address: AWS IAM policies can use the `aws:SourceIp` condition key, Microsoft Entra Conditional Access works with named locations defined by IP ranges, and Google Cloud's Access Context Manager can build access levels from IP ranges. A laptop on home, office and hotel networks has a different address in each place. Sending CLI traffic through one static IP gives the policy a single address to allow. Check each cloud's documentation for the cases where those conditions do not apply.

## Settings at a glance

| CLI | How it finds the proxy | Credentials | Proxy types |
|---|---|---|---|
| AWS CLI | `HTTPS_PROXY`, `HTTP_PROXY`, `NO_PROXY` | In the URL, Basic auth; NTLM is not supported | HTTP |
| Azure CLI (`az`) | `HTTPS_PROXY`, `HTTP_PROXY`, `NO_PROXY` | In the URL | HTTP |
| gcloud | `proxy/*` properties, or `https_proxy`, `http_proxy`, `no_proxy` | `proxy/username` and `proxy/password`, or `CLOUDSDK_PROXY_USERNAME` / `CLOUDSDK_PROXY_PASSWORD` | Properties: `http`, `http_no_tunnel`, `socks4`, `socks5`. Variables: HTTP only |

If you set the gcloud properties, they win over the variables, including `no_proxy`.

## AWS CLI

macOS and Linux:

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
export HTTP_PROXY="$HTTPS_PROXY"
export NO_PROXY="169.254.169.254"   # only needed on EC2 with an instance role
aws sts get-caller-identity
```

Windows PowerShell:

```powershell
$env:HTTPS_PROXY = 'http://USER:PASS@HOST:PORT'
$env:HTTP_PROXY  = $env:HTTPS_PROXY
aws sts get-caller-identity
```

AWS's documentation notes that if a variable is set twice in different cases, the lowercase one wins; set each once. The URL scheme stays `http://` even for `HTTPS_PROXY`, because it describes how the CLI talks to the proxy.

## Azure CLI

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
export HTTP_PROXY="$HTTPS_PROXY"
az account show
```

`az login` opens a browser for sign-in; the browser uses its own proxy settings, not these variables. If a company proxy that decrypts traffic causes certificate errors, Microsoft documents setting `REQUESTS_CA_BUNDLE` to a bundle that includes that proxy's certificate. A plain HTTP proxy that tunnels HTTPS does not need this.

## gcloud

With properties (stored in the active gcloud configuration):

```bash
gcloud config set proxy/type http
gcloud config set proxy/address HOST
gcloud config set proxy/port PORT
export CLOUDSDK_PROXY_USERNAME="USER"
export CLOUDSDK_PROXY_PASSWORD="PASS"
gcloud projects list --limit=1
```

Google's documentation points out that the two `CLOUDSDK_PROXY_*` variables keep the credentials out of shell history, logs and the configuration file. You can also run `gcloud config set proxy/username USER` and `gcloud config set proxy/password PASS`, but then the password is stored in the configuration.

With variables only, gcloud expects an HTTP proxy URL:

```bash
export https_proxy="http://USER:PASS@HOST:PORT"
gcloud projects list --limit=1
```

To go back to a direct connection: `gcloud config unset proxy/type`, `proxy/address` and `proxy/port`, and unset the variables.

For a SOCKS5 IP, gcloud's properties accept `proxy/type socks5`; the AWS and Azure CLIs are documented for HTTP proxies only. If one IP must serve all three, buy it as HTTP.

## Step by step

1. **Buy an HTTP static ISP IP** in the country your team works from. Proxy302 lets you choose HTTP or SOCKS5 per IP at purchase.
2. **Test the proxy on its own** before involving any CLI:

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   Note the `ip` value. `USER`, `PASS`, `HOST` and `PORT` are placeholders.
3. **Ask your cloud administrator to allow that address** in the relevant policy, if that is why you need it.
4. **Set the variables or properties** above in the shell where you run the CLI. Percent-encode special characters in the password inside URLs (`@` as `%40`, `:` as `%3A`).
5. **Run a read-only command** (`aws sts get-caller-identity`, `az account show`, `gcloud projects list --limit=1`).

## Common errors

| Symptom | Likely cause | Fix |
|---|---|---|
| AWS: `Failed to connect to proxy URL` | Wrong host or port, or the proxy is unreachable from this network | Run the curl test in step 2 |
| Any CLI: 407 or "Proxy Authentication Required" | Wrong username or password, or an unencoded `@` or `:` | Encode the password; copy the credentials again |
| EC2: credentials not found after setting the proxy | Metadata requests went to the proxy | Add `169.254.169.254` to `NO_PROXY` |
| gcloud ignores `https_proxy` | `proxy/*` properties are set and take priority | Check with `gcloud config list` and unset what you do not want |
| Access denied by policy even through the proxy | The allowed address does not match the exit | Compare the policy with the `ip` from step 2 |

## FAQ

**Does this change how my browser signs in?** No. These settings apply to the CLI process. Browser-based sign-in uses the browser's own network settings.

**Rotating or static for cloud CLIs?** Static. A source-address policy needs the same address every time, and rotating residential changes it between connections.

**Does the same setup work in CI?** Yes. Store the proxy URL as a CI secret and export it as `HTTPS_PROXY` in the job, or set the gcloud properties in a setup step. A static IP keeps the job's address the same on every run.

**Is there a free trial?** No. One static IP for one day costs $0.50; 30 days costs $7.99.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5), rotating residential and datacenter IPs. See [pricing](https://proxy302.com/pricing/) and the [locations page](https://proxy302.com/locations/). Related: [fixed IP for an API allowlist](../fixed-ip-for-api-allowlist/README.md), [PowerShell proxy settings](../powershell-invoke-webrequest-proxy/README.md).

*Facts checked 23 November 2026.*
