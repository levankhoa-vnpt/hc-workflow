---
name: respec
description: Chỉ dùng khi người dùng gọi rõ "$respec" hoặc gõ "/respec". Tiếp tục trao đổi để hoàn thiện spec đang dang dở — KHÔNG code.
---

ƯU TIÊN NGỮ CẢNH: nếu trong CHÍNH cuộc hội thoại này đã từng chạy
$spec và tạo ra 1 feature-id, dùng LUÔN feature-id đó và toàn bộ phân
tích đã có trong hội thoại — không cần đọc lại file _draft.md. Chỉ
đọc .opencode/specs/_draft.md khi cuộc hội thoại hiện tại KHÔNG có
ngữ cảnh $spec trước đó (ví dụ session mới mở lại).

Đây là skill TIẾP TỤC trao đổi để hoàn thiện spec, không phải phân
tích mới từ đầu. Bạn CHỈ được phép ghi .opencode/specs/_draft.md hoặc
(khi hoàn tất) .opencode/specs/<feature-id>.md +
.opencode/specs/CURRENT.md. Không tạo/sửa file nào khác, KHÔNG chạy
git, không code.

Phản hồi mới của người dùng dùng để bổ sung/làm rõ cho bối cảnh đã
có, KHÔNG bỏ qua phân tích cũ.

Áp dụng lại đúng bước 1, 2, 2b, 3 và rẽ nhánh 4 (NHÁNH A / NHÁNH B)
giống hệt logic của skill $spec — kết hợp bối cảnh cũ (từ _draft.md
hoặc hội thoại) với phản hồi mới để quyết định đã đủ thông tin chưa.
---
