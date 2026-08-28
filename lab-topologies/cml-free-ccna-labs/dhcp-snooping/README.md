# DHCP Snooping Trust Boundary

VLAN 10 access with a router gateway/DHCP server. Learner will enable DHCP Snooping and trust only the uplink.

**4 nodes** (alpine, iol-xe, ioll2-xe) — runs on CML-Free  ·  about 30 minutes  ·  beginner

## What you'll configure

- Explain the purpose of DHCP Snooping and the trust boundary on an access switch
- Enable DHCP Snooping globally and for a specific VLAN
- Trust only the uplink interface toward the legitimate DHCP server
- Verify DHCP Snooping operational status and bindings

## Importing

In CML choose **Lab > Import** and pick `dhcp-snooping.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
A small campus site is tightening security at the access edge. Users have reported intermittent loss of IP addresses after a contractor plugged in a personal Wi‑Fi router that began offering rogue DHCP leases. Your job is to enforce a clear trust boundary on the access switch so that only the legitimate DHCP server (the default gateway router) can send DHCP Offer/Ack messages. This is the first lab in the Layer 2 Security Hardening series and focuses solely on DHCP Snooping and correct trust placement.

### Prerequisites & Access
- Level and time: Beginner · ~30 minutes
- You will need: Cisco Modeling Labs, and room for 4 nodes (2 network devices, 2 hosts). That fits the 5-node limit on CML Free.
- Import `dhcp-snooping.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **PC1, PC2** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- SW1 (Layer‑2 access switch) connects upstream to R1 on Ethernet0/0. That single link is the uplink to the real DHCP server and default gateway.
- Two user hosts (PC1 and PC2) connect to SW1 on Ethernet0/1 and Ethernet0/2 respectively.
- The entire LAN is in VLAN 10 using the 10.0.0.0/24 subnet.
- R1 provides the gateway at 10.0.0.1 and runs an on‑box DHCP server for the 10.0.0.0/24 network.
- For this graded lab, R1 and the hosts are already configured. You will configure only SW1’s Layer‑2 security (DHCP Snooping) — no routing.

### IP Addressing Plan
- VLAN 10 Users: 10.0.0.0/24
  - Default Gateway (R1): 10.0.0.1/24
  - PC1: 10.0.0.10/24 (default route via 10.0.0.1)
  - PC2: 10.0.0.20/24 (default route via 10.0.0.1)

### Tasks
1. Identify the trust boundary
   - What: Determine which single interface on SW1 should be trusted for DHCP Snooping.
   - Why: Only the path facing the legitimate DHCP server must be trusted so rogue servers on access ports are blocked. In this topology, the uplink toward R1 is the only trusted interface; host‑facing ports remain untrusted by default.

2. Enable DHCP Snooping globally
   - What: Activate DHCP Snooping at the switch level.
   - Why: Global enablement is required before VLAN‑scoped enforcement can work.

3. Enable DHCP Snooping for VLAN 10
   - What: Arm DHCP Snooping for the user VLAN.
   - Why: DHCP Snooping is per‑VLAN; without binding the correct VLAN, no protection or bindings are created for that segment.

4. Trust only the uplink interface toward R1
   - What: On SW1’s Ethernet0/0 (uplink to R1), configure the port as trusted for DHCP Snooping.
   - Why: DHCP Offer/Ack must be allowed only from the real server. Access ports toward hosts should remain untrusted (default) so a rogue DHCP server on a user port is blocked.

5. Preserve existing Layer‑2 forwarding
   - What: Do not add routing, helper addresses, or unrelated features. Keep VLAN 10 and access modes as provided.
   - Why: This lab evaluates only deterministic DHCP Snooping configuration, not routing or other security features.

### Verification
Run these checks after you finish the configuration:
- From SW1:
  - show ip dhcp snooping — confirm global enable and that VLAN 10 is listed under the enabled VLANs. Confirm the uplink interface is trusted and access ports are untrusted.
  - show ip dhcp snooping binding — confirm the binding table output is available. It may be empty until a client renews/obtains a lease; the key is that the feature is enabled and ready.
- From PC1 and PC2:
  - ping 10.0.0.1 — both hosts should reach the default gateway.
  - ping between hosts (PC1 to PC2 and vice versa) — end‑to‑end L2 forwarding remains intact.

### Troubleshooting
- If show ip dhcp snooping does not list VLAN 10, you likely enabled only the global command and missed the per‑VLAN command.
- If legitimate DHCP responses appear blocked (in a dynamic environment), verify the trust is on the uplink interface facing the server, not on a host‑facing port.
- If output is unclear, confirm interface roles: Ethernet0/0 is the uplink to R1; Ethernet0/1 and Ethernet0/2 face hosts and must remain untrusted by default.
- Ensure you did not add unrelated features (routing, helper‑address) that could mask the root cause. This lab checks the deterministic config, not runtime client state.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Explain the purpose of DHCP Snooping and the trust boundary on an access switch
- [ ] Enable DHCP Snooping globally and for a specific VLAN
- [ ] Trust only the uplink interface toward the legitimate DHCP server
- [ ] Verify DHCP Snooping operational status and bindings

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] DHCP Snooping is enabled globally on SW1
- [ ] DHCP Snooping is enabled for VLAN 10 on SW1
- [ ] SW1 Ethernet0/0 (uplink toward R1) is trusted for DHCP Snooping
- [ ] Host-facing ports remain untrusted (default)
- [ ] PC1 and PC2 can ping the default gateway 10.0.0.1
- [ ] show ip dhcp snooping binding is available (may be empty until leases occur)

## If it doesn't work

- DHCP Snooping enabled globally but not for the correct VLAN
- Trust configured on the wrong interface (host-facing port) causing a security gap
- Uplink not trusted, dropping legitimate DHCP replies from the server
- Using the wrong VLAN number for snooping leading to no bindings

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Trust boundary misplaced: a host-facing port was trusted instead of the uplink
- VLAN 10 omitted from DHCP Snooping VLAN list
- Assuming DHCP Snooping is per-switch only and forgetting the per-VLAN activation

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ccna-l2-sec-1-dhcp-snooping-trust-boundary
