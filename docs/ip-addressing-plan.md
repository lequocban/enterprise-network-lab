# NET-003 — IP Addressing Plan (Final v1.0)

**Actual Sprint4, 07/11/2026:** Campus, T02–T07, W08/W09, Lo0/Lo100 và LAN hai Branch đã đối chiếu runtime/ZIP/restore theo [báo cáo Sprint4](sprint-reports/sprint-04-report.md). T01/FW/DMZ/overlay giữ phạm vi thiết kế cho sprint tiếp theo; không gán trạng thái runtime cho toàn IP Plan.

**Ngày chốt:** 09/10/2026  
**Phạm vi:** Thiết kế địa chỉ toàn hệ thống Enterprise Network Lab.  
**Trạng thái:** **FINAL DESIGN** — cố định để triển khai theo Sprint; không đồng nghĩa tất cả VLAN/WAN đã được cấu hình.

## 1. Quy ước địa chỉ

- Mạng HQ: `10.10.0.0/16`; Branch 1: `10.20.0.0/16`; Branch 2: `10.30.0.0/16`.
- Kết nối transit/WAN nội bộ lab: `10.255.0.0/24`. Loopback: `10.255.255.0/24`, gán từng địa chỉ /32.
- Mỗi VLAN HQ dùng gateway HSRP .1; IP SVI CORE1 .2; CORE2 .3. Cấu hình sáu VLAN10–60 đã đối chiếu trong Sprint 1; runtime và mức bằng chứng theo báo cáo/validation, không đồng nghĩa HA đã đạt.
- VLAN 99 là native VLAN Layer 2; **không cấp IP và không dùng làm VLAN quản trị**.
- Máy VPCS chỉ là thiết bị kiểm thử, **không tính trong số lượng node hạ tầng, không dành riêng một PC-MGMT**.
- Tất cả địa chỉ WAN bên dưới là private, chỉ dùng trong môi trường mô phỏng ISP/VPN, không coi là IP Internet public.

## 2. HQ: VLAN, SVI, gateway

| VLAN | Tên | Subnet | HSRP VIP | CORE1 | CORE2 |
|---:|---|---|---|---|---|
| 10 | MGMT | 10.10.10.0/24 | 10.10.10.1 | 10.10.10.2 | 10.10.10.3 |
| 20 | IT | 10.10.20.0/24 | 10.10.20.1 | 10.10.20.2 | 10.10.20.3 |
| 30 | HR | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.2 | 10.10.30.3 |
| 40 | ACCOUNTING | 10.10.40.0/24 | 10.10.40.1 | 10.10.40.2 | 10.10.40.3 |
| 50 | SERVER | 10.10.50.0/24 | 10.10.50.1 | 10.10.50.2 | 10.10.50.3 |
| 60 | GUEST | 10.10.60.0/24 | 10.10.60.1 | 10.10.60.2 | 10.10.60.3 |
| 99 | NATIVE | Không cấp subnet | — | — | — |

**Cập nhật trạng thái 12/10/2026:** VLAN10 có runtime Core up/up và HSRP Active/Standby; snapshot Sprint 1 có đầy đủ SVI/VIP VLAN10–60 theo bảng. Forwarding mở rộng được người thực hành xác nhận với `no ip cef`, một số log/runtime còn thiếu theo [báo cáo Sprint 1](sprint-reports/sprint-01-report.md) và [validation](test-results/sprint-01-validation.md). Full HA tiếp nhận Sprint 2 theo [roadmap](enterprise-network-design-implementation-roadmap.md), hiện chưa kiểm thử; phân bổ địa chỉ Final v1.0 giữ nguyên.

### Địa chỉ quản trị và server

| Thiết bị | Địa chỉ | Gateway | Ghi chú |
|---|---|---|---|
| ACCESS1 | 10.10.10.11/24 | 10.10.10.1 | Management SVI |
| ACCESS2 | 10.10.10.12/24 | 10.10.10.1 | Management SVI |
| SRV-INFRA (Alpine) | 10.10.50.10/24 | 10.10.50.1 | DHCP/DNS/NTP/Syslog triển khai Sprint 7 |

VPCS phục vụ test nhận IP tĩnh/DHCP trong từng VLAN theo bài thực hành; không khóa địa chỉ IP cho máy thử nghiệm trong tài liệu hạ tầng.

## 3. DMZ và hai chi nhánh

| Vùng | Mạng | Thiết bị/cổng gateway | Ghi chú |
|---|---|---|---|
| DMZ | 10.10.100.0/24 | FW1/ASAv 10.10.100.1 | DMZ do firewall quản lý |
| DMZ Web | 10.10.100.10/24 | Gateway 10.10.100.1 | Web server giả lập |
| Branch 1 LAN | 10.20.10.0/24 | BR1-R1 10.20.10.1 | Site LAN |
| Branch 2 LAN | 10.30.10.0/24 | BR2-R1 10.30.10.1 | Site LAN |

Không đặt SVI DMZ trên CORE1/CORE2. Truy cập DMZ đi qua ASAv và policy được triển khai ở Sprint 5.

## 4. Transit và simulated WAN: cấp phát cố định

| ID | Segment / link | Subnet | Thiết bị và địa chỉ |
|---|---|---|---|
| T01 | HQ Core ↔ FW Inside (VLAN 90) | 10.255.0.0/29 | FW1 .1; CORE1 .2; CORE2 .3; HSRP VIP .6 |
| T02 | FW Outside ↔ EDGE1/EDGE2 (VLAN 91) | 10.255.0.8/29 | FW1 .9; EDGE1 .10; EDGE2 .11 |
| T03 | EDGE1 ↔ ISP1 | 10.255.0.16/30 | EDGE1 .17; ISP1 .18 |
| T04 | EDGE2 ↔ ISP2 | 10.255.0.20/30 | EDGE2 .21; ISP2 .22 |
| T05 | BR1-R1 ↔ ISP1 | 10.255.0.24/30 | BR1-R1 .25; ISP1 .26 |
| T06 | BR2-R1 ↔ ISP2 | 10.255.0.28/30 | BR2-R1 .29; ISP2 .30 |
| T07 | ISP1 ↔ ISP2 (simulated provider backbone) | 10.255.0.32/30 | ISP1 .33; ISP2 .34 |

**VLAN 90, 91 là các phân đoạn Layer 2 đa truy cập, không phải đường point-to-point:** cần xây dựng kết nối chung thích hợp (ví dụ Ethernet switch node riêng) trước khi cấu hình. Đặc biệt ASA outside và hai EDGE cần cùng VLAN 91. Đường EDGE1/2 lên ISP và từ ISP đến Branch là /30 point-to-point. Số cổng vật lý của nhóm WAN sẽ được ghi trong `docs/network-architecture.md` khi thêm node vào EVE-NG.

**Định hướng routing/VPN:** các router BR kết nối đến ISP mô phỏng, IPsec Site-to-Site giữa HQ và Branch triển khai qua mạng ISP. ISP1↔ISP2 nối nhau để kiểm thử đường đi qua mạng provider giả lập. Chưa xác nhận routing/BGP/IPsec.

Dải `10.255.0.36–10.255.0.255` được giữ để mở rộng. Địa chỉ .0/.7 của /29 và .8/.15 là network/broadcast tương ứng; không cấp cho interface.

**Ngoại lệ thực hành Sprint 3 — 18/10/2026:** dành `10.255.0.36/30` từ phần dự trữ cho S3-L01 trong lab routing riêng: EDGE1 Gi0/0 `10.255.0.37`, EDGE2 Gi0/0 `10.255.0.38`. Nguồn, tiến độ cấu hình/traffic và giới hạn bằng chứng theo [thiết kế và kết quả sandbox](routing/sprint-03-routing-design.md). Không dùng segment này thay T01–T07 hoặc nối sandbox vào HQ. Dải `.40–.255` tiếp tục dự trữ; các subnet sandbox mở rộng chưa được cấp trong checkpoint này. Loopback0 hai Edge giữ `.11/.12` theo mục 5, phạm vi dùng ở lab tách biệt.

## 5. Loopback /32

**Bổ sung sandbox buổi2 — 20/10/2026:** sau đối chiếu cabling, dành thêm S3-L02 `10.255.0.40/30` (EDGE1 `.41`, BR1 `.42`), S3-L03 `10.255.0.44/30` (EDGE1 `.45`, BR2 `.46`) và S3-L05 `10.255.0.52/30` (EDGE2 `.53`, PC-HQ `.54`). [Thiết kế sandbox mục6](routing/sprint-03-routing-design.md#6-buổi-2--cablingcapability-đã-đối-chiếu-ip-mở-rộng-chờ-thực-hành) ghi cổng/IP, gateway và trạng thái chưa cấu hình. S3-L04 `.48/30` vẫn dành cho bài ISP sau, chưa dùng. Branch LAN và Loopback0 giữ đúng bảng v1.0; không thay T01–T07 hoặc nối sandbox vào topology Enterprise. Nội dung chưa cấp các subnet mở rộng ở ghi chú18/10 mô tả checkpoint cũ, được cập nhật bởi ghi chú này.

| Thiết bị | Loopback | Mục đích |
|---|---|---|
| CORE1 | 10.255.255.1/32 | Router ID |
| CORE2 | 10.255.255.2/32 | Router ID |
| EDGE1 | 10.255.255.11/32 | Router ID |
| EDGE2 | 10.255.255.12/32 | Router ID |
| ISP1 | 10.255.255.101/32 | Provider simulated router ID |
| ISP2 | 10.255.255.102/32 | Provider simulated router ID |
| BR1-R1 | 10.255.255.21/32 | Router ID |
| BR2-R1 | 10.255.255.22/32 | Router ID |

## 6. Kiểm tra thiết kế và nguyên tắc thay đổi

- Không có subnet nào trong T01–T07 chồng lấn nhau.
- SVI HQ, Branch LAN, DMZ, Transit và Loopback nằm trong các nhóm dải riêng.
- Native VLAN 99 không định tuyến; Guest sẽ bị giới hạn bằng ACL ở Sprint 5, **không phải chỉ nhờ VLAN**.
- Không triển khai LACP từ ACCESS1/2 tới hai Core độc lập như một Port-Channel duy nhất; sử dụng RSTP cho dual uplink.
- Sau bản **Final v1.0**, bất kỳ đổi subnet, VIP hoặc vai trò gateway nào phải ghi lý do và cập nhật cả bảng kết nối, cấu hình và test cases trong cùng một PR.

**Công việc triển khai còn lại:** kiểm chứng runtime/full HA các SVI/VIP đã cấu hình, dựng thêm các node Edge/ISP/Firewall/Branch, xác nhận chính xác interface vật lý và kiểm thử connectivity/segmentation theo Sprint. Thiết kế được chốt trước để tránh đổi IP giữa các bài lab.

## 7. Phạm vi đã chốt và quyết định cần bổ sung

Final v1.0 chốt **phân bổ địa chỉ**. Default/return routes, gateway ASA outside và cơ chế failover, đường OSPF Branch và endpoint VPN còn mở tại [WAN-01](sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện). Chưa thêm VIP phía Edge hoặc đổi vai trò gateway khi chưa có quyết định tương ứng.

Lab router tạm dùng `10.255.0.0/30` trong validation lịch sử phải chạy tách biệt T01 `10.255.0.0/29`; không ghép hai bài với địa chỉ .1/.2 trùng. Không dùng subnet test tạm để thay bảng transit này.

Kết quả review tĩnh nằm ở [Validation Sprint 0](test-results/sprint-00-validation.md#6-review-tĩnh-ip-plan); không thay thế kiểm thử runtime. Topology dự kiến và cổng cần đối chiếu tại [Logical](../diagrams/logical-topology.md) và [Physical](../diagrams/physical-topology.md).

## 8. Ngoại lệ mạng thử summarization Sprint 3

**Mốc mở rộng buổi6 — 21/10/2026:** dành S3-L04 `10.255.0.48/30` đã dự trữ cho EDGE2Gi0/1 `.49` ↔ ISP1Gi0/0 `.50`; ISP1Loopback100 `192.0.2.10/32` làm đích ngoài mô phỏng, Loopback0 giữ `10.255.255.101/32` theo mục5. [Thiết kế sandbox mục11](routing/sprint-03-routing-design.md) ghi routing/return path và trạng thái chưa triển khai. Không thay T03/T04 hoặc dùng endpoint này làm Internet Test Server Enterprise đã nghiệm thu. Các ghi chú “S3-L04 chưa dùng” trước đó mô tả checkpoint20/10, được mở rộng theo mốc này; chưa có actual IP/ping mới.

**Dành cho bước NET-303 ngày21/10/2026, chưa có output cấu hình:** BR1 Loopback110 `10.20.11.1/24` tạo prefix thử `10.20.11.0/24` trongArea10; BR2 Loopback110 `10.30.11.1/24` tạo `10.30.11.0/24` trongArea20. Đây là mạng mô phỏng riêng trong sandbox, không phải VLAN/host/server Enterprise đã triển khai; không thay LAN/gatewayBranch hiện tại hoặc Loopback0/router-id /32 ở mục5.

Range dự kiến tại ABR EDGE1: `10.20.10.0/23` và `10.30.10.0/23`, mỗi range chứa LAN/24 và prefixLoopback110/24 tương ứng. Phải kiểm route mask/traffic trước và sau theo [thiết kế sandbox](routing/sprint-03-routing-design.md), không coi việc dành địa chỉ này là summary đã chạy hoặc cấp toàn bộ/16 cho interface.

## 9. Phụ lục INT-03 cho topology tích hợp — thiết kế 23/10/2026

Bảng T01–T07 và HQ VLAN v1.0 vẫn là nguồn cấp phát chính. Các địa chỉ sau chỉ dùng trong bản phát triển Enterprise, không mang các link S3-Lxx của sandbox sang topology đó. [Thiết kế tích hợp](routing/enterprise-integration-design.md) quy định vai trò route, NAT, BGP và OSPF.

**Baseline INT-03 đã chốt:** W08/W09, I01, E01/E02, V01/V02, ASN và mapping T01/T02 bên dưới là phân bổ triển khai cho `Enterprise-Network-Integrated`; chưa phải cấu hình đã chạy. Thay đổi IP/cổng phải cập nhật bảng và physical topology trước khi áp lên thiết bị.

| ID | Mạng/địa chỉ | Thiết bị | Mục đích |
|---|---|---|---|
| W08 | `203.0.113.0/30` | ISP1 Gi0/3 `.1`, INET-TEST Gi0/0 `.2` | Uplink đích test tới ISP1 |
| W09 | `203.0.113.4/30` | ISP2 Gi0/3 `.5`, INET-TEST Gi0/1 `.6` | Uplink đích test tới ISP2 |
| I01 | `192.0.2.10/32` | INET-TEST Loopback100 | Cùng một đích ICMP qua hai ISP |
| E01 | `198.51.100.11/32` | EDGE1 Loopback100 | Nguồn thử BGP, không phải NAT pool |
| E02 | `198.51.100.12/32` | EDGE2 Loopback100 | Nguồn thử BGP, không phải NAT pool |
| V01 | `10.255.0.56/30` | EDGE1 Tunnel0 `.57`, BR1-R1 Tunnel0 `.58` | VPN HQ–BR1, OSPF Area10 |
| V02 | `10.255.0.60/30` | EDGE2 Tunnel0 `.61`, BR2-R1 Tunnel0 `.62` | VPN HQ–BR2, OSPF Area20 |

Các dải `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` được [IANA dành cho tài liệu/ví dụ](https://www.iana.org/assignments/iana-ipv4-special-registry). Chúng chỉ đóng vai địa chỉ Internet mô phỏng trong EVE; không quảng bá ra Internet thật. Các subnet VPN V01/V02 cùng dải dự trữ với một số link sandbox nhưng nằm ở lab khác và không chồng T01–T07.

ASN lab: doanh nghiệp `65010` (EDGE1/EDGE2), ISP1 `65101`, ISP2 `65102`, INET-TEST `65200`. T02 `10.255.0.8/29` được hai Edge quảng bá có kiểm soát tới ISP/INET-TEST để có đường trả về địa chỉ PAT FW `.9`; không xuất các LAN HQ/Branch/DMZ ra BGP provider. T01 `.0/29`: FW inside Gi0/0 `.1`, CORE1 SVI90 `.2`, CORE2 SVI90 `.3`, HSRP `.6`. T02 `.8/29`: FW outside Gi0/1 `.9`, EDGE1 `.10`, EDGE2 `.11`. VPCS `PC-OUT` dùng riêng INT-05 có thể nhận `.12/29`, gateway `.9` để thử outside→LAN deny trên chính FW; không tính là node hạ tầng. FW DMZ Gi0/2 `10.10.100.1/24`.

Default của hai Core hướng FW `.1`; FW trả sáu VLAN HQ /24 qua VIP `.6`. FW default ưu tiên EDGE1 `.10` có SLA track, backup EDGE2 `.11`; route tới BR1 ưu tiên `.10`, BR2 ưu tiên `.11` theo thiết kế. Gateway Branch underlay giữ ISP1 `.26` và ISP2 `.30`. Các địa chỉ/route này là **thiết kế**, chưa có cấu hình runtime trên topology Enterprise. Route discard AD250 và NAT exemption/failover được mô tả trong thiết kế tích hợp; không thêm IP gateway mới.
