# OSPF
Only dynamic routing protocol explicitly listed in CCNA test topics.

## Link State Protocols
OSPF is a link state dynamic protocol. Link state protocols mean every router creates a connectivity map of the whole network. Each router advertises information about its interfaces (connected networks) to its neighbors. These are passed along to other routers until all routers in the network develop the same map of the network. Each router independently uses this map to calculate the best routes to each destination.

Link state protocols use more CPU than other protocols because more information is shared. The tradeoff is that link state protocols tend to be faster in reacting to changes in the network than distance vector protocols.

## OSPF Basics
OSPF uses Dijkstra's shortest path algorithm. There are 3 versions. V1 is deprecated, and V2 is commonly used for IPv4 (and what is on the CCNA exam). V3 is for IPv6 networks (can also be used for IPv4, but OSPFv2 is more common for this).

Routers store information about the network in LSAs (Link State Advertisements), which are organized in a structure called the LSDB (Link State Database). Routers flood LSAs until all routers in the OSPF area develop the same map of the network.

The very basics of an LSA:
| LSA |
|-----|
| Router ID : 4.4.4.4 |
| IP: 192.168.4.0/24 |
| Cost: 1 |

If none of the router's interfaces have an IP of `4.4.4.4`, then the router either has a loopback address of `4.4.4.4`, or the RID was manually configured. The network on the interface for LSA to be flooded out of has an IP that is included in the LSA. After receiving the LSA and having a complete LSDB, then each router will use Dijkstra's to calculate the best route to the LSA.

Note: Each LSA has an aging timer (30 mins by default). The LSA will be flooded again after the timer expires.

## Basics Recap
1. **Become neighbors** with other routers connected to the same segment
2. **Exchange LSAs** with neighbor routers.
3. **Calculate the best routes** to each destination, and insert them into the routing table

 ## OSPF Areas
 OSPF uses areas to divide up the network. Small networks can be single-area without any negative effects on performance. However, in larger networks this will lead to performance issues due to Dijkstra's being O(E+log(V)). Additionally, the compute and memory resources necessary to store a very large LSDB scale exponentially.  Additionally, any small change on the network will cause every router to re-flood LSAs and run the SPF algorithm again.

 The scope of the CCNA only covers single area OSPF configuration.

 An **area** is a set of routers that contain the same LSDB
