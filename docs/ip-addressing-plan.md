# NET-003 — Enterprise IP Addressing Plan

> Project: Enterprise Network Design & Implementation (EVE-NG)
>
> Status: **DRAFT — chờ xác nhận WAN/Firewall/Branch topology**
>
> Cơ sở: Topology HQ do người thực hành cung cấp ngày 2026-10-09; các IP WAN/DMZ/Branch bên dưới là đề xuất, chưa có trong hình hiện tại.

## 1. Design rules

- HQ Core: CORE1 và CORE2 sử dụng SVI cho VLAN 10/20/30/40/50/60.
- HSRP: địa chỉ `.1` làm virtual gateway; CORE1 `.2`; CORE2 `.3` trong mỗi VLAN /24.
- CORE1 ưu tiên Active (priority 110), CORE2 Standby (priority 100); xác minh bằng failover khi cấu hình từng VLAN.
- VLAN 99: native VLAN cho trunk nội bộ, không cấp subnet/management IP.
- Trunk giữa các switch **không được gán IP trực tiếp**. Cần mở rộng danh sách VLAN cho phép ở Sprint 1 (ban đầu inter-core trunk chỉ cho VLAN 10).
- Hai uplink ACCESS1/ACCESS2 đến hai Core riêng biệt: chạy RSTP; không gộp chúng vào một EtherChannel thông thường nếu không có multi-chassis EtherChannel.
- PC-MGMT hiện cắm trực tiếp CORE1; nên chuyển về access switch để kiểm thử core failover có ý nghĩa.
- 10.255.0.0/24 là pool transit; 10.255.255.0/24 là pool loopback. Dùng IP private cho lab, không coi là địa chỉ Internet công cộng.

## 2. HQ VLAN / SVI / HSRP

| VLAN | Name | Subnet | HSRP VIP | CORE1 SVI | CORE2 SVI |
|---:|---|---|---|---|---|
| 10 | MGMT | 10.10.10.0/24 | 10.10.10.1 | 10.10.10.2 | 10.10.10.3 |
| 20 | IT | 10.10.20.0/24 | 10.10.20.1 | 10.10.20.2 | 10.10.20.3 |
| 30 | HR | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.2 | 10.10.30.3 |
| 40 | ACCOUNTING | 10.10.40.0/24 | 10.10.40.1 | 10.10.40.2 | 10.10.40.3 |
| 50 | SERVER | 10.10.50.0/24 | 10.10.50.1 | 10.10.50.2 | 10.10.50.3 |
| 60 | GUEST | 10.10.60.0/24 | 10.10.60.1 | 10.10.60.2 | 10.10.60.3 |
| 99 | NATIVE | Không cấp IP | — | — | — |

**Lưu ý:** VLAN 10 đã có cấu hình HSRP thành công; các VLAN còn lại là kế hoạch dành cho Sprint 1–2.

## 3. HQ hosts / management

| Node | Proposed IP | Mask | Default gateway | Notes |
|---|---|---|---|---|
| PC-MGMT | 10.10.10.10 | /24 | 10.10.10.1 | Hiện nối CORE1 e1/0 |
| ACCESS1 | 10.10.10.11 | /24 | 10.10.10.1 | Management SVI (L2) |
| ACCESS2 | 10.10.10.12 | /24 | 10.10.10.1 | Management SVI (L2) |
| PC-IT | 10.10.20.10 | /24 | 10.10.20.1 | ACCESS1 e0/2 |
| PC-HR | 10.10.30.10 | /24 | 10.10.30.1 | ACCESS1 e0/3 |
| PC-ACC | 10.10.40.10 | /24 | 10.10.40.1 | ACCESS2 e0/2 |
| PC-GUEST | 10.10.60.10 | /24 | 10.10.60.1 | ACCESS2 e0/3 |
| SRV1-Alpine | 10.10.50.10 | /24 | 10.10.50.1 | Sẽ kết nối ở Sprint 7 |

## 4. Actual HQ interface mapping — verified from supplied EVE-NG screenshot

| Device A | Port A | Device B | Port B | Intended link type |
|---|---|---|---|---|
| CORE1 | e0/0 | CORE2 | e0/0 | Inter-core 802.1Q trunk |
| CORE1 | e0/1 | ACCESS1 | e0/0 | 802.1Q trunk |
| CORE2 | e0/1 | ACCESS1 | e0/1 | 802.1Q trunk |
| CORE1 | e0/2 | ACCESS2 | e0/0 | 802.1Q trunk |
| CORE2 | e0/2 | ACCESS2 | e0/1 | 802.1Q trunk |
| CORE1 | e1/0 | PC-MGMT | eth0 | Access VLAN 10 (temporary) |
| ACCESS1 | e0/2 | PC-IT | eth0 | Access VLAN 20 |
| ACCESS1 | e0/3 | PC-HR | eth0 | Access VLAN 30 |
| ACCESS2 | e0/2 | PC-ACC | eth0 | Access VLAN 40 |
| ACCESS2 | e0/3 | PC-GUEST | eth0 | Access VLAN 60 |

**Cảnh báo:** Bảng chỉ xác nhận cổng/kết nối từ ảnh, không xác nhận tất cả switchport đã được cấu hình trunk/VLAN. Các loại port ghi phía trên là **design intent**.

## 5. Branch / DMZ (proposed)

| Zone | Subnet | Proposed gateway | Gateway owner |
|---|---|---|---|
| Branch 1 Users | 10.20.10.0/24 | 10.20.10.1 | B1 router |
| Branch 2 Users | 10.30.10.0/24 | 10.30.10.1 | B2 router |
| HQ DMZ | 10.10.100.0/24 | 10.10.100.1 | ASAv DMZ interface |
| DMZ Web Server | 10.10.100.0/24 | 10.10.100.1 | Proposed host IP: 10.10.100.10 |

## 6. WAN / Firewall transit proposal — not yet wired

> Các mạng dưới đây được cấp phát không chồng lấn từ 10.255.0.0/24. Chỉ triển khai khi đã vẽ và kiểm tra topology WAN/Firewall tương ứng. Multi-access /29 cần một broadcast domain chung (thường qua switch L2), không thể nối 3 cổng trực tiếp theo kiểu point-to-point mà vẫn thuộc cùng một LAN.

| Segment | Prefix | Devices / addresses | Design note |
|---|---|---|---|
| CORE ↔ ASAv INSIDE | 10.255.0.0/29 | FW inside `.1`, CORE1 SVI `.2`, CORE2 SVI `.3`, HSRP VIP `.6` | Cần VLAN transit (đề xuất VLAN 70) được CORE1/CORE2 cùng truy cập; FW route HQ về VIP `.6` |
| ASAv OUTSIDE ↔ EDGE1/EDGE2 | 10.255.0.8/29 | FW outside `.9`, EDGE1 `.10`, EDGE2 `.11` | Cần shared L2 segment / WAN switch riêng; ASA lựa chọn route theo policy/failover sau này |
| EDGE1 ↔ ISP1 | 10.255.0.16/30 | EDGE1 `.17`, ISP1 `.18` | Point-to-point |
| EDGE2 ↔ ISP2 | 10.255.0.20/30 | EDGE2 `.21`, ISP2 `.22` | Point-to-point |
| Branch1 ↔ ISP1 | 10.255.0.24/30 | B1 `.25`, ISP1 `.26` | Point-to-point, đề xuất |
| Branch2 ↔ ISP2 | 10.255.0.28/30 | B2 `.29`, ISP2 `.30` | Point-to-point, đề xuất |

### Important implementation notes

- Không thay đổi kết nối CORE1 e0/0 ↔ CORE2 e0/0 đang dùng trunk/HSRP thành routed port.
- Inside transit VLAN 70 có thể nối ASAv vào một switch L2 được dual-homed lên 2 Core, hoặc một switch transit riêng. Nếu ASA chỉ cắm CORE1, core failover **không** bảo vệ được đường vào firewall khi CORE1 tắt.
- ASA outside và EDGE1/EDGE2 dùng một broadcast domain riêng, tách khỏi VLAN người dùng; tránh bridge trực tiếp vùng ngoài vào campus VLAN.
- Đây là topology ban đầu có một firewall (single point of failure). Không gọi đây là HA firewall.
- Các interface ASAv/EDGE/ISP chưa thể điền số cổng từ ảnh HQ hiện tại; sẽ chốt tại sơ đồ WAN/DMZ.
- Chế độ eBGP/dual ISP và public simulation/test prefixes sẽ được chốt ở Sprint 4. Có thể dùng RFC 5737 documentation prefixes để mô phỏng IP public, nhưng không định tuyến thật ra Internet.

## 7. Loopback plan (proposed)

| Node | Loopback /32 |
|---|---|
| CORE1 | 10.255.255.1/32 |
| CORE2 | 10.255.255.2/32 |
| EDGE1 | 10.255.255.11/32 |
| EDGE2 | 10.255.255.12/32 |
| BRANCH1-R1 | 10.255.255.21/32 |
| BRANCH2-R1 | 10.255.255.22/32 |

## 8. NET-003 Acceptance Checklist

- [x] HQ VLAN/subnet/gateway design
- [x] HQ user/management/static server IP proposals
- [x] Interface mapping from actual HQ EVE-NG topology
- [x] Branch LAN, DMZ subnet proposals
- [x] Non-overlapping WAN transit subnet allocation **proposal**
- [x] Loopback /32 allocation proposal
- [ ] Confirm detailed WAN/Firewall/Branch node/interface topology
- [ ] Commit this file at `docs/ip-addressing-plan.md` in the project repository
- [ ] Validate actual inter-VLAN routing, HSRP failover, security policies in subsequent sprints

**Status rule:** NET-003 hoàn thành phần thiết kế sơ bộ; chưa đóng Story nếu WAN topology/commit chưa được xác nhận.
