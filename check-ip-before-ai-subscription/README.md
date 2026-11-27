# How do I check an IP before I use it for ChatGPT, Claude or Grok?

Run five checks through the proxy before the first sign-in: the exit country is one the service officially offers and the one you are actually in; IPinfo and IP2Location both put the address on an internet provider's network (dual ISP); the registration, geolocation and operator countries match (native); the address stays the same across repeated requests; and the service's API answers from that address with a login error rather than a region error. Use one stable IP per person, in a country where the service is officially offered and where you actually are. Passing every check does not make use from outside the provider's supported countries permitted, does not change the provider's usage policy or supported-country list, and cannot guarantee that an account is never restricted. Proxy302 maintains this guide and sells these proxies, including an AI subscription ISP for Claude and for ChatGPT/Grok.

## Before you check anything: the country

Each provider decides where its service is offered. Read the source, not a forum:

- Claude: [Anthropic's supported countries and regions](https://www.anthropic.com/supported-countries)
- ChatGPT: [OpenAI's supported countries and territories](https://help.openai.com/en/articles/7947663-chatgpt-and-openai-api-supported-countries-and-territories)
- Grok: [xAI's consumer terms of service](https://x.ai/legal/terms-of-service) and the availability notice in the Grok app. We did not find a single country list for Grok in the form the other two publish.

Pick the country you live and work in. If it is outside the provider's supported countries, no IP check on this page changes that.

## The five checks

You need the proxy's host, port, username and password. `USER`, `PASS`, `HOST` and `PORT` below are placeholders. For a SOCKS5 IP, replace `http://` with `socks5h://`.

### 1. Exit address and country

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

Read `ip`, `country` and `org`. A healthy answer looks like this (documentation addresses, not a real IP):

```json
{
  "ip": "203.0.113.24",
  "city": "Los Angeles",
  "region": "California",
  "country": "US",
  "org": "AS64500 Example ISP"
}
```

`country` must be the country you bought and the one you are in. Write down `ip`; the next steps look it up.

### 2. Dual ISP: two databases, one answer

Open these two pages in any browser, with the address from step 1 on the end. You are looking the address up, so the proxy does not need to be on.

| Database | Page | Field | Pass |
|---|---|---|---|
| IPinfo | `https://ipinfo.io/203.0.113.24` | ASN → Type | `ISP` |
| IP2Location | `https://www.ip2location.com/demo/203.0.113.24` | Usage Type | contains `(ISP)`, for example "(ISP) Fixed Line ISP" |

The address is dual ISP only when both pass. `Hosting` on IPinfo, or `(DCH)`, `(COM)` or `(MOB)` without `(ISP)` on IP2Location, means it is not. The free `ipinfo.io/json` answer from step 1 does not include the type, so use the web page. For the full reading guide, see [what a dual ISP IP is](../what-is-dual-isp-ip/README.md).

### 3. Native: three countries that should agree

```bash
# registry record for the address (203.0.113.24 is a documentation address)
whois 203.0.113.24 | grep -iE '^(country|netname|org-?name):'

# registry record for the network that announces it (AS64500 is a documentation number; use the one from "org")
whois AS64500 | grep -iE '^(country|as-?name|org-?name):'
```

Compare three things: the country in the address's registry record, the country IPinfo and IP2Location show, and the country of the network (ASN) that operates the range. Native means all three are the same. If the registry says one country and the databases say another, detection sites usually label the IP "broadcast". More detail: [native IP vs broadcast IP](../native-vs-broadcast-ip/README.md).

### 4. Stability

Repeat step 1 three times a few minutes apart. A static IP returns the same `ip` every time. If it changes, you have a rotating gateway or a session setting, which is the wrong product for a signed-in account.

### 5. The service's own front door

These requests carry no key, so they only test the network path from the proxy:

```bash
curl -sS -x "http://USER:PASS@HOST:PORT" https://api.anthropic.com/v1/models
curl -sS -x "http://USER:PASS@HOST:PORT" https://api.openai.com/v1/models
curl -sS -x "http://USER:PASS@HOST:PORT" https://api.x.ai/v1/models
```

From a working location each one answers 401 with a message that a key or credentials are missing (we got exactly that on 23 November 2026). For OpenAI, a 403 with `unsupported_country_region_territory` means the location is not accepted. A 401 tells you nothing about your account; it only shows that the connection works.

### Optional: a detection site

Open a mainstream IP detection site through the proxy and read the network type it reports. Report it as what it is: "detected as residential", not a statement about whose line it is.

## Reading the results

| Check | Pass | Fail: what to do |
|---|---|---|
| Country | Matches your purchase and where you are | Wrong country: confirm the order; the browser or app may not be using the proxy |
| Dual ISP | IPinfo `ISP` and IP2Location `(ISP)` | One says hosting or commercial: the address is not dual ISP; contact support with both screenshots |
| Native | Registry, database and operator country agree | Registry country differs: a broadcast address |
| Stability | Same `ip` each time | Changes: you are on a rotating product |
| Front door | 401 asking for a key | 403 region error, or a timeout: check the country and the proxy settings first |

## Last step: check inside the app you will use

The terminal checks above test the proxy. Your browser or desktop app is a separate program. Put the IP into one place only (one browser profile, a browser extension, or the system proxy), keep any other system-wide network tool switched off, open `https://ipinfo.io/json` in that same browser or profile, and only then sign in. From then on, that account uses that IP.

## What none of these checks can tell you

- **How a provider classifies the address.** Anthropic, OpenAI and xAI do not publish their method. IPinfo and IP2Location labels are those companies' data, not the provider's verdict.
- **Whether your account is in good standing.** Payment, sign-up details, how the account is used and the provider's rules matter more than any one label.
- **Whether you are allowed to use the service.** That depends on where you are, not where the IP is.

If a provider asks you to verify your phone number or identity, complete it honestly. An IP is not a substitute for that, and we do not help with getting around it.

## Proxy302's AI subscription ISP

The AI subscription ISP is a dedicated native static ISP IP picked per AI service, sold for 30 days. Static IPs of 30 days or longer are dedicated to the buyer. Before delivery, the IP is connected and its country is checked.

- **Claude tier and GPT/Grok tier:** new orders get dual-ISP ranges first; the cities marked Dual ISP on the order page are the ones that have them. In Proxy302's own checks, IPinfo and IP2Location both classify these IPs as ISP, mainstream detection sites detect them as residential, and Claude and ChatGPT see the country you bought. The networks belong to local operators such as AT&T in the US and KT in South Korea.
- **Cities (order page, 23 November 2026):** Claude tier: US cities, Frankfurt, Chiba, Osaka, London. GPT/Grok tier: US cities, London, Chiba, Osaka, Paris, Seoul.

| Tier, 30 days | Regular price | First IP of the tier |
|---|---|---|
| Claude (Claude Pro/Max) | 15 PTC | 7.5 PTC |
| GPT/Grok (ChatGPT Plus/Pro, SuperGrok) | 10 PTC | 5 PTC |

*Black Friday, 24 November to 4 December 2026 (UTC+8): from the second IP, the Claude tier costs 12 PTC and the GPT/Grok tier 8 PTC per 30 days, and the first IP of each tier stays at half the regular price. The table shows regular prices; [pricing](https://proxy302.com/pricing/) shows what applies on the day.*<!-- bf2026 -->

1 PTC = US$1. Order on the [AI subscriptions page](https://proxy302.com/use-cases/ai-subscriptions/).

## FAQ

**Can I run these checks before I buy?** You need an IP to check. There is no free trial; the cheapest way to rehearse the steps is a regular static ISP IP for one day at $0.50. AI subscription ISP tiers are sold for 30 days.

**One database says ISP, the other says data center. Is that close enough?** No. Dual ISP means both agree.

**Do I need the dual ISP result if the country is right?** The country is the requirement; dual ISP and native describe how the address is classified and registered. They help you avoid an address that looks like a hosting company's.

**Can two people use one IP if both pass the checks?** Use one IP per person. Several people on one address look like one account being used from many hands.

**My check passed last month and fails today.** Databases update. Note the date of every check, and contact support with the new screenshots.

## Related

- [Which IP to use for Claude and ChatGPT](../which-ip-for-claude-and-chatgpt/README.md)
- [What is a dual ISP IP](../what-is-dual-isp-ip/README.md)
- [How to tell if a proxy IP is residential](../check-if-ip-is-residential/README.md)

*Facts checked 23 November 2026.*
