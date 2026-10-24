# Sprint 3 — OSPF summarization tại ABR

**Ngày tiếp nhận:** 21/10/2026, Asia/Saigon.  
**Nguồn:** output người thực hành gửi trong chat; agent không điều khiển console.  
**Phạm vi:** NET-303 trong [routing sandbox](../routing/sprint-03-routing-design.md), kế thừa [multi-area0/10/20](sprint-03-multi-area-2026-10-21.md). Chưa nghiệm thu tích hợp Enterprise hoặc fullDoD.

## Trước area range — actual

- EDGE2 có4prefix OIA riêng: `10.20.10.0/24`, `10.20.11.0/24`, `10.30.10.0/24`, `10.30.11.0/24`, đềuAD110/metric3 qua`.37` Gi0/0.
- PC-HQ ping Loopback110 BR1 `10.20.11.1` và BR2 `10.30.11.1` mỗi đích5/5. RIB chứng minh quảng bá loopback đúng/24, không tự tạo output interface Loopback110 chưa nhận.

## Sau area range — actual

Theo thao tác được hướng dẫn, tại EDGE1 process10 gộpArea10 thành `10.20.10.0/23`, Area20 thành `10.30.10.0/23`; raw lệnh config chưa được gửi trong checkpoint này.

- EDGE2 RIB có hai OIA `/23`, AD110/metric3 qua`.37`; bốn prefix/24 không còn trong output OSPF nhận. Các transit `.40/30`, `.44/30` và Lo0 vẫn tồn tại.
- EDGE2 lookup `10.20.10.0` xác nhận matched`10.20.10.0/23`, inter-area, metric3, nguồnABR`.11`.
- BR1 lookupLANBR2 matched`10.30.10.0/23`, inter-area/metric3 qua`.41`; BR2 lookupLANBR1 matched`10.20.10.0/23`, inter-area/metric3 qua`.45`. Bằng chứng ở cả backbone và area đối diện.
- PC-HQ → `10.20.10.20`, `10.20.11.1`, `10.30.10.20`, `10.30.11.1`: cả4đích5/5reply. Không ghi10/10 hoặc khẳng định toàn bộ địa chỉ trong/23 có host.
- Người thực hành cung cấp lookupRIB thay lệnh `show ip ospf database summary 10.20.10.0` được yêu cầu; chưa có LSDBsummary chi tiết để đối chiếu riêng mask/advertising router ở checkpoint này.

**Kết luận:** so sánh trước/sau và forwarding tới4đích thật đạt chức năng; số route LAN liên quan giảm4→2. Không diễn giải là mọi route OSPF phải gộp hết. Phần Type3 chi tiết và bài mất contributor chưa có actual; chưa đóng toàn bộ NET-303/DoD.

## Bước tiếp — mất một contributor, chưa chạy

Nếu chưa lưu checkpoint summary/Loopback110 thì wr EDGE1/BR1/BR2 và chờ `[OK]` trước thử; EDGE2 không đổi config ở pha này nên không cần lưu lại chỉ vì bài range.

Shutdown riêng Loopback110 BR1. Expected route chi tiết `10.20.11.0/24` mất tại ABR nhưng summary `10.20.10.0/23` vẫn được quảng bá vì LAN`10.20.10.0/24` còn. EDGE2 còn/23; PC-HQ→PC-BR1 còn thông, tới Lo110BR1 thất bại. Lookup IP Lo110 trênABR có thể khớp summary/discard thay vì /24, không dùng matchedsummary làm bằng chứng contributor còn hoạt động. Đọc RIB/LSDB thực tế rồi hoàn nguyên no shutdown, ping Lo110 phục hồi; không wr pha lỗi. Chưa có actual cho các expected này.

## Mất contributor và khôi phục — actual21/10

- Khi tắt Lo110BR1 theo bài hướng dẫn, EDGE1 vẫn họcLAN `10.20.10.0/24` intra-area/metric2 qua`.42`; lookup`10.20.11.1` khớp `10.20.10.0/23` intra-area/metric2 trỏNull0. Chứng minh route chi tiết đã mất và discard summary vẫn tồn tại trênABR.
- PC-HQ → PC-BR1 vẫn5/5; PC-HQ → Lo110BR1 nhận5ICMP3/1 từ`10.255.0.37`, không cóecho reply. Sau hoàn nguyên, EDGE1 lookup`.11.1` lại khớp`10.20.11.0/24` qua`.42` metric2; PC-HQ → Lo110 phục hồi5/5.
- Chưa có lookup/LSDB EDGE2 trong pha contributor mất; không biến expected EDGE2 hoặcmaskType3 thành actual. Không thử tắt contributor cuối cùng hoặc mất toàn Branch trong bài này.

**Kết luận:** bài mất một contributor/Null0/khôi phục đạt chức năng cùng kết quả before/after range. Chưa đánh dấu fullDoD NET-303 hoặc xác nhận wr sau range/Lo110 nếu chưa có lời xác nhận cụ thể. Người học đã được giải thích khác nhau giữa wildcard chọn interface và area range gộp inter-area prefix; không thêm network statement hoặc đổi cấu hình chỉ từ phần hỏi lý thuyết.

**Bước tiếp:** buổi6 dựng ISP1/S3-L04 và đích ngoài mô phỏng, kiểm static default/return route trước khi phát default vào OSPF. Snapshot/startup/export và Type3summary chi tiết còn theo dõi ở checkpoint cuối, không yêu cầu lặp chỉ để tăng số gói.
