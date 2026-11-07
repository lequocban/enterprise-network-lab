# INT-03 — Đối chiếu routing sandbox với topology Enterprise

**Quyết định INT-03 chốt 23/10/2026 — baseline để bắt đầu INT-04:** bản phát triển là `LAB-NETWORK/test_import/Enterprise-Network-Integrated.unl`, kế thừa HQ Sprint 02 đã khôi phục; HQ gốc và sandbox Sprint 3 giữ riêng. Dùng EVE Bridge T01/VLAN90 cho CORE1 e1/2, CORE2 e1/0, FW Gi0/0; Bridge T02/VLAN91 cho FW Gi0/1, EDGE1 Gi0/0, EDGE2 Gi0/0. FW Gi0/2 dành DMZ. T03–T07 và W08/W09 dùng đúng cổng/IP ở [physical topology](../../diagrams/physical-topology.md) và [IP Plan mục 9](../ip-addressing-plan.md#9-phụ-lục-int-03-cho-topology-tích-hợp--thiết-kế-23102026). INET-TEST là IOSv có Lo100 `192.0.2.10/32`; BGP underlay AS65010/65101/65102/65200 và traffic matrix INT-04–06 dùng mục 4–5 dưới đây. VPN Sprint 6 dùng SVTI/IKEv2/IPsec + OSPF, V01/V02 đã cấp; chưa chuyển link/địa chỉ S3-Lxx của sandbox. Default/return/NAT/deny và phạm vi failover chọn theo mục 5. Đây là **quyết định thiết kế**, không phải bằng chứng cổng đã nối hay protocol đã chạy. Các chữ “đề xuất/chưa chốt” trong các checkpoint lịch sử bên dưới mô tả trạng thái trước quyết định này; những chi tiết runtime như MTU thực tế, adjacency, BGP và SLA/track phải được kiểm khi đến INT-04/05/06, không chặn bắt đầu dựng topology.

**Trạng thái thiết kế 23/10/2026:** INT-03 đã chốt baseline thiết kế/bàn giao; bắt đầu INT-04 trên lab tích hợp đã restore. Pilot đúng IOSv 15.6(2)T đã chứng minh IKEv2/IPsec SVTI, OSPF Area10 trên tunnel, route Loopback và ping hai chiều; người học xác nhận đã lưu cấu hình cả hai đầu. ASAv 9.8(1) và hai Core đã có preflight parser cần thiết. Xem [evidence pilot](../test-results/sprint-03-vti-pilot-2026-10-22.md). Nguồn: [mốc tích hợp](../enterprise-integration-milestones.md), [kiến trúc](../network-architecture.md), [IP Plan](../ip-addressing-plan.md), [export Sprint 3](../test-results/sprint-03-export-inspection-2026-10-22.md). **Cập nhật triển khai 24/10:** [buổi 2 Sprint 4](../test-results/sprint-04-provider-bgp-2026-10-24.md) đã kiểm IP/BGP trên ISP1, ISP2, INET-TEST trong lab Enterprise; Edge/FW/VPN và INT-04 toàn phần chưa nghiệm thu.

## 1. Nguồn và đích

- Nguồn Campus: `Enterprise-Network-HQ.unl` của [Sprint 02](../../eve-exports/README.md#bàn-giao-sprint-02--18102026) cùng snapshot cấu hình Core/Access và bằng chứng hiện có. Ảnh topology HQ gốc ngày 22/10 xác nhận chưa có node FW; ASAv phải được thêm trong bản phát triển, không phải trong baseline. Giữ nguyên lab/export gốc; tạo bản phát triển mới từ nguồn này, rồi smoke test Campus trước khi thêm WAN. Tên và đường dẫn UNL **thực tế** của bản phát triển sẽ ghi khi tạo ở INT-04, chưa có artifact để xác nhận lúc này.
- Nguồn học routing: `Enterprise-Network-S03-Routing.unl`, năm IOSv/ba VPCS, [ZIP v2](../../eve-exports/sprint-03-routing-2026-10-22-v2.zip) và tám cấu hình export. Không nhập nguyên lab này vào Campus; IP/transit và vị trí router khác kiến trúc đích.
- Đích: một topology Enterprise phát triển từ HQ gồm Campus, hai Edge, hai ISP và hai Branch tại INT-04; FW/DMZ ở INT-05; VPN HQ–hai Branch ở INT-06. Các node trên đường test phải chạy cùng lúc khi nghiệm thu.

## 2. Mapping giữ lại và thay đổi

- `BR1-R1` và `BR2-R1`: giữ vai trò gateway LAN `10.20.10.1/24`, `10.30.10.1/24` và RID Lo0 `10.255.255.21/.22`; đổi uplink trực tiếp tới EDGE1 (`S3-L02/L03`) thành T05 tới ISP1 và T06 tới ISP2. Chỉ giữ Lo110 `10.20.11.1/24` và `10.30.11.1/24` như prefix lab nếu có nhu cầu đo summary thực trên Enterprise; không gọi đó là LAN có client.
- `EDGE1` và `EDGE2`: giữ tên và RID Lo0 `.11/.12`; bỏ S3-L01 `.36/30` Edge–Edge, thay phía trong bằng VLAN91/T02 `.10/.11` qua FW outside `.9`, phía ngoài bằng T03 EDGE1–ISP1 `.17/.18` và T04 EDGE2–ISP2 `.21/.22`. Cổng vật lý chưa được chốt; không copy `ip ospf 10 area ...`, `area range` hay static default của sandbox vào các uplink này.
- `ISP1`: giữ vai trò provider mô phỏng và có thể giữ RID Lo0 `.101`; thay S3-L04 EDGE2–ISP1 `.49/.50` bằng T03/T05 và T07. Thêm `ISP2` Lo0 `.102` với T04/T06/T07. Cặp T07 `.33/.34` tạo đường provider qua hai ISP. ISP không tham gia OSPF doanh nghiệp.
- `PC-HQ` sandbox `.54/30` chỉ là endpoint học Area0; client HQ thật là PC-IT/HR/ACC/GUEST trong Campus `10.10.0.0/16`. `PC-BR1/BR2` có thể giữ vai trò client thử `10.20.10.20`/`10.30.10.20`, sau khi nối qua gateway Branch thật.
- ISP1 Loopback100 `192.0.2.10/32` là đích mô phỏng của sandbox. Trên Enterprise, địa chỉ đó được cấp cho INET-TEST Loopback100 để endpoint còn tồn tại khi mất riêng một ISP; không copy Loopback100 của ISP1 sandbox sang ISP1 tích hợp.
- ABR EDGE1 Area0/10/20, ASBR EDGE2, summary `10.20.10.0/23` và `10.30.10.0/23`, O*E2 default và các bài mismatch/withdrawal là hành vi đã học trong sandbox. Trên Enterprise, Branch nằm **sau ISP/VPN**, nên ABR/ASBR, area và summary chỉ áp khi transport hỗ trợ adjacency thật và đã chốt phạm vi OSPF. Không rải OSPF doanh nghiệp trên provider để tái tạo kết quả sandbox.

### Hướng routing/VPN đã chọn để kiểm chứng

Chọn **static IPsec Virtual Tunnel Interface (SVTI) giữa router IOSv hai đầu, chạy OSPF trên tunnel**, không dùng policy-based crypto map với static route làm phương án chính. Trong bài site-to-site này, VTI tạo interface L3 định tuyến và hỗ trợ multicast/định tuyến động, nên không cần lớp GRE chỉ để mang OSPF. [Cisco IOS 15M&T VTI guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_vpnips/configuration/15-mt/sec-sec-for-vpns-w-ipsec-15-mt-book/sec-ipsec-virt-tunnl.html) mô tả SVTI và lưu ý feature còn phụ thuộc release/image; [Cisco IOSv feature list](https://developer.cisco.com/docs/modeling-labs/2-6/iosv/) có IPsec, OSPF và GRE. Hai nguồn đó là cơ sở thiết kế, **không thay bài thử trên IOSv 15.6(2)T đang chạy**. Nếu SVTI không hoạt động trong image này, thử GRE được bảo vệ bởi IPsec trên bản lab riêng rồi quyết định fallback dựa trên actual; không tự đổi topology chính.

- Tunnel HQ–BR1: EDGE1 (IP underlay T03 `.17`) ↔ BR1-R1 (T05 `.25`), OSPF Area10 trên tunnel, Branch LAN và Lo110 Area10/passive. IP tunnel **đề xuất** từ dải dự trữ: `10.255.0.56/30`, EDGE1 `.57`, BR1 `.58`.
- Tunnel HQ–BR2: EDGE2 (T04 `.21`) ↔ BR2-R1 (T06 `.29`), OSPF Area20 trên tunnel, Branch LAN và Lo110 Area20/passive. IP tunnel **đề xuất**: `10.255.0.60/30`, EDGE2 `.61`, BR2 `.62`.
- EDGE1 và EDGE2 cùng T02/VLAN91 Area0, OSPF neighbor trực tiếp trên L2 shared segment; EDGE1 là ABR Area0/10, EDGE2 là ABR Area0/20. FW outside `.9` có mặt trên segment nhưng không làm OSPF peer. Kiểm L2/hello/network-type/adjacency trên image thật; không giữ giả định `FULL/-` từ link P2P sandbox. ISP1/ISP2 chỉ chuyển underlay, không nhận các prefix nội bộ/VPN từ OSPF doanh nghiệp.
- EDGE1 summary Area10 `10.20.10.0/23`, EDGE2 summary Area20 `10.30.10.0/23` chỉ khi cả hai /24 thực được quảng bá trong area tương ứng. Lo110 hiện là prefix mô phỏng, nên summary này vẫn có tính học tập; không biến Lo110 thành LAN sản xuất giả.
- HQ Core nằm sau ASAv, do đó **không** có adjacency OSPF Core↔Edge theo sơ đồ này. Cần thiết kế static/controlled route giữa Core–FW–Edge, gồm HQ `10.10.0.0/16` đi vào tunnel và hai summary Branch đi về; chỉ quảng bá HQ prefix vào OSPF với route-map/filter khi đã xác minh return path. Chặn default/BGP provider khỏi OSPF ngoài chủ đích. BGP underlay Sprint4 và OSPF overlay Sprint6 phải phân biệt rõ trong RIB/test.
- Endpoint VPN đặt trên IOSv Edge/Branch, không đặt trên ASAv; ASAv vẫn giữ vai trò firewall, NAT/policy cho HQ. Cần kiểm NAT exemption/ACL và đường về qua FW khi triển khai INT-05/06. Một tunnel duy nhất tới mỗi Branch không bảo đảm Branch còn thông khi đúng Edge/ISP đó hỏng; test mất tunnel phải kỳ vọng mất kết nối nếu chưa thiết kế đường dự phòng riêng.

**Cổng kiểm chứng trước khi chốt INT-03:** trên lab thử riêng dùng đúng image `vios-adventerprisek9-m-15.6.2T`, kiểm parser và dựng **một** cặp SVTI/IKE trên hai IOSv với underlay đã ping được. Bằng chứng tối thiểu: tunnel IP hai đầu ping, IKE/IPsec SA thực sự UP và counter encrypted/decrypted tăng khi gửi traffic, OSPF neighbor FULL trên tunnel, prefix từ loopback/LAN giả đi qua tunnel; gỡ underlay làm neighbor/route mất rồi khôi phục. Lệnh được parser nhận chưa đủ chứng minh feature chạy. Không dùng router sandbox đã lưu làm nơi thử cấu hình crypto; giữ ZIP v2 làm baseline. Kiểm MTU/fragmentation trước khi áp dụng client traffic. Nếu pilot không đạt, ghi lỗi thực và mới xem xét GRE/IPsec hoặc phương án static.

**Pilot nhỏ đề xuất:** EVE lab `Enterprise-Network-INT03-VTI-Pilot`, hai IOSv cùng image của Sprint3 (`VTI-HQ`, `VTI-BR1`), một dây Gi0/0↔Gi0/0. Underlay riêng `172.31.255.0/30`: HQ `.1`, BR1 `.2`; Lo0 thử `172.31.0.1/32` và `172.31.1.1/32`. Khi sang pha crypto, Tunnel0 `172.31.255.4/30`: HQ `.5`, BR1 `.6`, OSPF Area10 trên Tunnel0 và Lo0 passive. Các IP này chỉ thuộc pilot, **không** phải IP Enterprise. Chỉ start hai IOSv để kiểm tài nguyên. Pilot này là phép thử image/tunnel, không thay yêu cầu một topology Enterprise cuối kế thừa HQ.

## 3. Quyết định phải chốt để đóng INT-03

### Cổng ứng viên đã đối chiếu từ artifact

Export HQ Sprint 02 và ảnh topology HQ ngày 22/10 cho thấy CORE1 `e1/2` và CORE2 `e1/0` chưa nối dây; có thể dành hai cổng này làm ứng viên cho T01/VLAN90, cần xác nhận lại khi tạo bản phát triển. Lab gốc chưa có FW node; không yêu cầu kiểm tra cổng FW trong lab này. Hai cổng này cùng FW inside cần nối vào **một** EVE Bridge `T01-VLAN90`; T02/VLAN91 dùng EVE Bridge khác nối FW outside, EDGE1 và EDGE2. EVE Bridge là miền L2 nhiều đầu, phù hợp hai subnet /29; [EVE Community Cookbook 6.3, mục 9.1](https://www.eve-ng.net/wp-content/uploads/2025/04/EVE-CE-BOOK-6.3-2024.pdf) mô tả nó như unmanaged switch. Tách hai bridge; không nối chúng trực tiếp.

Cổng ứng viên cho router IOSv bốn Gi: EDGE1 `Gi0/0` T02, `Gi0/1` T03; EDGE2 `Gi0/0` T02, `Gi0/1` T04; ISP1 `Gi0/0` T03, `Gi0/1` T05, `Gi0/2` T07; ISP2 `Gi0/0` T04, `Gi0/1` T06, `Gi0/2` T07; BR1/BR2 `Gi0/0` T05/T06 và `Gi0/1` LAN hiện có. Đây là **phân bổ đề xuất**, chưa tạo link hay kiểm IOSv image thứ hai trên topology chung. FW inside/outside/DMZ phải đối chiếu số cổng và tên interface ASAv thật trước khi gán; không ghi là đã chốt.

Khi đưa VLAN90 lên hai Core, cần thiết kế STP, đường HSRP transit và phản ứng khi mất từng dây Core–bridge; chỉ thấy hai cổng rảnh chưa đủ chứng minh failover. Các trunk/Po hiện không cho VLAN90, nên không tự thêm VLAN90 vào Po1 trước khi có thiết kế vòng L2 và bài thử failure.

1. **Cổng/L2:** inventory và port còn trống thực tế của Core, FW, Edge, ISP, Branch; chọn L2 node/cổng cho VLAN90 (T01) và VLAN91 (T02). IP T01 `.1/.2/.3/VIP .6`, T02 `.9/.10/.11` đã cấp; cổng vật lý và cấu hình SVI/VIP transit chưa được xác nhận.
2. **Đường HQ–FW–Edge:** kiểm Core image với routing/CEF, đặt default Core qua FW inside, return route FW về các VLAN HQ, default FW outside và đường Edge về FW. Chốt cách FW chọn Edge khi một ISP hỏng; không tự tạo VIP Edge hoặc bypass FW. Tách mốc cấu hình FW thực tế tới INT-05.
3. **Underlay/BGP Sprint 4:** chọn ASN, peering trên T03/T04, route provider và đích Internet test; quyết định nguồn default và phát hiện upstream hỏng. Static default sandbox chỉ kiểm sự tồn tại next-hop, không chứng minh health tracking/BGP failover.
4. **VPN/routing Sprint 6:** phương án chính đã chọn là IOSv SVTI + OSPF với endpoint/area/tunnel IP đề xuất ở trên. Cần kiểm lệnh và tunnel/IPsec/OSPF thật trên image 15.6(2)T, chốt IKE phiên bản/thuật toán được hai đầu hỗ trợ, MTU và underlay reachability; sau đó cập nhật IP Plan, route/filter qua FW và test cases. GRE/IPsec chỉ là fallback có điều kiện.
5. **Traffic matrix:** ghi source/target thật cho INT-04 Edge→Internet-test và Campus regression; INT-05 mỗi client VLAN HQ→test qua FW, DMZ permit/deny; INT-06 PC-IT/HR/ACC theo policy↔PC-BR1/BR2 hai chiều qua VPN, cùng các ca mất ISP/tunnel, route withdrawal/return, NAT exemption và hồi quy. Chọn dịch vụ/ICMP/deny chính xác sau khi có policy; không dùng ping giữa router thay client test.

### Bộ ca đại diện đã có endpoint để lập kế hoạch

- **INT-04:** EDGE1/EDGE2 theo từng uplink → Internet test target (địa chỉ đích sẽ chốt ở NET-401); shutdown từng ISP/Edge rồi đo BGP route, next-hop và traffic theo nhánh còn lại. Hồi quy Campus PC-IT `10.10.20.20` ↔ PC-ACC `10.10.40.20`; chưa kỳ vọng HQ client đi Internet khi FW chưa nối.
- **INT-05:** PC-IT `10.10.20.20` → target qua FW/Edge/ISP; PC-GUEST `10.10.60.20` → mạng nội bộ phải bị deny theo policy; từ phía ngoài → DMZ Web `10.10.100.10` permit dịch vụ được công bố, → LAN nội bộ deny. Chốt dịch vụ/cổng và IP target khi làm policy; cần cả đường trả về, NAT và counter.
- **INT-06:** PC-IT `10.10.20.20` ↔ PC-BR1 `10.20.10.20` và ↔ PC-BR2 `10.30.10.20`, kiểm riêng hai chiều qua VPN với counter encrypted/decrypted; đồng thời thử nguồn GUEST tới Branch là deny nếu policy cuối chọn vậy. Mất từng tunnel/transport: route và traffic đổi đúng kỳ vọng thiết kế, rồi hồi phục. Đây là ca **dự kiến**, chưa có phép thử trên topology tích hợp.

**Điều kiện tiếp tục:** chốt năm nhóm trên, cập nhật IP Plan, physical/architecture và test case nhất quán rồi review INT-03. Cho tới lúc đó chỉ phần học Sprint 3 và artifact export được xác nhận; không triển khai NET-401–404 trên topology tích hợp theo giả định.

## 4. Phương án WAN/BGP để review trước Sprint 4 — 22/10/2026

**Mức xác nhận:** thiết kế đề xuất, chưa tạo node/link, chưa chạy BGP. Phần này làm cụ thể nhóm quyết định số 3; không tự đóng toàn bộ INT-03. T03–T07 giữ nguyên IP đã cấp; T02 được chuẩn bị cho iBGP giữa hai Edge ở Sprint 4, FW nối vào T02 ở Sprint 5. Việc chuẩn bị segment T02 sớm không có nghĩa HQ đã đi qua FW.

### Mapping cổng dự kiến

- T02, EVE Bridge `T02-VLAN91`: EDGE1 Gi0/0 `10.255.0.10/29`, EDGE2 Gi0/0 `10.255.0.11/29`; FW outside `10.255.0.9/29` bổ sung ở Sprint 5 sau khi xác nhận interface ASAv.
- T03: EDGE1 Gi0/1 `10.255.0.17/30` ↔ ISP1 Gi0/0 `10.255.0.18/30`.
- T04: EDGE2 Gi0/1 `10.255.0.21/30` ↔ ISP2 Gi0/0 `10.255.0.22/30`.
- T05: BR1-R1 Gi0/0 `10.255.0.25/30` ↔ ISP1 Gi0/1 `10.255.0.26/30`.
- T06: BR2-R1 Gi0/0 `10.255.0.29/30` ↔ ISP2 Gi0/1 `10.255.0.30/30`.
- T07: ISP1 Gi0/2 `10.255.0.33/30` ↔ ISP2 Gi0/2 `10.255.0.34/30`.
- Branch LAN giữ BR1/BR2 Gi0/1 `10.20.10.1/24` / `10.30.10.1/24`, VPCS `.20` trong LAN tương ứng. Không dùng cổng này làm link provider.

### Đích test độc lập với hai ISP

Đề xuất thêm **một IOSv `INET-TEST`**, cùng image 15.6(2)T đã dùng. Router này mô phỏng một upstream chung có endpoint ICMP Loopback100 `192.0.2.10/32`. Nếu đặt endpoint trên ISP1 thì tắt ISP1 cũng tắt đích, không kiểm được failover đến cùng một endpoint. Đây là quyết định thiết kế của lab, không phải mô hình đầy đủ của Internet hay server HTTP.

- W08, subnet đề xuất `203.0.113.0/30`: ISP1 Gi0/3 `.1` ↔ INET-TEST Gi0/0 `.2`.
- W09, subnet đề xuất `203.0.113.4/30`: ISP2 Gi0/3 `.5` ↔ INET-TEST Gi0/1 `.6`.
- EDGE1 Lo100 `198.51.100.11/32` và EDGE2 Lo100 `198.51.100.12/32` là source test BGP độc lập với uplink. Không thay Lo0/router-ID và không dùng các IP này làm public NAT pool ở Sprint 5.
- W08/W09 và Lo100 Edge đã được cấp trong [IP Plan phụ lục INT-03](../ip-addressing-plan.md#9-phụ-lục-int-03-cho-topology-tích-hợp--thiết-kế-23102026). W08/W09 đã cấu hình và kiểm ở buổi 2 Sprint 4; Lo100 Edge chờ buổi triển khai Edge. ISP1/2 dùng đủ bốn cổng Gi trong lab tích hợp.
- Tăng inventory WAN từ sáu lên bảy IOSv; T02 dùng EVE Bridge, không thêm switch VM. Giữ cấu hình tài nguyên IOSv đang dùng, đo `free -h`/`uptime` khi bật nhóm WAN. Có thể tắt nhóm Campus trong bài WAN và bật lại để regression; phải cùng UNL và ghi đúng nhóm node đang chạy. Chưa có bằng chứng máy đủ tải cho toàn Enterprise đồng thời.

### AS, peering và policy dự kiến

- Company AS `65010`: EDGE1, EDGE2; iBGP trực tiếp qua T02 `.10` ↔ `.11`, `next-hop-self` hai chiều.
- ISP1 AS `65101`, ISP2 AS `65102`; eBGP trên T03, T04 và T07.
- INET-TEST AS `65200`; eBGP với ISP1 trên W08 và ISP2 trên W09. Các ASN nằm trong dải private, chỉ dùng trong lab.
- BR1/BR2 không chạy BGP trong phương án này: static default lần lượt qua `10.255.0.26` / `10.255.0.30` làm underlay. Prefix HQ/Branch sau này được học cụ thể qua OSPF/VTI, không quảng bá LAN vào ISP.
- INET-TEST originate `192.0.2.10/32` và default `0.0.0.0/0`. Default có backing static Null0 **chỉ tại INET-TEST** để mô phỏng đích cuối Internet của lab; không mô tả nó là kết nối Internet thật. Prefix có thật như endpoint ICMP và source test dùng route cụ thể, thắng Null0 default.
- ISP1/2 originate đúng các subnet WAN khách hàng trực tiếp tương ứng T03–T06; trao đổi chúng qua T07 và quảng bá cho INET-TEST để có return path. Không `redistribute connected` toàn bộ.
- Edge xuất ra ISP hai prefix test `198.51.100.11/32` / `.12/32`, gồm route của chính mình và route peer iBGP khi có; bổ sung prefix T02 `10.255.0.8/29` cho return PAT tại FW theo mục 5. Không xuất default học từ ISP khác, không làm transit cho provider, không xuất HQ `10.10.0.0/16`, Branch LAN hoặc tunnel subnet.
- Edge nhận default, target ICMP `/32` và các subnet underlay T03–T06 bằng prefix-list/route-map giới hạn chính xác. ISP trao đổi company test `/32`, T02 `/29`, target/default và các subnet WAN theo allow-list; INET-TEST nhận company test `/32`, T02 `/29` và subnet WAN cần return path.
- Trên EDGE1, route nhận từ ISP1 có Local Preference `200`; trên EDGE2, route nhận từ ISP2 có Local Preference `100`. iBGP chia sẻ các route này: khi cả hai path hợp lệ, hai Edge ưu tiên exit ISP1; khi route từ ISP1 bị rút, dùng ISP2. Không đặt weight làm mất tác dụng so sánh Local Preference trong bài thử.

**Đối chiếu runtime Branch 26/10/2026:** cấu hình CP2 LP200 trên EDGE1 áp cho **mọi** prefix nhận từ ISP1. Nó khiến EDGE2 chọn T06 `10.255.0.28/30` qua iBGP EDGE1 `.10` dù BR2 nối trực tiếp ISP2; ping vẫn 10/10 nhưng chiều về bắt đầu qua EDGE1. [Biên bản](../test-results/sprint-04-branch-underlay-2026-10-26.md) và [ZIP CP2](../test-results/sprint-04-dual-isp-export-2026-10-26.md) có bằng chứng. **Tinh chỉnh đã đạt chức năng/lưu/export:** route-map EDGE1 chỉ gán LP200 cho P-UPSTREAM (default và `192.0.2.10/32`), P-WAN LP100. EDGE2 cài T06 trực tiếp qua ISP2 `.22`, ping/traceroute tới BR2 đạt; default/target trên EDGE2 vẫn chọn ISP1 LP200, hai Lo100→INET 10/10. EDGE1 `wr [OK]` sau sửa; [ZIP mới](../test-results/sprint-04-underlay-export-2026-10-26.md) có payload scoped LP và hai Branch/PC. ZIP CP2 giữ lịch sử trước thay đổi; restore chưa thử.
- Sprint 4 không originate BGP default từ Edge độc lập với provider. Khi INET-TEST mất cả hai peering, default/target phải bị rút qua BGP; khi chỉ một đường provider mất, có thể còn đường qua T07. BGP session/route withdrawal không tự phát hiện mọi trường hợp data plane bị blackhole khi session vẫn Established; bài silent-drop phải thiết kế health tracking riêng trước khi nghiệm thu khả năng đó.

### Ca kiểm thử cụ thể dự kiến

- `INT-04-TC-01`: EDGE1 source `198.51.100.11` và EDGE2 source `.12` → `192.0.2.10`, ICMP hai chiều bằng reply, mỗi nguồn 10 gói; đọc best path/next-hop, không chỉ số lượng BGP session.
- `INT-04-TC-02`: baseline cả hai Edge chọn đường ISP1 theo Local Preference; trace và counter phải phù hợp đường thực, không yêu cầu đường trả về đối xứng.
- `INT-04-TC-03`: shutdown ISP1, giữ INET-TEST và ISP2 chạy. Sau hội tụ, hai source test vẫn ping cùng endpoint; EDGE1 dùng peer EDGE2 qua T02, EDGE2 dùng ISP2. Ghi gói mất/thời gian nếu đo được, không suy SLA từ một lần ping cuối.
- `INT-04-TC-04`: khôi phục ISP1, xác nhận session/route/path ưu tiên trở lại. Sau đó mất riêng T04 phải không làm mất endpoint đang đi ISP1. Nếu mất ISP2 trong khi ISP1 bình thường, endpoint vẫn còn qua W08.
- `INT-04-TC-05`: mất riêng W08 khi ISP1 còn chạy: đường ISP1 → ISP2 qua T07 có thể giữ target/default; không mặc định mọi sự cố W08 đều buộc HQ đổi exit ISP2.
- `INT-04-TC-06`: tắt INET-TEST, kiểm target/default rút khỏi hai Edge; bật lại và kiểm recovery. Đây là thử route withdrawal của Internet mô phỏng, không phải SLA reachability Internet public.
- `INT-04-TC-07`: Campus regression PC-IT `10.10.20.20` ↔ PC-ACC `10.10.40.20` trên bản phát triển sau stop/start; kiểm lại Po1/Po11/HSRP và giữ lỗi IOL/CEF đang mở đúng phạm vi. HQ client chưa có đường ra WAN trước FW.
- Transport VPN: BR1 `.25` ↔ EDGE1 `.17`, BR2 `.29` ↔ EDGE2 `.21`; ping source WAN IP và route trả về trước cấu hình VTI. Mất ISP gắn với Branch sẽ làm Branch đó mất transport vì chưa có uplink dự phòng riêng.

### Trình tự thao tác kế tiếp

1. Backup `sprint-02-hq-before-wan-2026-10-22.zip` đã được kiểm: 8 node, đủ 8 payload cấu hình giải mã được, CRC/XML hợp lệ. Startup-configs đang OFF; import/restore chưa thử. Chưa cần export lại.
2. Hoàn tất nhóm quyết định còn lại của INT-03: T01/Core/FW routing và policy/failover qua FW; chốt phần đề xuất này vào IP Plan và sơ đồ, bao gồm inventory INET-TEST và giới hạn tải. Tên bản phát triển đề xuất `Enterprise-Network-Integrated`; chỉ ghi path là actual khi đã tạo.
3. Sau review INT-03, import backup thành một lab phát triển riêng, kiểm restore và smoke test Campus; giữ nguyên lab HQ gốc. Sau đó triển khai NET-401–404 theo mapping trên. FW/DMZ và OSPF/VTI tích hợp triển khai ở INT-05/06.

Tham chiếu cơ chế BGP: [Cisco BGP case studies](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/26634-bgp-toc.html) (iBGP next-hop-self và Local Preference); [IANA ASN registry](https://www.iana.org/assignments/as-numbers) (private ASN 64512–65534). Mapping, địa chỉ đã cấp trong phụ lục INT-03, cách dựng INET-TEST và ca kiểm thử trên là thiết kế đã chọn của dự án, chưa phải actual runtime.

## 5. Phương án Core–FW–Edge để kiểm chứng — 23/10/2026

**Checkpoint thiết bị 23/10:** [preflight thực tế](../test-results/int-03-asav-preflight-2026-10-23.md) xác nhận ASA 9.8(1), ASAv10/RAM 2048 MB, routed mode và ba cổng data Gi0/0–0/2. Parser `sla monitor ?` nhận ID 1–2147483647/schedule; `track ?` nhận ID 1–500. Cả CORE1/CORE2 nhận `standby 20 track ?` với object ID 1–1000. Chưa tạo object/route hay thử failover. Cảnh báo Unlicensed được giữ trong giới hạn nghiệm thu. Không cần tạo lại FW chỉ vì đây là node thử từ Sprint 0. Inside/outside/DMZ chưa cấu hình; mapping vật lý được chọn trong mục 5, cần xác nhận tại lúc nối dây.

Đã chọn phương án thiết kế dưới đây làm cơ sở triển khai. Version/interface và parser SLA/track ASAv, HSRP track Core đã được kiểm; còn đồng bộ IP Plan/physical/architecture và review resource trước khi đóng INT-03. Chưa cấu hình hoặc nghiệm thu tracking/failover thực tế. Giữ nguyên IP T01/T02, không bổ sung VIP Edge. Các nội dung mục này bổ sung và làm rõ những quyết định còn mở tại mục 3/4.

### T01 và dự phòng Core

- CORE1 e1/2 và CORE2 e1/0 là cổng access VLAN90 tới cùng EVE Bridge `T01-VLAN90`; FW Gi0/0 (`inside`) nối bridge này. SVI90: CORE1 `10.255.0.2/29`, CORE2 `.3/29`, HSRP VIP `.6`; FW inside `.1/29`.
- VLAN90 chỉ tồn tại trên hai cổng access nói trên và SVI90, không thêm vào Po1 hoặc trunk Access. Hai Core trao đổi HSRP90 qua bridge; không tạo thêm vòng L2 cho VLAN90 qua Po1. Khi kiểm tra phải đọc STP90 và trạng thái HSRP, không chỉ kiểm cổng up.
- Hai Core dùng default qua FW `.1`. FW trả về sáu subnet HQ `10.10.10.0/24`, `10.10.20.0/24`, `10.10.30.0/24`, `10.10.40.0/24`, `10.10.50.0/24`, `10.10.60.0/24` qua VIP `.6`. Chọn route cụ thể để tách rõ LAN do Core quản lý và DMZ `10.10.100.0/24` trực tiếp trên FW; một summary /16 sẽ bao phủ cả hai phạm vi.
- Parser HSRP object tracking của hai Core đã được kiểm. Khi triển khai INT-05, dự kiến tạo object 90 theo line protocol của cổng T01 trên mỗi Core, giảm priority 20 cho các nhóm HSRP VLAN10–60 khi link transit của Core đó down. Với baseline 110/100, Core còn đường FW sẽ nhận Active. Phải kiểm failover bằng client; chỉ có HSRP90 chưa đủ bảo vệ default gateway client khi mất dây Core–FW.
- FW1 và EVE Bridge vẫn là điểm lỗi đơn. Mất FW không được ghi kỳ vọng còn HQ Internet/VPN, dù Core/ISP có dự phòng.

### FW outside và failover

- FW Gi0/1 (`outside`) `10.255.0.9/29` nối EVE Bridge T02; Gi0/2 (`dmz`) dự kiến `10.10.100.1/24`. Default chính tới EDGE1 `.10`, backup tới EDGE2 `.11` với distance cao hơn. Dùng SLA ICMP track trực tiếp `.10` trên outside để bỏ next-hop chính khi EDGE1 không còn đáp ứng. Đây là track gateway Edge, không phải kiểm chất lượng Internet.
- Mất ISP1 khi EDGE1 vẫn hoạt động: iBGP cho phép EDGE1 đi qua EDGE2 tới ISP2; FW có thể tiếp tục dùng `.10`. Mất EDGE1: track của FW phải chọn `.11`. Silent blackhole khi Edge vẫn trả lời ICMP nằm ngoài khả năng phát hiện của bài này.
- Route FW tới BR1 `10.20.10.0/23`: chính EDGE1 `.10`, dự phòng EDGE2 `.11`; tới BR2 `10.30.10.0/23`: chính EDGE2 `.11`, dự phòng EDGE1 `.10`. Mỗi next-hop chính dùng track reachability của Edge tương ứng. Khi tunnel của một Branch mất nhưng Edge vẫn sống, traffic tới Branch đó phải bị drop; route backup qua Edge kia không được coi là VPN dự phòng cho Branch đó.
- Cơ sở feature: [Cisco ASA 9.12 static route tracking](https://www.cisco.com/c/en/us/td/docs/security/asa/asa912/configuration/general/asa-912-general-config/route-static.html). Phải đối chiếu version, routed mode, interface và parser trên ASAv thật trước chốt câu lệnh. Không tự gán cổng management làm inside/outside.

### Route quảng bá và NAT

- Hai Edge có route tĩnh tới sáu subnet HQ /24 và DMZ `10.10.100.0/24` qua FW `.9`; chỉ các route này được phép redistribute static vào OSPF overlay bằng prefix-list/route-map, metric/type/tag tường minh. Không redistribute toàn bộ static/BGP/default. ISP không được học các prefix LAN/DMZ/overlay này.
- Cần bổ sung **T02 `10.255.0.8/29`** vào allow-list BGP Edge → ISP và ISP → INET-TEST. Cả hai Edge originate prefix T02 khi có route connected. Đây là mạng transit private được provider mô phỏng mang trong lab để trả về địa chỉ PAT `.9`; không phải quảng bá RFC1918 trên Internet thật. Hai Lo100 test /32 vẫn được xuất như mục 4.
- HQ ra endpoint Internet mô phỏng: PAT trên FW sang outside `.9`; Edge không NAT lần hai. ISP/INET-TEST trả traffic về `.9` bằng BGP prefix T02. Việc chỉ quảng bá Lo100 test không đủ cho bài PAT Sprint 5.
- HQ ↔ Branch: identity NAT exemption hai chiều đặt trước PAT cho các cặp prefix được phép; ACL của FW vẫn kiểm soát lưu lượng. Gói HQ tới Branch giữ source `10.10.x.x`, rồi Edge mã hóa vào VTI. Branch học sáu prefix HQ qua OSPF. Không originate default OSPF; Branch giữ default ISP underlay và phải kiểm route tới peer VPN không trỏ vào chính tunnel.
- Chống leak khi OSPF/VTI mất: trên **cả hai Edge**, tạo static discard route tới từng summary Branch `/23` qua Null0 với distance 250; route OSPF AD110 thắng khi tunnel còn hoạt động. Trên **mỗi Branch**, tạo discard route cho từng HQ LAN `/24` và DMZ `/24` được phép qua Null0 distance 250, để mất route OSPF không rơi về default ISP. Chặn thêm prefix LAN/DMZ nội bộ đi rõ ra cổng WAN bằng ACL ở phía doanh nghiệp; ESP có outer WAN IP thuộc T03–T06 vẫn được phép. Các static Null0 chỉ làm chốt chặn forwarding, không được redistribute vào OSPF/BGP. Kiểm thử mất tunnel bằng RIB, packet counter/capture ngoài WAN và ping; không dùng riêng ICMP thất bại để kết luận không leak.
- DMZ publish TCP/80: static PAT FW outside `.9:80` → `10.10.100.10:80`, có ACL outside cho dịch vụ đó. Dùng INET-TEST nguồn Lo100 `192.0.2.10/32` làm client TCP/80 qua provider, kiểm HTTP response và translation trên FW; IOSv cần thử khả năng tạo HTTP request thực khi tới INT-05. Không cấp IP public mới.

### Policy và phép thử bắt buộc

- Guest → các VLAN HQ/DMZ/Branch: deny trên ingress SVI60 của cả hai Core; permit ICMP tới target mô phỏng để test egress. Inter-VLAN đi trực tiếp qua Core nên ACL chỉ đặt ở FW không đủ cô lập Guest. Kiểm riêng trường hợp HSRP chuyển Core.
- IT/HR/Accounting → `192.0.2.10`: permit ICMP qua FW/PAT; kiểm reply, session và translation. IT ↔ hai PC Branch: permit ICMP hai chiều sau INT-06, identity NAT và IPsec counters tăng. Guest → Branch: deny trước mã hóa.
- INET-TEST Lo100 `192.0.2.10/32` → `.9:80`: HTTP permit tới DMZ; ngoài ra VPCS `PC-OUT` tạm trên T02 (`10.255.0.12/29`, gateway `.9`) → PC-IT `10.10.20.20` ICMP phải deny tại FW. DMZ Web `10.10.100.10` → PC-IT `10.10.20.20`: không cho khởi tạo kết nối. VPCS chỉ dùng cho ICMP deny; TCP/80 phải có HTTP client/response thực.
- Mất ISP1: endpoint Internet còn qua ISP2; mất EDGE1: FW chuyển `.11`, Internet và BR2 phải phục hồi, BR1 mất vì tunnel HQ của BR1 chỉ kết thúc tại EDGE1. Mất ISP2 đối xứng làm BR2 mất underlay; không gọi đây là VPN HA hai ISP cho từng Branch.
- Mất từng link Core–T01: kiểm HSRP client chuyển đúng, return route qua `.6`, traffic hồi phục. Mất tunnel: OSPF/route Branch rút; route discard AD250 phải xuất hiện và traffic nội bộ không được rơi ra ISP dạng rõ. Theo dõi counter/capture ở WAN để chứng minh deny/drop, không chỉ dựa vào ping thất bại.

### Tài nguyên và thao tác kế tiếp

EVE đã từng báo 7,7 GiB RAM tổng. Bài BGP INT-04 cần bật đồng thời EDGE1/2, ISP1/2, INET-TEST để thử failover cùng một đích; có thể tắt Branch và Campus trong bài đó nhưng chúng vẫn phải nằm cùng UNL. Khi thử Campus, bật bốn IOL và VPCS; khi thử Branch transport, bật đúng Edge/ISP/Branch theo đường đi. Ghi `free -h`/`uptime` cùng nhóm node đang bật; nếu thiếu RAM cho một bài end-to-end, tăng RAM EVE trước khi nghiệm thu bài đó. Việc phân nhóm không thay thế yêu cầu các node trên **cùng một đường kiểm thử** phải cùng chạy. ASAv 2 GiB chỉ cần bật từ INT-05; giới hạn license unlicensed khiến kết quả không dùng để đánh giá throughput.

Các kiểm tra parser ASAv và hai Core đã hoàn tất. IP Plan/physical/architecture và traffic matrix đã đồng bộ trong baseline INT-03 đầu tài liệu. [Gói seed Sprint4](../../eve-exports/sprint-04-enterprise-seed-2026-10-23.zip) được tạo từ backup HQ chứa đủ tám config; CRC/XML/payload đã kiểm. User đã import vào `LAB-NETWORK/test_import`, bật Startup-configs ON và sau Stop/Wipe/Start xác nhận [khôi phục cả lab thành công](../test-results/sprint-04-seed-restore-2026-10-23.md). CORE1 có output hostname/sáu SVI/Po1/Po11; bảy node còn lại và smoke Campus hiện chỉ có xác nhận tổng quát của người thực hành. **INT-03 đã đạt thiết kế/bàn giao; tiếp theo INT-04 dựng WAN trong lab này.** FW/HSRP tracking thuộc INT-05; giữ nguyên HQ gốc.
