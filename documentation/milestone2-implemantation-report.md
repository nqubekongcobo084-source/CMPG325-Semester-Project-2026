# Milestone 2 Implementation Report

**Project ID:** CMPG325-2026-103
**Client ID:** CLI-103
**Organisation:** Kagisano-Molopo Local Municipality Offices (Ganyesa)
**Student:** NGCOBO, MAPHOLOBA (AN) - 44986866
**Academic Institution:** North West University
**Submission Date:** 02 October 2026

---

## 1. Introduction

This report documents the implementation phase of the CMPG 325 project. The
client, Kagisano-Molopo Local Municipality Offices in Ganyesa, required a
network that provides secure, segmented connectivity across multiple
departments, supports the assigned networking challenge of scoped multi-VLAN
DHCP address assignment, and accommodates the CR11 change request for VoIP
handsets.

The implementation follows the design approved in Milestone 1. This report
covers the actual build in Cisco Packet Tracer, the configuration applied to
each device, verification testing, and reflection on the process.

---

## 2. Client Requirements Recap

| Requirement | How It Is Addressed |
|-------------|---------------------|
| Assigned addressing block 192.168.46.0/24 | Subdivided with VLSM into 7 subnets |
| DHCP scoped multi-VLAN address assignment | Centralised server with 5 pools + router relay |
| Design constraint: new server in 6 months | Dedicated VLAN 60 /28 subnet |
| Change request CR11: VoIP handsets | Dedicated Voice VLAN 10 + CME extensions |
| Working, testable Packet Tracer simulation | All 26 tests passed |

---

## 3. Network Architecture

### 3.1 Topology

The network follows a Core-Distribution-Access hierarchy:

- **Core Layer:** Cisco 2811 router (CoreRouter) performs router-on-a-stick
  for inter-VLAN routing, DHCP relay, and CallManager Express.
- **Distribution Layer:** Cisco 2960-24TT (DistSW) aggregates traffic from
  all access switches via 802.1Q trunks.
- **Access Layer:** Five Cisco 2960-24TT switches serve departmental areas,
  each configured with data and voice VLANs.
- **Server Farm:** Dedicated access switch (Server-ASW) hosts the DHCP server
  on VLAN 60.

### 3.2 VLAN and IP Addressing Plan

| VLAN | Name | Subnet | Gateway | Purpose |
|------|------|--------|---------|---------|
| 10 | VOICE | 192.168.46.0/26 | 192.168.46.1 | IP Phones |
| 20 | FINANCE | 192.168.46.64/27 | 192.168.46.65 | Budget & Treasury |
| 30 | CORP | 192.168.46.96/27 | 192.168.46.97 | HR & Admin |
| 40 | COMMUNITY | 192.168.46.128/27 | 192.168.46.129 | Community Services |
| 50 | MGT_DEPT | 192.168.46.160/28 | 192.168.46.161 | Municipal Manager |
| 60 | SERVER | 192.168.46.176/28 | 192.168.46.177 | Server Farm |
| 99 | MGMT_SWITCH | 192.168.46.192/28 | 192.168.46.193 | Device Management |

### 3.3 Device Inventory

| Device | Model | Role |
|--------|-------|------|
| CoreRouter | Cisco 2811 | Router-on-a-stick, DHCP relay, CME |
| DistSW | Cisco 2960-24TT | Trunk aggregation |
| Finance-ASW | Cisco 2960-24TT | VLAN 20 + Voice VLAN 10 |
| Corp-ASW | Cisco 2960-24TT | VLAN 30 + Voice VLAN 10 |
| Community-ASW | Cisco 2960-24TT | VLAN 40 + Voice VLAN 10 |
| MgrOffice-ASW | Cisco 2960-24TT | VLAN 50 + Voice VLAN 10 |
| Server-ASW | Cisco 2960-24TT | VLAN 60 + VLAN 99 |
| DHCP-Server | Server-PT | Centralised DHCP |
| PC1-PC8 | PC-PT | Departmental workstations |
| Phone1-Phone4 | IP Phone 7960 | VoIP handsets |

---

## 4. Implementation Details

### 4.1 Layer 2 Configuration

**VLAN creation on all switches:** VLANs 10, 20, 30, 40, 50, 60, and 99
were created on all six switches with appropriate names.

**Trunk ports:** All inter-switch links were configured as 802.1Q trunks
with native VLAN 99:

    interface FastEthernet0/24
     switchport mode trunk
     switchport trunk native vlan 99
     switchport trunk allowed vlan 10,20,30,40,50,60,99

**Access ports with voice VLAN:** Departmental access ports were configured
with both a data VLAN and a voice VLAN, allowing a Cisco 7960 IP phone and
a PC to share a single switch port:

    interface range FastEthernet0/1-2
     switchport mode access
     switchport access vlan 20
     switchport voice vlan 10
     spanning-tree portfast

**Management IPs:** Each switch received a management IP on VLAN 99 for
out-of-band administration (192.168.46.194 through .199).

### 4.2 Layer 3 Configuration

Router-on-a-stick was implemented on CoreRouter's FastEthernet0/0 interface.
Seven sub-interfaces were created, one per VLAN:

    interface FastEthernet0/0.20
     description Gateway - Finance Dept
     encapsulation dot1Q 20
     ip address 192.168.46.65 255.255.255.224
     ip helper-address 192.168.46.178
     no shutdown

The `encapsulation dot1Q` command tags frames with the correct VLAN as
they leave the interface. The `ip helper-address` command tells the router
where to forward DHCP broadcasts — this is the DHCP relay agent.

### 4.3 DHCP Scoped Multi-VLAN Assignment (Assigned Challenge)

The DHCP server resides in VLAN 60 at 192.168.46.178. Five scoped pools
were created:

| Pool | Gateway | Start IP | Mask | Max Users |
|------|---------|----------|------|-----------|
| VOICE_POOL | 192.168.46.1 | 192.168.46.10 | 255.255.255.192 | 50 |
| FINANCE_POOL | 192.168.46.65 | 192.168.46.70 | 255.255.255.224 | 25 |
| CORP_POOL | 192.168.46.97 | 192.168.46.102 | 255.255.255.224 | 25 |
| COMM_POOL | 192.168.46.129 | 192.168.46.134 | 255.255.255.224 | 25 |
| MGT_POOL | 192.168.46.161 | 192.168.46.166 | 255.255.255.240 | 10 |

Because DHCP uses broadcast, and broadcasts do not cross VLAN boundaries,
the router must relay DHCP requests. This is done with `ip helper-address`
on each user-VLAN sub-interface. When a PC broadcasts a DHCP DISCOVER,
the router intercepts it, converts it to a unicast packet, and forwards
it to 192.168.46.178.

### 4.4 VoIP Configuration (CR11)

Voice traffic is separated onto VLAN 10. Access ports are configured with
`switchport voice vlan 10`, so voice frames from the IP phones are tagged
with VLAN 10 while data frames from the attached PCs remain in the
department VLAN.

CallManager Express was configured on CoreRouter with four extensions:

    telephony-service
     max-ephones 4
     max-dn 4
     ip source-address 192.168.46.1 port 2000
     auto assign 1 to 4

    ephone-dn 1
     number 1001
    ephone-dn 2
     number 1002
    ephone-dn 3
     number 1003
    ephone-dn 4
     number 1004

The DHCP server's VOICE_POOL includes a TFTP Server option set to
192.168.46.1 (the CoreRouter). This tells the phones where to find the
CallManager, allowing them to register automatically.

### 4.5 Future Server Readiness (Constraint)

VLAN 60 was dedicated to the server farm with a /28 subnet
(192.168.46.176 - 192.168.46.191). The DHCP server currently occupies
192.168.46.178. The new application/file server can be added as a static
host in this same segment without any re-addressing of existing VLANs,
directly satisfying the six-month design constraint.

---

## 5. Testing and Verification

All 26 tests passed. The full details are in the testing matrix
(`testing/testing-matrix.md`). Summary:

| Test Category | Tests | Passed |
|---------------|-------|--------|
| DHCP Assignment | 5 | 5 |
| DHCP Renewal | 1 | 1 |
| Inter-VLAN Connectivity | 3 | 3 |
| DHCP Relay Verification | 2 | 2 |
| VoIP / CME (CR11) | 6 | 6 |
| Layer 2 Verification | 8 | 8 |
| Layer 3 Verification | 1 | 1 |

Key evidence includes:

- DHCP leases from all 5 pools
- Successful inter-VLAN pings
- `show ephone` output showing all 4 phones registered
- Configuration screenshots of DHCP relay on the router

---

## 6. Troubleshooting Summary

Eight significant issues were encountered and resolved. Full details are
in `testing/troubleshooting-log.md`. Highlights:

1. Router interfaces required `no shutdown` (administratively down by default)
2. Native VLAN mismatch warnings during staged trunk configuration
3. VLAN 99 SVI required the VLAN database entry, not just the SVI
4. Packet Tracer's limited support for `show running-config interface`
5. CME not available on 2911 — required 2811 router model
6. Access switch trunks initially blocked DHCP traffic
7. First-ping packet loss is expected ARP behaviour
8. Packet Tracer does not implement `show telephony-service`

---

## 7. Reflection

The implementation was successful and the network meets all client
requirements. Key learnings:

**Technical:** The router-on-a-stick design is a cost-effective way to
provide inter-VLAN routing for a small-to-medium network. The DHCP relay
mechanism is essential for centralised DHCP in a segmented network — without
`ip helper-address`, each VLAN would need its own DHCP server.

**Practical:** Cisco Packet Tracer imposes model restrictions (such as CME
being unavailable on the 2911) that require careful device selection.
Documenting each troubleshooting step proved valuable — several issues
were resolved by referring back to previous diagnostic output.

**Design:** The VLSM addressing plan from Milestone 1 required no revisions
during implementation, which validates the importance of careful design
before building.

**Future improvements:** If deploying this in production, I would add ACLs
on the router to restrict inter-VLAN traffic (for example, only management
VLANs could reach the server farm). I would also implement port security
on access switches and configure a redundant core for high availability.

---

## 8. Conclusion

Milestone 2 delivered a fully functional, tested network for the
Kagisano-Molopo Local Municipality Offices. The assigned DHCP scoped
multi-VLAN challenge was implemented successfully. The CR11 VoIP change
request is accommodated through a dedicated Voice VLAN and working
CallManager Express. The design constraint for a future server is satisfied
by a reserved VLAN 60 segment.

The implementation is ready for the final submission stage, which will
include the technical report, video demonstration, and any ACL enhancements.