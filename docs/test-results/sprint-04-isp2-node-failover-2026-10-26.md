# Sprint 04, buổi 6 — Stop/Start ISP2

**Ngày nhận output:** 26/10/2026, Asia/Saigon; không có timestamp tuyệt đối cho thao tác Stop/Start.  
**Nguồn:** output EDGE1/EDGE2 do người thực hành gửi ở hai mốc ISP2 Stop và sau ISP2 Start; agent không điều khiển EVE.  
**Ca:** INT-04-TC-04, mất riêng node ISP2 và phục hồi.  
**Trạng thái:** **PASS chức năng Stop/Start ISP2**. Khi ISP2 Stop, primary ISP1 giữ traffic hai nguồn Lo100 tới INET 10/10. Sau Start, eBGP EDGE2–ISP2 `.22` lên với `PfxRcd 0` ở checkpoint đầu, rồi nhận lại 6 prefix; default và target có lại ISP2 LP100 backup bên cạnh ISP1 LP200 best. Chưa đo loss/thời gian hội tụ trong chuyển đổi.

## ISP2 Stop

- EDGE2 iBGP EDGE1 `.10` vẫn Established, nhận 6 prefix; eBGP ISP2 `.22` ở Idle. Default và target `192.0.2.10/32` chỉ còn một path best qua iBGP `.10`, AS_PATH `65101 65200`, LP200. RIB target BGP internal distance200 via `.10`.
- EDGE2 source Lo100 `198.51.100.12` ping target **10/10**, min/avg/max **2/3/4 ms**; traceroute EDGE1 `.10` → ISP1 `.18` → INET `.2`.
- EDGE1 target vẫn best qua eBGP ISP1 `.18`, AS_PATH `65101 65200`, LP200; RIB external distance20 via `.18`. EDGE1 source Lo100 `198.51.100.11` ping target **10/10**, 1/2/3 ms.

## Sau ISP2 Start — checkpoint ban đầu

- EDGE2 iBGP `.10` nhận 6 prefix; eBGP ISP2 `.22` xuất hiện lại, uptime 00:00:28, InQ/OutQ0, nhưng **PfxRcd 0**.
- EDGE2 default và target vẫn chỉ có một path, iBGP EDGE1 `.10` AS_PATH `65101 65200`, LP200 best; RIB target distance200 via `.10`. EDGE2 Lo100 `.12` ping target **10/10**, 2/2/4 ms.
- Output này chỉ chứng minh primary vẫn hoạt động và phiên EDGE2–ISP2 đã mở lại ở thời điểm đầu. Phần backup được đối chiếu ở mốc sau hội tụ bên dưới; không dùng `PfxRcd 0` ban đầu để kết luận cấu hình lỗi.

## Kiểm sau hội tụ trên ISP2

- ISP2 `show ip bgp summary`: peer EDGE2 `.21` nhận 3 prefix, peer ISP1 `.33` nhận 7, peer INET `.6` nhận 2; cả ba có số prefix thay vì trạng thái FSM, InQ/OutQ0.
- ISP2 default và `192.0.2.10/32` có hai path; best trực tiếp qua INET `.6`, AS_PATH `65200`, valid external. Path qua ISP1 `.33` cũng valid/non-best.
- `advertised-routes` tới EDGE2 `.21` liệt kê **6 prefix**, gồm `0.0.0.0/0` và `192.0.2.10/32`. Đây là bằng chứng phía quảng bá; output EDGE2 tiếp theo xác nhận các đường đã được nhận.
- EDGE1 ở output bổ sung vẫn best target qua ISP1 `.18`, LP200; RIB external distance20 qua `.18`; source Lo100 `.11` ping target **10/10**, 1/2/3 ms.

## EDGE2 sau hội tụ — recovery PASS

- EDGE2 `show ip bgp summary`: iBGP EDGE1 `.10` nhận 6 prefix, eBGP ISP2 `.22` nhận lại **6 prefix**, uptime 00:04:38, InQ/OutQ0.
- Default và target đều có hai path: eBGP ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best; iBGP EDGE1 `.10`, AS_PATH `65101 65200`, **LP200 best**. Như vậy đường dự phòng đã trở lại và chính sách chọn đường chính vẫn đúng.
- EDGE2 source Lo100 `.12` ping target **10/10**, min/avg/max **2/2/3 ms**.

Traceroute tại mốc Stop có một probe `*` ở hop cuối, trong khi ping end-to-end 10/10. Mẫu ping sau hội tụ không đo thời gian chuyển hoặc loss trong chuyển đổi. Không có thay đổi cấu hình được báo trong ca Stop/Start; không cần `wr` cho thao tác node runtime.
