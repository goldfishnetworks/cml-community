# OSPFv2 Single-Area Fundamentals: Adjacency and Area 0 Basics

Addressing and SSH ready. Configure OSPFv2 area 0 across the 3-router chain, advertise loopbacks and LANs, verify adjacencies and routes, and troubleshoot an area mismatch.

**5 nodes** (alpine, iol-xe) — runs on CML-Free  ·  about 55 minutes  ·  beginner

## What you'll configure

- Enable OSPFv2 process 1 on all routers with a stable router-id sourced from Loopback0.
- Advertise loopbacks and all connected LAN/transit networks in area 0 using correct wildcard masks.
- Suppress OSPF hellos on all interfaces by default, then re-enable them only on the router-to-router transit links.
- Verify neighbor adjacency states, LSDB sync, and route propagation across the chain.
- Reproduce an OSPF area mismatch on a transit link, analyze symptoms, and correct the configuration.

## Importing

In CML choose **Lab > Import** and pick `ospfv2-single-area.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
You are the lone network admin for a regional distributor that just acquired a small branch. The WAN is a simple router chain: Site A (the legacy branch) connects through HQ to Site B (the new branch). To standardize operations, leadership asked you to deploy OSPFv2 single-area (area 0) across the three routers so routes auto-learn and future growth remains simple. Your goal: bring up stable OSPF adjacencies, advertise each router’s Loopback0 and the site LANs, and verify that users at Site A can reach Site B and vice versa.

During a recent change window, an engineer reported that a new adjacency never reached FULL. Your lab includes a task to intentionally reproduce an area mismatch, learn how to recognize it fast, and fix it cleanly.

### Prerequisites & Access
- Level and time: Beginner · ~55 minutes
- You will need: Cisco Modeling Labs, and room for 5 nodes (3 network devices, 2 hosts). That fits the 5-node limit on CML Free.
- Import `ospfv2-single-area.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **RTR-SITEA-EDGE, RTR-HQ-CORE, RTR-SITEB-EDGE** — username `admin` / password `cisco123`; enable password `class123`.
- **CLIENT-A, CLIENT-B** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- RTR-SITEA-EDGE (left): Legacy branch router, hosts Client A on LAN 10.10.1.0/24 (gateway 10.10.1.1). Loopback0 is 10.255.0.1/32.
- RTR-HQ-CORE (center): HQ router acting as the chain core. Loopback0 is 10.255.0.2/32.
- RTR-SITEB-EDGE (right): New branch router, hosts Client B on LAN 10.20.1.0/24 (gateway 10.20.1.1). Loopback0 is 10.255.0.3/32.
- Client A (Linux): 10.10.1.10/24, default gateway 10.10.1.1.
- Client B (Linux): 10.20.1.10/24, default gateway 10.20.1.1.

Point-to-point transits:
- R1 (E0/0) ↔ R2 (E0/0): 10.0.12.0/30
- R2 (E0/1) ↔ R3 (E0/0): 10.0.23.0/30

### Objectives Recap
- Configure OSPFv2 process 1 on all routers in area 0.
- Use each router’s Loopback0 as the router-id and advertise it in OSPF.
- Advertise LANs and transits into area 0. Make every interface OSPF-passive by default, then re-enable OSPF hellos only on the transit links.
- Verify OSPF neighbors and routes. Confirm end-to-end reachability between Client A and Client B, and to the loopbacks.
- Intentionally misconfigure one transit in the wrong area, observe symptoms, then correct to area 0.

### IP Addressing Plan
- RTR-SITEA-EDGE
  - Loopback0: 10.255.0.1/32
  - Ethernet0/0: 10.0.12.1/30
  - Ethernet0/1: 10.10.1.1/24
- RTR-HQ-CORE
  - Loopback0: 10.255.0.2/32
  - Ethernet0/0: 10.0.12.2/30
  - Ethernet0/1: 10.0.23.1/30
- RTR-SITEB-EDGE
  - Loopback0: 10.255.0.3/32
  - Ethernet0/0: 10.0.23.2/30
  - Ethernet0/1: 10.20.1.1/24
- CLIENT-A: 10.10.1.10/24 — default gateway 10.10.1.1
- CLIENT-B: 10.20.1.10/24 — default gateway 10.20.1.1

### Tasks
1. Baseline validation (no OSPF yet)
   - On each router, check that all interfaces are up/up with the expected IPs. Why: Confirm layer-3 foundation; OSPF relies on correct addressing and link health.
   - Confirm Loopback0 exists with the 10.255.0.x/32 address. Why: We’ll use it as a stable router-id and as a reachable target to prove LSDB propagation.

2. Plan your OSPF
   - Decide on process-id 1 everywhere and a single backbone area 0. Why: Single-area is simpler and a CCNA staple; area 0 is the backbone.
   - Identify transit interfaces for adjacency: R1 E0/0, R2 E0/0 and E0/1, R3 E0/0. Why: Only router-to-router links should send OSPF hellos; LANs should be advertised but remain passive.

3. Configure OSPF on RTR-SITEA-EDGE (R1)
   - Create OSPF process 1; set router-id to match Loopback0; make every interface passive for OSPF by default, then re-enable hellos on E0/0 only.
   - Add network statements for: 10.0.12.0/30 (transit), 10.10.1.0/24 (LAN), and 10.255.0.1/32 (loopback) in area 0.
   - Why: This builds a single adjacency to HQ and advertises R1’s LAN and loopback without sending hellos onto the user LAN.

4. Configure OSPF on RTR-HQ-CORE (R2)
   - Create OSPF process 1 with router-id from Loopback0; make every interface passive for OSPF by default, then re-enable hellos on E0/0 and E0/1.
   - Add network statements for: 10.0.12.0/30, 10.0.23.0/30, and 10.255.0.2/32 in area 0.
   - Why: HQ must form adjacencies on both sides and flood LSAs across the chain.

5. Configure OSPF on RTR-SITEB-EDGE (R3)
   - Create OSPF process 1; router-id from Loopback0; make every interface passive for OSPF by default, then re-enable hellos on E0/0.
   - Add network statements for: 10.0.23.0/30 (transit), 10.20.1.0/24 (LAN), and 10.255.0.3/32 in area 0.
   - Why: R3 completes the chain and contributes the Site B LAN.

6. Deliberate fault injection: area mismatch
   - On R3 only, temporarily place the 10.0.23.0/30 network under area 1 instead of area 0. Observe: neighbor with R2 will not reach FULL; you may see no neighbor entry at all or a log indicating an area mismatch.
   - Why: Learn the signature of an area mismatch—adjacency fails even when IP addressing is correct.
   - Fix it: Reconfigure the 10.0.23.0/30 network back into area 0 and confirm the adjacency becomes FULL.

7. End-to-end validation
   - Confirm both adjacencies are FULL (R1–R2, R2–R3). Check that OSPF routes for the opposite branch’s LAN and all loopbacks are installed.
   - From Client A, ping Client B and R3’s loopback. From Client B, ping Client A and R1’s loopback. Why: Prove that the IGP propagated reachability across the chain and that return paths exist.

### Verification
- From Client A:
  - Ping 10.20.1.10 (Client B): Expect replies (0% packet loss).
  - Ping 10.255.0.3 (R3 loopback): Expect replies.
  - traceroute 10.20.1.10: Expect two router hops (R1 → R2 → R3) before the destination.
- From Client B:
  - Ping 10.10.1.10 (Client A): Expect replies.
  - Ping 10.255.0.1 (R1 loopback): Expect replies.

### Troubleshooting
- If neighbors don’t form:
  - Check interface status and IP/mask on both ends of the transit; errors here block OSPF entirely.
  - Compare areas on both sides of the link (show ip ospf interface brief). Mismatched areas prevent adjacency.
  - Ensure the transit interface is not passive; passive blocks hellos.
  - Confirm MTU/timers/defaults; mismatches can stall in EXSTART/EXCHANGE. Use defaults for this lab.
- If routes don’t appear:
  - Verify loopbacks and LANs are covered by OSPF network statements.
  - Confirm neighbor state is FULL; no FULL, no LSDB sync.
- If host pings fail:
  - Verify the default gateways on the hosts and the presence of OSPF routes on each router for the remote LAN.
  - Use traceroute from the host to find the hop where traffic stops.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Enable OSPFv2 process 1 on all routers with a stable router-id sourced from Loopback0.
- [ ] Advertise loopbacks and all connected LAN/transit networks in area 0 using correct wildcard masks.
- [ ] Suppress OSPF hellos on all interfaces by default, then re-enable them only on the router-to-router transit links.
- [ ] Verify neighbor adjacency states, LSDB sync, and route propagation across the chain.
- [ ] Reproduce an OSPF area mismatch on a transit link, analyze symptoms, and correct the configuration.

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] Client A: ping -c 3 10.20.1.10 should return 0% packet loss
- [ ] Client A: ping -c 3 10.255.0.3 should return 0% packet loss
- [ ] Client A: traceroute 10.20.1.10 should show hops via 10.0.12.2 then 10.0.23.2
- [ ] Client B: ping -c 3 10.10.1.10 should return 0% packet loss
- [ ] Client B: ping -c 3 10.255.0.1 should return 0% packet loss
- [ ] Routers: show ip ospf neighbor should list R1–R2 and R2–R3 as FULL
- [ ] Routers: show ip route ospf should include 10.10.1.0/24, 10.20.1.0/24, and the remote loopbacks

## If it doesn't work

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Area mismatch on R3 transit: 10.0.23.0/30 accidentally in area 1. Symptom: no FULL adjacency R2–R3; fix by returning it to area 0.
- Transit left passive: OSPF configured but adjacency absent. Symptom: zero neighbors; fix: re-enable OSPF hellos on the transit interfaces.
- Wildcard too broad/narrow: LAN or loopback not advertised. Symptom: missing routes; fix: correct network statements and wildcard masks.

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ospfv2-single-area-fundamentals-adjacency-and-area-0-basics
