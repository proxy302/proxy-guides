# Firefox 怎么单独设置带账号密码的代理？

打开 Firefox「设置」→「常规」→ 拉到最下面「网络设置」→「设置…」，选「手动配置代理」，在「HTTP 代理」填主机和端口，勾上「也将此代理用于 HTTPS」，确定；第一次打开网页时 Firefox 会弹框要用户名和密码，填进去即可。这样只有 Firefox 走代理，系统和别的程序不受影响。要注意：Firefox 的手动设置不会给 SOCKS5 代理发送账号密码，带账密的代理请买 HTTP 类型。本指南由 Proxy302 维护，我们也销售这些代理；静态 ISP 1 天 0.5 美元、30 天 7.99 美元，下单时选 HTTP 或 SOCKS5。

## 设置步骤

1. **准备好四项信息：** 主机（HOST）、端口（PORT）、用户名（USER）、密码（PASS），在订单页复制。确认这个 IP 的协议是 HTTP。
2. **打开连接设置。** 地址栏输入 `about:preferences`，在「常规」页最下面找到「网络设置」，点「设置…」。
3. **选「手动配置代理」**（英文界面叫 Manual proxy configuration）。
4. **「HTTP 代理」填 HOST，「端口」填 PORT。** 勾选「也将此代理用于 HTTPS」（Also use this proxy for HTTPS）。SOCKS 主机一栏留空。
5. **「不使用代理」**（No proxy for）里填不想走代理的地址，用逗号分隔，比如 `localhost, 127.0.0.1, .corp.example`。
6. **点「确定」**，打开任意网页，弹出「需要身份验证」的框，填 USER 和 PASS。勾选保存密码的话，下次不用再输。
7. **验证出口。** 打开 `https://ipinfo.io/json`，看 `ip`、`country`、`org` 是不是你买的那个 IP 和国家。

不同版本 Firefox 的中文字样可能略有出入，括号里的英文名对照着找就行。

## 每一栏是什么意思

| 界面上的选项 | 作用 | 用带账密的 HTTP 代理时怎么填 |
|---|---|---|
| 不使用代理 / 自动检测 / 使用系统代理设置 | 不走代理、按网络自动找、跟随系统设置 | 不选这几个 |
| 手动配置代理 | 自己填代理地址 | 选它 |
| HTTP 代理 + 端口 | 访问 http 网站用的代理 | HOST、PORT |
| 也将此代理用于 HTTPS | https 网站也用同一个代理 | 勾上 |
| SOCKS 主机 + SOCKS v4 / v5 | SOCKS 代理 | 留空 |
| 自动代理配置的 URL（PAC） | 用脚本决定走不走代理 | 不用 |
| 不使用代理（No proxy for） | 这些地址直连 | 本地地址、公司内网域名 |
| 如果密码已保存，不提示身份验证 | 有保存的密码就不再弹框 | 想省事可以勾 |
| 使用 SOCKS v5 时代理 DNS 查询 | SOCKS5 时由代理解析域名 | 用 HTTP 代理时不相关 |

## 为什么 SOCKS5 带账密不行

Firefox 的手动设置里只有 SOCKS 主机和端口，没有用户名密码栏，连接 SOCKS5 代理时也不会弹框要密码。Mozilla 的 bug 122752 记录的就是这个需求，到现在还没有在设置界面里实现。有两条路：

- **买 HTTP 类型的 IP**（推荐）。Proxy302 的静态 IP 下单时选协议，一个 IP 只有一种协议。
- **用浏览器扩展。** Firefox 给扩展的代理接口可以带 SOCKS 用户名和密码，一些代理扩展用的就是这个接口，具体以扩展自己的说明为准。

## 用 about:config 核对

在地址栏打开 `about:config`，搜索 `network.proxy`，能看到设置界面背后的值：

| 键 | 手动配置 HTTP 代理时的值 |
|---|---|
| `network.proxy.type` | `1`（手动）；`0` 是不走代理，`5` 是跟随系统 |
| `network.proxy.http` / `network.proxy.http_port` | HOST / PORT |
| `network.proxy.share_proxy_settings` | `true`（对应「也将此代理用于 HTTPS」） |
| `network.proxy.no_proxies_on` | 不走代理的地址列表 |
| `network.proxy.socks_remote_dns` | 只在 SOCKS 时有用；Firefox 128 起 SOCKS5 默认由代理解析域名 |

一般不需要在这里改，用来排查「设置到底生效没有」很方便。

## 不同账号用不同代理：分开配置文件

代理设置是跟着 Firefox 配置文件（profile）走的。地址栏打开 `about:profiles`，「创建新配置文件」，每个配置文件各填各的代理、各存各的登录状态。在不同配置文件之间切换，不会互相影响。一个账号对应一个配置文件、一个静态 IP，最清楚。

## 常见问题排查

| 现象 | 多半是 | 怎么办 |
|---|---|---|
| 页面显示「代理服务器拒绝连接」 | 主机或端口填错，或这个 IP 已到期 | 对照订单页重填；到期就续费 |
| 页面显示「找不到代理服务器」 | 主机名拼错 | 重新复制主机 |
| 账号密码框反复弹出 | 用户名或密码不对 | 从订单页重新复制，注意别带空格 |
| 填在 SOCKS 主机里连不上 | Firefox 不给 SOCKS5 发账密 | 换成 HTTP 类型的 IP，填在 HTTP 代理一栏 |
| ipinfo 显示的还是自己的 IP | 选的是「使用系统代理设置」或改完没点确定 | 重新打开设置确认选中「手动配置代理」 |
| 只有部分网站不走代理 | 写进了「不使用代理」列表 | 检查这一栏 |

## 常见问题

**Firefox 设了代理，Chrome 会跟着走吗？** 不会。手动配置只对 Firefox 生效；选「使用系统代理设置」时才跟系统走。

**电脑上还开着其他全局网络工具，要关吗？** 要关。两层叠在一起，出口是哪个就说不准了，排查也麻烦。

**WebRTC 会不会暴露真实 IP？** 可以在 `about:config` 里把 `media.peerconnection.enabled` 改成 `false` 关掉 WebRTC，检查方法见[指纹浏览器 WebRTC 泄露怎么查](../webrtc-leak-antidetect-zh/README.md)。

**有免费试用吗？** 没有。最便宜的测试是一个静态 ISP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（下单时选 HTTP 或 SOCKS5，1 天 0.5 美元、30 天 7.99 美元，每个国家同价）和按流量计费的动态住宅（1.5 美元/GB 起）。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/zh/pricing](https://proxy302.com/zh/pricing/)。只想让 Chrome 某个窗口走代理，见 [Chrome 单独一个窗口用代理](../chrome-proxy-one-window/README.md)（英文）。

*Facts checked 23 November 2026.*
