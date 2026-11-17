# How much does a residential proxy cost per GB?

Rotating residential proxy traffic is priced per GB, and what you actually pay depends on the line, the volume you prepay for and how much traffic your job really uses. At Proxy302 the Budget line is $1.50/GB pay as you go, falling to $1.124/GB with a 500 GB prepaid package, and the Premium line is $5/GB, falling to $4.25/GB at 300 GB. Proxy302 maintains this guide and sells these proxies, so these are our own prices as checked on 17 November 2026; prices change, so confirm on the live page.

## Price table

| Line | Pay as you go | 50 GB | 100 GB | 300 GB | 500 GB |
|---|---|---|---|---|---|
| Budget | $1.50/GB | $1.42/GB | $1.35/GB | $1.20/GB | $1.124/GB |
| Premium | $5.00/GB | $4.74/GB | $4.50/GB | $4.25/GB | not listed |

Prepaid traffic never expires, so a larger package does not carry a use-by date. Sticky sessions are available when a task needs one address for a short stretch.

## What a package costs in total

| Package | Total |
|---|---|
| Budget 50 GB | $71.00 |
| Budget 100 GB | $135.00 |
| Budget 300 GB | $360.00 |
| Budget 500 GB | $562.00 |
| Premium 50 GB | $237.00 |
| Premium 100 GB | $450.00 |
| Premium 300 GB | $1,275.00 |

## What drives the real bill

- **Page weight.** A page with heavy images and scripts uses far more traffic than its text. Block images where you can.
- **Retries.** Failed requests that you repeat still use traffic.
- **Volume.** Prepaying lowers the per-GB rate, and since traffic does not expire, buying a package you will use over months is safe.
- **Line.** The two lines differ in price; pick by how demanding your target is and test with a small amount.
- **Job type.** If you sign in to an account, per-GB is the wrong shape. A fixed IP is easier to budget.

## When per-GB is not the cheapest

For accounts that log in repeatedly, a static ISP IP is billed per IP rather than per GB: $0.50 for 1 day, $3.50 for 7 days, $7.99 for 30 days, the same in every country. Rotating datacenter traffic is $0.40/GB for targets that accept server addresses. Websites can tell datacenter addresses are servers, so test your target first.

## How to estimate your own need

1. Load a typical page of your target once and note the size in your browser's network tab.
2. Multiply by requests per day and by days.
3. Add a margin for retries.
4. Start with the 50 GB package or pay as you go before you commit to 300 or 500.

Run `curl -x http://USER:PASS@HOST:PORT -o /dev/null -w '%{size_download}\n' https://example.com` through the proxy to measure a single response.

## FAQ

**Is there a free trial?** No. Pay as you go lets you start small.

**Does unused traffic expire?** Prepaid traffic never expires.

**What is the cheapest rate?** $1.124/GB on the 500 GB Budget package.

**Which line should I choose?** Start with Budget and move to Premium if your target needs it.

**Do I pay per GB for static IPs?** No; those are per IP and term. See [ISP proxy vs residential proxy](../isp-proxy-vs-residential-proxy/README.md).

## Where Proxy302 fits

Proxy302 sells rotating residential by the GB, static ISP IPs per term, Amazon ISP and datacenter proxies. See [pricing](https://proxy302.com/pricing/) and the [homepage](https://proxy302.com).

*Facts checked 17 November 2026.*
