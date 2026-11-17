# What proxy should I use for an Amazon Seller Central account?

Use one static ISP IP per seller account and keep it for the life of the account, so every Seller Central sign-in comes from the same address. Put it in a browser profile that is used for that account only. A proxy does not make extra accounts acceptable: Amazon's Seller Code of Conduct allows one Seller Central account per region unless you have a legitimate business need for another, and verification is passed by submitting real documents, not by choosing an IP.

*Proxy302 maintains this guide and sells these proxies, including an Amazon ISP product.*

## What the proxy is for

A fixed IP keeps a legitimate account's connection consistent: the same address every time Seller Central is opened. That is all it does. It does not change Amazon's rules on multiple accounts, replace verification, or protect an account that breaks policy. Read the Seller Code of Conduct in Seller Central Help before opening a second account in any region, and ask Seller Support whether your business need qualifies.

## Choosing the IP

| Option | Fits | Proxy302 price |
|---|---|---|
| Amazon ISP | Seller Central sign-in, one IP per seller account | 12 PTC per 30 days (1 PTC = 1 USD), first IP half price |
| Static ISP IP | The same job on a general-purpose static IP | $7.99 per 30 days; $0.50 for a one-day test |
| Rotating residential | Logged-out checks, such as how your listing looks in another marketplace | $1.50/GB |
| Datacenter | Not recommended for Seller Central sign-in | Static from $0.25 per day |

Amazon ISP IPs are checked before delivery to confirm they are not datacenter addresses.

## Setting it up

1. **Count your accounts honestly.** One IP per seller account you are entitled to run.
2. **Buy the IP and pick a location you will keep.** Do not switch countries between sessions, and never choose an IP to make an account look like it operates from somewhere its registered business details say it does not.
3. **Create one browser profile per account.** In the profile's proxy settings, choose the same type (HTTP or SOCKS5) that the IP was bought with, and enter host, port, username and password.
4. **Confirm the exit IP first.** From a terminal:

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   Use `socks5h://` for a SOCKS5 IP. The `ip` should match your order and the `org` should be an internet provider, not a hosting company. Then open the same page inside the profile.
5. **Sign in and keep two-step verification with the person responsible.** Seller Central uses two-step verification; the phone or authenticator app belongs with the account owner.
6. **Give staff their own logins.** Add team members through Seller Central's user permissions instead of handing out the owner's password.
7. **Renew before the end date.** Auto-renew can be turned on per IP. An IP that lapses is released, and a new purchase brings a new address.

## When Amazon asks you to verify

Submit real documents that match what you registered, and follow any further steps Amazon's request names. The IP has no bearing on this step, and no proxy can help an application whose details are not true.

## Common errors

| Symptom | Likely cause and fix |
|---|---|
| Repeated new sign-in alerts | The address keeps changing: the profile is not using the proxy, or you sometimes sign in without it |
| Proxy check fails in the browser profile | Type mismatch between HTTP and SOCKS5, or a typo in a field |
| 407 Proxy Authentication Required | Wrong username or password for this IP |
| It worked last month, now it fails | The IP's term ended and it was released; renew earlier or enable auto-renew |
| `ipinfo` shows a hosting company | You bought a datacenter IP; use Amazon ISP or a static ISP IP |
| Two seller accounts on one address | Someone copied a profile's proxy into another profile; give each account its own IP |

## FAQ

**Can I run two seller accounts if each has its own IP?** Only if Amazon's policy permits it for your business. Separate IPs do not change that answer; Seller Support does.

**Can I use rotating residential for Seller Central?** No. The address changes, which is the opposite of what a signed-in seller account needs.

**Is there a free trial?** No. A one-day static ISP IP costs $0.50, and your first Amazon ISP IP is half price.
## Where Proxy302 fits

Proxy302 sells Amazon ISP for Seller Central and general static ISP IPs (1 to 90 days, HTTP or SOCKS5, same price in every country). Static IPs of 30 days or longer are dedicated to the buyer. For browser setup, see the [AdsPower integration page](https://proxy302.com/integrations/adspower/). Prices are on [proxy302.com/pricing](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
