# Topology vật lý — bảng nối cổng và kế hoạch mở rộng

**Ngày cập nhật:** 10/10/2026

**Trạng thái:** cabling HQ đã đối chiếu với [ảnh người thực hành cung cấp ngày 10/10/2026](../screenshots/sprint-00/hq-topology-2026-10-10.png); export sạch tám node/chín link và config bốn switch đã đối chiếu; runtime VLAN10 hai Core có evidence, toàn Campus chưa nghiệm thu. `TBD` là chưa xác định, không phải cổng đã tạo.

## 1. HQ theo bảng cổng đã báo cáo

| Thiết bị A | Cổng A | Thiết bị B | Cổng B | Vai trò |
|---|---|---|---|---|
| CORE1 | e0/0 | CORE2 | e0/0 | Inter-core, VLAN10 test; mở rộng trunk ở Sprint 1 |
| CORE1 | e0/1 | ACCESS1 | e0/0 | Uplink chính dự kiến |
| CORE2 | e0/1 | ACCESS1 | e0/1 | Uplink dự phòng dự kiến |
| CORE1 | e0/2 | ACCESS2 | e0/0 | Uplink chính dự kiến |
| CORE2 | e0/2 | ACCESS2 | e0/1 | Uplink dự phòng dự kiến |
| ACCESS1 | e0/2 | PC-IT, VPCS tạm | eth0 | Test VLAN20 |
| ACCESS1 | e0/3 | PC-HR, VPCS tạm | eth0 | Test VLAN30 |
| ACCESS2 | e0/2 | PC-ACC, VPCS tạm | eth0 | Test VLAN40 |
| ACCESS2 | e0/3 | PC-GUEST, VPCS tạm | eth0 | Test VLAN60 |

Bảng mô tả cabling khớp ảnh HQ được cung cấp ngày 10/10/2026, chưa xác nhận tất cả interface đang up hoặc trunk đã cấu hình. Trước thao tác Sprint 1, đối chiếu trạng thái/config thực tế. Ảnh mới không có VPCS tạm nối CORE1 e1/0; bài test đó vẫn giữ trong lịch sử validation, không bổ sung thành node hạ tầng cố định.

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

- Sprint 1: giữ link dự phòng shutdown lúc chuẩn bị VLAN/trunk; kiểm tra STP trước bật đủ link, theo [S1-01](../docs/sprint-reports/sprint-00-closeout.md#4-điểm-bắt-đầu-sprint-1).
- Sprint 2: hiện mỗi cặp Core–Core/Core–Access chỉ có một link trong bảng. Bài LACP cần thêm ít nhất một link cùng cặp endpoint và cập nhật bảng cổng trước test mất member.
- Không gom uplink tới hai Core độc lập vào một Port-Channel duy nhất.
- Một ASAv không tạo firewall HA. Các failure test phải tách mất Core, uplink, Edge, ISP và FW.

## 4. Điều kiện review

- [x] Bảng HQ khớp 5 link hạ tầng và 4 link VPCS trong ảnh được cung cấp ngày 10/10/2026.
- [x] Export tám node/chín link, config bốn switch và runtime VLAN10 hai Core đã đối chiếu; không suy ra tất cả uplink Access đã trunk/forwarding.
- [ ] Cổng mở rộng được đổi từ TBD sang tên thật đúng thời điểm.
- [x] Logical/physical, IP Plan v1.0 và evidence HQ nhất quán theo closeout 10/10.
- [x] Đã lưu ảnh HQ và liên kết nguồn trong validation; ảnh chỉ xác nhận cabling hiển thị.
- [x] Đã lưu export HQ sạch; WAN vẫn planned. Cổng mở rộng TBD được chốt trước sprint liên quan, không chặn review thiết kế Sprint 0.
