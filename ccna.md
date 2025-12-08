ccna
====

Network interface cards

- NIc
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
- Intelligent roting using MAC address
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
- 