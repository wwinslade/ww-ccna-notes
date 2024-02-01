# Routing Fundamentals and Static Routes
Routes are stored in a routing table. A routing table is kin to a MAC table in a switch, except for IP packets instead of ether frames.

Two main routing methods, dynamic and static.

Static routing: Manually configured by a sysadmin
Dynamic routing: Routers use a dynamic routing protocol to share routing info with eachother automatically to build their routing tables

Route: tells router: "to send a packet to destination X, you should send the packet to the next-hop Y." Or, if the dest is directly connected, forward the packet, or if the destination is the router, receive the packet for yourself.

To configure an interface on a router with an IP:
- `int xxxx`
- `ip addr <ip> <subnet>`
- `no shutdown`

To validate: `sho ip int brief`

To view routing table: `sho ip route`
	- Codes section lists different protocols used and their corresponding flags
	- L -> local, a route to the actual IP addr is configured on the interface
	- C -> connected, a route to the network the interface is connected to (using the netmask provided)

## Automatically added routes when configuring an interface with an IP
Note: When you configure an int with an IP and enable it with no shutdown, 2 routes per int will be automatically added: a connected route and a local route
	- Connected and local routes are neither static nor dynamic routes, they are automatically added when an int is configured

Connected routes connect entire subnets to the interface on the router, so that the router knows to send packets to any host on that subnet, it can send the traffic out of the interface

Local route is a route to the EXACT ip address configured on the interface (netmask of /32 or 255.255.255.255). Local route essentially represents the IP address of the ROUTER on the subnet connected to the interface, so that if it receives a packet with a destination of that /32 address, the router knows the packet is intended for itself

EX: If I configure int Gig0/1 with the following: `ip addr 10.10.10.0 255.255.255.0`, then the two routes will be added:

`sho ip route` now reveals:
10.10.10.0/24 s variably subnetted, 2 subnets, 2 masks
	C      10.10.10.0/24 is directly connected, GigabitEthernet0/1
	L       10.10.10.1/32 is directly connected, GigabitEthernet0/2

Assuming these are the only routes in the routing table, say we receive a packet with destination 10.10.11.1. We don't have a route avaliable for this because of the subnets configured, so the router will either send the packet using a different route, or it will drop the packet if there are no possible routes for it to take

## Route Selection
In our prior example, say we have a packet with a destination of 10.10.10.1. Both the local and connected routes match the packet, so which route will the router forward it to? The router will choose the most specific route. The /24 subnet route has 256 different Ips, while the /32 subnet has 1 ip. Therefore, the /32 route is more specific. The router will therefore receive the packet locally and keep it for itself.

Most specific matching route = matching route with the longest prefix length

NOTE: These 3 lines are not really routes, and generally can be ignored when looking at the routing table.
```
10.10.10.0/24 s variably subnetted, 2 subnets, 2 masks
	C      10.10.10.0/24 is directly connected, GigabitEthernet0/1
	L       10.10.10.1/32 is directly connected, GigabitEthernet0/2
```

Unlike switches, if a router doesn't know where the destination is (i.e. it doesn't have a matching route) then the router will drop the packet, unlike a switch which would flood the packet out

NOTE: If an interface is shutdown, its routes won't appear in the routing table.

## Default Gateway
Also called default route. A route to 0.0.0.0/0, includes all IP addresses between 0.0.0.0 and 255.255.255.255. Least specific route possible. 

End hosts usually have no need for any more specific routes, they just need to know that in order to send packets outside of the local network, they have to send packets to the default gateway.

Routers will attempt to any and all matching routes that are more specific than the default route. If there are none, then the router will forward the packet to the default gateway. This is often used to direct traffic to the internet.

In a routing table, if the line `Gateway of last resort is not set`, this means that no default route has been configured. If no matching routes are in the routing table, then the packet will be dropped since there is no default route. To set a default route, use the following command:
```
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```
This will result in the routing table looking something like below:
```
...
Gateway of last resort is 203.0.113.2 to network 0.0.0.0

S*	0.0.0.0/0 [1/0] via 203.0.113.2
S	10.0.0.0/8 [1/0] via 192.168.12.2
...
```
The `*` after the static route flag indicates that the route is a candidate for becoming the routers default route. There can be multiple of these, however, only one will be the active default gateway.

## Static Routing
Local and Connected routes aren't sufficient to forward packets to networks that aren't hard wired to the router (remote networks). Enter static and dynamic routes.

To add in a static route, each router in the path needs two routes, one to the destination, and one to the origin. This ensures *two way reachibility*, so that two networks can both receive and transmit packets to eachother. However, routers don't need routes to *all* the networks in the path to the destination. They just need routes to facilitate the next hop in the chain. It will prove useful to make a table of all the static routes that you will have to configure across the chain of routers before configuring them.

`ip route <ip-address> <netmask> <next-hop-ip>` is used to add the static route. In the [Jeremy's IT Lab](https://youtu.be/YCv4-_sMvYE?si=64sbRDNLN2xFct6c&t=1085) video, this looks like `ip route 192.168.4.0 255.255.255.0 192.168.13.3`

The above command creates a static route in the routing table that looks like:
```
S    192.186.4.0/24 [1/0] via 192.168.13.3
```
NOTE: The [1/0] refers to administrative distance and metric, respectively. This will be coverd at a later point.

This is essentially saying: to send a packet to destinations in network `192.168.0.4/24`, forward the packet to next hop `192.168.13.3`

NOTE: If there are multiple paths to the same destination, it is possible to configure the router to load balance between the paths. It is also possible to use one as a primary and the rest as backups.

## A Note on L2 Encapsulation
If we have a packet bound for a host on a remote network, the L3 header of a packet will have the source IP as the IP of the host sending traffic, and the destination IP as the IP of the end host the packet is intended for. However, it must be noted that this is not exactly true when it comes to L2 encapsulation. 

In this instance, the L2 header will have a source MAC of the device transmitting the packet, but the destination MAC will be that of the default gateway's connected interface. When the default gateway received the etherframe, the gateway will then re-encapsulate the packet in an L2 header that has a source MAC of the gateway, and a destination MAC of the next-hop device. 

This will occur at each router in the chain, until the packet reaches the router to which the destination host's network is attached to.

The source and destination IPs of the packet will never change, but the source and destination MACs in the L2 header will change at each hop in the path

## Configuring an exit-interface
An exit interface is a substitute for configuring a next hop. The router will then send packets matching this rule out of one of the router's interfaces. This is useful for routers that are directly connected to the intended destination.
`ip route <ip-address> <netmask> <exit-interface>`

You may specify both an exit interface and a next hop. This is useful for configuring a route between two routers that are directly connected.
`ip route <ip-address> <netmask> <exit-interface> <next-hop>`

Behold, the following routing table:
```
S	192.168.1.0/24 is directly connected, GigabitEthernet0/0
S	192.168.4.0/24 [1/0] via 192.168.24.4, GigabitEthernet0/1
	192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C		192.168.12.0/24 is directly connected, GigabitEthernet0/0
L		192.168.12.2/32 is directly connected, GigabitEthernet0/0
```
When you only specify an exit interface, the routing table shows the destination network as directly connected to that interface, even though this isn't the case. The first entry in the above routing table is an example of this, and resulted from calling `ip route 192.168.1.0 255.255.255.0 Gig0/0`. 
It is important to note the distinction here. Despite displaying as directly connected, the subnet `192.168.1.0/24` is not directly connected to the router. Routes in which only an exit interface are specified rely on Proxy ARP to function.

Specifying an exit interface, along with a next hop, yields the second line in the above routing table.

Generally, specifying only an exit interface is not an issue, but it is best to generally specify both a next hop and an exit interface.





