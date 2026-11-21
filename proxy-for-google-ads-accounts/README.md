# What proxy should I use for Google Ads accounts?

For most advertisers and agencies the answer is: manage the accounts through a Google Ads manager account, sign in from your normal office connection, and add a fixed IP only where a person's connection is unstable, shared or routed through a VPN exit. When you do need one, use one static ISP IP per person (or per office), keep it, and never sign in to an ads account over rotating addresses. Proxy302 maintains this guide and sells these proxies: a static ISP IP is $0.50 for one day or $7.99 for 30 days, the same price in every country, and static IPs of 30 days or longer are dedicated to the buyer. Google's Circumventing systems policy says you cannot create more Google Ads accounts after a suspension; a proxy does not change that, does not complete advertiser verification and does not make a policy-breaking account acceptable.

## What Google's own pages say

Read on 21 November 2026:

- **Managing several accounts.** A [manager account](https://support.google.com/google-ads/answer/6139186) lets you view and manage multiple Google Ads accounts from one place. Google describes it as made for agencies and anyone who manages multiple accounts.
- **New accounts after a suspension.** The [Circumventing systems policy](https://support.google.com/adspolicy/answer/15938075) lists trying to use Google Ads again after a suspension decision, and creating more accounts after a suspension, among its examples, and says you can't create multiple Google Ads accounts after getting suspended.
- **Related accounts.** The [account suspensions overview](https://support.google.com/adspolicy/answer/9841640) says accounts related to a suspended account may be suspended, and recommends reinstating suspended accounts first.
- **Verification.** [Advertiser verification](https://support.google.com/adspolicy/answer/9703665) asks selected advertisers to complete tasks, which can include submitting documents, by a deadline; verified details such as name and location can appear in ad disclosures.

Put together: there is an official way to run many accounts (a manager account), and an IP is not part of it. What an IP can do is keep the sign-in location of each person steady.

## When a fixed IP helps

- **Remote staff.** Someone who works from home or travels between countries signs in through one company address instead of whatever network they are on.
- **Shared exits.** A coworking space, hotel network or public VPN puts strangers behind the same address as your ads account.
- **Client requirements.** A client asks that their account only be opened from a known address.

If none of these apply, your office connection is fine.

## Choosing the IP

| Task | Option | Proxy302 price |
|---|---|---|
| Signing in to Google Ads | Static ISP IP, one per person or office | $7.99 per 30 days; $0.50 for one day |
| The same, where a server address is acceptable | Static datacenter IP | $0.25 per day, $3.99 per 30 days |
| Opening your landing pages as a visitor in another country, logged out | Rotating residential | $1.50/GB |
| The same from a mobile carrier network | Rotating mobile | $8/GB |

To see how an ad appears in a given location, try Google Ads' own Ad Preview and Diagnosis tool first; it does not need a proxy.

## Setting it up

1. **Link client accounts to your manager account** instead of collecting separate logins.
2. **Buy one static IP per person who signs in, in the country they work from.** Do not choose a country to make an account look like it operates from somewhere its billing and verification details say it does not.
3. **Use one browser profile per person** with the proxy set in it (HTTP or SOCKS5, as bought).
4. **Check the exit in that profile.** Open `https://ipinfo.io/json`, or from a terminal:

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   `country` must match the purchase; `org` should name an internet provider for a static ISP IP.
5. **Keep 2-Step Verification on each person's own Google Account.** Give each person their own access in the ads account rather than sharing one login.
6. **Renew on time.** Auto-renew can be turned on per IP; an IP whose term ends is released.

## When Google asks you to verify

Submit true details and documents for the business that pays for the ads. If an account is suspended, follow Google's appeal process for that account. Opening another account from a new IP is what the Circumventing systems policy describes as a violation.

## Common errors

| Symptom | Likely cause and fix |
|---|---|
| Repeated "new sign-in" security alerts | The person signs in sometimes through the proxy and sometimes without it |
| Landing page shows the wrong country version | Check the exit with ipinfo; some sites also use the site's own country selector |
| 407 Proxy Authentication Required | Wrong proxy username or password |
| Browser profile's proxy check fails | HTTP/SOCKS5 mismatch with the purchase |
| Proxy stopped working | The IP's term ended; renew earlier |

## FAQ

**Do I need a separate IP for every client account?** No. Accounts managed through one manager account are opened by your staff; give each person a steady connection.

**Can a proxy get a suspended account back?** No. Only Google's appeal process can.

**Rotating residential for signing in?** No. It is for logged-out checks only.

**Is there a free trial?** No. A one-day static ISP IP costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5, same price in every country), static and rotating datacenter IPs, rotating residential from $1.50/GB and rotating mobile at $8/GB. See [pricing](https://proxy302.com/pricing/) and [use cases](https://proxy302.com/use-cases/). For Meta's ads tools, read [which proxy for Facebook ads accounts](../proxy-for-facebook-ads-accounts/README.md).

*Facts checked 21 November 2026.*
