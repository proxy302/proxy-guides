# How do I use a proxy with a username and password in Ruby Net::HTTP?

Pass the proxy as extra arguments: `Net::HTTP.start(host, port, PROXY_HOST, PROXY_PORT, PROXY_USER, PROXY_PASS, use_ssl: true)`. Net::HTTP then sends a CONNECT request with Basic credentials for `https://` URLs and a normal proxied request for `http://` URLs. If you leave the proxy arguments out, Net::HTTP reads the `http_proxy` environment variable, including any `user:pass@` in it, and it uses that same variable for HTTPS requests too; `https_proxy` is never read. Net::HTTP has no SOCKS support, so order the IP as an HTTP proxy.

*Proxy302 maintains this guide and sells these proxies. The Net::HTTP code was run on Ruby 2.6 against a throwaway local proxy, and the behaviour was checked against the current `net/http` source (net-http 0.10, the version in recent Ruby releases) in November 2026. The Faraday and HTTParty snippets were checked against their documentation and source, not run.*

## Explicit proxy arguments

```ruby
require "net/http"
require "uri"

uri = URI("https://ipinfo.io/json")

Net::HTTP.start(uri.host, uri.port,
                ENV.fetch("PROXY_HOST"), Integer(ENV.fetch("PROXY_PORT")),
                ENV.fetch("PROXY_USER"), ENV.fetch("PROXY_PASS"),
                use_ssl: true, open_timeout: 15, read_timeout: 30) do |http|
  res = http.get(uri.request_uri)
  puts "#{res.code} #{res.body}"
end
```

```bash
PROXY_HOST=HOST PROXY_PORT=PORT PROXY_USER=USER PROXY_PASS=PASS ruby check.rb
```

The JSON should show the proxy's exit address and country. The argument order is fixed: target host, target port, proxy host, proxy port, proxy user, proxy password. Options such as `use_ssl:` go last as keywords.

The same works with `Net::HTTP.new` if you need the object first:

```ruby
http = Net::HTTP.new("ipinfo.io", 443, PROXY_HOST, PROXY_PORT, PROXY_USER, PROXY_PASS)
http.use_ssl = true
puts http.get("/ip").body
```

## Taking the proxy from the environment

`Net::HTTP.new(host, port)` with no proxy arguments uses `http_proxy` (and honours `no_proxy`). This catches people out in two ways:

| You set | `https://` request goes through the proxy? |
|---|---|
| `http_proxy=http://USER:PASS@HOST:PORT` | Yes |
| `https_proxy=http://USER:PASS@HOST:PORT` only | **No**, Net::HTTP connects directly |
| `HTTP_PROXY` (upper case) only | Yes, with a warning that the upper-case name is discouraged |

You can confirm what Net::HTTP picked before sending anything:

```ruby
http = Net::HTTP.new("ipinfo.io", 443)
puts "proxy? #{http.proxy?} via #{http.proxy_address}:#{http.proxy_port}"
```

To make sure a script never picks up a proxy from the environment by accident, pass `nil` as the third argument: `Net::HTTP.new(host, port, nil)`.

If the password contains `@`, `:` or `/`, percent-encode it in the environment variable (`p@ss` becomes `p%40ss`). Current net-http decodes it before building the header, but the old Ruby 2.6 that macOS still ships sends `p%40ss` literally and gets a 407. With explicit arguments, pass the password as it is, on any version.

## Faraday and HTTParty

Faraday 2 accepts a proxy URL with credentials:

```ruby
require "faraday"

conn = Faraday.new(url: "https://ipinfo.io", proxy: "http://USER:PASS@HOST:PORT")
puts conn.get("/json").body
```

HTTParty takes four options per request (or the `http_proxy` class method):

```ruby
require "httparty"

res = HTTParty.get("https://ipinfo.io/json",
                   http_proxyaddr: "HOST", http_proxyport: PORT,
                   http_proxyuser: "USER", http_proxypass: "PASS")
puts res.body
```

Both sit on top of Net::HTTP by default, so the same HTTP-only rule applies.

## Errors you may see

| Error | Meaning |
|---|---|
| `407 "Proxy Authentication Required"` (`Net::HTTPClientException`, `Net::HTTPServerException` on old Rubies) | Wrong username or password, or the credentials did not reach Net::HTTP |
| `EOFError: end of file reached` right after connecting | The port speaks SOCKS5, or the proxy hung up; Net::HTTP only talks HTTP to proxies |
| `Net::OpenTimeout` | Wrong host or port, or a firewall blocks the proxy port |
| Exit IP is your own | `https_proxy` set without `http_proxy`, or `nil` passed as the proxy address |

## FAQ

**Can Net::HTTP use a SOCKS5 proxy?** Not by itself. The simplest route is an HTTP proxy. At Proxy302 the protocol is chosen per IP at purchase, so choose HTTP for Ruby.

**Does `Net::HTTP.get(URI(...))` use the proxy?** Yes, the one-line helpers create a `Net::HTTP` with the default `:ENV` proxy setting, so they follow `http_proxy`.

**Will each request get a new IP on a rotating gateway?** Requests inside one `Net::HTTP.start` block reuse one connection and leave through one exit. A new `start` opens a new connection. For a deliberately fixed exit, use the session parameter your provider documents; Proxy302's console generates it for you.

**Where should the credentials live?** In environment variables or your secrets manager, not in the script, so they stay out of version control.

## Where Proxy302 fits

Proxy302 sells static residential (ISP) IPs as HTTP or SOCKS5, chosen at purchase, at $0.50 for one day or $7.99 for 30 days per IP in any country, and rotating residential traffic from $1.50/GB. See [pricing](https://proxy302.com/pricing/).

*Facts checked 20 November 2026.*
