# Sprint 01 — Quan sát traffic khi thử uplink ACCESS1

**Ngày tiếp nhận:** 11/10/2026, Asia/Saigon. Log không có timestamp; chưa xác định thời điểm chạy thiết bị.

## Nguồn và phạm vi

Người thực hành gửi log VPCS sau hướng dẫn shutdown rồi no shutdown ACCESS1 Ethernet0/0, ping từ PC-IT đến PC-ACC `10.10.40.20` bằng `ping 10.10.40.20 -t`. File chỉ chứa output ping, không chứa lệnh thao tác switch, IP nguồn, STP hoặc HSRP tại thời điểm thử.

[Log nguyên bản do người thực hành cung cấp](sprint-01-access1-uplink-ping-2026-10-11.txt) được sao chép nguyên trạng từ tệp đính kèm; agent không chạy lệnh trên thiết bị.

## Kết quả quan sát

- Sequence 1–24: nhận 24 phản hồi.
- Sequence 25–39: 15 timeout liên tiếp; phản hồi trở lại ở sequence 40.
- Sequence 40–119: nhận 80 phản hồi liên tiếp.
- Sequence 120–134: 15 timeout liên tiếp; phản hồi trở lại ở sequence 135.
- Sequence 135–152: nhận 18 phản hồi liên tiếp trước Ctrl+C.
- Tổng phần log: 152 phép thử, 122 phản hồi, 30 timeout. Mọi phản hồi hiển thị TTL 63.

Log chứng minh traffic phục hồi sau hai khoảng gián đoạn và có chuỗi phản hồi ổn định sau mỗi khoảng. Chưa thể xác định chính xác thời gian hội tụ từ sequence do thiếu timestamp/interval/timeout thực tế. Không gán hai khoảng gián đoạn cho shutdown/no shutdown như một sự kiện đã đối chiếu thời gian, và không suy ra vai trò cổng STP hoặc trạng thái HSRP từ ping.

## Trạng thái và quyết định phạm vi

- **S01-TC-11:** đã có bằng chứng traffic liên VLAN gián đoạn rồi phục hồi. Chưa có output STP trước/trong/sau, xác nhận phục hồi cấu hình hoặc phép thử cùng VLAN độc lập; chưa đánh dấu toàn bộ ca PASS theo tiêu chí gốc.
- **S01-TC-12:** **BỎ QUA theo yêu cầu người thực hành ngày 11/10/2026**. Không thực hiện thêm bài ngắt uplink ACCESS2 trong Sprint 1; đây là giới hạn coverage, không phải PASS và không chứng minh hai Access có failover tương đương.
- Có thể tiếp tục bước lưu baseline/bàn giao với ngoại lệ này được nêu rõ trong báo cáo. Trạng thái toàn Sprint 1 vẫn chờ kiểm tra cuối và artifact thực tế.
- Giữ workaround `no ip cef` trên hai Core theo quyết định trước đó. Không kết luận nguyên nhân hai khoảng mất gói là CEF hoặc shared link chỉ từ log này.

## Bước bàn giao tiếp theo

Xác nhận uplink ACCESS1 Ethernet0/0 đã được no shutdown, endpoint đã về VLAN/IP đích, thu trạng thái cuối và lưu running/startup bốn switch. Không cần gây lỗi ACCESS2 để tiếp tục theo phạm vi đã điều chỉnh.

**Cập nhật khi chốt báo cáo 11/10/2026:** cấu hình cuối hai Access không còn shutdown uplink, các cổng endpoint đúng VLAN đích. [Báo cáo](../sprint-reports/sprint-01-report.md) và [validation](sprint-01-validation.md) đã hoàn thiện; startup/restore và số đo STP trong lúc lỗi giữ là follow-up. Không cần lặp failure test hoặc thu lại running để hoàn thiện hồ sơ lần này.
