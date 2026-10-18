# Sprint 02 — S2-LACP-02: lượt A chưa đạt, đã xác nhận phục hồi Po11

**Trạng thái cập nhật17/10:** Po11 đã phục hồi hai P hai đầu. Người thực hành xác nhận đã trả ACCESS1 về src-dst-ip trước lượt tắt old e0/1; lượt tắt new tiếp theo cũng có src-dst-ip và log67/67 reply không timeout. Ghi nhận lỗi cũ không tái hiện trong hai lượt được báo; không quy kết đổi hashMAC đã sửa lỗi. Root cause và các yêu cầu đoHA đầy đủ chưa xác minh, lịch sử113timeout giữ nguyên; không tự đóng toànS02-TC-07/Sprint2.

**Nguồn:** tin nhắn người thực hành tiếp nhận16/10/2026, Asia/Saigon; các khối dưới trích đúng phần liên quan từ console, lược bỏ legend và phần không cần lặp. Hai khối CORE1 sau no shutdown trong tin nhắn giống nhau cả output/age/syslog, chỉ tính một quan sát, không coi là hai lần lấy mẫu độc lập. Không có output ping/baseline20/20/continuous/help ping/CORE2 trong lượt này.

## 1. Trong lỗi

CORE1:

```text
CORE1#show clock
*15:17:07.318 UTC Fri Oct 16 2026
CORE1#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
CORE1(config)#interface Ethernet0/1
CORE1(config-if)# shutdown
CORE1(config-if)#end
CORE1#show clock
*15:17:07.318 UTC Fri Oct 16 2026
```

Summary trích:

```text
1      Po1(SU)         LACP      Et0/0(P)    Et0/3(P)
11     Po11(SU)        LACP      Et0/1(D)    Et1/1(P)
```

STP VLAN20 CORE1 vẫn root aabb.cc00.1000/priority24596; Po11 Desg/FWD/cost100, Po1 Desg/FWD/cost56. CORE1 Active110/preempt cả sáu nhóm, standby peer10.10.X.3/VIP10.10.X.1. HSRP giữ role không chứng minh client forwarding đạt.

ACCESS1:

```text
ACCESS1#show clock
*15:17:42.686 UTC Fri Oct 16 2026
```

Summary/neighbor trích:

```text
11     Po11(SU)        LACP      Et0/0(P)    Et1/0(P)
Et0/0     SA      32768     aabb.cc80.1000  59s    0x0    0xB    0x2     0x3D
Et1/0     SA      32768     aabb.cc80.1000  23s    0x0    0xB    0x102   0x3D
```

STP VLAN20 ACCESS1 Po11 Root/FWD/cost56, e0/1 Altn/BLK/cost100; inconsistentports0. Hai đầu báo khác số member/cost trong các mẫu đã gửi. Thiếu trạng thái vật lý ACCESS1 e0/0 và mẫu nối tiếp để biết đây là chậm phát hiện lỗi hay trạng thái lỗi kéo dài. Đồng hồ CORE1 trước/sau shutdown trùng đến millisecond; dấu * và offset hai node chưa được xác minh, không dùng chênh hai clock làm thời gian hội tụ.

## 2. Sau mở lại member cũ

CORE1 đã nhập no shutdown e0/1. Summary/neighbor Po11 trích:

```text
11     Po11(SU)        LACP      Et0/1(w)    Et1/1(P)
Et0/1     FP      32768     aabb.cc80.3000 262s    0x0    0xB    0x1     0x36
Et1/1     SA      32768     aabb.cc80.3000   5s    0x0    0xB    0x101   0x3D
```

Po1 vẫn hai P/peer CORE2; CORE1 HSRP vẫn Active đủ sáu nhóm. Log nhận:

```text
*Oct 16 15:20:58.125: %SYS-5-CONFIG_I: Configured from console by console
*Oct 16 15:21:00.130: %LINK-3-UPDOWN: Interface Ethernet0/1, changed state to up
*Oct 16 15:21:01.134: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/1, changed state to up
```

w nghĩa waiting to be aggregated theo legend thiết bị; up/up chưa đủ để kết luận member bundled. Neighbor age262s là thông tin cũ, không kết luận ACCESS1 thực sự đổi mode passive từ FP ở mẫu này. Một lần lấy mẫu sát lúc no shutdown chưa chứng minh w bị kẹt lâu dài. Chưa có summary/neighbor ACCESS1 sau restore, ping sau restore hoặc xác nhận đủ hai P.

## 3. Giả thuyết và bước phân biệt

LACP slow có thể giữ thông tin peer đến timeout dài nếu không nhận physical link-down; Cisco mô tả slow periodic30s/long timeout90s tại [Troubleshoot EtherChannels](https://www.cisco.com/c/en/us/support/docs/switches/catalyst-9300-series-switches/220367-troubleshoot-etherchannels-on-catalyst-9.html). Đây chỉ là cơ chế tham khảo, không chứng minh nguyên nhân trên IOL15.2 hoặc bảo đảm cứ chờ90s sẽ hết lỗi. Chưa xác minh cổng vật lý remote, chiều truyền đường còn lại, trạng thái aggregator hoặc độ đáp ứng console/VM. Không gán nguyên nhân treo ACCESS1/image/duplex từ lịch sử S2-FWD-01/S2-IOL-01.

Giữ cấu hình vừa no shutdown và startup checkpoint ổn định. Thu mẫu mới trên cả hai đầu: show clock, show etherchannel summary, show lacp neighbor, show etherchannel 11 detail, chi tiết member cũ và mới; PC-IT ping VIP20.1 và ACC40.20 mỗi mẫu5. Nếu lệnh detail không hỗ trợ, giữ lỗi và các output còn lại. Nếu vẫn w, lấy lại clock/summary/neighbor sau khoảng15s để phân biệt trạng thái chờ tạm với tiếp tục không ghép; không coi hai đoạn dán giống nhau là hai mốc đo.

Mục tiêu trước tiên là xác nhận phục hồi hai P và traffic hiện tại; sau đó mới quyết định xử lý LACP/cổng/VM. Không wr trạng thái lỗi, không gỡ group/ép mode on/đổi timer/duplex hoặc shutdown thêm member đang sống để thử ngẫu nhiên. Bốn snapshot ổn định đã lưu trước bài lỗi giữ nguyên; lỗi chức năng không thuộc ngoại lệ Auto/Shr.

## 4. Phục hồi ACCESS1 và traffic — mẫu mới15:28:32UTC

**Nguồn:** [console ACCESS1 nguyên tệp](sprint-02-po11-member-a-recovery-access1-2026-10-16.txt), sao nguyên byte từ attachment đầu; hai attachment được so sánh byte và SHA256, nội dung trùng nhau nên chỉ lưu một bản, không coi là hai thời điểm hoặc CORE1/ACCESS1. SHA256 `c8e811e6fe7ceb8b8eb91469b25cc5a36c4adf3e637b3f0430b289cd774d0403`. [Ping PC-IT](sprint-02-po11-member-a-recovery-ping-2026-10-16.txt) chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng.

- ACCESS1 clock *15:28:32.018UTC16/10/2026. Po11 SU/LACP/e0/0(P)/e1/0(P); detail Ports2/Number of ports2, Ag-Inuse, hai member Up Mstr Assoc In-Bndl/local mode Active/bndl/key0xB/state0x3D. Minimum Links0 phía ACCESS1; không có yêu cầu min-links2 ở node này, không suy ra cấu hình CORE1 chưa thu.
- Neighbor cả hai cùng CORE1 aabb.cc80.1000/Oper Key0xB/SA/state0x3D; e0/0 age11s/remotePort0x2, e1/0 age7s/remotePort0x102. Thông tin peer mới đã cập nhật, khác mẫu age262s trước đó.
- Detail ghi last bundled e0/0 cách7m30s, last unbundled e0/0 cách10m18s. Đây là bằng chứng e0/0 có sự kiện rời/gia nhập lại bundle; hiệu hai age là2m48s giữa hai chuyển trạng thái trên ACCESS1. Không coi khoảng này là thời gian gián đoạn ping hoặc thời gian phát hiện lỗi, không ghép với clock CORE1 để đo chính xác vì chưa có log traffic/timestamp chung.
- Cả e0/0 và e1/0 up/up; CRC/input/output errors/collisions/interface resets/drops hiển thị0. e0/0 RX32804/TX1350, e1/0 RX2285/TX160; đây là counter tích lũy, không suy ra luồng ping dùng member nào hoặc phân bố tải chỉ từ giá trị này. Auto/Shr ngoại lệ giữ nguyên; mẫu này không có STP mới.
- PC-IT→10.10.40.20 có5 reply sequence1–5/TTL63/RTT2.367,1.100,1.750,1.304,1.281ms. Xác nhận liên VLAN IT→ACC đã phục hồi ở lúc ping, không suy ra chiều ngược/VIP riêng/60giây ổn định hoặc forwarding khi chỉ còn một member.

**Bước tiếp theo tối thiểu:** chỉ thu show clock/show etherchannel summary/show lacp neighbor mới ở CORE1. Không yêu cầu lặp full config, ACCESS1 detail, hoặc ping IT→ACC đã đủ5/5. Nếu C1 Po11 hai P đúng peer thì xác nhận restore đủ hai đầu; nếu còn w/D/suspended lấy detail/cổng để xử lý tiếp. Không wr vì startup ổn định đã lưu; chưa lặp bài lỗi trước khi xác nhận restore. Nguyên nhân sự cố lượt A vẫn chưa được kết luận là timeout/image/duplex/treo Access.

## 5. CORE1 phục hồi đủ hai member — mẫu15:34:22UTC

**Nguồn:** [console CORE1](sprint-02-po11-member-a-recovery-core1-2026-10-16.txt) chép từ tin nhắn, chỉ chuẩn hóa khoảng trắng cuối dòng. Clock *15:34:22.446UTC16/10/2026. Po11 SU/LACP/e0/1(P)/e1/1(P), cùng peer ACCESS1 aabb.cc80.3000/OperKey0xB/SA/state0x3D; age10s cả hai, remote ports0x1/0x101. Po1 SU/e0/0(P)/e0/3(P), peer CORE2 aabb.cc80.2000/key0x1/SA/state0x3D, age24s/1s.

Đã hết trạng thái w trong mẫu hiện tại. Restore hai đầu và IT→ACC sau restore đã có bằng chứng, không yêu cầu no shutdown/wr/thu full config lại. Không suy ra phục hồi đúng thời hạn từ các clock khác thời điểm hoặc từ last-bundled age. Nguyên nhân mất traffic khi chỉ còn member mới vẫn chưa xác minh; do đó S2-LACP-02 còn OPEN về khả năng chịu lỗi, chưa PASS S02-TC-07.

**Tiếp theo đã chuẩn bị:** một lượt A chẩn đoán có giới hạn, giữ ping liên tục/timestamp máy, chỉ shutdown C1e0/1; thu trạng thái vật lý member cũ và member còn lại/summary/neighbor ACCESS1 lúc khoảng10s và100s trong khi C1 vẫn shutdown. Nhằm phân biệt remote chưa phát hiện lỗi với đã loại member nhưng forwarding qua đường còn lại không đạt. Mở lại C1e0/1 sau mẫu100s hoặc sớm hơn nếu console bất thường; không để quá120s, không làm lượt B, không đổi timer trước bằng chứng. Đây là kế hoạch chưa thực thi, không tự ghi PASS hoặc mặc định nguyên nhân LACP slow/image.

## 6. Lượt A tái hiện — member cũ đã suspended nhưng ping vẫn mất

**Nguồn ping:** [nguyên attachment](sprint-02-po11-member-a-retest-ping-2026-10-16.txt), sao nguyên byte, SHA256 `b9dabe90a5f5be46dad33306c23af209300f21ee0d778f4f153117bd6f289fd7`. Nguồn switch: tin nhắn người thực hành, các trích đoạn dưới lược bỏ legend/chi tiết lặp; không có full dump hoặc show STP trong lượt. Người thực hành xác nhận ping bình thường trước shutdown, timeout suốt đến no shutdown mới trở lại. Không có command/log no shutdown mới; mẫu A1 hai P cuối được phân loại sau restore theo lời xác nhận, không gán là mẫu100s khi C1 vẫn shutdown.

CORE1 shutdown riêng Ethernet0/1, log config15:41:48.846UTC, summary:

```text
1      Po1(SU)         LACP      Et0/0(P)    Et0/3(P)
11     Po11(SU)        LACP      Et0/1(D)    Et1/1(P)
```

ACCESS1 mẫu trong lỗi:

```text
ACCESS1#show clock
*15:42:33.315 UTC Fri Oct 16 2026
11     Po11(SU)        LACP      Et0/0(s)    Et1/0(P)
Et0/0     SP      0         0000.0000.0000 114s    0x0    0x0    0x0     0x0
Et1/0     SA      32768     aabb.cc80.1000   5s    0x0    0xB    0x102   0x3D
Ethernet0/0 is up, line protocol is down (suspended)
Ethernet1/0 is up, line protocol is up (connected)
```

e0/0 RX38871/TX1500, broadcast5213, last input1m44s; e1/0 RX3189/TX207, broadcast179. CRC/input/output errors/collisions/interface resets/drops hiển thị0 cả hai. s là suspended theo legend: member cũ không còn bundled, physical up không đồng nghĩa được phép forwarding. Dòng SP/DevID0 không chứng minh peer được cấu hình passive; partner của member mới vẫn CORE1/key0xB/state0x3D.

ACCESS1 mẫu sau:

```text
ACCESS1#show clock
*15:44:08.153 UTC Fri Oct 16 2026
11     Po11(SU)        LACP      Et0/0(P)    Et1/0(P)
Et0/0     SA      32768     aabb.cc80.1000   2s    0x0    0xB    0x2     0x3D
Et1/0     SA      32768     aabb.cc80.1000  15s    0x0    0xB    0x102   0x3D
Ethernet0/0 is up, line protocol is up (connected)
Ethernet1/0 is up, line protocol is up (connected)
```

e0/0 RX38945/TX1583/broadcast5260, e1/0 RX3893/TX212/broadcast184; counter lỗi vẫn0. Delta giữa hai mẫu e0/0 RX+74/TX+83, e1/0 RX+704/TX+5, nhưng cửa sổ bao gồm restore, không gán toàn delta cho pha một member, không xác định loại frame/luồng ping chỉ từ counter tổng. Không cần clear counter hoặc kết luận link unidirectional từ chênh này.

**Ping thực đếm:** seq1–35 có35 reply; seq36–148 có113 timeout liên tiếp; seq149–169 có21 reply. Tổng169 kết quả hoàn tất,56 reply/113 timeout, không có seq bị thiếu trong log. TTL63 ở các reply, dùng -t/-i1000/-w1000. Không có timestamp từng gói hoặc mốc no shutdown chung nên không đổi113timeout thành113giây gián đoạn, không từ clock khác node suy ra thời gian phát hiện/hội tụ. Không có mẫu chiều ngược/VIP/real SVI trong lỗi.

**Kết luận:** S02-TC-07 lượt A FAIL chức năng theo sự kiện + log traffic: không duy trì/hồi phục luồng IT→ACC khi member cũ đang shutdown, reply trở lại sau mở đường cũ theo xác nhận. LACP đã loại member cũ ở A1 nhưng trạng thái SU/P của đường mới chưa đủ bảo đảm forwarding. Chưa có bằng chứng gốc nguyên nhân; timer chậm có thể góp phần giai đoạn đầu nhưng không giải thích đầy đủ loss kéo dài sau member đã suspended. Ngoại lệ Auto/Shr không miễn kết quả này.

**Bước phân biệt tiếp theo:** trước hết xác nhận CORE1 restore hai P sau lượt mới bằng summary; không thu lại full config/save. Khi tái tạo điều kiện để chẩn đoán, chỉ cần mẫu khi A1e0/0 đã s và e1/0P: PC-IT ping VIP20.1 và real CORE1 SVI20.2, cùng show spanning-tree vlan20/show mac address-table dynamic vlan20 ở C1/A1. Đây là dữ liệu mới nhằm phân biệt client→gateway/HSRP/forwarding liên VLAN với Po chỉ hình thành; không chạy lại nguyên bài chờ100s chỉ để đếm thêm loss. Thu xong mở lại C1e0/1 và xác nhận restore, không thử lượt B hoặc đổi timer/duplex/mode on trước phân tích.

## 7. Gateway/ARP và MAC — đường PC→CORE1 chưa thông

**Nguồn:** [console gateway/STP/MAC](sprint-02-po11-member-a-gateway-stp-mac-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. Giữ dòng cuối Group Port-channel Protocol Ports bị cắt; không dựng summary/member status từ header này. Ngữ cảnh người thực hành là pha một member của bài chẩn đoán; lượt này không có clock/mốc shutdown/restore mới.

- PC-IT ping10.10.20.1 và10.10.20.2 đều báo host(...)not reachable, không có ICMP reply. Trong [mã nguồn VPCS](https://github.com/GNS3/vpcs/blob/master/src/command.c), run_ping in thông báo này khi arpResolve tới host cùng subnet hoặc gateway thất bại; nguồn tham khảo không thay việc kiểm phiên bản lab. Vì đích20.2 là IP thật cùng VLAN, lỗi quan sát xuất hiện trước khi cần định tuyến sang VLAN40, không chỉ riêng VIP HSRP.
- CORE1 STP VLAN20 root aabb.cc00.1000/priority24596; Po11 Desg/FWD/cost100, Po1 Desg/FWD/cost56, e0/2 Desg/FWD/cost100. A1 Po11 Root/FWD/cost100 cùng root; e0/1 tớiCORE2 Altn/BLK, e0/2 Desg/FWD. Không có bằng chứng STP block Po11 trong VLAN20; đường CORE2 bị block đúng mô hình vòng, không tự mở hoặc ép role để che lỗi.
- A1 bảng dynamicVLAN20 có0050.7966.6805 ởe0/2, 0000.0c07.ac14/aabb.cc80.1000/aabb.cc80.2000 ởPo11. MAC0000.0c07.ac14 phù hợp VIP HSRPgroup20 đã cấu hình, nhưng entry có thể còn trong aging, không tự chứng minh traffic Core→Access đang đi ở đúng thời điểm mẫu.
- CORE1 bảng dynamicVLAN20 chỉ cóaabb.cc80.2000 ởPo1; không cóMAC PC0050.7966.6805. A1 đã học PC ở access port nhưng C1 chưa học sourcePC quaPo11 trong mẫu sau thao tác ping; gợi ý đoạn chuyển tiếp A1→C1 qua đường mới cần điều tra. Thiếu packet capture/counter delta/source-time để khẳng định unidirectional vật lý, bug hash, lỗi image hoặc mất ARP ở vị trí cụ thể. Không tạo staticARP/staticMAC để che triệu chứng.

**Bước tiếp theo đã chuẩn bị, chưa thực thi:** kiểm thử tạm đường mới CORE1e1/1↔A1e1/0 như trunk độc lập, giữ old member shutdown cả hai đầu và Po1/Core2/PC/SVI giữ nguyên. Shutdown new member hai đầu trước no channel-group11, xác nhận trunk dot1q/native99/allowed cũ rồi mở new cả hai, đọc STP và pingVIP/realSVI/ACC/MAC. Sau lấy mẫu shutdown new hai đầu, gán lại group11 mode active, mở new rồi old hai đầu, kiểm tra hai P/peer/traffic. Không wr trong thí nghiệm; startup/bốn snapshot ổn định giữ nguyên.

Nếu standalone hoạt động còn single-memberPo không hoạt động, hướng điều tra là cơ chế forwarding/aggregation khi member đổi, chưa đủ kết luận cụ thể bug image. Nếu standalone vẫn không thông, kiểm đường mới/VLAN/cổng/VM độc lập với LACP. Đây là phép cách ly có thay đổi runtime tạm và rollback rõ, không phải chuyển thiết kế cuối hoặc ghi PASS S02-TC-07. Nếu không phục hồi đúng thì dừng bài sau, giữ output lỗi.

## 8. Lỗi cú pháp khi bắt đầu cách ly —16:16:14UTC

**Nguồn:** [console CORE1](sprint-02-po11-isolation-command-error-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. CORE1 vào e0/1 shutdown, e1/1 shutdown, no channel-group11 báo %Incomplete command; sau đó nhập trunk matching và end. Log hai interface admin down lúc16:16:14.368UTC, config log16:16:14.515UTC.

Lệnh gỡ bị từ chối nên chưa xác nhận e1/1 rời group11; không coi các dòng trunk sau đó là đã tách nhóm hoặc thành công standalone. Shutdown đang đúng pha chuẩn bị; cần giữ đóng đến khi hai đầu đã tách/kiểm tra xong. Chưa có trạng thái ACCESS1 mới, không mặc định người thực hành đã chạy khối A1.

Đề xuất phủ định đầy đủ dòng đang có: no channel-group11 mode active trên e1/1. Sau đó kiểm running section/summary, giữ shutdown. Nếu còn báo lỗi, thu no channel-group ? và no channel-group11 ? tại config-if; cú pháp hợp lệ cần xác nhận parser image, không ghi lệnh mới đã thành công từ tài liệu chung. [Cisco IOS mô tả no channel-group để gỡ interface khỏi nhóm](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/interface/configuration/15-sy/ir-15-sy-book/ir-cfg-fec.html); tài liệu platform khác không thay output IOL.

Đã sửa hai khối chuẩn bị trong hướng dẫn nội bộ sang full no form có điều kiện kiểm kết quả. Không cần default interface/xóaPo11/đổi mode on hoặc wr; Po1 và các snapshot nguồn giữ nguyên. Lỗi parser này riêng với S2-LACP-02 forwarding, không chứng minh nguyên nhân loss trước đó.

## 9. Sau ACCESS1 treo và khởi động lại — hai đầu đang khác cấu hình tạm

**Nguồn:** [console sau restart](sprint-02-po11-isolation-after-access1-restart-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. Trước đó người thực hành báo A1 treo và hỏi Stop/Start, được hướng dẫn không wr cấu hình tạm; sau đó gửi mẫu này. Không có show clock/reload log hoặc phiên bản mới, không xác định nguyên nhân treo từ lần restart.

A1 summary Po11(SD)/LACP/e0/0(s)/e1/0(s); running e1/0 dài264byte có trunk matching/native99/allowed bảyVLAN/nonegotiate/group11modeactive, không shutdown trong section. C1 running e1/1 dài246byte có trunk matching/shutdown, không có channel-group. Đây là bằng chứng C1 new member đã tách, không cần lặp lệnh gỡ C1. Không có output command gỡ được chấp nhận để xác nhận chính xác parser dùng dạng nào; không tự thêm log thành công no channel-group11 mode active.

Po11SD/s phíaA1 phù hợp khi không có partner LACP hoạt động với trạng thái chẩn đoán C1: old port shutdown ởmốc trước, new port shutdown và đã tách. Không gọi đây là lỗi mới hoặc yêu cầu sửa LACP để đạt haiP khi mục tiêu hiện tại đang cáchly. A1 e1/0 còn group11 nên chưa được mởnewstandalone cả hai đầu; chưa có A1 runninge0/0 hiện tại hoặc Core1summary mới, không dựng chúng từ snapshot.

**Bước tiếp theo:** chỉ A1 e0/0shutdown, e1/0shutdown/no channel-group11 mode active; kiểm running e1/0/summary. Nếu CLI từ chối dùng ? và giữ đóng, không mở tiếp. Nếu e1/0 hết group/vẫnshutdown/trunk đúng, hoàn tất phần1; sau đó chỉ mở C1e1/1 và A1e1/0, old hai đầu giữshutdown. Thu trunk/STPVLAN20/MACVLAN20 và PC-IT pingVIP/realSVI/ACC; chú ý A1 rootport phải e1/0, C1 họcMACPC ởe1/1 để chứng minh đúngđường. Đạt hay không đều hoàn nguyên theo phần4 hướng dẫn; không wr, không xóaPo11/defaultinterface hoặc đổiPo1.

Chưa có actual cho A1 tách/openstandalone/traffic/rollback trong lượt này; S2-LACP-02 vẫn OPEN và lượtAFAIL lịch sử giữ nguyên. Chưa miễn lỗi forwarding theo ngoại lệ Auto/Shr hoặc kết luận image không hỗ trợ.

## 10. New link standalone trunk/FWD và ba mẫu ping đạt

**Nguồn:** [console standalone](sprint-02-po11-new-link-standalone-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng/dòng trắng phân cách. Không có show clock/summary mới hoặc command remove-group/open cổng. Người thực hành ghi cuối tin nhắn shut e0/1 vàe0/0/no shut e1/1 vàe0/1; cụm e0/1 cuối không khớp mapping newA1e1/0. Phân tích theo output rõ A1e1/0 trunk/RootFWD, không tự sửa tên port trong source hoặc kết luận đã tách bằng một cú pháp cụ thể.

- A1 trunk e0/1 vàe1/0 dot1q/native99/allowed-active-forwarding đủ bảyVLAN; VLAN20 e1/0 RootFWD/cost100 tớiC1root aabb.cc00.1000/priority24596. e0/1 nayDesgFWD/cost100, e0/2DesgFWD. Không áp expectedAltnBLK cũ cho output topology mới; cần đối chiếuCore2.
- C1 trunk chỉ e0/2 vàe1/1/native99/allowed-active-forwarding bảyVLAN; VLAN20 C1root, e1/1DesgFWD/cost100, e0/2DesgFWD/cost100. Po1 không xuất hiện cả trunk vàSTPVLAN20 trong mẫu; chưa có bằng chứng để gọiPo1 vẫn haiP/up/trunk hoặc khẳng định đãdown/bịxóa. Chỉ MAC entryPo1 không chứng minh link đang hoạt động.
- MACVLAN20 C1 chỉaabb.cc80.2000/DYNAMIC/Po1, không cóPC0050.7966.6805. Khối này được gửi trước phầnping trong tin nhắn, thiếutimestamp để khẳng định thứ tự lấy mẫu thực tế/aging; chưa đáp ứng yêu cầu chứng minh C1họcPC ởe1/1. Cần lấy ngay sauping để đối chiếu, không đổi giá trị MAC hoặc gán route path từping một mình.
- PC-IT→VIP20.1 có5/5/TTL255/RTT0.520,0.818,0.813,0.695,0.732ms. →realSVI20.2 có5/5/TTL255/RTT0.461,0.794,0.700,0.693,0.792ms. →ACC40.20 có5/5/TTL63/RTT3.002,2.586,1.262,1.168,1.154ms. Traffic hiện có phản hồi khi newports chạytrunk độclập; không nâng thànhHA/lượtA PASS hoặc traffic hai chiều.

**Kết luận có giới hạn:** newlink cóSTP/trunk hợp lệ và ba mẫu traffic thành công trong topology hiện tại, khác trạng thái single-memberPo11 thất bại. Hướng điều tra aggregation có thêm cơ sở nhưng chưa cô lập duy nhất biếnchannel: A1 đãrestart giữa các lượt, Po1 hiện vắng trongtrunk/STP và MACPC chưa chứng minh đường. Chưa kết luận chắc lỗiimage/cable/driver hoặc cầnđổiimage.

**Bước tiếp theo:** giữ trạng thái thử, chưawr/gán lạiPo11/đổiport thêm. Thu C1show etherchannel summary/show interfaces port-channel1; Core2show etherchannel summary/show interfaces trunk. PC-ITping20.2 5gói rồi ngayC1show mac address-table dynamic vlan20. Chỉ nếuPo1 bất thường mới đọc config/member để xử lý dựa nguồn; không mặc định no shutdown/resetPo1 trước output. Sau xác minhbaseline sẽ hoàn nguyênnewlink vềPo11 theo hướng dẫn và kiểm haiP/peer/STP/HSRP/traffic. RootcauseS2-LACP-02 và rollback vẫn chờ, startup/snapshots giữ nguyên.

## 11. Po1 còn hai P hai đầu; MACPC đã học trên new standalone

**Nguồn:** [console kiểmPo1/MAC](sprint-02-po11-standalone-po1-mac-check-2026-10-16.txt), chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng. Không có show clock/HSRP hoặc ping mới trong lượt này, dùng ba mẫu5/5 ởphần10 cùng MAC mới để đối chiếu; không tạo thêm mẫu ping.

- CORE1Po1SU/LACP/e0/0P/e0/3P; interfacePo1up/up/BW20000Kbit, đúng members e0/0/e0/3, các counter lỗi hiển thị0. RX20709/TX48441 là tích lũy, không suy khả năng chịu lỗi hoặc throughput. CORE2Po1SU/LACP/haiP cùng cổng; trunkPo1dot1q/native99/allowed-active-forwarding đủbảyVLAN, hai trunkAccess e0/1/e0/2 vẫnđủVLAN.
- CORE1Po11SD/LACP chỉ e0/1D: new e1/1 không trongPo11 ởmẫu này, phù hợpstandalone; Po11down là dự kiến khiold vẫnshutdown. Không yêu cầu sửaPo11 lênSU trong pha thử trước gánmember.
- CORE1MACVLAN20 hiện có0050.7966.6805/DYNAMIC/Et1/1 vàaabb.cc80.2000/DYNAMIC/Po1. Kết hợp A1e1/0RootFWD/trunk và ba mẫuping5/5 phần10, đã có bằng chứng newlink chạystandalone chuyển được traffic củaPC đúng phíaCORE1. Không còn dùng mẫu MACthiếuPC trước đó để khẳng định newlinkhỏng.
- Po1bundled/up haiđầu đã được xác minh, bác bỏ giả định Po1đangdown nếu chỉ dựa mẫutrunk/STPvắng trước. Tuy nhiên C1Po1operationaltrunk/STPVLAN20 vẫn chưa thu mới; C2trunk không chứng minh mode củaC1. Có thể lấy C1show interfaces port-channel1 switchport cùng lượt tiếp để khép điểm chưa rõ; không reset/sửaPo1 khi chưa cóoutput.

**Kết luận:** newlink/vlan/traffic có thể hoạt động ngoàiPo11; lỗi banđầu gắn với bối cảnh aggregation/mấtmember, hướng điều tra đó được ưu tiên. Không đủ kết luận chắc lỗiimage/hash hoặc restartA1 là yếu tố duy nhất; các kết quả trước/sau vẫn khác thời điểm/trạng thái.

**Bước tiếp theo đã chuẩn bị:** hoàn nguyên theo giai đoạn: oldmember giữshutdown, newmember shutdown/gánlạigroup11active vàmở cảhaiđầu; xácnhậnPo11SU chỉnewP, trunk/STP/MAC/PCtraffic trong trạng thái new-only. Sau lấy mẫu mởold ởcảhaiđầu để restorehaiP vàkiểmHSRP/traffic. Nếunew-onlyFAIL cũng giữoutput rồi mởold để trả baseline, không giữoutage dài hoặc ghiPASS. Khôngwr thínghiệm, không xóaPo11/đổiPo1/SVI/PC; chưa cóactual cácbước này. New-only được khởi tạo lại nếuđạt chưa thay thế bài mấtoldmember từ haiP.

## 12. Gán lại Po11 chỉ member mới — chức năng đã đạt

**Nguồn:** [CORE1 nguyên attachment](sprint-02-po11-new-member-only-core1-2026-10-16.txt), sao nguyênbyte SHA256 `dc6d8d4afd762e1cc6b7c9629067749afa14687be98874ca2db21d7303d6790f`; [A1/C1MAC-switchport/ping](sprint-02-po11-new-member-only-access1-ping-2026-10-16.txt) chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng, giữprefix^[[A ởreplyrealSVIseq4. Không cóshowclock/loggánmember/wr/fullconfig trong lượt.

AttachmentC1 cóhai bộsummary/neighbor/trunk/STPVLAN20; agepeer khác20s→22s, không phải hai bảntrùng tuyệtđối. Không cótimestamp để tính thời gian ổnđịnh từhai mẫu này.

- CORE1Po11SU/LACP/olde0/1D/newe1/1P, new nhậnA1aabb.cc80.3000/key0xB/remotePort0x101/SA/state0x3D. A1Po11SU/olde0/0D/newe1/0P, new nhậnC1aabb.cc80.1000/key0xB/remotePort0x102/SA/0x3D, age5s. DòngoldDevID0/SP/key0 không phải partner hợp lệ hoặc modepassive thật củapeer.
- Po11 haiđầu trunkdot1q/native99/allowed-active-forwarding đủVLAN10,20,30,40,50,60,99. C1STPVLAN20 root/Po11DesgFWD100/Po1DesgFWD56; A1Po11RootFWD100, e0/1tớiC2AltnBLK100/forwarding-none, e0/2DesgFWD100. TopologySTP trởlại roles phùhợp baseline mộtmember. Không cóinconsistentports mới trong lượt, không tự ghi0.
- CORE1Po1SU/haiP cảhaibộ; switchport output xác nhận Enabled/admin-opertrunk/dot1q/native99/nonegotiate/allowedbảyVLAN, trunk/STPVLAN20 hiện cóPo1. Kết hợpCore2Po1haiP/trunk ởphần11, đãkhép thiếu dữ liệu hiện trạngPo1local. Chưa giải thích nguyênnhân vì saoPo1vắng trongmẫu phần10, không suy ra lỗiđãxảyrahoặc resetPo1.
- C1MACPC0050.7966.6805 DYNAMICPo11, C2MACaabb.cc80.2000Po1. Kết hợpoldD/newP, STP vàping, xác nhận forwarding PC quaPo11 đangchỉcónewmember, khôngnhầm với old đượcmở.
- PC-IT→VIP20.1 có5/5/TTL255/RTT0.581,0.838,0.887,0.865,0.745ms. →realSVI20.2 có5/5/TTL255/RTT0.492,0.828,0.757,0.799,0.754ms. →ACC40.20 có5/5/TTL63/RTT2.866,1.172,1.440,1.204,1.169ms. Prefix^[[A không làm mấtreplyseq4, không nâng thànhmẫu20gói/60sổnđịnh hoặc chiều ngược.

**Kết luận:** newmember cóthể chuyểntraffic cảstandalone và trongPo11 đượcgánlại/khởitạo vớiold đóng. Điều này loại bỏ giả định tuyệtđối newlink khônghoạtđộng hoặcimage khônghỗtrợsingle-memberPo11; ưu tiên điều tra quátrìnhchuyển từhaiP xuốngmộtP/trạngthái forwarding hoặc treoVM. Chưa xácminh rootcause cụthể hoặc quan hệ nhânquả củarestartA1/gỡ-gánnew, không viếtđãfixbugimage.

**Tiếp theo:** mởC1e0/1 vàA1e0/0 (chỉno shutdown), xác nhậnPo11haiP/peer/HSRP/trunk-STP/traffic trởbaseline, giữPo1/SVI/PC. Khôngwr vìstartup trước đóđãđúng. Saurestoređạt mới lênlượtkiểmchuyển mấtold cótraffic/timestamp vàphụchồi; chưaPASS S02-TC-07/lượtB hoặc toànSprint2 từnew-only5/5. Không yêu cầu gỡgroup/restart/tạo snapshot từsummary/full dump lại trongbướcrestore.

## 13. Tiếp tục17/10 sau lưu cấu hình và tắt/bật máy — chưa có runtime mới

Người thực hành báo hôm trước hết limit nên đã lưu cấu hình và tắt máy, nay mới bật lại. Chưa có output wr/save hoặc boot mới, chưa xác định node/pha nào đã được lưu. Đây là thông tin thay đổi điều kiện tiếp tục, không phải bằng chứng rằng startup mới vẫn khớp các snapshot trước bài lỗi.

Nếu lưu lúc old member shutdown hoặc lúc standalone, trạng thái thử có thể tồn tại sau boot; vì vậy lời hướng dẫn trước "không cần wr vì startup đã đúng" chỉ áp dụng mốc trước tin nhắn này. Bốn snapshot ổn định cũ giữ nguyên làm nguồn, không sửa chúng theo lời báo đã save. Sau hoàn nguyên vận hành đúng, cần kiểm phần startup thay đổi và lưu đích ổn định nếu vẫn còn trạng thái thử; chỉ ghi persistence mới khi có source thật.

Bước tiếp theo là kiểm Po1/HSRP hai Core, Po11 và old member section CORE1e0/1/ACCESS1e0/0, IP/gateway PC-IT/ACC. Dựa output mới chọn phần mở old/gán nhóm/kiểm traffic phù hợp, không dựng lại lab hoặc chép baseline khi chưa biết trạng thái. IOSwr và VPCSsave là hai thao tác riêng. Chưa có actual mới, chưa kết luận mất cấu hình/boot thành công/restore hoặcHA đãPASS; S2-LACP-02 vẫnOPEN.

## 14. Kết quả thử từng member17/10 — phục hồi hai P, hash lúc thử chưa rõ

Ghi gọn từ tin nhắn người thực hành, không tạo thêm transcript hoặc snapshot cấu hình. Sau boot đã có Po1 haiP haiCore, HSRP C1Active110/C2Standby100 đủ sáu nhóm; C1e0/1 vàA1e0/0 còn shutdown. Sau mởold, hai đầu Po11haiP/peer đúng và PC-ITpingVIP/ACC mỗi5/5. Không có outputwr mới để xác nhận đã lưu lại baseline này.

Lượt tắtold C1e0/1 có log admin down15:13:08.814/line down15:13:09.818UTC17/10, C1Po11SU/e0/1D/e1/1P, Po1haiP. Người thực hành báo không mất gói, chưa có chuỗi ping đếm được của lượt này. Sau đó họ thửnew C1e1/1; log admin down15:17:01.569/line down15:17:02.574, mở lại/linkup15:18:04.575/lineup15:18:05.576. Trong lỗi C1Po11SU/e0/1P/e1/1D; sau phục hồi C1haiP vàA1haiP. A1 summary được xác định là mẫu sau khi hoàn thành ping, không dùng làm mẫu trong lỗi.

Chuỗi PC-IT→ACC40.20 có seq1–67 liên tục,67 reply/0timeout, TTL63, dùng -t/-i1000/-w1000 và Ctrl+C kết thúc. Chưa có timestamp từnggói hoặc mốc thao tác chung với consolePC nên không đồng nhất67gói thành đủ trước–trong–sau theo timeline, không kết luận hội tụ0ms hoặc đã đủ các yêu cầu thời gian/hai chiều của toàn caHA. Log interface cho thấy khoảng admin-down→link-up63.006s, đây là thời gian trạng thái cổng trong logCORE1, không phải thời gian mấttraffic.

A1 vàC1 show load-balance sau thử đều src-dst-ip. Chưa có show xác nhận src-dst-mac trước/trong hai lượt, không có command rollback hash trong nguồn mới: cần hỏi đổi vềIP trước hay sau các lượt, không tự kết luận doMAC/hash hay restart/gỡ-gán đãfix. Các lỗi113timeout trước đó giữ lịch sử, không sửa thànhPASS. Hiện không cần reset/gỡ nhóm/đổiimage hoặc lặp ngay bài thử; sau xác minh điều kiện và lưu baseline vận hành nếu chưawr, tiếp tục đánh giá đủ phạm vi S02-TC-07 rồi bài kế tiếp.

**Bổ sung xác nhận của người thực hành17/10:** đã hoàn nguyên ACCESS1 về src-dst-ip trước khi shutdown CORE1e0/1. Vì vậy lượt old-member thành công theo báo không diễn ra dưới hashMAC; lượt new-member sau đó có output sau thử src-dst-ip và67reply liên tiếp. Không cần ép chạy lại chỉ để làm rõ thời điểm đổihash. Ghi nhận chức năng hiện tốt trong điều kiện IP gốc; chưa xác định vì sao lỗi trước biến mất (không gán nhân quả cho restart/gỡ-gán/hash). Các yêu cầu trước–trong–sau có timestamp chung/hai chiều của toàncaHA vẫn giữ phạm vi riêng.
