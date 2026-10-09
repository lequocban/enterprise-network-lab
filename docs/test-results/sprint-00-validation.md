# Sprint 0 — Kết quả kiểm thử

**Ngày tổng hợp:** 09/10/2026  
**Nguồn:** Output console đã ghi lại trong quá trình lab. Đây không phải kết quả chạy tự động từ GitHub.

## 1. Kiểm tra EVE-NG

```text
free -h                       Total 7.7Gi | Available 6.3Gi | Swap 4.0Gi (unused)
df -h /                       116G total | 12G used | 100G available
nproc                         4
egrep -c '(vmx|svm)' /proc/cpuinfo      4
```

Đánh giá: tài nguyên và cờ nested virtualization đã nhận. Chưa dùng những con số này để kết luận hệ thống chịu được toàn bộ node Enterprise cùng lúc.

## 2. Router-to-router ping

R1 Gi0/0: `10.255.0.1/30`, R2: `10.255.0.2/30`. Ping R1 → R2: **4/5 (80%)**, RTT min/avg/max **1/1/2 ms**. Gói ping đầu mất có thể do ARP, nhưng chưa có capture để xác nhận nguyên nhân. Lưu cấu hình R1 thành công.

## 3. SVI và HSRP

```text
CORE1: Vlan10 10.10.10.2 up/up
CORE2: Vlan10 10.10.10.3 up/up

CORE1: Vl10 Group10 Priority110 P Active  local       10.10.10.3  VIP 10.10.10.1
CORE2: Vl10 Group10 Priority100 P Standby 10.10.10.2 local       VIP 10.10.10.1
```

VLAN10 tạo được trên Core, CORE1 có connected route `10.10.10.0/24`. Hai Core nhận diện peer và chọn đúng vai trò HSRP. Test shutdown/no shutdown SVI trên CORE1 được người thực hành xác nhận hoàn tất, nhưng chưa lưu đủ log để đo convergence.

Trong bước thử SVI có một **VPCS tạm nối trực tiếp CORE1 e1/0**. Máy này chỉ để đưa VLAN10 lên phục vụ test, không thuộc topology vận hành chính thức và không cần IP dự trữ cố định.

## 4. Chưa kiểm chứng

Inter-VLAN Routing nhiều VLAN; RSTP hội tụ khi mất uplink; EtherChannel; failover client end-to-end khi CORE1 mất nguồn; ASA/Alpine boot screenshot; WAN, OSPF/BGP, VPN, NAT, monitoring, automation.

## 5. Đầu việc bổ sung minh chứng

- Ảnh đầy đủ trạng thái ASAv và Alpine khi boot.
- Lưu `show ip int brief`, `show standby brief` trước/trong/sau failover.
- Chụp topology mới khi bỏ máy PC-MGMT test khỏi sơ đồ chuẩn.
- Từ Sprint 1 lưu test case cùng ngày với cấu hình.
