# **Device Management**

**Basic device setup**

```
!# Device name
(config)# hostname [hostname]

!# Encrypted Privileged EXEC mode password
(config)# enable secret [password]

!# Encrypt Running Config passwords
(config)# service password-encryption

!# Display a prevention access message (when CLI prompts)
(config)# banner login $[Message_goes_here]$

!# Display welcome message (after user EXEC mode login)
(config)# banner motd $[Message_goes_here]$

!# Disable DNS translation lookup
(config)# no ip domain-lookup

!# Minimun password length allowed
(config)# security passwords min-length 12
```

**Time Synchronization | Network Time Protocol (NTP)**
_**IMPORTANT:** Configure correctly in all devices so, **certificates**, **protocols** and **syslog** can be running correctly. The **convergence in NTP servers, is very slow ( 300 seg = 5 min )**_

```
!# Displays the current date and time
# show clock
!# Verifies if NTP clock is synchronized with the master NTP server
# show ntp status

!# Set time manually
# clock set HH:mm:ss [day] [month_3_letters] [year]

(config)# clock timezone UTC -6 !# MX UTC
(config)# ntp master  !# Configure device as NTP SERVER
(config)# ntp server [ntp_server_ipv4]  !# Configure device as NTP CLIENT
```

**Syslog**

```
---------------------------------------------------------
| # |  Severity   |             Description             |
---------------------------------------------------------
| 0 | Emergency   |  System unstable                    |
| 1 | Alert       |  Inmediatly correction needed       |
| 2 | Critical    |  Hard device errors                 |
| 3 | Error       |  Misconfigurations & bad conditions |
| 4 | Warning     |  Warning conditions (prevention)    |
| 5 | Notice      |  Action that may be handled         |
| 6 | Information |  Notification & show status         |
| 7 | Debugging*  |  Messages of normal information     |
---------------------------------------------------------
```

_Syslog is an open standard of logging messages that complies Cisco´s IOS. All log messages are prompted in the **command line (RAM logging buffer) by default**, but they can be centralized and send to **external syslog servers**._

```
!# Displays a syslog summary and buffered stored logs (RAM)
# show logging
```

```
!# Disable syslog in line console
(config)# no logging console

!# Displays events with X severity or higher in VTY lines
(config)# terminal monitor !# Enable seeing the console logs in VTY lines
(config)# logging monitor [severity_number]

!# Store in RAM logging buffer events with severity X or higher
(config)# logging buffered [severity_name]

!# External Logging Server
(config)# logging [logging_server_ipv4]
(config)% logging trap [severity_name]

!# Turn-off debugging output
(config)# undebug all
```

**Store configuration (by memory or server cloning)**

```
# copy running-config startup-config

By TFTP Server
# copy running-config tftp:[server_ip_address]
```

# **Neighbor Discovery Protocols (CDP & LLDP)**

_**Cisco Discover Protocol (CDP)** allows to know **OS version** and **IP addresses** from the Cisco connected devices by messages send each **60 sec**_

```
# show cdp neighbors [detail]?

!# Enable/Disable CDP for security
(config)# [no]? cdp run
```

_The open standard Link Layer Discovery Protocol differs (LLDP) on CDP in only discover 1 device per port (by physical interfaces), is disabled as default and not supported for all devices. Each LLDP discover message is sent **30 sec** by default_

```
# show lldp neighbors [detail]?

!# Enable/Disable CDP for security
(config)# [no]? lldp run

!# Allowing transmission on interface
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# [no]? lldp [transmit|receive]
```

# **Console Access & Remote Management**

## Out-of-band Management

```
(config)# line {console | aux} 0 {1-5 (depends_on_device_aux)}?
(config-line)# password [password]
(config-line)# exec-timeout [minutes] [seconds]
(config-line)# logging synchronous
(config-line)# login [local]?
```

## In-band management

Virtual Terminal Lines (VTY)

```
(config)# username [username] {privilege [privilege_level_at_login]}? secret [user_password]
(config)# line vty [start-line] [end-line]
(config-line)# password [password]
(config-line)# access-class [access_list_ID] in
(config-line)# logging synchronous
(config-line)# login [local]?
```

### _**Switch-only**_ remote management setup

```
(config)# interface vlan 1
(config-vlan)# ip address [ipv4] [netmask]
(config-vlan)# exit
(config)# ip default-gateway [gateway_ip_address]
```

_Note: Configure a password for VTY lines in remote management in **mandatory**, otherwise the connection will be refused_

### _**Router-only**_ remote management setup

_Note: As a good practice to manage a router, the interface **Loopback 0 /32** must be declared and addressed in order to always have an interface running logically in the device and reach it no mattering the physical connection_

### SSH v2

_**Pre-requisite**: A hostname, RSA Key **(min 768)** and IP domain name must be generated_

```
(config)# hostname [hostname]
(config)# ip domain-name [domain_name]
(config)# crypto key generate rsa

!# Remove RSA key
(config)# crypto key zeroize rsa

!# Verify RSA keys
# show crypto key mypubkey rsa
```

```
(config)# ip ssh version 2
(config)# ip ssh time-out [minutes]
(config)# username [username] secret [password] [privilege {0-15}]?
```

```
(config)# line vty 0 {1-15 (depends_on_device)}
(config-line)# transport input ssh
(config-line)# password [telnet_password]
(config-line)# login local
(config-line)# logging synchronous
(config-line)# exec-timeout [minutes]
```

´´´
!# Block VTY connections for N seconds after X simultaneously failed attemps between Y seconds
(config)# login block-for [seconds_blocked] attempts [failed_attemps_number] within [interval_of_failing_seconds]

!# Enable secure HTTPS SSH access
(config)# ip http secure-server
(config)# ip http authentication local
´´´

```
$ ssh -l [username] [ip_ad dress]
```

## AAA (Authentication-Authorization-Accounting)

_The AAA methods allows to **authenticate remote connection** by the usage of an **active directory service**, besides on having all the usernames-passwords locally stored in the network decice. Multiple AAA servers can be implemented to allow redundancy. The Cisco´s AAA server is the **Identity Services Engine (ISE)**_

- Authentication: Verification that somebody is who they say they are.
- Authorization: The specific capabilities that an user is allowed to do within the system.
- Accounting: Keeps track of the actions a suer has carried out.

### RADIUS & TACACS+

_**RADIUS:** Open standard protocol. Commonly used for end **administrators access level AAA services**, because it has more **granular authorization capabilities**._
_**TACACS+:** Open standard protocol. Commonly used for end **user level AAA services**, such as VPN access._

```
!# Setup a backup route (only enable to use if RADIUS or TACACS+ servers can´t be reached)
(config)# username [backup_username] secret [password]

!# Configure AAA external usage
(config)# aaa new-model

!# Repeat commands to register multiple AAA servers
(config)# [radius | tacacs] server [server_name]
(config-radius-server)# address ipv4 [ipv4_server]
(config-radius-server)# key [server_name_locally_significant]

(config-radius-server)# aaa group server [radius | tacacs+] [aaa_group_name]
(config-sg-radius)# server name [server_name] !# Repeat command as many RADIUS servers registered

(config-sg-radius)# aaa authentication login default group [aaa_group_name] local
```

# **Interface Configuration**

## Interface Speed (full/half duplex)

_Note: By default all interfaces are set as **auto** negotiating the max speed in link.**Both sides in the link must be in the same** (auto-auto or manually changed in both) state so the link can work properly_

```
!# Set manually the speed is recommended only on small networks
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# duplex [auto|full|half]
(config-if)# speed [mbps] !# Can´t be set on virtual devices (needs to have the physical cable attatched)
```

## Addresing

### DHCP

_Dynamic Host Configuration Protocol (DHCP): By defining a DHCP server all clients receive an **IP address**, **network mask**, **default gateway** and **DNS server(s)** information. **In IPv6 the multicast address for DHCP Discovery traffic is FF05::1:3**_

_DHCP Operations (in-order of process)_

1. **Discover**: Client sends a **broadcast message to locate DHCP server**
2. **Offer**: The server replies with an **unicast offering a (not in use) address from pool**
3. **Request**: If the address offered it´s **unique (checked by ARP)**, the client accepts it and take it **leased by a defined time**
4. **ACK**: The server is notified that the address has been accepted and **removes it from the pool** so it will not be offered again (at least for the lease time, then can be re-offered to other or the same client)

```
!# Displays the number of leased/available addresses and pool data
# show ip dhcp pool

!# Displays relation between clients(MAC Addresses) and their leased IPs
# show ip dhcp binding

!# Examinates the DHCP messages
# show ip dhcp server statistics
```

#### DHCP Server

```
(config)# {ip | ipv6} dhcp excluded-addresses [start_ip_range_excluded] [end_ip_range_excluded]   !# Not used for IPv6
(config)# {ip | ipv6} dhcp pool [dhcp_pool_name]
(config-dhcp)# network [dhcp_network_to_assign_addresses] [netmask]
(config-dhcp)# default-router [default_gateway_ip_to_propagate]
(config-dhcp)# dns-server [dns_server_ip_to_propagate]
(config-dhcp)# lease [days] [hours] [minutes]
```

**Stateless Address Auto-Configuration (SLAAC)**

_By using SLAAC the **router advertise their interface IPv6 subnet (Global Unicast address)** to all hosts by ICMP and use it to assign them addressing. A **DHCPv6 is still required** because **SLAAC ONLY advertise the subnet**, **assign addresses** and **it´s default gateway** but **CAN´T setup a DNS server for hosts**. When a host is using SLAAC it will send all traffic through **:: (unespecify address)**. SLAAC doen´t use ARP, it uses simmilar packets known as **Neighbor Discovery (Solicitations & Advetisements) through ICMP**._

_DHCPv6 represent a significant network security because it keeps track of IPv6-MAC addresses leased and because IPv6 is based on unicasting traffic an attacker can connect directly to a host. That´s why **the only reason to have DHCPv6 enable is for DNS acknowlegment** and we must **disable any possible IPv6 assignation from DHCP** converting all DHCP requests into **Stateless** to only conserve SLAAC IPv6_

```
!# Configure on the interface that connects (or is the nearest to) to LAN memebers (the DHCP clients)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ipv6 dhcp server [dhcp_ipv6_pool_name]

!# STATEFUL: Defines the prefix to assign all new IPv6 addresses
address prefix [ipv6_prefix]/[cidr]

!# STATELESS: Doesn´t assigns IPv6 addresses, only dns-server, default-gateway & domain-name
(config-if)# ipv6 nd other-config-flag
```

#### DHCP Relay Agent

_The **DHCP Discovery message (broadcast) is NOT forwarded for a router** if we have the DHCP server in other network. To enable DHCP communications and addressing response from one network to another we need to do the following configuration on any router in the middle of the client and server. If a **client is not reaching the DHCP server** it´ll have an **169.254.0.0** address._

```
!# Router (non-DHCP server) interface connected to LAN (the default gateway for clients)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}

!# DHCPv4
(config-if)# ip helper-address [next_hop_IPv4_to_reach_dhcp_server]

!# DHCPv6
(config-if)# ipv6 dhcp relay destination [next_hop_IPv6_to_reach_dhcp_server] [exit_interface_to_DHCPv6_server]
!# USE ONLY when STATEFUL DHCP wants to be enable
(config-if)# ipv6 nd managed-config-flag

Note: For next-hop IP, use the opposite address in the point-to-point link between the relay router (configuring) and the one that is the DHCP server (the one that has the next-hop IP locally connected)
```

#### DHCP Client

```
!# Configuration interface to dynamic addressing
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip address dhcp
```

### Manually Assigned

**Multilayer Switch ONLY**

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# no switchport
(config-if)# ip address [ipv4] [netmask]
```

#### IP4

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip address [ipv4] [netmask]
(config-if)# description [interface_description]
(config-if)# no shutdown
```

#### IP6

_Designed to support built-in security, host mobility and **solve problems in unicast communication that NAT couldn´t** (direct IP phone calls, firewalls, traversal and proxy servers 1-t0-1 reachability). **128 bit address (16x8 bits)**. A **dual stack** implementation supports both IPv4 and IPv6. IPv6 doesn´t support broadcast traffic but it supports **multicast via FF02::1**_

```
# show ipv6 int brief

!# Display link-local and global unicast address from neighbors
# show ipv6 neighbors
```

- **Global Unicast (2000::/3)**

  _Similar to **public addresses** in IPv4, are used to internet reachability **well-defined hosts with static identification** (like servers), commonly used for organizations with **2001:DB8:0::/48**_

- **Unique Local (FC00::/7)**

  _Similar to **private addresses** in IPv4 (from RFC 1918). Used only for internal LAN reachability._

- **Link-local (FE80::/10 - FEB0::/10)**
  _Defined link for communications that group hosts to allow **data-flow between it´s member hosts only** and it´s locally significant to allow **inter-host reachability in the LAN**._

- **EUI-64**

  _**Autogenerated IPv6 address** defining the network portion (64 bits) to use and by **MAC partitioning will be derived the host unique** assignation. The 48 bits in host **MAC is prefixed with FF:FE** and the **7th bit is modified** in order to get the 64 bits in host portion remaining. All **Link-local addresses are autogenerated with EUI-64 by default in Cisco devices**._

```
!# Enable IPv6 usage
(config)# ipv6 unicast-routing
```

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# description [interface_description]
(config-if)# no shutdown

!# Global Unique addressing
(config-if)# ipv6 address [ipv6_global_unique]/[cidr]?
!# EUI-64 autogenerated addressing
(config-if)# ipv6 address [ipv6_64_network_prefix]/64? eui-64

!# Override the EUI-64 default link-local
(config-if)# ipv6 address [link_local_address] link-local
```

# **DNS Hosts**

_Resolve FQDN via TCP or UDP using port 53_

```
(config)# ip domain-lookup
(config)# ip host [FQDN] [ip_address]
```

# **Routing Protocols**

```
!# Displays the routing table
# show ip route

!# Displays the routing protocol running
# show ip protocols

!# Display protocol configuration only
# show running-config | section [protocol]
```

```
----------------------------------------------------
|            Administrative distance table         |
----------------------------------------------------
|          Learned Route        |   Symbol   | AD  |
----------------------------------------------------
|             RIP               |     R      | 120 |
|             OSPF              |     O      | 110 |
|             EIGRP             |     D      | 90  |
|          Static Routes        |     S      | 1   |
| Default Gateway (last resort) |     S*     | 1   |
|         Local & Loopback      |     L      | 0   |
|        Direct (Connected)     |     C      | 0   |
----------------------------------------------------
```

_**Router ID**: **Highest ip address on any loopback or interface (if no loopback exists)** interface in the router that identifies the device. Also can be specified manually._

## Static Routes

_Manually defined routes to reach certain network via router´s interface or it´s gateway (neighbor attatched interface) to reach it. The **Floating Static Routes (FSR)** are **backup routes** defined with a mayor Administrative distance (AD) compared to the advertised route learned from certain protocol. (Ex. for an OSPF route (110) the AD in the FSR must be higher like 115)_

```
(config)# ip route [network_address] [subnet_mask] {[next_hop_address] | [exit_interface]} [AD_for_floating_routes]?
```

## Interior Gateway Protocols (IGP)

_Only form adjacency with directed connected networks and advertise the best routes to reach networks in the same administrative area. All IGP perform **ECMP (Equal Cost Multi Path) 4 paths max** if the same destination can be reached by same costing paths_

### Distance Vector Protocols

_Distance vector protocols send their entire routing table to directly connected neighbors, slow convergence but excellent for small administrative network usage_

#### EIGRP (Advanced distance-vector metric DUAL)

_Enhanced Interior Gateway Routing Protocol (EIGRP), **only supported on Cisco devices**. It´s best path decition making is based on the combination between **bandwidth + delay** by an algorith called **DUAL** (Diffusive Update Algorithm) that calculates a certain K values. Fast convergence time, efficient multicast messages, for large networks & the only configurable protocol for unequal cost load balancing._

```
!# Show adjacency via EIGRP
# show eigrp neighbors

!# All eigrp enabled interfaces and it´s statistics
# show ip eigrp interfaces
```

```
!# Router EIGRP process level
(config)# router eigrp [autonomous_system]
(config-router)# eigrp router-id [router_id]          !# Manual Router-ID assignation
(config-router)# network [network_to_advertise] [wildcard]
(config-router)# default-information originate        !# Default routes injection
(config-router)# redistribute {static | ospf | rip}   !# Specific routes injection from other protocols

# Interface level EIGRP
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip hello-time eigrp [autonomous_system] [secs]
(config-if)# ip hold-interval eigrp [autonomous_system] [secs]
```

_**Adjacency Requirements:** All **k values** and **autonomous system** must match in routers, so EIGRP can know and advertise networks_

#### RIPv2

_Routing Information Protocol (RIP), it´s best path decition making is based on the **hop count (max 15)** to reach the targeted network. Upadtes of networks are send every **30 sec** by the **224.0.0.9 multicast** address. RIPv2 support authentication and VLSM. RIPng supports !=v6 network advertisement. Protocol only used on lab/test & small networks_

```
!# All routes learned by RIP and the interfaces where reach them
# show ip rip database
```

```
(config)# router rip
(config-router)# version 2
(config-router)# no auto-summary
(config-router)# network [network_to_advertise]  !# Classful network

!# Manual summarisation through neighborg interfaces
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip summary-address rip [network_to_advertise] [subnet_mask]

!# Default route injection
(config)# router rip
(config-router)# default-information originate
```

### Link-state Protocols

_Link state protocols send detail information about itself an it´s directly connected interfaces to all the routers in the network, fast convergence, high reliability but more processing needed_

#### OSPF

_Open Shortest Path First (OSPF), depends on **bandwidth** for metric calculation using the **SPF** (Shortest Path First) algorithm and advertise links in their area to each other every **10 secs (Hello packets)** by **224.0.0.5 multicast**. It´s an open standard supported by any vendor._

_OSPF Operations & Packets Messages (In order of sending)_

1. **Hello Packets**: Sended to other devices in the same area to enable adjacencies every 10 secs as default and wait until 40 secs (default dead interval = 4x hello packets) if no response is made. **Discover Neighbours**
2. **Database Description (DBD)**: Packet containing the known networks from the router to members in area. **Form adjacencies**
3. **Link State Request (LSR)**: Message send to look for information about an unknown network in the DBD. **Flood Link State Database (LSDB)**
4. **Link State Advertisement (LSA)**: Routing update send from some router in the area who can solve the LSR. **Compute the Sortest Path**
5. **Link State Update (LSU)**: Contains a list of LSAs to be updated during flooding via **224.0.0.6 multicast** address. **Install best routes in routing table**
6. **LSAck**: Acknoledgement of LSU and LSDB updated correctly. **Respond to network cahnges**

_For each network segment or subnet declared a **Designted Router (DR)** and **Backup (BDR)** will be elected based on the **highest priority (0-255) default 1** or the **highest Router ID (loopback or interface address)** this process is set automatically by OSPF but can be changed manually._

_OSPF Router Types_

- **Backbone Routers**: All of it´s interfaces belong to the area 0, they mantain a full LSDB with other routers.
- **Area Border Routers (ABR)**: They manage inter-area (IA) and external-area (EA) routes, having interfaces in multiple areas. Separate the LSA flooding zones in multiple LSDBs (one per area connected). All ABR need to defined a **manual summarization for route ranges** attatched to it´s different areas, in order to enable LSDB segregation.
- **Autonomous System Boundary Routers (ASBR)**: Redistributes routes learned from another source (static, EIGRP, RIP or other protocols) via OSPF (O\*E2)

```
!# LSDB Link State Data Base
# show ip ospf database

!# All information about routers links connections
# show ip ospf neighbors

!# Detail information of OSPF adjacencies with OSPF
# show ip ospf interface brief
```

```
!# Router OSPF process level
(config)# router ospf [process_id]
(config-router)# log-adjacency-changes !# Advertise a log when OSPF link status change
(config-router)# auto-cost reference-bandwidth [base_kbps]
(config-router)# network [network_to_advertise] [wildcard] area [area_number]
(config-router)# default-information originate !# Propagate default-static routes to area memebers

!# MULTI-AREA: Network summarization for ABRs (IA) use to reduce entries in routing table
NOTE: The summarized subnet_mask must be EXACTLY the inverse of the wildcard declared when announcing the network in order for all the routes to be correctly advertised.
(config-router)# area [area_id] range [network_to_advertise] [subnet_mask]

!# Interface level OSPF
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip ospf [process_id] area [area_number]
(config-if)# ip ospf cost [new_cost]
(config-if)# ip ospf priority [new_priority]
(config-if)# ip ospf network point-to-point !# Advertise interface using the subnet mask
(config-if)# ip ospf {hello-interval | dead-interval} [value]
```

```
!# Reset OSPF to see Router-ID/priority changes (if doesn´t work, save & reload device)
# clear ip ospf process
```

_**Adjacency Requirements:** All **hello (10 sec default)**, **dead time (4x hello) intervals** and **area number (Area ID)** must match in routers, so EIGRP can know and advertise networks_

#### IS-IS

_Intermediate System - Intermediate System Protocol (IS-IS), it depends on the cost of the route but the bandwidth metric is not calculated as OSPF does by default. The cost can be manually configure to manipulate the path (by default works with hop count as RIP). Used for Service Providers and large organizations over MPLS for scalability._

## Adjacency & Passive Interfaces

_Prevents or activates the transmission of hello packets for network discovery, commonly used for dynamic routing protocols to advertise networks to it´s attatched interfaces. A **passive interface can´t form adjacency** and must be redirect manually (static route). A **loopback must be set always as passive interface** because as a logical interface it´ll waste resources advertising a non-existent physical interface_

```
!# Disable particular interfaces
(config)# router {eigrp | ospf | rip} [autonomous_system]?
(config-router)# passive-interface {gigabit | ethernet | loopback} {0-X}/{0-X}?

!# Disable all by default & enable individually
(config)# router {eigrp | ospf | rip} [autonomous_system]?
(config-router)# passive-interface default
(config-router)# no passive-interface {gigabit | ethernet | loopback} {0-X}/{0-X}?
```

## Exterior Gateway Protocols (EGP)

### BGP

_Border Gateway Protocol (BGP)_

# **Inter-Vlan & Encapsulation**

_Virtual Local Area Networks (VLAN) segments the broadcast domains in a switch by grouping ports. The VLAN is identified as another IP segment and the communication between multiple VLANs must be routed._

```
# show vlan brief
# show interface {gigabit | ethernet} {0-X}/{0-X} switchport
```

## **Switch Inter-VLAN Configuration**

```
!# VLAN Creation
(config)# vlan [vlan_id]
(config-vlan)# name [vlan_name]

!# Access Port & VLAN port assigntation
(config)# interface range {gigabit | ethernet} {0-X}/{0-X} - {0-X}
(config-if)# description [access_description]
(config-if)# switchport mode access
(config-if)# switchport access vlan [vlan_id]
(config-if)# switchport voice vlan [voice_vlan_id]  !# Only used when need to segregate data from VoIP

!# Trunk Port (802.1Q)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# description [trunk_description]
(config-if)# switchport trunk encapsulation dot1q
(config-if)# switchport mode nonegotiate !# Disables DTP
(config-if)# switchport mode trunk
(config-if)# switchport trunk native vlan [native_vlan_id]
(config-if)# switchport trunk allowed vlan [vlan_id1, vlan_id2, ...]
```

_**Administrative VLAN**: **Remote access management** VLAN for SSH/Telnet_
_**Native VLAN**: Manage **untagged traffic** and **coordinates the trunk VLAN sharing** between 2 ports by communicating allowed VLANs by the same native ID (must match in every switch with trunk connection)_

_**Dynamic Trunking Protocol (DTP)**: Is a **Cisco propietary protocol**, negotiates the trunking connection between switches. It´s recommended to turn it off DTP **(nonegotiate)** and set the switchport mode to access or trunk manually to prevent VLAN-Hopping Attacks. In DTP all ports are set to **auto mode by default** and can only make an adjacency when the status is **auto-desirable** or **desirable-desirable** in both links._

```
(config-if)# switchport mode dynamic [auto | desirable] !# DTP modes, not recommended to set in modern switches
```

_**VLAN Trunking Protocol (VTP):** Is a protocol that allows to add, edit or delete VLAN information in a centralized **VTP server (full crud, selected with the higher revision number)** to synchronise VLAN data (vlan.dat) with other switches **VTP clients**. The **VTP transparent** client does not participate in the VTP domain, but **will pass on all VLAN advertisements from servers to other clients without making updates to it´s vlan.dat**._

```
# show vtp status
```

```
(config)# vtp domain [vtp_domain_name]
(config)# vtp mode {server(default) | client | transparent}
```

## **Router Inter-VLAN Configuration**

_Router-on-a-stick (ROAS subinterfaces)_

**Note:** First off all you have to create a native subinterface (with the same native vlan_id on switch´s trunk link) in order to enable communication between vlans.

```
!# Enable interface attatched to trunk switchport
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)#no shutdown

!# Define virtual subinterfaces for each corssing VLAN
(config)# interface {gigabit | ethernet} {0-X}/{0-X}.[subinterface_number]
(config-subif)# encapsulation dot1q [vlan_ID] [native]?
(config-subif)# ip address [vlan_default_gateway] [netmask]
```

_Multilayer Switch (SVIs)_

```
!# Enable switch routing
(config)# ip routing

!# Define SVIs to manage each VLAN
(config)# interface vlan [vlan_id]
(config-if)# ip address [vlan_default_gateway] [netmask]

!# Configure router-switch connection (for WAN reachablity to other networks)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}.[subinterface_number]
(config-subif)# no switchport
(config-subif)# ip address [default_gateway] [netmask]
```

_Another way to route VLAN traffic can be having multiple interfaces of the router attatche to each different VLAN and enable the communication by a simple routing of interfaces. This will demand more physical media and it´s better to enable it by the other 2 techniques above. The usage of virtual subinterfaces (router-on-a-stick) represents more contention of bandwidth but requires less cables attatched._

```
!# Reset a switchport to default configuration
(config)# default interface {gigabit | ethernet} {0-X}/{0-X}
```

# **FHRP & Network Redundancy**

_First Hop Redundancy Protocols: Controls **Layer 3 redundancy and failover recovery**. The usage of **Floating Static Routes** via other router is a feasible solution but we will need to do the manual configuration on each route we need to backup. The problem starts when we look at the **access layer (where hosts)** are assigned with certain default gateway, and **if that gateway fails they need to have another entry to send outside traffic.**_

## HSRP

_The **Hot Standby Redundancy Protocol (HSRP)** is a Cisco´s protocol, creates a **Virtual Gateway IP (VIP)** to assign hosts, if any redundancy routers at distribution layer fails the remaining operational router takes over the operation with transparent service for the hosts. Selects an **active** router which will route all the network traffic through them and if it fails the remaining router in **standby** mode will take the ownership to be the active router._

_We can defined the router that can be active by modifying its **priority (default 100)**, if a tie priority happends the **highest IP** wins the active mode. But is **important to set a preemtion for this**, otherwise the interface will be flapping with the ownership._

```
!# Displays VIP information & active/standby assignation
# show standby {brief | neighbors}?
```

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip address [ip_address] [netmask]    !# Physical address DIFFERENT in both routers
(config-if)# standby [hsrp_id] ip [vip_address]   !# Virtual address EQUAL in both routers
(config-if)# standby [fhrp_id] priority [priority]
(config-if)# standby [fhrp_id] preempt            !# USE ONLY in the router you want to convert active with HIGHEST PRIORITY
(config-if)# standby [hsrp_id] authentication md5 key-string [password] !# Authenticates MD5 without key-chain
(config-if)# no shutdown
```

_Another FHRP protocols used are **Virtual Redundancy Router Protocol (VRRP) open standard** and **Cisco´s Gateway Load Balancing Protocol (GLBP)**_

# **STP & Loop Prevention**

_Spanning Tree Protocol (STP) **prevents loops in layer 2**, which cause 3 main problems: **broadcast storms**, **un-stable MAC tables (CAM)** and **multiple frame transmission**._

_STP Terminology_

- **BPDU**: Bridge Processing Data Unit, contains BridgeID information and set all enable ports in a switch to **blocking state (default)** until find out if can forward if no loop was detected **(50 sec to converge)**
- **BridgeID**: Contains **MAC** and **Priority (0-65535) default 32768**
- **Root Bridge**: Elected by lowest BridgeID, **lowest priority or lowest MAC** all of its ports are in forwarding state (**designated state**).
- **Root Port**: Interfaces from other switches (not Root Bridge) directly conected to the selected Root Bridge.
- **Blocked Port (blocked/alternated state)**: It´s the port attatched to the **highest BridgeID neighbor** or **highest Port/Interface Number to reach Root Bridge**, doesn´t forward traffic and it´s administrative down to prevent loop.

_STP Versions_

- **802.1D STP**: Use STP for all the VLANs in the LAN, the open standard.
- **802.1W RSTP**: Rapid Spanning Tree Protocol, improve the convergence time but keeps using one STP for all VLANs in the LAN.
- **802.1S MSTP**: Load balance the STP for different VLANs **(up to 16 VLAN instances):**
  - **PVST+**: **Per-VLAN Spanning-tree**, is a Cisco enhancement of 802.1D **(default on Cisco switches)**, and **for each VLAN the traffic assign a blocked port, optimizing the media usage**. The blocking ports are called "alternate" in this protocol.
  - **RPVSP+**: **Rapid Per-VLAN Spanning-tree**, is a Cisco enhancement of 802.1W, **improved PVST+ convergence time** and also separetes VLAN traffic.

_STP Configuration_

```
!# Displays the STP information (PVST+ default = ieee)
---------------------------------------------------
Note: The command divides in 3 outputs:
 "Root ID" that must match in all VLAN switch members
 "Bridge ID" that is information about that particular switch
 "Interface Info" shows the port role/status
---------------------------------------------------
# show spanning-tree vlan [vlan-id]

!# Display MAC from other connected switches & interfaces
# show mac address-table
```

```
!# Change Spanning-tree mode
(config)# spanning-tree mode {rapid-pvst | pvst | mst}

!# Manipulate Bridge Priority
(config)# spanning-tree vlan [vlan_id] root {primary | secondary}     !# Decrements automatically priority to win Root Bridge or Backup authority
```

_For **switchport access interfaces**, the usage of STP is no needed so we can skip the 50 secs of convergence by **port-fasting frames**_

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if) spanning-tree portfast
(config-if) spanning-tree bpduguard enable
(config-if) spanning-tree guard root !# USE ONLY when we don´t want to receive BridgeID challenges to change the actual Root Bridge through the interface

!# Enable globally BPDU Guard (highly recommended)
# spanning-tree portfast bpduguard default

!# Disables globally STP blocking states (not recommended unless we can garantee no loop in the switch)
# spanning-tree portfast default

```

# **EtherChannel & Link Aggregation (LAG)**

_EtherChannel bundles multiple physical ports into one logical port to increase the media utilization (in most cases blocked by STP). EtherChannel also **load balance the data flow** across to several **same media type links**. The EtherChannel connection requirements are: **matching mode configuration on both sides (depends on protocol used)**, **speed and duplex**, **access or trunk mode**, **native VLAN**, **allowed VLANs** and access VLAN (if is an access switchport)._

_Note: The **load balancing** of EtherChannel (and in almost every Cisco´s protocol) is **NOT Round Robin**. It means that even if links are aggregated or multiple paths know how to reach a network, the device will select only one to foward data and will switch from paths to send several source packets,**without fragmenting them and send them through all the links** (would break some applications if fragments arrives in different order)_

## LACP

_**Link Agregation Control Protocol (LACP)** is the open standard where both sides negotiate the port channel creation and maintenance. It´s the **preferred method to use**. It´s modes can be set as **active-active**, **active-passive** in both LACP sides._

## PAgP

_**Port Agregation Protocol (PAgP)** is the Cisco´s standard where both sides negotiate the port channel creation and maintenance. It´s modes can be set as **desirable-desirable**, **desirable-auto** in both PAgP sides._

## Static EtherChannel

_The switches **do not negotiate portchannel**, but both sides have to be set in mode **on-on** to come up._

```
!# Displays EtherChannel protocol, link status and interfaces aggregated
# show etherchannel summary

!# Verify that STP is not blocking aggregated links
# show spanning-tree vlan [vlan_id]
```

```
(config)# interface range {gigabit | ethernet} {0-X}/{0-X} - {0-X}
(config-if)# switchport trunk encapsulation dot1q
(config-if)# switchport mode trunk
(config-if)# switchport trunk native vlan [native_vlan_ID]
(config-if)# switchport trunk allowed vlan [vlan1_ID, vlan2_ID, ...]
(config-if)# channel-group [channel_ID] mode [{active | passive} | {desirable | auto} | on]
```

# **Access Control Lists (ACLs)**

_The ACLs secure L3 and L4 by **filtering and identifying (QoS) incoming** and **outgoing** traffic. They filter via comparing **Access Control Entries (ACE)** that can match **packets**, **ports** and **transport protocols (TCP/UDP)** in the routering devices._

```
!# Displays detail information of a particular ACL
# show access-list [acl_number | acl_name]

!# Displays ACL implementation on inteface
# show ip interface {gigabit | ethernet} {0-X}/{0-X} | include access list
```

1. **Create ACL Rules**

   _**The ACE ordering in the ACL matters**, the router will interpret (top -> bottom) all the instructions set and included in the policies lists. **By default the ACE last statement (implicity set)** in every ACL is **deny any any** restricting all undeclared traffic._

   ```
   !# Numbered ACL
   (config)#access-list [acl_number] {permit | deny | remark} {protocol} [source_ip_address] [wildcard | host | any] {gt | eq}? [port_number | app_name]? [destination_ip_address] [wildcard | host | any] {qualifier}? [port_number | app_name]?

   !# Named ACL
   (config)#ip access-list {standard | extended} [acl_name]
   (config-std-nacl)# {permit | deny |remark}  {protocol} [ip_address] [wildcard | host | any] {qualifier}? [port_number | app_name]?
   ```

   - **Standard ACL (1-99 & 1300-1999)**: Reference source address only. In standard ACLs the 0.0.0.0 wildcard equal to the "host" keyword is default.

   - **Extended ACL (100-199 & 2000-2699)**: Check for transport protocol, port number and source/destination IP addres.

     **ACL Qualifiers**

   - "**remark**": Adds an entry comment to the ACL
   - "**protocol**": Can be set as **tcp**, **udp**, **icmp**, **ip**, **eigrp**, **ospf** and some others.
   - "**qualifiers**": The way a protocol and port number will match que qualification
     - "**eq**": Match exactly that port number
     - "**lt**": Lower than the port number
     - "**gt**": Greater than the port number
     - "**neq**": All port numbers except the one given
   - "**host**": Keyword that match 0.0.0.0 wildward, aplying the ACL to only one host
   - "**any**": Keyword that match 255.255.255.255 wildcard, aplying the ACL to all subnet hosts

2. **Implement ACL direction (at interface)**

   _We can only apply only 1 ACL for each direction flow in the interface (max 2: 1 in and 1 out)_

   ```
   (config)# interface {gigabit | ethernet} {0-X}/{0-X}
   (config-if)# ip access-group [acl_number |  acl_name] {in | out}
   ```

# **Network Address Translation (NAT)**

_It started when IPv4 became to run out of public addresses registered by the IANA. That was solved by using **private addresses from RFC 1918 in LAN segments** and transalting them via NAT when going out the internet by WAN cannections. **NAT maps statically or dynamically one private IP with a public IP address to allow outside communication**_

```
!# Display the inside & outside (local and global) address mapping
# show ip nat translation

!# Removes all Dynamic NAT translation bindings in order to allow NAT configuration modifications
# clear ip nat translation *

!# Display inside/ouside interfaces and how many packets has been sended & translated
# show ip nat statistics
```

- **Local Addresses:** It´s the actual source & destination IP, from the **sender point of view**.
- **Global Addresses:** They are the transalted addresses that match the **receiver knowledgement of it´s private destination** & **which public (translated) address sends** the package.

## Static NAT

_Maps **1-to-1 private and public addresses**. Usualy used for servers which must **accept incoming connections**._

```
!# Configure interface connected to the WAN (public address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat outside

!# Configure interface connected to the LAN (private address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat inside

!# Map privat-public addresses
(config)# ip nat inside source static [private_ip_address] [public_ip_address]
```

## Dynamic NAT

_Uses a **pool of public addresses** (similar to DHCP). Is used for **hosts that needs to connect externally to the internet but not receiving incoming connections.** The addressing assignment is set as **FIFO**. The NAT pool must have enough addresses for all hosts in the inside LAN._

```
!# Configure interface connected to the WAN (public address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat outside

!# Configure interface connected to the LAN (private address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat inside

!# Set a NAT pool
(config)# ip nat pool [nat_pool_name] [start_public_addresses] [end_public_addresses] netmask [netmask]

!# Create an ACL for all INSIDE hosts that will be using the pool addreses
(config)# access-list [acl_number] permit [ip_matching_for_hosts] [wildcard]

!# Associate the ACL with the NAT pool
(config)# ip nat inside source list [acl_number] pool [nat_pool_name]
```

## Port Address Translation (PAT)

_Allows a **same public address to be re-used by many hosts**, the problem on Dynamic NAT is that we need the same amount (or more) public and private addreses. We can use a **dynamic pool** in which the **port number of the solicitant hosts will only be tracked once the pool reach the last available public address**. The same public address is used and the router know how to reach the host via it´s **unique port number**._

### Well-known Public Address (NO DHCP)

```
!# Configure interface connected to the WAN (public address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat outside

!# Configure interface connected to the LAN (private address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat inside

!# Set a NAT pool
(config)# ip nat pool [nat_pool_name] [start_public_addresses] [end_public_addresses] netmask [netmask]

!# Create an ACL for all INSIDE hosts that will be using the public address
(config)# access-list [acl_number] permit [ip_matching_for_hosts] [wildcard]

!# Associate the ACL with the PAT OVERLOAD
(config)# ip nat inside source list [acl_number] pool [nat_pool_name] overload
```

_The `overload` flag on the last command `enable the port usage for PAT`. But is the same configuration as the Dynamic NAT pool & ACL association._

### Dynamic Public Address (DHCP)

```
!# Configure interface connected to the WAN (public address) given by a DHCP Server in ISP
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip address dhcp
(config-if)# ip nat outside

!# Configure interface connected to the LAN (private address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat inside

!# Create an ACL for all INSIDE hosts that will be using the public address
(config)# access-list [acl_number] permit [ip_matching_for_hosts] [wildcard]

!# Associate the ACL with the PAT OVERLOAD
(config)# ip nat inside source list [acl_number] interface [outside_interface] overload
```

_Here we don´t need to create a pool, and we `match the ACL with the DHCP client interface`. Remember to include the `overload` flag on the last command to `enable the port usage for PAT`. _

# **WAN (Wide-Area-Networks)**

_WAN is a **geographically distributed networks that connects multiple LANs**. A MAN (Metropolitan Area Network) connects devices in a larger geograpical area than a LAN but smaller than a WAN (connection between two institutions with different location offices, remaining in the same state)._

## Leased Lines & Satellite

_Private **dedicated physical connection (P2P)** assigned from the ISP to garantee a **higher SLA (over delay & loss traffic)** to organization usage. Has **fixed reserved bandwidth** because it´s not shared with other companies. Commonly used with fiber-optic media through serial interfaces. The satellite share any fiber leased connection attribute including that are the only feasible medium for unreachable geographical areas._

## Multi-protocol Label Switching (MPLS)

_It´s a dedicated VPN in which the ISP provides the **same network to several companies** in order to communicate them by **sharing the infrastructure (mesh topology) and dividing the traffic**. Exists different SLA depending on the pricing. Every campus of the same company (in different LANs) has access to a **Provider Edge Router (PE)**, that gives them **direct centralized access to the MPLS infrastructure** through several **Customer Edge Routers (CE)** that are multiple points of access (can be departments or subnets) to this centralized PE._

## Point-to-Point Protocol Over Ethernet (PPPoE)

_**Digital Subscriber Line (DSL)**, **Cable** and **Wireless** (ex. 4G) are some of the **less expensive and more commonly used WAN connections** and all of them are part of the PPP suite where the clients can outbound their connection to the ISP infrastructure (internet)._

## WAN Topologies

- **Hub & Spoke | Star Topology:** All WAN access points are **centralized** making it more **simple and secure**. But the links between campus have a **single point of failure with suboptimal traffic flow (more delay)**.
- **Redundant Hub & Spoke:** Keeps the advantages of a simple star topology **increasing the fault of tolerance** in the network at a **higher cost with the same suboptimal traffic flow (more delay)**.
- **Partial Mesh:** The infrastructure **connects any campus to the others in the network**, has an **optimal traffic flow** but with a **more complex infrastructure** and connection cost.

# **Security Landscape**

## Security Terminology

- Threat: Potential cause of hraming IT asset.
- Vulnerability: Weakness that compromise the security/functionallity of a system.
- Exploit: Usage of a vulnerability to attack a system.
- Risk: The possibility of being attacked.
- Mitigation: Technique to eliminate or reduce the potential and seriouness in suffering an attack
- Malware: Any malicious software.
  - Viruses: Software inserted in other software that can be spread by human action.
  - Worms: Similar to viruses but the malware spreading is self-propagating by itself to other systems.
  - Trojan Horses: Often used to back-door installation. Looks like legimit software with attatched malware.
  - Ransomware: Kidnaps the system by encrypting all data with attacker´s key.
- IDS/IPS: Intrusion Detection/Prevention System. Inspect packets up to L7 looking for traffic pattern or anomaly behaviour in the network using signatures to defined policies to block certain trafic. This both mechanisms can be included in Next Generation Firewalls.
- Firewall: One-way protection mechanism, which prevents un-desired traffic to the network (but allows all outgoing communications). They mantain a connection a table contrary of how a Packet Filter (ACL) works.
- Transport Layer Security (TLS): SSL inprovement. Uses symmetric cryptography to encrypt transmitted data, commonly used in the HTTPS protocol for web site authentication.

## **Switch Security**

### DHCP Snooping

_The clients of a DHCP server need a relay agent (ip helper address) to foward the broadcast DHCP Discovery message and be reply with an IP leasing. If **another DHCP server is connected to the same clients subnet this DHCP Server can be the one replying with invalid information for client assignation**. To avoid that we defined a **trusted snooping DHCP server** in the LAN switch to being able to receive incoming DHCP Offers only by a well-known port._

```
(config)# ip dhcp snooping
(config)# ip dhcp snooping vlan [vlan_id] !# epeat for all DHCP client´s VLANs
(config)# interface {gigabit | ethernet} {0-X}/{0-X} !# Use on interface connected to Gateway (with the set ip-helper address)
(config-if)# ip dhcp snooping trust
```

### Dynamic ARP Inspection (DAI)

_**Man-in-the-middle and DoS** attack. A device attatched to the LAN **anticipates any ARP request from hosts to their default gateway** by doing a **Gratuitous ARP (CAM poisoning)**, where the attacker sends a **broadcast message to the switch indicating it´s MAC address and faking the real default gateway IP**. With that done, the switch **register in CAM the MitM fake MAC address** and all hosts will be unicasting traffic through it. To prevent this a **DHCP Snooping needs to be enable so the switch can map trusted assigned IPs with its real MACs**_

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X} !# Use on interface connected to Gateway (with the set ip-helper address)
(config-if)# ip arp inspection trust

(config)# ip arp inspection vlan [vlan_ID] !# epeat for all DHCP client´s VLANs
```

### Identity Based Networking (802.1X)

_Any **host (Supplicant)** that wants to connect in LAN or access WAN first send a message to the network **device (Authenticator)**, wich only conmmutes traffic to an **Authentication Server**. Then the supplicant is being asked for its username and password credentials and travel all the way back to the Autheticator > Authentication Server. **If the credentials are correct**, the Authenticator replys the Suplicant and the **commutation to other networks**, **interfaces in LAN (can be map to VLAN)** and **internet is allowed**._

### Switchports Security

_Defined a **mapping between allowed MAC addresses that can be attatched to certain switchport** and the **mechanisms in case of switchport violation**_

```
!# Display interface security settings
# show port-security interface {gigabit | ethernet} {0-X}/{0-X}

!# Display port-MAC association table
# show port-security address

!# Displays port summary its status and violations
# show port-security
```

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# switchport mode {access | trunk}
(config-if)# switchport port-security !# y default allow 1 MAC at the time (no mattering the exact MAC attatched) and shutdowns if more than 1 hosts are connected
(config-if)# switchport port-security violation {shutdown | protect | restrict}
(config-if)# switchport port-security maximum [max_MAC_allowed]
(config-if)# switchport port-security mac-address {sticky | MAC_address} !# ssociate specific MAC to the port to be learned in next plug-in or manually set
```

- **Shutdown (default)**: The interface is placed in **error-disabled state** blocking all traffic. To enable the port again, you need to manually de-attatch any unauthorized MAC, shutdown and bring up the interface. It can also be **re-activated automatically again by a timer** using following commands:
  ```
  (config)# errdisable recovery cause psecure-violation
  (config)# errdisable recovery interval [secs]
  ```
- **Protect**: Traffic from unauthorized users is blocked and **fowards only authorized MACs**.
- **Restrict**: Keeps track of unauthorized addresses, **logging them in a violation counter** and dropping it´s traffic.

## Virtual Private Network (VPN)

_Provides a **virtual tunnel between LANs** across a **shared public channel** where **traffic is encrypted and authorized** only when AAA from source succeeds. Typically a VPN is **set on both tunnel sides a firewall** and securely sended by the **IPsec or SSL protocols.** The **Site-to-site VPNs (basic P2P connectivity)** uses **AES** algorithm for encryption and some hashing mechanisms as **MD5 or SHA** to evaluate data integrity._

- **Split Tunneling:** Is a VPN strategy that divides user traffic, if the host wants to connect to the corporate network it has to pass all across the VPN tunnel until reach the edge Firewall, but for other trafic such as internet the data is not process and allows rapid connectivity. **Better performance**.
- **Full Tunneling:** All host traffic have to pass through the VPN tunnel and it´s audited by the edge Firewall. Can be a secure way to permit/deny user´s traffic to public internet but comes with more delay in the network response and connectivity. **More secure**.

- **Remote Access VPN:** is a private/secure connection from any host (by a software) from the internet to the organization firewall via a VPN tunnel.
- **IPsec Tunnel:** Open standard that encrypts L3 traffic but **do not support multicast (for routing protocols).** It´s the common Site-to-Site VPN mechanism. Uses **Internet Key Exchange (IKE) for sharing pre-generated keys** in both tunnel sides.
- **Generic Routing Encapsulation (GRE):** It´s an improvement over IPsec that allows multicasting
- **IPsec Virtual Tunnel Interface (IPsec VTI):** Cisco´s implementation of IPsec + GRE
- **Dynamic Multipoint VPN (DMPVPN):** Cisco´s. Scalable and simple hub and spoke style configuration that enables direct full mesh connectivity between offices.
- **FlexVPN:** Cisco´s. Similar to DMPVPN but newer.
- **Group Encrypted Transport VPN (GETVPN):** Cisco´s. Scalable and centralized VPN policy over non-public infrastructure (MPLS)

# **Simple Network Manager Protocol (SNMP)**

_Open standard for network device monitoring. Centralized device management in a **SMNP Manager (Network Management System Server NMS)** that **pull information/variable status (GET)** form other SMNP Agents and collects it in a **Management Information base (MIB)**. The SNMP Agents can also notify the server when a sensor or link went down by **pushing information/variable status (TRAP) into the server**. The actual version is **SMNPv3** that **supports encrypted community strings (password shared along server and agents) and authentication** is the most recommended to implement but can be not supported depending on the device. SNMP monitors network devices by defined variables (interface status, traffic, sensors, etc...) **each trackable variable from an SNMP Agent is called Object Identifier (OID)**. SNMP is part of NMS systems and can provide reports of the network devices._

_**Syslog vs. SNMP**: Even that **both can store logging information**, is that on **Syslog** we **store with more granularity all system reports** and with **SNMP** the **reporting is less detailed** but we can have **PUSH functionality to configure agents** which Syslog can´t do._

**SNMPv2**
_Use SNMPv2 only when SNMPv3 is not supported. The Community Strings travels as plain text._

```
!# Optional: Identificate SNMP AGENT to the SNMP MANAGER (recommended)
(config)# snmp-server contact [snmp_agent_identifier]
(config)# snmp-server location [location_description]

!# Configure community string & privileges for SNMP AGENTS
(config)# snmp-server community [community_string] [privilege]
!# Privilege types: "ro" = read-only , "rw" = read-write

!# Reference the SNMP MANAGER (for each agent)
(config)# snmp-server host [snmp_server_ipv4] [community_string_for_desired_privileges]
(config)# snmp-server enable traps [action_that_tiggered_trap_to_server]
```

**SNMPv3**
_SNMPv3 **encrypts and authenticates** communication between SNMP Manager and it´s Agents. It´s needed to set up **2 types of password**: an **authentication passoword** that will use the agent to verify is itself and a **private password** shared in the communication to allow encrypted privacy in agents communication._

```
(config)# snmp-server group [snmp_group_name] v3 {auth | noauth | priv*}

(config)# snmp-server user [snmp_agent_username] [snmp_group_name] v3 auth {md5 | sha*} [auth_password] priv {3des | aes* | des} [bits_for_encryption] [private_password]
```

_The **credentials for authentication** and **privacy encryption must match** while configuring the SNMP Manager, and it will depend on the software used on server side._

# **Quality of Service (QoS)**

_QoS basis on queuing theory **(FIFO)**. Modern **converged networks centralize data, video and voice (VoIP)** in the same network infrastructure making all services share the same bandwidth, wich can cause problems related with **jitter (variation in delay)**, **latency (delay)** and **loss of data (dropped packets when queue buffer is full)**._

_The point of having QoS is to **classified network traffic with FIFO queuing for dispatch packets in a prioritized order** dependig on it´s type, cost and demand. But this means that if a service is getting better attendance (usually VoIP have better QoS because it´s traffic packets are very small) the other services start to get worst performance. QoS also looks for a congestion-free network. The **congestion is caused by having higher input bandwidth than the output** ISP bandwidth can handle. **QoS only mitigates temporary periods of congestion, doesn´t solve it** (the only way to avoid congestion is to have same speed bandwith for in-out traffic, which cost more money)._

## Classification & Marking

_To know how kind of packets prioritize over the network we need to classify them based on the following characteristics._

### Class of Service (CoS)

_**L2 marking**. Is set in a **3 bit space into the 802.1Q header** that can go from 0-7, giving more priority in dispatching over packets marked at a higher number. The **CoS 0 (default)** is set for **best-effort traffic (BE)**._

- **CoS 7 & 6** are reserved for network usage (routing protocols and network control).
- **CoS 5 (EF)** is used for **VoIP payload (spoken voice)**. EF = Expedit Forwarding.
- **CoS 3 (AF)** is used for **voice signalling (the phone ringing & synchronization)**. AF = Assured Forwarding.

### Differentiated Service Code Point (DSCP)

_**L3 marking**. Is set in a **6 bit space into the IP header**, also called **Type of service (ToS)**. Dispatchs first packets marked with higher numbers from 0-7 with 64 different combination possible. As CoS the **ToS 0 (default)** is set for **best-effort traffic (BE or DF)**. Each numeric value of ToS can be called by an specific identifier which means the same._

- **ToS 46 (EF)** is used for **VoIP payload (spoken voice)**. EF = Expedit Forwarding.
- **ToS 34 (AF41)** is used for **Interactive video (SD)**.
- **ToS 32 (CS4)** is used for **Streaming video**.
- **ToS 26 (AF31)** is used for **Mission Critical Data**.
- **ToS 24 (CS3)** is used for **Voice Signalling (the phone ringing & synchronization)**. CS3 = Call Signalling Traffic.
- **ToS 8 (CS1)** is used for **Scavanger** traffic. Considered "junk traffic" that uses bandwidth (like gaming, peer-to-peer media sharing or worms) and slow-downs the overall performance in the networks.

### ACLs & Network Based Application Recognition (NBAR)

_In **CoS & DSCP the traffic is self-marked by it´s traffic source** but we can **manually define QoS marking by using ACLs** to **identify L3 and L4 traffic** that we want to prioritize._

_NBAR similarly to the ACL usage utilize well-known **data signatures from L4 up to L7** that Cisco´s have defined and we can use them to identify and then prioritize traffic._

## Shaping & Policing

_Can be used to **control/limits traffic rate** and **take actions if the rate gets above a configured limit**._

- **Shaping:** **Buffers the excedant traffic** and dispatch it when the traffic slow downs.
- **Policing:** **Drops or re-marks excess traffic** to enforce the specified rate limit.

## Congestion Management

_It´s the manipulation of the queue so get better service to the traffic that requires it. Cisco QoS uses **Modular Queuing CLI (MQC)** divided in **3 main sections**, **Class Maps** to define the traffic on interest, **Policy Maps** that sets the rules, priorities and actions to the traffic and the **Service Policies** which apply the policies to an interface._

- **Class Based Weighted fair Queuing (CBWFQ):** Gives **bandwidth guaranty based on traffic types**.
- **Low Latency Queue (LLQ):** Similar to CBWFQ but with a **priority queue**.

```
!# Group the interest traffic (Class Map)
(config)# class-map [traffic_group_name]
(config)# match ip dscp [dscp_alias] !# As "ef", "cs3", "af41", etc...

!# Define the percentage of utilization in case of congestion (Policy Map)
(config)# policy-map [policy_name]
(config)# class [traffic_group_name]
(config)# shape average [bandwidth_limit_rate_to_start_buffering]
(config)# priority percent [bandwidth_guaranteed_utilization_in_case_of_congestion]
(config)# {service-policy [nested_policies]}?

!# Setup utilization & shaping for remaining traffic types
(config)# class class-default
(config)# shape average [bandwidth_limit_rate_to_start_buffering]
(config)# fair-queue

!# Apply policy to the outbound interface in WAN for dispatching (Service Policy)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# bandwidth [100%_of_capable_interface_bandwidth_kbps]
(config-if)# service-policy out [policy_name]
```

# **Cloud Computing**

_Some cloud advantages, listed as follows:_

- **Scalability:** Ability to regulate the service in accordance with their current requirements. Flexibility through cloud bursting.
- **Business Agility:** Handle expected/unexpected changes in load. Reduced time to deploy an application into production.
- **Cost Efficiency:** Directly proportional cost with no provicioning for the peak as a permanent fixture. Large CapEx costs reduces to monthly OpEx cloud rentals. No owned hardware depreciation.
- **Competitive Advantage:** Organizations can respond quicly to evolving market trends and focus on growing their core business.
- **Productivity:** The IT staff focus in improving core applications rather than mantain or troubleshoot hardware infrastructure.
- **Availability & Reliability:** The provider data center is built by qualified specialist according to best practices such as ISO 9001(QoS) and 27001 (security). SLA guaranteed.

## Cloud Characteristics & Definitions

- **On-demand Self-service:** The clients define/configure the amount of network resources to use.
- **Rapid Elasticity:** The resources needed can increase/decrease according to the demand.
- **Broad Network Access:** The cloud services can be reached from any place over a network connection.
- **Resource pooling:** The resources are shared between several users (lowering the costs compared with On-Premises infrasctructure CapEx)- No separated deedicated servers. Virtualization.
- **Measured Service:** The clients only pays for the cloud resources provisioned.

**Virtualization:** It allows resource pooling where multiple hosts **share the same underlying hardware**.

**Clustering:** Contrary on virtualization is the combination of **multiple physical systems into a single virtual systems**. It provides redundancy and improved performance.

**Hypervisor:** Is software that allows one host computer or server to support multiple guest machines by virtually sharing hardware resources such as memory and processing.

- **Type 1 Hypervisors:** It´s run on top the **hardware** to allow multiple VMs. Used in Data Centers. VMware ESXi (vSphere suite), Microsoft Hyper-V, Red Hat KVM, Oracle VM Server, Citrix XenServer.
- **Type 2 Hypervisors:** It´s **run on top the host´s OS** for lab testing & virtualization. VMware Workstation, Parallels, VirtualBox, QEMU.

## Cloud Service Models

_Define where the customer and provider areas of responsability are, and at what level the customer gains access. The three models build in top of one another._

- **IaaS:** Infrastructure as a service. The provider supports the **facility**, **network**, **storage**, **computational power** and **hypervisor** needed to the customer needs. Only Data, Applications and OS is supported and mantained by the customer side.
- **PaaS:** Platform as a service. Combines all **IaaS services supported + the OS** that the host is running to provide the customer a **custom environment** to work with.
- **SaaS:** Software as a service. All the cloud services are offered on top the data center stack. The hosts can be running a **complete pre-defined data center environment and applications in the providers side** (as Office365) without having any infrastructure or specialized setup.
- **COLO:** Bare metal solution, **not cloud service model**. The **data center facilities (electricity, location, cooling, etc...) are set by a provider** with an space rental OpEx, but the client must bring their own hardware & software infrastructure to be cluster.
- **On-Premise:** Bare metal solution, **not cloud service model**. The network, computing and application infrastructure **(all data center) is CapEx provided by the customer**.

## Cloud Deployment Infrastructures

- **Public Cloud:** Common deployment model. Is an open model where anyone can use cloud´s infrastructure which it may be owned, managed and operated by a business. **AWS**, **MS Azure**, **IBM Bluemix**, **Salesforce**.
- **Private Cloud:** The **cloud infrastructure is provisioned for a single organization** (comprising multiple clients or business units). Most suitable for large companies because it represents an outweight initial effort and cost to setup infrastructure and automate workflows. The cloud can be provided by a third party or built in totally by the same user (ex. US Department on Defense Cloud mounted with AWS)
- **Community Cloud:** Cloud infrastructure provided for exclusive use by a group of organization that share the same cloud requirements (mission, security, policy and comliance considerations). Least common deployment model, used in governments environments.
- **Hybrid Cloud:** Combination on any cloud deployment infrastructures. Many clouds remain as unique entities but are bound together by standarized or propietary technologies that enables data and application portability. **Cloud bursting:** is the **usage of external cloud infrastructures to expand capabilities** (ex. if a private cloud demands more memory it can access the resources from a public cloud).

# **Wireless Networking**

_WiFi services are defined in the IEEE **802.11** standard._

- **Ad-Hoc Networks:** 2 or more stations (wireless devices) communicate directly with each other. **Peer-to-peer** connection with no scalability pourposes. Also known as **Independent Basic Service Set IBSS**:
  - **WPAN:** Wireless Personal Area Network. Devices are **<10 mts** like **Bluetooth**.
- **Infrastructure Mode:** Stations (wireless devices) **communicacte via a centralized Wireless Access Point (AP)**:
  - **WiFi Direct:** Operates connecting devices to an AP but also allow them to be part in a peer-to-peer communication. Doesn´t operate in Ad-Hoc IBSS mode even it can connect directly stations. It´s a WPAN derivation for infrastructure mode.
  - **WMAN:** Wireless Metropolitan Area. Covers a large area such as a city.
  - **WLAN:** Wireless Local Area Network. Provide access to a campus network (wired), without need for a cable. Devices are **<100 mts** of an **Access Point**.
- **Mesh Networks:** One AP ratio (signal frequency, often 2.4 GHz) is used to serve clients and the other connects to the backhaul (traffic dedicated to connect network devices, 5 GHz). Poppular nowadays for home usage in dual-band supported devices.

- **Wireless Bridges:** Connect wired network infrastructures to others wirelessly. Used for maintain connectivity between buildings where a cable is not possible.
- **Wireless Access Point (AP)**:
  - All AP speed is **Half-duplex**.
  - Provides connectivity between wireless stations and the rest network (wired) via a **Distribution System (DS)** that connects the AP to the wired networks.
  - Centralize access and control to the stations in an infrastructure mode, **the coverage area of this operations is called a Basic Service Area (BSA)**.
  - All the devices in the same BSS must be identified by a **BSSID retreived from it´s MAC address**.
  - All the coverage area from an AP, called **BSA or wireless cell**, is also **identified by one or multiple Service Set ID (SSID)**. Each SSID can have independant security settings and can be mapped to different VLANs.
  - The AP **broadcast** its WLAN information and SSID requirements through **beacon frames**. Can be disabled.
  - An **SSID can be replicated accross multiple AP** to give a **larger coverage area** through different frequency channels, known as **Extended Service Set (ESS)**

_Wireless ISM Frequencies_

- **2.4 GHz**: 802.11, 802.11b, 802.11g
- **5.0 GHz**: 802.11a, 802.11ac
- **2.4 & 5.0 GHz (dual-band)**: 802.11n

_Wireless Security Standards_

- **WEP**: Wireless Equivalent Privacy. RC4 Encryption.
- **WPA**: WiFi Protected Access. RC4 & TKIP Encryption.
  - **WPA Personal**: Uses pre-shared keys PSK´s
  - **WPA Enterprise**: Uses a **RADIUS AAA** server with protocol **802.1X**
- **WPA2**: AES & CCMP Encryption
- **WPA3**: AES, CCMP & KRACK attack protection.

## Wireless Lan Controller (WLC)

_Used as a central point of management for several Access Points. The 2 possible modes an Access Point operates is **standalone (autonomous system)** or **lightweight (controlled by a WLC)** based in the OS image installed. The way an AP in lighweight mode connects to the WLC is set as a **zero-touch provisioning (ZTP)** where the AP discovers by **DHCP(option 43)/DNS** the IP connection to the WLC and when established the AP downdloads it´s configurations from it. A WLC functions are: **authentication**, **roaming control**, **802.11-802.3 communication**, **radio frequency**, **security** and **QoS management**_
_The protocol used for WLCs to manage APs collections is the **Control And Provisioning of Wireless Access Points (CAPWAP)**. All WLC-AP communications are **encrypted inside DTLS CAPWAP tunnel** using **UDP ports 5246 & 5247**._

1. Configure Switch -> WLC connection

   ```
   !# Configure a DHCP for Access-Point addressing and connection to WLC
   (config)# ip dhcp excluded-addresses [start_excluded] [end_excluded]
   (config)# ip dhcp pool [pool_name]
   (dhcp-config)# network [access_points_network] [netmask]
   (dhcp-config)# default-router [management_svi_ipv4]
   (dhcp-config)# option 43 ip [wlc_ipv4]

   !# Associate a VLAN + SVI for each SSID & the management of wireless devices
   (config)# vlan [vlan_ID]
   (config-vlan)# name [ssid_name]
   (config-vlan)# exit
   (config)# interface vlan [vlan_ID]
   (config-if)# ip address [default_gateway_for_ssid_members] [netmask]

   !# Configure LAN Switch -> WLC link connection as TRUNK mode
   (config)# interface {ethernet | gigabit} {0-X}/{0-X}
   (config-if)# description [describe_link_2_WLC]
   (config-if)# switchport trunk encapsulation dot1q
   (config-if)# switchport mode trunk
   (config-if)# switchport trunk allowed vlan [management_and_ssid_vlans]

   !# Configure LAN Switch -> each Access-Point
   (config)# interface {ethernet | gigabit} {0-X}/{0-X}
   (config-if)# description [describe_link_2_AP]
   (config-if)# switchport mode access
   (config-if)# switchport access vlan [ssid_vlan_ID]
   (config-if)# spanning-tree portfast
   ```

2. Add any RADIUS AAA Server for WPA2 Enterprise authentication
3. In WLC:
   1. Create a DHCP pool for each SSID to addressing wireless hosts.
   2. Associate a logical interface to each SSID
   3. Link the WANs to their respectives logical interfaces, configuring AAA & security services
4. Verify hosts connectivity to the A

- **Standalone Access-Points**: _The link between the LAN switch & the Access-Point must be **trunk**, including all **SSID VLANs**._
- **Lightweight/Managed Access Points**: _The link between the LAN switch & the Access-Point must be **trunk**, including all **SSID VLANs**. The link between LAN switch & the Access-Point must be **access** including only the **AP Management VLAN**. The lightweight Access Points can support some WLC **real-time operations** like **client handshake**, **beacon** announcement, **performance monitoring**, **encryption/decryption** and **communicate clients in power-safe mode** in order to facilitate WLC & network performance. The **Flex-Connect** protocol **enables an lightweight AP to communicate hosts in the same BSA without passing the traffic through the WLC** to improve the LAN response performance and velocity._

# **Network Automation & Programmability**

_The automation of networks through programmability is used for:_

- Device configuration (Multiple devices in batch)
- Initial device provisioning
- Software & Configuration version control
- Collecting statistics from devices
- Compliance verification
- Reporting & Assurance (standarized device configuration)
- Troubleshooting (Automatically correction on events by scripts)

_Data Serialization_

- **JSON**: Readable, lightweight, most popular formatting type. Object/Dictionary (key-value pair) + Array with nesting data structures. REST.
- **XML**: Data contained in object tags. SOAP.
- **YAML**: Block-level identation, key-value pair & ´-´ identification of sections and data structures
- **YANG**: Standarized data modelling for operational and configuration of a network device.

## Application Programming Interfaces (API)

```
Protocol: http | https
URN: www.demo.cisco.com/api/resouce.html
URL: https://www.demo.cisco.com/api/resouce.html
URI: https://www.demo.cisco.com/api/resouce.html#fragment
Fragment: #fragment
```

_Network Management APIs_
**Northbound APIs**
_Connects the Control Layer with the upper application layers in most cases using REST APIs_

**Southbound APIS**
_Divides the SDN management into layers, usually exists an Infrastructure Layer (where all net hardware lives) and a Control Layer where the SDN Controller (DNA Center) manage everything_

- **NETCONF**: Remotely reads or applies changes to the network device. XML encoded. Uses YANG. Divided in **Content**, **Operations**, **Messages (Remote Procedure Calls RPC)** and **Transport (SSH or TLS)**.
- **RESTCONF**: Built over NETCONF, uses XML or JSON. Transport via HTTP and HTTPS.
- **gRPC**: **Google RPC open source**. Google Protocol Buffers encoding is used. Transport runs uver HTTP/2. Used to collecting telemetry statistics for IoT.

_API Operations_

- Create - POST
- Read - GET
- Update - PATCH
- Delete - DELETE

_Response Codes_

- 1XX: Informational
- 2XX: Success Connection
  - 200: OK
  - 201: Created
  - 204: No Content (successfully deleted)
- 3XX: Redirection
- 4XX: Client Error
  - 400: Bad Request | Malformed Syntax
  - 401: Unauthorized
  - 403: Forbidden
  - 404: Not Found
- 500: Internal Server Error

## Configuration Management Tools

_Centralize API management devices with little programming knowledge. Have established development practices including version control and testing._

### Ansible

- Popular Cisco´s devices management choice
- **Ansible Playbooks**: YAML files that instucts running settings
- **Ansible Inventory**: Python script modules/files defining all managed hosts by the **centralized control station without extra software**.
- Python2/Python3. Simpler to use.
- **Agentless**, don´t need plugins on devices to be managed.
- **Push model**, insert configurations to net devices.
- Communicates SSH by default

```
// Verify Ansible is running on Unix
$ cd ansible
$ ansible localhost -m ping

// Verify registered network hosts to manage (FQDNs)
$ sudo cat /etc/hosts

// Verify Ansible Inventory
$ sudo cat /etc/ansible/hosts

// Verify YAML config file for a device
$ cat host_vars/[device_fqdn].yml

// Verify Ansible connectivity through all Inventory
ansible$ ansible -m ping all

// Push configurations (YAML) to device(s)
ansible$ ansible-playbook [yaml_config_file].yml
```

### Puppet

- **Puppet Master**: Linux server that controls other agents
- **Manifest**: Defines the devices properties. Used to **monitoring configuration consistency**
- Ruby. Use of a propietary DSL rather than YAML.
- **Agentful**, needs an agent (plugin) running on net devices.
- **Pull model**, checks every 30 mins (default) net device status

### Chef

- Similar to Puppet with terminology change to ¨Recipies¨ and ¨Cookbook¨.
- Ruby scripted.
- Pull model.
- Agentful

## Software Define Networking

- Data Plane (Forwarding): The plain data received/sent that only travereses the device.
- Control Plane: Traffic dedicated on how the device will forward the traffic (ex. routing protocols, Spanning-tree updates) based on protocols decisitions.
- Management Plane: Traffic dedicated to the device configuration such as SSH, SNMP or an API

_In traditional networking infrastructures the devices manage their own data and control plane individually. With SDNs the data plane is isolated from the control plane by managing this one in a centralized controller. In Hybrid SDN architecture the devices keep retaining a little of the control plane intelligence but the majority of this stills managed by other entity._

**Cisco´s SDNs**

- **APIC**: Application Policy Infrastructure Controller. Designed to **manage data center** environments with Nexus Switches. Uses Cisco ACI.
- **DNA Center (legacy APIC-EM)**: Enterprise Module. Designed to manage enterprise environments such as campus, branch and WAN. Runs on **Cisco UCN**. **Intent Based Networking (IBN)** defining Application Policies for QoS assurance from a dashboard that manage globally configurations and monitors the network.

# **Notes**

**Show vs Debug:**

- **Show:** Command as "show running-config" presents a snapshot of time of the particular configuration at that time
- **Debug:** By the other hand, a "debug" command will prompt every change made once it starts to present any log

**NetConf OSX Commands**

- Windows
  - Display network addressing and configurations
    ```
    C:> ipconfig /all
    ```
  - DHCP binding (remove leased IP and assing another)
    ```
    C:> ipconfig /release
    C:> ipconfig /renew
    ```
  - Display/Flush ARP Table
    ```
    C:> arp -a
    ```
    ```
    C:> arp -d
    ```
- Unix & Linux

  - Display network addressing and configurations

    ```
    $ ifconfig

    !# In modern Linux the command is:
    $ ip address show
    ```

  - Display rounting info (default gateway)

    ```
    $ netstat -rn

    !# In modern Linux the command is:
    $ ip route show
    ```
