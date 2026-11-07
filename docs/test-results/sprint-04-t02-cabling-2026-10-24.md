# Sprint 04 — T02 và node WAN, checkpoint ảnh 24/10/2026

**Nguồn:** sáu ảnh giao diện EVE do người thực hành cung cấp ngày 24/10/2026, Asia/Saigon: [danh sách node ban đầu](../../screenshots/sprint-04/t02-edge-node-settings-2026-10-24.png), [sơ đồ T02](../../screenshots/sprint-04/t02-edge-bridge-topology-2026-10-24.png), [sơ đồ sau thêm ISP/INET](../../screenshots/sprint-04/t02-isp-inet-nodes-2026-10-24.png), [danh sách 13 node](../../screenshots/sprint-04/t03-t04-node-settings-2026-10-24.png), [cabling T03/T04](../../screenshots/sprint-04/t03-t04-edge-isp-cabling-2026-10-24.png) và [toàn bộ WAN/Branch sau nối dây](../../screenshots/sprint-04/wan-cabling-complete-2026-10-24.png). Người thực hành xác nhận đã mở Edit T02 và thấy **Type = bridge**. Ảnh là bằng chứng hiển thị node/cabling; Type là **xác nhận của người thực hành**, chưa có output CLI hoặc export UNL mới.

## Đã đối chiếu từ ảnh

- Giao diện **Configured Nodes** liệt kê 10 node: bốn IOL Core/Access, bốn VPCS HQ và hai node `EDGE1`, `EDGE2` template `vios`.
- Hai hàng EDGE1/EDGE2 hiển thị CPU `1`, RAM `1024 MB`, Ethernet `4`, console telnet. Ô Boot Image bị rút gọn trong ảnh (`vios-adv…`), nên chưa đối chiếu chính xác toàn bộ tên image từ ảnh này.
- Sơ đồ có một network object nhãn `T02-VLAN91`; EDGE1 **Gi0/0** và EDGE2 **Gi0/0** nối vào cùng biểu tượng đó. Dây Campus HQ vẫn hiển thị bên dưới; chưa có ảnh kiểm đếm lại từng link HQ sau thao tác.
- Không thấy FW nối vào T02. Đây là trạng thái đúng phạm vi Sprint 4 tại mốc ảnh; FW outside tiếp nhận Sprint 5.
- Ảnh mới cho thấy thêm `ISP1`, `ISP2`, `INET-TEST` ở cùng sơ đồ; tổng cộng **13 node hiển thị**. Ba node mới chưa nối dây trong ảnh, nên T03/T04/T07/W08/W09 còn ở bước tiếp theo. Người thực hành xác nhận đã tạo đúng mức 1 CPU/1024 MB/4 Ethernet cho ba node, nhưng ảnh này không mở bảng thuộc tính để đối chiếu độc lập.
- Ảnh **Configured Nodes** mới cho thấy cả 13 node; `ISP1`, `ISP2`, `INET-TEST` là template `vios`, mỗi node CPU `1`, RAM `1024 MB`, Ethernet `4`. INET-TEST dùng icon hình cloud theo lựa chọn hiển thị, **không phải** EVE Cloud/network object. Boot Image vẫn bị rút gọn trong ô dropdown.
- Ảnh cabling mới cho thấy **T03** EDGE1 Gi0/1 ↔ ISP1 Gi0/0 và **T04** EDGE2 Gi0/1 ↔ ISP2 Gi0/0 đúng mapping INT-03. Hai Edge vẫn Gi0/0 vào T02. Đây là kiểm tra dây hiển thị, chưa có IP hoặc BGP runtime.
- Người thực hành sau đó xác nhận đã nối **T07** ISP1 Gi0/2 ↔ ISP2 Gi0/2. Chưa có ảnh/export sau thao tác này để agent đối chiếu; ghi ở mức **xác nhận người thực hành**, không nâng thành kiểm tra cabling từ artifact.

## Mốc toàn bộ cabling WAN/Branch từ ảnh cuối buổi 1

Ảnh cuối 24/10 hiển thị **17 node** theo tên: 8 node HQ, EDGE1/2, ISP1/2, INET-TEST, BR1-R1/BR2-R1 và PC-BR1/PC-BR2. Các cặp cổng mới khớp mapping thiết kế:

- T02: EDGE1 Gi0/0 và EDGE2 Gi0/0 cùng vào bridge `T02-VLAN91`; Type bridge theo xác nhận người thực hành.
- T03: EDGE1 Gi0/1 ↔ ISP1 Gi0/0. T04: EDGE2 Gi0/1 ↔ ISP2 Gi0/0.
- T05: BR1-R1 Gi0/0 ↔ ISP1 Gi0/1. T06: BR2-R1 Gi0/0 ↔ ISP2 Gi0/1.
- T07: ISP1 Gi0/2 ↔ ISP2 Gi0/2. Cặp cổng nay đã thấy trong ảnh, bổ sung cho xác nhận trước đó.
- W08: ISP1 Gi0/3 ↔ INET-TEST Gi0/0. W09: ISP2 Gi0/3 ↔ INET-TEST Gi0/1.
- LAN Branch: BR1-R1 Gi0/1 ↔ PC-BR1 eth0; BR2-R1 Gi0/1 ↔ PC-BR2 eth0.

Các dây HQ hiện diện trong ảnh, nhưng chưa có export mới để đếm/đối chiếu toàn bộ 11 link kế thừa. Số **21 network object** là dự kiến từ 11 HQ + T02 + 5 link T03–T07 + W08/W09 + 2 LAN Branch; ảnh không hiển thị tổng này. ZIP cabling cuối buổi cần xác nhận node/network/interface và config payload thực tế.

## Phần chờ xác minh

- Trường EVE Network **Type = bridge** đã được người thực hành xác nhận khi mở Edit; chưa có ảnh Edit/export để đối chiếu độc lập. Không suy type chỉ từ tên `T02-VLAN91` hoặc biểu tượng cloud.
- Ảnh chưa cho số network object, thuộc tính Ethernet/full boot image từ export UNL, trạng thái chạy của hai Edge, IP `10.255.0.10/.11`, iBGP hoặc ping. Sau khi dựng đủ WAN, kiểm bằng export/config/runtime đúng phiên bản.
- Ảnh cuối đã cho thấy T05/T06/T07, W08/W09 và Branch LAN; export UNL vẫn cần đối chiếu metadata/số network, và full boot image/cấu hình startup chưa có trong ảnh.

**Kết luận:** cabling mới T02–T07, W08/W09 và hai LAN Branch **khớp theo ảnh**; Type bridge được người thực hành xác nhận. [ZIP cabling hiện tại đã được kiểm tĩnh](sprint-04-wan-export-inspection-2026-10-24.md): đúng 17 node/21 network, tên `ISP2` và giữ đủ 8 payload HQ. Bản export đầu tên `ISP12` đã được sửa trong EVE và thay bằng ZIP mới cùng tên; ảnh topology có trước lúc sửa tên. Gate B đạt phạm vi cabling/inventory. Chưa có IP/BGP runtime trên WAN; INT-04 vẫn đang triển khai. [Baseline Campus trước WAN](sprint-04-campus-baseline-2026-10-23.md) đã đạt theo phạm vi đại diện riêng.
