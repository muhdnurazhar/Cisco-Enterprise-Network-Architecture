# Cisco-Based-Network-Architecture

> A secure, dual-stack multi-site enterprise network featuring advanced routing, high availability, and certificate-based VPN connectivity.

## Introduction
This repository presents a comprehensive simulation of a production-grade enterprise network based on **WorldSkills Malaysia Module C (IT Network System Administration)**. 

The project demonstrates advanced Cisco networking skills in designing and implementing a secure, highly available, and scalable infrastructure connecting Headquarters, Branch office, and Remote sites across a simulated public internet.

## Project Objective
To build a reliable, secure, and resilient enterprise network that ensures:
- Constant uptime and redundancy
- Secure device and user access
- Seamless multi-protocol routing
- Encrypted communication between all sites

## Site Breakdown
- **Headquarters**: Core network with EIGRP, HSRP, and DMVPN Hub.
- **Branch**: OSPFv2/OSPFv3 domain with controlled DR/BDR and summarization.
- **ISP & Remote Sites**: eBGP connectivity, SCEP CA, and dynamic DMVPN spokes.

## Key Objectives
1. **Centralized Authentication** – RADIUS AAA with local PBKDF2 fallback.
2. **High Availability** – HSRP gateway redundancy + BGP default route failover.
3. **Multi-Protocol Routing** – Mutual redistribution with route tagging between EIGRP, OSPFv2, and OSPFv3.
4. **Secure WAN Connectivity** – Certificate-authenticated Dual-Stack DMVPN with IKEv2 IPsec.
## Technical Architecture & Implementation Workflow

<img width="872" height="385" alt="image" src="https://github.com/user-attachments/assets/b7d628f5-c49b-42f1-93c9-05bbca850df8" />

### Core Technologies
- **Switching**: VTPv3, EtherChannel (LACP & PAgP), Rapid STP, Port Security
- **Routing**: EIGRP Named Mode, OSPFv2/OSPFv3, BGP, Route Redistribution with tagging
- **Redundancy**: HSRP, BGP Default Route Failover
- **Security**: RADIUS AAA, PBKDF2 hashing, Port Security, IKEv2 IPsec + RSA Certificates
- **Services**: NAT, DHCP, NTP, SNMPv3
- **VPN**: Dual-Stack DMVPN (Phase 3) over IKEv2

# Network Infrastructure Verification & Audit Report

All objectives have been successfully implemented and tested.
---

### 1. Centralized Authentication & Device Access

**Verification:**
- RADIUS AAA configured on edge devices with fallback to local database.
- Users `radiusr1` (priv 15) and `radiusr2` (priv 3) authenticated via HQ-SRV.
- Local `admin` account uses secure PBKDF2 (Type 9) hashing.
### Verification Commands

**Step 1: Check RADIUS Server Status**
```bash
HQ-EDGE# show aaa servers 
RADIUS: id 1, priority 1, host 172.20.101.10, auth-port 1812, acct-port 1813
     State: current UP, duration 180s, previous duration 0s
     Dead: total time 0s, count 0
     Quarantined: No
     Authen: request 1, timeouts 0, failover 0, retransmission 0
             Response: accept 0, reject 0, challenge 0
             Response: unexpected 0, server error 0, incorrect 0, time 2362545ms
             Transaction: success 1, failure 0
             Throttled: transaction 0, timeout 0, failure 
```
**Step 2: Check Current User Login**
```bash
HQ-EDGE# show users
    Line       User       Host(s)              Idle       Location
*  0 con 0    radiusr1    idle                 00:00:00   172.20.101.10 (RADIUS)

  Interface User Privilege: Level 15 Granted via RADIUS attribute.
```
**Step 3: Check Local Backup User**
```bash
HQ-EDGE# show running-config | include username        
username admin privilege 15 secret 9 $9$.cl0q7sva2ju6v$PU0yeeGMRgppLtCxJlaBaz3RsEQajXHOh98M/YZyAUI
```

**Step 4: Test Failover (Backup Local Account)**

To verify that the device can still be accessed when the RADIUS server is unreachable, the following test was performed:

1. The links to the RADIUS server (HQ-SRV) were temporarily shut down:
```bash
HQ-EDGE#configure terminal  
Enter configuration commands, one per line.  End with CNTL/Z.
HQ-EDGE(config)#int range g0/1-2
HQ-EDGE(config-if-range)#shutdown 
*Jun 29 09:38:03.938: %DUAL-5-NBRCHANGE: EIGRP-IPv6 2026: Neighbor FE80::5054:FF:FE18:C203 (GigabitEthernet0/1) is down: interface down
*Jun 29 09:38:03.939: %DUAL-5-NBRCHANGE: EIGRP-IPv6 2026: Neighbor FE80::5054:FF:FE6D:E6F9 (GigabitEthernet0/2) is down: interface down
*Jun 29 09:38:03.946: %DUAL-5-NBRCHANGE: EIGRP-IPv4 2026: Neighbor 172.20.0.2 (GigabitEthernet0/1) is down: interface down
*Jun 29 09:38:03.946: %DUAL-5-NBRCHANGE: EIGRP-IPv4 2026: Neighbor 172.20.0.6 (GigabitEthernet0/2) is down: interface down
*Jun 29 09:38:05.876: %LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down
*Jun 29 09:38:05.908: %LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
*Jun 29 09:38:06.876: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down
*Jun 29 09:38:06.909: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to down
HQ-EDGE(config-if-range)#exit
HQ-EDGE(config)#exit
HQ-EDGE#exit
```

2. Console login was attempted using the local backup account.
```bash 




HQ-EDGE con0 is now available





Press RETURN to get started.




*Jun 29 09:38:24.494: %SYS-5-CONFIG_I: Configured from console by radiusr1 on console
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************

User Access Verification

Username: admin
Password: 

**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************
```

3. Login successful using local username admin and password Skills39.
```bash
HQ-EDGE#show users
    Line       User       Host(s)              Idle       Location
*  0 con 0     admin      idle                 00:00:00   

  Interface    User               Mode         Idle     Peer Address

HQ-EDGE#
```
### 2. High-Availability & Path Redundancy

**HSRP (Gateway Redundancy):**
- HQ-DSW1 & BR-DSW2 act as Active gateways for respective VLANs.
- Automatic failover tested successfully.

**Default Route Failover:**
- Normal condition: Default route via BGP from ISP.
- ISP link failure on HQ-EDGE → Route automatically failed over to BR-EDGE.
- Link restored → Route returned to primary BGP path.
- 
### Verification Commands

**Step 1: HSRP Verification (Dual Gateway)**
```bash
HQ-DSW1#show standby brief 
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl101       104  150 P Active  local           172.20.101.252  172.20.101.254
Vl102       204  100   Standby 172.20.102.252  local           172.20.102.254
```
```bash
BR-DSW2#show standby brief 
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl101       104  100   Standby 172.21.101.251  local           172.21.101.254
Vl102       204  150 P Active  local           172.21.102.251  172.21.102.254
```
**Step 2: Default Route Failover Test**

1. Normal Condition (Both BGP links up)
```bash
HQ-EDGE#show ip route | include 0.0.0.0
Gateway of last resort is 98.76.1.62 to network 0.0.0.0
B*    0.0.0.0/0 [20/0] via 98.76.1.62, 01:02:15
      10.0.0.0/32 is subnetted, 12 subnets
```
2. Failover Test (Simulate failure ISP links on HQ-EDGE)
```bash
HQ-EDGE#configure terminal 
Enter configuration commands, one per line.  End with CNTL/Z.
HQ-EDGE(config)#interface g0/0
HQ-EDGE(config-if)#shutdown 
*Jun 29 10:18:18.019: %BGP-5-NBR_RESET: Neighbor 98.76.1.62 reset (Interface flap)
*Jun 29 10:18:18.024: %BGP-5-ADJCHANGE: neighbor 98.76.1.62 Down Interface flap
*Jun 29 10:18:18.024: %BGP_SESSION-5-ADJCHANGE: neighbor 98.76.1.62 IPv4 Unicast topology base removed from session  Interface flapxit
```
3. Verification a new default route on backup links through BR-EDGE
```bash
HQ-EDGE#show ip route | include 0.0.0.0
Gateway of last resort is 172.20.0.6 to network 0.0.0.0
D*EX  0.0.0.0/0 [170/15360] via 172.20.0.6, 00:01:59, GigabitEthernet0/2
      10.0.0.0/32 is subnetted, 10 subnets
```
4. Restore HQ-EDGE links
```bash
HQ-EDGE(config)#int g0/0
HQ-EDGE(config-if)#no shutdown 
*Jun 29 10:22:31.674: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
*Jun 29 10:22:32.674: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
*Jun 29 10:22:38.906: %BGP-5-ADJCHANGE: neighbor 98.76.1.62 Up 
```

5. Verify default route return to BGP
```bash
HQ-EDGE(config)#do sh ip route 0.0.0.0
Routing entry for 0.0.0.0/0, supernet
  Known via "bgp 5000.3", distance 20, metric 0, candidate default path
  Tag 4259840001, type external
  Redistributing via eigrp 2026
  Advertised by eigrp 2026 metric 1000000 10 255 1 1500
  Last update from 98.76.1.62 00:01:24 ago
  Routing Descriptor Blocks:
  * 98.76.1.62, from 98.76.1.62, 00:01:24 ago
      Route metric is 0, traffic share count is 1
      AS Hops 1
      Route tag 4259840001
      MPLS label: none
```

### 3. Multi-Protocol Routing Optimization

**Verification:**
- Stable EIGRP adjacencies in HQ and DMVPN.
- Stable OSPFv2/OSPFv3 adjacencies in Branch (with correct DR/BDR election).
- Mutual redistribution performed at HQ-IR2.

**Route Tagging (Loop Prevention):**
- EIGRP → OSPF: **Tag 34**
- OSPF → EIGRP: **Tag 12**

**IPv6 Routing:**
- EIGRPv6 and OSPFv3 neighbors established.
- IPv6 redistribution working with correct tagging.

### Verification Commands

**Step 1: EIGRP Verfication on HQ & DMVPN**
```bash
HQ-IR2#show ip eigrp neighbors 
EIGRP-IPv4 VR(WSMB2026) Address-Family Neighbors for AS(2026)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
2   172.20.0.5              Gi0/0                    12 00:17:23  117   702  0  120
0   172.20.0.22             Gi0/2                    12 00:19:49   11   100  0  181
1   172.20.0.18             Gi0/1                    14 01:18:02    8   100  0  175


HQ-IR2#show ip route eigrp 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 172.20.0.5 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/61440] via 172.20.0.5, 00:07:14, GigabitEthernet0/0
      1.0.0.0/32 is subnetted, 1 subnets
D EX     1.1.1.1 [170/61440] via 172.20.0.5, 00:17:55, GigabitEthernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
D EX     2.2.2.2 [170/61440] via 172.20.0.5, 00:07:14, GigabitEthernet0/0
      3.0.0.0/32 is subnetted, 1 subnets
D EX     3.3.3.3 [170/61440] via 172.20.0.5, 00:07:14, GigabitEthernet0/0
      4.0.0.0/32 is subnetted, 1 subnets
D EX     4.4.4.4 [170/61440] via 172.20.0.5, 00:07:14, GigabitEthernet0/0
      8.0.0.0/32 is subnetted, 1 subnets
D EX     8.8.8.8 [170/61440] via 172.20.0.5, 00:07:14, GigabitEthernet0/0
      10.0.0.0/32 is subnetted, 12 subnets
D        10.20.0.1 [90/10880] via 172.20.0.5, 00:17:55, GigabitEthernet0/0
D        10.20.0.2 [90/16000] via 172.20.0.22, 00:17:50, GigabitEthernet0/2
                   [90/16000] via 172.20.0.18, 00:17:50, GigabitEthernet0/1
                   [90/16000] via 172.20.0.5, 00:17:50, GigabitEthernet0/0
D        10.20.0.4 [90/10880] via 172.20.0.22, 00:17:50, GigabitEthernet0/2
D        10.20.0.5 [90/10880] via 172.20.0.18, 00:17:50, GigabitEthernet0/1
D        10.30.1.1 [90/76805760] via 172.20.0.5, 00:07:05, GigabitEthernet0/0
D        10.30.1.2 [90/76805760] via 172.20.0.5, 00:07:06, GigabitEthernet0/0
      172.20.0.0/16 is variably subnetted, 13 subnets, 3 masks
D        172.20.0.0/30 [90/15360] via 172.20.0.5, 00:17:55, GigabitEthernet0/0
D        172.20.0.8/30 
           [90/15360] via 172.20.0.22, 00:17:50, GigabitEthernet0/2
D        172.20.0.12/30 
           [90/15360] via 172.20.0.18, 00:17:50, GigabitEthernet0/1
D        172.20.99.0/24 
           [90/15360] via 172.20.0.22, 00:17:50, GigabitEthernet0/2
           [90/15360] via 172.20.0.18, 00:17:50, GigabitEthernet0/1
D        172.20.101.0/24 
           [90/15360] via 172.20.0.22, 00:17:50, GigabitEthernet0/2
           [90/15360] via 172.20.0.18, 00:17:50, GigabitEthernet0/1
D        172.20.102.0/24 
           [90/15360] via 172.20.0.22, 00:17:50, GigabitEthernet0/2
           [90/15360] via 172.20.0.18, 00:17:50, GigabitEthernet0/1
      172.30.0.0/24 is subnetted, 2 subnets
D        172.30.10.0 
           [90/76810240] via 172.20.0.5, 00:07:05, GigabitEthernet0/0
D        172.30.20.0 
           [90/76810240] via 172.20.0.5, 00:07:06, GigabitEthernet0/0
      172.31.0.0/16 is variably subnetted, 3 subnets, 3 masks
D        172.31.1.0/24 
           [90/76805120] via 172.20.0.5, 00:17:55, GigabitEthernet0/0
```
**Step 2: OSPF Verfication on BR (Branch) Sites**
```bash
BR-IR1#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.21.0.5         0   FULL/  -        00:00:30    172.21.0.14     GigabitEthernet0/2
10.21.0.4         0   FULL/  -        00:00:34    172.21.0.10     GigabitEthernet0/1
10.21.0.1         0   FULL/  -        00:00:31    172.21.0.1      GigabitEthernet0/0
10.20.0.3         0   FULL/  -        00:00:30    172.31.0.1      GigabitEthernet0/3


BR-IR1#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 172.21.0.1 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 172.21.0.1, 01:19:37, GigabitEthernet0/0
      1.0.0.0/32 is subnetted, 1 subnets
O E2     1.1.1.1 [110/1] via 172.21.0.1, 00:09:07, GigabitEthernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
O E2     2.2.2.2 [110/1] via 172.21.0.1, 01:20:36, GigabitEthernet0/0
      3.0.0.0/32 is subnetted, 1 subnets
O E2     3.3.3.3 [110/1] via 172.21.0.1, 01:16:59, GigabitEthernet0/0
      4.0.0.0/32 is subnetted, 1 subnets
O E2     4.4.4.4 [110/1] via 172.21.0.1, 01:14:35, GigabitEthernet0/0
      8.0.0.0/32 is subnetted, 1 subnets
O E2     8.8.8.8 [110/1] via 172.21.0.1, 01:19:37, GigabitEthernet0/0
      10.0.0.0/32 is subnetted, 12 subnets
O E2     10.20.0.1 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
O E2     10.20.0.2 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
O E2     10.20.0.3 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
O E2     10.20.0.4 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
O E2     10.20.0.5 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
O        10.21.0.1 [110/4] via 172.21.0.1, 01:20:36, GigabitEthernet0/0
O        10.21.0.3 [110/5] via 172.21.0.14, 01:15:28, GigabitEthernet0/2
                   [110/5] via 172.21.0.1, 01:15:18, GigabitEthernet0/0
O        10.21.0.4 [110/2] via 172.21.0.10, 01:18:11, GigabitEthernet0/1
O        10.21.0.5 [110/2] via 172.21.0.14, 01:18:21, GigabitEthernet0/2
O E2     10.30.1.1 [110/20] via 172.31.0.1, 00:08:58, GigabitEthernet0/3
O E2     10.30.1.2 [110/20] via 172.31.0.1, 00:08:58, GigabitEthernet0/3
      172.20.0.0/24 is subnetted, 4 subnets
O E2     172.20.0.0 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
O E2     172.20.99.0 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
O E2     172.20.101.0 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
O E2     172.20.102.0 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
      172.21.0.0/16 is variably subnetted, 13 subnets, 3 masks
O        172.21.0.0/24 is a summary, 01:22:08, Null0
O        172.21.0.4/30 [110/4] via 172.21.0.1, 01:20:36, GigabitEthernet0/0
O        172.21.0.16/30 [110/4] via 172.21.0.14, 01:18:21, GigabitEthernet0/2
O        172.21.0.20/30 [110/4] via 172.21.0.10, 01:18:11, GigabitEthernet0/1
O IA     172.21.99.0/24 [110/2] via 172.21.0.14, 01:18:21, GigabitEthernet0/2
                        [110/2] via 172.21.0.10, 01:18:11, GigabitEthernet0/1
O IA     172.21.101.0/24 [110/2] via 172.21.0.14, 01:18:21, GigabitEthernet0/2
                         [110/2] via 172.21.0.10, 01:18:11, GigabitEthernet0/1
O IA     172.21.102.0/24 [110/2] via 172.21.0.14, 01:18:21, GigabitEthernet0/2
                         [110/2] via 172.21.0.10, 01:18:11, GigabitEthernet0/1
      172.30.0.0/24 is subnetted, 2 subnets
O E2     172.30.10.0 [110/20] via 172.31.0.1, 00:08:58, GigabitEthernet0/3
O E2     172.30.20.0 [110/20] via 172.31.0.1, 00:08:58, GigabitEthernet0/3
      172.31.0.0/16 is variably subnetted, 3 subnets, 3 masks
O E2     172.31.1.0/24 [110/20] via 172.31.0.1, 00:19:29, GigabitEthernet0/3
```

**Step 3: Route Redistribution & Tagging Verification**
```bash
HQ-IR2#sh route-map 
route-map RMv6-EIGRP-OSPF, deny, sequence 10
  Match clauses:
     tag 12 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map RMv6-EIGRP-OSPF, permit, sequence 20
  Match clauses:
  Set clauses:
    tag 34 
  Policy routing matches: 0 packets, 0 bytes
route-map RMv6-OSPF-EIGRP, deny, sequence 10
  Match clauses:
     tag 34 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map RMv6-OSPF-EIGRP, permit, sequence 20
  Match clauses:
  Set clauses:
    metric 1000000 1 255 1 1500
    tag 12 
  Policy routing matches: 0 packets, 0 bytes
route-map RM-EIGRP-OSPF, deny, sequence 10
  Match clauses:
     tag 12 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map RM-EIGRP-OSPF, permit, sequence 20
  Match clauses:
  Set clauses:
    tag 34 
  Policy routing matches: 0 packets, 0 bytes
route-map RM-OSPF-EIGRP, deny, sequence 10
  Match clauses:
     tag 34 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map RM-OSPF-EIGRP, permit, sequence 20
  Match clauses:
  Set clauses:
    metric 1000000 1 255 1 1500
    tag 12 
  Policy routing matches: 0 packets, 0 bytes



HQ-IR2#show ip eigrp topology 172.21.101.0/24        
EIGRP-IPv4 VR(WSMB2026) Topology Entry for AS(2026)/ID(10.20.0.3) for 172.21.101.0/24
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 1310720
  Descriptor Blocks:
  172.31.0.2, from Redistributed, Send flag is 0x0
      Composite metric is (1310720/0), route is External
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 10000000 picoseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 0
        Originating router is 10.20.0.3
      External data:
        AS number of route is 104
        External protocol is OSPF, external metric is 3
        **Administrator tag is 12**          ←←← This is the important tag
```

**Step 4: Branch Tagging Verification**
```bash
BR-IR1#sh ip route 172.20.101.0
Routing entry for 172.20.101.0/24
  Known via "ospf 104", distance 110, metric 20
  Tag 34, type extern 2, forward metric 4
  Last update from 172.31.0.1 on GigabitEthernet0/3, 00:28:28 ago
  Routing Descriptor Blocks:
  * 172.31.0.1, from 10.20.0.3, 00:28:28 ago, via GigabitEthernet0/3
      Route metric is 20, traffic share count is 1
      Route tag 34
```

**Step 5: IPv6 Routing Verification (OSPFv3 & EIGRPv6)**
Purpose:
To prove that IPv6 routing is also working with mutual redistribution between EIGRPv6 and OSPFv3, including proper tagging.

EIGRP IPv6:
```bash
HQ-IR2#show ipv6 eigrp neighbors 
EIGRP-IPv6 VR(WSMB2026) Address-Family Neighbors for AS(2026)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
2   Link-local address:     Gi0/2                    13 00:03:46  186  1116  0  12
    FE80::5054:FF:FEC5:513
1   Link-local address:     Gi0/1                    14 00:05:10   84   504  0  11
    FE80::5054:FF:FED3:F191
0   Link-local address:     Gi0/0                    12 00:05:27  112   672  0  7
    FE80::5054:FF:FEC0:8C2B
```

OSPF IPv6:
```bash
BR-IR1#show ipv6 ospf neighbor 

            OSPFv3 Router with ID (10.21.0.2) (Process ID 106)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
10.20.0.3         1   FULL/BDR        00:00:34    5               GigabitEthernet0/3
10.21.0.5         1   FULL/BDR        00:00:39    13              GigabitEthernet0/2
10.21.0.4         1   FULL/BDR        00:00:34    12              GigabitEthernet0/1
10.21.0.1         1   FULL/BDR        00:00:35    3               GigabitEthernet0/0
BR-IR1#
```

Verification of IPv6 Route Redistribution & Tagging: 

EIGRP: 
```bash
HQ-IR2#show ipv6 eigrp topology dead:beef:cafe:2101::/64        
EIGRP-IPv6 VR(WSMB2026) Topology Entry for AS(2026)/ID(10.20.0.3) for DEAD:BEEF:CAFE:2101::/64
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 1310720
  Descriptor Blocks:
  FE80::5054:FF:FEC0:B6C2, from Redistributed, Send flag is 0x0
      Composite metric is (1310720/0), route is External
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 10000000 picoseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 0
      External data:
        Originating router is 10.20.0.3 (this system)
        AS number of route is 106
        External protocol is OSPF, external metric is 3
        Administrator tag is 12 (0x0000000C)
```
OSPF: only tag 34 redisributed by EIGRP routing protocol on ABR (HQ-IR2)
```bash
BR-IR1#show ipv6 route ospf                     
IPv6 Routing Table - default - 12 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       RL - RPL, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       la - LISP alt, lr - LISP site-registrations, ld - LISP dyn-eid
       lA - LISP away, a - Application
OE2 DEAD:BEEF:CAFE::3333/128 [110/20], tag 34
     via FE80::5054:FF:FEB4:46CA, GigabitEthernet0/3
OE2 DEAD:BEEF:CAFE::4444/128 [110/20], tag 34
     via FE80::5054:FF:FEB4:46CA, GigabitEthernet0/3
OE2 DEAD:BEEF:CAFE:1101::/64 [110/20], tag 34
     via FE80::5054:FF:FEB4:46CA, GigabitEthernet0/3
OE2 DEAD:BEEF:CAFE:1102::/64 [110/20], tag 34
     via FE80::5054:FF:FEB4:46CA, GigabitEthernet0/3
OE2 DEAD:BEEF:CAFE:1999::/64 [110/20], tag 34
     via FE80::5054:FF:FEB4:46CA, GigabitEthernet0/3
O   DEAD:BEEF:CAFE:2101::/64 [110/2]
     via FE80::5054:FF:FE91:F565, GigabitEthernet0/2
     via FE80::5054:FF:FEED:3598, GigabitEthernet0/1
O   DEAD:BEEF:CAFE:2102::/64 [110/2]
     via FE80::5054:FF:FE91:F565, GigabitEthernet0/2
     via FE80::5054:FF:FEED:3598, GigabitEthernet0/1
O   DEAD:BEEF:CAFE:2999::/64 [110/2]
     via FE80::5054:FF:FE91:F565, GigabitEthernet0/2
     via FE80::5054:FF:FEED:3598, GigabitEthernet0/1
OE2 DEAD:BEEF:CAFE:3001::/64 [110/20], tag 34
     via FE80::5054:FF:FEB4:46CA, GigabitEthernet0/3
OE2 DEAD:BEEF:CAFE:3002::/64 [110/20], tag 34
     via FE80::5054:FF:FEB4:46CA, GigabitEthernet0/3
OE2 DEAD:BEEF:CAFE:FFFF::/64 [110/20], tag 34
     via FE80::5054:FF:FEB4:46CA, GigabitEthernet0/3
```

### 4. Secure WAN & VPN Tunnels

**DMVPN Status:**
- Tunnel100 is up for both IPv4 and IPv6.
- Dynamic spoke-to-spoke communication functional.

**IKEv2 IPsec Encryption:**
- AES-256 encryption with SHA-256 integrity and DH Group 14.
- RSA certificate authentication via SCEP (WIN-SRV).
- IPsec SAs active with successful encryption/decryption.

**Certificate Validation:**
- Valid certificates enrolled on all DMVPN routers.

**Connectivity Test:**
- Full reachability (IPv4 & IPv6) from spokes to HQ and between spokes.
- MTU optimized at 1400 bytes with TCP Adjust-MSS applied.

### Verification Commands

**Step 1: DMVPN Tunnel Status**

HQ-EDGE:
```bash
HQ-EDGE#show dmvpn 
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        T1 - Route Installed, T2 - Nexthop-override
        C - CTS Capable, I2 - Temporary
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel100, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 3.3.3.3              172.31.1.2    UP 00:10:20     D
     1 4.4.4.4              172.31.1.3    UP 00:10:14     D

Interface: Tunnel100, IPv6 NHRP Details 
Type:Hub, Total NBMA Peers (v4/v6): 2
    1.Peer NBMA Address: 3.3.3.3
        Tunnel IPv6 Address: DEAD:BEEF:CAFE:FFFF::2
        IPv6 Target Network: DEAD:BEEF:CAFE:FFFF::2/128
        # Ent: 1, Status: UP, UpDn Time: 00:10:20, Cache Attrib: D
    2.Peer NBMA Address: 4.4.4.4
        Tunnel IPv6 Address: DEAD:BEEF:CAFE:FFFF::3
        IPv6 Target Network: DEAD:BEEF:CAFE:FFFF::3/128
        # Ent: 1, Status: UP, UpDn Time: 00:10:14, Cache Attrib: D
```

REM1-RTR:
```bash
REM1-RTR#show dmvpn 
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        T1 - Route Installed, T2 - Nexthop-override
        C - CTS Capable, I2 - Temporary
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel100, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 1.1.1.1              172.31.1.1    UP 00:11:00     S

Interface: Tunnel100, IPv6 NHRP Details 
Type:Spoke, Total NBMA Peers (v4/v6): 1
    1.Peer NBMA Address: 1.1.1.1
        Tunnel IPv6 Address: DEAD:BEEF:CAFE:FFFF::1
        IPv6 Target Network: DEAD:BEEF:CAFE:FFFF::/64
        # Ent: 1, Status: UP, UpDn Time: 00:11:00, Cache Attrib: S
```


**Step 2: IKEv2 & IPSec Session**

HQ-EDGE IKEv2:
```bash
HQ-EDGE#show crypto ikev2 sa
 IPv4 Crypto IKEv2  SA 

Tunnel-id Local                 Remote                fvrf/ivrf            Status 
1         1.1.1.1/500           3.3.3.3/500           none/none            READY  
      Encr: AES-CBC, keysize: 256, PRF: SHA256, Hash: SHA256, DH Grp:14, Auth sign: RSA, Auth verify: RSA
      Life/Active Time: 86400/885 sec

Tunnel-id Local                 Remote                fvrf/ivrf            Status 
2         1.1.1.1/500           4.4.4.4/500           none/none            READY  
      Encr: AES-CBC, keysize: 256, PRF: SHA256, Hash: SHA256, DH Grp:14, Auth sign: RSA, Auth verify: RSA
      Life/Active Time: 86400/879 sec

 IPv6 Crypto IKEv2  SA 

```

HQ-EDGE IPSec:
```bash
HQ-EDGE# show crypto ipsec sa
interface: Tunnel100
   local ident  (1.1.1.1/255.255.255.255/47/0)
   remote ident (4.4.4.4/255.255.255.255/47/0)
   current_peer 4.4.4.4 port 500

    #pkts encaps: 365, #pkts encrypt: 365, #pkts digest: 365
    #pkts decaps: 361, #pkts decrypt: 361, #pkts verify: 361

     inbound esp sas:
      spi: 0xC6F3622
        transform: **esp-256-aes esp-sha256-hmac**
        Status: **ACTIVE(ACTIVE)**

     outbound esp sas:
      spi: 0x69CCECF8
        transform: **esp-256-aes esp-sha256-hmac**
        Status: **ACTIVE(ACTIVE)**
```

**Step 3: Certificate Chain Validation**
HQ-EDGE: 
```bash
HQ-EDGE#show crypto pki certificates 
Certificate
  Status: Available
  Certificate Serial Number (hex): 1E0000000C613E2DDF7167C77000000000000C
  Certificate Usage: General Purpose
  Issuer: 
    cn=WSMB2026-CA
    dc=wsmb2026
    dc=net
  Subject:
    Name: HQ-EDGE.wsmb2026.net 
    Serial Number: 93TGB181X85US5E98Q3S0
    hostname=HQ-EDGE.wsmb2026.net
    serialNumber=93TGB181X85US5E98Q3S0
  CRL Distribution Points: 
    http://98.76.54.10/CertEnroll/WSMB2026-CA.crl
  Validity Date: 
    start date: 16:27:05 MYT Jun 23 2026
    end   date: 16:27:05 MYT Jun 22 2028
  Associated Trustpoints: WSMB-CA 
  Storage: nvram:WSMB2026-CA#C.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 1675829B1A81C1BE4EF1C0D9FF3A6127
  Certificate Usage: Signature
  Issuer: 
    cn=WSMB2026-CA
    dc=wsmb2026
    dc=net
  Subject: 
    cn=WSMB2026-CA
    dc=wsmb2026
    dc=net
  Validity Date: 
    start date: 15:05:08 MYT Oct 1 2025
    end   date: 15:15:07 MYT Oct 1 2035
  Associated Trustpoints: WSMB-CA 
  Storage: nvram:WSMB2026-CA#6127CA.cer


```

REM1-RTR:
```bash
EM1-RTR#show crypto pki certificates 
Certificate
  Status: Available
  Certificate Serial Number (hex): 1E0000000DCD5DFEAD19091D6200000000000D
  Certificate Usage: General Purpose
  Issuer: 
    cn=WSMB2026-CA
    dc=wsmb2026
    dc=net
  Subject:
    Name: REM1-RTR.wsmb2026.net 
    Serial Number: 9CWC7KTNKQAX9SUYXF6R5
    hostname=REM1-RTR.wsmb2026.net
    serialNumber=9CWC7KTNKQAX9SUYXF6R5
  CRL Distribution Points: 
    http://98.76.54.10/CertEnroll/WSMB2026-CA.crl
  Validity Date: 
    start date: 22:46:07 MYT Jun 23 2026
    end   date: 22:46:07 MYT Jun 22 2028
  Associated Trustpoints: WSMB-CA 
  Storage: nvram:WSMB2026-CA#D.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 1675829B1A81C1BE4EF1C0D9FF3A6127
  Certificate Usage: Signature
  Issuer: 
    cn=WSMB2026-CA
    dc=wsmb2026
    dc=net
  Subject: 
    cn=WSMB2026-CA
    dc=wsmb2026
    dc=net
  Validity Date: 
    start date: 15:05:08 MYT Oct 1 2025
    end   date: 15:15:07 MYT Oct 1 2035
  Associated Trustpoints: WSMB-CA 
  Storage: nvram:WSMB2026-CA#6127CA.cer
````

**Step 4: Spoke-To-Spoke & Connectivity Test**

IPv4 Test Rechability REM1-RTR to HQ-SRV: 
```bash
REM1-RTR#traceroute 172.20.101.10 source tunnel 100
Type escape sequence to abort.
Tracing the route to 172.20.101.10
VRF info: (vrf in name/id, vrf out name/id)
  1 172.31.1.1 [AS 65000.1] 11 msec 9 msec 7 msec
  2 172.20.0.6 [AS 65000.1] 11 msec 9 msec 8 msec
  3 172.20.0.18 [AS 65000.1] 22 msec 13 msec 10 msec
  4 172.20.101.10 [AS 65000.1] 20 msec 21 msec 27 msec


REM1-RTR#ping 172.20.101.10 source tunnel 100      
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.20.101.10, timeout is 2 seconds:
Packet sent with a source address of 172.31.1.2 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 18/21/25 ms
```
IPv6 Test Rechability REM1-RTR to HQ-SRV: 
```bash
REM1-RTR#ping dead:beef:cafe:1101::10 source tunnel 100      
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to DEAD:BEEF:CAFE:1101::10, timeout is 2 seconds:
Packet sent with a source address of DEAD:BEEF:CAFE:FFFF::2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 18/21/29 ms


REM1-RTR#traceroute dead:beef:cafe:1101::10 
Type escape sequence to abort.
Tracing the route to DEAD:BEEF:CAFE:1101::10

  1 DEAD:BEEF:CAFE:FFFF::1 9 msec 9 msec 21 msec
  2  *  *  * 
  3 DEAD:BEEF:CAFE:1101::FFF2 44 msec 14 msec 13 msec
  4 DEAD:BEEF:CAFE:1101::10 21 msec 16 msec 15 msec
```

**Step 5: Dual-Stack Tunnel Verification (IPv6)**
```bash
REM1-RTR#show ipv6 interface tunnel 100
Tunnel100 is up, line protocol is up
  IPv6 is enabled
  Global unicast address(es):
    **DEAD:BEEF:CAFE:FFFF::2**, subnet is DEAD:BEEF:CAFE:FFFF::/64
  MTU is **1400** bytes
  Input features: IPv6 TCP Adjust MSS
  Output features: IPv6 TCP Adjust MSS
  Post_Encap features: IPSEC Post-encap output classification
```
## Final Summary
This project successfully demonstrates strong competency in enterprise network design, implementation, and troubleshooting. Key skills shown include:

- Complex multi-protocol routing with loop prevention
- High availability and redundancy design
- Secure VPN deployment using modern certificate-based authentication
- Dual-stack network implementation

The network is stable, secure, and follows industry best practices and ready for production environment.
