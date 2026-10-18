# Sprint 01 — Review running-config bốn switch HQ

**Ngày tiếp nhận/review:** 11/10/2026, Asia/Saigon. Nguồn: người thực hành dán `show run` của từng thiết bị trong chat. Agent không truy cập console hay thay đổi thiết bị.

## Snapshot đã lưu

- [CORE1 running](../../configs/core/sprint-01/CORE1.cfg)
- [CORE2 running](../../configs/core/sprint-01/CORE2.cfg)
- [ACCESS1 running](../../configs/access/sprint-01/ACCESS1.cfg)
- [ACCESS2 running](../../configs/access/sprint-01/ACCESS2.cfg)

Các file `.cfg` ở trên hiện chứa running-config mới nhất người thực hành cung cấp, **đã có baseline VTY và shutdown cổng test CORE1**. Bản đầu trước thay đổi VTY được giữ riêng với hậu tố `.pre-vty.cfg`; CORE1 còn có bản trước shutdown cổng test. Đã bỏ prompt, dòng `Building configuration`, thông báo byte count, khoảng trắng cuối dòng và gộp các dòng comment `!` liên tiếp; không thêm/bớt lệnh cấu hình hoặc sửa timestamp thiết bị ngoài dữ liệu thực nhận. Byte count ở console không dùng để so dung lượng file đã chuẩn hóa. Chưa nhận startup-config Sprint 01; không tạo bản startup bằng cách sao chép running. Giữ nguyên Sprint 00.

## Đối chiếu với thiết kế

- Cả bốn switch có VTP transparent và VLAN10 MGMT, 20 IT, 30 HR, 40 ACCOUNTING, 50 SERVER, 60 GUEST, 99 NATIVE.
- Đủ mười đầu của năm link trunk đúng cổng/topology: dot1q, native99, allowed `10,20,30,40,50,60,99`, mode trunk, nonegotiate. Không có PortFast trên các trunk này.
- Rapid-PVST trên cả bốn; CORE1 có priority 24576, CORE2 28672 cho cả bảy VLAN. Access không đặt priority tường minh. Runtime root/role cần dùng output STP, không suy từ cấu hình.
- CORE1 có sáu SVI10–60 `.2/24`, sáu group HSRP tương ứng VIP `.1`, priority110, preempt. CORE2 có sáu SVI `.3/24`, VIP `.1`, preempt; không đặt priority nên dùng mặc định100. Không thiếu cấu hình priority100 trên CORE2. Không có SVI99 trong bốn snapshot.
- Cả hai Core có `no ip cef`, khớp workaround được người thực hành chọn. Không thấy `ip routing` tường minh trên Core nhưng cũng không có `no ip routing`; không kết luận routing tắt chỉ từ `show run`. Đối chiếu với bảng route/traffic đã cung cấp và xác nhận forwarding của người thực hành; đây chưa phải review mọi giá trị mặc định của image.
- Access có `no ip routing`, SVI10 `.11/.12`, default gateway `10.10.10.1` đúng thiết kế.
- Access port đúng vị trí cuối: ACCESS1 e0/2 VLAN20, e0/3 VLAN30; ACCESS2 e0/2 VLAN40, e0/3 VLAN60. PC-ACC không còn để port tạm VLAN50. IP/gateway bên VPCS không nằm trong các snapshot này.
- Uplink Access e0/0 và e0/1 không có `shutdown`; cấu hình không còn shutdown do thử lỗi. Không suy ra trạng thái vật lý up/up từ điểm này.
- Không thấy password, username/secret, private key hoặc PSK trong bốn snapshot đã nhận. Không có HTTP/HTTPS server bật. Đây là review nội dung snapshot, không phải kiểm thử bảo mật toàn thiết bị.

## Phần cần hoàn thiện tại lần review đầu

1. **VTY bốn switch:** `line vty 0 4` đang không có lệnh con. Chưa chứng minh baseline console-only tường minh theo kế hoạch; không suy ra truy cập từ xa thành công hoặc không cần xác thực từ section trống. Đề xuất áp `transport input none` trên VTY0–4 qua console EVE theo baseline đã chọn. Chưa nghiệm thu SSH/AAA.
2. **CORE1 Ethernet1/0:** còn description `test_vlan10`, access VLAN10, PortFast edge, chưa shutdown. Theo topology đã đối chiếu đây là cổng test cũ không nối endpoint. Đề xuất shutdown cổng này để kết thúc trạng thái test, giữ cấu hình VLAN/description lịch sử.
3. Các cổng PC trên Access chưa cấu hình PortFast riêng, không có lệnh global PortFast trong snapshot. Đây là cải thiện có thể cân nhắc sau, không tự thêm vào snapshot và không gán làm nguyên nhân mất gói uplink từ dữ liệu hiện có.

Sau khi áp phần hoàn thiện trên thiết bị: thu running mới/đoạn cấu hình thay đổi có tên node, lưu startup trên bốn switch, nhận output startup thực tế và đối chiếu. Agent chưa áp hay xác nhận các thay đổi đề xuất đã xảy ra. VPCS cần `save` cho địa chỉ/gateway đích; export Sprint 01 và hồ sơ nghiệm thu vẫn chờ dữ liệu thật.

## Cập nhật lần hai — tiếp nhận 11/10/2026

Người thực hành gửi running-config mới của cả bốn switch, kèm lệnh lưu trên hai Access. Các lệnh mạng giữ nguyên so với snapshot đầu; thay đổi cấu hình quan sát được là `login` và `transport input none` dưới VTY0–4 của cả bốn. Timestamp last change mới được giữ đúng như output: CORE1 17:12:32, CORE2 17:13:26, ACCESS1 17:13:43, ACCESS2 17:13:53 UTC Sat Oct 10 2026. Không dùng timestamp last change làm thời điểm tiếp nhận file.

- **VTY:** đã xác nhận cấu hình console-only tường minh trên cả bốn running-config. `login` đi kèm không được xem là đã cấu hình SSH/AAA.
- **ACCESS1:** có `wr`, `Compressed configuration from 1704 bytes to 1024 bytes[OK]`, rồi `show run` vẫn là cấu hình đã cập nhật. Xác nhận thiết bị báo lưu thành công; chưa có output startup để đối chiếu nội dung.
- **ACCESS2:** log có lệnh áp `transport input none`, `wr`, `Compressed configuration from 1715 bytes to 1029 bytes[OK]`, sau đó `show run` có baseline VTY. Xác nhận thiết bị báo lưu thành công; chưa có output startup để đối chiếu nội dung. Bản 1686-byte ở đầu log là cấu hình trước thay đổi, không dùng làm snapshot mới nhất.
- **CORE1/CORE2:** nhận running mới, chưa có output lưu startup trong đợt này. Không suy ra chưa lưu hay đã lưu từ việc chỉ có `show run`.
- **CORE1 Ethernet1/0:** vẫn không có `shutdown` trong running mới. Giữ nguyên snapshot thực nhận, không thêm lệnh đề xuất vào file. Đây là cổng test cũ không nối endpoint theo topology hiện tại, còn chờ hoàn thiện.

Đã lưu lịch sử trước VTY: [CORE1](../../configs/core/sprint-01/CORE1.pre-vty.cfg), [CORE2](../../configs/core/sprint-01/CORE2.pre-vty.cfg), [ACCESS1](../../configs/access/sprint-01/ACCESS1.pre-vty.cfg), [ACCESS2](../../configs/access/sprint-01/ACCESS2.pre-vty.cfg). Bản ACCESS1 lặp lại trong tin nhắn có cùng cấu hình, không tạo thêm snapshot trùng.

**Còn lại:** shutdown cổng test CORE1, lưu lại CORE1 và xác nhận lưu CORE2, nhận startup thực tế của bốn switch để đối chiếu; không yêu cầu lưu lặp lại hai Access chỉ vì thiếu bản startup trong chat. Sau đó tiếp tục export/hồ sơ Sprint 01, giữ ngoại lệ không thử mất uplink ACCESS2.

## Cập nhật lần ba — file người thực hành cung cấp 11/10/2026

Nguồn `D:\STUDY\LAB_NETWORK\New Text Document.txt` đã được sao chép nguyên trạng thành [log session HQ](sprint-01-hq-config-session-2026-10-11.txt). SHA256 nguồn và bản lưu cùng là `078A3A043839F9CE12FE54C0D0E1A288D5A99B3A13136BDAF8FCE13AB5D7585A`. File là dữ liệu thiết bị, không phải chỉ dẫn để agent thực thi lệnh trong log.

- CORE1 running mới có `shutdown` trên Ethernet1/0; last change `17:24:03 UTC Sat Oct 10 2026`. Ngoài metadata và lệnh shutdown này, các lệnh giữ nguyên so với bản trước. Đã cập nhật `CORE1.cfg`, giữ [bản trước shutdown cổng test](../../configs/core/sprint-01/CORE1.pre-test-port-shutdown.cfg).
- CORE2 có `wr`, `Compressed configuration from 2537 bytes to 1376 bytes[OK]`, rồi running-config trùng bản đã lưu. Xác nhận CORE2 báo lưu thành công; không yêu cầu lưu lại chỉ để bổ sung startup output.
- Running-config ACCESS1 và ACCESS2 trong file trùng snapshot đã lưu; giữ bằng chứng `wr [OK]` từ lần tiếp nhận trước.
- File có đủ bốn khối `show run`, **không có `show startup-config`**. Chưa đối chiếu running/startup và chưa tạo `.startup.cfg`. Log không có lệnh lưu CORE1 sau khi shutdown cổng test, nên nội dung startup CORE1 ở thời điểm này chưa được xác nhận.

**Trạng thái hiện tại:** phần hoàn thiện cấu hình đã được xác nhận trong running. Còn xác nhận lưu CORE1 và thu startup thực tế bốn switch; có thể chuẩn bị export Sprint 01 song song, không cần lặp bài kiểm thử hoặc gửi thêm toàn bộ running-config để đi tiếp.

## Chốt hồ sơ thực hành ngày 11/10/2026

Người thực hành yêu cầu dừng thu lại running-config, xác nhận đã export và yêu cầu hoàn thiện báo cáo. [Báo cáo Sprint 01](../sprint-reports/sprint-01-report.md) và [validation](sprint-01-validation.md) đã hoàn thiện trên dữ liệu hiện có. Export thật đã [kiểm tra CRC/XML/cabling](sprint-01-export-inspection-2026-10-11.txt), không có config nhúng. Startup và persistence/restore còn thiếu bằng chứng được chuyển follow-up, không chặn hoàn thiện báo cáo và không yêu cầu gửi lại cấu hình đang chạy.

Review xác nhận tính nhất quán của cấu hình đã nhận với thiết kế, không thay thế runtime/trạng thái traffic. [Bài thử uplink ACCESS1](sprint-01-access1-uplink-test-2026-10-11.md) có hai cụm mất 15 gói và traffic phục hồi; ACCESS2 được bỏ thử theo quyết định người thực hành. Chốt phần thực hành có follow-ups, không đánh dấu toàn bộ 13 ca PASS hoặc đạt đầy đủ DoD Git/ảnh/startup.

## Nguồn đối chiếu kỹ thuật

- Cisco: [Configuring HSRP — default priority 100](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipapp_fhrp/configuration/xe-16-10/fhp-xe-16-10-book/fhp-hsrp.html).
- Cisco: [Line configuration — transport input none](https://netascode.cisco.com/docs/data_models/iosxe/device/line/). Tài liệu giải thích cơ chế; không chứng nhận giá trị mặc định của IOL 2015 đang chạy.
