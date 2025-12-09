
FRR = Free Range Routing
- Converts a linux machine into a router
- It supports multpile protocols like BGP, MPLS, PIM etc
- Each support is a seperate daemon. All this is manged by another daemon zebra



docker pull quay.io/frrouting/frr:10.5.0

docker run -d --privileged --name frr \
  --network host \
  quay.io/frrouting/frr:10.5.0


/etc/frr/daemons


By default, frr wont enable any daemon. You need to change in above file and restart frr service


vtysh - give terminal shell

- show daemons
- show int brief -> shows interfaces
- show bgp summary
- show ip route
- show running-config
