# Cấu hình export từ EVE — Sprint 3

Tám file `*.exported.cfg` tại đây được giải mã nguyên văn từ tám payload Base64 trong `eve-exports/sprint-03-routing-2026-10-22-v2.zip` (UNL `Enterprise-Network-S03-Routing.unl`, node ID 1–8). Nguồn là thao tác **Export all CFGs** trên các node đang chạy. Đây là snapshot do EVE trích xuất, không phải bản `show startup-config` độc lập của từng router.

Đối chiếu artifact, SHA-256 và giới hạn kiểm thử tại [biên bản export](../../../../docs/test-results/sprint-03-export-inspection-2026-10-22.md). Chưa thử import hoặc nạp lại cấu hình trên lab bản sao. Không dùng các file sandbox này để ghi đè cấu hình Enterprise tích hợp; phải map cổng, IP và vai trò theo INT-03 trước.
