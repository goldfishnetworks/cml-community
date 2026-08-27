# CCNA labs that fit the CML-Free node limit

20 self-contained CCNA practice labs, none larger than **5 nodes**, so the whole set runs on a
free Cisco Modeling Labs licence with nothing trimmed out.

Each lab covers one skill. The topology boots with a starting configuration, the instructions are attached
to the lab's Notes in CML as well as written out in the folder's README, and a verification checklist gives
you a way to prove the result from the device itself. The labs are independent — take them in any order, or
follow the sections below, which run roughly in CCNA blueprint order.

## What you need

- Cisco Modeling Labs, any tier. The largest topology here is 5 nodes, which is the CML-Free ceiling.
- Node definitions used: `alpine`, `iol-xe`, `ioll2-xe`.
- All of these are part of the CML reference platform, so there are no extra images to source.

## The labs

### Addressing and routing fundamentals

- [CCNA L1: Interface Addressing and Verification](interface-addressing/) — Routers are cabled but interfaces lack IPv4 addresses. Configure the given /24 addresses on the correct interfaces and bring them up. Verify directly-connected reachability only.
- [CCNA Static Routing: Bidirectional End-to-End Connectivity](static-routing/) — Baseline hub-and-spoke with two user LANs. Interfaces and addressing are configured. Static routes are intentionally NOT configured so learners can add them to enable bidirectional end-to-end connectivity.

### Switching

- [VLAN Fundamentals: Creating & Assigning Access Ports](vlans-access-ports/) — A single access switch with four hosts attached. Your task: create VLAN 10 and VLAN 20, assign access ports to each, and validate intra-VLAN reachability and inter-VLAN isolation (no routing present).
- [802.1Q Trunk Fundamentals: Static Trunk and VLANs](dot1q-trunking/) — Switches cabled but VLANs and trunking not yet configured. Implement a static 802.1Q trunk carrying VLANs 10 and 20, assign access ports, and verify same-VLAN reachability across switches.
- [Router-on-a-Stick Fundamentals: Two VLANs, One Trunk](router-on-a-stick/) — CCNA beginner lab for router-on-a-stick inter-VLAN routing using a single router and a single Layer-2 switch. Configure one subinterface per VLAN (10 and 20), a dot1q trunk on the switch uplink, and access ports for two hosts. Begin from an unrouted, non-trunked baseline.
- [CCNA Inter-VLAN Routing: Two Clients Across a Router](inter-vlan-routing/) — Deploy, verify, and troubleshoot end-to-end inter-VLAN L3 routing.
- [Spanning Tree Fundamentals: Root Election & Port Roles](spanning-tree/) — Two-switch redundant L2 loop with two parallel uplinks. Implement Rapid-PVST+ and deterministic root control; verify one SW2 uplink blocks. Hosts in VLAN 1: 10.0.1.10/24 and 10.0.1.11/24.
- [Static EtherChannel: Bundling Two Links (mode on)](etherchannel-static/) — Build a static Layer-2 EtherChannel (mode on) between SW1 and SW2 using two parallel links as members of Port-channel1. Port-channel1 is an access port in VLAN 10; PC-A and PC-B are in VLAN 10 and should ping across the bundle. Baseline ships without the EtherChannel configured.

### Dynamic routing

- [OSPFv2 Single-Area Fundamentals: Adjacency and Area 0 Basics](ospfv2-single-area/) — Addressing and SSH ready. Configure OSPFv2 area 0 across the 3-router chain, advertise loopbacks and LANs, verify adjacencies and routes, and troubleshoot an area mismatch.
- [EIGRP Fundamentals: First Adjacency & Route Exchange](eigrp-adjacency/) — Interfaces addressed and hosts configured; EIGRP not yet enabled. Goal: bring up EIGRP AS 100 between R1 and R2 and advertise R1's LAN.
- [eBGP Fundamentals: The First Peering](ebgp-peering/) — Build an eBGP session over a /30 between two ASes and advertise one /24 from each side using Loopback0. Interfaces and IPs are pre-configured; BGP is not yet applied.

### Redundancy and address services

- [HSRP Fundamentals: A Virtual Default Gateway](hsrp-gateway/) — Baseline addressing and L2 setup for a single VLAN LAN. Configure HSRP group 1 with VIP 10.0.10.254 on R1 and R2 to provide a resilient default gateway for CLIENT10.
- [DHCP Server Fundamentals: One Pool](dhcp-server/) — Configure a single IOS DHCP pool (network + default-router) so two Alpine clients on the same LAN lease addresses dynamically. SW1 remains pure L2.
- [CCNA NAT1: Static One-to-One NAT with ISP](static-nat/) — Starter topology for static NAT. Addresses and routes are in place, but NAT is not yet configured.

### Security

- [Standard ACL: Permit Host & Subnet, Deny Others](standard-acl/) — Addressing and management are configured. You will add static routes, PAT on RTR-A-EDGE, and a standard ACL on RTR-DC-GW to permit a single NATed host and a partner subnet while denying others.
- [CCNA Port Security 1: Enable & Verify on Access Ports](port-security/) — Baseline L2 campus with two switches and a trunk. VLAN 10 users on access ports. Objective: enable port security (maximum 1, violation shutdown) on two host access ports and verify.
- [DHCP Snooping Trust Boundary](dhcp-snooping/) — VLAN 10 access with a router gateway/DHCP server. Learner will enable DHCP Snooping and trust only the uplink.

### Device management

- [CCNA: SSH Access Fundamentals on R1](ssh-access/) — Management LAN addressing is present. Configure SSH access and local authentication on R1; verify from ADMIN.
- [CCNA NTP Client: Sync to an Authoritative Server](ntp-client/) — Two IOS routers on a single /30. Configure R1 as NTP master (stratum 3) and R2 as a client. No routing protocols or static routes.
- [CDP Neighbor Discovery and Edge Suppression](cdp-neighbor-discovery/) — CDP is globally OFF on R1. Your goal is to enable CDP on R1 to learn directly-connected router neighbors while keeping CDP disabled on the untrusted management-facing interface toward SRV-MGMT01.

## About

These are the free-tier labs from Goldfish Networks, contributed in full — topology, instructions and
verification checklist — so each one stands on its own inside CML. The site carries the same guides and
additionally grades a configuration you paste into it against the lab's answer key, which is the part that
does not travel into a repository. https://goldfishnetworks.com

Corrections to any topology here are welcome as issues or pull requests.
