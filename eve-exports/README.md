# Export HQ và cấu hình đi kèm

## Baseline Sprint 01 dùng tiếp

[sprint-01-hq-2026-10-11.zip](sprint-01-hq-2026-10-11.zip), **947 bytes**, chứa một `Enterprise-Network-HQ.unl` **4241 bytes**.

**SHA256 ZIP:** `5B84F2E09DBEC6101E1D7AE925A42DAC6A77575C37131726C4F4715EB94F3D83`.

[Kiểm tra 11/10/2026](../docs/test-results/sprint-01-export-inspection-2026-10-11.txt): CRC/XML PASS, tám node/chín link, tên/ID duy nhất và toàn bộ cổng HQ khớp physical topology. Bốn IOL dùng `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`; Core RAM512, Access256 theo metadata.

Tất cả node `config="0"`; không có file/payload cấu hình nhúng. Export chỉ chứa topology/metadata, phải đi kèm [bốn running cuối Sprint 01 và lịch sử](../configs/README.md), [review cấu hình](../docs/test-results/sprint-01-config-review-2026-10-11.md) và [validation](../docs/test-results/sprint-01-validation.md). Chưa có startup riêng để đối chiếu; ZIP không chứng minh persistence/restore. [Báo cáo Sprint 01](../docs/sprint-reports/sprint-01-report.md) ghi follow-ups, không yêu cầu xuất lại chỉ vì ZIP không nhúng config.

### Tái dựng Sprint 01 — chưa kiểm thử

1. Import vào lab riêng, giữ lab hiện có; kiểm tra tám node/chín link và đúng image/tài nguyên trước chạy.
2. Dùng cấu hình Sprint 01 thật đi kèm để tái tạo bảy VLAN và interface; VTP transparent. Không dùng cấu hình VLAN10-only Sprint 00 để mô tả baseline hiện tại.
3. Giữ `no ip cef` hai Core theo ngoại lệ đã chọn; trunk native99/allowed bảy VLAN, STP root priorities và SVI/VIP theo snapshot. Cấu hình VPCS theo địa chỉ/gateway đích vì ZIP không nhúng startup VPCS.
4. Kiểm tra trạng thái L2/L3/HSRP và traffic, lưu config trên lab bản sao sau review. Chỉ đánh dấu restore PASS khi có kết quả thực tế; chưa có test này trong hồ sơ Sprint 01.

## Baseline lịch sử Sprint 00

**Bản lưu Sprint 00:** [sprint-00-hq-2026-10-10.zip](sprint-00-hq-2026-10-10.zip), 952 bytes, chứa một `Enterprise-Network-HQ.unl` 4241 bytes. Các thông tin dưới mô tả baseline lịch sử, không phải cấu hình Campus hiện tại.

**SHA256:** `CAD0EE62F6A1213B08147D5BB7471EBFB873D0B5F28FD0DB047276B6395F539F`.

[Kiểm tra cuối ngày 10/10/2026](../docs/test-results/sprint-00-export-final-inspection-2026-10-10.txt): CRC/XML PASS, đúng tám node/chín link, tên/ID không trùng, không có node PC-GUEST dư. Cả chín kết nối khớp physical topology. Đây là kiểm tra artifact tại local; agent chưa import hoặc chạy thiết bị.

### Nội dung và cấu hình Sprint 00

- Bốn IOL switch và bốn VPCS; image IOL `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`.
- Thuộc tính RAM Core 512, Access 256; không có thuộc tính node vCPU.
- Các node `config="0"`, không có payload hoặc file cấu hình nhúng. ZIP chứa topology/metadata; **không phải backup đầy đủ startup-config hoặc VLAN database**.
- Dùng [12 snapshot config](../configs/README.md) và [evidence VLAN10](../docs/test-results/sprint-00-validation.md) đi kèm. CORE1/ACCESS2 khớp running/startup, CORE2/ACCESS1 chỉ running có VTY `login`; không coi hai loại bản là tương đương.
- Các biên bản export trước là lịch sử; hash cũ không xác định file hiện tại, các ZIP cũ/v2 không còn trong thư mục. Không tái tạo bản cũ từ file mới để làm bằng chứng.

### Tái dựng Sprint 00 — chưa kiểm thử import/reload

1. Import ZIP vào lab riêng, giữ lab đang dùng; đối chiếu đúng tám node/chín link và image trước chạy.
2. Cung cấp image đúng tên ngoài Git; kiểm tra RAM/tài nguyên trên EVE đích.
3. Áp dụng config/VLAN theo artifact: hai Core có VLAN10 MGMT; bản running sau test Core đã đối chiếu với baseline. Snapshot không thay backup đầy đủ VLAN database. Chọn chính sách VTY rõ ràng trước lưu startup.
4. Kiểm tra SVI, trunk/native1/allowed10, root/port role VLAN10, HSRP peer/VIP và ping; ghi actual và khôi phục nếu khác. Native99/toàn bộ Campus thuộc Sprint 1.
5. Chỉ ghi restore PASS khi có test thật; hiện chưa có kết quả import/reload/restore hoặc xác nhận cấu hình tự phục hồi từ ZIP.

Chuyển role/khôi phục SVI trên lab gốc đã PASS, không đồng nghĩa import ZIP đã thành công. Việc thử restore được bàn giao tại [closeout](../docs/sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện).
