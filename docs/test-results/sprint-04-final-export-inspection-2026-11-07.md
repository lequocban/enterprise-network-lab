# Sprint 04 — Kiểm ZIP cuối, 07/11/2026

**Ngày kiểm:** 07/11/2026, Asia/Saigon. **Phương pháp:** đọc ZIP/XML/Base64 và so sánh baseline trong workspace; agent không truy cập EVE.  
**Trạng thái:** **PASS kiểm tĩnh artifact; restore boot/traffic còn chờ**.

**Bổ sung sau checkpoint kiểm tĩnh:** [restore thật ngày 07/11](sprint-04-final-restore-2026-11-07.md) đã PASS theo runbook lưu startup/restart, với raw console. Phần “còn chờ” trong biên bản này mô tả thời điểm trước lượt restore.

[ZIP cuối](../../eve-exports/sprint-04-enterprise-final-2026-11-07.zip) **14.082 byte**, SHA-256 `245C9B3C6ED7343C946BBB531DFB93151226C404F8ECFB9EA66496F1AA813D50`. File trình duyệt lưu dưới tên tạm `Unconfirmed 618334.crdownload` đã có ZIP hoàn chỉnh: CRC đạt, một entry `Enterprise-Network-Integrated.unl`, XML đọc được. Agent kiểm digest và sao chép nguyên byte sang tên cuối do người thực hành chỉ định; giữ file nguồn và mọi ZIP cũ.

## Inventory/cabling/payload

- 17 node với ID/tên duy nhất, 21 network, mỗi network có đúng hai endpoint. Mapping toàn bộ 21 dây giống [baseline 26/10](sprint-04-underlay-export-2026-10-26.md).
- 17/17 payload Base64 giải mã được, ID payload khớp đủ node; tất cả node metadata `config="1"` (Exported). Ảnh Configured Nodes do người thực hành gửi cũng hiển thị Exported cho 17 node.
- Image, template, RAM, CPU, số Ethernet và NVRAM trong metadata giống baseline. T02 bridge/các endpoint giữ nguyên.
- Trích nguyên byte 17 cấu hình vào [configs/sprint-04-final](../../configs/sprint-04-final/README.md); [manifest](../../configs/sprint-04-final/manifest.json) có dung lượng/hash từng payload và danh sách endpoint.

## So sánh baseline

Sáu VPCS giống nguyên byte baseline, khớp IP/gateway runtime. Bốn IOL chỉ đổi comment `Last configuration change`. ISP1/ISP2/INET-TEST/BR1/BR2 chỉ bỏ comment thời điểm và dấu phân cách. EDGE1/EDGE2 thêm `exec-timeout 45 0` và `logging synchronous` ở console, đồng thời bỏ comment thời điểm; không có thay đổi khác khi so toàn bộ các dòng cấu hình. Vì vậy policy scoped LP, filter, interface, route và startup cổng thử lỗi tương ứng baseline đã phục hồi; không thấy cấu hình shutdown thử nghiệm mới trong artifact.

[Persistence IOSv](sprint-04-iosv-persistence-2026-11-07.md) và [hồi quy Campus/VPCS](sprint-04-campus-vpcs-regression-2026-11-07.md) có output runtime trên lab gốc. Các bằng chứng đó và kiểm ZIP tĩnh chưa chứng minh bản import mới boot/traffic thành công. Restore phải chạy trên thư mục bản sao riêng, kiểm ba nhóm BGP, Campus, Branch và sáu VPCS từ payload; INT-04 chưa chốt.
