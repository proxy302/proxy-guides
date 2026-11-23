# How do I set a proxy on an iPhone (Wi-Fi HTTP proxy) and check it works?

Open Settings, tap Wi-Fi, tap the (i) next to the network you are on, scroll to HTTP Proxy, tap Configure Proxy and choose Manual. Enter the proxy's server and port, turn on Authentication, enter the username and password, and tap Save. Then open `https://ipinfo.io/json` in Safari: the `ip` and `country` should be the proxy's, not yours. The iPhone's built-in proxy is HTTP only and applies to that one Wi-Fi network, not to mobile data. Proxy302 maintains this guide and sells these proxies: a static ISP IP bought as HTTP costs $0.50 for one day or $7.99 for 30 days, the same in every country, and static IPs of 30 days or longer are dedicated to the buyer.

## What you need

| Item | Where to find it | Note |
|---|---|---|
| Server | Your proxy's host | An IP address or a hostname |
| Port | Your proxy's port | Numbers only |
| Username and password | Your proxy's credentials | Entered after turning on Authentication |
| Protocol | Chosen when you bought the proxy | Must be HTTP; this screen cannot use SOCKS5 |

At Proxy302 these four values are on the console's My assets page, and the protocol of a static IP is fixed at purchase.

## Steps

1. **Join the Wi-Fi network** you will use the proxy on.
2. **Settings → Wi-Fi → (i)** next to that network.
3. **Scroll to HTTP Proxy → Configure Proxy → Manual.**
4. **Fill in Server and Port.**
5. **Turn on Authentication** and fill in Username and Password.
6. **Tap Save** in the top right.
7. **Verify.** Open Safari and go to `https://ipinfo.io/json`. Check `ip`, `country` and `org`. If you see your own address, the proxy is not in use.

To switch it off, go back to the same screen and choose Off.

## Test the credentials on a computer first

If the phone refuses to load pages, find out whether the problem is the proxy or the phone. From a computer (placeholders, not real values):

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

If curl works and the iPhone does not, recheck the fields on the phone. If curl fails too, the problem is the credentials, the port, or the protocol you bought.

## What follows the setting and what does not

- **Safari and most apps that use the system's networking** send web traffic through the proxy.
- **Some apps use their own networking** and may ignore it. If an app shows your IP somewhere, check there.
- **Mobile data is not covered.** iOS has no manual proxy screen for cellular. Organisations can set a global HTTP proxy on supervised devices through device management; that is an IT setup, not a personal one.
- **Each Wi-Fi network has its own setting.** Join another network and the proxy is off there until you set it again.
- **iCloud Private Relay and VPN profiles** add another layer. Turn them off while you test so you know which exit you are looking at.

For a SOCKS5 proxy you need an app that accepts SOCKS5 with a username and password; Proxy302 has a [Shadowrocket setup guide](https://proxy302.com/guides/shadowrocket/).

## Common problems

| Symptom | Cause | Fix |
|---|---|---|
| Pages never load | Wrong port, or a SOCKS5 proxy typed into this HTTP screen | Use an HTTP proxy and the right port |
| A login box keeps coming back | Wrong username or password, or Authentication left off | Retype both; test with curl |
| ipinfo shows your own IP | You are on mobile data, or on a different Wi-Fi | Join the network you configured |
| ipinfo shows an unexpected country | Another layer (Private Relay, a VPN profile) is active | Turn it off and check again |
| Worked yesterday, not today | The proxy's term ended | Renew before the end date; static IPs can auto-renew |

## Which IP to use on an iPhone

| Use | Option | Proxy302 price |
|---|---|---|
| One account you stay signed in to (social, shop, AI app) | Static ISP IP, bought as HTTP | $7.99 per 30 days; $0.50 for one day |
| Claude, ChatGPT or Gemini for one person | AI subscription ISP, 30 days | Claude tier 15 PTC, GPT/Grok 10 PTC, Gemini 8 PTC; first IP half price (1 PTC = US$1) |
| Browsing pages from another country without signing in | Rotating residential | $1.50/GB |

*Black Friday, 24 November to 1 December 2026 (UTC+8): from the second IP, the AI subscription ISP costs 12 PTC (Claude tier), 8 PTC (GPT/Grok tier) and 6 PTC (Gemini tier) per 30 days, and the first IP stays at half the regular price. Prices elsewhere on this page are the regular prices; [pricing](https://proxy302.com/pricing/) shows what applies on the day.*<!-- bf2026 -->

For AI services, use a country where the service is officially offered and where you actually are. An IP does not make use from an unsupported region permitted.

## FAQ

**Does the proxy apply to every app?** Not guaranteed. Apps that use iOS networking follow it; check inside the app you care about.

**Can I use one proxy for two accounts on the same phone?** For one account per platform, one IP is fine. For several accounts on the same platform, read that platform's rules first; see [which proxy for running multiple accounts](../proxy-for-multiple-accounts/README.md).

**Why does Safari ask for the password again?** The saved credentials did not match what the proxy expects; retype them on the Configure Proxy screen.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs from 1 to 90 days with HTTP or SOCKS5 chosen at purchase, AI subscription ISP IPs per AI service, and rotating residential traffic. Prices: [proxy302.com/pricing/](https://proxy302.com/pricing/). On a Mac or PC, see [Chrome with its own proxy](../chrome-proxy-one-window/README.md).

*Facts checked 21 November 2026.*
