# Sprint 04, buổi 8 — VPCS/Campus regression

**Ngày nhận output:** 07/11/2026, Asia/Saigon; output không có timestamp riêng.  
**Nguồn:** CLI EVE host, CORE1/CORE2/ACCESS1 và sáu VPCS do người thực hành gửi; agent không điều khiển EVE.  
**Trạng thái:** **PASS hồi quy Campus và đối chiếu IP/gateway sáu VPCS** sau bổ sung output PC-ACC→PC-IT 10/10 cùng `show ip` của sáu PC. Cấu hình IP/gateway cũng đã có trong ZIP cuối. Người thực hành chưa xác nhận riêng thao tác `save`/Stop/Start đủ sáu PC; startup trên bản restore sẽ được kiểm tiếp, không yêu cầu lặp lại các mẫu ping đã đạt.

## Tài nguyên và switching

- EVE host sau khi bật nhóm Campus: tổng RAM 7,7 GiB, used 2,3 GiB, available 5,1 GiB, swap used 0/4,0 GiB. Đây là số liệu tại mốc đo, không chứng minh mọi node đều đang chạy cùng lúc.
- CORE1 `Po1(SU)` LACP có `Et0/0(P) Et0/3(P)`; `Po11(SU)` LACP có `Et0/1(P) Et1/1(P)`. CORE2 `Po1(SU)` có `Et0/0(P) Et0/3(P)`. ACCESS1 `Po11(SU)` có `Et0/0(P) Et1/0(P)`. Các member khớp baseline Campus trước WAN.
- HSRP VLAN 10/20/30/40/50/60: CORE1 priority110, preempt, Active local; CORE2 priority100, preempt, Standby local. VIP mỗi VLAN là `10.10.X.1`. Các output hai Core đối ứng nhau.

## Ping VPCS actual

- PC-IT-01 → PC-ACC `10.10.40.20`: reply seq1–10, **10/10**. Cùng block output được dán hai lần giống hệt; ghi một mẫu, không cộng thành 20 gói độc lập.
- PC-HR-01 → PC-IT `10.10.20.20`: **10/10**. Đây là kiểm liên VLAN từ HR sang IT, không phải ping VIP HR.
- PC-ACC-01 → VIP VLAN40 `10.10.40.1`: **10/10**.
- PC-GUEST-01 → VIP VLAN60 `10.10.60.1`: **10/10**.
- PC-BR1 → gateway `10.20.10.1`: **10/10**.
- PC-BR2 → gateway `10.30.10.1`: **10/10**.

## Output bổ sung và cấu hình VPCS

PC-ACC-01→PC-IT `10.10.20.20 -c 10` bổ sung đủ seq1–10, **10/10**, RTT 0,901–2,176 ms. Cùng mẫu PC-IT→PC-ACC phía trên, hai chiều hồi quy sau WAN đều đạt.

`show ip` sáu PC đối chiếu được: PC-IT-01 `10.10.20.20/24`, gateway `10.10.20.1`; PC-HR-01 `10.10.30.20/24`, gateway `10.10.30.1`; PC-ACC-01 `10.10.40.20/24`, gateway `10.10.40.1`; PC-GUEST-01 `10.10.60.20/24`, gateway `10.10.60.1`; PC-BR1 `10.20.10.20/24`, gateway `10.20.10.1`; PC-BR2 `10.30.10.20/24`, gateway `10.30.10.1`. Sáu payload VPCS trong [ZIP cuối](sprint-04-final-export-inspection-2026-11-07.md) giống nguyên byte baseline 26/10 và khớp các địa chỉ này. Thao tác save/Stop/Start không có xác nhận riêng; kiểm boot VPCS từ artifact thuộc ca restore bản sao.

**Chốt startup VPCS:** agent đã boot sáu PC từ payload trên bản restore riêng, thu startup banner/show ip và sáu mẫu 10/10 sau warm-up; xem [biên bản restore](sprint-04-final-restore-2026-11-07.md). Phần chưa có xác nhận Stop/Start lab gốc ở checkpoint đầu không còn chặn nghiệm thu TC-11.
