# 🌐 CCNA Enterprise Mega Lab: Dual-Stack Resilient Network Infrastructure
[![CCNA](https://img.shields.io/badge/Certification-CCNA%20Implementing%20and%20Administering%20Cisco%20Solutions-blue.svg)](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/associate/ccna.html)
[![Packet Tracer](https://img.shields.io/badge/Simulation-Cisco%20Packet%20Tracer-orange.svg)](#)
[![OSPF](https://img.shields.io/badge/Routing-OSPFv2%20%7C%20OSPFv3-green.svg)](#)
[![Redundancy](https://img.shields.io/badge/High%20Availability-HSRP%20%7C%20Rapid%20PVST%2B-red.svg)](#)
[![Security](https://img.shields.io/badge/Security-Dynamic%20ARP%20Inspection%20%7C%20DHCP%20Snooping%20%7C%20Port%20Security-black.svg)](#)

Welcome to the documentation for my **CCNA Enterprise Mega Lab**. This project demonstrates a production-grade, enterprise-scale network design implemented in Cisco Packet Tracer. The network is built for high availability, robust security, dynamic dual-stack routing (IPv4 & IPv6), multi-WAN failover, centralized network services, and a secure corporate wireless infrastructure.

This lab showcases the practical application of CCNA-level concepts, modeling a multi-site enterprise (Office A and Office B) connected via a Core routing layer to redundant ISPs.

---

## 🗺️ Network Topology

Below is the complete network topology showing the Core, Distribution, Access, and Wireless layers, along with the server farm and dual ISP edge connections.

![CCNA Mega Lab Topology](Mega%20Lab%20Screenshot.png)

---

## 🚀 Key Technical Skills Demonstrated

This project is a comprehensive showcase of modern enterprise network engineering principles, specifically targeting:

### 1. High Availability & Layer 2 Redundancy
*   **HSRPv2 (Hot Standby Router Protocol):** Configured Active/Active SVI load balancing across distribution switches in both offices. Active gateways pre-emptively recover control upon link restoration.
*   **Rapid PVST+ (Spanning Tree):** Custom priority tuning to align the STP Root Bridge for each VLAN with its corresponding HSRP Active router, eliminating sub-optimal pathing. BPDU Guard and PortFast are applied on all edge ports to guarantee fast convergence and eliminate loops.
*   **Link Aggregation (EtherChannels):** Bundled physical links into logical trunks. Formed Cisco PAgP (desirable-desirable) in Office A and open standard LACP (active-active) in Office B. Configured Layer-3 LACP between the Core Switches (CSW1 & CSW2).

### 2. Advanced Dynamic & Static Routing
*   **OSPFv2 & OSPFv3 (Single-Area Area 0):** Configured dynamic routing across all Core routers, Distribution switches, and the WAN router (R1). Implemented Point-to-Point network types on physical transits to bypass DR/BDR election, and secured SVIs and Loopbacks by making them passive.
*   **Multi-WAN ISP Failover:** Established dual recursive default static routes to ISPA and ISPB, utilizing a floating static backup route (Administrative Distance = 2) via ISPB. Integrated failover route advertisement using OSPF dynamic injection.

### 3. Layer 2 Security Policies (Hardening)
*   **DHCP Snooping:** Prevented rogue DHCP servers. Trusted uplink ports and untrusted access ports with a strict rate limit of 15 pps (100 pps for WLC trunking) to prevent DHCP starvation attacks.
*   **Dynamic ARP Inspection (DAI):** Mitigated ARP spoofing/Man-in-the-Middle attacks by validating ARP replies against the DHCP snooping database. Enabled validation checks for source MAC, destination MAC, and IP bindings.
*   **Port Security:** Restricted access ports to the minimum necessary MAC addresses using sticky MAC learning, securing them with `restrict` violation logging to notify administrators of unauthorized connections without disrupting valid hosts.

### 4. Dual-Stack IPv6 Transition
*   **Dual-Stack Addressing:** Enabled IPv6 routing globally and configured IPv6 addresses on R1, CSW1, and CSW2. 
*   **EUI-64 Auto-Configuration:** Used global prefix routing combined with EUI-64 to auto-generate interface IDs from MAC addresses on transit links.
*   **IPv6 Floating Static Routes:** Provided redundant IPv6 path selections to next-hop gateways.

### 5. Centralized Network Services & Edge NAT
*   **Dynamic PAT & Static NAT:** Configured static one-to-one NAT mapping for the internal server (`SRV1`) to public IP `203.0.113.113` for external access, and pool-based PAT overload (`POOL1` `/29`) to enable internal VLAN subnets to browse the Internet.
*   **Network Infrastructure Services:** Configured R1 as the central DHCP Server with Option 43 support for wireless WLC registration. Deployed Syslog logging to `SRV1` with local buffer allocations, NTP Stratum 5 server synchronization with MD5 authentication key matching, DNS server configurations on switches, and SNMPv2c read-only community strings.
*   **Secure VTY Management (SSHv2):** Disabled Telnet. Hardened VTY lines with a standard ACL allowing remote logins exclusively from the Office A PC subnet, generated maximum modulus RSA keys, and forced local username credential validation with synchronous console log formatting.

---

## 📊 IP Addressing & Subnetting Plan

The enterprise network is partitioned into distinct subnets to optimize security, control broadcast domains, and support high availability.

### LAN Subnets & Gateway Redundancy (HSRPv2)

| Location | VLAN | Description | Subnet Range | Gateway VIP | SVI / Device IPs |
| :--- | :---: | :--- | :--- | :---: | :--- |
| **Office A** | 10 | PCs Subnet | `10.1.0.0/24` | `10.1.0.1` | DSW-A1 (`.2` - Active) <br> DSW-A2 (`.3` - Standby) |
| **Office A** | 20 | IP Phones Subnet | `10.2.0.0/24` | `10.2.0.1` | DSW-A1 (`.2` - Standby) <br> DSW-A2 (`.3` - Active) |
| **Office A** | 40 | Wireless Wi-Fi Subnet | `10.6.0.0/24` | `10.6.0.1` | DSW-A1 (`.2` - Standby) <br> DSW-A2 (`.3` - Active) <br> WLC1 Interface (`.4`) |
| **Office A** | 99 | Management Subnet | `10.0.0.0/28` | `10.0.0.1` | DSW-A1 (`.2` - Active) <br> DSW-A2 (`.3` - Standby) <br> ASW-A1 (`.4`), ASW-A2 (`.5`), ASW-A3 (`.6`) <br> WLC1 SVI (`.7`) |
| **Office B** | 10 | PCs Subnet | `10.3.0.0/24` | `10.3.0.1` | DSW-B1 (`.2` - Active) <br> DSW-B2 (`.3` - Standby) |
| **Office B** | 20 | IP Phones Subnet | `10.4.0.0/24` | `10.4.0.1` | DSW-B1 (`.2` - Standby) <br> DSW-B2 (`.3` - Active) |
| **Office B** | 30 | Internal Servers | `10.5.0.0/24` | `10.5.0.1` | DSW-B1 (`.2` - Standby) <br> DSW-B2 (`.3` - Active) <br> SRV1 (`.4` - Manual) |
| **Office B** | 99 | Management Subnet | `10.0.0.16/28` | `10.0.0.17` | DSW-B1 (`.18` - Active) <br> DSW-B2 (`.19` - Standby) <br> ASW-B1 (`.20`), ASW-B2 (`.21`), ASW-B3 (`.22`) |

### Core Links & Loopback Interfaces

| Link Description | IP Subnet Range (IPv4) | Device Interface IP Allocations | IPv6 Prefix / Address |
| :--- | :--- | :--- | :--- |
| **R1 <=> ISPA** | `203.0.113.0/30` | ISPA (`.1`), R1 G0/0/0 (`.2` - DHCP) | `2001:db8:a::2/64` (R1) |
| **R1 <=> ISPB** | `203.0.113.4/30` | ISPB (`.5`), R1 G0/1/0 (`.6` - DHCP) | `2001:db8:b::2/64` (R1) |
| **R1 <=> CSW1** | `10.0.0.32/30` | R1 G0/0 (`.33`), CSW1 G1/0/1 (`.34`) | `2001:db8:a1::/64` (EUI-64) |
| **R1 <=> CSW2** | `10.0.0.36/30` | R1 G0/1 (`.37`), CSW2 G1/0/1 (`.38`) | `2001:db8:a2::/64` (EUI-64) |
| **CSW1 <=> CSW2** | `10.0.0.40/30` | CSW1 Po1 (`.41`), CSW2 Po1 (`.42`) | IPv6 Enabled (Link-Local only) |
| **CSW1 <=> DSW-A1** | `10.0.0.44/30` | CSW1 G1/1/1 (`.45`), DSW-A1 G1/1/1 (`.46`) | - |
| **CSW1 <=> DSW-A2** | `10.0.0.48/30` | CSW1 G1/1/2 (`.49`), DSW-A2 G1/1/1 (`.50`) | - |
| **CSW1 <=> DSW-B1** | `10.0.0.52/30` | CSW1 G1/1/3 (`.53`), DSW-B1 G1/1/1 (`.54`) | - |
| **CSW1 <=> DSW-B2** | `10.0.0.56/30` | CSW1 G1/1/4 (`.57`), DSW-B2 G1/1/1 (`.58`) | - |
| **CSW2 <=> DSW-A1** | `10.0.0.60/30` | CSW2 G1/1/1 (`.61`), DSW-A1 G1/1/2 (`.62`) | - |
| **CSW2 <=> DSW-A2** | `10.0.0.64/30` | CSW2 G1/1/2 (`.65`), DSW-A2 G1/1/2 (`.66`) | - |
| **CSW2 <=> DSW-B1** | `10.0.0.68/30` | CSW2 G1/1/3 (`.69`), DSW-B1 G1/1/2 (`.70`) | - |
| **CSW2 <=> DSW-B2** | `10.0.0.72/30` | CSW2 G1/1/4 (`.73`), DSW-B2 G1/1/2 (`.74`) | - |
| **R1 Loopback0** | `10.0.0.76/32` | R1 Loopback0 (`10.0.0.76`) | Host Routing Identifier |
| **CSW1 Loopback0** | `10.0.0.77/32` | CSW1 Loopback0 (`10.0.0.77`) | Host Routing Identifier |
| **CSW2 Loopback0** | `10.0.0.78/32` | CSW2 Loopback0 (`10.0.0.78`) | Host Routing Identifier |
| **DSW-A1 Loopback0** | `10.0.0.79/32` | DSW-A1 Loopback0 (`10.0.0.79`) | Host Routing Identifier |
| **DSW-A2 Loopback0** | `10.0.0.80/32` | DSW-A2 Loopback0 (`10.0.0.80`) | Host Routing Identifier |
| **DSW-B1 Loopback0** | `10.0.0.81/32` | DSW-B1 Loopback0 (`10.0.0.81`) | Host Routing Identifier |
| **DSW-B2 Loopback0** | `10.0.0.82/32` | DSW-B2 Loopback0 (`10.0.0.82`) | Host Routing Identifier |

---

## 🔌 Physical Connection Matrix

The physical cabling matrix records port-to-port connections mapping the Core, Distribution, Access, and Host layers:

```
                  +-------------------+
                  |      Edge R1      |
                  +---------+---------+
                            | (G0/0 & G0/1)
           +----------------+----------------+
           | (G1/0/1)                        | (G1/0/1)
  +--------+--------+               +--------+--------+
  |    Core CSW1    +---------------+    Core CSW2    |
  +----+---+---+----+  (Po1 Trunk)  +----+---+---+----+
       |   |   |   |                     |   |   |   |
       |   |   |   +--------+   +--------+   |   |   |
       |   |   +-------+    |   |    +-------+   |   |
       |   +------+    |    |   |    |    +------+   |
       |          |    |    |   |    |    |          |
  +----+----+  +--+----+  +-+---+  +-+----+  +-------+
  | DSW-A1  |  | DSW-A2|  |DSW-B1|  |DSW-B2|  | Server / Wireless / hosts
  +----+----+  +----+--+  +--+---+  +--+---+  +-------+
       |            |        |         |
       | (Po1 L2)   |        | (Po1 L2)|
       +------------+        +---------+
```

### Core & Layer 3 Cabling
*   **R1** `G0/0` ➡️ **CSW1** `G1/0/1`
*   **R1** `G0/1` ➡️ **CSW2** `G1/0/1`
*   **R1** `G0/0/0` ➡️ **ISPA**
*   **R1** `G0/1/0` ➡️ **ISPB`
*   **CSW1** `G1/0/2` ➡️ **CSW2** `G1/0/2` (PortChannel 1 Link 1)
*   **CSW1** `G1/0/3` ➡️ **CSW2** `G1/0/3` (PortChannel 1 Link 2)

### Core to Distribution Trunk Cabling
*   **CSW1** `G1/1/1` ➡️ **DSW-A1** `G1/1/1`
*   **CSW1** `G1/1/2` ➡️ **DSW-A2** `G1/1/1`
*   **CSW1** `G1/1/3` ➡️ **DSW-B1** `G1/1/1`
*   **CSW1** `G1/1/4` ➡️ **DSW-B2** `G1/1/1`
*   **CSW2** `G1/1/1` ➡️ **DSW-A1** `G1/1/2`
*   **CSW2** `G1/1/2` ➡️ **DSW-A2** `G1/1/2`
*   **CSW2** `G1/1/3` ➡️ **DSW-B1** `G1/1/2`
*   **CSW2** `G1/1/4` ➡️ **DSW-B2** `G1/1/2`

### Switch-to-Switch (Distribution to Access) & Port-Channels
*   **DSW-A1** `G1/0/4` ➡️ **DSW-A2** `G1/0/4` (PAgP EtherChannel 1 Link 1)
*   **DSW-A1** `G1/0/5` ➡️ **DSW-A2** `G1/0/5` (PAgP EtherChannel 1 Link 2)
*   **DSW-B1** `G1/0/4` ➡️ **DSW-B2** `G1/0/4` (LACP EtherChannel 1 Link 1)
*   **DSW-B1** `G1/0/5` ➡️ **DSW-B2** `G1/0/5` (LACP EtherChannel 1 Link 2)
*   **DSW-A1** `G1/0/1` ➡️ **ASW-A1** `G0/1` | **DSW-A2** `G1/0/1` ➡️ **ASW-A1** `G0/2`
*   **DSW-A1** `G1/0/2` ➡️ **ASW-A2** `G0/1` | **DSW-A2** `G1/0/2` ➡️ **ASW-A2** `G0/2`
*   **DSW-A1** `G1/0/3` ➡️ **ASW-A3** `G0/1` | **DSW-A2** `G1/0/3` ➡️ **ASW-A3** `G0/2`
*   **DSW-B1** `G1/0/1` ➡️ **ASW-B1** `G0/1` | **DSW-B2** `G1/0/1` ➡️ **ASW-B1** `G0/2`
*   **DSW-B1** `G1/0/2` ➡️ **ASW-B2** `G0/1` | **DSW-B2** `G1/0/2` ➡️ **ASW-B2** `G0/2`
*   **DSW-B1** `G1/0/3` ➡️ **ASW-B3** `G0/1` | **DSW-B2** `G1/0/3` ➡️ **ASW-B3** `G0/2`

### Access Switch Host Connections
*   **ASW-A1** `F0/1` ➡️ **LWAP1** (Access, VLAN 40)
*   **ASW-A1** `F0/2` ➡️ **WLC1** (Trunk, VLANs 40,99 - Untagged 99)
*   **ASW-A2** `F0/1` ➡️ **IP Phone 1** (Access VLAN 10, Voice VLAN 20) ➡️ **PC1**
*   **ASW-A3** `F0/1` ➡️ **IP Phone 2** (Access VLAN 10, Voice VLAN 20) ➡️ **PC2**
*   **ASW-B1** `F0/1` ➡️ **LWAP2** (Access, VLAN 99)
*   **ASW-B2** `F0/1` ➡️ **IP Phone 3** (Access VLAN 10, Voice VLAN 20) ➡️ **PC3**
*   **ASW-B3** `F0/1` ➡️ **SRV1** (Access, VLAN 30)

---

## ⚙️ Detailed Configuration Details & Cisco CLI Syntax

This section highlights the technical implementation steps, including actual IOS configurations applied to achieve this operational state.

### Part 1: Device Initialization & AAA Hardening
Security begins at the console. All routers and switches are configured with local user authentication, inactivity timers, and synchronized log formatting to prevent CLI interruptions.
```ios
! Hostname and local administration credentials
hostname DSW-A1
username cisco secret 9 $9$Q8bA$7c5D9...   ! Type 9 Scrypt hashing preferred
enable secret 9 $9$j1s2$jeremysitlab...

! Restricting console port access
line con 0
 login local
 exec-timeout 30 0                          ! 30-minute inactivity timeout
 logging synchronous                        ! Prevents syslog prints from interrupting typing
```

### Part 2: Layer 2 EtherChannels, VLANs & Trunking
Aggregating switchports increases inter-switch bandwidth and provides link redundancy. Port-Channels are established, dynamic trunk negotiations are explicitly disabled, and VTPv2 is used for VLAN propagation.

```ios
! In Office A - Cisco-proprietary PAgP EtherChannel (desirable-desirable)
interface range GigabitEthernet1/0/4 - 5
 channel-group 1 mode desirable
 switchport mode trunk
 switchport trunk native vlan 1000          ! Mitigation against VLAN Hopping
 switchport nonegotiate                     ! Disable DTP (Dynamic Trunking Protocol)
 switchport trunk allowed vlan 10,20,40,99  ! Prune unused VLANs for security

! In Office B - Open Standard LACP EtherChannel (active-active)
interface range GigabitEthernet1/0/4 - 5
 channel-group 1 mode active
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport nonegotiate
 switchport trunk allowed vlan 10,20,30,99

! VTP Server Configuration (applied on DSW-A1 and DSW-B1)
vtp domain JeremysITLab
vtp mode server
vtp version 2

! VTP Client Configuration (applied on ASW-A1..A3 and ASW-B1..B3)
vtp domain JeremysITLab
vtp mode client
```

For the Access-to-WLC interface on `ASW-A1`, the port is configured to support the management and wireless traffic while keeping management untagged:
```ios
interface FastEthernet0/2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 40,99
 switchport mode trunk
 switchport nonegotiate
```

### Part 3: Layer 3 IPv4 Routing, L3 EtherChannels & HSRP Redundancy
Routing is enabled at the Distribution layer. A Layer-3 EtherChannel is constructed between the core switches, and HSRPv2 provides high availability with Active/Active gateway load-balancing.

```ios
! Enabling routing globally on Layer 3 Multilayer Switches
ip routing

! Layer-3 EtherChannel between CSW1 and CSW2 (PAgP)
! CSW1 Configuration
interface range GigabitEthernet1/0/2 - 3
 no switchport
 channel-group 1 mode desirable
interface Port-channel1
 no switchport
 ip address 10.0.0.41 255.255.255.252
```

HSRPv2 is tuned to ensure gateway load balancing. `DSW-A1` is the active gateway for VLANs 10 and 99 (Management), while `DSW-A2` is active for VLANs 20 and 40:
```ios
! DSW-A1 SVI Configuration (Active for VLAN 10 & 99)
interface Vlan10
 ip address 10.1.0.2 255.255.255.0
 standby version 2
 standby 2 ip 10.1.0.1
 standby 2 priority 105                    ! Higher priority than DSW-A2 (100)
 standby 2 preempt                          ! Ensures this switch takes over immediately
!
interface Vlan20
 ip address 10.2.0.2 255.255.255.0
 standby version 2
 standby 3 ip 10.2.0.1
 standby 3 preempt                          ! Priority remains default 100 (Standby)

! DSW-A2 SVI Configuration (Active for VLAN 20 & 40)
interface Vlan10
 ip address 10.1.0.3 255.255.255.0
 standby version 2
 standby 2 ip 10.1.0.1
 standby 2 preempt                          ! Standby for VLAN 10
!
interface Vlan20
 ip address 10.2.0.3 255.255.255.0
 standby version 2
 standby 3 ip 10.2.0.1
 standby 3 priority 105                    ! Active for VLAN 20
 standby 3 preempt
```

### Part 4: Spanning-Tree Optimization (Rapid PVST+)
To prevent loop-based downtime, Spanning Tree is configured for Rapid PVST+. Spanning-tree priorities are set to align with the HSRP active path:
```ios
! DSW-A1 STP tuning
spanning-tree mode rapid-pvst
spanning-tree vlan 10,99 priority 0         ! Root Bridge for VLAN 10 and 99
spanning-tree vlan 20,40 priority 4096      ! Secondary Root for VLAN 20 and 40

! DSW-A2 STP tuning
spanning-tree mode rapid-pvst
spanning-tree vlan 20,40 priority 0         ! Root Bridge for VLAN 20 and 40
spanning-tree vlan 10,99 priority 4096      ! Secondary Root for VLAN 10 and 99

! Edge port protection (Configured on all ASW access ports)
interface FastEthernet0/1
 spanning-tree portfast                     ! Transitions port to forwarding state instantly
 spanning-tree bpduguard enable             ! Disables port if a rogue switch is plugged in
```

### Part 5: Static and Dynamic Routing (OSPFv2)
OSPFv2 runs across the enterprise core. Network types are set to Point-to-Point on physical links to speed up neighbor adjacencies, while host-facing networks are set to passive.
```ios
! Core Switch OSPF Configuration (CSW1)
router ospf 1
 router-id 10.0.0.77                        ! RID matches Loopback 0
 network 10.0.0.34 0.0.0.0 area 0           ! Exact match networking
 network 10.0.0.41 0.0.0.0 area 0
 network 10.0.0.45 0.0.0.0 area 0
 network 10.0.0.49 0.0.0.0 area 0
 network 10.0.0.53 0.0.0.0 area 0
 network 10.0.0.57 0.0.0.0 area 0
 network 10.0.0.77 0.0.0.0 area 0
 passive-interface Loopback0

! Setting physical transit links to point-to-point network type
interface GigabitEthernet1/1/1
 ip ospf network point-to-point
```

On the WAN router R1, the dynamic default route is injected to the OSPF domain, and OSPF is configured directly on interfaces:
```ios
! R1 Configuration
interface GigabitEthernet0/0
 ip ospf 1 area 0
 ip ospf network point-to-point
!
router ospf 1
 router-id 10.0.0.76
 default-information originate              ! Injects dynamic default route to CSWs/DSWs
 passive-interface Loopback0
```

Dual static recursive default routes are configured on R1 for redundant Internet paths:
```ios
ip route 0.0.0.0 0.0.0.0 203.0.113.1        ! Main default route via ISP-A
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2      ! Floating default route via ISP-B (AD=2)
```

### Part 6: Network Infrastructure Services (DHCP, NAT, NTP, SNMP & SSH)
R1 acts as a centralized DHCP server. Multilayer switches relay the boot DHCP broadcasts to R1's loopback interface.

```ios
! DHCP Pools on R1
ip dhcp excluded-address 10.1.0.1 10.1.0.10
ip dhcp pool A-PC
 network 10.1.0.0 255.255.255.0
 default-router 10.1.0.1
 dns-server 10.5.0.4
 domain-name jeremysitlab.com
!
! Management Pools include Option 43 for WLC discovery
ip dhcp pool A-Mgmt
 network 10.0.0.0 255.255.255.240
 default-router 10.0.0.1
 dns-server 10.5.0.4
 option 43 hex f1040a000007                 ! Option 43 pointing to WLC SVI (10.0.0.7)

! DHCP Relay helper configured on Distribution SVIs
interface Vlan10
 ip helper-address 10.0.0.76                ! Relays DHCP requests to R1's Loopback0
```

Network translation allows internal subnets to access public web addresses, while presenting `SRV1` to the outside world:
```ios
! Access List defining inside local ranges
access-list 2 permit 10.1.0.0 0.0.0.255
access-list 2 permit 10.2.0.0 0.0.0.255
access-list 2 permit 10.3.0.0 0.0.0.255
access-list 2 permit 10.4.0.0 0.0.0.255
access-list 2 permit 10.6.0.0 0.0.0.255

! PAT Pool definition and translation map
ip nat pool POOL1 203.0.113.200 203.0.113.207 netmask 255.255.255.248
ip nat inside source list 2 pool POOL1 overload

! Static NAT mapping for SRV1
ip nat inside source static 10.5.0.4 203.0.113.113
```

Remote administration security is hardened via SSHv2 and protected with access list filters:
```ios
! Generate cryptographic keys and enforce SSHv2
ip domain-name jeremysitlab.com
crypto key generate rsa
 2048                                       ! Strong modulus size
ip ssh version 2

! SSH access restricted to Office A PC subnet
access-list 1 permit 10.1.0.0 0.0.0.255

! Apply security rules to VTY terminal lines
line vty 0 15
 access-class 1 in                          ! Apply ACL filter
 transport input ssh                        ! Blocks insecure Telnet sessions
 login local
 logging synchronous
```

Centralized time, status, and monitoring configurations:
```ios
! NTP Configurations on R1 (NTP Server)
ntp server 216.239.35.0                     ! Sync R1 with public internet time
ntp master 5                                ! Serve local clients at stratum 5

! NTP Client authentication on Switches
ntp authenticate
ntp authentication-key 1 md5 ccna
ntp trusted-key 1
ntp server 10.0.0.76 key 1                  ! Point to R1's Loopback0 SVI

! Logging and Syslog Server Configuration
logging host 10.5.0.4                       ! Export Syslog events to SRV1
logging trap debugging                      ! Send all levels
logging buffered 8192                       ! Keep local circular logs

! SNMP Read-Only Community
snmp-server community SNMPSTRING RO
```

### Part 7: Security Policies (ACLs, DHCP Snooping & DAI)
To implement a Zero-Trust network boundary between Office A and Office B, extended ACLs are written to allow only ICMP traffic between PC subnets, blocking other protocols while allowing normal out-of-boundary transit.

```ios
! Extended ACL OfficeA_to_OfficeB
ip access-list extended OfficeA_to_OfficeB
 permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 permit ip any any

! Apply ACL on the closest SVI (Inbound interface of DSW-A1 & DSW-A2)
interface Vlan10
 ip access-group OfficeA_to_OfficeB in
```

Access layer port security restricts endpoints to authorized MACs, preventing MAC table flooding:
```ios
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1        ! Only allow 1 MAC address
 switchport port-security violation restrict! Drop unauthorized frames and increment counter
 switchport port-security mac-address sticky! Learn MAC address and write to running-config
```

DHCP Snooping and DAI are activated to eliminate DHCP rogue offers and ARP poisoning:
```ios
! DHCP Snooping globally and per VLAN
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,99
no ip dhcp snooping information option     ! Disable DHCP Option 82 insertion

! Dynamic ARP Inspection (DAI)
ip arp inspection vlan 10,20,30,99
ip arp inspection validate src-mac dst-mac ip ! Comprehensive ARP payload checks

! Configure Trust settings on Uplinks (applied on trunk interfaces to distribution switches)
interface range GigabitEthernet0/1 - 2
 ip dhcp snooping trust
 ip arp inspection trust

! Interface Rate Limiting on Untrusted access ports
interface FastEthernet0/1
 ip dhcp snooping limit rate 15             ! Limits to 15 DHCP packets per second
```

### Part 8: IPv6 Transition (Dual-Stack & EUI-64 Routing)
IPv6 configurations prepare the network core for transition, generating host IDs dynamically via EUI-64:
```ios
! Enable IPv6 unicast routing globally
ipv6 unicast-routing

! Interface IPv6 addressing
interface GigabitEthernet0/0
 ipv6 address 2001:db8:a1::/64 eui-64       ! Auto-appends interface ID using MAC address
 ipv6 enable

! IPv6 Routing table static defaults
ipv6 route ::/0 2001:db8:a::1               ! Primary route
ipv6 route ::/0 2001:db8:b::1 2             ! Floating static route (AD=2)
```

### Part 9: Enterprise Wireless (WLC & LWAP Configuration)
The Cisco Wireless LAN Controller (`WLC1`) manages the corporate SSIDs, offloading traffic processing from the lightweight APs (`LWAPs`).
*   **GUI IP Control:** Accessible at `https://10.0.0.7` using credentials `admin / adminPW12`.
*   **Dynamic Wi-Fi Interface:** Configured dynamic interface `Wi-Fi` mapped to **VLAN 40**, bound to physical port **1**. Assigned IP address `10.6.0.4` with gateway `10.6.0.1` and primary DHCP server `10.0.0.76` (R1).
*   **WLAN Configuration:** Created WLAN `Wi-Fi` (ID: 1, SSID: `Wi-Fi`) secured with **WPA2 Policy**, AES encryption, and Pre-Shared Key (PSK) `cisco123`.
*   **LWAP Control:** Assured LWAP association to `WLC1` via Option 43 broadcast redirection.

---

## 🔍 Validation & Troubleshooting Operations

Below are critical verification commands to check the operational health of the enterprise network:

### Spanning-Tree & Redundancy Verification
```bash
# Verify HSRP state on Layer 3 switches
DSW-A1# show standby brief

# Verify the root bridge for Spanning Tree
DSW-A1# show spanning-tree vlan 10

# Verify the active EtherChannel bundles
DSW-A1# show etherchannel summary
```

### Routing Verification
```bash
# Verify OSPF neighbor adjacencies are FULL
R1# show ip ospf neighbor

# Check active routing tables
R1# show ip route
R1# show ipv6 route

# Trace routing paths across the core
PC1> tracert 203.0.113.113
```

### Security Verification
```bash
# Check DHCP snooping binding database
ASW-A1# show ip dhcp snooping binding

# Verify ARP inspection database matches
ASW-A1# show ip arp inspection vlan 10

# Monitor port security statistics
ASW-A2# show port-security interface f0/1
```

### NAT/PAT Translation Verification
```bash
# View active translations on the edge router R1
R1# show ip nat translations
```

---

*This CCNA Mega Lab repository serves as a testament to my solid foundation in enterprise switching, dynamic routing protocols, security compliance, network services implementation, and high availability design.*
