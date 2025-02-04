# CCNA 200-301

# **Device Management**

**Basic device setup**

```
!!! Device name
(config)# hostname [hostname]

!!! Encrypted Privileged EXEC mode password
(config)# enable secret [password]

!!! Encrypt Running Config passwords
(config)# service password-encryption

!!! Display a prevention access message (when CLI prompts)
(config)# banner login $[Message_goes_here]$

!!! Display welcome message (after user EXEC mode login)
(config)# banner motd $[Message_goes_here]$

!!! Disable DNS translation lookup
(config)# no ip domain-lookup

!!! Minimun password length allowed
(config)# security passwords min-length 12
```

**Time Synchronization | Network Time Protocol (NTP)**
_**IMPORTANT:** Configure correctly in all devices so, **certificates**, **protocols** and **syslog** can be running correctly. The **convergence in NTP servers, is very slow ( 300 seg = 5 min )**_

```
!!! Displays the current date and time
# show clock
!!! Verifies if NTP clock is synchronized with the master NTP server
# show ntp status

!!! Set time manually
# clock set HH:mm:ss [day] [month_3_letters] [year]

(config)# clock timezone UTC -6 !!! MX UTC
(config)# ntp master  !!! Configure device as NTP SERVER
(config)# ntp server [ntp_server_ipv4]  !!! Configure device as NTP CLIENT
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
!!! Displays a syslog summary and buffered stored logs (RAM)
# show logging
```

```
!!! Disable syslog in line console
(config)# no logging console

!!! Displays events with X severity or higher in VTY lines
(config)# terminal monitor !!! Enable seeing the console logs in VTY lines
(config)# logging monitor [severity_number]

!!! Store in RAM logging buffer events with severity X or higher
(config)# logging buffered [severity_name]

!!! External Logging Server
(config)# logging [logging_server_ipv4]
(config)% logging trap [severity_name]

!!! Turn-off debugging output
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

!!! Enable/Disable CDP for security
(config)# [no]? cdp run
```

_The open standard Link Layer Discovery Protocol differs (LLDP) on CDP in only discover 1 device per port (by physical interfaces), is disabled as default and not supported for all devices. Each LLDP discover message is sent **30 sec** by default_

```
# show lldp neighbors [detail]?

!!! Enable/Disable CDP for security
(config)# [no]? lldp run

!!! Allowing transmission on interface
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

!!! Remove RSA key
(config)# crypto key zeroize rsa

!!! Verify RSA keys
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
!!! Block VTY connections for N seconds after X simultaneously failed attemps between Y seconds
(config)# login block-for [seconds_blocked] attempts [failed_attemps_number] within [interval_of_failing_seconds]

!!! Enable secure HTTPS SSH access
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
!!! Setup a backup route (only enable to use if RADIUS or TACACS+ servers can´t be reached)
(config)# username [backup_username] secret [password]

!!! Configure AAA external usage
(config)# aaa new-model

!!! Repeat commands to register multiple AAA servers
(config)# [radius | tacacs] server [server_name]
(config-radius-server)# address ipv4 [ipv4_server]
(config-radius-server)# key [server_name_locally_significant]

(config-radius-server)# aaa group server [radius | tacacs+] [aaa_group_name]
(config-sg-radius)# server name [server_name] !!! Repeat command as many RADIUS servers registered

(config-sg-radius)# aaa authentication login default group [aaa_group_name] local
```

# **Interface Configuration**

## Interface Speed (full/half duplex)

_Note: By default all interfaces are set as **auto** negotiating the max speed in link.**Both sides in the link must be in the same** (auto-auto or manually changed in both) state so the link can work properly_

```
!!! Set manually the speed is recommended only on small networks
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# duplex [auto|full|half]
(config-if)# speed [mbps] !!! Can´t be set on virtual devices (needs to have the physical cable attatched)
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
!!! Displays the number of leased/available addresses and pool data
# show ip dhcp pool

!!! Displays relation between clients(MAC Addresses) and their leased IPs
# show ip dhcp binding

!!! Examinates the DHCP messages
# show ip dhcp server statistics
```

#### DHCP Server

##### **DHCPv4 Server (Static Binding)**

Assigns the same IP to a host based on it´s MAC address (1-on-1) commonly used for servers. The IP address will always be assigned to one particular host.

```
# debug ip dhcp server packet

(config)# ip dhcp pool [dhcp_pool_name]
(dhcp-config)# host [host_ip_address] [netmask]
(dhcp-config)# client-identifier [host_mac_address]

# undebug all
```

```
!!! DHCPv4 Client (Static Binding)
(config)# interface [interface]
(config-if)# ip address dhcp
(config-if)?# ip dhcp client client-id ascii [client_id_text]
```

##### **DHCPv4 Server (Dynamic Pool)**

```
(config)# ip dhcp excluded-addresses [start_ip_range_excluded] [end_ip_range_excluded]   !!! Not used for IPv6
(config)# ip dhcp pool [dhcp_pool_name]
(dhcp-config)# network [dhcp_network_to_assign_addresses] [netmask]
(dhcp-config)# default-router [default_gateway_ip_to_propagate]
(dhcp-config)# dns-server [dns_server_ip_to_propagate]
(dhcp-config)# lease [days] [hours] [minutes]
(dhcp-config)# domain-name [domain_name]
(dhcp-config)# option {43 | 69 | 70 | 150} [server_ipv4] !!! WLC, SMTP, POP3, TFTP
```

```
!!! DHCPv4 Client (Dynamic Binding)
(config)# interface [interface]
(config-if)# ip address dhcp

# renew dhcp [interface]
```

##### **DHCPv6**

_DHCPv6 represent a significant network security because it keeps track of IPv6-MAC addresses leased and because IPv6 is based on unicasting traffic an attacker can connect directly to a host. That´s why **the only reason to have DHCPv6 enable is for DNS acknowlegment** and we must **disable any possible IPv6 assignation from DHCP** converting all DHCP requests into **Stateless** to only conserve SLAAC IPv6_

###### **Stateful DHCPv6**

```
!!! Stateful Server
(config)#ipv6 unicast-routing

(config)# ipv6 dhcp pool [dhcp_pool_name]
(config-dhcpv6)# address prefix [ipv6_prefix_with_cidr]
(config-dhcpv6)# dns-server [dns_server]
(config-dhcpv6)# domain-name [domain_name]

!!! Interface facing to host´s LAN
(config)# interface [interface]
(config-if)# ipv6 dhcp server [dhcp_pool_name]
(config-if)?# ipv6 nd managed-config-flag

# show ipv6 dhcp pool
# show ipv6 dhcp binding
```

```
!!! Stateful Client
(config)#ipv6 unicast-routing

(config)# interface [interface]
(config-if)# ipv6 enable
(config-if)# ipv6 address dhcp
(config-if)# no shutdown

# show ipv6 interface brief
# show ipv6 dhcp interface [interface]
```

###### **Stateless DHCPv6**

```
!!! Stateless Server
(config)#ipv6 unicast-routing

(config)# ipv6 dhcp pool [dhcp_pool_name]
(config-dhcpv6)# dns-server [dns_server]
(config-dhcpv6)# domain-name [domain_name]

!!! Interface facing to host´s LAN
(config-if)# ipv6 enable
(config-if)# ipv6 dhcp server [dhcp_pool_name]
(config-if)# ipv6 nd other-config-flag

# show ipv6 dhcp pool
# show ipv6 dhcp binding
```

```
!!! Stateless Client
(config)#ipv6 unicast-routing

(config)# interface [interface]
(config-if)# ipv6 enable
(config-if)# ipv6 address autoconfig
(config-if)# no shutdown

# show ipv6 interface brief
# show ipv6 dhcp interface [interface]
```

###### **SLAAC DHCPv6**

By using Stateless Address Auto-Configuration (SLAAC) the **router advertise their interface IPv6 subnet (Global Unicast address)** to all hosts by ICMP and use it to assign them addressing. A **DHCPv6 is still required** because **SLAAC ONLY advertise the subnet**, **assign addresses** and **it´s default gateway** but **CAN´T setup a DNS server for hosts**. When a host is using SLAAC it will send all traffic through **:: (unespecify address)**. SLAAC doen´t use ARP, it uses simmilar packets known as **Neighbor Discovery (Solicitations & Advetisements) through ICMP**.

_Note: SLAAC uses the first IPv6 that finds in it`s same broadcast segment, you must be sure that at least one neighbor is currently using an IPv6 addressing reachable from the interface in order to auto-configure it_

```
(config)# ipv6 unicast-routing

(config)# interface [interface]
(config-if)# ipv6 enable
(config-if)# ipv6 address autoconfig
(config-if)# no shutdown
```

#### DHCP Relay Agent

The **DHCP Discovery message (broadcast) is NOT forwarded for a router** if we have the DHCP server in other network. To enable DHCP communications and addressing response from one network to another we need to do the following configuration on any router in the middle of the client and server. If a **client is not reaching the DHCP server** it´ll have an **169.254.0.0** address.

_Note: The relay configuration using the `ip helper-address` command, must be entered on the router interface that is facing the hosts LAN_

```
!!! Router (non-DHCP server) interface connected to host´s LAN (the default gateway for clients)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}

!!! DHCPv4
(config-if)# ip helper-address [next_hop_IP_to_reach_dhcp_server]

!!! DHCPv6
(config-if)# ipv6 dhcp relay destination [next_hop_IPv6_to_reach_dhcp_server] [exit_interface_to_DHCPv6_server]
(config-if)# ipv6 nd managed-config-flag  !!! Needed when enableing STATEFUL DHCP only

Note: For next-hop IP, use the opposite address in the point-to-point link between the relay router (configuring) and the one that is the DHCP server (the one that has the next-hop IP locally connected)
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

!!! Display link-local and global unicast address from neighbors
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
!!! Enable IPv6 usage
(config)# ipv6 unicast-routing
```

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# description [interface_description]
(config-if)# no shutdown

!!! Global Unique addressing
(config-if)# ipv6 address [ipv6_global_unique]/[cidr]?
!!! EUI-64 autogenerated addressing
(config-if)# ipv6 address [ipv6_64_network_prefix]/64? eui-64

!!! Override the EUI-64 default link-local
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
!!! Displays the routing table
# show ip route

!!! Displays the routing protocol running
# show ip protocols

!!! Display protocol configuration only
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
!!! Show adjacency via EIGRP
# show eigrp neighbors

!!! All eigrp enabled interfaces and it´s statistics
# show ip eigrp interfaces
```

```
!!! Router EIGRP process level
(config)# router eigrp [autonomous_system]
(config-router)# eigrp router-id [router_id]          !!! Manual Router-ID assignation
(config-router)# network [network_to_advertise] [wildcard]
(config-router)# default-information originate        !!! Default routes injection
(config-router)# redistribute {static | ospf | rip}   !!! Specific routes injection from other protocols

# Interface level EIGRP
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip hello-time eigrp [autonomous_system] [secs]
(config-if)# ip hold-interval eigrp [autonomous_system] [secs]
```

_**Adjacency Requirements:** All **k values** and **autonomous system** must match in routers, so EIGRP can know and advertise networks_

#### RIPv2

_Routing Information Protocol (RIP), it´s best path decition making is based on the **hop count (max 15)** to reach the targeted network. Upadtes of networks are send every **30 sec** by the **224.0.0.9 multicast** address. RIPv2 support authentication and VLSM. RIPng supports !=v6 network advertisement. Protocol only used on lab/test & small networks_

```
!!! All routes learned by RIP and the interfaces where reach them
# show ip rip database
```

```
(config)# router rip
(config-router)# version 2
(config-router)# no auto-summary
(config-router)# network [network_to_advertise]  !!! Classful network

!!! Manual summarisation through neighborg interfaces
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip summary-address rip [network_to_advertise] [subnet_mask]

!!! Default route injection
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
!!! LSDB Link State Data Base
# show ip ospf database

!!! All information about routers links connections
# show ip ospf neighbors

!!! Detail information of OSPF adjacencies with OSPF
# show ip ospf interface brief
```

```
!!! Router OSPF process level
(config)# router ospf [process_id]
(config-router)# log-adjacency-changes !!! Advertise a log when OSPF link status change
(config-router)# auto-cost reference-bandwidth [base_kbps]
(config-router)# network [network_to_advertise] [wildcard] area [area_number]
(config-router)# default-information originate !!! Propagate default-static routes to area memebers

!!! MULTI-AREA: Network summarization for ABRs (IA) use to reduce entries in routing table
NOTE: The summarized subnet_mask must be EXACTLY the inverse of the wildcard declared when announcing the network in order for all the routes to be correctly advertised.
(config-router)# area [area_id] range [network_to_advertise] [subnet_mask]

!!! Interface level OSPF
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip ospf [process_id] area [area_number]
(config-if)# ip ospf cost [new_cost]
(config-if)# ip ospf priority [new_priority]
(config-if)# ip ospf network point-to-point !!! Advertise interface using the subnet mask
(config-if)# ip ospf {hello-interval | dead-interval} [value]
```

```
!!! Reset OSPF to see Router-ID/priority changes (if doesn´t work, save & reload device)
# clear ip ospf process
```

_**Adjacency Requirements:** All **hello (10 sec default)**, **dead time (4x hello) intervals** and **area number (Area ID)** must match in routers, so EIGRP can know and advertise networks_

#### IS-IS

_Intermediate System - Intermediate System Protocol (IS-IS), it depends on the cost of the route but the bandwidth metric is not calculated as OSPF does by default. The cost can be manually configure to manipulate the path (by default works with hop count as RIP). Used for Service Providers and large organizations over MPLS for scalability._

## Adjacency & Passive Interfaces

_Prevents or activates the transmission of hello packets for network discovery, commonly used for dynamic routing protocols to advertise networks to it´s attatched interfaces. A **passive interface can´t form adjacency** and must be redirect manually (static route). A **loopback must be set always as passive interface** because as a logical interface it´ll waste resources advertising a non-existent physical interface_

```
!!! Disable particular interfaces
(config)# router {eigrp | ospf | rip} [autonomous_system]?
(config-router)# passive-interface {gigabit | ethernet | loopback} {0-X}/{0-X}?

!!! Disable all by default & enable individually
(config)# router {eigrp | ospf | rip} [autonomous_system]?
(config-router)# passive-interface default
(config-router)# no passive-interface {gigabit | ethernet | loopback} {0-X}/{0-X}?
```

## Exterior Gateway Protocols (EGP)

### BGP

_Border Gateway Protocol (BGP)_

# **Inter-Vlan & Encapsulation**

_Virtual Local Area Networks (VLAN) segments the broadcast domains in a switch by grouping ports. The VLAN is identified as another IP segment and the communication between multiple VLANs must be routed._

The VLAN numbers _1002 through 1005 are reserved for Token Ring and FDDI VLANs_. VIDs 1 and 1002 to 1005 are automatically created, and you cannot remove them.

The configurations for VIDs 1 to 1005 are written to the _vlan.dat_ file (VLAN database) that is stored in _flash memory_.

```
# show vlan brief
# show interfaces {gigabit | ethernet} {0-X}/{0-X} switchport
# show interfaces ({gigabit | ethernet} {0-X}/{0-X})? trunk
# show mac address-table interface {gigabit | ethernet} {0-X}/{0-X}
```

## **Switch Inter-VLAN Configuration**

```
!!! VLAN Creation
(config)# vlan [vlan_id]
(config-vlan)# name [vlan_name]

!!! Access Port & VLAN port assigntation
(config)# interface range {gigabit | ethernet} {0-X}/{0-X} - {0-X}
(config-if)# description [access_description]
(config-if)# switchport mode access
(config-if)# switchport access vlan [vlan_id]
(config-if)# switchport voice vlan [voice_vlan_id]  !!! Only used when need to segregate data from VoIP

!!! Trunk Port (802.1Q)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# description [trunk_description]
(config-if)# switchport trunk encapsulation dot1q
(config-if)# switchport mode nonegotiate !!! Disables DTP
(config-if)# switchport mode trunk
(config-if)# switchport trunk native vlan [native_vlan_id]
(config-if)# switchport trunk allowed vlan [vlan_id1, vlan_id2, ...]
```

_**Administrative VLAN**: **Remote access management** VLAN for SSH/Telnet_
_**Native VLAN**: Manage **untagged traffic** and **coordinates the trunk VLAN sharing** between 2 ports by communicating allowed VLANs by the same native ID (must match in every switch with trunk connection)_
_**Type or Tag Protocol Identifier:** The IEEE 802.1Q inserts an extra 4-byte VLAN header into the Ethernet header of the original frame. Is set to a value of *0x8100* to identify the frame as an IEEE 802.1Q-tagged frame (*not present in native VLAN* traffic) + the VLAN ID identifies the VLAN to which the frame belongs. Can describe also traffic priority, and a MAC address format description (flag=1 for non-canonical & flag=0 for canonical)_

_**Dynamic Trunking Protocol (DTP)**: Is a **Cisco propietary protocol**, negotiates the trunking connection between switches. It´s recommended to turn it off DTP **(nonegotiate)** and set the switchport mode to access or trunk manually to prevent VLAN-Hopping Attacks. In DTP all ports are set to **auto mode by default** and can only make an adjacency when the status is **auto-desirable** or **desirable-desirable** in both links._

```
(config-if)# switchport mode dynamic [auto | desirable] !!! DTP modes, not recommended to set in modern switches
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
!!! Enable interface attatched to trunk switchport
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)#no shutdown

!!! Define virtual subinterfaces for each corssing VLAN
(config)# interface {gigabit | ethernet} {0-X}/{0-X}.[subinterface_number]
(config-subif)# encapsulation dot1q [vlan_ID] [native]?
(config-subif)# ip address [vlan_default_gateway] [netmask]
```

_Multilayer Switch (SVIs)_

```
!!! Enable switch routing
(config)# ip routing

!!! Define SVIs to manage each VLAN
(config)# interface vlan [vlan_id]
(config-if)# ip address [vlan_default_gateway] [netmask]

!!! Configure router-switch connection (for WAN reachablity to other networks)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}.[subinterface_number]
(config-subif)# no switchport
(config-subif)# ip address [default_gateway] [netmask]
```

_Another way to route VLAN traffic can be having multiple interfaces of the router attatche to each different VLAN and enable the communication by a simple routing of interfaces. This will demand more physical media and it´s better to enable it by the other 2 techniques above. The usage of virtual subinterfaces (router-on-a-stick) represents more contention of bandwidth but requires less cables attatched._

```
!!! Reset a switchport to default configuration
(config)# default interface {gigabit | ethernet} {0-X}/{0-X}
```

# **FHRP Network Redundancy**

_First Hop Redundancy Protocols: Controls **Layer 3 redundancy and failover recovery**. The usage of **Floating Static Routes** via other router is a feasible solution but we will need to do the manual configuration on each route we need to backup. The problem starts when we look at the **access layer (where hosts)** are assigned with certain default gateway, and **if that gateway fails they need to have another entry to send outside traffic.**_

## HSRP

The **Hot Standby Redundancy Protocol (HSRP)** s an FHRP that Cisco designed to create a redundancy framework between network routers or switches to achieve default gateway failover capabilities. Only one router forwards traffic. HSRP is defined in Request for Comments (RFC) 2281.

It creates a **Virtual Gateway IP (VIP)** to assign hosts, if any redundancy routers at distribution layer fails the remaining operational router takes over the operation with transparent service for the hosts. Selects an **active** router which will route all the network traffic through them and if it fails the remaining router in **standby** mode will take the ownership to be the active router.

We can defined the router that can be active by modifying its **priority (default 100)**, if a tie priority happends the **highest IP** wins the active mode. But is **important to set a preemtion for this**, otherwise the interface will be flapping with the ownership. All the hello messages sent through HSRP to define active/standby are sent via **HSRPv1: 224.0.0.2 UDP port 1985** and **HSRPv2: 224.0.0.102**

_HSRP Interface Tracking_
HSRP can track interfaces and it can **decrement priority if any interface or the router fails**, not only the one in which the VIP is assigned. It helps the re-convergence time, by not depending on only the failing of the interface assigned as default gateway (for end-devices) to recover the HSRP process.

_HSRP Load Sharing with HSRP Multigroup_
HSRP do not support load sharing, it can only foward traffic through 1 active router. However **load-sharing can be archived by MHSRP having an active router per VLAN**. In order to define MHSRP you must:

- Create each VLAN
- Define STP roles per router (primary or secondary)
- Configure an SVI with physical and standby VIP address
- Define the `priority` and activate the `preempt` faculties

```
!!! Displays VIP information & active/standby assignation
# show standby {brief | neighbors}?

!!! Display VIP and Virtual MAC address (0000.0C07.ACXX)
# show ip arp

!!! Verify the HSRP state transition.
# debug standby events
```

```
(config)# interface {interface | SVI} [vlan_id]?
(config-if)# standby [hsrp_group] version {1 | 2} !!! Versions must match. V2 for IPv6

(config-if)# ip address [ip_address] [netmask]      !!! Physical address DIFFERENT in both routers
(config-if)# standby [hsrp_group] ip [vip_address]  !!! Virtual address EQUAL in both routers
                                                    !!! VIP must exists inside physiscal subnet space

(config-if)# standby [hsrp_group] track {interface | SVI} [vlan_id]? [decrement_value] !!! Decrements 10 by default
(config-if)# standby [hsrp_group] priority [priority]
(config-if)# standby [hsrp_group] preempt [delay minimum]? [msec]?  !!! 50% time above after reboot

(config-if)# standby [hsrp_group] authentication [password]
(config-if)# standby [hsrp_group] authentication md5 key-string [7]? [password] !!! Enter 7 if password is already hashed
(config-if)# standby [hsrp_group] authentication md5 key-chain [chain_name]

(config-if)# standby [hsrp_group] timers [msec]? [hello_time] [msec]? [hold_time]
(config-if)# no standby group timers  !!! Default in seconds (hello: 3sec, hold: 10sec)

(config-if)# no shutdown

----------------------
!!! Key-chain Creation
(config)# key chain chain-name
(config-keychain)# key key-number
(config-keychain-key)# key-string [0 | 7] string
(config-keychain-key)# exit
```

## VRRP

The **Virtual Redundancy Router Protocol (VRRP)** is an open FHRP standard that offers the ability to add more than two routers for additional redundancy. Only one router forwards traffic. VRRP is defined in RFC 5798.

VRRP differs from HSRP in that it **allows you to use an address of one of the physical VRRP group members as a virtual IP address**. In this case, the device with the used physical address is a **VRRP Master (Active)** whenever it is available. Uses the **224.0.0.18** multicast address, with the protocol number 112. In VRRP _preemption is enabled by default_. Only Cisco devices still support VRRP timers in milliseconds (otherwise always works in seconds) and authentication methods for plaintext and MD5.

```
!!! Display VRRP configuration, VIP and Virtual MAC address (0000.5E00.ACXX)
# show vrrp
```

```
(config)# interface {interface | SVI} [vlan_id]?
(config-if)# ip address [ip_address] [netmask]  !!! Physical address DIFFERENT in both routers
(config-if)# vrrp [vrrp_group] ip [vip_address] !!! Virtual address EQUAL in both routers (could be same as one of the physical)
(config-if)# vrrp [vrrp_group] priority [priority]
(config-if)# vrrp [vrrp_group] authentication {text | md5} [key-string]? [password] !!! Use key-string while MD5

```

## GLBP

_The **Gateway Load Balancing Protocol (GLBP)** is an FHRP that Cisco designed to allow multiple active forwarders to load-balance outgoing traffic._

# **STP & Loop Prevention**

## Spanning Tree Protocol (STP)

**Prevents loops in layer 2**, which cause 3 main problems:

- _Broadcast storms:_ Each switch on a redundant network floods broadcasts frames endlessly. Switches flood broadcast frames to all ports except the port on which the frame was received. These frames then travel around the loop in all directions.
- _Multiple frame transmission:_ Multiple copies of the same unicast frames may be delivered to a destination station, which can cause problems with the receiving protocol. Many protocols expect to receive only a single copy of each transmission. Multiple copies of the same frame can cause unrecoverable errors.
- _MAC database instability:_ This problem results from copies of the same frame being received on different ports of the switch. The MAC address table maps the source MAC address on a received packet to the interface it was received on. If a loop occurs, then the same source MAC address could be seen on multiple interfaces, causing instability. Data forwarding can be impaired when the switch consumes the resources that are coping with instability in the MAC address table.

_STP Terminology_

- **BridgeID**: **Priority (0-65535) default 32768 + VLAN 1 = 32769** with increments of 4096 + device **MAC Address**
- **Root Bridge (RB)**: Elected by lowest BridgeID, **lowest priority or lowest MAC** all of its ports are in forwarding state (**designated state**).
- **BPDU**: Bridge Processing Data Unit, BPDUs are frames that have information about Spanning Tree Protocol. They are used for root bridge election and for loop identification. By default, BPDUs are sent out every **2 seconds(hello-timer)** remains **15 sec listening/learning states(forward delay)** and converges up to **20 secs (aging time)**. All BPDUs are share through multicast address **01-80-c2-00-00-00**. Exists 3 BPDU types:
  - BPDU TCN (Topology Change Notification): Sended when a link goes down to inform the RB & other routers reporting the issue.
  - BPDU TCA (Topology Change Acknowledgement): Response from the TCN, acknoledge by a RB or any switch with designated state link that can communicate the message to the RB.
  - Configuration BPDU (TC): BPDU sended by RB only informing the new connection state for other switches.
- **Root Port**: Interfaces from other switches (not Root Bridge) directly conected to the selected Root Bridge.

_STP Port States: Disabled (shutdown), Blocking (receives BPDUs only), Listening (receive/send BPDUs), Learning(receive/send BPDUs & learn MAC addresses), Forwarding (all functional, sending packets)_

_STP Versions_

- **802.1D STP**: Open standard. One STP instance for an entire network (no mattering the VLANs).
- **PVST+ (Default)**: Cisco enhancement of 802.1D STP for each VLAN that is configured in the network.
- **802.1W RSTP**: IEEE 802.1w. Improve the convergence time, port roles & link costs of STP (one instance for all the VLANS as STP).
- **802.1S MSTP**:
  - Reduces CPU utilization & communication by grouping **(up to 16 VLAN instances)**
  - IEEE standard that is inspired by the earlier Cisco proprietary MISTP implementation.
    _Note: It is recommended that you **do not run MST on access ports** between switches. It is also recommended that you **do not manually prune VLANs from trunks.**_
- **Rapid PVST+**: Cisco enhancement of 802.1W RSTP (with multiple instances per each VLAN as PVST).

## STP & RSTP Configuration

```
!!! Displays all STP instances information
# show spanning-tree summary

!!! Displays the STP information (PVST+ default = ieee)
---------------------------------------------------
Note: The command divides in 3 outputs:
 "Root ID" that must match in all VLAN switch members
 "Bridge ID" that is information about that particular switch
 "Interface Info" shows the port role/status
---------------------------------------------------
# show spanning-tree vlan [vlan-id]

!!! Display MAC from other connected switches & interfaces
# show mac address-table

!!! Observe STP convergence messages
# debug spanning-tree events
# undebug all
```

```
!!! Configure spanning-tree mode
(config)# spanning-tree mode {rapid-pvst | pvst}

!!! Manually assign priority (0-65535 on increments of 4096)
(config)# spanning-tree vlan [vlan_id] priority [bridge_priority]

!!! Automatically assign priority
(config)# spanning-tree vlan [vlan_id] root {primary | secondary}

!!! Configure link cost (1-65,535)
(config)# interface [interface]
(config-if)# spanning-tree vlan [vlan_id] cost [cost]

!!! Configure port priority (0-255) default 128
(config)# interface [interface]
(config-if)# spanning-tree vlan [vlan_id] port-priority [port-priority]

!!! Configure link as point-to-point
(config)# interface [interface]
(config-if)# spanning-tree link-type {point-to-point | shared}

!!! Configure timers
(config)# spanning-tree vlan [vlan_id] {hello-time | forward-time | max-age} [seconds]
```

- _Shared (default):_ Half-duplex mode. Connected to shared media where multiple switches might exist.
- _Point-to-point:_ Full-duplex mode. Port is connected to a single switch/end device at the other end of the link.

## MSTP Configuration

```
!!! Configure multiple spanning-tree mode
(config)# spanning-tree mode mst

!!!
# show spanning-tree mst configuration

!!!
# show spanning-tree mst configuration digest

!!!
# show spanning-tree mst [instance_id]

!!!
(config)# spanning-tree mst [instance_id] root {primary | secondary}
(config)# spanning-tree mst [instance_id] priority [instance_priority]

!!!
(config)# spanning-tree mst configuration
(config-mst)# name CCNP
(config-mst)# revision 1

!!!
(config)# spanning-tree mst configuration
(config-mst)# instance 1 vlan 2,3
(config-mst)# instance 2 vlan 4,5
(config-mst)# end
```

_BPDU GUARDS_

For **switchport access interfaces**, the usage of STP is no needed so we can skip the 50 secs of convergence by **port-fasting frames**

```
(config)# interface [interface]
(config-if) spanning-tree portfast
(config-if) spanning-tree bpduguard enable
(config-if) spanning-tree guard root !!! USE ONLY when we don´t want to receive BridgeID challenges to change the actual Root Bridge through the interface

!!! Enable globally BPDU Guard (highly recommended)
# spanning-tree portfast bpduguard default

!!! Disables globally STP blocking states (not recommended unless we can garantee no loop in the switch)
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
!!! Displays EtherChannel protocol, link status and interfaces aggregated
# show etherchannel summary

!!! Verify that STP is not blocking aggregated links
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
!!! Displays detail information of a particular ACL
# show access-list [acl_number | acl_name]

!!! Displays ACL implementation on inteface
# show ip interface {gigabit | ethernet} {0-X}/{0-X} | include access list
```

1. **Create ACL Rules**

   _**The ACE ordering in the ACL matters**, the router will interpret (top -> bottom) all the instructions set and included in the policies lists. **By default the ACE last statement (implicity set)** in every ACL is **deny any any** restricting all undeclared traffic._

   ```
   !!! Numbered ACL
   (config)#access-list [acl_number] {permit | deny | remark} {protocol} [source_ip_address] [wildcard | host | any] {gt | eq}? [port_number | app_name]? [destination_ip_address] [wildcard | host | any] {qualifier}? [port_number | app_name]?

   !!! Named ACL
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
!!! Display the inside & outside (local and global) address mapping
# show ip nat translation

!!! Removes all Dynamic NAT translation bindings in order to allow NAT configuration modifications
# clear ip nat translation *

!!! Display inside/ouside interfaces and how many packets has been sended & translated
# show ip nat statistics
```

- **Local Addresses:** It´s the actual source & destination IP, from the **sender point of view**.
- **Global Addresses:** They are the transalted addresses that match the **receiver knowledgement of it´s private destination** & **which public (translated) address sends** the package.

## Static NAT

Maps **1-to-1 private and public addresses**. Usualy used for servers which must **accept incoming connections**.

```
!!! Configure interface connected to the inside LAN (private address)
(config)# interface [interface]
(config-if)# ip nat inside

!!! Configure interface connected to the outside WAN (public address)
(config)# interface [interface]
(config-if)# ip nat outside

!!! Map private-public addresses
(config)# ip nat inside source static [private_ip_address] [public_ip_address]
```

## Dynamic NAT

Uses a **pool of public addresses** (similar to DHCP). Is used for **hosts that needs to connect externally to the internet but not receiving incoming connections.** The addressing assignment is set as **FIFO**. The NAT pool must have enough addresses for all hosts in the inside LAN. Iy uses a **connection tracking** which incoming packets from the outside are mapped to their local destinatinations by the port defined per device in the NAT overload table.

```
!!! Configure interface connected to the inside LAN (private address)
(config)# interface [interface]
(config-if)# ip nat inside

!!! Configure interface connected to the outside WAN (public address)
(config)# interface [interface]
(config-if)# ip nat outside

!!! Set a NAT pool
(config)# ip nat pool [nat_pool_name] [start_public_addresses] [end_public_addresses] netmask [netmask]

!!! Create an ACL for all INSIDE hosts that will be using the pool addreses
(config)# access-list [acl_number] permit [ip_matching_for_hosts] [wildcard]

!!! Associate the ACL with the NAT pool
(config)# ip nat inside source list [acl_number] pool [nat_pool_name]
```

## Port Address Translation (PAT)

Also known as NAT Overloading. Allows a **same public address to be re-used by many hosts**, the problem on Dynamic NAT is that we need the same amount (or more) public and private addreses. We can use a **dynamic pool** in which the **port number of the solicitant hosts will only be tracked once the pool reach the last available public address**. The same public address is used and the router know how to reach the host via it´s **unique port number**.

```
!!! Configure interface connected to the inside LAN (private address)
(config)# interface [interface]
(config-if)# ip nat inside

!!! Configure interface connected to the outside WAN (public address)
(config)# interface [interface]
(config-if)# ip nat outside

!!! Create an ACL for all INSIDE hosts that will be using PAT
(config)# access-list [acl_number] permit [ip_matching_for_hosts] [wildcard]

!!! Associate the ACL with the NAT pool
(config)# ip nat inside source list [acl_number] interface [exit_interface] overload
```

## NAT Virtual Interface (NVI)

The NVI **removes the necessity to declare inside/outside interfaces**. It is only present on **Cisco IOS v12.3** and above, and **routes one more time** (compared to the 3 traditional NAT modes). The process of NVI first routes the inside local address through the input interface, then it does the translation and finally the translated IP is routed (one more time) to the exit interface as the inside global address (route[in-if]->translate->routes again[out-if])

```
!!! Configure interface connected to the inside LAN (private address)
(config)# interface [interface] !!! Entry interface from local network
(config-if)# ip nat enable

!!! Configure interface connected to the outside WAN (public address)
(config)# interface [exit_interface] !!! Exit interface to outside network
(config-if)# ip nat enable

!!! Create an ACL for all INSIDE hosts that will be using NIV
(config)# access-list [acl_number] permit [ipv4] [wildcard]

(config)# ip nat source list [acl_number] interface [exit_interface] [overload]?
```

### Well-known Public Address (NO DHCP)

```
!!! Configure interface connected to the WAN (public address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat outside

!!! Configure interface connected to the LAN (private address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat inside

!!! Set a NAT pool
(config)# ip nat pool [nat_pool_name] [start_public_addresses] [end_public_addresses] netmask [netmask]

!!! Create an ACL for all INSIDE hosts that will be using the public address
(config)# access-list [acl_number] permit [ip_matching_for_hosts] [wildcard]

!!! Associate the ACL with the PAT OVERLOAD
(config)# ip nat inside source list [acl_number] pool [nat_pool_name] overload
```

_The `overload` flag on the last command `enable the port usage for PAT`. But is the same configuration as the Dynamic NAT pool & ACL association._

### Dynamic Public Address (DHCP)

```
!!! Configure interface connected to the WAN (public address) given by a DHCP Server in ISP
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip address dhcp
(config-if)# ip nat outside

!!! Configure interface connected to the LAN (private address)
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# ip nat inside

!!! Create an ACL for all INSIDE hosts that will be using the public address
(config)# access-list [acl_number] permit [ip_matching_for_hosts] [wildcard]

!!! Associate the ACL with the PAT OVERLOAD
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
(config)# ip dhcp snooping vlan [vlan_id] !!! epeat for all DHCP client´s VLANs
(config)# interface {gigabit | ethernet} {0-X}/{0-X} !!! Use on interface connected to Gateway (with the set ip-helper address)
(config-if)# ip dhcp snooping trust
```

### Dynamic ARP Inspection (DAI)

_**Man-in-the-middle and DoS** attack. A device attatched to the LAN **anticipates any ARP request from hosts to their default gateway** by doing a **Gratuitous ARP (CAM poisoning)**, where the attacker sends a **broadcast message to the switch indicating it´s MAC address and faking the real default gateway IP**. With that done, the switch **register in CAM the MitM fake MAC address** and all hosts will be unicasting traffic through it. To prevent this a **DHCP Snooping needs to be enable so the switch can map trusted assigned IPs with its real MACs**_

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X} !!! Use on interface connected to Gateway (with the set ip-helper address)
(config-if)# ip arp inspection trust

(config)# ip arp inspection vlan [vlan_ID] !!! epeat for all DHCP client´s VLANs
```

### Identity Based Networking (802.1X)

_Any **host (Supplicant)** that wants to connect in LAN or access WAN first send a message to the network **device (Authenticator)**, wich only conmmutes traffic to an **Authentication Server**. Then the supplicant is being asked for its username and password credentials and travel all the way back to the Autheticator > Authentication Server. **If the credentials are correct**, the Authenticator replys the Suplicant and the **commutation to other networks**, **interfaces in LAN (can be map to VLAN)** and **internet is allowed**._

### Switchports Security

_Defined a **mapping between allowed MAC addresses that can be attatched to certain switchport** and the **mechanisms in case of switchport violation**_

```
!!! Display interface security settings
# show port-security interface {gigabit | ethernet} {0-X}/{0-X}

!!! Display port-MAC association table
# show port-security address

!!! Displays port summary its status and violations
# show port-security
```

```
(config)# interface {gigabit | ethernet} {0-X}/{0-X}
(config-if)# switchport mode {access | trunk}
(config-if)# switchport port-security !!! y default allow 1 MAC at the time (no mattering the exact MAC attatched) and shutdowns if more than 1 hosts are connected
(config-if)# switchport port-security violation {shutdown | protect | restrict}
(config-if)# switchport port-security maximum [max_MAC_allowed]
(config-if)# switchport port-security mac-address {sticky | MAC_address} !!! ssociate specific MAC to the port to be learned in next plug-in or manually set
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
!!! Optional: Identificate SNMP AGENT to the SNMP MANAGER (recommended)
(config)# snmp-server contact [snmp_agent_identifier]
(config)# snmp-server location [location_description]

!!! Configure community string & privileges for SNMP AGENTS
(config)# snmp-server community [community_string] [privilege]
!!! Privilege types: "ro" = read-only , "rw" = read-write

!!! Reference the SNMP MANAGER (for each agent)
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
!!! Group the interest traffic (Class Map)
(config)# class-map [traffic_group_name]
(config)# match ip dscp [dscp_alias] !!! As "ef", "cs3", "af41", etc...

!!! Define the percentage of utilization in case of congestion (Policy Map)
(config)# policy-map [policy_name]
(config)# class [traffic_group_name]
(config)# shape average [bandwidth_limit_rate_to_start_buffering]
(config)# priority percent [bandwidth_guaranteed_utilization_in_case_of_congestion]
(config)# {service-policy [nested_policies]}?

!!! Setup utilization & shaping for remaining traffic types
(config)# class class-default
(config)# shape average [bandwidth_limit_rate_to_start_buffering]
(config)# fair-queue

!!! Apply policy to the outbound interface in WAN for dispatching (Service Policy)
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

## Virtualization

The virtualization helps to **abstract physical resources** such as memory, CPU, network or storage from the services provided (applications), installing a **software-layer between the OS and server hardware (called Hypervisor)**. An Hypervisor is the combination of a lightweight OS to manage physiscal hardware and a software layer that manages the resources needed by every node or VM. _The most used hypervisor is **VMware ESXi (vSphere)**_.

The network resources, when virtualized, split the physical NIC from the server (as many VMs needed) by giving each VM OS a **virtual Ethernet connection called Uplink**, in which forwarding traffic to the hypervisor network control, than can be seen as a virtual switch vSwitch which centralized all virtual network traffic (from vNIC) to the real server NIC card.

The architecture of Network Virtualization has three main components:

- **Access Control:** Where the authentication for end-users is granted/denied (VLAN or ACLs)
- **Services Edge:** Where Policy Enforcement points are centralized, to control communications between logical partitions
- **Path Isolation:** Independant logical traffic paths (called overlay) over a shared physical network like VPNs. Path Isolation can be archieve in **L2 by VLANs(802.1Q) single-hop** and in **L3(VRF, GRE & MPLS-VPN) multi-hop.**

### Virtual Routing & Forwarding (VRF)

Creates different virtual routers (different routing tables) in the same physical device. The **VRFs must also be mapped to the appropriate VLANs** at the edge of the network to provide continuous virtualization across the Layer 2 and Layer 3 portions of the network. A VRF has it´s own IP routing table (RIB), CEF FIB, and instance to process routing protocols with the CEs.

VRF or VRF-Lite can be understand as a **L3 VLAN for routing advertisemnt propagations**, meaning that it can keep separated between groups the advertised networks from a dynamic routing protocol or a default route, preventing the other groups (other L3 VLANs) to install in their IP routing tables the networks learned and avoiding unauthorized access to them. _Note: Routing table from the main router can be changed by NOT showing some learned routes, that will be present in the associated VRF group routing table_

- Allows for true routing and forwarding separation (different control & forwarding plane per vRouter)
- Management and traffic troubleshooting simplified.
- Supports different default routes (one per vRouter)

```
!!! Display all VRF groups with their belonging interfaces
# show ip vrf

!!! Display the configured IP address & link-state per interface in every VRF group
# show ip vrf interfaces

!!! Displays routing table from VRF group
# show ip route vrf [vrf_name]
# show ip route !!! Global routing table

!!! Displays routing table from VRF group
# show ip protocols vrf [vrf_name]
# show ip protocols !!! Global routing protocols

!!! Verify connectivity from VRF learned routes
# ping vrf [vrf_name] [ipv4] [source]? [loopback_interface]?
```

_Before configuring VRF all dynamic routing protocols (OSPF, EIGRP, RIP, etc...) must be configured and be working correctly (should have desired connectivity)._

```
!!! Create the VRF group
(config)# ip vrf [vrf_name]

!!! Assign the physical interface that will be belonging to the VRF instance
(config)# interface [interface]
(config-if)# ip vrf forwarding [vrf_name]
(config-if)# ip address [ipv4] [submask]
(config-if)# no shut

!!! Configure OSPF instances (existant only inside the VRF group)
(config)# router ospf [as_number] vrf [vrf_name]
(config-router)# router-id [router_id]
(config-router)# network [ipv4] [wildcard] area [area_number]
(config-router)# network [ipv4] [wildcard] area [area_number]
```

### Generic Routing Encapsulation (GRE)

Tunneling protocol that supports IP, IPX, Apple Talk and multicast routing protocols. It is mounted ove **IP 47** with **no ecryption by default**, but can be used along with **IPsec** to provide authentication and data confidentiality across the tunneling (usually configured as hub-spoke topology). GRE is _stateless_ meaning that it does not include any flow control mechanisms, by default. GRE adds a _20-byte IP header_ and a _4-byte GRE header._

```
!!! Determine the link-state of the tunnel (up/down)
# show ip interface brief tunnel [tunnel_id]

!!! Verify the state of the GRE tunnel
# show interface tunnel [tunnerl_id]

!!! Validate the tunnel is seen as directly connected
# show ip route
```

_Before configuring GRE all dynamic routing protocols (OSPF, EIGRP, RIP, etc...) must be configured and be working correctly (should have desired connectivity)._

```
!!! Configure GRE on both sides of the tunnel
(config)# interface tunnel [tunnel_id]
(config-if)# tunnel mode gre ip
(config-if)# ip address [ipv4] [submask]

!!! Note: Switch the IP from the exit interfaces respectively
(config-if)# tunnel source [local_IPv4]
(config-if)# tunnel destination [other_side_IPv4]
```

### VPNs & IP-Sec

The VPNs (Virtual Private Networks) are used for enterprises to replace WAN connection to geograpically dispersed sites (branches inside the administrative network of the company) in the public network to lower the infrastructure cost, scales easily, **authenticate peers** and **cryptographic path protection**. A VPN connects securely a Site-A -> Site-B over an untrustworthy network.

The 3 typical logical VPN topologies that are used in site-to-site VPNs are as follows:

- **Point-to-point:** Simply connects securely a Site-A -> Site-B over an untrustworthy network
- **Hub-and-spoke:** A central node (mostly a Data Center) interconnects all other sites (spokes)
  - Tiered: The VPN acts as an Hub in one side and Spoke in the other side of the connection
  - Joined: Combination of 2 or more topologies (hub-and-spoke, point-to-point, or full mesh)
- **Meshed network**
  - Full: Expensive and difficult to configure/manage to manage topology where all sites are interconnected by VPN
  - Partial: Decrease the costs of full-mesh having high availability and interconnection in critical parts of the VPN, but in other sites of it it will have H&S or P2P

#### IPSec VPNs

IPSec provides security in network traffic (IP L4 & layers above only) by authentication, encrypting and sharing keys between a pair of pair of security gateways (hosts or each side from a VPN).

IPSec combines 3 security protocols:

- **IKE (Internet Key Exchange):** Is also called _ISAKMP_. Provides **key management and secure key sharing** for encryption algorithms (from Site-A -> Site-B in a VPN) also called **security association**. In order to generate & refresh the keys while the ISAKMP/IKE policy is active it must agree in the **HMAC**, a **Diffie-Hellman group (2 and 5 default)**, define the **encryption & authentication algorithms to use** and the encryption **keys valid time**.
- **AH (Authentication Header):** **Authenticates** user traffic with **NO encryption** (obsolete and not supported on Cisco ASA)
- **ESP (Encapsulating Security Payload):** **Authenticates** user traffic **with encryption** (that´s why is preferred over AH) to provide integrity, confidentiality and protection against replays.

IP Sec Modes:

- **Transparent Mode:** Encrypts only the payload (data)
- **Tunnel Mode:** Encrypts the payload and the IP (and upper layers) headers.

##### Dynamic Multipoint VPN (DMVPN)

DMVPN does not require a permanent VPN connection between sites, uses a centralized architecture that simplifies deployment, communication between branches, lower costs (because works over the public WAN and ofers zero-touch configuration with IPsec).

##### FlexVPN

Similarly to DMVPN centralizes the VPN architecture **but offers the capability to manage different types of VPN types** (remote-access, teleworker, site-to-site, mobility, etc...). It offers **flexibility in transport layer** (deployed over public or private MPLS networks), third-party compatibility, IP Multicast support, QoS per tunnel, **centralized policy control** (encryption, VRF and DNS policies) peer AAA/RADIUS peer and it is VRF awarness through MPLS. It relies on **IKEv2** for key management.

##### Virtual Tunnel Interfaces (VTI)

Simplifies the configuration to site-to-site VPN tunnels. _Note: It is not recommended to use the same IP for the VTI as the physical interface because it leads to recursive routing and flapping tunnel on dynamic routing protocols_

```
!!! Verify IPSec packet count (send/received) through the tunnel
# show crypto ipsec sa [detail]?

!!! Verify VTI status (up/up). Otherwise review ALL the ISAKMP values, must match on both routers
# show interface [tunnel0]

!!! Validate that the default route to reach via the tunnel is on routing table
# show ip route
```

```
!!! Create an ISAKMP policy
(config)# crypto isakmp policy [priority]
(config-isakmp)# authentication pre-shared
(config-isakmp)# hash {sha}
(config-isakmp)# encryption aes 128
(config-isakmp)# group [diffie_hellman_group]
(config-isakmp)# lifetime [secs]

!!! Generate a pre-shared key
(config)# crypto isakmp key [password] address [tunnel_otherside_IPv4]

!!! Define a crypto profile attatche to an encryption transform-set
(config)# crypto ipsec transform-set [set_name] esp-aes 128 esp-sha-hmac
(config)# crypto ipsec profile[crypto_profile_name]
(ipsec-profile)# set transform-set [set_name]

!!! Associate the crypto profile to a VTI
(config)# interface [tunnel0] !!! Define tunnel ID
(config-if)# description [description_message]
(config-if)# ip unnumbered [tunnel_local_interface]
(config-if)# tunnel source [tunnel_local_interface]
(config-if)# tunnel destination [tunnel_otherside_IPv4]
(config-if)# tunnel mode ipsec ipv4
(config-if)# tunnel protection ipsec profile [crypto_profile_name]

!!! Configure a default route associate to the VTI to route traffic
(config)# ip route [network_to_reach_other_side] [netmask] [tunnel0]
```

_Note: Tthe tunnel allows only host from both sides to commuicate, meaning that a simple ping from the router will NO work. To verify connectivity this issue please enter `ping [ip] source [any_local_IP]` or try from a host_

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

    !!! In modern Linux the command is:
    $ ip address show
    ```

  - Display rounting info (default gateway)

    ```
    $ netstat -rn

    !!! In modern Linux the command is:
    $ ip route show
    ```

\w*-?\w*(?=>|#|\()
config(?=\))
Enter configuration commands, one per line. End with CNTL/Z.
(Interface|IP-Address|OK\?|Method Status|Protocol)
Ambiguous command:
(>|\(|\)|\*|\.|\%|\#|\:|\,|\"|\?|RETURN)
(Ethernet|Serial|\bup|interface|\/|serial|ethernet)
\d+
(Unrecognized command)
Invalid input detected at '^' marker
.\*

# **Wireless Networking**

WiFi services are defined in the IEEE **802.11** standard.

- Frequency
- Wavelength
- Amplitude
- Free Path Loss
- SNR

**Antenna Types**

- Omnidirectional:
  - Dipole
- Directional
  - Yagi
  - Patch
  - Parabolic Dish

## ISM Frequencies (Microwave Spectrum)

- **2.4 GHz**: 802.11b, 802.11g
- **5.0 GHz**: 802.11a, 802.11ac
- **2.4 & 5.0 GHz (dual-band)**: 802.11n, 802.11ax(WiFi 6)

MIMO (802.11n/ac):

- MRC:
- Beamforming:
- Spatial Multiplexing:

MU-MIMO(802.11ac):

## Deployment Architectures

### Autonomous (Standalone AP)

Each AP is managed indepently with no interaction between APs. Used in **small environments at low costs**. The **communication that is NO in the same WVLAN (even if the clients are connected to the same AP device) must be routed through the wired network**, converting the traffic of 802.11 to 802.3. The only configuration available for autonomous AP is SSID, wireless security (basic) and power levels transimission.

Some limitations of a standalone deployment are the manual configuration of each AP that are **prone to configuration inconsistencies**, **no rogue detection/mitigation** and **no dynamic RRM, FSR (Fast Secure Roaming)**.

_**RRM:** Radio Resource Management. Is the dynamic allocation of a free WiFi channel, to prevent noise or interference by overlapping the same frequency._

### WLC (Centralized)

The Wireless LAN Controller (WLC) is the responsible of configuration, control plane and management of several APs. Eases the management of large networks and hierarchicaly control multiple APs. The WLC load balance the traffic when an AP becomes overloaded or in case of fault it adjust it based on the policies.

The centralized architecture works with an **Split-MAC** mode where the **AP receives in real time the IP/MAC from a user** when trying to connect to the network, but it is the **WLC which handles and mantains the relation of clients and their connections (association and re-association)**.

The communication between an AP and the WLC is enabled by the **CAPWAP (Control And Provisioning of Wireless Access Points)** that **enables every AP to discover an active controller in order to gather features** such as authentication, configuration, mobility and security by exchanging messages or statistics with the controller. _CAPWAP differentiates between data and control plane._ The CAPWAP messages between APs and the AP Manager (controller) are sent over **UDP 5247**. All the traffic is **routed by the WLC** even if 2 clients connected to the same AP want to communicate between them, but never needs to go back to the wired infrastructure unless the communication is trying to reach another WLC or VLAN different than the source (in that case the CAPWAP header is removed and the 802.3 frame gets routed)

The benefits:

- Centralized management and troubleshooting
- RRM & High Availability
- wIPS (roge detection & mitigation)
- RADIUS and Cisco ISE authentication
- Handles WLANs (to separate guest traffic)
- Roaming (for voice and data)

The limitations:

- All traffic must be forwarded to WLC (even if 2 users connected to the same Ap want to communicate between them)
- WLC can be bottlenecked (single point of failure)
- Poor use of LAN/WAN resources

### FlexConnect

The **WLC resides in a central site** and it communicates to **many APs across a CAPWAP tunnel through the WAN**. It´s the most suitable solution for enterprises trying to connect many campuses (with branch guest access also called **WebAuth**) or are trying to **migrate from an autonomous (standalone) architecture**. The **Split-tunneling**, allows the FlexConnect APs to define how to react in the case of an unreachable connection to the cental site WLC for WAN failure or malfunction:

- **Centralized Authorization:** The AP mantains existing sessions and handles all new client association for locally WLANs (but cannot authenticate users outside the WLANs which will be de-associated)
- **Local Switching WLAN:** In WAN outage (no connection to WLC) the local traffic remains switching and AP works in Autonomous (standalone) mode. WLANs (but will no reach other WLANs because it needs the control plane from the WLC, disconnecting clients from other WLANs and making them unreachable)

_Note: _

### Cloud (Meraki & Catalyst 9800)

Deploying a cloud means deploying a computer system, or a network of systems, from which computing resources are offered to remote users. Therefore, from a user perspective, the resources are transparently available, regardless of the user point of entry.

IaaS: Delivers the infrastructure in a virtualized environment (network only)
PaaS: Delivers a computing platform and stack (IaaS + OS)
SaaS: Ready-to-use applications or software

**Cisco Meraki Cloud-based:** AP devices automatically connect to the Cisco Meraki cloud over a secure link, register with their network and download their configuration. No WLC is needed and all policies can be implemented through a dashboard with zero-touch (easy implementation), BYOD support, automatic RF allocation for channels and analyticis from the network. The limitations of this system are that only Meraki devices can support it, is less flexible because it´s single architecture and limits the customization (compared to on-premises controll and cetrallized solutions). **No user data flows to the cloud controller** and no L3 roaming is supported.

**Catalyst 9800:** The Cisco Catalyst 9800 are next-generation WLC build for intent-based networks, capable to virtualize their OS to run as a VM and being managed using Cisco DNA Center or NETConf/YANG in **SD-Access deploy model** used for **Centralized, FlexConnect and SD-Access (for private cloud)** or **FlexConnect only (for public cloud)** architectures.

**Cisco Mobility Express:** Virtualized WLC integrated in a Cisco 802.11ac (Wave 2) AP running an Aironet CAPWAP image. The way it works is by having a Master AP which serves as WLC and AP at the same time, while manage other Subordinates APs in it's same VLAN. It´s an affordable and easy deployment method that supports L2 roaming, WPA2'PSK, WPA2'Enterprise(802.1X) and WLAN for guest traffic separation.
