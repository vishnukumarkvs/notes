linux networking
================

hostname

/etc/hostname
/etc/hosts
/etc/host.conf (old)  -- /etc/nsswitch.conf (new)

/etc/resolv.conf (has nameservers) - remove entries and u cant even dig. Can be automatically filled when connecting to internet or wifi

ip route show
netstat -r
netstat -tn (tcp connections) , use 'u' for udp
netstat - active connections
telnet 8.8.8.8 443 (checks if 443 port is open on 8.8.8.8 server)


ip link add dev dummy type dummy0
ip addr
ifconfig

NIS - network information service - centralized server which manages all its clients hostnames, other network related files

/proc - important file system 
/proc/cpuinfo
/proc/memifo
evrery process created will create s irectory here
ps command reads this filesystem
ps aux
top - terminal ui for process
btop, htop etc

/proc/sys folder has kernel level params and configuration

/etc has normal desktop level coniguration
users, passwords etc are here

/dev
device level configuration such has connected devices, network interfaces etc


df -h -> List volumes
du -sh -> disk usage of folder
rm , rm -rf -> delete folders
duf, dust -> cli tools

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

route cache
arp kernelt ables
arp -a

SLIP vs PP are deprecated. we use IPoE which gives internet over ethernet cable. It uses DHCP(Dynamix Host Configuratioon Protocol) to auto assign an Ip address to a device

TCP/IP firewall
- firewall is basically filtering packets
- we can use linux as a proxy server which can act as a firewall
- iptables is old, we use nftaables for doing packet filtering and firewall

- netstat -tnlp , ss -tnlp
- ss better tahn netstat , ss for sockets
- nft command line tool is used to mangae ip tables 
- nft list ruleset
- nft -i


NTP
- 


