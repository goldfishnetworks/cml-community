# CCNA Port Security 1: Enable & Verify on Access Ports

Baseline L2 campus with two switches and a trunk. VLAN 10 users on access ports. Objective: enable port security (maximum 1, violation shutdown) on two host access ports and verify.

**5 nodes** (alpine, ioll2-xe) — runs on CML-Free  ·  about 40 minutes  ·  beginner

## What you'll configure

- Configure access ports and a basic inter-switch 802.1Q trunk for a user VLAN
- Enable port security on host-facing access ports
- Explicitly configure and verify the feature's default enforcement policy so a port allows exactly one learned MAC address and disables itself on any violation
- Validate host connectivity and interpret show port-security outputs

## Importing

In CML choose **Lab > Import** and pick `port-security.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
A small campus floor just completed a switch refresh. Your team’s standard requires that all user-facing access ports be locked down with port security to reduce the risk of rogue devices and accidental loops. For this first step, your manager wants you to enable a basic, deterministic policy on exactly two user ports: the explicit defaults of maximum 1 secure MAC and violation shutdown. No sticky or static bindings yet—just the feature turned on and verified.

You will deploy port security on two access ports that connect to Linux hosts. The two access switches are linked by a basic 802.1Q trunk carrying the user VLAN so same-VLAN hosts on different closets can talk.

### Prerequisites & Access
- Level and time: Beginner · ~40 minutes
- You will need: Cisco Modeling Labs, and room for 5 nodes (2 network devices, 3 hosts). That fits the 5-node limit on CML Free.
- Import `port-security.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **SW-A-ACC1, SW-B-ACC1** — username `admin` / password `Lab@dmin1`.
- **PC-A1, PC-B1, PC-B2** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- SW-A-ACC1 and SW-B-ACC1 are Layer-2 access switches. They are connected by a single trunk on Ethernet0/0.
- VLAN 10 (Users) is defined on both switches. Access ports are placed into VLAN 10.
- PC-A1 plugs into SW-A-ACC1 Ethernet0/1.
- PC-B1 plugs into SW-B-ACC1 Ethernet0/1.
- PC-B2 plugs into SW-B-ACC1 Ethernet0/2 (an extra user port for scale; not part of the graded port-security task).
- There is no routing, no SVIs, and no default gateway required. This is a pure Layer-2 lab.

### IP Addressing Plan
- VLAN 10 Users: 10.10.10.0/24 (no gateway; L2-only lab)
  - PC-A1: 10.10.10.10/24
  - PC-B1: 10.10.10.20/24
  - PC-B2: 10.10.10.30/24

### Tasks
1. Confirm the VLAN and trunk baseline
   - What: On both switches, verify that VLAN 10 exists and that Ethernet0/0 is an 802.1Q trunk linking the two switches. Verify that host-facing ports are access ports in VLAN 10.
   - Why: Port security protects access ports. Ensuring correct Layer-2 placement and transport first avoids confusing port-security symptoms with basic VLAN/trunk issues.

2. Enable port security on two host-facing access ports
   - What: On SW-A-ACC1 interface Ethernet0/1 and on SW-B-ACC1 interface Ethernet0/1, turn on port security and explicitly configure its policy to match the feature's own defaults: cap the port at exactly 1 allowed secure MAC address, and set the violation action so the port shuts itself down (err-disables) the moment a second MAC shows up. Do not apply port security to the trunk (Ethernet0/0).
   - Why: Port security already behaves this way out of the box, but writing the policy out explicitly makes it obvious, reviewable, and gradable rather than relying on implicit defaults. Do not apply port security to the trunk (Ethernet0/0).

3. Do not configure sticky or static secure MACs
   - What: Leave sticky/static off. This lab focuses on enabling the feature with defaults, not on binding persistence or pre-defined MACs.
   - Why: Keep the behavior deterministic and simple for fundamentals. You will learn sticky and static bindings in later labs.

4. Generate light traffic to populate learned secure MACs (optional)
   - What: From PC-A1, ping PC-B1. This causes the switches to learn each host’s MAC address on its respective access port.
   - Why: With traffic, show port-security interface may display a learned secure MAC count on each protected port.

### Verification
Run these checks from the hosts and switches:
- From PC-A1:
  - ip addr show eth0 (expect: eth0 has 10.10.10.10/24)
  - ping 10.10.10.20 (expect: replies from PC-B1)
- From PC-B1:
  - ip addr show eth0 (expect: eth0 has 10.10.10.20/24)
- From SW-A-ACC1:
  - show port-security
  - show port-security interface ethernet0/1
    - Expect: Port Status secure-up, Maximum 1, Violation Shutdown
- From SW-B-ACC1:
  - show port-security
  - show port-security interface ethernet0/1
    - Expect: Port Status secure-up, Maximum 1, Violation Shutdown

### Troubleshooting
- No connectivity between hosts:
  - Ensure both access ports are in VLAN 10 and the inter-switch link is a trunk. If VLAN 10 is missing on one switch, add it.
- Port-security not active:
  - Confirm the host port is in access mode with VLAN 10 assigned before enabling port security.
- Trunk not carrying VLAN 10:
  - Confirm the Ethernet0/0 inter-switch link on both switches is operating as a trunk.
- Violation state or err-disable:
  - In this lab you set maximum 1 and violation shutdown, but only one host is connected to each protected port. If a port enters err-disabled, check for cabling mistakes (e.g., a small unmanaged device behind a port) and recover administratively after fixing the cause.
- Port security configured vs enabled: an interface accepts the port-security options (violation mode, sticky learning, maximum addresses) even when port security itself is switched off — so a port can read as fully configured and still not be protected. Check with `show port-security interface <interface>`: it must report `Port Security: Enabled`. If it reports `Disabled`, the interface is missing the command that turns the feature on, and a grading check for it will fail even though your other port-security lines are there.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Configure access ports and a basic inter-switch 802.1Q trunk for a user VLAN
- [ ] Enable port security on host-facing access ports
- [ ] Explicitly configure and verify the feature's default enforcement policy so a port allows exactly one learned MAC address and disables itself on any violation
- [ ] Validate host connectivity and interpret show port-security outputs
- [ ] Port-security shows as enabled on intended access ports.
- [ ] Expected secure MAC addresses are learned or statically defined.
- [ ] Violation counters change during the fault scenario.
- [ ] Affected interfaces are recovered and validated after remediation.

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] VLAN 10 exists on both switches
- [ ] Trunk on Ethernet0/0 is up between SW-A-ACC1 and SW-B-ACC1
- [ ] Ethernet0/1 on both switches are access ports in VLAN 10
- [ ] Port security enabled on SW-A-ACC1 E0/1 and SW-B-ACC1 E0/1
- [ ] show port-security interface on both secured ports shows secure-up, maximum 1, violation shutdown
- [ ] PC-A1 (10.10.10.10) can ping PC-B1 (10.10.10.20)

## If it doesn't work

- Confirm ports are in access mode with the correct VLAN before enabling port security
- Check that port-security is not applied to the trunk uplink
- Use show port-security and show port-security interface to confirm secure state

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Native VLAN or trunking mismatch resulting in hosts not learning MACs end-to-end
- Required user VLAN not present on one switch
- Host access port assigned to the wrong VLAN
- Port-security accidentally applied to the trunk instead of host ports

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ccna-port-security-1-enable-verify-on-access-ports
