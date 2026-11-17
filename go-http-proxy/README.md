# How do I use a proxy in Go net/http?

Set the transport's `Proxy` field to `http.ProxyURL(u)`, where `u` is parsed from `http://USER:PASS@HOST:PORT`. Go sends the credentials as `Proxy-Authorization` and tunnels HTTPS through CONNECT. The same field also accepts `socks5://USER:PASS@HOST:PORT`, and current Go treats `socks5` like `socks5h`, so the proxy resolves hostnames. Reach for `golang.org/x/net/proxy` when you want a SOCKS5 dialer of your own, for example for raw TCP or a custom `DialContext`.

*Proxy302 maintains this guide and sells these proxies. Both programs below compile and pass `go vet` on the current Go release.*

## HTTP proxy

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"net/url"
	"time"
)

func main() {
	proxyURL, err := url.Parse("http://USER:PASS@HOST:PORT")
	if err != nil {
		log.Fatal(err)
	}

	transport := http.DefaultTransport.(*http.Transport).Clone()
	transport.Proxy = http.ProxyURL(proxyURL)

	client := &http.Client{Transport: transport, Timeout: 30 * time.Second}

	resp, err := client.Get("https://ipinfo.io/json")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(resp.Status, string(body))
}
```

Cloning `http.DefaultTransport` keeps its sensible dial, TLS and idle timeouts. For SOCKS5 through the standard library alone, change the string to `socks5://USER:PASS@HOST:PORT`.

If the password has characters such as `@` or `/`, build the URL instead of parsing a string: `&url.URL{Scheme: "http", User: url.UserPassword(user, pass), Host: "HOST:PORT"}`.

## SOCKS5 with golang.org/x/net/proxy

```bash
go get golang.org/x/net/proxy
```

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"golang.org/x/net/proxy"
)

func main() {
	auth := &proxy.Auth{User: "USER", Password: "PASS"}
	dialer, err := proxy.SOCKS5("tcp", "HOST:PORT", auth, proxy.Direct)
	if err != nil {
		log.Fatal(err)
	}
	cd, ok := dialer.(proxy.ContextDialer)
	if !ok {
		log.Fatal("SOCKS5 dialer has no DialContext")
	}

	transport := http.DefaultTransport.(*http.Transport).Clone()
	transport.Proxy = nil // the dialer handles the proxy; ignore HTTP(S)_PROXY
	transport.DialContext = cd.DialContext

	client := &http.Client{Transport: transport, Timeout: 30 * time.Second}

	resp, err := client.Get("https://ipinfo.io/json")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	body, _ := io.ReadAll(resp.Body)
	fmt.Println(resp.Status, string(body))
}
```

The dialer passes the hostname to the proxy, so DNS happens there. Setting `Proxy = nil` matters: the cloned default transport would otherwise also apply any `HTTPS_PROXY` it finds in the environment.

## Choosing an approach

| Approach | Proxy type | Notes |
|---|---|---|
| `http.ProxyURL` with `http://` | HTTP | CONNECT for HTTPS; credentials from the URL |
| `http.ProxyURL` with `socks5://` | SOCKS5 | No extra module |
| `x/net/proxy.SOCKS5` + `DialContext` | SOCKS5 | Also usable outside `net/http` |
| `http.ProxyFromEnvironment` (the default) | Whatever `HTTP_PROXY` / `HTTPS_PROXY` say | Respects `NO_PROXY`; never proxies requests to `localhost` |

`Transport.Proxy` is a function of the request, so you can return a different proxy URL per host or per job.

## Rotation and keep-alive

The transport keeps idle connections and reuses them, so on a rotating residential gateway back-to-back requests to one host usually leave from the same IP. Set `transport.DisableKeepAlives = true` to open a new connection, and so get a fresh exit, for every request. When a multi-step flow needs one IP, switch the proxy URL to a sticky session, using the session parameter your provider documents (Proxy302's console generates it for you).

## Common errors

| Error text | Cause |
|---|---|
| `proxyconnect tcp: dial tcp HOST:PORT: connect: connection refused` | Wrong host or port |
| `Get "...": Proxy Authentication Required` | The proxy returned 407; check username and password |
| `socks connect tcp HOST:PORT->...: username/password authentication failed` | SOCKS5 credentials refused |
| `invalid port ":PORT" after host` | Placeholder not replaced |
| `net/url: invalid userinfo` | Unescaped special characters; use `url.UserPassword` |

## FAQ

**Do I need x/net/proxy for SOCKS5 in an `http.Client`?** No; `socks5://` in `Transport.Proxy` works.

**Why is my local test server not proxied?** `ProxyFromEnvironment` skips `localhost` by design. Set `Transport.Proxy` explicitly.

**Which proxy for a service that must keep one IP?** A static residential (ISP) IP; at Proxy302 it costs $7.99 per 30 days, the same in every country.

## Where Proxy302 fits

Every Proxy302 product arrives as a host, port, username and password, which go straight into `url.Parse` or `proxy.Auth` above. Rotating residential begins at $1.50/GB, and the full list is on the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
