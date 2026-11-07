# Sprint 04, buổi 2 — IP và kết nối trực tiếp provider

**Ngày nhận:** 24/10/2026, Asia/Saigon.  
**Nguồn:** output console ISP1, ISP2, INET-TEST và EVE host do người thực hành gửi; agent đọc kết quả, không điều khiển trực tiếp thiết bị.  
**Story/checkpoint:** NET-401, S4-S02-A.  
**Topology:** lab tích hợp theo [checkpoint cabling](sprint-04-wan-export-inspection-2026-10-24.md).  
**Trạng thái:** **PASS chặng IP/ping trực tiếp**. Chưa có actual BGP, target/default hoặc failover ở mốc này.

## IP/interface và connected route

- ISP1: Gi0/0 `10.255.0.18/30`, Gi0/1 `10.255.0.26/30`, Gi0/2 `10.255.0.33/30`, Gi0/3 `203.0.113.1/30`, Lo0 `10.255.255.101/32`; tất cả up/up. RIB có C `10.255.0.16/30`, `.24/30`, `.32/30`, `203.0.113.0/30`, Lo0 /32 và các route L tương ứng. Gateway of last resort chưa đặt.
- ISP2: Gi0/0 `10.255.0.22/30`, Gi0/1 `10.255.0.30/30`, Gi0/2 `10.255.0.34/30`, Gi0/3 `203.0.113.5/30`, Lo0 `10.255.255.102/32`; tất cả up/up. RIB có C `10.255.0.20/30`, `.28/30`, `.32/30`, `203.0.113.4/30`, Lo0 /32 và các route L tương ứng. Gateway of last resort chưa đặt.
- INET-TEST: Gi0/0 `203.0.113.2/30`, Gi0/1 `203.0.113.6/30`, Lo100 `192.0.2.10/32`, up/up. Gi0/2–0/3 chưa cấp IP, administratively down/down. RIB có hai /30 uplink, Lo100 /32 và các route L; gateway of last resort chưa đặt.

Các mask được đối chiếu từ `show ip route connected`, không suy mask chỉ từ `show ip interface brief`. Cổng ISP tới Edge/Branch cũng có connected route trong EVE; điều này đủ điều kiện exact RIB cho câu lệnh BGP `network` sau này, nhưng chưa chứng minh các peer Edge/Branch đã cấu hình hoặc reachable.

## Sáu mẫu ping thực tế

Mỗi lệnh dùng `repeat 10`, ICMP 100 byte, timeout 2 giây. Cả sáu mẫu đều có 10 dấu `!` và `Success rate is 100 percent (10/10)`; không có lượt warm-up riêng được cung cấp.

- ISP1 source `10.255.0.33` → `10.255.0.34`: 10/10, min/avg/max **1/2/3 ms**.
- ISP1 source `203.0.113.1` → `203.0.113.2`: 10/10, **1/1/2 ms**.
- ISP2 source `10.255.0.34` → `10.255.0.33`: 10/10, **1/2/3 ms**.
- ISP2 source `203.0.113.5` → `203.0.113.6`: 10/10, **1/2/3 ms**.
- INET-TEST source `203.0.113.2` → `203.0.113.1`: 10/10, **1/1/3 ms**.
- INET-TEST source `203.0.113.6` → `203.0.113.5`: 10/10, **1/1/2 ms**.

T07, W08 và W09 đã có ping hai chiều bằng đúng source interface. Chưa dùng ping này để nghiệm thu route đến Lo100 từ ISP hoặc BGP provider.

## Tài nguyên và lưu cấu hình

EVE host `free -h`: total 7.7 GiB, used 1.5 GiB, free 5.0 GiB, buff/cache 1.2 GiB, available **6.0 GiB**; swap total 4.0 GiB, used **0 B**. `uptime` hiển thị `19:16:52 up 1:38, 1 user`, load average **1.99, 0.83, 0.36**. Múi giờ đồng hồ host chưa được cung cấp nên không quy đổi `19:16:52` thành giờ Asia/Saigon. Ba router có output console; chưa có inventory node khác đang chạy tại đúng thời điểm mẫu host, không coi đây là benchmark toàn topology.

Người thực hành xác nhận **đã `wr` toàn bộ** ba router. Chưa có raw `[OK]`, dump startup hoặc bài restart/restore sau thay đổi; mức bằng chứng save là xác nhận thực hành.

**Tiếp theo:** cấu hình BGP AS65101/65102/65200 với prefix-list giới hạn, kiểm session → route/best path → RIB → ping source W08/W09 tới `192.0.2.10/32`. Chưa bổ sung static default trên ISP hoặc peer Edge ở checkpoint này.
