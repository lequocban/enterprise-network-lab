# Sprint 04 — Kiểm tra export WAN cabling, 24/10/2026

**Nguồn:** ZIP do người thực hành export từ lab tích hợp sau khi nối WAN/Branch; agent kiểm artifact tại local, chưa trực tiếp chạy EVE hoặc router.  
**Artifact hiện tại:** [sprint-04-enterprise-wan-cabling-2026-10-24.zip](../../eve-exports/sprint-04-enterprise-wan-cabling-2026-10-24.zip), **4.580 byte**, SHA-256 `C407EC95D2E56D3040B92E2A8046E26E043F62805C7127277B41E5ED1698505A`.  
**Trạng thái Gate B:** **ĐẠT kiểm tĩnh cabling/inventory và payload HQ** sau khi node ID12 được đổi tên thành `ISP2`. Buổi 1 đạt hai Gate theo phạm vi đã đặt; IP/BGP và hồi quy Campus sau mở rộng thuộc bài kiểm tiếp theo.

## Kết quả kiểm tĩnh

- ZIP CRC đạt, XML parse được; một member `Enterprise-Network-Integrated.unl`, lab name cùng tên.
- Có **17 node, 21 network object**, mỗi network đúng hai interface; tên/ID node là duy nhất. T02 có `name="T02-VLAN91"`, `type="bridge"`, nối EDGE1 Gi0/0 và EDGE2 Gi0/0.
- **11 network HQ** và từng cặp cổng giữ nguyên so với [seed 23/10](../../eve-exports/sprint-04-enterprise-seed-2026-10-23.zip).
- Có **8 payload cấu hình HQ** (4 IOL + 4 VPCS), đều Base64 giải mã được và **trùng từng byte với payload trong seed**. Trên tám node HQ, `config="1"` (Startup-configs ON); trên chín node WAN/Branch mới, `config="0"` và chưa có payload config. Điều này phù hợp checkpoint cabling trước cấu hình, không chứng minh WAN runtime hoặc lần restore mới.
- Bảy IOSv mới (hai Edge, hai ISP, INET-TEST, hai BR) đều có image `vios-adventerprisek9-m-15.6.2T`, CPU 1, RAM 1024 MB, 4 Ethernet; hai PC Branch là VPCS. EVE icon mây của INET-TEST không đổi template IOSv.
- 10 network mới khớp đúng cặp cổng thiết kế với tên `ISP2`: T02, T03–T07, W08/W09 và hai LAN Branch. Không có cổng mới nào bị nối hai network hoặc cổng lẻ không có peer trong export này. [Ảnh topology](../../screenshots/sprint-04/wan-cabling-complete-2026-10-24.png) phù hợp kết quả UNL; ảnh được chụp trước khi sửa tên ISP.

## Lịch sử sửa tên trong checkpoint

Bản export đầu tiên cùng tên file có **4.581 byte**, SHA-256 `21AEF47F9E4F06D5319773E5B2B18EE8D3B433E3AA5BA6B5E57E9D2A23DD9419`; node ID12 tên `ISP12`, dù dây T04/T06/T07/W09 đã đúng cổng. Người thực hành đổi Name trong EVE thành `ISP2`, xóa ZIP đầu và đặt export mới **cùng tên file**. Vì vậy digest cũ là **lịch sử kiểm tra**, không phải hash của link artifact hiện tại; repo không giữ bytes ZIP đầu.

Đã kiểm lại artifact hiện tại: node ID12 là `ISP2`; 17 node/21 network, 10 cặp WAN/Branch, 11 cặp HQ, T02 `type="bridge"`, bảy IOSv và 8 payload HQ đều giữ đúng như trước. ZIP CRC/XML đạt; không phát hiện interface bị gán trùng.

Kết quả kiểm tĩnh đi cùng bài test Campus [10/10 hai chiều trước WAN](sprint-04-campus-baseline-2026-10-23.md); chưa có bài hồi quy sau mở rộng, IP/BGP, traffic WAN, persistence hoặc restore bản mới. INT-04 tiếp tục ở Sprint 4.
