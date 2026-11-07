# Sprint 04 — ZIP underlay và scoped LP trước failure test

**Ngày kiểm:** 26/10/2026, Asia/Saigon.  
**Nguồn:** [ZIP checkpoint](../../eve-exports/sprint-04-branch-underlay-scoped-lp-2026-10-26.zip) trong workspace; agent kiểm bytes ZIP/XML, không điều khiển EVE.  
**Mục đích:** baseline hiện tại sau buổi 5, trước bài mất ISP/uplink buổi 6–7.

ZIP **14.242 byte**, SHA-256 `9ADB40281E74332D2D1E520BEBCB7FC0342E85410960AB9012B1199084EB14EB`. CRC đạt; một `Enterprise-Network-Integrated.unl` đọc được, **17 node/21 network**, tất cả network hai endpoint và mapping link giống [ZIP CP2](sprint-04-dual-isp-export-2026-10-26.md). Có **17/17 payload cấu hình** giải mã Base64 được, toàn bộ node metadata Startup-configs ON (`config="1"`). Tám payload HQ giống byte-for-byte CP2; EDGE2, ISP1, ISP2, INET-TEST cũng giống byte-for-byte CP2. Không Wipe hay sửa lab được suy từ phép kiểm tĩnh này.

Năm payload mới so với CP2 đã được trích nguyên byte:

- [EDGE1 scoped LP](../../configs/edge/sprint-04/EDGE1.scoped-lp.exported.cfg), 5.048 byte, SHA-256 `B9349AB2FD162D8DFE497D45AE32C9F57A2F87954538C9D46725AADA7F3A1350`. Có `PL-UPSTREAM` exact default/target; route-map `RM-ISP1-LP200` seq10 match `PL-UPSTREAM` set LP200, seq20 set LP100.
- [BR1-R1](../../configs/branch/sprint-04/BR1-R1.exported.cfg), 3.189 byte, SHA-256 `95C8474DF5FCB0447FF596175455B888B29A953F9BBB33D978F9F3F52A9972AF`. Có T05 `.25/30`, LAN `10.20.10.1/24`, static default qua `.26`.
- [BR2-R1](../../configs/branch/sprint-04/BR2-R1.exported.cfg), 3.189 byte, SHA-256 `E870685D4AE4F8A92A1EA82F6D434576BF36D519944C5F9D4E953F281BFA34BE`. Có T06 `.29/30`, LAN `10.30.10.1/24`, static default qua `.30`.
- [PC-BR1](../../configs/vpcs/sprint-04/PC-BR1.exported.cfg), 47 byte, SHA-256 `E1512AF61C088C4D2A5CBFEA0840BC4D76FFE5E67E127591979C15743DF1A623` (IP `.20/24`, gateway `10.20.10.1`).
- [PC-BR2](../../configs/vpcs/sprint-04/PC-BR2.exported.cfg), 47 byte, SHA-256 `8527139ECAD6C5D0902E5D8332621902F7A5E8B84972ECCB14928312E341B929` (IP `.20/24`, gateway `10.30.10.1`).

[Runtime buổi 5](sprint-04-branch-underlay-2026-10-26.md) đã kiểm WAN/PC và scoped LP bằng CLI/ping; ZIP xác nhận cấu hình tương ứng đã nằm trong artifact. Có `wr [OK]` BR1/BR2/EDGE1 sau thay đổi và `save` hai PC, nhưng chưa có dump startup độc lập. CRC/payload không chứng minh ZIP đã restore được; test stop/start và import trên bản sao thuộc checkpoint sau. Giữ [ZIP CP2](sprint-04-dual-isp-export-2026-10-26.md) nguyên byte làm baseline trước thay đổi.
