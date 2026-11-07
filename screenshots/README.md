# Ảnh minh chứng

Sprint04 cuối 07/11/2026: [17 node Startup-config Exported](sprint-04/final-configured-nodes-2026-11-07.png), [topology WAN có nhãn ASN/subnet](sprint-04/final-wan-annotated-2026-11-07.png) do người thực hành cung cấp; ảnh WAN là phần chụp màn hình, không hiển thị toàn Campus. [Restore](../docs/test-results/sprint-04-final-restore-2026-11-07.md) có evidence API/telnet trực tiếp, không có ảnh chụp màn hình restore tự động.

Tạo thư mục `sprint-<NN>/` khi có ảnh thật. Đặt tên theo node/nội dung/ngày thu thập, ví dụ `sprint-00/core1-hsrp-2026-10-09.png`; đây chỉ là tên minh họa.

Ảnh HQ ngày 10/10 đã đối chiếu với export cuối; artifact boot ASAv/Alpine tiếp nhận NET-503/701 khi mở lại node thử, không ghi đã có ảnh khi chưa có. Ảnh HSRP chỉ chứng minh trạng thái hiển thị tại thời điểm chụp; failover cần log/thao tác trước, trong và sau sự kiện.

Liên kết ảnh từ validation, ghi ngày và nội dung kiểm chứng. Không dùng ảnh mô phỏng hoặc ảnh của topology dự kiến để thay bằng chứng lab thực tế.

Sprint 04, 24/10/2026: [metadata hai Edge](sprint-04/t02-edge-node-settings-2026-10-24.png), [dây Gi0/0 tới T02](sprint-04/t02-edge-bridge-topology-2026-10-24.png) và [ba node ISP/INET mới](sprint-04/t02-isp-inet-nodes-2026-10-24.png) từ giao diện EVE; [biên bản đối chiếu](../docs/test-results/sprint-04-t02-cabling-2026-10-24.md) tách phần thấy trong ảnh với Type bridge/cấu hình node được người thực hành xác nhận.

Sprint 04, 24/10/2026: [metadata đủ 13 node](sprint-04/t03-t04-node-settings-2026-10-24.png) và [dây T03/T04](sprint-04/t03-t04-edge-isp-cabling-2026-10-24.png); cùng biên bản đối chiếu ở trên, chưa chứng minh IP/BGP runtime.

Sprint 04, 24/10/2026: [toàn bộ WAN và Branch sau nối dây](sprint-04/wan-cabling-complete-2026-10-24.png); cặp cổng theo ảnh được ghi trong [biên bản](../docs/test-results/sprint-04-t02-cabling-2026-10-24.md), còn chờ export ZIP để kiểm UNL và payload.

Đầu việc: [S0-03](../docs/sprint-reports/sprint-00-closeout.md#2-bộ-baseline-dùng-tiếp).
