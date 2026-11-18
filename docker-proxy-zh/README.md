# Docker 拉镜像、构建镜像怎么走代理？

Docker 有三处代理设置，各管一段。拉取和推送镜像（`docker pull`、`docker push`）是后台的守护进程 dockerd 在联网，代理要配在 `/etc/docker/daemon.json` 的 `proxies` 里（Docker Engine 23.0 起支持），或者配成 systemd 的环境变量，改完重启 Docker。`docker build` 里 `RUN` 步骤的下载和容器运行时的访问，读的是客户端 `~/.docker/config.json` 里的 `proxies`，保存即生效。Docker Desktop 不认 daemon.json 里的代理，要在它的设置页里填。只在终端里 `export HTTPS_PROXY` 对 `docker pull` 没用，因为真正去拉镜像的不是你敲命令的那个终端。

*本文由 Proxy302 维护，我们出售这类代理 IP。下面的配置依据 2026 年 11 月的 Docker 官方文档核对，没有在本机实际运行。*

## 先分清是哪一段要走代理

| 动作 | 谁在联网 | 配在哪里 | 要不要重启 |
|---|---|---|---|
| `docker pull`、`docker push`、`docker login` | 守护进程 dockerd | `daemon.json` 或 systemd 环境变量；Docker Desktop 用设置页 | 要 |
| `docker build` 里 `RUN apt-get`、`RUN pip install` | 构建用的临时容器 | `~/.docker/config.json`，或 `--build-arg` | 不要 |
| 容器跑起来以后访问外网 | 容器本身 | `~/.docker/config.json`，或 `docker run -e` | 不要，只对新容器生效 |

## Linux：给守护进程配代理

### 方法一：daemon.json（推荐）

编辑 `/etc/docker/daemon.json`。文件已经存在的话，把 `proxies` 这一段合并进去，别覆盖其他配置：

```json
{
  "proxies": {
    "http-proxy": "http://USER:PASS@HOST:PORT",
    "https-proxy": "http://USER:PASS@HOST:PORT",
    "no-proxy": "localhost,127.0.0.0/8"
  }
}
```

然后重启并检查：

```bash
sudo systemctl restart docker
docker info | grep -i proxy
```

`docker info` 会列出 HTTP Proxy、HTTPS Proxy 和 No Proxy 三行，账号密码会打码显示。注意 `https-proxy` 的值也是 `http://` 开头：它表示「访问 HTTPS 仓库时用这个代理」，代理本身仍是普通 HTTP 代理。

### 方法二：systemd 环境变量

适合旧版本 Docker。新建 `/etc/systemd/system/docker.service.d/http-proxy.conf`：

```ini
[Service]
Environment="HTTP_PROXY=http://USER:PASS@HOST:PORT"
Environment="HTTPS_PROXY=http://USER:PASS@HOST:PORT"
Environment="NO_PROXY=localhost,127.0.0.0/8"
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl show --property=Environment docker
```

两种方法同时存在时，daemon.json 优先。rootless 模式把文件放在 `~/.config/systemd/user/docker.service.d/`，命令换成不带 sudo 的 `systemctl --user`。

**密码里有特殊字符：** 先做 URL 编码，例如 `#` 写成 `%23`、`@` 写成 `%40`。写进 systemd 文件时，`%` 本身还要写成 `%%`，也就是 `%%23`。daemon.json 里不用这一步。

## 构建和容器：~/.docker/config.json

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://USER:PASS@HOST:PORT",
      "httpsProxy": "http://USER:PASS@HOST:PORT",
      "noProxy": "localhost,127.0.0.0/8"
    }
  }
}
```

保存后，新建的容器和新的构建会自动带上 `HTTP_PROXY`、`HTTPS_PROXY`、`NO_PROXY` 环境变量，已经在跑的容器不受影响。验证容器的出口：

```bash
docker run --rm curlimages/curl -s https://ipinfo.io/json
```

返回的 `ip` 和 `country` 应该是代理的，不是服务器自己的。

只想让某一次构建或某一个容器走代理，不改配置文件：

```bash
docker build --build-arg HTTPS_PROXY=http://USER:PASS@HOST:PORT .
docker run --rm -e HTTPS_PROXY=http://USER:PASS@HOST:PORT IMAGE
```

**注意密码会留在哪里：** 通过 config.json 或 `-e` 传进容器的代理是明文环境变量，用 `docker inspect` 能看到，`docker commit` 会把它带进镜像。不要在 Dockerfile 里用 `ENV` 写代理。`--build-arg` 传的代理变量只在构建过程里存在，不会写进构建结果。

## Docker Desktop（Mac、Windows）

Docker Desktop 忽略 daemon.json 里的代理。打开设置，在 Resources（新版本叫 VM resources）下的 Proxies 页选手动配置（Manual），把 `http://HOST:PORT` 填进 Web Server (HTTP) 和 Secure Web Server (HTTPS)。新版本把代理分成两项：Docker Desktop 自身的代理和 Containers 代理，拉镜像走后者，没单独设置时沿用前者。代理要求账号密码（Basic 认证）时，Docker Desktop 会弹窗询问，并存进系统的凭据管理里。官方文档写明 SOCKS5 代理需要 Business 订阅，个人用 HTTP 类型更省事。

## 常见报错

| 报错里有 | 原因 |
|---|---|
| `proxyconnect tcp: dial tcp HOST:PORT: connect: connection refused` | 代理地址或端口不对，或服务器出站被防火墙拦了 |
| `Proxy Authentication Required` | 账号密码错，或特殊字符没编码 |
| `proxyconnect tcp: tls: first record does not look like a TLS handshake` | 代理地址写成了 `https://`，改成 `http://` |
| `x509: certificate signed by unknown authority` | 网络里有设备在解密 HTTPS，需要把对方的 CA 证书加进系统信任；见 [SSL errors through a proxy](../proxy-ssl-errors/README.md) |
| `TLS handshake timeout`、`context deadline exceeded` | 代理或仓库响应慢，先用 `curl -x` 单独测代理 |
| 改了配置还是直连 | 改的是 config.json 却在测 `docker pull`，或改了 daemon.json 没重启 |
| `toomanyrequests` | Docker Hub 的拉取次数限制。按 Docker 的规则登录账号或调整用量，代理不改变这条规则 |

## 常见问题

**终端里已经设了 HTTPS_PROXY，为什么还是拉不下来？** 拉镜像的是 dockerd，它是系统服务，不读你当前终端的环境变量。

**能不能只让 Docker Hub 走代理，公司内部仓库不走？** 可以。把内部仓库的域名写进 `no-proxy`。

**SOCKS5 的 IP 能用吗？** Docker Engine 文档里的例子都是 HTTP 代理，没有写明守护进程对 SOCKS5 的支持；容器里的 apt、pip、curl 等工具对 SOCKS5 的支持也各不相同。给 Docker 用，买 HTTP 类型最省事。

**服务器拉私有仓库，仓库设了 IP 白名单，用什么 IP？** 用静态 ISP（静态住宅）IP。租期内地址不变，白名单上那一条一直有效。

## Proxy302 在哪里

Proxy302 卖静态 ISP（静态住宅）IP，购买时选 HTTP 或 SOCKS5，一个 IP 只有一种协议。每个 IP 1 天 0.50 美元、30 天 7.99 美元，所有国家同价，可以按 IP 开自动续费。没有免费试用，最便宜的测试是买 1 个 IP 用 1 天。价格见[价格页](https://proxy302.com/zh/pricing/)。

*事实核对于 2026 年 11 月 19 日。*
