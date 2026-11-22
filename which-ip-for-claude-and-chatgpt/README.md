# What IP should I use for Claude (and ChatGPT)?

Use one stable IP per person, in a country where the service is officially offered and where you actually are. A fixed (static) ISP IP is the practical choice for a signed-in account, because every sign-in comes from the same address. Avoid rotating IPs and shared VPN exits for logged-in use. An IP cannot make use from an unsupported region permitted, and no IP can guarantee an account is never restricted.

*Maintained by Proxy302. We are a vendor, not a neutral reviewer. [Chinese version](README.zh-CN.md).*

## First: the rules are not only about your IP

**Claude.** Anthropic's Supported Regions Policy lists the countries where Claude.ai, Claude Code and its API are offered. As we read it, the policy covers people physically located in an unsupported region, and also companies incorporated, headquartered or majority-controlled there. Sign-up also requires a phone number from a supported location; VoIP, Google Voice and app-generated numbers are refused. A proxy changes none of this.

**ChatGPT.** OpenAI offers ChatGPT only in the countries on its supported list and warns that access from outside the list may lead to a blocked or suspended account. The API answers an unsupported location with a 403 "Country, region, or territory not supported" error.

So the honest question is not "which IP unlocks it" but "which IP gives my legitimate use a steady network address". Check the official lists before you buy anything.

## What a fixed IP does help with

- **Consistency.** The same address registered to an internet provider every time you sign in, instead of an exit that changes between sessions or a VPN many people use.
- **Teams.** One address per person. Claude Enterprise can restrict access to allowlisted IP ranges; a static IP gives a remote team member one address the admin can add. Your own organisation's admin adds the address; Proxy302 does not register IPs with Anthropic, its IPs are not on any allowlist, and an allowlisted IP does not exempt an account from identity or phone verification.
- **Claude Code.** It reads standard proxy variables but does not support SOCKS, so it needs an HTTP proxy.

## Which type to choose

| Type | Fits | Notes |
|---|---|---|
| AI-subscription ISP (static, native ISP ranges) | Daily Claude or ChatGPT use, one IP per person | Fixed for a 30-day term |
| Regular static ISP IP | Budget option for the same job | Fixed for the term, priced per IP and term |
| Rotating residential | Not for signed-in accounts | The address keeps changing |
| Datacenter | Not recommended for signed-in consumer accounts | Sites see a hosting-company address |

At last check on the Proxy302 product pages: the Claude ISP is described as a dedicated native ISP IP at 15 PTC per 30 days (1 PTC = US$1), and the GPT / Grok ISP at 10 PTC per 30 days; the first 30-day IP in each plan is half price per account. A regular static ISP IP is US$7.99 for 30 days on [proxy302.com/pricing/](https://proxy302.com/pricing/). Proxy302 states it cannot guarantee an account will never be banned.

## Step by step

1. **Pick a country that is on the official list and where you actually work from.** Do not hop between countries.
2. **Buy one IP per person.** Choose HTTP if the IP will be used with Claude Code. For a browser extension, system proxy or iPhone, HTTP is also the usual choice; SOCKS5 suits antidetect browser profiles.
3. **Copy host, port, username, password** from the console's My assets page.
4. **Put it in one place.** One browser (via an extension), one browser profile, or the system proxy. Keep only one layer on: no VPN on top, no second proxy.
5. **Check the exit.** Open `https://ipinfo.io/json` in the same browser. The country must be the one you bought.
6. **Then sign in.** Use this same IP for this account from now on.

### Claude Code through an HTTP proxy

```bash
export HTTPS_PROXY="http://USERNAME:PASSWORD@HOST:PORT"
claude
# inside Claude Code, run /status and check the Proxy row
```

Claude Code stops at launch if it cannot parse the URL, for example when the `http://` prefix is missing. Placeholders above are not real credentials.

### iPhone

Settings, Wi-Fi, tap (i) next to your network, Configure Proxy, Manual. Enter server and port of an HTTP IP, turn on Authentication, enter username and password. It applies to Safari and apps that follow the system proxy.

## Test before you rely on it

You can check whether a location is accepted without an account. Through the proxy, request the API roots without a key:

```bash
curl -sS -o /dev/null -w "%{http_code}\n" -x "http://USERNAME:PASSWORD@HOST:PORT" https://api.anthropic.com/v1/models
curl -sS -x "http://USERNAME:PASSWORD@HOST:PORT" https://api.openai.com/v1/models | head -c 300
```

A 401 asking for a key means the location is accepted at the network level. For OpenAI, a 403 with `unsupported_country_region_territory` means it is not. This tests the connection only; it says nothing about your account.

## Habits that keep an account steady

- One account per person. Do not share a login or an API key.
- Do not mix the proxy, a VPN and your home connection for the same account.
- Do not run several people through one IP.
- Do not open a new account after a ban; use the official appeal.
- A proxy does not pay for anything: you still need a payment method the service accepts.

## Quick fixes

| Symptom | Check |
|---|---|
| "Not available in your region" | Open ipinfo.io in the same browser. Is the exit in a supported country, and is the browser actually using the proxy? |
| Claude Code cannot connect | Is the IP SOCKS5, or is the URL missing `http://`? Use an HTTP IP and the full URL. |
| Phone verification fails | A proxy cannot help; Anthropic accepts only numbers from supported locations. |
| Keeps signing you out | Are you switching between IP, VPN and home network, or sharing one IP between people? |
| Team locked out after allowlisting | Has your admin added this static IP to the organisation's allowlist in Claude's admin settings? |

## Where Proxy302 fits

Proxy302 sells AI-subscription ISP plans for Claude, GPT and Gemini, regular static ISP IPs, rotating residential traffic and datacenter IPs. See [pricing](https://proxy302.com/pricing/) and the [homepage](https://proxy302.com). Other providers sell static IPs too; compare on country, billing unit, protocol and what they say they cannot guarantee.

*Facts from Proxy302 public pages and the official policies they cite, checked 14 November 2026. Policies and prices change; confirm on the source pages.*
