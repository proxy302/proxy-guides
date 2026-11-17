# Rotating vs sticky residential sessions: when to use each

Use rotating sessions, where each new connection can leave from a different residential IP, for independent requests: collecting public pages, checking prices or search results, and viewing a site from many locations. Use a sticky session, which holds one exit IP for a limited window, when a few requests in a row must come from the same address, such as paging through results or a short multi-step form. Neither suits an account that stays signed in for days or weeks; give that account a static residential (ISP) IP.

*Proxy302 maintains this guide and sells these proxies. Nothing here is a way around any site's rules; it is about matching the connection to the job.*

## The three options side by side

| | Rotating | Sticky session | Static residential (ISP) |
|---|---|---|---|
| Exit IP | Can change on every new connection | Same IP for a window, then it may change | Same IP for the whole term |
| Best for | Many unrelated requests | Short flows that span several requests | Accounts and stores you sign in to repeatedly |
| How you select it | The plain gateway credentials | The session parameter your provider documents; Proxy302's console generates it for you | A fixed IP with its own host, port and login |
| Billing at Proxy302 | Per GB, from $1.50/GB | Per GB | Per IP and term, $7.99 for 30 days |

## See the difference yourself

Copy two connection strings from your console, one rotating and one sticky, and compare the exits:

```bash
pip install requests
export ROTATING_PROXY="http://USER:PASS@HOST:PORT"
export STICKY_PROXY="http://STICKY_USER:PASS@HOST:PORT"  # session login from your console
```

```python
import os
import requests

URL = "https://api.ipify.org"

def exits(proxy_url, n=3):
    proxies = {"http": proxy_url, "https": proxy_url}
    # requests.get() opens a new connection every time
    return [requests.get(URL, proxies=proxies, timeout=30).text for _ in range(n)]

print("rotating:", exits(os.environ["ROTATING_PROXY"]))
print("sticky:  ", exits(os.environ["STICKY_PROXY"]))
```

Typically the rotating line lists different addresses and the sticky line repeats one. If the rotating line repeats, your client is probably reusing a keep-alive connection: a single open connection keeps one exit on any gateway.

## Choosing by task

| Task | Session type |
|---|---|
| Crawling product or listing pages | Rotating |
| Checking how a page looks from 20 countries | Rotating, one country per request |
| Paging through results where the site ties pages to a visit | Sticky |
| Adding to a cart and reading the total | Sticky |
| Monitoring your own ads or listings without logging in | Rotating |
| A seller, ads or social account used daily | Static ISP IP, one per account |

## Working with sticky sessions

- **Give each worker its own session.** Two workers on one session share an IP and its rate limits.
- **Expect the IP to change eventually.** A sticky exit can end before its window does. When it changes, restart that flow from the beginning instead of continuing halfway.
- **Keep flows short.** Sticky sessions suit minutes of work, not a working day.
- **Keep the country fixed** for the whole flow.

## Common errors

| Symptom | Likely cause |
|---|---|
| Rotating proxy keeps returning one IP | Keep-alive connection reuse in your client |
| IP changes halfway through a flow | The sticky window ended or the exit dropped; restart the flow |
| All workers show the same IP | They share one session parameter |
| Many 429 responses on a sticky session | Too many requests through one address; slow down |
| A logged-in account gets security prompts | It is on rotating or sticky traffic; move it to a static IP and complete any verification honestly |

## FAQ

**How long does a sticky session last?** As long as your provider documents for the session setting you chose. Treat it as an upper bound.

**Is sticky more expensive?** At Proxy302 both are rotating residential traffic billed per GB; the budget line is $1.50/GB, or $1.124/GB if you prepay 500 GB, and prepaid gigabytes never run out of date.

**Can I log in to an account on a sticky session?** For a one-off, short task, yes. For an account you use every day, a static IP is the right tool.

## Where Proxy302 fits

In the Proxy302 console you generate rotating or sticky connection details for residential traffic (budget $1.50/GB, premium $5/GB). Accounts that need one address for weeks get a static residential (ISP) IP instead, from $0.50 for a day. See the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
