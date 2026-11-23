# Which static IP should a Mercado Livre seller in Brazil use?

A seller account on Mercado Livre Brazil (mercadolivre.com.br) is best run from one Brazilian static ISP (static residential) IP per account, kept for the life of the account and used only in the browser profile set aside for it. Logged-out research, such as checking how your listings and other sellers' prices appear to Brazilian shoppers, belongs on rotating residential traffic instead. Proxy302 maintains this guide and sells these proxies: a static ISP IP in Brazil costs $0.50 for one day or $7.99 for 30 days, the same as every other country, and static IPs of 30 days or longer are dedicated to the buyer. Mercado Livre sets the rules for who can sell and how accounts are verified; an IP does not change those rules or replace any of their checks.

## Who this is for

A business or individual registered to sell in Brazil whose account is opened from more than one place: an office, staff working from home, a laptop on the road, a cloud desktop. Each of those connections has its own address. A fixed Brazilian exit on the team's operations profile means the account is seen arriving from the same network every day.

Accounts on Mercado Livre Brazil are registered to a CPF (an individual) or a CNPJ (a company). The IP should fit the account you actually hold: a Brazilian account run by a Brazilian business uses a Brazilian IP. Do not choose an IP to make an account look like something it is not.

## Which proxy for which task

| Task | Proxy | Price at Proxy302 |
|---|---|---|
| Signing in to the seller account every day | Brazil static ISP IP, one per account | $7.99 per 30 days |
| Trying the setup first | Brazil static ISP IP for 1 day | $0.50 |
| Logged-out price and search checks as Brazilian shoppers see them | Rotating residential (Brazil) | $1.50/GB; prepaid 50 GB $1.42/GB, 100 GB $1.35/GB |

Prepaid rotating traffic does not expire. Do not sign in to the seller account over rotating traffic; the address changes between connections.

## Step by step

1. **Buy one Brazil static ISP IP per seller account.** Pick the protocol when you order: HTTP for a browser extension or the system proxy, SOCKS5 for antidetect browser profiles. One IP has one protocol.
2. **Set up one browser profile for the account.** Keep it for this account only, so its cookies and saved logins stay apart.
3. **Enter the connection details** (host, port, username, password) in that profile's proxy settings with the protocol you bought.
4. **Confirm the exit is in Brazil** before signing in. Open `https://ipinfo.io/json` in the profile; `country` should be `BR`. From a terminal:

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   The four capitalised words are placeholders. For a SOCKS5 IP, write `socks5h://` instead of `http://`.
5. **Sign in and keep it that way.** From now on this account opens only in this profile, through this IP.
6. **Turn on auto-renew.** An IP whose term ends unrenewed is released; buying again gives you a different address.

## Identity checks and account reviews

Mercado Livre can ask sellers to confirm who they are, for example with document photos or a face scan in the app, and it reviews accounts when something looks unusual. Complete every check honestly, with the same person and documents the account was registered with. A proxy has nothing to do with those checks and cannot help an account that does not belong to you. If an account is restricted, use Mercado Livre's own appeal and support channels rather than opening a new account.

## Habits that help

- One account, one IP, one profile. Never copy one profile's proxy into another.
- Turn off other system-wide network tools under the profile; one network layer only.
- Do not alternate between the static IP and an ordinary home connection for the same account.
- Keep a sheet: account, CPF or CNPJ holder, IP, profile, renewal date, who has access.

## Problems and likely causes

| Symptom | Likely cause | Fix |
|---|---|---|
| Proxy check in the profile fails | HTTP chosen for a SOCKS5 IP, or the reverse | Match the protocol on the order |
| 407 or repeated password prompts | Wrong proxy username or password | Copy them again from the order page |
| Country shows something other than BR | The profile is bypassing the proxy, or another tool is on top | Switch other tools off; recheck the profile |
| Stopped working after a month | Term ended, IP released | Renew before the end date |
| Frequent security emails about new sign-ins | Someone opens the account outside its profile | Agree that the account is used only in its profile |

## FAQ

**Can several Mercado Livre accounts use one IP?** Use one IP per account, so each account has its own steady record.

**Does it matter where in Brazil the IP is?** The country matters most. If the order page offers a choice of city, choosing where you operate is a reasonable default; then keep it.

**Can I use the same IP for the Mercado Pago side of the business?** If the same business runs both and the same person signs in, keeping both on the account's profile and IP is consistent. Follow Mercado Pago's own verification steps as they ask.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs in Brazil and the other countries on its [locations page](https://proxy302.com/locations/), and rotating residential traffic. Prices are on the [pricing page](https://proxy302.com/pricing/). Related guides: [IP for cross-border e-commerce](../cross-border-ecommerce-ip/README.md), [rotating vs sticky sessions](../rotating-vs-sticky-sessions/README.md).

*Facts checked 23 November 2026.*
