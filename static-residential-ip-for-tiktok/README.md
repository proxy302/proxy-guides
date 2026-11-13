# How do I set up a static residential IP for TikTok?

Buy one static ISP IP per TikTok account, in the country that account is aimed at. Copy its host, port, username and password into the browser profile or phone that account uses, check the exit with an IP-echo page, then sign in. Keep that IP for that account from then on. An IP gives the account a steady connection; it does not change TikTok's rules or lift a restriction.

*Maintained by Proxy302. We are a vendor, not a neutral reviewer. [Chinese version](README.zh-CN.md).*

## Read TikTok's terms first

TikTok's Terms of Service (last updated 1 December 2025, as we read them) contain two clauses that matter for running accounts: you must keep your password confidential and are responsible for all activity under the account, and you may not use automated scripts to collect information from or otherwise interact with the service. So whatever IP you use, the account should be operated by a person in the app, on the web, or through an approved tool, and the password should not be shared. These clauses do not depend on your network exit.

## Static or rotating?

The test is simple: does the same account have to sign in repeatedly from the same exit? If yes, use a static IP. If you only look at public content without logging in, a rotating residential IP costs less.

| | Static ISP IP | Rotating residential |
|---|---|---|
| Exit | Fixed for the whole lease | New IP per connection, or held for a session |
| Fits | One account that signs in from one place over time | Looking at public pages from different countries, logged out |
| Location | Country | Country, plus state or major city on some lines |
| Billing at Proxy302 | Per IP and term | Per GB |

## Step by step

**1. Choose the country.** Pick the market the account is aimed at. For a shop or creator account aimed at US viewers, choose a US exit. Once chosen, keep it and do not change it for testing. Whether a country can reach a platform changes over time; Proxy302 publishes an hourly platform access check you can look at first.

**2. Buy one IP per account.** In the Proxy302 console choose Static ISP, billed per IP and term, pick the region, a term and the quantity. At last check on [proxy302.com/pricing/](https://proxy302.com/pricing/): US$0.50 for 1 day, US$7.99 for 30 days, the same in every country. The smallest purchase is one IP for one day. Stock changes daily, so the order page decides which countries you can buy.

**3. Choose the protocol.** HTTP or SOCKS5, chosen at purchase; each IP carries one protocol. SOCKS5 is the usual choice for antidetect browser profiles, and HTTP for the iPhone Wi-Fi proxy and tools that only take HTTP.

**4. Copy the credentials.** Open My assets, view the IP's credentials, and copy host, port, username and password. Label each IP with its account and market so you do not mix them up later.

**5. Put it in one profile or one phone.**
- *Browser profile:* in an antidetect browser, create one profile per account, choose the custom proxy type that matches your IP (HTTP or SOCKS5) and paste the four fields. See [pairing an anti-detect browser with a residential or ISP IP](../antidetect-browser-with-residential-ip/README.md).
- *iPhone:* Settings, Wi-Fi, tap the (i) next to your network, Configure Proxy, Manual. Enter server and port from an HTTP IP, turn on Authentication, and enter the username and password. It applies to Safari and to apps that follow the system proxy; an app that ignores the system proxy will not use it.

**6. Check the exit before you log in.** Request `https://ipinfo.io/json` through the proxy. Confirm that `country` is the country you bought and look at the `org` field for the network. Different lookup services may report different cities, so go by country. Only when it matches, open TikTok.

```
curl -x socks5h://USER:PASS@HOST:PORT https://ipinfo.io/json
```

(Use `-x http://USER:PASS@HOST:PORT` for an HTTP IP. Placeholders, not real credentials.)

## Habits that keep an account steady

- One account, one IP, one profile or phone. Do not log two accounts in from one IP.
- Keep the account in the same country every session.
- Operate by hand, or publish through an approved tool. Do not automate with scripts.
- Do not share the password. Only the person responsible should use the account.
- Do not switch between the proxy, a VPN and your home connection for the same account. Keep one layer on.
- If an account is restricted, follow the appeal process TikTok gives you. An IP cannot reverse an enforcement action.

## Renewal

A static IP stays the same for the term. Renewing before it ends normally keeps the same IP; an expired IP is released and cannot be renewed after the end date. Auto-renew is a per-IP setting you can switch on or off in My assets, so check the box at checkout and the setting afterwards. If an IP ever has to move, it goes to another IP in the same country and you must update the profile.

## Troubleshooting

| Symptom | Check |
|---|---|
| Proxy check fails in the browser | Type (HTTP vs SOCKS5) must match the IP; paste all four fields with no scheme in the host field |
| 407 error | Wrong or missing username or password; use this IP's own credentials |
| Works yesterday, fails today | Lease may have ended; look at the end date in My assets |
| TikTok shows a different country | Profile is not using the proxy, or a VPN or second proxy sits on top |

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 day terms, HTTP and SOCKS5) and rotating residential traffic. See [pricing](https://proxy302.com/pricing/) and the [homepage](https://proxy302.com). Other vendors sell comparable products.

*Facts from Proxy302 public pages, checked 14 November 2026.*
