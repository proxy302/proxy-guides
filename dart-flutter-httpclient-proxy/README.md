# How do I use a proxy with a username and password in Dart and Flutter?

Set `findProxy` on a `dart:io` `HttpClient` and put the credentials inside the returned string: `client.findProxy = (uri) => 'PROXY USER:PASS@HOST:PORT';`. For `https://` URLs the credentials must be ready before Dart opens the CONNECT tunnel: either inside the `PROXY` string or registered in advance with `addProxyCredentials` for the same host and port. The `authenticateProxy` callback, which answers a 407 challenge, is not used for tunnels. Command-line Dart programs also read `https_proxy`, `http_proxy` and `no_proxy` by default, but Flutter apps on phones have no such variables, so set `findProxy` yourself. `dart:io` has no SOCKS support, so use an HTTP proxy. Proxy302 maintains this guide and sells these proxies.

*How we checked: we read the proxy code in the Dart SDK's `http_impl.dart` (main branch, November 2026) and the `HttpClient` API docs. We did not run Dart on our test machine.*

## dart:io HttpClient

```dart
import 'dart:convert';
import 'dart:io';

Future<void> main() async {
  final client = HttpClient()
    ..connectionTimeout = const Duration(seconds: 20)
    ..findProxy = (uri) => 'PROXY USER:PASS@HOST:PORT';

  try {
    final request = await client.getUrl(Uri.parse('https://ipinfo.io/json'));
    final response = await request.close();
    final body = await response.transform(utf8.decoder).join();
    print('${response.statusCode} $body');
  } finally {
    client.close();
  }
}
```

`USER`, `PASS`, `HOST` and `PORT` are placeholders. The output should show the proxy's IP and country.

### How the PROXY string is read

- The format is the one used by browser PAC files: `PROXY host:port`, `DIRECT`, or several entries separated by `;`.
- Credentials go before the host: `PROXY USER:PASS@HOST:PORT`. Dart splits the user from the password at the first `:` and the credentials from the host at the last `@`.
- The values are **not** percent-decoded. Write the password exactly as it is; `p%40ss` would be sent literally. A password containing `@` or `:` still works because of the split rules above; a username containing `:` does not.

### Keeping the password out of the string

Register Basic credentials for the proxy's host and port before the first request. Dart then adds them to every request to that proxy, including the CONNECT for `https://` URLs:

```dart
final client = HttpClient()
  ..findProxy = (uri) => 'PROXY HOST:PORT'
  ..addProxyCredentials('HOST', PORT, '', HttpClientBasicCredentials('USER', 'PASS'));
```

Here `PORT` is a number, and `HOST` and `PORT` must be exactly what `findProxy` returns. Digest credentials do not work this way, because they need a challenge from the proxy first.

### Send some hosts direct

```dart
client.findProxy = (uri) {
  if (uri.host == 'localhost' || uri.host.endsWith('.internal.example')) {
    return 'DIRECT';
  }
  return 'PROXY USER:PASS@HOST:PORT';
};
```

## Reading the proxy from the environment

On servers and in command-line tools, the default `findProxy` is `HttpClient.findProxyFromEnvironment`, which reads:

| Variable | Used for |
|---|---|
| `https_proxy` or `HTTPS_PROXY` | `https://` URLs |
| `http_proxy` or `HTTP_PROXY` | `http://` URLs |
| `no_proxy` or `NO_PROXY` | Comma-separated host suffixes that go direct |

```bash
export https_proxy="http://USER:PASS@HOST:PORT"
dart run bin/main.dart
```

Dart strips the `http://` prefix and cuts the value at the first `/`, so a password that contains `/` breaks this route; set `findProxy` in code in that case. Lowercase names are checked first.

## package:http

`package:http` uses `dart:io` underneath on mobile and desktop. Wrap your configured client in `IOClient`:

```dart
import 'dart:io';
import 'package:http/io_client.dart';

Future<void> main() async {
  final inner = HttpClient()..findProxy = (uri) => 'PROXY USER:PASS@HOST:PORT';
  final client = IOClient(inner);
  try {
    final res = await client.get(Uri.parse('https://ipinfo.io/json'));
    print(res.body);
  } finally {
    client.close();
  }
}
```

## Dio 5

```dart
import 'dart:io';
import 'package:dio/dio.dart';
import 'package:dio/io.dart';

final dio = Dio()
  ..httpClientAdapter = IOHttpClientAdapter(
    createHttpClient: () =>
        HttpClient()..findProxy = (uri) => 'PROXY USER:PASS@HOST:PORT',
  );
```

## Flutter

- **Android, iOS, desktop:** set `findProxy` as above. Pass the value at build time rather than hard-coding it:

  ```dart
  const proxyConf = String.fromEnvironment('PROXY_CONF'); // 'PROXY USER:PASS@HOST:PORT'
  ```

  ```bash
  flutter run --dart-define=PROXY_CONF="PROXY USER:PASS@HOST:PORT"
  ```

- **Do not ship real proxy credentials inside an app you publish.** Anything compiled into an app can be extracted. Use proxies like this for internal tools, tests and QA builds.
- **Flutter web:** the browser makes the requests and uses its own proxy settings; `dart:io` is not available.
- **`cupertino_http` and `cronet_http`:** these native clients follow the device's network settings rather than `findProxy`.

## Which approach where

| Situation | Use |
|---|---|
| Server or CLI, password without `/` | `https_proxy` environment variable |
| Any password, any platform except web | `findProxy` returning `PROXY USER:PASS@HOST:PORT` |
| Credentials kept out of the `PROXY` string | `addProxyCredentials(HOST, PORT, '', HttpClientBasicCredentials(USER, PASS))`, host and port matching `findProxy` |
| SOCKS5 IP | Not supported by `dart:io`; buy the IP as HTTP |

## Common errors

| Message | Cause | Fix |
|---|---|---|
| `HttpException: Proxy failed to establish tunnel (407 Proxy Authentication Required)` | HTTPS request with wrong credentials, or none registered in advance (an `authenticateProxy` callback is not consulted here) | Put `USER:PASS@` in the string, or register them with `addProxyCredentials`; recheck the password |
| `HttpException: Invalid proxy configuration …` | String not in `PROXY host:port` form, missing port, or `USER` without `:PASS` | Fix the format |
| `SocketException` with the proxy's host | Wrong host or port, or a SOCKS5 IP used as HTTP | Use the HTTP IP's host and port |
| Requests still leave from your own address | `findProxy` set on a different client than the one making requests | Configure the client you pass to `IOClient` or Dio |

## FAQ

**Can I check the exit IP from Dart?** Request `https://ipinfo.io/json` through the client, as in the first example, and compare `ip` and `country` with your order.

**Will a rotating gateway give each request a new IP?** Only on new connections. `HttpClient` keeps connections alive and reuses them, so requests in a row usually leave through the same exit. For one fixed address, use a static IP.

**Which IP type for an app's backend calls to an allowlisted API?** A static residential (ISP) IP, which keeps one address for its whole term.

**Is there a free trial?** No. One static IP for one day costs $0.50; 30 days costs $7.99.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (HTTP or SOCKS5, chosen per IP at purchase), rotating residential traffic from $1.50/GB and datacenter IPs. See [pricing](https://proxy302.com/pricing/). The same task in other languages: [Go](../go-http-proxy/README.md), [Rust reqwest](../rust-reqwest-proxy/README.md), [Java HttpClient](../java-httpclient-proxy/README.md).

*Facts checked 23 November 2026.*
