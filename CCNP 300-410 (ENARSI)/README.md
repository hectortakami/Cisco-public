# CCNP 300-410 ENARSI

## EIGRP

EIGRP uses *Reliable Transport Protocol (RTP)* which *runs above the IP layer as protocol number 88*, use both unicast/multicast (*224.0.0.10* or *FF02::A*) to take care of reliable exchange of information.

- *Partial Triggered Updates:* Do not send periodic updates to it´s neighbors, the updates are sent only when a metric or route change and only to the affected routers.
- *Fast Convergence:* Quickly adapts to topology changes discoverying alternative routes
- *Sophisticated Metric (EIGRP-Wide):* iOS 15 allows 64 bits (commonly 32 bits only) in load balancing metric to have more control in traffic flow and distribution *Note: Supported on bandwidths above 1 gigabit and up to 4.2 terabits only*
- *VLSM (IPv4 & IPv6):* Classless Routing Protocol that advertise subnets and can be manually configure tu summarize them.
- *Seamless connectivity across multiple data-link media:* Unlikely OSPF that must be configure differently depending on the cable media (Frame Relay or Ethernet). EIGRP operates in both LAN or WAN environments non depending on the media.

### Implementing

- *Neighbor Table:* Stores the primary IP address and the directly connected interface to the established router adjacencies
- *Topology Table:* Contains the relation between neighbors and it´s *advertised destination routes* with it´s *metric*
- *Routing Table:* Contains all the routes learned from the successors.

#### Classic vs Named Mode

- *Classic Mode*

  The EIGRP configuration for IPv4 and IPv6 must be done separate (not at the same mode level) and the association to the EIGRP AS is done by different levels (IPv6 associates it with the interface and IPv4 declare the network at router config level).

  *Note:* In EIGRP, *default routes CANNOT be directly injected* as in OSPF, where you can use the `default-information originate` command, we need to `redistribute` them defining specifically those protocols that we want to include. 

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
- *Named Mode*

  Centralizes the EIGRP configuration for IPv4 and IPv4. Uses *64-bit* wide metrics. EIGRP named mode configuration is available in Cisco IOS Software Release 15.0(1)M, 12.2(33)SRE, 12.2(33)XNE and later releases.
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

- *Bandwidth*: The least value of the bandwidth for all links between the router, which computes the metric, and the destination. This parameter is referred to as “throughput” in EIGRP named mode.
- *Delay*: The cumulative delay that is obtained as the sum of values of all delays for all links between the source and destination. This parameter is referred to as “latency” in EIGRP named mode.
- *Reliability*: The worst reliability between source and destination, which is based on keepalives.
- *Load*: The worst load on the link between the source and the destination, which is based on the packet rate and the configured bandwidth of the interface

The metric and default k-values for each EIGRP mode result as follows:

_It´s not recommended to change the default k-values for any of the 2 EIGRP modes_

- *Classic Mode (k=5)*: 
  - Uses *bandwidth* and *delay*
  - *k1=1*, k2=0, *k3=1*, k4=0, k5=0 
  - Delay for normal links (Microseconds)  
  - Metric formula =  [(K1 * bandwidth + [(K2 * bandwidth) / (256 – load)] + K3 * delay) * K5 / (K4 + reliability)] * 256
  - *Simplified Metric (using default k-values)* = ([10^7 / min-bandwidth-bps] * 256) + ([sum-delays-ms * 100] * 256)

- *Named Mode (k=6)*: 
  - Uses *throughtput* and *latency*
  - k1=1, k2=0, k3=1, k4=0, k5=0, *k6=0*
  - 64-bits wide metric
  - Supports RIP 
  - Latency for high-speed links (Picoseconds) 

#### Glossary
- *Reported Distance (RD):* Equals the metric formula. 
- *Feasibile Distance (FD):* Equals the metric formula + the neighbor cost to it´s reported path
- *Feasibility Condition (FC):* The reported distance must be *less* than feasible distance, which mantains the split horizon. Any EIGRP route wich metrics accomplish the FC will be considered as a FS but only the best (lowest FD) will be installed in the routing table as Successor.
  ```
  !!! FC = RD < FD

  # show ip eigrp topology
  !!! Feasibile Successors (P) means routes in passive state
  P 192.168.0.0/24, 1 successors, FD is 409600
                          FD     RD
        via 172.16.1.1 (409600/128256), Ethernet0/0   !!! Actual FS
        via 172.16.2.1 (1664000/128256), Ethernet0/1  !!! Next FS 
  ```
- *Split Horizon (loop-free topology):* Avoid in propagate the routes learned by the same interface where were received.
- *Sucessor:* The route with the best metric destination (lowest cost) that is installed in the routing table.
- *Feasibile Sucessor (FS):* The next best route that accomplish the feassible condition.

### Optimizing

#### EIGRP Queries (Problem)

*EIGRP Queries* are another package (as hello, update, hold and ACK) sended when an EIGRP route dont have a successor or FS left, it asks to all it´s neighbors in *multicast/unicast* manner (excepting the succesor by the split-horizon principle) for another FS route. If the neighbor doesn´t know another FS it spreads the query to others until someone replies. This creates a *cascade query propagation* that can be limit by using *Route Summarization* or *EIGRP stub routing features.* 

##### Stub Routing

- Improves network stability optimizing large scale EIGRP networks
- Reduce resource utilization (limiting the amount of query messages in the network)
- Simplifies remote router configuration (Hub-Spoke topologies)
- Speeds up convergence

Stub routers configured with the `eigrp stub` commands under the router mode (classic) or address-family (named) share it´s *directly connected and summary routes by default*. The modes configurable by the command are: 
- `connected` advertise directly connected routes *(default without being specified)*
- `summary` advertise summarized routes *(default without being specified)*
- `redistribute (D EX)` advertise networks learned by other dynamic routing protocols
- `static` advertise static and default routes
- `receive-only` do NOT advertise it networks but can lear them from other routers

If an EIGRP Query is not reply within *90 secs* (even with both an alternate FS route or a not found ackwoledgement) a *SIA-Query* is sended from the query emitter to it´s neighbor with the closest un-reply. If the direct neighbor doesn´t have the problem and it already send a reply for the query response with a *SIA-Reply*. If the neighbor of the non-replying neighbor didn´t answer in *3 minutes (default)* the non-reply link turns into *Stuck-In-Active (SIA) state* but the EIGRP process keeps going.


##### Route Summarization

- Limits the query scope
- Faster response (Any router having a FS with the query prefix or subnet replies)

EIGRP supports route summarization at any router, unlike OSPF, which requires that summarization be performed only at area border routers (ABR) or autonomous system border routers (ASBR). Route summarization *works best when the subnet planning process considers route summarization.*

#### Load Balancing

EIGRP can distribute traffic over multiple links leading to the same destination to increase the effective network bandwidth. It supports load balancing over equal-metric paths and also over unequal-metric paths.

- *Equal-Cost Multipath (ECMP):* Distribute traffic across and up to *4 equal cost paths* by default with the *same metric.* The maximum equal cost paths to use can be change using the `maximum-paths` command.
- *Equal-Metric Load Balancing:* 
  - The `variance` that is set to *1 as default*, but is disabled. It can be changed to use multiple links to foward traffic with the only condition to accomplish the FC in the FS links. 
  - The *variance can be set from 1-128* and is obtain with the maximum metric divided by the lowest metric from the links we want to use for load balancing. `variance = max-metric / min-metric`
  - The route must be *loop free*, by *meeting the feasibility condition*.
  - The metric of the route must be *lower than* the metric of the successor, multiplied by the value of the variance.

#### EIGRP Authentication

Authentication is used to ensure that your EIGRP routers only form neighbor relationships with legitimate routers and that they only accept EIGRP packets from legitimate routers.

*EIGRP Message Digest 5 (MD5)* authentication can be configured in *classic mode or named mode*, while *HMAC-SHA-256* authentication can be configured in *named mode only.* The EIGRP authentication prevents hackers to establish neighbor relationships and DoS attacks.

Using a *password is simpler to configure* but if the password needs to be changed, the _neighbor adjacency will drop when the password is changed_ on one of the routers. Using a *Keychain allows the passwords to be rotated* without dropping the neighbor adjacency.



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

*Cause Problems* 
- The interface between the devices is down.
- The routers have mismatching EIGRP autonomous systems.
- The EIGRP process is not enabled on one of the interfaces that connects the devices.
- One of the interfaces that connects the devices is configured as a passive interface.

*Solutions* 
- *Solution 1:* Make sure it´s not related with an L2 problem or the connected interfaces are in *shutdown state*.
- *Solution 2:*  If the interfaces are up, validate you have the *same EIGRP AS name/number* in both sides. If not, remove it and re-enter the correct data with the desired advetised networks.
- *Solution 3:* Verify that the interfaces between EIGRP neighbors are active in `no passive-interface` state. And the required *interfaces are associated* with the correct EIGRP process.
- *Solution 4:* In case you are using authentication, make sure to mach the exact same `authentication modes` for MD5 or HMAC-SHA-256. The `password` and `key-id & key-string (keychain)` must be *exactly the same* in all routers. And finally verify the correct configuration and *valid times of the keys in keychain*. _Note: It´s important to verify the configured or obtained NTP device time to validate keys expiration_

*Detection commands:* 
  - `show {ip | ipv6} interface brief` verify that the connection is `up/up`
  - `show {ip | ipv6} protocols` validate the configured AS number/name *(must match)* & passive interfaces
  - `show {ip | ipv6} eigrp interfaces` displays all interfaces where EIGRP is enabled *(handy for EIGRP IPv6)*
  - `show run | sec eigrp` displays EIGRP configurations
  - `show key chain` presents keychain configuration *(must match in case is used)*
  - `show clock` presents time and date to validate keys expirancy
  - `show {ip | ipv6} eigrp neighbors` verify that the neighborship was established

#### EIGRP Routing Issues

The neighborship can be established but the advertised networks from the EIGRP neighbor don´t appear in the routing table resulting on an unreacheable ping.

*Cause Problems* 
- Networks are not being advertised on remote routers.
- An access list is blocking advertisements of remote networks.
- Automatic route summarization is causing confusion in your discontiguous network.

*Solutions* 
- *Solution 1:* Make sure you *had advertised the route*. For *directly connected routes* in the EIGRP (classic/named) mode and add a `network` statement so the address can be shared. For *non-connected routes*, make sure to issue the `redistribute` command to share routes learned staticaly or by other dynamic routing protocols.
- *Solution 2:* The neighbor route can be configured as `stub receive-only` mode which doesn´t share route info.

_Note: If the problem persists, verify if there is an *ACL blocking* advertisement or the *default auto summarization* (for iOS before 15 Release) was correctly configured, this could cause router confussion summarizing all routes with classful masks known as a *discontinous network problem* run the `no auto summary` command in EIGRP router process to prevent this behaviour._

*Detection commands:*  
  - `ping [neighbors-neighbor-address]` validate reachability
  - `show {ip | ipv6} protocols` display networks advertisements and *Stub, receive-only* mode
  - `show {ip | ipv6} eigrp neighbors` verify that the neighborship was established
  - `show access-list` checks if exists any blocking statement for the route
  - `show {ip | ipv6} route` confirm that the desired route was learned by EIGRP (D)



## Notes

- *Infinite Ping*
  ```
  # ping [ipv4] repeat 100000 size 1000
  !!! Use [Ctrl + 6] to abort
  ```
- *Auth Keychain*
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


