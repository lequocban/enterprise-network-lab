# Sprint 02 — Preflight Po11 CORE1–ACCESS1, tiếp nhận 16/10/2026

**Trạng thái hồ sơ chuẩn bị:** checkpoint sau khởi động đã đạt, lỗi save/shutdown đã xử lý; hiện đã có [Po11 hai member và bốn snapshot running/startup khớp](sprint-02-po11-progress-2026-10-16.md). File này giữ lịch sử preflight/cabling/save trước chuyển channel, không yêu cầu lặp lại. Tiếp theo thử mất member Po11; HA chưa có kết quả. Ngày tiếp nhận Asia/Saigon, agent không thực thi trên switch.

**Nguồn:** [console CORE1/CORE2/ACCESS1](sprint-02-po11-preflight-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. [Po1 và nguồn cấu hình lịch sử](sprint-02-po1-progress-2026-10-12.md) giữ riêng; không gán timestamp16/10 cho toàn bộ các full config12/10.

## 1. Hai section xác nhận nguồn Po1

- CORE1 startup section Ethernet1/2 chỉ có `interface Ethernet1/2`, không có chữ lẫn hoặc lệnh con. CORE1 e1/1 running cũng chỉ có header, section29byte.
- CORE2 running Ethernet0/2 có header sạch, description TRUNK_TO_ACCESS2, trunk dot1q/native99/allowed10,20,30,40,50,60,99/nonegotiate, section229byte. Các lệnh con khớp section cùng cổng trong nguồn12/10; khác biệt trước là chữ lẫn ở header.
- Đã bổ sung [CORE1.po1-only.startup.cfg](../../configs/core/sprint-02/CORE1.po1-only.startup.cfg) và [CORE2.po1-only.running.cfg](../../configs/core/sprint-02/CORE2.po1-only.running.cfg) từ full nguồn12/10, chỉ thay header bị lẫn bằng header xác nhận16/10. Raw12/10 giữ nguyên. Hai cặp Po1 khớp127/116 dòng lệnh sau bỏ comment/dòng trắng; metadata last change12/10 giữ nguyên, không tuyên bố vừa thu full running/startup mới16/10.

## 2. Actual ACCESS1 trước Po11

- e0/0 description TRUNK_TO_CORE1, trunk dot1q/native99/allowed đủ bảy VLAN/nonegotiate, section227byte. e1/0 chỉ có header, section29byte, không có group hoặc cấu hình sử dụng hiển thị.
- EtherChannel summary có0 channel-group,0 aggregator; group11 chưa được dùng trên ACCESS1. CORE1 full config Po1 gần nhất chưa có Po11; lượt16/10 chỉ có section e1/1, không có summary toàn CORE1 mới. Đối chiếu summary CORE1 trong checkpoint tiếp theo trước chuyển đổi.
- e0/0 và e0/1 đều trunking/native99, allowed/active đủ bảy VLAN. Forwarding-not-pruned: e0/0 đủ bảy, e0/1 none.
- STP VLAN20: root aabb.cc00.1000, priority24596 = 24576+20, cost100, root port e0/0. ACCESS1 bridge aabb.cc00.3000, priority32788 = 32768+20. e0/0 Root/FWD, e0/1 Altn/BLK, e0/2 Desg/FWD; link type đều Shr. Vai trò phù hợp baseline trước Po11; trunk e0/1 có thể vẫn up nhưng không forwarding user traffic vì STP, không coi `none` là mất VLAN hoặc phải mở chặn bằng lệnh.

Hai cổng mới CORE1 e1/1 và ACCESS1 e1/0 chưa có cấu hình sử dụng hiển thị; chưa xác nhận link state, đầu nối vật lý hoặc shutdown từ các section trống. Lượt này chưa có ping client, runtime Po1/HSRP Core mới hoặc ảnh/export; không ghi các mục đó đã kiểm thử16/10. S2-IOL-01 giữ nguyên, không lặp thử duplex/ép P2p.

## 3. Checkpoint kế tiếp đã chuẩn bị

Shutdown CORE1 e1/1 và ACCESS1 e1/0, lưu thay đổi trước stop node; kiểm tra hai cổng administratively down và summary CORE1. Stop đúng CORE1/ACCESS1, thêm link CORE1 e1/1 ↔ ACCESS1 e1/0 rồi Start. Link cũ CORE1 e0/1 ↔ ACCESS1 e0/0 và đường ACCESS1 e0/1 ↔ CORE2 e0/1 giữ nguyên. Có gián đoạn có kế hoạch khi node dừng; không dùng thao tác này làm bài đo HA.

Sau start đối chiếu Po1 đủ hai P, HSRP ổn định, trunk/STP Access và ping VIP/ACC; các cổng mới vẫn shutdown. Chỉ sau checkpoint đó mới cấu hình Po11. Đích8 node/11 link là dự kiến nếu các dây khác giữ nguyên, chưa có bằng chứng thêm link hoặc export mới trong lượt preflight.

Giải nghĩa alternate/root port và blocking tham khảo [Cisco — Understand RSTP](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24062-146.html); trạng thái thực tế lấy từ output đã lưu.

## 4. Actual chuẩn bị cổng và lỗi lưu do prompt

Nguồn: [shutdown, copy save/error và retry ACCESS1](sprint-02-po11-port-preparation-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. Raw giữ nguyên lệnh copy thực đã chạy; người thực hành yêu cầu từ đây dùng wr trong hướng dẫn.

- CORE1 nhập shutdown e1/1; syslog xác nhận administratively down và protocol down. Po1 vẫn SU/LACP/e0/0(P)/e0/3(P), chỉ1 group, không có group11. Lệnh copy gặp `Destination filename [startup-config]? show interfaces ethernet1/1`, trả `%Error copying nvram:show (Invalid argument)`; chưa có save thành công sau shutdown trong lượt này, lệnh show đã bị đọc như câu trả lời tên file chứ không thực thi riêng.
- ACCESS1 cũng gặp lỗi prompt ở lần đầu; lần hai nhấn Enter tên đích mặc định và save OK,1714byte nén1033byte. e1/0 administratively down/down, Auto-duplex/Auto-speed, MTU1500/BW10000; input0/output702packet/56023byte, counter lỗi/CRC/collision/reset/drop0. Đây là trạng thái cổng đóng, không chứng minh cable/traffic qua link mới.
- Syslog CORE1 Oct16 14:09:42.901/44.905/45.913 và ACCESS1 14:10:24.381/26.383/27.388 giữ trong raw; không quy đổi giờ hoặc dùng làm số đo HA.

**Bước kế:** chỉ CORE1 chạy wr, chờ OK rồi `show startup-config | section ^interface Ethernet1/1` để xác nhận shutdown đã lưu trước stop. ACCESS1 không cần save lại. Sau đó tiếp tục thêm link đã chốt/kiểm tra sau start; chưa có bằng chứng cabling hoặc tạo channel trong lượt này. Không yêu cầu lặp shutdown hoặc nhập lệnh sai để tái hiện.

**Lý do giữ CORE1 e1/1:** [snapshot CORE1 Po1](../../configs/core/sprint-02/CORE1.po1-only.running.cfg) còn e1/0 description test_vlan10/access VLAN10/shutdown/PortFast edge. Cổng này không nối endpoint theo baseline, nhưng không phải section chưa cấu hình; chọn e1/1 để giữ port test cũ nguyên trạng. EtherChannel không bắt buộc e1/1; việc tái sử dụng e1/0 cần xử lý cấu hình cũ và đổi mapping, người thực hành mới hỏi lý do nên chưa tự chuyển cổng. Cặp mới vẫn CORE1 e1/1 ↔ ACCESS1 e1/0.

## 5. Actual sau Stop/Start và xác nhận đã thêm dây

Nguồn CLI: [CORE1 wr OK nhưng startup e1/1 không có shutdown](sprint-02-po11-core1-save-after-restart-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. Người thực hành trả lời trực tiếp đã Stop/Start CORE1 trước lần wr này và đã thêm đúng dây CORE1 e1/1 ↔ ACCESS1 e1/0. Đây là xác nhận cabling từ người thực hành; chưa có ảnh/export mới hoặc xác nhận riêng đã restart ACCESS1.

CORE1 wr OK,3300byte nén1674byte, nhưng `show startup-config | section ^interface Ethernet1/1` chỉ có `interface Ethernet1/1`. Không ghi đã lưu shutdown chỉ từ OK. Chuỗi copy lỗi → Stop/Start trước khi lưu thành công → startup thiếu shutdown phù hợp với việc mất thay đổi running chưa lưu khi khởi động lại. Không quy cho lỗi wr/NVRAM, không kết luận thời gian hội tụ hoặc bug image. Số byte trùng checkpoint cũ không thay so sánh full config; lượt này chỉ có section startup, không có running/interface state mới.

**Bước hiện tại:** không stop/nối dây lại. Chỉ CORE1 đặt shutdown e1/1, thu section running để thấy shutdown, wr/chờ OK rồi thu section startup và interface brief. Kiểm tra Po1/HSRP CORE1, trunk/STP/interface brief ACCESS1 và ping client sau start trong cùng checkpoint. ACCESS1 không cần save lặp chỉ vì CORE1 vừa sửa; nếu cổng e1/0 không còn shutdown thì xử lý theo actual trước tạo channel. Chưa có output của bước kiểm tra mới, chưa ghi cổng CORE1 hiện tại admin down hoặc Po1/HSRP sau start đã đạt từ log trước restart.

Giữ cặp e1/1 ↔ e1/0, cổng test CORE1 e1/0, các uplink cũ và Po1. Sau running/startup/runtime/client đạt mới sang tạo Po11. Việc Stop/Start vừa làm phục vụ cabling, không biến ca restore đã bỏ qua hoặc các ca HA thành PASS. Chưa tạo full config chuẩn bị Po11 mới từ các đoạn hướng dẫn.

## 6. Actual xác nhận sau khởi động — đạt checkpoint chuẩn bị

Nguồn: [running/startup e1/1, wr, Core/Access runtime và hai mẫu ping](sprint-02-po11-post-restart-check-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. Syslog cuối bị cắt ở `Line proto`, giữ nguyên đoạn nhận được, không tự hoàn thiện; interface brief đã đủ chứng cứ admin down. Không có show clock/timezone hoặc timestamp ping, không dùng mốc syslog đo HA.

- CORE1 running e1/1 section39byte có shutdown; startup section cũng có shutdown. wr OK3310byte/nén1681byte. Chênh lệch thiếu shutdown đã được xử lý; không yêu cầu save hoặc nhập lại khối sửa này.
- CORE1 e1/1 và cổng test e1/0 administratively down/down. Po1 up/up, summary SU/LACP/e0/0(P)/e0/3(P), chỉ1 group; Po11 chưa có. SVI10/20/30/40/50/60 IP .2 đúng, đều up/up. Các cổng unassigned hiển thị up không tự chứng minh dây/peer thật.
- CORE1 HSRP đủ sáu nhóm priority110/preempt/Active, standby peer10.10.X.3/VIP10.10.X.1 đúng. Đây là output CORE1 sau start; không gán thành show standby riêng mới của CORE2.
- ACCESS1 e1/0 administratively down/down; e0/0/e0/1/e0/2/e0/3 up/up, SVI10.11 up/up. Trunk e0/0/e0/1 native99/allowed/active đủ bảy VLAN; forwarding e0/0 đủ bảy/e0/1 none. VLAN20 root CORE1/cost100/e0/0; roles Root FWD/Altn BLK/Desg FWD trên e0/0/e0/1/e0/2 đúng baseline, link type Shr giữ ngoại lệ.
- PC-IT→VIP20.1 có5 phản hồi sequence1–5/TTL255; PC-IT→ACC40.20 có5 phản hồi sequence1–5/TTL63. Mỗi mẫu5/5, không suy thành nghiệm thu20/20 hoặc HA; log lỗi và mẫu trước giữ riêng.

**Bước tiếp theo:** chuyển cặp trunk CORE1–ACCESS1 sang Po11. Shutdown member cũ/new ở cả hai đầu trước cấu hình, tạo Po11/member LACP active tương thích, hoàn tất cả hai đầu rồi chỉ mở link cũ trước. Giữ CORE1 e1/1/ACCESS1 e1/0 shutdown đến checkpoint Po11 một member SU/P, trunk/HSRP/client đúng; khi đó mới thêm member mới. Po1, đường ACCESS1–CORE2 và port test CORE1 e1/0 giữ nguyên. Có thể gián đoạn tạm khi STP đổi đường trong pha chuyển; không dùng làm số đo chịu lỗi.

Chưa có output tạo Po11; chưa tạo full config chuẩn bị/channel mới từ guide hoặc section. Bộ bốn snapshot Po1 vẫn là mốc12/10 hoàn thiện nguồn16/10, không tự thêm shutdown e1/1 vào mốc cũ. Hướng dẫn từ đây dùng wr, chỉ lưu channel sau trạng thái mục tiêu được kiểm tra.
