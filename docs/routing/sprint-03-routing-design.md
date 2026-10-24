# Sprint 3 — Thiết kế routing sandbox và checkpoint thực hành

**Ngày tiếp nhận:** 18/10/2026, Asia/Saigon.  
**Cập nhật mới nhất:** 21/10/2026, Asia/Saigon.  
**Phạm vi hiện tại:** sandbox NET-301–304: ảnh trước có4router/3VPCS/6link; đã nhận consoleISP1/uplink mới và target, chưa có ảnh/export toàn bộ topology mở rộng.  
**Trạng thái:** [defaultinjection](../test-results/sprint-03-default-route-2026-10-21.md) có EDGE2ASBR/mộtneighborE1, haiBranchO*E2/0metric1/PCtarget5/5 vàtrace đúng. Tiếp Type5chi tiết/withdrawal/recovery; kiểmdefault bằnglookup0.0.0.0/fullRIB, khôngcoi showrouteđích`Networknotintable` làforwardingfail. Chưa cóactualwithdrawal, startupdump/restart/restore.

## 1. Quyết định phạm vi

Theo trình tự thực hành Sprint 3 đã bắt đầu, dùng một lab EVE riêng, tên dự kiến `Enterprise-Network-S03-Routing`, để kiểm chứng OSPF trước khi tích hợp Enterprise. Ảnh buổi2 tại mục6 đã xác nhận các node/cặp cổng hiển thị; tên lab và XML/export vẫn chưa đối chiếu riêng. Sandbox theo thiết kế không nối vào HQ, cloud hay bridge dùng chung với HQ.

Lý do: [thiết kế Enterprise](../network-architecture.md) còn mở đường OSPF Branch qua ISP/firewall/VPN tại WAN-01. Việc tạo adjacency trong sandbox không chứng minh kết nối qua các thành phần này. [Baseline Sprint 02](../sprint-reports/sprint-02-report.md) được giữ riêng; lỗi Po1, ngoại lệ CEF và giới hạn restore chưa thay đổi trạng thái.

Hai tên EDGE1/EDGE2 ở đây chỉ nhận diện node của sandbox. Không áp cấu hình thử vào các Edge của thiết kế Enterprise. Node router trong sandbox không tăng số node/link đã kiểm chứng của HQ.

## 2. Thiết bị và bằng chứng đã tiếp nhận

Nguồn là hai output `show version`, `show ip interface brief` và ảnh form tạo node do người thực hành cung cấp trong chat ngày 18/10/2026. Agent đọc dữ liệu được cung cấp, không trực tiếp điều khiển console.

- Cả hai router chạy `VIOS-ADVENTERPRISEK9-M`, IOSv `15.6(2)T`, release `fc2`.
- CLI cùng báo image `flash0:/vios-adventerprisek9-m`, bốn Gigabit Ethernet interface, bộ nhớ `984321K/62464K`.
- EDGE1 uptime tại mẫu là 2 phút; EDGE2 là 3 phút. Đây là bằng chứng boot/console, chưa phải kiểm thử ổn định dài hạn.
- Ảnh form chọn `Cisco vIOS Router`, image `vios-adventerprisek9-m-15.6.2T`, tạo 2 node với CPU `1`, RAM `1024 MB`, Ethernets `4`. Đây là thông số trong form, chưa có export đối chiếu allocation từng node hoặc tải EVE hiện tại.
- Hai node đều có `GigabitEthernet0/0` đến `GigabitEthernet0/3`; ở mẫu ban đầu tất cả `unassigned`, `administratively down/down`. Trạng thái sau cấu hình ở mục4.

Không cần lấy lại hai bộ version khi chưa có thay đổi. Output ping và `wr` sau cấu hình đã tiếp nhận tại mục4; chưa nhận full startup dump để đối chiếu độc lập.

## 3. Cổng và địa chỉ cho bước cấu hình tiếp theo

**S3-L01:** EDGE1 `GigabitEthernet0/0` ↔ EDGE2 `GigabitEthernet0/0`.

- Subnet: `10.255.0.36/30`, mask `255.255.255.252`.
- EDGE1: `10.255.0.37/30`.
- EDGE2: `10.255.0.38/30`.
- Network `.36`, broadcast `.39`, không cấp cho interface.
- Đây là cấp phát riêng từ phần dự trữ của [IP Plan](../ip-addressing-plan.md), không đè T01–T07 và không thay vai trò các transit Enterprise.
- Đã xác minh hai Gi0/0 đúng IP/up/up và ping trực tiếp hai chiều thành công; chưa có ảnh/export đối chiếu riêng cặp cổng và toàn bộ cabling sandbox.

**Loopback0:** EDGE1 `10.255.255.11/32`, EDGE2 `10.255.255.12/32`, mask `255.255.255.255`; giữ đúng phân bổ router-id trong IP Plan. Việc gán Loopback0 chưa cấu hình router-id OSPF; process/area triển khai ở bước OSPF sau.

Mục tiêu bước này là kết nối IP trực tiếp trên S3-L01 và tạo Loopback0. Chưa tạo static/default route. Hai loopback ở hai router khác nhau chưa có đường đến nhau chỉ từ connected route của S3-L01.

## 4. Tiêu chí kiểm tra và kết quả buổi 1

1. Xác nhận đúng dây Gi0/0 ↔ Gi0/0 trong lab riêng, không có link ra HQ.
2. Sau cấu hình, hai Gi0/0 có đúng IP và `up/up`; hai Loopback0 đúng /32 và `up/up`.
3. Ping warm-up tới IP Gi0/0 peer, sau đó mỗi chiều 10 gói; expected 10/10 ở lượt đo sau warm-up. Ghi actual, không đổi timeout ARP đầu tiên thành kết quả ổn định.
4. Nếu ping lỗi, đối chiếu interface/IP/mask/ARP và cabling trước khi thêm routing protocol.
5. Thu tải EVE và số node đang chạy trước mở rộng; thông số CPU/RAM trên form không thay phép đo tải.
6. Khi cấu hình/traffic đích đã đạt, dùng `wr` từng router, chờ `[OK]`; chưa có bằng chứng lưu thì không ghi persistence PASS.

**Actual tiếp nhận ngày18/10/2026, từ output người thực hành trong chat:**

- EDGE1 Gi0/0 `10.255.0.37`, Loopback0 `10.255.255.11` đều up/up. Ping tới `.38`: warm-up5/5, mẫu10/10, RTT min/avg/max1/1/2ms; `wr` có `[OK]`.
- EDGE2 Gi0/0 `10.255.0.38`, Loopback0 `10.255.255.12` đều up/up. Ping tới `.37`: warm-up5/5, mẫu10/10, RTT min/avg/max1/1/2ms; `wr` có `[OK]`.
- EVE `free -h`: RAM total7.7Gi, used1.4Gi, free5.5Gi, shared6.0Mi, buff/cache793Mi, available6.0Gi; swap4.0Gi, used0B.
- EVE `uptime`: giờ hiển thị20:44:29, uptime1:06, 2 user sessions, load average0.75/0.70/0.43. Chưa xác minh timezone của clock EVE; không đổi giờ này thành Asia/Saigon. Số users là phiên đăng nhập, không phải số router đang chạy.
- Danh sách đầy đủ node đang bật chưa được người thực hành nêu. Mẫu tài nguyên không chứng minh tải dài hạn hoặc khả năng chạy đồng thời toàn Enterprise/4IOSv.

**Kết luận:** checkpoint IP/link/ping hai chiều và lưu cấu hình hai router ĐẠT trong phạm vi buổi1. Không cần lặp ping/wr khi chưa có thay đổi. Chưa OSPF, chưa đối chiếu full running/startup hoặc restart/import/restore; không ghi các phần này PASS. Tên lab/isolation và cabling toàn bộ chưa có ảnh/export xác nhận riêng; giữ giới hạn đó trong hồ sơ, không suy từ ping. NET-301 chưa DONE.

## 5. Mốc mở rộng và tích hợp

**Hạn bắt buộc bổ sung18/10/2026:** theo [mốc tích hợp Enterprise](../enterprise-integration-milestones.md), cuối Sprint3 hoàn thành thiết kế INT-03 trước triển khai Enterprise Sprint4; cuối Sprint4 Campus/WAN cùng topology INT-04; cuối Sprint5 HQ qua FW/Edge INT-05; cuối Sprint6 client HQ ↔ hai Branch qua VPN INT-06. Đây là kế hoạch chưa nghiệm thu; checkpoint hai IOSv ở mục4 chỉ là kết quả chuẩn bị. Sandbox tiếp tục phục vụ bài học hiện tại, không thay sản phẩm cuối.

Đích học OSPF dự kiến là 4 router trong miền OSPF: HQ Area 0, Branch 1 Area 10, Branch 2 Area 20; thêm router ngoài miền ở bài default injection. Cổng/IP còn lại chỉ chốt khi đến bước mở rộng và phải cập nhật thiết kế trước áp dụng.

Trước tích hợp Enterprise phải chốt L2/cổng T01/T02, default/return routes Core–FW–Edge, transport OSPF tới Branch và vai trò ABR/ASBR thực. Bằng chứng sandbox, cấu hình Enterprise và nghiệm thu liên site được ghi riêng; WAN-01 vẫn mở.

## 6. Buổi 2 — cabling/capability đã đối chiếu, IP mở rộng chờ thực hành

**Nguồn tiếp nhận20/10/2026:** output hai Branch, `free -h`/`uptime` EVE và [ảnh topology do người thực hành cung cấp](../../screenshots/sprint-03/routing-sandbox-cabling-2026-10-20.png). Không suy trạng thái running/link forwarding từ màu dây hoặc biểu tượng node.

- BR1-R1/BR2-R1 cùng IOSv15.6(2)T fc2, image CLI `flash0:/vios-adventerprisek9-m`, memory `984321K/62464K`, có Gi0/0–Gi0/3. Các cổng Branch đều unassigned/admin down/down ở mẫu nhận; uptime tương ứng3/4phút. Chưa có output IP hoặc OSPF trên Branch.
- Ảnh có đúng4router/3VPCS/6link và nhãn cổng khớp thiết kế. S3-L01 vẫn EDGE1 Gi0/0 ↔ EDGE2 Gi0/0. Năm link thêm được ghi bên dưới; ảnh không thay export hoặc kiểm tra data plane.
- EVE RAM total7.7Gi, used1.6Gi, free5.3Gi, shared6.0Mi, buff/cache827Mi, available5.9Gi; swap4.0Gi/used0B. Uptime hiển thị09:36:50, up14min, 1user, load1.81/1.36/0.63. Timezone EVE chưa đối chiếu; thời gian này giữ nguyên như output, không quy đổi. Ảnh và các mẫu console chưa phải phép đo ổn định liên tục của mọi node đồng thời.

**Cổng thực tế trên ảnh; địa chỉ dưới được dành cho bước cấu hình, chưa phải actual:**

- S3-L02 `10.255.0.40/30`: EDGE1 Gi0/1 `.41` ↔ BR1-R1 Gi0/0 `.42`.
- S3-L03 `10.255.0.44/30`: EDGE1 Gi0/2 `.45` ↔ BR2-R1 Gi0/0 `.46`.
- S3-L05 `10.255.0.52/30`: EDGE2 Gi0/2 `.53` ↔ PC-HQ eth0 `.54`, gateway `.53`.
- BR1 LAN `10.20.10.0/24`: BR1-R1 Gi0/1 `.1` ↔ PC-BR1 eth0 `.20`, gateway `.1`.
- BR2 LAN `10.30.10.0/24`: BR2-R1 Gi0/1 `.1` ↔ PC-BR2 eth0 `.20`, gateway `.1`.
- Loopback0 BR1/BR2: `10.255.255.21/32` và `10.255.255.22/32` theo IP Plan. S3-L04 `.48/30` và Loopback110 chưa triển khai ở bước này; EDGE2 Gi0/1 dành cho ISP1 sau.

**Trình tự học và kiểm chứng:** cấu hình BR1 LAN + transit S3-L02 + Loopback0 trước, đọc connected/local routes; test PC-BR1 → gateway và router transit hai chiều. Đạt rồi làm BR2/PC-HQ, test từng link trực tiếp; sau đó bật single-area OSPF trên4router để trao đổi các mạng xa và test3cặp client hai chiều. Chưa có OSPF thì không dùng ping client xuyên nhiều router làm tiêu chí phải đạt.

Người học yêu cầu giải thích tổng quan, mục đích, đường đi/đường về và kiến thức liên quan trước lệnh. Dùng các checkpoint ngắn, giải thích output thật; không đưa khối cấu hình cả hệ thống một lần hoặc lặp kiểm tra đã đủ bằng chứng khi không có thay đổi.

## 7. Buổi 2 — checkpoint IP trực tiếp đạt, bắt đầu Single-Area OSPF

**Actual tiếp nhận20/10/2026 qua output người thực hành:**

- EDGE1 Gi0/0 `.37`, Gi0/1 `.41`, Loopback0 `.11` up/up. EDGE1 → BR1 `.42`10/10 RTT1/1/3ms; BR1 → EDGE1 `.41`10/10 RTT1/1/2ms.
- BR1 Gi0/0 `.42`, Gi0/1 `10.20.10.1`, Loopback0 `10.255.255.21` up/up; connected/local routes khớp LAN/transit/loopback. PC-BR1 showip xác nhận `10.20.10.20/24`, GW`10.20.10.1`, pingGW5/5.
- EDGE1 → BR2 `.46`10/10 RTT1/1/2ms; BR2 → EDGE1 `.45`10/10 RTT2/2/3ms. BR2 Gi0/0 `.46`, Gi0/1 `10.30.10.1`, Loopback0 `10.255.255.22` up/up.
- EDGE2 Gi0/0 `.38`, Gi0/2 `.53`, Loopback0 `.12` up/up; Gi0/1 còn shutdown/unassigned, chưa có ISP1.
- PC-BR2 → GW`10.30.10.1`5/5; PC-HQ → GW`10.255.0.53`5/5. Chưa có showip hai PC này trong bộ output mới, không tự tạo source-IP evidence.
- Người thực hành xác nhận đã hoàn thành phần BR1 và đã wr/save trên các thiết bị sau phần BR2/PC-HQ; ghi nhận đúng mức xác nhận, không tạo log `[OK]` hoặc startup dump chưa nhận. Không yêu cầu lưu lại phần IP đã đạt khi chưa đổi.

**Kết luận:** phần địa chỉ và kết nối trực tiếp đã đủ để chuyển sang bài OSPF. Chưa đạt S03-TC-02/03, chưa có bằng chứng client đi qua nhiều router.

**Cấu hình OSPF đích của pha hiện tại, chưa thực thi theo evidence:** process10; router-id EDGE1 `.11`, EDGE2 `.12`, BR1 `.21`, BR2 `.22` trong `10.255.255.0/24`; tất cả interface tham gia Area0. Dùng passive mặc định, chỉ mở neighbor trên EDGE1 Gi0/0–2, EDGE2 Gi0/0 và BR1/BR2 Gi0/0. Ba link router–router dùng network type point-to-point ở cả hai đầu. Gi0/2 EDGE2, Gi0/1 hai Branch và Loopback0 cả4router được advertise nhưng giữ passive. Không bật OSPF trên EDGE2 Gi0/1 hoặc cổng unused.

Expected sau cấu hình đủ4router: EDGE1 có3neighbor FULL, EDGE2/BR1/BR2 mỗi node1neighbor FULL; remote LAN học bằng OSPF. Trên EDGE1, LANBR1 `10.20.10.0/24` qua `.42`, LANBR2 `10.30.10.0/24` qua `.46`; chưa thêm default/summary/multi-area. Kiểm neighbor và RIB trước test3cặp client hai chiều; chỉ lưu OSPF mới sau kiểm chứng.

**Điều chỉnh cách hướng dẫn theo yêu cầu người học20/10:** đã có nền tảng cơ bản; tập trung mục tiêu, quyết định thiết kế và điểm chẩn đoán, giải thích sâu khi có câu hỏi/output cần phân tích. Không lặp giải thích subnet/gateway/C-L cơ bản ở mỗi checkpoint.

## 8. Buổi 4 — chuyển Area10 trước, trạng thái planned

Đã đủ baseline chức năng để bắt đầu chuyển area có kiểm soát. Đây vẫn là bài học sandbox, không đóng INT-03 hoặc chứng minh tích hợp Enterprise.

- Đổi EDGE1 Gi0/1 và BR1 Gi0/0 từ process10/Area0 sang process10/Area10; đổi cả BR1 Gi0/1 và Loopback0 sang Area10.
- Giữ nguyên IP/router-id, network type point-to-point trên transit và passive LAN/Lo0. Không clear process, đổi timer hoặc thêm static route để làm traffic đạt.
- EDGE1 Gi0/0 tới EDGE2, Gi0/2 tới BR2 và Lo0 vẫn Area0. EDGE2/BR2 vẫn giữ toàn bộ Area0 ở bước này; chưa chuyển Area20.
- Expected sau đổi đủ hai phía: EDGE1 có Area0/10 và là ABR, vẫn đủ3FULL; BR1 có mọi interface OSPF ở Area10 và một FULL. Trong lúc mới đổi một đầu transit, adjacency tạm mất là expected.
- EDGE2 học LANBR1 `10.20.10.0/24` dạng O IA, BR1 học mạng PC-HQ `10.255.0.52/30` dạng O IA; metric dự kiến vẫn3 với cost baseline. EDGE1 vẫn học LANBR1 intra-area O vì trực tiếp tham gia Area10, không yêu cầu route đó thành O IA trên EDGE1.
- Kiểm PC-HQ ↔ PC-BR1 sau chuyển; kết quả phải có nguồn/chiều thật. Chưa có actual cho các mục này.

Rollback: trả bốn interface vừa đổi về process10/Area0, giữ nguyên các thiết lập khác, kiểm FULL/route/traffic trước lưu. Hai Edge đã xác nhận wr sau baseline cost; nếu BR1 chưa lưu OSPF single-area thì wr và chờ `[OK]` trước khi chuyển area trên BR1. Chỉ lưu cấu hình multi-area sau khi đã kiểm chứng phục hồi traffic.

## 9. Area10 đã có bằng chứng, chuẩn bị Area20

Nguồn/actual21/10/2026 tại [checkpoint multi-area](../test-results/sprint-03-multi-area-2026-10-21.md): EDGE1 đủ3FULL; BR1 ba interface đềuArea10; EDGE2LANBR1 và BR1mạngPC-HQ đềuinter-area/metric3, HQ↔BR1hai chiều5/5. Không yêu cầu làm lại bướcArea10 hoặc đổi network type P2P đã chọn.

Bước kế: EDGE1Gi0/2 + BR2Gi0/0/Gi0/1/Lo0 đổiprocess10/Area0→Area20. Giữ IP/router-id/cost/type/passive, EDGE1Gi0/0/Lo0 và EDGE2Area0; BR1Area10. Expected EDGE1ABR0/10/20/3FULL, BR2Area20/1FULL; LANBR2 học OIA ở E2/BR1 và LANBR1/mạngPC-HQ học OIA ởBR2. Kiểm area/RIB/LSDB và traffic trước lưu hoặc sang summarization; chưa có actual cho phầnArea20.

## 10. Multi-area đạt chức năng, chuẩn bị summarization

Checkpoint mới nhất21/10 tại [kết quả multi-area](../test-results/sprint-03-multi-area-2026-10-21.md) thay trạng thái planned mục8/9: EDGE1Area0/10/20/3FULL, BR2Area20, inter-areaRIB/Type3 và6chiềuPC5/5. Console45phút/loggingsynchronous/wr theo xác nhận người thực hành. LSDB còn LSAType1Area0 cũ trong mẫu chuyển đổi, chưa có mẫu aging tiếp theo; không coi toàn bộLSDB đã sạch hoặc tự reset OSPF.

**Địa chỉ thử mới dành cho NET-303, chưa cấu hình theo evidence:**

- BR1 Loopback110 `10.20.11.1/24`, OSPF10/Area10; góp prefix `10.20.11.0/24` cùng LAN `10.20.10.0/24`.
- BR2 Loopback110 `10.30.11.1/24`, OSPF10/Area20; góp prefix `10.30.11.0/24` cùng LAN `10.30.10.0/24`.
- Loopback110 mô phỏng mạng, không có thêm node/VLAN/server thật. Giữ Loopback0/router-id hiện có. Dùng `ip ospf network point-to-point` trên Loopback110 để remote route giữ/24 thay vì loopback/32 mặc định; interface mới giữ passive theo `passive-interface default`.
- Trước range, EDGE2 phải học4route OIA /24 nêu trên, PC-HQ ping tới hai địa chỉLoopback110 được. Chưa có actual, chưa bật range trong bước chuẩn bị.
- Sau kiểm chứng trước range, tại ABR EDGE1 gộpArea10 thành `10.20.10.0/23`, Area20 thành `10.30.10.0/23`. Các /23 bao trọn chính xáchai /24 tương ứng; không gộp/16 hoặc bao thêm DMZ/HQ. Kiểm RIB/Type3 ngoài area và ping các đích thật; các route transit/Lo0 nằm ngoài range vẫn tồn tại.
- Rollback bài range bằng gỡ đúng area range mới thêm; kiểm route chi tiết trở lại. Bài tắt một contributor chỉ làm sau checkpoint summary hoạt động và hoàn nguyên trước lưu; không thêm static route để che thiếu prefix.

## 11. Buổi 6 — ISP mô phỏng và default injection, planned

Kế thừa summary/Lo110 đã khôi phục theo [actual](../test-results/sprint-03-summary-2026-10-21.md). Thêm ISP1 cùng imageIOSv15.6(2)T fc2, dự kiếnCPU1/RAM1024MB/4Ethernet; đo tài nguyên khi bật5router. Đích sau mở rộng là5IOSv+3VPCS/7link, chưa ghi là đã triển khai khi chưa có output/ảnh mới.

- S3-L04 `10.255.0.48/30`: EDGE2 Gi0/1 `10.255.0.49` ↔ ISP1 Gi0/0 `10.255.0.50`. Gi0/1 EDGE2 đã dành cho link này, không đổi Gi0/2PC-HQ.
- ISP1 Lo0 `10.255.255.101/32` theo IPPlan; Lo100 `192.0.2.10/32` là đích mô phỏng ngoài miền OSPF, không phải server/Internet public thật.
- EDGE2 có static default tới`.50`; ISP1 return route qua`.49` cho `10.20.0.0/16`, `10.30.0.0/16`, `10.255.0.0/24`, `10.255.255.0/24` đúng bài sandbox. Các route này không phải cấu hình provider/NAT/VPN của Enterprise.
- ISP1 không chạy OSPF; không bật OSPF trênEDGE2Gi0/1. ConsoleISP1 dùng exec-timeout45/loggingsynchronous theo thiết lập người học chọn.
- Kiểm link trực tiếp hai chiều và EDGE2 → `192.0.2.10` với source`.49`, staticdefaultS* và PC-HQ tới đích để xác nhận return path. Chưa phát default vào OSPF ở checkpoint chuẩn bị.
- Chỉ sau underlay/static/return path đạt mới dùng `default-information originate` trênEDGE2, khôngalways/route-map; kiểmBranch O*E2 vàType5, không leak route/32target vào OSPF. Failure withdrawal/khôi phục ở buổi7. Chưa có actual cho bướcISP/default này.

Rollback checkpoint chuẩn bị: gỡ đúng staticdefault/return routes và tắt các interface/node mới khi cần, giữ baseline OSPF/summary/các cổng cũ. Không gỡ area hoặc range để sửa bài upstream; không save pha lỗi.
