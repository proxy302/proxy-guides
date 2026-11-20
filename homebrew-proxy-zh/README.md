# Homebrew 怎么走代理（brew install、brew update 下载慢或失败）

在运行 `brew` 的终端里先设好环境变量：HTTP 代理用 `export https_proxy=http://USER:PASS@HOST:PORT` 和 `export http_proxy=http://USER:PASS@HOST:PORT`，SOCKS5 代理用 `export all_proxy=socks5h://USER:PASS@HOST:PORT`，然后照常 `brew update`、`brew install`。Homebrew 下载靠 curl 和 git，这几个变量是 Homebrew 官方手册里列出的，curl、git 都会照着走。只想让 brew 走代理、不影响终端里其他程序，就把变量写在命令前面或者做成一个别名。

*本指南由 Proxy302 维护，我们也销售文中提到的代理。内容在 2026 年 11 月对照 Homebrew 官方手册（Manpage 的环境变量一节）核对过；curl 和 git 读这些变量、以及表里的 curl 报错，在本机用临时搭的测试代理跑过，brew 本身没有实机运行（测试机上没有装 Homebrew）。*

## 三种用法

### 只对一条命令生效

```bash
https_proxy=http://USER:PASS@HOST:PORT http_proxy=http://USER:PASS@HOST:PORT brew install wget
```

关掉终端就没了，最不容易忘了关。

### 当前终端窗口一直生效

```bash
export https_proxy=http://USER:PASS@HOST:PORT
export http_proxy=http://USER:PASS@HOST:PORT
brew update
brew upgrade
```

用完 `unset https_proxy http_proxy` 取消。

### 做成别名，只给 brew 用

写进 `~/.zshrc`：

```bash
alias brewp='https_proxy=http://USER:PASS@HOST:PORT http_proxy=http://USER:PASS@HOST:PORT brew'
```

`source ~/.zshrc` 之后，`brewp install node` 走代理，`brew install node` 照旧直连。账号密码写在 `~/.zshrc` 里是明文，别把这个文件同步到公开仓库。

## 变量对照

| 变量 | 作用 | 写法 |
|---|---|---|
| `https_proxy` | 下载 `https://` 地址时用的代理，Homebrew 绝大多数下载都是这种 | `http://USER:PASS@HOST:PORT` |
| `http_proxy` | 下载 `http://` 地址时用的代理 | 同上 |
| `all_proxy` | Homebrew 手册里写的是 SOCKS5 代理 | `socks5h://USER:PASS@HOST:PORT` |
| `no_proxy` | 不走代理的主机，逗号分隔 | `localhost,127.0.0.1,.corp.example` |

注意：

- 代理地址的协议写 `http://`，哪怕要下载的是 `https://` 的东西。写成 `https://HOST:PORT` 会让 curl 用 TLS 去连代理，普通 HTTP 代理会报 SSL 错误。
- SOCKS5 用 `socks5h://`，让代理去解析域名。
- 密码里有 `@`、`:`、`/` 这类字符要先做百分号编码，比如 `p@ss` 写成 `p%40ss`。
- 在 Proxy302 买的 IP，HTTP 还是 SOCKS5 是下单时选定的，一个 IP 一种协议，按订单页上的协议填。

## 第一次安装 Homebrew

官方安装命令本身也要用 curl 下载脚本。先在同一个终端 `export` 好上面两个变量，再运行官网给的安装命令，安装脚本里的 curl 和 git 也会走代理。

## 检查是不是真的走了代理

```bash
export https_proxy=http://USER:PASS@HOST:PORT
curl -sS https://ipinfo.io/json
```

返回的 `ip` 和 `country` 是代理的出口，说明这个终端里的 curl 已经走代理，brew 下载也一样。

## 常见报错

| 报错 | 原因 |
|---|---|
| `curl: (56) CONNECT tunnel failed, response 407` | 账号密码错了，或密码里的特殊字符没编码 |
| `curl: (35)` 一类 SSL 报错 | 代理地址写成了 `https://`，改成 `http://` |
| `Failed to connect to HOST port PORT` | 地址或端口错，或本机防火墙拦了 |
| 设了变量还是很慢 | 在别的终端窗口设的；或者慢在代理到下载服务器这一段，换个国家的 IP 试试 |
| `brew update` 卡在 git | git 也读这几个变量；如果设过 `git config --global http.proxy`，以 git 自己的配置为准，见 [git 代理](../git-proxy/README.md) |

## 常见问题

**用镜像源还是用代理？** 国内镜像能加快 bottle 下载；代理适合镜像没有的东西、或公司网络只能经代理出网的情况。两者可以同时用，镜像地址加进 `no_proxy` 就不会绕路。

**会不会把电脑上所有程序都带上代理？** 不会。环境变量只影响这个终端里启动的程序。整机都要走代理是其他全局网络工具的做法，和这里不是一回事。

**用动态住宅还是静态 IP？** 下载软件包按流量算更划算，动态住宅 1.5 美元/GB 起；如果公司防火墙只放行固定出口，就用静态 IP。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（1 天 0.5 美元，30 天 7.99 美元，每个国家同价）和动态住宅流量（1.5 美元/GB 起）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/pricing](https://proxy302.com/pricing/)。

*事实核对于 2026 年 11 月 20 日。*
