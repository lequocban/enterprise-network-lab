# Sprint 3 — Checkpoint Single-Area OSPF

**Ngày tiếp nhận:** 20/10/2026, Asia/Saigon.  
**Nguồn:** output console bốn router và ba VPCS do người thực hành cung cấp trong chat; agent không trực tiếp điều khiển EVE.  
**Topology:** [thiết kế sandbox](../routing/sprint-03-routing-design.md), 4 IOSv15.6(2)T fc2, 3 VPCS, 6 link theo ảnh đã lưu. Tên/path UNL chưa đối chiếu export; không phải topology Enterprise đã tích hợp HQ Sprint1/2.

## Control plane và RIB

- EDGE1 có ba neighbor FULL/-: router-id `10.255.255.12` qua `10.255.0.38` Gi0/0, `.21` qua `.42` Gi0/1, `.22` qua `.46` Gi0/2.
- EDGE2 có một neighbor FULL/- với EDGE1 `.11`, peer `.37` Gi0/0. BR1 cũng FULL/- với EDGE1 `.11`, peer `.41` Gi0/0; BR2 FULL/- với EDGE1 `.11`, peer `.45` Gi0/0.
- EDGE1 học `10.20.10.0/24` qua `.42`, `10.30.10.0/24` qua `.46`, `10.255.0.52/30` qua `.38`, cả ba `[110/2]`; ba loopback remote cũng có route O `[110/2]` đúng next-hop.
- EDGE2 học hai LAN Branch `[110/3]` qua `.37`; BR1 học LANBR2 và mạng PC-HQ `[110/3]` qua `.41`; BR2 học LANBR1 và PC-HQ `[110/3]` qua `.45`. Các router leaf có transit/loopback remote phù hợp trong output OSPF.
- Không có default route theo các output nhận. Dữ liệu neighbor/RIB khớp pha single-area được hướng dẫn; chưa có bộ `show ip ospf interface`/`show ip protocols`/LSDB để đối chiếu riêng process/area/network type/passive của mọi interface.

## Data plane — actual 5 gói mỗi chiều

Output showip xác nhận PC-HQ `10.255.0.54/30`, GW`.53`; PC-BR2 `10.30.10.20/24`, GW`10.30.10.1`. PC-BR1 `10.20.10.20/24`, GW`10.20.10.1` đã có showip ở checkpoint trước.

- PC-HQ → PC-BR1:5/5 reply.
- PC-HQ → PC-BR2:5/5 reply.
- PC-BR1 → PC-HQ:5/5 reply.
- PC-BR1 → PC-BR2:5/5 reply.
- PC-BR2 → PC-HQ:5/5 reply.
- PC-BR2 → PC-BR1:5/5 reply.

Cả sáu chiều có TTL61 trong reply. Ghi đúng actual5/5, không đổi thành10/10 như mẫu dự kiến của S03-TC-03; không yêu cầu lặp chỉ để tăng số gói khi chưa có nghi vấn chức năng mới. Đây là bằng chứng kết nối ngắn tại mốc đo, không phải kiểm thử tải/độ ổn định dài hạn.

## Trace đại diện

Người thực hành chạy từ PC-BR1 tới PC-BR2, khác nguồn trace dự kiến PC-HQ nhưng vẫn là đường đại diện qua router trung gian:

```text
PC-BR1> trace 10.30.10.20
1  10.20.10.1
2  10.255.0.41
3  10.255.0.46
4  *10.30.10.20 (ICMP type:3, code:3, Destination port unreachable)
```

Đường đi PC-BR1 → BR1 → EDGE1 → BR2 → PC-BR2 đúng thiết kế. Với UDP traceroute của VPCS, ICMP3/3 từ chính địa chỉ đích là tín hiệu đã đến đích; không phân loại thành lỗi routing chỉ từ dòng port unreachable. Không tự tạo trace PC-HQ chưa chạy.

## Kết luận và bước tiếp theo

Đã có bằng chứng neighbor/RIB và ba cặp client hai chiều hoạt động. S03-TC-02 có neighbor/RIB; phần xác minh passive/network type chi tiết tiếp nhận buổi3. S03-TC-03 có bằng chứng chức năng5/5 mỗi chiều, chưa thực hiện mẫu10gói dự kiến. Không ghi toàn bộ DoD NET-301 hoặc Sprint3 DONE.

Wr/save của phần IP đã có xác nhận trước đó; chưa có xác nhận mới sau thêm OSPF trong lượt output này. Trước thử gây lỗi, lưu baseline OSPF bằng wr nếu chưa lưu sau bước OSPF, chờ `[OK]`; không yêu cầu lặp khi người thực hành đã xác nhận hoàn tất bước này.

Tiếp theo buổi3: kiểm interface/passive, test passive transit BR2 có kiểm soát và hoàn nguyên, sau đó thử metric/cost. Không lưu pha lỗi vào startup. Full running/startup snapshot, export Sprint3 và restart/restore chưa có trong checkpoint này; các mốc tích hợp INT-03/04/05/06 vẫn chưa nghiệm thu.

## Buổi 3 — passive transit và khôi phục

**Actual tiếp nhận20/10/2026:** theo bài đặt passive EDGE1 Gi0/2 rồi gỡ để khôi phục; người thực hành cung cấp output trạng thái lúc lỗi và sau hoàn nguyên, chưa có raw lệnh config/timestamp sự kiện riêng.

- Lúc lỗi, EDGE1 chỉ còn FULL với BR1/EDGE2; lookup `10.30.10.0` trả `% Subnet not in table`.
- BR2 không còn neighbor; Gi0/0 `.46`, Gi0/1 `10.30.10.1`, Lo0 `.22` vẫn up/up. Kết quả phân biệt rõ lỗi adjacency với link Ethernet down.
- PC-HQ → PC-BR2 nhận5ICMP type3/code1 từ gateway `10.255.0.53`, không có echo reply; PC-HQ → PC-BR1 vẫn5/5. Đây là fail đúng expected của phép thử, không phải hệ thống đã phục hồi lúc lỗi còn tồn tại.
- Sau hoàn nguyên, EDGE1 có đủ3FULL, route `10.30.10.0/24` trở lại qua `.46`, OSPF10/intra-area, AD110/metric2; PC-HQ → PC-BR2 phục hồi5/5.

**Kết luận:** phần passive transit/failure/recovery của S03-TC-04 ĐẠT theo expected; chưa hoàn thành phần cost/network-type verification của ca. Không đo thời gian hội tụ từ DeadTime/route age hoặc hai lượt đọc console. Chưa có xác nhận wr riêng sau OSPF; giữ giới hạn persistence đã nêu.

**Bước tiếp:** đọc cost/type EDGE1 Gi0/2, tạm đặt cost10 và so sánh route LANBR2 ở EDGE1/EDGE2 cùng chiều ngược ở BR2, kiểm một luồng client; gỡ override cost và đối chiếu baseline. Topology chỉ có một đường, không dùng kết quả này chứng minh HA/best-path qua hai đường.

## Buổi 3 — cost và baseline sau hoàn nguyên

**Actual tiếp nhận20/10/2026:**

- Khi tăng cost EDGE1 Gi0/2 lên10 theo bài thực hành, route LANBR2 trên EDGE1 metric11 qua `.46`, EDGE2 metric12 qua `.37`; cả hai intra-area/process10. PC-HQ → PC-BR2 vẫn5/5.
- Sau người thực hành báo gỡ override, đã nhận `show ip ospf interface Gi0/2` EDGE1: IP`.45/30`, Area0/process10/router-id`.11`, network type POINT_TO_POINT, cost1, Hello10/Dead40, adjacent với BR2`.22`.
- Lookup LANBR2 sau hoàn nguyên: EDGE1 intra-area metric2 qua `.46`, EDGE2 intra-area metric3 qua `.37`, khớp baseline.
- Người thực hành xác nhận đã wr trên hai Edge. Chưa có xác nhận tương ứng cho phần OSPF trên Branch hoặc full running/startup dump; không tạo thêm evidence save.

**Kết luận:** tăng metric, traffic trong pha tăng cost và hoàn nguyên metric/type có bằng chứng đạt. Chiều ngược BR2 → PC-HQ khi cost10 chưa có output riêng; không ghi đã chứng minh hướng đó hoặc yêu cầu lặp chỉ để bổ sung. Broadcast/DR-BDR là bài mở rộng chưa chạy, không cần thay đổi topology để ép thực hiện trước multi-area. NET-301 chưa được đánh dấu fullDoD do snapshot/LSDB/persistence còn giới hạn.

**Bước tiếp:** buổi4 chuyển riêng BR1 sang Area10 ở cả hai đầu transit, LAN/Lo0 BR1; giữ EDGE1–EDGE2/BR2 và Lo0 EDGE1 ở Area0. Kiểm FULL, area thực và route O IA trên router ngoài Area10 trước chuyển BR2. Nếu BR1 chưa lưu phần OSPF single-area, lưu baseline bằng wr trước thay đổi BR1; hai Edge vừa xác nhận wr thì không yêu cầu lưu baseline lại.
