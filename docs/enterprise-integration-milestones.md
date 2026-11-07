# Các mốc tích hợp topology Enterprise

**Ngày bổ sung:** 18/10/2026, Asia/Saigon.  
**Rà soát hoàn thiện tài liệu:** 20/10/2026, Asia/Saigon; không phát sinh kết quả chạy lab từ lần rà soát.  
**Trạng thái 23/10/2026:** INT-03 đạt ở mức thiết kế/bàn giao; INT-04/05/06/09/11 chưa nghiệm thu.  
**Nguồn phạm vi:** [Roadmap](enterprise-network-design-implementation-roadmap.md), [Architecture](network-architecture.md), [IP Plan](ip-addressing-plan.md), [bàn giao Sprint 02](sprint-reports/sprint-02-report.md).

## 1. Đích cuối và nguyên tắc kế thừa

Dự án có **một topology Enterprise tích hợp làm sản phẩm cuối**, kế thừa Campus HQ Sprint 1–2, mở rộng Edge/ISP/Branch, firewall/DMZ/VPN, rồi services/monitoring/automation. Lab routing Sprint 3 là môi trường học và kiểm chứng trung gian; hoàn thành lab đó không thay nghiệm thu topology chung.

Mỗi bản phát triển của topology chung phải có tên lab và đường dẫn UNL thực tế trong báo cáo; chốt định danh ở INT-03 và bắt đầu dùng ở INT-04. Các export theo sprint là phiên bản của cùng hướng phát triển, không phải những sản phẩm cuối độc lập. Không cần ghi đè lab cũ hoặc xóa sandbox để chứng minh đã tích hợp.

- Giữ nguyên bản gốc Sprint 02 và mọi bằng chứng cũ. ZIP ngày18/10 chỉ có topology; [backup trước WAN ngày23/10](test-results/sprint-02-before-wan-export-2026-10-23.md) và [gói seed Sprint4](../eve-exports/README.md#bản-khởi-đầu-sprint-4--23102026) có đủ tám payload. Import bản phát triển riêng, bật config set và smoke test Campus trước sửa các phần kế thừa; restore thực tế chưa được chứng minh chỉ từ kiểm ZIP.
- Từ INT-04, triển khai phần mới vào topology chung. Lab thử ngoài chỉ là phụ trợ và phải có mốc chuyển kết quả về topology chung.
- Một UNL có đủ biểu tượng chưa chứng minh liên thông; các mốc cần cả control plane, data plane và policy tương ứng.
- Có thể bật từng nhóm node để phù hợp RAM; khi nghiệm thu một đường đi, mọi node trên đường đó phải chạy đồng thời. Ghi nhóm node/tải tại lúc đo, không ghép những ảnh từ các lần chạy khác nhau thành một bài test end-to-end.
- Cổng, IP, VLAN và vai trò gateway tuân thủ IP Plan. Các /30 `S3-Lxx`, vai trò ABR/ASBR và đường trực tiếp trong sandbox chỉ được chuyển sang Enterprise sau khi mapping thiết kế đã được review.

Người thực hành triển khai và cung cấp output thật; agent hỗ trợ thiết kế, đọc evidence và cập nhật hồ sơ. Trạng thái PASS chỉ dựa trên actual có nguồn; viết tài liệu này không tự làm hoàn thành một mốc.

## 2. INT-03 — Chốt thiết kế tích hợp, cuối Sprint 3

**Hạn theo trình tự:** thực hiện tại buổi8 Sprint3; phải đạt trước khi triển khai topology Enterprise của Sprint4. Không cộng bước cấu hình Enterprise chưa có vào kết quả sandbox.

**Việc bắt buộc:**

- Chỉ định topology chung: tên lab, đường dẫn UNL, nguồn export/config HQ và cách quay lại baseline. Các tên lab hiện trong kế hoạch chỉ là dự kiến cho tới khi ghi định danh thực tế.
- Hoàn thiện mapping từng node/interface/IP của sandbox sang vai trò Enterprise: giữ lại, điều chỉnh hay chỉ dùng để học; mỗi thay đổi có lý do và test tương ứng. Không copy nguyên EDGE1/EDGE2 sandbox vào config đích.
- Chốt L2/cổng cho T01 VLAN90 và T02 VLAN91, các link T03–T07, Branch LAN và đích Internet test. Node L2 bổ sung phải có inventory/tài nguyên; cổng chưa tạo phải ghi planned.
- Chốt đường Core → FW → Edge và return routes, gateway ASA outside, nơi phát default, cơ chế nhận biết upstream hỏng. Không tự thêm VIP hoặc bypass FW ngoài IP Plan.
- Chốt endpoint/type VPN và transport phù hợp image; xác định phạm vi OSPF cuối, area/ABR/ASBR, prefix summary và default. Nếu dùng static routing qua VPN, phải ghi OSPF chạy ở đâu trong topology chung và phần nào của bài multi-area chỉ là bài học. Không ghi OSPF xuyên VPN đã đạt nếu thiết kế cuối không dùng nó.
- Kiểm tra capability cần thiết trên image Core/router/FW đã chọn trước khi chốt phương án phụ thuộc. Giữ ngoại lệ CEF và lỗi Po1 có phạm vi rõ; không áp workaround IOL sang mọi IOSv.
- Chốt traffic matrix cho INT-04/05/06: nguồn/đích, dịch vụ hoặc ICMP dùng thử, chiều, permit/deny, route đi/về và cách gây lỗi/hoàn nguyên. Dịch vụ chưa triển khai có thể dùng endpoint thử và phải ghi đúng giới hạn.

**Nghiệm thu:** bản thiết kế tích hợp có mapping NET-301–304 và các quyết định WAN-01 thuộc phạm vi INT-03; IP Plan, architecture, physical/logical nhất quán. Không còn TBD quyết định đường đi cần dùng ở Sprint4–6. Artifact dự kiến: `docs/routing/enterprise-integration-design.md`, tạo khi review có nội dung thật.

**Nếu chưa đạt:** phần học OSPF có thể ghi hoàn thành theo bằng chứng, nhưng INT-03 giữ OPEN; chưa chuyển sang triển khai Enterprise Sprint4. Không đổi hạn thành “tích hợp sau” mà thiếu phương án và điều kiện hoàn tất cụ thể.

## 3. INT-04 — Một topology chung có Campus và WAN, cuối Sprint 4

**Nghiệm thu kỹ thuật 07/11/2026: ĐẠT có follow-ups**, theo [báo cáo Sprint4](sprint-reports/sprint-04-report.md) và [restore ZIP cuối](test-results/sprint-04-final-restore-2026-11-07.md). Campus/WAN/Branch chung 17 node/21 network, BGP/failure/recovery/transport và hồi quy đạt. Startup BGP sau import đã được lưu và restart đối chiếu, quy trình được ghi rõ; đo convergence/loss và lỗi IOL kế thừa giữ follow-up. Các trạng thái 23/10 ở phần lịch sử phản ánh mốc trước triển khai. INT-05/06/09/11 chưa nghiệm thu.

**Phụ thuộc:** INT-03 đạt; thực hiện trong NET-401–404, trước INT-05.

**Việc bắt buộc:**

- Tạo bản phát triển từ HQ Sprint02 với cấu hình/VLAN thật, xác nhận chức năng Campus đại diện vẫn hoạt động; giữ nguyên mốc Sprint02 để đối chiếu.
- Thêm EDGE1/EDGE2, ISP1/ISP2, BR1/BR2 và các link WAN đã chốt **trong cùng topology chung**. Inventory/cổng/export phản ánh đúng node thật; vị trí FW/DMZ chưa chạy giữ planned.
- Triển khai underlay/Edge/BGP và đích Internet test trên topology chung; provider chỉ mang các route được thiết kế, không dùng OSPF doanh nghiệp trên ISP hoặc leak LAN để che thiếu VPN.
- Kiểm thử từ Edge tới đích test, BGP/route/next-hop primary-backup và failure/recovery đúng NET-404. Branch kiểm tra IP WAN/transport cần cho VPN sau này.
- Test hồi quy Campus sau thay đổi topology hoặc stop/start. HQ chưa đi qua FW ở mốc này thì ghi chưa có đường ra ngoài; không nối tắt Core–Edge để ghi HQ Internet PASS.

**Nghiệm thu:** Campus và các router WAN cùng UNL, các bài underlay/Edge chạy ngay trên topology đó; export/config/ảnh/test cùng phiên bản. Mốc này chưa chứng minh client HQ tới Branch hoặc policy firewall.

**Nếu chưa đạt:** NET-401–404 chỉ có kết quả lab riêng không đủ đóng INT-04; chưa chuyển nghiệm thu firewall tích hợp. Có thể tiếp tục chuẩn bị image/tài liệu độc lập.

## 4. INT-05 — HQ đi qua firewall/Edge, cuối Sprint 5

**Phụ thuộc:** INT-04 đạt; thực hiện NET-501–504, phải đạt trước kiểm thử VPN tích hợp Sprint6.

**Việc bắt buộc:**

- Tích hợp FW1, T01/T02 và DMZ theo thiết kế; dùng đường đi HQ → Core → FW → Edge → ISP/test target, return route đúng. Không giữ đường tắt chưa được thiết kế vượt policy firewall.
- Kiểm tra từ client VLAN HQ ra đích mô phỏng ngoài, NAT/PAT/session/counter và traffic trả về; source/target thực được ghi rõ. ISP test không được gọi là Internet public thật.
- Kiểm thử IT/HR/Accounting/Guest theo traffic matrix đã chốt; tối thiểu có permit hợp lệ, Guest bị chặn tới internal và Internet → internal bị từ chối khi không có ngoại lệ chủ đích.
- Kiểm thử DMZ: dịch vụ được công bố tới được từ vùng ngoài, các luồng DMZ → internal bị hạn chế đúng policy; không suy policy đạt chỉ từ ping một chiều.
- Đối chiếu vị trí NAT và kế hoạch NAT exemption cho Sprint6. Test hồi quy Campus và đường Edge sau tích hợp.

**Nghiệm thu:** đường client HQ qua FW/Edge và DMZ/policy có bằng chứng end-to-end trên topology chung. Chưa yêu cầu HQ–Branch trước khi VPN chạy; các trường hợp này vẫn ghi pending INT-06.

**Nếu chưa đạt:** không dùng ping giữa VPN endpoint hoặc cấu hình crypto để tuyên bố HQ đã có kết nối liên site hoàn chỉnh.

## 5. INT-06 — HQ và cả hai Branch liên thông qua VPN, cuối Sprint 6

**Phụ thuộc:** INT-05 đạt; là điều kiện bắt buộc đóng phần tích hợp mạng HQ/WAN/Security/Branch và bắt đầu nghiệm thu dịch vụ liên site Sprint7.

**Việc bắt buộc:**

- Tích hợp VPN HQ–BR1 và HQ–BR2 vào topology chung theo NET-601–603; kiểm tra route/return route, NAT exemption, firewall policy và MTU theo thiết kế đã chốt.
- Áp dụng phần routing của Sprint3 đã được map sang Enterprise, kiểm tra neighbor/route/LSDB/summary/default ở đúng nơi cần dùng. Nếu thay phương án, cập nhật mapping/thiết kế và retest trước khi nghiệm thu; sandbox PASS không thay kết quả routing tích hợp.
- Kiểm thử **client HQ ↔ client Branch1** và **client HQ ↔ client Branch2**, mỗi chiều riêng, dùng dịch vụ được traffic matrix cho phép. Client phải ở phía sau gateway, không chỉ là ping hai router WAN/tunnel endpoint.
- Ghi evidence SA/encrypted/decrypted counters hoặc capture thích hợp chứng minh traffic test thực đi qua VPN. Kiểm tra luồng bị cấm theo policy không được vô tình thông qua tunnel.
- Gây lỗi có kiểm soát với từng tunnel/transport đã thiết kế, quan sát route/traffic rồi khôi phục; bảo đảm không âm thầm rơi về đường rõ ngoài VPN. Mất đường duy nhất thì mất kết nối là expected; chỉ yêu cầu tiếp tục traffic nếu thiết kế thật có đường dự phòng.
- Test lại Campus, HQ ra đích ngoài, DMZ và các luồng deny đại diện sau thêm VPN. Lưu config của các node đã sửa và xác nhận phục hồi baseline.

**Nghiệm thu:** hai cặp site đều có bằng chứng traffic hai chiều qua VPN trên **cùng topology Enterprise kế thừa HQ Sprint1/2**; chức năng trước đó vẫn đúng. Đóng INT-06 không tự đóng lỗi Po1 hoặc chứng minh HA toàn hệ thống.

**Nếu chưa đạt:** giữ INT-06 OPEN; không ghi “đã tích hợp toàn bộ HQ/Branch”, không đẩy việc chứng minh liên site sang cuối portfolio. Việc chuẩn bị services độc lập có thể tiếp tục, nhưng chưa nghiệm thu services liên site dựa trên kết nối chưa đạt.

## 6. INT-09 và INT-11 — Hoàn thiện cùng hệ thống

**INT-09, cuối Sprint9:** Sprint7 đưa SRV-INFRA vào VLAN50/cổng đã chốt trên topology chung; Sprint8 monitoring và Sprint9 automation dùng inventory thiết bị của topology đó. Phải có test DHCP/DNS/NTP/Syslog/monitoring/backup đúng scope từng story, qua đường HQ/Branch khi có yêu cầu liên site; test lại luồng INT-06 sau đổi service/policy. Không coi inventory chỉ chứa router sandbox là inventory toàn Enterprise.

**INT-11, cuối Sprint11:** portfolio dùng topology chung đã kiểm chứng, hoàn thiện troubleshooting Sprint10, bộ export/config/inventory và hướng dẫn tái dựng. Thử restore trên bản riêng và ghi phạm vi/actual; CRC hợp lệ không thay restore. Nếu thiếu tài nguyên chạy tất cả đồng thời, ghi nhóm đã thử, điều kiện và giới hạn, không tuyên bố full-load PASS. Lỗi tồn đọng phải có trạng thái, phạm vi ảnh hưởng và follow-up thật.

Sandbox có thể giữ làm tài liệu học; nó không thay artifact nghiệm thu cuối. Tại INT-11, các thành phần thuộc scope chưa tích hợp phải được xử lý hoặc điều chỉnh scope một cách tường minh trước kết luận dự án hoàn thành.

## 7. Bằng chứng và quy tắc đóng mốc

Mỗi báo cáo sprint thêm một mục cho mốc liên quan với: **ID, trạng thái, topology/version, phụ thuộc, expected/actual, artifact và việc còn thiếu**. Dùng CHƯA BẮT ĐẦU / ĐANG LÀM / CHỜ XỬ LÝ / ĐẠT; ghi ngày giờ/múi giờ thật khi test.

Bộ evidence tối thiểu khi nghiệm thu runtime:

- Tên lab/đường dẫn UNL, inventory/cabling/IP thực và nhóm node đang bật, tài nguyên tại mốc đo.
- Config thật của node thay đổi, snapshot baseline để rollback; `wr [OK]` khác với đối chiếu startup hoặc restart/restore.
- Route/neighbor/policy/counter cần thiết; test source/destination/chiều, warm-up, số gói hoặc kết quả dịch vụ. Permit và deny đều có expected.
- Thao tác gây lỗi, trạng thái trước/trong/sau, hoàn nguyên và test hồi quy. Chỉ báo thời gian hội tụ khi đo có phương pháp/timestamp phù hợp.
- Export đúng phiên bản kèm hash và mô tả payload; output/ảnh/capture thật, không lấy bằng chứng sandbox gắn nhãn topology chung.

Các ca dùng ID `INT-04-TC-*`, `INT-05-TC-*`, `INT-06-TC-*`, `INT-09-TC-*`, `INT-11-TC-*`, theo [mẫu test case](test-results/test-case-template.md). Danh mục nguồn/đích/port cụ thể phải có ở INT-03; không đợi tới cuối Sprint6 mới viết tiêu chí liên site.

**Không đóng mốc khi thiếu điều kiện bắt buộc.** Nếu phát hiện sai hướng: dừng phần phụ thuộc, ghi gap, cập nhật thiết kế/IP/config/test nhất quán, hoàn nguyên thay đổi gây lỗi và retest phần bị ảnh hưởng. Các ca đã đạt không đổi thành chưa chạy khi không có thay đổi liên quan.

**INT-03 ĐẠT — 23/10/2026:** [baseline tích hợp](routing/enterprise-integration-design.md) đã chốt tên/path lab, mapping T01–T07/W08/W09, IP/ASN, Core–FW–Edge, BGP underlay, SVTI/OSPF overlay, route/return/NAT và traffic/failure matrix; [IP Plan](ip-addressing-plan.md#9-phụ-lục-int-03-cho-topology-tích-hợp--thiết-kế-23102026) và [physical topology](../diagrams/physical-topology.md) dùng cùng mapping. Các subnet transit/tunnel được kiểm không chồng lấn. ASAv/Core parser preflight và VTI pilot có evidence; seed `Enterprise-Network-Integrated` đã import trong `LAB-NETWORK/test_import` và người thực hành xác nhận khôi phục thành công sau Stop/Wipe/Start, CORE1 có output trực tiếp. Đây là **nghiệm thu thiết kế/bàn giao**, chưa phải WAN/BGP/FW/VPN runtime trên topology chung. Kiểm MTU, BGP, tracking và failover ở các mốc triển khai tương ứng. INT-04/05/06/09/11 chưa nghiệm thu.
