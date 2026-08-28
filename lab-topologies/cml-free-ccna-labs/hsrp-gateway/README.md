# HSRP Fundamentals: A Virtual Default Gateway

Baseline addressing and L2 setup for a single VLAN LAN. Configure HSRP group 1 with VIP 10.0.10.254 on R1 and R2 to provide a resilient default gateway for CLIENT10.

**4 nodes** (alpine, iol-xe, ioll2-xe) — runs on CML-Free  ·  about 35 minutes  ·  beginner

## What you'll configure

- Explain how HSRP provides a resilient default gateway using a shared virtual IP and MAC
- Configure HSRP group 1 with a common virtual IP on two routers in the same VLAN
- Observe deterministic active/standby selection with equal priorities
- Verify HSRP operation using show standby and show standby brief
- Validate host connectivity to the virtual default gateway

## Importing

In CML choose **Lab > Import** and pick `hsrp-gateway.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
A small branch LAN hosts users and a few on-prem applications. The operations team wants a default gateway that survives a router outage without changing host settings. Hot Standby Router Protocol (HSRP) solves this by presenting one virtual gateway IP and MAC that two routers share. One router is Active and forwards packets; the other is Standby and ready to take over instantly if needed.

In this first lab of the series, you will configure a basic HSRP pair on a single user VLAN. The host will point its default route at the virtual IP, not at either router’s real address. With equal default priorities, the router with the higher interface IP should become Active.

### Prerequisites & Access
- Level and time: Beginner · ~35 minutes
- You will need: Cisco Modeling Labs, and room for 4 nodes (3 network devices, 1 host). That fits the 5-node limit on CML Free.
- Import `hsrp-gateway.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **CLIENT10** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- RTR-A (R1) and RTR-B (R2) connect to a single access switch SW1 on VLAN 10.
- CLIENT10 is a Linux host on the same access VLAN 10.
- The LAN is 10.0.10.0/24. R1 = 10.0.10.1/24, R2 = 10.0.10.2/24, CLIENT10 = 10.0.10.10/24.
- The virtual default gateway for the LAN is 10.0.10.254 (HSRP group 1). CLIENT10 already points its default route to this virtual IP.

### IP Addressing Plan
- VLAN 10 Users: 10.0.10.0/24
  - R1 Ethernet0/0: 10.0.10.1/24
  - R2 Ethernet0/0: 10.0.10.2/24
  - CLIENT10 eth0: 10.0.10.10/24
  - Virtual Gateway (HSRP g1): 10.0.10.254

### Tasks
1. Review the baseline addressing on R1 and R2.
   - What: Confirm each router’s LAN interface (Ethernet0/0) has the correct IP/mask and is up.
   - Why: HSRP runs on the host-facing, L2-shared segment and requires matching Layer 3 reachability on the same subnet.

2. Prepare the access layer.
   - What: Ensure SW1 has VLAN 10 and that the ports toward R1, R2, and CLIENT10 are access ports in VLAN 10 and up.
   - Why: HSRP relies on both routers being in the same broadcast domain as the host.

3. Configure HSRP on the LAN interfaces of both routers using group 1 and the shared virtual IP 10.0.10.254.
   - What: Apply the HSRP group with the virtual IP on R1 and R2 on Ethernet0/0.
   - Why: Both routers must advertise the same group and virtual IP to form an Active/Standby pair.
   - Watch for: Do not configure priorities or preempt in this fundamentals lab—observe the default election behavior.

4. Validate the deterministic election with default priorities.
   - What: With equal default priority, verify which router becomes Active and which becomes Standby.
   - Why: By default, the router with the higher real interface IP address wins when priorities tie.

5. Confirm end-host gateway reachability.
   - What: From CLIENT10, ping 10.0.10.254.
   - Why: The host should rely on the virtual IP as its default gateway—the immediate test is ICMP reachability to the VIP.

6. Observe the virtual MAC and state.
   - What: On routers, inspect HSRP state for group 1 and note the virtual MAC address associated with the VIP.
   - Why: The virtual MAC is what hosts learn for the gateway on ARP; only the Active router responds with it for forwarding.

### Verification
- From CLIENT10: ping 10.0.10.254 should succeed.
- On R1 and R2: show the HSRP summary for group 1. Expect one Active, one Standby on VLAN 10 with virtual IP 10.0.10.254. With identical default priority 100, R2 (10.0.10.2) should be Active and R1 (10.0.10.1) Standby.
- Optionally view the virtual MAC (format 0000.0c07.acXX) and confirm it matches group 1.

### Troubleshooting
- If the host cannot ping the VIP:
  - Check that both routers’ Ethernet0/0 are up and in 10.0.10.0/24 with correct masks.
  - Confirm HSRP group number and virtual IP match on both routers and live on Ethernet0/0.
  - Verify SW1 access ports are in VLAN 10 and not err-disabled; ensure links are up.
  - Inspect HSRP state: ensure one router shows Active and the other Standby for group 1.
- If one router shows Listen or Init:
  - Confirm the group number and VIP are identical on both routers and there is L2 adjacency (same VLAN).
- If ARP seems stale on CLIENT10:
  - Re-try the ping; optionally clear ARP on the host or wait briefly for ARP resolution.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Explain how HSRP provides a resilient default gateway using a shared virtual IP and MAC
- [ ] Configure HSRP group 1 with a common virtual IP on two routers in the same VLAN
- [ ] Observe deterministic active/standby selection with equal priorities
- [ ] Verify HSRP operation using show standby and show standby brief
- [ ] Validate host connectivity to the virtual default gateway
- [ ] HSRP peers reach the expected active and standby states.
- [ ] Hosts use the virtual IP as their default gateway.
- [ ] Failover occurs within the expected convergence window.

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] CLIENT10 successfully pings the virtual gateway 10.0.10.254
- [ ] show standby brief on R2 shows Active for group 1 at 10.0.10.254
- [ ] show standby brief on R1 shows Standby for group 1 at 10.0.10.254
- [ ] The HSRP virtual MAC (0000.0c07.ac01) appears for group 1
- [ ] CLIENT10 default route points to 10.0.10.254 (already provided)

## If it doesn't work

- Both routers must use the same HSRP group and identical virtual IP on the same interface/VLAN
- Do not assign the virtual IP as a physical interface address on either router
- Ensure the switch ports are in the correct access VLAN and up/up
- If the host cannot ping the VIP, verify ARP and the HSRP state (Active/Standby present)
- Mismatched masks or shutdown interfaces prevent HSRP from forming properly

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Incorrect HSRP group or virtual IP on one router prevents an Active/Standby pair
- LAN interface down or wrong subnet mask stops HSRP from forming and blocks ARP to the VIP
- Access switch port in the wrong VLAN isolates one node from the HSRP multicast/ARP domain

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/hsrp-fundamentals-a-virtual-default-gateway
