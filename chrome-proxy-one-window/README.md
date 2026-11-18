# How do I open one Chrome window with its own proxy?

Launch a separate copy of Chrome with two flags: `--user-data-dir` pointing at a new, empty folder, and `--proxy-server=http://HOST:PORT`. The separate folder is what makes this work. If Chrome is already running on your normal profile, a plain relaunch only opens another window of that browser and the proxy flag is ignored. When the proxy needs a password, Chrome shows a sign-in box for it; the username and password cannot go into the flag, and Chrome cannot log in to a SOCKS5 proxy at all, so use an IP sold as HTTP. Your usual Chrome and your system settings stay as they are.

*Proxy302 maintains this guide and sells these proxies. The flags were run on Chrome 157 (headless, driven over the DevTools protocol) against local test proxies in November 2026; the sign-in box was not exercised in that test.*

## The command

**macOS** (Terminal):

```bash
open -na "Google Chrome" --args \
  --user-data-dir="$HOME/chrome-proxy-1" \
  --proxy-server="http://HOST:PORT"
```

`-n` asks macOS for a new instance even though Chrome is already open, and everything after `--args` goes to Chrome.

**Windows** (Command Prompt):

```bat
"C:\Program Files\Google\Chrome\Application\chrome.exe" --user-data-dir="%LOCALAPPDATA%\chrome-proxy-1" --proxy-server="http://HOST:PORT"
```

In PowerShell, put `&` in front of the quoted path and write the folder as `"$env:LOCALAPPDATA\chrome-proxy-1"`. To reuse it, create a desktop shortcut to `chrome.exe` and paste the two flags at the end of its Target field.

**Linux:**

```bash
google-chrome --user-data-dir="$HOME/.config/chrome-proxy-1" --proxy-server="http://HOST:PORT"
```

Microsoft Edge accepts the same two flags (`msedge.exe` on Windows, `"Microsoft Edge"` with `open -na` on macOS).

## What happens next

1. A fresh Chrome opens with no bookmarks, extensions or sign-ins. That is the empty profile in the new folder.
2. On the first page load, a sign-in box asks for the proxy's username and password. Enter them there.
3. Open `https://ipinfo.io/json`. The `ip` and `country` should belong to the proxy, not to your own connection.
4. Close that Chrome when done. Next time, run the same command and the profile, with its cookies and history, is still in that folder.

## More than one proxy at a time

Give each proxy its own folder:

```bash
open -na "Google Chrome" --args --user-data-dir="$HOME/chrome-proxy-1" --proxy-server="http://HOST1:PORT1"
open -na "Google Chrome" --args --user-data-dir="$HOME/chrome-proxy-2" --proxy-server="http://HOST2:PORT2"
```

Each folder is an independent browser with its own cookies. If the windows are for different accounts, keep each account in one folder with one IP, and keep using that pairing.

## Sending some sites around the proxy

```bash
--proxy-bypass-list="localhost;127.0.0.1;*.internal.example"
```

Entries are separated by semicolons, and `*.` matches subdomains. In our test, a host on the bypass list loaded directly while every other host went through the proxy.

## Flag reference

| Flag | Purpose |
|---|---|
| `--user-data-dir="FOLDER"` | Starts an independent Chrome with its own profile; required for the proxy flag to take effect while your normal Chrome is open |
| `--proxy-server="http://HOST:PORT"` | Sends page traffic through an HTTP proxy |
| `--proxy-server="socks5://HOST:PORT"` | SOCKS5, only for proxies that need no password |
| `--proxy-bypass-list="a;b"` | Hosts that skip the proxy |

## Common errors

| Chrome shows | Cause |
|---|---|
| Your own IP on ipinfo.io | `--user-data-dir` missing or pointing at a profile that was already open, so the flag never applied |
| `ERR_NO_SUPPORTED_PROXIES` | Username and password written inside `--proxy-server`; remove them and use the sign-in box |
| `ERR_SOCKS_CONNECTION_FAILED` | A SOCKS5 proxy that needs a password, or `socks5://` pointed at an HTTP port |
| `ERR_PROXY_CONNECTION_FAILED` | Nothing reachable at that host and port |
| `ERR_TUNNEL_CONNECTION_FAILED` | The proxy refused or failed the tunnel; see [ERR_TUNNEL_CONNECTION_FAILED](../err-tunnel-connection-failed/README.md) |
| `ERR_INVALID_AUTH_CREDENTIALS` | The proxy asked for a login and Chrome had no way to supply it; we saw this in headless runs, where no sign-in box can appear |
| `ERR_EMPTY_RESPONSE` on every site | `http://` pointed at a port that only speaks SOCKS5 |

## FAQ

**Can I avoid typing the password?** Not with the flag alone. For unattended use, drive Chrome with Playwright, which takes `{server, username, password}` (see the [Playwright guide](../playwright-proxy/README.md)), or use a proxy that authorises your IP address instead of a password.

**Does this change my system proxy?** No. Only the Chrome started with these flags uses the proxy; other apps and your normal Chrome keep their usual route.

**Does the flag cover everything in that window?** It covers page loads. WebRTC can open its own connections, so if that matters to you, check a WebRTC test page inside the window.

**Is a separate window the same as an antidetect browser profile?** No. It only separates cookies and routes traffic through one proxy. Whatever you sign in to, follow that site's terms and complete any verification it asks for honestly.

## Where Proxy302 fits

For this setup, buy the IP as HTTP; HTTP or SOCKS5 is fixed per IP at purchase. At Proxy302 a static residential (ISP) IP is $0.50 for a day, $3.50 for a week or $7.99 for 30 days, whichever country you pick, and each IP can be set to renew automatically. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
