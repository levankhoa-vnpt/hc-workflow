---
name: work
description: Chỉ dùng khi người dùng gọi rõ "$work" hoặc gõ "/work". Code tính năng lần đầu dựa trên spec đã duyệt — KHÔNG chạy git, KHÔNG build/commit.
---

Trước khi code, đọc NAMING_CONVENTIONS.md và áp dụng cho thành phần
MỚI; giữ nguyên convention cũ khi sửa code đã tồn tại.

0. Xác định nguồn yêu cầu theo THỨ TỰ ƯU TIÊN sau, dừng ở điều kiện
   đầu tiên khớp:
   a. NGỮ CẢNH SESSION: nếu trong CHÍNH cuộc hội thoại này đã từng
      chạy $spec hoặc $work hoặc $rework cho 1 feature-id cụ thể
      trước đó, dùng LUÔN feature-id đó — không đọc file CURRENT.md,
      không hỏi lại người dùng.
   b. Đối số đưa vào khớp tên file .opencode/specs/<đối-số>.md đang
      tồn tại: đọc toàn bộ file đó, feature-slug = đối số.
   c. Đối số RỖNG và không có ngữ cảnh session: đọc
      .opencode/specs/CURRENT.md lấy feature-id. Nếu file này không
      tồn tại/rỗng, DỪNG và báo: chưa có spec nào đang chờ, chạy
      $spec trước hoặc gọi $work kèm mô tả trực tiếp.
   d. Đối số không rỗng và không khớp file spec nào: coi đối số là mô
      tả yêu cầu trực tiếp, tự đặt feature-slug.

0b. RÀNG BUỘC CÔ LẬP: mỗi feature có 1 file spec riêng
    (.opencode/specs/<feature-id>.md) và 1 file progress riêng
    (.opencode/progress/<feature-id>.md) — đây là ranh giới của
    feature này. TUYỆT ĐỐI không đọc/tham chiếu nội dung spec hoặc
    progress của feature khác, không gộp yêu cầu của 2 feature làm 1,
    không sửa file nào không thuộc phạm vi đã xác định ở PHẠM VI THAY
    ĐỔI trong spec (hoặc đã tự phân tích nếu không có spec).

1. Kiểm tra .opencode/progress/<feature-slug>.md xem có việc dang dở
   không — nếu có, đọc và tiếp tục từ next_action.
2. Nếu đọc từ spec: dùng luôn repo/phạm vi đã phân tích sẵn. Nếu mô tả
   trực tiếp: đọc AGENTS.md, bảng định tuyến, tự xác định repo.
3. Tạo .opencode/progress/<feature-slug>.md nếu chưa có, status:
   in_progress, kèm link spec gốc nếu có.
4. Tự phân tích, lên kế hoạch.
5. TUYỆT ĐỐI KHÔNG chạy bất kỳ lệnh git nào (không checkout, không
   branch, không commit, không push, không merge). Giả định người
   dùng đã tự chuẩn bị đúng branch cần thiết trong thư mục gốc của
   mỗi repo trước khi giao việc. Code trực tiếp vào working directory
   hiện tại của repo đó.
6. Nếu có thay đổi schema/DB: CHỈ tạo file SQL migration theo
   convention deploy/database/YYYYMMDD/<tên>.sql — TUYỆT ĐỐI KHÔNG tự
   chạy/apply migration vào database thật.
7. TUYỆT ĐỐI KHÔNG chạy mvn, npm, hay bất kỳ lệnh build/test nào. Tự
   review code bằng cách đọc lại logic, so với acceptance criteria.
8. Nếu chưa đạt, quay lại bước 5.
9. Cập nhật progress file (status, files_touched theo từng repo,
   next_action).
10. Set progress file: status: waiting_review, ghi RÕ theo từng repo:
    danh sách file đã đổi (đường dẫn tính từ gốc repo), danh sách file
    SQL migration (nếu có). KHÔNG ghi tên branch.
11. DỪNG LẠI, báo người dùng: code đã xong (CHƯA build/commit), tóm
    tắt file đã đổi theo từng repo. Người dùng sẽ tự build/test/commit
    /push/deploy; nếu có lỗi sẽ dùng skill $rework.

CHỈ dừng hỏi ngoài điểm trên khi thật sự cần quyết định nghiệp vụ,
thiếu thông tin, hoặc thiếu credential.
---
