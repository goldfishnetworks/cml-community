# CCNA NAT1: Static One-to-One NAT with ISP

Starter topology for static NAT. Addresses and routes are in place, but NAT is not yet configured.

**4 nodes** (alpine, iol-xe) — runs on CML-Free  ·  about 40 minutes  ·  beginner

## What you'll configure

- Identify and mark inside vs outside interfaces correctly on an IOS router
- Configure a static one-to-one NAT mapping with ip nat inside source static
- Verify static NAT: permanent translation entry and statistics on R1
- Confirm end-to-end reachability from the inside host to a public server through translation
- Demonstrate bidirectional reachability to the mapped global IP from the public side

## Importing

In CML choose **Lab > Import** and pick `topology.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
Your team operates a small branch with an internet edge router (R1) connecting to an upstream provider (ISP). A single workstation (PC-A) in the branch must use a fixed public identity for application testing and inbound diagnostics from the public side. The provider assigns you the 203.0.113.0/29 segment on the R1–ISP link. You decide to deploy Static NAT so PC-A (192.168.10.10) always translates to 203.0.113.3.

Routing is already functional between R1 and the ISP, and a public server (PUB-SRV) exists one hop beyond the ISP to simulate an internet host. Your goal is to configure a bidirectional, permanent one-to-one static NAT mapping on R1, verify that PC-A reaches the public server using its public identity, and confirm that the public server can initiate back to PC-A via the mapped global IP.

### Prerequisites & Access
- Level and time: Beginner · ~40 minutes
- You will need: Cisco Modeling Labs, and room for 4 nodes (2 network devices, 2 hosts). That fits the 5-node limit on CML Free.
- Import `topology.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **PC-A, PUB-SRV** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- R1 (Cisco IOS) is the NAT edge. It has:
  - Ethernet0/0 toward the inside LAN 192.168.10.0/24 (gateway 192.168.10.1)
  - Ethernet0/1 toward the ISP on 203.0.113.0/29 (R1=203.0.113.1, ISP=203.0.113.2)
  - A static default route to the ISP so routing works before NAT.
- ISP (Cisco IOS) emulates the upstream provider:
  - Ethernet0/0 on 203.0.113.2/29, directly facing R1
  - Ethernet0/1 on 198.51.100.1/24 toward PUB-SRV
- PC-A (Alpine Linux) is the inside host:
  - 192.168.10.10/24 with default route to 192.168.10.1 (already configured)
- PUB-SRV (Alpine Linux) is a public server:
  - 198.51.100.10/24 with default route to 198.51.100.1 (already configured)

Traffic flow to test:
- Inside to outside: PC-A -> PUB-SRV should source-translate to 203.0.113.3
- Outside to inside: PUB-SRV -> 203.0.113.3 should map to 192.168.10.10 (static NAT is bidirectional)

### IP Addressing Plan
- Inside LAN: 192.168.10.0/24
  - R1 Ethernet0/0: 192.168.10.1/24 (gateway)
  - PC-A: 192.168.10.10/24, default via 192.168.10.1
- Outside transit: 203.0.113.0/29
  - R1 Ethernet0/1: 203.0.113.1/29
  - ISP Ethernet0/0: 203.0.113.2/29
  - Static NAT global address: 203.0.113.3 (on the same outside segment, reachable by the ISP)
- Public server network: 198.51.100.0/24
  - ISP Ethernet0/1: 198.51.100.1/24
  - PUB-SRV: 198.51.100.10/24, default via 198.51.100.1

### Tasks
1. Mark NAT interface roles on R1
- What: Identify and mark the private-facing interface as inside and the provider-facing interface as outside.
- Why: NAT only translates traffic that crosses from an inside interface to an outside interface (and vice versa for static entries). Without correct roles, translation will never occur.
- Watch for: Do not reverse these; ensure Ethernet0/0 is inside and Ethernet0/1 is outside.

2. Configure one-to-one static NAT for PC-A
- What: Create a static mapping binding 192.168.10.10 to 203.0.113.3.
- Why: The host must present a fixed, public identity. Static NAT is permanent and bidirectional.
- Watch for: Use a global address that belongs to the 203.0.113.0/29 segment so the ISP can return traffic directly to R1 for that global IP.

3. Preserve pre-NAT routing
- What: Leave the provided default route on R1 in place; it sends non-local traffic to the ISP.
- Why: NAT does not create routes. Routing must already work so that translated packets can be forwarded and return traffic can arrive.
- Watch for: The next-hop must be the ISP’s directly connected address (203.0.113.2) on the /29 link.

4. Validate bidirectional reachability
- What: Test inside-to-outside (PC-A to PUB-SRV) and outside-to-inside (PUB-SRV to 203.0.113.3) paths.
- Why: Static NAT is symmetrical. Success confirms both translation and routing are correct.
- Watch for: From R1, you should see a permanent translation, and counters should increment when you test from the hosts.

### Verification
Run host-side checks only from the Linux endpoints and IOS show commands only on R1:
- On PC-A:
  - ip addr (confirm 192.168.10.10/24)
  - ping -c 3 198.51.100.10 (should succeed)
- On PUB-SRV:
  - ip addr (confirm 198.51.100.10/24)
  - ping -c 3 203.0.113.3 (should succeed; reaches PC-A via the static mapping)
- On R1:
  - show ip nat translations (should list one permanent static entry mapping 192.168.10.10 to 203.0.113.3)
  - show ip nat statistics (hits should increase after ping tests)

Expected results:
- PC-A successfully pings PUB-SRV
- PUB-SRV successfully pings 203.0.113.3 and reaches PC-A through the translation
- R1 shows a permanent static NAT entry plus packet counters incrementing during tests

### Troubleshooting
- No translation at all:
  - Check interface roles on R1: inside must be Ethernet0/0 and outside must be Ethernet0/1.
- Outside host can’t reach 203.0.113.3:
  - Confirm the global IP is within 203.0.113.0/29 and that the ISP has that /29 as a connected network.
- Inside ping fails:
  - Verify PC-A’s default route points to 192.168.10.1 and that PC-A’s NIC is up.
- Counters don’t increase on R1:
  - Ensure your tests traverse R1; pings sourced from R1 itself do not use NAT.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Identify and mark inside vs outside interfaces correctly on an IOS router
- [ ] Configure a static one-to-one NAT mapping with ip nat inside source static
- [ ] Verify static NAT: permanent translation entry and statistics on R1
- [ ] Confirm end-to-end reachability from the inside host to a public server through translation
- [ ] Demonstrate bidirectional reachability to the mapped global IP from the public side
- [ ] ACL hit counts reflect expected traffic flows.
- [ ] NAT translations appear for intended inside hosts.
- [ ] Denied traffic is blocked without affecting permitted flows.

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] PC-A can ping 198.51.100.10 successfully
- [ ] PUB-SRV can ping 203.0.113.3 successfully (reaching 192.168.10.10)
- [ ] R1 shows a permanent static NAT mapping for 192.168.10.10 <-> 203.0.113.3
- [ ] R1 'show ip nat statistics' increments hits after host pings
- [ ] Routing works pre-NAT: R1 has a default route to 203.0.113.2 and ISP sees 203.0.113.0/29 connected

## If it doesn't work

- No translation appears: inside/outside roles may be reversed or missing
- Return traffic fails: mapped global address is not on the R1 outside segment
- Inside host unreachable: missing default route on host or gateway not 192.168.10.1
- Routing broken pre-NAT: confirm R1's default route toward ISP and ISP's connected reachability

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Inside/outside role reversal prevents translations from ever appearing
- Global address chosen outside the provider segment breaks return traffic
- Host default route misconfigured causes inside traffic to never reach R1 for translation

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ccna-nat1-static-one-to-one-nat-with-isp
