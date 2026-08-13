# ap-dung-menu-goi-thue-bao-20260807

## Yêu cầu gốc
> tôi cần thêm 1 cột nữa là áp dụng menu và không áp dụng menu. Nhưng gói thuê bao nào có áp dụng menu thì app mới check list menu để hiển thị.

## Repo cần chạm
- `hc_code_saas` (backend): thêm flag `apDungMenu` vào `DmKhac` (gói thuê bao), đọc flag để quyết định có lọc/hiển thị menu theo gói hay không.
- `hc_code_saas/deploy`: migration SQL thêm cột `ap_dung_menu`.
- `hc_code_frontend_saas` (web admin): màn hình `/hethong/goithuebao` — chỉ khi gói có `apDungMenu==1` mới gọi/đọc `thue_bao_chuc_nang` cho hiển thị menu.

## Phạm vi thay đổi
### Backend (hc_code_saas)
- `hc-saas/src/main/java/hc/saas/repository/entity/DmKhac.java`: thêm field `apDungMenu` (`Integer`, `@Column(name="ap_dung_menu")`).
- `hc-saas/src/main/java/hc/saas/service/impl/ThueBaoChucNangServiceImpl.java` (hoặc service cấp menu cho app): chỉ trả/lọc menu của gói khi `khac.apDungMenu == 1`; khi `apDungMenu == 0` thì bỏ qua danh sách menu gói (hành vi cụ thể theo quyết định nghiệp vụ — xem acceptance 5).
- Nơi cấp menu cho app (nếu là `api/saas/taikhoan/chucnang` hoặc tương đương): khi trả danh sách menu, kết hợp flag gói của tenant.

### Frontend (hc_code_frontend_saas)
- `app/shared/popup/them-sua-dm-khac/them-sua-dm-khac.component.html` + `.ts`: thêm trường chọn "Áp dụng menu" / "Không áp dụng menu" trong khối `chaMa === 'GOITHUEBAO'`, bind `apDungMenu` (mặc định `0`), gửi khi save.
- `app/entities/goi-thue-bao/goi-thue-bao.component.html` + `.ts`: thêm cột "Áp dụng menu" (Có/Không) trong danh sách gói.
- `app/shared/popup/cau-hinh-goi-thue-bao/cau-hinh-goi-thue-bao.component.ts` + `.html`: chỉ load/check danh sách menu khi `data.apDungMenu == 1`; khi `== 0` không hiện cây hoặc hiện cây trống/không lưu.

## Rủi ro / điểm cần lưu ý
- Không phá vỡ flow fix trước đó (UPSERT trangThai) của `ThueBaoChucNangServiceImpl`.
- Cột mới phải có `DEFAULT 0` để không ảnh hưởng dữ liệu cũ; migration dùng `ADD COLUMN IF NOT EXISTS` để an toàn.
- Quyết định nghiệp vụ cần chốt: khi `apDungMenu==0` thì gói hiển thị menu kiểu gì? (mặc định full theo quyền role, hay không khai thác menu gói).
- Chỉ SaaS quản lý menu app; không nhân bản bảng `dm_chucnang` vào `hc_code_app`.
- Naming: field Java camelCase; cột DB snake_case; viết code mới theo NAMING_CONVENTIONS.md; sửa file cũ giữ nguyên convention.

## Yêu cầu chi tiết cho /work
1. Backend: thêm `private Integer apDungMenu; @Column(name="ap_dung_menu")` vào `DmKhac`.
2. Migration `deploy/database/202608/<YYYYMMDD>_ddl_dm_khac_ap_dung_menu.sql`: `ALTER TABLE dm_khac ADD COLUMN IF NOT EXISTS ap_dung_menu INTEGER NOT NULL DEFAULT 0;` (kèm header mô tả). KHÔNG tự chạy DB.
3. Sửa logic cấp/lọc menu gói: khi `khac.apDungMenu==1` mới đọc `thue_bao_chuc_nang` ra list menu; khi `apDungMenu==0` trả theo quy tắc đã chốt.
4. FE SaaS: thêm field "Áp dụng menu" vào `them-sua-dm-khac` (select 0/1, label "Áp dụng menu" / "Không áp dụng menu"), hiển thị cột "Áp dụng menu" trong list `goi-thue-bao`; trong popup `cau-hinh-goi-thue-bao` chỉ chạy tree khi `apDungMenu==1`.

## Kết quả mong muốn (acceptance criteria)
1. Có cột mới `ap_dung_menu` trong DB `dm_khac` (mặc định 0) qua migration file.
2. Tạo/sửa gói thuê bao admin chọn được "Áp dụng menu" / "Không áp dụng menu" và lưu thành công (`code:"00"`).
3. List gói hiển thị cột "Áp dụng menu" = Có/Không đúng theo DB.
4. Gói `apDungMenu==1`: app đăng nhập lấy đúng menu từ `thue_bao_chuc_nang` và hiển thị theo gói + quyền.
5. Gói `apDungMenu==0`: app không lọc menu theo gói (theo quyết định đã chốt).
6. Lưu lại cấu hình cùng gói không lỗi unique constraint (kế thừa soft-delete có sẵn).

## Không làm trong scope
- Không chạy migration vào DB thật.
- Không merge branch.
- Không refactor toàn bộ RBAC/menu hiện tại.