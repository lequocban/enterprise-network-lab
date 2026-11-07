# Sprint 04, buổi 8 — Baseline và lưu IOSv trước kiểm persistence

**Ngày nhận output:** 07/11/2026, Asia/Saigon.  
**Nguồn:** CLI EVE host, EDGE1/EDGE2, ISP1/ISP2 và BR1-R1/BR2-R1 do người thực hành gửi; ảnh topology EVE do người thực hành đính kèm. Agent không điều khiển EVE.  
**Trạng thái:** **PASS checkpoint baseline và lưu theo xác nhận người thực hành**; chưa có output `[OK]` riêng của từng node trong tin nhắn này, chưa có Stop/Start sau lượt lưu, chưa chốt persistence hoặc restore ZIP.

## Runtime trước lưu

- EVE host `free -h`: tổng 7,7 GiB, used 1,7 GiB, available 5,7 GiB, swap 0/4,0 GiB. `uptime` lúc 14:06:10: up 1:13, 2 users, load average 2,47/2,04/1,90.
- Sau khi bật thêm hai Branch, `free -h`: used 1,9 GiB, available 5,5 GiB, swap vẫn 0/4,0 GiB. Số liệu cho thấy còn RAM ở hai mốc đã đo; không chứng minh toàn bộ 17 node đang chạy cùng lúc.
- ISP1 Gi0/0 `.18`, Gi0/1 `.26`, Gi0/2 `.33`, Gi0/3 `203.0.113.1`, Lo0 `.101` đều up/up. ISP2 Gi0/0 `.22`, Gi0/1 `.30`, Gi0/2 `.34`, Gi0/3 `203.0.113.5`, Lo0 `.102` đều up/up. Cổng W08 trên ISP1 đã trở lại up/up sau thử lỗi.
- EDGE1 Gi0/0 `.10`, Gi0/1 `.17`, Lo0 `.11`, Lo100 `198.51.100.11` đều up/up; Gi0/2–3 chưa dùng admin down. iBGP EDGE2 `.11` nhận 4 prefix, eBGP ISP1 `.18` nhận 6, InQ/OutQ0.
- EDGE2 Gi0/0 `.11`, Gi0/1 `.21`, Lo0 `.12`, Lo100 `198.51.100.12` đều up/up; Gi0/2–3 chưa dùng admin down. iBGP EDGE1 `.10` và eBGP ISP2 `.22` mỗi peer nhận 6 prefix, InQ/OutQ0.
- BR1-R1 Gi0/0 `.25`, Gi0/1 `10.20.10.1`, Lo0 `.21` đều up/up; static default via ISP1 `.26`. BR2-R1 Gi0/0 `.29`, Gi0/1 `10.30.10.1`, Lo0 `.22` đều up/up; static default via ISP2 `.30`. Gi0/2–3 chưa dùng admin down ở cả hai Branch.

Người thực hành xác nhận đã chạy `wr` trên đủ bảy IOSv: EDGE1, EDGE2, ISP1, ISP2, INET-TEST, BR1-R1, BR2-R1. Tin nhắn không kèm từng dòng `[OK]`, nên biên bản ghi đây là **xác nhận thao tác**, chưa phải dump/so sánh startup-config hoặc phép thử reboot sau lưu. Ảnh topology cho thấy nhãn WAN/Branch/Edge và đường nối đang hiển thị; ảnh không thay kết quả CLI về BGP hoặc cấu hình lưu.

**Tiếp theo:** Stop/Start sau lượt lưu theo nhóm, kiểm cổng/BGP/default/target và traffic; hồi quy VPCS/Campus, rồi export ZIP cuối và thử restore trên bản sao riêng. Không coi ZIP đã kiểm tĩnh ngày 26/10 là bản export cuối sau buổi 8.
