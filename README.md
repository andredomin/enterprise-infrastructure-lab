# enterprise-infrastructure-lab
Enterprise-style homelab for learning networking, Windows/Linux infrastructure, Active Directory, cybersecurity, monitoring, and virtualization.

## Objectives

* Manage Windows infrastructure.
* Design and manage isolated networks.
* Work with Active Directory and DNS.
* Integrate Linux systems with Active Directory.
* Automate tasks using PowerShell.
* Learn firewall and routing administration.
* Implement network monitoring and security.
* Work with IDS/IPS and SIEM solutions.
* Practice hardening, auditing, and incident response.
* Document procedures and troubleshooting professionally.


---

# Architecture

The laboratory is built around **OPNsense as the network gateway, firewall and NAT device**.

Active Directory is provided by `DC01`, while Windows 11 and Arch Linux operate as clients of the domain.

```text
                         INTERNET
                            │
                     VirtualBox NAT
                            │
                       OPNsense
                    WAN / Firewall / NAT
                            │
                       LAN: 10.10.10.1
                            │
                           LAB
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
      ┌───▼────┐       ┌────▼────┐       ┌────▼────┐
      │  DC01  │       │ Windows │       │  Arch   │
      │        │       │    11   │       │  Linux  │
      │ .10    │       │  .20    │       │  .30    │
      └────────┘       └─────────┘       └─────────┘
          │
     Active Directory
          DNS
     Kerberos / KDC
```

The editable architecture diagram is available in:

```text
docs/architecture/network-topology.drawio
```

A rendered version is available in:

```text
docs/architecture/network-topology.png
```

## Network

| Component    |         Address | Role                                  |
| ------------ | --------------: | ------------------------------------- |
| OPNsense LAN |    `10.10.10.1` | Gateway / Firewall / NAT              |
| DC01         |   `10.10.10.10` | Active Directory / DNS / Kerberos KDC |
| Windows 11   |   `10.10.10.20` | Domain Client                         |
| Arch Linux   |   `10.10.10.30` | Linux AD Client                       |
| LAB          | `10.10.10.0/24` | Isolated Laboratory Network           |

### Network design

The laboratory uses a VirtualBox **Internal Network (`LAB2`)** for the internal infrastructure.

The home network is not used as the laboratory's gateway.

Internet connectivity follows:

```text
LAB clients
    │
    ▼
OPNsense LAN
10.10.10.1
    │
    ▼
OPNsense WAN
    │
    ▼
VirtualBox NAT
    │
    ▼
Internet
```

OPNsense performs the routing, firewalling and NAT required for the laboratory's Internet access.

---

# Active Directory

The Windows infrastructure is based around the `LAB.LOCAL` Active Directory domain.

### DC01

```text
Hostname: DC01
IP:       10.10.10.10
Domain:   LAB.LOCAL
```

DC01 provides:

* Active Directory Domain Services
* DNS
* Kerberos KDC
* Domain identity management

The Kerberos KDC is provided by the domain controller as part of the Active Directory infrastructure; there is no separate Kerberos server in the lab.

---

# Clients

## Windows 11

Windows 11 is configured as a domain client of:

```text
LAB.LOCAL
```

It uses:

```text
Gateway: 10.10.10.1
DNS:     10.10.10.10
```

Windows uses the native Windows mechanisms for Active Directory integration.

---

## Arch Linux

Arch Linux is integrated with the Active Directory environment using:

* DNS
* Kerberos
* Samba

Arch uses the domain controller as its DNS server:

```text
DNS: 10.10.10.10
```

Kerberos configuration:

```text
Realm: LAB.LOCAL
KDC:   10.10.10.10
```

Samba is used for interoperability with Active Directory and for working with domain identities from the Linux environment.

> **Note:** Samba is documented here as part of the Linux/Active Directory integration. It is not being presented as a file server implementation for this phase.

---

# Roadmap

The laboratory will be developed progressively. Each phase will introduce new infrastructure, configuration and security concepts.

## Phase 1 — Network Foundation

* [x] Create isolated VirtualBox laboratory network
* [x] Deploy OPNsense
* [x] Configure OPNsense WAN
* [x] Configure OPNsense LAN
* [x] Configure `LAB` network
* [x] Configure laboratory gateway
* [x] Configure Internet access through OPNsense
* [x] Configure Active Directory
* [x] Configure DNS
* [x] Configure Windows 11 domain client
* [x] Integrate Arch Linux with Active Directory
* [x] Configure Kerberos
* [x] Configure Samba
* [x] Verify DNS resolution
* [x] Verify Internet connectivity
* [x] Document troubleshooting

## Phase 2 — Firewall & Network Security

* [ ] Design firewall policy
* [ ] Understand stateful firewall behaviour
* [ ] Create LAN rules
* [ ] Restrict unnecessary outbound traffic
* [ ] Configure NAT policies
* [ ] Test blocked and permitted traffic
* [ ] Document firewall decisions
* [ ] Implement least-privilege network access

## Phase 3 — IDS / IPS

* [ ] Deploy Suricata
* [ ] Configure network monitoring
* [ ] Configure IDS rules
* [ ] Test detection in the isolated laboratory
* [ ] Analyse alerts
* [ ] Understand false positives
* [ ] Configure IPS policies
* [ ] Document detection and response

## Phase 4 — Windows Security & Active Directory

* [x] Create organisational structure
* [x] Create users and groups
* [x] Design Group Policies
* [x] Implement security baselines
* [x] Configure account policies
* [x] Configure Windows auditing
* [x] Analyse Windows Event Logs
* [x] Review privileged accounts
* [x] Apply hardening measures
* [x] Document AD security decisions

## Phase 5 — Monitoring & SIEM

* [ ] Centralise infrastructure logs
* [ ] Collect Windows Event Logs
* [ ] Collect firewall logs
* [ ] Collect Linux logs
* [ ] Deploy a SIEM
* [ ] Create detection rules
* [ ] Build dashboards
* [ ] Correlate security events
* [ ] Investigate simulated incidents
* [ ] Document incident investigation

## Phase 6 — Network Segmentation

Expand the laboratory from a single isolated network into a segmented enterprise-style environment.

Planned VLANs:

| VLAN | Name       | Purpose                             |
| ---: | ---------- | ----------------------------------- |
|   10 | MANAGEMENT | Infrastructure management           |
|   20 | SERVERS    | Servers and infrastructure services |
|   30 | USERS      | Windows/Linux clients               |
|   40 | SECURITY   | IDS/IPS and security monitoring     |
|   50 | LAB        | Experimental systems                |

Goals:

* [ ] Configure VLANs
* [ ] Implement inter-VLAN routing
* [ ] Create firewall policies between VLANs
* [ ] Isolate management traffic
* [ ] Isolate security infrastructure
* [ ] Test segmentation
* [ ] Document the security model

## Future — Proxmox Infrastructure

The long-term goal is to move toward a more complete virtualised infrastructure based on Proxmox.

Planned concepts:

* [ ] Proxmox cluster
* [ ] High Availability
* [ ] Redundant storage
* [ ] Network segmentation
* [ ] Centralised monitoring
* [ ] Backup infrastructure
* [ ] Additional services

---

# Documentation

Each phase will have its own documentation under:

```text
docs/phases/
```

Example:

```text
docs/
├── architecture/
│   ├── network-topology.drawio
│   └── network-topology.png
│
└── phases/
    ├── 01-network-foundation
    ├── 02-firewall-network-security
    ├── 03-IDS/IPS
    ├── 04-windows-security-and-AD
    ├── 05-monitoring
    └── 06-siem
```

Documentation will focus not only on the final configuration, but also on the reasoning behind each implementation.

Where relevant, troubleshooting will follow this structure:

```text
Problem
   ↓
Initial hypothesis
   ↓
Tests
   ↓
Diagnosis
   ↓
Solution
   ↓
Verification
```

This is intended to make the repository useful as both a **technical portfolio and a personal knowledge base**.

---

# Scripts

Automation scripts will be stored under:

```text
scripts/
```

Planned structure:

```text
scripts/
├── powershell/
│   ├── active-directory/
│   ├── users/
│   ├── gpo/
│   └── maintenance/
│
└── linux/
    ├── samba/
    ├── kerberos/
    └── systemd/
```

Scripts will be added as they are developed and tested in the laboratory.

---

# Troubleshooting

Important troubleshooting cases will be documented separately under:

```text
docs/troubleshooting/
```

Examples include:

* OPNsense interface mapping
* DNS resolution
* Active Directory connectivity
* Kerberos authentication
* Linux/AD integration
* Network routing
* Firewall behaviour

The objective is to preserve the reasoning behind the solution rather than only documenting the final command or configuration.

---

# Repository Structure

Current repository structure:

```text
windows-infrastructure-lab/
│
├── README.md
│
├── docs/
│   ├── architecture/
│   │   ├── network-topology.drawio
│   │   └── network-topology.png
│   │
│   └── phases/
│       └── 01-network-foundation
│
├── scripts/
│   └── powershell/
│
├── .gitignore
└── CHANGELOG.md
```

The repository will grow organically as new infrastructure is implemented.

---

# Philosophy

This project is intentionally built as a **hands-on laboratory**.

The objective is not simply to reproduce a predefined configuration, but to understand:

* how the components interact,
* why a configuration is required,
* how failures can be diagnosed,
* how infrastructure can be secured,
* and how the entire environment can be documented and maintained.

The laboratory will therefore include both **successful implementations and documented failures**.

> Build it. Break it. Diagnose it. Fix it. Document it.
