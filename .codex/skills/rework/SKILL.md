---
name: rework
description: Chỉ dùng khi người dùng gọi rõ "$rework" hoặc gõ "/rework". Sửa lỗi/tinh chỉnh sau khi đã tự kiểm tra — không theo thứ tự, KHÔNG git/build.
---

Đây là skill sửa lỗi sau khi người dùng đã tự build/test/deploy và
phát hiện vấn đề, hoặc muốn chỉnh thêm. KHÔNG cần theo đúng trình tự
các bước như $work — sửa trực tiếp đúng chỗ liên quan.

0. Xác định feature theo THỨ TỰ ƯU TIÊN:
   a. NGỮ CẢNH SESSION: nếu trong CHÍNH cuộc hội thoại này đã từng
      chạy $spec hoặc $work hoặc $rework cho 1 feature-id cụ thể,
      dùng LUÔN feature-id đó, toàn bộ đối số là mô tả lỗi/yêu cầu
      sửa — không cần hỏi lại feature nào.
   b. Nếu KHÔNG có ngữ cảnh session, token đầu tiên của đối số PHẢI là
      feature-id khớp tên file .opencode/progress/<token>.md đang tồn
      tại — dùng làm feature-slug, phần còn lại là mô tả lỗi. Nếu
      không khớp, đọc .opencode/progress/CURRENT.md lấy feature-slug
      gần nhất.

0b. RÀNG BUỘC CÔ LẬP: chỉ đọc/tham chiếu đúng file spec và progress
    của feature-slug đang xử lý, không đụng tới feature khác.

1. Đọc .opencode/progress/<feature-slug>.md để biết file đã đổi, spec
   gốc (nếu có).
2. TUYỆT ĐỐI KHÔNG chạy bất kỳ lệnh git nào. Giả định người dùng đã tự
   đứng đúng branch cần sửa trong thư mục gốc của mỗi repo liên quan
   trước khi gọi $rework. Sửa trực tiếp vào working directory hiện
   tại.
3. Đọc mô tả lỗi, tự xác định file/vùng code cần sửa, sửa trực tiếp.
4. Nếu liên quan schema/DB: CHỈ tạo/sửa file SQL migration, KHÔNG tự
   chạy/apply.
5. TUYỆT ĐỐI KHÔNG chạy mvn, npm, hay bất kỳ lệnh build/test nào. Sửa
   xong là XONG.
6. Cập nhật .opencode/progress/<feature-slug>.md: thêm mục "Lịch sử
   rework" (mô tả lỗi + file đã sửa), giữ nguyên phần khác.
7. DỪNG LẠI, báo người dùng đã sửa file nào, tóm tắt thay đổi.

CHỈ dừng hỏi khi thật sự cần quyết định nghiệp vụ hoặc thiếu thông
tin.
---
