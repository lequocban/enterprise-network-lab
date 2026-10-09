# Báo cáo Sprint 0 — Chuẩn bị môi trường và thiết kế mạng

**Dự án:** Enterprise Network Design & Implementation (EVE-NG)  
**Thời gian tổng hợp:** 09/10/2026  
**Người thực hiện:** lequocban  
**Phạm vi:** NET-001 đến NET-004  
**Trạng thái:** Hoàn thành nền tảng kỹ thuật HQ; đã chốt thiết kế IP v1.0; còn bổ sung minh chứng và thiết kế cổng WAN khi triển khai các Sprint sau.

## 1. Công việc đã thực hiện

Sprint này tập trung vào việc chuẩn bị môi trường thực hành và thống nhất phương án thiết kế trước khi cấu hình hệ thống lớn.

- Cài EVE-NG Community 6.2 trên VMware Workstation, cấu hình VM 4 vCPU, 8 GB RAM, 120 GB disk.
- Kiểm tra tài nguyên, nested virtualization và kết nối giữa hai router IOSv.
- Xác nhận image IOL L2 hiện có dùng được các lệnh SVI và HSRP, từ đó giữ phương án hai Core làm gateway Layer 3 thay vì thêm router-on-a-stick.
- Dựng mạng HQ Campus gồm CORE1, CORE2, ACCESS1, ACCESS2; các VPCS đi kèm chỉ phục vụ test.
- Kiểm tra SVI VLAN10, trạng thái Active/Standby của HSRP và quy trình chuyển vai trò.
- Thiết kế và **chốt IP Addressing Plan v1.0** cho HQ, DMZ, Branch 1/2, transit và Loopback.
- Tổ chức GitHub repository để lưu tài liệu, cấu hình và các báo cáo Sprint tiếp theo.

## 2. Kết quả kiểm tra

| Nội dung | Kết quả ghi nhận | Đánh giá |
|---|---|---|
| EVE-NG VM | RAM 7.7 GiB, available 6.3 GiB; CPU 4 vCPU; còn 100 GB disk | Đạt |
| Router R1 ↔ R2 | Ping R1 đến R2 thành công 4/5 gói (80%), RTT 1/1/2 ms | Có kết nối; chưa có test 10/10 lưu lại |
| VLAN 10 CORE1 | 10.10.10.2/24, SVI up/up | Đạt |
| VLAN 10 CORE2 | 10.10.10.3/24, SVI up/up | Đạt |
| HSRP Group 10 | CORE1 Active (110), CORE2 Standby (100), VIP 10.10.10.1 | Đạt theo CLI |
| HSRP switchover | Đã được người thực hành xác nhận hoạt động | Cần lưu log trước/trong/sau |
| Inter-VLAN Routing | Chưa thử giữa các VLAN khác nhau | Sang Sprint 1 |
| ASAv và Alpine | Đã cài đặt | Chưa lưu đủ ảnh console trong repo |

Các giá trị trong bảng lấy từ output console đã được lưu trong trao đổi lab. Không tính các tính năng chưa thực hiện là kết quả hoàn thành.

## 3. Thiết kế thống nhất

HQ Campus dùng **CORE1 và CORE2** làm hai gateway dự phòng, ACCESS1/ACCESS2 nối hai uplink tới hai Core. RSTP sẽ xử lý vòng lặp Layer 2 trong Sprint 1; HSRP giữ gateway logic ổn định cho client. Chưa dùng EtherChannel giữa hai Core độc lập và một Access nếu image không hỗ trợ cơ chế multi-chassis.

Địa chỉ HQ dùng các VLAN 10/20/30/40/50/60. Gateway HSRP mỗi mạng dùng .1, CORE1 dùng .2, CORE2 dùng .3. Native VLAN 99 không cấp IP. DMZ đặt phía firewall, không đặt trên Core. Các mạng WAN/ISP giả lập có subnet transit riêng; địa chỉ đã được chốt trong **[IP Addressing Plan Final v1.0](../ip-addressing-plan.md)**.

**Không đưa VPCS vào danh sách thiết bị hạ tầng cố định.** Các máy tính trong lab được sử dụng để kiểm thử kết nối, VLAN và gateway rồi tháo hoặc thay đổi theo từng bài.

## 4. Tình trạng từng Story

| Story | Kết quả | Việc còn lại |
|---|---|---|
| NET-001 | EVE-NG, router, switch hoạt động; kiểm tra tài nguyên và ping | Lưu ảnh ASAv/Alpine boot vào repo |
| NET-002 | Topology HQ và HSRP đã kiểm tra | Vẽ sơ đồ cổng đầy đủ cho WAN/DMZ/Branches khi tạo node |
| NET-003 | Chốt IP Plan Final v1.0 | Áp cấu hình và kiểm thử theo Sprint |
| NET-004 | Repo GitHub đã có README, docs và quy trình review bằng PR | Review và merge bộ tài liệu Sprint 0 |

## 5. Vấn đề và cách xử lý

- **Giới hạn RAM 16 GB trên host:** chia bài lab theo Sprint, chỉ bật node cần dùng, theo dõi RAM sau khi tăng số node.
- **IOL L2 đóng vai trò Core:** hiện kiểm tra được SVI và HSRP; không kết luận khả năng chuyển tiếp đa VLAN khi chưa chạy test Inter-VLAN.
- **Hai uplink Access:** cần cấu hình RSTP trước khi bật toàn bộ trunk; tránh tạo vòng lặp Layer 2.
- **Một firewall ASAv:** có rủi ro single point of failure ở security edge, không ghi nhận firewall HA trong scope hiện tại.
- **IP WAN mô phỏng:** subnet và vai trò đã cố định; khi dựng thêm node phải tạo L2 transit VLAN 90/91 đúng thiết kế thay vì nối trực tiếp gây sai broadcast domain.
- **Minh chứng:** cần lưu ảnh console và kết quả test vào đúng thư mục ngay sau mỗi bài lab.

## 6. Kế hoạch Sprint 1

Triển khai theo thứ tự NET-101 (VLAN), NET-102 (802.1Q Trunk), NET-103 (Inter-VLAN Routing), NET-104 (Rapid STP). Cần chạy test phân đoạn trước khi bật routing, sau đó kiểm tra traffic liên VLAN và failover uplink. Mỗi Story lưu cấu hình, output test, lỗi nếu có và commit riêng.

## 7. Kết luận

Sprint 0 đã xây dựng được nền tảng EVE-NG và mô hình HQ hai Core có HSRP. Thiết kế địa chỉ v1.0 đã chốt để các Sprint sau triển khai thống nhất, không đổi dải mạng theo từng bài. Các mục còn thiếu chủ yếu là bằng chứng lưu trữ và bước triển khai WAN/DMZ/Branch, không được ghi là đã hoàn tất về mặt kỹ thuật.

**Tài liệu kèm theo:** [Architecture](../network-architecture.md) · [Final IP Plan](../ip-addressing-plan.md) · [Validation Log](../test-results/sprint-00-validation.md).
