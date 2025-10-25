DNS
===

PORT: 53

dns tools
dig
nslookup
browser > public dns server(1.1.1.1 or 8.8.8.8) > root server > top level domain resolver (.com) > authoritative resolver > server
local cache kicks in after first resolve, it  expires based on ttl set in that  hosted domain dns record

types of dns records
- A : fundamental record - maps domain name to ip address
- AAAA: domain name with ipv6 address
- CNAME: alias for domain name - domain to elb domain
- MX: mail server record
- NS: nameserver record
- TXT: plain text, for domain ownership
- PTR (Pointer) : reverse dns - opposite of A record - ip to domain mapping - used for verification
- SOA : Mandatory record - State Of Authority

tls cert verification
certigo connect
openssl

dig f5.com SOA
dig f5.com A

Nameserver hold theh dns records for that domain. Also called authoritative servers

If I Have a domian f5.com which is managed by GCP, and I want xc.f5.com to be in AWS Route53, GCP root domain needs to delegate this to my aws hostd zone

add route53 nameserver records to google dns zone

Hosted zone: Its like a container in a DNS Service where you manage the DNS records for a domain and its subdomains

Public : any public domain
Private: for internal resolution like in AWS internal domains inside a vpc

bgp.he.net - bgp routes and advertisements

### Coredns

- used as standard in kubernetes
- Uses Corefile configmap
- autopath - usefull for making external domain resolution faster but takes more cpu and memory. It also has effect on k8s api server

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        log
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
    int.abc.com:53 {
      errors
      cache 30
      forward 
    }  
```