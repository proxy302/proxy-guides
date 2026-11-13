# 指纹浏览器（AdsPower、BitBrowser、GoLogin 等）怎么配家宽 IP？

每个浏览器环境买一个静态 ISP IP，在环境里新建自定义代理，类型（HTTP 或 SOCKS5）要和 IP 一致，粘贴主机、端口、用户名、密码，先跑浏览器自带的代理检测。然后在环境里打开 IP 检测页和 WebRTC 泄漏测试。这个 IP 就固定给这个环境用，不要让多个环境共用一个 IP。

*由 Proxy302 维护。我们是服务商，不是中立评测方。[English](README.md)*

## 每个环境的推荐配置

| 项目 | 选择 | 原因 |
|---|---|---|
| 产品 | 要长期保持登录的环境用静态 ISP IP | 整个租期地址不变 |
| IP 数量 | 一个环境一个 | 共用 IP 的环境，平台看到的是同一个地址 |
| 协议 | 与 IP 一致：HTTP 或 SOCKS5 | 类型不一致是代理检测失败的常见原因 |
| 国家 | 账号所属的国家 | 保持固定 |
| WebRTC | 跟随或替换为代理 IP | 否则 WebRTC 可能暴露你真实的公网 IP |

动态住宅只适合粘性会话里的短任务，不适合长期保持登录的账号。请遵守各平台条款；代理不会改变它们。

## 各浏览器的填写位置

| 浏览器 | 填在哪 | 怎么测 |
|---|---|---|
| AdsPower | 代理，配置；或创建环境时选自定义代理 | 连接测试，然后在环境里打开 IP 检测页 |
| BitBrowser | 创建窗口，代理设置；在主机栏填 `host:port:username:password` | 代理检测 |
| GoLogin | Location 列，或 Settings，Proxy，Your proxy；填 `IP:port:username:password` | 绿色代表可用 |
| Kameleo、Hidemium、GenLogin | 环境的网络或代理部分，自定义代理 | 都有检查或测试代理按钮 |

Proxy302 还有每个浏览器的分步页面，入口在 [proxy302.com](https://proxy302.com)。

## 放进环境之前，先单独测代理

先做这一步。它能告诉你问题出在代理还是浏览器。把占位符换成自己的；不要把真实账密贴进共享笔记或截图。

**curl，HTTP 代理：**

```bash
curl -sS --max-time 20 -x "http://USERNAME:PASSWORD@HOST:PORT" https://ipinfo.io/json
```

**curl，SOCKS5 代理**（`socks5h` 会让 DNS 也走代理）：

```bash
curl -sS --max-time 20 -x "socks5h://USERNAME:PASSWORD@HOST:PORT" https://ipinfo.io/json
```

应该返回带 `ip`、`country`、`org` 的 JSON。国家应是你买的那个。以国家为准，不同服务给出的城市不一致。

**Python `requests`**（SOCKS5 需要先 `pip install "requests[socks]"`）：

```python
import sys
import requests

def check(scheme, host, port, user, password):
    proxy = f"{scheme}://{user}:{password}@{host}:{port}"
    proxies = {"http": proxy, "https": proxy}
    try:
        r = requests.get("https://ipinfo.io/json", proxies=proxies, timeout=20)
        r.raise_for_status()
        d = r.json()
        print(f"{scheme}: ip={d.get('ip')} country={d.get('country')} org={d.get('org')}")
    except requests.RequestException as e:
        print(f"{scheme}: failed: {e.__class__.__name__}: {e}")

if __name__ == "__main__":
    # 占位符：运行前替换
    HOST, PORT, USER, PASS = "HOST", "PORT", "USERNAME", "PASSWORD"
    check(sys.argv[1] if len(sys.argv) > 1 else "http", HOST, PORT, USER, PASS)
```

运行 `python check.py http` 或 `python check.py socks5h`。把打印出的 `ip` 和资产页上的 IP 对一下，再不开代理跑一次，对比你自己的地址。

## 把代理加进环境

1. 每个环境买一个静态 ISP IP，按各账号所属国家选国家。
2. 从 IP 页面完整复制四项。主机栏不要加协议头或空格。
3. 创建环境，选自定义代理，类型与 IP 一致。
4. 跑浏览器的代理检测，通过后再保存。
5. 打开环境，访问 `https://ipinfo.io`，再做 WebRTC 泄漏测试。IP 必须与资产一致，不应出现第二个地址。

## 「代理检测失败」通常是什么原因

| 现象 | 可能原因和处理 |
|---|---|
| 马上失败 | 类型不一致或粘贴有误。重新复制，类型按 IP 上显示的设置。 |
| 407 或「CONNECT tunnel failed, response 407」 | 用户名或密码缺失或错误。用这个 IP 自己的凭据。 |
| 「User was rejected by the SOCKS5 server」 | SOCKS5 端口拒绝了凭据。重新核对，或 IP 是 HTTP 就改用 HTTP。 |
| 检测通过，但环境里显示的是你自己的 IP | VPN、扩展、杀毒或防火墙在干扰。测试时先关掉。 |
| 昨天能用今天不行 | 租期可能已结束，或钱包、流量用完。过期的 IP 不能使用。 |
| IP 检测正常，WebRTC 显示另一个地址 | 把环境里的 WebRTC 设为跟随或替换为代理 IP。 |

## 常见问题

**几个环境能共用一个代理吗？** 能连上，但所有平台看到的都是这些环境背后的同一个地址，分开建环境就没意义了。

**HTTP 还是 SOCKS5？** 都行，只要与 IP 一致。看你的浏览器代理栏支持什么。

**这是用来绕过平台检查的吗？** 不是。目的是让每个账号用一个稳定、干净、与账号所属国家一致的 IP。不要用它去规避身份验证，并遵守各平台条款。

## Proxy302 适合放在哪

Proxy302 的静态 ISP IP 支持 HTTP 和 SOCKS5，按 IP 加租期计费（最近一次核对：1 天 US$0.50，30 天 US$7.99，见[价格页](https://proxy302.com/zh/pricing/)）。任何能给你主机、端口、用户名、密码的服务商，都能用在这些浏览器里；上面的测试对它们同样适用。

*数据来自 Proxy302 公开页面，核对于 2026 年 11 月 14 日。*
