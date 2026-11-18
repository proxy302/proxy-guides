# How do I use a proxy in PHP cURL?

Set `CURLOPT_PROXY` to `http://HOST:PORT` and `CURLOPT_PROXYUSERPWD` to `USER:PASS` on the cURL handle. For an IP sold as SOCKS5, write `socks5h://HOST:PORT` instead, which also lets the proxy resolve hostnames. cURL tunnels `https://` URLs through an HTTP proxy on its own, so `CURLOPT_HTTPPROXYTUNNEL` is not needed. In Guzzle, and in Laravel's HTTP client built on it, the whole setup is one `proxy` request option.

*Proxy302 maintains this guide and sells these proxies. The PHP code was checked against the php.net manual and Guzzle documentation; the underlying libcurl options were run with the curl command line against a local authenticating test proxy in November 2026.*

## Plain cURL (PHP 8)

```php
<?php
$ch = curl_init('https://ipinfo.io/json');

curl_setopt_array($ch, [
    CURLOPT_PROXY          => 'http://HOST:PORT',
    CURLOPT_PROXYUSERPWD   => 'USER:PASS',
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_CONNECTTIMEOUT => 15,
    CURLOPT_TIMEOUT        => 30,
]);

$body = curl_exec($ch);

if ($body === false) {
    fwrite(STDERR, 'cURL error ' . curl_errno($ch) . ': ' . curl_error($ch) . PHP_EOL);
    exit(1);
}

printf(
    "proxy answered %d, site answered %d\n%s\n",
    curl_getinfo($ch, CURLINFO_HTTP_CONNECTCODE),
    curl_getinfo($ch, CURLINFO_RESPONSE_CODE),
    $body
);
```

`CURLINFO_HTTP_CONNECTCODE` is the status the proxy gave to the tunnel request, and `CURLINFO_RESPONSE_CODE` is the website's own status. Printing both tells you at once whether a problem sits with the proxy or with the site.

There is no `curl_close()` at the end. Since PHP 8.0 the handle is freed when it goes out of scope, and PHP 8.5 deprecates the function.

### Passwords with special characters

`CURLOPT_PROXYUSERPWD` splits on the first colon, so a colon in the username breaks it. Setting the two parts separately avoids any parsing:

```php
curl_setopt($ch, CURLOPT_PROXYUSERNAME, 'USER');
curl_setopt($ch, CURLOPT_PROXYPASSWORD, 'PASS');
```

## SOCKS5

```php
curl_setopt_array($ch, [
    CURLOPT_PROXY        => 'socks5h://HOST:PORT',
    CURLOPT_PROXYUSERPWD => 'USER:PASS',
]);
```

`socks5h` sends the hostname to the proxy. Plain `socks5` resolves it on your server first, which can leak DNS lookups to your local resolver. The older style, `CURLOPT_PROXYTYPE => CURLPROXY_SOCKS5_HOSTNAME` with `CURLOPT_PROXY => 'HOST:PORT'`, does the same; a scheme written into `CURLOPT_PROXY` takes precedence over `CURLOPT_PROXYTYPE`.

## Guzzle 7

```php
use GuzzleHttp\Client;

$client = new Client(['timeout' => 30]);

$response = $client->get('https://ipinfo.io/json', [
    'proxy' => 'http://USER:PASS@HOST:PORT',
]);

echo $response->getStatusCode(), ' ', $response->getBody(), PHP_EOL;
```

To proxy only some traffic, pass an array: `'proxy' => ['http' => 'http://USER:PASS@HOST:PORT', 'https' => 'http://USER:PASS@HOST:PORT', 'no' => ['localhost', '.internal.example']]`. Because the credentials sit inside a URL here, run each through `rawurlencode()` if it contains `@`, `:`, `/` or `#`.

## Laravel HTTP client

```php
use Illuminate\Support\Facades\Http;

$response = Http::withOptions([
    'proxy' => 'http://USER:PASS@HOST:PORT',
])->timeout(30)->get('https://ipinfo.io/json');
```

`withOptions()` passes the array straight to Guzzle, so the array form above works here too.

## Options at a glance

| Goal | Plain cURL | Guzzle / Laravel |
|---|---|---|
| HTTP proxy | `CURLOPT_PROXY => 'http://HOST:PORT'` | `'proxy' => 'http://USER:PASS@HOST:PORT'` |
| Credentials | `CURLOPT_PROXYUSERPWD` or `CURLOPT_PROXYUSERNAME` + `CURLOPT_PROXYPASSWORD` | Inside the proxy URL, URL-encoded |
| SOCKS5 | `'socks5h://HOST:PORT'` | `'socks5h://USER:PASS@HOST:PORT'` |
| Skip the proxy for some hosts | `CURLOPT_NOPROXY => 'localhost,.internal.example'` | `'no' => [...]` in the array form |

## Common errors

`curl_error()` returns libcurl's own message, so the wording can differ slightly between libcurl versions.

| `curl_errno` and message | Cause |
|---|---|
| 56, `CONNECT tunnel failed, response 407` | Wrong or missing proxy username or password |
| 56, `CONNECT tunnel failed, response 502` (or 503) | The proxy could not reach the site; see [ERR_TUNNEL_CONNECTION_FAILED](../err-tunnel-connection-failed/README.md) for the same failure in a browser |
| 7, `Failed to connect to ... port ...` | Wrong host or port, or your server's firewall blocks outbound traffic to that port |
| 28, timed out | The proxy did not answer in time |
| 5, `Unsupported proxy syntax` | An unencoded `@` or `:` inside credentials written into a URL |
| 97, `connection to proxy closed` | SOCKS5 credentials refused, or a SOCKS scheme pointed at an HTTP port |
| 35, SSL connect error | `https://` written for a proxy that only speaks plain HTTP; see [SSL errors through a proxy](../proxy-ssl-errors/README.md) |

## FAQ

**Does PHP cURL read `http_proxy` from the environment?** libcurl does when no proxy option is set, but PHP-FPM and web servers often do not pass shell variables to PHP. Set the option in code so the behaviour does not depend on how PHP was started.

**Do I need `CURLOPT_HTTPPROXYTUNNEL`?** Not for `https://` URLs, which cURL tunnels anyway. It only forces a tunnel for `http://` URLs as well.

**How do I check the exit IP from PHP?** Request `https://ipinfo.io/json` through the proxy, as above, and compare `ip` and `country` with what you bought.

**Static or rotating for a PHP scraper?** Rotating residential for many independent page requests; a static residential (ISP) IP when the other side expects one fixed address, for example an allowlisted API.

## Where Proxy302 fits

Proxy302 sells rotating residential traffic at $1.50/GB on the budget line (prepaid traffic is cheaper per GB and does not expire) and static residential (ISP) IPs at $0.50 a day or $7.99 per 30 days per IP, HTTP or SOCKS5 chosen at purchase. Either drops into the options above. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
