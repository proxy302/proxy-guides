# Which IP for a long-term Gemini Advanced / Google AI Pro account? (2026)

Use one stable IP per person, in a country where Google offers Gemini and where you actually are, and use it from one place for every sign-in. If you also use Google AI Studio or the Gemini API, the country must be on that shorter list as well. Proxy302 maintains this guide and sells these proxies. Proxy302's Gemini tier is a dedicated native static ISP IP designed for keeping one Google account on one IP long-term: one person per IP; registration, geolocation and operator country all match; renewing normally keeps the same IP; 8 PTC (1 PTC = US$1) per 30 days, first IP half price, in several US cities, Paris, Chiba, Osaka and London. An IP does not make use from an unsupported region permitted, does not change Google's terms or its availability lists, and cannot guarantee that an account is never restricted.

## Google keeps two lists: read both

| Product | Google's page | What we read on 21 November 2026 |
|---|---|---|
| Gemini web app (where Google AI Pro, the plan that used to carry the Gemini Advanced name, is used) | [Where you can use the Gemini web app](https://support.google.com/gemini/answer/13575153) | Over 230 countries and territories; mainland China appears only as "Workspace only"; Russia is not listed; the mobile app may have different availability |
| Google AI Studio and the Gemini API | [Available regions for Google AI Studio and Gemini API](https://ai.google.dev/gemini-api/docs/available-regions) | A shorter list; mainland China, Hong Kong, Macau and Russia are not on it |

So one Google account can open the Gemini app but be refused by AI Studio, depending on where the person is. The United States, the United Kingdom, France and Japan, where the Gemini tier's cities are, were on both lists at our check. Plan prices, age rules and which features are offered where are on Google's own pages, and they change; read them before you pay for anything.

## What a fixed IP is actually for

- **Replacing a shared VPN exit or a rotating connection.** Those make one Google account look like many people, or like one person jumping between addresses.
- **A company network or allowlist.** An organisation that limits access to known addresses needs one egress IP per site or person. Your own company's admin adds the address; Proxy302 does not register IPs with Google or anyone else, its IPs are not on any allowlist, and an allowlisted IP does not exempt an account from identity or phone verification.
- **Travel between supported countries.** Signing in through your usual address keeps the account's history in one place. The country you are in must still be on Google's list.

## Where to put the IP

| Where you use Gemini | How the proxy is applied | Protocol |
|---|---|---|
| gemini.google.com or AI Studio in a browser | One browser profile with a proxy extension that accepts a username and password | HTTP (Chromium cannot authenticate to SOCKS5) |
| Gemini API from your code | Your HTTP client's proxy setting, or `HTTPS_PROXY` | HTTP |
| Gemini app on iPhone | Wi-Fi settings, Configure Proxy, Manual, Authentication on; see [iPhone proxy settings](../iphone-proxy-settings/README.md) | HTTP |
| Gemini app on Android | The built-in Wi-Fi proxy has no username or password fields | Needs an app that handles authentication |
| An antidetect browser profile | The profile's proxy settings | HTTP or SOCKS5, matching the purchase |

## Steps

1. **Pick the country.** It must be where you are and on Google's list, and on the AI Studio list too if you use it. Do not switch countries later.
2. **Buy one IP per Google account.** HTTP suits browser extensions, curl and SDKs. Pick a city or leave it on automatic.
3. **Put it in one place.** One browser profile, one phone or one API client. Turn off any VPN or second proxy underneath.
4. **Check the exit.** In the same browser open `https://ipinfo.io/json`; `country` must be the one you bought.
5. **If you use the API, test with your own key** (placeholders, not real values):

   ```bash
   curl -sS -x "http://USER:PASS@HOST:PORT" \
     -H "x-goog-api-key: $GEMINI_API_KEY" \
     "https://generativelanguage.googleapis.com/v1beta/models" | head -c 300
   ```

   A list of models means the request was accepted from that location. An error saying the user location is not supported means it was not, and no setting changes that.
6. **Sign in and keep the pairing.** That person, that Google account, that IP. Auto-renew is on by default at checkout; renewing normally keeps the same IP.

## Common mistakes

| Mistake | Result | Fix |
|---|---|---|
| Choosing a country only on the Gemini app list | AI Studio says it is not available in your region | Choose a country on both lists |
| Signing in with and without the proxy | The account sees changing addresses | One place, one IP, every time |
| Several people on one Google account | One account, many people | One account and one IP per person |
| SOCKS5 IP in a Chrome extension | Authentication fails | Buy the IP as HTTP |
| Expecting the IP to handle verification or payment | It cannot | Complete Google's steps honestly with your own details |

## FAQ

**Is Gemini Advanced still a product name?** Google now sells the paid Gemini plan as Google AI Pro, with Google AI Ultra above it. The IP question is the same for both.

**Gemini tier or a regular static ISP IP?** Both stay fixed for the term. The Gemini tier is a native IP for one person and its first IP is half price; a regular static ISP IP comes in 1 to 90 day terms ($0.50 to $23.97) if you want a short test.

**Can one IP serve Gemini in the browser and my API scripts?** Yes, for the same person, as long as both use the same HTTP proxy.

**Will the IP keep my account from being suspended?** No. A steady address removes one source of confusion; Google's terms and how the account is used decide the rest.

**Is there a free trial?** No. The cheapest test is a static ISP IP for one day at $0.50.

## Where Proxy302 fits

Proxy302 sells AI subscription ISP IPs for 30 days: Gemini tier 8 PTC, GPT/Grok tier 10 PTC, Claude tier 15 PTC, with the first IP of each tier 50% off. Details are on the [Gemini page](https://proxy302.com/use-cases/ai-subscriptions/gemini/) and the [AI subscriptions overview](https://proxy302.com/use-cases/ai-subscriptions/); prices are on [proxy302.com/pricing/](https://proxy302.com/pricing/). For Claude and ChatGPT, read [which IP to use for Claude and ChatGPT](../which-ip-for-claude-and-chatgpt/README.md).

*Facts checked 21 November 2026.*
