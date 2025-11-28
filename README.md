Cisco OSPF Configuration Lab

Completed: November 28, 2025
Author: Ruben Tot (@rubentot)
Tools: Cisco Packet Tracer 8.2
Source: Flackbox / Neil Anderson — comprehensive OSPF lab covering basics to multi-area and DR election
This lab demonstrates OSPF configuration from basic single-area setup to advanced features like cost manipulation for load balancing, default route injection, multi-area conversion with summarization, and DR/BDR election on multi-access networks. The lab was completed 100% independently using the lab exercise PDF (topologies and questions), with no external resources beyond verification commands. All analysis, configurations, and CLI outputs are original to showcase understanding.
The repository contains CLI outputs, screenshots, configurations, and a detailed write-up for validation.

1) Configure Loopbacks on R1–R5
interface loopback0
 ip address 192.168.0.X 255.255.255.255

2) Enable Single-Area OSPF on R1–R5
router ospf 1
 network 10.0.0.0 0.255.255.255 area 0
 network 192.168.0.0 0.0.0.255 area 0

3) Expected Router ID on R1 + Verification

Router ID = highest loopback → 192.168.0.1

show ip protocols

4) Verify Adjacencies Between R1–R5
show ip ospf neighbor

5) Verify 10.x.x.x and Loopback Routes on All Routers
show ip route

6) Set Reference Bandwidth (R1–R5)
router ospf 1
 auto-cost reference-bandwidth 100000

7) Calculate the Cost of FastEthernet Links

Reference (100000) / FastEthernet (100) = 1000

Verify:

show ip ospf interface FastEthernet 0/0

8) Effect of Bandwidth Change on Path Cost

Before change: cost ≈ 3
After change: cost ≈ 3000

show ip route

9) Which Path R1 Uses for 10.1.2.0/24?

R1 → R5 → R4 is chosen.

show ip route

10) Configure Load-Balancing for 10.1.2.0/24
# R1
interface f1/1
 ip ospf cost 1500

# R5
interface f0/0
 ip ospf cost 1500
interface f0/1
 ip ospf cost 1500

# R4
interface f1/0
 ip ospf cost 1500

11) Verify Load-Balancing
show ip route 10.1.2.0

12) Advertise 203.0.113.0/24 Only Inside OSPF
router ospf 1
 passive-interface f1/1
 network 203.0.113.0 0.0.0.255 area 0

13) Verify 203.0.113.0/24 Routes on All Routers
show ip route 203.0.113.0

14) Add Default Route on R4
ip route 0.0.0.0 0.0.0.0 203.0.113.2

15) Inject Default Route into OSPF
router ospf 1
 default-information originate

16) Verify Default Route on R1–R5
show ip route

17) Convert to Multi-Area OSPF (Areas 0 and 1)
R1 → Area 1
router ospf 1
 no network 10.0.0.0 0.255.255.255 area 0
 no network 192.168.0.0 0.0.0.255 area 0
 network 10.0.0.0 0.255.255.255 area 1
 network 192.168.0.0 0.0.0.255 area 1

R2 → ABR
router ospf 1
 no network 10.0.0.0 0.255.255.255 area 0
 network 10.1.0.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.255 area 1

R5 → ABR
router ospf 1
 no network 10.0.0.0 0.255.255.255 area 0
 network 10.1.3.0 0.0.0.255 area 0
 network 10.0.3.0 0.0.0.255 area 1

18) Verify Interfaces are in Correct Areas
show ip ospf interface

19) Verify Adjacencies in Multi-Area Design
show ip ospf neighbor

20) What Changes on R1’s Routing Table?

Routes outside Area 1 become Inter-Area (IA).

show ip route

21) Does R1 Have Fewer Routes? Why Not?

No — because OSPF does not auto-summarize.

22) Configure Summary Routes on ABRs
R2
router ospf 1
 area 0 range 10.1.0.0 255.255.0.0
 area 1 range 10.0.0.0 255.255.0.0

R5
router ospf 1
 area 0 range 10.1.0.0 255.255.0.0
 area 1 range 10.0.0.0 255.255.0.0

23) Verify R1 Sees 10.1.0.0/16 Summary
show ip route

24) Verify R1 Receives Summary from Both ABRs
show ip ospf database

25) Why Doesn't R1 Load-Balance Between R2 and R5?

Because interface F1/1 still has ip ospf cost 1500, making the R5 path more expensive.

26) Create Loopbacks on R6–R9
interface loopback0
 ip address 192.168.0.X 255.255.255.255

27) Enable OSPF on R6–R9 (Area 0)
router ospf 1
 network 172.16.0.0 0.0.0.255 area 0
 network 192.168.0.0 0.0.0.255 area 0

28) Set Reference Bandwidth (R6–R9)
router ospf 1
 auto-cost reference-bandwidth 100000

29) Expected DR/BDR & Verification

Expected:

DR = R9

BDR = R8

Verify:

show ip ospf interface FastEthernet0/0
show ip ospf neighbor

30) Make R6 the DR
interface FastEthernet0/0
 ip ospf priority 100
end
clear ip ospf process

31) Verify R6 is Now DR
show ip ospf interface FastEthernet0/0
show ip ospf neighbor

