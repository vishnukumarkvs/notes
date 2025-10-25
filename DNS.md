DNS
===

dns tools
dig
nslookup
browser > root server > top level domain resolver (.com) > authoritative resolver > server
local cache kicks in after first resolve, it  expires based on ttl set in that  hosted domain dns record

types of dns records
- A : fundamental record - maps domain name to ip address
- AAAA: domain name with ipv6 address
- CNAME: alias for domain name - domain to elb domain
- MX: mail server record
- NS: nameserver record
- TXT: plain text, for domain ownership
- PTR (Pointer) : reverse dns - opposite of A record - ip to domain mapping - used for verification

tls cert verification
certigo connect
openssl