# CCNA: SSH Access Fundamentals on R1

Management LAN addressing is present. Configure SSH access and local authentication on R1; verify from ADMIN.

**2 nodes** (alpine, iol-xe) — runs on CML-Free  ·  about 35 minutes  ·  beginner

## What you'll configure

- Enable SSH on a Cisco IOS router using only the necessary management-plane configuration lines
- Create and secure a local administrator account with privilege 15 and an enable secret
- Restrict remote access to SSH only and authenticate using the local user database
- Verify SSH service readiness from configuration and test an SSH login from a Linux host

## Importing

In CML choose **Lab > Import** and pick `topology.yaml`, or use **Add Lab from Repository** if you have this
repository configured as a lab repository. Devices boot with a starting configuration — hostnames and the
addressing that is already in place — so you begin on the tasks rather than on setup. The same instructions
below are attached to the lab's Notes in CML, so they travel with the topology.

## Tasks

### Scenario
Your team is standardizing secure management access to all network devices. A new branch router (R1) has been racked and cabled to a small management LAN. The operations requirement is clear: all remote management must use SSH with local authentication, and Telnet must not be offered. You have console access today but must enable deterministic SSH access so that an administrator workstation (ADMIN) on the same LAN can reach the router using a named account.

The security policy for Day 1 requires:
- A non-default hostname and a domain name
- A local admin account with privilege 15 protected by a secret (hashed)
- An enable secret for fallback and escalation
- VTY lines limited to SSH and tied to the local user database (login local)
- RSA key generation (an exec step) to activate the SSH service

### Prerequisites & Access
- Level and time: Beginner · ~35 minutes
- You will need: Cisco Modeling Labs, and room for 2 nodes (1 network device, 1 host). That fits the 5-node limit on CML Free.
- Import `topology.yaml`, then configure the devices yourself — the starter topology is deliberately unconfigured.

### Access & credentials

Open a device's console from the CML topology view (click the node, then **Console**).

- **ADMIN** (Alpine hosts) — username `cisco` / password `cisco`. These are the CML image defaults; the lab sets no password of its own.

These are the credentials the starter topology ships with. If a prompt rejects them, the device has not finished booting — wait for the console to settle and try again.

### Topology Walkthrough
- R1 (iol-xe) provides the default gateway for the management LAN on Ethernet0/0.
- ADMIN (alpine Linux) is the administrator’s workstation on the same management LAN.
- No routing protocols, VLANs, or transit ACLs are used in this lab. We focus strictly on management-plane hardening on a single device.

Interfaces and addressing:
- R1 Ethernet0/0: 10.0.0.1/24 (gateway for the management LAN)
- ADMIN eth0: 10.0.0.10/24 with default gateway 10.0.0.1

### IP Addressing Plan
- Management LAN: 10.0.0.0/24
  - R1 (gateway): 10.0.0.1/24
  - ADMIN: 10.0.0.10/24 (default route to 10.0.0.1)

### Tasks
1. Establish a non-default identity for the router
   - What: Ensure the router uses a distinct hostname (R1) and a domain name (example.local).
   - Why: Hostname + domain name enable RSA key generation for SSH and positively identify the device in logs and SSH fingerprints.

2. Create a secure local administrator account
   - What: Add a local username “admin” with privilege 15 using a secret (hashed). Also configure an enable secret for privileged EXEC.
   - Why: Local authentication with a hashed secret protects credentials; privilege 15 grants full administrative rights. The enable secret is your secure fallback.

3. Restrict remote access to SSH only and bind VTY to local auth
   - What: On line vty 0 4, restrict transport to SSH and require local authentication.
   - Why: This both disables Telnet and ensures SSH logins authenticate against the local user database.

4. Generate RSA keys (exec mode)
   - What: Generate the RSA key pair for SSH.
   - Why: SSH cannot start without keys. You’ll run an exec command to create them. This is operational and not a persistent config line.
   - Watch for: Use a strong modulus (e.g., 2048) during key generation.

5. Confirm service readiness from the router
   - What: Use show commands to validate SSH state and VTY configuration.
   - Why: Verifying from the device ensures deterministic readiness regardless of client issues.

6. Test SSH from the ADMIN workstation
   - What: Initiate an SSH session to 10.0.0.1 using the “admin” account.
   - Why: End-to-end validation proves that your configuration enables secure, functional remote management.

### Verification
Run these checks to confirm success:
- From R1:
  - show run | section line vty — Expect to see transport input ssh and login local under line vty 0 4.
  - show ip ssh — Expect SSH enabled with a version indication and key parameters after key generation.
- From ADMIN:
  - ping 10.0.0.1 — Expect replies from the gateway.
  - ssh admin@10.0.0.1 — Expect an SSH key fingerprint prompt on first connect, followed by a router prompt. After login, show privilege should indicate level 15; if not, use enable and the enable secret.

### Troubleshooting
- No SSH service? Confirm both a non-default hostname and ip domain-name exist, then run crypto key generate rsa. Without keys, SSH cannot start.
- Password rejected? Verify the local username exists with a secret (not a simple password) and that line vty 0 4 uses login local.
- Still seeing Telnet? Confirm line vty 0 4 has transport input ssh (this implicitly disables Telnet on modern IOS; on some releases you may see ssh only in the list).
- Stuck at low privilege? Check show privilege after login. If not at level 15, ensure the username includes privilege 15, and confirm an enable secret exists for escalation if needed.
- Basic reachability: From ADMIN, ping 10.0.0.1. If it fails, check ADMIN’s IP, mask, and default route. Ensure R1 Ethernet0/0 is up/up with the correct IP.

### Completion Checklist
Work through these before you call the lab done.
- [ ] Enable SSH on a Cisco IOS router using only the necessary management-plane configuration lines
- [ ] Create and secure a local administrator account with privilege 15 and an enable secret
- [ ] Restrict remote access to SSH only and authenticate using the local user database
- [ ] Verify SSH service readiness from configuration and test an SSH login from a Linux host

## Verifying your work

Each of these is something you can prove from the device before calling the lab done.

- [ ] R1 has a non-default hostname: 'hostname R1' is present
- [ ] 'ip domain-name example.local' is configured
- [ ] A local admin account exists with a secret: 'username admin privilege 15 secret ...'
- [ ] 'enable secret ...' is present
- [ ] VTY lines are SSH-only and use local authentication: 'line vty 0 4' with 'transport input ssh' and 'login local'
- [ ] ADMIN can 'ssh admin@10.0.0.1' and reach a router prompt, verifying remote access works

## If it doesn't work

- If SSH fails to start, confirm both a non-default hostname and an ip domain-name are configured; both are required to generate RSA keys
- If the SSH connection is refused or prompts for a password repeatedly, verify line vty 0 4 has transport input ssh and login local
- If you can ping R1 but SSH still fails, ensure you have generated RSA keys with crypto key generate rsa
- If you log in but land at low privilege, check the configured username privilege level and the need to use enable (and that an enable secret exists)
- If DNS lookups slow down CLI, confirm you typed the correct SSH command (ssh admin@10.0.0.1) rather than a bare hostname that triggers DNS

Once it works, these are worth breaking on purpose — each one produces a different symptom:

- SSH fails to start because no RSA keys exist: verify hostname and domain name, then generate keys
- VTY misconfiguration blocks logins: missing 'login local' causes login failures even with a valid user
- VTY allows Telnet: missing or incorrect 'transport input ssh' leaves Telnet exposed
- Privilege mismatch: user lands at level 1 due to missing privilege 15 on the username or using a plain password instead of secret
- Connectivity symptom: ADMIN can’t reach 10.0.0.1 due to incorrect host IP/default route or R1’s interface is down

---

Contributed by Goldfish Networks — https://goldfishnetworks.com/archive/ccna-ssh-access-fundamentals-on-r1
