# What proxy should I use for an eBay seller account?

Use one static ISP IP per eBay account, keep it for as long as the account runs, and use it from one browser profile, so every sign-in comes from the same address in the country your business actually operates from. Proxy302 maintains this guide and sells these proxies: a static ISP IP is $0.50 for a one-day test or $7.99 for 30 days, the same price in every country, and static IPs of 30 days or longer are dedicated to the buyer. eBay lets members hold more than one account, but not to get around restrictions, limits or other policy consequences; a proxy changes none of that and does not replace any verification eBay asks for.

## What eBay's policy says

eBay's [multiple accounts policy](https://www.ebay.com/help/policies/identity-policies/multiple-accounts-policy?id=4232), as we read it on 21 November 2026:

- Members may have multiple accounts on eBay, for example to keep buying and selling apart or to run separate product lines.
- Registering new accounts, or using other existing accounts, to avoid buying and selling restrictions, limits or other policy consequences is not allowed.
- Buying and selling policies apply equally to every account, and the buyer and seller standards have to be met on all of them.
- If restrictions or limits are applied to one account, similar ones may be applied to the member's other linked accounts.

The last point is the one people miss. Separate IPs do not make accounts separate in eBay's eyes when they belong to the same member. What a fixed IP does is keep each legitimate account's sign-in history steady.

## Choosing the IP

| Task | Option | Proxy302 price |
|---|---|---|
| Signing in to a seller account | Static ISP IP, one per account | $7.99 per 30 days; $0.50 for one day |
| The same, on a tight budget | Static datacenter IP | $0.25 per day, $3.99 per 30 days; eBay sees a server address |
| Logged-out checks: how a listing appears on another eBay site, what similar items sell for | Rotating residential | $1.50/GB, less with prepaid traffic |
| Checking how pages look on a mobile carrier network | Rotating mobile | $8/GB |

Rotating traffic is for looking, not for signing in. A seller account that appears from a new address on every visit is the opposite of what you want.

## Setting it up

1. **List the accounts you run and why.** One IP for each account eBay's policy allows you to have.
2. **Buy each IP in the country you operate from.** That is where your registered business details say you are. Do not choose an IP to make an account appear to operate from somewhere else, and do not switch countries between sessions.
3. **Create one browser profile per account.** Enter the IP's type (HTTP or SOCKS5, as bought), host, port, username and password in that profile only.
4. **Check the exit before signing in.**

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   Use `socks5h://` for a SOCKS5 IP. `country` should match the order and `org` should be an internet provider. Then open the same page inside the profile.
5. **Sign in, and keep the account's security with its owner.** The phone or authenticator used for sign-in codes stays with the person responsible for the account.
6. **Do not pass the owner's password around a team.** Each person who needs access should have their own sign-in.
7. **Renew before the end date.** Auto-renew can be set per IP. An IP whose term ends is released, and a new purchase gets a new address; see [what happens when a static IP expires](../static-ip-expiry-renew-zh/README.md) (Chinese).

## When eBay asks you to verify

Answer with real details that match what you registered: identity, business and payout information. The IP has no part in that step, and no proxy helps an application whose details are not true. If an account is restricted, use eBay's own appeal and contact routes; opening another account to carry on is exactly what the policy forbids.

## Common errors

| Symptom | Likely cause and fix |
|---|---|
| Frequent new sign-in alerts | The profile is not using the proxy every time, or someone signs in from another device without it |
| Proxy check fails in the profile | HTTP/SOCKS5 mismatch with the purchase, or a typo in a field |
| 407 Proxy Authentication Required | Wrong username or password for this IP |
| Worked last month, fails today | The term ended and the IP was released; renew earlier or turn on auto-renew |
| `ipinfo` shows a hosting company | You bought a datacenter IP; switch to static ISP for sign-ins |
| Two accounts on one address | A profile's proxy was copied into another profile; give each account its own IP |

## FAQ

**Can I run a separate account for each product line?** eBay's policy names that as an acceptable reason. The same standards apply to every account, and restrictions on one can reach the others.

**Do I need a proxy at all?** Not if you sign in from your own office connection. A fixed IP helps when your connection changes address often, when staff work remotely, or when you sit behind a shared VPN exit.

**Can I use rotating residential for Seller Hub?** No. Use it only for logged-out research.

**Is there a free trial?** No. A one-day static ISP IP costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs from 1 to 90 days ($0.50 to $23.97), HTTP or SOCKS5 chosen at purchase, the same price in every country, plus rotating residential traffic from $1.50/GB (prepaid 500 GB works out at $1.124/GB, and prepaid traffic does not expire). Setup in a browser profile: [AdsPower integration](https://proxy302.com/integrations/adspower/) or [antidetect browsers compared](https://proxy302.com/integrations/antidetect-browsers/). Prices: [proxy302.com/pricing/](https://proxy302.com/pricing/). Related: [which proxy for running multiple accounts](../proxy-for-multiple-accounts/README.md).

*Facts checked 21 November 2026.*
