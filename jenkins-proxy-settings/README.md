# How do I set a proxy with a username and password in Jenkins?

Open Manage Jenkins, then System, find the "HTTP Proxy Configuration" section and click Setup; enter the proxy's host in Server, its port in Port, your username and password, any hosts that should go direct in No Proxy Host (one per line, `*` as a wildcard), then use Advanced, Test URL and Validate Proxy to check it. On Jenkins versions before 2.435 the same form is on the Advanced page of the plugin manager. This proxy covers Jenkins's own outbound traffic, such as the update center, plugin downloads and tool installers. Build steps do not use it: `git`, `mvn`, `npm` or `curl` inside a job need their own proxy settings, usually the `https_proxy` and `http_proxy` environment variables. Proxy302 maintains this guide and sells these proxies; a static ISP IP costs $0.50 for one day or $7.99 for 30 days.

*Checked against the Jenkins handbook page on HTTP proxy configuration and the `hudson.ProxyConfiguration` Javadoc, November 2026. Not run locally.*

## Step 1: Jenkins's own proxy

1. **Manage Jenkins**, then **System** (Jenkins 2.435 or later). Older versions: **Manage Jenkins**, **Plugins**, **Advanced settings**.
2. In **HTTP Proxy Configuration**, click **Setup**.
3. Fill in the form:

   | Field | Value |
   |---|---|
   | Server | `HOST` (no `http://`) |
   | Port | `PORT` |
   | User name | `USER` |
   | Password | `PASS` |
   | No Proxy Host | Internal hosts, one per line, for example `*.internal.example` |

4. Open **Advanced**, put `https://updates.jenkins.io` in **Test URL**, click **Validate Proxy**.
5. Save, then go to the plugin manager and use **Check now** to confirm the update center loads.

Jenkins keeps the password as an encrypted `Secret`. The handbook also notes that Jenkins talks to the proxy itself over plain HTTP, so on a shared network the proxy credentials can be seen by someone watching that link.

An old issue in Jenkins's tracker reported that Validate Proxy ignored the No Proxy Host list, so a successful test does not prove your bypass rules work. Test an internal host from a job as well.

## Step 2: proxy for build steps

Jobs run ordinary programs, and those read their own settings. The cleanest way is to keep the proxy credentials in Jenkins's credential store and expose them only for the steps that need them.

1. **Manage Jenkins**, **Credentials**: add a "Username with password" credential, ID `p302-proxy`.
2. In a Pipeline:

```groovy
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'p302-proxy',
                                          usernameVariable: 'PROXY_USER',
                                          passwordVariable: 'PROXY_PASS')]) {
          sh '''
            export https_proxy="http://${PROXY_USER}:${PROXY_PASS}@HOST:PORT"
            export http_proxy="$https_proxy"
            export no_proxy="localhost,.internal.example"
            curl -sS --max-time 20 https://ipinfo.io/json
            npm ci
          '''
        }
      }
    }
  }
}
```

The single quotes around the `sh` script matter: the shell, not Groovy, expands `${PROXY_USER}` and `${PROXY_PASS}`, so the password is not written into the Pipeline script, and Jenkins masks it in the console log. If the password contains `@`, `:` or `/`, percent-encode it in the credential or choose one without those characters.

For freestyle jobs or a whole agent, you can instead set `https_proxy`, `http_proxy` and `no_proxy` as environment variables on the node (Manage Jenkins, Nodes, the node, Configure, Node Properties). That puts the password in the node configuration, so the credential method above is preferable.

## What uses which setting

| Traffic | Uses the HTTP Proxy Configuration? | What to set otherwise |
|---|---|---|
| Update center, plugin downloads | Yes | |
| Automatic tool installers (JDK, Maven, Node.js) | Yes | |
| Plugins that support Jenkins's proxy | Yes | |
| `git`, `mvn`, `npm`, `pip`, `curl` in build steps | No | `https_proxy` / `http_proxy`, or each tool's own config |
| Docker builds started by a job | No | Docker's own proxy settings |

Links for the tools: [git](../git-proxy/README.md), [Maven and Gradle](../maven-gradle-proxy-zh/README.md), [npm](../npm-proxy/README.md), [Docker](../docker-proxy-zh/README.md).

## Check the proxy outside Jenkins first

On the Jenkins controller:

```bash
curl -sS -o /dev/null -w '%{http_code}\n' -x "http://USER:PASS@HOST:PORT" https://updates.jenkins.io/update-center.json
```

`200` means the IP and credentials work. Buy the IP with the HTTP protocol; the form has no SOCKS option. The protocol is fixed per IP at purchase.

## Common errors

| Symptom | Likely cause |
|---|---|
| Plugin manager shows no available plugins | Update center unreachable: proxy not saved, wrong host or port, or a 407 |
| Validate Proxy reports 407 | Wrong username or password |
| Plugins install, but `git clone` in a job hangs | Build steps do not use Jenkins's proxy; set it for the job |
| Internal Git server now fails from a job | It goes through the proxy too; add it to `no_proxy` |
| Worked yesterday, fails today | The IP's term ended and it was released |

## FAQ

**Why would Jenkins need a static IP rather than rotating traffic?** If an artifact repository, cloud API or deployment target only accepts known addresses, a static IP gives the controller and agents one address to allowlist. For plain downloads, rotating residential at $1.50/GB is enough.

**Can I set the proxy with JVM system properties instead?** The handbook mentions Java's networking properties but recommends the System page. Use the form unless you manage Jenkins entirely from configuration files.

**Is there a free trial?** No. One static IP for one day costs $0.50.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (1 to 90 days, HTTP or SOCKS5 chosen at purchase) and rotating residential traffic. See [pricing](https://proxy302.com/pricing/). Related: [a fixed IP for an API allowlist](../fixed-ip-for-api-allowlist/README.md).

*Facts checked 27 November 2026.*
