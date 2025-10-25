DNS
===

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

