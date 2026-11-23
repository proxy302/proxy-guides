# conda 怎么走带账号密码的代理？

运行两条命令：`conda config --set proxy_servers.http http://USER:PASS@HOST:PORT` 和 `conda config --set proxy_servers.https http://USER:PASS@HOST:PORT`。它们写进用户目录下的 `.condarc`，以后 `conda install`、`conda create`、`conda update` 都会走这个代理。注意两行的值都以 `http://` 开头：键名 `http`、`https` 指要访问的网址类型，值里的 `http://` 指代理本身的协议。不想落到配置文件里，也可以只设环境变量 `HTTPS_PROXY`、`HTTP_PROXY`，conda 在没配 `proxy_servers` 时默认读它们。本指南由 Proxy302 维护，我们也销售这些代理；给 conda 用，买 HTTP 类型的静态 ISP 最省事（1 天 0.5 美元，30 天 7.99 美元）。

*本页按 conda 官方配置文档（2026 年 11 月版）核对，测试机上没有装 conda，命令没有实机运行。*

## 方法一：写进 .condarc（长期用）

```bash
conda config --set proxy_servers.http  http://USER:PASS@HOST:PORT
conda config --set proxy_servers.https http://USER:PASS@HOST:PORT
conda config --show proxy_servers
```

写进去的内容是这样的，也可以直接编辑 `~/.condarc`（Windows 在 `C:\Users\你的用户名\.condarc`）：

```yaml
proxy_servers:
  http: http://USER:PASS@HOST:PORT
  https: http://USER:PASS@HOST:PORT
```

USER、PASS、HOST、PORT 是占位符。值的开头要和代理本身的协议一致：HTTP 代理就写 `http://`，即使它负责的是 https 网址。

**不想把密码写进文件：** 值里只写 `http://HOST:PORT`。官方文档说，没写账号密码或认证失败时，conda 会提示你输入用户名和密码。

## 方法二：环境变量（临时用、脚本里用）

macOS、Linux：

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
export HTTP_PROXY="$HTTPS_PROXY"
conda install numpy
```

Windows PowerShell：

```powershell
$env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
$env:HTTP_PROXY  = $env:HTTPS_PROXY
conda install numpy
```

只对当前终端窗口有效，关掉就没了。`.condarc` 里一旦配了 `proxy_servers`，就以它为准，不再看环境变量。

## 方法三：只给某一个地址走代理

`proxy_servers` 的键也可以写成「协议://主机名」，只对访问这个主机的请求生效：

```yaml
proxy_servers:
  'https://conda.anaconda.org': http://USER:PASS@HOST:PORT
```

没写进来的地址按默认方式连接。公司内部的镜像站直连、外部的软件源走代理时，用这个写法。

## 三种方法怎么选

| 情况 | 用哪种 |
|---|---|
| 这台电脑长期要走代理 | 方法一，写进 `.condarc` |
| 只是这一次装包，或者在 CI 脚本里 | 方法二，环境变量 |
| 只有个别软件源要走代理 | 方法三，按主机名写 |

## 密码里有特殊字符

URL 里 `@`、`:`、`#`、`/` 有特殊含义，密码里有这些字符要先做百分号编码：`@` 写成 `%40`，`:` 写成 `%3A`，`#` 写成 `%23`，`/` 写成 `%2F`。不编码的话，conda 会把密码的一部分当成主机名。

## SOCKS5 类型的 IP

conda 联网用的是 Python 的 requests 库，requests 要装了 PySocks 才认 `socks5://`、`socks5h://`，而装 PySocks 本身又要联网。省事的做法是给 conda 用 HTTP 类型的 IP。Proxy302 的静态 IP 是下单时选协议，一个 IP 只有一种协议。

## 先确认代理本身能用

```bash
curl -sS -I --max-time 20 -x "http://USER:PASS@HOST:PORT" https://conda.anaconda.org/conda-forge/noarch/repodata.json -o /dev/null -w "%{http_code}\n"
```

`-I` 只取响应头，不下载整个索引文件。返回 `200`，说明账号密码和网络都没问题，再去查 conda 的配置。

## 常见报错

| 报错 | 多半是 | 怎么办 |
|---|---|---|
| `ProxyError: Conda cannot proceed due to an error in your proxy configuration` | 代理地址、端口或格式不对，或环境变量里还留着别的 `*_PROXY` | 用 `conda config --show proxy_servers` 和 `env | grep -i proxy` 对一遍 |
| 提示输入代理用户名密码，输了还是不行 | 账号密码错 | 从订单页重新复制 |
| `CondaHTTPError: HTTP 000 CONNECTION FAILED` | 连不上代理或软件源 | 先用上面的 curl 命令测代理 |
| 407 | 账号密码错，或特殊字符没编码 | 按上一节编码 |
| SSL 证书校验失败 | 多半是公司网络在解密流量；普通的 HTTP 代理转发 https 时不会动证书 | 让公司 IT 提供证书，配 `ssl_verify` 指向它，不要直接关掉校验 |

## 常见问题

**怎么取消代理？** `conda config --remove-key proxy_servers`，再把环境变量清掉。

**conda 环境里的 pip 会跟着走代理吗？** `.condarc` 只管 conda 自己。pip 要单独设置，见 [pip 怎么设置代理](../pip-proxy-zh/README.md)。环境变量那种方式两者都会读。

**下载慢，换代理会快吗？** 不一定，多了一跳。先用上面的 curl 命令分别测直连和走代理的耗时；慢在哪一段，见[代理 IP 很慢怎么查](../proxy-slow-diagnose-zh/README.md)。

**用动态还是静态？** 只是下载软件包，按流量计费的动态住宅也能用（1.5 美元/GB 起）。私有软件源按来源 IP 放行时，用静态 IP，租期内地址不变。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，1 天 0.5 美元、30 天 7.99 美元，每个国家同价）和动态住宅流量。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。

*Facts checked 23 November 2026.*
