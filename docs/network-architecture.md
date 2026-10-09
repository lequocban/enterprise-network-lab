# NET-002 — Thiết kế mạng doanh nghiệp

**Bản thiết kế:** v1.0 — 09/10/2026  
**Hiện trạng:** dựng HQ Campus; các phần Edge, DMZ và Branch triển khai theo Sprint sau.

## 1. Phạm vi

Mô hình có một trụ sở chính (HQ), hai chi nhánh (Branch 1/2), hai đường ISP mô phỏng, firewall ASAv, mạng DMZ và một server Alpine Linux cho dịch vụ nội bộ. Phần HQ dùng hai Core độc lập và hai Access switch, mục tiêu kiểm tra dự phòng và phân đoạn VLAN.

## 2. Kiến trúc logic đã chốt

```text
                ISP1 ------------- ISP2
                  |                  |
                EDGE1              EDGE2
                   \                /
                    [ VLAN 91 OUTSIDE ]
                            |
                         FW1 ASAv ------- DMZ (10.10.100.0/24)
                            |
                    [ VLAN 90 INSIDE ]
                        /         \
                     CORE1 ===== CORE2
                       | \       / |
                       |  \     /  |
                    ACCESS1   ACCESS2
                     IT/HR    ACC/GUEST

          BR1-R1 --- ISP1        BR2-R1 --- ISP2
          HQ–Branch connectivity via future IPSec VPN
```

Hình này mô tả quan hệ logic. ISP và Firewall **chưa được nối vào HQ Campus trong lab hiện tại**. VLAN 90/91 yêu cầu broadcast domain chung theo IP Plan, sẽ được thêm khi dựng Edge.

## 3. HQ Campus đang có trên EVE-NG

| Thiết bị A | Interface | Thiết bị B | Interface | Vai trò |
|---|---|---|---|---|
| CORE1 | e0/0 | CORE2 | e0/0 | Inter-core trunk VLAN 10 test |
| CORE1 | e0/1 | ACCESS1 | e0/0 | Uplink |
| CORE2 | e0/1 | ACCESS1 | e0/1 | Uplink dự phòng |
| CORE1 | e0/2 | ACCESS2 | e0/0 | Uplink |
| CORE2 | e0/2 | ACCESS2 | e0/1 | Uplink dự phòng |
| ACCESS1 | e0/2 | PC-IT (VPCS test) | eth0 | Kiểm thử VLAN 20 |
| ACCESS1 | e0/3 | PC-HR (VPCS test) | eth0 | Kiểm thử VLAN 30 |
| ACCESS2 | e0/2 | PC-ACC (VPCS test) | eth0 | Kiểm thử VLAN 40 |
| ACCESS2 | e0/3 | PC-GUEST (VPCS test) | eth0 | Kiểm thử VLAN 60 |

**PC-MGMT không phải thành phần thiết kế chính.** Ảnh EVE-NG ban đầu có một VPCS tạm thời cắm CORE1 e1/0 để đưa SVI VLAN10 lên và kiểm tra HSRP. Máy này sẽ được tháo khỏi sơ đồ chuẩn; bằng chứng lịch sử vẫn được mô tả trong log Sprint 0 nếu cần.

### Core hoạt động và giới hạn đã nhận biết

- CORE1/CORE2 chạy image Cisco IOL L2 nhưng đã chấp nhận `ip routing`, SVI và cấu hình HSRP.
- VLAN 10: CORE1 10.10.10.2 up/up, CORE2 10.10.10.3 up/up; HSRP VIP 10.10.10.1.
- CORE1 Active priority 110; CORE2 Standby priority 100; bật preempt cả hai.
- Chưa đủ bằng chứng kiểm tra Inter-VLAN Routing VLAN20↔30 và traffic failover end-to-end. Thực hiện ở Sprint 1/2.

## 4. Triển khai dự phòng

Dual uplink từ ACCESS1/2 tới hai Core **không tự động cho phép gom LACP khác Core**. Hai Core độc lập sẽ dùng RSTP để xử lý vòng lặp L2; một số cổng có thể ở trạng thái blocking/discarding. EtherChannel sẽ sử dụng những liên kết có endpoints tương thích, triển khai Sprint 2.

FW1 hiện chỉ có một firewall: không ghi nhận HA cho firewall dù có hai ISP. Khả năng dự phòng của gateway VLAN và tính sẵn sàng end-to-end phải được kiểm thử riêng.

## 5. Danh sách thiết bị triển khai chính

| Nhóm | Node | Trạng thái |
|---|---|---|
| Campus Core | CORE1, CORE2 | Đang dùng |
| Campus Access | ACCESS1, ACCESS2 | Có trong topology HQ |
| WAN/Edge | EDGE1, EDGE2, ISP1, ISP2 | Chưa dựng cùng HQ |
| Security | FW1 (Cisco ASAv) | Image đã cài, policy chưa triển khai |
| Branch | BR1-R1, BR2-R1 | Chưa dựng |
| Service | SRV-INFRA (Alpine) | Đã cài Alpine, dịch vụ sẽ cấu hình Sprint 7 |
| Test nodes | VPCS theo từng bài lab | Không tính vào hạ tầng cố định |

Mọi subnet, VIP, transit và Loopback dùng tài liệu [IP Addressing Plan Final v1.0](ip-addressing-plan.md).
