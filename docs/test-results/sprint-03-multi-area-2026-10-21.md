# Sprint 3 — Checkpoint Multi-Area OSPF

**Ngày tiếp nhận:** 21/10/2026, Asia/Saigon.  
**Nguồn:** output console do người thực hành cung cấp trong chat. Agent không chạy console.  
**Phạm vi:** [routing sandbox](../routing/sprint-03-routing-design.md), kế thừa [single-area](sprint-03-single-area-2026-10-20.md); không phải tích hợp Enterprise INT-03/04/05/06 đã nghiệm thu.

## Area 10 — kết quả đã nhận

- EDGE1 Gi0/0 `.37`, Gi0/1 `.41`, Gi0/2 `.45`, Lo0 `.11` đều up/up. Có đủ3neighbor FULL/- với BR2`.22` qua`.46`Gi0/2, EDGE2`.12` qua`.38`Gi0/0 và BR1`.21` qua`.42`Gi0/1.
- BR1 `show ip ospf interface brief`: Lo0`.21/32`, Gi0/1`10.20.10.1/24`, Gi0/0`.42/30` đều process10/Area10. Gi0/0P2P cóNbrs1/1; Gi0/1stateDR/Nbrs0/0 không phải neighbor transit bầu DR. Chưa có `show ip ospf interface Gi0/1` xác minh passive riêng.
- EDGE2 học LANBR1 `10.20.10.0/24`, OSPF10/inter-area/AD110/metric3 qua`.37`, nguồn ABR router-id`10.255.255.11`.
- BR1 học mạng PC-HQ `10.255.0.52/30`, OSPF10/inter-area/AD110/metric3 qua`.41`, nguồn ABR`.11`.
- PC-HQ → PC-BR1 và PC-BR1 → PC-HQ mỗi chiều5/5reply, TTL61. Đây là actual5gói, không ghi10/10.

**Kết luận:** bước chuyển Branch1 có neighbor, area phía BR1, inter-area route hai phía và traffic đạt. Chưa có interface brief/process summary/LSDB EDGE1 để đối chiếu riêng toàn bộ area và vai trò ABR; chưa có xác nhận wr sau chuyểnArea10. Không ghi toàn bộ NET-302 hoặc DoD DONE.

## Area 20 — bước tiếp theo, chưa chạy

Đổi EDGE1Gi0/2 và BR2Gi0/0 từArea0 sangArea20; chuyển cả Gi0/1 và Lo0 BR2 sangArea20. Giữ IP/RID/p2p/passive; EDGE1Gi0/0/Lo0 và toàn bộ EDGE2 vẫnArea0, BR1 vẫnArea10.

Expected: EDGE1 tham gia0/10/20, có3FULL và là ABR; BR2 có mọi interface OSPF ởArea20 và1FULL. EDGE2/BR1 học LANBR2 inter-area; BR2 học LANBR1 và mạng PC-HQ inter-area, metric3 với cost hiện tại. Đối chiếu LSDB/Type3, test3cặp PC hai chiều sau chuyển, hoàn nguyên nếu sai rồi mới lưu checkpoint multi-area.

Rollback Area20: trả đúng bốn interface vừa đổi vềArea0, giữ phầnArea10 đã đạt. Nếu chưa lưu Area10 thì wr EDGE1/BR1 và chờ `[OK]` trước bướcArea20. Không yêu cầu lặp save khi đã xác nhận lưu đúng checkpoint.

## Multi-area 0/10/20 — kết quả sau chuyển Area20

**Actual21/10/2026:**

- EDGE1 interface brief: Lo0/Gi0/0Area0, Gi0/1Area10, Gi0/2Area20, process10; ba transit P2P/Nbrs1/1. Neighbor brief đủ3FULL/- với EDGE2`.12`, BR1`.21`, BR2`.22` trên đúng cổng/peer.
- LSDB EDGE1 có Router Link States (Type1) theo các Area0/10/20, Summary Net Link States (Type3) với ADV Router`.11` quảng bá LAN/transit/loopback giữa các area. Chưa có area range ở bước này; tên Summary LSA không chứng minh đã gộp prefix.
- Area0 còn các Router LSA cũ của BR1`.21`/BR2`.22` có Age1258 trong mẫu, trong khi hai Branch đã chuyển area và các area mới có LSA trẻ hơn. Ghi nhận nguyên trạng; chưa có mẫu aging tiếp theo để kết luận đã xóa LSA cũ. Không clear process chỉ để làm sạch output hoặc suy ra loop/failure từ entry cũ.
- EDGE2 LANBR2 `10.30.10.0/24`: OSPF10/inter-area/AD110/metric3 qua`.37`, nguồn ABR`.11`.
- BR2 Lo0/Gi0/1/Gi0/0 đềuArea20; Gi0/0P2P/Nbrs1/1. RIB có OIA LANBR1 và PC-HQ metric3, các transit/loopback remote phù hợp qua`.45`. Chưa có lookupLANBR2 riêng trên BR1 trong lượt này, không tự tạo output đó.
- Ba cặp PC HQ–BR1, HQ–BR2, BR1–BR2 đều có echo reply5/5 ở từng chiều, tổng6chiều. Ghi đúng mẫu5gói; không đổi thành10/10.

**Kết luận:** multi-area đã có bằng chứng chức năng area/adjacency/Type1–Type3/inter-area RIB và client traffic hai chiều. Phần chức năng của NET-302 có nguồn; không tự đánh dấu fullDoD, CP2fullsnapshot hoặc restore khi còn thiếu artifact. Các mốc tích hợp Enterprise chưa nghiệm thu.

## Console và lưu checkpoint

Người thực hành yêu cầu timeout console45phút và logging synchronous. Đã hướng dẫn `line console 0`, `exec-timeout 45 0`, `logging synchronous`, rồi wr riêng trên4router. Người thực hành xác nhận “đã xong” ngày21/10; ghi hoàn thành/lưu theo xác nhận, không tạo raw config/wr hoặc kiểm thử timeout45phút giả. Không cấu hình VTY chỉ từ hướng dẫn console. Không yêu cầu lặp wr cho checkpoint này khi chưa thay đổi.

**Bước tiếp:** NET-303, thêm Loopback110 mạng thử10.20.11.0/24 và10.30.11.0/24 trong source area10/20, passive và network typepoint-to-point để quảng bá đúngmask. Đọc EDGE2 có đủ4prefix /24 và ping các loopback trước range; sau đó mới gộp2prefix mỗi Branch thành/23 tại ABR EDGE1. Chưa có actual cho Loopback110 hoặc summary.

## Area mismatch và khôi phục — actual 22/10/2026

- Pha thử đúng: BR2-R1 Gi0/0 được đổi tạm từ Area20 sang Area99. Syslog trên BR2 ghi neighbor `10.255.255.11` từ FULL xuống DOWN do interface detached; `show ip ospf interface brief` ghi Gi0/0 Area99, P2P, Nbrs0/0, trong khi `show ip interface brief` vẫn up/up. BR2 không còn OSPF neighbor và lookup default trả `% Network not in table`.
- EDGE1 trong pha lỗi vẫn FULL với EDGE2 `.12` và BR1 `.21`, không còn neighbor BR2 `.22`. Trước pha này từng có mismatch ngoài dự kiến trên EDGE1 Gi0/0 khi đặt Area20 đối diện EDGE2 Area0; EDGE1 sau đó được xác nhận trở lại Area0/P2P/cost1 bằng `show ip ospf interface` và running interface. Không gộp hai link lỗi thành một phép đo.
- Sau trả BR2 Gi0/0 về Area20, BR2 neighbor EDGE1 FULL, default OSPF10 external type2 qua `10.255.0.45` trở lại, AD110/metric1/tag10; PC-BR2 ping `192.0.2.10` 5/5.

**Kết luận:** S03-TC-12 có bằng chứng về mất và khôi phục adjacency/route dù link IP up/up; data plane được xác nhận sau khôi phục. Không có ping PC-BR2 trong pha lỗi nên không tự ghi traffic fail actual. Chưa có bằng chứng `wr` BR2/EDGE1 sau hoàn nguyên ở lượt này; persistence kiểm riêng tại buổi8. Sơ đồ cuối giữ Area0 trên EDGE1–EDGE2 và Area20 trên EDGE1–BR2.

## Buổi 8 — persistence BR2 sau stop/start, actual 22/10/2026

Người thực hành xác nhận đã `wr` trên các router. Startup-config BR2 trước phép thử có Gi0/0 `10.255.0.46/30`, `ip ospf network point-to-point`, `ip ospf 10 area 20`; Gi0/1 `10.30.10.1/24` Area20; OSPF10 RID `.22`, passive default, Gi0/0 non-passive, console 45 phút/logging synchronous. Trước đó EDGE1 có đủ 3 neighbor FULL.

Sau thao tác stop/start BR2 theo trình tự buổi8, BR2 có Gi0/0 `.46`, Gi0/1 `10.30.10.1`, Lo0 `.22`, Lo110 `10.30.11.1` up/up; OSPF Gi0/0 Area20/P2P/Nbrs1/1, Lo110 `10.30.11.1/24`; neighbor EDGE1 `.11` FULL. BR2 học lại default OSPF external type2 qua `.45`, AD110/metric1/tag10; PC-BR2 ping `192.0.2.10` 5/5, TTL252. Không có `show version` uptime sau restart trong mẫu; kết luận stop/start dựa trên trình tự người thực hành đã thực hiện và output sau đó.

**Kết luận:** persistence/functional recovery đạt cho BR2 và đường PC-BR2→target trong phạm vi thử. Chưa khởi động lại bốn IOSv khác; chưa đối chiếu full startup/running config của cả năm hoặc EVE export/import.
