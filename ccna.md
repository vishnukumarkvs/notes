ccna
====

Network interface cards

- NIC
- 25Gig network uses SFP
- SFP = Small Form-factor Pluggable transceiver. It is hot swappable which uses optical fibre as connection
- The above is also called 25Gig ethernet cable
- There are many SFP standards which vary on speeds. It depends on port type and size as well. You cant just hotswap to any SFP
- laptops have normal ethernet cable with RJ45 connector. This is electrical with copper cable. 1G home speed. Can go upto 40G max

MAC Addess
- Hexadecimal and unique



# Network topology

Bus topology
- Old
- 10 base 5 - older - big cable 
- 10 base 2 - old - small cable than base 5
- One way communication. MEaning only 1 device in network can talk at any given point of time
- Terminatiors are necessary
- break in linkage will break whole network

Now 10 base 2 and 5 are replaced with 10baseT. This has RJ45 connector. Its a UTP cable

UTP - Unshielded twisted pair. Twist is needed to reduce interference (EMI). We have flat UTP as well

Bus topology is replaced by Star Topology

Star Topology
- All devices are connected to a central hub
- Hub is dumb. Its a multi port repeater
- Whatever data is sent by a device, its replicated and flodded to all connected devices
- Still collisions will occur
- Its a physical star, but is a logical bus topology
- Single broadcast domain


- Unicast : 1 to 1
- Broadcast: 1 to many
- Multicast: 1 to subgroup

Bridge (or Bridge Network)
- Intelligent routing using MAC address
- But this is replaced by Switch
- Bridge has fewer ports
- Used CPU and routing is fully down via software. 

Switch
- Anyone can talk at any time.  Full duplex
- MAc address identifcation. Superior to bridge
- 24, 48 ports
- Uses ASIC, hardware to switch which is faster


Unmanaged vs Managed switches

Unmanaged
- Cheaper
- Cant configure
- Full broadcast

Managed
- Configurable
- Can create VLAN's
- Can creaqte a virtual lan like 8 ports out of 24 is a LAN and traffic in a vlan doesnt flow to other ports


Router
- Connect a local network to a wider network (or internet)
- Pc -> Switch -> Router
- LAN - yellow - 4 or more ports - Connect local devices like computer, printer etc - Gets private ips. Its a switch
- TEL - Grey - VoIP - Connect telephone to it
- WAN - Connect to ur ISP (Or Modem). Public IP.  - Not present in mine
- WLAN - Wifi - Wireless Local Area Network.  Wifi on/off - LAN works with this in off
- WPS - Wifi Protected Setup - Skip password to connect. On pressed, within 2 minutes, you can connect to ur device without password. This is a hazard/risk. Iphones not supported


route -n get default
ifconfig en1


Firewal
- Cisco firewal hardware device
- Has switch
- Can sit after or before router
- Has either IDS or IPS
- IDS : Intrusion Detection System - Dog detectes and barks
- IPS : Intrusion Pritection System - Big dog , detcets and protects


Router
- Can be configured using console port
- Has a power button
- 4321 rou

Switch
- No power button, we need to connect to power supply
- Switch has fans
- 3560 is present in packet tracer. More modern switches are not
- 