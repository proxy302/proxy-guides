# How do I re-check my ChatGPT, Claude or Grok IP after each renewal?

Within a day of every renewal, run the same checks you ran on the day you bought the IP and compare the answers with that first record: the exit address is unchanged, the country is still the one you bought, IPinfo and IP2Location still both classify the address as ISP (dual ISP), the registry, geolocation and operator countries still agree (native), and the service's API still answers with a missing-key error rather than a region error. If every line matches, keep going; if one changed, sort it out before the next sign-in, using the table below. The ground rules do not change at renewal: use one stable IP per person, in a country where the service is officially offered and where you actually are. An IP does not make use from outside the provider's supported countries permitted, does not change the provider's usage policy or supported-country list, and cannot guarantee that an account is never restricted. Proxy302 maintains this guide and sells these proxies, including an AI subscription ISP for Claude and for ChatGPT/Grok.

## Why bother after a renewal

Renewing before the term ends keeps the same IP, so most months nothing changes. The re-check is for the things that can drift without you noticing:

- **The address itself.** If a renewal failed (auto-renew off, not enough balance), the IP was released at the end of its term, and a new purchase gets a different address.
- **Third-party databases.** IPinfo, IP2Location and the regional registries update their records. A label can change while the IP stays the same.
- **Your own setup.** A browser profile loses its proxy setting, a colleague turns on another system-wide network tool, or someone signs in from a laptop that bypasses the proxy.

A dated record turns "it feels different" into a line you can compare and, if needed, send to support.

## What to record on day one

| Field | Where it comes from | Example (documentation values) |
|---|---|---|
| Date | Today | 2026-11-26 |
| Exit IP | `ipinfo.io/json` through the proxy | 203.0.113.24 |
| Country | Same response, `country` | US |
| Network (ASN) | Same response, `org` | AS64500 Example ISP |
| IPinfo type | `https://ipinfo.io/<ip>`, ASN section, Type | ISP |
| IP2Location usage type | `https://www.ip2location.com/demo/<ip>`, Usage Type | (ISP) Fixed Line ISP |
| Registry country | `whois <ip>` | US |
| API front door | `api.anthropic.com` / `api.openai.com` without a key | 401 |

Keep this table next to the account it belongs to, not in a chat thread.

## The re-check, step by step

`USER`, `PASS`, `HOST` and `PORT` are placeholders for your IP's connection details. For a SOCKS5 IP, replace `http://` with `socks5h://` in the curl commands.

1. **Confirm the renewal went through.** The order page should show a new end date for the same IP. If the IP was released, skip to "the address changed" in the table below.
2. **Exit address and country.**

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   Compare `ip`, `country` and `org` with day one.
3. **Dual ISP.** Open `https://ipinfo.io/<ip>` and `https://www.ip2location.com/demo/<ip>` in any browser (you are looking the address up, so the proxy does not need to be on). Both must still say ISP: Type `ISP` on IPinfo, a Usage Type containing `(ISP)` on IP2Location. One of them saying hosting or `(DCH)` means the address is no longer dual ISP in that database's view.
4. **Native.** Run `whois <ip>` and compare the registry country with the country the two databases show and with the operator's country. Native means all three agree.
5. **The service's front door.**

   ```bash
   curl -sS -o /dev/null -w "%{http_code}\n" -x "http://USER:PASS@HOST:PORT" https://api.anthropic.com/v1/models
   curl -sS -o /dev/null -w "%{http_code}\n" -x "http://USER:PASS@HOST:PORT" https://api.openai.com/v1/models
   ```

   Both should print `401` (no key sent). From OpenAI, a `403` whose body mentions `unsupported_country_region_territory` means the location is not accepted. A 401 says nothing about your account; it only shows the network path works.
6. **Inside the app you actually use.** Open `https://ipinfo.io/json` in the same browser profile, extension or system proxy the account uses. The terminal tests the proxy; this step tests your setup.

## A small script that keeps the record for you

This uses only the Python standard library and works with an HTTP IP (Python's `urllib` does not speak SOCKS5). You type in the two database labels from step 3; the script fetches the rest, appends a row to `ip-checks.csv` and lists anything that differs from the first row.

```python
#!/usr/bin/env python3
"""Re-check a static proxy IP and compare it with the first check on record.

Usage (HTTP proxy IP; standard library only):
  PROXY_URL='http://USER:PASS@HOST:PORT' python3 recheck_ip.py IPINFO_TYPE IP2LOCATION_USAGE
Example:
  PROXY_URL='http://USER:PASS@HOST:PORT' python3 recheck_ip.py "ISP" "(ISP) Fixed Line ISP"
"""
import csv
import datetime
import json
import os
import sys
import urllib.error
import urllib.request

PROXY = os.environ.get("PROXY_URL")
LOG = os.environ.get("LOG_FILE", "ip-checks.csv")
if not PROXY or len(sys.argv) != 3:
    sys.exit(__doc__)

opener = urllib.request.build_opener(
    urllib.request.ProxyHandler({"http": PROXY, "https": PROXY}))


def status_and_body(url):
    try:
        with opener.open(url, timeout=20) as resp:
            return resp.status, resp.read().decode()
    except urllib.error.HTTPError as err:
        return err.code, err.read().decode(errors="replace")


_, body = status_and_body("https://ipinfo.io/json")
info = json.loads(body)
row = {
    "date": datetime.date.today().isoformat(),
    "ip": info.get("ip", ""),
    "country": info.get("country", ""),
    "org": info.get("org", ""),
    "ipinfo_type": sys.argv[1],
    "ip2location_usage": sys.argv[2],
    "anthropic_api": status_and_body("https://api.anthropic.com/v1/models")[0],
    "openai_api": status_and_body("https://api.openai.com/v1/models")[0],
}

first = None
if os.path.exists(LOG):
    with open(LOG, newline="") as f:
        first = next(csv.DictReader(f), None)

with open(LOG, "a", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=list(row))
    if first is None:
        writer.writeheader()
    writer.writerow(row)

print(json.dumps(row, indent=2))
if first is None:
    print("First record saved to", LOG)
else:
    changed = [k for k in row if k != "date" and str(row[k]) != first[k]]
    if not changed:
        print("No change since the first check on", first["date"])
    for k in changed:
        print(f"CHANGED {k}: {first[k]} -> {row[k]}")
```

We ran it on Python 3.9 through a local test proxy with a username and password on 26 November 2026. A month later the end of the output looks like this when nothing moved (documentation address shown):

```text
  "ip": "203.0.113.24",
  "country": "US",
  ...
No change since the first check on 2026-11-26
```

and like this when a database relabelled the address:

```text
CHANGED ip2location_usage: (ISP) Fixed Line ISP -> (DCH) Data Center
```

If the password contains `@`, `:` or `#`, percent-encode it in `PROXY_URL` (`@` becomes `%40`).

## When a result changes

| What changed | What it usually means | What to do before the next sign-in |
|---|---|---|
| Exit IP | The term ended and the IP was released, or the app is not using the proxy you think it is | Check the order page. A released IP is gone: treat the new one as a new IP and run every day-one check on it. If the service then asks you to confirm a sign-in, do so honestly |
| Country | The profile is not using the proxy, another network tool sits on top, or a geolocation database moved the address | Compare the terminal result with the in-app result. If the terminal still shows the right country, the problem is local. If both show a new country for the same IP, contact support with the date and screenshots |
| IPinfo or IP2Location label | A database reclassified the range | Note the date, keep screenshots of both pages, contact support. Do not start hopping between IPs to chase a label |
| Registry country (native) | The range's registration was updated | Contact support with the `whois` output |
| API answers 403 region error | The location is not accepted at network level | Do not sign in. Check the country against the provider's list, then contact support |
| Detection site wording | One site's own label changed | Note it; it is one site's view, not the provider's |

The one thing not to do is to fix a changed result by switching the account between several IPs. Steadiness is the point of a fixed IP; a different address every few days works against it.

## Renewal habits that make the re-check boring

- Turn on auto-renew for the IP and keep enough balance for it.
- Put the renewal date in the same record as the check results.
- Re-check within a day of each renewal, and also whenever the service starts asking you to confirm sign-ins.
- Keep one layer only: the IP in one browser profile, extension or system proxy, with any other system-wide network tool switched off.

## Proxy302's AI subscription ISP

The AI subscription ISP is a dedicated native static ISP IP picked per AI service and sold for 30 days. Static IPs of 30 days or longer are dedicated to the buyer. Before delivery, the IP is connected and its country is checked.

New orders on the Claude tier and the GPT/Grok tier get dual ISP ranges first; the cities marked "Dual ISP" on the order page are the ones that have them. In Proxy302's own checks, IPinfo and IP2Location both classify these IPs as ISP, mainstream detection sites detect them as residential, and Claude and ChatGPT see the country you bought. The networks belong to local operators such as AT&T in the US and KT in South Korea. Cities on the order page (23 November 2026): Claude tier, US cities, Frankfurt, Chiba, Osaka and London; GPT/Grok tier, US cities, London, Chiba, Osaka, Paris and Seoul.

| Tier, 30 days | Regular price | First IP of the tier |
|---|---|---|
| Claude (Claude Pro/Max) | 15 PTC | 7.5 PTC |
| GPT/Grok (ChatGPT Plus/Pro, SuperGrok) | 10 PTC | 5 PTC |

*Black Friday, 24 Nov to 1 Dec 2026: from the second IP, the Claude tier costs 12 PTC and the GPT/Grok tier 8 PTC per 30 days, and the first IP of each tier stays at half the regular price.*<!-- bf2026 -->

1 PTC = US$1. Order on the [AI subscriptions page](https://proxy302.com/use-cases/ai-subscriptions/); all prices on [pricing](https://proxy302.com/pricing/).

## FAQ

**Does renewing give me a new IP?** No. Renewing before the end date keeps the same address. Only an IP whose term ends without renewal is released.

**How often should I re-check?** After each renewal, so roughly every 30 days, plus any time sign-in prompts start appearing more often.

**The IP is unchanged but one database now says hosting. Is my account at risk?** Nobody outside the provider knows how it weighs such labels. Record it and contact support; do not move the account to a random other IP in the meantime.

**Can I re-check through SOCKS5?** Yes with curl (`socks5h://USER:PASS@HOST:PORT`). The Python script above needs an HTTP IP.

**Do these checks tell me whether I am allowed to use the service?** No. That depends on where you are and on the provider's rules: [Anthropic's supported countries](https://www.anthropic.com/supported-countries), [OpenAI's supported countries](https://help.openai.com/en/articles/7947663-chatgpt-and-openai-api-supported-countries-and-territories) and [xAI's terms](https://x.ai/legal/terms-of-service). If a provider asks you to verify your identity or phone number, complete it honestly; an IP is not a substitute.

## Related

- [How do I check an IP before I use it for ChatGPT, Claude or Grok?](../check-ip-before-ai-subscription/README.md)
- [What is a dual ISP IP, and how do I check one myself?](../what-is-dual-isp-ip/README.md)
- [Native IP vs broadcast IP](../native-vs-broadcast-ip/README.md)

*Facts checked 26 November 2026.*
