# OSPF
Only dynamic routing protocol explicitly listed in CCNA test topics.

## OSPF Show Commands
`show ip ospf neighbor`
`show ip ospf interface <int>`

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

Each area maintains a unique LSDB.

The scope of the CCNA only covers single area OSPF configuration.

### Area Terminology
- **Area**: A set of routers that contain the same LSDB
- **Backbone Area**: A special area which all other areas must connect to
- **Internal Router**: A router in which *all* of its interfaces are in the same OSPF area
- **Area Border Router (ABR)**: A router with interfaces that are in more than one OSPF area
  - Note: ABRs maintain a seperate LSDB for each area they are connected to.
  - It is recommended that an ABR has at most two areas conencted to it
- **Backbone Routers**: A router connected to the backbone area
- **Intra-Area Route**: A route to a destination that is within the same OSPF area
- **Inter-Area Route**: A route to a destination that is in a different OSPF area

### OSPF Area Rules
1. OSPF Areas should be contiguous, meaning each area should be connected, not divided up
2. All OSPF Areas must have at least one ABR connected to the backbone area
3. OSPF interfaces in the same subnet must be in the same area

## Basic OSPF Configuration
`R1(config)# router ospf <process id>`
The process ID is locally significant for the router. Routers with different process IDs *can* become OSPF neighbors. Note that the process ID is totally unrelated to the OSPF area.

`R1(config-router)# network <network ID> <wildcard mask> <OSPF area num>`
Note: for the CCNA, you only need to configure single-area OSPF, which will usually be done with area number 0.

The network command tells OSPF to:
1. Look for any interfaces wiht an IP address contained in the range specified in the network command
2. Activate OSPF on the interface in the specified area
3. The router will then try to become OSPF neighbors with other OSPF activated neighbor routers

The network command doesn't tell the router to advertise any networks.

`R1(config-router)# passive-interface g2/0`
This command is the same as RIP and EIGRP. The passive interface command tells the router to stop sending OSPF hello messages out of the interface. However, the router will continue to send LSAs informing it's neighbors about the subnet configured on the interface.
This command should always be used on interfaces which don't have any OSPF neighbors.

`R1(config-router)# default-information originate` causes the router to create a new LSA with the default gateway of R1 and flood it to OSPF neighbors. 

## Router ID Determination
In order of selection priority:
1. Manual configuration
2. Highest IP address on a loopback interface
3. Highest IP address on a physical interface

To manually configure an RID: `R1(config-router)# router-id 1.1.1.1`. Will have to run `clear ip ospf process` to make it take effect. This is a bad idea in practice, as it will wipe all OSPF configs on the router, and there will be an outage while the router goes back through OSPF.

### ASBR
An Autonomous System Boundary Router is an OSPF router that connects the OSPF network to an external network. E.g. when we call `default-information originate` on a router that is connected to the internet, the router will become an ASBR

## OSPF Cost Metric
Automatically calculated based on bandwidth, or manual configuration. Calculated by dividing a reference bandwidth value by the interface's bandwidth. This defaults to 100 mbps.

Note: OSPF cost is always at least 1. For speeds faster than the reference bandwidth, the cost value gets converted to 1.

You should configure the reference bandwidth greater than the fastest links in your network for both futureproofing reasons as well as better cost calculation.
`R1(config-router)# auto-cost reference-bandwidth <reference bandwidth value>`
Need to make sure that the reference bandwidth is the same across the whole network.

Interface speed defaults to what it actually operates at. However, this can be manually changed. The bandwidth that the interface operates at will stay the same, but for various calculations including OSPF, the bandwidth will be changed. Don't do this.

A better approach is to use the `ip ospf cost` command to manually set an OSPF cost for the interface.

## OSPF Neighbors
Ensuring that routers successfully become OSPF neighbors is the main role of a CCNA. They do the rest automatically if they can become neighbors.

When OSPF is activated on an interface, the router starts sending OSPF hello messages out at regular intervals (governed by the hello timer, default is 10 seconds). Hello messages are multicast to `224.0.0.5`. They are encapsulated in an IP header, with a value of 89 in the protocol field.

### Neighbor States
- **Down** state
  - This is the first neighbor state
- **Init** state
  - Hello packet is received, but own RID is not in the hello packet
- **2-way** state
  -  Potential neighbor will send a hello packet with it's own RID
  -  Kind of like a SYN/ACK reply
  -  Has RID of both routers
- **2-way** state part 2
  - Original router sends another hello packet with it's own RID

Both routers have received a hello message with their own RIDs in them. They can begin sharing LSAs to build the common LSDB.

- **Exstart** state
  - Two routers now prepare to exchange info
  - One router is designated the router, one is designated the slave
  - Higher RID becomes the master, lower RID becomes the slave
  - Database Description packets are exchanged to determine master and slave relationship
- **Exchange** state
  - Routers exchange DBDs which contain a list of the LSAs in their LSDB
  - DBDs do not include detailed info about the LSA, just basic information
  - Routers compare the info in the DBD they received to the information in their own LSDB to determine which LSAs they must receive from their neighbor
- **Loading** state
  - Routers send Link State Request (LSR) messages to request that their neighbors send any LSAs they don't have
  - LSAs are sent in Link State Update (LSU) messages
  - Routers send LSAck messages to acknowledge receipt
- **Full** state
  - Routers have a full OSPF adjacency and identical LSDBs
  - Routers continue to send and listen for hello packets every 10 seconds
  - Each time a hello packet is received, the **Dead** timer is reset (40 seconds by default)
  - If the dead timer hits 0, the neighbor is removed
  - Routers will continue to share LSAs as the network changes to make sure each router has a complete and accurate map of the network

At this point, the routers have a full picture of the area topology and can start forwarding traffic
