# BÁO CÁO SPRINT 0 — Foundation & Network Design

| Thông tin | Giá trị |
|---|---|
| Dự án | **Enterprise Network Design & Implementation — EVE-NG** |
| Sprint | **Sprint 0 — Chuẩn bị môi trường và thiết kế** |
| Epic | **EPIC-NET-00 — Project Foundation** |
| Ngày tổng hợp | **09/10/2026** |
| GitHub | https://github.com/lequocban/enterprise-network-lab |
| Nhánh mặc định quan sát được | **`master`** |
| Trạng thái đánh giá | **Đã đạt nền tảng kỹ thuật HQ; còn đóng hồ sơ minh chứng và thiết kế WAN** |

## 1. Tóm tắt kết quả (Executive Summary)

Sprint 0 đã hình thành môi trường thực hành **EVE-NG hoạt động**, kiểm thử kết nối cơ bản giữa hai router Cisco IOSv và triển khai thử nghiệm kiến trúc **HQ Campus Dual Core**. Topology HQ hiện có CORE1, CORE2, ACCESS1, ACCESS2, bốn VPCS cho các phòng ban và một PC-MGMT phục vụ kiểm thử VLAN 10.

Kết quả kỹ thuật nổi bật: CORE1 và CORE2 cùng có SVI VLAN 10 ở trạng thái **up/up**, tham gia **HSRP Group 10**, bầu chọn thành công **CORE1 Active (priority 110)** và **CORE2 Standby (priority 100)**, sử dụng gateway ảo **10.10.10.1**.

**Đánh giá:** mục tiêu nền tảng kỹ thuật tại HQ đã đạt. Tuy nhiên, topology tổng thể có WAN/Firewall/DMZ/Branch, một số bằng chứng boot và hồ sơ GitHub vẫn chưa hoàn thiện theo *Definition of Done* chung của roadmap. Vì vậy báo cáo này ghi nhận trạng thái **sẵn sàng đóng Sprint 0 có điều kiện**, không đánh dấu toàn bộ tiêu chí là DONE khi chưa có bằng chứng.

## 2. Mục tiêu và phạm vi Sprint

Theo roadmap, Sprint 0 phải chuẩn bị môi trường EVE-NG, thiết kế kiến trúc, xây dựng IP Addressing Plan và khởi tạo GitHub Repository. Gồm bốn Story:

- **NET-001 — Environment Setup & Validation:** Xác nhận VM, image, console và connectivity.
- **NET-002 — Network Topology Design:** Thiết kế mô hình HQ và kiến trúc Enterprise mục tiêu.
- **NET-003 — IP Addressing Plan:** Quy hoạch VLAN, subnet, HSRP VIP, DMZ, Branch, Transit và Loopback.
- **NET-004 — Repository Foundation:** README, cấu trúc tài liệu, `.gitignore`, Git workflow.

**Ngoài phạm vi xác nhận của Sprint 0:** hoàn thiện các VLAN người dùng 20–60, RSTP, LACP, Inter-VLAN Routing nhiều VLAN, OSPF, BGP, Firewall Policy, NAT, IPSec, monitoring và automation. Đây là nội dung của các Sprint tiếp theo.

## 3. Môi trường và tài nguyên

| Thành phần | Cấu hình / kết quả | Đánh giá |
|---|---|---|
| Host | Lenovo Legion 5, Ryzen 7 7840H, RAM vật lý 16 GB | Cấu hình lab đã sử dụng |
| Hypervisor | VMware Workstation | Đang sử dụng |
| EVE-NG | Community 6.2, VM cấp 8 GB RAM, 4 vCPU, disk 120 GB | Hoạt động |
| RAM từ `free -h` | Tổng 7.7 GiB, available 6.3 GiB | **Verified** |
| CPU / virtualization | `nproc` = 4; `(vmx\|svm)` = 4 | **Verified** |
| Root filesystem | 116 GB tổng, 12 GB đã sử dụng, 100 GB còn trống | **Verified** |
| Router | Cisco IOSv 15.6.2T | Đã boot, cấu hình và ping |
| Switch/Core | Cisco IOL L2 15.2 | Đã chứng minh SVI/HSRP trên CORE1, CORE2 |
| Firewall | Cisco ASAv 9.8.x | Người dùng xác nhận đã cài; cần bổ sung ảnh boot |
| Server | Alpine Linux (thay Ubuntu để nhẹ hơn) | Người dùng xác nhận đã cài; dịch vụ cấu hình sau |
| Client | VPCS | Có mặt trong HQ topology |

**Giới hạn tài nguyên:** VM EVE-NG được cấp 8 GB RAM trên host 16 GB. Phương án vận hành là chạy từng nhóm node theo Sprint, không mặc định bật toàn bộ multi-site cùng một lúc.

## 4. Kết quả theo Story

| Story | Công việc đã thực hiện | Bằng chứng | Tình trạng Review |
|---|---|---|---|
| **NET-001** | Kiểm tra RAM/CPU/disk, nested virtualization, IOSv ping; IOL L2 chạy được; ASAv/Alpine đã cài | Console và ping log | **Đạt phần lớn; thiếu ảnh boot Linux/ASA** |
| **NET-002** | Vẽ và đấu nối HQ Dual Core + Dual Uplink; SVI VLAN10; HSRP hai Core | Topology screenshot, `show standby brief` hai Core | **HQ validation PASS; full-site topology pending** |
| **NET-003** | Chốt subnet HQ và HSRP plan; phân bổ sơ bộ Branch, DMZ, Transit, Loopback | `docs/ip-addressing-plan.md` | **Draft; chưa chốt WAN interface** |
| **NET-004** | Public GitHub repo có README, `.gitignore`, thư mục docs | GitHub public repository | **Repo hoạt động; chưa commit bộ cập nhật này** |

Người thực hành đã xác nhận **NET-002 hoàn thành và hệ thống hoạt động ổn định**. Trong báo cáo, xác nhận này được ghi nhận; các bài test chưa có output lưu trữ riêng không bị chuyển thành `Verified`.

## 5. Bằng chứng kiểm thử kỹ thuật

### 5.1. TC-NET001-01 — Tài nguyên EVE-NG

Kết quả thực tế từ console:

```text
free -h  -> RAM total: 7.7Gi, available: 6.3Gi
           Swap: 4.0Gi, used: 0B

df -h /  -> Filesystem: 116G, Used: 12G, Avail: 100G, Use%: 11%
nproc    -> 4
egrep -c '(vmx|svm)' /proc/cpuinfo -> 4
```

**Kết luận:** PASS cho khả năng nhận tài nguyên/nested virtualization. Đây là ảnh chụp tại một thời điểm, chưa đại diện cho tải cực đại khi chạy tất cả node.

### 5.2. TC-NET001-02 — IOSv Router-to-Router Ping

- R1 `Gi0/0`: `10.255.0.1/30`; R2: `10.255.0.2/30`.
- R1 thông báo Ethernet/line protocol chuyển sang **up**.
- Ping R1 → R2: **4/5 phản hồi, success rate 80%**, RTT **1/1/2 ms**.
- Cấu hình R1 đã `write memory` thành công.

**Kết luận:** PASS về khả năng định tuyến IP giữa hai router. Chưa có output ping lặp lại 10/10 và chiều R2→R1; gói ICMP đầu thất bại có thể liên quan ARP nhưng chưa được xác thực độc lập.

### 5.3. TC-NET002-01 — SVI và Connected Route

Sau khi đưa một cổng access vào VLAN 10, trạng thái SVI CORE1 chuyển **up/up** và bảng định tuyến xuất hiện:

```text
Vlan10  10.10.10.2  YES NVRAM  up  up
VLAN 10 (MGMT): active, access port Et1/0
C 10.10.10.0/24 is directly connected, Vlan10
L 10.10.10.2/32 is directly connected, Vlan10
```

**Kết luận:** PASS. Image IOL L2 đã chấp nhận `ip routing`, `interface vlan` và `standby`; tuy nhiên, **cần kiểm thử chuyển tiếp giữa VLAN 20/30** mới xác nhận đầy đủ vai trò multilayer switch.

### 5.4. TC-NET002-02 — HSRP Active/Standby

| Kiểm tra | CORE1 | CORE2 |
|---|---|---|
| VLAN10 SVI | `10.10.10.2/24`, up/up | `10.10.10.3/24`, up/up |
| HSRP Group | 10 | 10 |
| Priority | **110** | **100** |
| Preempt | Bật | Bật |
| Vai trò | **Active (local)** | **Standby (local)** |
| Thiết bị còn lại thấy được | Standby `10.10.10.3` | Active `10.10.10.2` |
| HSRP VIP | **10.10.10.1** | **10.10.10.1** |

**Kết luận: VERIFIED PASS.** Hai switch đã nhận diện peer và thống nhất Active/Standby, không xuất hiện split-brain trong ảnh chụp CLI được cung cấp.

### 5.5. TC-NET002-03 — HSRP Switchover

Kịch bản: shutdown SVI VLAN10 trên CORE1 → CORE2 chuyển Active → khôi phục CORE1, chờ preempt lấy lại Active. Người dùng xác nhận đã kiểm thử và “mọi thứ đều ổn”.

**Trạng thái: REPORTED PASS; cần bổ sung `show standby brief` trước/trong/sau sự cố.** Đây chỉ là thử chuyển vai trò gateway, chưa phải bài chứng minh client vẫn kết nối được nếu CORE1 mất nguồn hoàn toàn.

### 5.6. Topology HQ thực tế

![Ảnh topology HQ trên EVE-NG](../../diagrams/hq-campus-topology.svg)

Trong ảnh có 10 kết nối: CORE1↔CORE2, hai uplink đến từng ACCESS, PC-MGMT nối trực tiếp CORE1 và bốn PC phòng ban nối ACCESS1/2. Bảng interface chi tiết: [Network Architecture](../network-architecture.md).

## 6. Kết quả IP Addressing Plan

| Mạng | Subnet | Gateway / chủ thể | Trạng thái |
|---|---|---|---|
| HQ Management VLAN 10 | `10.10.10.0/24` | HSRP `10.10.10.1` | **Đã hoạt động** |
| HQ IT VLAN 20 | `10.10.20.0/24` | VIP `10.10.20.1` | Kế hoạch |
| HQ HR VLAN 30 | `10.10.30.0/24` | VIP `10.10.30.1` | Kế hoạch |
| HQ Accounting VLAN 40 | `10.10.40.0/24` | VIP `10.10.40.1` | Kế hoạch |
| HQ Server VLAN 50 | `10.10.50.0/24` | VIP `10.10.50.1` | Kế hoạch |
| HQ Guest VLAN 60 | `10.10.60.0/24` | VIP `10.10.60.1` | Kế hoạch |
| DMZ | `10.10.100.0/24` | ASA `10.10.100.1` dự kiến | Draft |
| Branch 1 | `10.20.10.0/24` | `10.20.10.1` dự kiến | Draft |
| Branch 2 | `10.30.10.0/24` | `10.30.10.1` dự kiến | Draft |
| WAN Transit | Pool `10.255.0.0/24` | Chia `/29`/`/30` theo kết nối | Draft |
| Loopback | Pool `10.255.255.0/24` | Mỗi Core/Router một `/32` | Draft |

Tham khảo [IP Addressing Plan](../ip-addressing-plan.md) để xem đầy đủ IP SVI, host, interface mapping và phân bổ transit ứng viên. Các mạng `/29` multi-access yêu cầu broadcast domain chung; không áp dụng như nối điểm-điểm nếu chưa kiểm tra sơ đồ cổng thực tế.

## 7. Quyết định thiết kế, rủi ro và giải pháp

| ID | Vấn đề | Ảnh hưởng | Xử lý |
|---|---|---|---|
| D-01 | Tận dụng IOL L2 làm Core vì SVI/HSRP chạy được | Đỡ thêm image; chưa chứng minh Inter-VLAN forwarding | Test VLAN 20↔30 trong Sprint 1 |
| D-02 | Thay Ubuntu Server bằng Alpine Linux | Giảm RAM, đổi quy trình cấu hình dịch vụ | Viết hướng dẫn dành riêng cho Alpine ở Sprint 7 |
| R-01 | Access có hai uplink và tồn tại vòng lặp L2 | STP có thể chặn cổng / sai root | Thiết kế Rapid STP, test hội tụ Sprint 1 |
| R-02 | PC-MGMT nối thẳng CORE1 | CORE1 mất nguồn thì client mất đường vật lý | Chuyển PC-MGMT về ACCESS1/2 trước test failover end-to-end |
| R-03 | Một Cisco ASAv | Single point of failure tại firewall | Ghi nhận giới hạn, không tuyên bố Firewall HA |
| R-04 | Chưa có interface map WAN/Firewall/Branch cuối cùng | Subnet transit `/29` còn mang tính đề xuất | Chốt logical/physical WAN trước cấu hình Sprint 3–6 |
| R-05 | Host chỉ có 16 GB RAM | Khó mở tất cả node cùng lúc | Chạy module theo Sprint, đo RAM thực tế |
| R-06 | Repo public có thể làm lộ thông tin | Rò rỉ secret/password/PSK | Sanitize cấu hình trước khi push |

**Lưu ý thiết kế LACP:** Không gộp uplink ACCESS1 đến hai Core độc lập thành một EtherChannel thông thường nếu không có công nghệ multi-chassis EtherChannel tương thích.

## 8. Sprint Review — Definition of Done

| Tiêu chí trong roadmap | Đánh giá | Căn cứ |
|---|---|---|
| EVE-NG hoạt động | **PASS** | Console + tài nguyên |
| Router và Core boot/chạy được | **PASS với node đã test** | Ping IOSv, HSRP Dual Core |
| Sơ đồ Logical Topology toàn doanh nghiệp | **PARTIAL** | Mới có mục tiêu/roadmap và HQ as-built |
| Sơ đồ Physical/Virtual Topology toàn doanh nghiệp | **PARTIAL** | Có HQ wired, chưa nối WAN/DMZ/Branch |
| IP Plan | **DRAFT** | HQ có baseline; WAN/DMZ/Branch chưa xác nhận final |
| GitHub Repository hoạt động | **PASS** | Public repo trên nhánh `master` |
| Minh chứng và docs đã commit | **PENDING** | Gói tài liệu đang chuẩn bị, chưa tự động push |
| Tự giải thích, retrospective và review | **CHƯA GHI NHẬN ĐỦ** | Cần người thực hành tự xác nhận/ghi lại |

**Sprint Review Outcome:** **Conditional closeout / chờ hoàn tất tài liệu**. Nền tảng kỹ thuật HQ đủ điều kiện để học tiếp Sprint 1. Để đóng Sprint 0 chính thức theo chuẩn DoD, còn cần hoàn thiện sơ đồ toàn hệ thống, lưu bằng chứng ASAv/Alpine và commit tài liệu vào GitHub.

## 9. Sprint Retrospective

**Những điểm làm tốt:** tái sử dụng image đã vận hành; kiểm tra khả năng SVI/HSRP bằng lệnh Cisco thực tế thay vì suy đoán qua nhãn L2; thiết kế sớm Dual Core và Dual Uplink; tách rõ as-built HQ và WAN kế hoạch.

**Những điểm cần cải thiện:** chụp/đặt tên ảnh ngay lúc test; ghi `PASS / REPORTED / DRAFT` nhất quán; commit sau từng Story; chốt interface WAN trước khi cấu hình IP chính thức; không dùng kết quả HSRP một VLAN để kết luận toàn bộ kiến trúc dự phòng đã hoàn thiện.

## 10. Sprint 1 — Kế hoạch tiếp theo

1. Lưu các minh chứng còn thiếu và hoàn tất Logical/Physical topology toàn hệ thống ở mức thiết kế.
2. Tạo VLAN `10,20,30,40,50,60,99` và gán access port đúng phòng ban.
3. Cấu hình trunk 802.1Q giữa Core↔Access; chốt native/allowed VLAN và kiểm tra STP.
4. Test cách ly VLAN trước khi bật liên VLAN; tiếp theo cấu hình SVI và xác minh IT↔HR/Server.
5. Chọn Rapid STP Root Primary/Secondary và thử shutdown uplink chính có kiểm soát.
6. Ghi cấu hình/ảnh và thực hiện Git workflow `feature/NET-101-vlan` → Pull Request → **`master`**.

## 11. Tài liệu tham chiếu

- [NET-002 Architecture / Interface Mapping](../network-architecture.md)
- [NET-003 IP Addressing Plan](../ip-addressing-plan.md)
- [Sprint 0 Test Results](../test-results/sprint-00-validation.md)
- [HQ EVE-NG topology image](../../diagrams/hq-campus-topology.svg)
- [Project Roadmap](../enterprise-network-design-implementation-roadmap.md)
- [Public GitHub Repository](https://github.com/lequocban/enterprise-network-lab)

> **Ghi chú phát hành:** Báo cáo được tổng hợp từ log/screenshot đã gửi trong cuộc trò chuyện và cấu trúc GitHub công khai kiểm tra ngày 09/10/2026. Báo cáo đang được đưa lên nhánh `docs/sprint-0-closeout` để review; chỉ được coi là phát hành chính thức sau khi merge vào `master`.