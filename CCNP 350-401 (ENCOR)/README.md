# CCNP 350-401 ENCOR

## Enterprise Network Architecture

### Control & Data Plane

In Layer 3 devices, the control plane and the data plane are relatively independent. The exchange of routing information is performed in the control plane and data packets are forwarded by the data plane.

_Control Plane_

- Exchange of routing information (from _Routing Protocols_ and _IP Routing Table_)
- Manages the internal data and control circuits for the packet-forwarding and control functions.
- Extracts the other routing and packet-forwarding-related control information from Layer 2 and Layer 3 bridging and routing protocols and the configuration data, and then conveying the information to the interface module for control of the data plane.
- Collects the data plane information, such as traffic statistics, from the interface module to the route processor.
- Handles certain data packets that are sent from the Ethernet interface modules to the route processor.

_Data Plane_
Manages the incoming/outcoming IP packages only fowarding traffic (_Fowarding table_)

### Network Architecture Parts

The following are some key concepts that you should address when creating a reliable and versatile network design:

- _Self-healing:_ Continuously on and available (Redundant & Resilient).
- _Self-defending:_ Protecting the organization and its users.
- _Self-optimizing:_ Adapting to changing needs, beyond the limits of basic standards (prioritize the mission critical services).
- _Self-aware:_ Driving change through insight into network activity by reporting insight traffic (tools as NetFlow, Network Based Application Recognition NBAR). Provides analytics in the user´s usage of the network.

1. _Enterprise Campus:_ It consists in several network segments grouped into one or multiple buildings in a small geographical area as a single network. Follows the 3-tier architecture:
   - _Access Layer:_ Convergence wirless, voice and data. Grant user access to network devices, generally incorporates switched LAN devices to access the corporate network. Controls PoE, wired and wireless connection. It can be presented with routing capabilities or without it:
     - _(Bridging) L2 Only Access Layer:_ VLANs terminated at distribution layer and 50% of the entire links are blocked for STP. It´s a cheaper alternative.
     - _(Routing) L2 + L3 Access Layer:_ All access and distribution devices participate in routing. VLANs terminate at access layer, requires more planning in order to keep separated the traffic (internal from external or guest traffic).
   - _Distribution Layer:_ Provides policy-based connectivity to segment workgroups and isolate network problems in a campus environment. Availability, Default Gateway Redundancy/Recovery, QoS and Load Balancing.
   - _Core Layer (Backbone):_ Provides non-stop connectivity, high redundancy, scalability, fast convergence and high availability to interconnect between several campus.
   - _Data Center Sub-Module:_ Centralizes server resources as DNS, email, etc. Typically supports network management services for the enterprise, including monitoring, logging, and troubleshooting. Spine-Leaf Architecture.
2. Enterprise Edge: Provides connectivity between the enterprise network (by it´s core layer) and the ISP. Also it contains sub-modules to connetct with remote access and site-to-site VPN & WAN services (MPLS, Metro Ethernet, Synchronous Optical Networks SONET, etc...)
3. _Service Provider Egde:_ Determined by the ISP SLAs. It provides connectivity between the enterprise main site and it´s remote locations.
4. Remote Locations: Geographical distant parts of an enterprise network (branch offices, teleworkers or remote data centers)

_Flat Network:_ No subnets, one broadcast domain (all devices in one giant switch) all devices share same BW
_SD-Access:_ Allows for software-defined segmentation and policy enforcement based on user identity and group membership, integrated with Cisco TrustSec technology.
_DNA Center:_ Manage automation of network device configuration.

## Switching Paths

_**MAC/CAM table:** The table is built by recording the **source MAC address** and **inbound port** of all incoming frames._

_**TCAM table:** The TCAM table stores **ACL**, **QoS**, and other information that is generally associated with **upper-layer processing**. Multiple TCAMs allow switches to **perform different checks in parallel, thus shortening the packet-processing time**._

### Switching Mechanisms

- **Process switching:** Slowest method, every packet examined by CPU (Cyclic Redundancy Check CRC), and all forwarding decisions made in software. It greatly degrades performance and is generally used only as a last re-sort or during troubleshooting.

- **Fast switching:** Faster method, first packet in each flow examined by CPU, and forwarding decision cached in hardware for subsequent packets in flow rewritting with corresponding link addresses and is sent over the outgoing interface.

- **Cisco Express Forwarding (CEF):**

  - Default switching mode
  - Fastest method, hardware forwarding table is created regardless of traffic flows and all packets switched using hardware

  A router with Cisco Express Forwarding enabled uses information from tables that are built by the CPU, such as the _routing table_ and the _Address Resolution Protocol (ARP) table_, to build hardware-based tables that are known as the _Forwarding Information Base (FIB) and adjacency tables_. These tables are then used to make hardware-based forwarding decisions for all frames in a data flow, even the first frame.

  ```
  !!! Verify the FIB table
  # show ip cef

  !!! Verify the content of the adjacency table
  # show adjacency

  !!! Verify the Cisco Express Forwarding status of the particular interface
  # show ip interface [interface] | include CEF

  ```

  ```
  !!! Disable Cisco Express Forwarding globally
  (config)# [no]? ip cef

  !!! Disable Cisco Express Forwarding per interface
  (config)# interface [interface]
  (config-if)# [no]? ip route-cache cef
  (config-if)# end
  ```

  Some features are NOT compatible with Cisco Express Forwarding such in a topology that is _using load-balanced Layer 3 paths_ where CEF could degradate the service.

## EBGP

BGP is used as an `Exterior Gateway Protocol (EGP)` to exchange routing information between different autonomous systems (collection of networks under a common administrative domain).

BGP uses `TCP 179`, which provides reliable connection. Administrative Distance for `EBGP (20)` and `IBGP (200)`. To enable BGP adjacency we need to _point manually to the neighbor's IP_ in the other side of the outgoing connection. Only 1 BGP instance (local AS) can be set up in a router.

### BGP Tables

- _BGP Neighbor Table:_ Keeps track of the adjacency with it´s neighbors _(explicity configured)_ by periodically sending a BGP/TCP keepalive messages (Open, Keepalive, Update and Notification). _Note: In order to enable adjacency, the only requirement is that both routers must be in **different AS** and be **directly connected with each other**_
- _BGP Table:_ Collection of every route updates from each neighbor that successfully establishes an adjacency. _Note: Only the best route is installed in the routing table, every other learned network will be placed in the BGP forwarding database to be a backup_

### Path Selection

BGP chooses the best path _based on routing policies_, not link characteristics (like bandwidth in IGP).

1. Prefer **highest weight** attribute which is _local significant_ to the router.
2. Prefer **highest local preference** which brings _priority to one of the neighbors_ to use it as reference to reach an AS. It is _only exchange on IBGP_ process.
3. Prefer **shortest AS-Path** (least number of autonomous systems). The AS-PATH _prepends all the AS crossed to reach the route_, meaning that (saw as an array) the destination AS will be in first position[0] and the origin AS will be at the end of the AS chain.
4. Prefer **lowest MED** attribute (exchanged between autonomous systems), this value is initially exhange between AS.

_Path Attributes_

These attributes help with calculating the best route when multiple paths to a particular destination exist.

- **Well-known:** These attributes are required to be present for every route in every update

  - _[Mandatory] **Origin**:_ Sets the way the way the route was injected using the `network` command or via aggregation (route summarization within BGP). `'i' learned by IGP`, `'?' = redistribution` and `'e' = EGP(obsolete)`
  - _[Mandatory] **AS-Path**:_ Sequence of AS numbers through which the network is accessible
  - _[Mandatory] **Next-Hop**:_ Indicates he IP address of the next-hop router or `0.0.0.0` if the route is local to the router.

  - _[Discretionary] **Local Preference**:_ Achieve a consistent routing policy for exiting an AS.
  - _[Discretionary] **Atomic Aggregate**:_ Attached to a route that is created as a result of route summarization (called aggregation in BGP).

- **Optional:** Attributes that BGP implementations are not required to determine best path, but can be used to determined the best path only if the router recognize it´s implementation.

  - _[Transitive] **Aggregator**:_ Identifies the AS and the router to created a route summarization.
  - _[Transitive] **Community**:_ This attribute is a numerical value that can be attached to certain routes when they pass a specific point in the network.

  - _[Non-Transitive] **MED**:_ Is also called the _BGP metric_, can be used to **indicate to EBGP neighbors about the preferred path into an AS**.

### EBGP Configuration

```
!!! Display BGP configuration
# show running-config | section bgp

!!! Display routes learned by BGP, neighbor ID, metrics & origin
# show ip bgp summary
----------------------------------------------------------------
Routes with '>' are the designated routes (installed on IP routing table).
If next-hop is '0.0.0.0' means that the route is directly connected to the local router.
----------------------------------------------------------------

!!! Verify BGP session statistics & neighbors AS, IDs & networks announced/learned
# show ip bgp summary

!!! Verify BGP link state (Established)
# show ip bgp neighbors [adjacent_neighbor_IPv4]
```

```
(config)# router bgp [local_AS]
(config-router)# neighbor [adjacent_neighbor_IPv4] remote-as [neighbor_AS]
(config-router)# network [IPv4] mask [submask]
----------------------------------------------------------------
BGP announce ONLY local and directly connected routes through the `network` command.
----------------------------------------------------------------
```
