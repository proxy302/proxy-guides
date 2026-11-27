# How do I use a proxy with a username and password in Swift URLSession?

On iOS 17, macOS 14 and later, create a `ProxyConfiguration` from the Network framework with `ProxyConfiguration(httpCONNECTProxy:)` or `ProxyConfiguration(socksv5Proxy:)`, call `applyCredential(username:password:)` on it, and put it in `URLSessionConfiguration.proxyConfigurations`. Do not put the username and password in `connectionProxyDictionary`: in our test URLSession ignored them, HTTPS requests failed with CFNetwork error 310 and plain HTTP came back 407. On older systems, keep the dictionary for host and port and answer the proxy's challenge in a session delegate. Proxy302 maintains this guide and sells these proxies; a static ISP IP costs $0.50 for one day or $7.99 for 30 days.

*We compiled and ran every snippet on macOS with Swift 6.4 against a local test proxy that requires Basic authentication, over both HTTP CONNECT and SOCKS5.*

## iOS 17 / macOS 14 and later

```swift
import Foundation
import Network

// PORT is a number, for example 10000
let proxyEndpoint = NWEndpoint.hostPort(host: "HOST", port: PORT)

// IP bought as HTTP: URLSession opens a CONNECT tunnel for https:// and http:// URLs alike
let proxy = ProxyConfiguration(httpCONNECTProxy: proxyEndpoint)
// IP bought as SOCKS5: use this line instead
// let proxy = ProxyConfiguration(socksv5Proxy: proxyEndpoint)

proxy.applyCredential(username: "USER", password: "PASS")

let config = URLSessionConfiguration.ephemeral
config.proxyConfigurations = [proxy]
let session = URLSession(configuration: config)

let (data, response) = try await session.data(from: URL(string: "https://ipinfo.io/json")!)
print((response as? HTTPURLResponse)?.statusCode ?? -1)
print(String(decoding: data, as: UTF8.self))
```

What we saw on the test proxy:

- **HTTP proxy:** URLSession first sends `CONNECT host:443` without credentials, gets 407, then repeats it with `Proxy-Authorization`. Plain `http://` URLs also went through a CONNECT tunnel (to port 80), not as an absolute-URI `GET`.
- **SOCKS5:** username and password authentication worked, and the hostname was sent to the proxy rather than resolved locally, so DNS goes through the proxy.
- **Wrong password:** the request throws an error described as "Authentication error" (POSIX code 80) instead of returning a 407 response.

If your app still supports older systems, wrap the block in `if #available(iOS 17.0, macOS 14.0, *) { ... }`.

## Useful properties

| Property | What it does |
|---|---|
| `excludedDomains` | Hosts that should skip the proxy, for example your own API |
| `matchDomains` | Only these hosts use the proxy |
| `allowFailover` | Whether a connection may fall back to going direct when the proxy fails. Apple's header says failover is not allowed by default; leave it off so a broken proxy shows up as an error instead of quietly using your own address |

`WKWebsiteDataStore` has a `proxyConfigurations` property on the same OS versions, so a `WKWebView` can use the same proxy.

## iOS 16, macOS 13 and earlier

Set host and port in the dictionary and supply the credentials from a task delegate:

```swift
import Foundation

final class ProxyAuthDelegate: NSObject, URLSessionTaskDelegate {
    func urlSession(_ session: URLSession, task: URLSessionTask,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        if challenge.protectionSpace.isProxy() && challenge.previousFailureCount == 0 {
            completionHandler(.useCredential,
                              URLCredential(user: "USER", password: "PASS", persistence: .forSession))
        } else {
            completionHandler(.performDefaultHandling, nil)
        }
    }
}

let config = URLSessionConfiguration.ephemeral
config.connectionProxyDictionary = [
    "HTTPSEnable": true,
    "HTTPSProxy": "HOST",
    "HTTPSPort": PORT,
]
let session = URLSession(configuration: config, delegate: ProxyAuthDelegate(), delegateQueue: nil)
```

The string keys are used because the `kCFNetworkProxiesHTTPS…` constants are marked unavailable on iOS. In our test this delegate route worked for `https://` URLs. Two catches for plain `http://` URLs: with only the HTTPS keys set, they went out directly from our own address, bypassing the proxy; and when we added the HTTP keys (`HTTPEnable`, `HTTPProxy`, `HTTPPort`), the proxy's 407 came back to the app without the delegate being called. Keep old-OS traffic on HTTPS, or move to `proxyConfigurations`.

## Which approach to use

| Situation | Use |
|---|---|
| iOS 17+ / macOS 14+, HTTP or SOCKS5 IP with username and password | `ProxyConfiguration` + `applyCredential` |
| Older OS, HTTPS URLs, HTTP proxy with username and password | `connectionProxyDictionary` + delegate |
| Older OS, SOCKS5 with username and password | Did not work in our test: URLSession offered the proxy only the no-authentication method. Buy the IP as HTTP |
| Credentials put in `connectionProxyDictionary` (`kCFProxyUsernameKey`) | Did not work in our test |

## Check the exit address

Before pointing the app at it, confirm the proxy itself works:

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

Use `socks5h://` for a SOCKS5 IP. Then compare with what the app prints from `https://ipinfo.io/json`.

## Common errors

| Symptom | Likely cause |
|---|---|
| "Authentication error" thrown | Wrong username or password for this IP |
| CFNetwork error 310 | HTTPS through an HTTP proxy failed, often credentials in the dictionary instead of `applyCredential` or a delegate |
| Requests go out from your own IP | `proxyConfigurations` set on a different configuration than the one the session uses, or set after the session was created |
| Connection refused or timeout | Wrong host or port, or HTTP and SOCKS5 mixed up (the protocol is fixed per IP at purchase) |
| Worked yesterday, fails today | The IP's term ended and it was released |

## FAQ

**Does `URLSession.shared` pick this up?** No. Build your own session from the configuration you changed. `URLSession.shared` follows the device's system proxy settings.

**Can I set this per request?** The proxy belongs to the session configuration. Use one session per proxy.

**Should the app ship with the proxy password inside it?** Avoid hard-coding it in a shipped app. Fetch it from your own backend or keychain at run time.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5 chosen at purchase) and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/). For the device-wide setting instead of in-app, see [iPhone proxy settings](../iphone-proxy-settings/README.md).

*Facts checked 27 November 2026.*
