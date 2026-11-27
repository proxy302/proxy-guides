# 团队几个人各用自己的 Claude、ChatGPT 账号，怎么给每人配一个双 ISP IP？

做法是一人一个账号、一人一个 AI 订阅专用 ISP IP：按每个人实际所在、而且是服务官方支持的国家挑城市，Claude 用户买 Claude 档，ChatGPT 用户买 GPT/Grok 档；购买页标了「双 ISP」的城市，新单优先分配双 ISP 号段。IP 只配在这个人自己的设备上，不共用账号，也不把一个 IP 分给几个人。价格：Claude 档 15 PTC/30 天，GPT/Grok 档 10 PTC/30 天，每档第一个 IP 半价（1 PTC = 1 美元）。本指南由 Proxy302 维护，我们也销售这些代理。先把边界说清楚：固定 IP 只是让每个人的连接稳定、出了问题好查；它不会让身在服务商支持地区以外的人变成可以用，不改变 Anthropic、OpenAI 的使用政策和支持国家名单，也保证不了账号永远不被限制。

黑五（11 月 24 日至 12 月 4 日，UTC+8）：AI 订阅专用 ISP 从第二个 IP 起，Claude 档 12 PTC、GPT/Grok 档 8 PTC（每 30 天），每档第一个 IP 仍是平时价的一半。<!-- bf2026 -->

## 团队为什么要一人一个 IP

- **共用出口和轮换地址都会让账号看起来不像一个人在用。** 一种是几个人的账号挤在同一个全局网络工具的出口上；另一种是一个账号今天一个地址、明天换一个，看起来像很多人在登录。
- **出了问题能单独处理。** 某个人的账号被要求验证或受限，查的只是他自己那个 IP 和他自己的使用记录，不牵连同事。
- **出差也稳定。** 同事在服务支持的国家之间出差，连接仍然固定在他原来那个 IP 上。
- **个人订阅按人买。** Claude Pro/Max、ChatGPT Plus/Pro 是给一个人用的。公司想统一付款、统一管理成员，用服务商自己的团队版或企业版，固定 IP 代替不了这些。

服务商公布的支持地区，以官方页面为准：[Anthropic 支持的国家和地区](https://www.anthropic.com/supported-countries)、[OpenAI 支持的国家和地区](https://help.openai.com/en/articles/7947663-chatgpt-and-openai-api-supported-countries-and-territories)。

## 第一步：排一张分配表

先把每个人用什么服务、人在哪个国家列出来，再对照我们能选的城市（11 月 23 日购买页）：

| 档位 | 可选城市 |
|---|---|
| Claude 档（Claude Pro/Max） | 美国多个城市、法兰克福、千叶、大阪、伦敦 |
| GPT/Grok 档（ChatGPT Plus/Pro、SuperGrok） | 美国多个城市、伦敦、千叶、大阪、巴黎、首尔 |

一个五人团队排出来大概是这样：

| 成员 | 服务 | 本人所在国家 | 档位 | 城市 |
|---|---|---|---|---|
| A | Claude Max | 美国 | Claude 档 | 美国，挑标了「双 ISP」的城市 |
| B | Claude Pro | 英国 | Claude 档 | 伦敦 |
| C | Claude Pro | 德国 | Claude 档 | 法兰克福 |
| D | ChatGPT Plus | 日本 | GPT/Grok 档 | 千叶或大阪 |
| E | ChatGPT Pro | 韩国 | GPT/Grok 档 | 首尔 |

两条规矩：

- **国家跟人走，不跟公司走。** 公司注册在美国、员工在日本，这个员工用日本的 IP。
- **对不上就不买。** 某位同事所在国家不在上面的城市里，就不要给他挑别国的 IP；这一位不适合用 AI 订阅专用 ISP。

## 第二步：算一下钱

单价是每人每 30 天：Claude 档 15 PTC，GPT/Grok 档 10 PTC；每档第一个 IP 半价（7.5 PTC、5 PTC），其余按平时价。团队要几个就按人数乘单价估算，下单页显示的金额为准。

黑五（11 月 24 日至 12 月 4 日，UTC+8）：从每档第二个 IP 起，Claude 档 12 PTC、GPT/Grok 档 8 PTC，第一个 IP 仍按平时价的一半算。<!-- bf2026 -->

## 第三步：下单、核对、发给本人

1. **用团队的 Proxy302 账号统一下单**，每人一个，城市照分配表选。新单优先分配双 ISP 号段，购买页标了「双 ISP」的城市就是。
2. **到手先核对，再发给同事。** 每个 IP 都查一遍：

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/json
   ```

   USER、PASS、HOST、PORT 是占位符，SOCKS5 的 IP 换成 `socks5h://`。`country` 要和分配表一致。然后拿 `ip` 去 IPinfo 网页看 ASN 的 Type 是不是 ISP，去 IP2Location 看 Usage Type 里有没有 (ISP)，两家都是才叫双 ISP。详细读法见[双 ISP IP 是什么，怎么自己查](../dual-isp-ip-zh/README.md)。
3. **只把这个人自己的那组连接信息发给他。** 主机、端口、用户名、密码通过团队的密码管理器单独共享，不贴在群里，也不放进共用文档。
4. **同事在自己的设备上配置。** 用 [Proxy302 客户端](https://proxy302.com/client/)，或者给常用浏览器单独建一个配置填代理；关掉其他全局网络工具，只留这一层。配好后打开 `https://ipinfo.io/json`，看到的 IP 应该就是分给他的那个。
5. **每个 IP 打开自动续费。** 到期不续，IP 会被释放，再买拿到的是另一个地址。

AI 订阅专用 ISP 是一人独享的原生 ISP IP，租期内不换。双 ISP 号段我们自己核对时，IPinfo 和 IP2Location 都判为 ISP，主流检测站检测为家宽，Claude、ChatGPT 识别的就是你买的国家。

## 第四步：记一本台账

| 成员 | 服务和套餐 | 账号邮箱 | 档位 | 城市 | IP | 双 ISP 核对日期 | 续费日 |
|---|---|---|---|---|---|---|---|
| A | Claude Max | a@example.com | Claude 档 | 美国 | 203.0.113.10 | 11-27 | 12-27 |

（表里的邮箱和 IP 是示例。）每次续费后，按[续费后要不要再查一遍](../recheck-ai-subscription-ip-after-renewal/README.md)把两家 IP 库再对一次，写进核对日期这一栏。

## 有人入职、离职时

- **离职：** 先按服务商的流程处理他的订阅和账号，再关掉他那个 IP 的自动续费，从台账里划掉。
- **入职：** 给新同事新买一个，不接手前任的 IP。一个 IP 对应一个账号的历史，分开更清楚。
- **换国家常驻：** 搬到另一个支持的国家长期工作，就按新国家重新买，旧的到期不续。

## 别这样做

- 几个人轮流登录同一个 Claude 或 ChatGPT 账号。
- 一个 IP 同时给几个同事的账号用。
- 给人在支持地区以外的同事配一个别国的 IP。
- 拿动态住宅或轮换出口给长期订阅账号用。
- 账号被要求验证时想办法绕过去。照实完成验证，IP 代替不了这一步。

## 常见问题

**同一个人既用 Claude 又用 ChatGPT，要两个 IP 吗？** 规矩是一人一个稳定 IP。各档是按对应服务挑选和核对的；两个服务都要长期用、又拿不准买哪档，下单前问客服。

**团队成员都在美国，能都买美国的吗？** 能。每人一个，城市可以不同，挑购买页标了「双 ISP」的。

**之前买的 IP 不是双 ISP 号段，会自动换吗？** 不会自动换，有需要提工单问客服。

**用了双 ISP，账号就不会被封吗？** 谁都保证不了。账号本身、怎么用、付款方式和平台规则比 IP 重要。

**能先买一个试试再给全队买吗？** 可以。每档第一个 IP 半价，先给一个人配好、走一遍核对流程，再给其他人下单。

## 相关

- [双 ISP IP 是什么，怎么自己查](../dual-isp-ip-zh/README.md)
- [原生 IP 和广播 IP 怎么区分](../native-ip-check-zh/README.md)
- [How do I check an IP before I use it for ChatGPT, Claude or Grok?](../check-ip-before-ai-subscription/README.md)
- [Proxy302：AI 订阅专用 ISP](https://proxy302.com/zh/use-cases/ai-subscriptions/)
- [Proxy302 价格](https://proxy302.com/pricing/)

*Facts checked 27 November 2026.*
