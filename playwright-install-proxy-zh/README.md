# playwright install 下载浏览器怎么走带账号密码的代理？

运行前设置 `HTTPS_PROXY=http://USER:PASS@HOST:PORT`（密码里的特殊字符先编码），再执行 `playwright install` 或 `npx playwright install`。只设 `HTTP_PROXY` 没用：我们实测时，只有 `HTTP_PROXY` 的情况下 Playwright 照样直连下载，换成 `HTTPS_PROXY` 才经过代理。Linux 上装系统依赖要用 root 单独跑 `install-deps`，因为提权后 `HTTPS_PROXY` 不会被带过去。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP 1 天 0.5 美元、30 天 7.99 美元，所有国家同价，下单时选 HTTP 协议；只是下载浏览器、不需要固定出口的，动态住宅按流量 1.5 美元/GB 起。

*2026 年 11 月 26 日在 macOS 上用 Playwright for Python 1.60.0 实测：通过本地带账号密码认证的测试代理下载 ffmpeg 组件，并对照了 Playwright 官方文档[「Install behind a firewall or a proxy」](https://playwright.dev/python/docs/browsers)一节。*

## 各平台写法

macOS、Linux（Python 版）：

```bash
HTTPS_PROXY='http://USER:PASS@HOST:PORT' playwright install chromium
```

Node.js 版：

```bash
HTTPS_PROXY='http://USER:PASS@HOST:PORT' npx playwright install chromium
```

Windows PowerShell：

```powershell
$Env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
playwright install chromium
```

Windows 命令提示符（cmd）：

```bat
set HTTPS_PROXY=http://USER:PASS@HOST:PORT
playwright install chromium
```

官方文档的例子写的是 `https://` 开头的代理地址，那是公司代理本身走 TLS 的情况；普通的 HTTP 代理照样写 `http://` 开头，下载的 HTTPS 文件经代理隧道传输。

## 实测记录

用 `PLAYWRIGHT_BROWSERS_PATH` 指到一个临时目录，下载最小的 ffmpeg 组件（约 1 MB），看测试代理的日志：

| 设置 | 结果 |
|---|---|
| 只设 `HTTP_PROXY` | 下载成功，但代理日志里没有记录，说明是直连 |
| 设 `HTTPS_PROXY`（账号密码写在地址里） | 下载成功，代理日志里有 `CONNECT cdn.playwright.dev:443` 和 `CONNECT playwright.download.prss.microsoft.com:443` |
| `HTTPS_PROXY` 里密码写错 | 代理连续返回 407，Playwright 只报 `Failed to install browsers` 和 `Download failure, code=1` |

第三行很关键：密码错时 Playwright 的报错里不会出现 407 字样。看到 `Download failure`，先用下面的 curl 单独测代理。

想先看看要下载哪些文件、从哪个地址下，不真的下载：

```bash
playwright install --dry-run chromium
```

它会列出安装位置和下载地址（含备用地址）。如果公司防火墙要按域名放行，把这些域名一起报给管理员。

## 先确认代理本身可用

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" -o /dev/null -w '%{http_code}\n' https://cdn.playwright.dev/
```

我们测的时候这里返回 `400`，这是下载站自己对根路径的回应，说明请求已经经过代理到达对方，代理是通的。返回 `000` 是连不上代理，返回 `407` 是代理账号密码不对。

## 几个相关的环境变量

| 变量 | 作用 |
|---|---|
| `HTTPS_PROXY` | 下载浏览器时用的代理 |
| `PLAYWRIGHT_BROWSERS_PATH` | 浏览器装到哪里，比如 `$HOME/pw-browsers`；设成 `0` 装进项目的 `node_modules` |
| `PLAYWRIGHT_DOWNLOAD_CONNECTION_TIMEOUT` | 下载的空闲超时，单位毫秒，默认 30 秒；网络慢时调大，比如 `120000` |
| `NODE_EXTRA_CA_CERTS` | 网络里有设备解密 HTTPS、报「self signed certificate in certificate chain」时，指向那台设备的根证书 |

## Linux 上装系统依赖

`playwright install --with-deps` 会顺带用包管理器装系统库，需要 root。官方文档提醒：让 Playwright 自己提权时，`HTTPS_PROXY` 这类变量不会被带过去。分两步做更稳：

```bash
# 1. 系统依赖：用 root 跑，apt 自己的代理另外配
sudo -E playwright install-deps chromium

# 2. 浏览器本体：普通用户跑
HTTPS_PROXY='http://USER:PASS@HOST:PORT' playwright install chromium
```

`sudo -E` 会保留当前环境变量；apt 也可以按 [apt、dnf 走代理](../apt-dnf-proxy-zh/README.md) 写进它自己的配置文件。Node 版把命令换成 `npx playwright install-deps chromium` 即可。

## 报错对照

| 现象 | 原因 | 怎么办 |
|---|---|---|
| 设了代理，下载还是很慢或失败，代理那边没记录 | 只设了 `HTTP_PROXY` | 改设 `HTTPS_PROXY` |
| `Download failure, code=1` | 多半是代理认证失败（407），也可能是连不上代理 | 用 curl 单独测代理 |
| `self signed certificate in certificate chain` | 网络里有设备在解密 HTTPS | 设 `NODE_EXTRA_CA_CERTS` 指向它的根证书 |
| 下载到一半停住、报超时 | 网络慢 | 调大 `PLAYWRIGHT_DOWNLOAD_CONNECTION_TIMEOUT` |
| `--with-deps` 时 apt 连不上 | 提权后代理变量丢了 | 按上面分两步装 |

## 常见问题

**这个代理会不会影响 Playwright 跑测试时浏览器的出口？** 不会。`HTTPS_PROXY` 这里只管下载浏览器。要让浏览器访问网站时走代理，在 `launch()` 或 `new_context()` 里传 `proxy` 参数，见 [Playwright 设置代理](../playwright-proxy/README.md)。

**SOCKS5 的 IP 能用来下载吗？** 我们只测了 HTTP 代理，官方文档也只给了 HTTP(S) 代理的例子。用 Proxy302 的话，下单时协议选 HTTP 最稳。

**下载一次浏览器大概多少流量？** 约等于要装的浏览器包本身的大小。只装用得到的那一个（比如只装 chromium）最省；`--dry-run` 可以先看要下哪些包。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，所有国家同价）和动态住宅流量（1.5 美元/GB 起，预付流量不过期）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。

*Facts checked 26 November 2026.*
