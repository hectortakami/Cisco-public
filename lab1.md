# Switch 3

```
-------------- IP ADDRESSING --------------
ipv6 unicast-routing

int lo 33
ip address 33.33.33.33 255.255.255.255
ipv6 address 2000:33:33:33::33/128
no shut
int vlan 1
ip address 192.168.31.33 255.255.255.0
ipv6 address 2000:192:168:31::33/64
no shut

-------------- OSPF CLASSIC --------------
router ospf 1
router-id 33.33.33.33
network 192.168.31.0 0.0.0.255 a 31
--------------  STUB AREA --------------
area 31 stub
no area 31 stub
area 31 stub no-summary !!! Allow only LSA 3 (default route)
```

# Router 1

```
-------------- IP ADDRESSING --------------
ipv6 unicast-routing

int lo 1
ip address 1.1.1.1 255.255.255.255
ipv6 address 2000:1:1:1::1/128
int s1/0
ip address 172.16.12.1 255.255.255.240
ipv6 address 2000:172:16:12::1/64
int e0/3
ip address 192.168.31.1 255.255.255.0
ipv6 address 2000:192:168:31::1/64
no shut
int e0/0
ip address 192.168.12.1 255.255.255.0
ipv6 address 2000:192:168:12::1/64
no shut

-------------- OSPF CLASSIC --------------
router ospf 1
router-id 1.1.1.1
network 172.16.12.0 0.0.0.15 a 112
network 192.168.12.0 0.0.0.255 a 112
network 192.168.31.0 0.0.0.255 a 31
network 1.1.1.1 0.0.0.0 a 51
-------------- VIRTUAL LINK --------------
area 112 virtual-link 2.2.2.2
--------------  STUB AREA --------------
area 31 stub
no area 31 stub
area 31 stub no-summary !!! Allow only LSA 3 (default route)

-------------- EIGRP NAMED --------------

---- PREFIX-LIST (Prevents to announce Lo1 IPv6-AF) ----
ipv6 prefix-list PFL_Lo1 permit 2000:192:168:31::/64

router eigrp EIGRP
address-family ipv4 unicast as 101
network 0.0.0.0
exit
address-family ipv6 unicast as 101
af-interface default
passive-interface
af-interface s1/0
no passive-interface
no shut
exit
af-interface lo1
shut
exit
topology base
distribute-list prefix-list PFL_Lo1 out
```

# Switch 1

```
-------------- IP ADDRESSING --------------
ipv6 unicast-routing

int vlan 1
ip address 192.168.12.11 255.255.255.0
ipv6 address 2000:192:168:12::11/64
no shut

-------------- OSPF CLASSIC --------------
router ospf 1
router-id 11.11.11.11
network 192.168.12.0 0.0.0.255 a 112
```

# Router 2

```
-------------- IP ADDRESSING --------------
ipv6 unicast-routing

int lo 2
ip address 2.2.2.2 255.255.255.255
ipv6 address 2000:2:2:2::2/128
int e0/0
ip address 192.168.23.2 255.255.255.0
ipv6 address 2000:192:168:23::2/64
no shut
int e0/1
ip address 192.168.12.2 255.255.255.0
ipv6 address 2000:192:168:12::2/64
no shut
int s1/1
ip address 172.16.12.2 255.255.255.240
ipv6 address 2000:172:16:12::2/64
no shut
int s1/0
ip address 172.16.23.2 255.255.255.240
ipv6 address 2000:172:16:23::2/64
no shut
int e0/2
ip address 192.168.24.2 255.255.255.0
no shut
exit

-------------- OSPF CLASSIC --------------
router ospf 1
network 172.16.23.0 0.0.0.15 a 0
network 192.168.23.0 0.0.0.255 a 0
network 172.16.12.0 0.0.0.15 a 112
network 192.168.12.0 0.0.0.255 a 112
-------------- VIRTUAL LINK --------------
area 112 virtual-link 1.1.1.1
-------- Modify AD (received for EIGRP->[OSPF] re-distribution) -----------
distance ospf external 90

-------------- EIGRP NAMED --------------
router eigrp EIGRP
address-family ipv4 unicast as 101
network 0.0.0.0
exit
address-family ipv6 unicast as 101
af-interface default
passive-interface
af-interface s1/0
no passive-interface
af-interface s1/1
no passive-interface
exit

--------------- HSRP TRACKING ----------------
int s1/0
shut
end

router ospf 4
network 192.168.24.0 0.0.0.255 a 0
end

conf t
track 10 interface ethernet 0/2 line-protocol
exit
ip sla 1
icmp-echo 192.168.24.4 source-interface e0/2
threshold 100
timeout 100
frequency 1
exit
ip sla schedule 1 start-time now life forever
track 200 ip sla 1 reachability
exit
int e0/0
standby 23 ip 192.168.23.254
standby 23 priority 110
standby 23 preempt
standby 23 track 10 decrement 21
no standby 23 track 10 decrement 21
standby 23 track 200 decrement 30
```

# Switch 2

```
-------------- IP ADDRESSING --------------
ipv6 unicast-routing

int vlan 1
ip address 192.168.23.22 255.255.255.0
ipv6 address 2000:192:168:23::22/64
no shut

-------------- OSPF CLASSIC --------------
router ospf 1
router-id 22.22.22.22
network 192.168.23.0 0.0.0.255 a 0
```

# Router 3

```
-------------- IP ADDRESSING --------------
ipv6 unicast-routing

int s1/1
ip address 172.16.23.3 255.255.255.240
ipv6 address 2000:172:16:23::3/64
no shut
int lo 3
ip address 3.3.3.3 255.255.255.255
ipv6 address 2000:3:3:3::3/128
int lo 10
ip address 10.0.0.3 255.255.255.0
ipv6 address 2000:10::3/64
int e0/1
ip address 192.168.23.3 255.255.255.0
ipv6 address 2000:192:168:23::3/64
no shut
int e0/0
ip address 192.168.34.3 255.255.255.0
ipv6 address 2000:192:168:34::3/64
no shut

-------------- OSPF CLASSIC --------------
router ospf 1
router-id 3.3.3.3
network 172.16.23.0 0.0.0.15 a 0
network 192.168.23.0 0.0.0.255 a 0
----------- DEFAULT INFORMATION ALWAYS -----------
!!! Redistribute all connected routes and not annonuced through OSPF (10.0.0.0 & 3.3.3.3) without default-routes created
default information originate always
--------- Re-Distribute EIGRP -> [OSPF] ---------
redistribute eigrp 2021 subnets

-------------- EIGRP CLASSIC --------------
router eigrp 2021
network 3.3.3.3 0.0.0.0
network 192.168.34.0 0.0.0.255
--------- Re-Distribute OSPF -> [EIGRP] ---------
redistribute ospf 1
default-metric 8000000 500 255 1 1514   !!! Values from Lo3

------------- EIGRP NAMED + CLASSIC -------------
router eigrp 101
network 0.0.0.0
exit
ipv6 eigrp 101
exit
int s1/1
ipv6 eigrp 101

--------------- HSRP TRACKING ----------------
router ospf 4
router-id 3.3.3.3
network 192.168.34.0 0.0.0.255 a 0

int e0/1
standby 23 ip 192.168.23.254
standby 23 preempt
```

# Router 4

```
-------------- IP ADDRESSING --------------
ipv6 unicast-routing

int e0/0
ip address 192.168.34.4 255.255.255.0
ipv6 2000:192:168:34::4/64
no shut
exit
int lo 4
ip address 4.4.4.4 255.255.255.255
no shut
exit
int e0/2
ip address 192.168.24.4 255.255.255.0
no shut
exit

-------------- EIGRP CLASSIC --------------
router eigrp 2021
network 192.168.34.0 0.0.0.255
-------- Modify AD (received for OSPF->[EIGRP] re-distribution) -----------
distance eigrp 90

--------------- HSRP TRACKING ----------------
router ospf 4
router-id 4.4.4.4
network 192.168.34.0 0.0.0.255 a 0
network 4.4.4.4 0.0.0.0 a 0
network 192.168.24.0 0.0.0.255 a 0
```

# VPC

```
ip 192.168.23.1/24 192.168.23.254
save
ping 4.4.4.4 -t
```

# Switch 4

```
--------------- ACTIVATE HSRP TRACKING ----------------
int e0/0
shut
```
