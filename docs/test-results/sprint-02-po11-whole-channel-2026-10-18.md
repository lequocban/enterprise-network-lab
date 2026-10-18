# Sprint 02 — mất toàn Po11, 18/10/2026

**Phạm vi:** S02-TC-08, kiểm đường dự phòng ACCESS1→CORE2 khi Po11 phía CORE1 shutdown. Ghi mốc quan trọng từ tin nhắn, không tạo thêm transcript/full config. Đã xác nhận chức năng tự phục hồi qua đường dự phòng theo STP, ping và lời người thực hành; phần đo thời gian chính xác/HSRP trong lỗi và output phục hồi hai P chưa đầy đủ, không nâng thành PASS toàn bộ HA.

CORE1 thực thi interface Port-channel11/shutdown. Log admin down cả Po11/e0/1/e1/1 lúc08:30:18.922UTC18/10; line down08:30:19.927UTC. Không có command/log no shutdown phíaCORE1 trong nguồn mới.

ACCESS1 VLAN20 vẫn nhận rootCORE1 aabb.cc00.1000/priority24596 nhưng rootport chuyển sangEthernet0/1, rootpathcost156. e0/1RootFWD/cost100, Po11DesgFWD/cost56 vàe0/2DesgFWD100 trong mẫu. Trunk e0/1/Po11 native99/allowed-active-forwarding đủbảyVLAN. Rootpath156 phù hợp100 tớiCORE2 và56 trên đườngCORE2→CORE1; chứng minh STP đã chọn hướng dự phòng, không tự khẳng định mọi traffic đều đã đi đúng đường trong suốt lượt.

LogA1Po11down08:31:35.046/line down08:31:36.046 rồiup08:31:38.785/line up08:31:39.790. Đây là trạng thái cục bộ A1; chưa xác minh tại sao up lại hoặc có liên quan tới thao tác mởCORE1 hay không. Không gán chắc do timeout LACP hoặc đồng bộ thời gian các node từ log này.

PC-IT→ACC40.20 dùng -t/-i1000/-w1000: seq1–22 có22reply, seq23–39 có17timeout, seq40–68 có29reply. Tổng68kếtquả/51reply/17timeout; TTL63 ởreply. Không có timestamp từnggói hoặc mốc thao tác chung, không đổi17timeout thành17giây hay suy thời gian hội tụ chính xác.

**Kết luận hiện tại:** người thực hành đã xác nhận chỉ mở lại CORE1 Po11 sau khi ping trả lời ổn định. Vì vậy traffic đã tự hồi phục khi đường chính vẫn shutdown, kết hợp ACCESS1 e0/1RootFWD chứng minh phần chức năng chuyển đường dự phòng. Không cần hỏi hoặc lặp lại để xác minh thứ tự này. Chưa có HSRP mới để xác nhận C1Active trong pha lỗi; không quy 17timeout thành 17giây hoặc hội tụ0ms.

**Tiếp theo:** người thực hành đã mở lại Po11 và yêu cầu sang bước tiếp; kiểm baseline haiP trước bài mới, không ép lặp toàn bộ phép thử Po11. Tiếp tục S02-TC-06 mất từng member Po1, dùng traffic tới realSVI CORE2 10.10.10.3 và đối chiếu CORE2STPVLAN10 RootPortPo1. Mất toànPo1 và fullCore failover vẫn là các bài riêng; không wr pha lỗi hoặc ghi Sprint2 hoàn tất từ kết quả này.
