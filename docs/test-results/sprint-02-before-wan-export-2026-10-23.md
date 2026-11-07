# Kiểm tra backup HQ trước mở rộng WAN — 23/10/2026

## Kết quả

Đã kiểm tra `eve-exports/sprint-02-hq-before-wan-2026-10-22.zip` ở mức cấu trúc và payload:

- ZIP: 3.736 byte; SHA-256 `AAEE4445EEF1FC1B68F777FA1B2986E192B2457F662DB03CFC6B2850773C5F81`.
- CRC và XML hợp lệ; chứa `Enterprise-Network-HQ.unl` (20.704 byte).
- Có 8 node và 8 payload cấu hình Base64 giải mã được, ID khớp với CORE1, CORE2, ACCESS1, ACCESS2, PC-IT, PC-HR, PC-ACC, PC-GUEST.
- Bốn cấu hình VPCS chứa đúng IP/gateway của VLAN IT, HR, Accounting và Guest. Cấu hình switch có hostname khớp tên node.
- Các node đều có thuộc tính `config="0"`: Startup-configs đang OFF. Đây không phải dấu hiệu thiếu payload trong ZIP.

## Phạm vi xác nhận

Backup có topology và payload cấu hình; không chứa image IOS/IOL. Chưa import hoặc khởi động bản sao, vì vậy khả năng restore chưa được xác nhận. Không cần Export All CFGs lại chỉ để bổ sung payload.

Nếu kiểm tra restore, hãy import ZIP thành một lab thử nghiệm riêng, giữ nguyên lab `Enterprise-Network-HQ`. Chỉ thao tác bật Startup-configs, wipe hoặc sửa cấu hình trên bản sao dùng để thử.
