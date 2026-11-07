# Sprint 04, buổi 2 — BGP provider, checkpoint runtime

**Ngày nhận output:** 24/10/2026, Asia/Saigon; không có timestamp tuyệt đối trên từng console router.  
**Nguồn:** `show ip bgp summary`, BGP table, RIB BGP, advertised-routes sáu đầu neighbor và hai mẫu ping do người thực hành gửi từ ISP1/ISP2/INET-TEST. Agent đọc output, không điều khiển trực tiếp router.  
**Story/checkpoint:** NET-401, S4-S02-B, sau [chặng IP/ping](sprint-04-provider-ip-2026-10-24.md).  
**Trạng thái:** **ĐẠT checkpoint buổi 2** về IP, eBGP provider, route/traffic và artifact cấu hình. Snapshot bổ sung ISP2 đã xác nhận alternate path; ZIP mới đã kiểm tĩnh. INT-04 chưa đóng.

## Session và route thực tế

- ISP1 router-ID `10.255.255.101`, AS65101: neighbor `10.255.0.34` AS65102 nhận **4** prefix, uptime 00:00:27; `203.0.113.2` AS65200 nhận **2**, uptime 00:01:27. BGP có 6 network/8 path.
- ISP2 router-ID `10.255.255.102`, AS65102: neighbor `10.255.0.33` AS65101 nhận **2** prefix, uptime 00:00:37; `203.0.113.6` AS65200 nhận **2**, uptime 00:00:37. BGP có 6 network/6 path.
- INET-TEST router-ID `192.0.2.10`, AS65200: nhận **4** prefix từ mỗi ISP (`203.0.113.1` và `.5`); BGP có 6 network/10 path.

Cả sáu đầu neighbor có State/PfxRcd là số và InQ/OutQ 0: ba quan hệ eBGP đang Established. Các mẫu không cùng thời điểm; không suy timing hội tụ từ uptime của chúng.

ISP1 học default và target `192.0.2.10/32` bằng hai candidate: next-hop `203.0.113.2`, AS_PATH `65200` là best; next-hop `10.255.0.34`, AS_PATH `65102 65200` là valid/non-best. RIB cài `B* 0.0.0.0/0` và target qua `203.0.113.2`; hai WAN ISP2 `10.255.0.20/30`, `.28/30` đi `10.255.0.34`.

Trong snapshot ban đầu, ISP2 có default và target best qua `203.0.113.6`, AS_PATH `65200`; RIB khớp. Hai WAN ISP1 `10.255.0.16/30`, `.24/30` học qua `10.255.0.33`. Snapshot đó mới có hai prefix nhận từ ISP1, nên tại thời điểm ấy chưa đủ bằng chứng về alternate default/target.

**Snapshot bổ sung ISP2:** neighbor ISP1 `10.255.0.33` đã nhận **4** prefix, neighbor INET-TEST `203.0.113.6` nhận **2**; BGP có 6 network/8 path. `show ip bgp 0.0.0.0` và `show ip bgp 192.0.2.10` đều có **2 path hợp lệ**: trực tiếp qua `203.0.113.6`, AS_PATH `65200` là best; qua ISP1 `10.255.0.33`, AS_PATH `65101 65200` là alternate. Prefix-list `PL-FROM-PEER` có đúng 7 permit dự kiến, và phần `router bgp 65102` gắn inbound/outbound prefix-list vào cả hai neighbor. Như vậy thiếu path ở snapshot trước đã được giải quyết bằng bằng chứng mới; không kết luận nguyên nhân cụ thể chỉ từ các mẫu console này.

INET-TEST originate default và Lo100 /32 (next-hop 0.0.0.0, weight 32768 trong BGP table). Bốn WAN khách hàng có mỗi prefix hai path; best tới T03/T05 qua `203.0.113.1`, tới T04/T06 qua `203.0.113.5`, RIB khớp. Output bổ sung `show ip route 0.0.0.0` xác nhận static default distance 1 qua `Null0`, được BGP AS65200 quảng bá; đây là backing route cho lệnh `network 0.0.0.0`.

## Traffic và quảng bá

- ISP1 source `203.0.113.1` → `192.0.2.10`, repeat 10: **10/10**, min/avg/max **1/2/3 ms**.
- ISP2 source `203.0.113.5` → `192.0.2.10`, repeat 10: **10/10**, **1/1/3 ms**.
- Mỗi ISP → INET-TEST: đúng bốn WAN /30 T03–T06, không có default hoặc target trong advertised-routes đó.
- ISP1 → ISP2: default, target, T03 `.16/30`, T05 `.24/30`, tổng **4** prefix.
- ISP2 → ISP1: default, target, T04 `.20/30`, T06 `.28/30`, tổng **4** prefix.
- INET-TEST → mỗi ISP: đúng default và target, tổng **2** prefix.

Các bảng route/prefix xuất đã gửi không có LAN HQ/Branch/DMZ hoặc Lo0 ISP. Đây là kiểm prefix hiện tại, chưa phải negative test với route cấm được chủ động originate. Prefix Company/T02 chưa originate khi Edge chưa triển khai; việc chúng chưa xuất hiện là expected.

## Artifact và mức xác nhận

Người thực hành xác nhận đã `wr` cả ba router và export CFG/lab. [ZIP provider](../../eve-exports/sprint-04-provider-bgp-2026-10-24.zip) có **7.257 byte**, SHA-256 `AECB95334FE0CA05ABDABDD785374D10702B29E45CAA127CA67A3A9D0701BF6C`. Kiểm CRC/XML đạt: một `Enterprise-Network-Integrated.unl`, **17 node/21 network**, cabling không đổi so với ZIP trước và **11 payload config**. Tám payload HQ giống hệt checkpoint cabling. Ba node provider có startup-configs ON trong metadata và payload giải mã được; cấu hình IP, BGP, prefix-list và static Null0 khớp hướng dẫn. Không thấy `no ip cef` trên ba IOSv.

Các file trích **nguyên byte từ payload export**: [ISP1](../../configs/isp/sprint-04/ISP1.exported.cfg) 5.053 byte, SHA-256 `3BE89FD1A4D6210226769CBE275EAA837D9B61086FD07DD212933EBF6160D2C9`; [ISP2](../../configs/isp/sprint-04/ISP2.exported.cfg) 5.053 byte, SHA-256 `9D96FEC330E850EA9AA761095BAEAFE0D3B04A5E8E63BA2C0709D2E1F4CF7245`; [INET-TEST](../../configs/internet-test/sprint-04/INET-TEST.exported.cfg) 4.188 byte, SHA-256 `311CCBA1F2E2391672C1DBC8BC97CAE30B0049BAE44822C1E3ED5B073A7563B2`. Nguồn là CFG trong ZIP, chưa có dump startup độc lập hoặc phép thử restart/restore; metadata ON và `wr` là các mức bằng chứng riêng.

**Giới hạn còn lại:** chưa làm negative test bằng route cố ý bị cấm, failover/recovery, restart/restore, Edge/iBGP/Local Preference hoặc Internet client HQ. Những nội dung đó không thuộc checkpoint provider buổi 2; không suy toàn bộ NET-401–404/INT-04 đã nghiệm thu.
