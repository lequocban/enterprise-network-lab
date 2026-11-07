# Sprint 04 — Internet Edge và BGP

**Chốt kỹ thuật:** 07/11/2026, Asia/Saigon.  
**Trạng thái:** **CLOSED WITH FOLLOW-UPS — NET-401–404 và INT-04 đạt phạm vi kỹ thuật**. Các follow-up về đo convergence, lỗi IOL kế thừa và quy trình restore được ghi bên dưới. Bộ hồ sơ bàn giao qua nhánh `lab/sprint-04-internet-edge`; PR/merge là bước riêng. Commit và trạng thái remote đối chiếu bằng lịch sử Git.

## Sản phẩm bàn giao

Một topology chung `Enterprise-Network-Integrated` gồm 17 node/21 network: Campus kế thừa, hai Edge, hai ISP, INET-TEST, hai Branch và sáu PC. Lab phát triển ở `LAB-NETWORK/test_import`; bản restore nghiệm thu ở `LAB-NETWORK/SPRINT04-RESTORE-2026-11-07`. [ZIP cuối](../../eve-exports/sprint-04-enterprise-final-2026-11-07.zip) có 17 payload Exported, SHA-256 `245C9B3C6ED7343C946BBB531DFB93151226C404F8ECFB9EA66496F1AA813D50`; [kiểm ZIP](../test-results/sprint-04-final-export-inspection-2026-11-07.md), [17 CFG nguyên byte](../../configs/sprint-04-final/README.md), [restore thật](../test-results/sprint-04-final-restore-2026-11-07.md) và [raw console](../test-results/sprint-04-restore-2026-11-07/README.md) cùng định danh artifact.

[Bài BGP/failover](../routing/bgp-failover.md) giải thích next-hop-self, scoped Local Preference, filtering, route chiều về và giới hạn HA. [Ảnh topology WAN](../../screenshots/sprint-04/final-wan-annotated-2026-11-07.png) và [17 node Exported](../../screenshots/sprint-04/final-configured-nodes-2026-11-07.png) do người thực hành cung cấp; API và console trong ca restore là evidence agent thu trực tiếp, không thay ảnh bằng mô phỏng.

## Nghiệm thu NET-401–404 và INT-04

- **NET-401 đạt:** provider AS65101/65102 và INET AS65200 học/quảng bá đúng default/target và return route; topology WAN chung đúng cổng/IP.
- **NET-402 đạt:** EDGE1 AS65010 eBGP ISP1, originate T02/Lo100 theo allow-list, source Lo100 ping INET 10/10.
- **NET-403 đạt:** EDGE2/iBGP/next-hop-self, ISP1 LP200 cho exact default/target và ISP2 LP100 backup. WAN Branch còn LP100 để tránh detour không cần thiết. Advertised-routes khớp filter ở các peer đã review.
- **NET-404 đạt chức năng:** mất ISP1/T03 đổi sang ISP2, mất T04/ISP2 giữ ISP1, W08 đi vòng T07, INET mất thì rút route, mất từng Edge thì Edge còn sống hoạt động; mỗi ca đã hoàn nguyên.
- **INT-04 đạt:** Campus/WAN/Branch trong cùng UNL, hồi quy và transport đúng phạm vi, ZIP/config/ảnh cùng version; import bản sao boot cả 17 node và smoke các nhóm đạt. NVRAM ban đầu thiếu BGP đã được xử lý bằng lưu running/startup và restart xác nhận, quy trình restore ghi rõ thao tác này.

## Test cases và nguồn bằng chứng

- **TC-01/02 đạt:** hai Lo100 tới INET 10/10, primary LP200/backup LP100; [EDGE2/iBGP](../test-results/sprint-04-edge2-ibgp-2026-10-25.md) và bộ restore cuối.
- **TC-03 đạt:** [Stop ISP1](../test-results/sprint-04-isp1-node-failover-2026-10-26.md), [shutdown T03](../test-results/sprint-04-t03-link-failover-2026-10-26.md); hai nguồn qua ISP2 sau hội tụ.
- **TC-04 đạt:** recovery primary, [T04](../test-results/sprint-04-t04-link-failover-2026-10-26.md), [ISP2](../test-results/sprint-04-isp2-node-failover-2026-10-26.md); backup trở lại.
- **TC-05 đạt:** [W08](../test-results/sprint-04-w08-upstream-failover-2026-10-26.md), AS_PATH/trace qua ISP1→ISP2→INET và hồi phục.
- **TC-06 đạt:** [INET Stop/Start](../test-results/sprint-04-inet-node-withdrawal-2026-10-26.md), default/target rút khỏi BGP/RIB cả hai Edge rồi phục hồi.
- **TC-07 đạt:** [Campus regression](../test-results/sprint-04-campus-vpcs-regression-2026-11-07.md) và restore: IT↔ACC mỗi chiều10/10, Po1/Po11 hai P, HSRP đủ sáu VLAN, trunk phù hợp.
- **TC-08 đạt:** [Stop/Start từng Edge](../test-results/sprint-04-edge-node-failover-2026-11-07.md), nguồn Edge còn sống10/10 và recovery.
- **TC-09 đạt:** [Branch underlay](../test-results/sprint-04-branch-underlay-2026-10-26.md) và restore cuối, PC↔gateway và Branch WAN↔Edge WAN mỗi chiều10/10.
- **TC-10 đạt phạm vi review/route xuất hiện tại:** prefix-list/route-map và advertised-routes của sáu quan hệ BGP đã đối chiếu. Canary Lo0 deny chủ động chưa chạy; không coi vắng prefix chưa originate là một phép thử deny.
- **TC-11 đạt theo runbook restore:** [persistence IOSv lab phát triển](../test-results/sprint-04-iosv-persistence-2026-11-07.md), ZIP CRC/XML/payload và [restore thật](../test-results/sprint-04-final-restore-2026-11-07.md); 7/7 cặp running/startup sau lưu/restart khớp dòng cấu hình, sáu VPCS boot đúng IP/gateway.

Các failure tests ở buổi 6–7 dùng output người thực hành gửi; ca restore và startup đối chiếu 07/11 do agent thu trực tiếp qua API/telnet. Giữ các output chưa hội tụ và snapshot copy nhầm trong lịch sử với giải thích, không sửa kết quả cũ thành mẫu đạt.

## Follow-up và giới hạn bàn giao

- Chưa đo thời gian hội tụ/loss liên tục. Mẫu sau hội tụ10/10 không chứng minh zero-loss trong chuyển đổi; W09/T07 bổ sung và canary deny có thể mở thành bài tiếp theo khi cần.
- Restore lần đầu thấy running BGP đã đủ nhưng startup chưa có phần BGP ở năm router. Đã lưu bảy IOSv, so sánh và restart đạt; đưa bước kiểm/lưu sau import vào runbook. Chưa kết luận nguyên nhân chỉ từ output console.
- IOL Core vẫn dùng `no ip cef`; lỗi Po1 khi chỉ một member kế thừa Sprint2 chưa được đóng. Regression này dùng Po1/Po11 đủ hai member.
- BGP default/INET là mô phỏng. FW/DMZ/NAT, HQ client→Internet/Branch và VPN chưa triển khai; nghiệm thu ở INT-05/06 sau này. Branch có một ISP mỗi site.
- EVE API cuối có 17 process node, mem33% và swap0%; chưa có kiểm tải toàn Enterprise. API node-status giữa hai lab trùng node IDs có giới hạn, audit ghi riêng và config set nguồn được đối chiếu nguyên byte.
- Nhánh bàn giao `lab/sprint-04-internet-edge` chứa hồ sơ Sprint4 và các tài liệu thiết kế tích hợp mà nó tham chiếu. PR/merge thực hiện riêng. Bản sửa roadmap định dạng lớn, hai kế hoạch Sprint2/Sprint10 và file tải tạm được giữ ngoài commit Sprint4 để review theo phạm vi.

Sprint5 có thể bắt đầu từ baseline ZIP này và runbook restore: thêm ASAv/T01/FW vào T02, xây default/return/NAT/policy rồi kiểm traffic client HQ. Không nối tắt Campus–Edge để thay bước firewall.
