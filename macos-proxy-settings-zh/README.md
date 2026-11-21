# Mac 怎么设置系统代理并验证出口

打开苹果菜单 → 系统设置 → 网络，点你正在用的网络（Wi-Fi 或以太网）→ 详细信息 → 代理，打开「网页代理（HTTP）」和「安全网页代理（HTTPS）」，两项都填同一个代理服务器和端口，打开「代理服务器要求密码」填用户名和密码，点「好」。然后在 Safari 里打开 `https://ipinfo.io/json`，看到的 `ip` 和 `country` 是代理的，就说明生效了。注意：终端里的 curl、git、pip 不读系统代理，要单独设。本指南由 Proxy302 维护，我们也卖这些代理：静态 ISP 下单时选 HTTP，1 天 0.5 美元、30 天 7.99 美元，所有国家同价，30 天及以上的静态 IP 是你独享的。

## 要准备什么

| 项目 | 说明 |
|---|---|
| 服务器 | 代理的主机，IP 或域名 |
| 端口 | 纯数字 |
| 用户名、密码 | 代理的账密 |
| 协议 | 买的时候定的。系统代理和浏览器用 HTTP 最省事；Chrome 不支持带账密的 SOCKS5 |

Proxy302 的这几项在控制台「我的资产」里，静态 IP 的协议购买时就定了，不能切换。

## 步骤（macOS Ventura 及以后的「系统设置」）

1. 苹果菜单 →「系统设置」→「网络」。
2. 点正在用的网络服务（比如 Wi-Fi），再点「详细信息…」。
3. 左边选「代理」。
4. 打开「网页代理（HTTP）」，填服务器和端口；打开「代理服务器要求密码」，填用户名和密码。
5. 打开「安全网页代理（HTTPS）」，填同一个服务器、端口和账密。这一项的意思是「访问 HTTPS 网站时用哪个代理」，代理本身还是 HTTP 代理，不要在地址前面加 `https://`。
6. 底部「忽略这些主机与域的代理设置」里保留局域网地址，比如 `*.local, 169.254/16`。
7. 点「好」。

界面文字以你的系统版本为准，苹果官方说明见 [Change proxy settings on Mac](https://support.apple.com/guide/mac-help/change-proxy-settings-on-mac-mchlp2591/mac)。代理设置跟着网络服务走：Wi-Fi 设了，换成以太网不会生效。

## 验证出口

**浏览器里：** Safari 打开 `https://ipinfo.io/json`，看 `ip`、`country`、`org`。Chrome 也读系统代理，第一次可能再弹一个代理登录框，填同一组账密。

**终端里看系统代理有没有设上：**

```bash
scutil --proxy
networksetup -getwebproxy "Wi-Fi"
networksetup -getsecurewebproxy "Wi-Fi"
```

`scutil --proxy` 里 `HTTPEnable : 1`、`HTTPSEnable : 1`，后面跟着你的主机和端口，就是设上了。

**终端程序走代理要单独设：**

```bash
# 只对当前终端窗口有效
export https_proxy="http://USER:PASS@HOST:PORT"
export http_proxy="http://USER:PASS@HOST:PORT"
curl -sS https://ipinfo.io/json

# 不设变量，单次指定
curl -sS -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

USER、PASS、HOST、PORT 是占位符。账密里有 `@`、`:` 这类字符要做 URL 编码。

## 用命令行一次设好（可选）

```bash
networksetup -setwebproxy "Wi-Fi" HOST PORT on USER PASS
networksetup -setsecurewebproxy "Wi-Fi" HOST PORT on USER PASS
# 关掉
networksetup -setwebproxystate "Wi-Fi" off
networksetup -setsecurewebproxystate "Wi-Fi" off
```

可能要输入管理员密码。密码会留在 shell 历史里，用完可以清掉，或者直接在系统设置里填。

## 哪些程序会走系统代理

| 程序 | 是否走系统代理 |
|---|---|
| Safari、Chrome、Edge | 走 |
| Firefox | 默认「使用系统代理设置」，可以在它自己的设置里改 |
| 多数基于 Chromium 的桌面 App | 一般走，具体看 App |
| curl、git、pip、npm、Homebrew | 不走，要设环境变量或各自的配置 |
| Claude 桌面版、Claude Code | 见[Claude 桌面版和 Claude Code 怎么设置代理](../claude-desktop-proxy-zh/README.md) |

## 常见问题排查

| 现象 | 原因 | 怎么办 |
|---|---|---|
| 网页全打不开 | 端口错了，或把 SOCKS5 的 IP 填进了 HTTP 栏 | 核对端口和购买时的协议 |
| 反复弹密码框 | 账密不对 | 重新复制粘贴，先用 curl 测 |
| ipinfo 显示的还是自己的 IP | 设在了另一个网络服务上，或浏览器有自己的代理设置 | 在正在用的网络里设；查浏览器设置 |
| 只有终端不走代理 | 终端不读系统代理 | 设 `https_proxy` 环境变量 |
| 国家不对 | 同时开着其他全局网络工具 | 只留一层出口 |
| 昨天好的今天不行 | IP 到期被收回 | 提前续费或开自动续费 |

## 常见问题

**HTTP 和 HTTPS 两栏要填不一样的吗？** 不用，填同一个代理。

**能填 SOCKS 代理吗？** 系统设置里有「SOCKS 代理」一栏，但 Chrome 不支持带账密的 SOCKS5，浏览器为主就用 HTTP。

**只想让一个浏览器走代理？** 用浏览器的独立配置或扩展，不动系统设置，见[Chrome 单独一个窗口用代理](../chrome-proxy-one-window/README.md)（英文）。

**有免费试用吗？** 没有。最便宜的测试是一个静态 ISP 用 1 天，0.5 美元。

## 相关

- [Windows 11 怎么设置代理并验证出口](../windows-proxy-settings-zh/README.md)
- [代理 IP 给的是「IP:端口:账号:密码」，在各种软件里怎么填](../proxy-format-zh/README.md)
- [Homebrew 怎么走代理](../homebrew-proxy-zh/README.md)
- [Chrome 提示代理服务器出现问题怎么办](../err-proxy-connection-failed-zh/README.md)
- [Proxy302 使用教程](https://proxy302.com/zh/guides/)
- [Proxy302 价格](https://proxy302.com/zh/pricing/)

*事实核对于 2026 年 11 月 21 日。*
