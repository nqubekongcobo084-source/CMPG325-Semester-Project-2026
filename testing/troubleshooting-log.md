# Milestone 2 Troubleshooting Log

This log documents every significant issue encountered during the Milestone 2
build and how each was diagnosed and resolved. This demonstrates the process
of implementation, not just the final result.

---

## Issue 1: Red triangle on CoreRouter-to-DistSW uplink

**Symptom:** After cabling the CoreRouter to DistSW, the link showed a red
triangle on the canvas instead of green, indicating the link was down.

**Diagnosis:** Ran `show ip interface brief` on the router and saw that
GigabitEthernet0/0 had a status of "administratively down."

**Cause:** Router interfaces in Cisco IOS are administratively shut down
by default, unlike switch ports which come up automatically when cabled.

**Fix:** Entered interface configuration mode and enabled the interface:

    interface GigabitEthernet0/0
     no shutdown

**Verification:** Link triangle turned green within 10 seconds and
`show ip interface brief` showed "up/up".

---

## Issue 2: Native VLAN mismatch warnings

**Symptom:** After configuring native VLAN 99 on DistSW's trunk ports,
the CLI displayed:

    %CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on
    FastEthernet0/1 (99), with Switch FastEthernet0/24 (1).

Also seen:

    %SPANTREE-2-BLOCK_PVID_LOCAL: Blocking FastEthernet0/4 on VLAN0099.

**Diagnosis:** DistSW's trunks were using native VLAN 99, while the access
switches on the other end were still using the default native VLAN 1.

**Cause:** This is expected during staged configuration. When you configure
one side of a trunk, the other side doesn't know about it yet.

**Fix:** Completed the trunk configuration on all five access switches,
each of which set `switchport trunk native vlan 99` on FastEthernet0/24.

**Verification:** Warnings stopped appearing as each switch was configured.
`show interfaces trunk` on DistSW confirmed all trunks with native VLAN 99.

---

## Issue 3: Vlan99 SVI showed up/down

**Symptom:** Running `show ip interface brief` on all switches showed:

    Vlan99    192.168.46.19X    YES manual up    down

The interface was administratively up, but protocol was down.

**Diagnosis:** Ran `show vlan brief` and discovered that VLAN 99 was not
in the VLAN database — even though `interface vlan 99` had been configured.

**Cause:** The VLAN itself was never created. A Cisco switch cannot bring
up an SVI for a VLAN that does not exist.

**Fix:** On each switch, re-entered the VLAN creation:

    configure terminal
    vlan 99
     name MGMT_SWITCH
    exit
    exit

**Verification:** `show ip interface brief` now shows `Vlan99 ... up up`
on every switch.

---

## Issue 4: `show running-config interface` returned "Invalid input"

**Symptom:** On CoreRouter, the command:

    show running-config interface GigabitEthernet0/0.20

Returned:

    % Invalid input detected at '^' marker.

**Diagnosis:** Tested variations and found that Packet Tracer does not
support the `running-config` filter combined with an interface name.

**Cause:** Packet Tracer implements only a subset of real Cisco IOS
commands.

**Fix:** Used the alternative command:

    show ip interface GigabitEthernet0/0.20

This displays the helper address in the output:

    Helper address is 192.168.46.178

**Verification:** DHCP relay configuration confirmed.

---

## Issue 5: `telephony-service` command not available on 2911

**Symptom:** Typing `telephony-service` on the Cisco 2911 router returned:

    % Invalid input detected at '^' marker.

**Diagnosis:** Confirmed the command was correct Cisco IOS syntax. Searched
the Packet Tracer documentation.

**Cause:** Packet Tracer only supports CallManager Express (CME) on
specific router models. The 2911 does not include the telephony feature;
the 2811 does.

**Fix:** Replaced the 2911 with a Cisco 2811 router:

1. Saved the .pkt file as a backup
2. Deleted the 2911 from the canvas
3. Placed a 2811 router and renamed it `CoreRouter`
4. Reconnected the uplink cable from 2811's FastEthernet0/0 to DistSW's G0/1
5. Re-entered the router-on-a-stick configuration using `FastEthernet0/0.X`
   sub-interfaces instead of `GigabitEthernet0/0.X`
6. Verified all sub-interfaces came up: `show ip interface brief`

**Note:** No changes were needed on any other device — switch, server, and
PC configurations were unaffected by the router swap.

**Verification:** After the swap, `telephony-service` was accepted.

---

## Issue 6: `show telephony-service` returned "Invalid input"

**Symptom:** After successfully configuring CME, the command
`show telephony-service` returned an invalid input error.

**Diagnosis:** Checked available show commands.

**Cause:** Packet Tracer does not implement this specific verification command,
even on models that support CME.

**Fix:** Used alternative verification commands:

    show running-config | section telephony-service
    show ephone

**Verification:** The first command displayed the telephony-service block.
The second confirmed all 4 phones registered.

---

## Issue 7: First ping attempt showed 25% packet loss

**Symptom:** PC1's first ping to PC3 returned 3 out of 4 replies:

    Request timed out.
    Reply from 192.168.46.102: bytes=32 time<1ms TTL=127
    Reply from 192.168.46.102: bytes=32 time<1ms TTL=127
    Reply from 192.168.46.102: bytes=32 time<1ms TTL=127

    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)

**Diagnosis:** This is the standard ARP cold-cache behaviour. The first
packet was dropped because the router did not yet know PC3's MAC address.

**Cause:** Address Resolution Protocol (ARP) has to resolve IP-to-MAC
mappings before packets can be forwarded.

**Fix:** No configuration fix was needed. Running the ping again returned
4/4 replies with 0% loss.

**Verification:** Second ping attempt showed 0% packet loss.

**Learning:** This is normal networking behaviour, not a misconfiguration.
The marker should be aware that first-ping packet loss is expected.

---

## Issue 8: Access switch trunks blocked DHCP traffic from VLAN 60

**Symptom:** During early testing, PCs could not obtain IP addresses via
DHCP. Each PC fell back to a 169.254.x.x APIPA address.

**Diagnosis:** Checked the access switch trunks with `show interfaces trunk`
and noticed they allowed only VLANs 10, 20, 99 (for Finance-ASW). VLAN 60,
where the DHCP server resides, was not allowed across the trunk.

**Cause:** The initial configuration restricted trunks to only the VLANs
required for the local department plus the voice and management VLANs.
This was logically restrictive but broke DHCP relay because the server
is centralised in VLAN 60.

**Fix:** Updated trunk configurations on all access switches to allow all
VLANs:

    interface FastEthernet0/24
     switchport trunk allowed vlan 10,20,30,40,50,60,99

**Verification:** After applying, all PCs received correct DHCP leases
from the appropriate pools.

---

## Summary

| Issue | Category | Status |
|-------|----------|--------|
| 1 | Router interface down | Resolved |
| 2 | Native VLAN mismatch | Resolved |
| 3 | VLAN 99 SVI down | Resolved |
| 4 | Invalid show command | Worked around |
| 5 | Router model limitation | Resolved (swap 2911 → 2811) |
| 6 | Invalid show command | Worked around |
| 7 | ARP cold-cache packet loss | Expected behaviour |
| 8 | Restrictive trunk VLAN list | Resolved |

All issues were diagnosed and resolved. The final network is fully
functional and all 26 tests passed.