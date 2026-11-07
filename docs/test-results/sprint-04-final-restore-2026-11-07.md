# Sprint 04 — Restore ZIP cuối trên bản sao

**Ngày:** 07/11/2026, Asia/Saigon. **Nguồn:** agent đăng nhập API EVE và telnet console theo ủy quyền người thực hành. Capture có UTC; bộ cuối BGP/Branch khoảng 15:20–15:21 UTC, VPCS 15:22 UTC, tức 22:20–22:22 Asia/Saigon.  
**Trạng thái:** **PASS restore và persistence chức năng, với bước lưu startup sau import được ghi rõ**.

## Định danh và cách thực hiện

- Nguồn: [ZIP cuối](../../eve-exports/sprint-04-enterprise-final-2026-11-07.zip), 14.082 byte, SHA-256 `245C9B3C6ED7343C946BBB531DFB93151226C404F8ECFB9EA66496F1AA813D50`.
- Lab gốc `/LAB-NETWORK/test_import/Enterprise-Network-Integrated.unl`, ID `f5f1a877-a874-48f4-9d08-efeff0f0f4eb`.
- Tạo thư mục trống và import ZIP vào `/LAB-NETWORK/SPRINT04-RESTORE-2026-11-07/Enterprise-Network-Integrated.unl`, ID `78ce863a-812a-4b65-b242-12bc93da62fa`. Không Wipe/xóa/ghi đè lab gốc. API nguồn và bản restore có 17/17 payload giống nguyên byte ZIP cuối sau kiểm; [audit](sprint-04-restore-2026-11-07/final-audit.json) lưu hashes và định danh.
- EVE Community API version `6.2.0-4`. Ban đầu không có process node chạy; sau Start có 4 IOL, 7 QEMU, 6 VPCS. Mốc cuối API mem33%, swap0%, disk13%, CPU17%; đây là snapshot, chưa phải load test. Với các node ID trùng giữa hai lab, API node-status hiển thị cùng trạng thái cổng console ở cả hai đường dẫn; không diễn giải thành 34 node đang chạy. Toàn bộ lệnh Start/Stop/console trong phiên này nhắm đường dẫn restore; không suy trạng thái riêng lab gốc chỉ từ API node-status.

## Kết quả boot, route và traffic

17 node boot từ payload Exported. Bốn IOL giữ Po1/Po11 hai member P; HSRP sau ổn định có CORE1 Active/CORE2 Standby đủ sáu VLAN. `no ip cef` trên hai Core vẫn là workaround kế thừa. Trunk/forwarding phù hợp baseline; chưa chạy lại bài một đầu member Po1.

Sáu VPCS thực thi startup file, IP/gateway đúng: IT `10.10.20.20/.1`, HR `10.10.30.20/.1`, ACC `10.10.40.20/.1`, Guest `10.10.60.20/.1`, BR1 `10.20.10.20/.1`, BR2 `10.30.10.20/.1`, đều /24. Sáu mẫu cuối sau warm-up có đủ reply seq1–10: IT→ACC, ACC→IT, HR→VIP30, Guest→VIP60, PC-BR1→gateway, PC-BR2→gateway. Không cần dựa vào xác nhận save/Stop/Start PC trên lab gốc để chấm boot VPCS từ artifact này.

Hai ISP nhận default/target từ INET; INET nhận 7 prefix từ mỗi ISP và có return route tới cả hai Lo100. EDGE1 best default/target qua ISP1 `.18`, LP200, RIB distance20; EDGE2 best iBGP `.10` LP200, RIB distance200, backup ISP2 `.22` LP100. Sau restart có lưu startup, hai ping Lo100→INET đạt 10/10 (EDGE1 RTT2/3/4 ms, EDGE2 2/2/3 ms).

BR1/BR2 giữ WAN/LAN/Lo0 và static default `.26/.30`. BR1 source `.25`→EDGE1 `.17` 10/10; EDGE1 source `.17`→BR1 `.25` 10/10. BR2 source `.29`→EDGE2 `.21` 10/10; EDGE2 source `.21`→BR2 `.29` 10/10; RIB EDGE2 tới T06 trực tiếp ISP2 `.22`, phù hợp scoped LP.

Advertised-routes sau hội tụ: mỗi Edge→ISP đúng T02 và hai Lo100 (3 prefix), ISP→Edge 6 prefix, ISP→peer/INET 7 prefix; các chiều iBGP và INET cũng có capture. Cấu hình filter không xuất default từ Edge ra ISP hoặc đưa Lo0/HQ/Branch LAN vào route xuất. Kiểm trạng thái/review policy, chưa chạy canary deny chủ động.

## Startup sau import và xử lý

**Đối chiếu bản gốc sau đó:** [kiểm riêng lab gốc](sprint-04-source-startup-check-2026-11-07.md) không tái hiện vấn đề này: 5/5 startup đã có đầy đủ BGP, khớp running và hai Lo100 ping10/10 sau boot, không chạy `wr` trên bản gốc trong phép kiểm. Chưa xác định nguyên nhân riêng của lần import ban đầu.

Snapshot đầu trên năm IOSv BGP cho running đã có BGP nhưng startup chưa chứa `router bgp`; không xác định nguyên nhân từ riêng snapshot này. Giữ raw output và các file `*.restore-initial.startup.cfg`. Đã chạy `write memory` trên bảy IOSv bản restore, thấy `[OK]`, đối chiếu các dòng cấu hình running/startup, rồi Stop/Start lại bảy IOSv. Sau khi toàn bộ peer nhận đủ prefix, thu bộ cuối: 7/7 cặp running/startup tương đương khi bỏ header, comment và dòng trống, BGP và traffic vẫn đạt. [Kết quả parity/hash](sprint-04-restore-2026-11-07/startup-parity.json), CFG trong các thư mục `configs/{edge,isp,internet-test,branch}/sprint-04/`.

Quy trình restore bàn giao cần kiểm running đã đủ và `wr` sau import, rồi kiểm startup/restart; ZIP chứa payload đầy đủ nhưng không thay cho kiểm NVRAM sau boot. Không chỉnh IP, ASN, BGP policy, timer hoặc tạo static fallback trong lần khắc phục này. Config set Exported của bản gốc/bản restore vẫn nguyên byte như ZIP.

## Giới hạn và lịch sử giữ lại

Các mẫu boot sớm có BGP chưa đủ prefix, Edge ping 0/10, một số VPCS ping 9/10 và warm-up timeout/host-not-reachable. Đây không được ghi thành 10/10. Bộ cuối đo sau khi BGP/HSRP và route ổn định; không có phép đo loss/thời gian hội tụ liên tục. Một lượt capture chồng readiness poll có console xen output và API412 khi session thay đổi; bỏ lượt đó khỏi bằng chứng chốt, giữ raw để truy vết. Snapshot cuối dùng console độc lập sau readiness.

[Chỉ mục raw và bộ snapshot nghiệm thu](sprint-04-restore-2026-11-07/README.md) phân biệt rõ các mốc. Bản restore được giữ chạy để xem lại; lab gốc và các ZIP lịch sử được giữ. FW/NAT/VPN/client HQ ra WAN chưa thuộc kết quả này.
