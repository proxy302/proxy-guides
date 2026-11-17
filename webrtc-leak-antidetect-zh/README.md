# 指纹浏览器配了代理，WebRTC 还会暴露真实 IP 吗？怎么检查？

有可能。WebRTC 是浏览器做音视频通话和点对点连接用的，它会向 STUN 服务器发 UDP 请求来探测本机的公网地址，而这类请求不一定经过你在窗口里设的代理。如果窗口的 WebRTC 没设成跟随代理（多数指纹浏览器叫「替换」），网页就可能同时拿到代理 IP 和你本机宽带的公网 IP。检查办法是在窗口里打开 WebRTC 检测页，或在控制台跑一段脚本，看列出的公网地址是不是只有代理那一个。

*本文由 Proxy302 维护，文中的代理 IP 由我们出售。*

## 为什么要管 WebRTC

这件事首先是隐私问题：你家宽带的地址本来不该出现在你不打算暴露的地方。其次是一致性：一个账号的连接同时冒出两个公网地址，前后对不上。把 WebRTC 设成跟随代理，网页看到的就始终是同一个出口。

## 指纹浏览器里的 WebRTC 选项

| 常见选项 | 效果 | 什么时候选 |
|---|---|---|
| 替换（用代理 IP） | WebRTC 报给网页的公网地址换成代理出口 | 配了代理的窗口，一般选它 |
| 真实 | 不做处理，可能露出本机公网地址 | 不配代理、本机直连的窗口 |
| 禁用 | 直接关掉 WebRTC | 用不到网页音视频的窗口；依赖 WebRTC 的网站功能会失效 |

比特浏览器、AdsPower 等软件的叫法略有出入，以你用的版本为准。

## 检查步骤

1. **先确认代理生效。** 在窗口里打开 `https://ipinfo.io/json`，`ip` 和国家与购买时一致，再往下查。
2. **打开 WebRTC 检测页。** 比如 browserleaks.com 的 WebRTC 页面，看它列出的公网地址。
3. **看结果。** 只出现代理 IP，或者什么公网地址都没有，就是正常的；出现了你本机宽带的地址，说明 WebRTC 没跟随代理。
4. **改设置再测。** 回到窗口设置，把 WebRTC 改成替换（或禁用），保存，重开窗口，重复第 2、3 步。

不知道自己本机公网地址是多少，就在不挂代理的普通浏览器里打开一次 ipinfo.io，记下来用于对照。

## 不依赖第三方网站，自己跑脚本查

在窗口里按 F12 打开开发者工具，切到控制台，粘贴：

```js
const pc = new RTCPeerConnection({ iceServers: [{ urls: 'stun:stun.l.google.com:19302' }] });
pc.createDataChannel('probe');
pc.onicecandidate = (e) => {
  if (!e.candidate) { pc.close(); return; }
  console.log(e.candidate.candidate);
};
pc.createOffer().then((offer) => pc.setLocalDescription(offer));
```

每一行输出是一个候选地址，重点看带 `typ srflx` 的行：里面的 IP 就是 STUN 服务器看到的公网地址。它要是你本机宽带的地址，就是泄露；是代理 IP 或根本没有 srflx 行，就没问题。带 `typ host` 的行如果是 `xxxx.local` 这样的名字或 `192.168.` 开头的局域网地址，不算公网暴露，现代浏览器默认会把局域网地址隐藏成 `.local` 名字。

## 普通 Chrome、Firefox 里怎么做

不用指纹浏览器、只在普通浏览器里配代理时：

- **Chrome**：企业策略 `WebRtcIPHandling` 设为 `disable_non_proxied_udp`，WebRTC 就只走代理，不再直接发 UDP。Google 出的「WebRTC Network Limiter」扩展提供同样的选项。
- **Firefox**：在 `about:config` 里把 `media.peerconnection.ice.proxy_only_if_behind_proxy` 设为 `true`，配了代理时 WebRTC 只走代理。

这样设置后，部分音视频通话可能连不上，因为大多数 HTTP 代理不转 UDP。

## 常见错误

| 现象 | 原因和处理 |
|---|---|
| ipinfo 显示代理 IP，WebRTC 页显示本机 IP | WebRTC 是「真实」模式，改成替换 |
| 改成替换后还是本机 IP | 没保存，或窗口没重开；关掉窗口重新打开再测 |
| 用动态住宅时，WebRTC 页和 ipinfo 显示的代理 IP 不同 | 轮换模式下两次请求走了不同出口，换成[粘性会话](../sticky-session-zh/README.md)再测 |
| 网页视频会议打不开 | WebRTC 被禁用了，或代理不转 UDP；需要通话的窗口别选禁用 |
| 检测页显示 `192.168.x.x` | 局域网地址，不是公网暴露 |

## 常见问题

**设了替换，就完全不会暴露了吗？** 替换解决的是 WebRTC 这一条路。电脑上同时开着其他全局网络工具、或者扩展插件自己发请求，都可能带来别的问题，设置完整体测一遍最稳。

**禁用 WebRTC 是不是最省事？** 省事，但有的网站会用 WebRTC 做通话和文件传输，禁用后这些功能不能用。配了代理的窗口，一般选替换。

**动态住宅 IP 也要设吗？** 要。不管是什么类型的代理，WebRTC 走不走代理都是浏览器这一侧的设置。

## Proxy302 在哪里

Proxy302 出售静态 ISP IP（1 天 US$0.50，30 天 US$7.99，HTTP 或 SOCKS5）和动态住宅（US$1.50/GB 起），可以填进比特浏览器、AdsPower 等软件的窗口。各家软件的接入步骤见[比特浏览器集成页](https://proxy302.com/zh/integrations/bitbrowser/)和 [AdsPower 集成页](https://proxy302.com/integrations/adspower/)，价格见 [proxy302.com/pricing](https://proxy302.com/pricing/)。

*事实核对于 2026 年 11 月 17 日。*
