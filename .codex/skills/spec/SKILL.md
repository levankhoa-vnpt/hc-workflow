---
name: spec
description: Chỉ dùng khi người dùng gọi rõ "$spec" hoặc gõ "/spec". Phân tích yêu cầu lần đầu cho 1 tính năng, ra file spec — KHÔNG code, KHÔNG chạy git.
---

Bạn đang ở chế độ PHÂN TÍCH. Bạn CHỈ được phép ghi 2 loại file:
.opencode/specs/_draft.md (nháp, khi còn thiếu thông tin) hoặc
.opencode/specs/<feature-id>.md + .opencode/specs/CURRENT.md (khi đã
đủ thông tin). Ngoài ra TUYỆT ĐỐI không tạo/sửa/xóa file nào khác,
KHÔNG chạy bất kỳ lệnh git nào, KHÔNG chạy mvn/npm, không code.

Nếu file .opencode/specs/_draft.md đã tồn tại, đọc nó trước để lấy
bối cảnh (yêu cầu gốc + câu hỏi trước đó) — coi đây là lần đầu phân
tích tính năng này (muốn tiếp tục trao đổi dở dang, dùng skill
$respec thay vì $spec).

Trước khi phân tích, đọc NAMING_CONVENTIONS.md và AGENTS.md.

1. Đọc AGENTS.md và bảng định tuyến để xác định yêu cầu liên quan tới
   (những) repo nào.
2. Đọc trực tiếp code hiện có liên quan để hiểu logic nghiệp vụ hiện
   tại, tránh đề xuất trùng lặp hoặc phá vỡ convention đang dùng.
2b. Nếu yêu cầu liên quan tới giao diện/luồng đang chạy và có
    chrome-devtools MCP khả dụng, mở môi trường test quan sát thực tế
    (đọc credential từ .opencode/test-env.local.md, nếu không có thì
    đọc .opencode/test-env.md lấy URL rồi hỏi người dùng). CHỈ XEM,
    không thao tác thay đổi dữ liệu thật.
3. Tự đánh giá: yêu cầu đã đủ CỤ THỂ và ĐO LƯỜNG ĐƯỢC chưa? Coi là
   CHƯA ĐỦ nếu: (a) nhiều cách hiểu về phạm vi, (b) không rõ loại việc
   (fix bug/thêm tính năng/sửa UI/tất cả), (c) thiếu tiêu chí hoàn
   thành đo lường được, (d) có quyết định nghiệp vụ chỉ người dùng
   mới trả lời được.

4. RẼ NHÁNH BẮT BUỘC — CHỌN MỘT, KHÔNG LÀM CẢ HAI:

   NHÁNH A — CHƯA ĐỦ: ghi/ghi đè .opencode/specs/_draft.md gồm: yêu
   cầu gốc, phân tích đã có, danh sách câu hỏi hiện tại. Rồi CHỈ xuất:
   ---
   SPEC CHƯA ĐỦ THÔNG TIN. Cần bạn trả lời (dùng $respec để tiếp tục):
   1. <câu hỏi>
   ---
   Dừng lại, chờ người dùng trả lời.

   NHÁNH B — ĐÃ ĐỦ: nếu .opencode/specs/_draft.md tồn tại, XÓA nó. Tự
   tạo feature-id kebab-case + ngày (vd: lock-tai-khoan-20260806, thêm
   hậu tố -2 nếu trùng). Ghi file .opencode/specs/<feature-id>.md:

   # <feature-id>

   ## Yêu cầu gốc
   <nguyên văn>

   ## Repo cần chạm
   <repo, lý do>

   ## Phạm vi thay đổi
   <file/module/class dự kiến>

   ## Rủi ro / điểm cần lưu ý
   <...>

   ## Yêu cầu chi tiết cho $work
   <mô tả đầy đủ, PHẢI là yêu cầu CODE thật sự>

   ## Kết quả mong muốn (acceptance criteria)
   <cụ thể, đo lường được>

   Ghi đè .opencode/specs/CURRENT.md với đúng feature-id (1 dòng,
   không kèm gì khác). CHỈ trả lời:
   ---
   SPEC ĐÃ SẴN SÀNG: .opencode/specs/<feature-id>.md
   Chạy tiếp: $work
   ---

   ĐẾN ĐÂY DỪNG HẲN. TUYỆT ĐỐI KHÔNG tự chuyển sang code dù yêu cầu có
   vẻ đơn giản/rõ ràng. Chỉ bắt đầu code khi người dùng tự gọi $work ở
   lượt riêng.
---
