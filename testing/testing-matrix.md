# Milestone 2 Testing Matrix

**Project ID:** CMPG325-2026-103
**Client:** Kagisano-Molopo Local Municipality Offices (Ganyesa)
**Student:** NGCOBO, MAPHOLOBA (AN) - 44986866
**Assigned Challenge:** DHCP (scoped multi-VLAN address assignment)
**Change Request:** CR11 - VoIP handsets

---

## 1. DHCP Assignment Tests

These tests verify the assigned networking challenge — DHCP with scoped
multi-VLAN address assignment. Each VLAN has its own DHCP pool on the
centralised server. The router relays DHCP broadcasts via `ip helper-address`.

| # | Source | VLAN | Expected IP Range | Actual IP | Gateway | Pass | Evidence |
|---|--------|------|-------------------|-----------|---------|------|----------|
| 1 | PC1 (Finance) | 20 | 192.168.46.70 - .94 | 192.168.46.70 | 192.168.46.65 | PASS | dhcp-pc1.txt |
| 2 | PC3 (Corp) | 30 | 192.168.46.102 - .126 | 192.168.46.103 | 192.168.46.97 | PASS | dhcp-pc3.txt |
| 3 | PC5 (Community) | 40 | 192.168.46.134 - .158 | 192.168.46.135 | 192.168.46.129 | PASS | dhcp-pc5.txt |
| 4 | PC7 (Manager) | 50 | 192.168.46.166 - .174 | 192.168.46.167 | 192.168.46.161 | PASS | dhcp-pc7.txt |
| 5 | Phone1 (Voice) | 10 | 192.168.46.10 - .62 | 192.168.46.10 | 192.168.46.1 | PASS | phone1-registered.png |

**Interpretation:** Each PC received an address from the correct pool. If a
PC were on the wrong VLAN, it would have received an address from a
different range or failed entirely.

---

## 2. DHCP Renewal Test

Verifies the DHCP server and relay remain operational over repeated
requests (not a one-time success).

| # | Source | Action | Result | Pass | Evidence |
|---|--------|--------|--------|------|----------|
| 6 | PC1 | `ipconfig /release` then `ipconfig /renew` | New lease granted (192.168.46.72) | PASS | dhcp-renew.txt |

---

## 3. Inter-VLAN Connectivity Tests

Verifies router-on-a-stick correctly routes traffic between VLANs.

| # | Source | Destination | Address | Replies | Pass | Evidence |
|---|--------|-------------|---------|---------|------|----------|
| 7 | PC1 (VLAN 20) | Own gateway (router Fa0/0.20) | 192.168.46.65 | 4/4 | PASS | ping-gateway.txt |
| 8 | PC1 (VLAN 20) | PC3 (VLAN 30) | 192.168.46.102 | 4/4 | PASS | ping-inter-vlan.txt |
| 9 | PC1 (VLAN 20) | DHCP Server (VLAN 60) | 192.168.46.178 | 4/4 | PASS | ping-server.txt |

**Interpretation:** Traffic successfully crossed VLAN boundaries. This
proves the router sub-interfaces with `encapsulation dot1Q` are functioning.

---

## 4. DHCP Relay Verification

Verifies the `ip helper-address` command is present and correct on
user-VLAN router sub-interfaces.

| # | Command | Expected Output | Pass | Evidence |
|---|---------|-----------------|------|----------|
| 10 | `show ip interface Fa0/0.20` on CoreRouter | Helper address 192.168.46.178 | PASS | router-dhcp-relay.txt |
| 11 | `show ip interface Fa0/0.30` on CoreRouter | Helper address 192.168.46.178 | PASS | router-dhcp-relay.txt |

---

## 5. VoIP / CME Verification (CR11)

Verifies the change request — management adopting VoIP handsets — is
accommodated.

| # | Command / Action | Expected Result | Pass | Evidence |
|---|------------------|-----------------|------|----------|
| 12 | `show ephone` on CoreRouter | 4 phones REGISTERED | PASS | ephone-registered.txt |
| 13 | Phone1 GUI | Displays extension 1001 | PASS | phone1-registered.png |
| 14 | Phone2 GUI | Displays extension 1002 | PASS | phone2-registered.png |
| 15 | Phone3 GUI | Displays extension 1003 | PASS | phone3-registered.png |
| 16 | Phone4 GUI | Displays extension 1004 | PASS | phone4-registered.png |
| 17 | Call test: Phone1 → Phone2 (dial 1002) | Rings and connects | PASS | voip-call-connected.png |

---

## 6. Layer 2 Verification

Verifies VLANs, trunks, and access ports are configured as designed.

| # | Device | Command | Expected | Pass | Evidence |
|---|--------|---------|----------|------|----------|
| 18 | DistSW | `show vlan brief` | VLANs 10,20,30,40,50,60,99 active | PASS | distsw-verify.txt |
| 19 | DistSW | `show interfaces trunk` | Fa0/1-5 and G0/1 trunking, native VLAN 99 | PASS | distsw-verify.txt |
| 20 | DistSW | `show ip interface brief` | Vlan99 = 192.168.46.194 up/up | PASS | distsw-verify.txt |
| 21 | Finance-ASW | verify | Vlan99 = 192.168.46.195 up/up | PASS | finance-asw-verify.txt |
| 22 | Corp-ASW | verify | Vlan99 = 192.168.46.196 up/up | PASS | corp-asw-verify.txt |
| 23 | Community-ASW | verify | Vlan99 = 192.168.46.197 up/up | PASS | community-asw-verify.txt |
| 24 | MgrOffice-ASW | verify | Vlan99 = 192.168.46.198 up/up | PASS | mgroffice-asw-verify.txt |
| 25 | Server-ASW | verify | Vlan99 = 192.168.46.199 up/up | PASS | server-asw-verify.txt |

---

## 7. Layer 3 Verification

| # | Device | Command | Expected | Pass | Evidence |
|---|--------|---------|----------|------|----------|
| 26 | CoreRouter | `show ip interface brief` | All sub-interfaces Fa0/0.10 - Fa0/0.99 up/up | PASS | router-subinterfaces.txt |

---

## Summary

| Category | Tests | Passed | Failed |
|----------|-------|--------|--------|
| DHCP Assignment | 5 | 5 | 0 |
| DHCP Renewal | 1 | 1 | 0 |
| Inter-VLAN Connectivity | 3 | 3 | 0 |
| DHCP Relay | 2 | 2 | 0 |
| VoIP / CME (CR11) | 6 | 6 | 0 |
| Layer 2 | 8 | 8 | 0 |
| Layer 3 | 1 | 1 | 0 |
| **Total** | **26** | **26** | **0** |

**Overall Result: ALL TESTS PASSED**