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

