# How do I use an authenticated proxy with PowerShell Invoke-WebRequest?

Pass the proxy address with `-Proxy` and the username and password as a credential object with `-ProxyCredential`: `Invoke-WebRequest https://ipinfo.io/json -Proxy 'http://HOST:PORT' -ProxyCredential $cred`. This works the same in Windows PowerShell 5.1 and PowerShell 7, and `Invoke-RestMethod` takes the same two parameters. PowerShell 7 can also read `HTTPS_PROXY`, `HTTP_PROXY`, `ALL_PROXY` and `NO_PROXY` with the credentials inside the URL, and only PowerShell 7 can use a SOCKS5 proxy. Proxy302 maintains this guide and sells these proxies; for PowerShell scripts, buy the IP as HTTP.

## One request

```powershell
$cred = Get-Credential -UserName 'USER' -Message 'Proxy password'
Invoke-RestMethod -Uri 'https://ipinfo.io/json' -Proxy 'http://HOST:PORT' -ProxyCredential $cred |
    Select-Object ip, country, org
```

`USER`, `HOST` and `PORT` are placeholders. `Get-Credential` asks for the password in a prompt. The `-UserName` and `-Message` parameters exist in PowerShell 5.1 and 7.

If `country` shows the country you bought, the proxy works.

## In a script, without a prompt

Read the password from an environment variable or a secret store instead of typing it into the script:

```powershell
$secure = ConvertTo-SecureString -String $env:PROXY_PASS -AsPlainText -Force
$cred   = [pscredential]::new($env:PROXY_USER, $secure)

$resp = Invoke-WebRequest -Uri 'https://ipinfo.io/json' `
    -Proxy 'http://HOST:PORT' -ProxyCredential $cred -UseBasicParsing
$resp.StatusCode
$resp.Content
```

- `-UseBasicParsing` matters only in Windows PowerShell 5.1, where it skips the Internet Explorer engine. PowerShell 7 accepts it and ignores it.
- Do not put the username and password into the `-Proxy` URL. Use `-ProxyCredential`.
- `-ProxyCredential` and `-ProxyUseDefaultCredentials` cannot be used in the same command. The second one sends your Windows sign-in, which is for company proxies, not for a proxy with its own username and password.

## Every web call in a session

Set default parameter values once, and every later `Invoke-WebRequest` and `Invoke-RestMethod` call in that session uses them:

```powershell
$PSDefaultParameterValues['Invoke-WebRequest:Proxy']           = 'http://HOST:PORT'
$PSDefaultParameterValues['Invoke-WebRequest:ProxyCredential'] = $cred
$PSDefaultParameterValues['Invoke-RestMethod:Proxy']           = 'http://HOST:PORT'
$PSDefaultParameterValues['Invoke-RestMethod:ProxyCredential'] = $cred
```

Put these lines in your profile (`$PROFILE`) to apply them to every session. Remove them with `$PSDefaultParameterValues.Remove('Invoke-WebRequest:Proxy')` and so on.

## PowerShell 7: environment variables

From PowerShell 7.0, the web cmdlets take their default proxy from .NET's `HttpClient.DefaultProxy`, which reads environment variables first. On Windows and macOS it falls back to the system proxy if they are not set; on Linux it falls back to no proxy.

```powershell
$env:HTTPS_PROXY = 'http://USER:PASS@HOST:PORT'
$env:HTTP_PROXY  = 'http://USER:PASS@HOST:PORT'
$env:NO_PROXY    = 'localhost,127.0.0.1,.corp.example'
Invoke-RestMethod 'https://ipinfo.io/json'
```

Three details:

- **Set them before the first web request in the session.** The default proxy is created once per process, so changing the variables afterwards has no effect until you open a new `pwsh` window. Or replace the default proxy directly; the next web call picks it up:

  ```powershell
  $wp = [System.Net.WebProxy]::new('http://HOST:PORT')
  $wp.Credentials = $cred.GetNetworkCredential()
  [System.Net.Http.HttpClient]::DefaultProxy = $wp
  ```
- **`NO_PROXY` matches names, not wildcards.** `.corp.example` with a leading dot matches `www.corp.example`; asterisks are not supported.
- **Skip the proxy for one call** with `-NoProxy`. It cannot be combined with `-Proxy` in the same command.

Percent-encode special characters in a password inside a URL: `@` becomes `%40`, `:` becomes `%3A`, `#` becomes `%23`. `[uri]::EscapeDataString('p@ss')` does it for you.

## SOCKS5

.NET accepts `socks5://USER:PASS@HOST:PORT` in the proxy environment variables, so in PowerShell 7 this works:

```powershell
$env:ALL_PROXY = 'socks5://USER:PASS@HOST:PORT'
Invoke-RestMethod 'https://ipinfo.io/json'
```

`ALL_PROXY` applies when `HTTP_PROXY` and `HTTPS_PROXY` are not set, so clear those first. Windows PowerShell 5.1 runs on .NET Framework, which has no SOCKS support. If you have the choice, an HTTP IP avoids the question.

## Windows PowerShell 5.1: one proxy for everything

In 5.1, web cmdlets without `-Proxy` use the Windows proxy settings, which have no place for a password, so you get a 407. Set a default proxy object with credentials for the session:

```powershell
$p = New-Object System.Net.WebProxy('http://HOST:PORT')
$p.Credentials = $cred.GetNetworkCredential()
[System.Net.WebRequest]::DefaultWebProxy = $p
Invoke-WebRequest 'https://ipinfo.io/json' -UseBasicParsing
```

## Which method to use

| Situation | Use |
|---|---|
| A single call, any version | `-Proxy` plus `-ProxyCredential` |
| Every call in an interactive session | `$PSDefaultParameterValues` |
| PowerShell 7 in CI or a container | `HTTPS_PROXY` / `HTTP_PROXY` with the credentials in the URL |
| SOCKS5 IP | PowerShell 7 with `ALL_PROXY=socks5://…` |
| Windows PowerShell 5.1, every call | `[System.Net.WebRequest]::DefaultWebProxy` |

## Common errors

| Message | Cause | Fix |
|---|---|---|
| PowerShell 7: `The proxy tunnel request to proxy 'http://HOST:PORT/' failed with status code '407'` | Wrong or missing credentials | Check `-ProxyCredential`, or the user and password in the variable |
| 5.1: `The remote server returned an error: (407) Proxy Authentication Required.` | Same | Same |
| 5.1: an error that the ServicePointManager does not support proxies with the socks5 scheme | SOCKS5 in Windows PowerShell | Use PowerShell 7 or an HTTP IP |
| `Parameter set cannot be resolved` | `-NoProxy` together with `-Proxy` | Use one or the other |
| Environment variable change ignored | Default proxy already created in this session | Open a new `pwsh` window |
| `curl -x …` fails in 5.1 with a parameter error | `curl` is an alias for `Invoke-WebRequest` in Windows PowerShell | Type `curl.exe` to run the real curl |

## FAQ

**Does `-Proxy` change the Windows proxy settings?** No. It applies to that one command.

**Will a rotating gateway give each call a new IP?** Only when a new connection is opened. If you want the same exit for a run, use a static IP, or the session parameter your provider documents (Proxy302's console generates it for you).

**Which IP type for a script that calls an allowlisted API?** A static residential (ISP) IP: the address stays the same for its whole term, so the allowlist entry keeps working. See [fixed IP for an API allowlist](../fixed-ip-for-api-allowlist/README.md).

**Is there a free trial?** No. One static IP for one day costs $0.50; 30 days costs $7.99.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (HTTP or SOCKS5, chosen at purchase), rotating residential traffic from $1.50/GB, and datacenter IPs. See [pricing](https://proxy302.com/pricing/). For the same job in other tools, see the [curl proxy cheatsheet](../curl-proxy-cheatsheet/README.md) and [C# HttpClient proxy](../csharp-httpclient-proxy/README.md).

*Facts checked 23 November 2026.*
