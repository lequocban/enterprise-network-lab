# Export HQ Sprint 00

**Bản dùng hiện tại:** [sprint-00-hq-2026-10-10.zip](sprint-00-hq-2026-10-10.zip), 952 bytes, chứa một `Enterprise-Network-HQ.unl` 4241 bytes.

**SHA256:** `CAD0EE62F6A1213B08147D5BB7471EBFB873D0B5F28FD0DB047276B6395F539F`.

[Kiểm tra cuối ngày 10/10/2026](../docs/test-results/sprint-00-export-final-inspection-2026-10-10.txt): CRC/XML PASS, đúng tám node/chín link, tên/ID không trùng, không có node PC-GUEST dư. Cả chín kết nối khớp physical topology. Đây là kiểm tra artifact tại local; agent chưa import hoặc chạy thiết bị.

## Nội dung và cấu hình đi kèm

- Bốn IOL switch và bốn VPCS; image IOL `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`.
- Thuộc tính RAM Core 512, Access 256; không có thuộc tính node vCPU.
- Các node `config="0"`, không có payload hoặc file cấu hình nhúng. ZIP chứa topology/metadata; **không phải backup đầy đủ startup-config hoặc VLAN database**.
- Dùng [12 snapshot config](../configs/README.md) và [evidence VLAN10](../docs/test-results/sprint-00-validation.md) đi kèm. CORE1/ACCESS2 khớp running/startup, CORE2/ACCESS1 chỉ running có VTY `login`; không coi hai loại bản là tương đương.
- Các biên bản export trước là lịch sử; hash cũ không xác định file hiện tại, các ZIP cũ/v2 không còn trong thư mục. Không tái tạo bản cũ từ file mới để làm bằng chứng.

## Tái dựng — chưa kiểm thử import/reload

1. Import ZIP vào lab riêng, giữ lab đang dùng; đối chiếu đúng tám node/chín link và image trước chạy.
2. Cung cấp image đúng tên ngoài Git; kiểm tra RAM/tài nguyên trên EVE đích.
3. Áp dụng config/VLAN theo artifact: hai Core có VLAN10 MGMT; bản running sau test Core đã đối chiếu với baseline. Snapshot không thay backup đầy đủ VLAN database. Chọn chính sách VTY rõ ràng trước lưu startup.
4. Kiểm tra SVI, trunk/native1/allowed10, root/port role VLAN10, HSRP peer/VIP và ping; ghi actual và khôi phục nếu khác. Native99/toàn bộ Campus thuộc Sprint 1.
5. Chỉ ghi restore PASS khi có test thật; hiện chưa có kết quả import/reload/restore hoặc xác nhận cấu hình tự phục hồi từ ZIP.

Chuyển role/khôi phục SVI trên lab gốc đã PASS, không đồng nghĩa import ZIP đã thành công. Việc thử restore được bàn giao tại [closeout](../docs/sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện).
