# What should I check before buying a cheap residential proxy?

Before buying a cheap residential proxy, check three things that often hide behind a low price: whether the traffic expires, whether the low rate only applies after a large minimum purchase, and whether the IPs are really registered to consumer internet providers rather than hosting companies. Then buy the smallest amount and test it on your own target. Proxy302 maintains this guide and sells these proxies, so read it as a vendor's checklist. Our rotating residential Budget line is $1.50/GB pay as you go and drops to $1.124/GB at 500 GB prepaid, prepaid traffic does not expire, and for signed-in accounts a static ISP IP costs $0.50 for a day or $7.99 for 30 days.

## Red flags in a "cheap" offer

| Red flag | Why it costs you | Question to ask |
|---|---|---|
| Traffic that lapses after a set period | You pay for gigabytes you never use, so the real rate rises | "When does purchased traffic expire?" |
| Lowest rate shown, but only for a big package or monthly commitment | The price you will actually pay is higher | "What is the rate at the amount I plan to buy?" |
| "Residential" with no way to see the network | You may get hosting addresses at a residential price | "Can I check the exits before I scale?" |
| No public price list | Prices can change per customer | "Where is the published price?" |
| Unclear billing unit | Uploads, failed requests or headers may count | "What exactly counts toward my GB?" |
| Promises to get past any site's checks | No honest provider can promise that, and it is not what a proxy is for | Treat it as a reason to look elsewhere |
| Refund or payment terms not written down | Disputes have nothing to refer to | "Where are the terms?" |

## Six checks to run

**1. Price at your volume.** Write down how many GB a month you expect, then read the rate for that amount. A low rate you will never reach is not your price.

**2. Validity.** Find the expiry rule in the pricing page or terms. If traffic lapses, divide the package price by what you will really use in that period.

**3. Smallest purchase.** Buy the least the provider allows. If the minimum is large, that is itself a cost of testing.

**4. Who the exits belong to.** Sample ten exits and read the network names (placeholders, not real credentials):

```python
# pip install requests
from collections import Counter
import requests

PROXY = "http://USER:PASS@HOST:PORT"
proxies = {"http": PROXY, "https": PROXY}

ips, orgs = set(), Counter()
for _ in range(10):
    # requests.get() opens a new connection each call, so a rotating gateway can give a new exit
    d = requests.get("https://ipinfo.io/json", proxies=proxies, timeout=20).json()
    ips.add(d.get("ip"))
    orgs[d.get("org", "unknown")] += 1

print(f"{len(ips)} distinct exits out of 10")
for org, n in orgs.most_common():
    print(n, org)
```

Expect several distinct addresses and network names of consumer internet providers. If most names are hosting or cloud companies, the traffic is not what the label says. For a deeper look, see [how to tell residential, ISP and datacenter IPs apart](../check-if-ip-is-residential/README.md).

**5. Billing honesty.** Note the bytes your own job downloaded, then compare with the usage the dashboard shows. They will not match to the byte, but they should be close.

**6. Your real target.** Run a day of the actual job and count retries. A cheap line that needs many retries is not cheap; see [the real cost per GB](../cheapest-residential-proxy-per-gb/README.md) for the arithmetic.

## Common mistakes

| Mistake | Result | Do this instead |
|---|---|---|
| Comparing the headline rate of two offers | The one with expiring traffic can cost more | Compare price per GB you will use |
| Prepaying the largest package for the discount | Money locked in before the line is proven | Pay as you go for the test |
| Skipping the exit check | You may be paying residential prices for hosting addresses | Sample exits and read the network names |
| Using cheap rotating traffic for logins | The account sees a new address each session | A fixed static ISP IP per account |
| Buying residential for a target that accepts servers | Paying for an address type you do not need | Test rotating datacenter ($0.40/GB at Proxy302) |

## FAQ

**Is a low per-GB price a bad sign?** Not by itself. It is a reason to check expiry, minimums and the exits before you scale.

**What is the cheapest way to test Proxy302?** Rotating residential pay as you go at $1.50/GB, or one static IP for one day at $0.50. There is no free trial.

**Does Proxy302's prepaid traffic expire?** No.

**Should I pick by price alone?** No. Pick the cheapest offer that passes your checks on your target.

## Where Proxy302 fits

Proxy302 publishes its prices. Rotating residential Budget: $1.50/GB, or $1.42, $1.35, $1.20 and $1.124 per GB at 50, 100, 300 and 500 GB prepaid. Premium: $5/GB, down to $4.25/GB at 300 GB. Prepaid traffic does not expire. Rotating datacenter is $0.40/GB; static ISP IPs run from $0.50 for one day to $23.97 for 90 days, the same in every country. See [pricing](https://proxy302.com/pricing/) and [proxy302.com](https://proxy302.com).

*Facts checked 17 November 2026.*
