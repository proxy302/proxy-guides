# What proxy should I use for Claude (claude.ai and the Claude apps)?

Use one stable, fixed IP per person, in a country where Anthropic officially offers Claude and where you actually are, and use it from one place: one browser profile, one computer's system proxy or one phone. Proxy302 maintains this guide and sells these proxies. Proxy302's Claude tier is a dedicated native static ISP IP designed for keeping a Claude account on one IP long-term: one person per IP; registration, geolocation and operator country all match; new orders get dual-ISP ranges first, which both IPinfo and IP2Location classify as an ISP network (cities marked Dual ISP on the order page); renewing normally keeps the same IP; 15 PTC (1 PTC = US$1) per 30 days, first IP half price. An IP does not make use from an unsupported region permitted, does not change Anthropic's Usage Policy or its [Supported Regions Policy](https://www.anthropic.com/supported-countries), and cannot guarantee that an account is never restricted.

*Black Friday, 24 November to 1 December 2026 (UTC+8): 60-day static ISP IPs are US$14.00 and 90-day IPs US$18.00 (1 to 30 days unchanged); from the second IP, the AI subscription ISP costs 12 PTC (Claude tier), 8 PTC (GPT/Grok tier) and 6 PTC (Gemini tier) per 30 days, and the first IP stays at half the regular price. Prices elsewhere on this page are the regular prices; [pricing](https://proxy302.com/pricing/) shows what applies on the day.*<!-- bf2026 -->

## Read the policy before buying anything

Anthropic's Supported Regions Policy lists the countries and regions where Claude.ai and its other products are available; anything not listed is unsupported. The policy is about where a person is physically located, and it also covers companies incorporated, headquartered or majority-controlled in an unsupported region. A proxy changes none of that, and it does not change sign-up or payment requirements either.

What a fixed IP is for:

- **Replacing a shared VPN exit or a rotating connection.** Those make one account look like many people, or like one person hopping between addresses.
- **A company network or allowlist.** An organisation that limits access to known addresses needs one egress IP per site or person. Your own organisation's admin adds the address; Proxy302 does not register IPs with Anthropic, its IPs are not on any allowlist, and an allowlisted IP does not exempt an account from identity or phone verification.
- **Travel between supported countries.** A team member keeps signing in through the company's one address.

## Where to put the proxy in each Claude app

| Where you use Claude | How the proxy is applied | Protocol |
|---|---|---|
| claude.ai in a browser | One browser profile with a proxy extension that supports a username and password | HTTP (Chromium cannot authenticate to SOCKS5) |
| Claude desktop app | Follows the system proxy, as far as Anthropic documents it (see below) | HTTP |
| iPhone and iPad app | Wi-Fi settings, Configure Proxy, Manual, with Authentication on | HTTP |
| Android app | The built-in Wi-Fi proxy screen has host and port fields only, no login | Check before you buy |
| Claude Code (terminal) | `HTTPS_PROXY` or the `env` block in its settings file; SOCKS is not supported | HTTP, see [Claude Code proxy settings](../claude-code-proxy-settings/README.md) |

**Desktop app.** Anthropic's [network proxy notes](https://claude.com/docs/third-party/claude-desktop/network-proxy) cover its enterprise third-party deployment: the app follows the operating system's proxy settings, skips a SOCKS proxy, and has no prompt for a proxy username and password, so Anthropic suggests a local forwarding proxy that handles the login. We found no separate note for the personal app. Check the exit before you rely on it.

## Steps

1. **Pick the country.** It must be on Anthropic's list and be where you are. Do not switch countries later.
2. **Buy one IP per person, as HTTP.** Every Claude surface above either needs HTTP or works best with it.
3. **Choose one place for it.** Browser profile, system proxy or phone. Turn off any VPN or second proxy.
4. **Check the exit on that device.** Open `https://ipinfo.io/json`; `country` must match your purchase.
5. **Test the network path** without an account (placeholders, not real credentials):

   ```bash
   curl -sS -o /dev/null -w "%{http_code}\n" \
     -x "http://USER:PASS@HOST:PORT" https://api.anthropic.com/v1/models
   ```

   A 401 means the request reached Anthropic and only lacks a key. It tells you nothing about an account.
6. **Sign in and keep it that way.** Plan renewal before day 30.

## Common mistakes

| Mistake | Result | Fix |
|---|---|---|
| Buying a SOCKS5 IP for Claude Code or a Chrome extension | Connection or login fails | Buy the IP as HTTP |
| Pointing the desktop app at a proxy that needs a password | Requests fail; there is no login prompt | Use a local forwarder, or set the proxy in the browser instead |
| Exit country not where you are | Conflicts with the Supported Regions Policy | Use the supported country you are in |
| Proxy on, then VPN on top | The account sees changing addresses | One layer only |
| One login shared by a team | Several people behind one account | One account and one IP per person |

## FAQ

**Does a proxy let me use Claude in an unsupported region?** No. The policy follows where you are and who controls your organisation.

**Claude tier or a regular static ISP IP?** Both stay fixed. The Claude tier is a dedicated native IP for one person, with dual-ISP ranges first on new orders, and its first IP is half price; a regular static ISP IP comes in 1 to 90 day terms if you want to test first.

**Can one HTTP IP serve claude.ai and Claude Code for the same person?** Yes; both can use the same HTTP proxy.

**Is there a free trial?** No. The cheapest test is a static IP for one day at $0.50.

## Where Proxy302 fits

Proxy302 sells AI subscription ISP IPs for 30 days: Claude tier 15 PTC, GPT/Grok tier 10 PTC, Gemini tier 8 PTC, with the first IP of each tier 50% off. Regular static ISP IPs cost $0.50 for one day up to $23.97 for 90 days, the same in every country, with HTTP or SOCKS5 chosen at purchase. See [pricing](https://proxy302.com/pricing/), [proxy302.com](https://proxy302.com) and the [Chinese site](https://proxy302.com/zh/). For both AI services side by side, read [which IP to use for Claude and ChatGPT](../which-ip-for-claude-and-chatgpt/README.md).

*Facts checked 21 November 2026.*
