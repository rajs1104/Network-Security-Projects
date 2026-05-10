# Low-Level Design (LLD)  
## FortiGate SD‑WAN + Dual WAN + IPsec VPN (CSR1000V Routers)

---

## 1. Device List

- **FGT1** — FortiGate‑VM (Branch firewall)  
- **CSR‑ISP1** — CSR1000V simulating ISP #1  
- **CSR‑ISP2** — CSR1000V simulating ISP #2  
- **CSR‑HQ** — CSR1000V simulating HQ router  
- **SW‑Branch** — Layer‑2 switch for Branch LAN  
- **PC‑Branch** — Branch test host  
- **PC‑HQ** — HQ test host  

---

## 2. Interface Mapping

### 2.1 FortiGate (FGT1)
- **port1** → WAN1 → CSR‑ISP1  
- **port2** → WAN2 → CSR‑ISP2  
- **port3** → LAN → SW‑Branch  

### 2.2 CSR‑ISP1
- **GigabitEthernet1** → FGT1 WAN1  
- **GigabitEthernet2** → CSR‑HQ (transit)  

### 2.3 CSR‑ISP2
- **GigabitEthernet1** → FGT1 WAN2  
- **GigabitEthernet2** → CSR‑HQ (optional transit)  

### 2.4 CSR‑HQ
- **GigabitEthernet1** → ISP cloud  
- **GigabitEthernet2** → HQ LAN  

---

## 3. IP Addressing

### 3.1 Branch LAN
- FGT1 port3: **10.250.10.1/24**  
- PC‑Branch: **10.250.10.10/24**, GW **10.250.10.1**

### 3.2 HQ LAN
- CSR‑HQ Gi2: **10.250.20.1/24**  
- PC‑HQ: **10.250.20.10/24**, GW **10.250.20.1**

### 3.3 WAN1 (FGT1 ↔ CSR‑ISP1)
- CSR‑ISP1 Gi1: **203.0.113.1/30**  
- FGT1 port1: **203.0.113.2/30**

### 3.4 WAN2 (FGT1 ↔ CSR‑ISP2)
- CSR‑ISP2 Gi1: **198.51.100.1/30**  
- FGT1 port2: **198.51.100.2/30**

### 3.5 HQ WAN (CSR‑HQ ↔ CSR‑ISP1)
- CSR‑HQ Gi1: **203.0.113.5/30**  
- CSR‑ISP1 Gi2: **203.0.113.6/30**

---

## 4. Routing

### 4.1 FortiGate (FGT1)
- Default route → **SD‑WAN zone**  
- Static route to HQ LAN:  
  - Destination: **10.250.20.0/24**  
  - Gateway: **IPsec interface (to‑HQ)**  

### 4.2 CSR‑ISP1
- Connected: `203.0.113.0/30`, `203.0.113.4/30`  
- Static route to HQ LAN:  
  - `10.250.20.0/24` → `203.0.113.5`  

### 4.3 CSR‑ISP2
- Connected: `198.51.100.0/30`  
- Optional: route to HQ WAN if HQ should be reachable via WAN2  

### 4.4 CSR‑HQ
- Connected: `203.0.113.4/30`, `10.250.20.0/24`  
- Static route to Branch LAN:  
  - `10.250.10.0/24` → IPsec tunnel interface  

---

## 5. SD‑WAN Design (FGT1)

### 5.1 SD‑WAN Members
- **Member 1:** port1 (WAN1), gateway 203.0.113.1, priority 1  
- **Member 2:** port2 (WAN2), gateway 198.51.100.1, priority 2  

### 5.2 Health Checks
- **hc‑wan1:** ping 203.0.113.1  
- **hc‑wan2:** ping 198.51.100.1  

### 5.3 SD‑WAN Rules
- **Rule 1 — Internet:**  
  - Source: 10.250.10.0/24  
  - Destination: all  
  - Preferred: WAN1 → WAN2  

- **Rule 2 — HQ traffic:**  
  - Source: 10.250.10.0/24  
  - Destination: 10.250.20.0/24  
  - Preferred: WAN1 → WAN2  

---

## 6. IPsec VPN Design

### 6.1 Phase 1 — FGT1
- Name: `to-HQ`  
- Remote gateway: 203.0.113.5  
- Interface: SD‑WAN  
- IKE version: 2  
- Authentication: Pre‑shared key  
- Encryption: AES256  
- Hash: SHA256  
- DH group: 14  
- Lifetime: 28800  

### 6.2 Phase 1 — CSR‑HQ
- Remote peer: 203.0.113.2  
- Same crypto parameters  

### 6.3 Phase 2 — FGT1
- Name: `to-HQ-p2`  
- Local subnet: 10.250.10.0/24  
- Remote subnet: 10.250.20.0/24  
- AES256 / SHA256  
- PFS group 14  
- Lifetime: 3600  

### 6.4 Phase 2 — CSR‑HQ
- Local: 10.250.20.0/24  
- Remote: 10.250.10.0/24  

---

## 7. Firewall Policies (FGT1)

### 7.1 LAN → Internet
- From: LAN  
- To: SD‑WAN  
- Source: 10.250.10.0/24  
- Destination: all  
- NAT: enabled  

### 7.2 LAN → HQ (VPN)
- From: LAN  
- To: to‑HQ  
- Source: 10.250.10.0/24  
- Destination: 10.250.20.0/24  
- NAT: disabled  

---

## 8. NAT

- **Outbound NAT:** enabled on LAN → Internet  
- **VPN traffic:** NAT disabled  

---

## 9. Monitoring & Logging

- Enable logging on:  
  - LAN → Internet  
  - LAN → HQ  
- Monitor:  
  - SD‑WAN link status  
  - VPN up/down  
  - Policy hits  

---

## 10. Build Order

1. Build EVE‑NG topology  
2. Configure IPs on FGT1 + CSR routers  
3. Verify link connectivity  
4. Configure routing  
5. Configure SD‑WAN  
6. Configure IPsec  
7. Add firewall policies  
8. Test:  
   - Branch → Internet  
   - Branch → HQ  
   - WAN failover  

---
