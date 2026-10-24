# Sprint 03 — Routing/OSPF và bàn giao Enterprise

**Cập nhật:** 23/10/2026, Asia/Saigon.  
**Trạng thái:** thực hành routing sandbox hoàn thành về chức năng. Báo cáo này chốt kết quả Sprint 3 trong sandbox; phần thiết kế tích hợp và các bước Sprint 4 được quản lý riêng.

## Kết quả đã chứng minh

- **NET-301:** lab `Enterprise-Network-S03-Routing` có adjacency OSPF FULL, route `O` và traffic ba cặp PC hai chiều; passive transit làm mất neighbor/route rồi phục hồi, cost 1→10→1 có metric tương ứng. Xem [single-area](../test-results/sprint-03-single-area-2026-10-20.md).
- **NET-302:** EDGE1 Area0/10/20, BR1 Area10, BR2 Area20; LSDB Type1/Type3, route `O IA`, ba cặp PC hai chiều 5/5. Area mismatch BR2 làm adjacency/default mất rồi khôi phục. Xem [multi-area](../test-results/sprint-03-multi-area-2026-10-21.md).
- **NET-303:** hai /24 mỗi Branch được tổng hợp thành /23 ở ABR; RIB ngoài area giảm bốn route LAN/Lo110 xuống hai summary, traffic đến các đích thật vẫn thông. Mất Lo110 BR1 vẫn giữ summary vì LAN contributor còn; discard Null0 trên ABR và khôi phục đã quan sát. Xem [summary](../test-results/sprint-03-summary-2026-10-21.md).
- **NET-304:** EDGE2 có static default đến ISP1 và phát Type5 default; hai Branch học `O*E2`, PC Branch đến target mô phỏng. Gỡ static làm default/traffic ngoài mất nhưng Branch↔Branch còn; thêm lại phục hồi. Xem [default](../test-results/sprint-03-default-route-2026-10-21.md).
- [ZIP v2](../../eve-exports/sprint-03-routing-2026-10-22-v2.zip) đạt CRC/XML, đúng node/link, có tám payload cấu hình giải mã được. BR2 có kiểm tra persistence sau stop/start; các node khác chưa có output restart riêng. [Pilot SVTI/IPsec/IKEv2/OSPF](../test-results/sprint-03-vti-pilot-2026-10-22.md) chứng minh overlay trên IOSv và withdrawal/recovery OSPF, tách khỏi sandbox chính.

## Ranh giới nghiệm thu và bàn giao

Kết quả trên là **sandbox**, không phải client HQ↔Branch trên topology Enterprise. Một số ca trong kế hoạch đặt 10/10 hoặc yêu cầu full snapshot/Type3 chi tiết chưa có output đủ; các checkpoint thực tế nhiều ca là 5/5, không nâng thành 10/10. Pilot chưa có MTU/restore riêng. Lỗi một đầu Po1 và ngoại lệ `no ip cef` thuộc Campus vẫn mở.

**Ngoại lệ DoD:** một số ca kế hoạch ghi 10/10 nhưng raw actual chỉ 5/5; chưa có full running/startup độc lập của mọi node. Các ngoại lệ này không thay đổi kết quả chức năng đã thấy, nhưng chưa được đánh dấu ca tương ứng PASS theo ngưỡng kế hoạch và chưa gọi toàn bộ Definition of Done hoàn tất.
