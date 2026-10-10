# Sprint 00 — Biên bản đóng sprint

**Ngày chốt:** 10/10/2026, Asia/Saigon.
**Trạng thái:** **CLOSED WITH FOLLOW-UPS — nền tảng được nghiệm thu, các giới hạn có nơi tiếp nhận.**
**Yêu cầu chốt:** người thực hành xác nhận đã hoàn tất export và yêu cầu kiểm tra/đóng Sprint 0.
**Mốc Git:** commit local chứa biên bản này; chưa xác nhận push, PR hoặc merge trên GitHub.

## 1. Kết quả nghiệm thu

- **NET-001 — Môi trường:** có output tài nguyên/ảo hóa EVE, version/console bốn switch, ping Core hai chiều 5/5, SVI/trunk/STP/routes VLAN10 và test HSRP shutdown/no shutdown có log khôi phục. Router IOSv có kết quả lịch sử 4/5: chứng minh có kết nối, chưa nghiệm thu ổn định. Alpine/ASAv chạy thử theo xác nhận người thực hành, chưa có artifact console riêng. Chấp nhận mức bằng chứng này để đóng phần chuẩn bị; các phần thiếu được chuyển rõ ở mục 3, không nâng thành test PASS.
- **NET-002 — Thiết kế:** đã review logical/physical/architecture, đối chiếu HQ với ảnh và export hiện tại: bốn switch, bốn VPCS, chín link, không còn node PC-GUEST dư. WAN/DMZ/Branch/services vẫn planned; cổng TBD có mốc chốt.
- **NET-003 — IP Plan:** review tĩnh 24 subnet/loopback không phát hiện chồng lấn; VIP/.2/.3 và mask VLAN10 khớp evidence. IP v1.0 giữ nguyên. Lab router tạm 10.255.0.0/30 phải tách transit T01 10.255.0.0/29.
- **NET-004 — Repository:** repo đã khởi tạo, có merge PR #2 trong lịch sử local. Bộ docs/evidence closeout được review và lưu bằng commit local; tài liệu dùng chung có workflow và liên kết hợp lệ. `ai-agent/` bị ignore, không được stage. Push/PR/merge bộ closeout là bước xuất bản riêng, chưa ghi nhận đã thực hiện.

Việc đóng sprint không yêu cầu triển khai VLAN20–60, WAN hoặc VPN. Mục ping router lặp 10/10 và boot artifact bổ sung trong kế hoạch thu thập là phần tăng cường minh chứng; không sửa kết quả 4/5 hay xác nhận người dùng thành bằng chứng chạy mới. Đây là quyết định closeout có giới hạn, không phải mọi checklist kiểm thử đều PASS.

## 2. Bộ baseline dùng tiếp

- [Export HQ hiện tại](../../eve-exports/sprint-00-hq-2026-10-10.zip): 952 bytes; một `Enterprise-Network-HQ.unl` 4241 bytes; CRC/XML/cabling PASS.
- SHA256 ZIP: `CAD0EE62F6A1213B08147D5BB7471EBFB873D0B5F28FD0DB047276B6395F539F`. [Biên bản kiểm tra cuối](../test-results/sprint-00-export-final-inspection-2026-10-10.txt) là nguồn cho file hiện tại; các hash cũ thuộc lần kiểm tra trước.
- [12 snapshot cấu hình](../../configs/README.md): tám bản ban đầu, bốn bản hai Core sau test. Không ghi đè bản gốc; CORE1/ACCESS2 khớp lệnh running/startup, CORE2/ACCESS1 còn khác VTY `login`.
- [Validation và output](../test-results/sprint-00-validation.md), [ảnh HQ](../../screenshots/sprint-00/hq-topology-2026-10-10.png), [logical](../../diagrams/logical-topology.md), [physical](../../diagrams/physical-topology.md), [IP Plan](../ip-addressing-plan.md).
- IOL trong export: `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`; RAM node Core 512, Access 256 theo thuộc tính export. Không đánh đồng với memory runtime IOS; node vCPU không được ghi trong export.
- ZIP **chỉ chứa topology/metadata**, không có startup-config nhúng; import/reload/restore toàn lab chưa thử. Cần snapshot config và thông tin VLAN10 MGMT đi kèm; [hướng dẫn tái dựng](../../eve-exports/README.md).

## 3. Việc chuyển tiếp và điều kiện thực hiện

- **NET-101/102, Sprint 1:** tạo VLAN10–60/99, SVI quản trị Access .11/.12, trunk/native99. Review VTY CORE2/ACCESS1 và chọn baseline quản trị trước lưu startup; chưa nghiệm thu SSH/remote access. Review cổng test CORE1 e1/0 trước dùng lại.
- **NET-104, Sprint 1:** giữ uplink dự phòng shutdown khi dựng VLAN/trunk, chốt root primary/secondary, kiểm tra root/port role trước bật từng link và test hội tụ. Rapid-pvst hiện có không thay thế test Campus.
- **NET-103, Sprint 1:** kiểm chứng forwarding thật VLAN20↔30 và VLAN30↔50, kiểm tra routing/runtime; không suy ra khả năng Inter-VLAN từ `ip cef`, SVI hoặc HSRP.
- **NET-202, Sprint 2:** test mất Core/node/uplink và đo client packet loss/convergence. Test Sprint 0 chỉ shutdown SVI và chuyển role. Log riêng 08:09/08:35 không dùng làm failure test; người dùng xác nhận có thao tác trước 08:35, chi tiết chưa có.
- **NET-301/302, trước bài WAN Sprint 3:** lưu IOSv image/version, console và ping lặp hai chiều sau warm-up; giữ kết quả 4/5 lịch sử. Thu thập allocation/tải VM khi tăng số node; RAM available không chứng minh chạy toàn Enterprise.
- **WAN-01, trước Sprint 3–6:** chốt L2 VLAN90/91, cổng transit, default/return routes, ASA outside gateway/failover, đường OSPF, NAT/VPN endpoint và Internet Test Server; xem [quyết định kiến trúc](../network-architecture.md#6-các-quyết-định-thiết-kế-còn-mở).
- **NET-503, trước Sprint 5:** bổ sung ASAv version/boot artifact khi mở lại node thử; tích hợp firewall theo sprint, không thêm vào HQ chỉ để đóng Sprint 0.
- **NET-701, trước Sprint 7:** bổ sung Alpine version/boot artifact, chốt Access/cổng VLAN50 trước triển khai dịch vụ.
- **Trước khi dựa vào bản restore:** import ZIP vào lab riêng, dùng image/config phù hợp, tái tạo VLAN10 theo evidence và kiểm tra SVI/trunk/STP/HSRP/ping. Đây là kiểm thử chưa chạy, cần làm trước thay đổi lớn hoặc khi cần nghiệm thu khả năng khôi phục.

## 4. Điểm bắt đầu Sprint 1

Dùng baseline hiện có, kiểm tra trạng thái thực tế trước thay đổi. Trình tự: giữ uplink dự phòng shutdown → VLAN → trunk/native → STP/root/port role → bật từng uplink dự phòng → test cùng VLAN → routing/Inter-VLAN. Mỗi phần lưu config, expected/actual, khôi phục và evidence theo [workflow](../repository-workflow.md). Chưa bắt đầu cấu hình Sprint 1 trong lần closeout này.
