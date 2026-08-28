# Spanning Tree Fundamentals: Root Election & Port Roles

Two-switch redundant L2 loop with two parallel uplinks. Implement Rapid-PVST+ and deterministic root control; verify one SW2 uplink blocks. Hosts in VLAN 1: 10.0.1.10/24 and 10.0.1.11/24.

**4 nodes** (alpine, ioll2-xe) — runs on CML-Free  ·  about 45 minutes  ·  beginner

## What you'll configure

- Enable Rapid-PVST+ globally on Layer-2 switches
- Deterministically elect the STP root by configuring bridge priority
- Interpret RSTP port roles and states (root, designated, alternate/blocking)
- Validate loop prevention with parallel inter-switch links
- Use IOS show commands to confirm STP status per VLAN and interface

## Importing

In CML choose **Lab > Import** and pick `topology.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
You are the on-call network associate supporting a small office with two access switches joined by two parallel uplinks for redundancy. A recent closet refresh left the network using default Spanning Tree behavior, so the actual root bridge might change whenever hardware is replaced. Leadership wants a predictable, documented root election so topology changes don’t accidentally shift the root and risk suboptimal paths or bridging loops.

Your goal is to put both switches into Rapid-PVST+ mode and make SW1 the deterministic root for VLAN 1 by configuring its bridge priority. You will confirm the resulting port roles: SW1’s links should be designated and forwarding; SW2 should choose exactly one root port and place the other uplink into the alternate (blocking) role so the physical loop is safely suppressed.

### Prerequisites & Access
- Level and time: Beginner · ~45 minutes
- You will need: Cisco Modeling Labs, and room for 4 nodes (2 network devices, 2 hosts). That fits the 5-node limit on CML Free.
- Import `topology.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **PC-A, PC-B** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- SW1 and SW2 are ioll2-xe Layer-2 switches with two parallel inter-switch links (Ethernet0/0 and Ethernet0/1). This physical loop requires STP to prevent broadcast storms.
- PC-A connects to SW1 on an access port in VLAN 1.
- PC-B connects to SW2 on an access port in VLAN 1.
- There is no routing and no SVIs. This is a pure Layer-2 design for a single user VLAN.
- The inter-switch links are trunks carrying VLAN 1; hosts live in the same 10.0.1.0/24 subnet.

### IP Addressing Plan
- VLAN 1 Users: 10.0.1.0/24 (no gateway; pure L2)
  - PC-A: 10.0.1.10/24
  - PC-B: 10.0.1.11/24

### Tasks
1. Establish the Layer-2 foundation
   - Confirm both inter-switch links are configured as 802.1Q trunks so BPDUs can traverse and VLAN 1 can reach both closets. This maintains a real loop that STP must manage.
   - Ensure the host-facing ports are access ports in VLAN 1 only. Avoid enabling trunking toward the hosts.

2. Harden the edge (safely)
   - On the host-facing access ports only, enable immediate transition to forwarding for end hosts (PortFast) and enable BPDU Guard to shut the port if a rogue switch appears. Do not apply PortFast or BPDU Guard to inter-switch trunks.

3. Enable Rapid-PVST+
   - Convert both switches from legacy PVST+ defaults to Rapid-PVST+ for faster convergence. This is a global mode change.

4. Control the root election deterministically
   - Set SW1’s bridge priority to a 4096-multiple lower than the default (e.g., 24576) for VLAN 1 so SW1 is elected root even if MAC addresses or hardware models change in the future.
   - Leave SW2 at the default bridge priority (32768), ensuring SW1 wins.

5. Validate the resulting port roles and loop prevention
   - On SW1, confirm it is the root for VLAN 1. Expect both inter-switch uplinks to be designated and forwarding.
   - On SW2, identify which uplink is the root port and which becomes alternate (blocking). Exactly one inter-switch link should be blocked to break the loop while retaining a hot-standby path.

6. End-to-end functional test
   - From PC-A, ping PC-B. Expect consistent success with no storms or flapping. This validates data-plane forwarding in VLAN 1 through the STP-controlled topology.

### Verification
- From SW1: check VLAN 1’s spanning tree status and confirm “This bridge is the root.”
- From SW2: confirm exactly one root port to SW1 and the other inter-switch link in the alternate/blocking role.
- From both switches: per-interface role/state checks to ensure trunks are participating and access ports are in PortFast.
- From PC-A: ping 10.0.1.11 and observe success.

### Troubleshooting
- If SW1 is not the root, compare the configured VLAN 1 priorities on both switches. The lower bridge ID (priority + MAC) wins.
- If both inter-switch links forward on SW2, confirm both switches are running Rapid-PVST+ in the same mode and that STP is enabled on the trunk ports.
- If a host can’t pass traffic, verify its access port VLAN and that the inter-switch trunks carry VLAN 1. Also ensure PortFast/BPDU Guard were applied only to access ports.
- If pings are intermittent, inspect “show spanning-tree vlan 1” for topology changes and confirm exactly one blocked port exists on SW2 between the parallel links.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Enable Rapid-PVST+ globally on Layer-2 switches
- [ ] Deterministically elect the STP root by configuring bridge priority
- [ ] Interpret RSTP port roles and states (root, designated, alternate/blocking)
- [ ] Validate loop prevention with parallel inter-switch links
- [ ] Use IOS show commands to confirm STP status per VLAN and interface
- [ ] The intended switch becomes the root bridge.
- [ ] Port roles and states match expected spanning-tree logic.
- [ ] End hosts maintain connectivity on the forwarding topology.

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] Rapid-PVST+ mode enabled on SW1 and SW2
- [ ] SW1 is root for VLAN 1 ('This bridge is the root')
- [ ] SW2 has one root port toward SW1 and the other inter-switch link alternate/blocking
- [ ] PC-A (10.0.1.10) can consistently ping PC-B (10.0.1.11)

## If it doesn't work

- Wrong switch elected as root due to a lower bridge priority
- Parallel inter-switch links both forwarding due to mode mismatch or disabled STP
- Host-facing ports misconfigured as trunks or missing PortFast/BPDU Guard
- Access VLAN mismatch between host-facing ports across closets

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- An unintended switch elected as root due to lower priority elsewhere; verify and adjust bridge priority.
- Trunk interface left as access; BPDUs and VLAN transport limited; verify switchport mode and VLAN tagging.
- Host access port misassigned to the wrong VLAN; verify interface access VLAN membership.
- Edge hardening misapplied to trunks (PortFast/BPDU Guard); verify features only on host-facing ports.

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/spanning-tree-fundamentals-root-election-port-roles
