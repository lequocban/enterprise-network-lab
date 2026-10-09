# Enterprise Network Design & Implementation

Lab mô phỏng mạng doanh nghiệp trên **EVE-NG Community 6.2** để thực hành switching, routing, high availability, firewall, VPN và vận hành hạ tầng. Công việc chia theo Sprint, mỗi tính năng được ghi lại cấu hình và kết quả kiểm thử.

## Phạm vi dự án

- **HQ:** 2 Core (CORE1/CORE2), 2 Access (ACCESS1/ACCESS2), mạng IT, HR, Accounting, Server, Guest, Management.
- **WAN:** 2 Edge router, 2 ISP giả lập, 2 chi nhánh.
- **Security:** Cisco ASAv, DMZ, ACL/NAT, VPN Site-to-Site.
- **Service/Operations:** Alpine Linux, DHCP, DNS, NTP, Syslog, monitoring và automation.

Các hạng mục WAN, DMZ, Branch, bảo mật và vận hành sẽ được bổ sung theo Sprint; chưa phải toàn bộ đều đã chạy.

## Môi trường

Lenovo Legion 5 (Ryzen 7 7840H, RAM 16 GB), VMware Workstation, EVE-NG VM (4 vCPU, 8 GB RAM, 120 GB disk). IOSv cho router; Cisco IOL L2 cho switching/Core test; ASAv cho firewall; Alpine thay Ubuntu để giảm RAM.

## Sprint 0 — Kết quả hiện tại

Đã kiểm tra kết nối router Cisco IOSv, dựng topology HQ hai Core và hai Access, xác nhận SVI VLAN10 và HSRP Active/Standby với VIP `10.10.10.1`. Địa chỉ IP cho toàn bộ dự án đã được thống nhất trong **IP Addressing Plan Final v1.0**.

**Phân biệt rõ:** hiện chỉ xác nhận kỹ thuật trên VLAN10; VLAN 20–60, Inter-VLAN, RSTP, OSPF, BGP, Firewall policy, VPN và monitoring triển khai từ các Sprint tiếp theo.

VPCS là máy test, không tính vào danh sách thiết bị hạ tầng. PC-MGMT từng được dùng để test SVI nhưng không đưa vào topology chuẩn.

## Tài liệu

- [Sprint 0 Report](docs/sprint-reports/sprint-00-report.md)
- [IP Addressing Plan Final v1.0](docs/ip-addressing-plan.md)
- [Network Architecture](docs/network-architecture.md)
- [Sprint 0 Validation](docs/test-results/sprint-00-validation.md)
- [Project Roadmap](docs/enterprise-network-design-implementation-roadmap.md)
- [Documentation Index](docs/README.md)

## Tiến độ

| Sprint | Nội dung | Trạng thái |
|---:|---|---|
| 0 | Môi trường, kiến trúc, IP Plan, Git | Báo cáo chờ review |
| 1 | VLAN, 802.1Q Trunk, SVI, RSTP | Tiếp theo |
| 2 | LACP, HSRP Failover | Chưa triển khai đầy đủ |
| 3–6 | OSPF, BGP, Firewall, VPN | Kế hoạch |
| 7–9 | Linux Services, Monitoring, Automation | Kế hoạch |
| 10–11 | Troubleshooting, Portfolio | Kế hoạch |

Đây là dự án lab phục vụ học tập và portfolio, không phải hạ tầng production.
