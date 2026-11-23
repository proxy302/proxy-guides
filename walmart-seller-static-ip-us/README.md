# Which static IP should a Walmart Marketplace seller use in the US?

Give each Walmart Seller Center account one US static ISP (static residential) IP, keep it for as long as the account exists, and use it only in the browser profile your team uses for that account. Give every person their own Seller Center user instead of passing the admin password around. For logged-out work, such as seeing how a listing appears to US shoppers, use rotating residential traffic instead. Proxy302 maintains this guide and sells these proxies: a US static ISP IP costs $0.50 for one day or $7.99 for 30 days (the same price in every country), and static IPs of 30 days or longer are dedicated to the buyer. Who may sell on Walmart, which documents you need and how the account is reviewed are decided by Walmart. An IP does not change any of that.

## Who this setup is for

A US-registered business that sells on walmart.com and has staff signing in from several places: an office, home connections, laptops on the road, a cloud desktop. Without a fixed exit, Seller Center sees the same account arrive from a new network every day. A single static IP on the team's operations profile gives the account one steady address, and individual user logins keep track of who did what.

It is not a way to look like a different seller or to run accounts Walmart has not approved. One business, its own approved account, its own people.

## Use Walmart's own user roles first

Seller Center has built-in team access. According to [Walmart Marketplace Learn](https://marketplacelearn.walmart.com/guides/Getting%20started/Account%20settings/Add-a-new-user-to-my-Seller-Center-account), only users with admin access can add users, from the People and permissions page, and there are three roles:

| Role | What it can do (Walmart's description, shortened) | Give it to |
|---|---|---|
| Admin | Everything, including API credentials, new users and accepting legal agreements for the seller | The owner, and only people who may sign for the company |
| Read and Write | Day-to-day work such as setting up items and viewing reports | Listing and operations staff |
| Read Only | View data, but not change it or create and download reports | Bookkeepers, reviewers |

Walmart also notes that the seller is responsible for what its users do. Separate logins plus one fixed IP gives you a clean record.

## Which proxy for which task

| Task | Proxy | Price at Proxy302 |
|---|---|---|
| Signing in to Seller Center every day | US static ISP IP, one per seller account | $7.99 per 30 days |
| Trying the setup before you commit | US static ISP IP for 1 day | $0.50 |
| Logged-out checks of listings, search results and prices as US shoppers see them | Rotating residential | $1.50/GB; prepaid 50 GB $1.42/GB, 100 GB $1.35/GB |

Rotating residential changes address between connections, so do not sign in to Seller Center through it. Prepaid traffic does not expire.

## Setting it up

1. **Buy one US static ISP IP per seller account.** Choose the protocol at purchase: HTTP is the simplest for a browser extension or system proxy; SOCKS5 suits antidetect browser profiles. One IP has one protocol.
2. **Create one browser profile for the account.** A separate browser profile keeps the account's cookies apart from personal browsing.
3. **Enter host, port, username and password** from the order page into that profile's proxy settings, with the same protocol you bought.
4. **Check the exit before signing in.** In the profile, open `https://ipinfo.io/json` and confirm `country` is `US`. From a terminal:

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   `USER`, `PASS`, `HOST` and `PORT` are placeholders; for a SOCKS5 IP use `socks5h://`.
5. **Sign in with each person's own Seller Center user,** inside that profile.
6. **Turn on auto-renew for the IP.** When a term ends without renewal, the IP is released and a new purchase gets a different address.
7. **Write it down.** One line per account: seller account, IP, profile name, who has which role, renewal date.

## Habits that keep the account steady

- One seller account, one IP, one profile. Do not reuse the IP for a second account.
- Keep only one network layer on: no other system-wide network tool running underneath the profile.
- Do not switch between the static IP and your home connection for the same account.
- Store the admin password in a password manager, not in a chat.
- When Walmart asks you to verify your business, identity or payment details, answer honestly with the documents you registered with. A proxy cannot stand in for that, and it does not help an application that is not genuine.

## Problems and likely causes

| Symptom | Likely cause | Fix |
|---|---|---|
| Profile's proxy check fails | Protocol in the profile does not match the IP | Use the protocol you bought |
| 407 or a login box keeps coming back | Wrong proxy username or password | Copy them again from the order page |
| Worked last month, fails today | The IP's term ended and it was released | Renew before the end date, or use auto-renew |
| ipinfo shows a country other than US | The profile is not using the proxy, or another tool sits on top | Turn off other network tools; recheck the profile's proxy |
| Staff keep getting new-sign-in emails | Some people sign in outside the profile | Agree that the account is opened only in its profile |

## FAQ

**Does it matter where in the US the IP is?** The country matters most. If the order page offers a choice of city, the area where your business operates is a sensible default. Pick one and keep it.

**Can two Walmart seller accounts use one IP?** Use one IP per account. It keeps each account's record separate and makes problems easier to trace.

**Does the Walmart Seller app on a phone need the proxy too?** Not necessarily. Each person signing in with their own user on their own phone is normal use. The fixed IP matters most for the desktop profile where most of the work happens.

**Is there a free trial?** No. The smallest purchase is one static IP for one day at $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs in the US and the other countries on its [locations page](https://proxy302.com/locations/), plus rotating residential traffic. See [pricing](https://proxy302.com/pricing/) and the [homepage](https://proxy302.com). For general guidance on seller accounts, see [proxy for an Amazon seller account](../proxy-for-amazon-seller-account/README.md) and [IP for cross-border e-commerce](../cross-border-ecommerce-ip/README.md).

*Facts checked 23 November 2026.*
