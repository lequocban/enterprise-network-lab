# Topology logic — thiết kế dự kiến v1.0

**Ngày review:** 10/10/2026

**Nguồn:** [Architecture](../docs/network-architecture.md) và [IP Plan](../docs/ip-addressing-plan.md).

**Trạng thái:** thiết kế đã review khi đóng Sprint 0. HQ đối chiếu ảnh/export tám node/chín link; WAN, firewall policy, Branch và services là planned. Sơ đồ không chứng minh mọi node đã chạy hoặc forwarding đúng.

```mermaid
flowchart TB
    ISP1[ISP1 - planned] ---|T07| ISP2[ISP2 - planned]
    ISP1 ---|T03| EDGE1[EDGE1 - planned]
    ISP2 ---|T04| EDGE2[EDGE2 - planned]
    EDGE1 --- OUTSIDE[VLAN91 - planned L2 segment]
    EDGE2 --- OUTSIDE
    OUTSIDE --- FW1[FW1 ASAv - image installed; deployment planned]
    FW1 --- DMZ[DMZ 10.10.100.0/24 - planned]
    DMZ --- WEB[DMZ Web 10.10.100.10 - planned]
    FW1 --- INSIDE[VLAN90 - planned L2 segment]
    INSIDE --- CORE1[CORE1 - HQ reported]
    INSIDE --- CORE2[CORE2 - HQ reported]
    CORE1 --- CORE2
    CORE1 --- ACCESS1[ACCESS1 - HQ reported]
    CORE2 --- ACCESS1
    CORE1 --- ACCESS2[ACCESS2 - HQ reported]
    CORE2 --- ACCESS2
    ACCESS1 --- USERS1[Test endpoints IT / HR]
    ACCESS2 --- USERS2[Test endpoints Accounting / Guest]
    ACCESS1 -. planned VLAN50; port TBD .-> INFRA[SRV-INFRA Alpine - services planned]
    ISP1 ---|T05| BR1[BR1-R1 - planned]
    ISP2 ---|T06| BR2[BR2-R1 - planned]
    BR1 --- LAN1[Branch1 10.20.10.0/24]
    BR2 --- LAN2[Branch2 10.30.10.0/24]
```

## Cách đọc

- Các đường mô tả quan hệ kết nối dự kiến; trạng thái forwarding/trunk/shutdown phải được kiểm tra riêng.
- VLAN10/20/30/40/50/60 dùng SVI trên hai Core, gateway HSRP theo IP Plan. Native VLAN99 không cấp IP.
- VLAN90 nối hai Core với FW inside; VLAN91 nối FW outside với hai Edge. Mỗi vùng cần broadcast domain chung, chưa chốt node Layer 2/cổng cụ thể.
- SRV-INFRA dự kiến nối Access qua VLAN50; lựa chọn Access/cổng phải chốt trong physical topology trước Sprint 7. Đường tới ACCESS1 trong sơ đồ là đề xuất, chưa xác nhận cabling.
- DMZ gateway nằm ở FW1, không trên Core. Test endpoints là VPCS linh động, không phải hạ tầng cố định.
- VPN HQ–Branch là lớp kết nối dự kiến qua ISP; chưa vẽ tunnel endpoint cố định vì quyết định terminate ở FW/Edge còn mở tại WAN-01.

## Điểm còn mở

Gateway outside của ASA và failover; default/return routes; OSPF Branch đi trên đường nào; VPN/NAT placement; Internet Test Server và cổng/server DMZ. Theo dõi tại [WAN-01](../docs/sprint-reports/sprint-00-closeout.md#3-việc-chuyển-tiếp-và-điều-kiện-thực-hiện).
