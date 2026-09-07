# Phase 1 — Network Foundation

## Overview

The first phase of the laboratory establishes the underlying network infrastructure required by the rest of the project.

The main objective is to create an **isolated laboratory network** with controlled Internet access through OPNsense, while keeping the home network separate from the laboratory environment.

## Objectives

* Create an isolated VirtualBox network.
* Deploy OPNsense as the laboratory gateway.
* Configure WAN and LAN interfaces.
* Configure routing and NAT.
* Establish Internet connectivity for laboratory systems.
* Deploy the Active Directory domain controller.
* Configure internal DNS.
* Connect Windows 11 to the domain.
* Integrate Arch Linux with Active Directory.
* Verify connectivity between all systems.
* Document troubleshooting and configuration decisions.

---

# 1. Initial Network

The laboratory is hosted in VirtualBox.

The home network is:

```text
192.168.1.0/24
```

The laboratory uses a separate VirtualBox Internal Network:

```text
LAB
10.10.10.0/24
```

This separation prevents the laboratory systems from depending directly on the home LAN.

### Target architecture

```text
                         INTERNET
                            │
                     VirtualBox NAT
                            │
                            ▼
                     ┌─────────────┐
                     │  OPNsense   │
                     │     WAN     │
                     │ Firewall/NAT│
                     └──────┬──────┘
                            │
                       LAN .1
                            │
                         LAB
                    10.10.10.0/24
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
      ┌───▼────┐       ┌────▼────┐       ┌────▼────┐
      │  DC01  │       │ Windows │       │  Arch   │
      │  .10   │       │    11   │       │  .30    │
      │         │       │   .20   │       │         │
      └────────┘       └─────────┘       └─────────┘
```

---

# 2. VirtualBox Network

The laboratory uses two different types of VirtualBox networking.

### OPNsense WAN

```text
Adapter 1
    │
    └── NAT
```

This interface provides OPNsense with upstream connectivity.

### OPNsense LAN

```text
Adapter 2
    │
    └── Internal Network: LAB2
```

This interface connects OPNsense to the isolated laboratory network.

The final interface mapping is:

| VirtualBox Adapter | Network | OPNsense Interface | Role |
| ------------------ | ------- | ------------------ | ---- |
| Adapter 1          | NAT     | `em0`              | WAN  |
| Adapter 2          | `LAB`  | `em1`              | LAN  |

This mapping is important because the physical/virtual interface assignment must correspond to the intended WAN/LAN roles.

---

# 3. OPNsense

OPNsense is used as the central network device for the laboratory.

Its responsibilities are:

* Default gateway
* Routing
* Stateful firewall
* NAT
* Internet access for the laboratory

### LAN configuration

```text
Interface: em1
IP:        10.10.10.1/24
```

All laboratory systems use this address as their default gateway.

### WAN

The WAN interface is connected to VirtualBox NAT:

```text
Interface: em0
Network:   VirtualBox NAT
```

The WAN side receives upstream connectivity from VirtualBox.

---

# 4. Routing and NAT

The laboratory clients do not connect directly to the home network or directly to VirtualBox NAT.

Their traffic follows this path:

```text
LAB Client
    │
    │ 10.10.10.1
    ▼
OPNsense LAN
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

OPNsense performs the routing and NAT required to provide Internet access to the isolated `LAB2` network.

The Active Directory server does **not** act as the laboratory router.

---

# 5. Addressing

| Host         |    IP Address |      Gateway |           DNS |
| ------------ | ------------: | -----------: | ------------: |
| OPNsense LAN |  `10.10.10.1` |            — |             — |
| DC01         | `10.10.10.10` | `10.10.10.1` | `10.10.10.10` |
| Windows 11   | `10.10.10.20` | `10.10.10.1` | `10.10.10.10` |
| Arch Linux   | `10.10.10.30` | `10.10.10.1` | `10.10.10.10` |

The domain is:

```text
LAB.LOCAL
```

DC01 provides the internal DNS service used by the laboratory clients.

---

# 6. Active Directory and DNS

DC01 provides the core Windows infrastructure services:

```text
DC01
10.10.10.10
    │
    ├── Active Directory
    ├── DNS
    └── Kerberos KDC
```

The laboratory clients use DC01 as their DNS server rather than using an external DNS server directly.

External DNS queries are forwarded by the domain DNS infrastructure.

This allows internal names such as:

```text
lab.local
```

to coexist with normal Internet DNS resolution.

---

# 7. Windows 11

Windows 11 was configured as a client of the `LAB.LOCAL` domain.

Network configuration:

```text
IP:      10.10.10.20
Gateway: 10.10.10.1
DNS:     10.10.10.10
```

Connectivity was verified through:

```text
Windows 11
     │
     ├──► 10.10.10.1
     ├──► 1.1.1.1
     └──► google.com
```

The successful tests confirmed:

* LAN connectivity.
* Routing through OPNsense.
* Internet connectivity.
* DNS resolution.

---

# 8. Arch Linux

Arch Linux was configured as a Linux client on the `LAB` network.

Network configuration:

```text
IP:      10.10.10.30
Gateway: 10.10.10.1
DNS:     10.10.10.10
```

Arch Linux uses `systemd-networkd` for network configuration.

The final configuration provides:

```text
Arch Linux
    │
    ├── Gateway → 10.10.10.1
    │
    └── DNS → 10.10.10.10
```

Connectivity was verified through:

```text
ping 10.10.10.1
ping 1.1.1.1
nslookup google.com
```

Arch Linux was also integrated with Active Directory using Kerberos and Samba.

Detailed Linux/AD integration is documented separately in the relevant phase documentation.

---

# 9. Verification

The final network was tested from each laboratory system.

## Gateway connectivity

```text
DC01       → 10.10.10.1
Windows 11 → 10.10.10.1
Arch Linux → 10.10.10.1
```

All clients successfully reached the OPNsense LAN interface.

## Internet connectivity

External IP connectivity was tested using:

```text
1.1.1.1
```

This confirmed that routing and NAT through OPNsense were working.

## DNS

External DNS resolution was tested using:

```text
nslookup google.com
```

Internal DNS was tested using:

```text
nslookup lab.local
```

The tests confirmed that:

```text
Clients
   │
   ▼
DC01 DNS
   │
   ├── Internal domain resolution
   │
   └── External DNS forwarding
```

was functioning correctly.

---

# 10. Troubleshooting

## OPNsense WAN/LAN interface mismatch

During the initial configuration, the OPNsense interface roles did not match the VirtualBox adapter assignments.

The relevant mapping was:

```text
VirtualBox Adapter 1 → NAT → em0
VirtualBox Adapter 2 → LAB → em1
```

However, the LAN role had initially been assigned to the wrong OPNsense interface.

As a result, the laboratory clients could not reach the expected gateway.

### Diagnosis

The problem was investigated at Layer 2/Layer 3 rather than assuming a firewall or routing problem.

Interface information and ARP behaviour were inspected from OPNsense.

The absence of the expected ARP activity on the interface indicated that the traffic was arriving on a different virtual interface than the one assigned to the LAN.

### Resolution

The OPNsense interface assignments were corrected:

```text
WAN → em0 → VirtualBox NAT
LAN → em1 → LAB
```

After correcting the mapping, the laboratory clients were able to reach:

```text
10.10.10.1
```

and subsequently access the Internet.

---

# 11. Configuration Persistence

An additional issue was encountered where the OPNsense LAN configuration did not persist correctly after reboot.

The investigation identified unnecessary virtual hardware attached to the virtual machine, including a duplicate virtual disk and installation media.

After removing the unnecessary devices and rebooting, the intended configuration persisted correctly.

Final LAN configuration:

```text
10.10.10.1/24
```

---

# 12. Final State

The network foundation is now operational.

```text
                         INTERNET
                            │
                     VirtualBox NAT
                            │
                         OPNsense
                    WAN: em0 / NAT
                    LAN: em1 / 10.10.10.1
                            │
                           LAB
                     10.10.10.0/24
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
       DC01 .10          W11 .20           Arch .30
          │                 │                 │
       AD / DNS          AD Client       Linux AD Client
       Kerberos                          Kerberos/Samba
```

### Current capabilities

* Isolated laboratory network.
* Centralised routing through OPNsense.
* Stateful firewall.
* NAT-based Internet access.
* Active Directory domain.
* Internal DNS.
* Kerberos/KDC provided by the domain controller.
* Windows domain client.
* Linux/Active Directory integration.
* Internet connectivity from all laboratory clients.
* Documented troubleshooting process.

---

# Next Phase

The next phase will focus on **firewall and network security**.

Planned work:

* Define explicit firewall policies.
* Understand stateful filtering.
* Restrict unnecessary traffic.
* Test allowed and denied connections.
* Review NAT behaviour.
* Begin applying least-privilege principles to the network.
