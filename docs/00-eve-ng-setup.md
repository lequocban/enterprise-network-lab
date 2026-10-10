# EVE-NG Environment

## Host
- Lenovo Legion 5
- Ryzen 7 7840H
- RAM: 16 GB

## EVE-NG VM
- EVE-NG Community 6.2
- RAM: 8 GB
- CPU: 4 vCPU
- Disk: 120 GB
- Network: VMware NAT

## Management
- EVE IP: 192.168.38.130

## Verification
- Web GUI: PASS
- Internet: PASS
- Nested virtualization: PASS
- VPCS connectivity: PASS

Các trạng thái trên là ghi nhận lịch sử. Ngày 10/10/2026 đã nhận output tài nguyên/ảo hóa, version/console bốn switch, config và export sạch; agent không trực tiếp chạy thiết bị. Sprint 0 đã đóng nền tảng với follow-up theo [biên bản](sprint-reports/sprint-00-closeout.md); ASAv/Alpine theo xác nhận, IOSv 4/5 lịch sử và metadata tải bổ sung khi tới sprint liên quan.

## Kiểm tra tài nguyên ngày 10/10/2026

Nguồn: [output EVE-NG](test-results/sprint-00-eve-environment-2026-10-10.txt), lúc `07:00:39 UTC` / `14:00:39 Asia/Saigon`.

- RAM nhận 7.7 GiB; available 5.7 GiB; swap 4.0 GiB chưa dùng.
- 4 vCPU; 4 kết quả cờ virtualization; `/dev/kvm` tồn tại.
- Filesystem `/` 116 GB; available 98 GB, dùng 12%.
- Đạt kiểm tra tài nguyên/ảo hóa ban đầu. Cấu hình VMware và danh sách node đang chạy lúc đo chưa được cung cấp trực tiếp; chưa dùng số liệu để kết luận chịu tải toàn topology.

Xem [Validation cập nhật](test-results/sprint-00-validation.md#7-bằng-chứng-được-cung-cấp-ngày-10102026) và [ảnh HQ](../screenshots/sprint-00/hq-topology-2026-10-10.png). Người thực hành xác nhận đã tạo thử ASAv/Alpine và chạy được trước đó; version và output/ảnh console của lần thử chưa được lưu vào repo.

## Image inventory cần hoàn thiện

**Metadata export cuối đã đối chiếu:** image đầy đủ bốn switch là `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin`, RAM node Core512/Access256 theo thuộc tính export; vCPU node không có trong XML. Chuỗi system image file dưới đây là cách CLI hiển thị, không dùng nó để ghi đè tên đầy đủ đã có trong export. [Kiểm tra cuối](test-results/sprint-00-export-final-inspection-2026-10-10.txt).

- Router: Cisco IOSv; tên/version image cụ thể chờ ghi nhận.
- CORE1/CORE2: runtime `I86BI_LINUXL2-ADVENTERPRISEK9-M`, `15.2(CML_NIGHTLY_20151103)FLO_DSGS7`, nhãn development build. CLI giữ nguyên chuỗi `unix:/opt/unetlab/addons/iol/bin/L2-ADVENTERPRISEK9-M-15.2-IRON-20151`; export xác nhận tên image đầy đủ `L2-ADVENTERPRISEK9-M-15.2-IRON-20151103.bin` và thuộc tính RAM `512` mỗi Core. Đây là hai nguồn riêng, không sửa chuỗi CLI cho giống tên image trong export. Chưa có vCPU node.
- ACCESS1/ACCESS2: cùng software/version và chuỗi CLI như hai Core; memory IOS runtime `147698K`, 8 Ethernet interface, NVRAM 1024K. Export xác nhận cùng tên image đầy đủ với Core, thuộc tính RAM `256` mỗi Access; chưa có vCPU node. RAM trong export không phải memory IOS runtime.
- Firewall: Cisco ASAv; người thực hành xác nhận đã tạo node thử và chạy được. Image/version, thời điểm test và output/ảnh console chưa có; chưa tích hợp vào HQ hiện tại.
- Linux: Alpine cho SRV-INFRA; người thực hành xác nhận node thử chạy được. Version và output/ảnh console chưa có; chưa triển khai dịch vụ hoặc kết nối server vào HQ.

Ghi RAM/vCPU từng node, console và cách import trong hồ sơ baseline khi thu thập; không đưa binary image vào repository.

## Console và baseline Core ngày 10/10/2026

Nguồn: [CORE1](test-results/sprint-00-core1-version-hsrp-2026-10-10.txt), [CORE2](test-results/sprint-00-core2-version-hsrp-2026-10-10.txt), do người thực hành cung cấp. Hai node đã boot và thực thi lệnh show được. Uptime hiển thị lần lượt 1 và 3 phút; không có timestamp riêng cho từng console.

Mỗi node báo `409842K bytes of memory`, 12 Ethernet interface, 1 Virtual Ethernet interface và NVRAM 1024K. Đây là giá trị IOS hiển thị, chưa xác nhận RAM/vCPU được cấp trong EVE-NG; không dùng uptime ngắn để kết luận độ ổn định lâu dài.

VLAN10: CORE1 `10.10.10.2` up/up, CORE2 `10.10.10.3` up/up. HSRP group 10: CORE1 Active priority 110; CORE2 Standby priority 100; cả hai preempt, VIP `10.10.10.1`. Config bổ sung cùng ngày xác nhận cả hai mask `/24`. CORE1 running/startup khớp về lệnh, khác metadata; CORE2 running có VTY `login`, startup không có. Forwarding liên VLAN cần kiểm tra riêng.

Baseline CORE1: [running](../configs/core/sprint-00/CORE1.cfg), [startup](../configs/core/sprint-00/CORE1.startup.cfg). Đã cấu hình `rapid-pvst`, Ethernet0/0 trunk dot1q chỉ cho VLAN10; Ethernet0/1–0/2 chưa có cấu hình trunk rõ ràng. Đây là snapshot trước hoàn thiện Campus, chưa xác nhận trạng thái trunk/root STP hoặc khôi phục. Chi tiết tại [Validation](test-results/sprint-00-validation.md#tc-s00-core1-cfg-01--lưu-và-đối-chiếu-runningstartup-core1).

Baseline CORE2: [running](../configs/core/sprint-00/CORE2.cfg), [startup](../configs/core/sprint-00/CORE2.startup.cfg). Hai bản cùng cấu hình VLAN10/HSRP, rapid-pvst và trunk e0/0 chỉ VLAN10; khác dòng `login` dưới `line vty 0 4`. Chưa đồng bộ hoặc test khôi phục; chi tiết tại [Validation](test-results/sprint-00-validation.md#tc-s00-core2-cfg-01--lưu-và-đối-chiếu-runningstartup-core2).

Runtime hai Core đã bổ sung: [CORE1](test-results/sprint-00-core1-vlan-trunk-stp-route-2026-10-10.txt), [CORE2](test-results/sprint-00-core2-vlan-trunk-stp-route-2026-10-10.txt). VLAN10 `MGMT` active; Et0/0 cả hai trunking dot1q, native VLAN1, VLAN10 allowed/active/forwarding. CORE1 là root RSTP VLAN10, CORE2 dùng root port Et0/0; cả hai base priority 32768, chưa có phân cấp root primary/secondary tường minh. Connected/local routes khớp SVI, chưa có gateway of last resort. Đây là trạng thái baseline, chưa test hội tụ hoặc Inter-VLAN.

CORE1 có thêm log `Oct 10 08:09:02.721` chuyển HSRP Standby → Active, không có timezone hay chuỗi sự kiện hai phía. Đã kiểm tra lại tại clock CORE1 08:15:17.257 UTC và CORE2 08:15:52.531 UTC ngày 10/10: SVI up/up, hai mẫu standby mỗi node cùng cho CORE1 Active/CORE2 Standby, peer/VIP khớp. Không có timestamp riêng từng mẫu để đo khoảng cách hoặc convergence. Nguyên nhân log trước đó chưa rõ; chưa nghiệm thu test gây lỗi. Chi tiết tại [Validation](test-results/sprint-00-validation.md#tc-s00-hsrp-recheck-01--xác-nhận-lại-vai-trò-hai-core-trước-test).

Phép thử HSRP có kiểm soát tiếp theo đã PASS: shutdown SVI CORE1 làm CORE1 về Init, CORE2 được quan sát Active lúc 08:28:11.644 UTC; no shutdown CORE1 khôi phục SVI up/up và CORE1 Active/CORE2 Standby theo các mẫu sau test. [Log trước/trong/sau](test-results/sprint-00-hsrp-switchover-2026-10-10.txt) có lệnh gây lỗi/khôi phục; chưa đo convergence hoặc client failover. Đây là khôi phục trạng thái vận hành SVI, chưa phải kiểm thử import/reload/khôi phục toàn lab. Tiếp theo đối chiếu config CORE1 sau test trước export; xem [Validation](test-results/sprint-00-validation.md#tc-s00-hsrp-switchover-01--shutdown-svi-core1-chuyển-vai-trò-và-khôi-phục).

Đã bổ sung running/startup sau test của cả hai Core thành bốn file `*.post-switchover*.cfg`: từng bản khớp lệnh với snapshot tương ứng trước test, không còn shutdown dưới SVI; CORE1 khớp running/startup, CORE2 vẫn khác VTY `login`. Log CORE1 Standby → Active lúc 08:35:49 được giữ riêng; người thực hành xác nhận có thao tác trước lần lấy output, chưa nêu chi tiết. Không suy ra lỗi tự phát/độ ổn định liên tục từ sự kiện này. Tiếp theo export topology HQ; [Validation sau test](test-results/sprint-00-validation.md#tc-s00-post-cfg-01--đối-chiếu-cấu-hình-hai-core-sau-phép-thử) liên kết đủ cấu hình/metadata.

## Console và trạng thái Access ngày 10/10/2026

Nguồn: [ACCESS1](test-results/sprint-00-access1-version-vlan-2026-10-10.txt), [ACCESS2](test-results/sprint-00-access2-version-vlan-2026-10-10.txt), do người thực hành cung cấp. Hai node đã boot, console thực thi lệnh được; uptime hiển thị lần lượt 0 và 2 phút, không có timestamp riêng.

Output IP chỉ liệt kê Ethernet0/0–0/3 và Ethernet1/0–1/3, chưa có SVI/IP quản trị. `show vlan brief` chỉ có VLAN1 và 1002–1005, liệt kê cả 8 port ở VLAN1; chưa có VLAN10–60/99 của thiết kế. Đây là baseline trước triển khai Campus, không phải kết quả hoàn tất VLAN/trunk/STP. Cấu hình quản trị Access `.11/.12` và VLAN/trunk thuộc Sprint 1.

Baseline cấu hình Access đã lưu: ACCESS1 [running](../configs/access/sprint-00/ACCESS1.cfg)/[startup](../configs/access/sprint-00/ACCESS1.startup.cfg), ACCESS2 [running](../configs/access/sprint-00/ACCESS2.cfg)/[startup](../configs/access/sprint-00/ACCESS2.startup.cfg). ACCESS2 khớp lệnh và thời gian last change; ACCESS1 chỉ running có VTY `login`. Cả hai có mode `rapid-pvst`, chưa có SVI quản trị hoặc cấu hình interface trunk/access VLAN tường minh. Chưa kiểm thử khôi phục hoặc xác nhận root/port role/hội tụ; chi tiết tại [Validation](test-results/sprint-00-validation.md#tc-s00-access-cfg-01--lưu-và-đối-chiếu-runningstartup-hai-access).

## Metadata từ export ngày 10/10/2026

[ZIP HQ hiện tại](../eve-exports/sprint-00-hq-2026-10-10.zip) có tám node/chín link đúng bảng cổng, CRC/XML PASS. PC-GUEST dư ID8 của bản đầu không còn, ID9 vẫn nối ACCESS2 e0/3. SHA256 cuối và metadata tại [biên bản kiểm tra cuối](test-results/sprint-00-export-final-inspection-2026-10-10.txt). ZIP không có config payload hoặc file config riêng; `.cfg`/evidence VLAN phải đi kèm, chưa thử import/khôi phục. Thuộc tính XML lab version `1` không phải version EVE-NG. Chi tiết tại [Validation export cuối](test-results/sprint-00-validation.md#tc-s00-export-final-01--export-hq-hiện-tại-sau-dọn-node-dư).

## Phạm vi topology Sprint 00

Topology hiện tại gồm CORE1, CORE2, ACCESS1, ACCESS2 và bốn VPCS test. ASAv và Alpine đã được tạo thử/chạy thử theo xác nhận của người thực hành ngày 10/10/2026; không cần giữ các node thử trong HQ để nghiệm thu việc chuẩn bị môi trường. Việc tích hợp firewall vào topology thuộc Sprint 5; server và dịch vụ thuộc Sprint 7, hoặc thêm endpoint VLAN50 khi cần bài test Sprint 1.

Đã chạy thử và đã lưu bằng chứng là hai trạng thái riêng. Có thể bổ sung version/ảnh khi mở lại node thử; trước mắt tiếp tục lưu baseline cấu hình HQ hiện có.
