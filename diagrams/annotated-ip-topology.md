# Topology có nhãn subnet, IP cổng và vùng mạng

> Đối chiếu ngày 24/10/2026. **Nét liền**: IP/link đã có bằng chứng runtime. **Nét đứt**: link đã cắm nhưng IP đầu Edge/Branch mới là phân bổ trong IP plan, chưa có bằng chứng cấu hình. Sơ đồ này bổ sung chú thích cho ảnh EVE; không thay ảnh/export lab.

```mermaid
flowchart TB
  subgraph INTERNET[INTERNET TEST - AS 65200]
    INET["INET-TEST<br/>Lo100 192.0.2.10/32<br/>Gi0/0 .2 - Gi0/1 .6"]
  end

  subgraph PROVIDER[PROVIDER - hai AS riêng]
    ISP1["ISP1 - AS 65101<br/>Lo0 10.255.255.101/32"]
    ISP2["ISP2 - AS 65102<br/>Lo0 10.255.255.102/32"]
  end

  subgraph BRANCH1[BRANCH 1 - LAN 10.20.10.0/24]
    BR1["BR1-R1<br/>Gi0/0 .25 - Gi0/1 10.20.10.1 dự kiến"]
    PCBR1["PC-BR1<br/>eth0: chưa xác nhận IP"]
  end

  subgraph BRANCH2[BRANCH 2 - LAN 10.30.10.0/24]
    BR2["BR2-R1<br/>Gi0/0 .29 - Gi0/1 10.30.10.1 dự kiến"]
    PCBR2["PC-BR2<br/>eth0: chưa xác nhận IP"]
  end

  subgraph HQEDGE[HQ INTERNET EDGE - AS 65010]
    EDGE1["EDGE1<br/>Gi0/1 .17 - Gi0/0 .10 dự kiến"]
    EDGE2["EDGE2<br/>Gi0/1 .21 - Gi0/0 .11 dự kiến"]
    T02(("T02 - bridge VLAN91<br/>10.255.0.8/29<br/>FW .9 dự kiến"))
  end

  subgraph CAMPUS[HQ CAMPUS - VLAN 10, 20, 30, 40, 50, 60]
    CORE1["CORE1<br/>SVI VLAN x: 10.10.x.2/24"]
    CORE2["CORE2<br/>SVI VLAN x: 10.10.x.3/24"]
    ACC1["ACCESS1<br/>MGMT VLAN10 .11/24"]
    ACC2["ACCESS2<br/>MGMT VLAN10 .12/24"]
    USERS1["PC-IT VLAN20 / PC-HR VLAN30<br/>IP từng máy: xem VPCS"]
    USERS2["PC-ACC VLAN40 10.10.40.20/24<br/>PC-GUEST VLAN60: xem VPCS"]
  end

  ISP1 ---|"W08 203.0.113.0/30<br/>Gi0/3 .1 ↔ Gi0/0 .2"| INET
  ISP2 ---|"W09 203.0.113.4/30<br/>Gi0/3 .5 ↔ Gi0/1 .6"| INET
  ISP1 ---|"T07 10.255.0.32/30<br/>Gi0/2 .33 ↔ Gi0/2 .34"| ISP2
  ISP1 -.-|"T03 10.255.0.16/30<br/>Gi0/0 .18 ↔ EDGE1 Gi0/1 .17 dự kiến"| EDGE1
  ISP2 -.-|"T04 10.255.0.20/30<br/>Gi0/0 .22 ↔ EDGE2 Gi0/1 .21 dự kiến"| EDGE2
  ISP1 -.-|"T05 10.255.0.24/30<br/>Gi0/1 .26 ↔ BR1 Gi0/0 .25 dự kiến"| BR1
  ISP2 -.-|"T06 10.255.0.28/30<br/>Gi0/1 .30 ↔ BR2 Gi0/0 .29 dự kiến"| BR2
  BR1 -.-|"LAN Branch 1<br/>Gi0/1 .1 ↔ eth0 IP chưa xác nhận"| PCBR1
  BR2 -.-|"LAN Branch 2<br/>Gi0/1 .1 ↔ eth0 IP chưa xác nhận"| PCBR2
  EDGE1 -.-|"Gi0/0 .10 dự kiến"| T02
  EDGE2 -.-|"Gi0/0 .11 dự kiến"| T02
  CORE1 ===|"Po1: e0/0 + e0/3<br/>trunk L2, KHÔNG có IP cổng"| CORE2
  CORE1 ===|"Po11: e0/1 + e1/1<br/>trunk L2, KHÔNG có IP cổng"| ACC1
  CORE2 ---|"e0/1 ↔ e0/1<br/>trunk L2, không IP"| ACC1
  CORE1 ---|"e0/2 ↔ e0/0<br/>trunk L2, không IP"| ACC2
  CORE2 ---|"e0/2 ↔ e0/1<br/>trunk L2, không IP"| ACC2
  ACC1 ---|"e0/2 ↔ PC-IT eth0<br/>VLAN20 10.10.20.0/24<br/>e0/3 ↔ PC-HR eth0<br/>VLAN30 10.10.30.0/24"| USERS1
  ACC2 ---|"e0/2 ↔ PC-ACC eth0<br/>VLAN40 10.10.40.0/24<br/>e0/3 ↔ PC-GUEST eth0<br/>VLAN60 10.10.60.0/24"| USERS2
```

## Nhãn nên đặt trong EVE

- Dùng **rectangle** cho năm vùng: Internet Test, Provider, Branch 1, Branch 2, HQ (Edge và Campus có thể tách hai rectangle con). Đặt rectangle phía sau node/link.
- Dùng **text** sát mỗi WAN link: subnet `/30` ở giữa dây; IP đầy đủ hoặc hậu tố `.x` ở sát đầu cổng. Với T02 dùng một nhãn `/29` ngay trên bridge, IP `.10/.11` sát hai Edge. `FW .9` là địa chỉ **dự kiến**, vì firewall chưa được nối trong ảnh.
- Trên dây Core–Access chỉ ghi `trunk L2`, `Po1` hoặc `Po11`; **không ghi .1/.2 lên cổng Ethernet**. IP `.1` là HSRP VIP của từng VLAN, `.2/.3` là SVI trên CORE1/CORE2. Ví dụ VLAN40: VIP `10.10.40.1`, CORE1 `10.10.40.2`, CORE2 `10.10.40.3`.
- Chỉ dùng nhãn `đã cấu hình` cho ba router Provider và các SVI/Po Campus đã có bằng chứng. Các IP Edge/Branch, T02 và LAN Branch trong sơ đồ là **theo kế hoạch**, chưa được xác nhận bằng output trên lab tích hợp.

Nguồn: [IP plan](../docs/ip-addressing-plan.md), [provider IP runtime](../docs/test-results/sprint-04-provider-ip-2026-10-24.md), [provider BGP runtime](../docs/test-results/sprint-04-provider-bgp-2026-10-24.md), [Campus baseline](../docs/test-results/sprint-04-campus-baseline-2026-10-23.md), [physical topology](physical-topology.md).
