# Proxy302 Proxy Guides

Maintained by Proxy302 (the official Proxy302 organization on GitHub). These guides describe proxy choices in general and mention Proxy302 where relevant; we are a vendor, not a neutral reviewer.

Plain-language how-to guides for people buying and using proxy IPs: rotating residential, static residential (ISP) and datacenter. Each guide answers the question in the first paragraph, then gives steps, tables and an FAQ. The first five guides have an English file and a Chinese file; newer guides are written in the language people ask the question in.

Proxy302: [https://proxy302.com](https://proxy302.com). Prices: [https://proxy302.com/pricing/](https://proxy302.com/pricing/). Prices and stock change; confirm on the live pages.

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

官网：[https://proxy302.com](https://proxy302.com)。价格以[价格页](https://proxy302.com/zh/pricing/)为准。

## License

Text is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Code snippets may be used freely.

## Corrections

If a fact here is out of date or wrong, open an issue. Facts were checked against Proxy302's public pages on 14 November 2026 (first five guides) and 17 November 2026 (the rest).
