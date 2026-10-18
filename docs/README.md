# Danh mục tài liệu

**Bàn giao Sprint2 cập nhật18/10/2026:** đủ bốn final running và export8 node/11 link đã kiểm tra. Có các kết quả HA chức năng, lỗi chuyển member Po1 vẫn mở; chưa nghiệm thu toàn bộ DoD hoặc restore cuối.

- [Báo cáo bàn giao Sprint2](sprint-reports/sprint-02-report.md): phạm vi kiểm thử theo đường đại diện, kết quả thực hành, cấu hình cuối và phần chưa nghiệm thu.
- [Điều tra Po1/HA ngày18/10](test-results/sprint-02-po1-member-check-2026-10-18.md): lỗi/retest, capture hai đầu và giả thuyết EVE/image chưa xác nhận.
- [Kiểm tra export Sprint2](test-results/sprint-02-export-inspection-2026-10-18.txt): hash/CRC/XML,8 node/11 link, không nhúng cấu hình.

Sprint 0 được đóng ngày 10/10/2026 theo phạm vi nền tảng, với follow-up và giới hạn được ghi rõ. Mốc đóng là commit local chứa bộ hồ sơ; chưa xác nhận push/PR/merge trên GitHub.

Sprint1 chốt phần thực hành có follow-ups ngày11/10/2026; báo cáo/validation và export đã kiểm tra được lưu thành commit local riêng trước Sprint02. Chưa push/PR/merge. Không quy đổi mốc này thành toàn bộ DoD hoặc13 ca PASS.

- [Biên bản đóng Sprint 00](sprint-reports/sprint-00-closeout.md): quyết định nghiệm thu, mức bằng chứng, việc bàn giao và mốc tiếp nhận.
- [Po11 và sự cố lượt A16/10/2026](test-results/sprint-02-po11-progress-2026-10-16.md): [phần12 biên bản](test-results/sprint-02-po11-member-a-issue-2026-10-16.md) có Po11 chỉ member mới SU/P hai đầu, MAC PC qua Po11 và ba mẫu ping5/5; Po1 trunk đúng. Tiếp tục hướng dẫn17/10 mở member cũ để phục hồi hai P, chưa có output thao tác mới. Sự cố chuyển member vẫn OPEN, snapshots142/87 dòng giữ nguyên.
- [Po11 chuẩn bị sau start, 16/10/2026](test-results/sprint-02-po11-preflight-review-2026-10-16.md): lịch sử nối dây, shutdown hai cổng mới và xác nhận lưu CORE1 trước chuyển sang channel.
- [Checkpoint Po1 Sprint2, cập nhật16/10](test-results/sprint-02-po1-progress-2026-10-12.md): hai member/runtime/lưu OK/ping HR–ACC–VIP mỗi mẫu5/5 từ12/10; bốn snapshot hoàn thiện bằng section xác nhận16/10, hai cặp khớp127/116 dòng. S2-FWD-01 phục hồi, S2-IOL-01 giữ nguyên; traffic qua channel/HA chưa nghiệm thu.
- [Kiểm tra baseline HQ Sprint 2, 12/10/2026](test-results/sprint-02-preflight-review-2026-10-12.md): startup thật bốn switch khớp baseline Sprint 01; runtime Core/Access, đủ sáu nhóm HSRP/trunk/STP/routes và tám lượt ping VPCS 5/5; đã có VLAN brief hai Core/chi tiết tám cổng member; IP bốn PC và capability LACP/duplex đã xác nhận; buổi 1 đã đóng theo quyết định; restore baseline trước LACP bỏ qua/chưa chạy, save VPCS chưa có xác nhận, duplex thực tế kiểm tra khi triển khai.
- [Báo cáo Sprint 1](sprint-reports/sprint-01-report.md): kết quả NET-101–104, workaround CEF, bài thử uplink, export và follow-ups có mốc tiếp nhận.
- [Validation Sprint 1](test-results/sprint-01-validation.md): actual theo 13 ca, phân biệt raw output, xác nhận thực hành, phần chưa đối chiếu và ca bỏ qua.
- [Điều tra CEF trên CORE1, Sprint 1](test-results/sprint-01-core1-cef-investigation-2026-10-10.md): output routing/ARP/source ping và xác nhận timeout → thành công → timeout khi bật–tắt–bật CEF; workaround tạm, NET-103 chưa nghiệm thu đầy đủ.
- [Review running-config Sprint 01](test-results/sprint-01-config-review-2026-10-11.md): bốn snapshot cuối đã có VTY/cổng test hoàn thiện, lịch sử trước thay đổi và giới hạn startup.
- [Thử uplink ACCESS1, Sprint 01](test-results/sprint-01-access1-uplink-test-2026-10-11.md): log ping, giới hạn phép đo và quyết định bỏ thử ACCESS2.
- [Kiểm tra export Sprint 01](test-results/sprint-01-export-inspection-2026-10-11.txt): hash/CRC/XML và chín kết nối HQ; ZIP không nhúng config.
- [Báo cáo Sprint 0](sprint-reports/sprint-00-report.md): kết quả/giới hạn và bước bắt đầu Sprint 1.
- [Validation](test-results/sprint-00-validation.md): output người thực hành, review IP, HSRP và kiểm tra export tại local.
- [Roadmap](enterprise-network-design-implementation-roadmap.md): mục tiêu/story từng sprint; checkbox chưa chạy giữ nguyên.
- [Architecture](network-architecture.md), [Logical topology](../diagrams/logical-topology.md), [Physical topology](../diagrams/physical-topology.md): HQ đã đối chiếu, phần Enterprise dự kiến và cổng TBD.
- [IP Plan Final v1.0](ip-addressing-plan.md): phân bổ đã review; không đồng nghĩa routing/WAN đã triển khai.
- [Môi trường/image inventory](00-eve-ng-setup.md), [Config](../configs/README.md), [Export/tái dựng](../eve-exports/README.md), [Ảnh](../screenshots/README.md): artifact và giới hạn tái dựng.
- [Workflow](repository-workflow.md), [Mẫu test case](test-results/test-case-template.md): quy ước lưu/review bằng chứng.

Đọc closeout/report của sprint trước cùng phạm vi và tiêu chí trong roadmap trước khi thực hành. Docs dùng chung không phụ thuộc vào kế hoạch nội bộ bị ignore; thay IP/cổng phải cập nhật tài liệu và test liên quan.
