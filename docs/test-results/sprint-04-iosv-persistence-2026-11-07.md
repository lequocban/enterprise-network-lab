# Sprint 04, buổi 8 — IOSv persistence sau Stop/Start

**Ngày nhận output:** 07/11/2026, Asia/Saigon; không có timestamp tuyệt đối cho toàn bộ lượt Stop/Start.  
**Nguồn:** output CLI INET-TEST, ISP1/ISP2, EDGE1/EDGE2, BR1-R1/BR2-R1 do người thực hành gửi; người thực hành xác nhận đã Stop/Start theo hướng dẫn sau khi `wr` đủ bảy IOSv. Agent không điều khiển EVE.  
**Trạng thái:** **PASS chức năng persistence của bảy IOSv**: sau Start, địa chỉ/cổng, BGP/default/target, static default Branch và hai mẫu ping Lo100 khớp baseline. Đây là bằng chứng boot từ NVRAM và traffic thực; chưa có phép so sánh toàn văn running/startup config hoặc restore ZIP trên bản sao.

## INET-TEST và ISP

- INET-TEST Gi0/0 `203.0.113.2`, Gi0/1 `.6`, Lo100 `192.0.2.10` đều up/up và Method `NVRAM`; Gi0/2–3 chưa dùng admin down. `show ip bgp summary` ở uptime peer 00:00:40–44 cho hai ISP ở trạng thái nhận **0 prefix**. Đây là ảnh chụp sớm sau Start; các output ISP/Edge tiếp theo cho thấy route đã xuất hiện và traffic hoạt động.
- ISP1 Gi0/0 `.18`, Gi0/1 `.26`, Gi0/2 `.33`, Gi0/3 `203.0.113.1`, Lo0 `.101` đều up/up/NVRAM. eBGP INET `.2` nhận 2 prefix; eBGP ISP2 `.34` và EDGE1 `.17` có prefix. Default và target best trực tiếp qua INET `.2`, AS_PATH `65200`. Output BGP table ISP1 ở checkpoint sớm có dòng `Not advertised to any peer`; không dùng dòng này để kết luận route không tới Edge ở mốc ổn định.
- ISP2 Gi0/0 `.22`, Gi0/1 `.30`, Gi0/2 `.34`, Gi0/3 `203.0.113.5`, Lo0 `.102` đều up/up/NVRAM. eBGP INET `.6` nhận 2 prefix; ISP1 `.33` nhận 7, EDGE2 `.21` nhận 3. Default và target có best trực tiếp INET `.6`, AS_PATH `65200`, và alternate qua ISP1 `.33`.

## EDGE1 và EDGE2

- EDGE1 Gi0/0 `.10`, Gi0/1 `.17`, Lo0 `.11`, Lo100 `198.51.100.11` đều up/up/NVRAM; Gi0/2–3 chưa dùng admin down. iBGP EDGE2 `.11` nhận 4 prefix, eBGP ISP1 `.18` nhận 6. Default và target `192.0.2.10/32` best eBGP `.18`, AS_PATH `65101 65200`, LP200. RIB target distance20 via `.18`. Ping từ Lo100 `.11` tới INET target **10/10**, min/avg/max **1/2/4 ms**.
- EDGE2 Gi0/0 `.11`, Gi0/1 `.21`, Lo0 `.12`, Lo100 `198.51.100.12` đều up/up/NVRAM; Gi0/2–3 chưa dùng admin down. iBGP EDGE1 `.10` và eBGP ISP2 `.22` đều nhận 6 prefix. Default và target có ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best, cùng iBGP EDGE1 `.10`, AS_PATH `65101 65200`, LP200 best. RIB target distance200 via `.10`. Ping từ Lo100 `.12` tới INET target **10/10**, 2/2/4 ms.

## Hai Branch

- BR1-R1 Gi0/0 `10.255.0.25`, Gi0/1 `10.20.10.1`, Lo0 `10.255.255.21` đều up/up/NVRAM; static default distance1 via ISP1 `.26`.
- BR2-R1 Gi0/0 `10.255.0.29`, Gi0/1 `10.30.10.1`, Lo0 `10.255.255.22` đều up/up/NVRAM; static default distance1 via ISP2 `.30`.
- Hai Branch chưa có ping PC/gateway mới trong lượt persistence; VPCS và Campus hồi quy thuộc checkpoint tiếp theo.

Các cổng Gi0/2–3 chưa dùng trên Edge/Branch/INET ở trạng thái admin down là phù hợp cấu hình xuất trước đó. Mốc này chứng minh các phần cấu hình cần cho dịch vụ còn hoạt động sau Stop/Start; không suy rằng ZIP cuối đã restore được hoặc đã kiểm mọi dòng startup-config.
