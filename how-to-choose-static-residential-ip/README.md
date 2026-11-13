# How do I choose a static residential IP?

Start with the job, then check four things: where the account lives (country), how many accounts you run (one IP each), which protocol your tool needs (HTTP or SOCKS5), and how you will test the IP before you rely on it. A static residential IP, also sold as an "ISP proxy", keeps the same address for the whole lease, so it suits logins you repeat for days or weeks. It does not suit large jobs that bill by traffic.

*Maintained by Proxy302. We are a vendor, not a neutral reviewer. [Chinese version](README.zh-CN.md).*

## 1. Decide whether you need a static IP at all

| Your job | Usually fits | Why |
|---|---|---|
| Sign in to the same account, store or back office again and again | Static residential (ISP) IP | The address does not change during the lease |
| Collect public pages, check prices or search results over many requests | Rotating residential | The exit changes; you pay per GB |
| Heavy traffic where a residential address is not required | Datacenter IP | Fixed and cheap, but sites see a hosting-company address |

An IP only changes the network address a site sees. It does not change a platform's rules and it cannot lift a restriction or undo a ban. Read the terms of the platform you use.

## 2. Pick the country from the account, not from the price

Choose the country the account belongs to, such as the market a shop sells in or the country the account was created in. Then keep it. Moving one account between countries is the habit most likely to cause trouble, and it is easy to avoid. On Proxy302 the per-IP price is the same in every country, so price should not drive this choice. Which countries are in stock changes daily; the order page is the reference.

## 3. One IP per account

Give each account its own IP and keep it in its own browser profile or phone. Two accounts on one address share one visible login location. If you run several accounts, buy the same number of IPs.

## 4. Lease or pay per GB

Proxy302 sells static ISP two ways:

- **Per-IP lease.** One fixed IP for the whole term. At last check on [proxy302.com/pricing/](https://proxy302.com/pricing/): US$0.50 for 1 day, US$7.99 for 30 days, US$15.98 for 60 days, US$23.97 for 90 days.
- **Per GB, no lease.** The console generates a Fixed ISP exit for free and charges US$3.50 per GB used. You choose the country only, and the same IP is not promised over time.

At those prices the break-even is roughly 2.3 GB in 30 days. Below that, per GB costs less; above that, or when you need one address for the whole term, lease it. If the account must keep one address, lease.

## 5. Choose HTTP or SOCKS5 before you buy

A static IP is issued with the protocol you pick, and there is no switch afterwards. Browsers, extensions and the iPhone Wi-Fi proxy usually take HTTP; many antidetect browsers and automation tools accept SOCKS5. Check your tool's proxy field first. If unsure, SOCKS5 is the more flexible choice for browser profiles, HTTP for tools that only read standard proxy variables.

## 6. Test before you commit

Buy the smallest option first. On Proxy302 the smallest purchases are one IP for one day or 1 GB of rotating residential. Then:

1. Copy host, port, username and password from the order page or the console's My assets page.
2. Request an IP-echo page through the proxy, for example `https://ipinfo.io/json`.
3. Confirm the `country` matches what you bought and the `org` field names an internet provider.
4. Go by the country. Lookup services often disagree on the city.

Guide 4 in this repository has copy-paste commands for this test: [how to test a proxy](../antidetect-browser-with-residential-ip/README.md).

## 7. Buyer's checklist

- Billing unit: per IP and term, or per GB.
- Smallest purchase and whether there is a trial.
- Country list, and whether the page distinguishes "listed" from "in stock".
- Protocols offered and whether you can change them later.
- Whether the renewal setting is under your control (on Proxy302 you can turn auto-renew on or off per IP in My assets; check the checkout box).
- What the provider says it cannot guarantee. Be wary of any provider that guarantees a platform will never restrict you.

## Where Proxy302 fits

Proxy302 offers static ISP IPs (HTTP and SOCKS5, terms of 1 to 90 days), rotating residential IPs from US$1.50/GB on the Budget line, and datacenter IPs from US$3.99 for 30 days. See [pricing](https://proxy302.com/pricing/) and the [homepage](https://proxy302.com). Other providers sell similar products; compare them on the checklist above.

## FAQ

**Is a static residential proxy the same as an ISP proxy?** In practice yes: an address registered to a consumer internet provider that stays the same. See [ISP proxy vs residential proxy](../isp-proxy-vs-residential-proxy/README.md).

**Can I be sure to get a specific city?** No. A city is a request at best, and static ISP is chosen by country.

**How many accounts per IP?** One.

**Will a static IP prevent restrictions?** No. It gives a steady address; it does not change platform rules.

*Facts from Proxy302 public pages, checked 14 November 2026. Prices change; confirm on the live pricing page.*
