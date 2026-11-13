# ISP proxy vs residential proxy: what is the difference?

An ISP proxy is an IP address registered to a consumer internet provider but served from a server, so it stays the same for as long as you hold it. A (rotating) residential proxy uses addresses on real household connections and usually changes per connection. In short: ISP proxies are fixed, residential proxies rotate. That is why an ISP proxy is also sold as a "static residential" proxy.

*Maintained by Proxy302. We are a vendor, not a neutral reviewer. [Chinese version](README.zh-CN.md).*

## Side by side

| | Rotating residential | Static ISP (ISP proxy) | Datacenter |
|---|---|---|---|
| Whose address | A household connection on a consumer ISP | An address registered to a consumer ISP, served from a server | A hosting or cloud company |
| Does it change | Per connection, or held for a sticky session | No, same IP for the whole term | No, same IP for the whole term |
| How IP databases tend to label it | isp / residential, sometimes cellular | isp | hosting |
| Typical billing | Per GB | Per IP and term | Per IP and term |
| Location choice | Country, plus state or major city on some lines | Country | Country |
| Typical use | Public-page collection, price checks, ad checks | Accounts and stores that sign in repeatedly | Heavy or server-to-server traffic |

At Proxy302, as of 14 November 2026 on [proxy302.com/pricing/](https://proxy302.com/pricing/): rotating residential from US$1.50/GB (Budget line) and US$5.00/GB (Premium line); static ISP US$7.99 per IP for 30 days (US$0.50 for 1 day); datacenter US$3.99 per IP for 30 days (US$0.25 for 1 day).

## Why the name "ISP proxy"

The name comes from who the address is registered to. A datacenter IP belongs to a hosting company. A household IP belongs to a consumer internet provider. An ISP proxy is registered to a consumer provider like the household IP, but runs on a server like the datacenter IP, and does not change. Other proxy vendors describe it the same way: registered with ISPs while physically hosted in a data center.

You can read the label yourself. IP data providers classify the organisation behind a network: IPinfo's ASN data labels networks as isp, hosting, business, education or government, and MaxMind's data has a user type such as residential, cellular or hosting. Two databases can label the same network differently, and both also sell proxy detection, so no type is invisible.

## When to pick which

**Pick rotating residential when** the job is many independent requests: collecting public pages, checking search results or ads from different countries, monitoring prices. You pay for the traffic you use, and you do not need the same address twice.

**Pick static ISP when** you sign in to the same account, store or back office repeatedly. The address should be the same every time, in the country the account belongs to.

**Pick datacenter when** you move a lot of traffic and the target accepts hosting-company addresses. It is the cheapest fixed IP, but some sites treat hosting addresses differently from home ones, so test your target.

Do not use a rotating IP for a signed-in account. The account would appear from a different address on each connection. If you must use rotating residential for a short login flow, use a sticky session in one country, not a global rotation.

## Common mix-ups

- **"Residential" does not mean "static."** Residential describes whose address it is; static or rotating describes whether it changes. ISP proxies are both residential-registered and static.
- **"ISP proxy" does not mean "your own ISP."** It is a proxy IP registered to an ISP, not your home connection.
- **A residential label is not a guarantee.** A platform may still treat an IP differently based on its history and your behaviour. A proxy changes the network address, not the platform's rules or your account's standing.
- **Static is not automatically better.** If you do not log in, a rotating residential IP is usually the cheaper tool.

## Per GB or per IP

Static ISP at Proxy302 can be leased per IP (a fixed IP for the term) or used per GB (US$3.50/GB, no lease, no promise of the same IP long term). Dividing the 30-day lease by the per-GB rate gives a break-even of about 2.3 GB: below that, per GB is cheaper; above it, or when you need one address, lease.

## Quick decision test

1. Do you sign in to an account? Yes: static ISP, one per account. No: continue.
2. Do you need many different exits, or a view from many countries? Yes: rotating residential.
3. Is it heavy server traffic to a site that accepts hosting addresses? Yes: datacenter.
4. Unsure: buy the smallest option (1 GB rotating residential or one IP for one day) and test against your real target.

Test any proxy by requesting `https://ipinfo.io/json` through it and checking the `country` and `org` fields. Commands are in [how to test a proxy](../antidetect-browser-with-residential-ip/README.md).

## FAQ

**Is an ISP proxy the same as a static residential proxy?** In practice, yes.

**Which is cheapest?** Per IP, datacenter. For many requests, rotating residential from US$1.50/GB.

**Can a website tell a residential IP is a proxy?** Sometimes. IP data companies sell residential-proxy flags. Using one stable, clean IP per account in the right country matters more than the label.

**Datacenter or residential for scraping?** Datacenter when the target accepts server IPs, since it is faster and cheaper. Switch to rotating residential when the target blocks hosting ranges. Follow each site's terms and robots rules.

## Where Proxy302 fits

Proxy302 sells all three types, with HTTP and SOCKS5. See [pricing](https://proxy302.com/pricing/) or the [homepage](https://proxy302.com).

*Facts from Proxy302 public pages and the IPinfo and MaxMind documentation, checked 14 November 2026.*
