# 代理报 407、连接超时、检测失败怎么排查？

407 的意思是已经连到代理服务器，但认证没通过，几乎都是用户名、密码或协议填错。连接超时说明连代理服务器这一步就没走通，要查地址、端口和本机网络。软件里的「检测失败」是个笼统提示，先用 curl 在命令行复现，看到具体报错再对症处理；排查顺序是本机网络、代理本身、最后才是目标网站。

*本文由 Proxy302 维护，我们出售文中提到的代理。*

## 先分清卡在哪一步

| 报错 | 卡在哪一步 | 常见原因 |
|---|---|---|
| 407 Proxy Authentication Required | 连上了代理，认证没过 | 账密错或漏填；密码里的特殊符号没编码；用了另一个 IP 的账密 |
| 连接超时（timed out） | 没连上代理 | 主机或端口写错；公司、学校网络拦了端口；IP 已到期 |
| Connection refused | 到了那台机器，端口不接受连接 | 端口号错 |
| 软件里「检测失败」 | 说不准 | 用下面的命令复现 |

## 排查步骤

**1. 确认本机能正常上网。** 不挂代理直接打开 `https://ipinfo.io/json`。打得开，说明出网没问题。电脑上如果还开着系统代理或其他全局网络工具，先关掉，避免两层代理叠在一起，把问题搅混。

**2. 用 curl 单独测代理。** HTTP 类型的 IP：

```bash
curl -v --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

SOCKS5 类型的 IP（`socks5h` 让域名也交给代理解析）：

```bash
curl -v --max-time 20 -x "socks5h://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

`-v` 会把握手的每一步打印出来。返回一段 JSON、里面的 `ip` 和订单上的一致，就说明代理本身没问题，毛病在软件的配置里。

**3. 超时就单测端口。** curl 一直卡着时，单独看 TCP 能不能连上：

```bash
nc -vz -w 10 HOST PORT
```

Windows 在 PowerShell 里用：

```powershell
Test-NetConnection HOST -Port PORT
```

端口不通，账密再对也没用。换一个网络（比如手机热点）再试，就能分辨是不是当前网络限制了这个端口。

**4. 密码里有特殊符号。** 账密拼进 URL 时，`@`、`:`、`#`、`%`、`/` 会让地址被拆错，结果看起来像 407 或「地址解析失败」。先编码再拼：

```python
from urllib.parse import quote

user, pwd = "USER", "PASS"
proxy = f"http://{quote(user, safe='')}:{quote(pwd, safe='')}@HOST:PORT"
print(proxy)
```

指纹浏览器这类有单独账密输入框的软件不用编码，原样填即可。

**5. 回到软件里看配置。** curl 通、软件不通，多半是这几样：代理类型选反（HTTP 的 IP 选成了 SOCKS5，或反过来）、主机框多了 `http://` 前缀、修改后没保存。写自动化脚本时还要记住：Chrome、Edge 等 Chromium 内核的浏览器不支持带账密的 SOCKS5，用 Playwright、Puppeteer 或 Selenium 驱动它们时，带账密的 IP 请选 HTTP 类型。

**6. 最后才看目标网站。** 代理通了、网站返回 403 或弹出验证，这是网站自己的判断，不是代理连接故障。需要验证就照网站的要求正常完成。

## 报错原文对照

| 看到的报错 | 含义和处理 |
|---|---|
| curl：`CONNECT tunnel failed, response 407` | HTTP 代理认证失败。重新复制这个 IP 自己的账密 |
| curl：`User was rejected by the SOCKS5 server` | SOCKS5 认证失败，同样先查账密 |
| curl：`Connection timed out` 或 `Timeout was reached` | 到代理的 TCP 没建立。查主机、端口，换网络试 |
| curl：`Connection refused` | 端口不对，重新复制 |
| curl：`Could not resolve proxy` | 主机名拼错，或本机 DNS 出了问题 |
| Python：`Tunnel connection failed: 407 Proxy Authentication Required` | 和 curl 的 407 一样，查账密和编码 |
| Python：`ConnectTimeout` | 连不上代理，按第 3 步查端口 |
| Python：`Missing dependencies for SOCKS support` | 没装 SOCKS 支持，执行 `pip install "requests[socks]"` |

## 常见问题

**昨天还能用，今天突然 407 或超时？** 先看到期日。静态 IP 的租期结束后会被收回，原来的账密跟着失效；动态住宅则看流量余额是不是用完了。

**换个网络就好了，说明什么？** 说明原来的网络拦了代理端口或限制了出站连接，和代理本身无关。

**检测显示的城市和我买的不一样，算失败吗？** 不算。各家 IP 库对城市的判断经常有出入，以国家为准。

**都查过了还是不行？** 把 `curl -v` 的完整输出发给服务商客服，发之前把密码打码。

## Proxy302 在哪里

Proxy302 出售静态 ISP IP（HTTP 或 SOCKS5，购买时选定，1 天 US$0.50 起，30 天 US$7.99）、动态住宅（US$1.50/GB 起）和机房 IP。买之前想先确认自己的网络能不能用代理，最省钱的办法是买一个 1 天的静态 IP，按上面的命令测一遍。价格见 [proxy302.com/pricing](https://proxy302.com/pricing/)，中文站 [proxy302.com/zh](https://proxy302.com/zh/)。

*事实核对于 2026 年 11 月 17 日。*
