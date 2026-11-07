# Sprint 04, buổi 6 — Shutdown/restore T04 trên EDGE2

**Ngày nhận output:** 26/10/2026, Asia/Saigon; không có timestamp tuyệt đối cho lúc shutdown/no shutdown.  
**Nguồn:** output CLI EDGE1/EDGE2 do người thực hành gửi theo hai mốc; agent không điều khiển router.  
**Ca:** INT-04-TC-04, mất riêng T04 và phục hồi. Tách khỏi ca [T03](sprint-04-t03-link-failover-2026-10-26.md) và ca [Stop/Start ISP1](sprint-04-isp1-node-failover-2026-10-26.md).  
**Trạng thái:** **PASS chức năng**: khi T04 down, EDGE2 tiếp tục ra ISP1 qua EDGE1 và hai Lo100 ping INET 10/10; sau phục hồi, eBGP ISP2 và đường backup LP100 trở lại, primary ISP1 LP200 vẫn được chọn. Chưa đo loss/thời gian hội tụ trong lúc chuyển đổi.

## T04 shutdown

- EDGE2 Gi0/1 `10.255.0.21` administratively down/down; Gi0/0 T02, Lo0 và Lo100 vẫn up/up. eBGP ISP2 `.22` **Idle**, iBGP EDGE1 `.10` vẫn Established và nhận 8 prefix.
- EDGE2 default và target `192.0.2.10/32` đều chỉ còn một path best qua iBGP EDGE1 `.10`, AS_PATH `65101 65200`, LP200. Không có output `show ip route 192.0.2.10` riêng ở mốc này; BGP best và traceroute xác nhận hướng chuyển tiếp quan sát được.
- EDGE2 source Lo100 `198.51.100.12` ping target **10/10**, min/avg/max **2/2/3 ms**; traceroute qua EDGE1 `.10` → ISP1 `.18` → INET `.2`.
- EDGE1 target vẫn best qua eBGP ISP1 `.18`, AS_PATH `65101 65200`, LP200; RIB external distance20 via `.18`. Source Lo100 `198.51.100.11` ping target **10/10**, 1/2/3 ms.

## T04 `no shutdown` và phục hồi

- EDGE2 Gi0/1 trở lại up/up; eBGP ISP2 `.22` nhận 6 prefix, iBGP EDGE1 `.10` nhận 6, InQ/OutQ0.
- EDGE2 target có lại hai path: ISP2 `.22`, AS_PATH `65102 65200`, LP100 valid/non-best; iBGP EDGE1 `.10`, AS_PATH `65101 65200`, LP200 best. RIB target BGP internal distance200 via `.10`. Source Lo100 `.12` ping target **10/10**, 2/2/3 ms.

Traceroute tại mốc shutdown có một probe `*` ở hop cuối nhưng hai probe khác tại hop đó trả lời và ping end-to-end 10/10. Không suy loss từ dấu `*`. Không có bằng chứng `wr` khi cổng down; đây là thao tác thử tạm đã hoàn nguyên. Không cần lưu lại nếu startup baseline đã là `no shutdown`. Ca Stop/Start ISP2 là phép thử độc lập tiếp theo, chưa có kết quả trong biên bản này.
