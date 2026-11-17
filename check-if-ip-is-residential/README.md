# How can I tell if a proxy IP is residential, ISP or datacenter?

Look up who the address block is registered to: WHOIS (or RDAP) and the ASN tell you whether it belongs to a consumer internet provider, which points to residential or ISP, or to a hosting or cloud company, which points to datacenter. Then read the reverse DNS name and compare the type labels in two or three IP databases. No single check is proof. An ISP (static residential) IP is registered to a consumer provider but runs on server hardware, so databases sometimes disagree about it.

*Proxy302 maintains this guide and sells these proxies, so run these checks on our IPs as well as anyone else's.*

## Step 1: get the exit IP

Look up the address websites actually see, not the proxy's host name:

```bash
curl -s -x "http://USER:PASS@HOST:PORT" https://ipinfo.io/ip; echo
```

## Step 2: run the lookups

Save as `iptype.sh` and run `bash iptype.sh 203.0.113.7` with your exit IP. It needs `whois`, `dig` and `curl`; macOS has all three, and on Linux they come from the `whois` and `dnsutils` (or `bind-utils`) packages.

```bash
#!/usr/bin/env bash
# Usage: bash iptype.sh IP
IP="$1"

echo "== ASN and AS name (public IP-to-ASN lookup)"
whois -h whois.cymru.com " -v $IP" | tail -n 1

echo "== Who the block is registered to"
whois "$IP" | grep -iE '^(orgname|org-name|organization|netname|descr|owner):' | head -n 6

echo "== Reverse DNS"
dig -x "$IP" +short

echo "== One IP database"
curl -s "https://ipinfo.io/$IP/json"
echo
```

For a fuller registry record, fetch it over RDAP: `curl -sL "https://rdap.org/ip/$IP"`. The `name` field and the entities list hold the registrant.

## Step 3: read the signals

| Signal | Rotating residential | ISP (static residential) | Datacenter |
|---|---|---|---|
| Registrant and ASN | A consumer internet provider | A consumer internet provider, sometimes its business arm | A hosting, cloud or server company |
| Reverse DNS | Often an address-based name with words like `dyn`, `dhcp`, `dsl`, `cable` or `fiber` | Varies; may be static-looking or empty | Server-style names, or none |
| IP database type | isp or residential | Usually isp; sometimes business or hosting | hosting |
| Over time | Changes between connections | Fixed for the term | Fixed |

Read across the rows. Registrant plus database type usually settle it; reverse DNS is a hint, and an empty result proves nothing.

## Comparing databases

Every IP database builds its labels differently and updates on its own schedule. Look the same IP up in two or three of them and compare the type, ASN and organisation fields; agreement matters more than any single answer. PeeringDB (`https://www.peeringdb.com/api/net?asn=NUMBER`) shows a network type that the network itself chose, so treat it as a weak hint: large consumer providers do not always label themselves that way.

## Common errors

| Problem | What is going on |
|---|---|
| The lookup shows your own address | You looked up without the proxy; do step 1 first |
| Two databases disagree | Normal; go by the registrant and the majority |
| WHOIS names an ISP but a database says hosting | Common for ISP proxies served from a data center, or for a block that changed hands |
| The city differs between sites | Databases disagree on cities; judge by country |
| `whois` returns several records | Read the most specific network block, usually the last one |

## FAQ

**Can a website still tell it is a proxy?** It can use the same databases plus its own data, so a residential label never makes an IP invisible. A proxy changes the network address, not the site's rules.

**Are ISP proxies residential?** They are registered to consumer providers like home connections, but they stay fixed. See [ISP proxy vs residential proxy](../isp-proxy-vs-residential-proxy/README.md).

**What does Proxy302 check before delivery?** Before an Amazon ISP IP is handed over, we check that it is not a datacenter address. That product serves Amazon Seller Central; each IP is 12 PTC (US$12) for 30 days, and your first one is half price.

**Which tool should I trust?** None alone. The registry record is the most direct evidence; databases are opinions built on top of it.

## Where Proxy302 fits

A single one-day static residential (ISP) IP from Proxy302 costs $0.50, a cheap way to run every check above on our addresses before ordering more; there is no free trial. All products are listed on the [pricing page](https://proxy302.com/pricing/).

*Facts checked 17 November 2026.*
