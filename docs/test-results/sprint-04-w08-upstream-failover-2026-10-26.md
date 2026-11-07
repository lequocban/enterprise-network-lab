# Sprint 04, buổi 7 — Shutdown/restore W08 ISP1–INET-TEST

**Ngày nhận output:** 26/10/2026, Asia/Saigon; không có timestamp tuyệt đối cho shutdown/no shutdown.  
**Nguồn:** output ISP1/EDGE1/EDGE2 do người thực hành gửi theo hai mốc; agent không điều khiển router.  
**Ca:** INT-04-TC-05, phần mất riêng W08 và recovery.  
**Trạng thái:** **PASS chức năng W08 down/recovery**: hai Edge giữ traffic tới INET qua ISP1→ISP2 khi W08 mất; ISP1 quay lại đường trực tiếp khi W08 phục hồi, hai Edge ping 10/10. Không suy thời gian hội tụ hoặc loss lúc chuyển.

## W08 down

- ISP1 Gi0/3 `203.0.113.1` administratively down/down; các cổng T03/T05/T07 vẫn up/up. eBGP INET `.2` ở Idle; eBGP ISP2 `.34` nhận 7 prefix và eBGP EDGE1 `.17` nhận 3.
- ISP1 default và target `192.0.2.10/32` còn một path best qua ISP2 `.34`, AS_PATH `65102 65200`, LP100. RIB target là BGP external distance20 via `.34`.
- EDGE1 target best qua ISP1 `.18`, AS_PATH `65101 65102 65200`, LP200; RIB target BGP external distance20 via `.18`. EDGE1 source Lo100 `198.51.100.11` ping target **10/10**, min/avg/max **2/2/3 ms**. Traceroute: ISP1 `.18` → ISP2 `.34` → INET `.6`.
- EDGE2 target có hai path: iBGP qua EDGE1 `.10`, AS_PATH `65101 65102 65200`, LP200 best; eBGP ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best. RIB target internal distance200 via `.10`. EDGE2 source Lo100 `198.51.100.12` ping target **10/10**, min/avg/max **2/3/5 ms**; traceroute EDGE1 `.10` → ISP1 `.18` → ISP2 `.34` → INET `.6`.
- Một probe `*` ở hop cuối traceroute; ping end-to-end đủ 10/10. Nhãn AS trong output traceroute tại `.34` không dùng thay BGP AS_PATH; địa chỉ `.34` và BGP ISP1 xác nhận bước qua ISP2.

## W08 `no shutdown` và ISP1 route recovery

- ISP1 Gi0/3 trở lại up/up; eBGP INET `.2` nhận 2 prefix, eBGP ISP2 `.34` vẫn nhận 7, eBGP EDGE1 `.17` nhận 3, InQ/OutQ0.
- ISP1 target có hai path: trực tiếp INET `.2`, AS_PATH `65200`, best; qua ISP2 `.34`, AS_PATH `65102 65200`, valid/non-best. RIB target external distance20 via `.2`.
- EDGE1 target qua ISP1 `.18`, AS_PATH trở lại `65101 65200`, LP200 best; source Lo100 `.11` ping target **10/10**, 2/2/4 ms.
- EDGE2 target có lại iBGP EDGE1 `.10`, AS_PATH `65101 65200`, LP200 best, và eBGP ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best. RIB target internal distance200 via `.10`; source Lo100 `.12` ping target **10/10**, 2/2/3 ms.

Một probe `*` ở hop cuối traceroute EDGE2 khi W08 down; các probe khác và ping end-to-end trả lời. Không có bằng chứng `wr` khi cổng đang shutdown. Nếu thao tác chỉ là shutdown/no shutdown tạm và startup baseline vốn `no shutdown`, không cần lưu lại. Chưa chạy ca INET-TEST node-down.
