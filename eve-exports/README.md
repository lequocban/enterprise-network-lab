# Export HQ và cấu hình đi kèm

## Sprint 04 — ZIP cuối, 07/11/2026

**Restore đã PASS theo runbook** trên `LAB-NETWORK/SPRINT04-RESTORE-2026-11-07`, kèm bước lưu startup sau import và restart kiểm lại. [Biên bản restore](../docs/test-results/sprint-04-final-restore-2026-11-07.md), [báo cáo đóng kỹ thuật](../docs/sprint-reports/sprint-04-report.md). Trạng thái chưa restore trong đoạn kiểm tĩnh dưới đây là checkpoint trước phép thử.

[sprint-04-enterprise-final-2026-11-07.zip](sprint-04-enterprise-final-2026-11-07.zip), **14.082 byte**, SHA-256 `245C9B3C6ED7343C946BBB531DFB93151226C404F8ECFB9EA66496F1AA813D50`. CRC/XML đạt: 17 node, 21 network, 17 payload và toàn bộ node config=1/Exported. Cabling và tài nguyên/image giống baseline 26/10; khác biệt cấu hình chỉ có timestamp/comment và hai thiết lập console trên Edge. [17 cấu hình/manifest](../configs/sprint-04-final/README.md), [biên bản kiểm](../docs/test-results/sprint-04-final-export-inspection-2026-11-07.md). File nguồn tải tạm được giữ nguyên sau khi sao chép sang tên cuối. Chưa import/restore từ ZIP này.

## Sprint 04 — Underlay + scoped LP trước failure test, 26/10/2026

[sprint-04-branch-underlay-scoped-lp-2026-10-26.zip](sprint-04-branch-underlay-scoped-lp-2026-10-26.zip), **14.242 byte**, SHA-256 `9ADB40281E74332D2D1E520BEBCB7FC0342E85410960AB9012B1199084EB14EB`. CRC/XML đạt, 17 node/21 network và **17/17 payload config**; link/cấu hình HQ không đổi so với CP2. EDGE1 chứa scoped LP mới, hai BR có WAN/LAN/static default, hai PC Branch có IP/gateway. Xem [biên bản kiểm ZIP](../docs/test-results/sprint-04-underlay-export-2026-10-26.md) và [runtime buổi 5](../docs/test-results/sprint-04-branch-underlay-2026-10-26.md). Chưa thử import/restore từ ZIP này; giữ bản CP2 bên dưới làm lịch sử trước sửa policy.

## Sprint 04 — Baseline dual ISP trước Branch, file 25/10/2026

[sprint-04-dual-isp-bgp-baseline-2026-10-25.zip](sprint-04-dual-isp-bgp-baseline-2026-10-25.zip), **13.360 byte**, SHA-256 `3033048D93169EFEB52CAA892E44FF5E7B2A71B3B24B8EED8C6246596EA4187A`. CRC/XML đạt, 17 node/21 network, mapping link không đổi, **13 payload** gồm 8 HQ byte-identical, 5 IOSv Edge/ISP/INET. EDGE1/EDGE2/ISP1/ISP2 đã có CFG mới; INET không đổi. BR1/BR2 chưa có payload trong ZIP này vì đây là baseline trước cấu hình Branch. Xem [biên bản kiểm export](../docs/test-results/sprint-04-dual-isp-export-2026-10-26.md) và [cấu hình trích nguyên byte](../configs/README.md). Chưa thử restore; không ghi đè ZIP này sau tinh chỉnh policy.

## Sprint 04 — Checkpoint BGP provider, 24/10/2026

[sprint-04-provider-bgp-2026-10-24.zip](sprint-04-provider-bgp-2026-10-24.zip), **7.257 byte**, SHA-256 `AECB95334FE0CA05ABDABDD785374D10702B29E45CAA127CA67A3A9D0701BF6C`. CRC/XML đạt: 17 node, 21 network, cabling giống checkpoint trước và 11 payload config. Tám payload HQ không đổi; ba payload mới của ISP1/ISP2/INET-TEST đã giải mã và lưu nguyên byte ở [configs](../configs/README.md). Startup-configs ON theo metadata của ba node provider; `wr` đã được người thực hành xác nhận. Xem [biên bản BGP và kiểm artifact](../docs/test-results/sprint-04-provider-bgp-2026-10-24.md). Chưa thử restart/restore từ ZIP này.

## Sprint 04 — Checkpoint cabling WAN, 24/10/2026

[sprint-04-enterprise-wan-cabling-2026-10-24.zip](sprint-04-enterprise-wan-cabling-2026-10-24.zip), **4.580 byte**, SHA-256 `C407EC95D2E56D3040B92E2A8046E26E043F62805C7127277B41E5ED1698505A`. CRC/XML đạt; 17 node, 21 network, đầy đủ T02–T07/W08/W09/Branch LAN đúng cặp cổng, tên `ISP2` đúng; 11 dây HQ và 8 payload cấu hình HQ giống hệt seed. T02 có type `bridge`; bảy IOSv mới dùng đúng image/tài nguyên. Gate B đạt kiểm tĩnh cabling/inventory; đây chưa phải IP/BGP runtime hoặc restore sau mở rộng. Export đầu có lỗi tên `ISP12` đã được người thực hành thay bằng file mới cùng tên, không còn bytes trong repo; digest cũ chỉ ghi lịch sử trong [biên bản kiểm tra](../docs/test-results/sprint-04-wan-export-inspection-2026-10-24.md).

## Sprint 03 — Routing sandbox, 22/10/2026

[sprint-03-routing-2026-10-22-v2.zip](sprint-03-routing-2026-10-22-v2.zip), **8.454 byte**, SHA-256 `62D2E10F1F65DA9D52B2560BD2BCC5FF537AFDDB3FD4B523B9613D3EC1A9669E`. CRC/XML đạt; một UNL gồm tám node (5 IOSv, 3 VPCS), bảy link đúng cặp cổng, 35 text objects và tám payload cấu hình đã giải mã/đối chiếu các mốc chính. ISP node đã mang tên đúng `ISP1`. Xem [biên bản kiểm tra](../docs/test-results/sprint-03-export-inspection-2026-10-22.md) và [cấu hình export](../configs/routing-sandbox/sprint-03/final/README.md).

Thuộc tính node `config="0"` vẫn là Startup-configs OFF; không đồng nghĩa ZIP thiếu payload. Import/restore trên bản sao chưa thử. [ZIP đầu tiên](sprint-03-routing-2026-10-22.zip) chỉ có topology và nhãn ISP sai `IPS1`, giữ lại làm lịch sử.

### Backup HQ trước mở rộng WAN — 23/10/2026

[sprint-02-hq-before-wan-2026-10-22.zip](sprint-02-hq-before-wan-2026-10-22.zip), **3.736 byte**, SHA-256 `AAEE4445EEF1FC1B68F777FA1B2986E192B2457F662DB03CFC6B2850773C5F81`. ZIP có topology và 8/8 payload cấu hình giải mã được; CRC/XML hợp lệ. Mọi node vẫn `config="0"` (Startup-configs OFF). Restore chưa thử. Xem [biên bản](../docs/test-results/sprint-02-before-wan-export-2026-10-23.md).

### Bản khởi đầu Sprint 4 — 23/10/2026

[sprint-04-enterprise-seed-2026-10-23.zip](sprint-04-enterprise-seed-2026-10-23.zip), **3.789 byte**, SHA-256 `821B911842A7FCC2A5175ADA3FA567C3D9F6E3E9CC285D34988E23A3BF221925`. Tạo từ backup HQ trên, chỉ đổi tên lab XML và file UNL thành `Enterprise-Network-Integrated`; không đổi node, network, link hay Base64 config. Kiểm XML/ZIP đạt: 8 node, 11 network, 8 payload giải mã được. User đã import vào `LAB-NETWORK/test_import`, bật Startup-configs ON cho tám node và sau Stop/Wipe/Start xác nhận cả lab khôi phục thành công. CORE1 có output trực tiếp cho hostname/SVI/Po; các node khác và smoke Campus là xác nhận của người thực hành, chưa có raw output trong hồ sơ. Wipe cần thiết lần đầu bật startup config theo EVE CE Cookbook 6.3, mục 8.1.3 và 10.3. Giữ nguyên ZIP nguồn và lab HQ gốc. Xem [biên bản restore](../docs/test-results/sprint-04-seed-restore-2026-10-23.md).

**Tránh ghi đè khi import:** file bên trong ZIP là `Enterprise-Network-Integrated.unl`, khác `Enterprise-Network-HQ.unl`; tên ZIP bên ngoài không quyết định tên lab. Trước upload, tạo một thư mục con trống `LAB-NETWORK/SPRINT04-DEV`, mở đúng thư mục đó rồi import. Nếu đã có `Enterprise-Network-Integrated.unl` ở thư mục đích, không upload tiếp. EVE cũng có chức năng Clone Lab tạo bản sao với tên mới; đó là lựa chọn không cần import ZIP. Chưa có kiểm nghiệm thao tác import/clone trên EVE thật.

## Bàn giao Sprint 02 — 18/10/2026

[sprint-02-hq-2026-10-18.zip](sprint-02-hq-2026-10-18.zip), **1007 bytes**, chứa một `Enterprise-Network-HQ.unl` **4762 bytes**. SHA256: `c9f18e4b7a22079a3d391a136b31bf8fd19f6b0bb1fc29777550848e840358ce`.

[Kiểm tra export](../docs/test-results/sprint-02-export-inspection-2026-10-18.txt): CRC/XML hợp lệ, tám node, mười một network có đúng hai endpoint mỗi network; toàn bộ11 kết nối khớp cổng đã triển khai. Po1 có CORE1 e0/0↔CORE2 e0/0 và e0/3↔e0/3; Po11 có CORE1 e0/1↔ACCESS1 e0/0 và CORE1 e1/1↔ACCESS1 e1/0. Giữ nguyên các ZIP Sprint00/01.

ZIP **chỉ chứa topology/metadata**: tất cả node `config=0`, không có payload cấu hình/VPCS startup hoặc image. Đi kèm [CORE1](../configs/core/sprint-02/CORE1.final.running.cfg), [CORE2](../configs/core/sprint-02/CORE2.final.running.cfg), [ACCESS1](../configs/access/sprint-02/ACCESS1.final.running.cfg), [ACCESS2](../configs/access/sprint-02/ACCESS2.final.running.cfg) và IP endpoint trong [báo cáo bàn giao](../docs/sprint-reports/sprint-02-report.md). Người thực hành đã xác nhận wr; startup cuối chưa có dump đối chiếu độc lập. Import/restore chưa thử; ZIP hợp lệ không phải bằng chứng khôi phục thành công. Lỗi chuyển member Po1 vẫn mở.

## Baseline Sprint 01 dùng tiếp

[sprint-01-hq-2026-10-11.zip](sprint-01-hq-2026-10-11.zip), **947 bytes**, chứa một `Enterprise-Network-HQ.unl` **4241 bytes**.

**SHA256 ZIP:** `5B84F2E09DBEC6101E1D7AE925A42DAC6A77575C37131726C4F4715EB94F3D83`.

[Kiểm tra 11/10/2026](../docs/test-results/sprint-01-export-inspection-2026-10-11.txt): CRC/XML PASS, tám node/chín link, tên/ID duy nhất và toàn bộ cổng HQ khớp physical topology. Bốn IOL dùng `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`; Core RAM512, Access256 theo metadata.

Tất cả node `config="0"`; không có file/payload cấu hình nhúng. Export chỉ chứa topology/metadata, phải đi kèm [bốn running cuối Sprint 01 và lịch sử](../configs/README.md), [review cấu hình](../docs/test-results/sprint-01-config-review-2026-10-11.md) và [validation](../docs/test-results/sprint-01-validation.md). Chưa có startup riêng để đối chiếu; ZIP không chứng minh persistence/restore. [Báo cáo Sprint 01](../docs/sprint-reports/sprint-01-report.md) ghi follow-ups, không yêu cầu xuất lại chỉ vì ZIP không nhúng config.

### Tái dựng Sprint 01 — chưa kiểm thử

1. Import vào lab riêng, giữ lab hiện có; kiểm tra tám node/chín link và đúng image/tài nguyên trước chạy.
2. Dùng cấu hình Sprint 01 thật đi kèm để tái tạo bảy VLAN và interface; VTP transparent. Không dùng cấu hình VLAN10-only Sprint 00 để mô tả baseline hiện tại.
3. Giữ `no ip cef` hai Core theo ngoại lệ đã chọn; trunk native99/allowed bảy VLAN, STP root priorities và SVI/VIP theo snapshot. Cấu hình VPCS theo địa chỉ/gateway đích vì ZIP không nhúng startup VPCS.
4. Kiểm tra trạng thái L2/L3/HSRP và traffic, lưu config trên lab bản sao sau review. Chỉ đánh dấu restore PASS khi có kết quả thực tế; chưa có test này trong hồ sơ Sprint 01.

## Baseline lịch sử Sprint 00

**Bản lưu Sprint 00:** [sprint-00-hq-2026-10-10.zip](sprint-00-hq-2026-10-10.zip), 952 bytes, chứa một `Enterprise-Network-HQ.unl` 4241 bytes. Các thông tin dưới mô tả baseline lịch sử, không phải cấu hình Campus hiện tại.

**SHA256:** `CAD0EE62F6A1213B08147D5BB7471EBFB873D0B5F28FD0DB047276B6395F539F`.

[Kiểm tra cuối ngày 10/10/2026](../docs/test-results/sprint-00-export-final-inspection-2026-10-10.txt): CRC/XML PASS, đúng tám node/chín link, tên/ID không trùng, không có node PC-GUEST dư. Cả chín kết nối khớp physical topology. Đây là kiểm tra artifact tại local; agent chưa import hoặc chạy thiết bị.

### Nội dung và cấu hình Sprint 00

- Bốn IOL switch và bốn VPCS; image IOL `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`.
- Thuộc tính RAM Core 512, Access 256; không có thuộc tính node vCPU.
- Các node `config="0"`, không có payload hoặc file cấu hình nhúng. ZIP chứa topology/metadata; **không phải backup đầy đủ startup-config hoặc VLAN database**.
- Dùng [12 snapshot config](../configs/README.md) và [evidence VLAN10](../docs/test-results/sprint-00-validation.md) đi kèm. CORE1/ACCESS2 khớp running/startup, CORE2/ACCESS1 chỉ running có VTY `login`; không coi hai loại bản là tương đương.
- Các biên bản export trước là lịch sử; hash cũ không xác định file hiện tại, các ZIP cũ/v2 không còn trong thư mục. Không tái tạo bản cũ từ file mới để làm bằng chứng.

### Tái dựng Sprint 00 — chưa kiểm thử import/reload

1. Import ZIP vào lab riêng, giữ lab đang dùng; đối chiếu đúng tám node/chín link và image trước chạy.
2. Cung cấp image đúng tên ngoài Git; kiểm tra RAM/tài nguyên trên EVE đích.
3. Áp dụng config/VLAN theo artifact: hai Core có VLAN10 MGMT; bản running sau test Core đã đối chiếu với baseline. Snapshot không thay backup đầy đủ VLAN database. Chọn chính sách VTY rõ ràng trước lưu startup.
4. Kiểm tra SVI, trunk/native1/allowed10, root/port role VLAN10, HSRP peer/VIP và ping; ghi actual và khôi phục nếu khác. Native99/toàn bộ Campus thuộc Sprint 1.
5. Chỉ ghi restore PASS khi có test thật; hiện chưa có kết quả import/reload/restore hoặc xác nhận cấu hình tự phục hồi từ ZIP.

Chuyển role/khôi phục SVI trên lab gốc đã PASS, không đồng nghĩa import ZIP đã thành công. Việc thử restore được bàn giao tại [closeout](../docs/sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện).
