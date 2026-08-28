# CCNA NTP Client: Sync to an Authoritative Server

Two IOS routers on a single /30. Configure R1 as NTP master (stratum 3) and R2 as a client. No routing protocols or static routes.

**2 nodes** (iol-xe) — runs on CML-Free  ·  about 25 minutes  ·  beginner

## What you'll configure

- Explain NTP roles: master (authoritative) vs. client and stratum behavior
- Configure an IOS router as an authoritative NTP master using ntp master
- Configure an IOS router as an NTP client using ntp server <ip>
- Verify NTP state with show ntp associations, show ntp status, and show
- Recognize that NTP convergence takes minutes and grading is based on

## Importing

In CML choose **Lab > Import** and pick `topology.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario

Your team is standardizing device time across a new lab site to ensure
consistent logs and easier incident correlation. You’re starting small: two
routers that directly connect on a point-to-point link. R1 will be the
authoritative time source for local devices (simulating a site that can
operate even if WAN time sources are unreachable). R2 will be the first NTP
client that learns time from R1.


The goal is to make the configuration deterministic: R1 will advertise
itself as a master at a specific stratum, and R2 will be configured to use
R1. You’ll verify using built-in NTP commands. Important: NTP takes minutes
to fully synchronize; for this lab the graded outcome is the configuration
itself and the expected association, not an immediate "synchronized" state.


### Prerequisites & Access

- Level and time: Beginner · ~25 minutes

- You will need: Cisco Modeling Labs, and room for 2 nodes (2 network
devices). That fits the 5-node limit on CML Free.

- Import `topology.yaml`, then configure the devices yourself — the
starter topology is deliberately unconfigured.


### Topology Walkthrough

- R1 and R2 are connected back-to-back on a single point-to-point Ethernet
link using the 10.0.0.0/30 network.

- R1 uses 10.0.0.1/30 on Ethernet0/0; R2 uses 10.0.0.2/30 on Ethernet0/0.

- There is no routing protocol and no static routes. All devices share one
directly connected subnet, so additional routing is unnecessary and out of
scope.

- R1 will be configured as an authoritative NTP master (stratum 3). R2 will
be configured as a client using R1 as its server.


### IP Addressing Plan

- Transit 10.0.0.0/30
  - R1 Ethernet0/0: 10.0.0.1/30
  - R2 Ethernet0/0: 10.0.0.2/30

### Tasks

1. Confirm Layer 3 adjacency
   - What: Ensure Ethernet0/0 on both routers has the correct IP/mask and is administratively up. Verify you can ping between 10.0.0.1 and 10.0.0.2.
   - Why: NTP requires simple IP reachability to the server.

2. Make R1 an authoritative NTP source
   - What: Configure R1 as a local authoritative master at a fixed stratum (use stratum 3).
   - Why: In lab settings or when upstream time is unavailable, a router can serve as a deterministic local reference. Clients will adopt stratum+1.

3. Point R2 at R1 as its NTP server
   - What: Configure R2 to use 10.0.0.1 as its NTP server.
   - Why: A client learns time from its server and will report one stratum below it (R2 should be stratum 4 after convergence).

4. Understand convergence behavior
   - What: Check association and status on R2. Expect to see 10.0.0.1 listed as a server. Synchronized state may take several minutes.
   - Why: NTP is conservative by design; the lab grades the deterministic configuration, not immediate lock.

### Verification

Run these checks on both routers:

- From R2, confirm IP reachability to R1 using ping 10.0.0.1.

- On R2:
  - show ntp associations — verify 10.0.0.1 appears as a server association.
  - show ntp status — after some time, expect to see that the clock is synchronized to 10.0.0.1 and the stratum reflects one higher than the master.
  - show clock — the displayed time should begin to align with R1 over time.
- On R1:
  - show ntp status — verify it is operating as a master (stratum 3) and not synchronized to any upstream source in this lab.

### Troubleshooting

- If R2 does not list 10.0.0.1 in show ntp associations:
  - Verify R2 has the correct server IP configured.
  - Confirm interfaces are up/up and that pings from R2 to 10.0.0.1 succeed.
  - Ensure R1 is configured as an authoritative master; otherwise R2 may have no valid source.
- If status is unsynchronized immediately after configuration:
  - Wait several minutes and recheck. NTP will not show synchronized instantly.
- If times seem far apart:
  - Remember that convergence is gradual and the lab grades configuration, not immediate time parity.

### Completion Checklist

Work through these before you call the lab done.

- [ ] Explain NTP roles: master (authoritative) vs. client and stratum
behavior

- [ ] Configure an IOS router as an authoritative NTP master using ntp
master <stratum>

- [ ] Configure an IOS router as an NTP client using ntp server <ip>

- [ ] Verify NTP state with show ntp associations, show ntp status, and show
clock

- [ ] Recognize that NTP convergence takes minutes and grading is based on
deterministic config

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] 'show ntp status' on R1 reports it is the authoritative reference clock at
- [ ] 'show ntp associations' on R2 lists 10.0.0.1 as a configured server
- [ ] R2 'show ntp associations' lists 10.0.0.1 as a server association
- [ ] R2 'show ntp status' shows reference to 10.0.0.1 (synchronization may
- [ ] Ping from R2 to 10.0.0.1 succeeds

## If it doesn't work

- Interface down or wrong IP/mask prevents reachability to the NTP server
- Missing ntp master on the server — clients have no valid source
- Client points to the wrong IP (typo or different interface)
- Clock appears unsynchronized because insufficient time has elapsed for NTP

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- Incorrect server address on the client prevents association
- Server lacks 'ntp master <stratum>' so clients never form a valid
- Interface shutdown or wrong mask breaks simple reachability
- Expecting immediate synchronization vs. allowing NTP time to settle

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ccna-ntp-client-sync-to-an-authoritative-server
