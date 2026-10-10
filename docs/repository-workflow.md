# Quy ước repository và bằng chứng

Đây là quy ước dùng chung; kế hoạch thao tác nội bộ của agent không thuộc bộ docs công khai.

## Git và review

- `master` giữ bản đã review; nhánh tài liệu dùng `docs/<chu-de>`, triển khai dùng `lab/sprint-<NN>-<chu-de>`.
- Mỗi commit/PR giải quyết phần có thể review độc lập; mô tả vấn đề, kết quả, validation và giới hạn.
- Commit local, push, PR và merge là các trạng thái riêng. Chỉ ghi đã xuất bản khi có bằng chứng tương ứng.
- Giữ thay đổi hiện có; không reset hoặc force-push để hoàn tất một sprint.

## Artifact

- `docs/` và `diagrams/`: thiết kế, báo cáo, nghiệm thu; ghi rõ planned và đã kiểm chứng.
- `configs/<nhom>/sprint-<NN>/`: cấu hình lấy từ thiết bị, tên node và `.cfg`; phân biệt running/startup, bản trước/sau test.
- `docs/test-results/`: output `.txt`, tổng hợp `.md`; `*.log` hiện bị ignore.
- `eve-exports/`: export thật, ghi hash/metadata và có/không cấu hình nhúng; không mặc định đã import/restore được.
- `screenshots/sprint-<NN>/`: ảnh thật có ngày và nguồn. Git không lưu thư mục rỗng; thêm README hoặc tạo thư mục khi có artifact.
- `ai-agent/`: ghi chú/kế hoạch nội bộ bị ignore; docs công khai không liên kết vào đây.
- Không đưa binary Cisco/VM image, private key hoặc credential vào Git. Review config/export trước stage.

## Nghiệm thu

Ghi nguồn, ngày, topology, expected/actual, lệnh gây lỗi và khôi phục. Agent đọc output người dùng không đồng nghĩa agent đã chạy thiết bị. Giữ kết quả lịch sử, thêm kết quả mới; không tạo output từ thiết kế hoặc tính convergence từ hai lần đọc console thưa.

Story thiết kế được review bằng deliverable/đối chiếu; story triển khai cần test phù hợp. Nếu đóng sprint có follow-up, ghi rõ phần chưa thử và story/mốc tiếp nhận. Thay IP/cổng phải cập nhật thiết kế/config/test liên quan cùng nhau.

Xem [biên bản Sprint 00](sprint-reports/sprint-00-closeout.md) và [mẫu test case](test-results/test-case-template.md).
