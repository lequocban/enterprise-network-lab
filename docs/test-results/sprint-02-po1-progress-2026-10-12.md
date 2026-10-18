# Sprint 02 — Checkpoint triển khai Po1, 12/10/2026

**Trạng thái cập nhật16/10/2026:** **ĐỦ BỐN SNAPSHOT Po1; HAI CẶP KHỚP127/116 DÒNG LỆNH.** Full nguồn12/10/lưu `[OK]`/HR–ACC–VIP mỗi mẫu5/5 giữ lịch sử; hai header lẫn chữ đã được xác nhận bằng section16/10, bổ sung CORE1 startup/CORE2 running, raw cũ giữ nguyên. Không coi đây là full dump mới16/10. Preflight Po11 đã có, chuẩn bị shutdown cổng mới/thêm dây; chưa tạo Po11 hoặc nghiệm thu HA/traffic qua channel. S2-FWD-01 phục hồi, S2-IOL-01 giữ nguyên; xem mục10.  
**Nguồn:** [trước ghép channel](sprint-02-po1-before-channel-2026-10-12.txt) và [Po1 một member hoạt động](sprint-02-po1-one-member-2026-10-12.txt), console người thực hành cung cấp, chép từ tin nhắn và chuẩn hóa khoảng trắng cuối dòng. Agent không thực thi lệnh trên switch. Output không có `show clock`/uptime, không gán giờ thu thiết bị hoặc chứng minh stop/start từ counter.

**Nguồn bổ sung hai member:**

- [CORE1 — session nguyên trạng](sprint-02-core1-po1-two-members-2026-10-12.txt), SHA256 `CC3DD0ABE5A44DC248A6CDDBE3581A90711A56A04BAD19C575E67E79019EDA43`.
- [CORE2 — session nguyên trạng](sprint-02-core2-po1-two-members-2026-10-12.txt), SHA256 `FA634FDC3CDBE7580E028517CDB63C0643DD4EDE7067C0D5B477DBDE679E9561`.
- [PC-IT — hai lượt ping10/10](sprint-02-po1-client-ping-2026-10-12.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng.

Hai file Core được sao chép nguyên trạng, hash khớp nguồn. CORE2 chứa cả trạng thái Et0/3(D) trước `no shut` và Et0/3(P) sau mở; lấy snapshot cuối cho checkpoint hiện tại, giữ lịch sử chuyển đổi trong raw session. Syslog CORE2 có mốc `Oct 12 14:20:34.760` cấu hình, `14:20:35.872` link up, `14:20:36.879` protocol up; lượt này không có `show clock` xác nhận timezone/năm, không quy đổi giờ hoặc dùng các mốc đó để nghiệm thu thời gian LACP/traffic hội tụ.

## 1. Actual checkpoint trước ghép channel

- CORE1 Active, priority110, preempt cho đủ group10/20/30/40/50/60; CORE2 Standby, priority100, preempt. Peer và VIP đúng các subnet `10.10.X.0/24`. Không có dual-active trong hai snapshot được gửi.
- Cả hai Core có Et0/0/Et0/1/Et0/2 trunking dot1q, native99. Allowed/active/forwarding-not-pruned đều `10,20,30,40,50,60,99` trong output hiện tại.
- Ethernet0/3 cả hai Core administratively down/down, MTU1500, Auto-duplex/Auto-speed, BW10000; counter input/output0, lỗi/CRC/collision0. Đã xác nhận cổng dự kiến member mới đang đóng ở hai đầu.
- Chưa có `show etherchannel summary`, traffic PC mới hoặc xác nhận/ảnh đầu nối EVE trong lượt này. Không suy ra e0/3 đã nối đúng peer từ trạng thái administratively down. Chưa nâng topology thực tế từ chín lên mười link chỉ dựa trên output này.

## 2. Actual checkpoint một member đã hoạt động

- Cả hai Core: group1, `Po1(SU)`, protocol `LACP`, `Et0/0(P)`, `Et0/3(D)`. Đã có channel Layer2 in-use và member cũ thực sự bundled; không còn chỉ là cấu hình dự kiến.
- `show lacp neighbor` trên e0/0 có partner thật: CORE1 thấy `aabb.cc80.2000`, CORE2 thấy `aabb.cc80.1000`; flags `SA`, Oper Key `0x1`. `SA` tại bảng partner là Slow LACPDUs/Active, không phải trạng thái HSRP hoặc tốc độ truyền dữ liệu.
- Dòng e0/3 hiện `FP`, Dev ID `0000.0000.0000`, key0. Chưa có partner hợp lệ trên member down trong output này; không dùng cờ `P` tại dòng đó để kết luận Core đối diện đã cấu hình passive. Khác với `(P)` trong summary là bundled.
- Trunk của hai Core hiện Po1 cùng e0/1/e0/2; member e0/0 không liệt kê thành trunk độc lập. Po1 native99, allowed/active/forwarding-not-pruned đủ bảy VLAN.
- CORE1 vẫn Active110, CORE2 Standby100, preempt, đủ sáu group và VIP đúng. Việc tạo LACP không tự yêu cầu HSRP đổi Active.
- Chưa có interface detail sau cấu hình để xác nhận duplex full thực tế, STP mới, client ping, running/startup mới hoặc link e0/3 có đúng peer. Không lấy việc Po1 lên với một member làm bằng chứng đã thử mất member.

## 3. Actual checkpoint hai member đã hoạt động

- **LACP:** snapshot cuối cả hai Core có group1, Po1(SU), protocol LACP, Et0/0(P), Et0/3(P). Cả hai member CORE1 nhận partner `aabb.cc80.2000`, cả hai member CORE2 nhận `aabb.cc80.1000`; flags SA, Oper Key0x1, remote Port Number0x1/0x4. Đã có bằng chứng hai member bundled cùng peer, khác checkpoint một member trước.
- **Interface:** bốn đầu member up/up, description MEMBER1/MEMBER2 đúng Po1, MTU1500, BW10000. Tất cả vẫn hiển thị **Auto-duplex, Auto-speed**, chưa xác nhận full-duplex bằng output. Các counter input errors/CRC/output errors/collisions/interface resets và output drops đều0 trong snapshot; không có phép đo delta theo tải/thời gian.
- **Trunk:** Po1 trên hai Core trunking dot1q/native99; allowed/active/forwarding-not-pruned đủ VLAN10/20/30/40/50/60/99. e0/1/e0/2 vẫn trunk riêng.
- **STP:** CORE1 root MAC `aabb.cc00.1000`, cost0 đủ bảy VLAN Campus. CORE2 nhận cùng root, **root port Po1, cost56** đủ bảy VLAN; baseline trước channel là e0/0/cost100. Đây là số đo từ output STP, không dùng nó làm công thức throughput hoặc kết luận tốc độ một luồng tăng gấp đôi. Inconsistent0 trên cả hai Core. VLAN1 vẫn riêng từng Core như baseline, không thuộc allowed list.
- **HSRP:** CORE1 Active110, CORE2 Standby100, preempt đủ sáu group/VIP đúng; không có dual-active trong snapshot. Có thêm member LACP không yêu cầu đổi gateway VIP hoặc chuyển HSRP Active.
- **Traffic:** IT→HR `10.10.30.20` 10/10, IT→ACC `10.10.40.20` 10/10; tổng20/20 phản hồi của **hai mẫu10 gói**, không ghi mỗi mẫu20/20. Xác nhận hai hướng liên VLAN đã thử từ IT, chưa có ping chiều ngược mới/GUEST/quản trị sau chuyển đổi. Không tự khẳng định các ping client đi qua Po1; ca traffic qua channel còn cần ping SVI Core/counter/capture tương ứng.

## 4. Actual đối chiếu section running và STP VLAN20

Nguồn: [sáu section interface và STP hai Core](sprint-02-po1-duplex-review-2026-10-12.txt), chép từ tin nhắn người thực hành, chuẩn hóa khoảng trắng cuối dòng. Lượt này không có timestamp thiết bị, full running/startup hoặc phản hồi khi nhập lệnh duplex.

- Bốn member đều có `channel-group 1 mode active`, trunk dot1q/native99, allowed đủ bảy VLAN và nonegotiate. Hai section Po1 có cùng cấu hình trunk. Không thấy sai khác trunk/mode LACP giữa các section đã gửi.
- Không section member nào hiển thị `duplex full`. Kết hợp interface trước vẫn báo Auto-duplex, hiện chưa có bằng chứng full-duplex. Không kết luận người thực hành bỏ quên lệnh: `show running-config` thông thường có thể bỏ các lệnh mang giá trị mặc định; hành vi hiển thị tùy release. Tham khảo [Cisco — show running-config](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/fundamentals/command/cf_command_ref/show_protocols_through_showmon.html).
- VLAN20: CORE1 root `aabb.cc00.1000`, bridge priority24596 = 24576 + VLAN20; Po1 `Desg FWD`, cost56. CORE2 bridge priority28692 = 28672 + VLAN20, nhận cùng root qua Po1 `Root FWD`, cost56. Vai trò/forwarding phù hợp thiết kế root trên CORE1.
- Po1 cả hai Core có type `Shr`; các trunk e0/1/e0/2 cũng `Shr`. Đây là cách STP phân loại shared link, không phải trạng thái blocked; trạng thái đã ghi là FWD. RSTP phân biệt shared/point-to-point khi hội tụ; không dùng riêng nhãn Shr để khẳng định duplex mismatch hoặc lỗi IOL. Tham khảo [Cisco — Understand RSTP](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24062-146.html).

## 5. Actual phép thử và ngoại lệ được chấp nhận — S2-IOL-01

**Nguồn:** [console CORE1 thử duplex trên e0/3](sprint-02-po1-duplex-probe-2026-10-12.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. Người thực hành xác nhận kết quả CORE2 tương tự; chưa có raw CORE2 riêng cho phép thử này, không sao chép CORE1 thành bằng chứng CORE2. Syslog có `Oct 12 14:39:27.830` nhưng không có show clock/timezone trong lượt, không dùng để đo hội tụ.

**Quan sát:** CLI CORE1 nhận `duplex full` không hiện lỗi; running e0/3 vẫn không hiện lệnh duplex, có shutdown và group1 active; interface administratively down/down, vẫn Auto-duplex/Auto-speed, các counter lỗi/CRC/collision/reset bằng0. Po1 vẫn SU, e0/0(P), e0/3(D). Chỉ xác nhận parser không báo lỗi và đường e0/0 còn bundled; chưa chứng minh lệnh tác động duplex vận hành hoặc không ảnh hưởng toàn bộ traffic.

**Quyết định 12/10/2026:** theo yêu cầu người thực hành, ghi **giới hạn cấu hình/hiển thị duplex quan sát trên image IOL — CHẤP NHẬN CÓ ĐIỀU KIỆN**. Image: `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`. Nguyên nhân cụ thể chưa xác minh; không khẳng định image không mô phỏng full-duplex hoặc rằng interface thực sự half/full. Dừng thử lặp lệnh duplex, không yêu cầu đổi image hoặc ép STP P2p để tiếp tục Sprint2. Trạng thái full-duplex ghi **chưa xác minh/ngoại lệ**, không ghi PASS.

**Phạm vi và điều kiện:** tiếp tục học VLAN/trunk/LACP/HSRP trên image hiện tại nếu mở lại e0/3 cho đủ hai member bundled đúng peer, không inconsistent/dual-active kéo dài, traffic hoạt động và không có lỗi tăng bất thường. Po11 nếu gặp cùng hiện tượng được dùng cùng nguyên tắc đánh giá, nhưng vẫn phải có bằng chứng bundle/trunk/STP/HSRP/traffic riêng. Ngoại lệ này không miễn kiểm thử chịu lỗi, lưu startup hoặc thay thế yêu cầu speed/duplex tương thích trên thiết bị thật.

**Ảnh hưởng cần giữ lại:** shared link type có thể ảnh hưởng khả năng chuyển nhanh của RSTP; thời gian hội tụ phải đo theo actual của lab, không cam kết không ảnh hưởng các ca mất toàn channel/Core và không suy rộng sang switch vật lý. Tham khảo [Cisco — RSTP, Link Type](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24062-146.html). Nếu member suspended/standalone, traffic lỗi, counter lỗi tăng hoặc HA không đạt, mở lại điều tra giới hạn này từ checkpoint ổn định; không bỏ qua kết quả FAIL vì ngoại lệ hiển thị.

## 6. Actual sau mở lại hai member — lỗi S2-FWD-01

Nguồn: [console hai Core và hai client](sprint-02-po1-client-failure-2026-10-12.txt), chép từ tin nhắn người thực hành, chuẩn hóa khoảng trắng cuối dòng. Không có show clock hoặc log thời điểm mở member; chỉ xác nhận thứ tự tiếp nhận, không dùng quan hệ trước/sau làm bằng chứng duplex/Po1 gây lỗi.

- **Channel:** cả hai Core Po1(SU), LACP, e0/0(P), e0/3(P); hai member cùng partner đối diện/Oper Key1, Port Number1/4. Trunk Po1/e0/1/e0/2 native99, allowed/active/forwarding đủ bảy VLAN; inconsistent0 cả hai Core. Điều kiện hình thành channel đạt trong snapshot; chưa chứng minh mọi luồng client được chuyển tiếp.
- **HSRP:** sáu nhóm CORE1 Active110/preempt, CORE2 Standby100/preempt, peer/VIP đúng. Không có dual-active trong snapshot. Đây là trạng thái giao thức giữa các Core, chưa chứng minh client có đường ARP/L2 đến gateway.
- **Ping liên Core:** `10.10.10.2 → 10.10.10.3`, 10 gói100byte, timeout2s, `.!!!!!!!!!`, **9/10**, RTT min/avg/max1ms. Gói đầu mất có thể liên quan ARP nhưng chưa có bảng ARP/capture xác nhận; không sửa thành10/10, không lấy VLAN10 thành công làm bằng chứng VLAN20/30/40 hoạt động hoặc chắc chắn mọi gói qua Po1.
- **PC-IT:** năm lượt tới ACC (một `-c10`, bốn `-c5`), một lượt tới GUEST, một tới HR và một trực tiếp gateway20.1 đều báo `host (10.10.20.1) not reachable`. Không chuyển tham số -c thành số ICMP thực đã gửi/mất khi VPCS không tới next hop. Chưa có show ip/ARP mới trong lượt này.
- **PC-ACC:** tới IT thấy4 timeout rồi Ctrl-C; tới HR thấy3 timeout rồi Ctrl-C. Ping `10.10.40.20` có5 phản hồi nhưng đây là IP chính PC-ACC theo baseline; tự ping chỉ kiểm tra local stack, không chứng minh link ACCESS2/gateway hoạt động.

**Đánh giá:** traffic client tại checkpoint này **FAIL**, S02-TC-03 chưa đạt toàn ca. Ưu tiên khoanh vùng PC-IT ↔ ACCESS1 ↔ Core trong VLAN20 và ARP tới VIP/SVI thật. Cần so sánh ping20.1/20.2/20.3, MAC learning/access port/trunk/STP Access, ARP/SVI Core trước quyết định sửa. Chưa kết luận lỗi duplex, LACP, CEF, HSRP virtual MAC, sai VLAN hoặc cabling; không gọi mất gói lặp lại là warm-up bình thường.

## 7. Kế hoạch khoanh vùng tại thời điểm lỗi — giữ để tham khảo

Thu PC-IT show ip, ping ba địa chỉ20.2/20.3/20.1 và show arp; ACCESS1 interface status/VLAN/trunk/STP VLAN20/MAC dynamic VLAN20/section e0/2; CORE1 ip interface brief/ARP PC-IT/MAC VLAN20. Đối chiếu baseline PC-IT `10.10.20.20/24`, gateway20.1, MAC0050.7966.6805, ACCESS1 e0/2 access VLAN20. Các trạng thái/cổng baseline là kỳ vọng để so sánh, không gán thành actual mới.

Giữ hiện trạng để thu bằng chứng trước, không clear ARP/MAC, bật–tắt CEF, đổi gateway client, ép P2p hoặc rebuild channel ở vòng đầu. Chỉ thay đổi sau khi định vị, mỗi lần một nguyên nhân. Nếu bằng chứng hướng tới member mới/image forwarding, có thể thử có kiểm soát e0/3 shutdown hai đầu để so sánh, nhưng phép thử đó **chưa thực hiện** và không thay kết luận từ output.

Khi lập vòng khoanh vùng, chưa lưu startup/chuyển Po11 khi lỗi còn. **Cập nhật:** người thực hành đã xác định ACCESS1 treo và báo phục hồi kèm ping5/5 ở mục8; không yêu cầu thực hiện lại bộ chẩn đoán trên để tiếp tục. Chưa có kết quả của bộ lệnh khoanh vùng, không ghi nó đã chạy/PASS. Không tự ghi phép shutdown thử duplex thành PASS ca chịu lỗi/đo HA, không tạo config Po1 từ section/block hướng dẫn. Các kết quả trước/sau sự cố giữ riêng theo nguồn.

Nguyên tắc ARP tới gateway ảo, kiểm tra đường L2 và bảng MAC tham khảo [Cisco — Troubleshoot HSRP Common Issues](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/10583-62.html); đây là phương pháp khoanh vùng, không phải xác nhận bug của image IOL.

## 8. Phục hồi S2-FWD-01 và tiếp tục checkpoint

**Nguồn:** [PC-IT ping ACC sau phục hồi](sprint-02-po1-client-recovery-2026-10-12.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. Người thực hành xác nhận: ACCESS1 bị treo, hiện đã hoạt động ổn. Không có timestamp thiết bị/log ACCESS1 hoặc mô tả thao tác phục hồi; không ghi đã reload/reboot/đổi cấu hình hoặc xác định nguyên nhân sâu của việc treo.

**Actual:** PC-IT→10.10.40.20 có5 phản hồi liên tiếp, sequence1–5, TTL63, RTT3.034/1.683/1.436/1.129/1.369ms, không có timeout trong mẫu. Ghi **5/5 của lượt này**, không sửa các mẫu lỗi trước, không nâng thành10/10 hay xác nhận mọi VLAN/chiều đã kiểm tra sau phục hồi. Không có output Core mới trong lượt này; runtime LACP/trunk/HSRP gần nhất ở mục6 được giữ làm tham chiếu.

**Kết luận:** S2-FWD-01 **đã phục hồi theo xác nhận người thực hành; IT→ACC5/5 xác nhận đường client này hoạt động lại**. ACCESS1 treo là nguyên nhân được người thực hành báo, chưa có log xác minh cơ chế treo. Không dùng sự cố để kết luận lỗi Po1, duplex hoặc CEF. Dừng vòng khoanh vùng cũ; nếu triệu chứng tái diễn mới thu runtime/ARP/MAC/log liên quan, không yêu cầu lặp toàn bộ chẩn đoán khi đã phục hồi.

**Bước tiếp theo:** lưu checkpoint trên CORE1/CORE2 và thu full running/startup thật; có thể kiểm tra nhanh VIP20.1 từ PC-IT trước lưu, không yêu cầu lặp toàn ma trận ping. Giữ ngoại lệ S2-IOL-01 và workaround no ip cef đã chọn. Sau đối chiếu snapshot mới tiếp tục Po11; ca traffic qua channel, mất member/full HA còn riêng, không tự ghi PASS toàn S02-TC-03 hoặc sprint chỉ từ phục hồi5/5. Chưa tạo file config Po1 từ các đoạn hướng dẫn.

## 9. Actual lưu Po1 và đối chiếu cấu hình

Nguồn: [wr, bốn bản running/startup và ba mẫu ping](sprint-02-po1-save-session-2026-10-12.txt), chép từ tin nhắn người thực hành; chuẩn hóa dòng trắng/phân cách comment và khoảng trắng cuối dòng, giữ nguyên mọi dòng lệnh kể cả hai dòng lẫn chữ. Không phải bản sao byte của file export thiết bị. Agent không thực thi lệnh trên Core.

- CORE1 `wr` báo `[OK]`, cấu hình3300byte, nén1674byte; show startup dùng1674/524288byte, uncompressed3300. Metadata last configuration change `14:57:01 UTC Mon Oct 12 2026`, không lấy làm thời điểm save/ping hoặc đo hội tụ.
- CORE2 `wr` báo `[OK]`, cấu hình3035byte, nén1583byte; show startup dùng1583/524288byte, uncompressed3035. Metadata last change `14:57:20 UTC Mon Oct 12 2026`, cùng giới hạn trên.
- Hai bản sạch đã lưu: [CORE1.po1-only.running.cfg](../../configs/core/sprint-02/CORE1.po1-only.running.cfg), **127 dòng lệnh**; [CORE2.po1-only.startup.cfg](../../configs/core/sprint-02/CORE2.po1-only.startup.cfg), **116 dòng lệnh** sau bỏ comment/dòng trắng, giữ indent/nội dung lệnh. Không dùng hướng dẫn làm nguồn config, không ghi đè pre-lacp.
- Hai Core có Po1 L2 trunk dot1q/native99/allowed bảy VLAN/nonegotiate; e0/0 và e0/3 group1 active, không shutdown. e0/1/e0/2 giữ trunk Access riêng; chưa có Po11, CORE1 e1/1 chưa cấu hình. C1 test e1/0 vẫn shutdown; no ip cef, SVI/HSRP/preempt và STP priorities24576/28672 giữ đúng checkpoint. Không có dòng duplex full; ngoại lệ S2-IOL-01 giữ nguyên.

**Hai khác biệt văn bản cần xác nhận:** so từng dòng lệnh, CORE1 running/startup có cùng127 dòng, chỉ khác dòng76: running `interface Ethernet1/2`, startup `interface Ethernet1/2RLY DEPLOYMENT DEVELO`. CORE2 có cùng116 dòng, chỉ khác dòng54: running `interface Ethernet0/2RLY DEPLOYMENT DEVELO`, startup `interface Ethernet0/2`. Đây là dòng bản dán bất thường, chưa coi là cấu hình thiết bị thật hoặc tự sửa/xóa trong nguồn. Chưa trích CORE1 startup và CORE2 running thành `.cfg` hợp lệ; chưa tuyên bố hai cặp khớp nguyên vẹn.

Chỉ cần CORE1 `show startup-config | section ^interface Ethernet1/2` và CORE2 `show running-config interface Ethernet0/2` để xác nhận lại section liên quan; không yêu cầu wr hoặc full dump lặp. Sau output sạch, bổ sung hai snapshot còn lại từ raw kết hợp section xác nhận, ghi rõ provenance của việc thay phần lẫn chữ.

**Traffic:** trong tin nhắn có IT→HR30.20, IT→ACC40.20, IT→VIP20.1, mỗi mẫu5 phản hồi sequence1–5. HR/ACC TTL63, VIP TTL255. Block ACC trùng năm RTT của lượt phục hồi trước, không cộng thành một lượt độc lập để tăng tổng số gói đã kiểm thử. Xác nhận các mẫu được cung cấp thành công, không suy ra mọi VLAN/chiều hoặc đường Po1. Ping liên Core9/10 trước vẫn giữ lịch sử; HA, traffic/counter/capture qua channel còn riêng.

**Checkpoint tiếp theo:** hình thành/vận hành Po1 và lưu thành công đã có; hoàn thiện hai dòng nguồn, chuẩn bị preflight Po11 CORE1–ACCESS1. Không yêu cầu lại bộ chẩn đoán ACCESS1 treo, thử duplex hoặc restore baseline đã bỏ qua. Chưa thay topology/export hoặc ghi Po11 đã chạy.

**Kiểm tra hồ sơ local ngày13/10/2026:** hai snapshot sạch khớp dòng lệnh với nguồn tương ứng; so baseline pre-lacp theo section chỉ có thay đổi ở Port-channel1/Ethernet0/0/Ethernet0/3, global và các interface khác không đổi. Xác nhận bốn nguồn chỉ có hai khác biệt văn bản đã ghi, các mẫu ping đúng5 phản hồi; liên kết local/Markdown đã kiểm tra. Đây là kiểm tra file, không phải output thiết bị mới, không thay thế hai section xác nhận hoặc nghiệm thu HA.

## 10. Hoàn thiện hai nguồn Po1, tiếp nhận16/10/2026

[Raw xác nhận section và preflight Po11](sprint-02-po11-preflight-2026-10-16.txt) có CORE1 startup Ethernet1/2 sạch/không lệnh con và CORE2 running Ethernet0/2 sạch/trunk ACCESS2 đúng. [Review mới](sprint-02-po11-preflight-review-2026-10-16.md) ghi nguồn và giới hạn thời gian.

Đã trích [CORE1.po1-only.startup.cfg](../../configs/core/sprint-02/CORE1.po1-only.startup.cfg) từ full startup12/10, chỉ thay header Ethernet1/2 bị lẫn chữ bằng header được xác nhận16/10; [CORE2.po1-only.running.cfg](../../configs/core/sprint-02/CORE2.po1-only.running.cfg) tương tự cho Ethernet0/2. Các lệnh con CORE2 khớp section xác nhận; không sao chép bản đối ứng thành nguồn khác, không sửa raw gốc hoặc metadata last change. Bộ Po1 hiện đủ bốn file; hai cặp khớp127/116 dòng lệnh sau bỏ comment/dòng trắng. Đây là snapshot checkpoint12/10 được hoàn thiện nguồn16/10, không chứng minh các section khác của thiết bị không thay đổi từ12/10 đến16/10.

Hai đoạn xác nhận đã hoàn tất; không yêu cầu save/full dump lại hoặc tiếp tục xử lý lỗi văn bản. Tiếp tục chuẩn bị Po11 từ preflight đã thu: CORE1 e1/1/ACCESS1 e1/0 chưa có cấu hình sử dụng hiển thị, ACCESS1 chưa có EtherChannel, VLAN20 e0/0 Root/FWD và e0/1 Altn/BLK. Chưa có bằng chứng shutdown/thêm dây/tạo channel mới; Po1 traffic/counter/capture qua channel và các ca HA vẫn riêng, không tự ghi toàn S02-TC-03 PASS từ việc đủ snapshot.

Quyết định bỏ qua thử restore baseline trước LACP và kết quả buổi 1 giữ tại [review preflight](sprint-02-preflight-review-2026-10-12.md). Quyết định bỏ qua không thay thế kết quả vận hành Po1.

Giải nghĩa cờ tham khảo [Cisco — Troubleshoot EtherChannels](https://www.cisco.com/c/en/us/support/docs/switches/catalyst-9300-series-switches/220367-troubleshoot-etherchannels-on-catalyst-9.html); trạng thái IOL thực tế được xác định từ các output đính kèm, không mặc định áp dụng mọi tính năng Catalyst trong tài liệu.
