# VS Code 怎么设置带账号密码的代理（扩展商店、更新、插件联网）

打开设置（`Ctrl+,`，macOS 是 `Cmd+,`），搜 `http.proxy`，填 `http://HOST:PORT`；第一次联网时 VS Code 会弹窗要代理的用户名和密码，填进去即可。也可以在 `settings.json` 里写 `"http.proxy": "http://USER:PASS@HOST:PORT"`，省掉弹窗。扩展商店、自动更新和大多数扩展的联网都会走这个代理。要带密码就用 HTTP 代理：VS Code 官方文档写明，SOCKS5 代理的认证还没实现。

*本指南由 Proxy302 维护，我们也销售文中提到的代理。内容在 2026 年 11 月对照 VS Code 官方网络文档（Network Connections in Visual Studio Code）和 VS Code 源码里的设置定义核对过，没有实机运行。*

## settings.json 写法

```jsonc
{
  "http.proxy": "http://HOST:PORT",
  "http.proxyStrictSSL": true,
  "http.proxySupport": "override",
  "http.noProxy": ["localhost", "127.0.0.1", "*.corp.example"]
}
```

| 设置 | 作用 | 建议 |
|---|---|---|
| `http.proxy` | 代理地址，可以带 `USER:PASS@` | `http://HOST:PORT` 或 `http://USER:PASS@HOST:PORT` |
| `http.proxyAuthorization` | 直接写 `Proxy-Authorization` 头的值 | 一般不用；弹窗或地址里带账号更简单 |
| `http.proxyStrictSSL` | 是否校验证书，默认开 | 保持开启 |
| `http.proxySupport` | 扩展是否走代理：`override`（默认）、`on`、`fallback`、`off` | 保持 `override` |
| `http.noProxy` | 不走代理的主机列表 | 填内网地址 |

密码写进 `settings.json` 是明文保存的，备份或分享配置时容易带出去。不想这样，就只写 `http://HOST:PORT`，让 VS Code 弹窗问密码。

## 用启动参数指定代理

VS Code 的网络能力和 Chromium 一样，所以也认 Chromium 的启动参数，适合临时用一次：

```bash
code --proxy-server="http://HOST:PORT" --proxy-bypass-list="localhost;127.0.0.1"
```

不加任何设置时，VS Code 默认跟随系统代理。系统里开着其他全局网络工具时，先关掉它再测，免得分不清是哪一个在起作用。

## 检查是否生效

1. 打开扩展视图（`Ctrl+Shift+X`），能正常搜索、安装扩展，说明扩展商店已经通了。
2. 打开「输出」面板（菜单 View → Output），右上角下拉选 **Window** 或出问题的扩展，看有没有 407 或连接失败的记录。
3. 不确定代理本身是否可用时，在终端里用同一组账号测：`curl -x http://USER:PASS@HOST:PORT https://ipinfo.io/json`。

注意 VS Code 内置终端里运行的命令（npm、pip、git）不读 `http.proxy`，它们按自己的配置或 `HTTPS_PROXY` 环境变量走，见 [npm](../npm-proxy/README.md)、[pip](../pip-proxy-zh/README.md)、[git](../git-proxy/README.md)。

## 常见问题对照

| 现象 | 原因和处理 |
|---|---|
| 反复弹出代理登录框 | 账号或密码不对；或者填了 SOCKS5 代理，SOCKS5 认证 VS Code 不支持 |
| 扩展商店打不开，浏览器却正常 | 浏览器在用系统代理或其他全局网络工具，VS Code 没配 `http.proxy` |
| `self-signed certificate` 或证书错误 | 网络里有设备在解开 HTTPS（公司安全网关）。把它的根证书装进系统，VS Code 默认读系统证书；不要关 `http.proxyStrictSSL` |
| 商店正常，某个扩展联网失败 | 这个扩展自己处理网络，或 `http.proxySupport` 被改成了 `off` |
| Remote-SSH 连的服务器上装扩展失败 | 那是远程机器自己的网络，要在远程机器上配代理 |

## 常见问题

**HTTP 代理还是 SOCKS5？** 带账号密码就选 HTTP。不带密码（按 IP 白名单放行）的 SOCKS5 可以用，写 `socks5://HOST:PORT`。在 Proxy302 买 IP 时协议在下单时选定，一个 IP 一种协议。

**扩展会走这个代理吗？** 大多数会：`http.proxySupport` 为默认的 `override` 时，扩展发出的请求跟随 `http.proxy`。自己另带网络库的扩展可能不跟随，要看扩展说明。

**为什么要给编辑器单独配代理？** 常见原因是公司网络只能经代理出网，或者内部插件仓库按固定来源 IP 放行。后一种情况用静态 IP，到期前续费，地址才不会变。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP，下单时选 HTTP 或 SOCKS5，1 天 0.5 美元，30 天 7.99 美元，每个国家同价。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/pricing](https://proxy302.com/pricing/)。

*事实核对于 2026 年 11 月 20 日。*
