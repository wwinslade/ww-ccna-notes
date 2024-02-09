# VLANs

## Definition of a LAN
A LAN is a single broadcast domain, including all devices within that broadcast domain.
A broadcast domain is a group of devices which will receive a broadcast frame (destination MAC of `ffff:ffff:ffff`) sent by any other member of the domain.

Note: Routers do not forward broadcast frames. They receive the frame, but will not forward it to remote networks. However, routers can send broadcast frames. So, despite routers not forwarding broadcast frames, two routers connected to eachother can be considered a broadcast domain.

## What is a VLAN
A VLAN is a way to partition a broadcast domain into a group of smaller broadcast domains, for purposes of security and network performance. 

Lots of unnecessary broadcast traffic can reduce the performance of the network greatly. VLANs help to reduce this. VLANs also help partition device access, which adds security to the network.



