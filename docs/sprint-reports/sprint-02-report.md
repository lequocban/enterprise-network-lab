# Sprint 2 — Bàn giao LACP và High Availability

**Ngày cập nhật:** 18/10/2026, Asia/Saigon.  
**Phạm vi:** NET-201 (LACP), NET-202 (HSRP/HA).  
**Trạng thái:** đã thu cấu hình cuối và kiểm tra export; bàn giao kết quả thực hành với lỗi Po1 còn mở. Chưa nghiệm thu DONE đầy đủ, chưa coi lỗi dự phòng một member là đã sửa.

Nguồn là cấu hình, console output, capture và xác nhận do người thực hành cung cấp. Agent kiểm tra file tại local, không điều khiển console EVE, không thực hiện import/restore. Theo yêu cầu người thực hành, kiểm thử tập trung ít nhất một VLAN đại diện cho mỗi đường; dừng lặp từng VLAN để tiết kiệm thời gian. Những phần chưa chạy/đo vẫn được ghi rõ.

## Kết quả thực hành

- Po1 CORE1–CORE2: e0/0 và e0/3 hai đầu, LACP active. Po11 CORE1–ACCESS1: CORE1 e0/1/e1/1, ACCESS1 e0/0/e1/0, LACP active. Runtime cuối được cung cấp có đủ hai member P của mỗi channel.
- Trunk native99, allowed10/20/30/40/50/60/99; rapid-PVST, CORE1 ưu tiên root, CORE2 dự phòng. Giữ workaround `no ip cef` và ngoại lệ hiển thị Auto-duplex/Shr của image; không suy image lỗi chỉ từ các nhãn này.
- Mất toàn Po11: VLAN20 tự chuyển qua ACCESS1→CORE2→Po1→CORE1; người thực hành xác nhận ping ổn định trước khi mở lại Po11. Xem [bằng chứng](../test-results/sprint-02-po11-whole-channel-2026-10-18.md).
- Mất toàn Po1: CORE2 VLAN20 chuyển Root Port sang e0/1 qua ACCESS1, cost156; ping tự phục hồi khi Po1 còn tắt. Sau mở lại, Po1 Root FWD/cost56 và traffic phục hồi.
- Mất CORE1: CORE2 Active cả sáu nhóm HSRP; các lượt IT20→ACC40, HR30↔client VLAN50 tạm, IT20↔client VLAN10 tạm có bằng chứng chuyển tiếp. Số timeout lần lượt18/18/19; không quy đổi thành thời gian hội tụ. Sau Start CORE1, sáu nhóm trở về CORE1 Active/CORE2 Standby và các luồng kiểm tra lại thành công.
- Mất CORE2: CORE1 vẫn Active sáu nhóm, Po11 hai P; mẫu IT→ACC có126/126 reply, GUEST→IT5/5. Sau Start CORE2, trở lại Standby và Po1 đủ hai P.
- Endpoint đã hoàn nguyên. Cấu hình ACCESS2 cuối có e0/2 VLAN40/e0/3 VLAN60; `show ip` xác nhận PC-ACC10.10.40.20/24 GW40.1 và PC-GUEST10.10.60.20/24 GW60.1. Người thực hành xác nhận hoàn thành bước lưu switch/VPCS; riêng xác nhận sau cùng nêu rõ đã `wr` tất cả switch.

Chi tiết kết quả/capture và các lượt khôi phục ở [checkpoint Po1/HA](../test-results/sprint-02-po1-member-check-2026-10-18.md). Lịch sử Po11 và lỗi/retest trước đó được giữ trong [hồ sơ Po11](../test-results/sprint-02-po11-member-a-issue-2026-10-16.md), không thay đổi kết quả lịch sử thành toàn bộ PASS.

## Lỗi Po1 chưa giải quyết

Khi chỉ shutdown CORE1 e0/3, có lượt Po1 SU/e0/0 P nhưng PC không tới được CORE2 cho tới khi mở lại e0/3. Có lượt phục hồi sau clear MAC VLAN20 và MAC C2 chuyển từ Po11 về Po1 trên C1; ở lượt capture khác MAC C2 trên C1 đã đúng Po1 mà traffic vẫn lỗi.

Capture phía CORE2 ghi năm request do C2 phát trên e0/3; người thực hành xác nhận đầu CORE1 đang shutdown, trong khi summary C2 vẫn hai P. Capture e0/0 còn có cùng một reply lặp11730 lần. Giữ giới hạn đồng bộ thời gian/capture không có direction flags; chưa định danh bug hoặc kết luận chắc chắn loop/image lỗi.

Khi shutdown e0/3 cả hai đầu: Po1 một member e0/0 chuyển tiếp hai chiều5/5, không cần clear MAC/ARP trong phép kiểm tra đó. Mở riêng e0/3 CORE1 rồi mở cả hai đầu vẫn ping được theo xác nhận. Điều này chứng minh đường còn lại dùng được, nhưng chưa chứng minh loại member và chuyển tiếp tự động luôn đúng khi mất một đầu.

Hai giả thuyết đang mở theo yêu cầu người thực hành: môi trường EVE-NG không ổn định; image IOL không ổn định/có lỗi. Chưa có đối chứng để xác định lớp gây lỗi. Không dùng shutdown hai đầu hoặc clear MAC thủ công làm tiêu chí PASS cho dự phòng tự động. Xem [hồ sơ điều tra](../test-results/sprint-02-po1-member-check-2026-10-18.md).

## Bộ bàn giao cuối

- [CORE1 running](../../configs/core/sprint-02/CORE1.final.running.cfg):142 dòng lệnh, khớp checkpoint Po1+Po11.
- [CORE2 running](../../configs/core/sprint-02/CORE2.final.running.cfg):116 dòng, khớp checkpoint Po1.
- [ACCESS1 running](../../configs/access/sprint-02/ACCESS1.final.running.cfg):87 dòng, khớp checkpoint Po11.
- [ACCESS2 running](../../configs/access/sprint-02/ACCESS2.final.running.cfg):72 dòng, khớp baseline Sprint01 sau hoàn nguyên.
- [Export Sprint02](../../eve-exports/sprint-02-hq-2026-10-18.zip):1007 bytes; một UNL4762 bytes; CRC/XML hợp lệ, tám node/mười một kết nối khớp toàn bộ cổng. [Biên bản kiểm tra](../test-results/sprint-02-export-inspection-2026-10-18.txt) có SHA256 và danh sách endpoint.
- [Capture điều tra](../../captures/sprint-2/): giữ các file do người thực hành cung cấp, không thay thế bằng dữ liệu mô phỏng.

Export chỉ có topology/metadata; tất cả node `config=0`, không có payload cấu hình hoặc VPCS startup. Cần đi kèm bốn cấu hình ngoài ZIP và IP endpoint. Không tạo `.final.startup.cfg` bằng cách chép running: đã có xác nhận wr, nhưng chưa có dump startup cuối để đối chiếu độc lập. Các bản running/startup lịch sử được giữ nguyên; [index cấu hình](../../configs/README.md) ghi nguồn và cách chuẩn hóa.

## Giới hạn và việc chuyển tiếp

- NET-201 chưa đạt toàn bộ acceptance mất từng member vì lỗi Po1 còn mở; cấu hình channel và các đường dự phòng khác đã có bằng chứng chức năng.
- NET-202 có failover/failback chức năng trên các luồng đã nêu, chưa có phép đo timestamp đồng bộ để nghiệm thu thời gian hội tụ. Chưa thu đầy đủ mọi chiều traffic theo kế hoạch gốc; VLAN60 chỉ có baseline hai chiều và lượt mất CORE2, chưa kiểm chứng khi CORE2 Active. Không yêu cầu lặp test VLAN60 theo quyết định thu hẹp phạm vi của người thực hành.
- Import/restore bộ cuối chưa thử. Bước tái dựng Sprint01 trước LACP đã bỏ qua theo yêu cầu, không đổi thành PASS. File ZIP hợp lệ không chứng minh có thể khôi phục toàn bộ trạng thái chạy.
- Chưa có export cấu hình startup/VPCS nhúng hoặc ảnh cuối mới; không tự tạo bằng chứng còn thiếu. IP endpoint dùng lại theo IP Plan: IT20.20, HR30.20, ACC40.20, GUEST60.20, đều /24 và gateway cùng subnet .1.
- Trước khi dựa vào Po1 cho yêu cầu HA nghiêm ngặt, tiếp tục xử lý lỗi với phép đối chứng EVE/image có kiểm soát và retest member. Chưa áp dụng thay image/timer/hash hoặc rebuild channel trong bộ final này.
- Bộ hồ sơ Sprint02 được lưu bằng commit local sau commit baseline Sprint01 trên nhánh `lab/sprint-02-lacp-hsrp`; chưa push/PR/merge. Bàn giao và commit không tự đóng story hoặc đánh dấu toàn bộ Sprint2 DONE.
