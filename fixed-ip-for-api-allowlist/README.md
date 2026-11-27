# Which proxy do I need for an API that only accepts allowlisted IP addresses?

Use a static proxy IP, not rotating traffic. An allowlist holds a fixed list of addresses, and a rotating gateway sends each new connection out from a different one, so most of your calls would be refused. Buy one static IP (two if you want a spare), give the API provider the address it actually exits from, and route only the API calls through it. A static datacenter IP is usually enough for server-to-server APIs; choose a static residential (ISP) IP when the service treats hosting networks differently or expects a consumer connection. Keep the IP renewed: when a static IP's term ends it is released, and a new purchase gets a new address that must be allowlisted again.

*Proxy302 maintains this guide and sells these proxies. The Python and curl commands were run against a throwaway local proxy in November 2026.*

## Static or rotating for this job

| Need | Proxy type | Why | Proxy302 price |
|---|---|---|---|
| Partner API, payment gateway, admin panel with an IP allowlist | Static datacenter IP | Fixed address, lowest cost | $0.25 for 1 day, $3.99 for 30 days |
| Allowlisted service that refuses hosting networks, or a consumer site account | Static residential (ISP) IP | Fixed address registered to a consumer ISP | $0.50 for 1 day, $7.99 for 30 days |
| Many independent requests, no allowlist | Rotating residential | New exit per connection is fine | from $1.50/GB |

A rotating gateway is the wrong tool here even with a sticky session: sticky sessions hold an address for a limited time, while an allowlist entry is meant to last months.

## Step 1: find the address the API will see

The host you connect to and the address the API sees can differ. Ask the internet, through the proxy:

```bash
curl -sS -x http://USER:PASS@HOST:PORT https://api.ipify.org; echo
```

Send that exact address to the API provider, or paste it into their allowlist page. Run the command twice, a few minutes apart; a static IP returns the same address both times.

## Step 2: route only the API calls through the proxy

Sending all of a server's traffic through the proxy adds a hop to everything and spends traffic you do not need to. Point just the client for that API at it.

Python with requests:

```python
import os
import requests

PROXY = os.environ["API_PROXY"]               # http://USER:PASS@HOST:PORT
PROXIES = {"http": PROXY, "https": PROXY}

def partner_get(path, **kwargs):
    """Every call to the allowlisted API leaves through the static IP."""
    return requests.get(f"https://api.partner.example{path}",
                        proxies=PROXIES, timeout=30, **kwargs)

print("exit IP:", requests.get("https://api.ipify.org", proxies=PROXIES, timeout=20).text)
```

Command-line tools and many SDKs read the standard variables. For a single command:

```bash
HTTPS_PROXY=http://USER:PASS@HOST:PORT ./sync-orders.sh
```

For a systemd service, put the variable in a drop-in so it applies only to that unit:

```ini
# sudo systemctl edit partner-sync.service
[Service]
Environment="HTTPS_PROXY=http://USER:PASS@HOST:PORT"
```

Run `sudo systemctl restart partner-sync` afterwards. Store the real credentials in your secrets manager or an `EnvironmentFile=` readable only by root, not in the repository.

Other languages: [Go](../go-http-proxy/README.md), [Node.js](../nodejs-proxy/README.md), [Java](../java-httpclient-proxy/README.md), [C#](../csharp-httpclient-proxy/README.md), [PHP](../php-curl-proxy/README.md).

## Step 3: plan for renewals and failures

- **Turn on auto-renew** for the IP, so the address never lapses. At Proxy302 it can be turned on per IP.
- **Allowlist two IPs** if the API allows it, and switch to the second when the first fails. Two static datacenter IPs for 30 days cost $7.98 in total.
- **Write the exit IP into your runbook**, next to the API account it belongs to, so the next person knows why it exists.
- **Check the address after any change** with the curl command in Step 1.

## FAQ

**Can I give the provider the proxy host name instead of an IP?** Usually not. Allowlists match the source address of incoming connections, which is the exit IP, and most accept only IP addresses or ranges.

**Does the proxy see my API traffic?** For `https://` APIs the proxy sees the destination host and the amount of data, not the content, because TLS runs end to end inside the tunnel.

**What if the API also checks the country?** Buy the static IP in the country the provider expects. At Proxy302 static IPs cost the same in every country.

## Where Proxy302 fits

Proxy302 sells static datacenter IPs ($0.25 for a day, $3.99 for 30 days) and static residential (ISP) IPs ($0.50 for a day, $7.99 for 30 days, $15.98 for 60 days, $23.97 for 90 days). A static residential IP is HTTP or SOCKS5, chosen at purchase. There is no free trial; a one-day IP is the cheapest test. See [pricing](https://proxy302.com/pricing/).

*Black Friday, 24 November to 4 December 2026 (UTC+8): 60-day static ISP IPs are US$14.00 and 90-day IPs US$18.00 (1 to 30 days unchanged). Prices elsewhere on this page are the regular prices; [pricing](https://proxy302.com/pricing/) shows what applies on the day.*<!-- bf2026 -->

*Facts checked 20 November 2026.*
