# INT-03 — ASAv preflight, 23/10/2026

Nguồn: output console do người thực hành cung cấp từ ASAv thử của Sprint 0. Chưa xác nhận tên/path lab thử; đây không phải node FW trong topology HQ tích hợp.

- ASA 9.8(1), image `boot:/asa981-smp-k8.bin`; ASDM 7.8(1), ASAv10, RAM 2048 MB.
- `show firewall`: `Firewall mode: Router` — đã xác nhận routed mode.
- Data interfaces có thật: GigabitEthernet0/0, 0/1, 0/2. Management0/0 riêng. Tất cả chưa có IP, administratively down.
- License: Unlicensed, chưa có entitlement; có cảnh báo memory resource allocation vượt permitted limit. Chưa có kết quả throughput/session hoặc cấu hình SLA/track. Không suy cảnh báo này thành EVE hết RAM, không yêu cầu giảm RAM hay tạo lại node.
- `show mode` bị parser từ chối ở prompt hiện tại. Dùng kết quả `show firewall` cho firewall mode; chưa có bằng chứng security-context mode. Không cần đổi mode chỉ vì lệnh này lỗi.
- Lần đầu vào config mode có prompt anonymous error reporting. Hướng dẫn chọn `N` cho bài lab; chưa có xác nhận lựa chọn của người dùng.
- Người dùng báo nhập tùy ý ở prompt enable vẫn vào privileged mode. Chưa có cấu hình authentication để xác định nguyên nhân; password mặc định blank là giả thuyết, không ghi là authentication đã được kiểm chứng. Có thể đặt enable password riêng trên node thử và kiểm đúng/sai nếu cần.

## Kết quả parser bổ sung — 23/10/2026

Người thực hành cung cấp: `sla monitor ?` hiển thị ID `1–2147483647` và `schedule`; `track ?` hiển thị ID `1–500`. Đã thoát config bằng `end`. Hai nhóm lệnh tồn tại trên image này; chưa tạo SLA/object/route hoặc thử chuyển gateway, nên chưa gọi failover PASS.

## Core HSRP parser — 23/10/2026

Trên CORE1 và CORE2 của lab HQ, dưới SVI20 hiện có, `standby 20 track ?` cùng trả về `<1-1000> Tracked object number`; người thực hành thoát bằng `end`. Cả hai image nhận cú pháp HSRP object tracking. Đây chỉ là kiểm parser, chưa tạo track object hoặc cấu hình giảm priority. Không yêu cầu thử thêm cú pháp trên lab gốc để hoàn tất thiết kế.

Phương án triển khai Sprint 5: object track số 90 theo line protocol của cổng T01 trên mỗi Core; HSRP nhóm 10/20/30/40/50/60 giảm 20 điểm khi object down. Kiểm cú pháp tạo object ở bản phát triển khi cấu hình thực tế, sau đó thử failover client. Có thể giữ ASAv hiện tại cho kiểm feature; không cần dựng lại chỉ vì thuộc Sprint 0.
