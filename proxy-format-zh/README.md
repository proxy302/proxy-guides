# 代理 IP 给的是「IP:端口:账号:密码」，在各种软件里怎么填？

这一串是四个字段用冒号连在一起：主机（IP 或域名）、端口、用户名、密码。有单独输入框的地方（指纹浏览器的代理设置、iPhone 的 Wi‑Fi 代理、系统代理），就拆开分别填。命令行和代码要的是「代理地址」，把它改写成 `http://用户名:密码@主机:端口`，SOCKS5 类型的 IP 写成 `socks5h://用户名:密码@主机:端口`。用户名或密码里有 `@`、`:`、`#`、`/` 这类符号时，写进地址之前要先编码，否则程序会把地址拆错。

*本文由 Proxy302 维护，我们出售这类代理 IP。文中的转换脚本和 curl 命令 2026 年 11 月对着本机带账号密码的测试代理跑过，包括密码里带 `@` 和 `:` 的情况。*

## 四个字段分别是什么

以 `HOST:PORT:USER:PASS` 为例：

| 位置 | 字段 | 填到哪里 |
|---|---|---|
| 第 1 段 | 主机 HOST | 「服务器」「主机」「代理地址」框，只写 IP 或域名，不带 `http://` |
| 第 2 段 | 端口 PORT | 「端口」框，纯数字 |
| 第 3 段 | 用户名 USER | 「用户名」「账号」框 |
| 第 4 段 | 密码 PASS | 「密码」框 |

协议（HTTP 还是 SOCKS5）不在这一串里，看订单页上这个 IP 是哪一种，然后在软件里选同一种。

## 三种常见写法

| 写法 | 样子 | 常见于 |
|---|---|---|
| 冒号四段式 | `HOST:PORT:USER:PASS` | 服务商交付、指纹浏览器批量导入 |
| 地址式（URL） | `http://USER:PASS@HOST:PORT` | curl、Python、Node.js、git、pip、环境变量 |
| 分框填写 | 四个输入框 | 系统代理、iPhone、指纹浏览器的单个环境 |

批量导入时，不同软件要求的字段顺序不一样，有的要 `用户名:密码@主机:端口`。导入前看一眼那个软件说明里给的格式示例，按它的顺序排。

## 各处具体怎么填

| 在哪里用 | 怎么填 |
|---|---|
| 指纹浏览器（AdsPower、比特浏览器等） | 先选和 IP 一致的类型，再分框填四项，或按软件支持的格式整串粘贴；详见 [AdsPower](../adspower-proxy-ip-zh/README.md)、[比特浏览器](../bitbrowser-ip-zh/README.md) |
| iPhone、iPad | 设置 → 无线局域网 → 当前网络旁的 (i) → 配置代理 → 手动：填服务器和端口，打开下方的认证开关（英文界面叫 Authentication）后填用户名和密码。只能用 HTTP 类型的 IP |
| Windows 系统代理 | 只填地址和端口。账号密码在浏览器第一次访问时的弹窗里填 |
| Chrome 启动参数 | `--proxy-server="http://HOST:PORT"`，不能带账号密码，登录框里再填；Chrome 不支持带账号密码的 SOCKS5 |
| curl | `curl -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json`，或者 `curl -x http://HOST:PORT -U "USER:PASS" https://ipinfo.io/json` |
| Python requests | `proxies = {"http": URL, "https": URL}`，`URL` 用地址式 |
| 环境变量 | `export HTTPS_PROXY="http://USER:PASS@HOST:PORT"` |

## 用脚本把四段式转成地址式

手上有一批 `HOST:PORT:USER:PASS`，用下面的 Python 脚本一次转完，特殊字符会自动编码：

```python
import sys
from urllib.parse import quote

def to_url(line: str, scheme: str = "http") -> str:
    # 只按前三个冒号拆，密码里的冒号会原样保留
    host, port, user, password = line.strip().split(":", 3)
    return f"{scheme}://{quote(user, safe='')}:{quote(password, safe='')}@{host}:{port}"

if __name__ == "__main__":
    scheme = sys.argv[1] if len(sys.argv) > 1 else "http"
    for line in sys.stdin:
        if line.strip():
            print(to_url(line, scheme))
```

把 IP 列表存成 `ips.txt`，每行一个，然后：

```bash
python3 convert.py < ips.txt            # 输出 http://...
python3 convert.py socks5h < ips.txt    # SOCKS5 类型的 IP
```

## 特殊字符怎么编码

只有把账号密码写进地址（URL）时才需要编码。填在单独的密码框里，或者用 curl 的 `-U` 参数，照原样填就行。

| 字符 | 编码后 |
|---|---|
| `@` | `%40` |
| `:` | `%3A` |
| `#` | `%23` |
| `/` | `%2F` |
| `?` | `%3F` |
| `%` | `%25` |
| `&` | `%26` |
| 空格 | `%20` |

不编码会怎样：我们测试时密码含 `@` 和 `:`，直接写进地址，curl 报 `Unsupported proxy syntax`，因为它把密码的后半截当成了主机和端口；编码后同一个密码就能正常连上。

## 最常见的几种填错

| 现象 | 多半是 |
|---|---|
| 检测立刻失败 | 主机框里多写了 `http://` 或 `socks5://`，或前后带了空格、换行 |
| 报 407、提示认证失败 | 用户名和密码填反了，或者抄错一位；四段式里第 3 段是用户名、第 4 段是密码 |
| 报端口无效 | 字段顺序排错了，把用户名放进了端口框 |
| 能连上但很快断开、一直检测不通 | 协议类型选反了：HTTP 的 IP 选成了 SOCKS5，或反过来 |
| 以前能用，今天不行 | 静态 IP 到期被收回了，这一串随之失效 |

## 常见问题

**主机给的是域名，不是 IP，能填吗？** 能。主机框里填域名和填 IP 一样。

**四段式能直接贴进 curl 吗？** 不能。curl 和大多数编程语言只认地址式，先用上面的脚本转换。

**同一串能不能同时填在两个软件里？** 技术上能连，但两个软件、两个账号就会用同一个出口。要长期登录的账号，一个账号配一个 IP 更稳妥。

**SOCKS5 地址为什么推荐写 `socks5h://`？** 带 `h` 表示让代理去解析网站域名，不在你本机查，访问记录和出口保持一致。

## Proxy302 在哪里

Proxy302 交付的连接信息就是主机、端口、用户名、密码四项，购买时选 HTTP 或 SOCKS5，一个 IP 只有一种协议。静态 ISP（静态住宅）IP 每个 1 天 0.50 美元、30 天 7.99 美元，所有国家同价。价格见[价格页](https://proxy302.com/zh/pricing/)。

*事实核对于 2026 年 11 月 19 日。*
