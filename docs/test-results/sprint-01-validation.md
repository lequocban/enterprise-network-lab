# Sprint 01 — Kết quả kiểm thử Campus Switching

**Ngày tổng hợp:** 11/10/2026, Asia/Saigon.  
**Phạm vi:** NET-101–104, HQ bốn switch/bốn VPCS; expected/actual của 13 ca được trình bày trong mục 3 dưới đây.  
**Người chạy thiết bị:** người thực hành; agent đọc output, kiểm tra cấu hình và export tại local. Không có phiên chạy thiết bị do agent thực hiện.

## 1. Cách đọc kết quả

- **ĐẠT PHẦN ĐỐI CHIẾU:** cấu hình/output hoặc kiểm tra artifact hỗ trợ trực tiếp kết luận được nêu; không tự nghiệm thu mọi mục của ca test.
- **XÁC NHẬN THỰC HÀNH:** người thực hành báo kết quả đúng/ổn và chuyển bước; chưa có log đầy đủ để định lượng hoặc tái kiểm chứng độc lập.
- **MỘT PHẦN / CHƯA ĐỐI CHIẾU:** có kết quả nhưng thiếu một thành phần acceptance gốc.
- **BỎ QUA:** quyết định phạm vi của người thực hành, không tính PASS.

Không quy đổi xác nhận chung thành 10/10 hoặc toàn bộ 13 ca PASS. Ngày tổng hợp/tiếp nhận không thay cho timestamp thiết bị; log thiếu thời gian được ghi rõ. Kết luận bàn giao nằm trong [báo cáo Sprint 01](../sprint-reports/sprint-01-report.md).

## 2. Baseline được đối chiếu

- Cấu hình cuối: [CORE1](../../configs/core/sprint-01/CORE1.cfg), [CORE2](../../configs/core/sprint-01/CORE2.cfg), [ACCESS1](../../configs/access/sprint-01/ACCESS1.cfg), [ACCESS2](../../configs/access/sprint-01/ACCESS2.cfg); nguồn và lịch sử ở [review config](sprint-01-config-review-2026-10-11.md).
- [Trích output runtime đã nhận trong chat](sprint-01-campus-runtime-excerpts-2026-10-11.txt): trunk hai Core, HSRP 10, STP VLAN 20 bốn switch, blocked/inconsistent ports và ping quản trị Access. Đây là các đoạn từ những thời điểm khác nhau, không phải session đồng bộ cuối sprint.
- [Log session HQ](sprint-01-hq-config-session-2026-10-11.txt): bốn running-config, CORE1 cổng test đã shutdown và CORE2 lưu thành công. Hai Access có output lưu thành công ở lượt nhận trước, được ghi trong review.
- [Export Sprint 01](../../eve-exports/sprint-01-hq-2026-10-11.zip): IOL L2 `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`, RAM Core 512/Access 256 MB theo metadata. Image và tài nguyên metadata không phải kiểm thử tải runtime.
- IP đích dùng trong hướng dẫn: IT `10.10.20.20`, HR `10.10.30.20`, ACC `10.10.40.20`, GUEST `10.10.60.20`, mask /24 và gateway `.1` cùng subnet. Endpoint tạm VLAN 50 `10.10.50.20`; không có snapshot cuối riêng của cả bốn VPCS.

## 3. Actual theo ca kiểm thử

### S01-TC-01 — Baseline và restore

- **Expected:** đối chiếu topology/image/config; tái dựng baseline trong lab riêng và kiểm tra VLAN 10/HSRP/ping.
- **Actual:** người thực hành báo các bước đầu đã hoàn thành và ổn; chưa có log riêng xác định import, reload, tái dựng VLAN database và kết quả từng bước. Export Sprint 01 mới được kiểm tra cấu trúc tại local, chưa import.
- **Trạng thái:** XÁC NHẬN THỰC HÀNH cho bước đầu; **chưa nghiệm thu restore Sprint 01**. Không biến kiểm tra ZIP thành restore PASS.

### S01-TC-02 — VLAN và access port

- **Expected:** bảy VLAN đúng tên trên bốn switch; endpoint đúng VLAN 20/30/40/60; VLAN 99 chỉ L2.
- **Actual:** đủ VLAN/tên trong cả bốn running-config VTP transparent; access ports đúng thiết kế; không có SVI 99. Core runtime trunk có đủ VLAN allowed/active. Chưa lưu `show vlan brief` cuối sprint của cả bốn switch.
- **Trạng thái:** ĐẠT PHẦN ĐỐI CHIẾU cấu hình; chưa có bộ VLAN runtime cuối đầy đủ. Bằng chứng: bốn snapshot và trích runtime mục 2.

### S01-TC-03 — Năm link trunk

- **Expected:** hai đầu cả năm link dot1q/native 99/allowed `10,20,30,40,50,60,99`, DTP tắt, không inconsistent.
- **Actual:** mười đầu trunk khớp từ running-config; `switchport nonegotiate` có trên từng đầu. Output hai Core có ba trunk native 99; hai Access có e0/1 tham gia STP, blocked trên bảy VLAN và không có inconsistent port. Không có output trunk đồng bộ hai đầu ở cùng một thời điểm cuối sprint.
- **Trạng thái:** ĐẠT PHẦN ĐỐI CHIẾU cấu hình và runtime đã nhận; không suy ra mỗi trunk đều forwarding mọi VLAN.

### S01-TC-04 — STP root và port role

- **Expected:** CORE1 root, CORE2 secondary cho VLAN 10/20/30/40/50/60/99; Access e0/0 Root/FWD, e0/1 Alternate/Discarding khi bình thường.
- **Actual:** priority cấu hình CORE1 24576, CORE2 28672. Output `show spanning-tree root` người thực hành gửi có CORE1 cost 0 cả bảy VLAN, CORE2 qua e0/0 cost 100. VLAN 20 trên Access đúng e0/0 Root FWD, e0/1 Altn BLK; e0/1 blocked cả bảy VLAN, không có inconsistent port. Output hiển thị `Shr`, chưa có đối chiếu duplex trong hồ sơ.
- **Trạng thái:** ĐẠT PHẦN ĐỐI CHIẾU root/role ở trạng thái bình thường; không nghiệm thu tốc độ rapid transition từ mode RSTP hoặc nhãn `Shr`.

### S01-TC-05 — Cùng VLAN qua hai Access

- **Expected:** PC-IT và PC-ACC tạm VLAN 20/IP `10.10.20.21` ping hai chiều sau warm-up, kèm bảng MAC.
- **Actual:** đã hướng dẫn đổi tạm, kiểm tra rồi hoàn nguyên; người thực hành xác nhận kết quả đúng mong đợi. Chưa có log ping/MAC riêng để ghi số gói/RTT từng chiều. Cấu hình cuối xác nhận port PC-ACC đã về VLAN 40.
- **Trạng thái:** XÁC NHẬN THỰC HÀNH; chưa xác nhận độc lập ngưỡng 10/10 hai chiều.

### S01-TC-06 — Tách VLAN trước routing

- **Expected:** IT VLAN 20 và HR VLAN 30 không liên lạc trước SVI/routing; thử cùng subnet nhưng khác VLAN để quan sát tách broadcast.
- **Actual:** có hướng dẫn và xác nhận chung đã thực hiện; chưa có log riêng lưu trạng thái SVI/ARP của pha trước routing. Sau đó đã tạo SVI, không lặp pha này chỉ để bổ sung hồ sơ.
- **Trạng thái:** XÁC NHẬN THỰC HÀNH, bằng chứng chi tiết hạn chế. Không dùng ping fail này làm Guest isolation/ACL PASS.

### S01-TC-07 — Forwarding từng Core và CEF

- **Expected:** lần lượt dùng gateway .2/.3 để thử VLAN 20↔30 qua từng Core trước khi chuyển về VIP .1.
- **Actual CORE1:** có connected/local routes VLAN 10/20/30, SVI up/up và ARP hai PC đúng. Ping tới HR từ source `10.10.20.2` đạt 5/5, tới IT từ source `10.10.30.2` đạt 5/5, RTT 1/1/2 ms mỗi phép thử; client-to-client ban đầu timeout. Người thực hành xác nhận CEF bật–tắt–bật cho timeout → thành công → timeout.
- **Actual CORE2:** người thực hành báo triệu chứng tương tự, sau workaround báo hoạt động ổn; thiếu log A/B/A và số gói riêng. Cấu hình cuối hai Core đều có `no ip cef` theo quyết định giữ workaround.
- **Trạng thái:** ĐÃ XỬ LÝ BẰNG WORKAROUND theo xác nhận; MỘT PHẦN về định lượng độc lập từng Core. [Biên bản CEF](sprint-01-core1-cef-investigation-2026-10-10.md) giữ chi tiết và giới hạn. Ping do Core tạo ra không thay thế test transit client-to-client.

### S01-TC-08 — SVI, VIP và management

- **Expected:** sáu SVI mỗi Core, .2/.3 và VIP .1 đúng; CORE1 Active/CORE2 Standby sau hội tụ; Access .11/.12.
- **Actual:** bốn config xác nhận địa chỉ, mask, sáu nhóm HSRP/preempt, priority 110 trên CORE1/mặc định 100 trên CORE2; Access `no ip routing`, gateway `10.10.10.1`. Có output HSRP 10 CORE1 Active/CORE2 Standby. Với nhóm 20–60, người thực hành báo ổn sau các bước mở rộng; chưa có output runtime đủ sáu nhóm ở thời điểm cuối.
- **Trạng thái:** ĐẠT PHẦN ĐỐI CHIẾU cấu hình và runtime VLAN 10; XÁC NHẬN THỰC HÀNH cho runtime mở rộng, không suy ra full HA đã được thử.

### S01-TC-09 — Liên VLAN và endpoint Server tạm

- **Expected:** VLAN 20↔30 và VLAN 30↔50 hai chiều qua VIP, bổ sung VLAN 40/60; hoàn nguyên endpoint tạm.
- **Actual:** người thực hành xác nhận ổn sau mở rộng SVI/HSRP 40/50/60. Đã hướng dẫn PC-ACC tạm VLAN 50 kiểm tra gateway, IT/HR, chiều ngược từ IT rồi trả VLAN 40. Người thực hành chuyển sang bước kế tiếp, chưa có log hoặc xác nhận chi tiết từng chiều HR↔VLAN 50. Log bài uplink có phản hồi từ PC-ACC `10.10.40.20` theo bài ping IT→ACC; snapshot cuối xác nhận cổng ACC về VLAN 40.
- **Trạng thái:** XÁC NHẬN THỰC HÀNH cho mở rộng, có log traffic tới VLAN 40; **chưa đủ bằng chứng nghiệm thu riêng VLAN 30↔50 hai chiều, 10/10 mỗi chiều**. VPCS VLAN 50 không phải dịch vụ SRV-INFRA; Guest isolation chưa triển khai.

### S01-TC-10 — Quản trị Access và VTY

- **Expected:** quản trị .11/.12 có return path, baseline VTY console-only và running/startup nhất quán.
- **Actual:** ACCESS1 ping VIP `10.10.10.1` và ACCESS2 `10.10.10.12` đều 5/5, RTT 1/1/1 ms; ACCESS2 ping VIP `10.10.10.1` và ACCESS1 `10.10.10.11` đều 5/5, RTT 1/1/1 ms. Test từ VLAN 20 tới Access đã được hướng dẫn, người thực hành xác nhận chung ổn nhưng thiếu log riêng. Bốn running có `transport input none` và `login` dưới VTY0–4; chưa có startup để so sánh.
- **Trạng thái:** ĐẠT PHẦN ĐỐI CHIẾU ping nội bộ VLAN 10 và VTY; return path liên VLAN theo xác nhận; startup chưa đối chiếu. Không nghiệm thu SSH/AAA.

### S01-TC-11 — Mất uplink ACCESS1

- **Expected gốc:** ping cùng VLAN xuyên Access, ghi STP trước/trong/sau shutdown e0/0 và khôi phục; bổ sung ping liên VLAN riêng.
- **Actual đã chạy:** bài ping liên tục theo hướng dẫn PC-IT→PC-ACC `10.10.40.20`, cùng đợt thử shutdown/no shutdown uplink ACCESS1. Log có 152 sequence, 122 phản hồi và 30 timeout: cụm 25–39 mất 15 gói, cụm 120–134 mất 15 gói. Sau đó lần lượt có 80 và 18 phản hồi liên tiếp. Không có timestamp lệnh gây lỗi hoặc port role trong lúc lỗi; không gán chính xác từng cụm cho shutdown/no shutdown. Cấu hình Access cuối không còn shutdown uplink.
- **Trạng thái:** MỘT PHẦN — traffic phục hồi có bằng chứng; ca STP độc lập cùng VLAN và convergence định lượng chưa đủ. [Log gốc](sprint-01-access1-uplink-ping-2026-10-11.txt), [biên bản](sprint-01-access1-uplink-test-2026-10-11.md). Không suy 15 gói thành 15 giây hoặc kết luận failover mất Core.

### S01-TC-12 — Mất uplink ACCESS2

- **Expected gốc:** như TC-11 cho ACCESS2 e0/0, sau khi ACCESS1 đã khôi phục.
- **Actual:** người thực hành chủ động yêu cầu bỏ bài này ngày tiếp nhận 11/10/2026.
- **Trạng thái:** BỎ QUA THEO PHẠM VI ĐÃ CHỌN; không PASS và không suy ra kết quả từ ACCESS1.

### S01-TC-13 — Baseline cuối và export

- **Expected:** cấu hình thật, endpoints hoàn nguyên, startup nhất quán, export/hash và hồ sơ bàn giao.
- **Actual:** bốn running cuối và năm snapshot lịch sử đã lưu; CORE1 e1/0 shutdown, access ports đúng VLAN, VTY đúng. CORE2/ACCESS1/ACCESS2 có `wr [OK]`; thiếu output lưu CORE1 sau thay đổi cổng test, thiếu cả bốn startup để đối chiếu. [Kiểm tra ZIP](sprint-01-export-inspection-2026-10-11.txt) đạt CRC/XML, tám node/chín link và exact cabling; không có config nhúng. Có báo cáo/validation local, chưa commit/push/PR và chưa có ảnh runtime Sprint 01 riêng.
- **Trạng thái:** ĐẠT kiểm tra export/cấu hình chạy; MỘT PHẦN đối với toàn bộ baseline theo DoD. Giữ follow-up, không yêu cầu gửi lại running hay lặp test trong đợt hoàn thiện báo cáo.

## 4. Kết luận

Chốt phần thực hành Campus theo cấu hình, runtime đã nhận và xác nhận người thực hành; giữ nguyên mức bằng chứng của từng ca. Workaround CEF được ghi tường minh, ACCESS2 không thử theo lựa chọn người thực hành. Startup/persistence/restore, convergence chính xác, một số log hai chiều và Git DoD chưa hoàn tất; xem việc bàn giao có mốc trong báo cáo. Không có kết luận rằng tất cả 13 ca PASS.

## 5. Kiểm tra bộ hồ sơ tại local

Ngày 11/10/2026, agent đã kiểm tra đủ 13 ID ca test không trùng; 156 liên kết nội bộ trong 13 file Markdown thay đổi đều trỏ tới artifact có thật. Bốn cấu hình cuối khớp log session đã nhận sau chuẩn hóa được mô tả; chín snapshot còn đủ và không tạo startup từ running. Số sequence/phản hồi/timeout khớp log gốc. Hash/CRC/XML của ZIP Sprint 01 được kiểm tra, ZIP Sprint 00 giữ nguyên hash lịch sử. Kiểm tra whitespace Git đạt; không chạy thêm test thiết bị hoặc đổi cấu hình lab trong đợt viết báo cáo này.
