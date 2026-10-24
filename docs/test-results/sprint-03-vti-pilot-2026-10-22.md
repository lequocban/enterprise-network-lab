# Sprint 3 — INT-03 VTI pilot

**Ngày:** 22/10/2026.  
**Lab:** `Enterprise-Network-INT03-VTI-Pilot`, hai IOSv.  
**Trạng thái:** SVTI/IKEv2/IPsec và OSPF overlay đạt; ping Loopback hai chiều 5/5; người thực hành xác nhận đã `wr` cả hai router. Failure/recovery OSPF đã đạt; MTU và import/restore còn chờ.

## Underlay

- `VTI-HQ Gi0/0`: `172.31.255.1/30`, up/up; Lo0 `172.31.0.1/32`.
- `VTI-BR1 Gi0/0`: `172.31.255.2/30`, up/up; Lo0 `172.31.1.1/32`.
- Ping underlay hai chiều: 5/5 theo output người thực hành. Sau khi cấu hình OSPF, người thực hành xác nhận đã `wr` trên cả VTI-HQ và VTI-BR1.

## SVTI / IKEv2

- Tunnel0 hai đầu: subnet `172.31.255.4/30` (HQ `.5`, BR1 `.6`), up/up theo `show ip interface brief` tại HQ.
- HQ ping tunnel peer `.6`: 5/5, RTT 3/4/5 ms.
- `show crypto ikev2 sa`: peer `.1`↔`.2`, trạng thái `READY`, AES-CBC 256, SHA256, DH group 14, PSK.
- `show crypto ipsec sa`: active ESP SAs, transform `esp-256-aes esp-sha256-hmac`; 5 packets encaps/encrypt/digest và 5 decaps/decrypt/verify; send/receive errors 0.

**Kết luận:** đúng image IOSv của pilot chấp nhận và chạy static IPsec VTI + IKEv2. Tunnel traffic thực được mã hóa/giải mã. MTU test, failure/recovery và import/restore chưa chạy. Khóa xuất hiện trong cấu hình pilot chỉ được dùng cho lab thử; không tái sử dụng ở Enterprise.

## OSPF overlay — actual mới

- Process 10, Area10; Tunnel0 point-to-point, Lo0 loopback.
- HQ neighbor RID `172.31.1.1` ở `FULL/-` qua `172.31.255.6`; BR1 neighbor RID `172.31.0.1` ở `FULL/-` qua `172.31.255.5`.
- Hai đầu học Loopback peer qua OSPF: HQ `O 172.31.1.1/32 [110/1001]` qua Tunnel0; BR1 `O 172.31.0.1/32 [110/1001]` qua Tunnel0.
- Cost Tunnel0 là 1000 trên cả hai phía; đây là actual output, chưa chỉnh cost. Chưa xác định vì sao image tính ra mức này; cần `show interface Tunnel0` nếu cần phân tích bandwidth. Route metric 1001 là actual.
- Ping HQ `172.31.0.1` → BR1 `172.31.1.1` source Lo0 đạt 5/5 (RTT 4/4/4 ms); ping ngược đạt 5/5 (RTT 3/4/5 ms). Người thực hành xác nhận `wr` cả hai router sau cấu hình OSPF.

## Bước kế tiếp

**Failure/recovery — đạt theo output:** ở trạng thái mất kết nối, `show ip ospf neighbor` không có neighbor và route `172.31.1.1/32` không có trong bảng. Sau đó neighbor trở lại `FULL`, route OSPF được cài lại qua Tunnel0 và ping Loopback hai chiều đạt 5/5. IKEv2 SA vẫn hiển thị `READY` ở snapshot khi route OSPF mất; do đó xác nhận được OSPF withdraw/reconverge, nhưng không kết luận IKE SA đã bị xóa trong pha lỗi. Cost 1000 hiện không gây chọn sai đường vì pilot chỉ có một tunnel; trước khi tích hợp nhiều đường, phải chốt cost/bandwidth theo mục tiêu định tuyến.
