# Sprint 04, buổi 6 — Shutdown/restore T03 trên EDGE1

**Ngày nhận output:** 26/10/2026, Asia/Saigon; console có log `%SYS-5-CONFIG_I` nhưng không có timestamp đầy đủ cho cả quá trình chuyển đường.  
**Nguồn:** output EDGE1/EDGE2 do người thực hành gửi theo hai mốc T03 `shutdown` và sau `no shutdown`; agent không điều khiển router.  
**Ca:** INT-04-TC-03, phần mất T03; INT-04-TC-04, recovery T03. Tách khỏi [ca Stop/Start ISP1](sprint-04-isp1-node-failover-2026-10-26.md).  
**Trạng thái:** **PASS ca T03 shutdown và recovery**: hai Edge chuyển qua ISP2 khi mất T03, hai Lo100 ping 10/10; sau `no shutdown`, cả hai Edge trở lại đường ISP1 LP200 và ping 10/10. Chưa đo thời gian hội tụ/loss trong chuyển đổi.

## T03 shutdown

- EDGE1 Gi0/1 `10.255.0.17` là **administratively down/down**; Gi0/0 T02, Lo0, Lo100 vẫn up/up. eBGP ISP1 `.18` ở **Idle** (0 message sau mất phiên); iBGP EDGE2 `.11` vẫn Established, nhận 8 prefix.
- EDGE1 BGP target `192.0.2.10/32` còn một path best qua iBGP EDGE2 `.11`, AS_PATH `65102 65200`, LP100. RIB target là BGP internal distance200, next-hop `.11`. EDGE1 source Lo100 `.11/32` ping target **10/10**, min/avg/max **2/3/4 ms**; trace hop EDGE2 `.11` → ISP2 `.22` → INET `.6`.
- EDGE2 target còn một path best eBGP ISP2 `.22`, AS_PATH `65102 65200`, LP100. RIB external distance20 via `.22`. EDGE2 source Lo100 `.12/32` ping target **10/10**, 1/2/3 ms; trace ISP2 `.22` → INET `.6`.
- Hai trace có một probe `*` tại hop cuối; ping hoàn chỉnh 10/10 ở cả hai nguồn. Không suy loss end-to-end hoặc thời gian hội tụ từ `*` hoặc từ mẫu ping sau route ổn định.

## T03 `no shutdown` và phục hồi

- EDGE1 Gi0/1 trở lại **up/up**; eBGP ISP1 `.18` nhận **6** prefix, uptime 00:00:18, InQ/OutQ0; iBGP EDGE2 `.11` nhận 4 prefix.
- EDGE2 BGP target có hai path: iBGP qua EDGE1 `.10`, AS_PATH `65101 65200`, **LP200 best**; eBGP ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best. EDGE2 source Lo100 `.12` ping target **10/10**, 2/2/4 ms.
- Output bổ sung EDGE1: target best qua ISP1 `.18`, AS_PATH `65101 65200`, LP200; RIB BGP external distance20 via `.18`, tag65101. EDGE1 source Lo100 `.11` ping INET target **10/10**, 2/2/3 ms. Như vậy control plane, RIB và data plane của cả hai Edge đã trở lại đường chính ở mẫu ổn định.

Không có bằng chứng `wr` trong trạng thái shutdown ở output gửi; ca dự kiến chỉ thay cổng tạm và hoàn nguyên, không yêu cầu lưu cấu hình thử. Chưa test T04/ISP2 trong lượt này. Nếu không có phép đo ping liên tục với thời điểm, không báo thời gian hội tụ hay số gói mất trong chuyển đổi. Giữ [ZIP trước lỗi](sprint-04-underlay-export-2026-10-26.md) nguyên byte.
