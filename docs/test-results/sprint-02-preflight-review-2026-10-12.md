# Sprint 02 — Kiểm tra baseline HQ, 12/10/2026

**Phạm vi:** bốn switch HQ và traffic VPCS của buổi 1 / S02-TC-01, trước LACP.  
**Trạng thái:** BUỔI 1 ĐÓNG THEO QUYẾT ĐỊNH NGƯỜI THỰC HÀNH NGÀY 12/10/2026. Đã đối chiếu startup bốn switch, runtime/VLAN, IP/gateway bốn PC, traffic smoke 5/5 và capability LACP/duplex. Bước thử restore Sprint 01 trước LACP được bỏ qua theo yêu cầu, không có kết quả PASS restore. Full-duplex/link type thực tế và save VPCS chưa được xác nhận bằng output.  
**Nguồn:** console switch và nội dung VPCS do người thực hành cung cấp ngày 12/10/2026 qua file đính kèm/tin nhắn; agent đọc output và đối chiếu file local, không chạy thiết bị hoặc lưu cấu hình trên switch/VPCS.

## 1. Bằng chứng đã lưu

- [CORE1 — session nguyên trạng](sprint-02-core1-preflight-2026-10-12.txt), SHA256 `63E97CFCE94DD46A4D10C6529A9D11D13E40A4268B1542077D17F641DD26D8C8`.
- [CORE2 — session nguyên trạng](sprint-02-core2-preflight-2026-10-12.txt), SHA256 `80DDAC49238E4BB2FD4484588E326903813CEF4FBD9EA81A817854FD00B9882D`.
- [ACCESS1 — session nguyên trạng](sprint-02-access1-preflight-2026-10-12.txt), SHA256 `9353346E6A27FD9471186D812C9D48470F0351B0156BA5427A60AD97BAF1D1E1`.
- [ACCESS2 — session nguyên trạng](sprint-02-access2-preflight-2026-10-12.txt), SHA256 `7E8DA3F2D1DC4376987B1F6184F41FA5FA902E190DD0F0AF50D9594225C6665F`.
- [VPCS — nội dung console người thực hành dán](sprint-02-vpcs-preflight-2026-10-12.txt): chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng; không phải file session gốc có timestamp.
- [CORE1 — VLAN và chi tiết cổng nguyên trạng](sprint-02-core1-port-preflight-2026-10-12.txt), SHA256 `AECE48A2CC1A290BCAC732BB1F983C3A806A85718085BA1CFD50C2E0EEA35714`, khớp file đính kèm nguồn.
- [CORE2/ACCESS1 — VLAN và chi tiết cổng](sprint-02-core2-access1-port-preflight-2026-10-12.txt): chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng; gồm cả `show ip` PC-IT được gửi lại, chưa có PC-ACC.
- [PC-ACC và trợ giúp LACP/duplex/speed ba switch](sprint-02-lacp-capability-preflight-2026-10-12.txt): chép từ tin nhắn, chuẩn hóa khoảng trắng cuối dòng; không có timestamp hoặc output save.
- [CORE1 — startup pre-LACP](../../configs/core/sprint-02/CORE1.pre-lacp.startup.cfg), trích từ output `show startup-config` của CORE1.
- [CORE2 — startup pre-LACP](../../configs/core/sprint-02/CORE2.pre-lacp.startup.cfg), trích từ output `show startup-config` của CORE2.
- [ACCESS1 — startup pre-LACP](../../configs/access/sprint-02/ACCESS1.pre-lacp.startup.cfg), trích từ output `show startup-config` của ACCESS1.
- [ACCESS2 — startup pre-LACP](../../configs/access/sprint-02/ACCESS2.pre-lacp.startup.cfg), trích từ output `show startup-config` của ACCESS2.

Bốn session switch ban đầu được sao chép nguyên trạng, hash khớp nguồn. File startup chỉ bỏ prompt và dòng thông báo dung lượng, giữ khối cấu hình từ `!` đến `end`; không tạo từ running. Không ghi đè artifact Sprint 01. Lượt VLAN/chi tiết cổng mới không có `show clock`; không gán giờ thiết bị của lượt trước cho lượt này.

`show clock` CORE1 hiển thị `12:54:31.571 UTC`, CORE2 `12:55:00.994 UTC`, Mon Oct 12 2026, tương ứng `19:54:31.571` và `19:55:00.994` Asia/Saigon. Đây là giờ thiết bị hiển thị, chưa đối chiếu với đồng hồ host; hai output được lấy ở thời điểm khác nhau, không dùng chênh lệch này để kết luận clock offset hoặc convergence.

ACCESS1 hiển thị `12:59:27.273 UTC`, ACCESS2 `12:59:50.629 UTC` cùng ngày, tương ứng `19:59:27.273` và `19:59:50.629` Asia/Saigon. Các snapshot Core/Access không được lấy đồng thời; không dùng chúng để chứng minh trạng thái ổn định liên tục trong cả khoảng thu thập.

## 2. Đối chiếu startup với baseline Sprint 01

- CORE1 startup khớp **112 dòng lệnh** của [running cuối Sprint 01](../../configs/core/sprint-01/CORE1.cfg).
- CORE2 startup khớp **101 dòng lệnh** của [running cuối Sprint 01](../../configs/core/sprint-01/CORE2.cfg).
- ACCESS1 startup khớp **72 dòng lệnh** của [running cuối Sprint 01](../../configs/access/sprint-01/ACCESS1.cfg).
- ACCESS2 startup khớp **72 dòng lệnh** của [running cuối Sprint 01](../../configs/access/sprint-01/ACCESS2.cfg).
- Phép so sánh loại dòng trắng/comment `!`, bỏ khoảng trắng cuối dòng, giữ nguyên thứ tự và thụt lề của các lệnh; không bỏ qua hoặc thêm lệnh cấu hình.
- Cả hai startup giữ VLAN/trunk native99/allowed bảy VLAN, rapid-pvst, STP priorities, sáu SVI/HSRP/preempt, `no ip cef` và VTY `transport input none`.
- CORE1 startup có `shutdown` trên Ethernet1/0; runtime cũng báo cổng này administratively down/down. Phần startup của thay đổi cổng test đã được xác nhận trong lượt mới này.
- Hai Access startup giữ `no ip routing`, SVI10 `.11/.12`, default gateway `10.10.10.1`, trunk/access VLAN đúng baseline và VTY `transport input none`.

**Kết luận:** đã có startup thật cả bốn switch khớp baseline cần giữ; không cần yêu cầu `wr` lặp chỉ để bù file thiếu. Không có full running-config mới trong session, nên kết luận là startup khớp **running Sprint 01 đã lưu**, không tuyên bố đã so toàn bộ running hiện tại với startup. Phần thu/đối chiếu startup của S1-F01 đã được bổ sung đủ bốn switch; kiểm thử persistence sau stop/start hoặc restore vẫn chưa chạy.

## 3. Runtime hai Core đã đối chiếu

- **SVI:** VLAN10/20/30/40/50/60 up/up cả hai Core; CORE1 `10.10.X.2`, CORE2 `10.10.X.3` đúng bảng địa chỉ.
- **HSRP:** đủ sáu group 10/20/30/40/50/60; CORE1 Active priority110, CORE2 Standby priority100; hai bên có preempt, nhận đúng peer và VIP `10.10.X.1`. Không có dual-active trong các snapshot này. Đây là runtime đầy đủ sáu nhóm bổ sung cho hồ sơ Sprint 1.
- **Trunk:** mỗi Core có e0/0, e0/1, e0/2 trunking 802.1Q, native99. Allowed/active/forwarding-not-pruned đều có `10,20,30,40,50,60,99` tại thời điểm thu trên Core. Kết hợp mục 4 đã có snapshot runtime cả mười đầu của năm trunk, tại các thời điểm thu khác nhau.
- **STP:** CORE1 root cả bảy VLAN Campus, Root MAC `aabb.cc00.1000`, cost0. CORE2 nhận cùng Root ID, cost100 qua e0/0 cho cả bảy VLAN; không có inconsistent port trên hai Core.
- **VLAN1:** hai Core tự hiện root cho VLAN1; VLAN1 không nằm trong allowed list của các trunk đã thu. Không lấy VLAN1 làm tiêu chí root Campus, không yêu cầu sửa thiết kế vì riêng dòng này.
- **Routes:** mỗi Core có sáu connected /24 và sáu local /32 tương ứng SVI. `Gateway of last resort is not set` phù hợp baseline HQ chưa nối WAN; không cần thêm default route cho kiểm tra liên VLAN connected.

`show ip interface brief` còn báo các Ethernet chưa dùng up/up; không dùng thông tin này để khẳng định cổng có dây hoặc đã phát sinh link mới. Chi tiết cổng bổ sung được đánh giá tại mục 6; chưa nghiệm thu duplex/link type cho LACP.

## 4. Runtime hai Access đã đối chiếu

- **VLAN:** cả hai có VLAN10 MGMT, 20 IT, 30 HR, 40 ACCOUNTING, 50 SERVER, 60 GUEST, 99 NATIVE ở trạng thái active. ACCESS1 e0/2 VLAN20, e0/3 VLAN30; ACCESS2 e0/2 VLAN40, e0/3 VLAN60 đúng baseline.
- **Quản trị:** SVI10 ACCESS1 `10.10.10.11`, ACCESS2 `10.10.10.12`, đều up/up; startup có mask /24 và default gateway `10.10.10.1`.
- **Trunk:** e0/0/e0/1 cả hai Access trunking dot1q, native99, allowed/active đủ bảy VLAN. e0/0 forwarding-not-pruned cả bảy; e0/1 hiển thị `none`, không tự kết luận trunk lỗi từ dòng này.
- **STP VLAN20:** cả hai dùng RSTP, nhận Root MAC CORE1 `aabb.cc00.1000`, cost100; e0/0 Root/FWD, e0/1 Alternate/BLK. ACCESS1 e0/2 là Designated/FWD cho client IT. Các trạng thái phù hợp dual uplink: đường dự phòng bị STP chặn, khớp forwarding `none` trên e0/1.
- **Inconsistent:** bằng 0 trên hai Access, kết hợp Core là 0 trên cả bốn snapshot.
- **Link type:** output Access vẫn ghi `Shr`; chi tiết ACCESS1 mới báo Auto-duplex, chưa xác nhận full-duplex hoặc nguyên nhân link type. Không tự sửa thành point-to-point từ snapshot này.

## 5. Traffic VPCS đã đối chiếu

- Có **tám lượt ping, mỗi lượt nhận đủ 5/5 phản hồi**, tổng 40/40; không có timeout trong nội dung được cung cấp. Đây là kiểm tra chức năng ngắn trước thay đổi.
- IT `10.10.20.20` → HR `10.10.30.20`, ACC `10.10.40.20`, GUEST `10.10.60.20`: mỗi đích 5/5. Các console HR, ACC, GUEST → IT cũng mỗi lượt 5/5, xác nhận liên lạc cả hai chiều đã thử.
- IT → ACCESS1 `10.10.10.11` và ACCESS2 `10.10.10.12`: mỗi lượt 5/5, xác nhận truy cập IP quản trị Access từ VLAN20 và có phản hồi trở về.
- `show ip` xác nhận IT `10.10.20.20/24`, HR `10.10.30.20/24`, GUEST `10.10.60.20/24`, mỗi máy dùng gateway `.1` của subnet tương ứng. Lượt bổ sung đã xác nhận PC-ACC `10.10.40.20/24`, gateway `10.10.40.1`, MAC `00:50:79:66:68:07`; đủ IP/mask/gateway bốn PC đúng baseline.
- GUEST liên lạc IT phù hợp baseline hiện chưa áp dụng ACL cô lập Guest; không coi đây là lỗi của buổi 1.
- Ghi đúng số gói thực tế là 5/5, không đổi thành 20/20. Có thể chuyển sang kiểm tra cổng với kết quả chức năng này; chưa nghiệm thu bài traffic dài, baseline đủ các VLAN hoặc failover HA. Không có output/ xác nhận `save` VPCS trong lượt gửi này.

## 6. VLAN Core và chi tiết cổng bổ sung

- `show vlan br` CORE1 và `show vlan brief` CORE2 đều có VLAN10/20/30/40/50/60/99 đúng tên và active. Đã bổ sung đủ VLAN database runtime bốn switch cùng các output Access trước đó.
- CORE1 e1/0 còn liệt kê trong VLAN10 vì là access port của bài test cũ; danh sách VLAN không chứng minh port đang mở. Startup/runtime trước đã xác nhận shutdown, không yêu cầu bật cổng này.
- Đã thu đủ tám đầu cổng dự kiến làm member: CORE1 e0/0, e0/1, e0/3, e1/1; CORE2 e0/0, e0/3; ACCESS1 e0/0, e1/0. Tất cả hiển thị up/up, MTU1500, `BW 10000 Kbit/sec`, `Auto-duplex, Auto-speed, media type is unknown`.
- Cả tám snapshot đều có 0 input errors/CRC/output errors/collisions/interface resets và total output drops0. Đây là counter tích lũy tại thời điểm thu, không phải phép đo tải hay bằng chứng link ổn định qua một khoảng kiểm thử định trước.
- Bốn cổng dự kiến thêm dây (CORE1 e0/3/e1/1, CORE2 e0/3, ACCESS1 e1/0) đều có `Last input never`, 0 packets input, nhưng counter output khác0. Không suy ra đã có dây/peer từ up/up hay output counter; vẫn cần đối chiếu đầu nối trong EVE trước sửa cabling.
- `BW 10000` là giá trị interface báo, không phải đo throughput thực tế hoặc xác nhận speed đã thương lượng. `Auto-duplex` không đủ kết luận full-duplex hay duplex mismatch. Chưa thay duplex/speed hoặc ép STP point-to-point.
- Cisco mô tả LACP ghép cổng có speed/duplex/native VLAN/VLAN range/trunk tương thích: [EtherChannel IOS 15.2(7)E](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960l/software/15-2_7_e/configuration_guide/lyr2/b_1527e_layer2_2960l_cg/m_lay2_ethchl.html). Đây là hướng dẫn nguyên tắc; capability CLI của image IOL đã kiểm tra tại mục 7. Không coi tài liệu Catalyst hoặc trợ giúp parser là bằng chứng LACP đã bundle.

## 7. Capability LACP/duplex/speed đã thu

- CORE1 e0/3 và CORE2 e0/3: `channel-group 1 mode ?` liệt kê `active`/`passive` cho LACP; ACCESS1 e1/0: `channel-group 11 mode ?` có cùng các tùy chọn. Đã xác nhận parser cung cấp LACP trên ba switch tham gia hai channel dự kiến, chưa tạo group hoặc kiểm tra negotiation/member bundled.
- `auto`/`desirable` trong menu là PAgP; `on` là EtherChannel tĩnh. Không nhầm `mode auto` với LACP passive hoặc auto-duplex.
- `duplex ?` cả ba interface liệt kê `auto/full/half`; hỗ trợ cú pháp `duplex full` nhưng chưa thực thi. Không dùng trợ giúp này để kết luận duplex vận hành đã là full.
- `speed ?` cả ba interface trả `% Unrecognized command`. Đây là kết quả capability của image/interface đang dùng; không yêu cầu gõ `speed 100/1000`, đổi image hoặc coi đó là lỗi traffic/LACP hiện tại. Trước nghiệm thu channel vẫn phải kiểm tra member tương thích và trạng thái hoạt động thực tế.
- Các lệnh được cung cấp chỉ chọn interface hiện có và xem trợ giúp, không có lệnh group/mode/duplex hoàn chỉnh. CORE1/ACCESS1 có output `end`; CORE2 kết thúc đoạn dán ở mode config-if, chưa có `end` trong bằng chứng.
- Chốt phần kiểm tra chỉ đọc của buổi 1 với duplex **Auto/chưa xác nhận full** được ghi rõ. Việc chuẩn hóa duplex hai đầu và đối chiếu STP link type thực hiện tại checkpoint triển khai trước chuyển trunk sang channel; bước thử restore trước triển khai được bỏ qua theo quyết định tại mục 9.

## 8. Giới hạn và phần còn lại

- Access chỉ có STP detail VLAN20; trunk forwarding list cho thấy e0/1 không forwarding các VLAN allowed tại thời điểm thu, nhưng chưa có port-role detail riêng cho toàn bộ bảy VLAN.
- Chưa có xác nhận `save` trên bốn VPCS; ghi nhận giới hạn này khi đóng buổi 1, không yêu cầu gửi lại `show ip` hoặc ping 5/5. Tiêu chí 20/20 trong kế hoạch và các cặp VLAN10/VLAN50 tạm của bài HA chưa được chạy trong lượt này.
- Đã thu chi tiết tám cổng và trợ giúp LACP/duplex/speed trên CORE1/CORE2/ACCESS1; còn xác minh cabling EVE, duplex/link type và member tương thích tại checkpoint triển khai. Chưa thêm link hoặc tạo Port-Channel.
- Chưa chạy stop/start, import/restore hoặc full Core/client failover. Startup khớp không tự nghiệm thu persistence sau reload/restore hay S1-F02/S1-F04.
- Không sửa kết quả Sprint 1 lịch sử thành PASS toàn bộ từ các output mới này. Phần mới bổ sung được ghi dưới Sprint 02 và ngày thu thật.

## 9. Quyết định tiếp tục ngày 12/10/2026

Người thực hành yêu cầu: “bỏ qua bước khôi phục được không, xem như là đã hoàn thành rồi”. Ghi nhận buổi 1 đã đóng theo quyết định và **S02-TC-02 — thử restore Sprint 01 trước LACP: BỎ QUA THEO YÊU CẦU / CHƯA CHẠY**, không phải PASS. Không dùng quyết định này để chứng minh persistence hoặc khả năng tái dựng lab.

Bước tiếp theo chuyển sang chuẩn bị cổng/cabling và LACP Po1 CORE1–CORE2, rồi Po11 CORE1–ACCESS1. Dùng bộ cấu hình/startup và export đã có làm tài liệu hoàn nguyên; chưa có bằng chứng thử hoàn nguyên toàn lab. Kiểm tra duplex/member/trunk/STP/HSRP/traffic vẫn thực hiện khi triển khai. Không yêu cầu thử restore trước LACP lần nữa. Quyết định áp dụng bước restore baseline đầu Sprint 2; các phép thử failover và khôi phục node/link sau gây lỗi vẫn giữ nguyên. Restore bộ cuối Sprint 2 là hạng mục riêng, chưa chạy.

Phạm vi LACP/HA chung tham chiếu [roadmap](../enterprise-network-design-implementation-roadmap.md); kết quả thật và ngoại lệ buổi 1 được ghi tại hồ sơ này.
