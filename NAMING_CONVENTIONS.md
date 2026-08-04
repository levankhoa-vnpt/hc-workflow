# Quy tắc đặt tên chuẩn — hc-workspace

> Dựa trên: Google Java Style Guide, Angular Style Guide (angular.io),
> Microsoft REST API Guidelines, PostgreSQL/SQL naming convention.
>
> **QUAN TRỌNG:** code hiện tại (`hc_code_app`, `hc_code_saas`...) dùng
> convention riêng (domain tiếng Việt, prefix Dm/Kb/Vt...). Chuẩn dưới
> đây áp dụng NGHIÊM NGẶT cho code MỚI. KHÔNG refactor code cũ theo
> chuẩn mới trừ khi được yêu cầu rõ ràng — tránh phá vỡ hệ thống đang
> chạy production.

## 1. Java (Backend) — theo Google Java Style Guide

| Đối tượng | Convention | Ví dụ |
|---|---|---|
| Class, Interface | `PascalCase`, danh từ | `PatientRecordService`, `OrderRepository` |
| Method | `camelCase`, động từ mở đầu | `calculateTotalCost()`, `isEligibleForDiscount()` |
| Variable, field | `camelCase`, danh từ mô tả rõ nghĩa | `totalAmount`, `patientList` (không viết tắt `pList`) |
| Constant (`static final`) | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT`, `DEFAULT_PAGE_SIZE` |
| Package | toàn bộ chữ thường, không dấu gạch dưới | `com.company.patient.service` |
| Boolean variable/method | tiền tố `is/has/can/should` | `isActive`, `hasPermission()`, `canDelete()` |
| Generic type | 1 chữ hoa | `<T>`, `<E>`, `<K, V>` |
| Test class | tên class + hậu tố `Test` | `PatientServiceTest` |

## 2. TypeScript / Angular (Frontend) — theo Angular Style Guide chính thức

| Đối tượng | Convention | Ví dụ |
|---|---|---|
| File | `kebab-case.loại.ts` | `patient-list.component.ts`, `auth.service.ts` |
| Class (Component/Service/...) | `PascalCase` + hậu tố loại | `PatientListComponent`, `AuthService` |
| Component selector | `kebab-case`, có prefix riêng dự án | `app-patient-list` |
| Interface | `PascalCase`, KHÔNG tiền tố `I` | `Patient` (không `IPatient`) |
| Method, variable | `camelCase` | `loadPatientList()`, `isLoading` |
| Observable variable | hậu tố `$` | `patients$`, `loading$` |
| Constant module-level | `UPPER_SNAKE_CASE` | `API_BASE_URL` |
| Enum | `PascalCase` tên, `UPPER_SNAKE_CASE` hoặc `PascalCase` giá trị | `enum Status { ACTIVE, INACTIVE }` |

## 3. REST API — theo Microsoft REST API Guidelines / Google API Design Guide

| Quy tắc | Chuẩn | Ví dụ |
|---|---|---|
| URL path | danh từ số nhiều, `kebab-case`, tiếng Anh | `/patients`, `/medical-records` |
| Hành động qua HTTP verb, KHÔNG nhét action vào path | `POST /patients` (tạo), `PUT /patients/{id}` (sửa), `DELETE /patients/{id}` (xóa) — thay vì `/create`, `/update` | |
| Nested resource | thể hiện quan hệ cha-con | `/patients/{id}/prescriptions` |
| Query param filter/sort/paginate | `camelCase` | `?sortBy=createdAt&pageSize=20` |
| Response field | `camelCase`, nhất quán 1 kiểu duy nhất | `{"patientId": 1, "fullName": "..."}` |
| Versioning | tiền tố path hoặc header | `/api/v1/patients` |
| HTTP status code | dùng đúng nghĩa chuẩn, không tự chế | `200/201/400/401/403/404/409/500` |

## 4. Database (PostgreSQL) — snake_case convention

| Đối tượng | Convention | Ví dụ |
|---|---|---|
| Table | `snake_case`, số nhiều | `patients`, `medical_records` |
| Column | `snake_case` | `full_name`, `created_at`, `is_active` |
| Primary key | `id` | |
| Foreign key | `<singular_table>_id` | `patient_id` |
| Index | `idx_<table>_<column(s)>` | `idx_patients_email` |
| Junction/many-to-many table | `<table1>_<table2>` | `patients_doctors` |
| Boolean column | tiền tố `is_/has_` | `is_active`, `has_insurance` |
| Timestamp column | hậu tố `_at` | `created_at`, `updated_at`, `deleted_at` |
| Enum-like status | dùng bảng riêng hoặc CHECK constraint, không magic number | |

## 5. Git — theo Conventional Commits

| Đối tượng | Convention | Ví dụ |
|---|---|---|
| Branch | `<type>/<slug-kebab-case>` | `feature/patient-record-lock`, `fix/login-2fa-timeout` |
| Commit message | `<type>: <mô tả ngắn, thì hiện tại>` | `feat: add healthcheck endpoint`, `fix: correct null pointer in order service` |
| Type hợp lệ | `feat`, `fix`, `refactor`, `test`, `docs`, `chore` | |

## 6. Nguyên tắc chung (áp dụng mọi ngôn ngữ)

- Tên phải tự giải thích được — đọc tên là hiểu, không cần đọc thêm comment.
- Không viết tắt tùy tiện (`usr` → `user`, `calc` → `calculate`), trừ các viết tắt chuẩn ngành đã quen thuộc (`id`, `url`, `http`).
- Nhất quán 1 thuật ngữ cho 1 khái niệm trong toàn bộ codebase (không lúc `patient` lúc `client` cho cùng 1 nghĩa).
- Độ dài tên tỷ lệ với phạm vi sử dụng: biến local ngắn (vòng lặp `i`, `j` chấp nhận được), biến/method cấp module/class đặt tên đầy đủ rõ nghĩa.
- Tên không mang thông tin kiểu dữ liệu (không đặt `patientListArray`, chỉ cần `patients`).

## 7. Áp dụng cho agent (`/spec`, `/work`)

- Khi tạo **code mới** (file mới, class mới, endpoint mới, bảng mới):
  bắt buộc tuân theo chuẩn ở trên.
- Khi **sửa code cũ** đã tồn tại: giữ nguyên convention hiện có của
  vùng code đó (domain tiếng Việt, prefix Dm/Kb/Vt, REST kebab-case
  tiếng Việt...) — KHÔNG tự ý đổi tên biến/method/bảng đã có trừ khi
  người dùng yêu cầu rõ ràng "refactor theo chuẩn mới".
- Nếu không chắc 1 đoạn code thuộc diện "mới" hay "cũ cần giữ nguyên",
  hỏi lại người dùng trước khi đặt tên.