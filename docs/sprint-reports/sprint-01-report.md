# Báo cáo Sprint 1 — Campus Switching

**Dự án:** Enterprise Network Design & Implementation (EVE-NG)  
**Người thực hiện lab:** lequocban  
**Ngày hoàn thiện báo cáo:** 11/10/2026, Asia/Saigon  
**Phạm vi:** EPIC-NET-01, NET-101–104  
**Trạng thái:** **HOÀN THÀNH THỰC HÀNH CÓ FOLLOW-UPS** — chốt phần thực hành để chuyển bước; không đồng nghĩa toàn bộ 13 ca PASS hoặc đạt DONE đầy đủ theo Definition of Done chung.  
**Mốc repository:** hồ sơ Sprint01 được lưu thành commit local riêng trước commit Sprint02 trên nhánh `lab/sprint-02-lacp-hsrp`; chưa push, tạo PR hoặc merge. Việc version hóa không nâng mức nghiệm thu thực hành.

## 1. Kết quả chính

HQ đã chuyển từ baseline VLAN 10 của Sprint 0 sang Campus gồm bảy VLAN, năm trunk, STP root được chỉ định, sáu SVI/gateway HSRP và quản trị hai Access. Người thực hành xác nhận kết nối hoạt động sau khi giữ workaround `no ip cef` trên hai Core. Bài thử uplink ACCESS1 có log traffic gián đoạn rồi phục hồi; bài ACCESS2 được bỏ theo yêu cầu. Export thật đã được kiểm tra cấu trúc và cabling tại local.

Nguồn báo cáo gồm cấu hình/output người thực hành cung cấp trong chat và file, xác nhận thực hành qua từng bước, cùng kiểm tra artifact local. Agent không chạy lệnh trên thiết bị EVE. Kết quả có raw output, review cấu hình và xác nhận chung được phân biệt trong [validation 13 ca](../test-results/sprint-01-validation.md).

## 2. Kết quả theo story

### NET-101 — VLAN và access port

- Tạo VLAN 10 MGMT, 20 IT, 30 HR, 40 ACCOUNTING, 50 SERVER, 60 GUEST, 99 NATIVE trên cả bốn switch; VTP transparent.
- ACCESS1 e0/2 VLAN 20, e0/3 VLAN 30; ACCESS2 e0/2 VLAN 40, e0/3 VLAN 60. Snapshot cuối xác nhận PC-ACC đã được trả port về VLAN 40 sau bài VLAN 50 tạm.
- Bài cùng VLAN qua hai Access và tách VLAN trước routing đã được hướng dẫn/thực hành, người thực hành xác nhận đúng mong đợi. Chưa có log riêng đủ MAC/ARP/số gói để nghiệm thu định lượng từng pha.
- **Kết luận:** cấu hình phân VLAN/access đã đối chiếu đạt; phần traffic L2 có xác nhận thực hành, giữ giới hạn log tại TC-05/06. Không đồng nhất VLAN segmentation với chặn truy cập liên VLAN sau khi bật routing.

### NET-102 — Trunk 802.1Q

- Cả mười đầu của năm link switch–switch có dot1q/static trunk, native 99, allowed `10,20,30,40,50,60,99` và `switchport nonegotiate`.
- Output hai Core có ba trunk native 99; Core runtime có VLAN allowed/active. Output Access cho thấy uplink dự phòng tham gia STP và không có inconsistent port ở thời điểm kiểm tra.
- Chuyển inter-core từ baseline native 1/allowed 10 sang cấu hình Campus; hai đầu cuối nhất quán trong snapshot.
- **Kết luận:** cấu hình cả năm trunk đã đối chiếu đạt, runtime đã nhận phù hợp. Trunking không đồng nghĩa mọi VLAN forwarding trên mọi link; STP vẫn chặn đường dự phòng.

### NET-104 — Rapid STP

- Rapid-PVST trên cả bốn switch; CORE1 base priority 24576, CORE2 28672 cho cả bảy VLAN, Access giữ mặc định.
- Output root xác nhận CORE1 root cả bảy VLAN; CORE2 đi về root qua e0/0. VLAN 20 trên cả hai Access có e0/0 Root/FWD, e0/1 Altn/BLK. Mỗi Access báo e0/1 blocked trên bảy VLAN và không có inconsistent port.
- Bài uplink ACCESS1 có hai khoảng mất 15 gói và traffic phục hồi sau mỗi khoảng. Chưa có output STP trong lúc lỗi/timestamp để định lượng convergence hoặc xác nhận từng chuyển role.
- **Kết luận:** root/role bình thường đã có bằng chứng; traffic phục hồi đã có log. Acceptance failure test gốc chỉ đạt một phần.

### NET-103 — Inter-VLAN Routing và gateway

- Hai Core có SVI 10/20/30/40/50/60 theo IP Plan: CORE1 `.2/24`, CORE2 `.3/24`, VIP `.1`, group bằng VLAN ID. CORE1 priority 110, CORE2 mặc định 100, preempt hai phía. VLAN 99 chỉ L2.
- ACCESS1/2 giữ `no ip routing`, SVI 10 `.11/.12`, default gateway `10.10.10.1`. Các ping Access→VIP/Access còn lại đạt 5/5 theo output; quản trị từ VLAN 20 được người thực hành xác nhận chung.
- HSRP 10 có output CORE1 Active/CORE2 Standby. Sau mở rộng, người thực hành báo các nhóm/gateway và kết nối hoạt động ổn; chưa có runtime cuối đủ sáu nhóm lưu riêng.
- Forwarding client-to-client được khắc phục bằng `no ip cef` theo xác nhận. Log uplink có phản hồi từ PC-ACC `10.10.40.20` trong bài ping từ IT. VLAN 50 có SVI/HSRP và bài VPCS tạm; chưa đủ log để nghiệm thu riêng HR↔VLAN 50 hai chiều, 10/10 mỗi chiều.
- **Kết luận:** cấu hình SVI/VIP được đối chiếu, forwarding được xác nhận với ngoại lệ CEF. Không ghi full HSRP HA, dịch vụ server hoặc mọi acceptance liên VLAN đã PASS độc lập.

## 3. Baseline bàn giao

- **Topology:** bốn IOL switch, bốn VPCS, năm link switch–switch và bốn link endpoint; không đổi cabling so với HQ Sprint 0. [Bảng cổng](../../diagrams/physical-topology.md).
- **IP/gateway:** giữ [IP Plan v1.0](../ip-addressing-plan.md). IP endpoint dùng trong thực hành: `10.10.20.20` (IT), `10.10.30.20` (HR), `10.10.40.20` (ACC), `10.10.60.20` (GUEST), /24, gateway `.1` tương ứng; chưa có snapshot VPCS cuối riêng.
- **Quản trị:** VTY0–4 có `transport input none` trên cả bốn, thao tác qua console EVE; chưa nghiệm thu SSH/AAA. CORE1 e1/0 cổng test cũ đã shutdown. Không còn shutdown uplink Access trong cấu hình cuối.
- **Image:** export dùng `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`, IOS 15.2 trong running; RAM metadata Core 512/Access 256 MB. Không có benchmark hiệu năng hoặc kiểm thử tải đầy đủ.
- **Cấu hình:** bốn running cuối và năm bản lịch sử; có log lưu thành công CORE2 và hai Access. Chưa có output lưu CORE1 sau shutdown cổng test hoặc startup bốn switch để so sánh. [Review và nguồn](../test-results/sprint-01-config-review-2026-10-11.md).

## 4. Lỗi forwarding và quyết định CEF

Khi chỉ mới có SVI 20/30 trên CORE1, PC ping gateway được nhưng client-to-client timeout dù bảng route connected, SVI up/up và ARP đúng. Core ping HR với source `10.10.20.2` và IT với source `10.10.30.2` đều 5/5. Các phép ping do Core tạo ra này kiểm tra đường tới endpoint/return path, không chứng minh đường transit client-to-client.

Người thực hành xác nhận phép đối chiếu CEF bật–tắt–bật cho **timeout → thành công → timeout**, và CORE2 có triệu chứng tương tự. Chưa có log đầy đủ từng pha trên hai Core hoặc bug ID xác định; không kết luận do Windows hay khẳng định lỗi chung của mọi IOL.

**Quyết định đã chọn:** giữ `no ip cef` trên CORE1/CORE2 để hoàn thành thực hành Sprint 1, không đổi image/cabling/IP giữa bài. Snapshot cuối xác nhận workaround đã có. Không nghiệm thu hiệu năng hay tính năng phụ thuộc CEF từ kết quả này; xem xét lại image trước mở rộng WAN Sprint 3. Không tự áp workaround lên Edge/Branch/firewall. [Biên bản điều tra](../test-results/sprint-01-core1-cef-investigation-2026-10-10.md).

## 5. Bài thử mất uplink ACCESS1

Người thực hành chạy `ping 10.10.40.20 -t` trong bài shutdown/no shutdown ACCESS1 e0/0. [Log gốc](../test-results/sprint-01-access1-uplink-ping-2026-10-11.txt) có:

- Sequence 1–24: 24 phản hồi.
- Sequence 25–39: 15 timeout; sequence 40–119: 80 phản hồi liên tiếp.
- Sequence 120–134: 15 timeout; sequence 135–152: 18 phản hồi liên tiếp trước Ctrl+C.
- Tổng đoạn log: 152 phép thử, 122 phản hồi, 30 timeout.

Traffic phục hồi sau mỗi cụm gián đoạn. Không có timestamp lệnh gây lỗi, interval/timeout thực tế hoặc STP trong lúc lỗi; không đổi 15 gói thành 15 giây, không xác định chính xác thời gian hội tụ, không gán từng cụm cho shutdown/no shutdown như một sự kiện đã đối chiếu. Nhãn `Shr` ở các output bình thường là thông tin cần lưu khi phân tích sau, chưa đủ để kết luận nguyên nhân mất gói.

Bài thực tế là ping liên VLAN, chưa tách riêng phép thử cùng VLAN như kế hoạch gốc. **ACCESS2 không thử theo quyết định người thực hành**, giữ trạng thái BỎ QUA. Bài này không chứng minh failover khi tắt cả Core hoặc mất inter-core. [Biên bản đầy đủ](../test-results/sprint-01-access1-uplink-test-2026-10-11.md).

## 6. Export và hồ sơ

[Export HQ Sprint 01](../../eve-exports/sprint-01-hq-2026-10-11.zip) có 947 bytes, chứa một `Enterprise-Network-HQ.unl` 4241 bytes.

**SHA256 ZIP:** `5B84F2E09DBEC6101E1D7AE925A42DAC6A77575C37131726C4F4715EB94F3D83`.

[Kiểm tra export tại local](../test-results/sprint-01-export-inspection-2026-10-11.txt) đạt CRC/XML, tám node/tên-ID duy nhất, chín network mỗi network hai đầu và toàn bộ chín link đúng bảng cổng. ZIP chỉ có topology/metadata, tất cả node `config=0`, không có config nhúng. Cần đi kèm bốn snapshot cấu hình và thông tin VLAN; chưa nghiệm thu import/restore Sprint 01 hoặc persistence sau reload.

Bộ hồ sơ gồm [validation](../test-results/sprint-01-validation.md), [review config](../test-results/sprint-01-config-review-2026-10-11.md), [runtime trích từ chat](../test-results/sprint-01-campus-runtime-excerpts-2026-10-11.txt), [session HQ](../test-results/sprint-01-hq-config-session-2026-10-11.txt), biên bản CEF/uplink và chín snapshot. [Ảnh topology Sprint 0](../../screenshots/sprint-00/hq-topology-2026-10-10.png) dùng tham chiếu cabling không đổi; chưa có bộ ảnh runtime Sprint 01 riêng. Không tạo output, startup hay ảnh giả để bù phần thiếu.

## 7. Giới hạn và việc bàn giao

- **S1-F01 — Startup/persistence:** chưa đối chiếu startup bốn switch; CORE1 sau shutdown cổng test chưa có output lưu. Giữ running cuối làm nguồn cấu hình được review; bổ sung trong lượt backup/restore kế tiếp trước khi dựa vào persistence. Không yêu cầu gửi lại running hoặc giữ mở báo cáo vì thiếu startup.
- **S1-F02 — Khôi phục:** export không nhúng config, chưa có restore Sprint 01 được kiểm chứng. Trước thay đổi lớn hoặc khi cần kiểm thử phục hồi, import vào lab riêng, dùng config/VLAN đi kèm và kiểm tra lại phần liên quan; không reload lab hiện tại chỉ để hoàn thiện báo cáo.
- **S1-F03 — CEF/image, trước Sprint 3:** giữ ngoại lệ hiện tại, đánh giá image hỗ trợ forwarding/CEF phù hợp trước mở WAN. Nếu thay image, kiểm thử lại chức năng chịu ảnh hưởng và cập nhật inventory; chưa có bug ID hoặc kết luận nguyên nhân sâu.
- **S1-F04 — HA/convergence, NET-202 Sprint 2:** full Core/client failover, đo loss/thời gian có timestamp, kiểm tra role trong lúc lỗi chưa nghiệm thu. Có thể ghép thu bằng chứng vào bài HA tiếp theo; không bắt buộc lặp ACCESS2 trong Sprint 1 đã chốt.
- **S1-F05 — Độ đầy đủ bằng chứng:** cùng VLAN, negative test trước routing, ping qua từng Core, HR↔VLAN 50 và management từ VLAN 20 có mức xác nhận/log khác nhau. Giữ trạng thái tại validation; bổ sung khi chạy bài liên quan tiếp theo, không tự nâng thành 10/10 hoặc mọi ca PASS.
- **S1-F06 — Hồ sơ Git/ảnh:** báo cáo/validation/config/export được lưu bằng commit local riêng; chưa push/PR và chưa có ảnh runtime Sprint01 riêng. Theo DoD chung, chưa ghi story triển khai DONE đầy đủ. Đây là trạng thái xuất bản hồ sơ, tách khỏi việc chốt thực hành.

Guest isolation/ACL, WAN/OSPF/BGP, NAT/firewall/VPN và dịch vụ Alpine vẫn thuộc sprint sau. Việc Guest ping được IT khi routing chưa có ACL là đúng phạm vi hiện tại.

## 8. Kiến thức rút ra và bước tiếp theo

VLAN tách broadcast domain; trunk mang nhiều VLAN nhưng STP quyết định đường forwarding. SVI cung cấp gateway/định tuyến cho subnet trực tiếp nên không cần static route hay OSPF chỉ để nối các VLAN đã connected trên cùng Core. HSRP giữ gateway VIP, còn STP xử lý đường L2; mất uplink Access không đồng nghĩa Active HSRP phải chuyển. Route/ARP đúng và ping từ Core thành công vẫn chưa đủ chứng minh client transit khi đường forwarding gặp lỗi.

Chuyển sang **Sprint 2 — NET-201 LACP và NET-202 HA** từ baseline này. Topology hiện mỗi cặp endpoint chỉ có một link, cần thêm link cùng cặp trước EtherChannel và cập nhật bảng cổng; không gom uplink tới hai Core độc lập vào một channel. Full failover và số đo hội tụ được thực hiện trong bài HA có traffic thật, giữ rõ ngoại lệ CEF và các follow-up hiện có.
