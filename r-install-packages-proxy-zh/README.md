# R 的 install.packages() 怎么走带账号密码的代理？

在 R 里先运行 `Sys.setenv(https_proxy = "http://USER:PASS@HOST:PORT", http_proxy = "http://USER:PASS@HOST:PORT")`，再 `install.packages()`；想一直生效，就把这两行写进用户目录下的 `.Renviron` 文件，重启 R。R 下载默认用 libcurl（`download.file.method` 为 `"auto"` 时，除了 `file://` 都用 libcurl），libcurl 会读这些环境变量，账号密码直接写在代理地址里。CRAN 镜像是 `https://` 开头的，关键是 `https_proxy` 这一个。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP IP 1 天 0.5 美元、30 天 7.99 美元，动态住宅 1.5 美元/GB 起。

*内容对照 R 的 `?download.file` 帮助页（Setting Proxies 一节和认证说明）核对；没有在装了 R 的机器上实跑，环境变量的行为用 curl 命令行验证过（同样基于 libcurl）。*

## 临时用一次

```r
Sys.setenv(
  http_proxy  = "http://USER:PASS@HOST:PORT",
  https_proxy = "http://USER:PASS@HOST:PORT",
  no_proxy    = "localhost,127.0.0.1"
)
install.packages("data.table")
```

- USER、PASS、HOST、PORT 是占位符。密码里有 `@`、`:`、`/`、`#` 要做百分号编码，比如 `p@ss` 写成 `p%40ss`。
- 代理是 SOCKS5 的，写成 `socks5h://USER:PASS@HOST:PORT`，`h` 表示域名交给代理解析。
- `Sys.setenv()` 只对当前 R 会话有效，关掉就没了。

## 一直生效：写进 .Renviron

`.Renviron` 在用户主目录下（Windows 一般是「文档」文件夹）。装了 usethis 的话，`usethis::edit_r_environ()` 会直接打开它。写入：

```text
http_proxy=http://USER:PASS@HOST:PORT
https_proxy=http://USER:PASS@HOST:PORT
no_proxy=localhost,127.0.0.1
```

保存后重启 R（RStudio 里是 Session › Restart R），确认读到了：

```r
Sys.getenv("https_proxy")
getOption("download.file.method")   # NULL 或 "libcurl" 都行，NULL 表示用默认的 auto
capabilities("libcurl")             # 应为 TRUE
```

`.Renviron` 里是明文密码，别把它提交到 Git 仓库。项目目录下也可以放一个 `.Renviron`，同样要加进 `.gitignore`。

## 测一下通不通

R 自带的 `curlGetHeaders()` 也走 libcurl，可以直接拿来测：

```r
curlGetHeaders("https://cloud.r-project.org/")
```

返回一串 `HTTP/... 200` 开头的响应头就说明通了。在终端里也可以先单独测代理：

```bash
curl -sS -o /dev/null -w '%{http_code}\n' -x "http://USER:PASS@HOST:PORT" https://cloud.r-project.org/
```

## 不同情况怎么设

| 情况 | 做法 |
|---|---|
| 临时装一个包 | `Sys.setenv()` 后 `install.packages()` |
| 每次打开 R 都要用 | 写进 `~/.Renviron` |
| Windows 上很旧的 R（4.2 以前） | 默认可能用 wininet，读的是系统「Internet 选项」；先 `options(download.file.method = "libcurl")` 再按上面设 |
| `remotes::install_github()`、`pak`、`renv` | 它们大多也通过 libcurl 下载，同样读这些环境变量；个别工具有自己的下载设置，以它们的文档为准 |
| 只想某个镜像不走代理 | 把它的主机名写进 `no_proxy` |

## 常见报错

| 报错 | 原因 |
|---|---|
| `cannot open URL 'https://cloud.r-project.org/...'` | 没走代理或代理连不上，先看 `Sys.getenv("https_proxy")` |
| `HTTP status was '407 Proxy Authentication Required'` | 账号密码错，或特殊字符没编码 |
| `Timeout of 60 seconds was reached` | 代理地址、端口错，或 HTTP、SOCKS5 写反了；大文件可以 `options(timeout = 600)` |
| 设了 `HTTPS_PROXY` 但 `http://` 的源不走代理 | libcurl 对 HTTP 只认小写的 `http_proxy` |
| 昨天能装，今天不行 | 静态 IP 到期被释放了 |

## 常见问题

**只是装包，用动态还是静态？** 动态住宅（1.5 美元/GB 起）或动态机房（0.4 美元/GB）就够。公司内网的包仓库或防火墙按来源 IP 放行时，用静态 IP。

**Bioconductor 的包也走这个代理吗？** `BiocManager::install()` 底层用的是 `install.packages()`，同样走。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（1 到 90 天，下单时选 HTTP 或 SOCKS5，所有国家同价）和动态住宅、动态机房流量。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/pricing](https://proxy302.com/pricing/)。相关：[pip 怎么设置代理](../pip-proxy-zh/README.md)、[conda 怎么走代理](../conda-proxy-zh/README.md)。

*Facts checked 27 November 2026.*
