# Sprint 3 — Đường ra mô phỏng và Default Route Injection

**Ngày tiếp nhận:** 21/10/2026, Asia/Saigon.  
**Nguồn:** output người thực hành cung cấp trong chat; agent không chạy console/EVE.  
**Phạm vi:** NET-304 trong [routing sandbox](../routing/sprint-03-routing-design.md), không phải kết nối ISP/Internet Enterprise đã tích hợp.

## Uplink, static default và return path — actual

- EDGE2 → ISP1 `.50`10/10, RTT1/1/3ms; ISP1 → EDGE2 `.49`10/10, RTT1/1/2ms.
- EDGE2 lookup`0.0.0.0`: route`0.0.0.0/0`, known via static/AD1/metric0/candidate default, next-hop`10.255.0.50`.
- EDGE2 → target`192.0.2.10`, source`10.255.0.49`,10/10, RTT1/1/2ms. Command/output ghi rõ source, không thay bằng nguồnLoopback0.
- ISP1 interface brief: Gi0/0`.50`, Lo0`10.255.255.101`, Lo100`192.0.2.10` up/up; Gi0/1–3 unassigned/admin down. Chưa có version/image/OSPF process inventory riêng của ISP1 trong checkpoint mới; không tạo các output đó.
- PC-HQ → `192.0.2.10`5/5, TTL254, chứng minh path đi/về cho client mạngPC-HQ. Chưa có traffic Branch tới target và chưa có evidence defaultOSPF/Type5.
- EVE free: total7.7Gi/used1.7Gi/free5.0Gi/shared6.0Mi/buff-cache963Mi/available5.7Gi, swap4.0Gi/used0B. Uptime hiển thị10:15:06/up2:03/2user/load2.08–2.35–2.49. Giữ clock/múi giờ EVE chưa đối chiếu; sốuser làphiênđăngnhập, không phải sốnode. Mẫu này không nghiệm thu tải dài hạn.

**Kết luận:** uplink/static default/EDGE2 target và PC-HQ return path đã có bằng chứng đạt. ISP1 tồn tại theo console; topology5router/3VPCS/7link đúng thiết kế dự kiến nhưng chưa có ảnh/export mở rộng đối chiếu toàn bộ. Staticreturnroutes4prefix, consoleISP1 và việc không chạy OSPF trênISP1 theo hướng dẫn chưa có full config xác nhận; Branch test tới target sẽ bổ sung bằng chứng return path tương ứng.

## OSPF default injection — bước tiếp, chưa chạy

Chỉ trên EDGE2 process10 thêm `default-information originate`, khôngalways/route-map. Expected EDGE2 thànhASBR, Type5DefaultLSA LSID0.0.0.0/ADVRouter`10.255.255.12`; router bên trong học external-type2 default, không phải OIA. Không advertise route/32target vào OSPF.

Kiểm EDGE2 vẫn chỉ có neighbor EDGE1, lookup target ởBR1/BR2 phải match default, không route cụ thể. Hai PCBranch ping target và trace đại diện quaBR→EDGE1→EDGE2→ISP1. Chưa có actual cho bước này; PC-HQ5/5 nêu trên là trướcinjection, không gắn nhãn phép thử cả3client cùng sauinjection.

Chỉ lưu phần config mới EDGE2/ISP1 sau kiểm chứng; nếu đã lưu đúng checkpoint thì không lặp. Buổi7 gỡ đúng staticdefault EDGE2, giữ originate để kiểm withdrawal/khôi phục; không đổi timer/clearprocess hoặc shutdowncảlab. Lệnh originate phản ánh defaultRIB, không chứng minh tự kiểm tra upstream Internet hoặc dualISPfailover.

## Default injection — actual21/10

- EDGE2 có đúng một neighborFULL/- với EDGE1`.11` qua`.37`Gi0/0. Output process ghi `It is an autonomous system boundary router`.
- BR1 fullRIB: Gatewayoflastresort`.41`, `O*E2 0.0.0.0/0 [110/1]` qua`.41`Gi0/0. BR2 tương ứng gateway`.45`, O*E2/0AD110/metric1 qua`.45`Gi0/0. Các routeLAN/summary/loopback nội bộ vẫn tồn tại trong hai RIB nhận.
- BR1 vàBR2 `show ip route 192.0.2.10` trả `% Network not in table` dù fullRIB códefault và clientpingthông. Đây không phải defaultinjectionfail; hướng dẫn trước dự kiến lookupđích phảiinra0/0 đã không phù hợp hànhviCLI IOSv này. Kiểmdefault bằng`show ip route 0.0.0.0`/fullRIB; dùng`show ip cef 192.0.2.10` khi cần xemFIBlookup. Chưa cóoutputCEF, không tạo kết quả đó.
- PC-BR1 → target5/5 vàPC-BR2 → target5/5, TTL252. PC-HQ5/5 có ởcheckpointunderlay trướcinjection, không gộp thànhba phép thử đồngthời sauinjection.
- TracePC-BR1 →192.0.2.10: `10.20.10.1`→`10.255.0.41`→`10.255.0.38`→`10.255.0.50` vớiICMP3/3 ởhop4. `.50` làinterface ISP1đãcóLo100target; tracehợpvớiBR1→E1→E2→ISP1, không coi3/3 làroutingfail hoặc tự sửa thànhsourceLo100tronglog.
- Type5LSDB chi tiết/ADV`.12` chưa được gửi; O*E2/processASBR làactual đãcó, không tạoLSDBraw. Chưa cóxácnhậnwr saudefaultinjection/ISPsetup ởlượt này.

**Kết luận:** defaultinjection vàforwarding haiBranch đãđạt chức năng. Không thêm /32target vàoOSPF hoặc staticrouteBranch để làm lệnhshowđích đổioutput. Cầnkiểm Type5 vàwithdrawal/recovery buổi7; chưafullDoD NET-304.

## Buổi 7 — withdrawal/recovery, kế hoạch trước thực hành

Nếu chưa lưu checkpointdefaultinjection/ISPsetup, wrEDGE2/ISP1vàchờ`[OK]` trước thử. TrênBR1 đọcType5default trướclỗi, rồiEDGE2gỡ đúng`ip route 0.0.0.0 0.0.0.0 10.255.0.50`, giữ`default-information originate`. Expected defaultRIB/LSA rút ởBranch sauhộitụ (LSA cóthể hiệnMaxAge trongphaflush); không đòi interface/neighbor nội bộdown.

Lookup0.0.0.0cảhaiBranch phải khôngcòn default; PC-BR1→targetfail, BR1→BR2LANcònthông. Thêm lại đúngstaticdefault E2, kiểmdefaultBranch/Type5/traffic trởlại. Khôngwrpha lỗi, khôngđo thời gian từhai console thưa vàkhông gọiđây làISPupstreamtracking/dualISPfailover. Chưa cóactualwithdrawal trongcheckpoint này.

## Buổi 7 — withdrawal/recovery, actual 21/10

- Trước lỗi, BR1 có Type-5 default LSA: LSID `0.0.0.0`, mask `/0`, Advertising Router `10.255.255.12`, metric type 2, metric 1, forwarding address `0.0.0.0`, route tag 10.
- Người thực hành gỡ đúng static default trên EDGE2, giữ OSPF origination. Cả BR1 và BR2 lookup `0.0.0.0` trả `% Network not in table`. Chưa có LSDB trong pha lỗi để xác nhận trực tiếp quá trình flush LSA.
- Khi default mất, PC-BR1 → `192.0.2.10` nhận 5 ICMP type 3/code 1 từ gateway `10.20.10.1`; PC-BR1 → PC-BR2 `10.30.10.20` vẫn 5/5.
- Sau khôi phục, BR1 nhận default OSPF extern 2 qua `.41`, BR2 qua `.45`: AD 110, metric 1, forward metric 2, Advertising Router `.12`, tag 10.
- BR2 có lại Type-5 default LSA `/0` từ `10.255.255.12`, metric type 2/metric 1, forwarding address 0, tag 10. Hai PC Branch → `192.0.2.10` mỗi máy 5/5, TTL 252.

**Kết luận:** bài rút và khôi phục default đạt chức năng; mất đường ra không làm mất traffic LAN giữa hai Branch trong mẫu kiểm tra. Không suy ra thời gian hội tụ hoặc khả năng theo dõi upstream ISP. Chưa có xác nhận `wr` sau khôi phục. Ảnh topology mới có 5 router, 3 VPCS, 7 link khớp sơ đồ dự kiến; chưa thay thế kiểm tra export/persistence cuối sprint.

## EDGE2 running/startup — actual 22/10/2026

- Running-config có `ip route 0.0.0.0 0.0.0.0 10.255.0.50` và OSPF10 với RID `.12`, `passive-interface default`, chỉ Gi0/0 non-passive, `default-information originate`.
- EDGE2 `wr` trả `[OK]`, tiếp theo IOSv báo GRUB config được ghi thành công. Startup-config sau đó có cùng static default và phần OSPF vừa nêu.

**Kết luận:** persistence của phần default injection trên EDGE2 đã được đối chiếu running/startup. Chưa có reboot EDGE2, full config dump hoặc bằng chứng save sau hoàn nguyên trên EDGE1/BR2 trong output lượt này.
