# How do I use an authenticated proxy with Java HttpClient?

Build the client with `.proxy(ProxySelector.of(new InetSocketAddress(HOST, PORT)))` and an `Authenticator` that hands back your proxy username and password, then start the JVM with `-Djdk.http.auth.tunneling.disabledSchemes=` (an empty value). The flag matters: the JDK ships with Basic authentication turned off for CONNECT tunnels, so every `https://` request through a password-protected proxy returns 407 even when the password is correct. `java.net.http.HttpClient` only uses HTTP proxies, so order the IP as HTTP, not SOCKS5.

*Proxy302 maintains this guide and sells these proxies. The code was checked against the JDK 21 and 25 API documentation, the OpenJDK source and the OkHttp 5 source in November 2026.*

## Why HTTPS fails with 407 on a correct password

An `https://` URL goes through an HTTP proxy as a CONNECT tunnel. The JDK's own `conf/net.properties` contains the line `jdk.http.auth.tunneling.disabledSchemes=Basic`, which tells the built-in clients never to send Basic credentials on a tunnel. Proxies that sell IPs with a username and password use Basic, so the client gives up and you get the proxy's 407. Plain `http://` URLs are not tunnelled and are not affected, which is why "HTTP works, HTTPS doesn't" is the classic symptom.

Setting the property to an empty value lifts the restriction for that JVM only.

## HttpClient (Java 11 and newer)

```java
import java.net.Authenticator;
import java.net.InetSocketAddress;
import java.net.PasswordAuthentication;
import java.net.ProxySelector;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;

public class ProxyCheck {
    public static void main(String[] args) throws Exception {
        String host = System.getenv("PROXY_HOST");
        int port = Integer.parseInt(System.getenv("PROXY_PORT"));
        String user = System.getenv("PROXY_USER");
        char[] pass = System.getenv("PROXY_PASS").toCharArray();

        HttpClient client = HttpClient.newBuilder()
                .proxy(ProxySelector.of(new InetSocketAddress(host, port)))
                .authenticator(new Authenticator() {
                    @Override
                    protected PasswordAuthentication getPasswordAuthentication() {
                        // Answer only the proxy's challenge, never a website's
                        if (getRequestorType() == RequestorType.PROXY) {
                            return new PasswordAuthentication(user, pass);
                        }
                        return null;
                    }
                })
                .connectTimeout(Duration.ofSeconds(20))
                .build();

        HttpRequest request = HttpRequest.newBuilder(URI.create("https://ipinfo.io/json"))
                .timeout(Duration.ofSeconds(30))
                .build();

        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.statusCode() + " " + response.body());
    }
}
```

Java 11 and later can run a single source file without a separate compile step:

```bash
export PROXY_HOST=HOST PROXY_PORT=PORT PROXY_USER=USER PROXY_PASS=PASS
java -Djdk.http.auth.tunneling.disabledSchemes= ProxyCheck.java
```

The printed JSON should show the proxy's exit address and country, not your own. Reading the credentials from environment variables keeps them out of the source and out of your Git history.

### Setting the property in code instead

If you cannot change the launch command, make this the first line of `main`, before anything touches `java.net.http`:

```java
System.setProperty("jdk.http.auth.tunneling.disabledSchemes", "");
```

The HttpClient reads the property once, when its internal classes load. Set it later and it has no effect. Editing `conf/net.properties` inside the JDK also works but changes the behaviour of every program on that installation.

## OkHttp

OkHttp builds its own CONNECT requests, so the JDK property does not apply. It asks a `proxyAuthenticator` for credentials before it opens each tunnel.

```java
import java.net.InetSocketAddress;
import java.net.Proxy;
import okhttp3.Credentials;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

public class OkHttpProxyCheck {
    public static void main(String[] args) throws Exception {
        String host = System.getenv("PROXY_HOST");
        int port = Integer.parseInt(System.getenv("PROXY_PORT"));
        String credential = Credentials.basic(System.getenv("PROXY_USER"), System.getenv("PROXY_PASS"));

        OkHttpClient client = new OkHttpClient.Builder()
                .proxy(new Proxy(Proxy.Type.HTTP, new InetSocketAddress(host, port)))
                .proxyAuthenticator((route, response) -> {
                    if (response.request().header("Proxy-Authorization") != null) {
                        return null; // credentials already sent and refused: stop retrying
                    }
                    return response.request().newBuilder()
                            .header("Proxy-Authorization", credential)
                            .build();
                })
                .build();

        Request request = new Request.Builder().url("https://ipinfo.io/json").build();
        try (Response response = client.newCall(request).execute()) {
            System.out.println(response.code() + " " + response.body().string());
        }
    }
}
```

Dependency: `com.squareup.okhttp3:okhttp:5.5.1` (the current release at the time of writing). The `null` check matters: without it, a wrong password makes OkHttp retry with the same bad header until it hits its limit.

## Which client, which settings

| Client | Proxy setting | Credentials | JDK flag needed for HTTPS |
|---|---|---|---|
| `java.net.http.HttpClient` | `ProxySelector.of(...)` | `Authenticator` with `RequestorType.PROXY` | Yes |
| `HttpURLConnection` | `openConnection(new Proxy(Proxy.Type.HTTP, ...))` | `Authenticator.setDefault(...)` | Yes |
| OkHttp 4 or 5 | `.proxy(new Proxy(Proxy.Type.HTTP, ...))` | `.proxyAuthenticator(...)` | No |

With `HttpURLConnection`, the same missing flag shows up as `java.io.IOException: Unable to tunnel through proxy. Proxy returns "HTTP/1.1 407 Proxy Authentication Required"`.

## What about SOCKS5?

`java.net.http.HttpClient` does not support SOCKS. If your `ProxySelector` returns a SOCKS entry, the client skips it and connects directly, so the request silently leaves from your own address. Classic `java.net` sockets do speak SOCKS5 and take credentials from a default `Authenticator` or the `java.net.socks.username` and `java.net.socks.password` properties, but the simple route in Java is an HTTP IP.

## Common errors

| Symptom | Cause and fix |
|---|---|
| Status 407 on `https://`, fine on `http://` | Basic is disabled for tunnels. Add `-Djdk.http.auth.tunneling.disabledSchemes=` |
| Still 407 after adding the flag | The property was set after the first request, or the username or password is wrong. Test the same credentials with `curl -x http://USER:PASS@HOST:PORT https://ipinfo.io/json` |
| Exit IP is your own address | The selector returned a SOCKS proxy (ignored), or the code built a client without `.proxy(...)` |
| `java.net.ConnectException` | Wrong host or port, or a firewall blocks outbound connections to the proxy port |
| `HttpTimeoutException` | The proxy accepted the connection but the request did not finish within `.timeout(...)` |

## FAQ

**Is it safe to clear `jdk.http.auth.tunneling.disabledSchemes`?** Basic sends the password base64-encoded, not encrypted, on the hop from your machine to the proxy. The JDK default is cautious for that reason. Clear it only in programs that are meant to use your proxy.

**Can I set a `Proxy-Authorization` header by hand instead?** Not for HTTPS. The HttpClient filters that header off CONNECT requests under the same property.

**Will a rotating gateway give me a new IP per request?** Only on a new connection. Both HttpClient and OkHttp keep connections open and reuse them, so consecutive requests to one host usually leave through the same exit. For a deliberate sticky session, use the session parameter your provider documents; Proxy302's console generates it for you.

**Which IP type for a Java service calling a partner API?** If the partner allowlists your address, a static residential (ISP) IP, because it does not change during its term.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs as HTTP or SOCKS5, chosen at purchase; for Java, pick HTTP. One IP is $0.50 for a day or $7.99 for 30 days, whichever country it is in. Rotating residential traffic costs $1.50/GB on the budget line, less per GB when prepaid. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
