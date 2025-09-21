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