# Sprint 04, buổi 3 — EDGE1 ↔ ISP1, checkpoint runtime

**Ngày nhận output:** 25/10/2026, Asia/Saigon; console router không có timestamp tuyệt đối.  
**Nguồn:** output CLI do người thực hành gửi từ EVE host, EDGE1, ISP1 và INET-TEST. Agent không điều khiển trực tiếp EVE/router.  
**Story:** NET-402, S4-S03; kế tiếp [provider buổi 2](sprint-04-provider-bgp-2026-10-24.md).  
**Trạng thái:** **PASS checkpoint buổi 3** về T03, eBGP, route chiều về và nguồn Lo100. Người thực hành xác nhận đã lưu EDGE1/ISP1; ISP1 summary bổ sung có đủ ba neighbor Established. Chưa có startup dump/export CFG mới hoặc thử restart/restore. Dual ISP/Local Preference chưa nghiệm thu.

## IP, T03 và tài nguyên

- EVE host: RAM tổng 7,7 GiB, đang dùng 2,5 GiB, available 4,9 GiB; swap 4,0 GiB, đang dùng 0. `uptime` tại 12:21:18 báo up 11 phút, load average 3,30 / 1,00 / 0,35. Output không tự xác nhận chính xác nhóm node đang bật; nhóm bốn IOSv là chỉ dẫn buổi 3.
- EDGE1 Gi0/0 `10.255.0.10`, Gi0/1 `10.255.0.17`, Lo0 `10.255.255.11`, Lo100 `198.51.100.11` đều up/up. Gi0/2–3 chưa cấp IP, administratively down.
- Connected RIB có T02 `10.255.0.8/29`, T03 `10.255.0.16/30` và hai loopback. `show ip route 10.255.0.8` xác nhận connected Gi0/0 và “Advertised by bgp 65010”.
- EDGE1 source `.17` ping ISP1 `.18` trên T03: **10/10**, min/avg/max **1/1/2 ms**.

## BGP, RIB và filter quan sát được

- EDGE1 router-ID `10.255.255.11`, AS65010; neighbor ISP1 `10.255.0.18` AS65101 có `State/PfxRcd` **6**, InQ/OutQ 0, uptime 00:00:40: eBGP Established tại thời điểm lấy mẫu.
- EDGE1 có 8 network/8 path: default, target `192.0.2.10/32`, T02, bốn WAN khách hàng và Lo100 của chính EDGE1. Default/target best qua `10.255.0.18`, AS_PATH `65101 65200`; target đã cài RIB BGP distance 20, next-hop `.18`.
- T03 `10.255.0.16/30` hiện `r>` trong BGP table: BGP path qua ISP1 hợp lệ/được chọn trong BGP nhưng route connected cùng prefix đã có trong RIB với distance 0, nên BGP path không được cài. Đây là RIB-failure do connected route ưu tiên, không phải mất T03.
- EDGE1 `advertised-routes` sang ISP1 có **đúng 2 prefix**: T02 `10.255.0.8/29` và Lo100 `198.51.100.11/32`. Prefix EDGE2 `.12/32` chưa xuất khi EDGE2/iBGP chưa có; không ghi là filter đã chủ động từ chối prefix cấm.
- ISP1 BGP/RIB có Lo100 `.11/32` từ `10.255.0.17`, AS_PATH `65010`, best/external; INET-TEST có **2 path** đến `.11/32`: qua ISP2 AS_PATH `65102 65101 65010` và qua ISP1 `65101 65010` là best. RIB INET cài route via `203.0.113.1`, xác nhận đường trả về qua ISP1.
- Summary ISP1 bổ sung: router-ID `10.255.255.101`, AS65101, 8 network/10 path; neighbor EDGE1 `10.255.0.17` AS65010 nhận **2** prefix, ISP2 `10.255.0.34` AS65102 nhận **4**, INET-TEST `203.0.113.2` AS65200 nhận **2**. Ba hàng có InQ/OutQ 0, `State/PfxRcd` là số: cả ba session Established tại thời điểm lấy mẫu.

## Data plane và giới hạn

EDGE1 source `198.51.100.11` ping `192.0.2.10` repeat 10: **10/10**, min/avg/max **1/2/3 ms**. Traceroute cùng source: hop 1 `10.255.0.18` (AS65101), hop 2 `203.0.113.2` (AS65200); một probe `*` ở hop 2 nhưng hai probe khác có reply và ping 10/10. Không suy mất gói dữ liệu end-to-end từ một probe traceroute không trả lời.

Provider checkpoint trước buổi này đã có `wr` và ZIP. Buổi 3 thay đổi **EDGE1 và ISP1**; người thực hành xác nhận **hai router đã lưu** sau khi được yêu cầu `wr`/đợi `[OK]`, nhưng không gửi riêng hai dòng `[OK]` console. Đây là xác nhận thao tác, chưa phải đối chiếu startup dump. Chưa có export CFG/ZIP mới hoặc thử restart/restore. Không yêu cầu lặp ping đã đạt nếu cấu hình không đổi.

Chưa có EDGE2/iBGP/Local Preference 200/100, thử mất ISP, Branch transport hoặc traffic HQ qua FW ở checkpoint này.
