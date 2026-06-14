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
