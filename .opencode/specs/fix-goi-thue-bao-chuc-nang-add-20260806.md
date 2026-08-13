# fix-goi-thue-bao-chuc-nang-add-20260806

## Yêu cầu gốc
API POST https://hcgw-test.vncare.vn/api/saas/chucnang/goi-thue-bao/add đang báo lỗi khi lưu lại các chức năng đã lưu trước đó (lưu lần 2 trở đi trên cùng một gói thuê bao).

## Repo cần chạm
- hc_code_saas (BE) — endpoint `/goi-thue-bao/add` thuộc `DmChucNangController`, service `ThueBaoChucNangServiceImpl.add()`, repository `ThueBaoChucNangRepository`.
- hc_code_frontend_saas (FE) — popup `CauHinhGoiThueBaoModalComponent` gọi API này khi bấm Lưu.

## Phạm vi thay đổi
- Backend: `ThueBaoChucNangServiceImpl.add()` — phương thức xóa chức năng cũ trước khi thêm mới có thể không xóa đúng, gây vi phạm unique constraint `uk_tbchucnang(khac_id, chucnang_id)` khi lưu lại.
- Backend: `ThueBaoChucNangRepository.deleteByKhac(DmKhac khac)` — cần kiểm tra implementation Spring Data JPA có hoạt động đúng với entity proxy hay không.
- FE: `cau-hinh-goi-thue-bao.component.ts` — phương thức `save()` gọi POST `/goi-thue-bao/add`.

## Rủi ro / điểm cần lưu ý
- Unique constraint `uk_tbchucnang` trên bảng `thue_bao_chuc_nang` (khac_id, chucnang_id) có thể bị vi phạm nếu `deleteByKhac` không xóa hết bản ghi cũ trước khi insert.
- `deleteByKhac(DmKhac khac)` dùng entity proxy từ `khacService.getById()` — cần verify Spring Data JPA derive query đúng với parameter type `DmKhac` (không phải `Integer khacId`).
- Transaction `@Transactional` trên `add()` đảm bảo rollback nếu lỗi, nhưng cần kiểm tra error message trả về FE có rõ ràng không.

## Yêu cầu chi tiết cho /work
1. Đọc logs lỗi thực tế từ API `POST /api/saas/chucnang/goi-thue-bao/add` khi gọi lần 2 trên cùng gói thuê bao để xác định chính xác lỗi (unique constraint violation, null pointer, hay khác).
2. Nếu lỗi là unique constraint: sửa `deleteByKhac` trong `ThueBaoChucNangRepository` để đảm bảo xóa đúng tất cả bản ghi cũ trước khi insert mới. Có thể cần đổi parameter từ `DmKhac` sang `Integer khacId` hoặc dùng `@Query` annotation.
3. Nếu lỗi là khác: phân tích và fix theo đúng nguyên nhân.
4. Verify bằng cách: gọi API lưu chức năng cho 1 gói thuê bao → gọi lại lần 2 → xác nhận không còn lỗi và dữ liệu được cập nhật đúng.
5. FE: đảm bảo popup hiển thị thông báo lỗi rõ ràng nếu API trả về lỗi, và reload danh sách chức năng sau khi lưu thành công.

## Kết quả mong muốn (acceptance criteria)
1. Gọi `POST /api/saas/chucnang/goi-thue-bao/add` lần 1 cho 1 gói thuê bao → thành công, code "0".
2. Gọi lại `POST /api/saas/chucnang/goi-thue-bao/add` lần 2 cho cùng gói thuê bao với cùng hoặc khác danh sách chức năng → thành công, code "0", không còn lỗi unique constraint.
3. Dữ liệu trong bảng `thue_bao_chuc_nang` phản ánh đúng danh sách chức năng đã lưu lần cuối (không còn bản ghi cũ bị trùng).
4. FE popup hiển thị thông báo thành công hoặc lỗi rõ ràng sau khi gọi API.