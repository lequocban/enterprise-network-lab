# Enterprise Network Design & Implementation
## Roadmap học tập + triển khai Lab EVE-NG theo Sprint / Epic / Story / Task

> **Mục tiêu dự án:** Xây dựng một mô hình mạng doanh nghiệp nhiều site đủ chất lượng để đưa vào CV/GitHub, đồng thời dùng chính dự án này làm lộ trình học từ nền tảng CCNA lên các kỹ năng gần thực tế doanh nghiệp: switching, routing, redundancy, firewall, VPN, Linux services, monitoring và automation.

---

# 1. Tổng quan dự án

## 1.1. Bối cảnh giả lập

Công ty có:

- 01 trụ sở chính (**HQ**)
- 02 chi nhánh (**Branch 1**, **Branch 2**)
- 02 ISP tại HQ để dự phòng đường truyền
- Khu vực **DMZ**
- Hệ thống server nội bộ
- Mạng người dùng theo từng phòng ban
- Mạng Guest tách biệt
- Kết nối Site-to-Site VPN giữa HQ và các chi nhánh
- Hệ thống giám sát tập trung
- Hệ thống tự động backup/config thiết bị mạng

## 1.2. Công nghệ dự kiến

### Switching
- VLAN
- 802.1Q Trunk
- RSTP
- EtherChannel / LACP
- Port Security
- BPDU Guard
- DHCP Snooping

### Routing
- Static Route
- Inter-VLAN Routing
- OSPF
- Multi-Area OSPF
- Default Route
- Route Summarization
- eBGP
- Dual ISP

### High Availability
- HSRP
- Redundant Core
- Redundant Uplink
- Failover testing

### Security
- ACL
- NAT/PAT
- Firewall
- DMZ
- Site-to-Site IPSec VPN
- Guest isolation

### Infrastructure Services
- DHCP
- DNS
- NTP
- Syslog
- Web Server

### Monitoring
- SNMP
- Syslog
- Zabbix hoặc LibreNMS

### Automation
- SSH
- Python
- Netmiko/NAPALM (tùy chọn)
- Ansible
- Backup cấu hình tự động

---

# 2. Kiến trúc đề xuất

```text
                             INTERNET
                         +------+------+
                         |             |
                       ISP-1         ISP-2
                         |             |
                       EDGE1---------EDGE2
                          \           /
                           \         /
                            FIREWALL
                               |
                    +----------+----------+
                    |                     |
                   DMZ                 INTERNAL
                    |                     |
             Web / DNS Public        CORE1 ===== CORE2
                                      ||           ||
                              +-------++-----------++-------+
                              |                        |
                           ACCESS1                  ACCESS2
                              |                        |
                        Users / HR / IT       Accounting / Guest

                             HQ NETWORK
                                  |
                              WAN / VPN
                              /       \
                             /         \
                      BRANCH-1       BRANCH-2
```

---

# 3. Kế hoạch địa chỉ IP mẫu

| Site | VLAN/Zone | Chức năng | Subnet đề xuất |
|---|---|---|---|
| HQ | VLAN 10 | Management | 10.10.10.0/24 |
| HQ | VLAN 20 | IT | 10.10.20.0/24 |
| HQ | VLAN 30 | HR | 10.10.30.0/24 |
| HQ | VLAN 40 | Accounting | 10.10.40.0/24 |
| HQ | VLAN 50 | Server | 10.10.50.0/24 |
| HQ | VLAN 60 | Guest | 10.10.60.0/24 |
| HQ | DMZ | Public servers | 10.10.100.0/24 |
| Branch 1 | LAN | Users | 10.20.10.0/24 |
| Branch 2 | LAN | Users | 10.30.10.0/24 |
| WAN | Transit | Router links | 10.255.0.0/24 |

> Có thể thay đổi địa chỉ nếu muốn tự thiết kế IP Plan riêng.

---

# 4. Cấu trúc quản lý công việc

## Epic
Một nhóm tính năng lớn của hệ thống.

Ví dụ:
- EPIC-NET-01: Campus Switching
- EPIC-NET-02: Routing
- EPIC-NET-03: High Availability
- EPIC-NET-04: Security
- EPIC-NET-05: Infrastructure Services
- EPIC-NET-06: Monitoring
- EPIC-NET-07: Automation
- EPIC-NET-08: Documentation & Portfolio

## Story
Một mục tiêu có giá trị cụ thể.

Ví dụ:

> Là Network Engineer, tôi muốn triển khai VLAN cho từng phòng ban để giới hạn broadcast domain và tách biệt lưu lượng.

## Task
Công việc kỹ thuật cụ thể để hoàn thành Story.

Ví dụ:
- Tạo VLAN
- Gán port access
- Cấu hình trunk
- Kiểm tra VLAN database

## Sub-task
Bước nhỏ hơn bên trong Task.

Ví dụ:
- Tạo VLAN 20
- Đặt tên IT
- Gán Gi0/2 vào VLAN 20

---

# 5. Definition of Done chung

Một Story chỉ được xem là **DONE** khi:

- [ ] Đã hiểu lý thuyết chính
- [ ] Đã cấu hình thành công trên lab
- [ ] Đã kiểm thử bằng ping/show/debug hoặc Wireshark
- [ ] Đã ghi lại cấu hình quan trọng
- [ ] Đã chụp ảnh minh chứng
- [ ] Đã cập nhật tài liệu Markdown
- [ ] Đã commit lên Git
- [ ] Có thể tự giải thích lại cách hoạt động mà không nhìn tài liệu

---

# 6. Roadmap theo Sprint

---

# Sprint 0 — Chuẩn bị môi trường và thiết kế

## Mục tiêu
Chuẩn bị EVE-NG, xác định kiến trúc mạng và tạo bộ khung project.

## Kiến thức cần học
- EVE-NG cơ bản
- Các loại node
- Cloud / NAT trong EVE-NG
- Mô hình Core / Distribution / Access
- IP subnetting
- Git và GitHub cơ bản
- Jira Scrum cơ bản

## EPIC-NET-00 — Project Foundation

### Story NET-001 — Cài đặt và kiểm tra EVE-NG

**Mục tiêu:** Có môi trường lab hoạt động ổn định.

#### Tasks
- [ ] Cài EVE-NG
- [ ] Kiểm tra CPU virtualization
- [ ] Import image router
- [ ] Import image switch
- [ ] Import Ubuntu Server
- [ ] Import firewall phù hợp
- [ ] Test console thiết bị
- [ ] Test kết nối node với nhau

#### Acceptance Criteria
- [ ] Router boot được
- [ ] Switch boot được
- [ ] Linux boot được
- [ ] Hai router ping nhau thành công

---

### Story NET-002 — Thiết kế topology tổng thể

#### Tasks
- [ ] Vẽ topology Logical
- [ ] Vẽ topology Physical
- [ ] Xác định số router
- [ ] Xác định số switch
- [ ] Xác định firewall
- [ ] Xác định server
- [ ] Xác định ISP giả lập

#### Deliverable
`diagrams/logical-topology.png`

`diagrams/physical-topology.png`

---

### Story NET-003 — Thiết kế IP Addressing Plan

#### Tasks
- [ ] Chia subnet HQ
- [ ] Chia subnet Branch 1
- [ ] Chia subnet Branch 2
- [ ] Chia transit subnet
- [ ] Chia DMZ subnet
- [ ] Định nghĩa Loopback IP

#### Deliverable
`docs/ip-addressing-plan.md`

---

### Story NET-004 — Khởi tạo GitHub Repository

#### Tasks
- [ ] Tạo repo
- [ ] Viết README ban đầu
- [ ] Tạo cấu trúc thư mục
- [ ] Thêm `.gitignore`
- [ ] Tạo branch strategy đơn giản

#### Cấu trúc

```text
enterprise-network-lab/
├── README.md
├── diagrams/
├── configs/
├── docs/
├── routing/
├── security/
├── monitoring/
├── automation/
├── troubleshooting/
└── screenshots/
```

## Sprint Review
- [ ] Topology hoàn chỉnh
- [ ] EVE-NG hoạt động
- [ ] Repo GitHub hoạt động
- [ ] IP Plan hoàn chỉnh

---

# Sprint 1 — Campus Switching

## EPIC-NET-01 — Enterprise Switching

## Kiến thức cần học
- Broadcast domain
- VLAN
- Access port
- Trunk
- Native VLAN
- DTP
- Router-on-a-stick
- Layer 3 Switch
- RSTP

---

### Story NET-101 — Triển khai VLAN

> Là Network Engineer, tôi muốn tách các phòng ban thành các VLAN riêng biệt.

#### VLAN Plan

| VLAN | Name |
|---:|---|
| 10 | MGMT |
| 20 | IT |
| 30 | HR |
| 40 | ACCOUNTING |
| 50 | SERVER |
| 60 | GUEST |
| 99 | NATIVE |

#### Tasks
- [ ] Tạo VLAN
- [ ] Đặt tên VLAN
- [ ] Gán port access
- [ ] Kiểm tra VLAN database

#### Commands cần hiểu

```bash
show vlan brief
show interfaces switchport
```

#### Test
- [ ] PC cùng VLAN ping được nhau
- [ ] PC khác VLAN chưa ping được nhau

---

### Story NET-102 — Triển khai 802.1Q Trunk

#### Tasks
- [ ] Cấu hình trunk Core ↔ Access
- [ ] Đặt Native VLAN
- [ ] Giới hạn VLAN được phép đi qua trunk
- [ ] Disable DTP nếu image hỗ trợ

#### Test

```bash
show interfaces trunk
```

---

### Story NET-103 — Triển khai Inter-VLAN Routing

#### Tasks
- [ ] Tạo SVI
- [ ] Gán Gateway VLAN
- [ ] Bật Layer 3 routing
- [ ] Test liên VLAN

#### Acceptance Criteria
- [ ] VLAN 20 ping VLAN 30
- [ ] VLAN 30 ping Server VLAN

---

### Story NET-104 — Triển khai Rapid STP

#### Tasks
- [ ] Chọn Root Primary
- [ ] Chọn Root Secondary
- [ ] Xác định blocked port
- [ ] Test failover

#### Test

```bash
show spanning-tree
```

#### Lab Failure Test
- [ ] Shutdown uplink chính
- [ ] Quan sát STP convergence

---

# Sprint 2 — EtherChannel và High Availability

## EPIC-NET-03 — Network High Availability

## Kiến thức cần học
- LACP
- EtherChannel
- First Hop Redundancy
- HSRP
- Active/Standby
- Virtual IP

---

### Story NET-201 — EtherChannel

#### Tasks
- [ ] Tạo LACP giữa CORE1 và CORE2
- [ ] Tạo LACP giữa Core và Access
- [ ] Kiểm tra channel-group

#### Commands

```bash
show etherchannel summary
show interfaces port-channel
```

#### Failure Test
- [ ] Shutdown một interface thành viên
- [ ] Xác nhận Port-Channel vẫn hoạt động

---

### Story NET-202 — HSRP Gateway Redundancy

#### Tasks
- [ ] Tạo SVI trên CORE1
- [ ] Tạo SVI trên CORE2
- [ ] Cấu hình Virtual IP
- [ ] Cấu hình priority
- [ ] Cấu hình preempt
- [ ] Theo dõi trạng thái Active/Standby

#### Ví dụ

```text
VIP:   10.10.20.1
CORE1: 10.10.20.2
CORE2: 10.10.20.3
```

#### Test
- [ ] Client dùng VIP làm gateway
- [ ] Shutdown CORE1
- [ ] Client vẫn ping được mạng ngoài

#### Deliverable
`screenshots/hsrp-failover.png`

---

# Sprint 3 — Enterprise Routing với OSPF

## EPIC-NET-02 — Dynamic Routing

## Kiến thức cần học
- Link-state routing
- OSPF Neighbor
- Router ID
- DR/BDR
- Cost
- Area
- LSDB
- Default route
- Passive interface

---

### Story NET-301 — Single-Area OSPF

#### Tasks
- [ ] Cấu hình router-id
- [ ] Advertise transit network
- [ ] Advertise LAN
- [ ] Cấu hình passive-interface
- [ ] Kiểm tra neighbor

#### Commands

```bash
show ip ospf neighbor
show ip ospf database
show ip route ospf
show ip protocols
```

---

### Story NET-302 — Multi-Area OSPF

#### Design

```text
HQ       Area 0
Branch1  Area 10
Branch2  Area 20
```

#### Tasks
- [ ] Chuyển HQ thành backbone
- [ ] Branch 1 vào Area 10
- [ ] Branch 2 vào Area 20
- [ ] Kiểm tra LSDB
- [ ] Kiểm tra inter-area routes

---

### Story NET-303 — OSPF Summarization

#### Tasks
- [ ] Xác định subnet có thể summarize
- [ ] Cấu hình summary
- [ ] So sánh routing table trước/sau

---

### Story NET-304 — Default Route Injection

#### Tasks
- [ ] Tạo default route ra Edge
- [ ] Advertise default route vào OSPF
- [ ] Kiểm tra Branch học default route

---

# Sprint 4 — Internet Edge và BGP

## EPIC-NET-02 — Advanced Routing

## Kiến thức cần học
- ASN
- eBGP
- iBGP khái niệm
- BGP Neighbor
- Best Path
- Local Preference
- AS Path
- Default Route
- Dual ISP

---

### Story NET-401 — ISP Simulation

#### Tasks
- [ ] Tạo ISP1
- [ ] Tạo ISP2
- [ ] Tạo Internet Test Server
- [ ] Định tuyến giữa ISP và server

---

### Story NET-402 — eBGP với ISP1

#### Tasks
- [ ] Chọn ASN Company
- [ ] Chọn ASN ISP
- [ ] Tạo BGP neighbor
- [ ] Advertise public/test prefix
- [ ] Kiểm tra BGP table

#### Commands

```bash
show ip bgp
show ip bgp summary
show ip route bgp
```

---

### Story NET-403 — Dual ISP

#### Tasks
- [ ] Peer ISP1
- [ ] Peer ISP2
- [ ] Thiết lập primary/backup
- [ ] Test đường đi

---

### Story NET-404 — BGP Failover Testing

#### Failure Scenario
- [ ] Shutdown ISP1
- [ ] Ping Internet Server
- [ ] Quan sát route chuyển ISP2

#### Deliverable
`routing/bgp-failover.md`

---

# Sprint 5 — Network Security, ACL, NAT và DMZ

## EPIC-NET-04 — Network Security

## Kiến thức cần học
- Standard ACL
- Extended ACL
- Stateful Firewall
- NAT
- PAT
- DMZ
- Security Zone
- Least Privilege

---

### Story NET-501 — ACL Segmentation

#### Chính sách mẫu

| Source | Destination | Policy |
|---|---|---|
| IT | Server | Allow |
| HR | Server | Allow limited |
| Guest | Internal | Deny |
| Guest | Internet | Allow |
| Accounting | Server | Allow |
| Internet | Internal | Deny |

#### Tasks
- [ ] Viết traffic matrix
- [ ] Tạo ACL
- [ ] Apply đúng interface/direction
- [ ] Test từng rule

---

### Story NET-502 — NAT/PAT

#### Tasks
- [ ] Xác định inside/outside
- [ ] NAT internal ra Internet
- [ ] PAT cho user
- [ ] Static NAT cho DMZ nếu cần

#### Test
- [ ] User truy cập Internet
- [ ] Kiểm tra NAT table

```bash
show ip nat translations
show ip nat statistics
```

---

### Story NET-503 — Deploy Firewall

#### Tasks
- [ ] Tạo Inside
- [ ] Tạo Outside
- [ ] Tạo DMZ
- [ ] Tạo policy
- [ ] Test stateful filtering

---

### Story NET-504 — DMZ

#### DMZ Services
- Web Server
- DNS Server tùy chọn

#### Rules
- [ ] Internet → Web: TCP 80/443
- [ ] Internet → Internal: DENY
- [ ] DMZ → Internal: hạn chế
- [ ] Internal → DMZ: Allow theo nhu cầu

---

# Sprint 6 — Site-to-Site VPN

## EPIC-NET-04 — Secure Branch Connectivity

## Kiến thức cần học
- IPSec
- IKE
- Phase 1
- Phase 2
- Encryption
- Hash
- Pre-shared Key
- Interesting Traffic
- Tunnel concepts

---

### Story NET-601 — HQ ↔ Branch 1 IPSec VPN

#### Tasks
- [ ] Xác định interesting traffic
- [ ] Cấu hình IKE policy
- [ ] Cấu hình transform-set
- [ ] Cấu hình crypto map/tunnel
- [ ] Apply interface
- [ ] Test tunnel

---

### Story NET-602 — HQ ↔ Branch 2 VPN

Lặp lại tương tự Branch 1.

---

### Story NET-603 — VPN Verification

#### Test
- [ ] Ping HQ → Branch
- [ ] Ping Branch → HQ
- [ ] Capture traffic với Wireshark
- [ ] Kiểm tra encrypted packet counter

---

# Sprint 7 — Linux Infrastructure Services

## EPIC-NET-05 — Infrastructure Services

## Kiến thức cần học
- Ubuntu Server
- DNS
- DHCP
- NTP
- Syslog
- SSH
- Linux networking

---

### Story NET-701 — Ubuntu Infrastructure Server

#### Tasks
- [ ] Tạo Ubuntu Server
- [ ] Cấu hình static IP
- [ ] Cấu hình hostname
- [ ] Update package
- [ ] Enable SSH

---

### Story NET-702 — DHCP Server

#### Tasks
- [ ] Tạo DHCP scope
- [ ] Exclude gateway
- [ ] Đặt DNS
- [ ] Cấu hình DHCP Relay

#### Test
- [ ] Client VLAN 20 nhận IP
- [ ] Client VLAN 30 nhận IP
- [ ] Gateway/DNS đúng

---

### Story NET-703 — DNS Server

#### Tasks
- [ ] Cài DNS
- [ ] Tạo zone
- [ ] Tạo A Record
- [ ] Test bằng `nslookup` / `dig`

---

### Story NET-704 — NTP

#### Tasks
- [ ] Cài NTP
- [ ] Trỏ router/switch về NTP Server
- [ ] Kiểm tra đồng bộ thời gian

---

### Story NET-705 — Central Syslog

#### Tasks
- [ ] Cấu hình rsyslog/syslog server
- [ ] Trỏ Cisco device về server
- [ ] Tạo test log
- [ ] Kiểm tra log tập trung

---

# Sprint 8 — Monitoring

## EPIC-NET-06 — Network Monitoring

## Công cụ gợi ý
- Zabbix
hoặc
- LibreNMS

## Kiến thức cần học
- SNMP
- Polling
- Trap
- Metrics
- Availability
- Alert

---

### Story NET-801 — SNMP Configuration

#### Tasks
- [ ] Enable SNMP trên router
- [ ] Enable SNMP trên switch
- [ ] Restrict source nếu có thể
- [ ] Test SNMP query

---

### Story NET-802 — Deploy Monitoring Server

#### Tasks
- [ ] Cài Zabbix/LibreNMS
- [ ] Add router
- [ ] Add switch
- [ ] Add server
- [ ] Add firewall

---

### Story NET-803 — Dashboard

Theo dõi:

- CPU
- RAM
- Interface bandwidth
- Interface status
- Device uptime
- Packet loss
- Latency

#### Deliverable
`screenshots/monitoring-dashboard.png`

---

### Story NET-804 — Alerting

#### Tasks
- [ ] Alert device down
- [ ] Alert interface down
- [ ] Alert high CPU
- [ ] Test bằng shutdown interface

---

# Sprint 9 — Network Automation

## EPIC-NET-07 — Automation

## Kiến thức cần học
- SSH
- Python
- YAML
- Inventory
- Ansible
- Idempotency

---

### Story NET-901 — Python Device Connectivity

#### Tasks
- [ ] Tạo management IP
- [ ] Enable SSH
- [ ] Tạo user automation
- [ ] Viết Python connect router
- [ ] Chạy `show ip interface brief`

---

### Story NET-902 — Automatic Config Backup

#### Mục tiêu

```text
Devices
   |
   v
Python
   |
   v
backup/
├── CORE1.cfg
├── CORE2.cfg
├── R1.cfg
└── R2.cfg
```

#### Tasks
- [ ] Connect danh sách thiết bị
- [ ] Execute show running-config
- [ ] Save file
- [ ] Gắn timestamp
- [ ] Error handling

---

### Story NET-903 — Ansible Inventory

#### Tasks
- [ ] Tạo inventory
- [ ] Tạo group router
- [ ] Tạo group switch
- [ ] Test ping/connectivity

---

### Story NET-904 — Ansible Configuration

#### Automation mục tiêu
- NTP
- Syslog
- DNS
- Banner
- VLAN
- Backup config

#### Acceptance Criteria
- [ ] Chạy playbook thành công
- [ ] Chạy lại không gây lỗi
- [ ] Có log output

---

# Sprint 10 — Troubleshooting Engineering

## EPIC-NET-09 — Troubleshooting

## Mục tiêu
Tự tạo lỗi rồi xử lý như một Network Engineer thực tế.

---

### Scenario TRB-001 — VLAN mismatch

**Triệu chứng:** PC không ping được gateway.

**Nhiệm vụ:**
- [ ] Kiểm tra VLAN
- [ ] Kiểm tra access port
- [ ] Fix lỗi
- [ ] Viết Root Cause

---

### Scenario TRB-002 — Native VLAN mismatch

- [ ] Tạo lỗi trunk
- [ ] Quan sát log
- [ ] Fix

---

### Scenario TRB-003 — STP Root sai

- [ ] Thay priority
- [ ] Quan sát traffic path
- [ ] Khôi phục thiết kế

---

### Scenario TRB-004 — EtherChannel mismatch

- [ ] Sai LACP mode
- [ ] Quan sát suspended port
- [ ] Fix

---

### Scenario TRB-005 — HSRP Failover

- [ ] Shutdown Active Router
- [ ] Theo dõi Standby chuyển Active
- [ ] Ghi thời gian gián đoạn

---

### Scenario TRB-006 — OSPF Area Mismatch

- [ ] Tạo mismatch
- [ ] Kiểm tra neighbor
- [ ] Debug
- [ ] Fix

---

### Scenario TRB-007 — OSPF Passive Interface lỗi

- [ ] Đặt passive nhầm interface
- [ ] Xác định nguyên nhân
- [ ] Fix

---

### Scenario TRB-008 — ACL sai direction

- [ ] Apply ACL sai chiều
- [ ] Test
- [ ] Xác định lỗi
- [ ] Fix

---

### Scenario TRB-009 — NAT lỗi

- [ ] Sai inside/outside
- [ ] Kiểm tra translation
- [ ] Fix

---

### Scenario TRB-010 — VPN không lên

Nguyên nhân giả lập:
- PSK mismatch
- Interesting traffic ACL mismatch
- Phase 1 mismatch

#### Deliverable

```text
troubleshooting/
├── vlan-mismatch.md
├── ospf-area-mismatch.md
├── nat-failure.md
├── vpn-failure.md
└── hsrp-failover.md
```

---

# Sprint 11 — Documentation và Portfolio

## EPIC-NET-08 — Documentation & Career Portfolio

---

### Story NET-1101 — Hoàn thiện README

README nên có:

1. Project Overview
2. Business Requirements
3. Architecture
4. Technologies
5. Topology
6. IP Plan
7. Routing Design
8. Security Design
9. Services
10. Monitoring
11. Automation
12. Failover Tests
13. Troubleshooting
14. Screenshots
15. Lessons Learned

---

### Story NET-1102 — Chuẩn hóa Configuration Archive

```text
configs/
├── core/
│   ├── CORE1.cfg
│   └── CORE2.cfg
├── edge/
├── branches/
├── access/
└── firewall/
```

---

### Story NET-1103 — Thu thập bằng chứng

Screenshot cần có:

- [ ] Full topology
- [ ] VLAN
- [ ] Trunk
- [ ] STP
- [ ] EtherChannel
- [ ] HSRP
- [ ] OSPF neighbors
- [ ] BGP table
- [ ] NAT
- [ ] Firewall policy
- [ ] VPN
- [ ] Monitoring dashboard
- [ ] Python backup
- [ ] Ansible output

---

### Story NET-1104 — Viết Project Summary cho CV

Ví dụ:

> **Enterprise Network Infrastructure Lab — EVE-NG**  
> Designed and implemented a multi-site enterprise network with redundant core switching, VLAN segmentation, RSTP, LACP, HSRP, OSPF and eBGP. Implemented firewall segmentation, DMZ, NAT, ACL and Site-to-Site IPSec VPN. Integrated Linux-based DHCP, DNS, NTP and Syslog services, SNMP monitoring, and Python/Ansible-based network automation.

---

# 7. Backlog nâng cao sau khi hoàn thành project chính

## EPIC-ADV-01 — Advanced Routing
- [ ] BGP Local Preference
- [ ] AS-Path Prepending
- [ ] Route Map
- [ ] Prefix List
- [ ] Policy-Based Routing

## EPIC-ADV-02 — Advanced Security
- [ ] AAA
- [ ] TACACS+
- [ ] RADIUS
- [ ] Zone-Based Firewall
- [ ] IDS/IPS

## EPIC-ADV-03 — Advanced Automation
- [ ] Jinja2 templates
- [ ] NetBox
- [ ] REST API
- [ ] Git-based config management

## EPIC-ADV-04 — Enterprise Wireless
- [ ] Corporate WiFi
- [ ] Guest WiFi
- [ ] VLAN mapping
- [ ] WLAN security

## EPIC-ADV-05 — CCNP-Level
- [ ] VRF
- [ ] GRE Tunnel
- [ ] DMVPN
- [ ] MPLS concepts
- [ ] VXLAN/EVPN

---

# 8. Suggested Sprint Schedule

| Sprint | Nội dung | Độ khó |
|---:|---|---|
| 0 | Setup + Design | ★ |
| 1 | VLAN + Trunk + STP | ★★ |
| 2 | EtherChannel + HSRP | ★★ |
| 3 | OSPF | ★★★ |
| 4 | BGP + Dual ISP | ★★★★ |
| 5 | ACL + NAT + Firewall + DMZ | ★★★★ |
| 6 | IPSec VPN | ★★★★ |
| 7 | Linux Services | ★★★ |
| 8 | Monitoring | ★★★ |
| 9 | Automation | ★★★★ |
| 10 | Troubleshooting | ★★★★ |
| 11 | Documentation + CV | ★★ |

---

# 9. Mẫu Jira Workflow

```text
BACKLOG
   ↓
TO DO
   ↓
STUDY
   ↓
IMPLEMENTING
   ↓
TESTING
   ↓
DOCUMENTING
   ↓
DONE
```

## Ý nghĩa

### BACKLOG
Chưa ưu tiên.

### TO DO
Sẽ làm trong Sprint hiện tại.

### STUDY
Đang học lý thuyết trước khi cấu hình.

### IMPLEMENTING
Đang triển khai lab.

### TESTING
Đang kiểm thử.

### DOCUMENTING
Đang viết tài liệu.

### DONE
Đã hoàn tất đủ Definition of Done.

---

# 10. Mẫu Issue Jira

## Story

**Title**

```text
[SWITCHING] Implement VLAN segmentation at HQ
```

**Description**

```text
As a Network Engineer,
I want to separate departments into different VLANs
so that broadcast domains and security boundaries are separated.
```

**Acceptance Criteria**

```text
- VLAN 20 IT created
- VLAN 30 HR created
- VLAN 40 Accounting created
- Access ports assigned correctly
- Same-VLAN hosts can communicate
- Different VLANs cannot communicate before L3 routing
```

---

## Task

```text
Configure VLAN 20 on ACCESS1
```

### Sub-task

```text
Assign Gi0/2 - Gi0/5 to VLAN 20
```

---

# 11. Story Points gợi ý

| Point | Ý nghĩa |
|---:|---|
| 1 | Rất đơn giản |
| 2 | Đơn giản |
| 3 | Trung bình |
| 5 | Khá phức tạp |
| 8 | Phức tạp |
| 13 | Nên chia nhỏ |

Ví dụ:

| Story | Point |
|---|---:|
| VLAN | 3 |
| Trunk | 2 |
| STP | 5 |
| HSRP | 5 |
| OSPF | 5 |
| Multi-Area OSPF | 8 |
| BGP | 8 |
| Firewall | 8 |
| VPN | 8 |
| Monitoring | 5 |
| Automation | 8 |

---

# 12. Daily Learning Template

Mỗi buổi học/lab ghi lại:

```text
## Date

## Topic

## What I learned

## What I configured

## Commands used

## Problems encountered

## Root cause

## Fix

## Evidence

## Questions remaining
```

---

# 13. Mẫu Test Case

```text
Test ID: TC-OSPF-001

Objective:
Verify OSPF adjacency between CORE1 and EDGE1.

Precondition:
Interfaces are UP.

Steps:
1. Run show ip ospf neighbor.
2. Verify EDGE1 appears as FULL.
3. Check OSPF route table.

Expected:
Neighbor state = FULL.

Result:
PASS / FAIL
```

---

# 14. Mẫu Troubleshooting Report

```text
Incident ID: INC-001

Problem:
Branch 1 cannot reach HQ.

Symptoms:
- Local gateway reachable
- HQ network unreachable
- OSPF neighbor absent

Investigation:
show ip ospf neighbor
show ip ospf interface
show run | section ospf

Root Cause:
OSPF Area mismatch.

Resolution:
Changed interface from Area 20 to Area 10.

Verification:
OSPF neighbor reaches FULL state.
Branch can ping HQ.

Lessons Learned:
Always verify area ID on both sides.
```

---

# 15. Tiêu chí hoàn thiện project để đưa CV

Project nên đạt ít nhất:

## Network Design
- [ ] Có topology hợp lý
- [ ] Có IP addressing plan
- [ ] Có redundancy

## Switching
- [ ] VLAN
- [ ] Trunk
- [ ] RSTP
- [ ] EtherChannel

## Routing
- [ ] OSPF
- [ ] BGP

## High Availability
- [ ] HSRP
- [ ] Dual ISP failover

## Security
- [ ] ACL
- [ ] NAT
- [ ] Firewall
- [ ] DMZ
- [ ] VPN

## Services
- [ ] DHCP
- [ ] DNS
- [ ] NTP
- [ ] Syslog

## Monitoring
- [ ] SNMP
- [ ] Dashboard

## Automation
- [ ] Python hoặc Ansible
- [ ] Config backup

## Documentation
- [ ] GitHub README
- [ ] Diagrams
- [ ] Config archive
- [ ] Troubleshooting reports
- [ ] Screenshots

---

# 16. Cách học trong từng Story

Không nên học hết lý thuyết rồi mới làm lab.

Dùng vòng lặp:

```text
Study
  ↓
Small Lab
  ↓
Enterprise Implementation
  ↓
Testing
  ↓
Break It
  ↓
Troubleshoot
  ↓
Document
  ↓
Git Commit
```

Ví dụ với OSPF:

```text
1. Học OSPF Neighbor
2. Lab 2 router
3. Đưa OSPF vào topology Enterprise
4. Test routing
5. Tạo Area mismatch
6. Troubleshoot
7. Viết ospf.md
8. Commit GitHub
```

---

# 17. Milestone

## Milestone 1 — Campus Network
Hoàn thành Sprint 0-2.

Kết quả:
- Switching hoàn chỉnh
- Redundancy Layer 2/3 cơ bản

## Milestone 2 — Routed Enterprise
Hoàn thành Sprint 3-4.

Kết quả:
- HQ ↔ Branch routing
- Internet connectivity
- Dual ISP

## Milestone 3 — Secure Enterprise
Hoàn thành Sprint 5-6.

Kết quả:
- Firewall
- NAT
- DMZ
- VPN

## Milestone 4 — Operational Enterprise
Hoàn thành Sprint 7-9.

Kết quả:
- Infrastructure services
- Monitoring
- Automation

## Milestone 5 — Portfolio Ready
Hoàn thành Sprint 10-11.

Kết quả:
- Troubleshooting reports
- GitHub portfolio
- CV-ready project

---

# 18. Câu hỏi bạn phải tự trả lời được khi phỏng vấn

Sau khi hoàn thành project, hãy chắc chắn bạn trả lời được:

1. Vì sao chia VLAN?
2. Vì sao dùng RSTP?
3. EtherChannel giải quyết vấn đề gì?
4. HSRP hoạt động như thế nào?
5. Vì sao dùng OSPF thay static route?
6. Area 0 có vai trò gì?
7. OSPF cost được tính thế nào?
8. BGP khác OSPF ở đâu?
9. Vì sao doanh nghiệp cần Dual ISP?
10. NAT và PAT khác nhau thế nào?
11. ACL Standard và Extended khác nhau thế nào?
12. DMZ dùng để làm gì?
13. Vì sao Guest không được truy cập Server VLAN?
14. IPSec Phase 1 và Phase 2 khác nhau thế nào?
15. SNMP dùng để làm gì?
16. Syslog khác SNMP ra sao?
17. Khi OSPF neighbor không lên thì kiểm tra gì trước?
18. Khi client không nhận DHCP thì kiểm tra gì?
19. Khi Internet mất nhưng LAN vẫn chạy thì kiểm tra ở đâu?
20. Automation đem lại lợi ích gì cho Network Engineer?

---

# 19. Kết luận

Nếu hoàn thành toàn bộ roadmap này, project của bạn không còn là một bài lab CCNA đơn lẻ mà trở thành một mô hình gần với môi trường Enterprise thực tế.

Quan trọng nhất không phải là cấu hình thật nhiều công nghệ, mà là bạn phải có khả năng:

- Giải thích vì sao thiết kế như vậy
- Cấu hình được
- Kiểm thử được
- Làm hỏng có chủ đích
- Troubleshoot được
- Viết tài liệu được
- Tự động hóa được
- Trình bày được trên GitHub và trong phỏng vấn

Đó mới là phần biến project này thành một project đủ tốt để đưa vào CV.
