# CCNA Static Routing: Bidirectional End-to-End Connectivity

Baseline hub-and-spoke with two user LANs. Interfaces and addressing are configured. Static routes are intentionally NOT configured so learners can add them to enable bidirectional end-to-end connectivity.

**5 nodes** (alpine, iol-xe) — runs on CML-Free  ·  about 45 minutes  ·  beginner

## What you'll configure

- Configure default routes on stub routers toward a hub to reach all non-local networks
- Configure specific static routes on a hub router for remote LANs
- Select correct next-hop IPs and masks for static routes
- Verify routing tables and end-to-end reachability using client-originated tests
- Troubleshoot first-hop and return-path failures using interface status and traceroute

## Importing

In CML choose **Lab > Import** and pick `static-routing.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
You are the new network administrator for a small enterprise with two branch user LANs (Site A and Site B) that connect through a central hub router. Static routing is the chosen design: each branch is a stub that should point its default route to the hub, while the hub holds specific static routes back to each branch LAN. A recent WAN refresh replaced the routers and IPs were re-addressed, but static routes were not put back. Users at Site A cannot reach Site B and vice versa.

Your job: implement the correct static routing to restore full, bidirectional connectivity between the two user LANs, validate from the clients, and be prepared to troubleshoot first-hop and return-path issues.

### Prerequisites & Access
- Level and time: Beginner · ~45 minutes
- You will need: Cisco Modeling Labs, and room for 5 nodes (3 network devices, 2 hosts). That fits the 5-node limit on CML Free.
- Import `static-routing.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **RTR-A-EDGE, RTR-HUB, RTR-B-EDGE** — username `admin` / password `CCNAlab!23`; enable password `CCNAlab!23`.
- **CLIENT-A, CLIENT-B** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- RTR-A-EDGE (Site A) connects to the hub over a /30 WAN transit and provides the default gateway for the Site A user LAN (10.10.10.0/24).
- RTR-B-EDGE (Site B) connects to the hub over a separate /30 WAN transit and provides the default gateway for the Site B user LAN (10.20.20.0/24).
- RTR-HUB sits in the middle and has two point-to-point links: one to Site A and one to Site B. It does not run a dynamic routing protocol in this lab; it must be configured with specific static routes back to each branch LAN.
- CLIENT-A is a host on the Site A user LAN with its default gateway set to the Site A router.
- CLIENT-B is a host on the Site B user LAN with its default gateway set to the Site B router.

All router-to-router links are unique /30 point-to-point networks. LAN addressing uses /24 networks with the router interface as the .1 gateway and the host as .10.

### IP Addressing Plan
- Site A LAN 10.10.10.0/24
  - RTR-A-EDGE Ethernet0/1: 10.10.10.1/24 (gateway)
  - CLIENT-A: 10.10.10.10/24, default gateway 10.10.10.1
- Site B LAN 10.20.20.0/24
  - RTR-B-EDGE Ethernet0/1: 10.20.20.1/24 (gateway)
  - CLIENT-B: 10.20.20.10/24, default gateway 10.20.20.1
- Transit A–HUB 10.0.12.0/30
  - RTR-A-EDGE Ethernet0/0: 10.0.12.1/30
  - RTR-HUB Ethernet0/0: 10.0.12.2/30
- Transit HUB–B 10.0.23.0/30
  - RTR-HUB Ethernet0/1: 10.0.23.1/30
  - RTR-B-EDGE Ethernet0/0: 10.0.23.2/30

### Tasks
1. Stub default routes (Site A)
- What: On RTR-A-EDGE, install a single default route that points toward the hub router’s IP on the A–HUB transit.
- Why: A stub router can reach all non-local networks through one exit toward the hub; this also simplifies the routing table and operations.
- Watch for: The next-hop must be the neighbor’s IP on the shared transit network, not the local interface.

2. Stub default routes (Site B)
- What: On RTR-B-EDGE, install a single default route pointing toward the hub’s IP on the HUB–B transit.
- Why: This provides a first-hop for all off-subnet traffic leaving Site B toward the rest of the enterprise.
- Watch for: Correct mask for a default route and the precise next-hop address on the /30.

3. Specific static routes (Hub)
- What: On RTR-HUB, configure two specific static routes: one for the Site A user LAN and one for the Site B user LAN, each pointing to the respective branch router as next-hop.
- Why: The hub is the distribution point and must know how to return traffic to each branch LAN for true bidirectional connectivity.
- Watch for: Correct destination prefixes and next-hop addresses on their transit links.

4. Routing hygiene checks
- What: On all routers, confirm routed interfaces are up/up and IP addressing matches the plan.
- Why: A single down or misaddressed interface can make static routing appear broken.
- Watch for: Interfaces accidentally left administratively down, /30 addresses that don’t match the peer’s subnet, or typos in LAN gateways.

5. Management-plane completeness (already present)
- What: Each router has SSH management configured with local authentication and VTY access-class filtering. Verify that it remains intact after your changes.
- Why: Ensures enterprise hygiene while performing changes and supports secure remote access.

### Verification
Run these checks from the endpoints so you validate real user experience:
- From CLIENT-A (10.10.10.10):
  - Ping 10.20.20.10. Expect replies.
  - Traceroute to 10.20.20.10. Expect hops via 10.10.10.1 (RTR-A-EDGE), 10.0.12.2 (RTR-HUB), 10.0.23.2 (RTR-B-EDGE), then the destination.
- From CLIENT-B (10.20.20.10):
  - Ping 10.10.10.10. Expect replies.
  - Traceroute to 10.10.10.10. Expect the reverse path via 10.20.20.1, 10.0.23.1, 10.0.12.1, then the destination.
- On each router: confirm the correct static routes are installed and that no required interface is down.

### Troubleshooting
- Interface state: Use show ip interface brief. Any required interface showing administratively down or down/down must be brought up. Verify the configured IP matches its connected /30 or /24.
- Next-hop accuracy: Ensure each static route’s next-hop is the neighbor’s IP on the shared link, never the router’s own address or an off-link IP.
- Directionality/return path: If pings succeed in one direction but fail in the other, the hub is usually missing a specific route back to the source LAN.
- Traceroute isolation: Run traceroute from the failing host to see where the path stops. If it stops at the local router, check the stub’s default route. If it stops at the hub, check the hub’s static back to that LAN. If it reaches the far router but not the host, re-check the far LAN gateway and host default gateway.
- Overlapping subnets: Confirm each point-to-point link uses its own /30 and that LAN /24s are not reused elsewhere.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Configure default routes on stub routers toward a hub to reach all non-local networks
- [ ] Configure specific static routes on a hub router for remote LANs
- [ ] Select correct next-hop IPs and masks for static routes
- [ ] Verify routing tables and end-to-end reachability using client-originated tests
- [ ] Troubleshoot first-hop and return-path failures using interface status and traceroute
- [ ] Static routes appear with expected administrative distance.
- [ ] Traffic prefers the primary path before failure.
- [ ] Backup routes activate when the primary path is lost.

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] CLIENT-A can ping 10.20.20.10 with 0% packet loss
- [ ] CLIENT-B can ping 10.10.10.10 with 0% packet loss
- [ ] Traceroute from CLIENT-A to 10.20.20.10 shows hops via 10.10.10.1 -> 10.0.12.2 -> 10.0.23.2
- [ ] RTR-A-EDGE routing table contains a single 0.0.0.0/0 static via 10.0.12.2
- [ ] RTR-B-EDGE routing table contains a single 0.0.0.0/0 static via 10.0.23.1
- [ ] RTR-HUB routing table has 10.10.10.0/24 via 10.0.12.1 and 10.20.20.0/24 via 10.0.23.2
- [ ] All router WAN and LAN interfaces are up/up with correct /30 or /24 masks

## If it doesn't work

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Stub default missing on RTR-A-EDGE: CLIENT-A traceroute stops at 10.10.10.1
- Stub default missing on RTR-B-EDGE: CLIENT-B traceroute stops at 10.20.20.1
- Hub lacks route to Site A LAN: CLIENT-B traceroute stops at 10.0.12.2
- Hub lacks route to Site B LAN: CLIENT-A traceroute stops at 10.0.23.1
- Wrong next-hop (to self) on any static: traffic black-holes immediately at the misconfigured router
- Misaddressed /30 on a transit: interfaces show up/up on one side and down/down on the other or ARP fails; traceroute halts at first hop

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ccna-static-routing-fundamentals-bidirectional-end-to-end-connectivity
