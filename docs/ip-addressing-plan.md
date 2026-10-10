# NET-003 — IP Addressing Plan (Final v1.0)

**Ngày chốt:** 09/10/2026  
**Phạm vi:** Thiết kế địa chỉ toàn hệ thống Enterprise Network Lab.  
**Trạng thái:** **FINAL DESIGN** — cố định để triển khai theo Sprint; không đồng nghĩa tất cả VLAN/WAN đã được cấu hình.

## 1. Quy ước địa chỉ

- Mạng HQ: `10.10.0.0/16`; Branch 1: `10.20.0.0/16`; Branch 2: `10.30.0.0/16`.
- Kết nối transit/WAN nội bộ lab: `10.255.0.0/24`. Loopback: `10.255.255.0/24`, gán từng địa chỉ /32.
- Mỗi VLAN HQ dùng gateway HSRP .1; IP SVI CORE1 .2; CORE2 .3. VLAN 10 đã kiểm tra CLI; các VLAN còn lại triển khai từ Sprint 1.
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

**Đã kiểm tra:** VLAN 10 CORE1/CORE2 up/up, HSRP group 10 Active/Standby, virtual gateway 10.10.10.1. Các giá trị VLAN 20–60 là thiết kế đã chốt, chưa phải kết quả chạy lab.

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

## 5. Loopback /32

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

**Công việc triển khai còn lại:** tạo các interface VLAN 20–60, dựng thêm các node Edge/ISP/Firewall/Branch, xác nhận chính xác interface vật lý và kiểm thử connectivity/segmentation theo Sprint. Thiết kế được chốt trước để tránh đổi IP giữa các bài lab.

## 7. Phạm vi đã chốt và quyết định cần bổ sung

Final v1.0 chốt **phân bổ địa chỉ**. Default/return routes, gateway ASA outside và cơ chế failover, đường OSPF Branch và endpoint VPN còn mở tại [WAN-01](sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện). Chưa thêm VIP phía Edge hoặc đổi vai trò gateway khi chưa có quyết định tương ứng.

Lab router tạm dùng `10.255.0.0/30` trong validation lịch sử phải chạy tách biệt T01 `10.255.0.0/29`; không ghép hai bài với địa chỉ .1/.2 trùng. Không dùng subnet test tạm để thay bảng transit này.

Kết quả review tĩnh nằm ở [Validation Sprint 0](test-results/sprint-00-validation.md#6-review-tĩnh-ip-plan); không thay thế kiểm thử runtime. Topology dự kiến và cổng cần đối chiếu tại [Logical](../diagrams/logical-topology.md) và [Physical](../diagrams/physical-topology.md).
