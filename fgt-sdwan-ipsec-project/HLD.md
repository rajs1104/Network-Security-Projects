# High-Level Design (HLD)  
## FortiGate SD-WAN + Dual WAN + IPsec VPN (EVE-NG)

## 1. Purpose

The purpose of this design is to simulate a small branch office deployment using a single FortiGate firewall with:
- Dual WAN connectivity
- SD-WAN for link failover and traffic steering
- An IPsec VPN tunnel to a simulated HQ
- A protected LAN segment with NAT and security policies

This lab is intended for design, implementation, and troubleshooting practice, and to serve as a portfolio-ready project.

---

## 2. High-Level Requirements

- Provide internet access for branch LAN users via two WAN links (ISP1 and ISP2)
- Use SD-WAN to manage both WAN links and provide automatic failover
- Establish an IPsec VPN tunnel between the branch FortiGate and an HQ router
- Allow branch LAN to reach HQ network over the VPN
- Allow HQ to reach branch LAN over the VPN
- Protect the LAN with firewall policies and NAT
- Provide basic logging on the FortiGate (local logs)

---

## 3. Logical Topology

### 3.1 Components

- FortiGate-VM (Branch firewall)
- ISP1 router (simulated in EVE-NG)
- ISP2 router (simulated in EVE-NG)
- HQ router (simulated in EVE-NG)
- LAN switch (EVE-NG switch or simple bridge)
- Test host (Linux/Windows VM in EVE-NG)

### 3.2 Topology Overview

- FortiGate has:
  - WAN1 connected to ISP1 router
  - WAN2 connected to ISP2 router
  - LAN interface connected to branch LAN switch
- HQ router is reachable over a routed path behind ISP1/ISP2 (simulated)
- An IPsec VPN tunnel is established between FortiGate and HQ router
- Branch LAN users access:
  - Internet via SD-WAN (WAN1/WAN2)
  - HQ network via IPsec VPN

---

## 4. IP Addressing Overview

> Exact IPs and masks are defined in the LLD.

- WAN1 segment: FortiGate WAN1 ↔ ISP1
- WAN2 segment: FortiGate WAN2 ↔ ISP2
- LAN segment: Branch LAN (single /24 network)
- VPN endpoints: One IP on FortiGate side, one IP on HQ router side

---

## 5. SD-WAN Design

- WAN1 and WAN2 are members of an SD-WAN zone
- SD-WAN is used as the egress for:
  - Internet-bound traffic from the branch LAN
  - IPsec VPN traffic (primary over WAN1, failover to WAN2)
- Health checks (ping to upstream/ISP or public IP) detect link failure

---

## 6. IPsec VPN Design

- Site-to-site IPsec VPN between:
  - Branch: FortiGate
  - HQ: HQ router
- Phase 1:
  - IKEv2 (or IKEv1)
  - Pre-shared key authentication
- Phase 2:
  - Encryption and integrity algorithms defined in LLD
  - Subnets:
    - Branch LAN subnet
    - HQ LAN subnet
- Routing:
  - Static routes or SD-WAN rules send HQ traffic into the VPN

---

## 7. Security Policy Design

- LAN → Internet:
  - Allow HTTP/HTTPS/DNS
  - NAT enabled
- LAN → HQ:
  - Allow required protocols over IPsec
- HQ → LAN:
  - Allow return traffic over IPsec
- Management:
  - Allow management access from a specific host/subnet (optional)

---

## 8. Logging and Monitoring

- Local logging enabled on the FortiGate
- Log:
  - Firewall accepts/denies
  - VPN up/down
  - SD-WAN link status
- Monitoring via GUI/CLI

---

## 9. Assumptions and Constraints

- Lab environment in EVE-NG
- No HA, no FortiManager, no FortiAnalyzer
- No advanced features (ZTNA, SSL VPN, automation)
- All devices are simulated

---

## 10. Out of Scope

- High availability (HA)
- Cloud integration
- Advanced security services
- Centralized management/logging
- User identity integration (FSSO, LDAP)

