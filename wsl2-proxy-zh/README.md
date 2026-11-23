# WSL2 里怎么走带账号密码的代理？

Windows 的系统代理只能填地址和端口，没有地方存账号密码；WSL 的 autoProxy 功能把 Windows 的代理地址带进 Linux，同样带不进账号密码，所以 WSL2 里要自己设：在 `~/.bashrc` 里写 `export https_proxy="http://USER:PASS@HOST:PORT"`（大小写各写一份，再加 `no_proxy`），并在 `%UserProfile%\.wslconfig` 的 `[wsl2]` 下写 `autoProxy=false`，免得两套设置互相覆盖。`sudo apt`、Docker、git、systemd 服务不读你的 `~/.bashrc`，要各自单独配。本指南由 Proxy302 维护，我们也销售这些代理；给 WSL 里的命令行工具用，HTTP 类型的静态 ISP 最省事（1 天 0.5 美元，30 天 7.99 美元）。

*本页按微软 WSL 文档（.wslconfig 设置和网络两篇，2026 年 10 月更新版）核对；测试机是 macOS，没有在 Windows 上实机运行。*

## 第 1 步：关掉 autoProxy

`autoProxy` 默认是开的（只在 Windows 11 上有这个设置），作用是让 WSL 使用 Windows 的 HTTP 代理信息。Windows 那边没有账号密码，带进来的代理地址一用就是 407。在 Windows 里编辑 `C:\Users\你的用户名\.wslconfig`（没有就新建）：

```ini
[wsl2]
autoProxy=false
```

然后在 PowerShell 里执行 `wsl --shutdown`，等几秒再打开 WSL。微软文档提醒：改完配置要等 WSL 完全停下来（大约 8 秒）再启动，设置才会生效。微软也建议优先用开始菜单里的「WSL Settings」应用改配置，效果和改这个文件一样。

Windows 10 没有 autoProxy，跳过这一步。

## 第 2 步：在 WSL 里写代理变量

```bash
cat >> ~/.bashrc <<'EOF'
export http_proxy="http://USER:PASS@HOST:PORT"
export https_proxy="$http_proxy"
export HTTP_PROXY="$http_proxy"
export HTTPS_PROXY="$http_proxy"
export no_proxy="localhost,127.0.0.1,::1"
export NO_PROXY="$no_proxy"
EOF
source ~/.bashrc
```

USER、PASS、HOST、PORT 是占位符。大小写各写一份，是因为不同工具读的不一样，比如 curl 只认小写的 `http_proxy`。用 zsh 的写进 `~/.zshrc`。

密码里有 `@`、`:`、`#`、`/`，先做百分号编码：`@` 写成 `%40`，`:` 写成 `%3A`，`#` 写成 `%23`，`/` 写成 `%2F`。

## 第 3 步：验证

```bash
env | grep -i proxy
curl -sS --max-time 20 https://ipinfo.io/json
```

第一条确认只有你写的那一套变量；第二条看 `ip`、`country` 是不是你买的 IP 和国家。

## 不读 ~/.bashrc 的那些程序

| 程序 | 为什么不生效 | 怎么配 |
|---|---|---|
| `sudo apt` | sudo 默认丢掉普通用户的环境变量 | 写 `/etc/apt/apt.conf.d/95proxy`，见 [apt、dnf 怎么走代理](../apt-dnf-proxy-zh/README.md) |
| Docker 拉镜像 | 拉镜像的是 dockerd 服务 | 见 [Docker 拉镜像怎么走代理](../docker-proxy-zh/README.md) |
| git | 读环境变量，但很多人在 git 配置里另有设置 | 见 [git 怎么走代理](../git-proxy/README.md)（英文） |
| systemd 服务 | 服务不经过你的 shell | `sudo systemctl edit 服务名`，在 `[Service]` 下加 `Environment="HTTPS_PROXY=http://USER:PASS@HOST:PORT"`；systemd 文件里密码编码后的 `%` 要写成 `%%` |
| pip | 读环境变量，也可以单独配 | 见 [pip 怎么设置代理](../pip-proxy-zh/README.md) |
| Windows 上的程序 | WSL 里的设置只管 Linux 这一侧 | 在 Windows 里各自设置 |

## NAT 模式和镜像模式有没有影响

代理在远端机器上（HOST 是代理服务商给的地址）时，两种模式都一样，WSL 直接连 HOST:PORT。

只有一种情况要留意：你在 Windows 本机跑了一个本地转发程序（监听 `127.0.0.1`），想让 WSL 连它。

- **镜像模式**（`.wslconfig` 里 `networkingMode=mirrored`，Windows 11 22H2 及以上）：WSL 里可以直接连 `127.0.0.1`。
- **默认的 NAT 模式**：WSL 里的 `127.0.0.1` 是 Linux 自己，要用 Windows 主机的地址，查法是 `ip route show | grep -i default | awk '{ print $3}'`；本地转发程序也要监听这个网卡，不能只听 `127.0.0.1`。

## 常见问题排查

| 现象 | 多半是 | 怎么办 |
|---|---|---|
| `CONNECT tunnel failed, response 407` | 走的是 autoProxy 带进来的无密码地址，或账号密码错 | 关 autoProxy；`env \| grep -i proxy` 看实际的值 |
| `Could not resolve proxy` | HOST 拼错 | 从订单页重新复制 |
| 终端里 curl 能用，`sudo apt update` 不行 | sudo 丢了环境变量 | 用 apt 配置文件 |
| 改了 `.wslconfig` 没变化 | WSL 没有完全停下 | `wsl --shutdown` 后等几秒再开 |
| 新开的终端又没有代理 | 写进了只对当前窗口有效的 `export` | 写进 `~/.bashrc` 或 `~/.zshrc` |

## 常见问题

**只想让某一条命令走代理？** 不改 `~/.bashrc`，在命令前面临时带上：`https_proxy="http://USER:PASS@HOST:PORT" curl https://ipinfo.io/json`。

**WSL 里还要开其他全局网络工具吗？** 不要叠加。只保留这一层代理，出口才确定，排查也简单。

**SOCKS5 的 IP 能用吗？** curl 认 `socks5h://USER:PASS@HOST:PORT`，但 apt、Docker 等工具各不相同。给 WSL 整体用，买 HTTP 类型最省事；Proxy302 的静态 IP 是下单时选协议。

**有免费试用吗？** 没有。最便宜的测试是一个静态 ISP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，1 天 0.5 美元、30 天 7.99 美元，每个国家同价）和动态住宅流量（1.5 美元/GB 起）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。Windows 本身怎么设代理，见 [Windows 11 怎么设置代理](../windows-proxy-settings-zh/README.md)。

*Facts checked 23 November 2026.*
