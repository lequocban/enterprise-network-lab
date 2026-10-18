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

## Sprint 1 — Hoàn thành thực hành có follow-ups, 11/10/2026

VLAN/trunk, STP root và sáu SVI/HSRP đã triển khai; bốn running-config cuối được review. Forwarding hoạt động theo xác nhận người thực hành với `no ip cef` trên hai Core. Bài uplink ACCESS1 có log hai khoảng mất 15 gói rồi phục hồi; ACCESS2 bỏ thử theo yêu cầu.

Export Sprint01 đạt CRC/XML, đúng tám node/chín link; ZIP không nhúng config. [Báo cáo](docs/sprint-reports/sprint-01-report.md) và [validation13 ca](docs/test-results/sprint-01-validation.md) giữ mức bằng chứng tại mốc thực hành, các phần startup/restore và convergence chưa đối chiếu đầy đủ. Hồ sơ Sprint01 được lưu bằng commit local riêng trước Sprint02; chưa push/PR/merge. Đây là mốc chốt thực hành, chưa phải toàn bộ DoD hoặc mọi ca PASS.

## Tài liệu

- [Sprint 0 Report](docs/sprint-reports/sprint-00-report.md) · [Biên bản đóng và việc bàn giao](docs/sprint-reports/sprint-00-closeout.md)
- [Sprint 1 Report](docs/sprint-reports/sprint-01-report.md) · [Validation Sprint 1](docs/test-results/sprint-01-validation.md)
- [Điều tra CEF Sprint 1](docs/test-results/sprint-01-core1-cef-investigation-2026-10-10.md) · [Review cấu hình](docs/test-results/sprint-01-config-review-2026-10-11.md)
- [Validation](docs/test-results/sprint-00-validation.md) · [Export và tái dựng](eve-exports/README.md) · [Cấu hình](configs/README.md)
- [Logical topology](diagrams/logical-topology.md) · [Physical topology](diagrams/physical-topology.md)
- [IP Plan Final v1.0](docs/ip-addressing-plan.md) · [Architecture](docs/network-architecture.md)
- [Roadmap](docs/enterprise-network-design-implementation-roadmap.md) · [Workflow](docs/repository-workflow.md) · [Danh mục tài liệu](docs/README.md)

## Tiến độ và bước tiếp theo

- Sprint 0: đóng nền tảng, follow-up theo biên bản.
- Sprint 1: hoàn thành thực hành có follow-ups; báo cáo/config/export đã lưu local, không ghi toàn bộ test PASS hoặc DoD DONE.
- Sprint2: [Bàn giao ngày18/10/2026](docs/sprint-reports/sprint-02-report.md): đủ bốn final running, Po1/Po11 hai member ở runtime cuối và export8 node/11 link đã kiểm tra. Các bài mất channel/Core có bằng chứng HA chức năng; [lỗi chuyển member Po1](docs/test-results/sprint-02-po1-member-check-2026-10-18.md) còn mở. ZIP chỉ chứa topology; restore và số đo thời gian HA chưa nghiệm thu đầy đủ. Không đánh dấu toàn bộ Sprint2 DONE.
- Sprint 3–6: WAN, OSPF/BGP, firewall và VPN.
- Sprint 7–9: Linux services, monitoring, automation.
- Sprint 10–11: troubleshooting và portfolio.

Baseline dùng tiếp là cấu hình và export Sprint 01, giữ ngoại lệ CEF và việc bàn giao trong báo cáo. Trước LACP Sprint 2, thêm link cùng cặp endpoint và cập nhật bảng cổng. Kế hoạch nội bộ ở `ai-agent/` được ignore.

Đây là lab học tập/portfolio, chưa phải hạ tầng production.
