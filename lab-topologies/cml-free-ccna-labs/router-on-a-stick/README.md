# Router-on-a-Stick Fundamentals: Two VLANs, One Trunk

CCNA beginner lab for router-on-a-stick inter-VLAN routing using a single router and a single Layer-2 switch. Configure one subinterface per VLAN (10 and 20), a dot1q trunk on the switch uplink, and access ports for two hosts. Begin from an unrouted, non-trunked baseline.

**4 nodes** (alpine, iol-xe, ioll2-xe) — runs on CML-Free  ·  about 45 minutes  ·  beginner

## What you'll configure

- Translate a two-VLAN addressing plan into router-on-a-stick subinterfaces
- Configure an 802.1Q trunk between a router and a Layer-2 switch
- Create VLANs on a Layer-2 switch and assign access ports correctly
- Verify inter-VLAN reachability from end hosts and with IOS show commands
- Recognize and correct trunk allow-list and access-port misconfiguration issues

## Importing

In CML choose **Lab > Import** and pick `router-on-a-stick.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
A small branch office has just segmented its user floor into two VLANs: VLAN 10 for Engineering and VLAN 20 for Sales. The Layer-2 switch was recently refreshed, and the router-on-a-stick uplink must be rebuilt so that both VLANs can reach each other through the router. Until now, users have been unable to contact colleagues in the other department.

Your job is to configure inter-VLAN routing strictly using router-on-a-stick: a single router subinterface per VLAN riding over an 802.1Q trunk to a single Layer-2 switch. You must also ensure that the switch has the correct VLAN database and that each host is on the right access port.

### Prerequisites & Access
- Level and time: Beginner · ~45 minutes
- You will need: Cisco Modeling Labs, and room for 4 nodes (2 network devices, 2 hosts). That fits the 5-node limit on CML Free.
- Import `router-on-a-stick.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **RTR-BRANCH1, SW-ACCESS1** — username `admin` / password `Lab@dmin1`.
- **PC-A, PC-B** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- RTR-BRANCH1 (Cisco IOS router) connects to SW-ACCESS1 on interface Ethernet0/0. This single link must be an 802.1Q trunk. The router’s physical Ethernet0/0 carries no IP; instead, each VLAN will have its own subinterface with an IP address that serves as the default gateway.
- SW-ACCESS1 (Layer-2 switch) provides access ports for two hosts and one trunk uplink to the router. No Layer-3 SVIs are used on the switch.
- PC-A lives in VLAN 10 with address 10.0.10.10/24 and default gateway 10.0.10.1.
- PC-B lives in VLAN 20 with address 10.0.20.10/24 and default gateway 10.0.20.1.

Cabling
- RTR-BRANCH1 Ethernet0/0 ↔ SW-ACCESS1 Ethernet0/0 (router-on-a-stick trunk)
- SW-ACCESS1 Ethernet0/1 ↔ PC-A eth0 (VLAN 10 access)
- SW-ACCESS1 Ethernet0/2 ↔ PC-B eth0 (VLAN 20 access)

### IP Addressing Plan
- VLAN 10 Engineering: 10.0.10.0/24
  - Gateway (router subinterface): 10.0.10.1/24
  - PC-A: 10.0.10.10/24, default route via 10.0.10.1
- VLAN 20 Sales: 10.0.20.0/24
  - Gateway (router subinterface): 10.0.20.1/24
  - PC-B: 10.0.20.10/24, default route via 10.0.20.1

### Tasks — VLAN names are graded exactly as written — VLAN 10 → USERS_VLAN10, VLAN 20 → USERS_VLAN20. Capitalisation is the only variation accepted. 1. Create VLANs on the switch
   - What: Declare VLAN 10 and VLAN 20 in the switch VLAN database.
   - Why: Access ports cannot forward in the intended broadcast domains unless the VLANs exist.
   - Watch for: Typos in VLAN numbers and names; ensure both VLANs are present in 'show vlan brief'.

2. Assign access ports to the correct VLANs
   - What: Make SW-ACCESS1 Ethernet0/1 an access port in VLAN 10 for PC-A, and Ethernet0/2 an access port in VLAN 20 for PC-B. Enable portfast for faster host convergence.
   - Why: Hosts must be in their respective VLANs to receive the correct L2/L3 treatment and default gateway.
   - Watch for: Accidentally leaving a port in the default VLAN 1 or setting it to trunk.

3. Configure the router-on-a-stick trunk and subinterfaces
   - What: On SW-ACCESS1, make Ethernet0/0 a dot1q trunk. On RTR-BRANCH1, leave Ethernet0/0 with no IP, and create exactly one subinterface per VLAN (Ethernet0/0.10 and Ethernet0/0.20), each explicitly tagged for 802.1Q encapsulation matching its own VLAN number and carrying the correct gateway IP.
   - Why: Inter-VLAN routing in this design occurs only across router subinterfaces over the 802.1Q trunk.
   - Watch for: The 802.1Q tag must match the VLAN ID; do not use a native VLAN here. Ensure the physical link is up/no shutdown.

4. Restrict the trunk's allow-list, then test a failure (required)
   - What: Restrict the router-uplink trunk so it permits exactly VLAN 10 and VLAN 20 — no other VLAN, including the default VLAN 1, should be allowed to cross this link. This is a required step, not optional hardening. Briefly test an outage by pruning VLAN 20 from the allow-list, then restore both VLANs.
   - Why: Controlled allow-lists prevent unwanted VLANs from traversing trunks, and the lab expects that the uplink's allowed-VLAN list is set to exactly '10,20'. Seeing the failure helps you recognize the symptom when it happens in production.
   - Watch for: If VLAN 20 is removed from the allowed list, PC-A → PC-B pings will fail. Restore the allow-list to exactly '10,20'.

### Verification
Run checks from the end hosts and then the network devices.
- From PC-A:
  - ip addr show eth0 (expect 10.0.10.10/24)
  - ip route (expect default via 10.0.10.1)
  - ping 10.0.20.10 (should succeed after configuration)
  - traceroute 10.0.20.10 (first hop should be 10.0.10.1)

- From PC-B:
  - ip addr show eth0 (expect 10.0.20.10/24)
  - ip route (expect default via 10.0.20.1)

- On RTR-BRANCH1:
  - show ip interface brief (expect Ethernet0/0.10 and .20 up/up)
  - show ip route (expect connected routes for 10.0.10.0/24 and 10.0.20.0/24)

- On SW-ACCESS1:
  - show vlan brief (expect Eth0/1 in VLAN 10, Eth0/2 in VLAN 20)
  - show interfaces trunk (expect Eth0/0 trunking with VLANs 10,20 allowed and active)

### Troubleshooting
- No ping between VLANs:
  - Check that RTR-BRANCH1 has subinterfaces Ethernet0/0.10 and Ethernet0/0.20 with the correct encapsulation dot1Q tags and IP addresses.
  - Verify SW-ACCESS1 Eth0/0 is an 802.1Q trunk; ensure VLANs 10 and 20 are not pruned.
  - Confirm host ports: Eth0/1 should be access in VLAN 10; Eth0/2 should be access in VLAN 20.
  - Look for a missing VLAN in the switch database.
- Traceroute first hop not 10.0.10.1 from PC-A:
  - The router subinterface for VLAN 10 may be down or mis-tagged.
- Native VLAN mismatch warnings:
  - In this lab, do not configure a native VLAN on the trunk. If you changed it on the switch, remove the change or align it with the router subinterface only if you explicitly make that VLAN native on both ends.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Translate a two-VLAN addressing plan into router-on-a-stick subinterfaces
- [ ] Configure an 802.1Q trunk between a router and a Layer-2 switch
- [ ] Create VLANs on a Layer-2 switch and assign access ports correctly
- [ ] Verify inter-VLAN reachability from end hosts and with IOS show commands
- [ ] Recognize and correct trunk allow-list and access-port misconfiguration issues

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] VLAN 10 and VLAN 20 exist in the switch VLAN database
- [ ] SW-ACCESS1 E0/1 is access in VLAN 10; E0/2 is access in VLAN 20
- [ ] SW-ACCESS1 E0/0 is an 802.1Q trunk permitting VLANs 10 and 20
- [ ] RTR-BRANCH1 has subinterfaces E0/0.10 (10.0.10.1/24) and E0/0.20 (10.0.20.1/24) up/up
- [ ] PC-A can ping and traceroute to PC-B with first hop 10.0.10.1
- [ ] show ip route on RTR-BRANCH1 lists connected 10.0.10.0/24 and 10.0.20.0/24

## If it doesn't work

- Detect a missing VLAN gateway (absent router subinterface)
- Identify a trunk that is not in 802.1Q mode or is pruned to the wrong allow-list
- Spot a host connected to the wrong access VLAN
- Confirm that the router physical interface carries no IP address and is up

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Native VLAN mismatch on the trunk produces warnings and potential untagged traffic issues
- Required VLAN omitted from trunk allow-list blocks inter-VLAN forwarding for that VLAN
- Host port assigned to the wrong VLAN isolates the endpoint from its gateway

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/router-on-a-stick-fundamentals-two-vlans-one-trunk
