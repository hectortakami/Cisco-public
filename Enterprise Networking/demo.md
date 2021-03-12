################## ALL #######################
en
conf t
hostname HTAKAMI-SW1
no ip domain lookup

line console 0
logging synchronous
password cisco
login

line vty 0 4
logging synchronous
privilege level 15
password cisco
login
transport input all

do wr
end

################## R1 #######################
conf t
int e0/0
ip address 192.168.1.1 255.255.255.0
no shut
exit

--------- ROAS TO SW1 --------------
int e0/0.10
encapsulation dot1q 10
ip address 10.10.10.10 255.255.255.0
no shut
end
wr

################## R2 #######################
conf t
int e0/0
ip address 192.168.1.2 255.255.255.0
no shut
exit

--------- ROAS TO SW2 --------------
int e0/0.20
encapsulation dot1q 20
ip address 20.20.20.20 255.255.255.0
no shut
end
wr

--------- DHCP TO VLAN20 --------------
conf t
ip dhcp excluded-address 20.20.20.1 20.20.20.200
ip dhcp pool VLAN20-POOL
network 20.20.20.0 /24
default-router 20.20.20.20
lease 0 0 10

################## SW1 #######################

--------- TRUNK TO R1 --------------
conf t
int e0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allow vlan 1,10
switchport nonegotiate
end
wr

################## SW2 #######################

--------- TRUNK TO R2 --------------
conf t
int e0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allow vlan 1,20
switchport nonegotiate
end
wr

################## VPC1 #######################
--------- GET DHCP --------------
dhcp
