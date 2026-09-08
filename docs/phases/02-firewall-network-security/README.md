# Phase 2 — Firewall & Network Security

## Overview

This phase focuses on implementing firewall policies and network security controls using OPNsense.

The objective is to move from basic network connectivity to a controlled, security-oriented network architecture based on stateful firewalling, explicit access rules, outbound traffic restrictions, NAT policies, and least-privilege network access.

The lab network is isolated from the home network and uses OPNsense as its default gateway and firewall.

---

## Objectives

* Design a documented firewall security policy.
* Understand stateful firewall behaviour.
* Create and manage LAN firewall rules.
* Restrict unnecessary outbound traffic.
* Configure and document NAT policies.
* Test permitted and blocked network traffic.
* Document firewall decisions and their rationale.
* Implement least-privilege network access.

---

## Network Context

The lab network uses the following addressing scheme:

| Component    |         Address | Role                    |
| ------------ | --------------: | ----------------------- |
| OPNsense LAN |    `10.10.10.1` | Firewall / Gateway      |
| DC01         |   `10.10.10.10` | Domain Controller / DNS |
| Windows 11   |   `10.10.10.20` | Domain workstation      |
| Arch Linux   |   `10.10.10.30` | Linux workstation       |
| Lab Network  | `10.10.10.0/24` | Isolated lab network    |

All lab systems use OPNsense (`10.10.10.1`) as their default gateway.

The lab network is connected through the VirtualBox `LAB2` internal network.

---

# 1. Design Firewall Policy

The firewall policy will be designed according to the principle:

> Allow required traffic and deny unnecessary traffic.

Initial policy objectives:

| Source | Destination | Policy            | Rationale                                  |
| ------ | ----------- | ----------------- | ------------------------------------------ |
| LAB    | Internet    | Allow             | Required for updates and external services |
| LAB    | LAB         | Allow as required | Internal lab communication                 |
| LAB    | OPNsense    | Allow as required | Required network services                  |
| WAN    | LAB         | Deny              | Prevent unsolicited inbound access         |
| WAN    | OPNsense    | Deny              | Protect the firewall                       |

Firewall decisions will be based on:

* Source
* Destination
* Protocol
* Port
* Direction
* Business or technical requirement

Rules will be documented together with their security rationale.

---

# 2. Understand Stateful Firewall Behaviour

OPNsense provides stateful firewalling.

When an internal host establishes an allowed connection to an external destination, OPNsense maintains the state of that connection.

For example:

```text
Windows 11
10.10.10.20
      |
      | Outbound connection
      v
OPNsense
10.10.10.1
      |
      v
Internet
```

Once the connection is established and tracked, the corresponding return traffic is recognised as part of the existing connection.

This means that an explicit rule allowing unsolicited inbound traffic is not required simply to allow responses to connections initiated from the LAN.

The distinction between:

* New connections
* Established connections
* Return traffic

will be considered when designing the firewall rules.

---

# 3. Create LAN Rules

LAN firewall rules will control traffic originating from the `10.10.10.0/24` network.

Rules will be created according to actual lab requirements rather than allowing unrestricted traffic by default.

Examples of traffic that may require explicit consideration include:

* DNS
* HTTP/HTTPS
* Internal AD services
* Windows update traffic
* Linux package repositories
* Required communication between lab hosts

Rules will be ordered carefully because firewall rule order affects traffic evaluation.

### Evidence

Relevant firewall rules will be captured in:

```text
screenshots/
└── 03-lan-rules.png
```

---

# 4. Restrict Unnecessary Outbound Traffic

Outbound traffic will be reviewed to identify services that do not require Internet access.

The objective is to reduce unnecessary network exposure and apply least-privilege principles to outbound connectivity.

Potential restrictions may include:

* Unnecessary protocols
* Unnecessary ports
* Unnecessary external destinations
* Traffic that should remain inside the lab network

Restrictions will only be implemented where they do not interfere with required lab functionality.

Testing will be performed after each significant restriction.

---

# 5. Configure NAT Policies

NAT will be configured to allow the isolated lab network to access external networks through the OPNsense WAN interface.

The expected traffic flow is:

```text
LAB
10.10.10.0/24
      |
      v
OPNsense LAN
10.10.10.1
      |
      v
OPNsense WAN
      |
      v
VirtualBox NAT
      |
      v
Internet
```

OPNsense will perform the appropriate outbound NAT for the lab network.

NAT configuration will be reviewed to ensure that:

* Lab clients can reach required external destinations.
* Internal addressing is not exposed unnecessarily.
* NAT rules match the intended network architecture.

---

# 6. Test Blocked and Permitted Traffic

Firewall behaviour will be verified through controlled connectivity tests.

Tests will include both permitted and blocked traffic.

Examples:

### Permitted traffic

```text
LAB → Internet
LAB → DNS
LAB → Required internal services
```

### Blocked traffic

```text
WAN → LAB
Unauthorized service → LAB host
Unnecessary outbound traffic
```

Testing will be performed from the lab hosts where appropriate.

Results will be documented as evidence that firewall rules behave as intended.

---

# 7. Document Firewall Decisions

Each significant firewall rule will have a documented purpose.

The documentation will explain:

* What traffic is being controlled.
* Why the traffic is allowed or denied.
* Which systems are affected.
* Which protocol or port is involved.
* What security objective the rule supports.

Example:

| Rule                         | Decision | Reason                                  |
| ---------------------------- | -------- | --------------------------------------- |
| LAN → DNS                    | Allow    | Required for name resolution            |
| LAN → HTTPS                  | Allow    | Required for external services          |
| WAN → LAN                    | Deny     | Prevent unsolicited inbound connections |
| Unnecessary outbound traffic | Deny     | Reduce attack surface                   |

This ensures that firewall configuration remains understandable and maintainable.

---

# 8. Implement Least-Privilege Network Access

The final objective is to move from broad network access toward explicitly required access.

The principle is:

```text
Default
   ↓
Restrict
   ↓
Identify requirement
   ↓
Allow only required traffic
```

Network access will be evaluated based on actual requirements rather than convenience.

The resulting configuration should minimise:

* Unnecessary inbound access
* Unnecessary outbound access
* Unnecessary exposed services
* Excessive network trust

This approach will provide a foundation for future network segmentation, IDS/IPS, monitoring, and SIEM implementation.

---

# Verification

The following checks will be performed during this phase:

* [ ] Firewall policy documented
* [ ] Stateful firewall behaviour understood
* [ ] LAN rules configured
* [ ] Unnecessary outbound traffic reviewed
* [ ] NAT configuration verified
* [ ] Permitted traffic tested
* [ ] Blocked traffic tested
* [ ] Firewall decisions documented
* [ ] Least-privilege access implemented

---

# Screenshots

Planned evidence:

```text
screenshots/
├── 01-firewall-policy.png
├── 02-stateful-firewall.png
├── 03-lan-rules.png
├── 04-outbound-restrictions.png
├── 05-nat-policy.png
├── 06-firewall-tests.png
├── 07-firewall-decisions.png
└── 08-least-privilege.png
```

Screenshots should demonstrate configuration or verification rather than document every individual click.

Sensitive information such as passwords, private keys, tokens, or unnecessary personal information should not be included.

---

# Phase Deliverables

At the end of Phase 2, the lab should demonstrate:

* A documented firewall security policy.
* Understanding of stateful firewall operation.
* Controlled LAN access.
* Restricted unnecessary outbound traffic.
* Documented NAT behaviour.
* Verified permitted and blocked traffic.
* Documented firewall decisions.
* Least-privilege network access.

---

# Roadmap

```text
Phase 1 — Network Foundation
        ↓
Phase 2 — Firewall & Network Security
        ↓
Phase 3 — IDS/IPS
        ↓
Phase 4 — Windows Security & Active Directory
        ↓
Phase 5 — Monitoring & SIEM
        ↓
Phase 6 — Network Segmentation
        ↓
Future — Proxmox / Virtualization
```

---

## Status

**In Progress**
