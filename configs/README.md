# Cấu hình thiết bị

Lưu cấu hình **lấy từ thiết bị đã chạy lab**, không tạo config giả từ bảng IP để làm bằng chứng.

## Tổ chức

- `core/sprint-<NN>/`: CORE1, CORE2.
- `access/sprint-<NN>/`: ACCESS1, ACCESS2.
- `edge/`, `branch/`, `firewall/`: bổ sung thư mục sprint khi triển khai nhóm tương ứng.
- Mỗi file dùng tên node và `.cfg`; ghi nguồn running/startup trong báo cáo kiểm thử.

README này giữ thư mục `configs/` khi clone. Thư mục con rỗng không được Git lưu; tạo lại khi có config thật.

## Baseline cần bổ sung

- [x] CORE1 Sprint 00: [running-config](core/sprint-00/CORE1.cfg) và [startup-config](core/sprint-00/CORE1.startup.cfg), người thực hành cung cấp ngày 10/10/2026; lệnh cấu hình khớp nhau, metadata khác. Chưa kiểm thử khôi phục.
- [x] CORE2 Sprint 00: [running-config](core/sprint-00/CORE2.cfg) và [startup-config](core/sprint-00/CORE2.startup.cfg), người thực hành cung cấp ngày 10/10/2026. Đã lưu và đối chiếu; running có `login` dưới VTY 0–4, startup không có. Chưa đồng bộ hoặc kiểm thử khôi phục.
- [x] ACCESS1 Sprint 00: [running-config](access/sprint-00/ACCESS1.cfg) và [startup-config](access/sprint-00/ACCESS1.startup.cfg); running có VTY `login`, startup không có, các lệnh còn lại khớp.
- [x] ACCESS2 Sprint 00: [running-config](access/sprint-00/ACCESS2.cfg) và [startup-config](access/sprint-00/ACCESS2.startup.cfg); lệnh cấu hình và thời gian last change khớp.
- [x] Đã lưu running/startup bốn switch HQ dùng trong Sprint 0; chưa kiểm thử khôi phục hoặc backup đầy đủ VLAN database.
- [x] Image/version bốn switch và các cặp startup/running được đối chiếu trong validation; CORE2 và ACCESS1 chưa đồng bộ hoàn toàn.
- [x] Đã kiểm tra tám snapshot hiện có: không thấy mật khẩu, key hoặc PSK; cần review lại khi bổ sung cấu hình quản trị.
- [x] Có liên kết từ [Validation Sprint 0](../docs/test-results/sprint-00-validation.md) tới config thật.
- [x] Hai Core có thêm bốn snapshot sau test: CORE1 [running](core/sprint-00/CORE1.post-switchover.cfg)/[startup](core/sprint-00/CORE1.post-switchover.startup.cfg), CORE2 [running](core/sprint-00/CORE2.post-switchover.cfg)/[startup](core/sprint-00/CORE2.post-switchover.startup.cfg). Từng bản khớp lệnh với bản trước test tương ứng; CORE2 vẫn khác VTY giữa running/startup. Tổng hiện có 12 snapshot, đã giữ nguyên tám bản đầu.

Nguồn và kết quả đối chiếu tại [Validation Sprint 0](../docs/test-results/sprint-00-validation.md). Export HQ sạch đã kiểm tra, đi kèm 12 snapshot. Sprint 0 đóng với follow-up: VTY CORE2/ACCESS1 tiếp nhận Sprint 1; khôi phục SVI vận hành đã kiểm tra, import/reload/restore toàn lab vẫn chưa thử, theo [closeout](../docs/sprint-reports/sprint-00-closeout.md).
