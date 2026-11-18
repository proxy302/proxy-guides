# How do I use a proxy with a username and password in C# HttpClient?

Create a `WebProxy` for `http://HOST:PORT`, put `new NetworkCredential("USER", "PASS")` in its `Credentials` property, hand it to a `SocketsHttpHandler` with `UseProxy = true`, and build one long-lived `HttpClient` on that handler. On .NET 6 and later the same code accepts `socks5://HOST:PORT`, and the credentials are then sent as SOCKS5 username/password authentication. No NuGet package is needed for either.

*Proxy302 maintains this guide and sells these proxies. The code was checked against Microsoft's .NET networking documentation and the dotnet/runtime source in November 2026.*

## HTTP proxy

```csharp
using System.Net;

var proxy = new WebProxy("http://HOST:PORT")
{
    Credentials = new NetworkCredential("USER", "PASS"),
};

var handler = new SocketsHttpHandler
{
    Proxy = proxy,
    UseProxy = true,
};

using var client = new HttpClient(handler) { Timeout = TimeSpan.FromSeconds(30) };

string body = await client.GetStringAsync("https://ipinfo.io/json");
Console.WriteLine(body);
```

This is a complete `Program.cs` for a .NET 6+ console app with implicit usings. The output should show the proxy's IP and country. In real code, load the username and password from configuration or user secrets rather than writing them into the source.

Passing credentials through `NetworkCredential` has a side benefit: characters such as `@`, `:` or `#` in the password need no URL encoding, which they would if you wrote `http://USER:PASS@HOST:PORT`.

## SOCKS5 proxy (.NET 6 and newer)

Change only the address:

```csharp
var proxy = new WebProxy("socks5://HOST:PORT")
{
    Credentials = new NetworkCredential("USER", "PASS"),
};
```

.NET sends the target hostname to the SOCKS5 server rather than resolving it locally, and it treats `socks5://` and `socks5h://` the same way. .NET Framework 4.x has no SOCKS support; there, use an HTTP IP.

## ASP.NET Core with IHttpClientFactory

```csharp
builder.Services.AddHttpClient("proxied")
    .ConfigurePrimaryHttpMessageHandler(() => new SocketsHttpHandler
    {
        Proxy = new WebProxy("http://HOST:PORT")
        {
            Credentials = new NetworkCredential("USER", "PASS"),
        },
        UseProxy = true,
    });
```

Then inject `IHttpClientFactory` and call `CreateClient("proxied")`. Clients without this registration keep their normal network route.

## Proxy from environment variables

With no `Proxy` set on the handler, `HttpClient` uses `HttpClient.DefaultProxy`, which reads `HTTPS_PROXY`, `HTTP_PROXY`, `ALL_PROXY` and `NO_PROXY`:

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
dotnet run
```

Microsoft's documentation says the value must start with `http`, may include a username and password, and must have nothing after the port. If none of those variables is set, Windows falls back to the user's proxy settings, macOS to the system proxy settings, and Linux to no proxy at all.

## Which way to choose

| Situation | Approach |
|---|---|
| One service, one proxy | `SocketsHttpHandler` with `WebProxy` and `NetworkCredential` |
| Several named clients in ASP.NET Core | `AddHttpClient(...).ConfigurePrimaryHttpMessageHandler(...)` |
| Tool or script you don't want to recompile | `HTTPS_PROXY` environment variable |
| Non-HTTP traffic or an IP sold as SOCKS5 | `socks5://` in `WebProxy` (.NET 6+) |

## Rotation and connection reuse

`SocketsHttpHandler` keeps connections open and sends later requests over them, so a rotating gateway tends to give the same exit to consecutive requests for one host. To ask for a fresh connection on a given request, close it after use:

```csharp
using var request = new HttpRequestMessage(HttpMethod.Get, "https://ipinfo.io/json");
request.Headers.ConnectionClose = true;
using var response = await client.SendAsync(request);
```

When you want the opposite, a fixed exit for a short multi-step flow, use the session parameter your provider documents; Proxy302's console generates it for you.

Do not build a new `HttpClient` and handler for every request. That opens a new socket each time and can exhaust local ports under load; one client per proxy is the usual pattern.

## Common errors

| Message | Likely cause |
|---|---|
| `The proxy tunnel request to proxy 'http://HOST:PORT/' failed with status code '407'.` | Credentials missing or wrong for an HTTP proxy |
| Inner exception `Failed to authenticate with the SOCKS server.` | Wrong username or password on a SOCKS5 proxy |
| Inner exception `SOCKS server requested username & password authentication.` | `Credentials` not set on a SOCKS5 `WebProxy` |
| `Only the 'http', 'https', 'socks4', 'socks4a', 'socks5', and 'socks5h' schemes are allowed for proxies.` | A typo such as `socks://` in the proxy address |
| `The SSL connection could not be established, see inner exception.` | Often an `https://` proxy address for a plain HTTP proxy; see [SSL errors through a proxy](../proxy-ssl-errors/README.md) |
| `TaskCanceledException` after the timeout | The proxy or the site did not answer in time |

## FAQ

**Does `UseProxy = true` matter?** It is the default, but setting it explicitly makes the intent clear when someone reads the code later.

**Can I put the credentials in the URL, like `http://USER:PASS@HOST:PORT`, inside `WebProxy`?** Use `Credentials` instead. The environment-variable route accepts credentials in the URL; for `WebProxy` the documented way is the `Credentials` property.

**HTTP or SOCKS5 for .NET?** Either works on .NET 6+. HTTP also works on .NET Framework and with browsers driven from the same machine.

**Which IP for a service whose partner allowlists one address?** A static residential (ISP) IP, which keeps the same address for its whole term.

## Where Proxy302 fits

For a .NET service that needs one fixed address, a Proxy302 static residential (ISP) IP is $0.50 for a day or $7.99 for 30 days, priced the same in every country; HTTP or SOCKS5 is picked when you buy it. For many independent requests, rotating residential traffic is $1.50/GB pay as you go. A one-day IP is also the cheapest way to try the code above, since there is no free trial. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
