# Sprint 01 — Điều tra forwarding liên VLAN và CEF trên CORE1

**Story:** NET-103 / S01-TC-07, phần CORE1.  
**Ngày tiếp nhận:** 10/10/2026, Asia/Saigon; chưa có timestamp thiết bị của phép thử.  
**Nguồn:** output console và xác nhận trực tiếp của người thực hành trong chat. Agent không chạy thiết bị.  
**Trạng thái:** tái hiện triệu chứng theo xác nhận người thực hành; workaround đã cho kết quả thành công theo xác nhận. Chưa nghiệm thu đầy đủ S01-TC-07/NET-103.

## Điều kiện và bằng chứng đã nhận

- PC-IT `10.10.20.20/24`, gateway `10.10.20.2`, MAC `00:50:79:66:68:05`.
- PC-HR `10.10.30.20/24`, gateway `10.10.30.2`, MAC `00:50:79:66:68:06`.
- CORE1 Vlan20 `10.10.20.2/24`, Vlan30 `10.10.30.2/24`, cả hai up/up theo output. Có connected/local routes đúng hai subnet/SVI. Không có default route; không cần default/static/OSPF cho hai subnet trực tiếp này.
- Cấu hình SVI người thực hành gửi gồm description và IP; không có ACL gắn trực tiếp trong các đoạn đó. Chưa kết luận mọi cơ chế lọc toàn thiết bị đều đã được kiểm tra.
- Image tham chiếu từ Sprint 0: IOL L2 `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`. Chưa có output version Sprint 01 mới để xác nhận image/runtime không thay đổi.

Trích nguyên văn các dòng kết quả console đã được cung cấp:

```text
CORE1#ping 10.10.30.20 source 10.10.20.2
Packet sent with a source address of 10.10.20.2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
CORE1#ping 10.10.20.20 source 10.10.30.2
Packet sent with a source address of 10.10.30.2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

ARP CORE1 được quan sát có `10.10.20.20 → 0050.7966.6805 → Vlan20` và `10.10.30.20 → 0050.7966.6806 → Vlan30`. Hai phép ping khác source kiểm tra đường trả về của client, không thay thế phép thử traffic client-to-client qua Core.

## Phép thử CEF bật–tắt–bật

Hướng dẫn đã đưa ra: ping lại client-to-client sau warm-up, thu CEF/adjacency, dùng `no ip cef` tạm trên CORE1, retest, bật lại bằng `ip cef`, retest.

Người thực hành xác nhận kết quả: **“timeout → thành công → timeout”**.

- CEF bật ban đầu: client-to-client timeout.
- CEF tắt: client-to-client thành công theo xác nhận.
- CEF bật lại: timeout tái hiện theo xác nhận.

Chưa nhận output ping/CEF/adjacency nguyên bản của cả ba pha; chưa biết số gói, chiều nào đã chạy đủ, RTT, interval và thời gian mỗi pha. Không suy diễn thành 10/10 hai chiều hoặc xác nhận bug ID cụ thể. Trạng thái CEF hiện tại trên thiết bị cần đọc thực tế; nếu thao tác đúng thứ tự hướng dẫn thì pha cuối là bật lại.

## Kết luận và bước xử lý

Kết quả đối chiếu liên hệ lỗi transit forwarding với đường CEF bật trong môi trường hiện tại. Connected routes, ARP và ping do Core tạo ra có thể đúng trong khi traffic client-to-client vẫn thất bại. Chưa xác định nguyên nhân sâu là lỗi build, trạng thái FIB/adjacency hay yếu tố khác của implementation.

Đề xuất cho bài học tiếp theo:

1. Tạm dùng `no ip cef` trên CORE1, xác nhận CEF không chạy và retest client-to-client hai chiều, ghi actual. Chưa lưu startup trong lúc chẩn đoán.
2. Kiểm chứng CORE2 độc lập qua gateway `.3`, trước tiên với CEF bật. Nếu gặp cùng triệu chứng, thu dữ liệu rồi đối chiếu bật–tắt–bật riêng; không mặc định hai Core đều có lỗi giống nhau.
3. Sau khi hai Core forward được, chốt cấu hình lab có ghi rõ ngoại lệ CEF hoặc đổi image hỗ trợ L3/CEF phù hợp, rồi mới mở rộng HSRP/VLAN. Thay image phải cập nhật inventory và kiểm thử phần chịu ảnh hưởng.
4. Không nghiệm thu hiệu năng hoặc tính năng phụ thuộc CEF từ workaround. Không đánh dấu toàn NET-103 Done từ kết quả của riêng CORE1.

## Quyết định giữ workaround để hoàn thành Sprint 1

**Người thực hành chốt ngày tiếp nhận 10/10/2026:** giữ workaround để hoàn thành Sprint 1. Trước đó người thực hành cho biết CORE2 có tình trạng tương tự CORE1; chưa cung cấp output riêng/số gói hoặc xác nhận chi tiết từng pha của CORE2.

- Giữ image hiện có và áp dụng `no ip cef` trên **cả CORE1 và CORE2** trong Sprint 1. Đây là quyết định cấu hình lab có ngoại lệ, không phải kết luận lỗi chung của mọi image IOL hoặc bug ID cụ thể.
- Xác minh CEF không chạy, ping VLAN20↔30 qua từng Core, rồi cấu hình HSRP20/30; trả gateway client về VIP `.1` theo IP Plan.
- Sau khi role/gateway/traffic đạt, lưu startup trên thiết bị và thu snapshot Sprint 01 thật. Quyết định đã được chốt, nhưng agent chưa áp dụng cấu hình và chưa nhận snapshot startup có workaround; không coi đây là bằng chứng đã lưu thiết bị.
- Tiếp tục VLAN40/50/60, management return path, failure test từng uplink và baseline/report Sprint 01. Chưa đánh dấu NET-103 hoặc toàn sprint Done.
- Ghi giới hạn hiệu năng/CEF; xem xét image thay thế trước khi mở rộng WAN Sprint 3. Không tự áp dụng `no ip cef` lên Edge/Branch/ASAv.

## Phần cần bổ sung khi nghiệm thu

**Cập nhật chốt thực hành 11/10/2026:** [running cuối CORE1](../../configs/core/sprint-01/CORE1.cfg) và [CORE2](../../configs/core/sprint-01/CORE2.cfg) đều có `no ip cef`; người thực hành xác nhận hoạt động ổn và chọn hoàn thiện [báo cáo Sprint 01](../sprint-reports/sprint-01-report.md) với ngoại lệ này. Các đề xuất chẩn đoán ở trên là lịch sử; không yêu cầu lặp A/B/A để hoàn thiện báo cáo. Checkbox dưới giữ phần bằng chứng còn thiếu, không tự đổi thành PASS từ snapshot running.

- [ ] Output version và config/runtime trước/sau thật của CORE1.
- [ ] CEF/adjacency trước thay đổi; output xác nhận trạng thái khi bật/tắt.
- [ ] Ping từng pha, chiều, số gói gửi/nhận/mất và thời điểm thực tế.
- [x] Người thực hành quyết định giữ `no ip cef` trên hai Core để hoàn thành Sprint 1.
- [ ] Có snapshot startup/runtime thật tương ứng với quyết định và giới hạn đã ghi.
- [ ] Kết quả CORE2 độc lập, client qua VIP .1 và các acceptance NET-103 còn lại.

Nguồn nguyên tắc kiểm tra, không phải xác nhận lỗi của IOL build: [Cisco — Verify CEF Switching](https://www.cisco.com/c/en/us/support/docs/ios-nx-os-software/ios-software-releases-120-mainline/47205-cef-whichpath.html), [Cisco — Enable/Disable CEF](https://www.cisco.com/c/en/us/td/docs/ios/ipswitch/configuration/guide/convert/ips_cef/cef_enable_disable.html). Kết quả liên quan: [Báo cáo Sprint 1](../sprint-reports/sprint-01-report.md).
