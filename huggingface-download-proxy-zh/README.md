# Hugging Face 下载模型（hf download、huggingface_hub）怎么走带账号密码的代理？

在运行 `hf download` 或 Python 脚本之前设置环境变量 `HTTPS_PROXY=http://USER:PASS@HOST:PORT`（密码里的特殊字符要百分号编码）就行。huggingface_hub 1.x 用的 HTTP 客户端会读这个变量；大文件走的 Xet 存储（`hf_xet`）也会读，我们实测两部分都从代理出去了。用 HTTP 代理最省事；SOCKS5 要另装扩展，而且我们没有验证 Xet 那部分。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP 1 天 0.5 美元、30 天 7.99 美元，下单时选 HTTP 或 SOCKS5；只是下载模型、不需要固定出口的，按流量计费的动态住宅 1.5 美元/GB 起。

*2026 年 11 月 26 日在 macOS、Python 3.9、huggingface_hub 1.8.0、hf_xet 1.8.0 上，通过一个本地带账号密码认证的测试代理实测。*

## 最短的写法

macOS、Linux：

```bash
export HTTPS_PROXY='http://USER:PASS@HOST:PORT'
hf download hf-internal-testing/tiny-random-gpt2 config.json
```

Windows PowerShell：

```powershell
$Env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
hf download hf-internal-testing/tiny-random-gpt2 config.json
```

Python 里也一样，变量要在 `import huggingface_hub` 之前设好（[官方文档](https://huggingface.co/docs/huggingface_hub/package_reference/environment_variables)说环境变量在导入时读取）：

```python
import os
os.environ["HTTPS_PROXY"] = "http://USER:PASS@HOST:PORT"

from huggingface_hub import hf_hub_download
path = hf_hub_download("hf-internal-testing/tiny-random-gpt2", "config.json")
print(path)
```

`transformers`、`diffusers` 的 `from_pretrained()` 下载时也是调 huggingface_hub，同样认这个变量。

## 实测：代理日志里看到了什么

我们下载了一个小模型仓库里的 `model.safetensors`（Xet 存储的文件），测试代理记录到的连接：

```text
CONNECT huggingface.co:443
CONNECT cas-server.xethub.hf.co:443   ua=...hf_xet/1.8.0
CONNECT us.aws.cdn.hf.co:443          ua=...hf_xet/1.8.0
```

也就是说，元数据请求（huggingface.co）和大文件分块下载（xethub、CDN）都经过了代理。只设 `HTTPS_PROXY` 就够；如果你的网络还要求防火墙放行，把这几个域名一起报给管理员。

## 几个要注意的地方

| 情况 | 说明 |
|---|---|
| 密码有 `@`、`:`、`#` | 写进 URL 前要编码：`@` 写成 `%40`，`:` 写成 `%3A` |
| 只设了 `HTTP_PROXY` | Hugging Face 全是 HTTPS 地址，要设 `HTTPS_PROXY`（两个都设也行） |
| 想用 SOCKS5 | huggingface_hub 1.x 用 httpx，要装 `pip install "httpx[socks]"`，写 `socks5://USER:PASS@HOST:PORT`；2.x 换成了 httpx2，对应装 `httpx2[socks]`。Xet 那部分走不走 SOCKS5 我们没测，建议直接买 HTTP 协议的 IP |
| 内网的服务不想走代理 | 设 `NO_PROXY=localhost,127.0.0.1,.internal.example` |
| macOS、Windows 上没设变量也走了代理 | Python 会读系统代理设置；不想要就在系统里关掉，或者设 `NO_PROXY=*` |
| 下载大文件老超时 | 加大 `HF_HUB_DOWNLOAD_TIMEOUT`（默认 10 秒），例如 `export HF_HUB_DOWNLOAD_TIMEOUT=60` |

## 只想让 huggingface_hub 走代理，不影响别的程序

环境变量会影响同一个进程里所有读它的库。只想让 huggingface_hub 用代理，可以给它单独换一个 HTTP 客户端（huggingface_hub 1.x）：

```python
import httpx
from huggingface_hub import set_client_factory, hf_hub_download

def proxied_client() -> httpx.Client:
    return httpx.Client(
        proxy="http://USER:PASS@HOST:PORT",
        follow_redirects=True,
        timeout=None,
    )

set_client_factory(proxied_client)
hf_hub_download("hf-internal-testing/tiny-random-gpt2", "config.json")
```

注意：这只换掉 huggingface_hub 自己的请求。Xet 的大文件下载是 `hf_xet` 单独发的，它读的是环境变量，所以下载大模型时还是要设 `HTTPS_PROXY`。

## 先确认代理本身是通的

```bash
curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" -o /dev/null -w '%{http_code}\n' https://huggingface.co/api/models/gpt2
```

返回 `200` 说明账号密码、网络都没问题，再去查 Python 那边。返回 `407` 是代理账号密码不对。

## 报错对照

| 报错 | 原因 | 怎么办 |
|---|---|---|
| `ProxyError` 或带 `407` 的报错 | 代理账号密码不对，或特殊字符没编码 | 重新复制、编码后再试 |
| `ConnectError`、`ConnectTimeout` | 连不上代理的主机和端口 | 用上面的 curl 命令单独测 |
| 报缺少 `socksio` | 用了 `socks5://` 但没装 SOCKS 扩展 | 装扩展，或换 HTTP 协议的 IP |
| 小文件能下，大文件卡住 | 网络慢或超时太短 | 调大 `HF_HUB_DOWNLOAD_TIMEOUT`，必要时设 `HF_XET_HIGH_PERFORMANCE=1` |
| 需要登录的私有仓库 403 | 不是代理问题，是 token 权限 | `hf auth login` 或设 `HF_TOKEN` |

## 常见问题

**用动态 IP 还是静态 IP？** 只是下载公开模型，按流量计费的动态住宅就够。服务器在防火墙后、要求固定出口加白名单的，用静态 IP。

**`HF_ENDPOINT` 要改吗？** 不用。走代理时访问的还是官方地址，`HF_ENDPOINT` 保持默认。

**下载一个大模型会用掉多少流量？** 基本等于模型文件的大小，加上少量请求开销。按流量计费前先看一下仓库里文件的总大小。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，所有国家同价）和动态住宅流量（1.5 美元/GB 起，预付流量不过期）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。相关：[Ollama 拉模型走代理](../ollama-pull-proxy-zh/README.md)、[pip 设置代理](../pip-proxy-zh/README.md)。

*Facts checked 26 November 2026.*
