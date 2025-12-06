#  Defense-in-Depth Network Security Architecture  
### A Multi-Layered Security Approach using pfSense, Zeek, ELK, Nginx (WAF), and ClamAV

---

##  Overview  
This project implements a complete **Defense-in-Depth** security architecture inside a **GNS3 simulation environment**.  
It combines **firewalling, segmentation, WAF, malware scanning, VPN**, and **centralized monitoring**, using entirely open-source tools.

The system delivers **visibility, layered protection, network isolation, and attack prevention** similar to enterprise-grade networks.

---

##  Authors  
- **Muhammad Akash Waris** (Roll No. 23I-2110)  
- **Habib Ahmed** (Roll No. 23I-2078)  
- **Date:** 29-11-2025

---

##  Table of Contents  
- [Overview](#-overview)  
- [Architecture Summary](#-architecture-summary)  
- [Network Zones & Topology](#-network-zones--topology)  
- [Implementation Phases](#-implementation-phases)  
- [Testing & Results](#-testing--results)  
- [VPN Remote Access](#-vpn-remote-access)  
- [Challenges & Solutions](#-challenges--solutions)  
- [Conclusion](#-conclusion)  
- [References](#-references)

---

#  Architecture Summary  

This project integrates the following components:

| Component | Purpose |
|----------|---------|
| **pfSense** | Perimeter firewall, NAT, segmentation, rule filtering |
| **Open vSwitch (OVS)** | VLAN segmentation & SPAN traffic mirroring |
| **Nginx Reverse Proxy + WAF** | Blocks SQLi/XSS and routes traffic securely |
| **ClamAV Antivirus Gateway** | Scans and blocks malicious uploads |
| **Zeek + ELK Stack** | Network traffic visibility, analytics, dashboards |

This architecture ensures end-to-end security across all network layers.

---

#  Network Zones & Topology  

### VLANs & Addressing
| Zone | VLAN | CIDR | Purpose |
|------|------|------|---------|
| **LAN** | 20 | 10.0.2.0/24 | Employee network |
| **Admin** | 10 | 10.0.1.0/24 | Zeek & ELK monitoring |
| **DMZ** | 30 | 10.0.3.0/24 | Public-facing Proxy & Web Server |
| **VPN** | 40 | 10.0.4.0/24 | Remote access zone |

### Key Characteristics
- **OVS** for virtual segmentation  
- **SPAN port** sends mirrored traffic to Zeek  
- Strict **inter-zone firewall policies**

---

#  Implementation Phases  

## **Phase 1 — Perimeter Security using pfSense**
- Default **BLOCK** on WAN  
- Allowed only:
  - Port **80** → Reverse Proxy
  - **OpenVPN (UDP 1194)**
- DMZ is **isolated** from LAN → prevents lateral movement  
- Management/Admin VLAN protected: only authorized access allowed  
- **pfBlocker** enabled to block malicious IPs automatically  

---

## **Phase 2 — Security Gateway (Nginx Reverse Proxy, WAF, ClamAV)**

###  Web Application Firewall (WAF)
Nginx filters malicious payloads using regex-based rule sets  
Blocks:
- SQL Injection (`UNION SELECT`, `OR 1=1`, `DROP TABLE`)  
- XSS (`<script>`, `alert()`, `javascript:`)  

###  Antivirus Gateway (ClamAV)
- Custom Python middleware intercepts uploads  
- Scans files using `clamscan`  
- Deletes malicious files (e.g., EICAR test file)  
- Shows **SECURITY ALERT** page  

###  Reverse Proxy
- Nginx forwards clean traffic to backend Web Server  
- Protects internal infrastructure from direct exposure  

---

## **Phase 3 — Traffic Analysis Pipeline (Zeek + ELK)**  

### Zeek IDS
- Configured in **cluster-of-one** mode  
- Captures mirrored traffic from OVS SPAN  
- Outputs structured JSON logs  

### Logstash → Elasticsearch
- Custom pipeline parses Zeek logs  
- Sends structured data into Elasticsearch indexes  

### Kibana Dashboards  
- Real-time traffic visualization  
- Threat indicators  
- Source/Destination IP analysis  
- Protocol distribution & trends  

### Self-Healing Service
A custom `systemd` service ensures DDoS or large traffic bursts do NOT crash Zeek.

---

#  Testing & Results  

##  SQL Injection Test
**Payload:** `' OR 1=1 --`  
**Result:**  
WAF blocks request → **403 Forbidden**  

##  Malware Upload Test  
**File:** `eicar.com.txt`  
**Result:**  
Antivirus Gateway intercepts → deletes → alerts user  

##  Network Visibility Test  
Kibana dashboard displays:  
- Traffic logs  
- Connection states  
- Protocols  
- Suspicious activities  

This proves the ELK pipeline and Zeek IDS are functioning fully.

---

#  VPN Remote Access  

### OpenVPN Configuration  
- Root CA + certificates created on pfSense  
- MFA: **Certificate + Credentials**  
- Client receives virtual IP from **10.0.4.0/24**  
- Split tunneling ensures only office traffic goes through VPN  

### Firewall Rules  
- WAN → allows UDP 1194  
- VPN clients → can access LAN  
- VPN clients → cannot access Admin VLAN  

### Verification  
- External Linux client connects successfully  
- Pings internal network (10.0.2.1)  
- Traffic tunnels securely through pfSense  

---

#  Challenges & Solutions  

###  Routing Conflict with OpenVPN  
**Issue:**  
OpenVPN interface was accidentally set to **Static IPv4**, causing routing table conflict.

**Solution:**  
Changed interface type → **None**  
This allowed OpenVPN to manage routes correctly.

**Outcome:**  
VPN fully functional with stable connectivity.

---

#  Conclusion  
This project successfully demonstrates a powerful, scalable, and enterprise-grade **Defense-in-Depth architecture** using open-source tools.

It provides:
- Robust perimeter protection  
- Web application filtering  
- Malware detection  
- Full network visibility  
- Secure remote access  

The combined system protects against **SQLi, XSS, malware, unauthorized access, spoofing, and internal threats**.

---

#  References  
- pfSense Firewall Documentation  
- Zeek Network Security Monitor  
- Elastic Stack (ELK) Documentation  
- Nginx Reverse Proxy & WAF Guides  
- ClamAV Antivirus Documentation  

---

