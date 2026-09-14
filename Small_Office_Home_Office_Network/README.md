# SOHO Network Implementation Project — Multi-Site Edition

## 📌 Project Overview

This project began as a single-site SOHO (Small Office / Home Office) network and has been expanded into a **two-site network** built collaboratively by two CCNA students. Each member independently designed, configured, and secured their own LAN in Cisco Packet Tracer, and the two sites were then interconnected over a WAN link to form one combined topology.

**Objective:** Design and configure two independently functioning LANs — each with DHCP, structured IP addressing, and switch/router security — and connect them over a WAN link to simulate a small multi-site organization (e.g. a head office and a branch office).

| Site | Lead | Design Style | Address Block |
|------|------|--------------|----------------|
| **Site A — Main Office** | Clifford Karimi | Flat LAN + NAT/PAT to internet | `192.168.1.0/24` |
| **Site B — Branch Office** | Mwas | VLAN-segmented LAN with VLSM | `192.168.10.0/24` |

---

## 🗺️ Topology Diagrams

- **Site A (Main Office)** — router, core switch, department switches, access point, DHCP server, and internet-facing WAN interface.
- **Site B (Branch Office)** — router, core switch, three VLAN-segmented department switches (Admin, Emergency Room/general dept, Radiology/archives), and a DHCP server.
- **Combined Topology** — both site routers connected via a WAN link, forming a single multi-site network.

> Add the topology screenshots to an `/assets` folder in the repo and reference them here, e.g.:
> `![Site A Topology](assets/site-a-topology.jpg)`
> `![Site B Topology](assets/site-b-topology.jpg)`
> `![Combined Topology](assets/combined-topology.jpg)`

---

## 🏢 Site A — Main Office (Clifford Karimi)

### IP Addressing Scheme

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | G0/0 (LAN) | 192.168.1.1 | 255.255.255.0 | N/A |
| R1 | G0/1 (WAN) | 203.0.113.2 | 255.255.255.252 | 203.0.113.1 |
| SW-Main | VLAN 1 | 192.168.1.13 | 255.255.255.0 | 192.168.1.1 |
| SW-Sales | VLAN 1 | 192.168.1.12 | 255.255.255.0 | 192.168.1.1 |
| SW-Finance | VLAN 1 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| Finance_PC | DHCP | DHCP Assigned | 255.255.255.0 | 192.168.1.2 |
| Sales_PC | DHCP | DHCP Assigned | 255.255.255.0 | 192.168.1.3 |
| IT_PC | DHCP | DHCP Assigned | 255.255.255.0 | 192.168.1.4 |

> **DHCP Pool:** 192.168.1.2 – 192.168.1.254
> **Reserved (excluded):** 192.168.1.11 – 192.168.1.20 (maintenance/static devices)

### Security Configuration

| Feature | Status |
|---------|--------|
| Local username/password (all devices) | ✅ Configured |
| Enable secret (privileged mode) | ✅ Configured |
| Console & VTY authentication | ✅ Configured |
| SSH only (Telnet disabled) | ✅ Configured — SSH v2, 2048-bit RSA key |
| Password encryption (`service password-encryption`) | ✅ Configured |
| SSH brute-force retry limiting | ✅ Configured |
| AAA / RADIUS-TACACS+ | ⬜ Future enhancement |
| ACLs restricting SSH to specific IPs | ⬜ Future enhancement |

**Access credentials** (lab use only — see full security note below):
Username `netadmin`, password `letmein`, used consistently across R1, SW-Main, SW-Sales, and SW-Finance for console, privileged EXEC, and SSH access.

### Troubleshooting Reference

| Symptom | Possible Cause | Solution |
|---------|---------------|----------|
| PC doesn't get IP address | DHCP pool missing/disabled | `show ip dhcp binding`, `show run \| section dhcp` |
| Can't ping default gateway | Wrong IP config | `ipconfig`, `show ip interface brief` |
| No internet access | Missing default route or NAT | `show ip route`, `show ip nat statistics` |
| Switch unreachable | Management VLAN IP not set | `show ip interface brief` on VLAN 1 |
| NAT not translating | Wrong inside/outside interfaces | Confirm `ip nat inside` / `ip nat outside` |
| Can't resolve domain names | DNS misconfigured | Check `dns-server` in DHCP pool (e.g. `8.8.8.8`) |
| PCs can't reach each other | Switch connectivity issue | `show interfaces status` |

---

## 🏬 Site B — Branch Office (Mwas)

### VLSM Addressing Plan — Block `192.168.10.0/24`

| Department | Users | VLAN | Subnet | Gateway | Usable Range |
|------------|-------|------|--------|---------|--------------|
| Dept 123 | 50 | VLAN 20 | 192.168.10.0/26 | 192.168.10.1 | .2 – .62 |
| Archives | 20 | VLAN 10 | 192.168.10.64/27 | 192.168.10.65 | .66 – .94 |
| *Reserved* | — | — | 192.168.10.96 – .255 | — | — |

### Topology

`R1 (G0/1, trunk) → Mwas-Switch → Archives-Switch3 (VLAN 10) + 123-Switch4 (VLAN 20)`

### Router Configuration

```
enable
conf t
int g0/1
 no shut
int g0/1.20
 encapsulation dot1Q 20
 ip address 192.168.10.1 255.255.255.192
int g0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.65 255.255.255.224
ip dhcp excluded-address 192.168.10.1 192.168.10.5
ip dhcp excluded-address 192.168.10.65 192.168.10.69
ip dhcp pool V20
 network 192.168.10.0 255.255.255.192
 default-router 192.168.10.1
 dns-server 8.8.8.8
ip dhcp pool V10
 network 192.168.10.64 255.255.255.224
 default-router 192.168.10.65
 dns-server 8.8.8.8
enable secret cisco12345
username admin secret admin123
ip domain-name mwas.local
crypto key generate rsa modulus 1024
line vty 0 4
 login local
 transport input ssh
no ip http server
end
write
```

### Switch Configuration

**Mwas-Switch (distribution/trunk):**
```
vlan 10
vlan 20
int g0/1
 switchport mode trunk
int f0/1
 switchport mode trunk
int f0/2
 switchport mode trunk
```

**Archives-Switch3 (VLAN 10 access):**
```
vlan 10
int f0/24
 switchport mode trunk
int range f0/1-10
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

**123-Switch4 (VLAN 20 access):**
```
vlan 20
int f0/24
 switchport mode trunk
int range f0/1-10
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security mac-address sticky
```

### Security Configuration

| Feature | Status |
|---------|--------|
| DHCP (per-VLAN pools) | ✅ Configured |
| IP addressing via VLSM | ✅ Configured |
| Switch port security (sticky MAC) | ✅ Configured |
| Router security (SSH, enable secret, local auth) | ✅ Configured |
| **Stretch objective:** sticky MAC address port security | ✅ Configured |

### Test / Verification Commands

```
show ip dhcp binding
show vlan brief
ping 192.168.10.1
ping 192.168.10.65
```

---

## 🔗 Combined Topology — WAN Interconnection

Both routers were connected over a WAN link to bring Site A and Site B into a single network. This is the current combined state and next steps:

| Item | Status |
|------|--------|
| Physical/logical WAN link between R1 (Site A) and R1 (Site B) | ✅ Connected |
| Each site's internal LAN (DHCP, VLANs, security) | ✅ Independently functional |
| Routing between Site A (192.168.1.0/24) and Site B (192.168.10.0/24) | ⬜ To confirm/configure (static routes or a dynamic routing protocol such as OSPF) |
| End-to-end ping between a Site A host and a Site B host | ⬜ To verify after routing is in place |

> **Next step:** add static routes on each router pointing to the other site's network across the WAN link (or configure OSPF once Module 3 routing content is covered), then verify with `ping` and `traceroute` between the two sites.

---

## ⚠️ Security Notes

> **IMPORTANT:** All credentials in this document are for **educational/lab purposes only** on a simulated Packet Tracer network. Never reuse these usernames/passwords on real or production devices.

Both sites independently implement the CCNA 1 security baseline: local authentication, enable secret, SSH-only remote access, and password encryption. Site B additionally applies port security with sticky MAC learning as its stretch objective.

---

## 📈 Key Learning Outcomes

| Skill | Demonstrated In |
|-------|------------------|
| IPv4 addressing & planning | Site A (flat) + Site B (VLSM) |
| VLANs & trunking (802.1Q) | Site B |
| Inter-VLAN routing (router-on-a-stick) | Site B |
| DHCP configuration (single & multi-pool) | Both sites |
| NAT/PAT for internet access | Site A |
| Switch port security (sticky MAC) | Site B |
| Router/switch hardening (SSH, local auth, encryption) | Both sites |
| Multi-site network design & WAN interconnection | Combined project |
| Version control collaboration (Git/GitHub) | Combined project |

---

## 🚀 Future Enhancements

### Routing & Interconnection
- [ ] Static or dynamic routing (OSPF) between Site A and Site B
- [ ] Verify full end-to-end connectivity across both sites

### Module 2 — Switching
- [ ] Extend VLAN segmentation to Site A (currently flat VLAN 1)
- [ ] Spanning Tree configuration and root bridge placement
- [ ] EtherChannel for switch uplinks

### Module 3 — Routing & Advanced Features
- [ ] Replace static routing with OSPF across both sites
- [ ] ACLs restricting SSH/traffic between sites
- [ ] HSRP for gateway redundancy

### Security & Management
- [ ] AAA with RADIUS/TACACS+
- [ ] Centralized syslog server
- [ ] NTP time synchronization
- [ ] SNMP monitoring
- [ ] Per-device unique credentials (currently shared across devices)

---

## 🤝 Team & Contributions

| Member | Role | Contribution |
|--------|------|---------------|
| **Clifford Karimi** | Project Lead | Original SOHO topology, Site A design/config, project documentation, GitHub repo management |
| **Mwas** | Contributor | Site B design/config (VLSM addressing, VLANs, port security) |

### Contributing
This is a student collaboration project. To contribute:
1. Clone the repo (or fork, if you don't have collaborator access)
2. Create a branch named after yourself
3. Build your LAN in your own `.pkt` file (do not edit the shared master file directly)
4. Push your branch and notify the project lead for manual integration

---

## 📚 References

- [Cisco CCNA Official Cert Guide](https://www.ciscopress.com/)
- [Cisco Packet Tracer Documentation](https://www.netacad.com/courses/packet-tracer)
- [Subnetting Practice](https://subnettingpractice.com/)
- Cisco Networking Academy — Introduction to Networks (CCNA 1)

---

## 📌 Project Status

| Metric | Status |
|--------|--------|
| Site A Design & Config | ✅ Complete |
| Site B Design & Config | ✅ Complete |
| WAN Interconnection | ✅ Physically connected |
| Inter-site Routing | ⬜ In progress |
| Documentation | ✅ Complete |
| Deployment Ready | ⚠️ Lab environment only |

---

## ⚖️ License

This project is for **educational purposes only**. All configurations follow Cisco best practices and are intended for learning; the design does not represent a production environment.

---

*This project was completed as part of the Cisco CCNA certification journey.*
**Last Updated:** September 2026
**Version:** 2.0 — Multi-Site Edition
