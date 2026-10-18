# NET-002 — Thiết kế mạng doanh nghiệp

**Bản thiết kế:** v1.0 — 09/10/2026  
**Hiện trạng 10/10/2026:** Sprint 0 đã đóng nền tảng với follow-up; HQ tám node/chín link đối chiếu export sạch. Edge, DMZ và Branch vẫn planned.

**Cập nhật 12/10/2026:** Campus Sprint 1 đã chốt thực hành có follow-ups theo [báo cáo](sprint-reports/sprint-01-report.md) và [validation](test-results/sprint-01-validation.md); cabling vẫn tám node/chín link. LACP/HA thuộc Sprint 2 trong [roadmap](enterprise-network-design-implementation-roadmap.md), chưa thêm link hoặc nghiệm thu failover.

**Sơ đồ Markdown:** [Logical topology](../diagrams/logical-topology.md) · [Physical topology](../diagrams/physical-topology.md). Đã review hồ sơ thiết kế và đối chiếu HQ với ảnh/export; mức bằng chứng và việc bàn giao tại [kế hoạch hoàn thiện](sprint-reports/sprint-00-closeout.md).

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
| CORE1 | e0/0 | CORE2 | e0/0 | Inter-core trunk native99, allowed 10/20/30/40/50/60/99 |
| CORE1 | e0/1 | ACCESS1 | e0/0 | Uplink |
| CORE2 | e0/1 | ACCESS1 | e0/1 | Uplink dự phòng |
| CORE1 | e0/2 | ACCESS2 | e0/0 | Uplink |
| CORE2 | e0/2 | ACCESS2 | e0/1 | Uplink dự phòng |
| ACCESS1 | e0/2 | PC-IT (VPCS test) | eth0 | Kiểm thử VLAN 20 |
| ACCESS1 | e0/3 | PC-HR (VPCS test) | eth0 | Kiểm thử VLAN 30 |
| ACCESS2 | e0/2 | PC-ACC (VPCS test) | eth0 | Kiểm thử VLAN 40 |
| ACCESS2 | e0/3 | PC-GUEST (VPCS test) | eth0 | Kiểm thử VLAN 60 |

**PC-MGMT không phải thành phần thiết kế chính.** Ảnh EVE-NG ban đầu có một VPCS tạm thời cắm CORE1 e1/0 để đưa SVI VLAN10 lên và kiểm tra HSRP. Máy này không có trong ảnh/export HQ hiện tại; bằng chứng lịch sử vẫn được mô tả trong log Sprint 0 nếu cần.

### Core hoạt động và giới hạn đã nhận biết

- Snapshot cuối Sprint 1 có sáu SVI/nhóm HSRP VLAN10/20/30/40/50/60, CORE1 `.2`, CORE2 `.3`, VIP `.1`; priority 110/100 và preempt. VLAN99 chỉ L2; cả bốn switch có VLAN/trunk và rapid-pvst theo report.
- Runtime HSRP VLAN10 có CORE1 Active/CORE2 Standby; trạng thái mở rộng và forwarding liên VLAN được người thực hành xác nhận với workaround `no ip cef`. Một số log hai chiều/từng Core và runtime đủ sáu nhóm còn thiếu, giữ mức bằng chứng trong validation Sprint 1.
- Snapshot Core không có `ip routing` tường minh; không suy ra routing tắt hoặc client transit đạt chỉ từ một dòng cấu hình. Sprint 2 kiểm tra routes và traffic client thật, giữ ngoại lệ CEF; đánh giá image trước Sprint 3.
- Full Core/client failover, số đo convergence có timestamp, startup/persistence và restore chưa nghiệm thu đầy đủ; tiếp nhận Sprint 2 trước thao tác tương ứng.

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

## 6. Các quyết định thiết kế còn mở

Theo dõi tại [WAN-01](sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện). Phân bổ subnet đã chốt không đồng nghĩa next-hop, giao thức routing hoặc endpoint VPN đã chốt.

- Trước Sprint 3: cách tạo L2 segment VLAN90/91, cổng/node transit, default route Core → FW và return routes, đường đi của OSPF Branch.
- Trước Sprint 4: gateway ASA outside, cơ chế phát hiện/chuyển đường khi mất Edge hoặc upstream, node/subnet Internet Test Server.
- Trước Sprint 5–6: NAT placement, VPN endpoint/peer, NAT exemption và ảnh hưởng failover tới kết nối.
- SRV-INFRA dự kiến ở VLAN50; Access/cổng phải xác định trước bài test Server VLAN. DMZ Web là endpoint dự kiến, chưa có bằng chứng node đang chạy.

Khi chốt mỗi quyết định, ghi phương án, lý do, yêu cầu image, cổng/IP liên quan, failure cases và story tiếp nhận. Chưa cấp thêm VIP ngoài bảng IP v1.0 trong lần chỉnh sửa docs này.

## 7. Trình tự Campus và LACP

Sprint 1 kiểm tra mode STP hiện tại và giữ link dự phòng shutdown lúc chuẩn bị VLAN/trunk; cấu hình/kiểm tra STP trước khi bật đủ uplink, rồi test routing và failover. Không suy ra STP mặc định bị tắt từ việc chưa chạy RSTP.

Sprint 2 cần mỗi channel có ít nhất hai link cùng cặp endpoint. Thiết kế dự kiến thêm một link CORE1–CORE2 và một link CORE1–ACCESS1, tận dụng link cũ để tạo Po1/Po11. Bảng cổng HQ hiện có một link cho mỗi cặp; phải cập nhật physical topology khi thêm link. Không tạo một channel trải qua CORE1 và CORE2 độc lập.
