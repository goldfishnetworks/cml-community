# Standard ACL: Permit Host & Subnet, Deny Others

Addressing and management are configured. You will add static routes, PAT on RTR-A-EDGE, and a standard ACL on RTR-DC-GW to permit a single NATed host and a partner subnet while denying others.

**5 nodes** (alpine, iol-xe) — runs on CML-Free  ·  about 55 minutes  ·  beginner

## What you'll configure

- Assign addressing to /24 LANs and /30 point-to-point links without overlap
- Configure static routes so LAN-to-LAN traffic forwards end-to-end
- Implement source NAT (PAT) on the source edge and verify translations
- Create a standard numbered ACL to permit one host and one subnet with implicit deny
- Apply the standard ACL near the destination in the correct direction so return traffic is not black-holed
- Validate access with pings from hosts and observe ACL hit counters and NAT translations
- Troubleshoot ACL placement, wildcard math, pre-/post-NAT matching, and mispointed static routes

## Importing

In CML choose **Lab > Import** and pick `standard-acl.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
A healthcare provider is rolling out a small remote clinic (Site A) that needs access to a protected application server in the main data center. Site A uses PAT at its edge router so all client traffic appears as the router’s transit IP. Security policy at the data center says: “Allow the NATed Site A host, allow one pre-approved subnet for partners, and deny everyone else.” Your task is to implement this with a standard numbered ACL placed near the destination so return traffic is not impacted.

You will bring up addressing on all links, add static routes for end-to-end reachability, configure PAT at the Site A edge, and then build and apply the standard ACL on the data center gateway. You’ll validate the policy using pings from two different end hosts: one from Site A (should be permitted) and one from an untrusted LAN at the data center (should be denied).

### Prerequisites & Access
- Level and time: Beginner · ~55 minutes
- You will need: Cisco Modeling Labs, and room for 5 nodes (2 network devices, 3 hosts). That fits the 5-node limit on CML Free.
- Import `standard-acl.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **RTR-A-EDGE, RTR-DC-GW** — username `admin` / password `C1sco-Lab!`; enable password `cisco123`.
- **CLIENT-A, SRV-APP, CLIENT-B** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- RTR-A-EDGE (Site A):
  - LAN-A users: 10.10.10.0/24 (gateway 10.10.10.1)
  - Transit to DC: 10.0.12.0/30 (RTR-A-EDGE=10.0.12.1, RTR-DC-GW=10.0.12.2)
  - Performs PAT so traffic from 10.10.10.0/24 appears as 10.0.12.1 on the WAN
- RTR-DC-GW (Data Center):
  - Protected Server LAN: 10.20.20.0/24 (gateway 10.20.20.1)
  - Untrusted Partner/Guest LAN: 10.30.30.0/24 (gateway 10.30.30.1)
  - Hosts a standard ACL on the server-facing interface to enforce policy
- CLIENT-A (Site A): 10.10.10.10/24, default gateway 10.10.10.1 (NATed at Site A)
- SRV-APP (Data Center): 10.20.20.10/24, default gateway 10.20.20.1 (protected resource)
- CLIENT-B (Data Center Untrusted LAN): 10.30.30.10/24, default gateway 10.30.30.1 (used to prove a deny)

### Objectives Recap
- Static routing: Ensure both client LANs and the server LAN are reachable through the /30 transit link.
- PAT on Site A edge: Translate 10.10.10.0/24 to the edge router’s transit IP.
- Standard ACL near the destination: On RTR-DC-GW, outbound on the server-facing interface, permit a single host (the NATed Site A address) and a specific subnet, then deny others.
- Verification: Prove that CLIENT-A can reach SRV-APP and that CLIENT-B cannot; confirm ACL hit counters and NAT translations.

### IP Addressing Plan
- RTR-A-EDGE
  - Ethernet0/0: 10.0.12.1/30
  - Ethernet0/1: 10.10.10.1/24
- RTR-DC-GW
  - Ethernet0/0: 10.0.12.2/30
  - Ethernet0/1: 10.20.20.1/24
  - Ethernet0/2: 10.30.30.1/24
- CLIENT-A: 10.10.10.10/24 — default gateway 10.10.10.1
- SRV-APP: 10.20.20.10/24 — default gateway 10.20.20.1
- CLIENT-B: 10.30.30.10/24 — default gateway 10.30.30.1

### Tasks
1. Baseline health checks (no ACL yet)
   - Why: Before enforcing policy, verify physical and IP connectivity to simplify later troubleshooting.
   - What to do:
     - On each router, confirm all relevant interfaces are up/up and have correct IPs (/24 for LANs, /30 for transits).
     - From each host, verify it has the correct IP/mask/gateway. You can ping its local default gateway to confirm L2/L3 on the LAN.

2. Configure static routing end-to-end
   - Why: Without proper routes, NAT and ACL results are misleading. Build reachability first.
   - What to do:
     - On RTR-A-EDGE, add a default route pointing to the DC side of the transit (next-hop 10.0.12.2). This sends unknown traffic toward the data center.
     - On RTR-DC-GW, add a specific route back to 10.10.10.0/24 via 10.0.12.1. This ensures server replies return through the transit.
     - Verify: From CLIENT-A, ping 10.0.12.2 and 10.20.20.1. From SRV-APP, ping 10.0.12.1.

3. Configure PAT at Site A
   - Why: The data center will filter based on the NATed source address; this highlights pre-/post-NAT order of operations.
   - What to do:
     - Mark the Site A LAN interface as NAT inside and the transit interface as NAT outside.
     - Use a numbered standard list — this lab requires it to be numbered **access-list 1** — for the inside network, and configure NAT overload so inside sources translate to the WAN interface IP.
     - Verify: From CLIENT-A, ping 10.20.20.10 (should now be able to reach the DC if ACL is not yet applied). On RTR-A-EDGE, observe an active translation.

4. Build the standard ACL policy near the destination
   - Why: A standard ACL matches only the source address. To protect a destination, place it near the destination so it filters the forward path only and does not black-hole replies.
   - What to do:
     - On RTR-DC-GW, create a standard numbered ACL — this lab requires it to be numbered **access-list 11** — that:
       1. Permits the single NATed host (the Site A router’s transit IP — the post-NAT source).
       2. Permits the pre-approved partner subnet — this lab uses the subnet **10.50.50.0/24** (a /24 distinct from the NAT host and any existing local LANs). Use the /24 wildcard so only that subnet is allowed.
       3. Relies on the implicit deny to block all other sources (including the untrusted 10.30.30.0/24 host).
     - Apply ACL 11 outbound on the interface facing the protected server LAN (the interface toward 10.20.20.0/24). Outbound is essential: it filters the forward client-to-server path and avoids dropping server replies.
     - Do not apply it inbound on the server-facing interface; that would only see server-sourced traffic and would not protect the destination as intended.

5. Validate and observe counters
   - Why: You must prove both a permit and a deny from end hosts.
   - What to do:
     - From CLIENT-A (Site A), ping SRV-APP 10.20.20.10. Expect success. On RTR-DC-GW, view ACL hit counters to confirm the permit line incremented. On RTR-A-EDGE, view NAT translations.
     - From CLIENT-B (Untrusted LAN), ping SRV-APP 10.20.20.10. Expect failure. Confirm the ACL’s implicit deny counter increments.

### Verification
- From CLIENT-A (10.10.10.10):
  - ping 10.20.20.10 → Success (permitted via NATed source host)
- From CLIENT-B (10.30.30.10):
  - ping 10.20.20.10 → Fails (denied by implicit deny since its subnet is not explicitly permitted)
- From routers (reference only):
  - On RTR-DC-GW: show the ACL counters for both the permitted host and total denies.
  - On RTR-A-EDGE: show IP NAT translations to confirm PAT is active for CLIENT-A.

### Troubleshooting
- Placement and direction: A standard ACL protecting a server must be placed near the destination and applied outbound on the server-facing interface. Inbound on that interface inspects server replies instead of client requests, causing odd failures.
- NAT vs ACL matching: The DC router only sees the post-NAT source (the Site A router’s transit IP). If you match the pre-NAT host (10.10.10.10), your ACL will never permit it. Always match what the device actually sees.
- Wildcard math: A /24 subnet uses a 0.0.0.255 wildcard. If you intend to allow exactly one /24 but mistakenly use a broader mask (like 0.0.255.255), you could allow far more than intended. Put specific permits first.
- Static routes: A next-hop must be the neighbor’s IP, not your own. A route “to self” black-holes reachability.
- Order of statements: Put the specific host permit before the subnet permit to avoid accidental shadowing. The implicit deny at the end does the rest.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Assign addressing to /24 LANs and /30 point-to-point links without overlap
- [ ] Configure static routes so LAN-to-LAN traffic forwards end-to-end
- [ ] Implement source NAT (PAT) on the source edge and verify translations
- [ ] Create a standard numbered ACL to permit one host and one subnet with implicit deny
- [ ] Apply the standard ACL near the destination in the correct direction so return traffic is not black-holed
- [ ] Validate access with pings from hosts and observe ACL hit counters and NAT translations
- [ ] Troubleshoot ACL placement, wildcard math, pre-/post-NAT matching, and mispointed static routes
- [ ] Static routes appear with expected administrative distance.
- [ ] Traffic prefers the primary path before failure.
- [ ] Backup routes activate when the primary path is lost.

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] CLIENT-A: ping 10.20.20.10 succeeds
- [ ] CLIENT-B: ping 10.20.20.10 fails
- [ ] RTR-A-EDGE: show ip nat translations shows an active ICMP entry for 10.10.10.10 translating to 10.0.12.1
- [ ] RTR-DC-GW: show access-lists (or show ip access-lists) displays permit counters incrementing for the host ACE and deny counters increasing overall
- [ ] RTR-A-EDGE: default route to 10.0.12.2 is present
- [ ] RTR-DC-GW: route to 10.10.10.0/24 via 10.0.12.1 is present

## If it doesn't work

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- ACL applied inbound on the server-facing interface: CLIENT-A fails, counters don’t increment as expected for forward traffic; return packets may be disrupted
- ACL references pre-NAT source 10.10.10.10 instead of post-NAT 10.0.12.1: CLIENT-A fails but NAT shows translations
- Incorrect wildcard mask (e.g., 0.0.255.255) unintentionally allows or denies broader ranges
- Static route mispointed to self or wrong next-hop: neither client reaches the server
- Missing NAT inside/outside designation on RTR-A-EDGE: no translations occur and connectivity fails

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/standard-acl-fundamentals-permit-host-subnet-deny-others
