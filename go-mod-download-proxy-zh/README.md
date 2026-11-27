# go mod download、go get 怎么走带账号密码的代理？GOPROXY 和 HTTPS_PROXY 不是一回事

在终端里设环境变量 `HTTPS_PROXY=http://USER:PASS@HOST:PORT`（再把 `HTTP_PROXY` 设成同样的值），然后照常运行 `go mod download`、`go get`、`go install`。go 命令用 Go 标准库的 `net/http` 下载，会读 `HTTP_PROXY`、`HTTPS_PROXY`、`NO_PROXY`（大小写都认），账号密码写在代理地址里就行。`GOPROXY` 管的是去哪个**模块镜像**拿代码（默认 `https://proxy.golang.org,direct`），不是网络代理，别把代理服务器的地址填进 `GOPROXY`。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP IP 1 天 0.5 美元、30 天 7.99 美元，动态住宅 1.5 美元/GB 起。

*内容对照 Go 模块参考文档（go.dev/ref/mod）和 `net/http` 的 `ProxyFromEnvironment` 文档核对，没有实机运行（测试机没装 Go）。*

## 两个变量各管什么

| 变量 | 管什么 | 例子 |
|---|---|---|
| `HTTPS_PROXY` / `HTTP_PROXY` | 网络代理：go 命令发出的 HTTP 请求从哪台代理出去 | `http://USER:PASS@HOST:PORT` |
| `NO_PROXY` | 哪些主机不走网络代理 | `git.corp.example,localhost` |
| `GOPROXY` | 模块镜像：从哪个模块服务拿代码，`direct` 表示直接找代码仓库，`off` 表示不联网 | `https://proxy.golang.org,direct` |
| `GOPRIVATE` | 哪些模块是私有的：不走模块镜像、也不查校验和数据库 | `github.com/your-org/*` |
| `GOSUMDB` | 校验和数据库，默认 `sum.golang.org` | 一般不用改 |

模块镜像和校验和数据库也都是 HTTPS 网站，所以设好 `HTTPS_PROXY` 后，访问它们也会经过网络代理。

## 设置方法

**macOS、Linux：**

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
export HTTP_PROXY="http://USER:PASS@HOST:PORT"
export NO_PROXY="git.corp.example,localhost"   # 内网 Git、内网模块镜像放这里

go mod download
```

**Windows PowerShell：**

```powershell
$env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
$env:HTTP_PROXY  = "http://USER:PASS@HOST:PORT"
go mod download
```

- USER、PASS、HOST、PORT 是占位符。密码里有 `@`、`:`、`/` 要做百分号编码，比如 `p@ss` 写成 `p%40ss`。
- 代理是 SOCKS5 的，写 `socks5://USER:PASS@HOST:PORT`，Go 的 `net/http` 支持这个写法。
- **别用 `go env -w HTTPS_PROXY=...`。** `go env -w` 只接受 go 命令自己的变量（GOPROXY、GOPRIVATE 这些），写 `HTTPS_PROXY` 会报「unknown go command variable」。网络代理放在系统或 shell 的环境变量里。

## 验证是不是真的走了代理

先单独测代理：

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://proxy.golang.org/ -o /dev/null -w '%{http_code}\n'
```

再让 go 命令去拿一个模块的版本列表（会联网查询）：

```bash
go list -m -versions golang.org/x/text
```

能列出版本就说明通了。想看细节，`go mod download -x` 会把执行的命令打印出来。

## 私有仓库和 direct 模式

`GOPRIVATE` 里的模块、或者 `GOPROXY` 落到 `direct` 的时候，go 命令会调用 `git` 这类版本控制工具直接去仓库拉代码：

- 仓库地址是 `https://` 的：git 本身也读 `https_proxy`、`HTTPS_PROXY`，上面的环境变量一般就够了；git 里单独配过 `http.proxy` 的，以 git 的配置为准。见 [git 怎么走代理](../git-proxy/README.md)。
- 仓库地址是 `git@...` 的 SSH：环境变量管不到 SSH，要在 `~/.ssh/config` 里配，见 [SSH 走代理](../ssh-through-proxy/README.md)。
- 内网 Git 服务器不该走代理的，加进 `NO_PROXY`。

## 公司自建模块镜像时

`GOPROXY` 指向公司内网的镜像（比如 `https://goproxy.corp.example`），这个地址就要放进 `NO_PROXY`，不然请求会被送到外面的代理去，内网地址从外面当然访问不到。

## 常见报错

| 报错 | 原因 |
|---|---|
| `proxyconnect tcp: ... connection refused` | 代理地址或端口错，或者 SOCKS5 的 IP 写成了 `http://` |
| `Proxy Authentication Required` / 407 | 账号密码错，或者特殊字符没编码 |
| `dial tcp: lookup proxy.golang.org: no such host` | 根本没走代理，检查环境变量是不是设在了当前终端 |
| `verifying module: ... sum.golang.org` 超时 | 校验和数据库访问不了；代理设好后一般就好，私有模块用 `GOPRIVATE` |
| `go: unknown go command variable HTTPS_PROXY` | 用了 `go env -w`，改成系统环境变量 |
| 内网模块拉不下来 | 内网地址也被送去了代理，加进 `NO_PROXY` |

## 常见问题

**`GOPROXY` 能不能填 `http://USER:PASS@HOST:PORT`？** 不能这么用。`GOPROXY` 要填的是实现了模块代理协议的服务，普通网络代理不是。

**`go build` 也要代理吗？** 依赖已经下载到模块缓存里，`go build` 就不再联网；只有缺依赖时才会下载。

**用动态还是静态 IP？** 只是下载依赖，动态住宅按流量（1.5 美元/GB 起）或动态机房（0.4 美元/GB）就够。私有仓库或公司防火墙按来源 IP 放行时，用静态 IP。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（1 到 90 天，下单时选 HTTP 或 SOCKS5，所有国家同价）和动态住宅、动态机房流量。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/pricing](https://proxy302.com/pricing/)。写 Go 程序本身要走代理，看 [Go net/http 代理](../go-http-proxy/README.md)。

*Facts checked 27 November 2026.*
