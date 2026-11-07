# Sprint 04, buổi 4 — EDGE2/iBGP, checkpoint trước Local Preference

**Ngày nhận output:** 25/10/2026, Asia/Saigon; console router không có timestamp tuyệt đối.  
**Nguồn:** output EVE host, EDGE1, EDGE2, ISP2, INET-TEST do người thực hành gửi. Agent không điều khiển trực tiếp router.  
**Story:** NET-403, S4-S04. **Trạng thái:** **PASS chức năng buổi 4**: T02/T04 và eBGP/iBGP hoạt động; S4-LP-01 được giải quyết bằng soft inbound refresh; hai Edge chọn ISP1 LP200, cả hai Lo100 ping target 10/10, RIB/trace EDGE2 đúng, outbound filter đúng và người thực hành xác nhận `wr`. Chưa có export CFG/ZIP mới hoặc restart/restore.

## IP, link và nguồn trả về

- EVE host: RAM tổng 7,7 GiB, dùng 1,7 GiB, available 5,7 GiB, swap dùng 0/4,0 GiB; `uptime` lúc 12:37:26 báo up 27 phút, load average 1,80/1,63/1,18. Output không tự xác nhận số node thực sự đang chạy.
- EDGE2 Gi0/0 `10.255.0.11/29`, Gi0/1 `10.255.0.21/30`, Lo0 `10.255.255.12/32`, Lo100 `198.51.100.12/32` đều up/up; connected RIB có T02 `10.255.0.8/29`, T04 `10.255.0.20/30` và loopback. Gi0/2–3 administratively down.
- EDGE2 source `.21` → ISP2 `.22` trên T04: **10/10**, min/avg/max 1/1/3 ms. EDGE2 source `.11` → EDGE1 `.10` trên T02 bridge: **10/10**, 1/1/3 ms.
- ISP2 BGP router-ID `10.255.255.102`, AS65102: neighbor EDGE2 `.21` AS65010 nhận 2 prefix; ISP1 `.33` AS65101 nhận 6; INET `.6` AS65200 nhận 2. Ba neighbor có InQ/OutQ 0 và `State/PfxRcd` dạng số.
- EDGE2 BGP router-ID `10.255.255.12`, AS65010: ban đầu chỉ eBGP ISP2 `.22` nhận 6 prefix. ISP2 BGP nhận Lo100 EDGE2 `.12/32` trực tiếp, AS_PATH `65010`. INET có hai path đến `.12/32`: qua ISP1 AS_PATH `65101 65102 65010` và qua ISP2 `65102 65010` là best, nên có route trả về trong BGP table; chưa có `show ip route 198.51.100.12` mới từ INET trong mẫu này.

## iBGP đã lên, policy chọn đường chưa đạt

- Sau cấu hình iBGP, EDGE1 nhận 6 prefix từ EDGE2 `.11` và 6 từ ISP1 `.18`; EDGE2 nhận 6 từ EDGE1 `.10` và 6 từ ISP2 `.22`. Mỗi Edge có 9 network/14 path và hai session Established (InQ/OutQ 0).
- Cả default `0.0.0.0/0` lẫn target `192.0.2.10/32` ở EDGE1 đều có hai path: từ EDGE2 qua `.11`, AS_PATH `65102 65200`, LP100, valid/internal; từ ISP1 `.18`, AS_PATH `65101 65200`, LP100, best/external. RIB EDGE1 cài cả hai prefix qua `.18`.
- EDGE2 cũng có hai path cho cả default/target: từ EDGE1 qua `.10`, AS_PATH `65101 65200`, LP100, valid/internal; từ ISP2 `.22`, AS_PATH `65102 65200`, LP100, best/external. RIB EDGE2 cài cả hai prefix qua `.22`, administrative distance 20.
- Do LP ngang nhau, snapshot này chọn path eBGP trực tiếp trên mỗi Edge. Chưa thể ghi đã chọn exit ISP1 trên EDGE2 hay đã áp LP200 thành công. Chỉ thấy hai candidate và `next-hop-self` có next-hop `.10/.11` phù hợp, chưa xác nhận route-map tồn tại/gắn đúng hoặc đã tái xử lý inbound route sau khi thay policy.

**Bằng chứng bổ sung S4-LP-01:** EDGE1 `show running-config | section router bgp` xác nhận neighbor `.18` có `route-map RM-ISP1-LP200 in` cùng prefix-list inbound; `show route-map RM-ISP1-LP200` có `set local-preference 200`. Người thực hành chạy `clear ip bgp 10.255.0.18 soft in` đúng neighbor. Output EDGE1 ngay sau lệnh còn thấy default LP100 (Refresh Epoch 1), nhưng target đã lên LP200 (Refresh Epoch 2) và còn một path best từ ISP1. Output EDGE2 tiếp theo cho **default và target đều có iBGP qua EDGE1 `.10` LP200, best**, cùng eBGP ISP2 `.22` LP100, valid/non-best. Đây là tiến triển đúng của policy; không suy snapshot EDGE1 default đã ổn định từ output tức thời trước khi refresh xong.

Việc EDGE1 chỉ còn **một** path target sau khi EDGE2 chọn đường iBGP có thể là trạng thái bình thường: BGP mặc định chỉ quảng bá best path; EDGE2 không còn gửi path ISP2 non-best trở lại EDGE1. Khi ISP1 mất, EDGE2 sẽ chọn path ISP2 rồi quảng bá lại; điều đó phải được kiểm ở bài failover sau, không được suy từ snapshot hiện tại. Gate buổi 4 yêu cầu LP/best/RIB đúng, **không yêu cầu hai path đồng thời ở cả hai Edge**.

## Snapshot ổn định và traffic sau policy

- EDGE1 default và target `192.0.2.10/32` đều **một path best** qua ISP1 `10.255.0.18`, AS_PATH `65101 65200`, **LP200**, Refresh Epoch 2. EDGE2 cả hai prefix có hai path: iBGP qua EDGE1 `.10`, AS_PATH `65101 65200`, **LP200 best**; eBGP trực tiếp ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best. S4-LP-01 không còn là lỗi chọn best path trong BGP table.
- Summary sau refresh: EDGE1 nhận **2** prefix từ iBGP EDGE2 `.11` và **6** từ eBGP ISP1 `.18`; EDGE2 nhận **8** từ iBGP EDGE1 `.10` và **6** từ eBGP ISP2 `.22`. Hai session trên mỗi Edge Established, InQ/OutQ 0. Số prefix iBGP bất đối xứng phù hợp việc EDGE2 không tiếp tục quảng bá path ISP2 chưa được chọn cho default/target; không suy tất cả prefix khác đã kiểm hết từ summary.
- EDGE2 source Lo100 `198.51.100.12` ping INET target `192.0.2.10` repeat 10: **10/10**, min/avg/max **2/2/3 ms**. Traceroute: hop 1 EDGE1 `.10`, hop 2 ISP1 `.18`, hop 3 INET `.2`; một probe `*` ở hop 3 nhưng hai probe khác reply và ping 10/10. Đây là bằng chứng data plane EDGE2 chọn exit ISP1 tại thời điểm thử.
- EDGE1 `advertised-routes` tới ISP1 `.18` đúng ba prefix: T02 `10.255.0.8/29`, Lo100 own `.11/32`, Lo100 EDGE2 `.12/32` học qua iBGP. EDGE2 tới ISP2 `.22` cũng đúng ba prefix đối xứng. Không có default/target trong hai bảng outbound; đây là kiểm prefix hiện tại, chưa phải negative canary test chủ động.
- Người thực hành xác nhận **đã `wr`** sau lượt kiểm. Theo bước đã yêu cầu, ba node thay đổi là EDGE1, EDGE2, ISP2; chưa có raw `[OK]` riêng, startup dump/ZIP mới hoặc phép thử restart/restore.

**Bằng chứng chốt:** EDGE1 source `198.51.100.11` ping INET target `192.0.2.10` repeat 10 đạt **10/10**, min/avg/max **2/2/3 ms** sau khi policy đổi. EDGE2 RIB cài default và target bằng `bgp 65010`, **distance 200**, type internal, next-hop **`10.255.0.10`**; route tag 65101. Như vậy BGP best path, RIB và phép đo traffic cùng chỉ ra exit ISP1. Output `EDGE1#ping 198.51.100.11 repeat 10` trước đó chỉ ping chính Loopback100 của EDGE1, đã được thay bằng mẫu source đúng và giữ trong lịch sử kiểm tra. Chưa thử failure/recovery hoặc restart/restore.

**Checkpoint artifact tiếp theo:** cần export CFG của EDGE1/EDGE2/**ISP1/ISP2** và ZIP lab dual-ISP trước bài failure/recovery buổi 6–7; ISP1 đã thêm peer EDGE1 ở buổi 3, ISP2 đã thêm peer EDGE2 ở buổi 4, nên CFG trong ZIP provider buổi 2 của hai ISP đều cũ. Chưa ghi artifact mới là đã có khi chưa nhận file.
