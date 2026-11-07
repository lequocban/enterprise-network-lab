# NET-002 — Thiết kế mạng doanh nghiệp

**Actual 07/11/2026:** Sprint4/INT-04 đạt kỹ thuật trên topology Campus/WAN/Branch chung 17 node/21 network, đã restore từ ZIP cuối; [báo cáo](sprint-reports/sprint-04-report.md) phân biệt runtime với FW/DMZ/VPN/services còn planned. Các mốc bên dưới giữ lịch sử thiết kế và triển khai.

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
| Internet test | INET-TEST (IOSv, đề xuất 22/10) | Chưa tạo; đề xuất nối hai ISP để endpoint ICMP còn sống khi tắt một ISP; xem thiết kế INT-03 mục 4 |
| Security | FW1 (Cisco ASAv) | Image có sẵn trên EVE, nhưng **chưa có node trong lab HQ gốc**; sẽ thêm vào bản phát triển tích hợp, policy ở Sprint 5 |
| Branch | BR1-R1, BR2-R1 | Chưa dựng |
| Service | SRV-INFRA (Alpine) | Đã cài Alpine, dịch vụ sẽ cấu hình Sprint 7 |
| Test nodes | VPCS theo từng bài lab | Không tính vào hạ tầng cố định |

Mọi subnet, VIP, transit và Loopback dùng tài liệu [IP Addressing Plan Final v1.0](ip-addressing-plan.md).

## 6. Các quyết định thiết kế còn mở

**Ràng buộc tích hợp bổ sung18/10/2026:** theo [các mốc bắt buộc](enterprise-integration-milestones.md), chốt thiết kế WAN-01 cần cho Sprint4–6 tại INT-03 cuối Sprint3; đưa Campus/WAN vào một topology phát triển từ HQ ở INT-04 cuối Sprint4, nối HQ qua FW tại INT-05 cuối Sprint5 và chứng minh client HQ ↔ hai Branch qua VPN tại INT-06 cuối Sprint6. Sandbox Sprint3 chỉ là bước học; mọi thay đổi phải map lại vào kiến trúc đích trước áp dụng. Các hạn dưới là yêu cầu thiết kế trước tích hợp, không bắt buộc dựng FW/VPN trong sandbox.

Theo dõi tại [WAN-01](sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện). Phân bổ subnet đã chốt không đồng nghĩa next-hop, giao thức routing hoặc endpoint VPN đã chốt.

- INT-03, cuối Sprint3 trước triển khai Enterprise Sprint4: chốt cách tạo L2 segment VLAN90/91, cổng/node transit, default/return routes Core–FW–Edge, gateway ASA outside, cơ chế chuyển đường/upstream và Internet Test Server.
- INT-03: chốt đường OSPF Branch/transport, VPN endpoint/type, NAT placement/exemption và failure cases; triển khai kiểm chứng theo INT-04/05/06, không coi quyết định thiết kế là runtime đã đạt.

### Phương án tích hợp chốt để triển khai theo mốc — 23/10/2026

- Một bản phát triển `Enterprise-Network-Integrated` kế thừa [backup HQ có đủ 8 payload](test-results/sprint-02-before-wan-export-2026-10-23.md). Tên là định danh thiết kế; path UNL thật ghi sau khi tạo. HQ gốc chưa có FW.
- INT-04: hai Edge, hai ISP, hai Branch và INET-TEST ở cùng UNL với Campus; BGP dùng doanh nghiệp AS65010, ISP1 AS65101, ISP2 AS65102 và INET-TEST AS65200. EDGE1/2 cùng EVE Bridge T02/VLAN91 cho iBGP; default nhận từ Internet test qua provider, Local Preference 200/100. Chỉ chuyển sang test client HQ khi FW đã nối ở INT-05.
- INT-05: FW1 ASAv9.8(1) routed, Gi0/0 inside T01 `.1`, Gi0/1 outside T02 `.9`, Gi0/2 DMZ `10.10.100.1/24`. Hai Core nối EVE Bridge T01 bằng cổng access VLAN90, dùng SVI `.2/.3` và HSRP `.6`; default Core tới FW `.1`, FW return sáu VLAN HQ /24 qua `.6`. FW default `.10` tracked, `.11` backup; ISP failover do Edge/iBGP. FW dùng PAT outside `.9` cho Internet mô phỏng, identity NAT cho HQ–Branch.
- INT-06: EDGE1↔BR1 VTI Area10, EDGE2↔BR2 VTI Area20; hai Edge cùng Area0 T02. ISP chỉ mang underlay, không tham gia OSPF doanh nghiệp. EDGE1/2 tạo route tĩnh HQ/DMZ qua FW có lọc khi đưa vào OSPF. Edge và Branch có route discard distance250 cho LAN nội bộ nếu OSPF tunnel rút, tránh đi rõ theo default ISP. Một tunnel/site là giới hạn dự phòng VPN trong bài.
- Guest inter-VLAN được chặn trên SVI60 của cả hai Core vì đường đó không đi qua FW. Một ASAv và mỗi EVE Bridge transit là điểm lỗi đơn. [Thiết kế routing/policy và failure cases](routing/enterprise-integration-design.md#5-phương-án-corefwedge-để-kiểm-chứng--23102026) là nguồn chi tiết. ASAv và Core đã qua kiểm parser, chưa có test failover thực tế.
- SRV-INFRA dự kiến ở VLAN50; Access/cổng phải xác định trước bài test Server VLAN. DMZ Web là endpoint dự kiến, chưa có bằng chứng node đang chạy.

Khi chốt mỗi quyết định, ghi phương án, lý do, yêu cầu image, cổng/IP liên quan, failure cases và story tiếp nhận. Chưa cấp thêm VIP ngoài bảng IP v1.0 trong lần chỉnh sửa docs này.

## 7. Trình tự Campus và LACP

Sprint 1 kiểm tra mode STP hiện tại và giữ link dự phòng shutdown lúc chuẩn bị VLAN/trunk; cấu hình/kiểm tra STP trước khi bật đủ uplink, rồi test routing và failover. Không suy ra STP mặc định bị tắt từ việc chưa chạy RSTP.

Sprint 2 cần mỗi channel có ít nhất hai link cùng cặp endpoint. Thiết kế dự kiến thêm một link CORE1–CORE2 và một link CORE1–ACCESS1, tận dụng link cũ để tạo Po1/Po11. Bảng cổng HQ hiện có một link cho mỗi cặp; phải cập nhật physical topology khi thêm link. Không tạo một channel trải qua CORE1 và CORE2 độc lập.
