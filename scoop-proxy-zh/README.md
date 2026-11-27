# Windows 上 Scoop 怎么走带账号密码的代理？

装好 Scoop 以后，运行 `scoop config proxy USER:PASS@HOST:PORT` 就行，不用写 `http://`；之后 `scoop install`、`scoop update` 下载安装包、拉 bucket 都会走这个代理。还没装 Scoop 的，先下载安装脚本，再带上 `-Proxy` 和 `-ProxyCredential` 参数运行。取消用 `scoop config rm proxy`。密码里别带 `@` 和 `:`，Scoop 自己能用反斜杠转义，但它转给 git 的时候不会处理转义。本指南由 Proxy302 维护，我们也销售这些代理：静态 ISP IP 1 天 0.5 美元、30 天 7.99 美元，动态住宅 1.5 美元/GB 起。

*内容对照 Scoop 官方 wiki「Using Scoop behind a proxy」、ScoopInstaller/Install 的说明和 Scoop 源码（`lib/download.ps1`、`lib/core.ps1`）核对，没有在 Windows 上实跑。*

## 已经装了 Scoop

```powershell
scoop config proxy USER:PASS@HOST:PORT
scoop config proxy          # 查看当前设置
scoop install 7zip          # 试一下
```

- 格式是 `[用户名:密码@]主机:端口`，前面不加 `http://`，Scoop 会自己按 HTTP 代理处理。所以买 IP 时选 HTTP 协议。
- USER、PASS、HOST、PORT 是占位符。
- 设置存在 Scoop 的配置文件里（一般是 `~\.config\scoop\config.json`），是明文，别把这个文件分享出去。

几个特殊写法（来自官方 wiki）：

| 写法 | 意思 |
|---|---|
| `USER:PASS@HOST:PORT` | 指定代理和账号密码 |
| `currentuser@default` | 用系统「Internet 选项」里的代理，加当前 Windows 登录用户的凭据 |
| `none` | 绕开系统代理，直接连 |
| 不设置（`scoop config rm proxy`） | 等同 `default`：用「Internet 选项」里的代理，不带凭据 |

## 还没装 Scoop

官方安装脚本支持代理参数。PowerShell（不用管理员）里：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
$cred = Get-Credential   # 弹窗里填代理的用户名和密码
irm get.scoop.sh -Proxy 'http://HOST:PORT' -ProxyCredential $cred -OutFile install.ps1
.\install.ps1 -Proxy 'http://HOST:PORT' -ProxyCredential $cred
```

第一条是 Scoop 官方安装步骤里的执行策略设置，只对当前用户生效。装完以后再按上一节 `scoop config proxy` 设一遍，安装参数不会保存成 Scoop 的设置。

## Scoop 里各部分怎么走代理

| 部分 | 怎么走 |
|---|---|
| 下载安装包（默认下载器） | 用 `scoop config proxy` 的设置 |
| 开了 aria2 加速下载 | Scoop 把同一个代理地址和账号密码通过 `--all-proxy`、`--all-proxy-user`、`--all-proxy-passwd` 传给 aria2 |
| `scoop bucket add`、`scoop update` 拉 bucket（git） | Scoop 在调用 git 时临时设 `HTTPS_PROXY`、`HTTP_PROXY` 为你的代理设置 |

最后一行就是密码别带 `@`、`:` 的原因：Scoop 自己解析时认反斜杠转义（`p\@ss`），但交给 git 的是原样的字符串，git 看到反斜杠不会当转义处理，结果就是安装能成、拉 bucket 失败。最简单的办法是用只含字母和数字的密码。git 本身的代理设置见 [git 怎么走代理](../git-proxy/README.md)。

## 先确认代理本身可用

PowerShell 里：

```powershell
$cred = Get-Credential
Invoke-RestMethod https://ipinfo.io/json -Proxy 'http://HOST:PORT' -ProxyCredential $cred
```

返回的 `country` 是你买的国家，就说明 IP 和账号密码没问题。也可以用 curl（Windows 10 以后自带 `curl.exe`）：

```powershell
curl.exe -sS -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
```

更多写法见 [PowerShell Invoke-WebRequest 走代理](../powershell-invoke-webrequest-proxy/README.md)。

## 常见问题排查

| 现象 | 原因 |
|---|---|
| 下载报 407 | 账号密码错 |
| 安装正常，`scoop update` 或 `scoop bucket add` 失败 | 密码里有 `@`、`:` 用了转义，git 那一步解析错；或者 git 自己另外配了代理 |
| 设置了代理却还是直连 | 设成了 `none`，用 `scoop config proxy` 看一下 |
| 连接超时 | 主机、端口错，或者买的是 SOCKS5 的 IP（Scoop 按 HTTP 代理用） |
| 昨天还能用，今天不行 | 静态 IP 到期被释放了 |

## 常见问题

**系统已经设了代理，Scoop 还要单独设吗？** 系统代理不需要账号密码时，Scoop 默认就会用它；需要账号密码的，用 `scoop config proxy USER:PASS@HOST:PORT` 明确写上更省事。系统代理怎么设见 [Windows 11 怎么设置代理](../windows-proxy-settings-zh/README.md)。

**只是装软件，用动态还是静态？** 下载安装包用动态住宅（1.5 美元/GB 起）或动态机房（0.4 美元/GB）就够；公司防火墙按来源 IP 放行时用静态 IP。

**有免费试用吗？** 没有。最便宜的测试是 1 个静态 IP 用 1 天，0.5 美元。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP（1 到 90 天，下单时选 HTTP 或 SOCKS5，所有国家同价）和动态住宅、动态机房流量。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/pricing](https://proxy302.com/pricing/)。

*Facts checked 27 November 2026.*
