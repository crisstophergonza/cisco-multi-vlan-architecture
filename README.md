# Multi-VLAN Enterprise Campus Network with Dynamic DHCP Allocation

## 📌 Architectural Overview
This deployment engineers a scalable, secure, and production-ready Local Area Network (LAN) architecture for a multi-floor corporate office environment using **Cisco Packet Tracer**. 

The design addresses critical enterprise infrastructure challenges by segmenting department broadcast domains to mitigate network overhead, establishing high-performance inter-switch trunk connections, and centralizing automated IPv4 address management directly through the Cisco IOS gateway.

### Key Technical Objectives
* **Broadcast Domain Reduction:** Segregated campus subnets using standards-compliant Virtual LANs (VLANs).
* **Inter-VLAN Routing Excellence:** Engineered high-throughput routing using a single physical interface sliced into virtual sub-interfaces (Router-on-a-Stick).
* **Automated IP Lifecycle Management:** Deployed resilient, scoped DHCP pools natively on the gateway router to handle dynamic device onboarding.
* **Infrastructure Optimization:** Overcame physical hardware memory constraints by actively modifying Switch Database Manager (SDM) allocation templates.

---

## 🗺️ Logical Network Topology
Below is the verified structural topology map showing the collapsed core switching layer aggregating multi-floor traffic up to the corporate gateway router:

![Network Architecture Topology](image_37c3e7.png)

### Core IP Addressing & Subnetting Blueprint
The addressing architecture utilizes a structured private Class C allocation space divided into clean operational subnets:

| Subnet Function | Assigned VLAN | Network Address | Default Gateway | Allocated DHCP Range |
| :--- | :---: | :--- | :--- | :--- |
| **Floor 1: Reception** | VLAN 10 | `192.168.10.0/24` | `192.168.10.1` | `192.168.10.11 - .254` |
| **Floor 2: Admin & Finance** | VLAN 20 | `192.168.20.0/24` | `192.168.20.1` | `192.168.20.11 - .254` |
| **Switch Management Fabric**| VLAN 1 (Native) | `192.168.1.0/24` | N/A | Static Assignment |

---

## 🛠️ Production Configuration Highlights

### 1. Gateway Sub-Interface Routing (Router-on-a-Stick)
Rather than wasting scarce physical routing ports, the core gateway handles multi-subnet encapsulation using IEEE 802.1Q tagging on virtual sub-interfaces:

```router-cli
Core-Router# show running-config interface gigabitEthernet 0/0/0.10
interface GigabitEthernet0/0/0.10
 description --- Internal Gateway for Reception (Floor 1) ---
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
Core-Router# show running-config interface gigabitEthernet 0/0/0.20
interface GigabitEthernet0/0/0.20
 description --- Internal Gateway for Admin/Finance (Floor 2) ---
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

### To optimize operational resource efficiency, automated pools were mapped directly into the Cisco IOS microcode, leaving explicit room for statically assigned infrastructure assets (like printers or local switch management ports):

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
!
ip dhcp pool POOL_VLAN_10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

During live deployment testing, endpoints initially generated APIPA initialization failures (169.254.x.x), indicating that broadcast layer parameters were broken.

Diagnostics & Root Cause Analysis:
1- Checked Layer 2 convergence using show interfaces trunk on the core management layer switch (SW1-Management).
2- Identified that even though the ports were manually set to switchport mode trunk, they were dropping tagged packets due to a missing local VLAN database schema.
3- Encountered a syntax rejection error when modifying the VLAN database, diagnosing it as a hardware memory allocation limit imposed by the active Switch Database Manager (SDM) template.

Technical Remediation Plan
To resolve the memory constraint and open the data plane pipes, the database was prioritized for advanced LAN capabilities, forced into a hardware reload, and trunk interfaces were successfully cleared:

SW1-Management# configure terminal
SW1-Management(config)# sdm prefer lanbase-routing
SW1-Management(config)# end
SW1-Management# write memory
SW1-Management# reload

Architecture Validation & Verification
Following systemic alignment, endpoint network cards were cycled to force a fresh Discover/Offer process.

Result: Dynamic handshakes were immediately negotiated successfully. The endpoints seamlessly discarded their APIPA addresses, acquiring valid corporate leases with fully mapped subnet masks, gateways, and upstream corporate DNS details.

---
