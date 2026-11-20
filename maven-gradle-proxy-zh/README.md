# Maven 和 Gradle 下载依赖怎么走带账号密码的代理

Maven 在 `~/.m2/settings.xml` 里加一段 `<proxies>`，写上代理的 host、port、username、password；Gradle 在 `~/.gradle/gradle.properties` 里写 `systemProp.http.proxyHost` 和 `systemProp.https.proxyHost` 两组参数（各带 `proxyPort`、`proxyUser`、`proxyPassword`）。两者都只认 HTTP 代理最省事，所以买 IP 时选 HTTP 协议。写在用户目录下的配置文件里，密码不会跟着项目代码进仓库。IDE 里的代理设置管不到这两个构建工具，要单独配。

*本指南由 Proxy302 维护，我们也销售文中提到的代理。内容在 2026 年 11 月对照 Maven 官方代理指南、Maven Resolver 文档和 Gradle 官方网络文档核对过，没有实机运行（测试机上没有装 JDK）。*

## Maven：settings.xml

```xml
<!-- ~/.m2/settings.xml -->
<settings>
  <proxies>
    <proxy>
      <id>my-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>HOST</host>
      <port>PORT</port>
      <username>USER</username>
      <password>PASS</password>
      <nonProxyHosts>localhost|127.0.0.1|*.corp.example</nonProxyHosts>
    </proxy>
  </proxies>
</settings>
```

几个容易弄错的地方：

- `<protocol>http</protocol>` 写的是**代理本身**的协议，不是仓库的协议。仓库是 `https://` 也照样写 `http`：只定义了 http 代理时，Maven 会拿它去连 https 仓库。
- `nonProxyHosts` 用竖线 `|` 分隔，可以用 `*` 通配，适合把公司内部的 Nexus 排除在外。
- 改完直接跑 `mvn -U dependency:resolve` 或任意构建，看依赖能不能下来。

### 密码不想明文写

Maven 3 可以加密：先 `mvn --encrypt-master-password` 生成主密码写进 `~/.m2/settings-security.xml`，再用 `mvn --encrypt-password` 加密代理密码，把输出的 `{...}` 整串填进 `<password>`。

### Maven 4 的一个坑

Maven 4 截至 2026 年 11 月还是预览版。它默认改用 JDK 自带的 HttpClient 下载依赖，而 JDK 默认不允许在 HTTPS 隧道上发 Basic 认证，结果是密码没错也报 407。两个办法任选一个：

```bash
mvn -Djdk.http.auth.tunneling.disabledSchemes= clean package
mvn -Dmaven.resolver.transport=apache clean package
```

第二个是换回基于 Apache HttpClient 的下载方式。JDK 这条传输也不支持 SOCKS 代理。Maven 3.9 系列默认就是 Apache HttpClient，没有这个问题。

## Gradle：gradle.properties

```properties
# ~/.gradle/gradle.properties
systemProp.http.proxyHost=HOST
systemProp.http.proxyPort=PORT
systemProp.http.proxyUser=USER
systemProp.http.proxyPassword=PASS
systemProp.http.nonProxyHosts=localhost|127.0.0.1|*.corp.example

systemProp.https.proxyHost=HOST
systemProp.https.proxyPort=PORT
systemProp.https.proxyUser=USER
systemProp.https.proxyPassword=PASS
```

- 两组都要写。现在的仓库基本都是 `https://`，只写 `http.*` 等于没配。
- HTTPS 那组没有 `nonProxyHosts`，用的是 `systemProp.http.nonProxyHosts`。Gradle 文档专门注明这不是笔误。
- 放在用户目录 `~/.gradle/`（或 `GRADLE_USER_HOME`）下，对所有项目生效；放在项目根目录的 `gradle.properties` 也行，但别把密码提交进仓库。
- Gradle Wrapper 下载 Gradle 发行包时也会读这两个位置的 `systemProp.*`。

SOCKS5 代理的写法：

```properties
systemProp.socksProxyHost=HOST
systemProp.socksProxyPort=PORT
systemProp.java.net.socks.username=USER
systemProp.java.net.socks.password=PASS
```

## 报错对照

| 现象 | 可能原因 |
|---|---|
| Maven 报 `407 Proxy Authentication Required` | 账号密码不对；用的是 Maven 4 预览版时见上面「Maven 4 的一个坑」 |
| Gradle 依赖能下，Wrapper 下载发行包报 407 | Wrapper 用的是 JDK 自带的网络库，隧道上的 Basic 认证默认被禁；可以在 `gradle.properties` 加一行 `systemProp.jdk.http.auth.tunneling.disabledSchemes=` 再试 |
| 只有 http 仓库能下，https 仓库连不上 | Gradle 只写了 `http.*` 那组 |
| `Connection refused` 或超时 | host 或端口错了，或者把 SOCKS5 的端口当 HTTP 用 |
| 内部仓库也走了代理、被拒 | 没加进 `nonProxyHosts` |

不确定是代理还是构建工具的问题时，先用 curl 测同一组账号：`curl -x http://USER:PASS@HOST:PORT https://repo.maven.apache.org/maven2/ -I`，返回 200 说明代理没问题。

## 常见问题

**IntelliJ 里设了代理，为什么 Gradle 同步还是失败？** IDE 的代理只管 IDE 自己的联网（插件、更新等），Gradle 和 Maven 是单独的进程，要按本页配。英文版说明见 [JetBrains IDE 代理设置](../jetbrains-ide-proxy/README.md)。

**用静态 IP 还是动态 IP？** 下依赖用哪种都行。如果私有仓库按来源 IP 放行，就用静态 IP，到期前续费，地址才不会变。

**能不能只在 CI 里用代理？** 可以。Maven 用 `mvn -s ci-settings.xml ...` 指定一份 CI 专用的 settings 文件；Gradle 可以把 `gradle.properties` 作为机密文件放进 CI 的 `GRADLE_USER_HOME`，或在命令行传 `-Dhttps.proxyHost=...` 这类参数。本机开发不受影响。

## Proxy302 能提供什么

Proxy302 卖静态住宅（ISP）IP，协议在下单时选 HTTP 或 SOCKS5，每个国家同价：1 天 0.5 美元，30 天 7.99 美元；也卖动态住宅流量，1.5 美元/GB 起。官网：[proxy302.com/zh](https://proxy302.com/zh/)，价格：[proxy302.com/pricing](https://proxy302.com/pricing/)。

*事实核对于 2026 年 11 月 20 日。*
