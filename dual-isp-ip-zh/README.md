# 双 ISP IP 是什么，怎么自己查（2026）

双 ISP IP，就是两家互相独立的 IP 库都把这个地址归到运营商网络：IPinfo 网页上 ASN 一栏的类型（Type）是 ISP，IP2Location 的使用类型（Usage Type）里带 (ISP)。只要有一家写的是机房、托管或企业，就不算双 ISP。两家的查询页都是公开的，拿到 IP 后两三分钟就能自己查完。Proxy302 的 Claude 档（15 PTC/30 天）和 GPT/Grok 档（10 PTC/30 天）新单优先分配双 ISP 号段，购买页标了「双 ISP」的城市就是；每档第一个 IP 半价，1 PTC = 1 美元。本指南由 Proxy302 维护，我们也卖这些代理。要说清楚的是：这个标签只代表两家 IP 库怎么归类，不是任何平台的结论，也没有哪个 IP 能保证账号不被限制。

*黑五（北京时间 2026 年 11 月 24 日至 12 月 4 日）：AI 订阅 ISP 从第二个 IP 起，30 天 Claude 档 12 PTC、GPT/Grok 档 8 PTC、Gemini 档 6 PTC，首个 IP 仍是平时价的半价。本页其他地方写的是平时价，当天价格以[定价页](https://proxy302.com/zh/pricing/)为准。*<!-- bf2026 -->

## 两家库各看哪一栏

| | IPinfo | IP2Location |
|---|---|---|
| 查询页 | `https://ipinfo.io/` 后面接 IP | `https://www.ip2location.com/demo/` 后面接 IP |
| 看哪一栏 | ASN 里的 Type | Usage Type |
| 算 ISP 的写法 | `ISP` | 带 `(ISP)` 的值，比如「(ISP) Fixed Line ISP」；同一行里还有 (MOB) 也算 |
| 不算的写法 | `Hosting`、`Business`、`Education` | `(DCH)` 机房、`(CDN)`、`(COM)` 商业、`(ORG)`、`(EDU)`，以及只有 `(MOB)` 没有 `(ISP)` |

两个容易踩的地方：

- **IPinfo 免费 JSON 里没有类型。** `curl https://ipinfo.io/json` 只给 `org`（ASN 编号和名称），类型要到 IPinfo 网页上看。
- **只看一家不够。** 各家数据来源和算法不一样，常有一家说运营商、另一家说机房的情况。两家一致，才叫双 ISP。

## 自己查：四步

1. **先拿到出口 IP。** 一定要经过代理去查，不然查到的是你自己的网络：

   ```bash
   curl -sS --max-time 20 -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/ip
   ```

   SOCKS5 的 IP 把前缀换成 `socks5h://`。USER、PASS、HOST、PORT 是占位符。

2. **到 IPinfo 查。** 在普通浏览器里打开 `https://ipinfo.io/` 加这个 IP（这一步是查地址，不用开代理），找 ASN 那块的 Type；顺便看隐私检测里的 Hosting，不应该显示 Detected。

3. **到 IP2Location 查。** 打开 `https://www.ip2location.com/demo/` 加这个 IP，看结果表里的 Usage Type。

4. **对国家。** 两个页面显示的国家都应该是你买的国家。把查询日期记下来，两家库都会更新，过一阵再查可能不一样。

想多一个参考，可以经过代理打开 IPPure 这类检测站看它给的网络类型。检测站给的是检测结果，所以准确的说法是「检测为家宽」或「检测为住宅」。

## 结果怎么读

| IPinfo 类型 | IP2Location 使用类型 | 结论 |
|---|---|---|
| ISP | (ISP) Fixed Line ISP | 双 ISP |
| ISP | 同一行里 (ISP) 和 (MOB) 都有 | 双 ISP |
| ISP | (DCH) Data Center/Web Hosting/Transit | 两家不一致，不算 |
| ISP | 只有 (MOB) Mobile ISP | 不算，IP2Location 没写 (ISP) |
| Business | (COM) Commercial | 不算，是企业网络 |
| Hosting | (DCH) | 机房地址 |

## 双 ISP、原生、独享，是三件事

- **双 ISP** 说的是网络类型：两家库都说是运营商网络。
- **原生** 说的是地点：地址段的登记国、定位库显示的国家、广播这个段的运营商所在国，三个一致。双 ISP 不一定原生，详见[原生 IP 和广播 IP 怎么区分](../native-ip-check-zh/README.md)。
- **独享** 说的是谁在用：查询页看不出来，只能看卖家的条款。Proxy302 的 AI 订阅 ISP 是一人独享；普通静态 IP 方面，30 天及以上的静态 IP 是你独享的。

## 双 ISP 说明不了什么

- **平台怎么看这个 IP。** Anthropic、OpenAI 这些服务都没公开怎么给 IP 归类，IPinfo 和 IP2Location 的标签不是它们的判断。
- **具体是不是哪一户在用。** IP 库描述的是地址段属于什么网络。
- **账号安不安全。** 账号本身、怎么用、付款方式、平台规则，比一个标签重要得多。

## Proxy302 的 AI 订阅 ISP 怎么用双 ISP

Proxy302 的 AI 订阅 ISP 是给一个 AI 账号长期固定在一个 IP 上用的独享原生静态 ISP IP：一人独享；登记国、定位国、运营商所在国一致；续费一般保留同一个 IP；交付前会拨通并核对国家。

- **Claude 档、GPT/Grok 档：** 新单优先分配双 ISP 号段，购买页标了「双 ISP」的城市就是，没标的城市还是之前的原生号段。网络来自本国运营商，比如美国 AT&T、韩国 KT、越南 SPT，以及巴西本地运营商。我们自己核对时，IPinfo 和 IP2Location 都判为 ISP，主流检测站检测为家宽，Claude、ChatGPT 识别的就是你买的国家。
- **Gemini 档：** 用的是原生号段，不在这次双 ISP 的范围里。
- **之前买的 IP** 不会自动换，有需要提工单问客服。
- 双 ISP 静态 IP 也单独上架了：[proxy302.com/zh/proxies/dual-isp/](https://proxy302.com/zh/proxies/dual-isp/)。

| AI 订阅 ISP（30 天） | 价格 | 第一个 IP |
|---|---|---|
| Claude 档 | 15 PTC | 7.5 PTC |
| GPT/Grok 档 | 10 PTC | 5 PTC |
| Gemini 档 | 8 PTC | 4 PTC |

这些 IP 给身在 AI 服务官方支持地区的人自己用。IP 不会让不支持的地区变成可以用，也不改变服务商的使用政策和支持地区名单。

## 常见问题

**双 ISP 就是住宅 IP 吗？** 不完全是。双 ISP 只说明两家库都把它归到运营商网络；检测站说它是住宅，是另一项检测结果。

**IPinfo 说 ISP，IP2Location 说 DCH，算不算？** 不算，两家不一致。

**买了双 ISP 的 IP，Claude 账号就不会被封吗？** 谁都保证不了。选 Claude 支持的、你本人所在的国家，按平台规则用。

**能先查再买吗？** 交付后按上面的步骤自己查。只想练手的话，普通静态 ISP 1 天 0.5 美元；AI 订阅 ISP 按 30 天卖。

## 相关

- [原生 IP 和广播 IP 怎么区分，自己怎么查](../native-ip-check-zh/README.md)
- [怎么判断一个 IP 是住宅、ISP 还是机房](../residential-or-datacenter-ip-check-zh/README.md)
- [Claude 账号老被封，和 IP 有关系吗](../claude-account-suspended-ip-zh/README.md)
- [Proxy302：什么是双 ISP IP](https://proxy302.com/zh/guides/what-is-dual-isp-ip/)
- [Proxy302：AI 订阅专用 ISP](https://proxy302.com/zh/use-cases/ai-subscriptions/)
- [Proxy302 价格](https://proxy302.com/zh/pricing/)

*事实核对于 2026 年 11 月 21 日。*
