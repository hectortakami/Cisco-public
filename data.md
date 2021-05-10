# Basic Configuration

```
D(config)# no ip domain lookup
D(config)# line console 0
D(config-line)# logging synchronous
D(config-line)# exec-timeout [minutes] [secs]

en
conf t
no ip domain lookup
line console 0
logging synchronous
exec-timeout 0 0
end
wr

```

# MAC Addresses

```
!!! Show MAC table
SW# show mac address-table

!!! Show a router interface MAC address
R(config)# interface [interface]
R(config-if)# no shut
R# show interface [interface]

!!! Refresh MAC address table
SW# clear mac address-table dynamic

!!! Modify/Verify STP timers (global or per VLAN)
SW(config)# mac address-table aging-time [secs] [vlan]? [vlan_id]?
SW# show mac address-table aging-time
```

# Switching Methods (CEF)

```
!!! Verify the CEF (default and IP addresses configured per interface)
D# show ip cef

!!! Displays the local exit interface w/ the next-hop IP
D# show adjacency

!!! Disable CEF on an interface (when having connection to non-Cisco devices)
D(config)# [no]? ip cef !!! Disable CEF globally
D(config)# interface [interface]
D(config-if)# [no]? ip route-cache cef

```

# DTP & Trunk Links

```
!!! Display switchport (administrative and operational mode) of an interface
SW# show interface [interface] switchport

!!! Verify DTP negotiation protocols and ALLOWED VLANS
SW# show interface trunk

!!! Configure trunk/access or negotiated links
SW(config)# interface [interface]
SW(config-if)# switchport trunk encapsulation {isl | dot1q}
SW(config-if)# switchport trunk native vlan [vlan_id]
SW(config-if)# switchport trunk allowed vlan [add]? [vlan_id]
SW(config-if)# switchport mode {trunk | access | dynamic desirable}
SW(config-if)# [no]? switchport nonegotiate
```

# Layer 3 Switch VLAN Routing

```
(config)# ip routing

!!! SVI
(config)# interface vlan [vlan_id]
(config-if)# ip address [ipv4] [subnet]
(config-if)# no shut

```

# Static Routes

```
R(config)# ip route [ipv4] [mask] [next_hop]
```

# ROAS

```
R(config)# interface [interface]
R(config-if)# no ip add
R(config-if)# no shut

R(config)# interface [subinterface]
R(config-subif)# encapsulation dot1q 1 [native]?
R(config-subif)# ip address 192.168.1.1 255.255.255.0
R(config-subif)# no shut
-----------------------------------------------------------------------------------------------------------------------
NOTE: Tthe physical interface or at least 1 sub-if associated to the NATIVE VLAN for control plane (OSPF hello-packets)
-----------------------------------------------------------------------------------------------------------------------

SWL3(config)# int e0/0
SWL3(config)# switchport trunk encapsulation dot1q
SWL3(config)# switchport mode trunk
SWL3(config)# switchport nonegotiate


```

# Reset Configuration

```
(config)# default [interface]

(config)# no interface [subinterface]
```

# STP Manipulation

```
!!! Display STP mode, interface states and port priorities
SW# show spanning-tree [summary]?

!!! Manipulate STP cost & port priority
SW(config)# interface [interface]
SW(config)# spanning-tree vlan [vlan_id] cost [1-20000000]
SW(config)# spanning-tree vlan [vlan_id] port-priority [0-240] !!! 64 per hop

!!! Enhance STP converge time
SW(config)# spanning-tree backbonefast
SW(config)# spanning-tree uplinkfast

!!! Modify STP mode
SW(config)# spanning-tree mode {pvst | rapid-pvst | mst }
```

# Multiple Spanning-Tree (MST)

```
!!! Display MST Root & priorities
# show spanning-tree (mst [instance_number])?

!!! Display instances and VLAN associations
# show spanning-tree mst configuration
(config-mst)# show current !!! Same output at MST configuration level

(config)# spanning-tree mode mst
(config)# spanning-tree mst configuration
(config-mst)# name [region_name]
(config-mst)# revision [number]
(config-mst)# instance [instance_number] vlan [vlan_mapping]

(config)# spanning-tree mst [instance_number] root {primary | secondary}
(config)# spanning-tree mst [instance_number] priority [priority]
(config)# no spanning-tree mst [instance_number] priority   !!! Reset priority (INCLUDE ´priority´ keyword)
```

# Etherchannel

```
!!! Display Etherchannel state and interface bundle
# show etherchannel summary
# show etherchannel port-channel

!!! Verify the Load-balancing method
# show etherchannel load-balance

!!! Review the Etherchannel traffic per interface in the bundle
# clear counters
# show interface [interface] | include packets output
---------------------------------------------------------------
# test etherchannel load-balance interface port-channel [port_id] {mac | ip}


(config)# interface range [range]
(config-if-range)# switchport trunk encapsulation dot1q
(config-if-range)# switchport mode trunk
(config-if-range)# switchport trunk native vlan [native_vlan]
(config-if-range)# switchport trunk allowed vlan [add]? [vlans]
(config-if-range)# channel-group [port_id] mode desirable
(config-if-range)# no shut

(config)# port-channel load-balance {src-dst-ip | src-ip | dst-ip}


!!! Remove Etherchannel port-channel
(config)# no interface port-channel [port_id]
```

# Re-distribution Filtering

```
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                            PREFIX LISTS
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
!!! Display prefix-list information
# show ip prefix-list { [pl_name] | detail | summary }? !!! Prefix-list is created on the router with 2 PROTOCOLS

(config)# ip prefix-list [pl_name] seq [sequence_number] {permit | deny} [summarized_network_with_cidr]

----------------------- OSPF -> [EIGRP] ----------------------------------
(config)# router eigrp [as_number]
(config-router)# redistribute {ospf | ospfv3} [process_id] [metric]? ([default_bw_kbps] [default_delay_divided_by_10] 255 1 [default_mtu])? [include-connected] !!! 1500 100 255 1 1500
(config-router)# distribute-list prefix [pl_name] {in | out} ospf [process_id]

----------------------- EIGRP -> [OSPF] ----------------------------------
(config)# router ospf [process_id]
(config-router)# redistribute eigrp [as_number] subnets   !!! Re-distribution is made on the router with 2 PROTOCOLS
(config-router)# distribute-list prefix [pl_name] {in | out} eigrp [as_number]
```

```
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                            ROUTE MAPS
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

!!! Configure Prefix-Lists to implement
(config)# route-map [routemap_name] deny [sequence_number_+10]
(config-route-map)# match ip address prefix-list [pl_name]  !!! If no 'match' created, applies to everything
(config-route-map)# set metric
(config-route-map)# exit
(config)# route-map [routemap_name] permit [sequence_number_+10]
(config-route-map)# match ip address prefix-list [pl_name]  !!! If no 'match' created, applies to everything
(config-route-map)# set metric
(config-route-map)# exit

----------------------- EIGRP -> [OSPF] ----------------------------------
(config)#router eigrp 12
(config-router)#redistribute ospf 2 route-map OSPF_2_EIGRP
```

# EIGRP+

```
!!! Verify EIGRP configuration, passive interfaces & networks advertised
# show ip protocols

!!! Display adjacent neighbors per AS
# sh ip eigrp neighbors

!!! Display all Feassible Routes learned from routes (Succesors & F.Succeassors)
# show ip eigrp topology [all-links]?

!!! Display routing table and review EIGRP learned routes (D)
# show ip route eigrp

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                            CLASSIC EIGRP
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

(config)# [ipv6]? router eigrp [as_number]
(config-router)# network [ipv4] [wildcard]
!!! ECMP
(config-router)# maximum-paths [1-32]
(config-router)# traffic-share { balanced | min across-interfaces }
!!! UCMP
(config-router)# variance [1-255]
(config-router)# [no]? eigrp stub {connected | summary | receive-only | static | redistributed} !!! Remove the stub with kw "no" each time a stub distribution route is changed
(config-router)# redistribute ospf [process_id] [metric]? ([default_bw_kbps] [default_delay_divided_by_10] 255 1 [default_mtu])?
(config-router)# default-metric [bw_kbps] [delay_divided_by_10] 255 1 [mtu] !!! Use 'show int [interface]'

(config)# interface [interface]
(config-if)# ipv6 eigrp [as_number]
(config-if)# ip authentication mode eigrp 2021 md5
(config-if)# ip hello-interval eigrp [as_number] [secs]
(config-if)# ip hold-time eigrp [as_number] [secs]
(config-if)# bandwidth [bps]
(config-if)# delay [tens_bps] !!!Delay will be MULTIPLIED x10 by the router to show it as microns
(config-if)# ip summary-address eigrp [as_number] [summarized_ipv4] [submask]


+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                            NAMED EIGRP
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

(config)# ipv6 unicast-routing

!!! Key Chain
(config)# clock timezone utc -6

(config)# key chain [key_chain_name]
(config-keychain)# key [key_number]
(config-keychain-key)# key-string [password]
(config-keychain-key)# send-lifetime local 15:30:00 5 Feb 2021 infinite
# show key-chain


(config)# router eigrp [as_name]
!!! Global AF level
(config-router)# address-family { ipv4 | ipv6 } unicast as [as_number]
(config-router-af)# network [ipv4] [wildcard]
(config-router-af)# [no]? eigrp stub {connected | summary | receive-only | static | redistributed}
!!! Interface level
(config-router-af)# af-interface { default | [interface] }
(config-router-af-if)# authentication mode {md5 | hmac-sha-256}
(config-router-af-if)# authentication key-chain [key_chain_name]
(config-router-af-if)# [no]? passive-interface
(config-router-af-if)# hello-interval [secs]
(config-router-af-if)# hold-time [secs]
(config-router-af-if)# summary-address [summarized_ipv4] [submask]
(config-router-af-if)# exit
!!! Topology Base
(config-router-af)# topology base
(config-router-af-topology)# variance [1-255]
(config-router-af-topology)# maximum-paths [1-32]
(config-router-af-topology)# traffic-share { balanced | min across-interfaces }
(config-router-af-topology)# exit


+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                            EIGRP ERROR LOGS
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

-----------------------------------------------------------------------
 %DUAL-6-NBRINFO: EIGRP-IPv4 ... is blocked: not on common subnet
-----------------------------------------------------------------------
 Multiple subnet segments running in the same EIGRP AS broadcast domain

 SOLUTION:  In connected SWITCHES, assign to differents VLANS each
            router access interface per subnet advertised
-----------------------------------------------------------------------

-----------------------------------------------------------------------
 %DUAL-5-NBRCHANGE: EIGRP-IPv4: ... is down: holding time expired
-----------------------------------------------------------------------
 Flapping interface, an adjacency is made but it´s dropped continiuously

 SOLUTION:  The hold-timer is smaller than the hello-interval, modify
            the timer values on the interface (hold must be 4X )
-----------------------------------------------------------------------
```

# OSPF+

```
# show ip route ospf

# show ip protocols

!!! Display timers, costs, areas and DR/BDR status
# show ip ospf interface [brief]?

!!! Review adjacency neighboring process
# [no]? debug ip ospf adj
# undebug all

!!! Display priorities, neighboring status (FULL), DR/BDR/DROTHER & interface info
# show ip ospf neighbor

!!!
# show ip ospf database { network | router }?


+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                            CLASSIC OSPF
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
(config)# ip routing

(config)# router ospf [process_id]
(config-router)# network [ipv4] [wildcard] area [area_number]
(config-router)# router-id [router_id]
(config-router)# area [process_id] authentication message-digest

!!! %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from
!!! Expected behaviour of this error, until BOTH sides of the virtual-link POINT TO IT´S NEIGHBOR
(config-router)# area [area_id] virtual-link [neighbor_router_id]
(config-router)# do clear ip ospf process !!! Type 'yes'
(config-router)# area [area_id] range [summarized_ipv4] [subnet]    !!! ABR summarization (per area)
(config-router)# default-information originate [always]? (metric [metric] metric-type {1 | 2})?   !!! Re-distribute static routes
(config-router)# [no]? area [area_id] stub { nssa | no-summary }?   !!! Apply to every router in the stub area & REMOVE stub on every change
(config-router)# redistribute eigrp [as_number] subnets   !!! Re-distribution is made on the router with 2 PROTOCOLS
(config-router)# distribute-list prefix [pl_name] {in | out} eigrp [as_number]

(config)# interface {[svi] | [interface]}
(config-if)# no switchport
(config-if)# ip address [ipv4] [submask]
(config-if)# ip ospf authentication {message-digest | key-chain}
(config-if)# ip ospf message-digest-key [key_number] md5 [password]
(config-if)# ip ospf network {broadcast | point-to-point | non-broadcast} !!! Must match on both interfaces sharing the area
(config-if)# no shut

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                                OSPFv3
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
(config)#ipv6 unicast-routing

(config)# ipv6 router ospf [process_id]
(config-rtr)# router-id [router_id]
(config-rtr)# exit

(config)# interface [interface]
(config-if)# ipv6 address [ipv6_with_cidr]
(config-if)# ipv6 ospf [process_id] area [area_id]
-----------------------------------------------------------------------

R1(config)# router ospfv3 [process_id]
R1(config-router)# address-family {ipv4 | ipv6} unicast
R1(config-router-af)# redistribute eigrp [as_number] (metric-type {1 | 2})?


+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
                            OSPF ERROR LOGS
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

-----------------------------------------------------------------------
%OSPF-4-ASBR_WITHOUT_VALID_AREA: Router is currently an ASBR while having only one area which is a stub area
-----------------------------------------------------------------------
 On OSPF stub area the re-distribution of other routing protocols is invalid

 SOLUTION:  1. Remove any stub configuration on EVERY router with the stub area 'no area [area_id] stub'
            2. Convert the stub area to NSSA 'area [area_id] stub nssa'
            (Optional) re-distribute again 'redistribute eigrp [as_number] subnets'
-----------------------------------------------------------------------
```

http://pawsitive.bold-themes.com/buddy
http://pawsitive.bold-themes.com/coco
http://pawsitive.bold-themes.com/bella

# VRF

```
!!! Configure on the middle router (ISP)

# show ip vrf
# sh ip route vrf [vrf_name]
# sh ip protocols vrf [vrf_name]

(config)# ip vrf [vrf_name]
(config-vrf)# exit

(config)# interface [interface]
(config-if)# ip vrf forwarding [vrf_name]
(config-if)# ip address [ipv4] [submask]
(config-if)# no shut

(config)# router ospf [process_id] vrf [vrf_name]
(config-router)# router [router_id]
(config-router)# network [network] [wildcard] area [area_id]
```

# GRE

```
!!! The IP from both devices A & B must be reachable
!!! Every dynamic routing protocol must annonce the IP subnet on both sides

-------------- Side A --------------
(config)# interface tunnel [tunnel_id]
(config-if)# ip address [ipv4_A] [submask_A]
(config-if)# tunnel mode gre ip
(config-if)# tunnel source [exit_interface_to_WAN]
(config-if)# tunnel destination [ipv4_B]

-------------- Side B --------------
(config)# interface tunnel [tunnel_id]
(config-if)# ip address [ipv4_B] [submask_B]
(config-if)# tunnel mode gre ip
(config-if)# tunnel source [exit_interface_to_WAN]
(config-if)# tunnel destination [ipv4_A]
```

# BGP

```
!!! Displays BGP table (learned routes announced to other routers)
# show ip bgp [summary]?

!!! Displays Neighbor table
# show ip bgp neighbors [| section {external | internal}]?

!!! Displays IP routing table
# show ip route bgp

# show ip bgp neighbors [neighbor_loopback_ipv4] | include BGP neighbor | TTL

# ping [neighbor_in_other_AS] source [local_loopback_name]


(config)# router bgp [as_number]
(config-router)# bgp router-id [router_id]
(config-router)# neighbor [neighbor_loopback_ipv4] remote-as [neighbor_AS]
(config-router)# neighbor [neighbor_loopback_ipv4] weight [0-65535]
(config-router)# network [ipv4] mask [subnet]
(config-router)# neighbor [neighbor_loopback_ipv4] update-source [local_loopback_name] !!! Local loopback name mirrored as neighbor IP
(config-router)# neighbor [neighbor_loopback_ipv4] ebgp-multihop [hops]?
(config-router)# neighbor [neighbor_loopback_ipv4] disable-connected-check  !!! Use only when connection is 1 HOP-AWAY as an alternative for 'ebgp-multihop'
(config-router)# ip route [neighbor_loopback_ipv4] [submask] [neighbor_link_ipv4]
(config-router)# bgp cluster-id [peer_group]    !!! Use it to reflect routes to other peers
(config-router)# neighbor [neighbor_loopback_ipv4] route-reflector-client
??(config-router)# do clear ip bgp *  !!! ***DO NOT USE THIS COMMAND IN PRODUCTION***


!!! CONFIGURE IN THE AS-BORDER router to interconnect discontinuous AS neighbors (reachable through extended ping)
(config)#ip route 0.0.0.0 0.0.0.0 [neighbor_interface_ip_in_other_AS_link_ISP]

(config-router)# network 0.0.0.0    !!! Works only if only exists a default gateway
(config-router)# neighbor [neighbor_loopback_ipv4] next-hop-self    !!! Repeat FOR EVERY connected neighbor
(config)# clear ip bgp [neighbor_loopback_ipv4] out

```
