# Progress: fix-quyen-goi-chucnang-in-query-20260807

status: waiting_review
next_action: user-review

## Bug
`DmTaiKhoanServiceImpl.getChucNangByTaiKhoan()` báo lỗi tại
`packageMenuCodes.addAll(phanQuyenGoiChucNangRepository.findMaChucNangByMaGoiIn(packageCodes));`.

## Root cause
Native query thiếu ngoặc trong mệnh đề IN — PostgreSQL không chấp nhận `IN :maGoi`,
phải là `IN (:maGoi)`. JPA/Spring chỉ bind List param khi có ngoặc.

## Fix
- `hc-saas/src/main/java/hc/saas/repository/PhanQuyenGoiChucNangRepository.java`:
  `WHERE ma_goi IN :maGoi AND trangthai = 1` → `WHERE ma_goi IN (:maGoi) AND trangthai = 1`.

## Branch (chưa commit)
- hc_code_saas: fix/quyen-goi-chucnang-in-query-20260807

## Build
- Java/Maven không có trong environment → chưa verify compile (đổi 1 chuỗi SQL, không ảnh hưởng Java).

## Note
- `findMaChucNangByMaGoi` (không IN) dùng `= :maGoi` → không bị lỗi, không đổi.
- Lỗi này đang bị catch-silent trong `getChucNangByTaiKhoan` (`// ignore`) nên menu có thể bị trống thay vì crash.