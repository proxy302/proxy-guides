# uv 装包、装 Python 怎么走带账号密码的代理？

最简单的是设环境变量：`export HTTPS_PROXY="http://USER:PASS@HOST:PORT"`（再把 `HTTP_PROXY` 设成同一个值），之后 `uv pip install`、`uv sync`、`uv add`、`uv python install` 都会走这个代理。uv 官方文档列出的代理变量是 `HTTP_PROXY`、`HTTPS_PROXY`、`ALL_PROXY` 和 `NO_PROXY`。较新的 uv 还能把代理写进用户级配置文件 `uv.toml`（键名 `http-proxy`、`https-proxy`、`no-proxy`），代理地址认 `http://`、`https://`、`socks5://`、`socks5h://` 四种开头。本指南由 Proxy302 维护，我们也销售这些代理；静态 ISP 1 天 0.5 美元、30 天 7.99 美元，下单时选 HTTP 或 SOCKS5。

*本页按 uv 官方文档的环境变量和设置两份参考（2026 年 11 月）以及 uv 源码里的代理地址校验核对；测试机上没有装 uv，命令没有实机运行。*

## 方法一：环境变量

macOS、Linux：

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
export HTTP_PROXY="$HTTPS_PROXY"
export NO_PROXY="localhost,127.0.0.1"
uv pip install requests
```

Windows PowerShell：

```powershell
$env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
$env:HTTP_PROXY  = $env:HTTPS_PROXY
uv pip install requests
```

USER、PASS、HOST、PORT 是占位符。只对当前终端有效；想一直生效，写进 `~/.bashrc`、`~/.zshrc` 或 PowerShell 的 `$PROFILE`。

## 方法二：写进 uv.toml

```toml
# macOS、Linux：~/.config/uv/uv.toml
# Windows：%APPDATA%\uv\uv.toml
http-proxy  = "http://USER:PASS@HOST:PORT"
https-proxy = "http://USER:PASS@HOST:PORT"
no-proxy    = ["localhost", "127.0.0.1"]
```

- `https-proxy` 的意思是「访问 https 网址时用的代理」，代理本身是 HTTP 类型就写 `http://` 开头。
- 写在用户级配置里，不要写进项目的 `pyproject.toml` 或仓库里的 `uv.toml`，密码会跟着代码被提交。
- 这几个键是较新版本才有的。`uv --version` 太旧、读配置时报不认识这个字段，就升级 uv 或改用环境变量。

## SOCKS5 类型的 IP

uv 的代理地址校验接受 `socks5://` 和 `socks5h://`。`socks5h://` 由代理解析域名：

```bash
export ALL_PROXY="socks5h://USER:PASS@HOST:PORT"
uv pip install requests
```

`ALL_PROXY` 是给所有请求用的通用代理。Proxy302 的静态 IP 下单时选协议，一个 IP 只有一种；HTTP 和 SOCKS5 拿不准选哪个，选 HTTP，各种工具都认。

## 一次验证两件事

```bash
uv run --with requests python -c "import requests; print(requests.get('https://ipinfo.io/json', timeout=20).json()['country'])"
```

这条命令先让 uv 通过代理下载 requests，再用 requests 通过同一个 `HTTPS_PROXY` 去查出口国家。能打印出你买的国家代码，说明 uv 和代理都没问题。

## 密码里有特殊字符

`@`、`:`、`#`、`/` 要先做百分号编码：`@` 写成 `%40`，`:` 写成 `%3A`，`#` 写成 `%23`，`/` 写成 `%2F`。可以用 Python 算：

```bash
python3 -c "import urllib.parse; print(urllib.parse.quote('p@ss:w/rd', safe=''))"
# p%40ss%3Aw%2Frd
```

## 其他相关设置

| 设置 | 作用 |
|---|---|
| `UV_HTTP_TIMEOUT` | 读超时秒数，默认 30；代理较慢时可以调大 |
| `NO_PROXY` / `no-proxy` | 公司内部的包索引直连 |
| `--system-certs` / `UV_SYSTEM_CERTS` | 使用系统证书库（旧名 `--native-tls`、`UV_NATIVE_TLS`）；公司网络解密流量时用 |
| `SSL_CERT_FILE` | 指定 CA 证书文件 |

普通的 HTTP 代理转发 https 请求时走的是 CONNECT 隧道，不改证书，用 Proxy302 的代理不需要动证书设置。

## 常见报错

| 现象 | 多半是 | 怎么办 |
|---|---|---|
| 报错里有 407 | 账号密码错，或特殊字符没编码 | 重新复制，按上一节编码 |
| `invalid proxy URL scheme ...: expected http, https, socks5, or socks5h` | 地址开头写错了，比如 `socks://` | 改成这四种之一 |
| 连接超时 | HOST、PORT 错，或公司防火墙不放行 | 先用 `curl -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json` 测代理 |
| 内部包索引也被送去了代理 | 没写进 `NO_PROXY` | 把内部索引的域名加进去 |
| 证书校验失败 | 公司网络在解密流量 | 用 `--system-certs` 或 `SSL_CERT_FILE` |

## 常见问题

**uv 和 pip 的代理设置通用吗？** 环境变量通用。pip 自己的 `--proxy` 参数和 `pip.conf` 只管 pip，见 [pip 怎么设置代理](../pip-proxy-zh/README.md)。

**`uv python install` 下载 Python 也走代理吗？** 走，用的是同一套网络设置。

**动态还是静态？** 只是装包，按流量计费的动态住宅也行（1.5 美元/GB 起）。私有索引按来源 IP 放行时，用静态 ISP，租期内地址不变。

**有免费试用吗？** 没有。最便宜的测试是一个静态 ISP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，1 天 0.5 美元、30 天 7.99 美元，每个国家同价）和动态住宅流量（1.5 美元/GB 起）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。

*Facts checked 23 November 2026.*
