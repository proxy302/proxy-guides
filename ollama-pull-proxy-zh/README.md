# Ollama 拉模型怎么走带账号密码的代理？

给 Ollama 的服务端设 `HTTPS_PROXY=http://USER:PASS@HOST:PORT`，然后重启服务。拉模型的是后台的 `ollama serve`（macOS、Windows 上是那个常驻应用，Linux 上是 systemd 服务），不是你敲 `ollama pull` 的那个终端，所以只在终端里 `export` 变量没有用。不要设 `HTTP_PROXY`：Ollama 官方 FAQ 说拉模型只走 HTTPS，设了 `HTTP_PROXY` 反而可能打断客户端和本机服务之间的连接。本指南由 Proxy302 维护，我们也销售这些代理；模型文件动辄几个 GB，按 IP 和天数计费的静态 ISP（1 天 0.5 美元，30 天 7.99 美元）比按流量计费的动态住宅（1.5 美元/GB 起）更好算账。

*本页按 Ollama 官方 FAQ（docs.ollama.com/faq，2026 年 11 月）核对；测试机上没有装 Ollama，命令没有实机运行。*

## Linux（systemd 安装）

```bash
sudo systemctl edit ollama.service
```

在打开的编辑器里加：

```ini
[Service]
Environment="HTTPS_PROXY=http://USER:PASS@HOST:PORT"
```

保存后：

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
systemctl show ollama --property=Environment
```

最后一条能看到变量已经进了服务。USER、PASS、HOST、PORT 是占位符。密码里的特殊字符先做百分号编码（`@` 写成 `%40`，`:` 写成 `%3A`），而 systemd 文件里 `%` 本身要写成 `%%`，所以 `@` 最后写成 `%%40`。

## macOS（Ollama 应用）

```bash
launchctl setenv HTTPS_PROXY "http://USER:PASS@HOST:PORT"
```

然后从菜单栏退出 Ollama，再重新打开。官方 FAQ 用的就是 `launchctl setenv` 给应用传变量。用这种方式设的变量重启电脑后会消失，要重新执行一次。

不想动应用，也可以先退出应用，在终端里自己起服务：

```bash
HTTPS_PROXY="http://USER:PASS@HOST:PORT" ollama serve
```

## Windows

Ollama 在 Windows 上继承用户和系统的环境变量：

1. 从任务栏右下角退出 Ollama。
2. 打开「编辑帐户的环境变量」，新建用户变量 `HTTPS_PROXY`，值填 `http://USER:PASS@HOST:PORT`。
3. 从开始菜单重新启动 Ollama。

## Docker

```bash
docker run -d -e HTTPS_PROXY="http://USER:PASS@HOST:PORT" \
  -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

变量要传给容器；宿主机上的 Docker 守护进程代理只管拉 `ollama/ollama` 这个镜像，不管容器里拉模型。

## 验证

```bash
ollama pull all-minilm
```

先拉一个几十 MB 的小模型试试。能下载就说明代理通了。想确认出口，先单独测代理：

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

## 各平台设置一览

| 平台 | 在哪设 | 改完要做什么 |
|---|---|---|
| Linux（systemd） | `systemctl edit ollama.service` 的 `[Service]` 段 | `daemon-reload` 后 `restart` |
| macOS 应用 | `launchctl setenv HTTPS_PROXY ...` | 退出并重开应用；重启电脑后重设 |
| Windows | 用户环境变量 `HTTPS_PROXY` | 退出并重开 Ollama |
| Docker | `docker run -e HTTPS_PROXY=...` | 重建容器 |

## 常见报错

| 现象 | 多半是 | 怎么办 |
|---|---|---|
| 报错里有 `407 Proxy Authentication Required` | 账号密码错，或特殊字符没编码 | 重新复制；按上面的方法编码 |
| 报错里有 `proxyconnect tcp` 和 `connection refused` | HOST 或 PORT 填错，或这个 IP 已到期 | 对照订单页；到期就续费 |
| 设了变量，下载还是直连 | 变量设在了终端里，服务端没拿到 | 按上面的平台方法设在服务上并重启 |
| `ollama list` 之类的命令也报代理错误 | 终端里设了 `HTTP_PROXY`，本机请求被送去了代理 | 去掉 `HTTP_PROXY`，或设 `NO_PROXY=localhost,127.0.0.1` |
| 证书错误 | 公司网络在解密流量 | 按官方 FAQ 把公司证书装进系统（Docker 镜像里也要装） |

## 常见问题

**SOCKS5 的 IP 能用吗？** 官方 FAQ 只写了 HTTPS 代理的用法。给 Ollama 用，买 HTTP 类型最稳；Proxy302 的静态 IP 是下单时选协议，一个 IP 只有一种。

**代理会影响本地对话、API 调用吗？** 你的程序连本机 `localhost:11434` 这一段不经过代理。`HTTPS_PROXY` 管的是 Ollama 服务自己往外连的请求，比如拉模型。

**下载断了怎么办？** 重新执行同一条 `ollama pull`。断得频繁，先用 curl 测一下代理本身的速度和延迟，见[代理 IP 很慢怎么查](../proxy-slow-diagnose-zh/README.md)。

**有免费试用吗？** 没有。最便宜的测试是一个静态 ISP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，1 天 0.5 美元、30 天 7.99 美元，每个国家同价）和动态住宅流量（1.5 美元/GB 起，预付流量不过期）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。

*Facts checked 23 November 2026.*
