# How do I set a SOCKS5 or HTTP proxy with a username and password in Telegram Desktop?

In Telegram Desktop open Settings, Advanced, Connection type, choose "Use custom proxy", then "Add proxy". Pick SOCKS5 or HTTP to match your proxy, fill in Hostname, Port, Username and Password, and save. Telegram connects through it straight away and shows "available" or "not available" next to the entry. The proxy only affects Telegram, not your browser or other apps. Proxy302 maintains this guide and sells these proxies; a static ISP IP from Proxy302 costs $0.50 for one day or $7.99 for 30 days, and can be bought as SOCKS5 or HTTP. Menu names below are taken from Telegram Desktop's own interface text as published in its [source code](https://github.com/telegramdesktop/tdesktop) in November 2026; your version may word them slightly differently.

## Why use a fixed proxy in Telegram at all

- **A company network** that only lets traffic out through an approved proxy.
- **A business account used from several places.** Telegram lists every session with an IP address and a location estimate. A team account that appears from one steady address is easier to read and to audit than one that jumps between home, office and mobile networks.
- **Keeping work and personal traffic apart** on one computer: the proxy applies to Telegram only.

A proxy does not change Telegram's own rules. If Telegram asks for a login code, a password or other confirmation, complete it as usual.

## Step by step

1. Open Telegram Desktop and go to **Settings**.
2. Open **Advanced** and click **Connection type**.
3. In the **Proxy settings** window, choose **Use custom proxy**. Telegram opens the proxy list; click **Add proxy**.
4. Choose the type: **SOCKS5** or **HTTP**. (MTPROTO and WEB are Telegram-specific proxy types with a secret instead of a username and password; they are not what a residential or ISP proxy gives you.)
5. Fill in:

   | Field | What to enter |
   |---|---|
   | Hostname | `HOST` from your order page |
   | Port | `PORT` |
   | Username | `USER` |
   | Password | `PASS` |

6. Click **Save**. The proxy appears in the list with its status: "connecting", "available (ping: … ms)" or "not available".
7. Close the window. Back in **Advanced**, the **Connection type** line now ends in "with proxy".

To switch it off later, go back to **Connection type** and choose **Disable proxy**, or **Use system proxy settings** if you want Telegram to follow the operating system instead.

## Which type to pick

| Your proxy | Telegram type | Notes |
|---|---|---|
| SOCKS5 with username and password | SOCKS5 | Works directly. Can also be shared as a `tg://socks?...` link |
| HTTP with username and password | HTTP | Works directly. Telegram does not create share links for HTTP proxies, so enter it by hand on each computer |
| A provider gave you `HOST:PORT:USER:PASS` | Whichever protocol you bought | The four parts go into the four fields above |

With Proxy302 the protocol is fixed when you buy a static IP: one IP has one protocol. Choose the matching type in Telegram; picking HTTP for a SOCKS5 IP (or the reverse) shows "not available".

## Adding a SOCKS5 proxy from a link

Telegram understands links of this form, which you can open on any computer where Telegram is installed:

```text
tg://socks?server=HOST&port=PORT&user=USER&pass=PASS
```

Percent-encode special characters in the username and password (`@` becomes `%40`, `&` becomes `%26`). Treat such a link like a password: anyone who has it can use your proxy and your traffic allowance or IP.

## Check that the proxy is really in use

- **In Telegram:** Settings, Privacy and Security, **Active sessions**. Session details in this list carry an IP address and a location, and Telegram notes that the location is an estimate based on the IP. For this computer's session the address should be the proxy's exit IP, not your home connection.
- **From a terminal,** test the proxy itself before blaming Telegram:

  ```bash
  # SOCKS5 IP
  curl -sS --max-time 20 -x "socks5h://USER:PASS@HOST:PORT" https://ipinfo.io/json
  # HTTP IP
  curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
  ```

  If curl works and Telegram does not, the problem is in the Telegram entry (type, typo, a stray space).

## Troubleshooting

| What you see | Likely cause | Fix |
|---|---|---|
| "not available" right after saving | Wrong type (SOCKS5 vs HTTP) or wrong port | Match the type to the protocol you bought |
| Stuck on "connecting" | Host or port wrong, or a local firewall blocks the port | Run the curl test above |
| Works in curl, not in Telegram | Typo or trailing space in a field | Delete the entry and add it again |
| Worked last month, fails today | The IP's term ended and it was released | Renew before the end date; auto-renew can be set per IP |
| Several proxies in the list and Telegram keeps switching | "Auto-switch proxies" is on | Keep one entry, or turn auto-switch off |
| Calls or media feel slower | Everything now travels through the proxy | Pick an IP in a country close to you and the people you talk to |

## FAQ

**Does the proxy apply to my browser too?** No. Telegram's proxy settings affect Telegram only. Use the system proxy or a browser profile for the browser.

**What about Telegram on a phone?** The mobile apps have their own proxy screen under Data and Storage. The same four fields apply.

**Static or rotating proxy for Telegram?** Static. A rotating proxy gives the account a different address on new connections, which shows up as a scattered session history.

**Is there a free trial?** No. The cheapest test is one static IP for one day at $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (SOCKS5 or HTTP, chosen at purchase, the same price in every country: $0.50 for one day, $7.99 for 30 days) and rotating residential traffic. See [pricing](https://proxy302.com/pricing/) and the [homepage](https://proxy302.com). How the two protocols differ: [SOCKS5 vs HTTP proxy](../socks5-vs-http-proxy/README.md).

*Facts checked 26 November 2026.*
