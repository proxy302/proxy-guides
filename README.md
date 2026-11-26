# Proxy302 Proxy Guides

Maintained by Proxy302 (the official Proxy302 organization on GitHub). These guides describe proxy choices in general and mention Proxy302 where relevant; we are a vendor, not a neutral reviewer.

Plain-language how-to guides for people buying and using proxy IPs: rotating residential, static residential (ISP) and datacenter. Each guide answers the question in the first paragraph, then gives steps, tables and an FAQ. The first five guides have an English file and a Chinese file; newer guides are written in the language people ask the question in.

Read these guides as a website: [proxy302.github.io/proxy-guides](https://proxy302.github.io/proxy-guides/).

Proxy302: [https://proxy302.com](https://proxy302.com). Prices: [https://proxy302.com/pricing/](https://proxy302.com/pricing/). Prices change; confirm on the live pages.

## Guides

| Question | Short answer |
|---|---|
| [How do I choose a static residential IP?](how-to-choose-static-residential-ip/README.md) | Pick the job first, then the account's country, one IP per account, HTTP or SOCKS5, and test with the smallest purchase before committing. |
| [ISP proxy vs residential proxy: what is the difference?](isp-proxy-vs-residential-proxy/README.md) | An ISP proxy is a fixed address registered to a consumer ISP; a rotating residential proxy uses household addresses that change per connection. |
| [How do I set up a static residential IP for TikTok?](static-residential-ip-for-tiktok/README.md) | One static IP per account, in the account's market; paste it into that account's profile or phone, check the exit, then sign in. |
| [How do I pair an anti-detect browser with a residential or ISP IP?](antidetect-browser-with-residential-ip/README.md) | One IP per profile, matching proxy type, run the proxy check and a WebRTC test; includes curl and Python test snippets. |
| [What IP should I use for Claude and ChatGPT?](which-ip-for-claude-and-chatgpt/README.md) | One stable IP per person in a supported country where you actually are; a proxy does not change either service's rules. |
| [What is the best residential proxy for TikTok?](best-residential-proxy-for-tiktok/README.md) | Static ISP for accounts you log in to, one per account; rotating residential only for logged-out viewing. |
| [Which proxy for Facebook ads accounts?](proxy-for-facebook-ads-accounts/README.md) | One static ISP IP per ad account and browser profile, kept for the account's life; never rotate a logged-in ad account. |
| [Which proxy for running multiple accounts?](proxy-for-multiple-accounts/README.md) | One account, one static IP, one browser profile; rotating traffic is for scraping, not logins. |
| [Which proxy works best with AdsPower?](proxy-for-adspower/README.md) | Static ISP IP per AdsPower profile for logged-in accounts; set it as a custom HTTP or SOCKS5 proxy and run the check. |
| [Which proxy works best with Multilogin?](proxy-for-multilogin/README.md) | Static ISP IP per Multilogin profile; match the protocol, check the exit inside the profile, keep the IP. |
| [How do I choose an ISP proxy provider?](how-to-choose-an-isp-proxy-provider/README.md) | Check who the netblock is registered to, public per-IP prices, short test terms, renewal rules and delivery checks. |
| [How much does a residential proxy cost per GB?](residential-proxy-price-per-gb/README.md) | Per-GB price depends on line and prepaid volume; page weight decides the bill more than the rate. |
| [How do I make Claude Code use an HTTP proxy?](claude-code-proxy-settings/README.md) | Set `HTTPS_PROXY` (or the `env` block in Claude Code's settings file); SOCKS is not supported; confirm with `/status`. |
| [What proxy should I use for an Amazon Seller Central account?](proxy-for-amazon-seller-account/README.md) | One static ISP IP per seller account, kept for the account's life; it doesn't change Amazon's account rules or replace verification. |
| [Datacenter vs residential proxy: which do I need?](datacenter-vs-residential-proxy/README.md) | Datacenter for APIs, monitoring and sites that treat all visitors alike; residential for local consumer views and signed-in accounts. |
| [How do I use a proxy with Python requests?](python-requests-proxy/README.md) | Map http and https to one proxy URL; for SOCKS5 install `requests[socks]` and use `socks5h://`. One-off calls can rotate; a Session keeps one exit. |
| [How do I use a proxy with httpx or aiohttp?](httpx-aiohttp-proxy/README.md) | httpx 0.28 takes `proxy=`; aiohttp takes `proxy=` plus `proxy_auth=` and HTTP proxies only. SOCKS5 needs `httpx[socks]` or `aiohttp-socks`. |
| [How do I set a proxy in Playwright?](playwright-proxy/README.md) | Pass `{server, username, password}` to `launch()` or `new_context()`; with a password use HTTP, because Playwright rejects authenticated SOCKS5. |
| [How do I use an authenticated proxy with Puppeteer?](puppeteer-proxy/README.md) | Launch with `--proxy-server` and call `page.authenticate()`; `createBrowserContext({proxyServer})` gives one proxy per context. |
| [How do I use a proxy with Selenium?](selenium-proxy/README.md) | `--proxy-server` for proxies without a password; for a password, run a local forwarder. selenium-wire is archived. |
| [How do I use rotating residential proxies with Scrapy?](scrapy-proxy/README.md) | Set `meta["proxy"]` or a small middleware and retry 403/429; Scrapy reuses tunnels, so send `Connection: close` to rotate on retry. |
| [How do I use a proxy in Node.js?](nodejs-proxy/README.md) | undici `ProxyAgent` with undici `fetch`, or `NODE_USE_ENV_PROXY=1`; axios with `https-proxy-agent` and `proxy: false`; SOCKS5 via `socks-proxy-agent`. |
| [curl proxy cheat sheet: HTTP, HTTPS, SOCKS5, auth, exit IP](curl-proxy-cheatsheet/README.md) | `-x` with a full proxy URL, `-U` for credentials, `socks5h://` for SOCKS5; `-w %{http_connect}` separates the proxy's status from the site's. |
| [How do I use a proxy in Go net/http?](go-http-proxy/README.md) | `Transport.Proxy = http.ProxyURL(u)` handles `http://` and `socks5://`; disable keep-alives to rotate per request. |
| [SOCKS5 vs HTTP proxy: which should I use?](socks5-vs-http-proxy/README.md) | HTTP for web work and browser automation (Chromium can't authenticate to SOCKS5); SOCKS5 for non-HTTP apps, with `socks5h`. |
| [Rotating vs sticky residential sessions: when to use each](rotating-vs-sticky-sessions/README.md) | Rotating for independent requests, sticky for short multi-step flows, a static ISP IP for accounts that stay logged in. |
| [How can I tell if a proxy IP is residential, ISP or datacenter?](check-if-ip-is-residential/README.md) | Check the block's registrant and ASN, the reverse DNS name and two or three IP databases; no single check is proof. |
| [How do I cut bandwidth when scraping through residential proxies?](proxy-for-web-scraping-bandwidth/README.md) | Block images, fonts and media, call JSON endpoints, use compression and caching, retry only 403/429 with a cap; worked cost example included. |
| [What is the best proxy for ChatGPT?](proxy-for-chatgpt/README.md) | One fixed IP per person in a country OpenAI supports, where you actually are; an IP doesn't change OpenAI's rules. Proxy302 GPT/Grok tier 10 PTC/30 days. |
| [What proxy should I use for Claude?](proxy-for-claude/README.md) | One fixed IP per person in a supported country where you are, set in one place; Claude tier 15 PTC/30 days. An IP doesn't change Anthropic's policies. |
| [How do I choose a residential proxy provider?](how-to-choose-a-residential-proxy-provider/README.md) | Check public per-GB prices and prepaid tiers, traffic expiry, sticky sessions, targeting, a small test, protocols and written terms. |
| [What is the best static residential proxy?](what-is-the-best-static-residential-proxy/README.md) | The one whose IPs pass your own checks (ISP registrant, country, databases, blocklists, stays fixed); test one IP for a day at $0.50. |
| [What is the cheapest residential proxy per GB, really?](cheapest-residential-proxy-per-gb/README.md) | Money paid ÷ GB actually used after expiry, minimums and retries; worked examples from $1.50 to $1.124/GB. |
| [What should I check before buying a cheap residential proxy?](cheap-residential-proxy-providers/README.md) | Whether traffic expires, whether the low rate needs a big minimum, and whether exits belong to consumer ISPs; buy small and test. |
| [What is the best ISP proxy for TikTok?](best-isp-proxy-for-tiktok/README.md) | A static ISP IP in the account's country, kept for the account's life, one per phone or profile; TikTok's terms still apply. |
| [How do I use an authenticated proxy with Java HttpClient?](java-httpclient-proxy/README.md) | Use `ProxySelector.of` plus an `Authenticator` for `RequestorType.PROXY`, and clear `jdk.http.auth.tunneling.disabledSchemes`, or HTTPS returns 407. |
| [How do I use a proxy with a username and password in C# HttpClient?](csharp-httpclient-proxy/README.md) | `WebProxy` with `NetworkCredential` on a `SocketsHttpHandler`; `socks5://` works on .NET 6+, and `HTTPS_PROXY` works without code changes. |
| [How do I use a proxy in PHP cURL?](php-curl-proxy/README.md) | `CURLOPT_PROXY` plus `CURLOPT_PROXYUSERPWD`, `socks5h://` for SOCKS5; Guzzle and Laravel take one `proxy` option. |
| [How do I make wget use a proxy with a username and password?](wget-proxy/README.md) | Set `https_proxy=http://HOST:PORT` and pass `--proxy-user`/`--proxy-password` (or put them in `~/.wgetrc`); wget has no SOCKS support. |
| [How do I make git clone and push go through a proxy?](git-proxy/README.md) | `git config --global http.proxy` (or per-URL `http.<url>.proxy`), `socks5h://` for SOCKS5; SSH remotes need a `ProxyCommand`. |
| [How do I use npm behind a proxy?](npm-proxy/README.md) | `npm config set https-proxy` and `proxy`; pnpm reads the same `.npmrc` keys; Yarn 2+ uses `httpsProxy` in `.yarnrc.yml`. |
| [How do I open one Chrome window with its own proxy?](chrome-proxy-one-window/README.md) | Start Chrome with a new `--user-data-dir` and `--proxy-server=http://HOST:PORT`, then sign in to the proxy in the prompt; Chrome cannot log in to SOCKS5. |
| [What does ERR_TUNNEL_CONNECTION_FAILED mean, and how do I fix it?](err-tunnel-connection-failed/README.md) | Chrome reached the proxy but its CONNECT got 403, 502 or 503; read the status with curl `%{http_connect}` and fix the cause. |
| [Why do I get SSL errors when I use a proxy?](proxy-ssl-errors/README.md) | Either the proxy URL says `https://` for a plain HTTP proxy (use `http://`), or something intercepts TLS (trust its CA; never turn verification off). |
| [What does curl's "Proxy CONNECT aborted" error mean?](curl-proxy-connect-aborted/README.md) | The proxy closed the connection before answering curl's CONNECT: usually an `http://` URL on a SOCKS5 port, a refused client or an unreachable target; five checks. |
| [Why do I get ECONNRESET in Node.js when I send requests through a proxy?](node-econnreset-proxy/README.md) | The proxy or the site reset the connection; read `err.cause`, close idle keep-alive sockets early and retry idempotent requests with undici `RetryAgent`. |
| [Why does a website return 403 Forbidden only when I use a proxy?](site-403-only-through-proxy/README.md) | Check who sent it (`%{http_connect}` vs `%{http_code}`); if the site, look at the IP type, country and request rate. A proxy does not override a site's rules. |
| [How do I use a proxy with a username and password in Ruby Net::HTTP?](ruby-net-http-proxy/README.md) | Pass host, port, user and password to `Net::HTTP.start`; from the environment it reads only `http_proxy`, even for HTTPS; no SOCKS support. |
| [How do I SSH to a server through an HTTP or SOCKS5 proxy?](ssh-through-proxy/README.md) | A `ProxyCommand` with `nc -X connect` / `nc -X 5`, or `ncat --proxy-auth` when the proxy needs a password; also covers proxies that only allow port 443. |
| [Which proxy do I need for an API that only accepts allowlisted IP addresses?](fixed-ip-for-api-allowlist/README.md) | A static IP, not rotating traffic: datacenter is usually enough ($3.99/30 days), ISP for services that refuse hosting networks; route only the API calls through it. |
| [How do I use a proxy with a username and password in Rust reqwest?](rust-reqwest-proxy/README.md) | `Proxy::all("http://HOST:PORT")?.basic_auth(..)` on `ClientBuilder::proxy`; the `socks` feature plus `socks5h://` for SOCKS5. |
| [How do I send Postman requests through a proxy with a username and password?](postman-proxy/README.md) | Settings → Proxy → custom proxy configuration, tick HTTP and HTTPS, turn on Proxy auth; Newman reads `HTTPS_PROXY`/`HTTP_PROXY` instead. |
| [How do I set a proxy with a username and password in IntelliJ IDEA and other JetBrains IDEs?](jetbrains-ide-proxy/README.md) | Settings → System Settings → HTTP Proxy → Manual plus Proxy authentication; this covers the IDE only, Gradle and Maven need their own settings. |
| [What is a dual ISP IP, and how do I check one myself? (2026)](what-is-dual-isp-ip/README.md) | IPinfo's ASN type and IP2Location's usage type must both say ISP; check both public lookup pages yourself. Proxy302 Claude and GPT/Grok tiers give new orders dual-ISP ranges first. |
| [Native IP vs broadcast IP: what the labels mean and when it matters (2026)](native-vs-broadcast-ip/README.md) | Native means registry, geolocation and announcing network name one country; broadcast means the registry points elsewhere. Check with ipinfo and whois; it says nothing about residential vs datacenter. |
| [Which IP for a long-term Gemini Advanced / Google AI Pro account? (2026)](proxy-for-gemini/README.md) | One fixed IP per person in a country on Google's Gemini list (and the shorter AI Studio list) where you are; Proxy302 Gemini tier 8 PTC/30 days. An IP doesn't change Google's rules. |
| [What proxy should I use for an eBay seller account?](proxy-for-ebay-seller-account/README.md) | One static ISP IP per account in one browser profile, kept for the account's life; eBay allows multiple accounts but not to avoid restrictions, and limits can reach linked accounts. |
| [What proxy should I use for an Etsy shop, or several Etsy shops?](proxy-for-etsy-shop/README.md) | One static ISP IP and one browser profile per Etsy account; Etsy allows multiple shops with a separate account and email each, listed on every public profile. |
| [What proxy should I use for Google Ads accounts?](proxy-for-google-ads-accounts/README.md) | Use a manager account for many accounts; add one static ISP IP per person only where their connection is unstable or shared. New accounts after a suspension break Google's policy. |
| [How do I set a proxy on an iPhone (Wi-Fi HTTP proxy) and check it works?](iphone-proxy-settings/README.md) | Settings → Wi-Fi → (i) → Configure Proxy → Manual, turn on Authentication, then check ipinfo.io in Safari; HTTP only, one Wi-Fi network, not mobile data. |
| [Which proxy should I use for price monitoring?](proxy-for-price-monitoring/README.md) | Rotating residential per GB for sites that price by country or refuse servers; datacenter when the target treats all visitors alike or allowlists one IP. Estimate GB from page weight first. |
| [How do I check an IP before I use it for ChatGPT, Claude or Grok?](check-ip-before-ai-subscription/README.md) | Through the proxy, confirm the exit country is one the service supports and where you are, that IPinfo and IP2Location both call it ISP (dual ISP), that registry, database and operator countries match (native), that the address stays the same, and that the service's API answers with a login error rather than a region error. |
| [Which static IP should a Walmart Marketplace seller use in the US?](walmart-seller-static-ip-us/README.md) | One US static ISP IP per Walmart Seller Center account, kept for the account's life in one browser profile, with each team member signing in as their own Seller Center user; rotating residential only for logged-out checks. |
| [Which static IP should a Mercado Livre seller in Brazil use?](mercado-livre-seller-static-ip-brazil/README.md) | One Brazilian static ISP IP per Mercado Livre seller account, used only in that account's browser profile and kept long-term; Brazilian rotating residential for logged-out price and search checks. |
| [How do I use an authenticated proxy with PowerShell Invoke-WebRequest?](powershell-invoke-webrequest-proxy/README.md) | Pass `-Proxy 'http://HOST:PORT'` with a PSCredential in `-ProxyCredential` (works in 5.1 and 7); PowerShell 7 also reads HTTPS_PROXY/HTTP_PROXY/ALL_PROXY/NO_PROXY with credentials in the URL and is the only one that can use SOCKS5. |
| [How do I send a Kubernetes pod's outbound traffic through a proxy?](kubernetes-pod-egress-proxy/README.md) | Set HTTPS_PROXY/HTTP_PROXY/NO_PROXY (and lowercase copies) on the container, building the URL from a Secret with $(VAR) references, keep cluster names and the API server IP in NO_PROXY, and configure containerd separately for image pulls. |
| [How do I run the AWS CLI, gcloud and Azure CLI through a proxy with a username and password?](cloud-cli-proxy-aws-gcloud-az/README.md) | Export HTTPS_PROXY=http://USER:PASS@HOST:PORT for aws and az; for gcloud use the same variables or the proxy/* properties with CLOUDSDK_PROXY_USERNAME/PASSWORD; on cloud VMs add 169.254.169.254 to NO_PROXY. |
| [How do I use a proxy with a username and password in Dart and Flutter?](dart-flutter-httpclient-proxy/README.md) | Set `findProxy` to return `PROXY USER:PASS@HOST:PORT` (or register Basic credentials with addProxyCredentials for the same host and port) because HTTPS tunnels never consult authenticateProxy; dart:io has no SOCKS support. |
| [How do I re-check my ChatGPT, Claude or Grok IP after each renewal?](recheck-ai-subscription-ip-after-renewal/README.md) | Within a day of each renewal, rerun the purchase-day checks (same exit IP and country, IPinfo and IP2Location both ISP, native, API answers 401 not a region error), compare them with your first record, and resolve any change before the next sign-in. |
| [Which static IP should an eBay UK (ebay.co.uk) seller use?](ebay-uk-seller-static-ip/README.md) | One UK static ISP IP per ebay.co.uk seller account, kept for the account's life in one browser profile, with staff added through eBay's Team access instead of sharing the owner's password. |
| [How do I use a proxy with a username and password in libcurl (C or C++)?](libcurl-proxy-c-cpp/README.md) | Set CURLOPT_PROXY to http://HOST:PORT (or socks5h://HOST:PORT) and pass the credentials with CURLOPT_PROXYUSERNAME and CURLOPT_PROXYPASSWORD, then read CURLINFO_HTTP_CONNECTCODE on failure, where 407 means wrong credentials. |
| [How do I set a SOCKS5 or HTTP proxy with a username and password in Telegram Desktop?](telegram-desktop-proxy/README.md) | Settings, Advanced, Connection type, Use custom proxy, Add proxy, choose SOCKS5 or HTTP to match the IP, fill in Hostname, Port, Username and Password, and save. |
| [How do I run the Android Emulator through a proxy with a username and password?](android-emulator-proxy/README.md) | Start the emulator with -http-proxy http://USER:PASS@HOST:PORT or set it in Extended controls, Settings, Proxy and restart; it carries all TCP traffic through an HTTP proxy, with no SOCKS5, UDP or bypass list. |
| [How do I make an Electron app use a proxy with a username and password?](electron-app-proxy-auth/README.md) | Call session.setProxy({ proxyRules: 'http://HOST:PORT' }) and answer the app 'login' event for authInfo.isProxy with callback(USER, PASS), using an HTTP proxy because Chromium has no SOCKS5 username and password support. |
| [How do I connect to a WebSocket through an authenticated proxy in Python?](python-websocket-proxy/README.md) | In websockets 15+ pass proxy="http://USER:PASS@HOST:PORT" to connect(); in aiohttp pass proxy= and proxy_auth=BasicAuth to ws_connect(), and remember aiohttp's environment lookup uses wss_proxy, not HTTPS_PROXY, for wss:// URLs. |

*Black Friday, 24 November to 1 December 2026 (UTC+8): from the second IP, the AI subscription ISP costs 12 PTC (Claude tier), 8 PTC (GPT/Grok tier) and 6 PTC (Gemini tier) per 30 days, and the first IP stays at half the regular price. Prices elsewhere on this page are the regular prices; [pricing](https://proxy302.com/pricing/) shows what applies on the day.*<!-- bf2026 -->

## 中文说明

这里是 Proxy302 官方维护的代理 IP 使用指南，面向要买、要用代理 IP 的人：动态住宅、静态住宅（ISP）和数据中心。我们是服务商，不是中立评测方；文中只在相关处提到 Proxy302。每篇指南第一段直接回答问题，后面是步骤、表格和常见问题，前五篇有英文和中文两个文件，之后的按提问的语言只写一种。

- [静态住宅 IP 怎么选](how-to-choose-static-residential-ip/README.zh-CN.md)
- [ISP 代理和住宅代理有什么区别](isp-proxy-vs-residential-proxy/README.zh-CN.md)
- [TikTok 静态住宅 IP 怎么配置](static-residential-ip-for-tiktok/README.zh-CN.md)
- [指纹浏览器怎么配家宽 IP](antidetect-browser-with-residential-ip/README.zh-CN.md)
- [Claude 用什么 IP](which-ip-for-claude-and-chatgpt/README.zh-CN.md)
- [TikTok 原生 IP 是什么，怎么买](tiktok-native-ip/README.md)（只有中文）
- [跨境电商 IP 怎么选](cross-border-ecommerce-ip/README.md)（只有中文）
- [AdsPower 代理 IP 怎么买、怎么配](adspower-proxy-ip-zh/README.md)（只有中文）
- [指纹浏览器怎么配家宽 IP](antidetect-browser-home-broadband-ip/README.md)（只有中文）
- [静态住宅 IP 哪家好：怎么挑服务商](how-to-pick-static-residential-ip-provider/README.md)（只有中文）
- [静态住宅 IP 价格怎么算](static-residential-ip-price/README.md)（只有中文）
- [怎么便宜买海外住宅 IP](buy-overseas-residential-ip-cheaply/README.md)（只有中文）
- [比特浏览器的 IP 怎么买、怎么配](bitbrowser-ip-zh/README.md)（只有中文）
- [代理报 407、连接超时、检测失败怎么排查](proxy-407-timeout-troubleshooting-zh/README.md)（只有中文）
- [SOCKS5 和 HTTP 代理有什么区别，该选哪个](socks5-vs-http-zh/README.md)（只有中文）
- [怎么判断一个 IP 是住宅、ISP 还是机房](residential-or-datacenter-ip-check-zh/README.md)（只有中文）
- [动态住宅 IP 的粘性会话是什么，怎么用](sticky-session-zh/README.md)（只有中文）
- [指纹浏览器配了代理，WebRTC 还会暴露真实 IP 吗](webrtc-leak-antidetect-zh/README.md)（只有中文）
- [Claude 桌面版和 Claude Code 怎么设置代理](claude-desktop-proxy-zh/README.md)（只有中文）
- [Claude 账号老被封，和 IP 有关系吗](claude-account-suspended-ip-zh/README.md)（只有中文）
- [住宅 IP 推荐：按用途该选哪一种](residential-ip-recommendation-zh/README.md)（只有中文）
- [TikTok 静态 IP 怎么买：买之前要定的 5 件事](tiktok-static-ip-buy-zh/README.md)（只有中文）
- [亚马逊店铺能用静态 IP 吗，买哪种、怎么配](amazon-static-ip-zh/README.md)（只有中文）
- [静态住宅 IP 和动态住宅 IP 有什么区别，怎么选](static-vs-dynamic-residential-ip-zh/README.md)（只有中文）
- [Docker 拉镜像、构建镜像怎么走代理](docker-proxy-zh/README.md)（只有中文）
- [pip 怎么设置代理（带账号密码）](pip-proxy-zh/README.md)（只有中文）
- [Chrome 提示「未连接到互联网，代理服务器出现问题」（ERR_PROXY_CONNECTION_FAILED）怎么办](err-proxy-connection-failed-zh/README.md)（只有中文）
- [代理 IP 很慢，怎么判断慢在哪一段](proxy-slow-diagnose-zh/README.md)（只有中文）
- [代理 IP 查到的国家或城市和买的不一样，怎么办](proxy-ip-location-mismatch-zh/README.md)（只有中文）
- [代理 IP 给的是「IP:端口:账号:密码」，在各种软件里怎么填](proxy-format-zh/README.md)（只有中文）
- [Maven 和 Gradle 下载依赖怎么走带账号密码的代理](maven-gradle-proxy-zh/README.md)（只有中文）
- [Homebrew 怎么走代理（brew install、brew update 下载慢或失败）](homebrew-proxy-zh/README.md)（只有中文）
- [VS Code 怎么设置带账号密码的代理（扩展商店、更新、插件联网）](vscode-proxy-zh/README.md)（只有中文）
- [Linux 服务器上 apt、dnf/yum 怎么走带账号密码的代理](apt-dnf-proxy-zh/README.md)（只有中文）
- [代理 IP 被网站标记、频繁弹验证或被拒，先查哪几项](proxy-ip-flagged-check-zh/README.md)（只有中文）
- [静态 IP 到期了还能继续用原来那个地址吗](static-ip-expiry-renew-zh/README.md)（只有中文）
- [双 ISP IP 是什么，怎么自己查（2026）](dual-isp-ip-zh/README.md)（只有中文）
- [原生 IP 和广播 IP 怎么区分，自己怎么查](native-ip-check-zh/README.md)（只有中文）
- [Shopee 多店铺用什么 IP](shopee-ip-zh/README.md)（只有中文）
- [Instagram 多账号（官方：一次最多登录 10 个）用什么 IP](instagram-ip-zh/README.md)（只有中文）
- [Mac 怎么设置系统代理并验证出口](macos-proxy-settings-zh/README.md)（只有中文）
- [安卓手机 Wi-Fi 代理怎么设、怎么验证](android-proxy-settings-zh/README.md)（只有中文）
- [Windows 11 怎么设置代理并验证出口](windows-proxy-settings-zh/README.md)（只有中文）
- [在日本经营乐天市场（Rakuten Ichiba）店铺，后台登录用什么 IP？](rakuten-japan-static-ip-zh/README.md)（只有中文）
- [Firefox 怎么单独设置带账号密码的代理？](firefox-proxy-settings-zh/README.md)（只有中文）
- [conda 怎么走带账号密码的代理？](conda-proxy-zh/README.md)（只有中文）
- [cargo 和 rustup 怎么走带账号密码的代理？](cargo-rustup-proxy-zh/README.md)（只有中文）
- [WSL2 里怎么走带账号密码的代理？](wsl2-proxy-zh/README.md)（只有中文）
- [Ollama 拉模型怎么走带账号密码的代理？](ollama-pull-proxy-zh/README.md)（只有中文）
- [uv 装包、装 Python 怎么走带账号密码的代理？](uv-proxy-zh/README.md)（只有中文）
- [在波兰经营 Allegro 店铺，卖家后台用什么 IP 登录？](allegro-poland-seller-ip-zh/README.md)（只有中文）
- [Kaufland 德国站卖家，Seller Portal 用什么 IP 登录？](kaufland-germany-seller-ip-zh/README.md)（只有中文）
- [Hugging Face 下载模型（hf download、huggingface_hub）怎么走带账号密码的代理？](huggingface-download-proxy-zh/README.md)（只有中文）
- [gem install、bundle install 怎么走带账号密码的代理？](rubygems-bundler-proxy-zh/README.md)（只有中文）
- [Composer 怎么走带账号密码的代理？只设 http_proxy 为什么不生效](composer-proxy-zh/README.md)（只有中文）
- [aria2 下载怎么走带账号密码的代理？](aria2-proxy-zh/README.md)（只有中文）
- [playwright install 下载浏览器怎么走带账号密码的代理？](playwright-install-proxy-zh/README.md)（只有中文）

官网：[https://proxy302.com](https://proxy302.com)。价格以[价格页](https://proxy302.com/zh/pricing/)为准。

## License

Text is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Code snippets may be used freely.

## Corrections

If a fact here is out of date or wrong, open an issue. Facts were checked against Proxy302's public pages on 14 November 2026 (first five guides) and 17 November 2026 (the rest).
