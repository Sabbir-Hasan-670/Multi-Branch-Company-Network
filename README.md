# 🏢 Multi-Branch Company Network (Head Office + 2 Branches) — README

> **CCNA 200-301 Practice • Cisco Packet Tracer**  
> এই README (Bangla + English) আপনার ল্যাবকে শুরু থেকে শেষ পর্যন্ত গাইড করবে। কমান্ডগুলো কপি–পেস্টযোগ্য। আপনি চাইলে পরে নিজের IP plan/নাম বদলে নিতে পারবেন।

---

## 📦 Project Summary
- **Goal:** Head Office (HO), Branch 1 (BO1), Branch 2 (BO2) — এই তিনটি সাইটকে **OSPF dynamic routing** দিয়ে connect করা; **DHCP, DNS, Web** server ইন্টিগ্রেশন; ঐচ্ছিকভাবে **NAT/PAT**।
- **Tools:** Cisco Packet Tracer (only)
- **Devices (suggested):**
  - HO: 1 Router (R-HO), 1 Switch (S-HO), 3 PCs, 1 Server (DNS/DHCP/Web)
  - BO1: 1 Router (R-BO1), 1 Switch (S-BO1), 2 PCs
  - BO2: 1 Router (R-BO2), 1 Switch (S-BO2), 2 PCs

---

## 🧩 Topology (High-Level)

```
              [Head Office]
           ┌─────────────────┐
           │ 192.168.1.0/24  │
           │ DNS/DHCP/WEB    │
           └──┬──────────────┘
              │ 10.0.0.0/30 (S0/0/0)
        ┌─────┴─────┐
   10.0.0.4/30      │
 (S0/0/1)           │
  [R-BO2]         [R-BO1]
192.168.20.0/24  192.168.10.0/24
```

**Cable Tips:**
- Router↔Router (serial DCE/DTE) — *if your router has serial modules; otherwise use G0/1↔G0/1 and adjust IPs*
- Router↔Switch↔PC — Copper straight-through

---

## 🧠 IP Addressing Plan (Default; editable)

| Location | Interface | IP Address | Subnet Mask | Purpose |
|-----------|------------|-------------|--------------|----------|
| R-HO (LAN) | G0/0 | **192.168.1.1** | 255.255.255.0 | HO LAN GW |
| R-HO (WAN to BO1) | S0/0/0 | **10.0.0.1** | 255.255.255.252 | HO↔BO1 |
| R-HO (WAN to BO2) | S0/0/1 | **10.0.0.5** | 255.255.255.252 | HO↔BO2 |
| R-BO1 (LAN) | G0/0 | **192.168.10.1** | 255.255.255.0 | BO1 LAN GW |
| R-BO1 (WAN) | S0/0/0 | **10.0.0.2** | 255.255.255.252 | to HO |
| R-BO2 (LAN) | G0/0 | **192.168.20.1** | 255.255.255.0 | BO2 LAN GW |
| R-BO2 (WAN) | S0/0/0 | **10.0.0.6** | 255.255.255.252 | to HO |
| Server (HO) | NIC | **192.168.1.2** | 255.255.255.0 | DNS/DHCP/Web |
| PCs | NIC | **DHCP** | — | Clients |

> **Edit Tip:** If your Packet Tracer image uses different serial labels (e.g., S0/0/1 vs S0/1/0), just adjust the interface names in commands.

---

## ⚙️ Step 1 — Physical & Layer-1 Setup
1. Place devices: 3×Routers (e.g., 2911), 3×Switches (2960), 7×PCs, 1×Server.
2. Connect:
   - R-HO S0/0/0 ↔ R-BO1 S0/0/0 (Serial DCE on one side; set clock rate)
   - R-HO S0/0/1 ↔ R-BO2 S0/0/0 (Serial DCE on one side; set clock rate)
   - Each site: Router G0/0 ↔ Switch; Switch ↔ PCs/Server
3. Power on interfaces (`no shutdown`) and verify link LEDs.

---

## ⚙️ Step 2 — Base Router Configs

### R-HO (Head Office Router)
```bash
enable
configure terminal
hostname R-HO

interface gigabitEthernet 0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

interface serial 0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
exit

interface serial 0/0/1
 ip address 10.0.0.5 255.255.255.252
 clock rate 64000
 no shutdown
exit

do write memory
```

### R-BO1 (Branch 1 Router)
```bash
enable
configure terminal
hostname R-BO1

interface gigabitEthernet 0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface serial 0/0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

do write memory
```

### R-BO2 (Branch 2 Router)
```bash
enable
configure terminal
hostname R-BO2

interface gigabitEthernet 0/0
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

interface serial 0/0/0
 ip address 10.0.0.6 255.255.255.252
 no shutdown
exit

do write memory
```

---

## ⚙️ Step 3 — OSPF Dynamic Routing

> **Bangla Note:** তিন রাউটারেই OSPF চালালে তারা একে অন্যের LAN/WAN নেটওয়ার্ক শেয়ার করবে; ফলে static route দরকার হবে না।

### R-HO
```bash
configure terminal
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.4 0.0.0.3 area 0
exit
do write memory
```

### R-BO1
```bash
configure terminal
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
exit
do write memory
```

### R-BO2
```bash
configure terminal
router ospf 1
 network 192.168.20.0 0.0.0.255 area 0
 network 10.0.0.4 0.0.0.3 area 0
exit
do write memory
```

**Verification (any router):**
```bash
show ip ospf neighbor
show ip route ospf
show ip protocols
```

---

## ⚙️ Step 4 — Server Roles (HO Server)

### 4.1 Server NIC
- IP: **192.168.1.2**
- Mask: **255.255.255.0**
- Gateway: **192.168.1.1**
- DNS: **192.168.1.2**

### 4.2 DHCP (on Server → Services → DHCP)
```
Pool Name: HeadOffice
Default Gateway: 192.168.1.1
DNS Server: 192.168.1.2
Start IP: 192.168.1.10
Subnet Mask: 255.255.255.0
Maximum Users: 50
Service: ON
Save: YES
```

> চাইলে Branch সাইটগুলোতেও আলাদা DHCP চালাতে পারো (BO1/BO2 রাউটার বা আলাদা সার্ভারে), কিন্তু শুরুতে HO থেকে শুধু ডেমো করলেই যথেষ্ট।

### 4.3 DNS (on Server → Services → DNS)
```
DNS Service: ON
Name: www.company.local
Address: 192.168.1.2
Add → Save
```

### 4.4 Web (on Server → Services → HTTP/HTTPS)
- HTTP: ON, HTTPS: ON
- Edit HTML (example):
```html
<html>
  <h1>Welcome to Our Multi-Branch Network</h1>
  <p>Connected via OSPF. Hello from Head Office!</p>
</html>
```

---

## ⚙️ Step 5 — PCs (Clients)
- **IP Assignment:** Desktop → IP Configuration → **DHCP** (auto)
- Verify clients receive IP **192.168.x.y**, gateway, and DNS.

---

## 🧪 Step 6 — End-to-End Testing

### 6.1 Basic Ping
- From **BO1 PC** → `ping 192.168.1.2` (HO Server)
- From **BO2 PC** → `ping 192.168.10.1` (R-BO1)

### 6.2 DNS Test
- From any PC: `ping www.company.local`
  - Should resolve to **192.168.1.2**

### 6.3 Web Test
- Open PC Browser → `http://www.company.local`  
  - Should load your HTML page.

### 6.4 OSPF Routes Visible
- On any router: `show ip route`  
  - Look for routes with **O** (OSPF).

---

## ⚙️ Step 7 — (Optional) NAT/PAT at Head Office

> যদি “Internet” সিমুলেট করতে চাও, R-HO থেকে WAN ইন্টারফেসকে **ip nat outside** এবং LAN-কে **ip nat inside** করো; তারপর PAT enable করো।

```bash
configure terminal
access-list 1 permit 192.168.0.0 0.0.255.255

interface gigabitEthernet 0/0
 ip nat inside
exit
interface serial 0/0/1
 ip nat outside
exit

ip nat inside source list 1 interface serial 0/0/1 overload
do write memory
```

**Check NAT:**
```bash
show ip nat translations
show ip nat statistics
```

---

## 🛠️ Troubleshooting Checklist

- **Interface Down?** → `show ip interface brief` (Down/Administratively down হলে `no shutdown` দিন)
- **Wrong Cabling?** → Serial DCE side-এ `clock rate` দিন (e.g., 64000)
- **OSPF Neighbors না দেখাচ্ছে?** → Network statements-এর wildcard mask এবং interface networks মিলিয়ে দেখুন
- **PC IP পাচ্ছে না?** → DHCP Service ON? Gateway/DNS ঠিক আছে?
- **DNS resolve হচ্ছে না?** → DNS এন্ট্রি ঠিক আছে? PC DNS server = 192.168.1.2?
- **Ping one-way?** → ACL/NAT/Firewall off for lab; routing table check

---

## 🧾 Change/Customize This Lab
- IP ranges পরিবর্তন করতে চাইলে **IP Plan** টেবিল আগে বদলান, তারপর commands-এর IP গুলো আপডেট করুন।
- Interface নাম (G0/0, S0/0/0) আপনার ডিভাইস অনুযায়ী এডজাস্ট করুন।
- চাইলে Branch রাউটারগুলোতে **local DHCP pools** চালাতে পারেন:
```bash
! Example: DHCP on R-BO1
ip dhcp pool BO1
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.1.2
```

---

## ✅ Final Verification Commands (Quick Sheet)

```bash
show ip interface brief
show running-config
show ip route
show ip ospf neighbor
show ip protocols
show ip nat translations
show ip nat statistics
ping <IP>
traceroute <IP>
```

---

## 📚 Notes
- এই README Packet Tracer lab-এর জন্য অপ্টিমাইজড।
- বাস্তবে (production) security hardening (ACL/Firewall), redundancy (HSRP/STP), এবং monitoring (SNMP/Syslog) যুক্ত করবেন।

— Happy Lab Building! 🚀
