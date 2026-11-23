# How do I send a Kubernetes pod's outbound traffic through a proxy?

Kubernetes has no proxy setting for pods. You give the container the standard variables, `HTTPS_PROXY`, `HTTP_PROXY` and `NO_PROXY` (plus lowercase copies), keep the proxy username and password in a Secret, assemble the URL with `$(VAR)` references in the pod spec, and list every in-cluster name and the API server address in `NO_PROXY` so cluster traffic stays direct. Only programs that read those variables use them: curl, Go, Python and .NET programs do; Java does not; Node.js only with `NODE_USE_ENV_PROXY=1`. Image pulls are done by the node's container runtime, which needs its own setting. Proxy302 maintains this guide and sells these proxies; for pods, an HTTP static ISP IP is the simplest fit.

## 1. Store the credentials in a Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: egress-proxy
type: Opaque
stringData:
  PROXY_USER: "USER"
  PROXY_PASS: "PASS"   # percent-encode @ : / # in advance, e.g. @ -> %40
```

The value ends up inside a URL, so store the password already percent-encoded.

## 2. Build the proxy URL in the pod spec

Kubernetes expands `$(NAME)` when `NAME` is defined earlier in the same `env` list, including variables that come from a Secret:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fetcher
spec:
  replicas: 1
  selector:
    matchLabels: { app: fetcher }
  template:
    metadata:
      labels: { app: fetcher }
    spec:
      containers:
        - name: app
          image: curlimages/curl:latest
          command: ["sh", "-c", "while true; do curl -sS https://ipinfo.io/json; sleep 300; done"]
          env:
            - name: PROXY_USER
              valueFrom: { secretKeyRef: { name: egress-proxy, key: PROXY_USER } }
            - name: PROXY_PASS
              valueFrom: { secretKeyRef: { name: egress-proxy, key: PROXY_PASS } }
            - name: HTTPS_PROXY
              value: "http://$(PROXY_USER):$(PROXY_PASS)@HOST:PORT"
            - name: HTTP_PROXY
              value: "$(HTTPS_PROXY)"
            - name: https_proxy
              value: "$(HTTPS_PROXY)"
            - name: http_proxy
              value: "$(HTTPS_PROXY)"
            - name: NO_PROXY
              value: "localhost,127.0.0.1,.svc,.cluster.local,10.96.0.1,10.96.0.0/12,10.244.0.0/16"
            - name: no_proxy
              value: "$(NO_PROXY)"
```

`HOST` and `PORT` are placeholders. Replace the address ranges with your cluster's own (step 3). The lowercase copies are there because curl ignores uppercase `HTTP_PROXY`, and some tools only read one case.

Environment variables are read when the container starts. After changing them, restart the pods: `kubectl rollout restart deployment/fetcher`.

## 3. What goes into NO_PROXY

| Entry | Why |
|---|---|
| `localhost,127.0.0.1` | Sidecars and health checks inside the pod |
| `.svc,.cluster.local` | Service names written in full, such as `api.billing.svc.cluster.local` |
| Short service names your app calls, such as `redis` | A bare name has no dot, so `.svc` does not match it |
| The API server's ClusterIP | In-cluster clients reach it by IP. Find it with `kubectl get svc kubernetes -n default -o jsonpath='{.spec.clusterIP}'` |
| Service and pod address ranges, such as `10.96.0.0/12` | Calls made straight to an IP. Only some clients understand ranges (next table) |
| Cloud metadata address `169.254.169.254` | Node and workload identity on most clouds |

## 4. Which programs follow the variables

| Runtime | Reads the variables | Address ranges (CIDR) in NO_PROXY |
|---|---|---|
| curl | Yes; uppercase `HTTP_PROXY` is ignored, lowercase `http_proxy` is read | Yes (we tested curl 8.7) |
| Go `net/http` default transport | Yes | Yes |
| Python `requests` | Yes | Yes, for IP addresses |
| Python `urllib` | Yes | No (we tested Python 3.9: the range was ignored) |
| Node.js 24 built-in `fetch` | Only with `NODE_USE_ENV_PROXY=1` | No (we tested Node 24.14: the range was ignored) |
| .NET `HttpClient` | Yes | Names only; use a leading dot for subdomains |
| Java | No | Not applicable: set `-Dhttps.proxyHost`/`-Dhttps.proxyPort` through `JAVA_TOOL_OPTIONS` and add an `Authenticator` in code; see [Java HttpClient proxy](../java-httpclient-proxy/README.md) |

When a client ignores ranges, list the exact names and IPs it calls instead.

## 5. Check from inside the cluster

```bash
kubectl run proxytest --rm -it --restart=Never --image=curlimages/curl \
  --env="HTTPS_PROXY=http://USER:PASS@HOST:PORT" \
  --command -- curl -sS --max-time 20 https://ipinfo.io/json
```

`country` should be the country of your proxy. This command puts the password in your shell history and the pod spec, so use it once for testing and use the Secret for real workloads. If it times out, check that a NetworkPolicy or firewall lets pods reach `HOST:PORT`.

## 6. Image pulls are a separate setting

The kubelet asks the node's container runtime (containerd on most clusters) to pull images. Pod variables do not reach it. On self-managed nodes, give containerd its own variables:

```ini
# /etc/systemd/system/containerd.service.d/http-proxy.conf
[Service]
Environment="HTTPS_PROXY=http://USER:PASS@HOST:PORT"
Environment="NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,10.244.0.0/16,.svc,.cluster.local"
```

Then `sudo systemctl daemon-reload && sudo systemctl restart containerd`. In a systemd file, a `%` in an encoded password must be written as `%%`. On managed node groups, follow your cloud provider's instructions for node proxies.

## Common errors

| Symptom | Likely cause |
|---|---|
| `CONNECT tunnel failed, response 407` | Wrong user or password, or special characters not percent-encoded in the Secret |
| Calls to other services time out or return proxy errors | The service name or its IP is missing from `NO_PROXY` |
| `kubectl` or an operator inside the pod fails against the API server | The API server ClusterIP is not in `NO_PROXY` |
| The app ignores the proxy completely | The runtime does not read the variables (Java, Node without `NODE_USE_ENV_PROXY`) |
| `ImagePullBackOff` on a node behind a firewall | containerd has no proxy setting |

## FAQ

**Can I give the whole cluster one outbound IP without touching each app?** A proxy only covers programs that use it. For all traffic, cluster-level egress gateways or a cloud NAT gateway with a fixed address are the usual tools. A proxy suits the case where only a few workloads need a particular exit.

**Static or rotating for pods?** A static residential (ISP) IP when a partner allowlists your address or a job must keep one identity; rotating residential for crawlers that make many independent requests.

**SOCKS5?** Go, curl and Python `requests` (with `requests[socks]`) can use `socks5h://`, but many runtimes cannot. HTTP is the safer choice across mixed workloads, and with Proxy302 the protocol is chosen per IP at purchase.

**Is there a free trial?** No. One static IP for one day costs $0.50; 30 days costs $7.99.

## Where Proxy302 fits

Proxy302 sells static ISP IPs (HTTP or SOCKS5), rotating residential traffic from $1.50/GB, and datacenter IPs. See [pricing](https://proxy302.com/pricing/) and [Docker proxy settings](../docker-proxy-zh/README.md) (Chinese) for the container side.

*Facts checked 23 November 2026.*
