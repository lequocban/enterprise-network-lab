# Sprint 0 — Kết quả kiểm thử

**Ngày tổng hợp:** 09/10/2026  
**Nguồn:** Output console đã ghi lại trong quá trình lab. Đây không phải kết quả chạy tự động từ GitHub.

**Cập nhật bằng chứng:** 10/10/2026, từ output và ảnh do người thực hành cung cấp; xem mục 7. Agent không trực tiếp chạy lệnh trên thiết bị.

**Closeout 10/10/2026:** CLOSED WITH FOLLOW-UPS theo [biên bản](../sprint-reports/sprint-00-closeout.md). Kết quả theo từng thời điểm dưới đây được giữ nguyên; mục 8 ghi quyết định cuối và nơi tiếp nhận phần chưa thử. Không dùng việc đóng sprint để nâng một test thiếu evidence thành PASS.

## 1. Kiểm tra EVE-NG

```text
free -h                       Total 7.7Gi | Available 6.3Gi | Swap 4.0Gi (unused)
df -h /                       116G total | 12G used | 100G available
nproc                         4
egrep -c '(vmx|svm)' /proc/cpuinfo      4
```

Đánh giá: tài nguyên và cờ nested virtualization đã nhận. Chưa dùng những con số này để kết luận hệ thống chịu được toàn bộ node Enterprise cùng lúc.

## 2. Router-to-router ping

R1 Gi0/0: `10.255.0.1/30`, R2: `10.255.0.2/30`. Ping R1 → R2: **4/5 (80%)**, RTT min/avg/max **1/1/2 ms**. Gói ping đầu mất có thể do ARP, nhưng chưa có capture để xác nhận nguyên nhân. Lưu cấu hình R1 thành công.

Đây là lab kết nối router tạm, tách biệt transit T01 `10.255.0.0/29` của thiết kế v1.0. Không chạy chung/nối hai segment có địa chỉ trùng. Kết quả 4/5 chứng minh có kết nối tại thời điểm test, chưa đủ kết luận ổn định; cần ping lặp lại hai chiều trong S0-03.

## 3. SVI và HSRP

```text
CORE1: Vlan10 10.10.10.2 up/up
CORE2: Vlan10 10.10.10.3 up/up

CORE1: Vl10 Group10 Priority110 P Active  local       10.10.10.3  VIP 10.10.10.1
CORE2: Vl10 Group10 Priority100 P Standby 10.10.10.2 local       VIP 10.10.10.1
```

VLAN10 tạo được trên Core, CORE1 có connected route `10.10.10.0/24`. Hai Core nhận diện peer và chọn đúng vai trò HSRP. Ghi nhận lịch sử lúc tổng hợp chưa đủ log switchover; ngày 10/10 đã bổ sung phép thử có kiểm soát, đạt chuyển vai trò và khôi phục ở TC-S00-HSRP-SWITCHOVER-01. Chưa đo convergence hoặc gián đoạn lưu lượng client.

Trong bước thử SVI có một **VPCS tạm nối trực tiếp CORE1 e1/0**. Máy này chỉ để đưa VLAN10 lên phục vụ test, không thuộc topology vận hành chính thức và không cần IP dự trữ cố định.

## 4. Chưa kiểm chứng

Inter-VLAN Routing nhiều VLAN; RSTP hội tụ khi mất uplink; EtherChannel; failover client end-to-end khi CORE1 mất nguồn; ASA/Alpine boot screenshot; WAN, OSPF/BGP, VPN, NAT, monitoring, automation.

## 5. Đầu việc bổ sung minh chứng

- Ảnh đầy đủ trạng thái ASAv và Alpine khi boot.
- Đã lưu lệnh gây lỗi/khôi phục và trạng thái HSRP trước/trong/sau ngày 10/10 tại TC-S00-HSRP-SWITCHOVER-01; full failover client thuộc Sprint 2.
- Đã có ảnh HQ, config và export sạch cuối ngày 10/10; kiểm tra tại TC-S00-POST-CFG-01 và TC-S00-EXPORT-FINAL-01.
- Từ Sprint 1 lưu test case cùng ngày với cấu hình.

Các đầu việc thu thập baseline tiếp tục theo tiêu chí NET-001/002 trong roadmap và báo cáo. Dùng [mẫu test case](test-case-template.md), lưu output `.txt` vì `*.log` bị Git ignore. Kết quả mục 1–3 là lịch sử tổng hợp 09/10/2026; bằng chứng mới ngày 10/10/2026 được thêm ở mục 7, không thay thế test khác loại.

## 6. Review tĩnh IP Plan

**Ngày review:** 09/10/2026

**Nguồn:** bảng phân bổ trong [IP Plan Final v1.0](../ip-addressing-plan.md).

**Loại kiểm tra:** đối chiếu tài liệu, không truy cập hoặc chạy lệnh trên thiết bị.

- Kiểm tra 24 subnet/loopback đã cấp: 6 VLAN HQ, DMZ, 2 Branch LAN, 7 transit và 8 loopback /32.
- Không phát hiện các subnet/loopback này chồng lấn nhau. Các dải /16 bao chứa HQ/Branch là phạm vi quy hoạch, không tính như subnet interface riêng.
- Địa chỉ được cấp trong T01–T07 nằm trong subnet và không sử dụng network/broadcast.
- Native VLAN99 không có subnet; gateway DMZ nằm ở FW; .1 VIP/.2 CORE1/.3 CORE2 nhất quán với bảng VLAN HQ.

**Giới hạn:** không chứng minh interface, routing, firewall hoặc failover đã hoạt động. Gateway outside/failover, routing và VPN placement còn mở; theo dõi WAN-01.

## 7. Bằng chứng được cung cấp ngày 10/10/2026

### TC-S00-ENV-01 — Tài nguyên và ảo hóa EVE-NG

**Nguồn:** [Output môi trường](sprint-00-eve-environment-2026-10-10.txt), do người thực hành cung cấp.

**Thời gian trong output:** `2026-10-10T07:00:39+00:00`, tương ứng **14:00:39 ngày 10/10/2026, Asia/Saigon**. Đây là cách quy đổi múi giờ, chưa thay đổi timezone của VM.

- Hostname: `eve-ng`; CPU nhận 4 vCPU.
- RAM: tổng 7.7 GiB, dùng 1.7 GiB, available 5.7 GiB. Swap 4.0 GiB, chưa dùng.
- Filesystem `/`: 116 GB, dùng 13 GB, available 98 GB, 12% sử dụng.
- CPU flags virtualization: kết quả đếm 4; `/dev/kvm` tồn tại, owner/group `root:kvm`, quyền `crw-rw----`.

**Đánh giá:** đạt kiểm tra ban đầu về tài nguyên hiển thị và môi trường ảo hóa. Thông số nhận trong Linux phù hợp quy mô VM đã mô tả; chưa có ảnh cấu hình VMware và danh sách node đang chạy lúc đo để đối chiếu trực tiếp allocation/tải. Không suy ra khả năng chạy toàn bộ Enterprise từ số RAM available.

### TC-S00-CORE-01 — Kết nối CORE1 ↔ CORE2

**Nguồn:** [Output ping hai Core](sprint-00-core-connectivity-2026-10-10.txt). Console ping không có timestamp riêng; ngày là ngày cung cấp bằng chứng.

- CORE1 → `10.10.10.3`: **5/5**, RTT min/avg/max **1/1/1 ms**.
- CORE2 → `10.10.10.2`: **5/5**, RTT min/avg/max **1/1/1 ms**.

**Đánh giá:** PASS cho hai lượt ping được cung cấp. Output `show ip interface brief` bổ sung ở TC-S00-CORE-02 xác nhận địa chỉ đích khớp SVI VLAN10 của hai Core. Config bổ sung ở TC-S00-CORE1-CFG-01 và TC-S00-CORE2-CFG-01 xác nhận cả hai dùng mask `/24`; source thực tế của lượt ping chưa được ghi rõ. Đây chưa phải hai lượt 10 gói mỗi chiều, chưa phải test R1/R2 IOSv và không kiểm chứng VIP, HSRP switchover hoặc Inter-VLAN.

### TC-S00-TOPO-01 — Đối chiếu cabling HQ

**Nguồn:** [Ảnh topology HQ](../../screenshots/sprint-00/hq-topology-2026-10-10.png), bản gốc người thực hành cung cấp được sao chép vào repo, không chỉnh sửa nội dung.

Đã đối chiếu 5 link hạ tầng và 4 link VPCS với [Physical topology](../../diagrams/physical-topology.md): CORE1 e0/0–CORE2 e0/0; CORE1/CORE2 e0/1–ACCESS1 e0/0/e0/1; CORE1/CORE2 e0/2–ACCESS2 e0/0/e0/1; các VPCS ở e0/2/e0/3 của Access đúng bảng.

**Đánh giá:** cabling hiển thị khớp tài liệu; ảnh không có PC-MGMT tạm. Ảnh không chứng minh VLAN/trunk/STP, trạng thái forwarding, console Access hoặc cổng đang up. Đã bổ sung config bốn switch, runtime VLAN10 hai Core và export sạch tại TC-S00-EXPORT-FINAL-01; toàn Campus vẫn chưa nghiệm thu.

### TC-S00-CORE-02 — Version, SVI và trạng thái HSRP hai Core

**Nguồn:** [Console CORE1](sprint-00-core1-version-hsrp-2026-10-10.txt), [Console CORE2](sprint-00-core2-version-hsrp-2026-10-10.txt), người thực hành cung cấp ngày 10/10/2026. Không có timestamp riêng cho từng lệnh; agent không trực tiếp truy cập node.

**Mục tiêu:** xác định image/version đang chạy, kiểm tra SVI VLAN10 và HSRP baseline theo thiết kế.

- Cả hai báo software `I86BI_LINUXL2-ADVENTERPRISEK9-M`, version `15.2(CML_NIGHTLY_20151103)FLO_DSGS7`; CLI ghi early deployment development build. Chuỗi system image file giống nhau: `unix:/opt/unetlab/addons/iol/bin/L2-ADVENTERPRISEK9-M-15.2-IRON-20151`.
- CORE1: Vlan10 `10.10.10.2`, Method NVRAM, up/up; group 10 priority 110, preempt, Active local; peer Standby `10.10.10.3`.
- CORE2: Vlan10 `10.10.10.3`, Method NVRAM, up/up; group 10 priority 100, preempt, Standby local; peer Active `10.10.10.2`.
- Virtual IP trên cả hai là `10.10.10.1`; hai phía nhận diện vai trò/peer nhất quán.
- Hai Core boot và phản hồi console; uptime tương ứng 1 và 3 phút. Mỗi node báo memory `409842K`, 12 Ethernet interface, 1 Virtual Ethernet interface, NVRAM 1024K. Chưa có thông số cấp phát node từ EVE-NG.

**Đánh giá:** PASS cho version/console, địa chỉ/trạng thái SVI và HSRP baseline trong phạm vi output đã nhận. Mask không xuất hiện trong `show ip interface brief`; config bổ sung tại TC-S00-CORE1-CFG-01 và TC-S00-CORE2-CFG-01 xác nhận cả hai Core `/24`.

**Giới hạn:** đã có running/startup-config hai Core; CORE2 còn khác biệt VTY giữa hai bản. VLAN/trunk/STP/routes, recheck vai trò và test chuyển role có kiểm soát đã bổ sung ở các test case phía dưới. Log HSRP riêng lúc 08:09 vẫn chưa rõ nguyên nhân. Còn thiếu ping VIP, client failover và Inter-VLAN; chưa đo convergence. Method NVRAM không thay thế kiểm tra toàn bộ startup-config. `show ip interface brief` trên IOL không đủ để suy ra tất cả các port hiển thị up/up đều có dây nối hoặc forwarding đúng.

### TC-S00-ACCESS-01 — Version, console và VLAN baseline hai Access

**Nguồn:** [Console ACCESS1](sprint-00-access1-version-vlan-2026-10-10.txt), [Console ACCESS2](sprint-00-access2-version-vlan-2026-10-10.txt), người thực hành cung cấp ngày 10/10/2026; không có timestamp riêng. Agent không trực tiếp chạy node.

**Mục tiêu:** xác nhận Access boot/console được, ghi version và trạng thái trước triển khai Campus.

- Cả hai chạy `I86BI_LINUXL2-ADVENTERPRISEK9-M`, `15.2(CML_NIGHTLY_20151103)FLO_DSGS7`, cùng version với Core. Chuỗi image file hiển thị là `unix:/opt/unetlab/addons/iol/bin/L2-ADVENTERPRISEK9-M-15.2-IRON-20151`.
- Mỗi node báo memory `147698K`, 8 Ethernet interface và NVRAM 1024K. Board ID ACCESS1 `67108912`, ACCESS2 `67108928`; uptime lần lượt 0 và 2 phút. RAM/vCPU cấp phát trong EVE-NG chưa được cung cấp.
- `show ip interface brief` chưa hiển thị SVI/IP quản trị; tám Ethernet interface có IP `unassigned`, trạng thái up/up trong CLI.
- `show vlan brief` chỉ có VLAN1 và 1002–1005; tám port được liệt kê dưới VLAN1. VLAN10/20/30/40/50/60/99 chưa xuất hiện.

**Đánh giá:** PASS cho boot/console/version của hai Access trong Sprint 0. VLAN và quản trị Access chưa triển khai theo thiết kế; tiếp nhận ở NET-101/102 và kế hoạch Sprint 1, không đánh dấu phần Campus hoàn tất.

**Giới hạn:** running/startup-config đã bổ sung tại TC-S00-ACCESS-CFG-01; chưa có `show interfaces trunk` hoặc STP output. Không kết luận trunk/STP đã hoạt động hoặc toàn bộ port có dây nối chỉ từ up/up. IP `unassigned` trên Ethernet của switch Layer 2 không phải bằng chứng lỗi; việc cần bổ sung cho quản trị là SVI theo IP Plan khi tới bước cấu hình.

### TC-S00-IMAGE-01 — Ghi nhận chạy thử ASAv và Alpine

**Nguồn:** người thực hành xác nhận trong trao đổi ngày 10/10/2026 đã tạo thử node server Alpine và ASAv, cả hai chạy được. Ngày chạy thử, tên lab và image/version cụ thể chưa được cung cấp; chưa có output/ảnh console để đối chiếu trực tiếp.

**Trạng thái:** đã chạy theo xác nhận của người thực hành; bằng chứng version/console còn chờ bổ sung. Không ghi đây là kết quả được agent trực tiếp kiểm thử hoặc xác nhận firewall policy/services đã hoạt động.

Ảnh HQ bổ sung trong cùng trao đổi vẫn gồm hai Core, hai Access và bốn VPCS, khớp cabling đã ghi nhận. ASAv/Alpine không thuộc topology HQ hiện tại. Sprint 0 kiểm tra khả năng chạy node; không yêu cầu tích hợp firewall/server vào HQ. Không cần dựng lại hoặc thêm các node này chỉ để tiếp tục lưu baseline Campus.

Version/output/ảnh của node thử có thể bổ sung khi mở lại. Giữ việc thiếu artifact trong checklist nghiệm thu; tiếp tục công việc độc lập là thu thập cấu hình HQ. Tích hợp firewall/server và cấu hình dịch vụ vẫn theo sprint tương ứng.

### TC-S00-CORE1-CFG-01 — Lưu và đối chiếu running/startup CORE1

**Nguồn:** output `show running-config` và `show startup-config` do người thực hành cung cấp ngày 10/10/2026. Nội dung gửi lặp hai lần; chỉ lưu một bộ: [running-config CORE1](../../configs/core/sprint-00/CORE1.cfg), [startup-config CORE1](../../configs/core/sprint-00/CORE1.startup.cfg). File `.cfg` giữ phần cấu hình, bỏ prompt/preamble và chuẩn hóa khoảng trắng cuối dòng. Agent không chạy lệnh trên thiết bị.

**Metadata console:** running báo `1272 bytes`, last change `07:42:47 UTC Sat Oct 10 2026`; startup báo `Using 785 out of 524288 bytes, uncompressed size = 1271 bytes`, last change `14:49:27 UTC Thu Oct 8 2026`. Đây là thời gian thay đổi cấu hình, không phải timestamp thu thập.

- Các lệnh cấu hình running và startup khớp nhau khi bỏ dòng chú thích, dòng trống và metadata; không kết luận hai bản giống từng byte. Không cần ghi đè startup chỉ để đồng bộ timestamp.
- Vlan10 có `10.10.10.2 255.255.255.0`; HSRP group 10 VIP `10.10.10.1`, priority 110 và preempt, khớp baseline CORE1 đã nhận.
- Mode STP đã cấu hình `rapid-pvst`; output root/port role/state bổ sung ở TC-S00-CORE-RUNTIME-01. Chưa có bài test hội tụ để nghiệm thu RSTP Campus.
- Ethernet0/0 có description `TRUNK_TO_CORE2`, encapsulation dot1q, mode trunk và allowed VLAN chỉ `10`. Runtime bổ sung xác nhận trunking, native VLAN1; native VLAN99 chưa được cấu hình trong bản này.
- Ethernet0/1 và Ethernet0/2 chưa có lệnh cấu hình trunk rõ ràng. Hoàn thiện uplink Campus theo trình tự VLAN/trunk/STP ở Sprint 1.
- Ethernet1/0 còn cấu hình test: access VLAN10, `spanning-tree portfast edge`. Ảnh HQ hiện tại không có VPCS tại cổng này; giữ lại trong snapshot, review nhu cầu cổng test trước Sprint 1.
- Không thấy lệnh `ip routing` trong bản được cung cấp. `ip cef` và HSRP không đủ chứng minh forwarding liên VLAN; kiểm chứng runtime/chuyển tiếp ở Sprint 1, chưa kết luận trạng thái routing từ riêng snapshot này.

**Đánh giá:** PASS cho việc lưu và đối chiếu baseline cấu hình CORE1. Đã bổ sung `show vlan brief` tại TC-S00-CORE-RUNTIME-01; chưa backup đầy đủ VLAN database hoặc kiểm thử import/khôi phục. Không đánh dấu cấu hình Campus hoặc toàn bộ baseline HQ hoàn tất.

### TC-S00-CORE2-CFG-01 — Lưu và đối chiếu running/startup CORE2

**Nguồn:** output `show running-config` và `show startup-config` do người thực hành cung cấp ngày 10/10/2026. Lưu riêng [running-config CORE2](../../configs/core/sprint-00/CORE2.cfg) và [startup-config CORE2](../../configs/core/sprint-00/CORE2.startup.cfg), bỏ prompt/preamble và chuẩn hóa khoảng trắng cuối dòng. Agent không chạy lệnh trên thiết bị.

**Metadata console:** running báo `1142 bytes`, last change `07:42:46 UTC Sat Oct 10 2026`; startup báo `Using 713 out of 524288 bytes, uncompressed size = 1134 bytes`, last change `14:53:56 UTC Thu Oct 8 2026`. Đây là thời gian thay đổi cấu hình, không phải timestamp thu thập.

- **Có khác biệt lệnh:** dưới `line vty 0 4`, running có `login`, startup không có. Các lệnh còn lại khớp khi bỏ chú thích/dòng trống. Không ghi hai bản đã đồng bộ; giữ nguyên hai snapshot để review baseline quản trị trước khi quyết định lưu đè hoặc chỉnh VTY.
- Cả hai bản có Vlan10 `10.10.10.3 255.255.255.0`, HSRP group 10 VIP `10.10.10.1` và preempt. Không có dòng cấu hình priority tường minh; output runtime đã lưu tại TC-S00-CORE-02 báo priority 100, Standby, khớp vai trò dự kiến.
- Cả hai có `spanning-tree mode rapid-pvst`. Ethernet0/0 description `TRUNK_TO_CORE1`, trunk dot1q, allowed VLAN chỉ `10`, tương ứng đầu CORE1. Output operational trunk/root/port role đã bổ sung tại TC-S00-CORE-RUNTIME-01; chưa có test hội tụ, không kết luận Campus hoàn tất.
- Ethernet0/1–0/2 chưa có cấu hình trunk rõ ràng; native VLAN99 chưa được cấu hình trong bản này. Hoàn thiện VLAN/trunk/STP ở Sprint 1.
- Không thấy `ip routing`; chưa kết luận forwarding liên VLAN từ riêng config, `ip cef` hoặc HSRP. Đã bổ sung `show vlan brief` CORE2 tại TC-S00-CORE-RUNTIME-01; chưa backup đầy đủ VLAN database.

**Đánh giá:** đã lưu và đối chiếu được baseline CORE2; IP/mask/HSRP khớp thiết kế. **Running/startup chưa khớp hoàn toàn** do dòng VTY `login`, cần quyết định baseline và kiểm tra lại khi xử lý phần quản trị. Khác biệt tìm thấy không nằm ở VLAN10/HSRP/trunk/STP. Chưa kiểm thử import/khôi phục, chưa đánh dấu toàn bộ baseline HQ hoàn tất.

### TC-S00-ACCESS-CFG-01 — Lưu và đối chiếu running/startup hai Access

**Nguồn:** output `show run` và `show start` của ACCESS2, sau đó ACCESS1, do người thực hành cung cấp ngày 10/10/2026. Agent không chạy lệnh trên thiết bị. File `.cfg` bỏ prompt/preamble và chuẩn hóa khoảng trắng cuối dòng; thời gian last change không phải timestamp thu thập.

- ACCESS1: [running](../../configs/access/sprint-00/ACCESS1.cfg), [startup](../../configs/access/sprint-00/ACCESS1.startup.cfg). Running báo `823 bytes`, last change `08:04:09 UTC Sat Oct 10 2026`. Startup báo `Using 536 out of 524288 bytes, uncompressed size = 816 bytes`, last change `08:00:54 UTC Sat Oct 10 2026`.
- ACCESS2: [running](../../configs/access/sprint-00/ACCESS2.cfg), [startup](../../configs/access/sprint-00/ACCESS2.startup.cfg). Running báo `816 bytes`; startup báo `Using 533 out of 524288 bytes, uncompressed size = 816 bytes`. Cả hai last change `08:02:24 UTC Sat Oct 10 2026`.
- ACCESS2 running/startup khớp về các lệnh và thời gian last change; không suy ra đã kiểm thử khôi phục hoặc giống từng byte từ số byte hiển thị.
- ACCESS1 running có `login` dưới `line vty 0 4`, startup không có; các lệnh còn lại khớp khi bỏ chú thích/dòng trống. Giống CORE2, cần chốt baseline quản trị trước khi quyết định đồng bộ và lấy bằng chứng mới; giữ nguyên snapshot gốc.
- Cả hai đã cấu hình `spanning-tree mode rapid-pvst`. Các Ethernet0/0–0/3 và Ethernet1/0–1/3 chưa có lệnh interface cụ thể; không có SVI quản trị hoặc cấu hình trunk/access VLAN tường minh. Đối chiếu với `show vlan brief` đã nhận: chỉ VLAN1 và 1002–1005. VLAN database không được coi là đã backup đầy đủ chỉ từ running-config.

**Đánh giá:** PASS cho lưu/đối chiếu snapshot hai Access trước triển khai Campus. Đã có đủ tám file running/startup của bốn switch HQ. CORE1 và ACCESS2 khớp lệnh; CORE2 và ACCESS1 còn khác VTY `login`. VLAN/trunk/quản trị Access tiếp nhận ở Sprint 1; mode rapid-pvst chưa thay thế kiểm tra root/port role/hội tụ. Chưa kiểm thử import/khôi phục hoặc nghiệm thu toàn bộ Sprint 0.

### TC-S00-CORE-RUNTIME-01 — VLAN10, trunk, STP và routes hai Core

**Nguồn:** [output CORE1](sprint-00-core1-vlan-trunk-stp-route-2026-10-10.txt), [output CORE2](sprint-00-core2-vlan-trunk-stp-route-2026-10-10.txt), người thực hành cung cấp ngày 10/10/2026. Agent không chạy lệnh; không có timestamp riêng cho từng lệnh. Giữ cả lỗi gõ `show lan br` trên CORE2 và lệnh đúng chạy ngay sau đó; lỗi gõ không phải lỗi tính năng.

**Expected:** VLAN10 hiện hữu, trunk Core–Core cho phép và chuyển tiếp VLAN10; hai Core nhận cùng STP root; có connected/local route đúng SVI. Đây là kiểm tra baseline VLAN10, chưa nghiệm thu toàn Campus hoặc hội tụ khi mất link.

- Cả hai có VLAN10 `MGMT` active. CORE1 liệt kê Et1/0 dưới VLAN10; các port Et0/1–0/2 hiện dưới VLAN1 ở cả hai Core. VLAN10 của CORE2 không có access port được liệt kê; trunk được xem ở output riêng, không suy ra VLAN10 bị cô lập từ cột Ports trống.
- Et0/0 cả hai: mode `on`, encapsulation `802.1q`, status `trunking`, native VLAN **1**. VLAN allowed, active và forwarding/not pruned đều là **10**. Hai đầu khớp trong baseline hiện tại; native VLAN99 theo thiết kế chưa triển khai, tiếp nhận Sprint 1.
- VLAN10 chạy protocol `rstp`. CORE1 tự nhận root: bridge/root MAC `aabb.cc00.1000`, priority tổng `32778` (base `32768` + VLAN10). CORE2 có bridge MAC `aabb.cc00.2000`, cùng priority, nhận root CORE1 qua Et0/0, cost 100. Hai Core có cùng base priority, chưa có phân cấp root primary/secondary tường minh trong snapshot; trạng thái root hiện tại phù hợp so sánh MAC của hai Core.
- CORE1 Et0/0 `Desg FWD`, Et1/0 `Desg FWD`, Type `Shr Edge` ở cổng test. CORE2 Et0/0 `Root FWD`. Không có bằng chứng đường dự phòng VLAN10 qua Access hoặc bài test hội tụ; output STP VLAN10 này chưa đại diện STP VLAN1/toàn bộ topology.
- Cả hai có connected route `10.10.10.0/24` qua Vlan10, local route tương ứng `.2/32` và `.3/32`; `Gateway of last resort is not set`. Phù hợp baseline một VLAN hiện tại; chưa có default/WAN route và chưa chứng minh forwarding giữa các VLAN.
- Xen giữa các lệnh CORE1 có log `*Oct 10 08:09:02.721: %HSRP-5-STATECHANGE: Vlan10 Grp 10 state Standby -> Active`. Log không ghi timezone và không có chuỗi sự kiện hai phía hoặc lệnh gây lỗi. Chỉ xác nhận đã ghi nhận một lần chuyển trạng thái; chưa xác định nguyên nhân, thời gian hội tụ hoặc vai trò CORE2 tại thời điểm đó. Không dùng log đơn lẻ làm PASS cho test switchover.

**Đánh giá:** PASS cho VLAN10/trunk Core–Core và trạng thái STP/routes được cung cấp. HSRP đã được kiểm tra lại tại TC-S00-HSRP-RECHECK-01; phép thử có kiểm soát sau đó đạt tại TC-S00-HSRP-SWITCHOVER-01. Nguyên nhân log riêng lúc 08:09 vẫn chưa rõ; chưa nghiệm thu toàn bộ Sprint 0.

### TC-S00-HSRP-RECHECK-01 — Xác nhận lại vai trò hai Core trước test

**Nguồn:** [CORE1](sprint-00-core1-hsrp-recheck-2026-10-10.txt), [CORE2](sprint-00-core2-hsrp-recheck-2026-10-10.txt), người thực hành cung cấp ngày 10/10/2026. Agent không chạy lệnh trên thiết bị.

**Expected:** SVI VLAN10 up/up, CORE1 Active priority110 và CORE2 Standby priority100, cả hai preempt và nhận đúng peer/VIP trước khi gây lỗi có kiểm soát.

- CORE1 `show clock`: `*08:15:17.257 UTC Sat Oct 10 2026`, tương ứng 15:15:17.257 Asia/Saigon theo đồng hồ thiết bị hiển thị. Vlan10 `10.10.10.2`, Method NVRAM, up/up. Cả hai lượt standby: group10, priority110, preempt, Active local, Standby `10.10.10.3`, VIP `10.10.10.1`.
- CORE2 `show clock`: `*08:15:52.531 UTC Sat Oct 10 2026`, tương ứng 15:15:52.531 Asia/Saigon theo đồng hồ thiết bị hiển thị. Vlan10 `10.10.10.3`, Method NVRAM, up/up. Cả hai lượt standby: group10, priority100, preempt, Standby local, Active `10.10.10.2`, VIP `10.10.10.1`.
- Kết quả vai trò/peer/VIP nhất quán giữa hai phía trong các mẫu đã nhận, không thấy hai Core cùng Active. Hai lượt trên mỗi node không có timestamp riêng, nên không khẳng định khoảng cách đúng 10 giây hoặc chứng minh ổn định liên tục; hai node cũng không được lấy mẫu đồng thời.

**Đánh giá:** PASS cho SVI và HSRP baseline trong các mẫu được cung cấp; đủ để hướng dẫn tiếp phép thử shutdown/no shutdown SVI VLAN10 và lưu trước/trong/sau. Không suy ra nguyên nhân log Standby → Active trước đó, độ chính xác đồng hồ, convergence, ping VIP hoặc client failover. Chưa thực hiện test gây lỗi trong bằng chứng này.

### TC-S00-HSRP-SWITCHOVER-01 — Shutdown SVI CORE1, chuyển vai trò và khôi phục

**Nguồn:** [Log trước/trong/sau](sprint-00-hsrp-switchover-2026-10-10.txt), do người thực hành cung cấp ngày 10/10/2026. Agent không chạy lệnh hoặc thay đổi thiết bị. Giữ lệnh `show int br` bị báo lỗi và lệnh đúng ngay sau đó; lỗi gõ không phải lỗi bài test.

**Topology/phạm vi:** hai Core trao đổi VLAN10 qua trunk Et0/0; SVI CORE1 `10.10.10.2/24`, CORE2 `10.10.10.3/24`, HSRP group10 VIP `10.10.10.1`. Chỉ shutdown SVI CORE1; không tắt node, trunk hoặc uplink Access.

**Expected:** CORE2 nhận Active khi SVI CORE1 bị shutdown; sau no shutdown, CORE1 trở lại Active theo priority110/preempt, CORE2 Standby, SVI cả hai up/up và peer/VIP khớp.

- **Trước test:** CORE1 clock `08:26:01.661 UTC`, Active110/preempt, Standby peer `.3`; CORE2 clock `08:26:15.229 UTC`, Standby100/preempt, Active peer `.2`. VIP cả hai `.1`.
- **Gây lỗi:** CORE1 chạy `conf t` → `int vlan 10` → `shut` → `end`. Log interface administratively down lúc `08:27:23.007`, line protocol down và HSRP Active → Init lúc `08:27:24.012`. Clock quan sát `08:27:25.551 UTC`; CLI SVI administratively down/down, HSRP Init, Active/Standby unknown.
- **Trong lỗi:** CORE2 clock `08:28:11.644 UTC`, HSRP Active local, priority100/preempt, Standby unknown, VIP `.1`. Không có log thời điểm chính xác CORE2 chuyển Active; đây là thời điểm quan sát trạng thái đã Active.
- **Khôi phục:** CORE1 chạy `conf t` → `int vlan 10` → `no shut` → `end`. Log interface up `08:28:50.468`, line protocol up `08:28:51.474`, HSRP Listen → Active `08:28:52.765`.
- **Sau khôi phục:** CORE1 clock `08:29:00.821 UTC`, SVI `.2` up/up, Active110, Standby peer `.3`. CORE2 clock `08:29:34.570 UTC`, SVI `.3` up/up, Standby100, Active peer `.2`. Cả hai preempt/VIP `.1`, khớp baseline trước test.

**Actual/đánh giá: PASS** cho chuyển vai trò HSRP khi shutdown SVI CORE1 và khôi phục trạng thái vận hành sau no shutdown. Có lệnh gây lỗi/khôi phục cùng bằng chứng hai phía, thay thế trạng thái chỉ xác nhận lịch sử cho phép thử này.

**Giới hạn:** không đo convergence hoặc packet loss từ khoảng cách các lần `show clock`; các mẫu không đồng thời, độ đồng bộ/độ chính xác clock chưa được xác minh. Không có ping/capture client, không nghiệm thu full node failover, Inter-VLAN hoặc HA end-to-end. Log riêng lúc 08:09 không được giải thích hồi tố bằng bài test mới. Output không có lệnh lưu startup; cần đối chiếu config CORE1 sau test trước export, vẫn theo dõi khác biệt VTY CORE2/ACCESS1. Khôi phục SVI vận hành đã xác nhận, import/reload/khôi phục toàn lab chưa kiểm thử.

### TC-S00-POST-CFG-01 — Đối chiếu cấu hình hai Core sau phép thử

**Nguồn:** người thực hành cung cấp running/startup cả CORE1 và CORE2 ngày 10/10/2026; agent không chạy lệnh trên thiết bị. Giữ snapshot trước test, lưu riêng bản sau test:

- CORE1: [running sau test](../../configs/core/sprint-00/CORE1.post-switchover.cfg), [startup sau test](../../configs/core/sprint-00/CORE1.post-switchover.startup.cfg).
- CORE2: [running sau test](../../configs/core/sprint-00/CORE2.post-switchover.cfg), [startup sau test](../../configs/core/sprint-00/CORE2.post-switchover.startup.cfg).
- [Metadata console và sự kiện HSRP xen giữa](sprint-00-post-switchover-config-review-2026-10-10.txt). File cấu hình bỏ prompt/preamble và khoảng trắng cuối dòng; không khẳng định giống từng byte.

**Đối chiếu:** các lệnh cấu hình của từng bản sau test khớp bản tương ứng trước test khi bỏ chú thích/dòng trống. Không còn lệnh `shutdown` dưới SVI VLAN10 trong running được cung cấp; IP/mask, HSRP, trunk và mode STP giữ nguyên. CORE1 running/startup khớp lệnh; CORE2 vẫn chỉ running có `login` dưới `line vty 0 4`, startup không có. Không tự sửa hoặc ghi đè startup/snapshot gốc.

**Metadata:** CORE1 running `1272 bytes`, last change `08:35:12 UTC Sat Oct 10 2026`; startup `Using 785 out of 524288 bytes, uncompressed size = 1271 bytes`, last change `14:49:27 UTC Thu Oct 8 2026`. CORE2 running `1142 bytes`, last change `08:35:11 UTC Sat Oct 10 2026`; startup `Using 713 out of 524288 bytes, uncompressed size = 1134 bytes`, last change `14:53:56 UTC Thu Oct 8 2026`. Last change không phải timestamp thu thập.

**Sự kiện riêng:** xen giữa running/startup CORE1 có log `*Oct 10 08:35:49.118: %HSRP-5-STATECHANGE: Vlan10 Grp 10 state Standby -> Active`. Người thực hành xác nhận có thao tác trước lần lấy output này, nhưng chưa nêu thao tác cụ thể/node/interface/thời điểm. Không tự kết luận reload, preempt hoặc lỗi tự phát; log không thuộc phép thử có kiểm soát lúc 08:26–08:29. Giữ PASS cho test trước đó, không dùng snapshot config làm chứng minh vai trò/độ ổn định liên tục sau log mới.

**Đánh giá:** PASS cho lưu/đối chiếu cấu hình hai Core sau test; CORE1 khớp running/startup, CORE2 còn khác biệt VTY đã biết. Đủ để tiếp tục thu thập export topology kèm các snapshot và giới hạn này; chưa nghiệm thu khôi phục từ export hoặc toàn bộ Sprint 0.

### TC-S00-EXPORT-01 — Kiểm tra ZIP topology HQ lần đầu (lịch sử)

**Nguồn lịch sử:** [kết quả kiểm tra export lần đầu](sprint-00-export-inspection-2026-10-10.txt). ZIP 973 bytes của lần này không còn trong thư mục; file mang tên HQ hiện tại đã được thay bằng bản sạch 952 bytes, kiểm tra riêng tại TC-S00-EXPORT-FINAL-01. Không áp dụng hash/nhận xét node dư của bản cũ cho file mới. Agent chỉ kiểm tra tại local, không import vào EVE hoặc thay đổi lab.

**Artifact:** ZIP 973 bytes, chứa một file `Enterprise-Network-HQ.unl` 4411 bytes. SHA256 `A6D8EF6A334878095FEA17F13ACCE116265AA3016500D5C86BAB90FC97F2D459`. Lab name `Enterprise-Network-HQ`, XML version attribute `1`; không dùng thuộc tính này như version EVE-NG.

**Expected:** bốn switch và bốn VPCS test, chín link theo cabling HQ; đủ metadata để đối chiếu và mô tả rõ có/không cấu hình nhúng.

- Cả chín network bridge có đúng hai endpoint; năm link hạ tầng và bốn link VPCS khớp sơ đồ/bảng cổng. Không có PC-MGMT tạm nối CORE1 e1/0 trong export.
- **Có node thừa:** chín node, gồm bốn switch và năm VPCS. Có hai `PC-GUEST`: ID9 có eth0 nối ACCESS2 e0/3 qua network9, đúng topology; ID8 không có interface kết nối, ở `left=1770`, `top=1395`, nằm xa phần sơ đồ chính. Cần xác định và dọn riêng ID8 trong EVE, giữ ID9, rồi xuất revision mới; không chỉnh file export gốc thay cho bằng chứng lab đã sửa.
- Cả bốn switch dùng image đầy đủ `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`. Thuộc tính RAM CORE1/CORE2 `512`, ACCESS1/ACCESS2 `256`; Ethernet groups tương ứng `3`/`2`, NVRAM `1024`. Đây là metadata node trong export, khác memory IOS runtime; không có thuộc tính vCPU hoặc bằng chứng tải/node lúc đo EVE.
- ZIP chỉ có `.unl`, không có file config riêng; XML không có khối `config`/`configs` chứa payload, các node đều `config="0"`. **Chưa có cấu hình thiết bị nhúng.** Phải đi kèm snapshot `.cfg` và evidence VLAN; không coi topology ZIP là backup đầy đủ startup/VLAN database.
- Không thấy trường credential/key trong XML đã kiểm tra. Chưa thử import/khôi phục; không sửa ZIP hoặc xóa node từ phía agent.

**Đánh giá:** PASS cho việc nhận/đọc ZIP và đối chiếu chín kết nối; **chưa nghiệm thu topology sạch** do PC-GUEST ID8 dư. Giữ export này làm evidence, chờ revision sau dọn node; không đánh dấu Sprint 0 Done từ riêng export.

### TC-S00-EXPORT-FINAL-01 — Export HQ hiện tại sau dọn node dư

**Nguồn:** [ZIP HQ hiện tại](../../eve-exports/sprint-00-hq-2026-10-10.zip), kiểm tra read-only ngày 10/10/2026 theo yêu cầu người thực hành chốt sprint. [Biên bản cuối](sprint-00-export-final-inspection-2026-10-10.txt) ghi hash/CRC/node/link. [Kiểm tra revision trước](sprint-00-export-v2-inspection-2026-10-10.txt) giữ như lịch sử, tên file/hash của lần đó không xác định ZIP hiện tại.

**Expected:** bốn switch, bốn VPCS, chín link theo physical topology, không trùng tên/ID hoặc node PC-GUEST dư; biết rõ giới hạn config nhúng.

**Actual:** ZIP 952 bytes, SHA256 `CAD0EE62F6A1213B08147D5BB7471EBFB873D0B5F28FD0DB047276B6395F539F`, một `Enterprise-Network-HQ.unl` 4241 bytes. CRC/XML PASS; tám node tên/ID duy nhất, chín network mỗi network hai endpoint, cả chín link khớp bảng cổng. PC-GUEST ID8 dư không còn; ID9 vẫn nối ACCESS2 e0/3. Không có PC-MGMT tạm. Bốn switch dùng image đầy đủ `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`, RAM Core512/Access256 theo export, không có node vCPU.

**Đánh giá: PASS** cho artifact topology sạch và đối chiếu HQ. Tất cả node config=0, không có payload/file cấu hình nhúng; đi kèm 12 snapshot `.cfg` và VLAN evidence. Chưa import/reload/restore toàn lab, không ghi PASS cho các phần này. Không có credential/key field được quan sát trong XML.

## 8. Quyết định đóng Sprint 00 ngày 10/10/2026

**CLOSED WITH FOLLOW-UPS:** chấp nhận phạm vi nền tảng và hồ sơ thiết kế theo [biên bản](../sprint-reports/sprint-00-closeout.md). ZIP cuối đã sạch, config/runtime và phép thử HSRP SVI có log khôi phục; logical/physical/IP được review, workflow/liên kết/ignore kiểm tra và bộ hồ sơ được lưu bằng commit local. Push/PR/merge đợt closeout chưa thực hiện.

Giữ mức bằng chứng thực tế: IOSv 4/5 là kết nối lịch sử, chưa chứng minh ổn định; ASAv/Alpine boot theo xác nhận người dùng, artifact thiếu. Chấp nhận các ngoại lệ này để kết thúc phần chuẩn bị, bàn giao trước NET-301/302, NET-503 và NET-701. VTY CORE2/ACCESS1, Campus/Inter-VLAN, full HA client, WAN và restore chưa thử có story/mốc tiếp nhận; không chuyển thành PASS. Các ghi chú chờ nghiệm thu trong test case phía trên mô tả thời điểm thu thập; quyết định hiện tại theo mục này.

### Việc tiếp theo

1. Bắt đầu Sprint 1 từ baseline hiện tại: VLAN/trunk/native99, SVI Access/VTY, STP/root và Inter-VLAN theo NET-101–104; kiểm tra STP trước bật đủ uplink dự phòng.
2. Nhận các phần bổ sung theo [mục bàn giao](../sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện); không yêu cầu lặp test HSRP SVI hoặc export sạch đã đạt khi chưa có thay đổi mới.
3. Nếu cần xuất bản bộ closeout, thực hiện push/PR/review riêng; không force-add `ai-agent/`. Trước dùng bản restore để thay đổi lớn, cần test import và config/VLAN/runtime trong lab riêng.
