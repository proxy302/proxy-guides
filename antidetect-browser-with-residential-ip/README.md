# How do I pair an anti-detect browser (AdsPower, BitBrowser, GoLogin) with a residential or ISP IP?

Buy one static ISP IP per browser profile, create a custom proxy in the profile with the same type (HTTP or SOCKS5) as the IP, paste host, port, username and password, and run the browser's proxy check. Then open an IP-check page and a WebRTC leak test inside the profile. Keep that IP with that profile. Do not put several profiles on one IP.

*Maintained by Proxy302. We are a vendor, not a neutral reviewer. [Chinese version](README.zh-CN.md).*

## Recommended setup for every profile

| Item | Choose | Why |
|---|---|---|
| Product | Static ISP IP for any profile that stays signed in | The address stays the same for the whole term |
| How many IPs | One per profile | Profiles sharing an IP share an address the platform can see |
| Protocol | The one shown on the IP: HTTP or SOCKS5 | A type mismatch is a common cause of a failed check |
| Country | The country the account belongs to | Keep it fixed |
| WebRTC | Follow or replace with the proxy IP | Otherwise WebRTC can show your real public IP |

Rotating residential fits only short tasks inside a sticky session, not accounts that stay logged in. Follow each platform's terms; a proxy does not change them.

## Where the fields go

| Browser | Where to paste | How to test |
|---|---|---|
| AdsPower | Proxy, Configuration, or a custom proxy when creating a profile | Connection test, then an IP-check page in the profile |
| BitBrowser | Create window, proxy settings; `host:port:username:password` in the host field | Proxy detection |
| GoLogin | Location column or Settings, Proxy, Your proxy; `IP:port:username:password` | Green means it works |
| Kameleo, Hidemium, GenLogin | Profile network or proxy section, custom proxy | Each has a Check or Test Proxy button |

Proxy302 also has step-by-step pages per browser, linked from [proxy302.com](https://proxy302.com).

## Test the proxy before it goes into a profile

Do this first. It tells you whether a failure is the proxy or the browser. Replace the placeholders; never paste real credentials into shared notes or screenshots.

**curl, HTTP proxy:**

```bash
curl -sS --max-time 20 -x "http://USERNAME:PASSWORD@HOST:PORT" https://ipinfo.io/json
```

**curl, SOCKS5 proxy** (`socks5h` also sends DNS through the proxy):

```bash
curl -sS --max-time 20 -x "socks5h://USERNAME:PASSWORD@HOST:PORT" https://ipinfo.io/json
```

You should get JSON with an `ip`, a `country` and an `org`. The country should be the one you bought. Go by country; services disagree on cities.

**Python with `requests`** (`pip install "requests[socks]"` for SOCKS5):

```python
import sys
import requests

def check(scheme, host, port, user, password):
    proxy = f"{scheme}://{user}:{password}@{host}:{port}"
    proxies = {"http": proxy, "https": proxy}
    try:
        r = requests.get("https://ipinfo.io/json", proxies=proxies, timeout=20)
        r.raise_for_status()
        d = r.json()
        print(f"{scheme}: ip={d.get('ip')} country={d.get('country')} org={d.get('org')}")
    except requests.RequestException as e:
        print(f"{scheme}: failed: {e.__class__.__name__}: {e}")

if __name__ == "__main__":
    # placeholders: replace before running
    HOST, PORT, USER, PASS = "HOST", "PORT", "USERNAME", "PASSWORD"
    check(sys.argv[1] if len(sys.argv) > 1 else "http", HOST, PORT, USER, PASS)
```

Run `python check.py http` or `python check.py socks5h`. Compare the printed `ip` with the IP shown on your asset page, and run it once without a proxy to see your own address for contrast.

## Add the proxy to a profile

1. Buy one static ISP IP per profile and pick the country of each account.
2. Copy all four fields from your IP page. Do not add a scheme or spaces to the host field.
3. Create the profile and choose a custom proxy with the type matching the IP.
4. Run the browser's proxy check and save only after it passes.
5. Open the profile, visit `https://ipinfo.io`, then run a WebRTC leak test. The IP must match your asset and no second address should appear.

## "Proxy check failed": what it usually means

| Symptom | Likely cause and fix |
|---|---|
| Fails immediately | Type mismatch or a bad paste. Copy again and set the type shown on the IP. |
| 407 or "CONNECT tunnel failed, response 407" | Wrong or missing username or password. Use this IP's own credentials. |
| "User was rejected by the SOCKS5 server" | Credentials refused on the SOCKS5 port. Re-check them, or use HTTP if the IP is HTTP. |
| Check passes but the profile shows your own IP | A VPN, extension, antivirus or firewall is interfering. Turn them off for the test. |
| Worked yesterday, fails today | The lease may have ended or the wallet or traffic ran out. An expired IP cannot be used. |
| IP check fine, WebRTC shows another address | Set WebRTC in the profile to follow or replace with the proxy IP. |

## Common questions

**Can several profiles share one proxy?** They can connect, but every platform then sees the same address behind all of them, which defeats the point of separate profiles.

**HTTP or SOCKS5?** Either, as long as it matches the IP. Check what your browser's proxy field supports.

**Is this a way to get around platform checks?** No. The aim is one stable, clean IP per account in the account's own country. Do not use it to dodge identity verification, and follow each platform's terms.

## Where Proxy302 fits

Proxy302 static ISP IPs support HTTP and SOCKS5 and are priced per IP and term (at last check US$0.50 for 1 day, US$7.99 for 30 days; see [pricing](https://proxy302.com/pricing/)). Any provider that gives you host, port, username and password will work in these browsers; the tests above work for all of them.

*Facts from Proxy302 public pages, checked 14 November 2026.*
