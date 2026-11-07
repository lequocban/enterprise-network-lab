# Sprint 04, buổi 5 — Branch underlay và gateway LAN

**Ngày nhận output:** 26/10/2026, Asia/Saigon. Nguồn là console BR1-R1/BR2-R1, EDGE1/EDGE2 và PC-BR1/PC-BR2 do người thực hành gửi; agent không điều khiển router.  
**Story:** INT-04-TC-09 theo [kế hoạch tích hợp](../routing/enterprise-integration-design.md).  
**Trạng thái:** **PASS checkpoint buổi 5** về Branch underlay, PC–gateway và tinh chỉnh LP: từng cặp WAN/PC đạt 10/10; EDGE2→BR2 đi trực tiếp ISP2, hai Lo100→INET vẫn 10/10 với upstream ISP1 LP200. BR1/BR2 và EDGE1 sau sửa policy có `wr [OK]`, hai PC đã `save`. [ZIP mới](sprint-04-underlay-export-2026-10-26.md) có đủ 17 payload. Chưa có bằng chứng VPN, PC Branch↔HQ, kiểm rò prefix chủ động hoặc restore.

## BR1

- BR1-R1 Gi0/0 `10.255.0.25`, Gi0/1 `10.20.10.1`, Lo0 `10.255.255.21` đều up/up; Gi0/2–3 admin down. Static default distance1 tới ISP1 `10.255.0.26`.
- BR1 `.25`→ISP1 `.26` 10/10, RTT 2/2/2 ms; BR1 `.25`→EDGE1 `.17` 10/10, 2/2/3 ms.
- EDGE1 RIB có T05 `10.255.0.24/30` qua ISP1 `.18`, BGP external distance20. EDGE1 `.17`→BR1 `.25` **10/10**, 2/2/2 ms. Hai chiều WAN đã có route và traffic thật.
- PC-BR1 `10.20.10.20/24`, gateway `10.20.10.1`; hai mẫu ping gateway riêng biệt **10/10** mỗi mẫu, `save` báo xong. BR1 `wr` có `[OK]`, sau đó log GRUB báo ghi thành công.

## BR2

- BR2-R1 Gi0/0 `10.255.0.29`, Gi0/1 `10.30.10.1`, Lo0 `10.255.255.22` đều up/up; Gi0/2–3 admin down. Static default distance1 tới ISP2 `10.255.0.30`.
- BR2 `.29`→ISP2 `.30` **10/10**, 1/1/2 ms; BR2 `.29`→EDGE2 `.21` **10/10**, 2/2/4 ms.
- EDGE2 `.21`→BR2 `.29` **10/10**, 2/2/5 ms. Tuy nhiên RIB EDGE2 cho T06 `10.255.0.28/30` là **iBGP distance200, next-hop EDGE1 `10.255.0.10`**, tag65101, AS hops2. Vì vậy lượt chiều về đạt kết nối nhưng hop đầu đi qua EDGE1, không phải ISP2 trực tiếp. Đoạn tiếp theo EDGE1→ISP1→ISP2 là suy luận từ policy/BGP baseline; chưa có traceroute hoặc RIB EDGE1 đương thời cho chính T06 trong mẫu này.
- PC-BR2 `10.30.10.20/24`, gateway `10.30.10.1`; ping gateway **10/10**, `save` báo xong. BR2 `wr` có `[OK]`, log GRUB sau đó báo ghi thành công.

## Nhận định và việc tiếp nhận

ZIP [CP2 dual ISP](sprint-04-dual-isp-export-2026-10-26.md) cho thấy route-map LP200 inbound ISP1 trên EDGE1 không có `match`; nó gán LP200 cho cả default/target lẫn bốn WAN T03–T06. EDGE2 nhận route T06 qua iBGP từ EDGE1 với LP200, thắng route T06 trực tiếp từ ISP2 LP100. Đây là giải thích phù hợp với RIB thực tế; không suy đường trực tiếp từ ping thành công.

**Đã tinh chỉnh policy theo output BR2:** trên EDGE1 tạo `PL-UPSTREAM` permit exact default và target; route-map `RM-ISP1-LP200` seq10 match prefix-list và set LP200, seq20 catch-all set LP100. Người thực hành chạy `clear ip bgp 10.255.0.18 soft in` đúng neighbor. EDGE1 BGP default và target vẫn **LP200 best via ISP1 `.18`**. EDGE2 RIB T06 `10.255.0.28/30` đã đổi từ iBGP distance200 via EDGE1 `.10` sang **eBGP distance20 via ISP2 `.22`**, tag65102, AS hops1. EDGE2 `.21`→BR2 `.29` retest **10/10**, RTT 2/2/3 ms; traceroute hop1 ISP2 `.22`, hop2 BR2 `.29`. Một probe `*` ở hop2 nhưng ping 10/10; không suy mất kết nối từ probe trace đơn lẻ.

**Kiểm giữ ưu tiên Internet sau thay đổi:** EDGE2 BGP default và target đều có hai path: iBGP qua EDGE1 `.10`, AS_PATH `65101 65200`, **LP200 best**; eBGP trực tiếp ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best. EDGE2 source Lo100 `.12` ping INET target **10/10**, 2/2/3 ms; EDGE1 source Lo100 `.11` ping cùng target **10/10**, 1/2/4 ms. Như vậy policy mới giải quyết đường BR2 mà không làm mất mục tiêu Internet primary ISP1 theo BGP/traffic hiện quan sát.

**Lưu cấu hình mới:** EDGE1 `wr` sau sửa route-map trả `[OK]`; log GRUB tiếp theo báo cấu hình đã ghi xuống disk thành công. Cùng với hai BR `wr [OK]` và hai VPCS `save`, checkpoint chức năng/lưu buổi 5 đạt theo output. [ZIP mới](sprint-04-underlay-export-2026-10-26.md) đã chứa cấu hình sau tinh chỉnh và Branch; giữ [ZIP CP2](sprint-04-dual-isp-export-2026-10-26.md) nguyên byte để so trước/sau. `wr`/`save`/ZIP chưa thay bài thử restart/restore.

Output `EDGE1#noset` trong mẫu không cho thấy cấu hình đã đổi; không suy thay đổi router từ một lệnh/chuỗi chưa có phản hồi. Đã có `wr [OK]` hai BR và `save` hai PC. Chưa có `free -h`/`uptime` hoặc danh sách node đang bật riêng cho phép đo Branch ngày26/10. Chưa có export CFG/ZIP chứa Branch mới; không suy persistence sau restart từ riêng `wr`/`save`.
