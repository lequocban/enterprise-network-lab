# Sprint 04, buổi 7 — Stop/Start INET-TEST và rút route

**Ngày nhận output:** 26/10/2026, Asia/Saigon; không có timestamp tuyệt đối gắn với từng lệnh.  
**Nguồn:** output ISP1/ISP2/EDGE1/EDGE2 do người thực hành gửi theo mốc INET-TEST Stop và Start; agent không điều khiển EVE.  
**Ca:** INT-04-TC-06.  
**Trạng thái:** **PASS chức năng route withdrawal và recovery**. Khi INET Stop, default/target biến mất khỏi BGP/RIB của cả hai Edge; sau Start, hai ISP nhận lại 2 prefix, hai Edge chọn ISP1 LP200 với ISP2 LP100 backup và ping Lo100→INET 10/10. Không suy timing/loss trong chuyển đổi.

## INET-TEST Stop — route withdrawal PASS

- ISP1 eBGP INET `.2` ở Active, ISP2 eBGP INET `.6` ở Idle; các phiên mỗi ISP với Edge và ISP còn lại vẫn nhận prefix.
- EDGE1 giữ iBGP EDGE2 `.11` và eBGP ISP1 `.18`; `show ip bgp 0.0.0.0`, `show ip bgp 192.0.2.10`, `show ip route 0.0.0.0`, `show ip route 192.0.2.10` đều trả `% Network not in table`.
- EDGE2 giữ iBGP EDGE1 `.10` và eBGP ISP2 `.22`; cùng bốn lệnh BGP/RIB đều trả `% Network not in table`.
- Đích INET dừng nên ca này được chấm bằng route withdrawal ở cả hai Edge. Không tạo default tĩnh thay thế.

## Sau INET-TEST Start — provider và EDGE1 phục hồi

- ISP1 peer INET `.2` nhận 2 prefix; ISP2 peer INET `.6` nhận 2; các phiên Edge/peer ISP vẫn nhận prefix, InQ/OutQ0.
- EDGE1 default và target `192.0.2.10/32` trở lại best qua ISP1 `.18`, AS_PATH `65101 65200`, LP200. RIB target BGP external distance20 via `.18`. EDGE1 source Lo100 `198.51.100.11` ping target **10/10**, min/avg/max **1/2/4 ms**.

## EDGE2 sau Start — snapshot cũ và đối chiếu mới

- Trong block được ghi là sau recovery, `show ip bgp summary` của EDGE2 vẫn là table version20, 7 network/10 path, peer `.10` và `.22` mỗi bên nhận 4 prefix, cùng các bộ đếm/uptime như block INET Stop. Bốn lệnh BGP/RIB target/default vẫn báo `% Network not in table`.
- Sau các dòng đó, EDGE2 source Lo100 `198.51.100.12` ping target **10/10**, 2/2/3 ms. Người thực hành xác nhận đã copy nhầm block output cũ ở phần BGP/RIB và gửi một bộ lệnh mới chạy liên tiếp để đối chiếu.
- Snapshot mới bắt đầu `show clock` lúc **12:54:30 UTC, 26/10/2026** (đồng hồ router; không đồng nhất với múi giờ hồ sơ Asia/Saigon). EDGE2 iBGP EDGE1 `.10` và eBGP ISP2 `.22` đều nhận **6 prefix**, InQ/OutQ0, table version24.
- Default và target đều có hai path: iBGP qua EDGE1 `.10`, AS_PATH `65101 65200`, **LP200 best**; eBGP ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best. RIB default và target đều là BGP internal distance200 via `.10`.
- EDGE2 source Lo100 `.12` ping target **10/10**, min/avg/max **2/3/4 ms**. Control plane, RIB và traffic cùng khớp ở snapshot này; recovery EDGE2 được chốt PASS.

Không có thay đổi cấu hình được báo trong ca Stop/Start node; không cần `wr`. Baseline hai Edge đã được xác nhận để sang thử Stop từng Edge.
