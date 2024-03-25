# Dynamic Trunking Protocol
- Cisco proprietary protocol
- Allows switches to determine whether an interface becomes an access port or a trunking port
- Enabled by default on all Cisco switches
- For security switches, manual configuration is best. DTP should be disabled on all switchports

- A switchport in `dynamic desirable` mode will try to form a trunk with other Cisco switches. Will form a trunk if connected to another switchport in the following modes:
  - `switchport mode trunk`
  - `switchport mode dynamic desirable`
  - `switchport mode auto`

- A switchport in `dynamic auto` mode is more passive than dynamic desirable. Dynamic auto will tell a switch "if you want to form a trunk, I'll form a trunk", but dynamic auto doesn't actively try to form a trunk with another switch.
  -  

# VLAN Trunking Protocol
