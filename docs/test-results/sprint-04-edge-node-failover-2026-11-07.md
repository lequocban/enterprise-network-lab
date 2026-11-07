# Sprint 04, buổi 7 — Stop/Start từng Edge

**Ngày nhận output:** 07/11/2026, Asia/Saigon; console không có timestamp tuyệt đối cho thao tác Stop/Start.  
**Nguồn:** output EDGE1/EDGE2 do người thực hành gửi theo bốn mốc; agent không điều khiển EVE.  
**Ca:** INT-04-TC-08, hai lượt node-down độc lập.  
**Trạng thái:** **PASS chức năng**. Edge còn chạy giữ kết nối Lo100 tới INET 10/10; sau Start cả hai Edge trở về primary ISP1 LP200 và backup ISP2 LP100. Không có phép đo loss/thời gian hội tụ trong lúc chuyển đổi.

## A — EDGE1 Stop, EDGE2 còn chạy

- EDGE2 iBGP EDGE1 `.10` ở Idle; eBGP ISP2 `.22` vẫn nhận 6 prefix. Default và target `192.0.2.10/32` chỉ còn path qua ISP2 `.22`, AS_PATH `65102 65200`, LP100 best. RIB target BGP external distance20 via `.22`.
- EDGE2 source Lo100 `198.51.100.12` ping target **10/10**, min/avg/max **1/2/4 ms**; traceroute ISP2 `.22` → INET `.6`.
- Sau EDGE1 Start, EDGE1 iBGP EDGE2 `.11` nhận 4 prefix và eBGP ISP1 `.18` nhận 6. Target best qua ISP1 `.18`, AS_PATH `65101 65200`, LP200; RIB external distance20 via `.18`; source Lo100 `.11` ping **10/10**, 1/2/3 ms.
- EDGE2 iBGP EDGE1 `.10` và eBGP ISP2 `.22` đều nhận 6 prefix. Target có iBGP EDGE1 `.10` LP200 best và eBGP ISP2 `.22` LP100 backup; RIB internal distance200 via `.10`; source Lo100 `.12` ping **10/10**, 2/2/6 ms.

## B — EDGE2 Stop, EDGE1 còn chạy

- EDGE1 iBGP EDGE2 `.11` ở Active; eBGP ISP1 `.18` vẫn nhận 6 prefix. Default và target chỉ còn path ISP1 `.18`, AS_PATH `65101 65200`, LP200 best; RIB target external distance20 via `.18`.
- EDGE1 source Lo100 `.11` ping target **10/10**, 2/2/3 ms; traceroute ISP1 `.18` → INET `.2`.
- Sau EDGE2 Start, EDGE1 iBGP `.11` nhận lại 4 prefix, eBGP ISP1 `.18` nhận 6; target tiếp tục best qua ISP1 `.18` LP200, RIB external distance20 via `.18`; source Lo100 `.11` ping **10/10**, 2/2/4 ms.
- EDGE2 iBGP EDGE1 `.10` và eBGP ISP2 `.22` đều nhận 6 prefix. Target có iBGP `.10` LP200 best và ISP2 `.22` LP100 backup; RIB internal distance200 via `.10`; source Lo100 `.12` ping **10/10**, 2/2/4 ms.

Người thực hành gõ nhầm nhãn bước A.2 “start edge 2” và B.1 “stop edge 1”, nhưng các output sau đó lần lượt chứng minh EDGE1 khởi động lại và EDGE2 dừng: EDGE1 có BGP table version khởi tạo lại ở A.2, EDGE1 thấy iBGP EDGE2 `.11` Active ở B.1, rồi EDGE2 có BGP table version khởi tạo lại ở B.2. Không cần chạy lại chỉ để sửa nhãn. Mỗi traceroute có một probe `*` ở hop cuối trong khi các probe khác và ping end-to-end trả lời; không coi đó là mất kết nối. Các mẫu ping sau hội tụ không đo được loss hoặc thời gian chuyển đường. Stop/Start node không thay cấu hình, nên không cần `wr` cho riêng phép thử này.
