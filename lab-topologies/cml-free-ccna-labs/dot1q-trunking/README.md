# 802.1Q Trunk Fundamentals: Static Trunk and VLANs

Switches cabled but VLANs and trunking not yet configured. Implement a static 802.1Q trunk carrying VLANs 10 and 20, assign access ports, and verify same-VLAN reachability across switches.

**5 nodes** (alpine, ioll2-xe) — runs on CML-Free  ·  about 45 minutes  ·  beginner

## What you'll configure

- Create VLANs and assign correct access ports on Layer-2 switches.
- Configure a static 802.1Q trunk that explicitly forces 802.1Q tagging, hardens the native VLAN, and restricts the allowed VLAN list precisely.
- Verify trunks and transported VLANs using show interfaces trunk and show vlan brief.
- Validate same-VLAN reachability across the trunk and confirm isolation between different VLANs.
- Diagnose and fix trunk allow-list drift and native VLAN mismatches.

## Importing

In CML choose **Lab > Import** and pick `topology.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
Your company’s HQ wiring closets were refreshed. After the change review, users in one closet could no longer reach peers in the same VLAN across the hall. An audit reveals one trunk may have drifted from the intended allow-list, and native VLAN settings were standardized to a native VLAN. Your task: build a clean, static 802.1Q trunk between the two access switches, carry VLANs 10 (Users) and 20 (Services), verify forwarding path end-to-end, and then reproduce/diagnose the change-drift outage before restoring service.

### Prerequisites & Access
- Level and time: Beginner · ~45 minutes
- You will need: Cisco Modeling Labs, and room for 5 nodes (2 network devices, 3 hosts). That fits the 5-node limit on CML Free.
- Import `topology.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **SW-HQ-ACC1, SW-HQ-ACC2** — username `admin` / password `Lab@dmin1`.
- **CLIENT10-A, CLIENT10-B, CLIENT20-B** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- SW-HQ-ACC1 and SW-HQ-ACC2 are Layer-2 access switches cabled back-to-back on Ethernet0/0. This link must be a static 802.1Q trunk that carries VLANs 10 and 20. We also standardize the native VLAN to 999 and disable trunk negotiation so both sides form a static trunk deterministically, without relying on DTP.
- CLIENT10-A connects to SW-HQ-ACC1 Ethernet0/1 and belongs to VLAN 10 (10.10.10.10/24).
- CLIENT10-B connects to SW-HQ-ACC2 Ethernet0/1 and belongs to VLAN 10 (10.10.10.20/24). These two should ping across the trunk when the configuration is correct.
- CLIENT20-B connects to SW-HQ-ACC2 Ethernet0/2 and belongs to VLAN 20 (10.20.20.10/24). This host demonstrates that different VLANs remain isolated without routing.
- There is no Layer-3 gateway in this lab; inter-VLAN routing is intentionally out of scope.

### IP Addressing Plan
- VLAN 10 Users: 10.10.10.0/24
  - CLIENT10-A: 10.10.10.10/24 (eth0 on SW-HQ-ACC1 E0/1)
  - CLIENT10-B: 10.10.10.20/24 (eth0 on SW-HQ-ACC2 E0/1)
- VLAN 20 Services: 10.20.20.0/24
  - CLIENT20-B: 10.20.20.10/24 (eth0 on SW-HQ-ACC2 E0/2)
- No default gateways are required; testing is same-subnet within each VLAN.

### Tasks
1. Create VLANs on both switches
   - What: Create VLAN 10 (name Users) and VLAN 20 (name Services). Also create VLAN 999 (name NATIVE) for the native VLAN.
   - Why: Endpoints in a VLAN only forward when the switch knows the VLAN. The native VLAN reserves untagged/native traffic for non-user purposes.

2. Assign access ports
   - What: Place SW-HQ-ACC1 Ethernet0/1 in access VLAN 10. On SW-HQ-ACC2, place Ethernet0/1 in access VLAN 10 and Ethernet0/2 in access VLAN 20. Enable the access-port feature that skips the spanning-tree listening/learning delay so host links come up immediately.
   - Why: Hosts connect to access ports that send/receive untagged frames in the user’s VLAN.

3. Configure the static 802.1Q trunk
   - What: On both ends (Ethernet0/0), turn the link into a static 802.1Q trunk: explicitly select 802.1Q as the trunk’s tagging encapsulation, force trunk mode without relying on DTP negotiation, harden the native VLAN to 999, and restrict the allowed VLAN list to exactly VLANs 10 and 20.
   - Why: A static trunk with a precise allow-list creates deterministic transport and avoids accidental VLAN leakage.

4. Verify the trunk and VLAN transport
   - What: Use show interfaces trunk and show vlan brief on both switches.
   - Why: Confirm encapsulation (802.1Q), native VLAN, and that VLANs 10 and 20 are allowed and active on the trunk.

5. Validate end-host behavior
   - What: From CLIENT10-A, ping 10.10.10.20 (CLIENT10-B). Then attempt ping 10.20.20.10 (CLIENT20-B).
   - Why: Same-VLAN hosts on different switches should succeed via the trunk; cross-VLAN traffic must fail (no routing in this lab).

6. Reproduce the change-drift outage and troubleshoot
   - What: Intentionally remove VLAN 10 from the allowed list on SW-HQ-ACC2’s trunk. Observe that CLIENT10-A can no longer ping CLIENT10-B.
   - Why: This simulates the real-world drift where one side’s trunk prunes a needed VLAN.
   - Diagnose: Use show interfaces trunk and show vlan brief to see the discrepancy (allowed vs active VLANs on the trunk). Restore VLAN 10 to the allowed list and retest.
   - Optional: Temporarily mis-set the native VLAN on one side (e.g., 1 vs 999) and observe show interfaces trunk reporting the mismatch. Re-align to 999.

### Verification
Run these from the endpoints and switches:
- CLIENT10-A: `ping 10.10.10.20` → should succeed (same VLAN over the trunk).
- CLIENT10-A: `ping 10.20.20.10` → should fail (different VLAN, no router).
- On both switches: `show interfaces trunk` → Port Ethernet0/0 with encapsulation 802.1Q, native VLAN 999, allowed VLANs 10,20, and both active.
- On both switches: `show vlan brief` → Access ports appear in the intended VLANs (E0/1 in VLAN 10 on both sides; E0/2 in VLAN 20 on SW-HQ-ACC2).

### Troubleshooting
- Allowed list drift: If the VLAN 10 ping fails across the trunk, check `show interfaces trunk` for “Vlans allowed on trunk.” Restore the trunk’s allowed VLAN list so it explicitly permits both VLAN 10 and VLAN 20 again.
- Native VLAN mismatch: If show output indicates different native VLANs, align both ends so the trunk’s native VLAN is set back to the standardized native VLAN, 999.
- Wrong access VLAN: If a host can’t reach its same-VLAN peer, verify its switch port’s access VLAN with `show vlan brief` and correct that port’s access VLAN assignment so it matches the VLAN the endpoint should belong to.
- VLAN not created: If an access VLAN shows as inactive or absent, make sure that VLAN has actually been created in the local VLAN database on each switch.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Create VLANs and assign correct access ports on Layer-2 switches.
- [ ] Configure a static 802.1Q trunk that explicitly forces 802.1Q tagging, hardens the native VLAN, and restricts the allowed VLAN list precisely.
- [ ] Verify trunks and transported VLANs using show interfaces trunk and show vlan brief.
- [ ] Validate same-VLAN reachability across the trunk and confirm isolation between different VLANs.
- [ ] Diagnose and fix trunk allow-list drift and native VLAN mismatches.

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] VLAN 10 and VLAN 20 exist on both switches.
- [ ] Trunk E0/0 on both switches is up, encapsulation dot1q, native VLAN 999.
- [ ] Trunk allowed list includes VLANs 10 and 20 on both ends.
- [ ] Access ports: SW-HQ-ACC1 E0/1 in VLAN 10; SW-HQ-ACC2 E0/1 in VLAN 10; SW-HQ-ACC2 E0/2 in VLAN 20.
- [ ] CLIENT10-A (10.10.10.10) can ping CLIENT10-B (10.10.10.20).
- [ ] CLIENT10-A cannot ping CLIENT20-B (10.20.20.10) due to inter-VLAN isolation.

## If it doesn't work

- Trunk forms but does not carry a required VLAN due to an incomplete allowed list.
- Native VLAN mismatch detected; traffic may be misclassified/untagged inconsistently.
- Endpoint plugged into an access port assigned to the wrong VLAN.
- Trunk left dependent on DTP negotiation instead of forced static trunking, so it may not form deterministically on both ends.
- VLAN missing from the local VLAN database on one switch.

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Native VLAN mismatch across the trunk.
- Required VLAN (e.g., VLAN 10) omitted from the trunk allowed list on one end.
- Host port assigned to the wrong VLAN during onboarding.

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/802-1q-trunk-fundamentals-static-trunk-and-vlans
