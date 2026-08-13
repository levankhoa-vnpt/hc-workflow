# Progress: fix-goi-thue-bao-chuc-nang-add-20260806

status: waiting_review
next_action: user-review

## Spec
- .opencode/specs/fix-goi-thue-bao-chuc-nang-add-20260806.md

## Quy tắc của hệ thống (quan trọng)
Hệ thống KHÔNG xóa record vật lý; chỉ chuyển `trangthai = -1` để đánh dấu xóa (soft-delete).

## Root Cause (đã xác định lại)
`ThueBaoChucNangRepository.deleteByKhac(_Id)` là xóa VẬT LÝ — sai với quy tắc soft-delete của hệ thống. Khi lưu lại (re-save) các chức năng đã chọn trước đó, rows tồn tại từ lần trước vẫn còn → vi phạm unique constraint `uk_tbchucnang(khac_id, chucnang_id)`.

## Giải pháp: UPSERT + soft-delete (thay thế delete+insert)
- Không xóa record.
- Với mỗi chucNangId được gửi lên: nếu row đã tồn tại → cập nhật `khac`, `chucnang`, `trangthai=1` (KICH_HOAT) — KHÔNG insert mới → không vi phạm unique.
- Nếu chưa có → tạo mới `trangthai=1`.
- Row cũ KHÔNG nằm trong danh sách gửi lên → đặt `trangthai=-1` (soft-delete).
- Nếu `chucNangIds` rỗng → đặt `trangthai=-1` cho toàn bộ row của khac.

## Fix Applied
- `ThueBaoChucNang.java`: thêm trường `trangThai` (`@Column(name="trangthai")`).
- `ThueBaoChucNangRepository.java`: bỏ `deleteByKhac_Id`; thêm `getByKhac_IdAndTrangThaiGreaterThan(khacId, trangThai)`.
- `ThueBaoChucNangServiceImpl.java`: `add()` = UPSERT; `getChucNangByKhacId()` lọc `trangThai > XOA` (chỉ lấy row chưa bị xóa).
- `20260806_ddl_thue_bao_chuc_nang.sql`: thêm cột `trangthai INTEGER NOT NULL DEFAULT 1`.
- NEW `20260807_ddl_thue_bao_chuc_nang_trangthai.sql`: `ALTER TABLE... ADD COLUMN IF NOT EXISTS trangthai ...`.

## Branch
- Repo: hc_code_saas
- Branch hiện tại: feature/phan-quyen-menu-cho-goi-thue-bao

## Files touched
- hc-saas/src/main/java/hc/saas/repository/entity/ThueBaoChucNang.java
- hc-saas/src/main/java/hc/saas/repository/ThueBaoChucNangRepository.java
- hc-saas/src/main/java/hc/saas/service/impl/ThueBaoChucNangServiceImpl.java
- deploy/database/202608/20260806_ddl_thue_bao_chuc_nang.sql
- deploy/database/202608/20260807_ddl_thue_bao_chuc_nang_trangthai.sql (NEW)

## Build
- Java build (mvn compile): NOT VERIFIED (Maven không có trong environment).

## Test trên môi trường test
- Bị report lỗi: POST /api/saas/chucnang/goi-thue-bao/add với khacId=88 → code 101, constraint uk_thbchucucng.
- Sau khi fix + deploy: chạy lại add với cùng khacId để xác nhận không còn lỗi.