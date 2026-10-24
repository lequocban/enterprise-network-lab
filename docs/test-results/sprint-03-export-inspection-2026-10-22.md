# Sprint 3 — Kiểm tra ZIP export

**Ngày kiểm tra:** 22/10/2026, Asia/Saigon.  
**Artifact:** [sprint-03-routing-2026-10-22.zip](../../eve-exports/sprint-03-routing-2026-10-22.zip), 4.957 byte.  
**SHA-256:** `5135EBA83D14C9A9A732D71B9F70B8640781102FBFC3E442FAE9187DA386490C`.

ZIP đọc được; CRC từng member và XML parse đạt. ZIP chỉ có `Enterprise-Network-S03-Routing.unl` (38.169 byte). XML có đúng 8 node: 5 `vios` image `vios-adventerprisek9-m-15.6.2T`, 3 `vpcs`; 7 bridge network, mỗi network đúng 2 interface; 35 text objects. Bảy cặp cổng: EDGE1 Gi0/0 ↔ EDGE2 Gi0/0; EDGE1 Gi0/1 ↔ BR1 Gi0/0; EDGE1 Gi0/2 ↔ BR2 Gi0/0; EDGE2 Gi0/2 ↔ PC-HQ eth0; BR1 Gi0/1 ↔ PC-BR1 eth0; BR2 Gi0/1 ↔ PC-BR2 eth0; EDGE2 Gi0/1 ↔ node ISP Gi0/0.

**Sai lệch cần sửa:** node ISP trong UNL tên `IPS1`, trong khi thiết kế/console dùng `ISP1`. Cần sửa tên node trên EVE rồi export một artifact v2, giữ ZIP đầu tiên làm bằng chứng lịch sử. Không đổi hostname IOS chỉ để khớp lỗi nhãn EVE.

**Payload:** tất cả 8 node có `config="0"`; ZIP không có file cấu hình router, VPCS hay image. Các text labels IP trên topology không xác nhận cấu hình đang chạy. `Export all CFGs` của EVE là thao tác config-set riêng, chưa có bằng chứng UI hoàn tất trong lượt này. ZIP hợp lệ chỉ chứng minh topology được đóng gói; không chứng minh import/restore hoặc persistence của cả lab.

**Trạng thái S03-TC-14:** CRC/XML/node/link/cổng đạt; tên node và payload bàn giao chưa đạt. Chờ ZIP v2 và cấu hình thật đi kèm trước kết luận cuối.

## EVE Export all CFGs — actual sau ZIP đầu tiên

Ảnh notification do người thực hành cung cấp: EDGE1, EDGE2, BR1-R1, BR2-R1 đều trả `Export not supported (19)` sau `Starting export`; PC-HQ báo `config exported`. Ảnh Startup-configs cho thấy năm IOSv đang OFF/xám; ba VPCS OFF/đỏ. `Export All: done` chỉ là hoàn thành tác vụ tổng, không phủ nhận lỗi của từng IOSv. Ảnh không hiển thị trọn thông báo ISP1/PC-BR1/PC-BR2; mô tả của người thực hành là toàn bộ router lỗi, cần phân biệt với từng thông báo nhìn thấy trong ảnh.

EVE Community Cookbook liệt kê Cisco vIOS L3 là loại có hỗ trợ export ở mức tính năng; lý do cụ thể lỗi code19 trong server/image/template hiện tại chưa được xác định. Không bật Startup-configs ON hoặc wipe node để chữa lỗi chưa chẩn đoán. Cấu hình router cần được lưu tách riêng bằng output `show running-config` và `show startup-config`; VPCS có thể export qua EVE theo ảnh, trái với giả định trước đây rằng VPCS không hỗ trợ. ZIP đầu vẫn chỉ chứng minh topology.

**Thông tin bổ sung của người thực hành:** toàn bộ node đã được Stop trước khi Export all CFGs; lệnh shell chỉ đọc tìm thấy `/opt/unetlab/config_scripts/config_vios.py` và `config_viosl2.py`. Script vIOS chính thức của EVE đọc cấu hình qua console Telnet `127.0.0.1:<port>`, vì vậy trạng thái router đã Stop là nguyên nhân phù hợp với lỗi; đây là suy luận từ script và trình tự thao tác, chưa có phép thử export lại khi router đang chạy. Bước kiểm chứng ít tác động: Start một IOSv, chờ boot vào EXEC rồi Export CFG riêng; nếu thành công mới Start các router còn lại và Export all CFGs. Không cập nhật script hoặc wipe để kiểm giả thuyết này. Cần đính chính chỉ dẫn trước đó đã xem code19 như vấn đề hỗ trợ bản cài khi chưa biết node đang Stop.

**Thử lại khi node chạy:** người thực hành xác nhận đã Start và Export all thành công. Ảnh Startup-configs sau thao tác có cả 5 router IOSv và 3 VPCS hiển thị đỏ/OFF thay cho 5 router xám/OFF ở ảnh trước; tên node ISP đã được sửa thành `ISP1`. Đây là bằng chứng UI cho config set khả dụng nhưng chưa được chọn làm nguồn boot sau wipe, không phải bằng chứng payload đã nằm trong ZIP hoặc restore. Chờ export ZIP v2 và kiểm nội dung file thật.

## ZIP v2 — kết quả kiểm tra cuối artifact

**Artifact:** [sprint-03-routing-2026-10-22-v2.zip](../../eve-exports/sprint-03-routing-2026-10-22-v2.zip), 8.454 byte. SHA-256: `62D2E10F1F65DA9D52B2560BD2BCC5FF537AFDDB3FD4B523B9613D3EC1A9669E`.

CRC ZIP đạt, XML parse đạt. ZIP có một `Enterprise-Network-S03-Routing.unl` (62.007 byte), tám node gồm 5 IOSv và 3 VPCS, bảy network hai đầu đúng cặp cổng thiết kế, 35 text objects. Node ISP đã là `ISP1`. Khác v1, UNL có `<configs>` với tám `<config>` Base64, ID khớp từng node. Đã giải mã và lưu bản đọc được tại [configs/routing-sandbox/sprint-03/final](../../configs/routing-sandbox/sprint-03/final/README.md).

Kiểm nội dung cả tám payload: EDGE1 có hostname và hai area range; EDGE2 có static default tới `10.255.0.50` và `default-information originate`; BR1/BR2 có IP LAN và Loopback110 theo thiết kế; ba VPCS có tên/IP; ISP1 có hostname, Loopback100 `192.0.2.10` và đường về qua `10.255.0.49`. Đây là kiểm tra cấu trúc và các mốc cấu hình chính, không phải diff toàn bộ cấu hình với startup độc lập.

Các node vẫn có thuộc tính XML `config="0"` dù tám payload tồn tại. Thuộc tính này tương ứng Startup-configs OFF: cấu hình export đã nằm trong ZIP nhưng chưa được chọn để nạp khi wipe/boot. Không đổi ON hoặc wipe trên lab hiện tại chỉ để kiểm ZIP. Import/restore trên bản sao chưa thử; không coi kiểm CRC/XML hay giải mã là bằng chứng phục hồi hoạt động.

**S03-TC-14:** đạt kiểm tra integrity, topology, tên node và payload cấu hình của artifact v2. Kiểm thử import/restore tách riêng, trạng thái chưa chạy. ZIP v1 giữ lại làm bằng chứng sự cố ban đầu.
