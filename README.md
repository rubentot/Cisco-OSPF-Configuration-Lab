Cisco OSPF Configuration Lab

**Completed:*** November 28, 2025
***Author:*** Ruben Tot (@rubentot)
***Tools:*** Cisco Packet Tracer 8.2
***Source:*** Flackbox / Neil Anderson — comprehensive OSPF lab covering basics to multi-area and DR election

This lab demonstrates OSPF configuration from basic single-area setup to advanced features like cost manipulation for load balancing, default route injection, multi-area conversion with summarization, and DR/BDR election on multi-access networks. The lab was completed 100% independently using the lab exercise PDF (topologies and questions), with no external resources beyond verification commands. All analysis, configurations, and CLI outputs are original to showcase understanding.
The repository contains CLI outputs, screenshots, configurations, and a detailed write-up for validation.



![Lab Topology](./images/3.JPG)

Core Configurations (All Routers)
Basic OSPF Enablement

```Bash
 router ospf 1
 network 10.0.0.0 0.255.255.255 area 0
 network 192.168.0.0 0.0.0.255 area 0
 auto-cost reference-bandwidth 100000  ; For modern bandwidth costs
 ```

 Cost Manipulation Example (for load balancing)
 
```Bash
 interface FastEthernet1/1
 ip ospf cost 1500
```

Default Route Injection
```Bash
ip route 0.0.0.0 0.0.0.0 203.0.113.2
router ospf 1
default-information originate
passive-interface FastEthernet1/1  ; Prevent advertising internals externally
```

Multi-Area and Summarization

router ospf 1
 area 1 range 10.0.0.0 255.255.0.0  ; Summarize into Area 0

DR Priority
```Bash
interface FastEthernet0/0
 ip ospf priority 100
```

Question Highlights & Answers
OSPF Basic Configuration

Q1: Enable loopback on R1-R5 with 192.168.0.x/32.
A: Sets stable Router ID.
```Bash
interface Loopback0
 ip address 192.168.0.1 255.255.255.255  ; For R1
```
Q2: Enable single-area OSPF, advertise 10.x and loopbacks (exclude 172.16/203.0.113).
A: Uses wildcard masks for precise advertisement.
```Bash
router ospf 1
 network 10.0.0.0 0.255.255.255 area 0
 network 192.168.0.0 0.0.0.255 area 0
```

Q3: Expected Router ID on R1?
A: Loopback: 192.168.0.1 (highest loopback preferred). Verified with show ip protocols.
```Bash
R1#sh ip protocols 
*** IP Routing is NSF aware *** 

Routing Protocol is "ospf 1" 
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 192.168.0.1 
  Number of areas in this router is 1. 1 normal 0 stub 0 
nssa 
  Maximum path: 4 
  Routing for Networks: 
    10.0.0.0 0.255.255.255 area 0 
    192.168.0.0 0.0.0.255 area 0 
  Routing Information Sources: 
    Gateway
Distance      Last Update 
    192.168.0.1
110      00:00:25 
    192.168.0.2
110      00:00:25 
    192.168.0.3
110      00:00:25 
    192.168.0.4
110      00:00:25 
    192.168.0.5
110      00:00:25 
  Distance: (default is 110)
```


Q4: Verify adjacencies.
A:show ip ospf neighbor shows FULL states with neighbors.

```Bash
R1#show ip ospf neighbor 

Neighbor ID     Pri   State
Dead Time   Address
Interface 
192.168.0.5       1   FULL/BDR        00:00:31    10.0.3.2
FastEthernet1/1 
192.168.0.2       1   FULL/DR
00:00:39    10.0.0.2
FastEthernet0/0
```

Q5: Verify routes in table.
A: All 10.x and loopbacks present as O routes. Example output from R1 shown in lab.

```Bash
R1#sh ip route 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP 
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2 
       E1 - OSPF external type 1, E2 - OSPF external type 2 
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2 
       ia - IS-IS inter area, * - candidate default, U - per-user static route 
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP 
       + - replicated route, % - next hop override 

Gateway of last resort is not set 


10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks 
C 10.0.0.0/24 is directly connected, FastEthernet0/0 
L 10.0.0.1/32 is directly connected, FastEthernet0/0 
C 10.0.1.0/24 is directly connected, FastEthernet0/1 
L 10.0.1.1/32 is directly connected, FastEthernet0/1 
C 10.0.2.0/24 is directly connected, FastEthernet1/0 
L 10.0.2.1/32 is directly connected, FastEthernet1/0 
C 10.0.3.0/24 is directly connected, FastEthernet1/1 
L 10.0.3.1/32 is directly connected, FastEthernet1/1 
O 10.1.0.0/24 [110/2] via 10.0.0.2, 00:03:13, FastEthernet0/0 
O 10.1.1.0/24 [110/3] via 10.0.0.2, 00:02:51, FastEthernet0/0 

   [110/3] via 10.0.3.2, 00:02:51, FastEthernet1/1 
O 10.1.2.0/24 [110/3] via 10.0.3.2, 00:02:51, FastEthernet1/1 
O 10.1.3.0/24 [110/2] via 10.0.3.2, 00:02:51, FastEthernet1/1 

192.168.0.0/32 is subnetted, 5 subnets 
C 192.168.0.1/32 is directly connected, Loopback0 
O 192.168.0.2/32 [110/2] via 10.0.0.2, 00:03:25, FastEthernet0/0 
O 192.168.0.3/32 [110/3] via 10.0.0.2, 00:03:13, FastEthernet0/0 
O 192.168.0.4/32 [110/3] via 10.0.3.2, 00:02:51, FastEthernet1/1 
O 192.168.0.5/32 [110/2] via 10.0.3.2, 00:03:25, FastEthernet1/1
```

Q6: Set reference bandwidth for 100 Gbps cost=1.
A:auto-cost reference-bandwidth 100000 on all routers.
```Bash
R1(config)#router ospf 1 
R1(config-router)#auto-cost reference-bandwidth 100000
```
OSPF Cost

Q7: Cost on FastEthernet links?
A: 1000 (100000 / 100). Verified with show ip ospf interface.
```Bash
R1#show ip ospf interface FastEthernet 0/0 
FastEthernet0/0 is up, line protocol is up 
Internet address is 10.0.0.1/24, Area 0 
Process ID 1, Router ID 192.168.0.1, Network Type BROADCAST, Cost: 1000
```

Q8: Effect on cost to 10.1.2.0/24 from R1?
A: Increases from 3 to 3000 due to higher reference.
Before:
```Bash
R1#sh ip route 
O 10.1.2.0/24 [110/3] via 10.0.3.2, 00:02:51, FastEthernet1/1
```

After:
```Bash
R1#sh ip route 
O 10.1.2.0/24 [110/3000] via 10.0.3.2, 00:01:04, FastEthernet1/1 
```

Q9: Path in table for 10.1.2.0/24?
A: Via R5 (10.0.3.2) as lowest cost.
```Bash
R1#sh ip route 
O 10.1.2.0/24 [110/3000] via 10.0.3.2, 00:01:04, FastEthernet1/1 
```

Q10: Change this so that traffic from R1 to 10.1.2.0/24 will be load balanced via both R2 and R5.
A: Since we changed the reference bandwidth, all interfaces have a cost of 1000. The current path from R1 > R5 > R4 has a cost of 3000 (the cost of the destination interface itself is also counted in the total cost). The path from R1 > R2 > R3 > R4 has a cost of 4000. The easiest way to configure both paths to have the same cost is to configure the links from R1 > R5 and R5 > R4 to have a cost of 1500 each. (R1 > R5 = 1500, plus R5 > R4 = 1500, plus cost of 10.1.2.0/24 interface on R4 = 1000. Total = 4000).

```Bash
R1(config)#int f1/1 
R1(config-if)#ip ospf cost 1500 

R5(config)#int f0/0 
R5(config-if)# ip ospf cost 1500 
R5(config)#int f0/1 
R5(config-if)# ip ospf cost 1500 

R4(config)#int f1/0 
R4(config-if)# ip ospf cost 1500
```

Q11: Verify that traffic to the 10.1.2.0/24 network from R1 is load balanced via both R2 and R5.
A: Two routes shown in table.
```Bash
R1#sh ip route 
O 10.1.2.0/24 [110/4000] via 10.0.3.2, 00:00:25, FastEthernet1/1 
   [110/4000] via 10.0.0.2, 00:00:25, FastEthernet0/0
```


Default Route Injection

Q12: Ensure that routers R1 to R5 have a route to the 203.0.113.0/24 network. Internal routes must not be advertised to the Service Provider at 203.0.113.2.
A: The 203.0.113.0/24 network must be added to the OSPF process on R4, and interface FastEthernet 1/1 facing the service provider configured as a passive interface to avoid sending out internal network information.
```Bash
R4(config)#router ospf 1 
R4(config-router)#passive-interface f1/1 
R4(config-router)#network 203.0.113.0 0.0.0.255 area 0
```

Q13: Verify that routers R1 to R5 have a path to the 203.0.113.0/24 network.
A: Route present in table
```Bash
R1#sh ip route 
O 203.0.113.0/24 [110/3001] via 10.0.3.2, 00:00:03, FastEthernet1/1 
   [110/3001] via 10.0.0.2, 00:00:03, FastEthernet0/0
```
Q14: Configure a default static route on R4 to the Internet via the service provider at 203.0.113.2.
A:
```Bash
R4(config)#ip route 0.0.0.0 0.0.0.0 203.0.113.2
```
Q15: Ensure that routers R1 to R5 learn via OSPF how to reach the Internet.
A:
```Bash
R4(config)#router ospf 1 
R4(config-router)#default-information originate
```
Q16: Verify routers R1 to R5 have a route to the Internet.
A: Default route as O*E2, load balanced
```Bash
R1#sh ip route 
O*E2 0.0.0.0/0 [110/1] via 10.0.3.2, 00:00:06, FastEthernet1/1 
   [110/1] via 10.0.0.2, 00:00:06, FastEthernet0/0
```

Multi-Area OSPF

![Lab Topology](./images/1.JPG)

Q17: Convert the network to use multi-area OSPF. R3 and R4 should be backbone routers, R1 a normal router in Area 1, and R2 and R5 ABRs as shown in the diagram below. Save your changes to the startup config and reboot the routers to ensure the changes take effect.
A: R3 and R4 require no change as all their interfaces are already in Area 0. R1’s interfaces need to be reconfigured to be in Area 1 rather than Area 0

```Bash
R1(config)#router ospf 1 
R1(config-router)#network 10.0.0.0 0.255.255.255 area 1 
R1(config-router)#network 192.168.0.0 0.0.0.255 area 1 
R1#copy running-config startup-config 
R1#reload
```

R2 interface FastEthernet 0/1 should remain in Area 0. FastEthernet 0/0 needs to be reconfigured to be in Area 1. I used a 10.0.0.0/8 network statement originally so I need to remove that and add more granular statements.
```Bash
R2(config)#router ospf 1 
R2(config-router)#no network 10.0.0.0 0.255.255.255 area 0 
R2(config-router)#network 10.1.0.0 0.0.0.255 area 0 
R2(config-router)#network 10.0.0.0 0.0.0.255 area 1 
R2#copy running-config startup-config 
R2#reload
```

R5 interface FastEthernet 0/0 should remain in Area 0. FastEthernet 0/1 needs to be reconfigured to be in Area 1.
```Bash
R5(config)#router ospf 1 
R5(config-router)#no network 10.0.0.0 0.255.255.255 area 0 
R5(config-router)#network 10.1.3.0 0.0.0.255 area 0 
R5(config-router)#network 10.0.3.0 0.0.0.255 area 1 
R5#copy running-config startup-config 
R5#reload
```

Q18: Verify the router’s interfaces are in the correct areas.
A:show ip ospf interface confirms.
```Bash
R2#show ip ospf interface 
Loopback0 is up, line protocol is up 
  Internet address is 192.168.0.2/32, Area 0 
FastEthernet0/1 is up, line protocol is up 
  Internet address is 10.1.0.2/24, Area 0 
FastEthernet0/0 is up, line protocol is up 
  Internet address is 10.0.0.2/24, Area 1 
! Output truncated
```

Q19: Verify routers R1 to R5 have formed adjacencies with each other.
A:
```Bash
R1#show ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface 
192.168.0.5       1   FULL/DR         00:00:33    10.0.3.2        FastEthernet1/1 
192.168.0.2       1   FULL/DR         00:00:31    10.0.0.2        FastEthernet0/0
```

Q20: What change do you expect to see on R1’s routing table? Verify this (give the routing table a few seconds to converge).
A: The networks beyond R2 and R5 will appear as Inter Area routes (apart from the default route which will appear as an external route as it was redistributed into OSPF).
```Bash
R1#sh ip route 
10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks 
C 10.0.0.0/24 is directly connected, FastEthernet0/0 
O IA 10.1.0.0/24 [110/2000] via 10.0.0.2, 00:03:10, FastEthernet0/0 
O IA 10.1.1.0/24 [110/3000] via 10.0.0.2, 00:03:10, FastEthernet0/0 
O IA 10.1.2.0/24 [110/4000] via 10.0.0.2, 00:03:10, FastEthernet0/0 
O IA 10.1.3.0/24 [110/4500] via 10.0.0.2, 00:02:59, FastEthernet0/0 
192.168.0.0/32 is subnetted, 5 subnets 
C 192.168.0.1/32 is directly connected, Loopback0 
O IA 192.168.0.2/32 [110/1012] via 10.0.0.2, 00:03:10, FastEthernet0/0 
O IA 192.168.0.3/32 [110/2012] via 10.0.0.2, 00:03:10, FastEthernet0/0 
O IA 192.168.0.4/32 [110/3012] via 10.0.0.2, 00:03:10, FastEthernet0/0 
O IA 192.168.0.5/32 [110/4512] via 10.0.0.2, 00:02:59, FastEthernet0/0 
O IA 203.0.113.0/24 [110/3001] via 10.0.0.2, 00:03:10, FastEthernet0/0 
O*E2 0.0.0.0/0 [110/1] via 10.0.0.2, 00:02:59, FastEthernet0/0
```
Q21: Do you see less routes in R1’s routing table? Why or why not?
A: R1 has the same amount of routes in its routing table because OSPF does not perform automatic summarisation. You must configure manual summarisation to reduce the size of the routing table.
Q22: Configure summary routes on the Area Border Routers for the 10.0.0.0/16 and 10.1.0.0/16 networks.
A:
```Bash
R2(config)#router ospf 1 
R2(config-router)#area 0 range 10.1.0.0 255.255.0.0 
R2(config-router)#area 1 range 10.0.0.0 255.255.0.0 

R5(config)#router ospf 1 
R5(config-router)#area 0 range 10.1.0.0 255.255.0.0 
R5(config-router)#area 1 range 10.0.0.0 255.255.0.0
```

Q23: Verify R1 now sees a single summary route for 10.1.0.0/16 rather than individual routes for the 10.1.x.x networks.
A:
```Bash
R1#sh ip route 
10.0.0.0/8 is variably subnetted, 9 subnets, 2 masks 
C 10.0.0.0/24 is directly connected, FastEthernet0/0 
C 10.0.1.0/24 is directly connected, FastEthernet0/1 
C 10.0.2.0/24 is directly connected, FastEthernet1/0 
C 10.0.3.0/24 is directly connected, FastEthernet1/1 
O IA 10.1.0.0/16 [110/2000] via 10.0.0.2, 00:00:04, FastEthernet0/0 
192.168.0.0/32 is subnetted, 5 subnets 
C 192.168.0.1/32 is directly connected, Loopback0 
O IA 192.168.0.2/32 [110/1012] via 10.0.0.2, 00:05:27, FastEthernet0/0 
O IA 192.168.0.3/32 [110/2012] via 10.0.0.2, 00:05:27, FastEthernet0/0 
O IA 192.168.0.4/32 [110/3012] via 10.0.0.2, 00:05:27, FastEthernet0/0 
O IA 192.168.0.5/32 [110/4512] via 10.0.0.2, 00:05:16, FastEthernet0/0 
O IA 203.0.113.0/24 [110/3001] via 10.0.0.2, 00:05:27, FastEthernet0/0 
O*E2 0.0.0.0/0 [110/1] via 10.0.0.2, 00:05:16, FastEthernet0/0
```

Q24: Verify R1 is receiving a summary route for the 10.1.0.0/16 network from both R2 and R5.
A:
```Bash
R1#sh ip ospf database 
            OSPF Router with ID (192.168.0.1) (Process ID 1) 

                Router Link States (Area 1) 

Link ID         ADV Router      Age         Seq#       Checksum Link count 
192.168.0.1     192.168.0.1     18          0x80000005 0x00536E 5 
192.168.0.2     192.168.0.2     27          0x80000003 0x0069ED 1 
192.168.0.5     192.168.0.5     1890        0x80000003 0x00C490 1 

                Net Link States (Area 1) 

Link ID         ADV Router      Age         Seq#       Checksum 
10.0.0.1        192.168.0.1     18          0x80000002 0x00DF0E 
10.0.3.1        192.168.0.1     18          0x80000002 0x00E8FE 

                Summary Net Link States (Area 1) 

Link ID         ADV Router      Age         Seq#       Checksum 
192.168.0.5     192.168.0.5     408         0x80000006 0x00e987 
10.0.3.0        192.168.0.5     408         0x80000007 0x007e7d 
192.168.0.4     192.168.0.5     408         0x80000009 0x00bbd1 
203.0.113.0     192.168.0.5     408         0x8000000a 0x00ebdb 
192.168.0.3     192.168.0.5     408         0x8000000b 0x00f4ab 
192.168.0.2     192.168.0.5     408         0x8000000c 0x003084 
10.0.0.0        192.168.0.5     408         0x8000000d 0x002c09 
192.168.0.2     192.168.0.2     1079        0x80000005 0x001c5c 
192.168.0.3     192.168.0.2     1079        0x80000006 0x004446 
192.168.0.4     192.168.0.2     1079        0x80000008 0x006932 
203.0.113.0     192.168.0.2     1079        0x80000009 0x00993c 
192.168.0.5     192.168.0.2     398         0x80000006 0x00308a 
10.0.3.0        192.168.0.2     398         0x80000007 0x00c282 
192.168.0.1     192.168.0.5     393         0x80000010 0x003c74 
10.1.0.0        192.168.0.5     82          0x80000015 0x007679 
10.1.0.0        192.168.0.2     67          0x8000001f 0x000ee4 

                Summary ASB Link States (Area 1) 

Link ID         ADV Router      Age         Seq#       Checksum 
192.168.0.4     192.168.0.2     27          0x80000002 0x00EE9B 
192.168.0.4     192.168.0.5     1889        0x80000002 0x00433A 

                Type-5 AS External Link States 

Link ID         ADV Router      Age         Seq#       Checksum Tag 
0.0.0.0         192.168.0.4     207         0x80000002 0x00152F 1
```

Q25: R1 is routing traffic to 10.1.0.0/16 via R2 only. Why is it not load balancing the traffic through both R2 and R5?
A: We configured the link from R1 to R5 to have a higher cost than the link from R1 to R2 earlier.
```Bash
R1#sh run | begin interface FastEthernet1/1 
Building configuration... 

Current configuration : 100 bytes 
! 
interface FastEthernet1/1 
 ip address 10.0.3.1 255.255.255.0 
 ip ospf cost 1500
```

DR and BDR Designated Routers

![Lab Topology](./images/2.JPG)

Q26: Enable a loopback interface on routers R6 to R9. Use the IP address 192.168.0.x/32, where ‘x’ is the router number. For example 192.168.0.6/32 on R6.
A:

```Bash
interface Loopback0 
 ip address 192.168.0.6 255.255.255.255  ; For R6
```

 Q27: Enable OSPF for Area 0 on the Loopback 0 and FastEthernet 0/0 interfaces on routers R6 to R9.
A:
```Bash
router ospf 1 
 network 172.16.0.0 0.0.0.255 area 0 
 network 192.168.0.0 0.0.0.255 area 0
 ```
Q28: Set the reference bandwidth on routers R6 to R9 so that a 100 Gbps interface will have a cost of 1.
A:
```Bash
router ospf 1 
 auto-cost reference-bandwidth 100000
 ```

Q29: Which routers do you expect to be the DR and BDR on the Ethernet segment? Verify this.
A: OSPF priority has not been set so all routers will have the default of 1. R9 and R8 will be elected as the DR and BDR respectively because they have the highest Router IDs (because they have the highest IP addresses on their loopback interfaces).

```Bash
R6#show ip ospf interface FastEthernet 0/0 
FastEthernet0/0 is up, line protocol is up 
Internet address is 172.16.0.6/24, Area 0 
Process ID 1, Router ID 192.168.0.6, Network Type BROADCAST, Cost: 1000 
Transmit Delay is 1 sec, State DROTHER, Priority 1 
Designated Router (ID) 192.168.0.9, Interface address 172.16.0.9 
Backup Designated Router (ID) 192.168.0.8, Interface address 172.16.0.8 

R6#show ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface 
192.168.0.8       1   FULL/BDR        00:00:31    172.16.0.8      FastEthernet0/0 
192.168.0.7       1   2WAY/DROTHER    00:00:39    172.16.0.7      FastEthernet0/0 
192.168.0.9       1   FULL/DR         00:00:39    172.16.0.9      FastEthernet0/0 

R9#show ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface 
192.168.0.8       1   FULL/BDR        00:00:31    172.16.0.8      FastEthernet0/0 
192.168.0.7       1   2WAY/DROTHER    00:00:39    172.16.0.7      FastEthernet0/0 
192.168.0.6       1   FULL/DROTHER    00:00:39    172.16.0.6      FastEthernet0/0
```

Q30: Set R6 as the Designated Router without changing any IP addresses.
A: Configure a higher OSPF priority on R6.
```Bash
interface FastEthernet0/0 
 ip ospf priority 100 
end 
clear ip ospf process
```

Q31: Verify R6 is the Designated Router.
A:
```Bash
R6#show ip ospf interface FastEthernet 0/0 
FastEthernet0/0 is up, line protocol is up 
Internet address is 172.16.0.6/24, Area 0 
Process ID 1, Router ID 192.168.0.6, Network Type BROADCAST, Cost: 1000 
Transmit Delay is 1 sec, State DR, Priority 100 
Designated Router (ID) 192.168.0.6, Interface address 172.16.0.6 
Backup Designated Router (ID) 192.168.0.8, Interface address 172.16.0.8 

R6#show ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface 
192.168.0.8       1   FULL/BDR        00:00:31    172.16.0.8      FastEthernet0/0 
192.168.0.7       1   2WAY/DROTHER    00:00:39    172.16.0.7      FastEthernet0/0 
192.168.0.9       1   FULL/DROTHER    00:00:39    172.16.0.9      FastEthernet0/0 

R9#show ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface 
192.168.0.8       1   FULL/BDR        00:00:31    172.16.0.8      FastEthernet0/0 
192.168.0.7       1   2WAY/DROTHER    00:00:39    172.16.0.7      FastEthernet0/0 
192.168.0.6     100   FULL/DR         00:00:39    172.16.0.6      FastEthernet0/0
```








 
