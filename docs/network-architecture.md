# NET-002 — Enterprise Network Architecture

**Document status:** HQ as-built / full enterprise design planned  
**Revision:** 2026-10-09  
**Basis:** EVE-NG topology screenshot and observed CLI outputs; enterprise scope from project roadmap.

## 1. Business / lab scope

Emulate a company with **one HQ and two branches**, dual simulated ISP uplinks at HQ, a security perimeter with DMZ, isolated user/server/guest segments, and future VPN, monitoring and automation.

## 2. Logical design — planned enterprise

```text
            Simulated Internet / ISP1 + ISP2
                         |      |
                      EDGE1   EDGE2
                          \    /
                         Cisco ASAv
                        /         \
                      DMZ       HQ INSIDE
                                 |
                    CORE1 ===== CORE2
                      | \       / |
                      |  \     /  |
                    ACCESS1   ACCESS2
                     IT/HR    ACC/GUEST

            Branch 1 & Branch 2: WAN/VPN (future sprints)
```

Diagram is **conceptual only**, not a detailed firewall or WAN interface implementation. Dedicated transit broadcast domains and actual links must be defined before edge configuration.

## 3. HQ campus — as-built

![EVE-NG HQ campus topology](../diagrams/hq-campus-topology.svg)

- **CORE1 / CORE2:** IOL L2 image with successfully demonstrated SVI + HSRP capability. `ip routing` command accepted; routed forwarding between distinct VLANs is still a separate Sprint 1 verification.
- **ACCESS1:** departmental attachment for IT and HR.
- **ACCESS2:** departmental attachment for Accounting and Guest.
- **PC-MGMT:** temporarily attached directly to CORE1 (not redundant).
- **Core-to-Core:** Ethernet0/0 trunk currently supports HSRP on VLAN 10.
- **Dual-homing:** both Access switches link to both independent Cores. L2 loops require intentional STP/RSTP policy before enabling all trunk VLANs.

### Actual HQ interface mapping (from EVE-NG screenshot)

| A | Port A | B | Port B | Link function |
|---|---|---|---|---|
| CORE1 | e0/0 | CORE2 | e0/0 | Inter-core 802.1Q trunk |
| CORE1 | e0/1 | ACCESS1 | e0/0 | Planned trunk |
| CORE2 | e0/1 | ACCESS1 | e0/1 | Planned trunk / redundancy |
| CORE1 | e0/2 | ACCESS2 | e0/0 | Planned trunk |
| CORE2 | e0/2 | ACCESS2 | e0/1 | Planned trunk / redundancy |
| CORE1 | e1/0 | PC-MGMT | eth0 | Test access VLAN 10 |
| ACCESS1 | e0/2 | PC-IT | eth0 | Planned access VLAN 20 |
| ACCESS1 | e0/3 | PC-HR | eth0 | Planned access VLAN 30 |
| ACCESS2 | e0/2 | PC-ACC | eth0 | Planned access VLAN 40 |
| ACCESS2 | e0/3 | PC-GUEST | eth0 | Planned access VLAN 60 |

**Note:** Port names and wires are confirmed by the screenshot; the intended trunk/access modes for Sprint 1 cannot be assumed already configured.

## 4. Layer 3 and gateway strategy

| VLAN | Role | Virtual gateway | CORE1 | CORE2 | State |
|---:|---|---|---|---|---|
| 10 | MGMT | 10.10.10.1 | 10.10.10.2 | 10.10.10.3 | **Verified** SVI and HSRP Active/Standby |
| 20 | IT | 10.10.20.1 | 10.10.20.2 | 10.10.20.3 | Planned |
| 30 | HR | 10.10.30.1 | 10.10.30.2 | 10.10.30.3 | Planned |
| 40 | ACCOUNTING | 10.10.40.1 | 10.10.40.2 | 10.10.40.3 | Planned |
| 50 | SERVER | 10.10.50.1 | 10.10.50.2 | 10.10.50.3 | Planned |
| 60 | GUEST | 10.10.60.1 | 10.10.60.2 | 10.10.60.3 | Planned |

VLAN 99 is reserved as the future native VLAN, without an SVI/IP. HSRP on VLAN 10 uses priority **110/preempt CORE1**, **100/preempt CORE2**; both point to VIP **10.10.10.1**.

## 5. Design constraints and redundancy boundaries

1. **L2 dual uplink is not automatically zero-downtime:** RSTP must be configured and tested on the looped access/core topology.
2. **No cross-chassis EtherChannel without compatible multi-chassis technology:** ACCESS1 e0/0 to CORE1 and e0/1 to CORE2 must not be combined in a normal LACP bundle.
3. **Single firewall is a single point of failure:** dual ISP does not imply redundant security perimeter.
4. **PC-MGMT has no physical redundancy** while attached to CORE1 e1/0. Move to ACCESS1/2 for end-to-end failover testing.
5. **WAN/DMZ/Branch diagrams not yet wired:** document transit VLANs and interface map before implementing Sprint 3–6.
6. **Lab resource budget:** EVE VM has 8 GB allocated; stage the lab by subnet/module rather than powering all node groups on at once.

## 6. Technology placement by sprint

| Sprint | Domain | Intended feature |
|---:|---|---|
| 1 | HQ Access/Core | VLAN, trunk/native VLAN, SVI, RSTP, end-to-end tests |
| 2 | HQ redundancy | Valid LACP links, full HSRP / failover cases |
| 3 | HQ + Branch routers | Single- and multi-area OSPF |
| 4 | ISP/Edge | BGP, dual-ISP failover |
| 5 | Security | ACL/NAT/ASA/DMZ |
| 6 | WAN security | Site-to-site IPSec |
| 7–9 | Operations | Alpine infrastructure services, monitoring, automation |

## 7. Design deliverables still needed

- [x] Captured HQ as-built topology image.
- [x] HQ physical connection mapping and core IP/HSRP design.
- [ ] Final **logical-topology.png** including HQ, Branch 1/2, ISP1/2, Firewall and DMZ.
- [ ] Final **physical-topology.png** including all final interface labels.
- [ ] Dedicated inside/outside firewall transit implementation and IP validation.
- [ ] Repeatable end-to-end Inter-VLAN, RSTP and LACP/HSRP failover evidence.