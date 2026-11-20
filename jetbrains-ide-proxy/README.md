# How do I set a proxy with a username and password in IntelliJ IDEA and other JetBrains IDEs?

Open **Settings** (on macOS, **IntelliJ IDEA | Settings**), go to **Appearance & Behavior | System Settings | HTTP Proxy** (or type "proxy" in the settings search box), choose **Manual proxy configuration**, pick **HTTP** or **SOCKS**, enter the host name and port, tick **Proxy authentication**, fill in the login and password, and press **Check connection**. PyCharm, WebStorm, GoLand, PhpStorm, CLion, Rider and the other JetBrains IDEs use the same screen. It covers the IDE's own traffic: plugin downloads, updates, licence checks and similar. Builds run by Gradle or Maven are separate processes and need their own proxy settings.

*Proxy302 maintains this guide and sells these proxies. Checked against JetBrains' IntelliJ IDEA documentation ("HTTP Proxy" settings page) in November 2026, not run.*

## The settings, field by field

| Field | What to enter |
|---|---|
| Manual proxy configuration | Selected |
| HTTP / SOCKS | Match the proxy's protocol. At Proxy302, HTTP or SOCKS5 is fixed per IP at purchase |
| Host name | `HOST` |
| Port number | `PORT` |
| No proxy for | Hosts that should go direct, for example `localhost, 127.0.0.1, *.corp.example` |
| Proxy authentication | Ticked |
| Login / Password | `USER` / `PASS` |
| Remember | Ticked if the IDE should keep the password in its password store |

**Check connection** asks for a URL and tries to open it through the proxy. Use `https://ipinfo.io/json`; a success message means the proxy accepted your login and reached the internet.

The other choices on the page are **No proxy** and **Auto-detect proxy settings**. Auto-detect relies on your operating system or network to supply the proxy, which is the usual set-up on corporate networks, not for a proxy you bought yourself.

## What uses the IDE proxy, and what does not

| Traffic | Uses the IDE's HTTP Proxy setting? | Where to configure it otherwise |
|---|---|---|
| Plugin Marketplace, IDE updates, licence checks | Yes | |
| Gradle builds and dependency downloads | No, Gradle runs its own JVM | `gradle.properties` (`systemProp.https.proxyHost` and related keys) |
| Maven builds | No | `~/.m2/settings.xml`, `<proxies>` section |
| npm, pip and other tools in the terminal | No | Their own config or `HTTPS_PROXY`; see [npm](../npm-proxy/README.md), [pip (Chinese)](../pip-proxy-zh/README.md) |
| Git operations | Git's own settings | [git proxy](../git-proxy/README.md) |

For Gradle and Maven settings with a username and password, see the [Maven and Gradle proxy guide (Chinese)](../maven-gradle-proxy-zh/README.md).

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| Check connection reports 407 | Login or password wrong, or Proxy authentication not ticked |
| Check connection times out | Wrong host or port, or HTTP selected for a SOCKS5 port (or the reverse) |
| Marketplace works, Gradle sync fails | Expected: Gradle needs its own proxy settings |
| Works on the office network, fails at home | Auto-detect picked up a different proxy; switch to Manual |
| Asked for the password on every start | Remember was not ticked, or the OS keychain is locked |

## FAQ

**Is there a way to set the proxy before the IDE starts, for the first plugin download?** The welcome screen has its own settings entry (Customize, then All settings) where the same HTTP Proxy page is available before any project is opened.

**HTTP or SOCKS for the IDE?** Either works for a proxy without a login. With a login, the HTTP option is the one we can recommend from testing other JVM tools; it is also the type Gradle and Maven handle most simply.

**Does the IDE send my code through the proxy?** Only network calls the IDE makes go through it (Marketplace, updates, AI or cloud features you enable). Local editing and indexing do not touch the network.

**Should the proxy be static or rotating?** For an IDE, static. A company firewall or a licence server that allowlists one address expects the same source every time.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs as HTTP or SOCKS5, chosen at purchase, at $0.50 for one day or $7.99 for 30 days per IP. See [pricing](https://proxy302.com/pricing/).

*Facts checked 20 November 2026.*
