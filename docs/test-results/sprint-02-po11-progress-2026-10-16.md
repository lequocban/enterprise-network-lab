# Sprint 02 — Po11 CORE1–ACCESS1, tiến độ16/10/2026

**Trạng thái mới nhất — điều kiện tiếp tục17/10:** người thực hành báo đã lưu cấu hình và tắt/bật máy; [phần13 biên bản](sprint-02-po11-member-a-issue-2026-10-16.md) ghi nhận cần kiểm lại sau boot, chưa có output mới/node/pha đãsave. Checkpoint cuối phần12 là Po11 chỉ new member hoạt động/MACPC quaPo11/ba mẫu5/5 vàPo1trunk đúng; không mặc định hiện trạng hoặc startup sau save mới vẫn khớp snapshot cũ. Kiểm Po1/HSRP/Po11/old sections/IP PC trước tiếp tục. Snapshot nguồn giữ nguyên, S2-LACP-02 OPEN/chưaPASSrestore hoặcHA, agent không chạy thiết bị.

**Nguồn:** [console một member](sprint-02-po11-one-member-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng; giữ lỗi gõ `how` và các ký tự `^[[A`. Không có show clock/timestamp trong lượt, ngày tiếp nhận theo Asia/Saigon. [Chuẩn bị cổng/cabling và xử lý save](sprint-02-po11-preflight-review-2026-10-16.md) giữ lịch sử riêng.

## 1. Actual channel và peer

- CORE1 có2 group/aggregator: Po1 SU/LACP/e0/0(P)/e0/3(P), Po11 SU/LACP/e0/1(P)/e1/1(D). ACCESS1 có1 group/aggregator, Po11 SU/LACP/e0/0(P)/e1/0(D). Hai member cũ đã bundled, member mới down phù hợp pha giữ shutdown; chưa có interface detail mới để xác nhận mọi thuộc tính vật lý.
- ACCESS1 e0/0 nhận partner CORE1 aabb.cc80.1000, flagsSA/priority32768, Oper Key0xB (=11), remote Port Number0x2, state0x3D. e1/0 down có Dev ID0/key0/state0; dòng SP không dùng để kết luận peer thực sự cấu hình passive hoặc có partner hợp lệ.
- CORE1 gõ `how lacp neighbor`, CLI báo Invalid input. Đây là lỗi cú pháp thiếu chữ s; chưa có bảng neighbor chiều CORE1 trong lượt này. Không kết luận LACP hỏng vì lỗi gõ, không yêu cầu gán group lại; thu đúng show lacp neighbor cùng checkpoint hai member.

## 2. Actual trunk, STP và HSRP

- Po11 hai đầu trunking dot1q/native99; allowed/active/forwarding-not-pruned đủ VLAN10,20,30,40,50,60,99. CORE1 còn Po1/e0/2 trunk; ACCESS1 còn e0/1 trunk riêng tới CORE2, forwarding-none như baseline.
- VLAN20: CORE1 vẫn root aabb.cc00.1000/priority24596; Po11 Desg/FWD/cost100, Po1 Desg/FWD/cost56. ACCESS1 nhận cùng root qua **Po11 Root/FWD/cost100**, thay root port vật lý e0/0 trước đó; e0/1 Altn/BLK và e0/2 Desg/FWD. STP đang nhìn channel như port logic. Link type Shr ghi theo S2-IOL-01, không ép P2p. Chưa có inconsistentports mới hoặc STP root đủ bảy VLAN trong lượt.
- CORE1 Active110/preempt đủ sáu nhóm HSRP, standby peer10.10.X.3/VIP10.10.X.1 đúng. Không có show standby CORE2 riêng mới; giữ phạm vi kết luận theo output CORE1.

## 3. Actual traffic và giới hạn checkpoint

PC-IT→VIP20.1 có5 phản hồi sequence1–5/TTL255, RTT0.465/0.961/0.632/0.757/0.994ms; dòng số5 có prefix ký tự ^[[A, vẫn giữ trong raw và đếm đúng một phản hồi. PC-IT→ACC40.20 có5 phản hồi sequence1–5/TTL63, RTT2.848/1.257/1.413/1.429/1.419ms. Ghi hai mẫu5/5, không nâng thành20/20 hoặc chứng minh chịu lỗi/băng thông.

Checkpoint một member đạt phần hình thành/trunk/STP/HSRP/traffic đã thử. S02-TC-04 chưa PASS toàn ca vì còn member mới/peer hai đầu/kiểm tra cuối/snapshot. Không ghi việc mở một member lúc cấu hình thành ca mất member đã thử. Chưa tạo config từ hướng dẫn/summary; snapshot Po1 cũ không đại diện CORE1 sau thêm Po11.

## 4. Bước tiếp theo đã chuẩn bị

Chỉ no shutdown CORE1 e1/1 và ACCESS1 e1/0; giữ member cũ, Po1, trunk CORE2 và cổng test CORE1 e1/0. Sau mở cả hai đầu thu summary, show lacp neighbor đúng cú pháp, trunk/STP VLAN20/inconsistentports, chi tiết hai cổng mới và HSRP CORE1; kiểm tra lại VIP/ACC từ PC-IT. Expected Po11 SU/hai P cùng partner ở mỗi đầu, Po1 vẫn hai P, runtime/traffic đúng, không tăng lỗi bất thường.

Khi các mục đó đạt, dùng wr trên CORE1/ACCESS1 và thu full running/startup thật để lưu checkpoint hai channel; không save nếu member suspended/standalone, traffic lỗi hoặc trạng thái không đạt. S2-IOL-01 giữ nguyên, không đòi Full/P2p/đổi image; HA và traffic qua Po1 còn riêng. Chưa có bằng chứng đã mở member mới hoặc đã lưu Po11 trong lượt một member này.

## 5. Checkpoint hai member — kết quả mới nhất16/10

**Nguồn:** [console hai member](sprint-02-po11-two-members-2026-10-16.txt), chép từ tin nhắn người thực hành, chuẩn hóa khoảng trắng cuối dòng và dòng trắng phân cách. Giữ nguyên thứ tự command/output; không có show clock/timestamp, wr hoặc full config trong lượt. Ngày là ngày tiếp nhận theo Asia/Saigon.

- CORE1 Po11 SU/LACP/e0/1(P)/e1/1(P); cả hai nhận ACCESS1 aabb.cc80.3000, Oper Key0xB, remote ports0x1/0x101, flagsSA/state0x3D. ACCESS1 Po11 SU/LACP/e0/0(P)/e1/0(P); cả hai nhận CORE1 aabb.cc80.1000/key0xB/remote ports0x2/0x102/SA/0x3D. Neighbor CORE1 đã thu đúng cú pháp, phần thiếu do how ở checkpoint trước đã được bổ sung. Peer ID là LACP System ID theo output, không thay bằng STP Bridge ID khác giá trị.
- CORE1 Po1 vẫn SU/LACP/e0/0(P)/e0/3(P), cùng peer CORE2 aabb.cc80.2000/key0x1/remote ports0x1/0x4/SA/0x3D. Chưa có output CORE2 mới riêng; không gán checkpoint CORE1 thành kiểm tra trực tiếp cả hai Core.
- Trunk Po11 hai đầu dot1q/native99, allowed/active/forwarding đủ VLAN10,20,30,40,50,60,99. ACCESS1 e0/1 vẫn trunk riêng/forwarding-none; CORE1 e0/2 và Po1 vẫn trunk đủ bảy VLAN.
- VLAN20: CORE1 root aabb.cc00.1000/priority24596; Po11 Desg/FWD/cost56, Po1 Desg/FWD/cost56. ACCESS1 nhận root qua Po11 Root/FWD/cost56, e0/1 Altn/BLK/cost100, e0/2 Desg/FWD/cost100. Po11 cost thực tế giảm100→56 so với một member, không đặt cost bằng tay. Inconsistentports bằng0 trên cả hai. Chưa thu root/role riêng đủ bảy VLAN; không nâng phạm vi VLAN20 thành mọi VLAN.
- CORE1 e1/1 description PO11_TO_ACCESS1_MEMBER2, ACCESS1 e1/0 PO11_TO_CORE1_MEMBER2; cả hai up/up/Auto-duplex/Auto-speed/BW10000Kbit. Các counter CRC/input/output errors/collisions/interface resets/drops hiển thị bằng0 tại thời điểm chụp. C1 RX28/TX517, A1 RX33/TX57; không có delta theo thời gian để kết luận không tăng lỗi lâu dài hoặc phân bố tải. S2-IOL-01 giữ nguyên; không ép Full/P2p.
- HSRP CORE1 Active110/preempt đủ sáu nhóm, standby peer10.10.X.3/VIP10.10.X.1. PC-IT→VIP20.1 có5/5 reply/TTL255, RTT0.497/0.892/0.575/0.719/0.636ms; PC-IT→ACC40.20 có5/5/TTL63, RTT2.698/1.415/1.945/1.348/1.507ms. Không ghi ping HR/reverse hoặc HA đã chạy trong lượt này.

**Kết luận:** phần hình thành Po11 hai member/peer/trunk/STP VLAN20/HSRP CORE1/traffic đã thử đạt. S02-TC-04 còn hồ sơ checkpoint và các kiểm tra phạm vi rộng hơn của kế hoạch; chưa đóng toàn ca hoặc Sprint2. Không dùng mẫu hai P này làm bằng chứng chịu lỗi hoặc throughput tăng gấp đôi.

**Tiếp theo:** trên CORE1/ACCESS1 dùng wr, chờ OK rồi thu full show running-config và show startup-config. Chỉ tạo CORE1.po1-po11.running.cfg/.startup.cfg và ACCESS1.po11.running.cfg/.startup.cfg từ nguồn thật, giữ các checkpoint cũ. Sau đối chiếu save/config và hoàn thiện baseline liên quan sẽ chuyển sang bài mất từng member/channel/Core. Chưa có nguồn để tạo bốn snapshot mới trong lượt này.

## 6. Checkpoint cấu hình đã lưu — tiếp nhận16/10

**Nguồn độc lập:** [console CORE1](sprint-02-po11-core1-config-2026-10-16.txt) và [console ACCESS1](sprint-02-po11-access1-config-2026-10-16.txt). Chép từ tin nhắn, chuẩn hóa khoảng trắng/comment trống/dòng trắng; giữ command lines, metadata và thứ tự output. Bản running ACCESS1 đầu bị cắt sau Po11, giữ trong console nguồn nhưng không dùng tạo snapshot; bản running thứ hai đủ đến end được trích độc lập với startup. Không ghép từ baseline hoặc sao running sang startup.

- CORE1: running báo3808byte; startup dùng1844/524288byte, uncompressed3808byte; last configuration change14:52:34 UTC16/10/2026 (=21:52:34 Asia/Saigon). [Running](../../configs/core/sprint-02/CORE1.po1-po11.running.cfg) và [startup](../../configs/core/sprint-02/CORE1.po1-po11.startup.cfg) cùng142 dòng lệnh sau loại comment/dòng trắng và khớp đúng thứ tự.
- ACCESS1: running báo2210byte; startup dùng1241/524288byte, uncompressed2210byte; last change14:52:57 UTC16/10/2026 (=21:52:57 Asia/Saigon). [Running](../../configs/access/sprint-02/ACCESS1.po11.running.cfg) và [startup](../../configs/access/sprint-02/ACCESS1.po11.startup.cfg) cùng87 dòng lệnh và khớp đúng thứ tự. Số byte nén là metadata thiết bị, không dùng kích thước file đã chuẩn hóa để suy ngược.
- So với CORE1.po1-only.running.cfg chỉ khác interface Port-channel11, Ethernet0/1, Ethernet1/1. Po1/e0/0/e0/3, trunk ACCESS2/e0/2, cổng test e1/0 shutdown/PortFast, STP priority24576, SVI/HSRP, no ip cef và VTY giữ nguyên.
- So với ACCESS1 running cuối Sprint1 chỉ khác Port-channel11, Ethernet0/0, Ethernet1/0. Ethernet0/1 tới CORE2 vẫn trunk riêng, không channel-group11; PC_IT/e0/2 VLAN20, PC_HR/e0/3 VLAN30, management10.10.10.11/default-gateway10.10.10.1/no ip routing và STP/VTY giữ nguyên.
- Po11 và cả hai member mỗi đầu cùng dot1q/native99/allowed10,20,30,40,50,60,99/nonegotiate; member channel-group11 mode active, không shutdown. Không tự thêm duplex full, ip routing, ACL hay tính năng khác vào nguồn cấu hình.

**Kết luận:** checkpoint persistence và bốn snapshot mới đã đủ, không yêu cầu wr/full dump lại. Không có dòng wr/OK trong nguồn, không tạo log save giả; việc startup chứa cùng cấu hình là bằng chứng persistence tại lúc thu. Timestamp last change không phải thời điểm save hoặc thời điểm chụp runtime phần5, đồng hồ thiết bị chưa đồng bộ nguồn chung cho phép đo HA.

**Bước tiếp theo:** thử một member Po11 mỗi lượt, bắt đầu shutdown riêng CORE1 e0/1 để giữ đường mới e1/1 ↔ ACCESS1 e1/0; ghi traffic/trạng thái trước–trong–sau và phục hồi đủ hai P trước lượt khác. Không wr trong trạng thái gây lỗi. Hai lượt mất member, mất channel/Core, traffic qua Po1, baseline phạm vi rộng và export/restore cuối vẫn chưa có kết quả; không ghi PASS cho chúng từ checkpoint này.
