# Enterprise Network Design & Implementation

Lab mạng doanh nghiệp trên **EVE-NG Community 6.2**, thực hành switching, routing, high availability, firewall, VPN và vận hành theo sprint.

## Phạm vi

- HQ: hai Core, hai Access; IT, HR, Accounting, Server, Guest và Management.
- WAN: hai Edge, hai ISP mô phỏng, hai chi nhánh.
- Security: ASAv, DMZ, ACL/NAT và Site-to-Site VPN.
- Services/Operations: Alpine, DHCP/DNS/NTP/Syslog, monitoring và automation.

WAN/DMZ/Branch/services là thiết kế dự kiến, chưa phải toàn bộ đã chạy. VPCS là endpoint test linh động.

## Môi trường

Lenovo Legion 5 (Ryzen 7 7840H, RAM 16 GB), VMware Workstation; EVE VM dự kiến 4 vCPU, 8 GB RAM, 120 GB disk. IOSv cho router, IOL L2 cho switch/Core, ASAv cho firewall và Alpine cho server. Output Linux hiện có xác nhận 4 vCPU/RAM 7.7 GiB; chưa có kiểm thử tải toàn Enterprise.

## Sprint 0 — Đã đóng ngày 10/10/2026

**CLOSED WITH FOLLOW-UPS:** nghiệm thu nền tảng với giới hạn và việc bàn giao rõ trong [biên bản đóng sprint](docs/sprint-reports/sprint-00-closeout.md).

- Export HQ sạch: tám node/chín link, CRC/XML/cabling PASS; không còn VPCS dư.
- Bốn switch có console/version; 12 snapshot cấu hình và runtime VLAN10/trunk/STP/routes đã lưu.
- Ping hai Core 5/5 mỗi chiều; HSRP shutdown/no shutdown SVI CORE1 đạt chuyển role và khôi phục.
- Logical/physical và IP Plan v1.0 đã review; repo/workflow/evidence được chốt bằng commit local. Push/PR/merge bộ closeout chưa thực hiện.

ZIP chỉ chứa topology/metadata, phải đi kèm config và VLAN evidence; import/reload/restore chưa thử. IOSv ping lịch sử 4/5, boot ASAv/Alpine theo xác nhận người thực hành; artifact bổ sung có mốc tiếp nhận. CORE2/ACCESS1 còn khác VTY running/startup. Không ghi các giới hạn này thành PASS hoặc nghiệm thu failover client/Inter-VLAN.

## Tài liệu

- [Sprint 0 Report](docs/sprint-reports/sprint-00-report.md) · [Biên bản đóng và việc bàn giao](docs/sprint-reports/sprint-00-closeout.md)
- [Validation](docs/test-results/sprint-00-validation.md) · [Export và tái dựng](eve-exports/README.md) · [Cấu hình](configs/README.md)
- [Logical topology](diagrams/logical-topology.md) · [Physical topology](diagrams/physical-topology.md)
- [IP Plan Final v1.0](docs/ip-addressing-plan.md) · [Architecture](docs/network-architecture.md)
- [Roadmap](docs/enterprise-network-design-implementation-roadmap.md) · [Workflow](docs/repository-workflow.md) · [Danh mục tài liệu](docs/README.md)

## Tiến độ và bước tiếp theo

- Sprint 0: đóng nền tảng, follow-up theo biên bản.
- Sprint 1: VLAN/trunk/SVI, STP và Inter-VLAN; chưa bắt đầu triển khai đầy đủ.
- Sprint 2: LACP và full HSRP/client failover.
- Sprint 3–6: WAN, OSPF/BGP, firewall và VPN.
- Sprint 7–9: Linux services, monitoring, automation.
- Sprint 10–11: troubleshooting và portfolio.

Bắt đầu Sprint 1 bằng kiểm tra baseline, giữ uplink dự phòng shutdown khi dựng VLAN/trunk, kiểm tra STP trước bật đủ link, rồi test Inter-VLAN. Chỉ làm việc trong thư mục dự án; ưu tiên docs/Markdown, kế hoạch nội bộ ở `ai-agent/` được ignore.

Đây là lab học tập/portfolio, chưa phải hạ tầng production.
