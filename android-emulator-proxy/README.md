# How do I run the Android Emulator through a proxy with a username and password?

Start the emulator with `-http-proxy http://USER:PASS@HOST:PORT` (for example `emulator -avd Pixel_8_API_35 -http-proxy http://USER:PASS@HOST:PORT`), or enter the same details under Extended controls, Settings, Proxy and restart the emulator. This "Emulator Proxy" sends all of the virtual device's TCP traffic through an HTTP proxy, with authentication, and needs no setting inside Android. It does not carry UDP, does not take a SOCKS5 proxy, and has no bypass list. Proxy302 maintains this guide and sells these proxies: an HTTP static ISP IP costs $0.50 for one day or $7.99 for 30 days, the same in every country. The details below follow Google's [Android Emulator proxy documentation](https://developer.android.com/studio/run/emulator-networking-proxy) as of November 2026; we did not run an emulator for this page.

## Two different proxies in the emulator

Google's documentation separates them, and mixing them up is the most common source of confusion.

| | Emulator Proxy | Android System Proxy |
|---|---|---|
| Where you set it | `-http-proxy` flag, `http_proxy` variable, or Extended controls, Settings, Proxy | Inside Android: Settings, Network and internet, Wi-Fi, the network's proxy |
| What it covers | All TCP traffic from the virtual device | Mainly HTTP and HTTPS from apps that honour the system proxy |
| Username and password | Yes | Android's Wi-Fi proxy screen has no credential fields |
| Good for | Giving the whole device one exit, or getting out through a company proxy | Inspecting your app's traffic with a debugging proxy and its certificate |
| HTTPS inspection | No; traffic is tunnelled | Yes, with the tool's certificate installed |

Only one of the two can be active at a time. For "make the emulator appear from this IP", you want the Emulator Proxy.

## Option 1: command line

```bash
# list your virtual devices
emulator -list-avds

# start one through an authenticated HTTP proxy
emulator -avd Pixel_8_API_35 -http-proxy http://USER:PASS@HOST:PORT
```

- The `http://` prefix may be left out; `USER:PASS@HOST:PORT` also works.
- If you do not pass `-http-proxy`, the emulator looks for an `http_proxy` environment variable in the same format and uses it.
- Running several emulators at once: Google notes the flag should be set on the first launch.
- Startup options cannot be changed while the emulator runs. Close it and start again to change the proxy.
- Percent-encode special characters in the password (`@` becomes `%40`, `:` becomes `%3A`).

Google now points to the newer Android CLI (`android emulator ...`) for managing emulators and marks the old `emulator` tool as deprecated; the `-http-proxy` option is still documented for the `emulator` command, which ships in the SDK's `emulator/` folder.

## Option 2: Extended controls

1. In the running emulator, click **More** (the three dots) to open **Extended controls**.
2. Go to **Settings**, then **Proxy**.
3. Choose manual proxy configuration and enter the host name and port. Turn on proxy authentication and enter the username and password.
4. Apply, then **restart the emulator**. Google's documentation says changes made here take effect only after a restart.

The emulator saves these settings for that device and restores them next time.

When the emulator runs inside Android Studio, it reads the IDE's own HTTP proxy settings (Settings, Appearance and Behavior, System Settings, HTTP Proxy) once at launch to fill in the Emulator Proxy. Changing the IDE setting later does not update a running emulator.

## Check the exit from inside the device

Open Chrome in the emulator and go to `https://ipinfo.io/json`. The `ip` and `country` should be the proxy's. From a terminal on your computer, test the proxy itself:

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

Compare the two. If curl shows the proxy IP and the emulator shows your own, the emulator did not pick up the proxy: check the flag spelling, then cold-boot (`-no-snapshot-load`) so a saved snapshot does not hide the change.

## Limits to know

| Limit | What it means in practice |
|---|---|
| HTTP proxies only | Buy the IP as HTTP. A SOCKS5 IP cannot be used with `-http-proxy` |
| TCP only, no UDP | QUIC and other UDP traffic does not go through; apps usually fall back to TCP |
| No bypass list | You cannot exclude hosts. If your app talks to a dev server on your own computer and that stops working with the proxy on, test that part with the proxy off |
| Settings apply per launch | Restart after changing them |

## Typical uses

- **Testing your own app as users in another country see it:** localized store listings, prices, content and language from an exit in that country.
- **Working behind a company proxy** that is the only way out of the office network.
- **QA on a stable address:** a static IP means the same exit for every test run, so server logs and allowlists are easy to match.

Testing how your app behaves is fine; using an emulator and a proxy to get around another service's account checks or regional rules is not what this guide is for.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| No internet in the emulator at all | Wrong host or port, or a SOCKS5 IP used as HTTP |
| Pages load but ipinfo shows your own IP | Proxy not applied: flag typo, or the emulator was not restarted |
| `407` in app logs | Wrong username or password, or special characters not encoded |
| Your app cannot reach a dev server on your computer | The Emulator Proxy has no bypass list; test that part with the proxy off |
| It worked last month | The IP's term ended and it was released; renew before the end date |

## FAQ

**Can I set a different proxy per emulator?** Yes. Each launch takes its own `-http-proxy`, and Extended controls settings are saved per device.

**Does the Android Wi-Fi proxy inside the emulator support a password?** No credential fields there. Use the Emulator Proxy for authenticated proxies. For phones, see [Android Wi-Fi proxy settings](../android-proxy-settings-zh/README.md) (Chinese).

**Static or rotating?** For repeatable tests, static: every run leaves from the same address. Rotating residential suits large logged-out checks across many locations.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (choose HTTP at purchase for the emulator) in the countries on its [locations page](https://proxy302.com/locations/), and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/).

*Facts checked 26 November 2026.*
