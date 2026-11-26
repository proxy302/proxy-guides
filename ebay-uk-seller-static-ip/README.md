# Which static IP should an eBay UK (ebay.co.uk) seller use?

Give each ebay.co.uk seller account one UK static ISP (static residential) IP, keep that IP for as long as the account trades, and open the account only in the browser profile that carries it. Staff should get their own access through eBay's Team access feature instead of the owner's password. Use rotating residential traffic only for logged-out work such as looking at how listings and search results appear to UK buyers. Proxy302 maintains this guide and sells these proxies: a UK static ISP IP costs $0.50 for one day or $7.99 for 30 days, the same price in every country, and static IPs of 30 days or longer are dedicated to the buyer. eBay decides who may sell, which details it needs and how it reviews an account; an IP changes none of that.

## When a fixed UK IP is worth having

The setup is for a business that sells on ebay.co.uk from the UK and whose people sign in from more than one network: the office line, home broadband, a phone hotspot on the way to the post office, a laptop at a trade fair. Each of those is a different address. Putting day-to-day Seller Hub work in one profile with one UK IP gives the account a single steady exit, while each person's own login shows who did what.

Pick the IP in the country your business really operates from. If your company is registered and run somewhere else, use an IP there; a UK address does not turn a seller into a UK business, and making an account look as if it operates from elsewhere is not what this is for.

## Start with eBay's own team tools

eBay UK's [Team access](https://www.ebay.co.uk/help/account/multiuser-account-access/multiuser-account-access?id=5197) (the feature used to be called multi-user account access) lets an account owner invite other eBay members to work in Seller Hub. As eBay's help page describes it on 26 November 2026:

| Point | What eBay says, in short |
|---|---|
| Who can use it | The owner must be opted in to Seller Hub |
| How to add someone | My eBay, Account, Team access, Add user; enter name, email and permissions |
| What invitees get | Only the Seller Hub sections you grant; they act on your behalf there |
| What stays private | Your password and personal details are not shared |
| Security | Invited accounts must turn on two-step verification |
| Record | The owner can see an activity log of who did what |
| Invitations | Expire after 24 hours if not accepted |

So the fixed IP keeps the account's network steady, and Team access keeps the people separate. You need both; neither replaces the other.

## What eBay says about more than one account

eBay's [multiple accounts policy](https://www.ebay.co.uk/help/policies/identity-policies/multiple-accounts-policy?id=4232) allows a member to hold more than one account for genuine reasons, and does not allow extra accounts to get around restrictions, limits or other policy consequences. Restrictions on one account can be extended to linked ones. A separate IP per account keeps each account's history tidy; it does not make two accounts of the same member unrelated in eBay's eyes.

## Which proxy for which job

| Job | Proxy | Proxy302 price |
|---|---|---|
| Daily Seller Hub work for one account | UK static ISP IP | $7.99 per 30 days |
| Trying the setup first | UK static ISP IP for 1 day | $0.50 |
| Fewer renewals | UK static ISP IP for 60 or 90 days | $15.98 or $23.97 |
| Logged-out checks of listings, search and prices as UK buyers see them | Rotating residential | $1.50/GB; prepaid 100 GB $1.35/GB |

*Black Friday, 24 Nov to 1 Dec 2026: static ISP IPs cost $14.00 for 60 days and $18.00 for 90 days, with 1 to 30 days unchanged.*<!-- bf2026 -->

Never sign in to Seller Hub through rotating traffic: the address changes between connections, which is the opposite of what an account needs.

## Setting it up

1. **Buy one UK static ISP IP per seller account.** Choose the protocol at purchase; it stays fixed for that IP. HTTP is the easy choice for a browser extension or system proxy; SOCKS5 fits antidetect browser profiles.
2. **Make one browser profile for the account.** A separate profile keeps its cookies and saved logins away from personal browsing.
3. **Enter host, port, username and password** from the order page into that profile's proxy settings, using the protocol you bought.
4. **Check the exit before signing in.** In the profile, open `https://ipinfo.io/json`; `country` should be `GB`. From a terminal:

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   `USER`, `PASS`, `HOST` and `PORT` are placeholders. For a SOCKS5 IP use `socks5h://`.
5. **Sign in as the owner once, then invite staff through Team access.** Each person signs in with their own eBay account inside the same profile when they work on this account.
6. **Turn on auto-renew for the IP.** An IP whose term ends without renewal is released, and buying again gives a new address.
7. **Keep a one-line record per account:** account, IP, profile, who has which Team access permissions, renewal date.

## If eBay asks you to verify

Give the same identity, business and payout details you registered with. The IP plays no part in that step and cannot help an application whose details are not genuine. If an account is restricted, use eBay's own appeal and contact routes; opening another account to keep selling is what the policy forbids.

## Troubleshooting

| What you see | Likely cause | Fix |
|---|---|---|
| ipinfo shows a country other than `GB` | The profile is not using the proxy, or another network tool sits on top | Switch other tools off and recheck the profile's proxy fields |
| Proxy check in the profile fails | HTTP and SOCKS5 mixed up | Use the protocol shown on the order page |
| 407 or a repeated login box | Proxy username or password wrong | Copy them again from the order page |
| Staff get new sign-in emails all the time | Some people open the account outside its profile | Agree that this account is opened only in its profile |
| Worked last month, not today | The IP's term ended and it was released | Renew before the end date, or keep auto-renew on |

## FAQ

**Does it matter which UK city the IP is in?** The country matters most. If the order page offers cities, choose the one nearest to where the business runs and then leave it alone.

**Can two eBay accounts share one IP?** Keep one IP per account so each account's sign-in record stays separate and problems are easier to trace.

**Do staff using the eBay app on their own phones need the proxy?** Not necessarily. Each person signing in with their own Team access account on their own phone is ordinary use. The fixed IP matters most for the desktop profile where most of the work happens.

**Is there a free trial?** No. The cheapest test is one static IP for one day at $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs in the UK and the other countries on its [locations page](https://proxy302.com/locations/), from 1 to 90 days, HTTP or SOCKS5 chosen at purchase, plus rotating residential traffic. See [pricing](https://proxy302.com/pricing/). For eBay in general, read [what proxy to use for an eBay seller account](../proxy-for-ebay-seller-account/README.md).

*Facts checked 26 November 2026.*
