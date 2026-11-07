# Sprint 04 — Kiểm ZIP baseline dual ISP trước Branch

**Ngày kiểm:** 26/10/2026, Asia/Saigon. File mang tên checkpoint `2026-10-25`; metadata local ghi LastWriteTime 25/10/2026.  
**Nguồn:** [ZIP baseline](../../eve-exports/sprint-04-dual-isp-bgp-baseline-2026-10-25.zip) có sẵn trong workspace; agent kiểm trực tiếp bytes ZIP/XML, không điều khiển EVE.  
**Mục đích:** CP2 trước khi thêm cấu hình Branch và trước các bài failure/recovery buổi 6–7.

ZIP **13.360 byte**, SHA-256 `3033048D93169EFEB52CAA892E44FF5E7B2A71B3B24B8EED8C6246596EA4187A`. CRC đạt; có đúng một `Enterprise-Network-Integrated.unl`, XML đọc được, **17 node/21 network**, mỗi network có hai endpoint và toàn bộ mapping link giống [ZIP provider buổi 2](sprint-04-provider-bgp-2026-10-24.md). Không suy từ CRC/XML rằng import hoặc boot lại đã thành công.

Có **13 payload cấu hình** giải mã Base64 được: 8 HQ giống byte-for-byte với ZIP provider, 5 IOSv gồm EDGE1, EDGE2, ISP1, ISP2, INET-TEST. Metadata Startup-configs ON (`config="1"`) trên cả 5 IOSv này; BR1-R1/BR2-R1 vẫn OFF và chưa có payload trong ZIP, phù hợp baseline được xuất **trước** thay đổi Branch. INET-TEST 4.188 byte giống byte-for-byte với ZIP provider. Bốn payload Edge/ISP đã đổi được trích nguyên byte:

- [EDGE1](../../configs/edge/sprint-04/EDGE1.dual-isp.exported.cfg), 4.836 byte, SHA-256 `DF393E916CF6DC49282DAE1633DE3F69DBDF03E08039BA4B2671A3A707F62840`.
- [EDGE2](../../configs/edge/sprint-04/EDGE2.dual-isp.exported.cfg), 4.836 byte, SHA-256 `EE2BF7B934AFD3785929810991E44962002FEC3268DBCCFEEA730E16488A07A4`.
- [ISP1](../../configs/isp/sprint-04/ISP1.dual-isp.exported.cfg), 5.692 byte, SHA-256 `02B0D48F810E6E0BE9589154E28D537906AAC2F1DE4D0A1CA59D6C10CF653520`.
- [ISP2](../../configs/isp/sprint-04/ISP2.dual-isp.exported.cfg), 5.692 byte, SHA-256 `F86B6434A51EDB0B4B15066F74A29AB91130163860714BA95AFC7B9144CECC4A`.

Đối chiếu mốc cấu hình: EDGE1/EDGE2 có AS65010, peer iBGP, route-map LP200/LP100; ISP1/ISP2 có peer Company AS65010 tương ứng. Không thấy `no ip cef` trên năm IOSv. Cấu hình EDGE1 export có route-map `RM-ISP1-LP200` **không có match clause**, nên LP200 áp cho mọi prefix nhận từ ISP1; kết quả runtime BR2 ngày26/10 cho thấy T06 chọn đường vòng qua EDGE1. [Biên bản Branch](sprint-04-branch-underlay-2026-10-26.md) ghi giới hạn này.

**Mức bằng chứng:** `wr` Edge/ISP đã được người thực hành xác nhận ở các buổi trước; ZIP chứa CFG, không có dump startup độc lập hoặc thử restore. Giữ ZIP này nguyên byte làm mốc trước khi tinh chỉnh policy và trước Branch; không ghi đè bằng export sau thay đổi.
