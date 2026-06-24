# Cisco-Based-Network-Architecture
> A secure multi-site enterprise network featuring advanced multi-protocol routing, high-availability switching, and Dual-Stack DMVPN over IKEv2 IPsec.

## Introduction
> This repository showcases a production-ready, multi-site enterprise network simulated using Cisco modeling tools. The project demonstrates advanced engineering skills in bridging core headquarters, regional branches, and remote offices over a simulated public internet transit backbone.

## Project Objective & Site Breakdown
The infrastructure has been divided into 3 segments:
* **Headquarters:** Centralizes corporate core services, EIGRP routing, dual-gateway HSRP, and acts as the DMVPN Hub.
* **Branch:** Simulates a remote office executing OSPFv2/OSPFv3 routing with manual DR/BDR election, area summarization.
* **ISP & Remote Sites:** Simulates the public internet core running eBGP routing, SCEP certificate authority, and dynamic spoke VPN nodes.

## Project Objective
To build a reliable enterprise network solution that ensures constant uptime, seamless connection between multi-site branches, and strong network security.

**1. Centralized Authentication & Device Access:** Secures network devices using RADIUS AAA console login, local user backups, and PBKDF2 password hashing.

**2. High-Availability & Path Redundancy:** Eliminates single points of failure by implementing an default route failover mechanism across BGP edges, combined with HSRP dual gateway switching.

**3. Multi-Protocol Routing Optimization:** Resolves complex multi-vendor routing by setting up mutual route redistribution using cryptographic tags to prevent data loops between EIGRP, OSPFv2, OSPFv3 boundaries.

**4. Secure WAN & VPN Tunnels:** Connects sites safely using certificate-authenticated Dual-Stack DMVPN and high-grade IKEv2 IPsec encryption.

## Technical Architecture & Implementation Workflow

<img width="872" height="385" alt="image" src="https://github.com/user-attachments/assets/b7d628f5-c49b-42f1-93c9-05bbca850df8" />

### HEADQUARTERS 
> Acts as the corporate network center hosting core computing services, primary routing policies, and internal local distribution.
* **High-Availability Switching:** Deploys VTP for automated VLAN management, provisions LACP EtherChannel with load balancing, and Rapid Spanning Tree for failover.
* **Advanced EIGRP:** Deploys EIGRP Named Mode optimized for secure neighbor adjacencies, efficient route summarization, and automated default route injection.
* **First Hop Gateway Redundancy:** Configures HSRP services active standby to ensure VLAN can handle into balance local endpoint traffic.
* **Edge Access Security:** Restricts unauthorized physical port access by using switchport Port-Security and ErrDisable recovery timers.

### BRANCH
> Simulates a distributed regional hub executing independent routing domains while maintaining strict connectivity with Headquarters.
* **Dual-Stack Interior Routing:** Implements OSPFv2 and OSPFv3 across a single backbone area with manual DR/BDR election tuning on local distribution blocks.
* **Infrastructure Management:** Activates secure SNMPv3 monitoring directed to a centralized performance monitoring server.
* **Cisco Proprietary Switching:** Sets up automated trunk configurations running Cisco proprietary PAgP EtherChannel with load balancing rules.
* **Localized Edge Services:** Deploys local dynamic NAT translation pools and IP Helper DHCP assignment scopes to handle address distribution for clients.

### ISP & WAN 
> Simulates global internet transit providers, public core DNS services, and distributed remote branch endpoints.
* **External Core Border Routing:** Establishes eBGP peer sessions between all edge devices and using different Autonomous Systems to distribute public route maps.
* **Dynamic Multipoint GRE over IPsec:** Constructs a Dual-Stack DMVPN using RSA digital certificates obtained over Simple Certificate Enrollment Protocol (SCEP).
* **IKEv2 IPsec Tunnel Security:** Configured secure IKEv2 IPsec tunnels with certificate authentication, using strong encryption such as AES-256, SHA-256, and DH Group 14 to protect data traffic between sites.

# Network Infrastructure Verification & Audit Report

This section provides comprehensive verification procedures, live system outputs, and behavioral analysis to prove the absolute operational integrity of the simulated enterprise architecture. Each validation directly aligns with the defined project objectives.

---

## 1. Centralized Authentication & Device Access

### Overview
Network security begins with securing access to the control plane of the infrastructure itself. This module validates that network administrators log into corporate devices using a centralized security database, preventing unauthorized configuration changes while ensuring a secure failback mechanism.

### How It Works
When an engineer connects to a core router or switch console, the device does not check its local memory. Instead, it queries a centralized RADIUS server hosted on `HQ-SRV` (172.20.101.10) using secure communication keys. If the RADIUS server is online, it validates the user and assigns their specific privilege level. If the server goes completely offline, the system automatically switches to a high-security local database as a backup.

### Verification Commands

#### *Step 1: Verify AAA authentication configuration and active server pools on HQ-EDGE*
```text
HQ-EDGE#show aaa servers 

RADIUS: id 1, priority 1, host 172.20.101.10, auth-port 1812, acct-port 1813
     State: current UP, duration 180s, previous duration 0s
     Dead: total time 0s, count 0
     Quarantined: No
     Authen: request 1, timeouts 0, failover 0, retransmission 0
             Response: accept 0, reject 0, challenge 0
             Response: unexpected 0, server error 0, incorrect 0, time 2362545ms
             Transaction: success 1, failure 0
             Throttled: transaction 0, timeout 0, failure 0
     Author: request 0, timeouts 0, failover 0, retransmission 0
             Response: accept 0, reject 0, challenge 0
             Response: unexpected 0, server error 0, incorrect 0, time 0ms
             Transaction: success 0, failure 0
             Throttled: transaction 0, timeout 0, failure 0
     Account: request 0, timeouts 0, failover 0, retransmission 0
             Request: start 0, interim 0, stop 0
             Response: start 0, interim 0, stop 0
             Response: unexpected 0, server error 0, incorrect 0, time 0ms
             Transaction: success 0, failure 0
             Throttled: transaction 0, timeout 0, failure 0
     Elapsed time since counters last cleared: 3m
     Estimated Outstanding Access Transactions: 0
     Estimated Outstanding Accounting Transactions: 0
     Estimated Throttled Access Transactions: 0
     Estimated Throttled Accounting Transactions: 0
     Maximum Throttled Transactions: access 0, accounting 0
     Requests per minute past 24 hours:
             high - 0 hours, 3 minutes ago: 1
             low  - 0 hours, 0 minutes ago: 0
             average: 0
```
#### *Step 2: Test live console access mapping to RADIUS authenticated session*
```text
HQ-EDGE# show users
    Line       User       Host(s)              Idle       Location
*  0 con 0    radiusr1    idle                 00:00:00   172.20.101.10 (RADIUS)

  Interface User Privilege: Level 15 Granted via RADIUS attribute.
```
# Step 3: Verify execution of secure PBKDF2 (Type 8/9) hashing on the local backup database
HQ-EDGE# show running-config | include username
username admin privilege 15 secret 9 $9$vE6m$M2R1S...
