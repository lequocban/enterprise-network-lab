# Bằng chứng restore Sprint 04 — 07/11/2026

Nguồn: agent truy cập API EVE và console telnet theo ủy quyền. Tất cả lệnh chạy trên `/LAB-NETWORK/SPRINT04-RESTORE-2026-11-07/Enterprise-Network-Integrated.unl`. Caption raw dùng UTC; ngày giờ bàn giao theo Asia/Saigon.

## Snapshot nghiệm thu ổn định

- [CORE1](CORE1-151601.txt)
- [CORE2](CORE2-151603.txt)
- [ACCESS1](ACCESS1-151444.txt)
- [ACCESS2](ACCESS2-151447.txt)
- [INET-TEST](INET-TEST-152053.txt)
- [ISP1](ISP1-152054.txt)
- [ISP2](ISP2-152054.txt)
- [EDGE1](EDGE1-152103.txt)
- [EDGE2](EDGE2-152104.txt)
- [BR1-R1](BR1-R1-152100.txt)
- [BR2-R1](BR2-R1-152106.txt)
- [PC-IT](PC-IT-152232.txt)
- [PC-HR](PC-HR-152232.txt)
- [PC-ACC](PC-ACC-152232.txt)
- [PC-GUEST](PC-GUEST-152246.txt)
- [PC-BR1](PC-BR1-152246.txt)
- [PC-BR2](PC-BR2-152246.txt)

[Startup parity 7/7](startup-parity.json) · [Trạng thái/tài nguyên cuối](final-audit.json) · [Biên bản restore](../sprint-04-final-restore-2026-11-07.md).

## Giữ lịch sử boot và đối chiếu

Các file thời điểm sớm còn được giữ: một số ping PC 9/10, warm-up timeout/host not reachable, Edge ping 0/10 và BGP chưa đủ prefix khi boot. Chỉ snapshot ổn định ở trên được dùng chấm mẫu 10 gói. Snapshot đầu startup thiếu phần BGP ở năm router; đã lưu running xuống startup và restart kiểm lại. Một lượt capture song song với readiness poll có output console xen nhau và API 412 do phiên đăng nhập thay đổi; không dùng lượt đó làm bằng chứng chốt. Các capture cuối được thu sau readiness và không chạy poll đồng thời.
