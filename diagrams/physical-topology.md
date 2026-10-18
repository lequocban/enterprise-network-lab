# Topology vật lý — bảng nối cổng và kế hoạch mở rộng

**Ngày cập nhật:** 18/10/2026

**Trạng thái hiện tại:** HQ Sprint2 có tám node/mười một link, đã đối chiếu [export18/10](../docs/test-results/sprint-02-export-inspection-2026-10-18.txt) với bốn final running. Mục0 là trạng thái hiện tại; bảng mục1 và các checkpoint ngày trước ở mục3 là lịch sử trước/đang triển khai. `TBD` là chưa xác định, không phải cổng đã tạo. Cabling hợp lệ không đồng nghĩa lỗi dự phòng Po1 đã được sửa.

## 0. Cabling Sprint 2 đã kiểm tra

- Po1: CORE1 e0/0↔CORE2 e0/0 (link cũ), CORE1 e0/3↔CORE2 e0/3 (link mới).
- Po11: CORE1 e0/1↔ACCESS1 e0/0 (link cũ), CORE1 e1/1↔ACCESS1 e1/0 (link mới).
- CORE2–ACCESS1, hai trunk tới ACCESS2 và bốn dây VPCS giữ đúng các cổng ở bảng baseline mục1; tổng cộng11 link. Các link cũ thuộc Po1/Po11 nay là member LACP, không phải trunk độc lập.
- CORE1 e1/0 vẫn là cổng test shutdown, không có dây trong export. PC-ACC và PC-GUEST đã về VLAN40/60. Không thêm server VLAN50 cố định từ các bài VPCS tạm.
- [Báo cáo Sprint2](../docs/sprint-reports/sprint-02-report.md) phân biệt runtime cuối hai P, các bài HA chức năng và lỗi tự loại member Po1 đang mở. Chưa thử import/restore ZIP.

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
| T01 VLAN90 | CORE1 TBD, CORE2 TBD, FW inside TBD | Chưa tạo segment chung; lựa chọn L2 node/cổng còn mở | Trước Sprint 3 |
| T02 VLAN91 | FW outside TBD, EDGE1 TBD, EDGE2 TBD | Chưa tạo segment chung; lựa chọn L2 node/cổng còn mở | Trước Sprint 3 |
| T03 | EDGE1 TBD ↔ ISP1 TBD | Planned | Trước Sprint 3/4 |
| T04 | EDGE2 TBD ↔ ISP2 TBD | Planned | Trước Sprint 3/4 |
| T05 | BR1-R1 TBD ↔ ISP1 TBD | Planned | Trước Sprint 3/4 |
| T06 | BR2-R1 TBD ↔ ISP2 TBD | Planned | Trước Sprint 3/4 |
| T07 | ISP1 TBD ↔ ISP2 TBD | Planned | Trước Sprint 3/4 |
| DMZ | FW DMZ TBD ↔ DMZ Web TBD, qua L2 nếu cần | Planned | Trước Sprint 5 |
| VLAN50 | Access TBD ↔ SRV-INFRA eth TBD | Planned; Access/cổng chưa chốt | Trước bài test Server VLAN ở Sprint 1 hoặc Sprint 7 nếu chưa dùng |
| Internet test | ISP/server và cổng TBD | Node/subnet chưa chốt | Trước NET-401 |

T01/T02 cần chốt trước Sprint 3. Khi chọn Ethernet switch node riêng cho transit, bổ sung số node/cổng vào inventory; chưa tính các node chưa quyết định là đã có.

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
