# How do I use a proxy with a username and password in OkHttp (Kotlin, Java, Android)?

Give the `OkHttpClient.Builder` two things: `.proxy(Proxy(Proxy.Type.HTTP, InetSocketAddress.createUnresolved("HOST", PORT)))` for the address, and `.proxyAuthenticator { ... }` that adds a `Proxy-Authorization` header built with `Credentials.basic("USER", "PASS")`. Do not add that header in an interceptor: for HTTPS, OkHttp opens a CONNECT tunnel first, and the CONNECT request never passes through interceptors, so the proxy answers 407. Proxy302 maintains this guide and sells these proxies; a static ISP IP costs $0.50 for one day or $7.99 for 30 days, and for OkHttp you should buy it with the HTTP protocol.

*Checked against the OkHttp source and its `Authenticator` documentation (OkHttp 4.x and 5.x) in November 2026. We did not compile these snippets; the machine used for this page has no JDK.*

## Kotlin

```kotlin
import java.net.InetSocketAddress
import java.net.Proxy
import okhttp3.Credentials
import okhttp3.OkHttpClient
import okhttp3.Request

val proxyCredential = Credentials.basic("USER", "PASS")

val client = OkHttpClient.Builder()
    .proxy(Proxy(Proxy.Type.HTTP, InetSocketAddress.createUnresolved("HOST", PORT)))
    .proxyAuthenticator { _, response ->
        if (response.request.header("Proxy-Authorization") != null) {
            null // these credentials were already sent and refused: stop retrying
        } else {
            response.request.newBuilder()
                .header("Proxy-Authorization", proxyCredential)
                .build()
        }
    }
    .build()

val request = Request.Builder().url("https://ipinfo.io/json").build()
client.newCall(request).execute().use { response ->
    println(response.code)
    println(response.body.string()) // OkHttp 4.x: response.body!!.string()
}
```

`PORT` is a number. On Android, run the call off the main thread (a coroutine on `Dispatchers.IO`, or `enqueue`).

## Java

```java
import java.net.InetSocketAddress;
import java.net.Proxy;
import okhttp3.Credentials;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

String proxyCredential = Credentials.basic("USER", "PASS");

OkHttpClient client = new OkHttpClient.Builder()
    .proxy(new Proxy(Proxy.Type.HTTP, InetSocketAddress.createUnresolved("HOST", PORT)))
    .proxyAuthenticator((route, response) -> {
        if (response.request().header("Proxy-Authorization") != null) {
            return null; // already tried, give up
        }
        return response.request().newBuilder()
            .header("Proxy-Authorization", proxyCredential)
            .build();
    })
    .build();

Request request = new Request.Builder().url("https://ipinfo.io/json").build();
try (Response response = client.newCall(request).execute()) {
    System.out.println(response.code());
    System.out.println(response.body().string());
}
```

## How the authenticator is called

| Request | What OkHttp does |
|---|---|
| `https://` URL through an HTTP proxy | Before every CONNECT, OkHttp calls the proxy authenticator with a fake 407 whose challenge is `OkHttp-Preemptive`. Returning a request with `Proxy-Authorization` sends the credentials up front |
| `http://` URL through an HTTP proxy | The request is sent; if the proxy answers 407, the authenticator is called with that real response |
| Proxy refuses the credentials | The second call sees `Proxy-Authorization` already present; returning `null` stops the retries. For a CONNECT this surfaces as `IOException: Failed to authenticate with proxy`; for a plain `http://` request you get the 407 response |

The code above handles all three without checking the challenge scheme.

## Why `createUnresolved`

`InetSocketAddress("HOST", PORT)` looks the host name up the moment it is built, which on Android's main thread throws `NetworkOnMainThreadException`. With `createUnresolved`, OkHttp resolves the proxy's name itself when it connects.

## Retrofit, Coil and other libraries on OkHttp

Pass the same client in: `Retrofit.Builder().client(client)`, or the library's equivalent. Anything that accepts an `OkHttpClient` then uses the proxy.

## SOCKS5

OkHttp has no username and password setting of its own for SOCKS proxies. `Proxy.Type.SOCKS` hands the connection to the platform's socket code; on a desktop JVM that code can take SOCKS credentials from `java.net.Authenticator`, which is global to the process. On Android we would not rely on it. Buy the IP with the HTTP protocol for OkHttp; the protocol is fixed per IP at purchase.

## Check the proxy outside the app first

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

If curl gets a JSON answer with the country you bought, the IP and credentials are fine, and any remaining problem is in the client setup.

## Common errors

| Symptom | Likely cause |
|---|---|
| `IOException: Failed to authenticate with proxy` on an `https://` URL | No `proxyAuthenticator` (the default returns `null`), the header was added in an interceptor, or the proxy refused the credentials and the authenticator gave up |
| 407 response on an `http://` URL | Wrong username or password; the authenticator correctly gave up after one try |
| `NetworkOnMainThreadException` while building the client | `InetSocketAddress("HOST", PORT)` resolved on the main thread; use `createUnresolved` |
| Connection refused or timeout | Wrong host or port, or the IP was bought as SOCKS5 |
| Exit IP is your own | A different `OkHttpClient` instance (without the proxy) made the call |
| Worked yesterday, fails today | The IP's term ended and it was released |

## FAQ

**Can I use one proxy for some hosts and none for others?** Use `.proxySelector(...)` with your own `ProxySelector` instead of `.proxy(...)`, and return `Proxy.NO_PROXY` for hosts that should go direct.

**Does a long-lived client keep the same exit IP?** With a static IP, yes: the address does not change during its term. Reuse one `OkHttpClient` for the whole app; building one per request wastes connections.

**Where should the password live in an Android app?** Not in the APK. Fetch it from your backend at run time.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5 chosen at purchase) and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/). Related: [Java HttpClient with an authenticated proxy](../java-httpclient-proxy/README.md), [Android proxy settings](../android-proxy-settings-zh/README.md).

*Facts checked 27 November 2026.*
