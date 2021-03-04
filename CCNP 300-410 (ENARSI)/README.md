# CCNP 300-410 ENARSI

## EIGRP

EIGRP uses _Reliable Transport Protocol (RTP)_ which _runs above the IP layer as protocol number 88_, use both unicast/multicast (_224.0.0.10_ or _FF02::A_) to take care of reliable exchange of information.

- _Partial Triggered Updates:_ Do not send periodic updates to it´s neighbors, the updates are sent only when a metric or route change and only to the affected routers.
- _Fast Convergence:_ Quickly adapts to topology changes discoverying alternative routes
- _Sophisticated Metric (EIGRP-Wide):_ iOS 15 allows 64 bits (commonly 32 bits only) in load balancing metric to have more control in traffic flow and distribution _Note: Supported on bandwidths above 1 gigabit and up to 4.2 terabits only_
- _VLSM (IPv4 & IPv6):_ Classless Routing Protocol that advertise subnets and can be manually configure tu summarize them.
- _Seamless connectivity across multiple data-link media:_ Unlikely OSPF that must be configure differently depending on the cable media (Frame Relay or Ethernet). EIGRP operates in both LAN or WAN environments non depending on the media.

### Implementing

- _Neighbor Table:_ Stores the primary IP address and the directly connected interface to the established router adjacencies
- _Topology Table:_ Contains the relation between neighbors and it´s _advertised destination routes_ with it´s _metric_
- _Routing Table:_ Contains all the routes learned from the successors.

#### Classic vs Named Mode

- _Classic Mode_

  The EIGRP configuration for IPv4 and IPv6 must be done separate (not at the same mode level) and the association to the EIGRP AS is done by different levels (IPv6 associates it with the interface and IPv4 declare the network at router config level).

  _Note:_ In EIGRP, _default routes CANNOT be directly injected_ as in OSPF, where you can use the `default-information originate` command, we need to `redistribute` them defining specifically those protocols that we want to include.

  ```
  ************ IPv4 ************
  (config)# router eigrp [autonomous-system]
  (config-router)# eigrp router-id [router-id]
  (config-router)# network [ipv4]
  (config-router)# [no]? passive-interface {ethernet|gigabit} [number]
  (config-router)# eigrp stub {connected | redistributed | static | summary | receive-only}?
  (config-router)# redistribute {static | connected | ospf | bgp }
  (config-router)# variance [variance-value]

  (config)# interface {ethernet|gigabit} [number]
  (config-if)# ip hello-interval eigrp [as-number] [seconds]
  (config-if)# ip hold-time eigrp [as-number] [seconds]
  (config-if)# ip authentication key-chain eigrp [as-number] [keychain-name]
  (config-if)# ip authentication mode eigrp [as-number] md5 !!! hmac-sha-256 not supported on this mode

  ************ IPv6 ************
  (config)# ipv6 unicast-routing
  (config)# ipv6 router eigrp [as-number]
  (config-rtr)# eigrp router-id [router-id]
  (config-rtr)# [no]? passive-interface {ethernet|gigabit} [number]
  (config-rtr)# eigrp stub {connected | redistributed | static | summary | receive-only}?
  (config-rtr)# redistribute {static | connected | ospf | bgp }
  (config-rtr)# variance [variance-value]


  (config)# interface {ethernet|gigabit} [number]
  (config-if)# ipv6 eigrp [as-number]
  (config-if)# ipv6 hello-interval eigrp [as-number] [seconds]
  (config-if)# ipv6 hold-time eigrp [as-number] [seconds]
  (config-if)# ipv6 authentication key-chain eigrp [as-number] [keychain-name]
  (config-if)# ip authentication mode eigrp [as-number] md5 !!! hmac-sha-256 not supported on this mode
  ```

- _Named Mode_

  Centralizes the EIGRP configuration for IPv4 and IPv4. Uses _64-bit_ wide metrics. EIGRP named mode configuration is available in Cisco IOS Software Release 15.0(1)M, 12.2(33)SRE, 12.2(33)XNE and later releases.

  ```
  (config)# router eigrp [virtual-instance-name]

  ************ IP ADDRESS-FAMILY ************
  (config-router)# address-family {ipv4 | ipv6} autonomous-system [as-number]
  (config-router-af)# eigrp router-id [router-id]
  (config-router-af)# network [network-address]
  (config-router-af)# eigrp stub {connected | redistributed | static | summary | receive-only}?

  ************ INTERFACE LEVEL ************
  (config-router-af)# af-inteface {default|ethernet|gigabit} [number]?
  (config-router-af-interface)# [no]? passive-interface
  (config-router-af-interface)# hello-interval [seconds]
  (config-router-af-interface)# hold-time [seconds]
  (config-router-af-interface)# summary-address [network-summarization]/[cidr]
  (config-router-af-interface)# authentication key-chain [keychain-name]
  (config-router-af-interface)# authentication mode md5
  (config-router-af-interface)# authentication mode hmac-sha-256 [password]
  (config-router-af)# exit

  ************ TOPOLOGY LEVEL ************
  (config-router-af)# topology base
  (config-router-af-topology)# redistribute {static | connected | ospf | bgp }
  (config-router-af-topology)# variance [variance-value]
  (config-router-af-topology)# exit
  ```

#### DUAL Metrics (K-values)

EIGRP uses a composite metric to determine the best path to the destination. The metric value derives from a formula, which can use the following parameters:

- _Bandwidth_: The least value of the bandwidth for all links between the router, which computes the metric, and the destination. This parameter is referred to as “throughput” in EIGRP named mode.
- _Delay_: The cumulative delay that is obtained as the sum of values of all delays for all links between the source and destination. This parameter is referred to as “latency” in EIGRP named mode.
- _Reliability_: The worst reliability between source and destination, which is based on keepalives.
- _Load_: The worst load on the link between the source and the destination, which is based on the packet rate and the configured bandwidth of the interface

The metric and default k-values for each EIGRP mode result as follows:

_It´s not recommended to change the default k-values for any of the 2 EIGRP modes_

- _Classic Mode (k=5)_:

  - Uses _bandwidth_ and _delay_
  - _k1=1_, k2=0, _k3=1_, k4=0, k5=0
  - Delay for normal links (Microseconds)
  - Metric formula = [(K1 _ bandwidth + [(K2 _ bandwidth) / (256 – load)] + K3 _ delay) _ K5 / (K4 + reliability)] \* 256
  - _Simplified Metric (using default k-values)_ = ([10^7 / min-bandwidth-bps] _ 256) + ([sum-delays-ms _ 100] \* 256)

- _Named Mode (k=6)_:
  - Uses _throughtput_ and _latency_
  - k1=1, k2=0, k3=1, k4=0, k5=0, _k6=0_
  - 64-bits wide metric
  - Supports RIP
  - Latency for high-speed links (Picoseconds)

#### Glossary

- _Reported Distance (RD):_ Equals the metric formula.
- _Feasibile Distance (FD):_ Equals the metric formula + the neighbor cost to it´s reported path
- _Feasibility Condition (FC):_ The reported distance must be _less_ than feasible distance, which mantains the split horizon. Any EIGRP route wich metrics accomplish the FC will be considered as a FS but only the best (lowest FD) will be installed in the routing table as Successor.

  ```
  !!! FC = RD < FD

  # show ip eigrp topology
  !!! Feasibile Successors (P) means routes in passive state
  P 192.168.0.0/24, 1 successors, FD is 409600
                          FD     RD
        via 172.16.1.1 (409600/128256), Ethernet0/0   !!! Actual FS
        via 172.16.2.1 (1664000/128256), Ethernet0/1  !!! Next FS
  ```

- _Split Horizon (loop-free topology):_ Avoid in propagate the routes learned by the same interface where were received.
- _Sucessor:_ The route with the best metric destination (lowest cost) that is installed in the routing table.
- _Feasibile Sucessor (FS):_ The next best route that accomplish the feassible condition.

### Optimizing

#### EIGRP Queries (Problem)

_EIGRP Queries_ are another package (as hello, update, hold and ACK) sended when an EIGRP route dont have a successor or FS left, it asks to all it´s neighbors in _multicast/unicast_ manner (excepting the succesor by the split-horizon principle) for another FS route. If the neighbor doesn´t know another FS it spreads the query to others until someone replies. This creates a _cascade query propagation_ that can be limit by using _Route Summarization_ or _EIGRP stub routing features._

##### Stub Routing

- Improves network stability optimizing large scale EIGRP networks
- Reduce resource utilization (limiting the amount of query messages in the network)
- Simplifies remote router configuration (Hub-Spoke topologies)
- Speeds up convergence

Stub routers configured with the `eigrp stub` commands under the router mode (classic) or address-family (named) share it´s _directly connected and summary routes by default_. The modes configurable by the command are:

- `connected` advertise directly connected routes _(default without being specified)_
- `summary` advertise summarized routes _(default without being specified)_
- `redistribute (D EX)` advertise networks learned by other dynamic routing protocols
- `static` advertise static and default routes
- `receive-only` do NOT advertise it networks but can lear them from other routers

If an EIGRP Query is not reply within _90 secs_ (even with both an alternate FS route or a not found ackwoledgement) a _SIA-Query_ is sended from the query emitter to it´s neighbor with the closest un-reply. If the direct neighbor doesn´t have the problem and it already send a reply for the query response with a _SIA-Reply_. If the neighbor of the non-replying neighbor didn´t answer in _3 minutes (default)_ the non-reply link turns into _Stuck-In-Active (SIA) state_ but the EIGRP process keeps going.

##### Route Summarization

- Limits the query scope
- Faster response (Any router having a FS with the query prefix or subnet replies)

EIGRP supports route summarization at any router, unlike OSPF, which requires that summarization be performed only at area border routers (ABR) or autonomous system border routers (ASBR). Route summarization _works best when the subnet planning process considers route summarization._

#### Load Balancing

EIGRP can distribute traffic over multiple links leading to the same destination to increase the effective network bandwidth. It supports load balancing over equal-metric paths and also over unequal-metric paths.

- _Equal-Cost Multipath (ECMP):_ Distribute traffic across and up to _4 equal cost paths_ by default with the _same metric._ The maximum equal cost paths to use can be change using the `maximum-paths` command.
- _Equal-Metric Load Balancing:_
  - The `variance` that is set to _1 as default_, but is disabled. It can be changed to use multiple links to foward traffic with the only condition to accomplish the FC in the FS links.
  - The _variance can be set from 1-128_ and is obtain with the maximum metric divided by the lowest metric from the links we want to use for load balancing. `variance = max-metric / min-metric`
  - The route must be _loop free_, by _meeting the feasibility condition_.
  - The metric of the route must be _lower than_ the metric of the successor, multiplied by the value of the variance.

#### EIGRP Authentication

Authentication is used to ensure that your EIGRP routers only form neighbor relationships with legitimate routers and that they only accept EIGRP packets from legitimate routers.

_EIGRP Message Digest 5 (MD5)_ authentication can be configured in _classic mode or named mode_, while _HMAC-SHA-256_ authentication can be configured in _named mode only._ The EIGRP authentication prevents hackers to establish neighbor relationships and DoS attacks.

Using a _password is simpler to configure_ but if the password needs to be changed, the _neighbor adjacency will drop when the password is changed_ on one of the routers. Using a _Keychain allows the passwords to be rotated_ without dropping the neighbor adjacency.

### Troubleshooting

#### Verification Commands

```
!!! Displays all interface EIGRP configurations
# show run | sec eigrp

!!! Dispay networks advertised & passive interfaces per each EIGRP AS
# show {ip | ipv6} protocols

!!! Display the EIGRP routes present in the routing table (successor routes)
# show {ip | ipv6} route eigrp

!!! Display metrics for the successor, feasible succesor & paths that don´t apply to the FC
# show {ip | ipv6} eigrp topology [all-links]?

!!! Display interface hello/hold timers (default 5 and 15 secs)
# show {ip | ipv6} eigrp interfaces [detail]?

!!! Display Router-IDs, stub peers and adjacent interfaces
# show {ip | ipv6} eigrp neighbors [detail]?

!!! Displays statistics for hellos, updates, and ACK packets
# show {ip | ipv6} eigrp traffic [detail]?

!!! Verify the authentication mode and key chain associations
# show eigrp address-family {ipv4 | ipv6} interfaces detail ({ethernet|gigabit} [number]) | include Auth

!!! Display EIGRP message changes, updates, and Auth status
# debug eigrp packets [terse]?
# no debug all !!! Removes all debug messages (not use debug in prod env)
```

#### EIGRP Neighbor Adjacency Issues

_Cause Problems_

- The interface between the devices is down.
- The routers have mismatching EIGRP autonomous systems.
- The EIGRP process is not enabled on one of the interfaces that connects the devices.
- One of the interfaces that connects the devices is configured as a passive interface.

_Solutions_

- _Solution 1:_ Make sure it´s not related with an L2 problem or the connected interfaces are in _shutdown state_.
- _Solution 2:_ If the interfaces are up, validate you have the _same EIGRP AS name/number_ in both sides. If not, remove it and re-enter the correct data with the desired advetised networks.
- _Solution 3:_ Verify that the interfaces between EIGRP neighbors are active in `no passive-interface` state. And the required _interfaces are associated_ with the correct EIGRP process.
- _Solution 4:_ In case you are using authentication, make sure to mach the exact same `authentication modes` for MD5 or HMAC-SHA-256. The `password` and `key-id & key-string (keychain)` must be _exactly the same_ in all routers. And finally verify the correct configuration and _valid times of the keys in keychain_. _Note: It´s important to verify the configured or obtained NTP device time to validate keys expiration_

_Detection commands:_

- `show {ip | ipv6} interface brief` verify that the connection is `up/up`
- `show {ip | ipv6} protocols` validate the configured AS number/name _(must match)_ & passive interfaces
- `show {ip | ipv6} eigrp interfaces` displays all interfaces where EIGRP is enabled _(handy for EIGRP IPv6)_
- `show run | sec eigrp` displays EIGRP configurations
- `show key chain` presents keychain configuration _(must match in case is used)_
- `show clock` presents time and date to validate keys expirancy
- `show {ip | ipv6} eigrp neighbors` verify that the neighborship was established

#### EIGRP Routing Issues

The neighborship can be established but the advertised networks from the EIGRP neighbor don´t appear in the routing table resulting on an unreacheable ping.

_Cause Problems_

- Networks are not being advertised on remote routers.
- An access list is blocking advertisements of remote networks.
- Automatic route summarization is causing confusion in your discontiguous network.

_Solutions_

- _Solution 1:_ Make sure you _had advertised the route_. For _directly connected routes_ in the EIGRP (classic/named) mode and add a `network` statement so the address can be shared. For _non-connected routes_, make sure to issue the `redistribute` command to share routes learned staticaly or by other dynamic routing protocols.
- _Solution 2:_ The neighbor route can be configured as `stub receive-only` mode which doesn´t share route info.

_Note: If the problem persists, verify if there is an *ACL blocking* advertisement or the *default auto summarization* (for iOS before 15 Release) was correctly configured, this could cause router confussion summarizing all routes with classful masks known as a *discontinous network problem* run the `no auto summary` command in EIGRP router process to prevent this behaviour._

_Detection commands:_

- `ping [neighbors-neighbor-address]` validate reachability
- `show {ip | ipv6} protocols` display networks advertisements and _Stub, receive-only_ mode
- `show {ip | ipv6} eigrp neighbors` verify that the neighborship was established
- `show access-list` checks if exists any blocking statement for the route
- `show {ip | ipv6} route` confirm that the desired route was learned by EIGRP (D)

## OSPF

As EIGRP, OSPF works on top of IP and uses protocol number _89_. It does not rely on the functions of TCP or UDP. It has 2 versions.

- _OSPFv2:_ Uses multicast and unicast, rather than broadcast. _224.0.0.5_ to send information to all OSPF routers and _224.0.0.6_ to send information to DR or Backup Designated Router (BDR) routers.
- _OSPFv3:_ _Supports IPv6_ and uses _FF02::5_ to send information to all OSPF routers and _FF02::6_ to send information to DR or Backup Designated Router (BDR) routers.

### Implementing

#### Classic vs Named Mode

- _OSPFv2_

  ```
  (config)# ipv6 unicast routing
  (config)# access-list [ACL_number] {permit | deny} [ipv4] [subnet]
  (config)# ip prefix-list seq {seq_number | PFL_name} PFL {permit | deny} [ipv4_cidr]

  (config)# [ipv6] router ospf [as_number]
  (config-router)# router-id [id]
  (config-router)# [no]? passive-interface {default | [interface]}
  (config-router)# auto-cost reference-bandwidth [10000] !!! 10Gbps
  (config-router)# area [area_number] filter-list prefix [PFL_name] {in | out}
  (config-router)# area [area_number] range [ABR_summary_prefix] [not-advertise]?
  (config-router)# summary-prefix [ASBR_summary_prefix]
  (config-router)#[no]? distribute-list [ACL_number] {in | out}
  (config-router)#[no]? distribute-list prefix [PFL_name] {in | out}
  (config-router)# default information originate {always | metric | metric-type | route-map}?

  (config)# interface [interface]
  (config-if)# {ip | ipv6} ospf [as_number] area [area_number]
  (config-if)# {ip | ipv6} ospf cost [0-65,535]
  (config-if)# {ip | ipv6} ospf {hello-interval | dead-interval} [seconds]
  (config-if)# {ip | ipv6} ospf network {point-to-point | broadcast}

  # clear {ip | ipv6} ospf process
  ```

- _OSPFv3_

  ```
  (config)# router ospfv3 [as_number]
  (config-router)# address-family {ipv4 | ipv6} unicast
  (config-router-af)# router-id [id]
  (config-router-af)# [no]? passive-interface {default | [interface]}
  (config-router-af)# auto-cost reference-bandwidth [10000] !!! 10Gbps
  (config-router-af)# area [area_number] range [ABR_summary_prefix] [not-advertise]?
  (config-router-af)# summary-prefix [ASBR_summary_prefix]
  (config-router-af)# default-information originate

  (config)# interface [interface]
  (config-if)# ospfv3 [as_number] {ipv4 | ipv6} area [area_number]
  (config-if)# ospfv3 [as_number] {ipv4 | ipv6} cost [0-65,535]
  (config-if)# ospfv3 [as_number] {ipv4 | ipv6} {hello-interval | dead-interval} [seconds]
  (config-if)# ospfv3 [as_number] {ipv4 | ipv6} network {point-to-point | broadcast}
  ```

Default information originate modes:

- Always
- Metric

### Troubleshooting

#### Verification Commands (OSPFv3 & OSPFv2)

```
!!! Verify OSPF AS, networks & passive intefaces
# show {ip | ipv6} protocols

!!! Verify OSPF adjacency
# show ospfv3 neighbor
# show {ip | ipv6} neighbor

!!! Verify OSPF  & timers in an interface
# show ospfv3 interface [interface]
# show {ip | ipv6} ospf interface [interface]

!!! Displays all routes learned by OSPF
# show ip route ospfv3
# show {ip | ipv6} route ospf

!!! Displays all OSPF routes and it´s LSA level
# show ospfv3 database
# show {ip | ipv6} ospf database

!!! Verify Prefix-List statements
# show ip prefix-list
```

## DMVPN

The Dynamic Multipoint VPN helps to scale IPsec VPN hub-to-spoke and spoke-to-spoke designs. The benefits of an DMVPN are:

- **Hub router configuration reduction**: The hub router defines the crypto ACLs to manage all spoke routers mGRE tunnels.
- **Automatic IPsec initiation**: GRE uses **Next Hop Resolution Protocol (NHRP)** to configure and resolve the peer destination address, allowing IPsec to be immediatly integrated in the GRE point-to-point tunnel withouth peering configuration.
- **Dynamically addressed spoke routers**: Using the **NHRP**, the spoke routers know how to connect to other spokes to form a full-mesh IPsec VPN network through the hub router.

DMVPN supports 2 types of authentication:

- _PSK:_ Used for hub-spoke deployments.
- _PKI-based IKE authentication:_ Recommended for fully meshed DMPVPNs

**DMVPN Building Blocks**

- **mGRE**: Allow the establishment of spoke-to-spoke tunnels, supporting IP multicast (with dynamic routing protocols) and non-multicast protocols to build tunnels between devices using only one GRE instance. A single interface (VTI) is enabled to manage all mGRE peers (in GRE is nneded a VTI for each peer).
- **NHRP**: Maps VTIs with physical interfaces, so spokes (clients) can dynamically discover other spokes to create direct tunneling between them, through the hub (server) which mantains the database relating external (physical) and internal (tunnel VTI) addresses.
- **IKE + IPsec**: Provides key management and transmission protection to encapsulate GRE tunnels of on-demand (dynamic) sessions between spokes

### GRE

GRE uses the **IP protocol 47** to encapsulate and manage the multiprotocol transport and IP multicasting between 2 or more sites. Using IPsec, the GRE tunnel is encrypted in 2 modes:

- **Transport Mode (default):** Encrypts data only and authenticates the IP header, without cipher.
- **Tunnel Mode:** Encrypts up to the IP header by creating a new tunnel IP packet.

Some limitations of GRE are that dynamic routing protocols (such as EIGRP, OSPF, BGP or ODT) are necessarily to verify the end-to-end tunnel state and in some cases the **IPsec + GRE header (20 bytes)** is fregmented by the MTU size limit, causing **MTU & IP fragmentation related issues**.

```
!!! Verify tunnel state as 'up' and protocol 'multi-GRE/IP'
# show interface tunnel [tunnel_id]

!!! Display NHRP mappings ('Type: static' in spoke output for Hub connections & 'Type: dynamic' for other spokes)
# show ip nhrp

!!! Verify connectivity to Hub or other spokes (full-mesh) as next-hop
# show ip route next-hop-override
```

### HUB Configuration

```
!!! Debug all spokes-to-hub connections (tunnel state 'UP-ACTIVE'), IKE SA information & # of packets "enc'ed"/"dec'ed" (if no packets there must be a problem with IPsec SAs)
# show dmvpn detail

!!! Verify the ISAKMP SA status per spoke (expected = 'QM_IDLE')
# show crypto isakmp sa

!!! Verify the IPsec status
# show crypto ipsec sa

0. (Optional) Create an IKE policy (if not default policy is used)
(config)# crypto isakmp policy [ike_number]
(config-isakmp)# encryption aes 256
(config-isakmp)# hash sha384
(config-isakmp)# authentication pre-share
(config-isakmp)# group 14
(config-isakmp)# exit

1. Create an ISAKMP PSK pre-shared key
(config)# crypto isakmp key [password] address 0.0.0.0

2. Define a transform-set & assign it to an IPsec profile
(config)# crypto ipsec transform-set [set_name] esp-aes 256 esp-sha384-hmac
(cfg-crypto-trans)# mode { tunnel | transport } !!! When using DMVPN + IPsec 'mode transport' should be used to reduce overhead
(cfg-crypto-trans)# exit
(config)# crypto ipsec profile [ipsec_profile_name]
(cfg-crypto-trans)# set transform-set [set_name]
(cfg-crypto-trans)# exit

3. Configure VTI for mGRE (point-to-multipont)
(config)# interface tunnel [tunnel_number]
(config-if)# ip address [vti_hub_ipv4] [submask]  !!! Must be in the same segment as SPOKES
(config-if)# tunnel protection ipsec profile [ipsec_profile_name]
(config-if)# tunnel mode gre multipoint
(config-if)# tunnel source [interface_to_WAN]
(config-if)# tunnel key [tunnel_key]
(config-if)# ip nhrp network-id [nhrp_network_id]
(config-if)# ip nhrp authentication [nhrp_key]
(config-if)# ip nhrp map multicast dynamic
(config-if)# ip mtu 1400
(config-if)# ip tcp adjust-mss 1360
(config-if)# ip ospf network point-to-multipoint  !!! Use if OSPF will be enabled

4. Announce VTI & internal LAN networks through OSPF
(config)# router ospf [process_id]
(config-router)# network [vti_hub_ipv4] [submask] area [area_id]  !!! Must be in the same segment & area as SPOKES
(config-router)# network [internal_network] [submask] area [area_id]

------------ PHASE 2: SPOKE-to-SPOKE -------------

5. Change VTI to work as broadcast
(config)# interface tunnel [tunnel_number]
(config-if)# ip ospf network broadcast

------------ PHASE 3: Redirect -------------

(config)# interface tunnel [tunnel_number]
(config-if)# ip nhrp redirect
(config-if)# ip ospf network point-to-multipoint
```

### Spokes Configuration

```
********************** SPOKES ***********************
!!! Debug network IDs or auth key mismatch (if 'req-failed > 0')
# show ip nhrp nhs detail

-------------- PHASE 1: HUB-to-SPOKE ---------------
0. (Optional) Create an IKE policy (must MATCH with HUB)
(config)# crypto isakmp policy [ike_number]
(config-isakmp)# encryption aes 256
(config-isakmp)# hash sha384
(config-isakmp)# authentication pre-share
(config-isakmp)# group 14
(config-isakmp)# exit

1. Create an ISAKMP PSK pre-shared key (must MATCH with HUB)
(config)# crypto isakmp key [password] address 0.0.0.0

2. Define a transform-set & assign it to an IPsec profile
(config)# crypto ipsec transform-set [set_name] esp-aes 256 esp-sha384-hmac
(cfg-crypto-trans)# mode { tunnel | transport } !!! When using DMVPN + IPsec 'mode transport' should be used to reduce overhead
(cfg-crypto-trans)# exit
(config)# crypto ipsec profile [ipsec_profile_name]
(cfg-crypto-trans)# set transform-set [set_name]
(cfg-crypto-trans)# exit

3. Configure the spokes as GRE (point-to-point) tunnels
(config)# interface tunnel [tunnel_number]
(config-if)# ip address [vti_hub_ipv4] [submask]
(config-if)# tunnel mode gre ip !!! Default mode
(config-if)# tunnel source [interface_to_WAN]
(config-if)# tunnel destination [hub_interface_to_WAN_ipv4]
(config-if)# tunnel key [tunnel_key]
(config-if)# ip nhrp network-id [nhrp_network_id]
(config-if)# ip nhrp authentication [nhrp_key]
(config-if)# ip nhrp nhs [vti_hub_ipv4]
(config-if)# ip nhrp map multicast [hub_interface_to_WAN_ipv4]
(config-if)# ip nhrp map [vti_hub_ipv4] [hub_interface_to_WAN_ipv4]
(config-if)# tunnel protection ipsec profile [ipsec_profile_name]
(config-if)# ip address [vti_spoke_ipv4] [submask]
(config-if)# ip mtu 1400
(config-if)# ip tcp adjust-mss 1360
(config-if)# ip ospf network point-to-multipoint

4. Announce VTI & internal LAN networks through OSPF
(config)# router ospf [process_id]
(config-router)# network [vti_spoke_ipv4] [submask] area [area_id]  !!! Must be in the same segment & area as HUB
(config-router)# network [internal_network] [submask] area [area_id]

------------ PHASE 2: SPOKE-to-SPOKE -------------

5. Change GRE point-to-point to mGRE
(config)# interface tunnel [tunnel_number]
(config-if)# no tunnel destination
(config-if)# tunnel mode gre multipoint
(config-if)# ip ospf network broadcast
(config-if)# ip ospf priority 0

-------------- PHASE 3: Shortcut -----------------

(config)# interface tunnel [tunnel_number]
(config-if)# ip nhrp shortcut
(config-if)# no ip ospf priority 0
(config-if)# ip ospf network point-to-multipoint
(config-if)# shutdown
(config-if)# no shutdown

# traceroute 192.168.3.1 source Lo0
```

## Miscellaneous Commands

- _Infinite Ping_
  ```
  # ping [ipv4] repeat [100000] size 1000
  !!! Use [Ctrl + 6] to abort
  ```
- _Infinite Traceroute_

  ```
  # traceroute [ipv4] probe [100000]
  !!! Use [Ctrl + 6] to abort
  ```

- _Auth Keychain_

  ```
  !!! Display all keychains and keys store along with its valid dates
  # show key chain

  (config)# key chain [keychain-name]
  (config-keychain)# key [key-number]
  (config-keychain)# key-string [key-name]

  !!! Key time expiration (optional)  00:00:00   Jan 1 2020   23:00:00  Dec 31 2029
  (config-keychain)# accept-lifetime [hh:mm:ss] [start-date] [hh:mm:ss] [end-date]

  %%% REVIEW %%%
  (config-keychain)# send-lifetime 22:45:00 Apr 8 2019 infinite
  ```
