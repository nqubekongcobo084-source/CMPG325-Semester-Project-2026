# CMPG 325 Computer Networks Project — Kagisano-Molopo Local Municipality

**Project ID:** CMPG325-2026-103
**Client ID:** CLI-103
**Organisation:** Kagisano-Molopo Local Municipality Offices (Ganyesa)
**Industry:** Municipal Services
**Student:** NGCOBO, MAPHOLOBA (AN) — 44986866
**Institution:** North West University
**Academic Year:** 2026

---

## Project Overview

Design, simulate, and test a network for a municipal government office
using Cisco Packet Tracer. The network provides secure departmental
segmentation, centralised DHCP services, and VoIP support for management.

### Requirements Addressed

| Requirement | Status |
|-------------|--------|
| Address block 192.168.46.0/24 | Subdivided with VLSM |
| DHCP scoped multi-VLAN assignment | Complete |
| Design constraint: new server in 6 months | VLAN 60 reserved |
| Change request CR11: VoIP handsets | Voice VLAN 10 + CME |
| Working Packet Tracer simulation | Complete |
| Documentation and testing evidence | Complete |

---

## Network at a Glance

### VLANs

| VLAN | Name | Subnet | Gateway | Purpose |
|------|------|--------|---------|---------|
| 10 | VOICE | 192.168.46.0/26 | 192.168.46.1 | IP Phones |
| 20 | FINANCE | 192.168.46.64/27 | 192.168.46.65 | Budget & Treasury |
| 30 | CORP | 192.168.46.96/27 | 192.168.46.97 | HR & Admin |
| 40 | COMMUNITY | 192.168.46.128/27 | 192.168.46.129 | Community Services |
| 50 | MGT_DEPT | 192.168.46.160/28 | 192.168.46.161 | Municipal Manager |
| 60 | SERVER | 192.168.46.176/28 | 192.168.46.177 | Server Farm |
| 99 | MGMT_SWITCH | 192.168.46.192/28 | 192.168.46.193 | Device Management |

### Devices

- CoreRouter (Cisco 2811) — router-on-a-stick, DHCP relay, CME
- DistSW (Cisco 2960-24TT) — trunk aggregation
- Finance-ASW, Corp-ASW, Community-ASW, MgrOffice-ASW (Cisco 2960-24TT) — access layer
- Server-ASW (Cisco 2960-24TT) — server farm access
- DHCP-Server (Server-PT) — scoped multi-VLAN DHCP
- PC1-PC8 — workstations
- Phone1-Phone4 (Cisco 7960) — VoIP handsets

---

## Repository Structure
.
├── README.md
├── documentation/
│ └── milestone2-implementation-report.md
├── packet-tracer/
│ └── ganyesa_milestone2.pkt
├── configuration/
│ ├── corerouter-config.txt
│ ├── distsw-config.txt
│ ├── finance-asw-config.txt
│ ├── corp-asw-config.txt
│ ├── community-asw-config.txt
│ ├── mgroffice-asw-config.txt
│ └── server-asw-config.txt
└── testing/
├── testing-matrix.md
├── troubleshooting-log.md
└── screenshots/
└── (all verification screenshots)

---

## How to Open and Use

1. Install Cisco Packet Tracer (v8.x or 9.x)
2. Open `packet-tracer/ganyesa_milestone2.pkt`
3. Wait 30-60 seconds for all devices to initialise
4. Verify:
   - PC1 has IP 192.168.46.70 via DHCP
   - Phone1 through Phone4 display extensions 1001-1004

---

## Testing Summary

All 26 tests passed. See `testing/testing-matrix.md` for the full matrix.

| Category | Passed |
|----------|--------|
| DHCP Assignment (all VLANs) | 5/5 |
| DHCP Renewal | 1/1 |
| Inter-VLAN Connectivity | 3/3 |
| DHCP Relay Verification | 2/2 |
| VoIP / CME (CR11) | 6/6 |
| Layer 2 Verification | 8/8 |
| Layer 3 Verification | 1/1 |

---

## Key Technical Implementations

### DHCP Scoped Multi-VLAN (Assigned Challenge)

- Centralised DHCP server at 192.168.46.178 in VLAN 60
- Five scoped pools (one per user VLAN)
- Router uses `ip helper-address` to relay DHCP broadcasts across VLAN boundaries

### VoIP Support (CR11)

- Dedicated Voice VLAN 10
- Cisco 7960 phones registered via CallManager Express
- Extensions 1001-1004
- Successful call test between extensions

### Future Server Readiness (Constraint)

- VLAN 60 dedicated to server farm
- /28 subnet (192.168.46.176-191)
- Additional servers can be added without re-addressing

---

## Milestone Status

| Milestone | Date | Status |
|-----------|------|--------|
| Client Design Review | 28 Aug 2026 | Submitted |
| Client Implementation Review | 02 Oct 2026 | Submitted |
| Final Submission | 16 Oct 2026 | Pending |

---

## References

- Cisco IOS documentation
- CMPG 325 Project Handbook (2026)
- Project Brief CMPG325-2026-103