# gem install、bundle install 怎么走带账号密码的代理？

最稳的做法是两个变量都设：`export https_proxy='http://USER:PASS@HOST:PORT'` 和 `export http_proxy='http://USER:PASS@HOST:PORT'`，再运行 `gem install` 或 `bundle install`。原因是 rubygems.org 是 HTTPS 地址，RubyGems 3.1 及以后的版本访问 HTTPS 源时只认 `https_proxy`（或 `HTTPS_PROXY`），只设 `http_proxy` 会直接连、不走代理；Bundler 则会在没有 `https_proxy` 时退回用 `http_proxy`。单次安装也可以用 `gem install rails --http-proxy http://USER:PASS@HOST:PORT`。只支持 HTTP 代理，不支持 SOCKS5。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP 1 天 0.5 美元、30 天 7.99 美元，所有国家同价，下单时选 HTTP 协议。

*依据：RubyGems 源码（`lib/rubygems/request.rb`，3.1.0 到 4.1.1 各版本）、Bundler 4.1 的 `bundle config` 手册；2026 年 11 月 26 日在 macOS 自带的 Ruby 2.6（RubyGems 3.0.3）上用本地带认证的测试代理实测了 `gem search --remote`。*

## 四种设置方法

| 方法 | 写法 | 管到哪里 |
|---|---|---|
| 环境变量 | `https_proxy` 和 `http_proxy` 都设 | gem、bundle，以及终端里其他读这两个变量的工具 |
| 只给 gem 的参数 | `gem install NAME --http-proxy URL`（简写 `-p URL`） | 只这一次命令 |
| `~/.gemrc` 配置文件 | 一行 `http_proxy: http://USER:PASS@HOST:PORT` | gem 每次都用；Bundler 4.1 也读这个键 |
| 分开写账号密码 | `https_proxy=http://HOST:PORT` 加 `HTTPS_PROXY_USER`、`HTTPS_PROXY_PASS` | 账号密码不想写进 URL 时用 |

### 环境变量（推荐）

```bash
export https_proxy='http://USER:PASS@HOST:PORT'
export http_proxy='http://USER:PASS@HOST:PORT'
export no_proxy='localhost,127.0.0.1,gems.internal.example'

gem install rake
bundle install
```

Windows PowerShell：

```powershell
$Env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
$Env:HTTP_PROXY  = "http://USER:PASS@HOST:PORT"
gem install rake
```

大写、小写都认，同时存在时先看小写。

### `--http-proxy` 参数

```bash
gem install rake --http-proxy 'http://USER:PASS@HOST:PORT'
```

这个参数名叫 http-proxy，但 HTTPS 的源也用它。写了 `--no-http-proxy` 则这一次完全不走代理。

### `~/.gemrc`

```yaml
http_proxy: http://USER:PASS@HOST:PORT
```

我们在 RubyGems 3.0.3 上试过：`http_proxy:` 这样写会生效，而前面带冒号的 `:http_proxy:` 不生效。新版本两种写法都认，所以统一用不带冒号的写法最省心。这个文件里有密码，记得 `chmod 600 ~/.gemrc`。

## 为什么「只设了 http_proxy」会不走代理

RubyGems 按目标地址的协议找变量：访问 `https://rubygems.org` 就找 `https_proxy` / `HTTPS_PROXY`。从 3.1.0 开始，找不到就直接连，不再退回去用 `http_proxy`。现在的 Ruby 3.x 自带的都是新版本。

我们在老版本 3.0.3 上实测，只设 `http_proxy` 时还会退回去走代理，所以「以前这样能用、换了新 Ruby 就不行」多半就是这个原因。

Bundler 不一样：4.1 的 [bundle config 手册](https://bundler.io/man/bundle-config.1.html)写明，HTTPS 源优先用 `https_proxy`，没有时才用 `http_proxy`；代码注释也说以前的版本就是这样。两个变量都设，就不用记这些差别。

## 密码里有特殊字符

写进 URL 时，用户名和密码里的 `@`、`:`、`#`、`/` 要做百分号编码，比如密码 `p@ss:1` 写成 `p%40ss%3A1`。不想编码，就用分开的变量：

```bash
export https_proxy='http://HOST:PORT'
export HTTPS_PROXY_USER='USER'
export HTTPS_PROXY_PASS='PASS'
```

RubyGems 只在 URL 里没有账号密码时才去读这两个变量（访问 HTTP 源时对应的是 `HTTP_PROXY_USER`、`HTTP_PROXY_PASS`）。

## SOCKS5 不行

RubyGems 的 `--http-proxy` 只接受 `http://`、`https://` 等地址，底层的 Net::HTTP 也不支持 SOCKS。用 Proxy302 的话，买静态 IP 时协议选 HTTP；手上只有 SOCKS5 的，需要在本机加一层转发，把 SOCKS5 变成 HTTP 代理。

## 先确认代理本身可用

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" -o /dev/null -w '%{http_code}\n' https://rubygems.org/api/v1/gems/rake.json
```

返回 `200` 说明账号密码和网络都没问题；`407` 是代理账号密码不对。

## 报错对照

| 报错或现象 | 原因 | 怎么办 |
|---|---|---|
| 设了代理，安装还是很慢，像是没生效 | 只设了 `http_proxy`，新版 RubyGems 对 HTTPS 源不用它 | 加上 `https_proxy` |
| `407 Proxy Authentication Required` | 账号密码错，或特殊字符没编码 | 重新复制、编码，或改用 `HTTPS_PROXY_USER` / `HTTPS_PROXY_PASS` |
| `Errno::ECONNREFUSED` | 代理主机、端口写错 | 用上面的 curl 单独测 |
| `bundle install` 走了代理，`gem install` 没走 | 两者对 `http_proxy` 的处理不同 | 设 `https_proxy` |
| 原生扩展编译时下载失败 | 编译脚本自己下载依赖，可能不读这些变量 | 看具体 gem 的说明，或者先在本机装好系统依赖 |
| 加了 `sudo` 就不走代理 | `sudo` 默认丢掉当前用户的环境变量 | 用 `sudo -E`，或者用 rbenv、asdf 这类装在用户目录的 Ruby |

## 常见问题

**`bundle config` 里能设代理吗？** Bundler 的代理来自 RubyGems 配置文件里的 `http_proxy` 键和上面的环境变量，没有单独的 `bundle config` 代理项。

**CI 里怎么设？** 在流水线的环境变量里加 `https_proxy` 和 `http_proxy`，密码放在 CI 的密钥设置里，不要写进仓库。

**用动态 IP 还是静态 IP？** 只是装 gem，按流量计费的动态住宅就够（1.5 美元/GB 起）。私有 gem 源按来源 IP 做白名单的，用静态 IP。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，一个 IP 一种协议）和动态住宅流量。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。同类工具：[pip 设置代理](../pip-proxy-zh/README.md)、[Homebrew 走代理](../homebrew-proxy-zh/README.md)。

*Facts checked 26 November 2026.*
