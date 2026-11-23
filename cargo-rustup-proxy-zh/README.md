# cargo 和 rustup 怎么走带账号密码的代理？

cargo：在 `~/.cargo/config.toml` 里写 `[http]` 段，`proxy = "http://USER:PASS@HOST:PORT"`；也可以用环境变量 `CARGO_HTTP_PROXY`。都没设时，cargo 会去读 git 全局配置里的 `http.proxy`，再读 `HTTPS_PROXY`、`https_proxy`、`http_proxy`。rustup：设环境变量 `https_proxy=http://USER:PASS@HOST:PORT` 就行，官方文档说大多数情况只要这一个。git 依赖拉不下来时，打开 `net.git-fetch-with-cli = true`，让系统的 git 按它自己的代理设置去拉。本指南由 Proxy302 维护，我们也销售这些代理；给开发工具用，HTTP 类型的静态 ISP 最省事（1 天 0.5 美元，30 天 7.99 美元）。

*本页按 Cargo Book 的配置说明和 rustup 手册的「Network proxies」一节核对（2026 年 11 月），测试机上没有装 Rust 工具链，命令没有实机运行。*

## cargo：写配置文件

```toml
# ~/.cargo/config.toml（Windows：%USERPROFILE%\.cargo\config.toml）
[http]
proxy = "http://USER:PASS@HOST:PORT"
timeout = 60          # 每个请求的超时秒数，默认 30

[net]
retry = 3             # 网络出错时重试次数
git-fetch-with-cli = true   # git 依赖用系统 git 拉取（见下文）
```

USER、PASS、HOST、PORT 是占位符。写在项目目录的 `.cargo/config.toml` 里也行，只对这个项目生效；但里面有密码，别提交进仓库。

## cargo：只用环境变量

```bash
export CARGO_HTTP_PROXY="http://USER:PASS@HOST:PORT"
cargo fetch
```

Windows PowerShell：

```powershell
$env:CARGO_HTTP_PROXY = "http://USER:PASS@HOST:PORT"
cargo fetch
```

## cargo 按什么顺序找代理

| 顺序 | 来源 | 备注 |
|---|---|---|
| 1 | `http.proxy`（config.toml）或 `CARGO_HTTP_PROXY` | 推荐用这个，最明确 |
| 2 | git 全局配置里的 `http.proxy` | 之前给 git 配过代理，cargo 会顺带用上 |
| 3 | `HTTPS_PROXY` / `https_proxy` | https 请求用 |
| 4 | `http_proxy` | http 请求用 |

如果 cargo 走了一个你没想到的代理，按这个顺序挨个查：项目目录和 `~/.cargo` 下的 `config.toml`（cargo 会从当前目录往上逐级读取）、`git config --global --get http.proxy`、`env | grep -i proxy`。

## git 依赖：交给系统 git

`Cargo.toml` 里写了 `git = "https://..."` 的依赖，cargo 默认用内置的 git 库去拉。内置库对代理的支持不如命令行 git 全面，拉不下来时在 config.toml 里打开：

```toml
[net]
git-fetch-with-cli = true
```

之后 git 依赖由系统里的 `git` 命令拉取，代理按 git 自己的设置走，配置方法见 [git 怎么走代理](../git-proxy/README.md)（英文）。

## rustup：一个环境变量

```bash
export https_proxy="http://USER:PASS@HOST:PORT"
rustup update
```

Windows PowerShell：

```powershell
$env:https_proxy = "http://USER:PASS@HOST:PORT"
rustup update
```

- rustup 手册里的例子也有 `socks5://` 写法；Proxy302 的 IP 是下单时选协议，SOCKS5 的 IP 就把前缀换成 `socks5://`。
- 安装脚本 `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` 里的 curl 也读 `https_proxy`，先设变量再装就行。
- rustup 手册说 curl 后端（`RUSTUP_USE_CURL=1`）已经不推荐使用，正常情况不用开。

## 密码里有特殊字符

`@`、`:`、`#`、`/` 要先做百分号编码：`@` 写成 `%40`，`:` 写成 `%3A`，`#` 写成 `%23`，`/` 写成 `%2F`。不编码的话，工具会把密码的一部分当成主机名。

## 先测代理，再查 cargo

```bash
curl -sS -I --max-time 20 -x "http://USER:PASS@HOST:PORT" https://index.crates.io/config.json -o /dev/null -w "%{http_code}\n"
```

返回 `200` 说明代理没问题。cargo 还是失败，用 `CARGO_HTTP_DEBUG=true CARGO_LOG=network=debug cargo fetch` 打开网络调试日志，看请求卡在哪一步。

## 常见报错

| 报错里出现 | 多半是 | 怎么办 |
|---|---|---|
| `[56]`、`CONNECT tunnel failed, response 407` | 账号密码错，或特殊字符没编码 | 重新复制，按上一节编码 |
| `[7]`、`Couldn't connect to server` | 主机或端口错，或公司防火墙不放行 | 先跑上面的 curl 测试 |
| `[28]`、`Timeout was reached` | 网络慢或代理不通 | 调大 `http.timeout`，查代理 |
| `spurious network error`，重试后成功 | 偶发网络抖动 | 不用处理；频繁出现再查代理延迟 |
| 只有 git 依赖失败 | 内置 git 库没走上代理 | 打开 `net.git-fetch-with-cli` |
| rustup 能装，cargo 不走代理 | 只设了 `https_proxy`，但 git 配置或 config.toml 里有别的代理 | 按「查找顺序」表逐个核对 |

## 常见问题

**设了代理，crates.io 下载反而慢？** 多了一跳，路线也变了。用 curl 的 `-w "%{time_total}\n"` 分别测走代理和不走代理的耗时再决定，排查方法见[代理 IP 很慢怎么查](../proxy-slow-diagnose-zh/README.md)。

**私有 registry 按来源 IP 放行，用什么 IP？** 静态 ISP（静态住宅）IP，租期内地址不变，白名单上那一条一直有效。

**CI 里怎么配？** 把代理地址存成 CI 的密钥，在任务里导出成 `CARGO_HTTP_PROXY` 和 `https_proxy`，不要写进仓库里的 config.toml。

**有免费试用吗？** 没有。最便宜的测试是一个静态 ISP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，1 天 0.5 美元、30 天 7.99 美元，每个国家同价）和动态住宅流量（1.5 美元/GB 起）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。Rust 代码里怎么走代理，见 [Rust reqwest 代理](../rust-reqwest-proxy/README.md)（英文）。

*Facts checked 23 November 2026.*
