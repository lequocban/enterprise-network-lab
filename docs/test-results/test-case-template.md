# Mẫu test case — chưa phải kết quả kiểm thử

Sao chép mẫu vào báo cáo kiểm thử sprint khi bắt đầu bài test. Giữ trạng thái **Chưa chạy** cho tới khi có actual result và evidence thật. Quy ước lưu output nằm trong [Repository workflow](../repository-workflow.md).

## Thông tin

- Test ID: `<SPRINT-NN-TC-NN>`
- Story / đầu việc: `<NET-xxx / S0-xx / S1-xx>`
- Ngày giờ và múi giờ: `<ngày thu thập thật, Asia/Saigon>`
- Người thực hiện: `<tên>`
- Trạng thái: **Chưa chạy / PASS / FAIL / Chờ xác minh**
- Node / image / version: `<thiết bị và phiên bản thực tế>`
- Topology / baseline config: `<link tương đối>`
- Mục tiêu: `<điều cần chứng minh>`

## Điều kiện trước khi test

- `<node đang bật và tài nguyên lúc đo nếu liên quan>`
- `<interface/VLAN/routing/ACL cần có>`
- `<IP/mask/gateway của endpoint test>`
- `<giới hạn hoặc khác biệt với thiết kế cuối>`

## Các bước và kết quả

| Bước | Thao tác/lệnh | Kết quả mong đợi | Kết quả thực tế | Bằng chứng |
|---|---|---|---|---|
| 1 | `<thao tác>` | `<expected>` | Chưa chạy | Chưa có |
| 2 | `<thao tác>` | `<expected>` | Chưa chạy | Chưa có |

## Đo kết nối hoặc failover

- Hướng/source/destination: `<địa chỉ thực tế>`
- Ping warm-up: `<có/không và kết quả>`
- Số gói / khoảng cách giữa các gói: `<giá trị thực tế>`
- Gói gửi/nhận/mất và RTT: `<actual>`
- Sự kiện gây lỗi và thời điểm: `<nếu có>`
- Trạng thái trước/trong/sau: `<link output>`
- Thời gian hồi phục: `<cách đo và actual; không đo thì ghi chưa đo>`
- Giới hạn kết luận: `<ví dụ SVI up/up chưa chứng minh client forwarding>`

## Kết luận và khôi phục

- Kết luận theo acceptance criteria: `<đạt/chưa đạt/chưa đủ bằng chứng>`
- Lỗi phát hiện / xử lý / retest: `<mô tả hoặc không có>`
- Khôi phục baseline sau gây lỗi: `<thao tác và kiểm tra>`
- Config/output/ảnh/capture: `<link thật>`
- Commit/PR: `<chỉ điền khi thực sự đã có>`
