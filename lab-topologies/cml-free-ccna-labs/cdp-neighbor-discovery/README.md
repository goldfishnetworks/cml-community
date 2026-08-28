# CDP Neighbor Discovery and Edge Suppression

CDP is globally OFF on R1. Your goal is to enable CDP on R1 to learn directly-connected router neighbors while keeping CDP disabled on the untrusted management-facing interface toward SRV-MGMT01.

**5 nodes** (alpine, iol-xe, ioll2-xe) — runs on CML-Free  ·  about 35 minutes  ·  beginner

## What you'll configure

- Explain why CDP only discovers directly connected Cisco devices
- Enable CDP globally and verify neighbor formation on point-to-point links
- Disable CDP advertisements on untrusted host-facing interfaces
- Use show cdp neighbors and show cdp neighbors detail to inventory adjacencies
- Differentiate discovery on the management LAN vs. router-to-router links

## Importing

In CML choose **Lab > Import** and pick `cdp-neighbor-discovery.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
Your network operations team is building a deterministic discovery and documentation workflow for the access and WAN edge. CDP helps operations verify cabling, detect model/OS mismatches, and cross-check the inventory database. However, CDP frames should not be exposed to untrusted endpoints on user or server edges.

A recent change template accidentally disabled CDP on R1, preventing neighbor mapping. Your task is to re-enable CDP globally so R1 learns about its directly connected Cisco neighbors (R2 and R3) over point-to-point links, while keeping CDP suppressed on the host-facing management interface toward the MGMT server.

### Prerequisites & Access
- Level and time: Beginner · ~35 minutes
- You will need: Cisco Modeling Labs, and room for 5 nodes (4 network devices, 1 host). That fits the 5-node limit on CML Free.
- Import `cdp-neighbor-discovery.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **SRV-MGMT01** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- RTR-SITEA-R1, RTR-SITEA-R2, and RTR-SITEA-R3 are Cisco IOS routers.
- SW-SITEA-ACC1 (ioll2-xe) bridges a single flat management LAN 10.0.0.0/24.
- SRV-MGMT01 (Alpine) at 10.0.0.100 sits on the same management LAN; it represents central services (NMS, syslog, DNS) but you will not configure those in this lab.
- For accurate CDP neighbor discovery, routers are also directly connected point-to-point: R1<->R2 and R1<->R3 (and R2<->R3). CDP is link-local and does not traverse the switch, so each router only discovers other routers it is directly cabled to.
- Each router also connects to the management LAN via SW1 with these addresses: R1=10.0.0.1, R2=10.0.0.2, R3=10.0.0.3.

### IP Addressing Plan
- Management LAN: 10.0.0.0/24
  - R1 Ethernet0/1: 10.0.0.1/24
  - R2 Ethernet0/2: 10.0.0.2/24
  - R3 Ethernet0/1: 10.0.0.3/24
  - SRV-MGMT01 eth0: 10.0.0.100/24 (default route to 10.0.0.1)
- CDP-only direct links (no IP addresses assigned):
  - R1 Ethernet0/0 <-> R2 Ethernet0/0
  - R1 Ethernet0/2 <-> R3 Ethernet0/2
  - R2 Ethernet0/1 <-> R3 Ethernet0/0

### Tasks
1. Revisit CDP fundamentals
   - What: Recognize that CDP discovers only directly connected Cisco neighbors and does not traverse switches. Routers on a shared LAN will see the switch, not other routers, as neighbors.
   - Why: Prevents false assumptions that a device can discover non-adjacent nodes across a Layer-2 domain.

2. Enable CDP globally on R1
   - What: Turn on the global CDP process on R1.
   - Why: CDP is currently disabled globally on R1; without it, no neighbor data is learned even on properly cabled links.

3. Suppress CDP on the untrusted host-facing edge
   - What: On R1, disable CDP advertisements and listening on the management-facing access port that connects toward the MGMT host via the access switch.
   - Why: Best practice is to avoid broadcasting platform and interface details to untrusted endpoints.

4. Keep CDP active on the trusted router-to-router links
   - What: Ensure the router-to-router interfaces remain up and capable of sending/receiving CDP.
   - Why: This allows building an accurate topology map of direct adjacencies.

5. Verify discovery outcomes on R1
   - What: Use show commands to confirm R1 learns R2 and R3 as neighbors over the point-to-point links and does not advertise toward the management edge.
   - Why: Confirms you met the security and visibility intent simultaneously.

### Verification
Run these checks after completing the tasks:
- From R1:
  - show cdp neighbors → Expect R1 to list R2 (on the R1-R2 link) and R3 (on the R1-R3 link). The management-facing port should show no neighbor.
  - show cdp neighbors detail → Expect platform, software version, and port IDs for R2 and R3. No host device should appear.
  - show cdp interface Ethernet0/1 → Expect CDP disabled on the management-facing interface.
- From SRV-MGMT01:
  - ping 10.0.0.1, 10.0.0.2, 10.0.0.3 → Should succeed on the same subnet (no routing is configured in this lab).

### Troubleshooting
- If no neighbors appear on R1, confirm:
  - CDP is enabled globally (show cdp).
  - The router-to-router interfaces are up/up (show ip interface brief) and physically cabled as shown.
  - You are checking the correct interfaces (CDP is per-interface and link-local).
- If you see SW1 but not R2/R3 on the management LAN, remember: a switch terminates CDP; routers do not discover each other across a switch.
- If the MGMT host appears as a neighbor: verify that CDP is disabled on R1’s management-facing interface.
- If management pings fail: verify SRV-MGMT01 has 10.0.0.100/24 on eth0 and that the routers’ management interfaces are up/up on 10.0.0.0/24.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Explain why CDP only discovers directly connected Cisco devices
- [ ] Enable CDP globally and verify neighbor formation on point-to-point links
- [ ] Disable CDP advertisements on untrusted host-facing interfaces
- [ ] Use show cdp neighbors and show cdp neighbors detail to inventory adjacencies
- [ ] Differentiate discovery on the management LAN vs. router-to-router links

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] R1 has CDP globally enabled (show cdp shows a non-zero timer/holdtime).
- [ ] R1 show cdp neighbors lists exactly R2 and R3 on the point-to-point links.
- [ ] R1 management-facing interface shows CDP disabled (show cdp interface Ethernet0/1).
- [ ] R1 show cdp neighbors detail returns platform and port IDs for R2 and R3.
- [ ] SRV-MGMT01 can ping 10.0.0.1/2/3 on the shared management LAN.

## If it doesn't work

- No neighbors appear when CDP is disabled globally
- An interface shows no neighbors if it is administratively down or cabled incorrectly
- Seeing the switch as a neighbor on the management LAN but not other routers is expected (CDP is link-local)
- Host-facing ports should not advertise CDP; verify per-interface disablement

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Global discovery process disabled: If no devices appear anywhere, verify the global CDP state before chasing interface issues.
- Local interface-level suppression: A single link shows no neighbor while others do — check per-interface CDP enablement.
- Switch in the middle: Seeing only a switch on a shared LAN is expected; do not expect router neighbors across the switch.
- Physical or admin-down state: Confirm the target interface is up/up and cabled to the intended peer.

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ndm-03-ccna-cdp-neighbor-discovery-and-edge-suppression
