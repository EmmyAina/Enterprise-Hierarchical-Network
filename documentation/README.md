# Network Implementation Documentation: Hierarchical Enterprise Design

## 1. Executive Summary
This document outlines the architecture, configuration, and verification of a high-availability enterprise network implemented in Cisco Packet Tracer. The design follows the Cisco three-tier hierarchical model (Core, Distribution, and Access) to ensure a scalable, resilient, and manageable infrastructure.

## 2. IP Addressing & VLAN Schema
The network utilizes a private `10.0.0.0/8` address space for internal LANs and a provider-assigned public IP for external connectivity.

### **VLAN Configuration**
| VLAN ID | Name | Department | Subnet | Gateway (Core-SW1) |
| :--- | :--- | :--- | :--- | :--- |
| 10 | Staff | Sales/Staff | 10.10.0.0/24 | 10.10.0.1 |
| 20 | IT | IT Department | 10.20.0.0/24 | 10.20.0.1 |
| 30 | Admin | Administration | 10.30.0.0/24 | 10.30.0.1 |
| 90 | Servers | Server Farm | 10.90.0.0/24 | 10.90.0.1 |
| 99 | Management | Network Mgmt | 10.99.0.0/24 | 10.99.0.1 |

*Note: Access ports for Staff (VLAN 10) are configured with `spanning-tree portfast` for immediate connectivity.*

### **Infrastructure Links**
* **Core Switch to Core Router:** Point-to-point `/30` links on `10.255.255.0/30` and `10.255.255.4/30`.
* **Internet Edge:** WAN interface configured with `203.0.113.1/30`.

## 3. Layer 2: Redundancy & Optimization
### **Rapid PVST+ Implementation**
To prevent Layer 2 loops while maintaining high availability, **Rapid Per-VLAN Spanning Tree Plus** was implemented across all switches.

* **Load Balancing:** * **Core-SW1:** Primary Root Bridge for VLANs 10, 30, and 90.
    * **Core-SW2:** Primary Root Bridge for VLANs 20 and 99.
* **Failover:** Each Core switch serves as the secondary root for the other's primary VLANs to ensure redundancy.
* **Trunking:** All inter-switch links are configured as **802.1Q trunks**, restricted to the necessary VLAN IDs (10, 20, 30, 90, 99).

## 4. Layer 3: Routing Strategy
### **Dynamic Routing (OSPF)**
**OSPF Process 1** was enabled to provide dynamic routing across the Core layer.
* **Area 0:** All VLAN subnets and point-to-point links are advertised into Area 0 for sub-second convergence.
* **Default Route:** A static default route `0.0.0.0 0.0.0.0` points toward the ISP and is redistributed into the OSPF domain.

### **Inter-VLAN Routing**
Routing between departments is handled by **Switched Virtual Interfaces (SVIs)** on the multilayer Core switches.
* **Gateway Redundancy:** Core-SW1 holds the first usable IP of each subnet, while Core-SW2 holds the second.

## 5. Network Services & Security
### **DHCP & Relay**
* **DHCP Server:** Core-R1 is configured with centralized DHCP pools for Staff, IT, and Admin departments.
* **DHCP Relay:** SVIs on the Core switches are configured with `ip helper-address` pointing to the router to forward client requests across Layer 3 boundaries.

### **NAT & PAT**
* **Secure Internet Access:** Port Address Translation (PAT/NAT Overload) is configured on Core-R1.
* **Access Control:** An ACL permits the `10.0.0.0/8` private range to be translated to the public outside interface.

## 6. Verification Results
The implementation was successfully validated using the following benchmarks:
1. **Connectivity:** Verified end-to-end communication from Access Layer PCs to the Internet (100.100.100.100).
2. **Translation Table:** Confirmed active NAT/PAT sessions on the Core Router.
3. **Routing Integrity:** Verified OSPF neighbor adjacencies and the presence of the Gateway of Last Resort (`O*E2`) in the routing table.
