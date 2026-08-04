# hc-workflow — Trung tâm điều phối OpenCode cho hệ thống HomeClinic

> Đây là "cửa duy nhất" để giao mọi việc code cho AI agent trong workspace
> 6 repo. Không mở session trực tiếp trong 5 repo kia — sẽ "mù" vì không
> có AGENTS.md/progress ở đó.

## 1. Cấu trúc workspace

```
C:\Users\Khoa\Documents\hc-workspace\
├── hc_code_frontend        (Angular 9 — web phòng khám + mobile PWA)
├── hc_code_frontend_saas   (Angular 9 — web SaaS admin + mobile PWA)
├── hc_code_saas             (Java Spring Boot — backend nền tảng SaaS)
├── hc_code_saas_job         (Java Spring Boot — worker job theo lịch)
├── hc-workflow              ← MỞ PROJECT NÀY, luôn mở ở đây
└── hc_code_app               (Java Spring Boot — backend monolith phòng khám)
```

Chi tiết công nghệ, cấu trúc, convention từng repo + bảng định tuyến
domain → repo: xem `AGENTS.md`.

## 2. Cách dùng hằng ngày

Mở session OpenCode tại `hc-workflow`, dùng 2 lệnh:

### Bước A — `/spec` (phân tích yêu cầu, không code)

```
/spec <mô tả yêu cầu thô, chưa cần chi tiết kỹ thuật>
```

Agent sẽ:
- Đọc `AGENTS.md` + code liên quan trong (các) repo phù hợp
- **Không tạo/sửa file nào** — chỉ đọc và phân tích
- Hỏi lại nếu yêu cầu thiếu thông tin nghiệp vụ hoặc mơ hồ
- Xuất ra: repo cần chạm, phạm vi thay đổi, rủi ro, và **1 khối lệnh
  `/work` hoàn chỉnh** sẵn sàng để chạy tiếp

### Bước B — `/work` (thực thi)

Copy nguyên khối `/work ...` mà `/spec` xuất ra, dán vào (cùng session
hoặc session mới đều được):

```
/work <mô tả yêu cầu đã làm rõ>
Kết quả mong muốn: <acceptance criteria cụ thể, đo lường được>
```

Agent sẽ tự động: phân tích → tạo branch → code → build/test → tự
review → lặp lại đến khi đạt, rồi dừng đúng ở các điểm cần bạn duyệt.

## 3. Các điểm DỪNG bắt buộc (gate) — agent KHÔNG tự vượt qua

| Điểm dừng | Ai làm tiếp |
|---|---|
| Trước `git push` | Bạn duyệt lệnh push |
| Sau khi tạo Pull Request | Bạn tự **merge** PR (agent không bao giờ tự merge) |
| SQL migration (nếu có thay đổi schema) | Agent chỉ tạo file `.sql`, **bạn tự chạy** vào database |
| Sau khi bạn merge + deploy xong | Gõ `continue` để agent tự kiểm tra trên môi trường test qua Chrome DevTools MCP |

## 4. Khi bị gián đoạn (hết quota, đổi session/máy/tài khoản)

Không cần kể lại từ đầu. Ở agent/session mới, mở lại `hc-workflow` và gõ:

```
/work tiếp tục <tên feature>
```

Agent sẽ tự đọc `.opencode/progress/<feature-slug>.md` để biết đã làm
gì, đang ở bước nào, việc tiếp theo là gì.

## 5. Các ràng buộc quan trọng cần nhớ

- **Không tạo/sửa file trong 5 repo kia** — mọi cấu hình, AGENTS.md,
  progress file chỉ đặt trong `hc-workflow`.
- **Agent không tự merge code** vào bất kỳ nhánh nào (dev/release/test).
- **Agent không tự chạy SQL migration vào database thật** — chỉ tạo
  file migration, người dùng tự chạy tay.
- Deploy do CI/CD tự lo, agent chỉ chờ rồi tự kiểm tra kết quả sau khi
  deploy xong.
- `.opencode/progress/<feature-slug>.md` là nguồn sự thật duy nhất cho
  việc bàn giao — không dựa vào "trí nhớ" riêng của 1 agent/model.

## 6. Cấu hình đã thiết lập (`opencode.json`)

- `permission.external_directory`: cho phép session ở `hc-workflow` đọc/ghi
  sang cả 5 repo còn lại mà không cần đặt file gì bên trong chúng.
- `permission.bash`: `git push`, `git merge`, và các lệnh chạm database
  (`flyway`, `liquibase`, `psql`, `mysql`) đều yêu cầu duyệt tay (`ask`).
- `mcp.chrome-devtools`: agent tự đọc console/network lỗi trên môi
  trường test thật sau khi deploy.
- `command.spec`: lệnh phân tích yêu cầu, không code.
- `command.work`: lệnh thực thi toàn bộ vòng đời tính năng.

## 7. Công cụ bổ trợ (cân nhắc sau, không bắt buộc)

| Công cụ | Vai trò | Khi nào cần |
|---|---|---|
| Serena (MCP) | Hiểu code ở mức symbol/LSP, giảm token | Khi agent tốn nhiều lượt "dò tìm" code |
| Playwright | Agent tự viết & chạy e2e test | Khi cần agent tự tạo bộ test |
| Operator / Vibe Kanban | Dashboard điều phối nhiều agent song song | Khi mở rộng chạy nhiều agent song song thật sự |