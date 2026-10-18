# Cấu hình thiết bị

Lưu cấu hình **lấy từ thiết bị đã chạy lab**, không tạo config giả từ bảng IP để làm bằng chứng.

## Baseline Sprint 01 đã lưu

Đã nhận và cập nhật running-config bốn switch ngày 11/10/2026: [CORE1](core/sprint-01/CORE1.cfg), [CORE2](core/sprint-01/CORE2.cfg), [ACCESS1](access/sprint-01/ACCESS1.cfg), [ACCESS2](access/sprint-01/ACCESS2.cfg). Cả bốn đã có baseline VTY `transport input none`; cổng test CORE1 Ethernet1/0 đã shutdown. Bản trước VTY được giữ với hậu tố `.pre-vty.cfg`, CORE1 có thêm `.pre-test-port-shutdown.cfg`. Tại mốc chốt 11/10, CORE2 và hai Access có output lưu thành công; chưa có startup-config để đối chiếu, chưa xác nhận lưu CORE1 sau shutdown cổng test. Nguồn, chuẩn hóa và giới hạn tại thời điểm đó nằm trong [biên bản review](../docs/test-results/sprint-01-config-review-2026-10-11.md); bằng chứng startup mới được bổ sung riêng bên dưới.

Bộ Sprint 01 gồm **chín snapshot: bốn running cuối và năm bản lịch sử**; không có bản startup giả từ running. [Báo cáo](../docs/sprint-reports/sprint-01-report.md) đã hoàn thiện với follow-up startup/restore; đi kèm [export Sprint 01](../eve-exports/sprint-01-hq-2026-10-11.zip) chỉ chứa topology. Không cần thu lại running để hoàn thiện báo cáo.

**Cập nhật 12/10/2026 — đủ startup bốn switch:** đã nhận [CORE1](core/sprint-02/CORE1.pre-lacp.startup.cfg), [CORE2](core/sprint-02/CORE2.pre-lacp.startup.cfg), [ACCESS1](access/sprint-02/ACCESS1.pre-lacp.startup.cfg) và [ACCESS2](access/sprint-02/ACCESS2.pre-lacp.startup.cfg) pre-LACP startup thật, khớp lần lượt 112/101/72/72 dòng lệnh của running cuối Sprint 01 sau loại comment/dòng trắng. CORE1 e1/0 shutdown đã có trong startup. [Review và runtime mới](../docs/test-results/sprint-02-preflight-review-2026-10-12.md) xác nhận sáu nhóm HSRP, trunk và VLAN/access port/SVI quản trị/STP VLAN20 hai Access đúng baseline; không cần `wr` lặp chỉ để bổ sung hồ sơ. Traffic VPCS đã có tám lượt 5/5; đã có VLAN brief hai Core/chi tiết tám cổng member; IP bốn PC và capability LACP/duplex đã xác nhận; buổi 1 đã đóng theo quyết định; restore baseline trước LACP bỏ qua/chưa chạy, save VPCS chưa có xác nhận, duplex thực tế kiểm tra khi triển khai. Bốn running Sprint 01 giữ nguyên và tiếp tục được dùng lại; chỉ bổ sung running nếu cấu hình đã thay đổi.

## Tổ chức

**Bàn giao Sprint02 — nguồn mới ngày18/10/2026:** đã lưu [CORE1 final running](core/sprint-02/CORE1.final.running.cfg), [CORE2 final running](core/sprint-02/CORE2.final.running.cfg) và [ACCESS2 final running](access/sprint-02/ACCESS2.final.running.cfg) từ ba full `show run` người thực hành gửi trong chat. Output báo lần lượt 3808/3035/1715 bytes; last change trên thiết bị lần lượt 15:23:32/15:24:18/13:50:38 UTC Sun Oct18/2026. File bỏ prompt, header console, dòng trắng/comment phân cách dư; giữ nguyên thứ tự, nội dung và thụt dòng lệnh cùng comment last change. Vì vậy kích thước file không bằng byte count IOS. Đối chiếu bỏ comment/dòng trắng: CORE1 khớp142 lệnh với checkpoint Po1+Po11; CORE2 khớp116 lệnh với checkpoint Po1; ACCESS2 khớp72 lệnh với running cuối Sprint01. Giữ nguyên mọi snapshot lịch sử. Người thực hành xác nhận “đã wr hết rồi”; chưa nhận startup dump mới nên không tạo `.final.startup.cfg` bằng cách chép running hoặc ghi đã đối chiếu startup cuối. Bổ sung cùng ngày: đã nhận full `show run` ACCESS1 (2210 bytes, last change13:50:38 UTC Sun Oct18/2026), lưu [ACCESS1 final running](access/sprint-02/ACCESS1.final.running.cfg), khớp87 dòng lệnh với checkpoint Po11 ngày16/10. Bộ final running hiện đủ bốn switch; không đổi tên hay ghi đè bản lịch sử. Snapshot final không đồng nghĩa lỗi chuyển dự phòng Po1 đã được sửa; xem [hồ sơ lỗi](../docs/test-results/sprint-02-po1-member-check-2026-10-18.md).

**Checkpoint Po1 hoàn thiện nguồn16/10:** đủ [CORE1 running](core/sprint-02/CORE1.po1-only.running.cfg)/[startup](core/sprint-02/CORE1.po1-only.startup.cfg)127 dòng lệnh và [CORE2 running](core/sprint-02/CORE2.po1-only.running.cfg)/[startup](core/sprint-02/CORE2.po1-only.startup.cfg)116 dòng; từng cặp khớp sau bỏ comment/dòng trắng. Hai file bổ sung lấy từ full nguồn12/10, chỉ thay header bị lẫn bằng section xác nhận16/10; raw/metadata cũ giữ nguyên, không coi là full dump mới16/10. [Provenance và review](../docs/test-results/sprint-02-po1-progress-2026-10-12.md). Không cần save hoặc xác nhận lại hai đoạn này. Các pre-lacp giữ riêng; S2-FWD-01 phục hồi, S2-IOL-01 giữ nguyên. [Preflight Po11](../docs/test-results/sprint-02-po11-preflight-review-2026-10-16.md) lưu lịch sử chuẩn bị; trạng thái Po11 mới hơn ở checkpoint dưới đây, HA chưa nghiệm thu.

**Checkpoint Po11 đã lưu16/10:** [review nguồn/runtime](../docs/test-results/sprint-02-po11-progress-2026-10-16.md) có SU/LACP/hai P đúng peer hai đầu, trunk/STP VLAN20/inconsistent0/HSRP CORE1/VIP–ACC5/5. Đã trích độc lập bốn snapshot đầy đủ: CORE1 [running](core/sprint-02/CORE1.po1-po11.running.cfg)/[startup](core/sprint-02/CORE1.po1-po11.startup.cfg) khớp142 dòng lệnh, ACCESS1 [running](access/sprint-02/ACCESS1.po11.running.cfg)/[startup](access/sprint-02/ACCESS1.po11.startup.cfg) khớp87 dòng sau bỏ comment/dòng trắng. Startup chứa đúng cấu hình mới; không cần wr hoặc thu lại. Không có dòng wr/OK trong nguồn mới nên không tạo log save giả. Po1 snapshots cũ giữ riêng; CORE2 dùng checkpoint Po1, ACCESS2 dùng baseline đã có vì bài Po11 không sửa hai node đó. HA/export mới còn chờ.

- `core/sprint-<NN>/`: CORE1, CORE2.
- `access/sprint-<NN>/`: ACCESS1, ACCESS2.
- `edge/`, `branch/`, `firewall/`: bổ sung thư mục sprint khi triển khai nhóm tương ứng.
- Mỗi file dùng tên node và `.cfg`; ghi nguồn running/startup trong báo cáo kiểm thử.

README này giữ thư mục `configs/` khi clone. Thư mục con rỗng không được Git lưu; tạo lại khi có config thật.

## Baseline cần bổ sung

- [x] CORE1 Sprint 00: [running-config](core/sprint-00/CORE1.cfg) và [startup-config](core/sprint-00/CORE1.startup.cfg), người thực hành cung cấp ngày 10/10/2026; lệnh cấu hình khớp nhau, metadata khác. Chưa kiểm thử khôi phục.
- [x] CORE2 Sprint 00: [running-config](core/sprint-00/CORE2.cfg) và [startup-config](core/sprint-00/CORE2.startup.cfg), người thực hành cung cấp ngày 10/10/2026. Đã lưu và đối chiếu; running có `login` dưới VTY 0–4, startup không có. Chưa đồng bộ hoặc kiểm thử khôi phục.
- [x] ACCESS1 Sprint 00: [running-config](access/sprint-00/ACCESS1.cfg) và [startup-config](access/sprint-00/ACCESS1.startup.cfg); running có VTY `login`, startup không có, các lệnh còn lại khớp.
- [x] ACCESS2 Sprint 00: [running-config](access/sprint-00/ACCESS2.cfg) và [startup-config](access/sprint-00/ACCESS2.startup.cfg); lệnh cấu hình và thời gian last change khớp.
- [x] Đã lưu running/startup bốn switch HQ dùng trong Sprint 0; chưa kiểm thử khôi phục hoặc backup đầy đủ VLAN database.
- [x] Image/version bốn switch và các cặp startup/running được đối chiếu trong validation; CORE2 và ACCESS1 chưa đồng bộ hoàn toàn.
- [x] Đã kiểm tra tám snapshot hiện có: không thấy mật khẩu, key hoặc PSK; cần review lại khi bổ sung cấu hình quản trị.
- [x] Có liên kết từ [Validation Sprint 0](../docs/test-results/sprint-00-validation.md) tới config thật.
- [x] Hai Core có thêm bốn snapshot sau test: CORE1 [running](core/sprint-00/CORE1.post-switchover.cfg)/[startup](core/sprint-00/CORE1.post-switchover.startup.cfg), CORE2 [running](core/sprint-00/CORE2.post-switchover.cfg)/[startup](core/sprint-00/CORE2.post-switchover.startup.cfg). Từng bản khớp lệnh với bản trước test tương ứng; CORE2 vẫn khác VTY giữa running/startup. Tổng hiện có 12 snapshot, đã giữ nguyên tám bản đầu.

Nguồn và kết quả đối chiếu tại [Validation Sprint 0](../docs/test-results/sprint-00-validation.md). Export HQ sạch đã kiểm tra, đi kèm 12 snapshot. Sprint 0 đóng với follow-up: VTY CORE2/ACCESS1 tiếp nhận Sprint 1; khôi phục SVI vận hành đã kiểm tra, import/reload/restore toàn lab vẫn chưa thử, theo [closeout](../docs/sprint-reports/sprint-00-closeout.md).
