# Sprint 04 — Campus baseline trước khi thêm WAN

**Ngày nhận output:** 23/10/2026, Asia/Saigon; output không có timestamp riêng.  
**Nguồn:** console CORE1, CORE2, ACCESS1, hai VPCS và EVE host do người thực hành gửi trong phiên buổi 1 Sprint 4. Các dòng dưới là kết quả thực tế từ tin nhắn, không phải agent truy cập EVE trực tiếp.  
**Trạng thái checkpoint A, bổ sung 24/10/2026:** **ĐẠT theo phạm vi baseline Campus đại diện**; Po1/Po11/HSRP và ping 10 gói hai chiều có output thực. Cho phép bắt đầu dựng WAN trong lab tích hợp. Đây chưa phải INT-04 hay hồi quy sau khi thêm WAN. Người thực hành xác nhận path lab trên giao diện EVE: `/root/LAB-NETWORK/test_import/Enterprise-Network-Integrated.unl`; không diễn giải `/root` này thành đường dẫn file Linux của EVE.

## Kết quả đã có

- `CORE1#show etherchannel sum`: `Po1(SU)` LACP với `Et0/0(P) Et0/3(P)`; `Po11(SU)` LACP với `Et0/1(P) Et1/1(P)`.
- `CORE2#show etherchannel sum`: `Po1(SU)` LACP với `Et0/0(P) Et0/3(P)`.
- `ACCESS1#show etherchannel sum`: `Po11(SU)` LACP với `Et0/0(P) Et1/0(P)`; hai đầu CORE1↔ACCESS1 đều có hai member bundled ở thời điểm đọc.
- `CORE1#show standby br`: VLAN10/20/30/40/50/60 đều **Active local**, priority 110, preempt `P`; Standby lần lượt là CORE2 `10.10.10.3` đến `10.10.60.3`; VIP mỗi VLAN là `10.10.X.1`.
- `CORE2#show standby br`: cùng sáu VLAN đều **Standby local**, priority 100, preempt `P`; Active lần lượt là CORE1 `10.10.10.2` đến `10.10.60.2`; VIP giống CORE1.
- `PC-IT-01> ping 10.10.40.20 -c 10`: có reply `icmp_seq=1` đến `10`, tức **10/10 reply** đến PC-ACC; TTL 63; RTT trong mẫu 1.067–1.587 ms. Tin nhắn không kèm `show ip` của PC-IT-01, nên IP nguồn/gateway thực tế chưa được đối chiếu trực tiếp trong checkpoint này.
- `PC-ACC-01> show ip`: `10.10.40.20/24`, gateway `10.10.40.1`, MTU 1500. Prompt thực tế `PC-ACC-01` khác nhãn `PC-ACC` trong seed; IP/gateway khớp thiết kế.
- `PC-ACC-01> ping 10.10.20.20` **không có `-c 10`**: reply `icmp_seq=1` đến `5`, tức **5/5 reply** trong mẫu mặc định này; TTL 63. Đây là bằng chứng chiều ngược hoạt động, không đổi thành 10/10.
- **Bổ sung nhận ngày 24/10/2026:** `PC-ACC-01> ping 10.10.20.20 -c 10` có reply `icmp_seq=1` đến `10`, tức **10/10 reply**; TTL 63, RTT 1.188–1.770 ms. Giữ mẫu 5/5 phía trên làm lịch sử, không viết lại thành mẫu 10 gói.
- EVE host `free -h`: total `7.7Gi`, used `1.7Gi`, available `5.7Gi`, swap total `4.0Gi`, swap used `0B`. `uptime`: `09:22:31 up 1:38`, load average `2.63, 2.66, 2.43`. Chưa có danh sách node đang bật tại đúng mốc đo, nên không gán mẫu này cho một nhóm node cụ thể.

Po1/Po11 và role HSRP phù hợp baseline Sprint 2 ở các đầu đã quan sát. Ping chứng minh traffic IT→ACC **10/10** và ACC→IT **10/10** trong hai mẫu 10 gói riêng; trước đó còn có mẫu chiều ngược 5/5. Kết quả `SU/P` không tự đóng lỗi mất traffic khi tắt một đầu member Po1 được bàn giao từ Sprint 2.

## Giới hạn và thông tin cần lấy ở bước tiếp

- `PC-IT-01 show ip` khi cần xác nhận IP/gateway nguồn. Nhãn VPCS `PC-IT-01` khác tên `PC-IT` trong seed, chưa có đối chiếu việc đổi nhãn; không tự xem đây là thay đổi địa chỉ hoặc thiết kế.
- Ghi danh sách node đang bật khi dùng số liệu tài nguyên để quyết định mở nhóm IOSv; kết quả host đã có không xác nhận nhóm node tại thời điểm đo.
- `show ip interface brief`/trunk nếu phát hiện sai lệch hoặc cần đối chiếu phạm vi cấu hình; chưa có output mới cho hai lệnh này.

Chưa có bằng chứng node/link WAN đã được tạo hoặc BGP chạy. Sau khi Campus baseline đủ bằng chứng, dựng WAN theo [mapping cổng đã chốt](../../diagrams/physical-topology.md#2-kết-nối-mở-rộng-dự-kiến) và [IP Plan](../ip-addressing-plan.md#9-phụ-lục-int-03-cho-topology-tích-hợp--thiết-kế-23102026), rồi lưu inventory/cabling actual riêng.
