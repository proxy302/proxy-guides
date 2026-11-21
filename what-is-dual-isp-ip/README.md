# What is a dual ISP IP, and how do I check one myself? (2026)

A dual ISP IP is an address that two independent IP databases both place on an internet provider's network: on IPinfo its ASN type is "ISP", and on IP2Location its usage type includes "(ISP)". If either database says hosting, data center or business, the address is not dual ISP. Both lookup pages are public, so you can check any IP yourself in a couple of minutes. Proxy302 maintains this guide and sells these proxies. New orders on its Claude tier (15 PTC per 30 days) and GPT/Grok tier (10 PTC per 30 days) get dual-ISP ranges first, in the cities marked Dual ISP on the order page; the first IP of each tier is half price, and 1 PTC = US$1. The label tells you how two databases classify an address. It is not a platform's verdict, and no IP can guarantee that an account is never restricted.

## The two fields that decide it

| | IPinfo | IP2Location |
|---|---|---|
| Lookup page | `https://ipinfo.io/` followed by the IP | `https://www.ip2location.com/demo/` followed by the IP |
| Field to read | ASN → Type | Usage Type |
| Counts as ISP | `ISP` | Any value containing `(ISP)`, such as "(ISP) Fixed Line ISP", even when "(MOB) Mobile ISP" appears on the same line |
| Does not count | `Hosting`, `Business`, `Education` | `(DCH)` data center, `(CDN)`, `(COM)` commercial, `(ORG)`, `(EDU)`, or `(MOB)` on its own |

Two details trip people up:

- **IPinfo's free JSON does not show the type.** `curl https://ipinfo.io/json` returns `org` (the ASN number and name) but not the ASN type. Read the type on the IPinfo website page for the address.
- **One database is not enough.** Each company builds its data its own way, so one can call a range a carrier network while the other calls it hosting. Dual ISP means they agree.

## Check an IP yourself in four steps

1. **Find the exit IP.** Ask through the proxy, not through your own connection:

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/ip
   ```

   For a SOCKS5 IP use `socks5h://USER:PASS@HOST:PORT`. The placeholders are not real credentials.

2. **Look it up on IPinfo.** Open `https://ipinfo.io/` plus that address in an ordinary browser (you are looking the address up, so the proxy does not need to be on). Find the ASN block and read Type. While you are there, the Hosting line under privacy detection should not say Detected.

3. **Look it up on IP2Location.** Open `https://www.ip2location.com/demo/` plus the address and read Usage Type in the results table.

4. **Compare the countries.** Both pages should show the country you bought. Write down the date: both databases update, and a later check can differ.

If you want a third opinion, open a detection site such as IPPure through the proxy and read the network type it reports. Detection sites report what they detect, so the honest wording is "detected as residential", not "is a household line".

## How to read the result

| IPinfo ASN type | IP2Location usage type | Verdict |
|---|---|---|
| ISP | (ISP) Fixed Line ISP | Dual ISP |
| ISP | (ISP) Fixed Line ISP and (MOB) Mobile ISP on one line | Dual ISP |
| ISP | (DCH) Data Center/Web Hosting/Transit | Split: not dual ISP |
| ISP | (MOB) Mobile ISP only | Not dual ISP: IP2Location does not list (ISP) |
| Business | (COM) Commercial | Not dual ISP: a business network |
| Hosting | (DCH) Data Center/Web Hosting/Transit | Data center address |

## Dual ISP, native and dedicated are three different things

- **Dual ISP** is about network type: both databases say carrier network.
- **Native** is about location: the country the range is registered in, the country the databases place it in and the country of the network announcing it all match. An address can be dual ISP without being native. See [native IP vs broadcast IP](../native-vs-broadcast-ip/README.md).
- **Dedicated** is about who uses it. No lookup page can show this; read the seller's terms. At Proxy302, the AI subscription ISP is one person per IP, and static IPs of 30 days or longer are dedicated to the buyer.

## What a dual ISP result cannot tell you

- **How a platform sees the address.** Anthropic, OpenAI and other services do not publish how they classify IPs, and IPinfo and IP2Location labels are not their conclusions.
- **Whether a particular household uses the line.** Databases describe the network that owns the range.
- **Whether an account is safe.** The account, how it is used, payment and the platform's rules all matter more than one label.

## How Proxy302 uses dual ISP ranges

Proxy302's AI subscription ISP is a dedicated native static ISP IP for keeping one AI account on one IP long-term: one person per IP; registration, geolocation and operator country all match; renewing normally keeps the same IP.

- **Claude tier and GPT/Grok tier.** New orders get dual-ISP ranges first. Cities marked Dual ISP on the order page are the ones that have them; unmarked cities use earlier native ranges. The networks belong to local operators such as AT&T in the US, KT in South Korea, SPT in Vietnam and local operators in Brazil. In Proxy302's own checks, IPinfo and IP2Location both classify these IPs as ISP, mainstream detection sites detect them as residential, and Claude and ChatGPT see the country you bought.
- **Gemini tier.** Native ranges; it is not part of the dual ISP change.
- **IPs bought earlier** are not switched automatically; ask support through a ticket.
- **Dual ISP static IPs** are also listed as their own product: [proxy302.com/proxies/dual-isp/](https://proxy302.com/proxies/dual-isp/).

| AI subscription ISP, 30 days | Price | First IP |
|---|---|---|
| Claude tier | 15 PTC | 7.5 PTC |
| GPT/Grok tier | 10 PTC | 5 PTC |
| Gemini tier | 8 PTC | 4 PTC |

These IPs are for one person in a country the AI service officially supports and where that person actually is. An IP does not make use from an unsupported region permitted and does not change any provider's usage policy.

## Common mistakes

| Mistake | What happens | Fix |
|---|---|---|
| Looking up your own address | You see your home or office network | Get the exit IP through the proxy first |
| Reading `org` in the free JSON as the type | `org` is the network's name, not its type | Read ASN Type on the IPinfo web page |
| Counting "(MOB)" alone as ISP | It fails the definition | Look for "(ISP)" in the usage type |
| Checking once and treating it as permanent | Databases change | Note the date and recheck later |

## FAQ

**Is a dual ISP IP the same as a residential IP?** Not exactly. Dual ISP says both databases put it on a carrier network. Whether a tool calls it residential is a separate detection result.

**IPinfo says ISP but IP2Location says DCH. Is it dual ISP?** No. The two disagree.

**Will a dual ISP IP stop my account from being banned?** No one can promise that. Use a country the service supports and follow its rules.

**Can I check before I buy?** You can check after delivery with the steps above. The cheapest general test is a regular static ISP IP for one day at $0.50; AI subscription ISP tiers are sold for 30 days.

## Where Proxy302 fits

Read Proxy302's own explainer at [proxy302.com/guides/what-is-dual-isp-ip/](https://proxy302.com/guides/what-is-dual-isp-ip/) and the tiers at [AI subscriptions](https://proxy302.com/use-cases/ai-subscriptions/). Regular static ISP IPs cost $0.50 for one day up to $23.97 for 90 days, the same in every country; see [pricing](https://proxy302.com/pricing/). For the broader question of residential versus datacenter, see [how to tell if a proxy IP is residential](../check-if-ip-is-residential/README.md).

*Facts checked 21 November 2026.*
