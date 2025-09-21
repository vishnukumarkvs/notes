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
local cache ki

tls cert verification
certigo connect
openssl

