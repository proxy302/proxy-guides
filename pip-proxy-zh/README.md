# pip 怎么设置代理（带账号密码）？

临时用一次，在命令后面加 `--proxy http://USER:PASS@HOST:PORT`；想一直用，执行 `pip config set global.proxy http://USER:PASS@HOST:PORT`，写进 pip 的配置文件；也可以只设环境变量 `HTTPS_PROXY`。三种写法里代理地址都以 `http://` 开头，哪怕要访问的 PyPI 是 HTTPS。SOCKS5 类型的 IP 要先装上 PySocks，地址写成 `socks5h://USER:PASS@HOST:PORT`，否则 pip 会报 `Missing dependencies for SOCKS support`。

*本文由 Proxy302 维护，我们出售这类代理 IP。下面的命令 2026 年 11 月在 pip 26.0 上，对着本机带账号密码的 HTTP、SOCKS5 测试代理逐条跑过。*

## 方法一：只对这一条命令生效

```bash
pip install requests --proxy http://USER:PASS@HOST:PORT
pip install -r requirements.txt --proxy http://USER:PASS@HOST:PORT
```

不想真的装，只想确认代理能不能连上 PyPI，用 `--dry-run`：

```bash
pip install --dry-run --no-deps six --proxy http://USER:PASS@HOST:PORT
```

最后一行出现 `Would install six-...`，就说明经代理连通了。

## 方法二：写进配置文件，以后每次都生效

```bash
pip config set global.proxy http://USER:PASS@HOST:PORT
```

命令会打印 `Writing to ...`，后面就是配置文件的位置，不同系统不一样，以这一行为准。查看和删除：

```bash
pip config list
pip config unset global.proxy
```

注意 `pip config list` 会把密码原样显示出来，截图或贴日志前先打码。

只想让某个虚拟环境走代理，先激活它，再加 `--site`，配置会写进这个环境自己的 `pip.conf`，不影响别的环境：

```bash
pip config --site set global.proxy http://USER:PASS@HOST:PORT
```

## 方法三：环境变量

macOS、Linux：

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
pip install requests
```

Windows 命令提示符用 `set HTTPS_PROXY=http://USER:PASS@HOST:PORT`，PowerShell 用 `$env:HTTPS_PROXY="http://USER:PASS@HOST:PORT"`。环境变量只在当前窗口有效，关掉就没了；同一个窗口里的 git、curl、npm 也会读它。

## SOCKS5 类型的 IP

pip 自己不带 SOCKS 支持，要先装 PySocks：

```bash
pip install pysocks
pip install requests --proxy socks5h://USER:PASS@HOST:PORT
```

`socks5h` 让代理去解析 `pypi.org` 这类域名。麻烦在于：如果你这台机器只能经 SOCKS5 出网，第一步装 PySocks 就装不上。办法是在能联网的机器上下载 PySocks 的 wheel 文件，拷过来用 `pip install 文件名.whl` 安装。没有这层顾虑的话，直接买 HTTP 类型更省事。

## 三种方法怎么选

| 情况 | 用哪种 |
|---|---|
| 偶尔装一次包 | `--proxy` 参数 |
| 这台电脑上 pip 一直要走代理 | `pip config set global.proxy` |
| 只有某个项目的虚拟环境要走 | `pip config --site set global.proxy` |
| CI、Docker 构建里临时用 | `HTTPS_PROXY` 环境变量 |
| 手上的 IP 是 SOCKS5 | 先装 PySocks，再用 `socks5h://` |

## 常见报错

pip 有个容易误导人的地方：连不上的时候，最后一行写的是 `Could not find a version that satisfies the requirement` 和 `No matching distribution found`，看着像包名写错了。真正的原因在上面几行 `WARNING: Retrying ...` 里。

| `WARNING: Retrying` 里写着 | 原因 |
|---|---|
| `Tunnel connection failed: 407 Proxy Authentication Required` | 账号或密码错 |
| `Failed to establish a new connection: [Errno 61] Connection refused`（Linux 上是 Errno 111） | 地址或端口不对，或本机出站被拦 |
| `Your proxy appears to only use HTTP and not HTTPS` | 代理地址写成了 `https://`，改成 `http://` |
| `EOF occurred in violation of protocol` | 多半也是写成了 `https://`，代理直接断开了连接 |
| `CERTIFICATE_VERIFY_FAILED` | 网络里有设备在解密 HTTPS，把对方的 CA 证书交给 pip：`pip config set global.cert 证书路径` |
| 直接报 `Missing dependencies for SOCKS support.` | 用了 SOCKS5 地址但没装 PySocks |

## 常见问题

**密码里有 `@`、`#`、`:` 怎么办？** 先做 URL 编码：`@` 写成 `%40`，`#` 写成 `%23`，`:` 写成 `%3A`。不编码的话，pip 会把密码的一部分当成地址。

**设了代理，下载反而慢了？** 多了一跳，路线也变了。先用 `curl -x http://USER:PASS@HOST:PORT -o /dev/null -w "%{time_total}\n" https://pypi.org/simple/` 比较一下走代理和不走代理的耗时，再决定这台机器要不要让 pip 一直走代理。

**用了公司内部的镜像源，还要走代理吗？** 内部源一般不用。如果代理是用 `HTTPS_PROXY` 环境变量设的，把内部源的域名写进 `NO_PROXY`，pip 访问它时就会直连。用 `--proxy` 或 `global.proxy` 设的代理不看 `NO_PROXY`，对所有地址都生效，我们测试时就是这样；这种情况只在访问外部源时再加代理。

**服务器装包、要固定出口给对方加白名单，用什么 IP？** 静态 ISP（静态住宅）IP，租期内地址不变。

## Proxy302 在哪里

Proxy302 的静态 ISP（静态住宅）IP 购买时选 HTTP 或 SOCKS5，给 pip 用选 HTTP 最省事。每个 IP 1 天 0.50 美元，30 天 7.99 美元，所有国家同价；动态住宅按流量计费，普通线每 GB 1.50 美元，预付流量不过期。价格见[价格页](https://proxy302.com/zh/pricing/)。

*事实核对于 2026 年 11 月 19 日。*
