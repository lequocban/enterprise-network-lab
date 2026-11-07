# Sprint 04 — Internet Edge, BGP và phục hồi

**Mốc thực hành:** 24/10–07/11/2026. Kết quả và artifact bàn giao theo [báo cáo Sprint 4](../sprint-reports/sprint-04-report.md).

## Đường đi và policy thực tế

EDGE1/EDGE2 cùng AS65010, iBGP qua T02 `10.255.0.8/29`, next-hop-self hai chiều. EDGE1 `.10`/EDGE2 `.11` là IP Gi0/0 trên bridge; bridge chưa có firewall. EDGE1 Gi0/1 `.17` eBGP ISP1 `.18` AS65101; EDGE2 Gi0/1 `.21` eBGP ISP2 `.22` AS65102. T07 nối ISP1 `.33` với ISP2 `.34`. INET-TEST AS65200 cung cấp default và Lo100 `192.0.2.10/32` cho hai ISP; static default Null0 chỉ ở INET để tạo default mô phỏng.

EDGE1 dùng `RM-ISP1-LP200` seq10 match prefix-list exact `PL-UPSTREAM` gồm default và target rồi set LP200; seq20 permit các route còn được inbound prefix-list cho phép, set LP100. EDGE2 ISP2 LP100. Do đó Internet target/default trên cả hai Edge chọn ISP1, còn các WAN Branch ưu tiên path phù hợp thay vì bị ép hết sang ISP1. EDGE2→BR2 WAN `.29` đi trực tiếp ISP2 `.22` sau tinh chỉnh scoped LP. Local Preference được xét trước độ dài AS_PATH: khi W08 mất nhưng ISP1 còn học qua ISP2, hai Edge vẫn chọn exit ISP1 LP200 dù AS_PATH dài hơn.

Baseline: EDGE1 target/default next-hop `.18`, LP200, RIB eBGP distance20; EDGE2 best iBGP `.10`, LP200, RIB distance200, có alternate eBGP `.22` LP100. Hai nguồn test là EDGE1 Lo100 `198.51.100.11`, EDGE2 Lo100 `.12`; INET có return route cho cả hai. Traffic reply không buộc đi đối xứng với chiều gửi.

## Filter và originate

Mỗi Edge originate T02 `/29` và Lo100 riêng; có thể quảng bá Lo100 của peer qua iBGP rồi xuất sang ISP theo allow-list. Advertised-routes tới mỗi ISP sau hội tụ có đúng ba prefix: T02 và hai Lo100. Default/target không xuất lại từ Edge ra ISP. Lo0 và HQ/Branch LAN không nằm trong bộ route xuất. INET chỉ quảng bá default/target cho ISP; ISP originate WAN khách hàng mình sở hữu và trao đổi các prefix đã cho phép qua T07. Không dùng OSPF doanh nghiệp trên ISP.

[Output restore](../test-results/sprint-04-restore-2026-11-07/README.md) có advertised-routes ở hai chiều iBGP, các phiên ISP và INET; 14 snapshot running/startup cùng ZIP cho phép review prefix-list/route-map. Đây là kiểm policy và route đang xuất thực tế; chưa chạy canary chủ động bị deny, không gọi sự vắng mặt Lo0 là bằng chứng đã thử deny một route được originate.

## Các ca failure/recovery đã chạy

- [Stop ISP1](../test-results/sprint-04-isp1-node-failover-2026-10-26.md) và [shutdown riêng T03](../test-results/sprint-04-t03-link-failover-2026-10-26.md): EDGE1 qua iBGP EDGE2 `.11`, EDGE2 qua ISP2 `.22`, LP100; hai Lo100 ping INET 10/10 sau hội tụ. ISP1/T03 hồi phục thì ISP1 LP200 trở lại.
- [Shutdown T04](../test-results/sprint-04-t04-link-failover-2026-10-26.md) và [Stop ISP2](../test-results/sprint-04-isp2-node-failover-2026-10-26.md): primary ISP1 giữ traffic; ISP2 LP100 trở lại làm backup sau hoàn nguyên.
- [Mất W08 ISP1–INET](../test-results/sprint-04-w08-upstream-failover-2026-10-26.md): ISP1 học target/default qua ISP2 trên T07; EDGE1 AS_PATH `65101 65102 65200` vẫn LP200. Trace EDGE2→EDGE1→ISP1→ISP2→INET; hai nguồn 10/10. W08 hồi phục thì ISP1 trực tiếp INET, AS_PATH ngắn lại.
- [Stop INET-TEST](../test-results/sprint-04-inet-node-withdrawal-2026-10-26.md): default/target bị rút khỏi BGP và RIB ở cả hai Edge. Sau Start, route/traffic hai nguồn trở lại.
- [Stop từng Edge](../test-results/sprint-04-edge-node-failover-2026-11-07.md): Edge còn sống ping INET 10/10 qua ISP tương ứng; sau Start primary/backup phục hồi. Không đo từ Lo100 của router đang tắt.

Các mẫu được chấm là sau hội tụ. Chưa có phép đo mất gói liên tục hoặc thời gian chuyển đường. Trace có probe `*` không tự chứng minh loss end-to-end. Branch chỉ có một ISP mỗi site; mất ISP đó làm Branch mất underlay theo thiết kế.

## Restore và thao tác vận hành

Import [ZIP cuối](../../eve-exports/sprint-04-enterprise-final-2026-11-07.zip) vào thư mục trống riêng; 17 node phải có Startup-config Exported. Start INET, hai ISP rồi hai Edge; đợi peer nhận đủ prefix và INET có return route hai Lo100 trước khi lấy mẫu 10 gói. Chạy Campus/Branch theo tài nguyên thực tế.

Trong lần restore 07/11, năm router BGP có running đúng nhưng startup chưa chứa `router bgp` ở snapshot đầu. Đã `write memory` cả bảy IOSv, đối chiếu running/startup và restart lại; bộ snapshot cuối khớp các dòng cấu hình và BGP/traffic đạt. Quy trình bàn giao vì vậy gồm **kiểm startup sau import, lưu khi running đã đầy đủ, rồi restart để kiểm lại**; giữ payload ZIP và snapshot boot tách biệt. Chi tiết và output ban đầu nằm trong [biên bản restore](../test-results/sprint-04-final-restore-2026-11-07.md).

Campus giữ workaround `no ip cef` trên hai IOL Core và lỗi Po1 một member kế thừa Sprint2 chưa được đóng bằng regression này. Không áp workaround đó lên IOSv. FW/NAT/DMZ và VPN chưa triển khai trong Sprint4; client HQ ra Internet/Branch sẽ kiểm ở các mốc tiếp theo.
