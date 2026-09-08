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

The firewall policy follows the principle:

> Allow required traffic and deny unnecessary traffic.

The initial policy objectives are:

| Source | Destination | Policy            | Rationale                                      |
| ------ | ----------- | ----------------- | ---------------------------------------------- |
| LAB    | Internet    | Allow             | Required for updates and external services     |
| LAB    | DC01        | Allow as required | Required for DNS and Active Directory services |
| LAB    | OPNsense    | Allow as required | Required network services                      |
| WAN    | LAB         | Deny              | Prevent unsolicited inbound access             |
| WAN    | OPNsense    | Deny              | Protect the firewall                           |

Firewall decisions are based on:

* Source
* Destination
* Protocol
* Port
* Direction
* Technical requirement

Rules are documented according to their purpose and security rationale.

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

is considered when designing and evaluating firewall behaviour.

---

# 3. Create LAN Rules

LAN firewall rules control traffic originating from the `10.10.10.0/24` network.

The default `Allow LAN to any` rule was disabled in favour of explicit access rules.

The following traffic is explicitly permitted where required by the lab:

| Service                  | Destination          | Protocol / Port |
| ------------------------ | -------------------- | --------------- |
| DNS                      | DC01 (`10.10.10.10`) | TCP/UDP 53      |
| Kerberos                 | DC01                 | TCP/UDP 88      |
| LDAP                     | DC01                 | TCP/UDP 389     |
| SMB                      | DC01                 | TCP 445         |
| RPC Endpoint Mapper      | DC01                 | TCP 135         |
| RPC Dynamic              | DC01                 | TCP 49152-65535 |
| Global Catalog           | DC01                 | TCP 3268        |
| Global Catalog SSL       | DC01                 | TCP 3269        |
| Kerberos Password Change | DC01                 | TCP/UDP 464     |
| HTTP                     | Internet             | TCP 80          |
| HTTPS                    | Internet             | TCP 443         |

Rules are evaluated from top to bottom. Specific ALLOW rules are therefore placed before the final catch-all BLOCK rule.

### Evidence

Relevant firewall configuration is captured in:

```text
screenshots/
└── 03-lan-rules.png
```

---

# 4. Restrict Unnecessary Outbound Traffic

The default unrestricted outbound rule was disabled:

```text
Default allow LAN to any
```

A catch-all rule was enabled to block traffic that is not explicitly permitted:

```text
Block unnecessary outbound traffic
```

The rule uses:

```text
Source:      LAN net
Destination: any
Protocol:    any
Action:      Block
```

This rule is placed at the **bottom** of the LAN rule set, after all required ALLOW rules.

The resulting policy is:

```text
Specific required traffic
        ↓
      ALLOW
        ↓
Unmatched traffic
        ↓
      BLOCK
```

This reduces unnecessary outbound network access while preserving required connectivity for Active Directory services and external HTTP/HTTPS traffic.

### Evidence

```text
screenshots/
└── 04-outbound-restrictions.png
```

---

# 5. Configure NAT Policies

Outbound NAT was reviewed under:

**Firewall → NAT → Source NAT**

OPNsense provides automatically generated Source NAT rules for the lab's outbound connectivity through the WAN interface.

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

No additional manual outbound NAT rule was required for the current lab architecture.

The NAT configuration was reviewed to ensure that:

* Lab clients can reach required external destinations.
* Internal lab addressing is translated before reaching the upstream network.
* NAT behaviour matches the intended network architecture.

### Evidence

```text
screenshots/
└── 05-nat-policy.png
```

---

# 6. Test Blocked and Permitted Traffic

Firewall behaviour is verified through controlled connectivity tests from the lab hosts.

Tests cover both permitted and blocked traffic.

### Permitted traffic

```text
LAB → Internet HTTPS
LAB → DC01 DNS
LAB → Required Active Directory services
```

Example tests:

```bash
curl -4 -I --connect-timeout 5 https://example.com
```


### Blocked traffic

Traffic that is not explicitly permitted is expected to be blocked by the final catch-all rule.

Example:

```bash
nc -zv -w 3 1.1.1.1 25
```

Firewall logs can be reviewed through OPNsense Live View to verify the corresponding decisions.

### Evidence

```text
screenshots/
└── 06-firewall-tests.png
```

---

# 7. Document Firewall Decisions

Each significant firewall rule has a documented purpose.

The firewall follows a simple security principle:

> Allow required traffic and deny unnecessary traffic.

| Rule                         | Decision | Reason                                  |
| ---------------------------- | -------- | --------------------------------------- |
| LAN → DC01 DNS               | Allow    | Required for name resolution            |
| LAN → DC01 Kerberos          | Allow    | Required for AD authentication          |
| LAN → DC01 LDAP              | Allow    | Required for directory services         |
| LAN → DC01 SMB               | Allow    | Required for AD/SMB communication       |
| LAN → DC01 RPC               | Allow    | Required for RPC-based AD communication |
| LAN → DC01 Global Catalog    | Allow    | Required for directory queries          |
| LAN → Internet HTTP          | Allow    | Required for external HTTP services     |
| LAN → Internet HTTPS         | Allow    | Required for secure external services   |
| Unnecessary outbound traffic | Deny     | Reduce unnecessary network exposure     |

The firewall configuration is therefore based on explicit technical requirements rather than unrestricted connectivity.

---

# 8. Implement Least-Privilege Network Access

The final configuration applies least-privilege principles to network access.

Instead of allowing unrestricted LAN connectivity, traffic is evaluated according to its actual requirement.

```text
Default
   ↓
Restrict
   ↓
Identify requirement
   ↓
Allow only required traffic
   ↓
Block everything else
```

The resulting configuration minimises:

* Unnecessary inbound access
* Unnecessary outbound access
* Unnecessary exposed services
* Excessive network trust

The lab therefore uses explicit ALLOW rules for required services followed by a final catch-all BLOCK rule.

This provides a foundation for future network segmentation, IDS/IPS, monitoring, and SIEM implementation.

### Evidence

```text
screenshots/
└── 08-least-privilege.png
```

---

# Verification

The following checks were performed during this phase:

* [x] Firewall policy documented
* [x] Stateful firewall behaviour understood
* [x] LAN rules configured
* [x] Unnecessary outbound traffic reviewed
* [x] NAT configuration verified
* [x] Permitted traffic tested
* [x] Blocked traffic tested
* [x] Firewall decisions documented
* [x] Least-privilege access implemented

---

# Troubleshooting

## Catch-all block rule preventing permitted traffic

### Symptom

After enabling the `Block unnecessary outbound traffic` rule, Arch Linux could resolve DNS and communicate with DC01, but HTTPS connections to the Internet timed out.

For example:

```text
DNS resolution:      Working
Arch → DC01:         Working
Arch → Internet:     Blocked
```

### Cause

The catch-all BLOCK rule had been placed above the specific ALLOW rules.

OPNsense evaluates firewall rules from top to bottom. Therefore, the general BLOCK rule matched the traffic before the corresponding ALLOW rule could be evaluated.

### Resolution

The `Block unnecessary outbound traffic` rule was moved to the bottom of the LAN rule set, below all required ALLOW rules.

The default `Allow LAN to any` rule remained disabled.

### Lesson learned

Catch-all deny rules should be placed **after specific allow rules** so that required exceptions are evaluated first.

This reinforced the importance of both rule design and rule ordering when implementing least-privilege firewall policies.

---

# Screenshots

Planned evidence:

```text
screenshots/
├── 01-firewall-baseline.png
├── 02-stateful-firewall.png
├── 03-lan-rules.png
├── 04-outbound-restrictions.png
├── 05-nat-policy.png
├── 06-firewall-tests.png
```

The firewall decisions documented in section 7 are represented through the rule configuration and the outbound restriction evidence rather than requiring a duplicate screenshot.

Screenshots should demonstrate configuration or verification rather than document every individual click.

Sensitive information such as passwords, private keys, tokens, or unnecessary personal information should not be included.

---

# Phase Deliverables

At the end of Phase 2, the lab demonstrates:

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

**Complete**
