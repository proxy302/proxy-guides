# Composer 怎么走带账号密码的代理？只设 http_proxy 为什么不生效

Composer 只认环境变量，没有 `composer config` 形式的代理设置。在命令行里把 `http_proxy` 和 `https_proxy` 都设成 `http://USER:PASS@HOST:PORT`，再运行 `composer install` 就行。一定要设 `https_proxy`：Packagist、GitHub 都是 HTTPS 地址，从 Composer 2.8 起，HTTPS 请求不再退回去用 `http_proxy`，只设 `http_proxy` 等于没设。账号密码里的特殊字符要分别做百分号编码。设完用 `composer diagnose` 看一眼代理那几行是不是 OK。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP 1 天 0.5 美元、30 天 7.99 美元，所有国家同价，下单时选 HTTP 协议。

*依据：Composer 官方文档[《How to use Composer behind a proxy》](https://getcomposer.org/doc/faqs/how-to-use-composer-behind-a-proxy.md)、Composer 源码（`ProxyManager`、`ProxyItem`、`DiagnoseCommand`）和更新日志，2026 年 11 月 26 日核对；本机没有装 PHP，没有实机运行。*

## 设置方法

macOS、Linux：

```bash
export http_proxy='http://USER:PASS@HOST:PORT'
export https_proxy='http://USER:PASS@HOST:PORT'
export no_proxy='localhost,127.0.0.1,packages.internal.example'

composer install
```

Windows PowerShell：

```powershell
$Env:http_proxy  = "http://USER:PASS@HOST:PORT"
$Env:https_proxy = "http://USER:PASS@HOST:PORT"
composer install
```

只想这一条命令用代理：

```bash
https_proxy='http://USER:PASS@HOST:PORT' http_proxy='http://USER:PASS@HOST:PORT' composer require monolog/monolog
```

Composer 先找小写变量名，找不到再找大写的（`HTTPS_PROXY`、`NO_PROXY` 也认）。官方建议用小写。

## 各个变量管什么

| 变量 | 管哪些请求 | 备注 |
|---|---|---|
| `https_proxy` | HTTPS 地址（Packagist、GitHub、大多数私有源） | 2.8 起不再退回用 `http_proxy` |
| `http_proxy` | HTTP 地址 | 只在命令行（CLI）里读 |
| `CGI_HTTP_PROXY` | 非命令行场景下的 HTTP 请求 | 比如 Composer 被嵌进 Web 应用里运行时 |
| `no_proxy` | 不走代理的域名 | 逗号分隔，可以写端口、IP、CIDR 网段；写 `*` 表示全部直连 |

`no_proxy=example.com` 会同时排除 `example.com` 和它的子域名；域名前面的点没有意义，会被去掉。

## 账号密码里有特殊字符

用户名和密码要**分别**做百分号编码，再用冒号连起来。官方文档的例子：用户名 `me@company`、密码 `p@ssw$rd`，写成：

```text
http://me%40company:p%40ssw%24rd@HOST:PORT
```

用户名里不能有冒号，编码了也不行，因为代理会按第一个冒号切开用户名和密码。

## 用 composer diagnose 检查

```bash
composer diagnose
```

设了代理时，输出里会多出检查代理的行。代理地址是 `http://` 开头的，会看到 `Checking HTTP proxy with http` 和 `Checking HTTP proxy with https` 两行。后面是 `OK` 就说明 Composer 已经通过代理拿到了 Packagist 的数据；被 `no_proxy` 排除的显示 `SKIP`；如果提示代理在「modifying traffic on the fly」，说明有设备在改写经过的流量。只有 `Checking HTTP proxy with http` 一行、没有 https 那行，就是 `https_proxy` 没设上。

再从同一个终端确认代理本身：

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" -o /dev/null -w '%{http_code}\n' https://repo.packagist.org/packages.json
```

## SOCKS5 和 HTTPS 代理

- 官方文档只讲 HTTP 和 HTTPS 两种代理地址，没有提 SOCKS。用 Proxy302 的话，买静态 IP 时协议选 HTTP。
- 「HTTPS 代理」指的是用 TLS 连到代理本身（地址写 `https://HOST:443`），需要 PHP 7.3 及以上、curl 7.52.0 及以上。普通的 HTTP 代理照样能访问 HTTPS 网站，地址前缀写 `http://` 就对了。

## 报错对照

| 现象或报错 | 原因 | 怎么办 |
|---|---|---|
| 设了代理，下载还是直连、很慢 | 只设了 `http_proxy` | 加上 `https_proxy` |
| `407 Proxy Authentication Required` | 账号密码错，或特殊字符没编码 | 分别编码用户名和密码 |
| 连不上代理 | 地址里漏了端口，Composer 按默认的 80（`https://` 开头时是 443）去连 | 写全 `HOST:PORT` |
| ``unsupported `https_proxy` syntax`` | 地址里混进了换行、制表符，或者整串解析不了 | 重新写这一行，别从带格式的文档里直接粘 |
| `composer diagnose` 里代理那一行不是 OK | 代理连不上，或者中间有设备改写了 HTTPS 流量 | 先用 curl 测代理，再看网络里有没有解密 HTTPS 的设备 |
| 用 `sudo composer` 就不走代理 | `sudo` 默认不带当前用户的环境变量 | 别用 sudo 跑 Composer，或者用 `sudo -E` |

## 常见问题

**`composer config` 能设代理吗？** 不能。Composer 只从环境变量读代理。要长期生效，把 `export` 写进 `~/.bashrc`、`~/.zshrc`，或者 CI 的环境变量设置里。

**以前只设 `http_proxy` 也能用，现在为什么不行了？** 2.7.3 之前 HTTPS 请求会退回用 `http_proxy`，2.7.3 开始对这种情况给出警告，2.8.0 起彻底去掉了这个退回。

**`HTTP_PROXY_REQUEST_FULLURI` 还要设吗？** 不用，官方说这两个旧变量已经不再使用。

**用动态 IP 还是静态 IP？** 只是装依赖，按流量计费的动态住宅就够（1.5 美元/GB 起）。私有包仓库按来源 IP 放行的，用静态 IP。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，所有国家同价）和动态住宅流量（预付流量不过期）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。同类工具：[npm 走代理](../npm-proxy/README.md)、[pip 设置代理](../pip-proxy-zh/README.md)。

*Facts checked 26 November 2026.*
