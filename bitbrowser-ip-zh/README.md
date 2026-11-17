# 比特浏览器的 IP 怎么买、怎么配？

比特浏览器里每个窗口配一个自己的 IP：要长期登录账号的窗口用静态 ISP（静态住宅）IP，买的时候定好 HTTP 或 SOCKS5，在窗口的代理设置里选同一种类型，填主机、端口、账号、密码，检测通过再打开。窗口多的时候，把 IP 整理成 `host:port:user:pass` 一行一个，再走批量导入。检测失败时，先看类型有没有对上、账密有没有填错。

*本文由 Proxy302 维护，我们出售文中提到的代理 IP。*

## 先想清楚每个窗口要什么 IP

| 窗口用途 | 选什么 | Proxy302 价格 |
|---|---|---|
| 长期登录一个店铺或社媒账号 | 静态 ISP IP，一窗一个 | 30 天 US$7.99 |
| 新号先跑几天看看 | 静态 ISP IP 短租 | 1 天 US$0.50，3 天 US$1.50，7 天 US$3.50 |
| 亚马逊卖家后台 | Amazon ISP | 每 30 天 12 PTC（1 PTC = 1 美元），首个 IP 半价 |
| 不登录，只看公开页面 | 动态住宅，按流量 | US$1.50/GB 起 |
| 目标网站不在意地址类型 | 静态机房 IP | 1 天 US$0.25，30 天 US$3.99 |

静态 ISP 在各个国家同一个价。没有免费试用，想先试，买一个 1 天的 IP 就行。

## 给一个窗口配 IP

比特浏览器不同版本的菜单叫法略有出入，下面按通用流程写，按钮名称以你电脑上的界面为准。

1. **下单。** 在 Proxy302 选国家、租期和协议。协议在购买时就定了，一个 IP 只对应 HTTP 或 SOCKS5 其中一种。
2. **抄下四项。** 在订单的 IP 详情里找到主机、端口、用户名、密码。
3. **建窗口。** 新建窗口或编辑已有窗口，代理方式选自定义，代理类型选和这个 IP 一样的协议。
4. **填写。** 四项分别填进对应的框。主机框只写地址，不要带 `http://`、`socks5://` 这类前缀，前后也别留空格。有的版本支持把整串 `host:port:user:pass` 粘进主机框自动拆开，能用就用，省得逐格复制。
5. **检测。** 点代理检测，出口 IP 和国家与订单一致再保存。
6. **进窗口再确认。** 打开窗口，先访问一个查 IP 的网页看一眼，然后才登录账号。

## 批量导入：格式和顺序

几十个窗口一个个粘，最容易出的事是把甲窗口的 IP 填进乙窗口。比较稳的做法是先在表格里排好，一行对应一个窗口：

```
HOST:PORT:USER:PASS
HOST:PORT:USER:PASS
```

旁边多留一列写窗口名或账号备注，导入后能逐个对照。另外注意：

- 四段顺序固定为主机、端口、用户名、密码，用英文冒号分隔。
- 导入模板里如果有代理类型一列，每一行都要和那个 IP 的协议一致；HTTP 和 SOCKS5 混着买时尤其要逐行看。
- 导入完别急着全部打开，先每个窗口单独检测一次。

导入前想在电脑上先把整张清单测一遍，可以用下面这段 Python。把清单存成 `ips.txt`，一行一个；SOCKS5 的 IP 需要先 `pip install "requests[socks]"`，并把 `SCHEME` 改成 `socks5h`。

```python
from urllib.parse import quote
import requests

SCHEME = "http"  # SOCKS5 类型的 IP 改成 "socks5h"

with open("ips.txt", encoding="utf-8") as f:
    for line in f:
        line = line.strip()
        if not line:
            continue
        host, port, user, pwd = line.split(":", 3)
        proxy = f"{SCHEME}://{quote(user, safe='')}:{quote(pwd, safe='')}@{host}:{port}"
        try:
            r = requests.get("https://ipinfo.io/json",
                             proxies={"http": proxy, "https": proxy}, timeout=20)
            info = r.json()
            print(host, "OK", info.get("ip"), info.get("country"))
        except requests.RequestException as e:
            print(host, "FAIL", type(e).__name__)
```

`quote` 会把密码里的 `@`、`#` 等符号编码，避免拼出错误的地址。打印 FAIL 的那几行，先单独查清楚再导入。

## 检测失败怎么排查

| 现象 | 先查什么 |
|---|---|
| 一点检测就失败 | 代理类型和 IP 的协议不一致；或主机框多了前缀、空格 |
| 提示 407 或认证失败 | 用户名或密码错，或者填成了另一个 IP 的账密 |
| 检测转很久后超时 | 本机网络或公司防火墙拦了这个端口；用上面的脚本在电脑上单独测 |
| 检测通过，窗口里看到的却是本机 IP | 窗口设置没保存上，或电脑上还开着其他全局网络工具 |
| 以前好好的，今天检测不过 | IP 租期到了，已被收回；记得到期前续费 |
| 国家对，城市对不上 | 各家 IP 库对城市的判断常不一致，以国家为准 |

更细的报错对照见[代理报 407、超时怎么排查](../proxy-407-timeout-troubleshooting-zh/README.md)。

## 常见问题

**一个 IP 能填进两个窗口吗？** 能连上，但两个窗口里的账号就从同一个地址登录了，分窗口的意义没了。一个窗口一个 IP。

**选 HTTP 还是 SOCKS5？** 比特浏览器两种都支持，选哪种都行，关键是窗口里的类型要和 IP 一致。两者的区别见[SOCKS5 和 HTTP 代理有什么区别](../socks5-vs-http-zh/README.md)。

**到期后还能用同一个 IP 吗？** 到期前续费，就还是这个 IP，也可以按 IP 开自动续费。到期没续，IP 会被释放，再买拿到的是新地址。

**平台要求账号验证怎么办？** 照平台的要求如实完成。代理的作用只是让这个账号每次都从同一个地址连上来，替代不了验证。

## Proxy302 在哪里

Proxy302 出售静态 ISP IP（1 到 90 天，HTTP 或 SOCKS5，可按 IP 开自动续费）、Amazon ISP、动态住宅和机房 IP。30 天及以上的静态 IP 为买家独享。比特浏览器的配置也写在我们的[比特浏览器集成页](https://proxy302.com/zh/integrations/bitbrowser/)，价格见[价格页](https://proxy302.com/pricing/)，中文站是 [proxy302.com/zh](https://proxy302.com/zh/)。

*事实核对于 2026 年 11 月 17 日。*
