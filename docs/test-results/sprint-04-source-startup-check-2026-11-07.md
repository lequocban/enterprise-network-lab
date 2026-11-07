# Sprint 04 — Kiểm startup BGP trên bản gốc

**Ngày kiểm:** 07/11/2026, Asia/Saigon. **Nguồn:** agent trực tiếp truy cập API/telnet theo yêu cầu người thực hành.  
**Kết luận:** **Không tái hiện lỗi startup thiếu BGP trên bản gốc**. Cả năm router BGP có đúng `router bgp` trong startup và running; các dòng cấu hình khớp khi bỏ header, comment, dòng trống. Sau boot/hội tụ, hai Edge ping Lo100→INET 10/10.

## Phân biệt instance và giới hạn thao tác

Bản gốc `/LAB-NETWORK/test_import/Enterprise-Network-Integrated.unl`, lab ID `f5f1a877-a874-48f4-9d08-efeff0f0f4eb`. Bản restore có ID `78ce863a-812a-4b65-b242-12bc93da62fa` và đường dẫn riêng. Vì hai lab dùng cùng node ID/cổng console và API status có thể hiển thị cùng trạng thái, đã tạm Stop bảy IOSv của bản restore, xác nhận năm cổng console BGP 32776/32778/32779/32780/32781 đóng, rồi Start chỉ năm router BGP ở đường dẫn **bản gốc**. Uptime capture là 0 phút, phù hợp lần boot này.

Không chạy `wr`, thay startup set, Wipe hoặc lệnh configuration trên bản gốc. `terminal length 0` chỉ dùng để thu output không phân trang. Sau kiểm, Stop năm router BGP bản gốc và Start lại bảy IOSv bản restore để đưa nhóm chạy về trạng thái trước phép kiểm.

## Kết quả

- EDGE1 và EDGE2: startup/running đều có `router bgp 65010`, đầy đủ neighbor/prefix-list và route-map inbound. Hai bộ dòng cấu hình khớp.
- ISP1: startup/running đều có `router bgp 65101`, các peer Edge/ISP2/INET và bộ lọc; hai bộ dòng khớp.
- ISP2: startup/running đều có `router bgp 65102`, các peer Edge/ISP1/INET và bộ lọc; hai bộ dòng khớp.
- INET-TEST: startup/running đều có `router bgp 65200`, originate default/target và hai ISP; hai bộ dòng khớp.
- Sau hội tụ, EDGE1 default/target qua ISP1 `.18`, LP200, RIB distance20; EDGE2 best qua iBGP EDGE1 `.10`, LP200, RIB distance200, backup ISP2 `.22` LP100.
- EDGE1 source Lo100 `198.51.100.11`→INET `192.0.2.10`: **10/10**, RTT1/2/3 ms. EDGE2 source Lo100 `198.51.100.12`→INET: **10/10**, RTT2/2/4 ms.

Các snapshot sớm có PfxRcd0 khi peer mới lên; bộ traffic cuối được lấy sau khi đủ prefix. Bản gốc không cần thao tác lưu bổ sung để khắc phục vấn đề này. Kết quả chỉ xác nhận trạng thái hiện tại của năm router gốc; chưa xác định nguyên nhân startup thiếu BGP ở lần import mới trước đó.

[So sánh cấu hình 5/5](sprint-04-source-check-2026-11-07/config-comparison.json), [EDGE1 traffic](sprint-04-source-check-2026-11-07/EDGE1-153609.txt), [EDGE2 traffic](sprint-04-source-check-2026-11-07/EDGE2-153609.txt). Các raw startup/running tương ứng được dẫn theo trường `evidence` trong JSON; giữ output boot chưa sẵn sàng tách với bộ chốt.
