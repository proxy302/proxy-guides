# aria2 下载怎么走带账号密码的代理？

用 `--all-proxy`：`aria2c --all-proxy='http://HOST:PORT' --all-proxy-user='USER' --all-proxy-passwd='PASS' URL`，HTTP、HTTPS、FTP 下载都会经过这个代理。账号密码也可以直接写进地址（`http://USER:PASS@HOST:PORT`），但特殊字符要编码；分开写的好处是不用编码。aria2 的代理只支持 HTTP 代理，没有 SOCKS5 选项。长期用就把这几行写进 `aria2.conf`。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP 1 天 0.5 美元、30 天 7.99 美元，所有国家同价，下单时选 HTTP 协议；只是下载大文件、不在乎出口是否固定的，动态住宅 1.5 美元/GB 起。

*依据：aria2 1.37.0 [官方手册](https://aria2.github.io/manual/en/html/aria2c.html)的代理选项和 ENVIRONMENT 一节，2026 年 11 月 26 日核对；本机没装 aria2，没有实机运行。*

## 命令行

```bash
# 账号密码分开写（推荐，不用编码）
aria2c --all-proxy='http://HOST:PORT' \
       --all-proxy-user='USER' \
       --all-proxy-passwd='PASS' \
       'https://example.com/file.iso'

# 账号密码写进地址（特殊字符要编码，比如 @ 写成 %40）
aria2c --all-proxy='http://USER:PASS@HOST:PORT' 'https://example.com/file.iso'
```

## 选项一览

| 选项 | 作用 |
|---|---|
| `--all-proxy=PROXY` | 所有协议都用这个代理 |
| `--all-proxy-user`、`--all-proxy-passwd` | 上面那个代理的用户名、密码 |
| `--http-proxy`、`--https-proxy`、`--ftp-proxy` | 只给某一种下载地址用的代理，会覆盖 `--all-proxy` |
| `--http-proxy-user` / `--http-proxy-passwd` 等 | 对应协议代理的账号密码 |
| `--no-proxy=LIST` | 逗号分隔的不走代理的主机、域名、网段 |
| `--proxy-method=get\|tunnel` | 代理请求的方式，默认 `get`；HTTPS 下载不管怎么设都用 `tunnel` |

PROXY 的格式是 `[http://][USER:PASSWORD@]HOST[:PORT]`。手册特别说明：地址里就算写成 `https://` 或 `ftp://`，aria2 也一律按 `http://` 处理。也就是说，填的必须是一个 HTTP 代理。

## 写进配置文件

aria2 默认先找 `~/.aria2/aria2.conf`，没有就用 `$XDG_CONFIG_HOME/aria2/aria2.conf`（一般是 `~/.config/aria2/aria2.conf`）。配置文件里去掉选项前面的两个横线：

```ini
all-proxy=http://HOST:PORT
all-proxy-user=USER
all-proxy-passwd=PASS
no-proxy=localhost,127.0.0.1,192.168.0.0/16
```

文件里有密码，记得 `chmod 600 ~/.aria2/aria2.conf`。临时不想走代理时，命令行加 `--all-proxy=''` 覆盖掉。

## 用环境变量

aria2 也认 `http_proxy`、`https_proxy`、`ftp_proxy`、`all_proxy`、`no_proxy`，格式同上，可以带账号密码：

```bash
export all_proxy='http://USER:PASS@HOST:PORT'
aria2c 'https://example.com/file.iso'
```

优先级：命令行选项 > 环境变量 > 配置文件。手册的原意是环境变量会覆盖配置文件里的同名设置，命令行又会覆盖环境变量。排查「为什么没走我设的代理」时，先看有没有别的地方设了更高优先级的值。

账号密码分开写和写进地址同时存在时，**后出现的覆盖先出现的**。比如配置文件里写了 `all-proxy-user`，命令行里又给了带账号密码的地址，以命令行这个为准。

## 几个容易踩的坑

- **`--no-proxy` 里写网段，只管直接写 IP 的下载地址。** 手册说明当前实现不会把下载地址里的主机名解析成 IP 再去和网段比较，所以 `192.168.0.0/16` 这类写法只对 `http://192.168.1.10/...` 这样的地址生效；要排除内网域名，就把域名本身写进去。
- **多线程下载。** `-x 16 -s 16` 这类分段下载会同时开很多个连接，全部经过同一个代理；用静态 IP 时，这些连接的出口都是同一个地址。
- **HTTP（不是 HTTPS）地址下载失败。** 默认的 `get` 方式会把完整网址交给代理去取。个别代理只允许隧道方式，这时加 `--proxy-method=tunnel` 再试。
- **SOCKS5 的 IP 用不了。** aria2 没有 SOCKS 选项。用 Proxy302 的话，下单时协议选 HTTP；手上只有 SOCKS5 的，要在本机加一层转发，把它变成 HTTP 代理。

## 先确认代理本身可用

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

能看到代理的出口 IP 和国家，再去查 aria2 的设置。

## 报错对照

| 现象或报错 | 多半是 | 怎么办 |
|---|---|---|
| 提示 407、认证失败 | 用户名、密码错，或者写进地址时没编码 | 改用 `--all-proxy-user`、`--all-proxy-passwd` 分开写 |
| 连接被拒绝、超时 | 代理主机或端口写错，或者填的是 SOCKS5 端口 | 用上面的 curl 测；确认买的是 HTTP 协议 |
| 好像没走代理 | 优先级更高的地方有别的设置，或者被 `--no-proxy` 排除了 | 按「命令行 > 环境变量 > 配置文件」逐个查 |
| HTTPS 能下，HTTP 地址下不了 | 代理不接受 `get` 方式 | 加 `--proxy-method=tunnel` |
| 前一阵能用，今天不行 | 静态 IP 到期被释放 | 到期前续费，或按 IP 打开自动续费 |

## 常见问题

**aria2 的 RPC（比如配合浏览器扩展或图形界面）也能用这些设置吗？** 能。RPC 方式启动的 aria2 同样读配置文件；也可以在添加任务时，通过任务选项传 `all-proxy` 等参数。

**下载用动态 IP 还是静态 IP？** 下载公开文件，按流量计费的动态住宅更划算；预付流量不过期，100GB 包每 GB 1.35 美元。下载源要按来源 IP 放行的，用静态 IP。

**会不会比直连慢？** 多了一跳，取决于代理所在国家离你和下载源多远。先用 1 天的 IP 试速度。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，一个 IP 一种协议，所有国家同价）和动态住宅流量。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。相关：[代理很慢，怎么判断慢在哪一段](../proxy-slow-diagnose-zh/README.md)、[代理格式 IP:端口:账号:密码 怎么填](../proxy-format-zh/README.md)。

*Facts checked 26 November 2026.*
