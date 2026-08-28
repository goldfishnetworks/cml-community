# DHCP Server Fundamentals: One Pool

Configure a single IOS DHCP pool (network + default-router) so two Alpine clients on the same LAN lease addresses dynamically. SW1 remains pure L2.

**4 nodes** (alpine, iol-xe, ioll2-xe) — runs on CML-Free  ·  about 35 minutes  ·  beginner

## What you'll configure

- Configure a basic IOS DHCP server with one pool using network and default-router
- Understand pure Layer-2 switching versus Layer-3 gateway roles
- Verify client leases from Linux (udhcpc, ip addr) and from IOS (show ip dhcp pool, show ip dhcp binding)
- Troubleshoot common DHCP pitfalls: disabled interface, wrong gateway address, client not requesting

## Importing

In CML choose **Lab > Import** and pick `dhcp-server.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
You are the new admin for a small branch office. The site has a single router acting as the LAN gateway and a compact Layer-2 access switch that fans out to user devices. Static IPs on end hosts were error-prone and time-consuming, so the network team is moving to centrally managed, automated addressing. Your task is to stand up the canonical DHCP service on the router so wired clients can obtain addresses automatically.

This is Lab 1 in a progressive DHCP & Address Services series. Here you will build the one-pool baseline correctly and verify it both from the router and from the Linux clients.

### Prerequisites & Access
- Level and time: Beginner · ~35 minutes
- You will need: Cisco Modeling Labs, and room for 4 nodes (2 network devices, 2 hosts). That fits the 5-node limit on CML Free.
- Import `dhcp-server.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **CLIENT-A, CLIENT-B** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- DHCP-SRV (IOS-XE router): The default gateway for the LAN. Its Ethernet0/0 interface is the gateway at 10.10.10.1/24 and will host the DHCP server for this subnet.
- SW1 (IOS Layer-2 switch): A pure access switch. It provides shared media for the single LAN and has no Layer-3 configuration.
- CLIENT-A and CLIENT-B (Alpine Linux hosts): Endpoints connected to SW1. They will use a DHCP client (udhcpc) to request addresses from DHCP-SRV.

Everything resides on one broadcast domain (10.10.10.0/24). There is no relay involved in this lab.

### IP Addressing Plan
- LAN 10.10.10.0/24
  - Default gateway: 10.10.10.1 (DHCP-SRV Ethernet0/0)
  - DHCP pool name: LAN
  - Scope: 10.10.10.0/24 (do not add exclusions in this lab)
  - Default-router option: 10.10.10.1

Note: Lease time may remain at the platform default. Do not configure DNS, domain-name, or exclusions yet; those refinements come later in the series.

### Tasks
1. Prepare the LAN gateway interface on the router
   - What: Ensure DHCP-SRV Ethernet0/0 is up with IP 10.10.10.1/24.
   - Why: The default-router you will advertise in DHCP must be a real, reachable interface on the client subnet. If the interface is shut or incorrectly addressed, clients cannot use it as a gateway.

2. Create the canonical DHCP pool on the router
   - What: Build a single pool named LAN that specifies the exact client subnet (network 10.10.10.0 255.255.255.0) and the default-router 10.10.10.1.
   - Why: The pool scope tells the router which addresses are usable, and the default-router option tells clients their L3 gateway. Leave the lease as the default and do not add exclusions in this first lab.

3. Keep the switch purely Layer-2
   - What: Confirm SW1 remains a simple access switch. Host-facing ports operate as access, and no SVI or routing is configured.
   - Why: This keeps the gateway and DHCP server centralized on the router. The switch is only bridging frames on this single LAN.

4. Trigger DHCP on the clients
   - What: From CLIENT-A and CLIENT-B, run the Linux DHCP client (udhcpc -i eth0) after the router’s pool is in place.
   - Why: The hosts must actively request a lease. Do not set a static address on the clients; the goal is to confirm dynamic assignment inside the pool’s range.

5. Confirm end-to-end LAN reachability
   - What: From each client, ping the gateway (10.10.10.1) and then ping the other client.
   - Why: Verifies both the L2 path and that the leased IPs and gateway are working for basic connectivity.

### Verification
Run these checks and confirm the expected outcomes:
- From CLIENT-A and CLIENT-B
  - ip addr show eth0: Each host should have an address in 10.10.10.0/24 with the expected /24 mask.
  - ping 10.10.10.1: Gateway should respond.
  - ping <peer’s 10.10.10.x>: The two clients should reach each other.

- From DHCP-SRV
  - show ip dhcp pool: The LAN pool should show utilization and at least two active leases.
  - show ip dhcp binding: You should see two dynamic bindings corresponding to the clients’ MAC addresses.

### Troubleshooting
- Interface down: If clients get no address, ensure DHCP-SRV Ethernet0/0 is up/up and exactly 10.10.10.1/24.
- Pool mis-scope: If the pool network does not match the interface subnet, no addresses will be offered. Double-check network and mask.
- Wrong default-router: If clients get an address but can’t reach the gateway, verify the pool’s default-router matches the router’s real interface IP.
- Client not requesting: Re-run udhcpc -i eth0 on each client after the pool is configured. If it timed out earlier, it won’t magically retry unless you start it again.
- Switch L3 creep: Ensure SW1 has no SVI with an IP. It must be a pure Layer-2 bridge for this lab.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Configure a basic IOS DHCP server with one pool using network and default-router
- [ ] Understand pure Layer-2 switching versus Layer-3 gateway roles
- [ ] Verify client leases from Linux (udhcpc, ip addr) and from IOS (show ip dhcp pool, show ip dhcp binding)
- [ ] Troubleshoot common DHCP pitfalls: disabled interface, wrong gateway address, client not requesting

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] DHCP-SRV Ethernet0/0 is up/up at 10.10.10.1/24
- [ ] Router has 'ip dhcp pool LAN' with 'network 10.10.10.0 255.255.255.0' and 'default-router 10.10.10.1'
- [ ] On DHCP-SRV: 'show ip dhcp pool' shows at least two addresses in use
- [ ] On DHCP-SRV: 'show ip dhcp binding' lists two dynamic bindings
- [ ] CLIENT-A: 'ip addr show eth0' displays an address in 10.10.10.0/24
- [ ] CLIENT-B: 'ip addr show eth0' displays an address in 10.10.10.0/24
- [ ] CLIENT-A and CLIENT-B can ping 10.10.10.1 (gateway)
- [ ] CLIENT-A and CLIENT-B can ping each other by their leased 10.10.10.x addresses

## If it doesn't work

- Client never gets an address: verify router LAN interface is up/up with the correct /24 and the DHCP pool exists
- Wrong gateway appears on client: ensure the pool default-router matches the actual interface IP
- Bindings do not increment: confirm clients actually run udhcpc after the pool is configured

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Client times out on DHCP: Check that the router’s LAN interface is not administratively down and the DHCP pool exists
- Client receives an address but no connectivity: Verify the pool's default-router equals the router interface IP on that subnet
- Bindings never increase: Confirm hosts are actually running udhcpc after the pool is created (rerun the client if needed)

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/dhcp-server-fundamentals-one-pool
