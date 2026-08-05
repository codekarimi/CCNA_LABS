 SOHO Network Implementation Project
📌 Project Overview
This project simulates a Small Office / Home Office (SOHO) network environment using Cisco devices. The network provides internet connectivity, DHCP services, and Network Address Translation (NAT) for multiple end-user devices in a small business setting.

Objective: Design and configure a fully functional SOHO network that allows all internal devices to access the internet through a single public IP address.

### 📊 IP Addressing Scheme

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | G0/0 (LAN) | 192.168.1.1 | 255.255.255.0 | N/A |
| R1 | G0/1 (WAN) | 203.0.113.2 | 255.255.255.252 | 203.0.113.1 |
| SW-Main | VLAN 1 | 192.168.1.13 | 255.255.255.0 | 192.168.1.1 |
| SW-sales | VLAN 1 | 192.168.1.12 | 255.255.255.0 | 192.168.1.1 |
| SW-finance | VLAN 1 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| Finace_PC | DHCP | DHCP Assigned | 255.255.255.0 | 192.168.1.2 |
| Sales_PC | DHCP | DHCP Assigned | 255.255.255.0 | 192.168.1.3 |
| IT_PC | DHCP | DHCP Assigned | 255.255.255.0 | 192.168.1.4 |

> **DHCP Pool:** 192.168.1.2 – 192.168.1.254  
> **Addresses:** 192.168.1.11 – 192.168.1.20 (Reserved for maintaince)
>
## 🔐 Credentials & Security

> ⚠️ **IMPORTANT:** These credentials are for **educational/lab purposes only**. Never use default or weak passwords in production environments.

### Device Access Credentials

| Device | Access Method | Username | Password | Privilege Level |
|--------|---------------|----------|----------|-----------------|
| **R1 (Router)** | Console | `netadmin` | `letmein` | User EXEC (Level 1) |
| | Privileged EXEC | `netadmin` | `letmein` | Privileged EXEC (Level 15) |
| | SSH | `netadmin` | `letmein` | User EXEC (Level 1) |
| **SW_Main (Core Switch)** | Console | `netadmin` | `letmein` | User EXEC (Level 1) |
| | Privileged EXEC | `netadmin` | `letmein` | Privileged EXEC (Level 15) |
| | SSH | `netadmin` | `letmein` | User EXEC (Level 1) |
| **SW_Sales** | Console | `netadmin` | `letmein` | User EXEC (Level 1) |
| | Privileged EXEC | `netadmin` | `letmein` | Privileged EXEC (Level 15) |
| | SSH | `netadmin` | `letmein` | User EXEC (Level 1) |
| **SW_Finance** | Console | `netadmin` | `letmein` | User EXEC (Level 1) |
| | Privileged EXEC | `netadmin` | `letmein` | Privileged EXEC (Level 15) |
| | SSH | `netadmin` | `letmein` | User EXEC (Level 1) |

### Security Notes

| Security Feature | Implementation | Status |
|------------------|----------------|--------|
| **Username/Password** | Local username database configured on all devices | ✅ Configured |
| **Enable Secret** | Privileged mode access using password authentication | ✅ Configured |
| **Console Authentication** | Local username/password required on console port | ✅ Configured |
| **VTY Authentication** | Local username/password required for remote access | ✅ Configured |
| **SSH Only** | Telnet disabled, SSH version 2 enabled on all devices | ✅ Configured |
| **Password Encryption** | `service password-encryption` enabled | ✅ Configured |
| **AAA** | Authentication, Authorization, Accounting | ⬜ Future Enhancement |

### 🔑 Access Commands Quick Reference

| Access Method | Command |
|---------------|---------|
| **Console Access** | Connect console cable → enter username `netadmin` and password `letmein` |
| **Enter Privileged Mode** | `enable` → enter password `letmein` |
| **SSH to R1** | `ssh -l netadmin 192.168.1.1` → enter password `letmein` |
| **SSH to SW_Main** | `ssh -l netadmin 192.168.1.13` → enter password `letmein` |
| **SSH to SW_Sales** | `ssh -l netadmin 192.168.1.12` → enter password `letmein` |
| **SSH to SW_Finance** | `ssh -l netadmin 192.168.1.11` → enter password `letmein` |

> **Note:** Telnet is **disabled** on all devices. Only SSH is permitted for remote access.



### ⚠️ Security Best Practices Applied

- [x] All devices use consistent `netadmin` username
- [x] Local username database configured for authentication
- [x] Enable secret configured for privileged mode access
- [x] Console, VTY, and enable mode all require authentication
- [x] **Telnet disabled** – only SSH allowed for remote access
- [x] SSH version 2 enabled for secure encrypted connections
- [x] RSA key generated with 2048-bit modulus
- [x] SSH authentication retries limited to prevent brute force
- [x] Passwords encrypted in running configuration with `service password-encryption`
- [ ] **To Do:** Change default username for different access levels
- [ ] **To Do:** Implement different passwords for different devices
- [ ] **To Do:** Configure AAA and RADIUS/TACACS+ for enterprise-level security
- [ ] **To Do:** Implement ACLs to restrict SSH access to specific IPs


## 🧪 Troubleshooting Guide

| Symptom | Possible Cause | Solution |
|---------|---------------|----------|
| **PC doesn't get IP address** | DHCP pool missing or disabled | Check `show ip dhcp binding` and `show run \| section dhcp` |
| **Can't ping default gateway** | Wrong IP config on PC or router | Verify `ipconfig` and `show ip interface brief` |
| **No internet access** | Missing default route or NAT | Check `show ip route` and `show ip nat statistics` |
| **Switch unreachable** | Management VLAN IP not set | Verify `show ip interface brief` on VLAN 1 |
| **NAT not translating** | Wrong inside/outside interfaces | Ensure `ip nat inside` on G0/0 and `ip nat outside` on G0/1 |
| **Client can't ping by domain name** | DNS server misconfigured | Check `dns-server` in DHCP pool or use `8.8.8.8` |
| **PCs can't ping each other** | Switch connectivity issue | Check `show interfaces status` on both switches |

---

## 📈 Key Learning Outcomes

Through this project, I have demonstrated:

| Skill | Proficiency |
|-------|-------------|
| **IP Addressing** | ✅ IPv4 address planning and allocation |
| **Router & Switch CLI** | ✅ Basic configuration, security, and management |
| **DHCP Configuration** | ✅ Server setup, pool creation, address exclusion |
| **Static Routing** | ✅ Default route configuration |
| **NAT & PAT** | ✅ Address translation for internet access |
| **Network Troubleshooting** | ✅ Layer 2/3 verification and issue resolution |
| **Network Documentation** | ✅ Professional README and topology diagrams |

### Key Skills Gained

- **Network Design:** Planned IP addressing scheme for a small office network
- **Device Configuration:** Applied basic configurations on Cisco routers and switches
- **Service Implementation:** Set up DHCP and NAT services for network clients
- **Testing & Validation:** Verified functionality using show commands and ping tests
- **Documentation:** Created comprehensive project documentation

---

## 📊 Project Timeline

| Phase | Task | Duration |
|-------|------|----------|
| 1 | Network Design & IP Planning | 30 minutes |
| 2 | Router Basic Configuration | 20 minutes |
| 3 | Switch Configuration | 15 minutes |
| 4 | DHCP Server Setup | 15 mi## 🚀 Future Enhancements

Once I complete CCNA Modules 2 & 3, I plan to upgrade this project with:

### Module 2 - Switching Enhancements
- [ ] **VLANs** – Separate Sales, HR, and Guest networks
- [ ] **Inter-VLAN Routing** – Enable communication between VLANs
- [ ] **Trunking** – 802.1Q trunk ports between switches
- [ ] **Spanning Tree** – STP configuration and root bridge placement
- [ ] **Port Security** – Restrict MAC addresses on access ports

### Module 3 - Routing & Advanced Features
- [ ] **OSPF** – Replace static routing with dynamic routing
- [ ] **SSH** – Replace telnet with secure shell
- [ ] **ACLs** – Implement traffic filtering rules
- [ ] **EtherChannel** – Link aggregation for switch uplinks
- [ ] **HSRP** – Gateway redundancy with second router

### Security & Management
- [ ] **Syslog** – Centralized logging server
- [ ] **NTP** – Time synchronization across devices
- [ ] **SNMP** – Network monitoring setup
- [ ] **AAA** – Authentication, authorization, accounting

---

## 📚 References

- [Cisco CCNA Official Cert Guide](https://www.ciscopress.com/)
- [Cisco Packet Tracer Documentation](https://www.netacad.com/courses/packet-tracer)
- [Subnetting Practice](https://subnettingpractice.com/)
- Cisco Networking Academy - Introduction to Networks (Module 1)

---

## 🤝 Contributing

This is a student project for learning purposes. If you have suggestions for improvements:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request with your changes

---

## 👨‍💻 Author

**Clifford Karimi**  
- CCNA Student (Module 1 Completed)  
- GitHub: [github.com/codekarimi](https://github.com/codekarimi  
- LinkedIn: [linkedin.com/in/cliffordkarimi](https://linkedin.com/in/cliffordkarimi)  
- Email: ckarimi676@gmail.com

---

## 📝 Acknowledgments

- **Cisco Networking Academy** – For the foundational networking knowledge
- **Packet Tracer Team** – For providing an excellent simulation tool

---

## ⚖️ License

This project is for **educational purposes only**.

All configurations are based on Cisco best practices and are intended for learning. The network design does not represent any production environment.

---

## 📌 Project Status

| Metric | Status |
|--------|--------|
| **Design** | ✅ Complete |
| **Configuration** | ✅ Complete |
| **Testing** | ✅ Complete |
| **Documentation** | ✅ Complete |
| **Deployment Ready** | ⚠️ Lab environment only |

---

**Last Updated:** August 2026  
**Version:** 1.0  
**Status:** ✅ Complete – Verified and Tested

---

*This project was completed as part of the Cisco CCNA certification journey.*on | 20 minutes |
| 6 | Testing & Verification | 30 minutes |
| 7 | Documentation | 45 minutes |
| | **Total Time** | **~3 hours** |

---

