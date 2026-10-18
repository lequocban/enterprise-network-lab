# Sprint02 — mất từng member Po1, 18/10/2026

**Kết quả:** lượt mất e0/0 duy trì traffic theo người thực hành; khi mở e0/0 có gián đoạn rồi hồi phục. Lượt mất e0/3 không tự hồi phục theo người thực hành, chưa đạt chức năng; chưa có nguồn xác nhận phục hồi e0/3 sau lượt này. Ghi gọn mốc quan trọng, không tạo transcript dài hoặc snapshot từ summary.

Trong lượtA, C1Po1SU/e0/0D/e0/3P; C2Po1SU/e0/0s/e0/3P. Trong lượtB, C1Po1SU/e0/0P/e0/3D; C2Po1SU/e0/0P/e0/3s. C1Po11 vẫnhaiP cảhai mẫu. C2VLAN10 nhận rootC1aabb.cc00.1000/priority24586 quaPo1RootFWD/cost100 trongcảhai lượt, e0/1AltnBLK/e0/2DesgFWD. Không gọiPo1 khỏe chức năng chỉ từSU/P hoặcSTP.

Chuỗi pingPC-IT→CORE2realSVI10.10.10.3 có replyseq1–146; timeout147–161 (15gói); reply162–187; timeout188–292 (105gói trong phần cuối, chưa córeplylại). Người thực hành gán đợt mất đầu với thao tác mở e0/0, đợt cuối với tắt e0/3. Không cótimestamp từnggói/mốc thao tác chung để suy giây hội tụ hay tách chính xác cácpha từsequence.

C2log08:56:29.463UTC18/10 e0/3line down,08:56:36.971 EC-5-L3DONTBNDL2 e0/3suspended/LACP currently not enabled on remote port. Thông báo nằm ở member bị chủ động shutdown phíaC1, cóthể phùhợp mấtLACP từpeer đó; không tự chứng minh thiếuchannel-group trong cấu hình, hoặc giải thích vì sao membercòn lại e0/0P không chuyển được traffic. SummaryS vẫn chỉPo1Layer2, không kết luậnLayer3 chỉ từ tênmessage.

**Tiếp theo tối thiểu:** ưu tiên no shutdownC1e0/3 để phục hồi haiP/traffic, khôngwr lỗi. Nếu đang còn pha e0/3tắt và chưarestore, có thể thu nhanh C1ping10.10.10.3 source10.10.10.2 repeat5 timeout1 và PC-ITping10.10.20.3 -c5 rồi mởlại ngay. Hai probe phân biệt đường cùngVLAN10 giữaCore với luồngPC/VLAN20, không đổihash/duplex/group hoặc restart để thử ngẫu nhiên. Nếuđãrestore thì khôngtắt lại chỉ để bổ sungprobe; gửi summary haiđầu/neighbor hiện tại trước. Không sang mất toànPo1/Core hoặc ghiPASS S02-TC-06 khi lượtB chưa khắcphục.

Tham khảo giải nghĩa thông báo: [Cisco system message guide](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3750x_3560x/software/release/15-0_1_se/system/message/smg_3750x/msg_desc.html). Output trênIOL và bối cảnh shutdown vẫn quyết định phạmvi kếtluận; rootcause chưa được xác minh.

## Cập nhật tiếp theo ngày 18/10/2026 — phục hồi MAC và kiểm tra HA

Phần này thay thế hướng dẫn bước tiếp theo ở checkpoint ban đầu; giữ nguyên kết quả lỗi lịch sử. Nguồn là output và xác nhận người thực hành gửi trong chat, không phải lệnh do agent chạy. Chỉ ghi mốc quan trọng theo ưu tiên phản hồi nhanh.

- Khi e0/3 tắt, C1 ping cùng VLAN10 sang C2 có lượt 0/5; sau mở lại e0/3, hai Core đủ hai member P, ping VLAN10 đạt 5/5 và ARP hai chiều hợp lệ. Người thực hành xác nhận mẫu PC tới 10.10.20.3 timeout thuộc lúc e0/3 còn tắt.
- Thử tắt e0/3 cả hai đầu: Po1 SU, e0/0 P/e0/3 D cả hai Core. C1 ping VLAN10 đạt 5/5 ở lượt này, nhưng PC tới C2 VLAN20 chỉ hồi phục sau mở lại e0/3. Chuỗi PC timeout seq7–57 (51 gói). Không suy số giây từ sequence.
- Trong lượt khoanh vùng VLAN20, C1 ping 10.10.20.3 source 10.10.20.2 đạt 0/5; C1 Po1 Desg FWD cost100, C2 Po1 Root FWD cost100. C1 học MAC C2 aabb.cc80.2000 qua Po11 và MAC của chính C1 aabb.cc80.1000 qua Et0/1; C2 học PC 0050.7966.6805 qua Po1, ARP 10.10.20.20 hợp lệ.
- Output được gửi sau `clear mac address-table dynamic vlan 20` trên C1: ping cùng VLAN20 đạt 4/5, MAC C2 chuyển về Po1; PC tới C2 VLAN20 đạt 7/7. Đây là phục hồi sau thao tác làm mới MAC, chưa phải sửa tận gốc hoặc chứng minh image lỗi. Không có output xác nhận đã clear trên C2 nên không ghi thao tác đó là đã thực hiện.
- ACCESS1 sau đó có Po11 Root FWD/cost56, Et0/1 Desg FWD; MAC C2 qua Po11, PC qua Et0/2. MAC C2 qua Po11 là phù hợp trên ACCESS1, khác với mục cùng MAC trên C1. Mẫu CORE2 trước đó có Et0/1 Altn BLK; không kết luận vòng lặp chỉ từ bảng MAC.
- Đã khôi phục hai member P của Po1 cả hai Core và Po11 trên C1; C2 Po1 Root FWD cost56; PC tới C2 VLAN20 và ACC40 đều 5/5. Người thực hành xác nhận đã `wr` trước chuyển sang bài mất toàn Po1. TC-06 chưa đạt đầy đủ vì lượt lỗi cần can thiệp thủ công, nguyên nhân học MAC sai chưa rõ.

### Mất toàn Po1 — xác nhận chức năng, chưa đo thời gian

C1 shutdown Port-channel1 lúc 09:42:30.723 UTC theo log thiết bị: Po1 SD/hai member D, Po11 SU/hai P. C2 VLAN20 nhận root C1 qua Et0/1 Root FWD/cost156; Et0/2 Altn BLK. ACCESS1 Po11 Root FWD/cost56, Et0/1 Desg FWD. C2 vẫn Standby cả sáu nhóm HSRP, nhận C1 Active.

PC-IT tới 10.10.20.3 timeout seq9–26, 30, 32–34, 37–40, 43–45, 48–49; reply liên tục seq50–101 (52 gói). Người thực hành xác nhận đây là kết quả khi chưa khôi phục Po1. Chuyển đường dự phòng VLAN20 có bằng chứng chức năng; thiếu timestamp chung/chuỗi ổn định đo theo thời gian và 20/20 sau ổn định nên chưa nghiệm thu tiêu chí đo HA đầy đủ.

Sau mở lại Po1: C1 Po1/Po11 đều SU/hai P, C2 Po1 Root FWD cost56. PC tới C2 VLAN20 đạt 3/3, tới ACC40 đạt 5/5.

### Mất toàn CORE1 và bật lại — lượt chức năng đầu tiên

Trong pha CORE1 tắt theo trình tự người thực hành báo: C2 Active cả sáu nhóm HSRP (Standby unknown), là STP Root VLAN20; ACCESS1 Et0/1 Root FWD/cost100. PC-IT tới ACC40 reply seq1–15, timeout16–33 (18 gói), reply34–78 (45 gói liên tục). Output này được gửi trước phần ghi rõ “sau khi start core 1”, xác nhận phục hồi traffic trong pha CORE1 còn tắt.

Sau Start CORE1: C1 Active/priority110 và C2 Standby/priority100 cả sáu nhóm; C1 Po1/Po11 SU/hai P, C2 Po1 SU/hai P. PC-IT tới ACC40 đạt 5/5. Đã kiểm chứng chức năng failover và preempt trở lại cho luồng IT20 → ACC40, không suy chính xác 18 giây hội tụ từ 18 timeout.

Chưa đóng Sprint2/NET-201/NET-202: TC-06 còn lỗi học MAC chưa rõ nguyên nhân; lượt full Core mới có luồng IT→ACC, chưa đủ hai chiều/sáu VLAN/ba lượt, phép đo thời gian, kiểm tra mất CORE2 và hồ sơ restore cuối theo kế hoạch. Các snapshot cấu hình cũ giữ nguyên; không tạo cấu hình mới từ summary.

## Checkpoint cuối ngày 18/10/2026 — lỗi Po1 tái diễn

Người thực hành chọn tiêu chí kiểm tra đường dự phòng với ít nhất một VLAN đại diện cho mỗi đường, dừng kiểm thử lặp từng VLAN để tiết kiệm thời gian. Không yêu cầu chạy thêm lượt CORE1 off riêng cho VLAN60. Điều này thu hẹp phần kiểm thử tiếp theo; không biến các phần chưa đo thành PASS.

Các kết quả bổ sung sau checkpoint đầu: HR30↔client VLAN50 tạm có baseline hai chiều 5/5; khi CORE1 tắt, C2 Active sáu nhóm/root VLAN50, HR timeout seq12–29 (18 gói) rồi reply30–50, chiều ngược 5/5; sau Start C1 Active/C2 Standby, HR→VLAN50 5/5. IT20↔client VLAN10 tạm có baseline hai chiều 5/5; pha CORE1 tắt, C2 Active sáu nhóm/root VLAN10, IT timeout10–28 (19 gói), reply29–57, chiều ngược 5/5; sau Start vai trò đúng và IT→VLAN10 5/5. Người thực hành sau đó xác nhận đã hoàn nguyên endpoint theo hướng dẫn (ACC về VLAN40, GUEST về VLAN60); không có output chi tiết của thao tác lưu/hoàn nguyên trong xác nhận đó.

Mất CORE2: C1 vẫn Active sáu nhóm/Standby unknown, Po1 SD/e0/0 s/e0/3 s, Po11 SU/hai P. IT→ACC reply seq1–126, không timeout trong mẫu; GUEST→IT 5/5. Sau Start CORE2, HSRP C1 Active/C2 Standby, Po1 hai P cả hai Core, Po11 hai P trên C1; IT→ACC 5/5. Baseline IT20↔GUEST60 sau đó có hai chiều 5/5; chưa kiểm chứng VLAN60 forwarding khi C2 Active.

Lượt tái kiểm Po1 cuối: C1 shutdown e0/3, log admin down 10:37:13.537 UTC/line down10:37:14.542; Po1 SU/e0/0 P/e0/3 D, Po11 SU/hai P. PC-IT→10.10.20.3 reply seq1–16, timeout17–65 (49 gói), reply66–78. C1 mở lại e0/3, log link up10:38:48.246/line up10:38:49.246. Người thực hành xác nhận ping chỉ hồi phục khi mở lại e0/3 và Po1 trở lại hai P. Không có timestamp từng gói để suy thời gian gián đoạn chính xác; không có bảng MAC trong pha lỗi của lượt này nên chưa khẳng định cùng nguyên nhân với lượt clear MAC.

**TC-06 vẫn chưa đạt chức năng chuyển tiếp tự động khi chỉ còn e0/0.** Dừng kiểm thử diện rộng; bước tiếp theo là khoanh vùng nguyên nhân Po1 bằng bằng chứng chuyển tiếp/frame, giữ cấu hình ổn định giữa các lần kiểm tra. Không ghi image lỗi, không tự đóng ngoại lệ học MAC và không cần chạy lại các VLAN đã phủ đường đi.

## Phân tích ba capture Po1 ngày 18/10/2026

Nguồn do người thực hành cung cấp, tên cổng theo ánh xạ người thực hành khi capture trên CORE1:

- `captures/sprint-2/capture_e00.pcapng`: 2905 frame, 257 ICMP Echo Request VLAN20, không có Echo Reply; 30 LACP.
- `captures/sprint-2/capture_e01.pcapng`: 3316 frame, 262 Echo Request và 118 Echo Reply của cùng luồng PC-IT 10.10.20.20 tới C2 10.10.20.3; 29 LACP.
- `captures/sprint-2/capture_e11.pcapng`: 42 frame, có 28 LACP, không có ICMP/ARP trong khoảng ghi nhận. Một luồng không xuất hiện trên member này không chứng minh member hỏng.

Đọc trực tiếp block pcapng/khung Ethernet, 802.1Q, IPv4, ICMP, ARP, LACP và PVST bằng Python standard library; chưa có tshark trong PATH hoặc đường cài đặt thông thường. Đã kiểm tra độ dài/trailer block và checksum IPv4/ICMP: không thấy lỗi checksum trong 638 frame ICMP của hai file đầu. Cả ba capture không có EPB direction flags; không dùng một frame đơn lẻ để khẳng định hướng ingress/egress hoặc đã đến CPU CORE2.

Echo Reply trên e0/1 có seq1–48, sau đó seq148–217; không có reply seq49–147 trên cả ba capture. Request seq49–146 vẫn xuất hiện trên e0/0 và e0/1. Ví dụ cùng request seq49 byte-identical ở e0/1 frame1180 và e0/0 frame1023, Ethernet src0050.7966.6805/dst aabb.cc80.2000/VLAN20/TTL64. Seq148 có reply ở e0/1 frame2603. Đây là quan sát theo sequence capture, không gán chính xác mốc shutdown/no shutdown từ log của lượt khác.

Trong đoạn seq102–146, mỗi request có hai bản byte-identical trên mỗi file e0/0 và e0/1, cách nhau dưới 1ms. Là dấu hiệu cần kiểm tra khả năng frame quay lại/nhân bản; chưa kết luận loop hoặc nguyên nhân do image vì thiếu capture đầu CORE2 và thông tin hướng gói.

LACP e0/0 vẫn trao đổi hai chiều theo MAC nguồn C1/C2, actor/partner state0x3d trong các gói đã ghi. Frame điều khiển hoạt động không chứng minh ICMP được xử lý/trả lời. e0/0 có ARP PC hỏi C2 và C2 trả lời lúc đầu capture; cuối có ARP C1 hỏi C2 và reply từ C2. Không có cơ sở kết luận đường e0/0 hoàn toàn không truyền frame.

Output runtime gửi cùng capture: C1 Po1 SU/e0/0 P/e0/3 D; Po11 SU/hai P. C1 MAC PC qua Po11, MAC C2 qua Po1 (đúng đường trong mẫu này); `show ip arp 10.10.20.3` trên C1 không có mục. C2 Po1 SU/e0/0 P/e0/3 s, Po1 Root FWD/cost100, Et0/1 Altn BLK/Et0/2 Desg FWD. Lần này không xác nhận lại hiện tượng MAC C2 học sai Po11 trên C1. ARP trống trên C1 không tự chứng minh lỗi, vì PC và C2 cùng subnet/VLAN20 và C1 đang chuyển frame Layer2 cho luồng này.

**Khoanh vùng tiếp theo:** kiểm tra phía CORE2 xử lý request và đường phản hồi, gồm ARP/MAC của PC trong pha lỗi và ping C2→PC với source10.10.20.3; nếu cần capture C2 e0/0 và e0/3 để xác định request đến đầu nào/reply xuất hiện trên member nào. Chưa đủ bằng chứng phân biệt drop trước CPU, không tạo reply, gửi reply sai member hoặc lỗi mô phỏng. Giữ root cause OPEN, không xóa MAC/ARP hoặc đổi hash trong cùng phép quan sát.

## Capture phía CORE2 — chiều trả lời và dấu hiệu lặp frame

Trước lượt capture này, output bổ sung của C2 có ARP 10.10.20.20→0050.7966.6805 hợp lệ, MAC PC qua Po1, nhưng ping source10.10.20.3 tới PC đạt 0/5. Đủ để loại trừ thiếu ARP/mục MAC chọn sai channel trong snapshot đó; chưa loại trừ lỗi chọn member hoặc chuyển tiếp thực tế.

Nguồn mới do người thực hành cung cấp:

- `captures/sprint-2/capture_e00_core2.pcapng`: 17546 frame; SHA256 `413605895033fbe37436741d1e4266f16d7761d31450ee565236ee2cba29410e`; timestamp nhúng 14:27:35.033–14:32:11.943 UTC ngày18/10/2026.
- `captures/sprint-2/capture_e03_core2.pcapng`: 791 frame; SHA256 `b4344437a693756319191c17ce12986bd5dfedb42ba0834b1dae48056b82a35e`; timestamp nhúng 14:28:04.973–14:32:13.824 UTC. Không dùng timestamp này để đồng bộ với log thiết bị của lượt khác.

Đọc trực tiếp pcapng/Ethernet/VLAN/IPv4/ICMP/LACP bằng Python standard library như lượt trước. Không có lỗi checksum IPv4/ICMP trong hai file. Không có EPB direction flags. Tên interface theo ánh xạ người thực hành, không phải interface name đã xác nhận từ IDB.

1. C2 e0/0 có 114 Echo Request PC20.20→C2 20.3. C2 e0/3 có 114 Echo Reply chiều ngược, khớp tuple IP/identifier/sequence cần đối chiếu theo từng đợt ping (sequence có reset nên không nhóm chỉ bằng seq). Có bằng chứng C2 tạo reply; không tiếp tục mô tả toàn bộ lỗi là C2 không nhận request/không tạo reply.
2. Đúng năm Echo Request C2 20.3→PC20.20, identifier4/seq0–4, chỉ thấy ở e0/3, frame406/412/414/418/422, timestamp14:30:19.022–14:30:23.029 UTC. Không thấy các request này trên e0/0. Đây là lựa chọn member e0/3 quan sát được, chưa chứng minh lựa chọn member đang suspended tại thời điểm ping.
3. Output summary gửi kèm chính lượt này là C2 Po1 SU/e0/0 P/e0/3 P, khác mẫu e0/3 s trước. Trên capture e0/3, C2 có actor state0x7d/partner zero lúc14:28:19.472–14:29:16.454; có trao đổi trở lại, C2 actor/partner state0x3d/0x3d lúc14:29:45.920 và14:30:13.819, trước năm request do C2 phát. Có biến động tiếp lúc14:30:41.424 rồi trao đổi trở lại. Cần xác nhận thao tác shutdown/no shutdown CORE1 e0/3 tương ứng; không gán cả capture thành một pha lỗi đơn nhất.
4. Bất thường lớn trên e0/0: 15525 Echo Reply nhưng chỉ có8 nội dung frame khác nhau. Cùng một reply seq53/IPID0xd80b xuất hiện11730 lần ở frame1524–13267, timestamp14:31:15.476–14:31:16.206 (0.730 giây theo capture), byte-identical/TTL255. Các reply seq59/63/64 cũng lặp hàng trăm tới hàng nghìn lần. Đây là dấu hiệu lặp/nhân bản frame cần ưu tiên điều tra; chưa xác định vòng vật lý, phản xạ nội bộ IOL hay duplication từ cơ chế capture.

**Kết luận tạm thời:** đã thấy C2 gửi ICMP qua e0/3 và có lượng frame lặp bất thường trên e0/0. Chưa đủ bằng chứng kết luận C2 cố gửi qua member đã down/suspended vì summary và LACP tại mốc ping cho thấy hai member có thể đã tái tham gia. Đã hỏi người thực hành: khi chạy ping5gói trên C2, CORE1 e0/3 còn shutdown hay đã no shutdown? Chờ xác nhận để chọn phép cô lập tiếp theo; chưa đổi hash, timer, STP hoặc image.

**Xác nhận bổ sung của người thực hành:** khi chạy năm gói ping trên C2 trong lượt capture này, CORE1 e0/3 vẫn đang shutdown. Kết hợp xác nhận này với năm request xuất hiện trên C2 e0/3 và summary hai P: C2 chọn member có đầu xa được người thực hành báo đã tắt. Đây là bất đồng trạng thái đầu xa/control plane/data plane cần xử lý, chưa định danh lỗi image hoặc bug cụ thể. Capture vẫn có các đợt LACP trao đổi lại trên e0/3 nên giữ nguyên giới hạn về việc chưa đồng bộ chính xác thao tác console với timestamp capture. Phép kiểm tra tiếp theo: trong khi giữ C1 e0/3 shutdown, shutdown C2 e0/3 để loại member ở cả hai đầu, xác nhận chỉ e0/0 P rồi kiểm tra C2→PC/PC→C2, không clear MAC/ARP; sau đó hoàn nguyên e0/3 hai Core, không lưu trạng thái thử.

### Kết quả cô lập member hai đầu

Output người thực hành gửi tiếp: C1 và C2 đều Po1 SU/e0/0 P/e0/3 D; C1 Po11 SU/hai P. C2 ping10.10.20.20 source10.10.20.3 đạt5/5; PC-IT ping10.10.20.3 đạt5/5. Trong output mới không có clear MAC/ARP; người thực hành làm theo phép cô lập vừa hướng dẫn, không có bằng chứng về thay đổi cấu hình khác.

Đã chứng minh khả năng forwarding của đường e0/0 và Po1 một member ở trạng thái loại e0/3 cả hai đầu. Kết hợp lượt trước có request C2 gửi qua e0/3 trong khi người thực hành xác nhận C1 e0/3 shutdown: lỗi tập trung vào việc nhận biết/loại member hoặc cập nhật lựa chọn member khi chỉ tắt một đầu. Chưa phân biệt được cơ chế LACP, cập nhật data plane hay hành vi IOL/EVE; không định danh bug hoặc gọi đây là bản sửa tận gốc. TC-06 vẫn chưa đạt chuyển dự phòng tự động; phép shutdown hai đầu là cách cô lập/phục hồi đã kiểm chứng, không thay thế bài mất link một đầu.

Trạng thái cuối thực sự được gửi là e0/3 D cả hai Core; chưa nhận output xác nhận hoàn nguyên e0/3 sau phép kiểm tra này. Cần no shutdown e0/3 cả hai Core, xác nhận hai P rồi mới tiếp tục hoặc lưu cấu hình ổn định. Không lưu shutdown thử vào startup.

### Bổ sung trình tự khôi phục và giả thuyết môi trường/image

Người thực hành xác nhận tiếp: khi e0/3 cả hai Core đều shutdown, ping vẫn thành công; mở riêng e0/3 trên CORE1 trong khi đầu CORE2 còn shutdown, ping vẫn thành công; sau đó mở e0/3 cả hai Core, ping vẫn thành công. Xác nhận này cập nhật trạng thái hoàn nguyên của checkpoint trước. Chưa có summary mới xác nhận hai P hoặc số gói ping từng pha nên không tạo các output/số liệu đó từ lời xác nhận.

Trình tự này cho thấy kết quả có thể phụ thuộc trạng thái trước đó/thứ tự thao tác: khi C2 đã chủ động loại e0/3, traffic qua e0/0 vẫn được duy trì trong quá trình mở lại đường e0/3. Không coi việc mở một đầu thành công là tương đương với tự phục hồi khi tắt một đầu từ trạng thái hai member đang hoạt động.

Theo yêu cầu người thực hành, bổ sung hai giả thuyết đang mở:

- **Môi trường lab EVE-NG không ổn định:** trạng thái liên kết ảo, kết nối giữa node, lập lịch/thời gian chạy VM hoặc cơ chế capture có thể không phản ánh nhất quán các thay đổi. Bối cảnh có ACCESS1 từng treo trong các buổi trước, nhưng lần ACCESS1 bị off do người thực hành xác nhận gần đây không được dùng làm bằng chứng EVE tự lỗi. Chưa có log EVE/hypervisor hoặc phép đối chứng để xác nhận giả thuyết.
- **Image IOL không ổn định hoặc có lỗi:** việc cập nhật trạng thái LACP, lựa chọn member hoặc chuyển tiếp có thể phụ thuộc trạng thái nội bộ sau chuỗi shutdown/no shutdown. Các quan sát về gửi traffic qua e0/3 có đầu xa được báo shutdown, trạng thái hai đầu không nhất quán và frame lặp là cơ sở đặt giả thuyết, chưa chứng minh bug cụ thể. Chưa có phép đối chứng với image khác hoặc cùng image trong môi trường khác.

Hai giả thuyết có thể liên quan nhau và cùng tồn tại với khả năng lỗi cấu hình/trạng thái chuyển tiếp; hiện chưa đủ dữ liệu để quy trách nhiệm cho một lớp cụ thể. Giữ nguyên cấu hình và bằng chứng capture hiện có; không đánh dấu root cause đã xác định hoặc TC-06 tự phục hồi đã đạt chỉ từ các pha khôi phục thành công.
