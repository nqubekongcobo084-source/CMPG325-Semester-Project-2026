# CMPG325-Semester-Project-2026
CMPG325-2026-103
# Kagisano-Molopo Local Municipality Facility Network Design (Ganyesa)

## Project ID: CMPG325-2026-103
### Individual Semester Portfolio of Evidence
**Department of Computer Science and Information Systems**  
**North-West University (NWU)**  
**Milestone Name:** Milestone 1: Client Design Review  
**Submission Date:** 28 August 2026  

---

##  Student Information
* **Student Name:** Ngcobo, Mapholoba (Mapholoba AN)
* **Student Number:** 44986866
* **Project ID:** CMPG325-2026-103
* **Client ID:** CLI-103
* **Assigned Client:** Kagisano-Molopo Local Municipality Offices (Ganyesa)
* **Industry:** Municipal Services

---

##  Table of Contents
1. [Executive Summary & Client Background](#-executive-summary--client-background)
2. [Design Constraints & Change Requests Analysis](#-design-constraints--change-requests-analysis)
3. [Physical & Logical Network Architecture](#-physical--logical-network-architecture)
4. [Variable Length Subnet Masking (VLSM) IP Addressing Plan](#-variable-length-subnet-masking-vlsm-ip-addressing-plan)
5. [DHCP Scoped Multi-VLAN Address Assignment](#-dhcp-scoped-multi-vlan-address-assignment)
6. [Packet Tracer Implementation Build Sheet](#-packet-tracer-implementation-build-sheet)
7. [GitHub Repository Directory Layout](#-github-repository-directory-layout)
8. [Project Implementation Roadmap](#-project-implementation-roadmap)


---

##  Executive Summary & Client Background
The **Kagisano-Molopo Local Municipality Offices**, situated in **Ganyesa**, function as the administrative nucleus for local government service delivery. In the municipal services industry, network infrastructure must satisfy rigorous standards for service availability, security, and departmental isolation to protect civic data and guarantee service continuity [2, 30].

The primary goal of this design is to transition the Ganyesa administrative campus from a flat-network topology to a highly secure, scalable, and resilient **hierarchical core-distribution-access architecture** [2, 7, 30, 35]. This design logical partitions the campus into distinct virtual local area networks (VLANs), isolating municipal activities, avoiding broadcast storms, and establishing strong security boundaries [3, 9, 31, 40]:

*   **Budget & Treasury (Finance Office - VLAN 20):** Manages municipal billing, revenue collection, payroll, and financial reports. This department demands the highest cryptographic and logical security boundaries to protect public funds [3, 31].
*   **Corporate Services (HR & Administration - VLAN 30):** Manages human resource records, internal communications, legal documents, and council resolutions. This segment requires high-throughput data processing [3, 31].
*   **Community & Technical Services (VLAN 40):** Coordinates local infrastructure maintenance, water, waste, and social services. Stable connectivity is required to process public applications [4, 32].
*   **Office of the Municipal Manager (Executive Office - VLAN 50):** Houses executive directors and the Municipal Manager. This segment handles critical strategic planning and requires low-latency, prioritized access [4, 32].
*   **Server Farm (VLAN 60):** Houses centralized core network resources (DHCP Server) and acts as the secure landing zone for upcoming server deployments [5, 12, 33, 44].
*   **Out-of-Band Switch Management (VLAN 99):** Provides a separate logical network dedicated entirely to switch and router administration via secure Virtual Terminal lines (VTY/SSH) [11, 20, 39, 42, 50].

---

##  Design Constraints & Change Requests Analysis
Enterprise municipal networks must be defensively engineered to seamlessly adapt to emerging business directives. This architecture integrates two key parameters specified in the client brief:

### 1. Stated Design Constraint: New Server Provisioning
*   **Stated Constraint:** A new application or file server is planned within six months [5, 33].
*   **Engineering Adaptation:** Rather than integrating servers into existing workstation subnets, we establish a dedicated, secure **Server Farm segment (VLAN 60)** [5, 33]. This segment is provisioned with a dedicated `/28` subnet providing 14 usable IP addresses [5, 33]. When the new server is deployed, it can be assigned a static IP within VLAN 60 without disrupting other subnets or requiring re-addressing [5, 33]. Access Control Lists (ACLs) will be positioned on the core router sub-interface to restrict inter-VLAN lateral traffic to this segment [5, 33].

### 2. Stated Change Request: VoIP Handset Integration (CR11)
*   **Change Request (CR11):** Management adopts VoIP handsets — voice traffic must be accommodated on the existing design [6, 34].
*   **Engineering Adaptation:** We introduce a dedicated **Voice VLAN (VLAN 10)** across the entire campus [6, 34]. VoIP traffic is highly sensitive to packet loss, jitter, and delay [6, 34]. Placing the IP handsets on a dedicated logical network allows us to implement Quality of Service (QoS) on switches to prioritize CoS 5 (Voice) and DSCP EF (Expedited Forwarding) traffic [6, 34]. To minimize physical costs, we employ **Cisco 7960 IP Phones** which feature a built-in three-port switch [6, 34]. This allows a workstation and an IP phone to share a single physical switch port, reducing copper cabling costs and switch port consumption [6, 34].

---

##  Physical & Logical Network Architecture

### Hierarchical Core-Distribution-Access Model
The Ganyesa Municipal campus is structured around a high-availability **Hierarchical Star Topology** based on the Core-Distribution-Access model [7, 35]:

1.  **Core Layer:** Consists of a single **Cisco 2911 ISR Router** positioned at the root of the hierarchy [7, 35, 37]. It manages inter-VLAN routing using Router-on-a-Stick via `802.1Q` sub-interfaces (G0/0.10 through G0/0.99), hosts DHCP relay agents for all user VLANs, and provides external WAN connectivity [7, 35, 37].
2.  **Distribution Layer:** A single **Cisco Catalyst 2960-24TT Switch** aggregates downstream traffic [8, 36, 38]. It receives the trunk from the core router and re-distributes it as trunk links to the access switches and the server farm segment [38]. This switch performs no routing, simplifying traffic concentration [38].
3.  **Access Layer:** Consists of four departmental **Cisco Catalyst 2960-24TT switches** deployed in individual blocks (Finance, Corporate, Community, Executive) [8, 36, 38]. They run `802.1Q` VLAN tagging and assign end-user devices to their respective broadcast domains [8, 36, 38]. Switchports are configured with a data access VLAN for the PC and a separate voice VLAN for the daisy-chained IP phone [38].
4.  **Server Farm Segment:** A dedicated VLAN 60 segment hanging off the distribution switch on its own trunk [39]. It houses the centralized DHCP server and provides static addressing for upcoming file/application servers [39].
5.  **Out-of-Band Management (VLAN 99):** Dedicates a separate VLAN across all switches, providing management IPs reachable independently of user data traffic for SSH administration [39].

---

##  Variable Length Subnet Masking (VLSM) IP Addressing Plan
The Ganyesa campus has been assigned the classless address block **192.168.46.0/24** (256 total IP addresses) [9, 42]. To utilize this space with maximum efficiency and zero overlap, **Variable Length Subnet Masking (VLSM)** is applied [9, 42]. Subnets are allocated sequentially starting from the largest host requirement down to the smallest [9, 42].

### VLSM Allocation Table

| VLAN ID | VLAN Name | Department / Purpose | Required Hosts | Subnet Mask (CIDR) | Network Address | Usable IP Range | Broadcast Address | Default Gateway |
| :---: | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | `VOICE_VLAN` | VoIP Handsets (CR11 CR) | 50 | 255.255.255.192 (/26) | 192.168.46.0 | 192.168.46.1 – 192.168.46.62 | 192.168.46.63 | 192.168.46.1 |
| **VLAN 20** | `FINANCE_VLAN` | Budget & Treasury (Finance) | 25 | 255.255.255.224 (/27) | 192.168.46.64 | 192.168.46.65 – 192.168.46.94 | 192.168.46.95 | 192.168.46.65 |
| **VLAN 30** | `CORP_VLAN` | Corporate Services (HR/Admin)| 25 | 255.255.255.224 (/27) | 192.168.46.96 | 192.168.46.97 – 192.168.46.126 | 192.168.46.127 | 192.168.46.97 |
| **VLAN 40** | `COMM_VLAN` | Community & Tech Services | 25 | 255.255.255.224 (/27) | 192.168.46.128 | 192.168.46.129 – 192.168.46.158 | 192.168.46.159 | 192.168.46.129 |
| **VLAN 50** | `MGT_DEPT_VLAN`| Municipal Manager Office | 10 | 255.255.255.240 (/28) | 192.168.46.160 | 192.168.46.161 – 192.168.46.174 | 192.168.46.175 | 192.168.46.161 |
| **VLAN 60** | `SERVER_VLAN` | Server Farm (DHCP / File) | 5 | 255.255.255.240 (/28) | 192.168.46.176 | 192.168.46.177 – 192.168.46.190 | 192.168.46.191 | 192.168.46.177 |
| **VLAN 99** | `MGMT_SWITCH_VLAN`| Out-of-Band Switch Mgmt | 10 | 255.255.255.240 (/28) | 192.168.46.192 | 192.168.46.193 – 192.168.46.206 | 192.168.46.207 | 192.168.46.193 |
| **WAN** | `SPARE_WAN` | WAN Campus Routing Link | 2 | 255.255.255.252 (/30) | 192.168.46.208 | 192.168.46.209 – 192.168.46.210 | 192.168.46.211 | 192.168.46.209 |

###  Unassigned IP Buffer (Reserve)
*   **Subnet Range:** `192.168.46.212` to `192.168.46.255` (representing 44 total IP addresses) [11, 43].
*   **CIDR Equivalence:** `192.168.46.224/28` and `192.168.46.240/28` (or one `/27` and one `/29`).
*   **Purpose:** Serves as a contiguous **17.1% addressing reserve** to support future department subnets as Kagisano-Molopo Municipality expands [11, 43].

---

## DHCP Scoped Multi-VLAN Address Assignment
In a secure segmented network, VLANs cannot communicate or pass broadcast packets (such as DHCP DISCOVER) directly to other subnets [12, 43]. To enable centralized IP lease management, we implement a secure **DHCP Relay architecture** [12, 43]:

1.  **Centralized DHCP Server:** Positioned statically in the Server Farm (VLAN 60) with IP **192.168.46.178/28** [12, 44]. Five distinct scope pools are created: `VOICE_POOL`, `FINANCE_POOL`, `CORP_POOL`, `COMM_POOL`, and `MGT_POOL` [12, 44]. Each pool excludes the first 5 IP addresses in its subnet for static appliances, printers, and default gateways [12, 44].
2.  **DHCP Relay Agent Configuration:** The Cisco 2911 Router physical interface (G0/0) is carved into logical sub-interfaces corresponding to each VLAN [13, 45]. To bridge the broadcast boundaries, we configure the `ip helper-address` command on each sub-interface [13, 45]. This converts local DHCP broadcasts into unicast packets and forwards them directly to the DHCP Server at **192.168.46.178** [13, 45].

### Cisco IOS Configuration Blueprint

```ios
! --- CONFIGURE GATEWAY SUB-INTERFACES & DHCP HELPER ---
interface GigabitEthernet0/0.10
 description Ganyesa Voice Department Gateway
 encapsulation dot1Q 10
 ip address 192.168.46.1 255.255.255.192
 ip helper-address 192.168.46.178
exit

interface GigabitEthernet0/0.20
 description Ganyesa Finance Department Gateway
 encapsulation dot1Q 20
 ip address 192.168.46.65 255.255.255.224
 ip helper-address 192.168.46.178
exit

interface GigabitEthernet0/0.30
 description Ganyesa Corporate Department Gateway
 encapsulation dot1Q 30
 ip address 192.168.46.97 255.255.255.224
 ip helper-address 192.168.46.178
exit
```

---

## Packet Tracer Implementation Build Sheet
To implement this topology inside **Cisco Packet Tracer (Milestone 2)**, use the following hardware inventory, cabling matrices, and SVI build tables:

### 8.1 Device Inventory
| Device Name | Packet Tracer Model | Role / Description |
| :--- | :--- | :--- |
| **CoreRouter** | Cisco 2911 ISR | Inter-VLAN routing (Router-on-a-Stick), DHCP relay, WAN uplink [18, 48] |
| **DistSW** | Catalyst 2960-24TT | Trunk aggregation and traffic concentration [18, 48] |
| **Finance-ASW** | Catalyst 2960-24TT | Access switch: VLAN 20 (Data) & VLAN 10 (Voice) [18, 48] |
| **Corp-ASW** | Catalyst 2960-24TT | Access switch: VLAN 30 (Data) & VLAN 10 (Voice) [18, 48] |
| **Community-ASW**| Catalyst 2960-24TT | Access switch: VLAN 40 (Data) & VLAN 10 (Voice) [18, 48] |
| **MgrOffice-ASW**| Catalyst 2960-24TT | Access switch: VLAN 50 (Data) & VLAN 10 (Voice) [18, 48] |
| **Server-ASW** | Catalyst 2960-24TT | Access switch: VLAN 60 (Server Farm) & VLAN 99 (Mgmt) [18, 48] |
| **DHCP-Server** | Server-PT | Central DHCP Server (Static IP) [18, 48] |
| **PC1 – PC8** | PC-PT | End-user workstations (2 per department block) [18, 48] |
| **Phone1 – Phone4**| IP Phone (7960) | One Cisco 7960 per block, daisy-chained to a workstation [18, 48] |
| **ISP-Router** | Cisco 1841 / Cloud | WAN link edge router [18, 48] |

### 8.2 Physical Connections and Cabling Matrix
| From Device | Port | To Device | Port | Cable Type | Logical Link Type |
| :--- | :---: | :--- | :---: | :--- | :--- |
| **CoreRouter** | `G0/0` | **DistSW** | `G0/1` | Copper Straight-Through | Trunk (All VLANs) [19, 49] |
| **CoreRouter** | `G0/1` | **ISP-Router** | `G0/0` | Copper Straight (or Serial) | WAN Link (VLAN WAN) [19, 49] |
| **DistSW** | `Fa0/1` | **Finance-ASW** | `Fa0/24` | Copper Straight-Through | Trunk (VLAN 10, 20) [19, 49] |
| **DistSW** | `Fa0/2` | **Corp-ASW** | `Fa0/24` | Copper Straight-Through | Trunk (VLAN 10, 30) [19, 49] |
| **DistSW** | `Fa0/3` | **Community-ASW**| `Fa0/24` | Copper Straight-Through | Trunk (VLAN 10, 40) [19, 49] |
| **DistSW** | `Fa0/4` | **MgrOffice-ASW**| `Fa0/24` | Copper Straight-Through | Trunk (VLAN 10, 50) [19, 49] |
| **DistSW** | `Fa0/5` | **Server-ASW** | `Fa0/24` | Copper Straight-Through | Trunk (VLAN 60, 99) [19, 49] |
| **Finance-ASW** | `Fa0/1` | **Phone1 (Switch)**| `1` | Copper Straight-Through | Access Link (VLAN 10 & 20) [19, 49] |
| **Phone1** | `PC Port` | **PC1** | `Fa0` | Copper Straight-Through | Workstation Daisy-Chain [19, 49] |
| **Server-ASW** | `Fa0/1` | **DHCP-Server** | `FastEthernet`| Copper Straight-Through | Access Link (VLAN 60) [19, 49] |

### 8.3 Device Interface IP Addressing Build Sheet
| Device Name | Interface / Sub-Interface | IP Address | Subnet Mask (CIDR) | Logical Role |
| :--- | :--- | :--- | :--- | :--- |
| **CoreRouter** | `G0/0.10` | 192.168.46.1 | 255.255.255.192 (/26) | VLAN 10 (Voice) Default Gateway [20, 50] |
| **CoreRouter** | `G0/0.20` | 192.168.46.65 | 255.255.255.224 (/27) | VLAN 20 (Finance) Default Gateway [20, 50] |
| **CoreRouter** | `G0/0.30` | 192.168.46.97 | 255.255.255.224 (/27) | VLAN 30 (Corporate) Default Gateway [20, 50] |
| **CoreRouter** | `G0/0.40` | 192.168.46.129 | 255.255.255.224 (/27) | VLAN 40 (Community) Default Gateway [20, 50] |
| **CoreRouter** | `G0/0.50` | 192.168.46.161 | 255.255.255.240 (/28) | VLAN 50 (Executive) Default Gateway [20, 50] |
| **CoreRouter** | `G0/0.60` | 192.168.46.177 | 255.255.255.240 (/28) | VLAN 60 (Server Farm) Default Gateway [20, 50] |
| **CoreRouter** | `G0/0.99` | 192.168.46.193 | 255.255.255.240 (/28) | VLAN 99 (Management) Default Gateway [20, 50] |
| **CoreRouter** | `G0/1` | 192.168.46.209 | 255.255.255.252 (/30) | WAN External Interface [20, 50] |
| **DistSW** | `VLAN 99 SVI` | 192.168.46.194 | 255.255.255.240 (/28) | Distribution Switch Management IP [20, 50] |
| **Finance-ASW** | `VLAN 99 SVI` | 192.168.46.195 | 255.255.255.240 (/28) | Finance Switch Management IP [20, 50] |
| **Corp-ASW** | `VLAN 99 SVI` | 192.168.46.196 | 255.255.255.240 (/28) | Corporate Switch Management IP [20, 50] |
| **Community-ASW**| `VLAN 99 SVI` | 192.168.46.197 | 255.255.255.240 (/28) | Community Switch Management IP [20, 50] |
| **MgrOffice-ASW**| `VLAN 99 SVI` | 192.168.46.198 | 255.255.255.240 (/28) | Executive Switch Management IP [20, 50] |
| **Server-ASW** | `VLAN 99 SVI` | 192.168.46.199 | 255.255.255.240 (/28) | Server Farm Switch Management IP [20, 50] |
| **DHCP-Server** | `Static NIC` | 192.168.46.178 | 255.255.255.240 (/28) | Dynamic DHCP Server static host [20, 50] |

### 8.4 Cisco IOS Port-Tagging Guidelines
To support the daisy-chained Cisco IP Phones alongside administrative PCs, the access port must carry both VLANs [21, 51]. Apply the following configurations on access switch ports [21, 51]:

```ios
interface Range FastEthernet0/1 - 2
 description Workstation & IP Phone Dual-Port Link
 switchport mode access
 switchport access vlan 20     ! Tags PC traffic into Finance
 switchport voice vlan 10      ! Tags Phone traffic into Voice
 spanning-tree portfast        ! Speeds up port convergence
 spanning-tree bpduguard enable ! Disables port if loop detected
```

Trunk ports (e.g., uplink ports) must be restricted to only the active VLANs traversing that link to isolate broadcast domains [21, 51]:

```ios
interface FastEthernet0/24
 description IEEE 802.1Q Uplink Trunk to DistSW
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
```

---

##  Repository Directory Layout
To satisfy the **GitHub Portfolio of Evidence** (15% weighting) requirements, this repository maintains a clean, navigable structure [15, 16, 46, 47]:

```directory
CMPG325-2026-103-Ganyesa/
│
├── README.md                          # Main landing page & documentation summary
│
├──  documentation/                  # Milestone deliverables & reports
│   ├── client-design-review-v2.pdf    # Fully integrated Milestone 1 Report
│   └── network-specifications.docx    # Word adaptation of specifications
│
├──  packet-tracer/                  # Cisco Packet Tracer designs
│   ├── milestone_1_draft.pkt          # Initial workspace draft
│   └── ganyesa_final_topology.pkt     # Completed Milestone 2 Packet Tracer file
│
├──  configuration/                  # Plaintext Cisco IOS running configurations
│   ├── Ganyesa_Core_Router.cfg
│   ├── Ganyesa_Dist_Switch.cfg
│   └── Ganyesa_Finance_ASW.cfg
│
└──  testing/                        # Connection verification & validation screenshots
    ├── ping_matrices/                 # End-to-end ICMP verification logs
    └── dhcp_lease_confirmations/      # Dynamic IP lease validation screenshots
```

---

## Project Implementation Roadmap
This network design has been structured to strictly align with the critical milestones specified by the North-West University Department of Computer Science [16, 17, 47]:

*   ** 14 August 2026:** Project Commencement & Requirements Allocation.
*   ** 28 August 2026 (Milestone 1):** Client Design Review (15% Grade Weight) — *Complete logical/physical design, VLSM Table, Repository Layout, and Git structures [16, 17, 47].*
*   ** 02 October 2026 (Milestone 2):** Client Implementation Review (35% Grade Weight) — *Working Packet Tracer simulation, centralized DHCP validation, SVI management interfaces, and ping logs [17, 47].*
*   ** 16 October 2026 (Final Submission):** Fully operational simulated campus network, comprehensive Technical Report, Video Demonstration with inset webcam, and public GitHub portfolio submission [17, 47].


