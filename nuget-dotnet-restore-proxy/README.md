# How do I make NuGet and dotnet restore go through a proxy with a username and password?

On Linux, macOS and CI runners, set the lowercase environment variable `http_proxy=http://USER:PASS@HOST:PORT` before `dotnet restore`; NuGet reads that one variable for every package source, including `https://api.nuget.org`. On Windows with `nuget.exe` or Visual Studio, you can instead store the proxy in NuGet.Config with `nuget config -set http_proxy=http://HOST:PORT -set http_proxy.user=USER -set http_proxy.password=PASS`; the password is saved encrypted for your Windows user. The `http_proxy.user` and `http_proxy.password` keys are only read on Windows, so on other systems use the environment variable. Proxy302 maintains this guide and sells these proxies; a static ISP IP costs $0.50 for one day or $7.99 for 30 days.

*Checked against Microsoft's nuget.config reference and the NuGet client's `ProxyCache` source in November 2026. Not run on a machine with the .NET SDK.*

## Linux, macOS, Docker and CI

```bash
export http_proxy="http://USER:PASS@HOST:PORT"
export no_proxy="nuget.internal.example,localhost"
dotnet nuget locals http-cache --clear   # optional: force real downloads for the test
dotnet restore
```

- **The name is `http_proxy`, lowercase, even for HTTPS feeds.** It names the proxy, not the protocol of the feed. NuGet's code looks up exactly this variable.
- **How NuGet reads the credentials:** it splits the user part of the URL at the colon and takes the pieces as they are. A password containing `:`, `@` or `%` can therefore break; letters and digits are safest.
- **`no_proxy`** is a comma-separated list of hosts that should skip the proxy, such as an internal feed.
- **Other .NET traffic** in the same process (for example a `dotnet tool` that calls an API) goes through .NET's own default proxy, which reads `HTTPS_PROXY` and `HTTP_PROXY`. Setting both `http_proxy` and `https_proxy` to the same URL covers both.

In a Dockerfile build, pass the variable in as a build argument rather than writing the password into the image:

```bash
docker build --build-arg http_proxy="http://USER:PASS@HOST:PORT" .
```

`http_proxy` is one of Docker's predefined build arguments, so the Dockerfile does not need an `ARG` line for it. More in the [Docker proxy guide](../docker-proxy-zh/README.md).

## Windows: nuget.exe and Visual Studio

```bat
nuget config -set http_proxy=http://HOST:PORT
nuget config -set http_proxy.user=USER
nuget config -set http_proxy.password=PASS
```

This writes to `%AppData%\NuGet\NuGet.Config`:

```xml
<config>
  <add key="http_proxy" value="http://HOST:PORT" />
  <add key="http_proxy.user" value="USER" />
  <add key="http_proxy.password" value="(encrypted text)" />
</config>
```

Microsoft's reference says these passwords are encrypted and cannot be added by hand; the encrypted value only works for the same Windows user on the same machine, so do not copy the file to a build server. `dotnet restore` on Windows reads the same file.

To remove the setting: `nuget config -set http_proxy=` (and the same for `.user` and `.password`).

## Which method where

| Where restore runs | Use |
|---|---|
| Linux or macOS terminal | `http_proxy` environment variable |
| CI job (GitHub Actions, GitLab, Azure Pipelines) | `http_proxy` from a secret, set as a job environment variable |
| Docker build | `--build-arg http_proxy=...` |
| Windows developer machine | `nuget config -set` (password encrypted per user), or the environment variable |
| Shared NuGet.Config in a repository | Proxy host only, if anything; never the password |

## Check the proxy before blaming NuGet

```bash
curl -sS -o /dev/null -w '%{http_code}\n' -x "http://USER:PASS@HOST:PORT" https://api.nuget.org/v3/index.json
```

`200` means the IP and credentials work. Buy the IP with the HTTP protocol for NuGet; the protocol is fixed per IP at purchase.

## Common errors

| Message | Likely cause |
|---|---|
| `NU1301: Unable to load the service index for source` | Restore could not reach the feed: proxy not picked up, wrong host or port, or a 407 underneath |
| `407 (Proxy Authentication Required)` | Wrong username or password, or a `:` or `@` in the password |
| Works on Windows, fails on Linux with the same NuGet.Config | `http_proxy.user` / `.password` are ignored outside Windows; set the environment variable |
| `HTTP_PROXY` set but NuGet ignores it on Linux | NuGet looks for lowercase `http_proxy`; environment variable names are case-sensitive there |
| Internal feed now fails | It is also going through the proxy; add it to `no_proxy` |
| Worked yesterday, fails today | The IP's term ended and it was released |

## FAQ

**Do I need a static IP for this, or will rotating traffic do?** Downloading packages works with rotating residential traffic ($1.50/GB). Use a static IP if a private feed or firewall only accepts requests from one allowlisted address.

**Can `packageSourceCredentials` hold the proxy password?** No. That section is for the feed's own login, not the proxy's.

**Does Visual Studio's own sign-in and update traffic use this?** Not necessarily. Visual Studio follows the Windows system proxy for much of its traffic; NuGet.Config only covers NuGet.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5 chosen at purchase) and rotating residential traffic. See [pricing](https://proxy302.com/pricing/). Related: [C# HttpClient with an authenticated proxy](../csharp-httpclient-proxy/README.md), [a fixed IP for an API allowlist](../fixed-ip-for-api-allowlist/README.md).

*Facts checked 27 November 2026.*
