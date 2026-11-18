# Chrome 提示「未连接到互联网，代理服务器出现问题」（ERR_PROXY_CONNECTION_FAILED）怎么办？

这个报错的意思是：Chrome 被设成经代理上网，但它连不上这个代理本身。原因只有几类：代理地址或端口填错了、本机的代理软件已经关掉、买的代理 IP 已经到期、或者当前网络拦了这个端口。网站没有问题，换哪个网站都一样打不开。最常见的情况是，以前开过某个代理软件或其他全局网络工具，关掉以后系统代理设置没有恢复，Chrome 还在找一个已经不存在的地址。所以第一步是看清 Chrome 正在用哪个代理，再决定是把它改对，还是把它关掉。

*本文由 Proxy302 维护，我们出售这类代理 IP。报错页面的文字和各个错误代码，是 2026 年 11 月在 Chrome 157 上对着本机测试代理实际触发、照原样抄下来的。*

## 报错页面长什么样

页面标题是「未连接到互联网」，下面一行写「代理服务器出现问题，或者地址有误。」，再下面是「请试试以下办法：联系系统管理员、检查代理服务器地址」，最底下是错误代码 `ERR_PROXY_CONNECTION_FAILED`。

如果页面写的是「无法访问此网站」，代码是 `ERR_TUNNEL_CONNECTION_FAILED`，那是另一回事：Chrome 已经连上了代理，是代理不肯或没能去连那个网站，见下面的对照表。

## 第一步：找出 Chrome 在用哪个代理

Chrome 平时不自己存代理，用的是系统设置，例外是扩展程序和启动参数。按这个顺序看：

1. **系统代理设置。** Chrome 里打开「设置 → 系统 → 打开您计算机的代理设置」，会直接跳到系统的设置页。
   - Windows 10、11：设置 → 网络和 Internet → 代理。看「手动设置代理」里的「使用代理服务器」是否打开、填的地址和端口是什么；「使用设置脚本」也要看一眼。
   - macOS：系统设置 → 网络 → 选中正在用的网络（例如 Wi‑Fi）→ 详细信息 → 代理。看哪几项被勾选、填了什么。
2. **扩展程序。** 装了代理类扩展的话，Chrome 设置的「系统」那一页会提示代理设置由某个扩展控制。在 `chrome://extensions` 里先停用它试试。
3. **启动参数。** 如果 Chrome 是从一个自己建的快捷方式打开的，看快捷方式里有没有 `--proxy-server=...`。

## 第二步：不打算用代理，就把它关掉

地址是 `127.0.0.1` 加某个端口，多半是本机某个代理软件留下的：软件关了，设置还在。把系统设置里的代理开关关掉，或者重新打开那个软件，二选一。改完刷新页面就能打开。

## 第三步：要用代理，就测这个地址通不通

把系统设置里看到的地址和端口代进去（买的代理 IP，就用订单页上的主机和端口）：

```bash
curl -sS -m 15 -o /dev/null -w "proxy=%{http_connect} site=%{http_code}\n" -x "http://USER:PASS@HOST:PORT" https://www.example.com/
```

- 打印 `proxy=200 site=200`：代理是好的，问题在 Chrome 那边的设置，回到第一步对照地址、端口有没有抄错。
- 报 `Failed to connect to HOST port PORT`、`Connection refused` 或超时：Chrome 说得没错，这个地址确实连不上。往下看。

只想看端口通不通，不带账号密码也行。macOS、Linux：`nc -vz HOST PORT`；Windows PowerShell：`Test-NetConnection HOST -Port PORT`，看结果里的 `TcpTestSucceeded` 是不是 True。

## 第四步：连不上时逐项排除

| 检查 | 怎么看 |
|---|---|
| 地址、端口是不是抄错了 | 和订单页逐字对一遍，主机栏不要带 `http://`，前后不要有空格 |
| IP 有没有到期 | 看订单页的到期时间。静态 IP 到期后会被收回，原来的地址就连不上了 |
| 协议是不是选反了 | 买的是 SOCKS5，系统里却按 HTTP 代理填了（或反过来） |
| 当前网络有没有拦端口 | 换一个网络（例如手机热点）再测一次。换了就好，说明原来的网络限制了出站端口 |
| 本机安全软件 | 防火墙、杀毒软件可能拦了 Chrome 或这个端口，临时关掉测一次 |

改好以后，Chrome 有时还会记着这个代理「失败过」。打开 `chrome://net-internals/#proxy`，点「Clear bad proxies」，或者重启 Chrome。

## Chrome 的几个代理报错对照

| 代码 | 卡在哪一步 |
|---|---|
| `ERR_PROXY_CONNECTION_FAILED` | 连代理这一步就失败了：地址、端口、网络 |
| `ERR_TUNNEL_CONNECTION_FAILED` | 连上了代理，代理没能打通到网站的连接，详见 [ERR_TUNNEL_CONNECTION_FAILED](../err-tunnel-connection-failed/README.md)（英文） |
| `ERR_SOCKS_CONNECTION_FAILED` | SOCKS5 代理失败，常见原因是这个代理要账号密码，而 Chrome 不支持给 SOCKS5 填账号密码 |
| `ERR_INVALID_AUTH_CREDENTIALS` | 代理要求登录，但没有拿到账号密码 |
| `ERR_NO_SUPPORTED_PROXIES` | 代理设置本身写法不对，例如把账号密码写进了 `--proxy-server` |
| `ERR_EMPTY_RESPONSE`（每个网站都这样） | 按 HTTP 填了一个只支持 SOCKS5 的端口 |

## 常见问题

**别的软件能上网，只有 Chrome 不行？** 先确认那些软件是否也读系统代理：很多软件不读，它们能上网只说明网络本身是通的。如果系统代理设置是空的，Chrome 仍报这个错，多半是装了管代理的扩展，或者是从带 `--proxy-server` 的快捷方式启动的。

**代理要账号密码，Chrome 在哪里填？** 系统设置里一般只填地址和端口。第一次打开网页时，Chrome 会弹出登录框，在那里填用户名和密码。

**指纹浏览器的某个环境里出现这个报错？** 道理一样，是这个环境的代理连不上。在环境的代理设置里核对类型、主机、端口，并用它自带的检测按钮测一次。

**只想让一个 Chrome 窗口走代理，不动系统设置？** 可以用 `--user-data-dir` 加 `--proxy-server` 单独启动一个 Chrome，见 [How do I open one Chrome window with its own proxy?](../chrome-proxy-one-window/README.md)（英文）。

## Proxy302 在哪里

Proxy302 卖静态 ISP（静态住宅）IP，购买时选 HTTP 或 SOCKS5，一个 IP 只有一种协议；给 Chrome 用选 HTTP。每个 IP 1 天 0.50 美元、7 天 3.50 美元、30 天 7.99 美元，所有国家同价，可以按 IP 开自动续费，避免到期后地址被收回。价格见[价格页](https://proxy302.com/zh/pricing/)。

*事实核对于 2026 年 11 月 19 日。*
