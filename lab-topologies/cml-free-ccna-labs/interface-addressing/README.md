# CCNA L1: Interface Addressing and Verification

Routers are cabled but interfaces lack IPv4 addresses. Configure the given /24 addresses on the correct interfaces and bring them up. Verify directly-connected reachability only.

**5 nodes** (alpine, iol-xe, ioll2-xe) — runs on CML-Free  ·  about 25 minutes  ·  beginner

## What you'll configure

- Configure IPv4 addresses and masks on IOS router interfaces using ip address <addr> 255.255.255.0
- Bring interfaces administratively up using no shutdown
- Verify interface state and addressing with show ip interface brief and show running-config interface
- Validate directly-connected reachability with ping between neighbors and a host-to-gateway test

## Importing

In CML choose **Lab > Import** and pick `interface-addressing.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
You are the new network technician for a small branch site. Two routers connect over a shared Ethernet transit, and R1 also serves as the branch default gateway for a user LAN. A helpdesk ticket reports that the user PC in the branch cannot reach its gateway and the routers cannot exchange basic test pings. Your task is to implement the given IPv4 addressing on the routers and verify directly-connected connectivity only. This is day-one addressing work — no routing protocols or static routes are used yet.

### Prerequisites & Access
- Level and time: Beginner · ~25 minutes
- You will need: Cisco Modeling Labs, and room for 5 nodes (4 network devices, 1 host). That fits the 5-node limit on CML Free.
- Import `interface-addressing.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **CLIENT01** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- RTR-BRANCH-R1 is the branch router with two interfaces:
  - Ethernet0/0 connects to the shared transit segment toward RTR-EDGE-R2.
  - Ethernet0/1 connects to the Branch LAN where CLIENT01 resides (through an access switch).
- RTR-EDGE-R2 is the neighboring router on the same transit LAN as R1.
- SW-CORE-TRANSIT provides the multi-access transit segment for R1 and R2.
- SW-BRANCH-ACC1 connects R1’s LAN-facing interface to the user PC (CLIENT01).
- CLIENT01 is a pre-addressed host used to verify the gateway on the Branch LAN.

All verification in this lab is strictly limited to directly-connected reachability:
- CLIENT01 must reach its default gateway on R1.
- R1 must reach R2 over the transit.

No routing protocols and no static routes are configured or required.

### IP Addressing Plan
Use the exact given addresses and masks on the specified interfaces:
- Transit LAN 10.1.1.0/24 (mask 255.255.255.0)
  - R1 Ethernet0/0: 10.1.1.1/24
  - R2 Ethernet0/0: 10.1.1.2/24
- Branch LAN 192.168.1.0/24 (mask 255.255.255.0)
  - R1 Ethernet0/1 (Gateway): 192.168.1.1/24
  - CLIENT01: 192.168.1.10/24, default gateway 192.168.1.1 (already set)

Discipline for this series:
- Configure IPv4 using dotted-decimal masks (255.255.255.0 for /24).
- Apply the address to the correct physical interface and issue no shutdown.

### Tasks
1. On R1, implement the LAN gateway address and bring the interface up.
   - What: Assign 192.168.1.1 255.255.255.0 to Ethernet0/1 and ensure it is up.
   - Why: CLIENT01 depends on this gateway for local LAN testing.
   - Watch for: Correct interface selection (Ethernet0/1) and the exact mask.

2. On R1, implement the transit address and bring the interface up.
   - What: Assign 10.1.1.1 255.255.255.0 to Ethernet0/0 and ensure it is up.
   - Why: R1 must communicate with its directly-connected neighbor R2 for basic tests.
   - Watch for: Up/up state and the correct dotted-decimal mask.

3. On R2, implement the transit address and bring the interface up.
   - What: Assign 10.1.1.2 255.255.255.0 to Ethernet0/0 and ensure it is up.
   - Why: Enables directly-connected testing to R1 on the shared transit.
   - Watch for: Exact address, exact mask, and administrative state.

4. Verify interfaces are correctly addressed and operational.
   - What: Use show ip interface brief and show running-config interface on both routers.
   - Why: Confirms the configuration and physical/logical state.
   - Watch for: Status and protocol both show up/up on the configured ports.

5. Validate directly-connected reachability only.
   - What: From CLIENT01, ping its gateway 192.168.1.1.
   - Why: Proves the branch LAN gateway is working.
   - What: From R1, ping 10.1.1.2.
   - Why: Proves the transit link is addressing-correct and operational.
   - Watch for: Successful echo replies; any failure suggests addressing/mask/state issues.

### Verification
Perform these checks once you’ve configured the router interfaces:
- On R1 and R2: show ip interface brief — the configured interfaces display the correct IP addresses and show up/up.
- From CLIENT01: ping 192.168.1.1 — should succeed.
- From R1: ping 10.1.1.2 — should succeed.
- Optional on CLIENT01: ip neigh (ARP) should show 192.168.1.1 resolved to a MAC after a successful ping.

### Troubleshooting
- If pings fail from CLIENT01 to 192.168.1.1:
  - Ensure R1 Ethernet0/1 has 192.168.1.1 255.255.255.0 and is up/up.
  - Confirm CLIENT01 has 192.168.1.10/24 and default route via 192.168.1.1.
- If R1 cannot ping 10.1.1.2:
  - Verify both R1 Ethernet0/0 and R2 Ethernet0/0 are addressed in 10.1.1.0/24 with mask 255.255.255.0.
  - Check that both interfaces are up/up and cabled to the transit switch.
- Look for common mistakes:
  - Address applied to the wrong interface.
  - Mask entered incorrectly (e.g., 255.255.0.0 instead of 255.255.255.0).
  - Interface left administratively down.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Configure IPv4 addresses and masks on IOS router interfaces using ip address <addr> 255.255.255.0
- [ ] Bring interfaces administratively up using no shutdown
- [ ] Verify interface state and addressing with show ip interface brief and show running-config interface
- [ ] Validate directly-connected reachability with ping between neighbors and a host-to-gateway test

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] R1: Ethernet0/0 shows 10.1.1.1 and up/up in show ip interface brief
- [ ] R1: Ethernet0/1 shows 192.168.1.1 and up/up in show ip interface brief
- [ ] R2: Ethernet0/0 shows 10.1.1.2 and up/up in show ip interface brief
- [ ] CLIENT01 can ping 192.168.1.1 successfully
- [ ] R1 can ping 10.1.1.2 successfully

## If it doesn't work

- Interface remains administratively down or protocol down/down due to missing no shutdown or cabling error
- Mask mismatch on a shared LAN prevents ARP/neighbor resolution
- Host default gateway misconfigured, preventing host-to-gateway pings
- Wrong IP assigned to the wrong interface (transit vs LAN)

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Incorrect mask on one side of a shared LAN breaks ARP and pings (verify both ends show 255.255.255.0)
- Interface configured on the wrong port (check cabling vs interface names before assigning addresses)
- Host default gateway mismatch prevents LAN-to-gateway pings (verify host route and gateway IP)
- Interface left shutdown after addressing (verify admin/line protocol state)

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ccna-l1-interface-addressing-and-verification
