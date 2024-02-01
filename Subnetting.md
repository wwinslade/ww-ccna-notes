# Subnetting
## Background: Classful Subnetting
Subnetting is a huge part of network engineering and a big portion of the CCNA exam.

Classful inter-domains are no longer used. CIDR lets us go classless. This optimizes the usage of the IPv4 space and frees up a ton of addresses, as classful domains proved to waste thousands of IP addresses. But, understanding classfull IPv4 addresses is useful for understanding why we have gone classless.
| Class  |  First Octet (0b) | First Octet Range | CIDR Prefix |
|--------|-------------------|-------------------|-------------|
| A      | 0xxxxxxx          | 0-127             | /8  |
| B      | 10xxxxxx          | 128-191           | /16 |
| C      | 110xxxxx          | 192-223           | /24 |
| D      | 1110xxxx          | 224-239           |
| E      | 1111xxxx          | 240-255           |

Certain class A addresses are reserved for special use, i.e. `127.0.0.0/8` for loopback. In general, going down from class A to class C, there are less networks avaliable within the class, but the tradeoff is that there are more avaliable IPs to be assigned to devices. At class C, there are many avaliable networks, but far fewer IPs on that network that can be assigned to devices.

IANA (Internet Assigned Numbers Authority) is in charge of assigning IPv4 addresses/networks to companies based on their size, and ensuring that unqiue IPs were given out.

## Moving to CIDR
With the advent of CIDR, classes were removed. This allowed large networks to be split down into smaller subnets than classful allowed, thus freeing up more IPv4 addresses.

To find the number of hosts allowed on a subnet, simply do `2^n - 2`, where *n* is the number of host bits (in other words, 32 minus your CIDR slash notation value).
