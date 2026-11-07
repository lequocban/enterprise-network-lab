# Restore seed Sprint 4 — 23/10/2026

- Lab bản sao đã import: `LAB-NETWORK/test_import/Enterprise-Network-Integrated.unl` (đường dẫn từ giao diện EVE trong phiên người dùng); lab HQ gốc và ZIP seed giữ nguyên.
- Ảnh người dùng: Startup-configs có đủ CORE1, CORE2, ACCESS1, ACCESS2 và bốn PC; cả tám công tắc ở trạng thái ON. Sau khi Start, người dùng báo cả tám node vẫn trắng cấu hình. Vì vậy **restore boot chưa PASS**; ON chỉ xác nhận chọn config set, không xác nhận đã nạp vào NVRAM.
- Gói seed đã kiểm trước đó: 8 node, 11 network, 8 payload Base64 giải mã được, ZIP CRC/XML đạt. Hiện không có bằng chứng payload mất khi import.
- Nguyên nhân phù hợp nhất: chưa Wipe sau lần đầu bật Startup-config. EVE-NG CE Cookbook 6.3, mục 8.1.3 và 10.3, yêu cầu Wipe sau khi đổi startup config và trong lần đầu bật tính năng này; Wipe xóa NVRAM/snapshot nhưng config set đã lưu vẫn còn.
- CORE1 **PASS bước nạp config** sau Stop → Wipe → Start trên lab bản sao: user cung cấp `show run | include hostname` → `hostname CORE1`; `show ip int br` có Vlan10 `10.10.10.2`, Vlan20 `10.10.20.2`, Vlan30 `.2`, Vlan40 `.2`, Vlan50 `.2`, Vlan60 `.2`, đều up/up; Po1 và Po11 up/up. Đó là bằng chứng cấu hình seed đã được EVE nạp vào CORE1; chưa chứng minh bảy node khác hay kết nối toàn Campus.
- Người dùng sau đó xác nhận **khôi phục hoàn toàn thành công** trên bản sao. Ghi nhận bảy node còn lại và bước kiểm tra Campus là **PASS theo xác nhận người thực hành**; chưa có raw output riêng của từng node, ping, HSRP hoặc EtherChannel trong phiên này để trích số liệu/role cụ thể. Không biến xác nhận này thành bằng chứng runtime chi tiết hoặc thành kết quả INT-04 WAN.
- Trạng thái: seed HQ đã khôi phục trên lab tích hợp theo xác nhận người dùng; CORE1 có output đối chiếu trực tiếp. Lab HQ gốc vẫn là nguồn giữ lại. INT-03 và Sprint 3 cần đối chiếu riêng các điều kiện thiết kế/DoD, không tự đóng chỉ vì restore đạt.

Nguồn: [EVE-NG CE Cookbook 6.3](https://www.eve-ng.net/wp-content/uploads/2025/04/EVE-CE-BOOK-6.3-2024.pdf), mục 8.1.3 và 10.3.
