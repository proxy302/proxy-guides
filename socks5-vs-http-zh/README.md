# SOCKS5 和 HTTP 代理有什么区别，该选哪个？

HTTP 代理看得懂 HTTP 请求，访问 HTTPS 网站时用 CONNECT 方法替你打通一条隧道；SOCKS5 不关心上层跑的是什么协议，只负责把 TCP 连接转过去，所以能转的程序更多。单就打开网页来说，两者效果差不多，该选哪个主要看你用的软件支持什么：Chromium 内核浏览器的自动化脚本、Claude Code、iPhone 的 Wi‑Fi 代理要选 HTTP，指纹浏览器和 curl 两种都行。

*本文由 Proxy302 维护，我们出售 HTTP 和 SOCKS5 两种代理 IP。*

## 两者到底差在哪

| | HTTP 代理 | SOCKS5 代理 |
|---|---|---|
| 工作在哪一层 | 应用层，按 HTTP 协议说话 | 会话层，只转连接，不解读内容 |
| 能转的流量 | 普通 HTTP，加上经 CONNECT 隧道的 HTTPS | 任意 TCP；UDP 是否可用以服务商说明为准 |
| 域名在哪解析 | 代理那一端 | 由客户端决定：`socks5h://` 在代理端，`socks5://` 在本机 |
| 账密认证 | HTTP Basic 认证 | 用户名/密码子协商（RFC 1929） |
| 本身加不加密 | 不加密；HTTPS 网站的内容由 TLS 保护 | 同样不加密，同样靠 TLS |
| Chromium 内核浏览器 | 支持带账密 | 不支持带账密的 SOCKS5 |
| Python `requests` | 开箱即用 | 要先装 `requests[socks]` |

两点常被误会：第一，访问 HTTPS 网站时，HTTP 代理只负责建隧道，看不到也改不了隧道里的内容，这一点和 SOCKS5 没有区别；第二，名字里的「HTTPS 代理」通常指客户端到代理这一段也用 TLS 加密的代理，和「用代理访问 HTTPS 网站」不是一回事。

## 按你的软件来选

| 你要接代理的软件 | 建议 |
|---|---|
| 比特浏览器、AdsPower 等指纹浏览器 | 两种都行，窗口里的类型和 IP 协议一致即可 |
| Playwright、Puppeteer、Selenium 驱动 Chrome | 带账密时选 HTTP |
| Claude Code | HTTP（官方文档写明不支持 SOCKS 代理） |
| iPhone、iPad 的 Wi‑Fi 手动代理 | HTTP |
| curl、Python `requests` | 两种都行 |
| Telegram | SOCKS5 |
| 需要转发非网页 TCP 连接的程序 | SOCKS5 |

在 Proxy302，协议是购买时选定的，一个 IP 只有一种。所以先想好这个 IP 要接进哪个软件，再下单。

## 代码里分别怎么写

**curl：**

```bash
# HTTP 类型
curl -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/ip
# SOCKS5 类型，域名交给代理解析
curl --socks5-hostname "HOST:PORT" --proxy-user "USER:PASS" https://ipinfo.io/ip
```

**Python `requests`**（两行按你那个 IP 的协议二选一）：

```python
import requests

PROXY = "http://USER:PASS@HOST:PORT"         # HTTP 类型的 IP
# PROXY = "socks5h://USER:PASS@HOST:PORT"    # SOCKS5 类型，需 pip install "requests[socks]"

r = requests.get("https://ipinfo.io/json", proxies={"http": PROXY, "https": PROXY}, timeout=20)
print(r.json().get("ip"))
```

**Playwright（Node.js）**，驱动 Chromium，用 HTTP 类型：

```js
const { chromium } = require('playwright');

(async () => {
  const browser = await chromium.launch({
    proxy: { server: 'http://HOST:PORT', username: 'USER', password: 'PASS' },
  });
  const page = await browser.newPage();
  await page.goto('https://ipinfo.io/json');
  console.log(await page.textContent('body'));
  await browser.close();
})();
```

## 选错类型时的表现

| 现象 | 原因和处理 |
|---|---|
| 指纹浏览器一检测就失败 | 窗口里的类型和 IP 协议不一致，改成一致 |
| 自动化脚本里 Chrome 连不上带账密的代理 | 用了带账密的 SOCKS5，换 HTTP 类型的 IP |
| Python 报 `Missing dependencies for SOCKS support` | 没装 `requests[socks]` |
| 出口 IP 对了，DNS 检测却显示本地运营商的 DNS | 用了 `socks5://`，改成 `socks5h://` |
| Claude Code 配了 `socks5://` 地址后连不上 | 它不支持 SOCKS，换 HTTP 类型的 IP |

## 常见问题

**SOCKS5 比 HTTP 快吗？** 协议本身的开销差别很小，快慢主要看线路和目标网站，不必为了速度专门选 SOCKS5。

**SOCKS5 更匿名吗？** 访问 HTTPS 网站时两者暴露的信息基本一样：代理看得到你连的是哪个域名和端口，看不到页面内容。

**买了 HTTP 的 IP，能直接改成 SOCKS5 吗？** 在 Proxy302 不能，一个 IP 只有购买时选的那一种协议；需要另一种，就按那种协议另买一个。

**两种都支持的软件，选哪个？** 随意。如果要在多台设备、多个软件间换着用，HTTP 的兼容面更广。

## Proxy302 在哪里

Proxy302 的静态 ISP IP 下单时可选 HTTP 或 SOCKS5，每个国家同价：1 天 US$0.50，30 天 US$7.99，90 天 US$23.97，可按 IP 开自动续费。没有免费试用，拿不准选哪种协议时，可以各买一个 1 天的 IP，在自己的软件里实际测一遍。价格见 [proxy302.com/pricing](https://proxy302.com/pricing/)，中文站 [proxy302.com/zh](https://proxy302.com/zh/)。

*事实核对于 2026 年 11 月 17 日。*
