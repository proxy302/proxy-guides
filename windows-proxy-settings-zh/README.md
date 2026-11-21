# Windows 11 怎么设置代理并验证出口

打开「设置」→「网络和 Internet」→「代理」，在「手动设置代理」下点「使用代理服务器」旁边的「设置」，打开开关，填「代理 IP 地址」和「端口」，勾上「请勿将代理服务器用于本地(Intranet)地址」，点「保存」。Windows 这个界面没有用户名密码栏：代理要账密时，Edge 或 Chrome 第一次打开网页会弹出登录框，填进去即可。然后在浏览器里打开 `https://ipinfo.io/json`，看到代理的 IP 和国家就是生效了；命令行里的 curl.exe 不读这个设置，要单独指定。本指南由 Proxy302 维护，我们也卖这些代理：静态 ISP 下单时选 HTTP，1 天 0.5 美元、30 天 7.99 美元，所有国家同价，30 天及以上的静态 IP 是你独享的。

## 要准备什么

| 项目 | 说明 |
|---|---|
| 代理 IP 地址 | 代理的主机，填 IP 或域名都行 |
| 端口 | 纯数字 |
| 用户名、密码 | 系统界面不填，浏览器弹框时填 |
| 协议 | 用 HTTP。这个界面按 HTTP 代理用；Chrome 和 Edge 也不支持带账密的 SOCKS5 |

Proxy302 的这几项在控制台「我的资产」里，静态 IP 的协议购买时就定了。

## 步骤

1. 「开始」→「设置」→「网络和 Internet」→「代理」。
2. 「手动设置代理」下，点「使用代理服务器」旁边的「设置」。
3. 在「编辑代理服务器」里打开「使用代理服务器」。
4. 填「代理 IP 地址」和「端口」。
5. 例外列表里填不走代理的地址，用分号隔开，比如 `*.local;192.168.*`。
6. 勾选「请勿将代理服务器用于本地(Intranet)地址」。
7. 点「保存」。

微软官方说明：[Use a proxy server in Windows](https://support.microsoft.com/en-us/windows/use-a-proxy-server-in-windows-03096c53-0554-4ffe-b6ab-8b1deee8dae1)。界面文字以你的系统版本为准。关掉时回到这里把开关关上。

## 验证出口

**浏览器里：** Edge 或 Chrome 打开 `https://ipinfo.io/json`，弹出代理登录框就填账密，然后看 `ip`、`country`、`org`。

**看系统代理有没有写进去**（PowerShell 或命令提示符）：

```powershell
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ProxyEnable
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ProxyServer
```

`ProxyEnable` 是 `0x1`、`ProxyServer` 是你的 `HOST:PORT`，就是设上了。

**命令行单独测代理：**

```powershell
# curl.exe 不读系统代理，直接带上代理参数
curl.exe -sS -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json

# PowerShell 自带命令，账密用弹窗输入
Invoke-RestMethod https://ipinfo.io/json -Proxy "http://HOST:PORT" -ProxyCredential (Get-Credential)
```

USER、PASS、HOST、PORT 是占位符。在 PowerShell 里一定要写 `curl.exe`，旧版 PowerShell 里单写 `curl` 指向的是另一个命令。

## 命令行工具怎么走代理

很多命令行工具读 `HTTPS_PROXY` 环境变量：

```powershell
# 只对当前 PowerShell 窗口有效
$env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
$env:HTTP_PROXY  = "http://USER:PASS@HOST:PORT"
```

用 `setx` 可以长期写入，但账密会明文存在用户环境变量里，自己的电脑再考虑。

另外 Windows 还有一套 WinHTTP 代理，部分系统服务和程序用它，和上面的设置是分开的。用 `netsh winhttp show proxy` 查看；需要时用管理员权限运行 `netsh winhttp set proxy HOST:PORT`，这一套同样不能填账密。

## 常见问题排查

| 现象 | 原因 | 怎么办 |
|---|---|---|
| 网页全打不开 | 端口错，或填了 SOCKS5 的 IP | 核对端口，用 HTTP 的 IP |
| 登录框反复弹出 | 账密不对 | 重新复制，先用 curl.exe 测 |
| 浏览器没弹框直接报错 | 浏览器扩展或策略接管了代理 | 关掉代理类扩展再试 |
| 命令行查到的是自己的 IP | curl.exe 不读系统代理 | 加 `-x` 或设环境变量 |
| 国家不对 | 同时开着其他全局网络工具，它可能接管系统代理 | 只留一层出口 |
| 昨天好的今天不行 | IP 到期被收回 | 提前续费或开自动续费 |

## 常见问题

**能不能把账密写进系统设置？** 这个界面没有地方填。浏览器弹框时填，或者在需要的程序里单独配置带账密的代理地址。

**只想让一个浏览器走代理？** 用浏览器扩展或单独启动参数，不动系统设置，见[Chrome 单独一个窗口用代理](../chrome-proxy-one-window/README.md)（英文）。

**系统代理会影响所有软件吗？** 不会。浏览器和部分软件读它，很多命令行工具和游戏不读。

**有免费试用吗？** 没有。最便宜的测试是一个静态 ISP 用 1 天，0.5 美元。

## 相关

- [Mac 怎么设置系统代理并验证出口](../macos-proxy-settings-zh/README.md)
- [Chrome 提示「未连接到互联网，代理服务器出现问题」怎么办](../err-proxy-connection-failed-zh/README.md)
- [代理 IP 给的是「IP:端口:账号:密码」，在各种软件里怎么填](../proxy-format-zh/README.md)
- [pip 怎么设置代理（带账号密码）](../pip-proxy-zh/README.md)
- [Proxy302 使用教程](https://proxy302.com/zh/guides/)
- [Proxy302 价格](https://proxy302.com/zh/pricing/)

*事实核对于 2026 年 11 月 21 日。*
