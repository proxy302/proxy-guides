# Claude 桌面版和 Claude Code 怎么设置代理？

Claude Code 读标准的代理环境变量：启动前设好 `HTTPS_PROXY=http://USER:PASS@HOST:PORT`，或者写进 `~/.claude/settings.json` 的 `env`，它不支持 SOCKS 代理。桌面版的 Code 标签里，本地会话同样可以用 settings.json 或「保存的本地环境」来设这些变量。桌面版聊天窗口本身怎么选代理，Anthropic 只在企业第三方部署版（Claude Desktop on 3P）的文档里写明「默认跟随操作系统的代理设置」；个人版没有找到单独说明，以官方文档为准。

*本文由 Proxy302 维护，我们出售文中提到的 HTTP 代理 IP。*

这里讲的是网络配置：公司内网要求走代理、防火墙只放行固定出口，或者需要给管理员一个固定 IP 加白名单。代理只改变网络路径，不改变 Anthropic 的使用政策，也不改变 Claude 提供服务的国家和地区。

## 先看你用的是哪一个

| 你在用 | 怎么设代理 | 官方出处 |
|---|---|---|
| Claude Code 命令行 | 环境变量，或 `~/.claude/settings.json` 的 `env` | [Enterprise network configuration](https://code.claude.com/docs/en/network-config) |
| 桌面版 Code 标签的本地会话（claude.ai 登录） | settings 文件的 `env`，或保存的本地环境里的环境变量 | 同上，及 [Desktop 文档](https://code.claude.com/docs/en/desktop) |
| 桌面版 Code 标签的 SSH 会话 | 写在远程机器的 `~/.claude/settings.json` 里 | Desktop 文档 |
| 企业第三方部署的桌面版 | 默认跟随系统代理（含 PAC），管理员可固定 | [Network proxy](https://claude.com/docs/third-party/claude-desktop/network-proxy) |
| 个人版桌面聊天窗口 | 文档未单独说明 | 以官方文档为准 |

## Claude Code 命令行

macOS、Linux：

```bash
export HTTPS_PROXY="http://USER:PASS@HOST:PORT"
claude
```

Windows PowerShell：

```powershell
$env:HTTPS_PROXY = "http://USER:PASS@HOST:PORT"
claude
```

想一直生效，写进 `~/.claude/settings.json`：

```json
{
  "env": {
    "HTTPS_PROXY": "http://USER:PASS@HOST:PORT"
  }
}
```

官方文档里几条要点：

- 读取顺序是 `https_proxy`、`HTTPS_PROXY`、`http_proxy`、`HTTP_PROXY`，用第一个有值的，设一个就够。
- `NO_PROXY` 填要直连的地址，空格或逗号分隔，填 `*` 表示全部直连。
- 账密直接写进 URL，走 Basic 认证；NTLM、Kerberos 这类认证，官方建议改用 LLM 网关。
- 不支持 SOCKS 代理，所以买 IP 时要选 HTTP 协议。
- 变量在启动时读一次，改了要重开。URL 解析不了（比如漏了 `http://`），Claude Code 会直接停止启动并报出变量名。
- 后台代理（`claude agents`、`--bg`）由一个共用的后台进程托管，shell 里 export 的变量不一定能传到，官方建议写进 settings.json。

验证：在 Claude Code 里输入 `/status`，看 Proxy 一行显示的地址；需要细节时用 `claude --debug` 启动，日志在 `~/.claude/debug/` 下。

## 桌面版 Code 标签

**本地会话**：两种写法任选。一是上面的 `~/.claude/settings.json`；二是在桌面版里新建或编辑一个保存的本地环境，在「Environment variables」里一行写一个 `HTTPS_PROXY=http://USER:PASS@HOST:PORT`，保存后从这个环境新开会话。注意：如果桌面版启动时的环境里已经有同名变量，settings 文件里的值会被忽略，这时用保存的本地环境来设。

**SSH 会话**：远程 Linux 机器上，代理要写进那台机器的 `~/.claude/settings.json`。写在 `~/.zshrc`、`~/.bash_profile` 里的 export 不会被会话读到，表现为会话卡住或连接 API 失败。

## 桌面版聊天窗口

Anthropic 面向企业第三方部署版的 Network proxy 文档写明：

- 没有额外配置时，应用跟随操作系统的代理设置，包括 PAC 脚本和自动检测。
- 系统给出的如果是 SOCKS 代理，应用会跳过。
- 没有交互式的代理登录：需要用户名和密码的代理，请求会失败。官方建议在本机跑一个本地转发代理，由它去向你的代理服务器认证，再把系统代理指向 `http://127.0.0.1:端口`。
- 管理员可以用 `egressProxyUrl` 或 `egressProxyPacUrl` 固定代理（需 Claude Desktop 1.44121.1 或更高），这两个键不接受 SOCKS 地址，也不接受带账密的地址。
- 应用自身的更新检查和下载只跟随系统代理。

个人版聊天窗口的代理行为，我们在官方文档里没有找到单独说明，以官方文档为准。

## 常见错误

| 现象 | 原因和处理 |
|---|---|
| Claude Code 一启动就报代理变量有误 | URL 缺 `http://`，或密码里的 `@`、`:`、`#` 没做百分号编码 |
| `/status` 里看不到 Proxy，或标为无效 | 变量没传进来，或写法解析不了；检查拼写后重开 |
| 填了 `socks5://` 后连不上 | 不支持 SOCKS，换 HTTP 协议的 IP |
| 桌面版 Code 标签不走代理，命令行却正常 | 桌面版启动环境里已有同名变量；改用保存的本地环境 |
| 公司代理报证书错误 | 代理在检查 TLS；把它的根证书装进系统证书库，或设 `NODE_EXTRA_CA_CERTS` |
| 3P 桌面版提示 Can't reach | 系统代理需要账密或不可达；按上文改用本地转发代理 |

## 常见问题

**Claude Code 支持 SOCKS5 吗？** 不支持，官方文档写明不支持 SOCKS 代理。

**`HTTPS_PROXY` 和 `HTTP_PROXY` 都要设吗？** 不用，Claude Code 取第一个有值的变量。

**给公司白名单用，买哪种 IP？** 静态 IP，地址在租期内不变，白名单才一直有效。名单由你们公司自己的管理员添加；Proxy302 不替你登记，和 Anthropic 没有对接，我们的 IP 不在任何白名单里，也不能让账号免去身份或手机验证。

## Proxy302 在哪里

Proxy302 出售静态 ISP IP，下单时选 HTTP 协议即可用于 Claude Code，各国同价：1 天 US$0.50，30 天 US$7.99，90 天 US$23.97，可按 IP 开自动续费。30 天及以上的静态 IP 为买家独享。价格见 [proxy302.com/pricing](https://proxy302.com/pricing/)，中文站 [proxy302.com/zh](https://proxy302.com/zh/)。

*黑五（北京时间 2026 年 11 月 24 日至 12 月 4 日）：静态 ISP 60 天 US$14.00、90 天 US$18.00（1 到 30 天不变）。本页其他地方写的是平时价，当天价格以[定价页](https://proxy302.com/zh/pricing/)为准。*<!-- bf2026 -->

*事实核对于 2026 年 11 月 17 日。*
