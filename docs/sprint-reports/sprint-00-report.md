# Báo cáo Sprint 0 — Chuẩn bị môi trường và thiết kế mạng

**Dự án:** Enterprise Network Design & Implementation (EVE-NG)
**Người thực hiện:** lequocban
**Ngày đóng:** 10/10/2026, Asia/Saigon
**Phạm vi:** NET-001–004
**Trạng thái:** **CLOSED WITH FOLLOW-UPS** — nền tảng được nghiệm thu, phần chưa kiểm chứng có story/mốc tiếp nhận.
**Nguồn quyết định:** [Biên bản đóng Sprint 00](sprint-00-closeout.md). Commit local chứa bộ hồ sơ này là mốc closeout; chưa ghi nhận push/PR/merge GitHub cho đợt bổ sung.

## 1. Kết quả

- EVE có 4 vCPU, RAM 7.7 GiB/available 5.7 GiB, disk available 98 GB, virtualization flags và thiết bị KVM. Không kết luận chịu tải toàn Enterprise từ một lần đo.
- Bốn switch có version/console thật. Image IOL đầy đủ trong export là `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`; RAM node Core 512/Access 256 theo metadata, khác memory IOS runtime.
- HQ có đúng bốn switch, bốn VPCS, chín link. ZIP hiện tại đã bỏ node PC-GUEST dư, CRC/XML/cabling PASS; [kiểm tra cuối](../test-results/sprint-00-export-final-inspection-2026-10-10.txt).
- VLAN10 MGMT: CORE1 .2/24, CORE2 .3/24, VIP .1. Ping hai Core mỗi chiều 5/5. Et0/0 trunk dot1q/native1, VLAN10 forwarding; CORE1 root RSTP, CORE2 root port Et0/0. Base priority cả hai 32768, chưa cấu hình root primary/secondary tường minh.
- HSRP shutdown SVI CORE1: CORE1 về Init, CORE2 nhận Active; no shutdown khôi phục CORE1 Active/CORE2 Standby và hai SVI up/up. [Log trước/trong/sau](../test-results/sprint-00-hsrp-switchover-2026-10-10.txt) chứng minh chuyển role/khôi phục, chưa đo convergence hoặc failover lưu lượng client.
- Có 12 snapshot config; bốn bản hai Core sau test khớp lệnh với bản trước tương ứng và không còn shutdown SVI. CORE1/ACCESS2 khớp running/startup, CORE2/ACCESS1 chỉ running có VTY `login`, giữ nguyên và bàn giao Sprint 1.
- Logical/physical/architecture và IP Plan v1.0 đã review. Review tĩnh 24 subnet/loopback không phát hiện chồng lấn; chưa triển khai phần WAN/DMZ/Branch.
- Repo, workflow công khai và cấu trúc artifact đã có; `ai-agent/` bị ignore và docs công khai không phụ thuộc vào thư mục này.

## 2. Nghiệm thu theo story

- **NET-001:** chấp nhận nền tảng HQ từ output môi trường, boot/console switch, ping và config/test thực tế. IOSv ping lịch sử 4/5 chứng minh có kết nối; Linux/ASAv boot theo xác nhận người thực hành, artifact chưa đủ. Các giới hạn này được giữ rõ và chuyển theo mục 3; không đánh dấu ping ổn định 10/10 hoặc boot artifact PASS khi chưa có.
- **NET-002:** logical/physical đã review; HQ đối chiếu ảnh/export hiện tại đúng tám node/chín link. Phần Enterprise dự kiến và cổng TBD có mốc chốt.
- **NET-003:** IP Plan Final v1.0 giữ nguyên, review tĩnh và SVI/VIP baseline nhất quán; routing/failover chuyển WAN-01.
- **NET-004:** repository đã khởi tạo, merge PR #2 có trong lịch sử local. Bộ hồ sơ closeout được review và lưu bằng commit local; xuất bản GitHub là bước riêng.

Đóng sprint theo phạm vi chuẩn bị môi trường/thiết kế, với các ngoại lệ minh chứng được nêu rõ. Điều này không thay đổi các acceptance chưa chạy thành PASS; xem [biên bản](sprint-00-closeout.md#1-kết-quả-nghiệm-thu).

## 3. Giới hạn và việc bàn giao

- **Sprint 1, NET-101–104:** VLAN/trunk/native99, SVI Access .11/.12, baseline VTY CORE2/ACCESS1, root priority và test STP trước mở đủ uplink; kiểm chứng Inter-VLAN thật.
- **Sprint 2, NET-201/202:** LACP cần thêm link cùng cặp endpoint; không gom hai Core độc lập vào một channel. Full Core/client failover, packet loss và convergence chưa thử.
- **Trước Sprint 3, NET-301/302:** IOSv version/console và ping lặp hai chiều sau warm-up; giữ kết quả 4/5 lịch sử. Bổ sung allocation/tải VM khi mở rộng số node.
- **WAN-01, trước Sprint 3–6:** L2 transit90/91, default/return routes, ASA outside gateway/failover, OSPF Branch, NAT/VPN endpoint; [quyết định kiến trúc](../network-architecture.md#6-các-quyết-định-thiết-kế-còn-mở).
- **NET-503 trước Sprint 5 / NET-701 trước Sprint 7:** bổ sung version/boot artifact ASAv/Alpine khi mở lại; không thêm node vào HQ chỉ để đóng Sprint 0.
- ZIP không có cấu hình nhúng; import/reload/restore toàn lab và backup VLAN database đầy đủ chưa thử. [Hướng dẫn tái dựng](../../eve-exports/README.md) phải đi cùng config/VLAN evidence.
- Log riêng 08:09/08:35 không thay cho failure test; người thực hành xác nhận có thao tác trước 08:35, chưa có chi tiết. Không kết luận lỗi tự phát hoặc ổn định liên tục từ snapshot.

## 4. Bắt đầu Sprint 1

Lưu baseline và kiểm tra trạng thái → giữ uplink dự phòng shutdown → VLAN → trunk/native → STP/root/port role → bật từng uplink và xác nhận → test cùng VLAN → routing/Inter-VLAN. Không triển khai Sprint 1 trong lần closeout này.

**Hồ sơ:** [Validation](../test-results/sprint-00-validation.md) · [Closeout](sprint-00-closeout.md) · [Architecture](../network-architecture.md) · [IP Plan](../ip-addressing-plan.md) · [Workflow](../repository-workflow.md).
