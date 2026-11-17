# How do I choose a residential proxy provider?

Choose a rotating residential proxy provider on things you can verify before you spend much: a published per-GB price with prepaid tiers, a clear answer on whether traffic expires, control over sticky sessions, the country or city targeting your job needs, a small test purchase, the protocols your tools speak, and written refund and renewal terms. Proxy302 maintains this guide and sells these proxies. Our rotating residential Budget line is $1.50/GB pay as you go, falling to $1.124/GB with a 500 GB prepaid package, prepaid traffic does not expire, and the Premium line starts at $5/GB. Run the checks below on any provider, including us. (For fixed IPs, see [how to choose an ISP proxy provider](../how-to-choose-an-isp-proxy-provider/README.md); this page is about traffic billed by the GB.)

## Seven checks before you commit

| Check | How to verify it | What a good answer looks like |
|---|---|---|
| Per-GB price | The public pricing page, not a sales quote | Pay-as-you-go rate and every prepaid tier listed |
| Traffic expiry | Pricing page or terms, searched for "expire" or "validity" | Either no expiry, or a stated period you can live with |
| Sticky sessions | Documentation | You can hold one exit for a short flow and choose when to let it go |
| Location targeting | Order page or docs | The country you need, plus city if your job really needs it, with the line it requires named |
| Small test | Checkout | You can buy a small amount before a large package |
| Protocols | Docs or order page | The protocol your software takes (HTTP for most browser automation) |
| Refunds and renewals | Terms page | Written conditions, not "contact sales" |

## How to run a cheap evaluation

**1. Write down your job first.** Target sites, countries, requests per day, and whether any step needs the same IP twice in a row. This decides which checks matter.

**2. Buy the smallest amount.** Pay as you go is the right shape for a test. Do not prepay a large package until the test passes.

**3. Confirm rotation.** Each `curl` call below opens a new connection, so a rotating gateway should show different addresses (placeholders, not real credentials):

```bash
for i in 1 2 3 4 5; do
  curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json \
    | python3 -c 'import sys,json; d=json.load(sys.stdin); print(d.get("ip"), d.get("country"), d.get("org"))'
done
```

The `org` field should name internet providers, and `country` should match what you asked for.

**4. Confirm sticky control.** Swap in a sticky login (the session parameter your provider documents; Proxy302's console generates it for you) and run the loop again. You should see one address repeated.

**5. Measure what you are billed for.** Add `-o /dev/null -w '%{size_download}\n'` to a request against your real target to see the bytes of one page. Compare your own total with the usage the provider's dashboard shows after the test. A large gap is worth a question.

**6. Test the real target.** A gateway that reaches ipinfo.io may still fail on your site. Count how many requests need a retry; retries are billed traffic too.

**7. Read the terms, then scale.** Only after steps 3 to 6 pass should you compare prepaid tiers.

## Common mistakes

| Mistake | Why it hurts | Better |
|---|---|---|
| Comparing headline rates only | Expiring or unused traffic raises the real price per GB | Compare cost per GB you will actually use |
| Prepaying before testing | You are locked in to traffic that may not suit the target | Test pay as you go first |
| Using rotating traffic for a signed-in account | The address changes under the account | Use a static ISP IP for accounts |
| Paying for city targeting you do not need | It may require a pricier line | Start with country; add city only if results differ |
| Forgetting retries in the budget | Failed attempts consume traffic | Measure retries during the test |
| Ignoring protocol | Some browser tools cannot use authenticated SOCKS5 | Check what your software accepts |

## FAQ

**What is the single most useful check?** Whether traffic expires. A low rate on traffic that lapses can cost more than a higher rate that keeps.

**How big should the test be?** Big enough to run your real job for a day. Proxy302's pay-as-you-go rate lets you start small.

**Do I need the Premium line?** Start on Budget. Move up only if your own test shows Budget cannot do the job; see [the real cost per GB](../cheapest-residential-proxy-per-gb/README.md).

**Is there a free trial?** Proxy302 has none.

## Where Proxy302 fits

Proxy302 sells rotating residential traffic on two lines. Budget: $1.50/GB, or prepaid at $1.42 (50 GB), $1.35 (100 GB), $1.20 (300 GB) and $1.124 (500 GB) per GB. Premium: $5/GB, or $4.74, $4.50 and $4.25 per GB at 50, 100 and 300 GB. Prepaid traffic does not expire. It also sells rotating datacenter at $0.40/GB and static ISP IPs from $0.50 a day. See [pricing](https://proxy302.com/pricing/) and [proxy302.com](https://proxy302.com).

*Facts checked 17 November 2026.*
