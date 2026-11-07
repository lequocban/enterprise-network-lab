# Sprint 04, buổi 6 — Stop/Start ISP1, failover node-down

**Ngày nhận output:** 26/10/2026, Asia/Saigon; console không có timestamp tuyệt đối cho sự kiện Stop/Start.  
**Nguồn:** CLI EVE host, EDGE1 và EDGE2 do người thực hành gửi, theo ba mốc baseline → ISP1 Stop → ISP1 Start. Agent không điều khiển EVE.  
**Ca:** INT-04-TC-03, phần ISP1 node-down; INT-04-TC-04, recovery cùng node.  
**Trạng thái:** **PASS chức năng failover sau hội tụ và hồi phục primary** đối với hai nguồn Lo100 tới cùng INET-TEST. Không có số liệu mất gói/thời gian hội tụ trong giai đoạn chuyển đổi; chưa thử shutdown T03/T04 hoặc Stop ISP2.

## Mốc A — baseline trước Stop

- EVE host: RAM tổng 7,7 GiB, used 1,8 GiB, available 5,7 GiB; swap dùng 0/4,0 GiB. `uptime` lúc 08:02:52 báo up 42 phút, load average 4,30/2,20/2,18. Output không tự xác nhận chính xác danh sách node chạy, nhưng hai Edge và các neighbor liên quan có output runtime.
- EDGE1 AS65010: iBGP EDGE2 `.11` nhận 4 prefix, eBGP ISP1 `.18` nhận 6; cả hai Established/InQ-OutQ0. Target `192.0.2.10/32` best qua ISP1 `.18`, AS_PATH `65101 65200`, LP200; RIB eBGP distance20 via `.18`. Source Lo100 `.11`→INET target **10/10**, min/avg/max 2/2/3 ms.
- EDGE2 AS65010: iBGP EDGE1 `.10` nhận 6, eBGP ISP2 `.22` nhận 6; cả hai Established. Target có hai candidate: ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid; iBGP EDGE1 `.10`, AS_PATH `65101 65200`, **LP200 best**. RIB iBGP distance200 via `.10`. Source Lo100 `.12`→target **10/10**, 2/2/3 ms. Lệnh ping đầu tại prompt `EDGE2>` bị parser từ chối; sau `enable`, cùng lệnh tại `EDGE2#` chạy thành công. Đây là privilege mode, không phải lỗi BGP.

## Mốc B — ISP1 Stop

- EDGE1 default và target còn một path best qua iBGP EDGE2 `.11`, AS_PATH `65102 65200`, LP100. RIB có `B* 0.0.0.0/0 [200/0]` và target `/32 [200/0]` via `.11`. T03 `10.255.0.16/30` vẫn hiện connected Gi0/1 trong RIB sau Stop; do đó carrier/connected route không đủ để kết luận ISP1 còn chạy, trong khi BGP upstream qua ISP1 đã bị rút.
- EDGE2 default/target còn một path best eBGP ISP2 `.22`, AS_PATH `65102 65200`, LP100; RIB `B*` và target distance20 via `.22`. Route tới Lo100 EDGE1 `.11/32` qua iBGP `.10`, hỗ trợ return path cho source EDGE1 khi ra ISP2.
- EDGE1 source Lo100 `.11`→target **10/10**, 2/2/3 ms; trace hop 1 EDGE2 `.11`, hop 2 ISP2 `.22`, hop 3 INET `.6`.
- EDGE2 source Lo100 `.12`→target **10/10**, 2/2/3 ms; trace hop 1 ISP2 `.22`, hop 2 INET `.6`. Mỗi trace có một probe `*` ở hop cuối nhưng hai probe còn lại reply và ping 10/10; không suy mất traffic end-to-end từ `*` đó.

Không có `show ip bgp summary` ở mốc B trong output gửi, nên trạng thái FSM/uptime peer ISP1 tại đúng mốc này chưa được lưu riêng. BGP table/RIB/traffic đủ chứng minh hai Edge đã đổi đường sau hội tụ. Chỉ có mẫu ping ổn định sau khi route đổi, không đo số gói mất trong chuyển đổi hoặc thời gian hội tụ.

## Mốc C — ISP1 Start và hồi phục

- EDGE1 `show ip bgp summary`: eBGP ISP1 `.18` nhận 6 prefix, iBGP EDGE2 `.11` nhận 4, InQ/OutQ0. BGP default và target trở lại best qua ISP1 `.18`, AS_PATH `65101 65200`, LP200. EDGE1 Lo100 `.11`→target **10/10**, 2/2/3 ms; trace EDGE1→ISP1 `.18`→INET `.2`.
- EDGE2 default và target lại có hai path: iBGP EDGE1 `.10`, AS_PATH `65101 65200`, LP200 best; eBGP ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best. EDGE2 Lo100 `.12`→target **10/10**, 2/2/3 ms; trace EDGE2 `.10`→ISP1 `.18`→INET `.2`.

User không gửi trực tiếp `ISP1#show ip bgp summary` hoặc RIB EDGE2 sau Start; không ghi riêng ba session phía ISP1 đã được nhìn thấy ở console ISP1 trong mốc C. E1/E2 best path, summary E1 và hai ping/traces chứng minh primary chức năng đã quay lại. ISP1 Stop/Start là một phép thử node runtime; chưa phải import/restore từ [ZIP baseline trước lỗi](sprint-04-underlay-export-2026-10-26.md).

**Giới hạn và tiếp theo:** không thay đổi cấu hình trong ca; không cần `wr` khi đã phục hồi. BR1 có một ISP nên underlay của nó dự kiến mất khi ISP1 Stop, nhưng BR1/PC-BR1 không được đo trong ca này. Các ca T03 shutdown, T04 shutdown, ISP2 Stop phải tách riêng sau khi baseline hiện tại được xác nhận; không chồng nhiều lỗi hoặc lấy thời gian boot làm thời gian BGP hội tụ.
