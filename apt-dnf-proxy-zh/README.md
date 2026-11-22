# Linux 服务器上 apt、dnf/yum 怎么走带账号密码的代理

Debian、Ubuntu 用 apt：新建 `/etc/apt/apt.conf.d/95proxy`，写一行 `Acquire::http::Proxy "http://USER:PASS@HOST:PORT/";`（https 源默认沿用这一行，也可以再写一行 `Acquire::https::Proxy`）。RHEL、Rocky、AlmaLinux、Fedora 用 dnf：在 `/etc/dnf/dnf.conf` 的 `[main]` 下加 `proxy=http://HOST:PORT`、`proxy_username=USER`、`proxy_password=PASS`。CentOS 7 的 yum 写在 `/etc/yum.conf`，键名相同。写进配置文件比设环境变量可靠，因为 `sudo` 默认会丢掉普通用户的环境变量。

*本指南由 Proxy302 维护，我们也销售文中提到的代理。内容在 2026 年 11 月对照 apt-transport-http 手册页（APT 3.x）和 dnf、dnf5 的配置手册核对过，没有实机运行（测试机是 macOS）。*

## apt（Debian、Ubuntu）

```bash
sudo tee /etc/apt/apt.conf.d/95proxy >/dev/null <<'EOF'
Acquire::http::Proxy "http://USER:PASS@HOST:PORT/";
Acquire::https::Proxy "http://USER:PASS@HOST:PORT/";
EOF
sudo chmod 600 /etc/apt/apt.conf.d/95proxy
sudo apt update
```

- 代理地址支持 `http://`、`https://` 和 `socks5h://` 三种写法。SOCKS5 代理写 `socks5h://USER:PASS@HOST:PORT/`，由代理解析域名（apt 1.3 起支持）。
- 内网镜像不想走代理，单独给它写 `DIRECT`：

  ```text
  Acquire::http::Proxy::mirror.corp.example "DIRECT";
  ```

- 只想临时用一次、不落配置文件：

  ```bash
  sudo apt -o Acquire::http::Proxy="http://USER:PASS@HOST:PORT/" update
  ```

- 密码里有 `@`、`:`、`/` 要做百分号编码，比如 `p@ss` 写成 `p%40ss`。

## dnf（RHEL 8/9/10、Rocky、AlmaLinux、Fedora）

```ini
# /etc/dnf/dnf.conf
[main]
proxy=http://HOST:PORT
proxy_username=USER
proxy_password=PASS
proxy_auth_method=basic
```

然后 `sudo dnf makecache` 测试。

- 新版 Fedora 用的 dnf5 保留了同样的键名。
- `proxy_auth_method` 默认是 `any`，写成 `basic` 能少一次来回协商。
- 某个仓库不走代理：在它的 `.repo` 文件里写一行空的 `proxy=`。
- `dnf.conf` 默认所有用户可读。里面有密码时，可以 `sudo chmod 600 /etc/dnf/dnf.conf`。

## yum（CentOS 7 等老系统）

```ini
# /etc/yum.conf
[main]
proxy=http://HOST:PORT
proxy_username=USER
proxy_password=PASS
```

## 环境变量为什么常常不管用

很多教程让你 `export http_proxy=...` 再 `sudo apt update`。问题在于 `sudo` 默认重置环境变量，代理变量到不了 apt。要么用 `sudo -E`，要么像上面那样写进配置文件。服务器上长期用，配置文件更稳。

## 先确认代理本身可用

```bash
curl -sS -x http://USER:PASS@HOST:PORT https://deb.debian.org/debian/ -o /dev/null -w '%{http_code}\n'
```

返回 `200` 说明账号密码和网络都没问题，再去查 apt 或 dnf 的配置。

## 报错对照

| 报错 | 原因 |
|---|---|
| apt：`407  Proxy Authentication Required` | 账号密码错，或特殊字符没编码 |
| apt：`Could not connect to HOST:PORT` 或超时 | 地址、端口错，或防火墙没放行到代理的出站 |
| dnf：`Curl error (56)` 加 `CONNECT tunnel failed, response 407` | `proxy_username` / `proxy_password` 不对 |
| dnf：`Curl error (7)` | 连不上代理，检查 host 和端口 |
| 只有内网源失败 | 内网源也走了代理，按上面的方法排除 |
| 用 SOCKS5 的端口配成 `http://` | 握手失败；在 Proxy302 买的 IP 协议是下单时选定的，按订单页填 |

## 常见问题

**Docker 构建时 `apt-get` 走不了代理怎么办？** 容器里的 apt 不读宿主机的配置，要在构建参数或 Dockerfile 里另配，见 [Docker 代理](../docker-proxy-zh/README.md)。

**服务器下载远处的软件源很慢，用代理会快吗？** 不一定。先用上面的 curl 命令分别测直连和走代理的耗时，再决定；慢在哪一段的排查方法见 [代理 IP 很慢怎么查](../proxy-slow-diagnose-zh/README.md)。

**动态 IP 还是静态 IP？** 只是下载软件包，按流量计费的动态住宅就够（1.5 美元/GB 起）。如果软件源或防火墙按来源 IP 放行，用静态 IP。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5；1 天 0.5 美元，30 天 7.99 美元，每个国家同价）和动态住宅流量。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/pricing](https://proxy302.com/pricing/)。

*事实核对于 2026 年 11 月 20 日。*
