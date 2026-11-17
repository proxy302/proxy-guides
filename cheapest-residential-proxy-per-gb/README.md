# What is the cheapest residential proxy per GB, really?

The cheapest residential proxy is the one with the lowest cost per GB you actually use, not the lowest headline rate. Work it out as money paid divided by the gigabytes of useful responses you get, after counting traffic that expires unused, any minimum purchase you have to make, and the traffic burned by retries. Proxy302 maintains this guide and sells these proxies. Our rotating residential Budget line is $1.50/GB pay as you go and $1.124/GB with a 500 GB prepaid package, and prepaid traffic does not expire, so nothing you buy is lost to a deadline; the Premium line starts at $5/GB. The worked examples below use those prices.

## What changes the real price

| Cost factor | Effect on your real price per GB | What to check |
|---|---|---|
| Headline rate | The starting point only | Pay-as-you-go rate and every prepaid tier |
| Traffic expiry | Unused GB is paid for but never used | Whether, and when, purchased traffic lapses |
| Minimum purchase | Forces you to buy more than a test needs | Smallest amount you can buy |
| Retries | Failed attempts are billed like successful ones | Your own retry count during a test |
| Page weight | Images and scripts multiply the GB per page | Bytes per page in your browser's network tab |
| Wrong product | Per-GB billing for a signed-in account is the wrong shape | Whether a per-IP product fits better |

## Worked examples with Proxy302 prices

**1. Estimate the month.** Suppose your job fetches 5,000 pages a day and each page weighs 0.5 MB after you block images. That is 2.5 GB a day, about 75 GB a month. Pay as you go: 75 × $1.50 = $112.50.

**2. Same cash, more traffic.** Over three months that job needs about 225 GB. A 300 GB Budget package costs $360 ($1.20/GB). Paying as you go for 225 GB costs $337.50. The package costs $22.50 more but leaves 75 GB in hand, which keeps because prepaid traffic does not expire. Per GB of traffic you own, $1.20 beats $1.50. Smaller jobs see the same effect: 100 GB prepaid is $135.00, exactly what 90 GB costs pay as you go.

**3. What expiry would do.** Imagine the same $135 for 100 GB, but the traffic had to be used within a month and your job used 40 GB in that month. You would have paid $135 for 40 GB, or $3.375 per used GB, more than double the pay-as-you-go rate. This is the check to make with any provider that sets a validity period.

**4. Retries.** If a quarter of your billed traffic turns out to be failed attempts, $135 for 100 GB buys 75 GB of useful responses, or $1.80 per useful GB. Cutting retries is a discount you give yourself: retry only on 403 or 429, cap the attempts, and stop fetching assets you do not parse. See [cutting bandwidth when scraping](../proxy-for-web-scraping-bandwidth/README.md).

**5. When the Premium line is worth it.** Divide the rates. Pay as you go, $5.00 ÷ $1.50 ≈ 3.3. At 300 GB prepaid, $4.25 ÷ $1.20 ≈ 3.5. So Premium is cheaper per useful page only if, on your target, Budget would use more than about 3.3 to 3.5 times as much traffic for the same output, for example because most Budget attempts fail, or if Budget cannot do the job at all. Run the same set of pages through each line in a small test and compare useful GB per dollar.

## How to find your own number

1. Load one typical page of your target and note its transfer size, or measure through the proxy (placeholders, not real credentials):

   ```bash
   curl -sS -o /dev/null -w '%{size_download}\n' -x "http://USER:PASS@HOST:PORT" https://example.com/
   ```

2. Multiply by pages per day and by 30, then divide by 1,000,000,000 for GB per month.
3. Run a day of the real job pay as you go and count retries.
4. Recalculate with the retry share added, then compare pay as you go with each prepaid tier.

## Common mistakes

| Mistake | What it costs | Better |
|---|---|---|
| Choosing by the lowest advertised rate | Expiring or wasted traffic raises the real rate | Divide money by GB actually used |
| Prepaying the biggest package on day one | Ties up money before the job is proven | Test pay as you go, then prepay |
| Retrying every error without a cap | Pays repeatedly for the same page | Retry 403 and 429 only, with a limit |
| Using residential where server addresses work | Overpaying for the address type | Rotating datacenter is $0.40/GB if your target accepts it |

## FAQ

**What is Proxy302's lowest residential rate?** $1.124/GB with the 500 GB Budget package ($562).

**Does prepaid traffic expire?** No.

**Is there a free trial?** No. Pay as you go lets you test without a package.

**Is datacenter always cheaper?** Per GB, yes ($0.40), but sites can see a server address, so test your target first.

## Where Proxy302 fits

Budget line: $1.50/GB, or prepaid at $1.42 (50 GB), $1.35 (100 GB), $1.20 (300 GB) and $1.124 (500 GB) per GB. Premium line: $5/GB, or $4.74, $4.50 and $4.25 per GB at 50, 100 and 300 GB. Prepaid traffic does not expire. See [pricing](https://proxy302.com/pricing/), [proxy302.com](https://proxy302.com) and the full price table in [residential proxy cost per GB](../residential-proxy-price-per-gb/README.md).

*Facts checked 17 November 2026.*
