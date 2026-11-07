# Topology vật lý — bảng nối cổng và kế hoạch mở rộng

**Ngày cập nhật:** 07/11/2026

**Actual Sprint4:** topology tích hợp 17 node/21 network giữ đúng 11 link Campus và T02–T07/W08/W09/hai LAN Branch; [ZIP cuối](../eve-exports/sprint-04-enterprise-final-2026-11-07.zip) và [restore](../docs/test-results/sprint-04-final-restore-2026-11-07.md) đã kiểm. FW/T01/DMZ giữ planned. Các trạng thái HQ riêng và Edge/Branch chờ bên dưới là checkpoint lịch sử.

**Trạng thái hiện tại:** HQ Sprint2 có tám node/mười một link, đã đối chiếu [export18/10](../docs/test-results/sprint-02-export-inspection-2026-10-18.txt) với bốn final running. Mục0 là trạng thái hiện tại; bảng mục1 và các checkpoint ngày trước ở mục3 là lịch sử trước/đang triển khai. `TBD` là chưa xác định, không phải cổng đã tạo. Cabling hợp lệ không đồng nghĩa lỗi dự phòng Po1 đã được sửa.

**Sandbox Sprint 3 — 18/10/2026:** đường riêng S3-L01 theo thiết kế EDGE1 Gi0/0 ↔ EDGE2 Gi0/0, `10.255.0.36/30`. Tiến độ IP/interface/traffic và giới hạn đối chiếu cabling/ảnh/export theo [thiết kế và kết quả sandbox](../docs/routing/sprint-03-routing-design.md). Không cộng hai node/link thử vào inventory HQ hoặc thay các cổng WAN planned bên dưới.

**Checkpoint lab tích hợp:** [export cabling 24/10](../docs/test-results/sprint-04-wan-export-inspection-2026-10-24.md) có 17 node/21 network, toàn bộ cặp cổng đúng bảng mục2 và giữ nguyên 11 mạng HQ. Node ISP thứ hai trong UNL đã sửa từ `ISP12` thành **`ISP2`**; bản export đầu được thay bằng bản mới cùng tên, hash cũ chỉ còn trong lịch sử biên bản. [Buổi 2](../docs/test-results/sprint-04-provider-bgp-2026-10-24.md) đã cấu hình IP/BGP phía ISP1, ISP2, INET-TEST và kiểm ZIP mới; phía Edge/Branch vẫn chờ các buổi sau.

## 0. Cabling Sprint 2 đã kiểm tra

- Po1: CORE1 e0/0↔CORE2 e0/0 (link cũ), CORE1 e0/3↔CORE2 e0/3 (link mới).
- Po11: CORE1 e0/1↔ACCESS1 e0/0 (link cũ), CORE1 e1/1↔ACCESS1 e1/0 (link mới).
- CORE2–ACCESS1, hai trunk tới ACCESS2 và bốn dây VPCS giữ đúng các cổng ở bảng baseline mục1; tổng cộng11 link. Các link cũ thuộc Po1/Po11 nay là member LACP, không phải trunk độc lập.
- CORE1 e1/0 vẫn là cổng test shutdown, không có dây trong export. PC-ACC và PC-GUEST đã về VLAN40/60. Không thêm server VLAN50 cố định từ các bài VPCS tạm.
- [Báo cáo Sprint2](../docs/sprint-reports/sprint-02-report.md) phân biệt runtime cuối hai P, các bài HA chức năng và lỗi tự loại member Po1 đang mở. Chưa thử import/restore ZIP.

Ảnh topology lab gốc `Enterprise-Network-HQ` do người thực hành cung cấp ngày 22/10 xác nhận hiện có CORE1/2, ACCESS1/2 và bốn VPCS; **không có node FW**. Các link đang dùng thể hiện đúng sơ đồ Po1/Po11 và uplink Access. Vì vậy FW1 và các segment transit chỉ được thêm vào bản phát triển riêng, không cấu hình/đấu thêm trên baseline gốc.

## 1. Baseline HQ trước LACP theo ảnh/export Sprint 01

| Thiết bị A | Cổng A | Thiết bị B | Cổng B | Vai trò |
|---|---|---|---|---|
| CORE1 | e0/0 | CORE2 | e0/0 | Inter-core trunk native99, allowed bảy VLAN |
| CORE1 | e0/1 | ACCESS1 | e0/0 | Uplink chính, trunk native99 |
| CORE2 | e0/1 | ACCESS1 | e0/1 | Uplink dự phòng, trunk native99 |
| CORE1 | e0/2 | ACCESS2 | e0/0 | Uplink chính, trunk native99 |
| CORE2 | e0/2 | ACCESS2 | e0/1 | Uplink dự phòng, trunk native99 |
| ACCESS1 | e0/2 | PC-IT, VPCS tạm | eth0 | Test VLAN20 |
| ACCESS1 | e0/3 | PC-HR, VPCS tạm | eth0 | Test VLAN30 |
| ACCESS2 | e0/2 | PC-ACC, VPCS tạm | eth0 | Test VLAN40 |
| ACCESS2 | e0/3 | PC-GUEST, VPCS tạm | eth0 | Test VLAN60 |

Bảng mô tả cabling khớp ảnh HQ ngày 10/10/2026 và ZIP Sprint 01 kiểm tra ngày 11/10/2026. Năm trunk đã có cấu hình hai đầu được review; trạng thái forwarding phải đọc theo VLAN/STP. CORE1 e1/0 không nối endpoint trong export, đã shutdown ở snapshot cuối; bài test VLAN10 cũ giữ trong lịch sử, không bổ sung thành node hạ tầng cố định.

## 2. Kết nối mở rộng dự kiến

| Segment | Thành phần/cổng | Hiện trạng | Mốc chốt |
|---|---|---|---|
| T01 VLAN90 | CORE1 e1/2, CORE2 e1/0, FW Gi0/0 inside cùng EVE Bridge `T01-VLAN90` | Baseline INT-03; chưa tạo bridge/FW trên bản tích hợp, xác nhận cổng khi nối | Triển khai INT-05 |
| T02 VLAN91 | FW Gi0/1 outside, EDGE1 Gi0/0, EDGE2 Gi0/0 cùng EVE Bridge `T02-VLAN91` | EDGE1 `.10/29`, EDGE2 `.11/29` up/up; ping EDGE2→EDGE1 10/10, iBGP Established. EDGE2 best LP200, RIB default/target qua `.10`, trace tới INET qua EDGE1/ISP1. FW chưa nối | Buổi 1 cabling; buổi 3–4 Edge; FW INT-05 |
| T03 | EDGE1 Gi0/1 ↔ ISP1 Gi0/0 | `.17/.18/30` up/up, ping 10/10, eBGP Established; EDGE1 Lo100→INET 10/10 | Buổi 1 cabling; buổi 2 ISP; buổi 3 EDGE1 |
| T04 | EDGE2 Gi0/1 ↔ ISP2 Gi0/0 | `.21/.22/30` up/up, ping 10/10, eBGP Established; chưa đo Lo100 end-to-end sau iBGP | Buổi 1 cabling; buổi 2 ISP; buổi 4 EDGE2 |
| T05 | BR1-R1 Gi0/0 ↔ ISP1 Gi0/1 | `.25/.26/30` up/up, BR1→ISP1 10/10; BR1 `.25`↔EDGE1 `.17` hai chiều 10/10 | Buổi 1 cabling; buổi 2 ISP; buổi 5 Branch |
| T06 | BR2-R1 Gi0/0 ↔ ISP2 Gi0/1 | `.29/.30/30` up/up, BR2→ISP2 10/10; BR2 `.29`↔EDGE2 `.21` hai chiều 10/10. Sau scoped LP200, EDGE2 RIB/trace chiều về T06 qua ISP2 `.22` trực tiếp | Buổi 1 cabling; buổi 2 ISP; buổi 5 Branch |
| T07 | ISP1 Gi0/2 ↔ ISP2 Gi0/2 | `.33/.34/30` up/up, ping hai chiều 10/10, eBGP Established | Buổi 1 cabling; buổi 2 runtime |
| Branch LAN | BR1-R1 Gi0/1 ↔ PC-BR1 eth0; BR2-R1 Gi0/1 ↔ PC-BR2 eth0 | BR1 `.1`/PC `.20` và BR2 `.1`/PC `.20` đúng LAN; mỗi PC→gateway 10/10, hai PC đã `save` | Buổi 1 cabling; buổi 5 runtime |
| DMZ | FW Gi0/2 ↔ DMZ Web eth0, nối trực tiếp | Mapping thiết kế; FW `.1/24`, Web `.10/24`; chưa tạo | Triển khai INT-05 |
| PC-OUT test | VPCS eth0 ↔ EVE Bridge T02/VLAN91 | Chỉ thêm khi thử policy INT-05; `10.255.0.12/29`, gateway FW `.9`; không tính vào hạ tầng | Triển khai INT-05 |
| VLAN50 | Access TBD ↔ SRV-INFRA eth TBD | Planned; Access/cổng chưa chốt | Trước bài test Server VLAN ở Sprint 1 hoặc Sprint 7 nếu chưa dùng |
| W08 / Internet test | ISP1 Gi0/3 ↔ INET-TEST Gi0/0 | `203.0.113.1/.2/30` up/up, ping hai chiều 10/10, eBGP Established; ISP1→Lo100 10/10 | Buổi 1 cabling; buổi 2 runtime |
| W09 / Internet test | ISP2 Gi0/3 ↔ INET-TEST Gi0/1 | `203.0.113.5/.6/30` up/up, ping hai chiều 10/10, eBGP Established; ISP2→Lo100 10/10 | Buổi 1 cabling; buổi 2 runtime |

Phương án WAN/BGP và endpoint ICMP độc lập hai ISP được mô tả trong [thiết kế INT-03 mục 4](../docs/routing/enterprise-integration-design.md#4-phương-án-wanbgp-để-review-trước-sprint-4--22102026). INET-TEST là IOSv thứ bảy trong **thiết kế**, chưa có trong HQ. IP W08/W09/Lo100/Tunnel đã cấp ở [phụ lục INT-03](../docs/ip-addressing-plan.md#9-phụ-lục-int-03-cho-topology-tích-hợp--thiết-kế-23102026). Lo100 Edge dùng làm source test, không phải NAT pool. Đo RAM khi bật nhóm WAN; khi cần thử một đường đi, tất cả node trên đường đó phải cùng chạy.

Theo [mốc tích hợp bổ sung18/10/2026](../docs/enterprise-integration-milestones.md), chốt thiết kế/cổng T01/T02 tại INT-03 cuối Sprint3 trước triển khai Enterprise Sprint4; triển khai FW/transit ở INT-05 cuối Sprint5. Khi chọn Ethernet switch node riêng cho transit, bổ sung số node/cổng vào inventory; chưa tính các node chưa quyết định là đã có.

## 3. Dự phòng và thay đổi cabling

- Sprint 1 đã mở đủ link sau chốt root; trạng thái bình thường Access e0/0 Root/FWD, e0/1 Altn/BLK. Bài thử ACCESS1 và giới hạn ACCESS2 nằm trong [validation](../docs/test-results/sprint-01-validation.md).
- Sprint 2, runtime12/10: [Po1 CORE1–CORE2 đã có e0/0(P) và e0/3(P) ở hai Core](../docs/test-results/sprint-02-po1-progress-2026-10-12.md); neighbor nhận cùng partner với remote Port Number0x1/0x4, phù hợp cặp e0/0 ↔ e0/0 và e0/3 ↔ e0/3 đã hướng dẫn. Bảng mục1 là baseline trước LACP; chưa có ảnh/export mới để đối chiếu toàn bộ cabling. Đích Po1-only là tám node/mười link nếu các link khác giữ nguyên, không ghi ZIP cũ thành mười link.
- Checkpoint Po1: runtime/lưu OK/ping từ12/10 giữ lịch sử, bộ bốn snapshot hoàn thiện nguồn16/10; S2-FWD-01 phục hồi, S2-IOL-01 giữ nguyên. Không có bằng chứng thay đổi cabling trong lượt xác nhận nguồn.
- [Po11 hai member/checkpoint đã lưu16/10](../docs/test-results/sprint-02-po11-progress-2026-10-16.md): CORE1 e0/1 ↔ ACCESS1 e0/0 và CORE1 e1/1 ↔ ACCESS1 e1/0 đều bundled P, peer mapping phù hợp; cổng mới up/up. ACCESS1 VLAN20 dùng Po11 Root/FWD/cost56, đường CORE2 vẫn riêng/Altn BLK; Po1 CORE1 vẫn hai P. Bốn snapshot CORE1/ACCESS1 running-startup đã khớp, không còn bước save/thu lại. Port test CORE1 e1/0 giữ nguyên; tiếp theo thử mất từng member Po11. Ảnh/export mới chưa có; nếu dây khác không đổi dự kiến8 node/11 link, không ghi ZIP cũ thành export mới.
- Không gom uplink tới hai Core độc lập vào một Port-Channel duy nhất.
- Một ASAv không tạo firewall HA. Các failure test phải tách mất Core, uplink, Edge, ISP và FW.

## 4. Điều kiện review

- [x] Bảng HQ khớp 5 link hạ tầng và 4 link VPCS trong ảnh được cung cấp ngày 10/10/2026.
- [x] Export tám node/chín link, config bốn switch và runtime VLAN10 hai Core đã đối chiếu; không suy ra tất cả uplink Access đã trunk/forwarding.
- [ ] Cổng mở rộng được đổi từ TBD sang tên thật đúng thời điểm.
- [x] Logical/physical, IP Plan v1.0 và evidence HQ nhất quán theo closeout 10/10.
- [x] Đã lưu ảnh HQ và liên kết nguồn trong validation; ảnh chỉ xác nhận cabling hiển thị.
- [x] Đã lưu export HQ sạch; WAN vẫn planned. Cổng mở rộng TBD được chốt trước sprint liên quan, không chặn review thiết kế Sprint 0.
