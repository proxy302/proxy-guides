# Claude 用什么 IP？（ChatGPT 同理）

每个人用一个稳定的 IP，国家要在服务官方支持的范围内，并且是你实际所在的地方。对已登录的账号，固定的（静态）ISP IP 是比较实用的选择，因为每次登录都来自同一个地址。已登录使用时避免轮换 IP 和多人共用的公共出口。IP 无法让不受支持地区的使用变得被允许，也没有任何 IP 能保证账号永远不被限制。

*由 Proxy302 维护。我们是服务商，不是中立评测方。[English](README.md)*

## 先说清楚：规则不只看 IP

**Claude。** Anthropic 的《支持地区政策》列出了 Claude.ai、Claude Code 和 API 提供服务的国家。按我们的理解，该政策针对人实际所在的位置，也涵盖在不受支持地区注册、总部设在那里或由那里控股的公司。注册还需要一个来自受支持地区的手机号；VoIP、Google Voice 和 App 生成的号码会被拒绝。代理改变不了这些。

**ChatGPT。** OpenAI 只在其支持列表上的国家提供 ChatGPT，并提示在列表之外访问可能导致账号被封禁或暂停。API 对不受支持的位置会返回 403「Country, region, or territory not supported」。

所以真正该问的不是「哪个 IP 能解锁」，而是「哪个 IP 能让我合规的使用有一个稳定的网络地址」。买之前先查官方列表。

## 固定 IP 能帮上什么

- **一致性。** 每次登录都是同一个登记在互联网服务商名下的地址，而不是会话之间变化的出口，或很多人共用的其他全局网络工具。
- **团队。** 每人一个地址。Claude Enterprise 可以限制只允许白名单 IP 段访问；静态 IP 让远程成员有一个管理员能加进白名单的地址。名单由你们公司自己的管理员添加；Proxy302 不替你登记，和 Anthropic 没有对接，我们的 IP 不在任何白名单里，也不能让账号免去身份或手机验证。
- **Claude Code。** 它读取标准代理环境变量，但不支持 SOCKS，所以要用 HTTP 代理。

## 选哪种类型

| 类型 | 适合 | 说明 |
|---|---|---|
| AI 订阅 ISP（静态，原生 ISP 段） | 日常使用 Claude 或 ChatGPT，每人一个 IP | 30 天租期内固定 |
| 普通静态 ISP IP | 同样用途的预算方案 | 租期内固定，按 IP 加租期计费 |
| 动态住宅 | 不适合已登录账号 | 地址一直在变 |
| 数据中心 | 不建议用于已登录的个人账号 | 网站看到的是机房地址 |

截至最近一次核对 Proxy302 产品页：Claude ISP 页面描述为 dedicated native ISP IP，30 天 15 PTC（1 PTC = US$1），GPT / Grok ISP 30 天 10 PTC；每个账号在每个套餐里的第一个 30 天 IP 半价。普通静态 ISP IP 在 [proxy302.com/zh/pricing/](https://proxy302.com/zh/pricing/) 上是 30 天 US$7.99。Proxy302 说明它不能保证账号永远不会被封。

*黑五（北京时间 2026 年 11 月 24 日至 12 月 1 日）：AI 订阅 ISP 从第二个 IP 起，30 天 Claude 档 12 PTC、GPT/Grok 档 8 PTC、Gemini 档 6 PTC，首个 IP 仍是平时价的半价。本页其他地方写的是平时价，当天价格以[定价页](https://proxy302.com/zh/pricing/)为准。*<!-- bf2026 -->

## 操作步骤

1. **选一个在官方列表上、且是你实际工作所在的国家。** 不要在国家之间来回跳。
2. **每人买一个 IP。** 如果要给 Claude Code 用，选 HTTP。浏览器扩展、系统代理、iPhone 一般也选 HTTP；指纹浏览器环境适合 SOCKS5。
3. **从控制台「我的资产」复制** 主机、端口、用户名、密码。
4. **只放在一个地方。** 一个浏览器（通过扩展）、一个浏览器环境，或系统代理。只保留一层：不要再叠其他全局网络工具，也不要再叠第二个代理。
5. **检查出口。** 在同一个浏览器里打开 `https://ipinfo.io/json`，国家必须是你买的那个。
6. **再登录。** 此后这个账号一直用这个 IP。

### 让 Claude Code 走 HTTP 代理

```bash
export HTTPS_PROXY="http://USERNAME:PASSWORD@HOST:PORT"
claude
# 在 Claude Code 里运行 /status，看 Proxy 一行
```

如果 URL 解析不了，比如缺了 `http://` 前缀，Claude Code 会在启动时停下。上面是占位符，不是真实账密。

### iPhone

设置，无线局域网，点网络旁的 (i)，配置代理，手动。填 HTTP IP 的服务器和端口，打开「鉴权」，填用户名和密码。它对 Safari 和遵循系统代理的 App 生效。

## 用之前先测

不需要账号，就能测一个位置是否被接受。通过代理，不带密钥请求 API 根地址：

```bash
curl -sS -o /dev/null -w "%{http_code}\n" -x "http://USERNAME:PASSWORD@HOST:PORT" https://api.anthropic.com/v1/models
curl -sS -x "http://USERNAME:PASSWORD@HOST:PORT" https://api.openai.com/v1/models | head -c 300
```

返回 401 要求提供密钥，说明该位置在网络层面被接受。对 OpenAI 来说，返回 403 且带 `unsupported_country_region_territory` 就是不被接受。这只测连接，不代表你的账号状态。

## 让账号保持稳定的习惯

- 一人一个账号。不共享登录或 API 密钥。
- 同一个账号不要在代理、其他全局网络工具和家里网络之间混用。
- 不要让多个人走同一个 IP。
- 被封后不要另开新账号，走官方申诉。
- 代理不能替你付款：你仍然需要服务方接受的付款方式。

## 快速排错

| 现象 | 检查 |
|---|---|
| 「你所在的地区不可用」 | 在同一个浏览器里打开 ipinfo.io。出口是否在受支持国家？浏览器是否真的在用代理？ |
| Claude Code 连不上 | IP 是不是 SOCKS5，或 URL 是否缺 `http://`？用 HTTP 的 IP 和完整 URL。 |
| 手机验证失败 | 代理帮不上；Anthropic 只接受来自受支持地区的号码。 |
| 总被退出登录 | 是否在代理、其他全局网络工具和家里网络之间切换，或一个 IP 多人共用？ |
| 开了白名单后团队被锁在外面 | 你们的管理员有没有在 Claude 管理设置里把这个静态 IP 加进组织的白名单？ |

## Proxy302 适合放在哪

Proxy302 卖面向 Claude、GPT、Gemini 的 AI 订阅 ISP 套餐、普通静态 ISP IP、动态住宅流量和数据中心 IP。见[价格页](https://proxy302.com/zh/pricing/)和[官网](https://proxy302.com)。其他服务商也卖静态 IP；可按国家、计费单位、协议，以及它们自己说明做不到什么来比较。

*数据来自 Proxy302 公开页面及其引用的官方政策，核对于 2026 年 11 月 14 日。政策和价格会变，请以原始页面为准。*
