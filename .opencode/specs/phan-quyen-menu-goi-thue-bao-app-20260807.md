# phan-quyen-menu-goi-thue-bao-app-20260807

## Mục tiêu
Phân quyền menu app theo gói thuê bao phòng khám, để mỗi tenant chỉ thấy menu/chức năng được cấu hình cho gói đang thuê.

## Repo tác động
- `hc_code_app/hc-common`: backend app đọc/lọc quyền menu theo gói thuê bao.
- `hc_code_frontend/web`: frontend app dùng danh sách menu đã lọc, bổ sung màn hình cấu hình nếu cần.

## Tham khảo SaaS
- Route FE SaaS: `/hethong/goithuebao`.
- Module FE SaaS: `hc_code_frontend_saas/web/src/main/webapp/app/entities/dungChung.module.ts`, path `goithuebao`.
- Màn danh sách gói: `hc_code_frontend_saas/web/src/main/webapp/app/entities/goi-thue-bao/`.
- Popup cấu hình: `hc_code_frontend_saas/web/src/main/webapp/app/shared/popup/cau-hinh-goi-thue-bao/`.
- API SaaS tham khảo:
  - `GET /api/saas/chucnang/goi-thue-bao?khacId=<id>` lấy menu đã gán cho gói.
  - `POST /api/saas/chucnang/goi-thue-bao/add` lưu menu cho gói.
- BE SaaS tham khảo:
  - `DmChucNangController#getChucNangByKhac`
  - `DmChucNangController#addChucNangChoGoiThueBao`
  - `ThueBaoChucNangServiceImpl`
  - `ThueBaoChucNangRepository`
  - `AddThueBaoChucNangReq`
  - `ThueBaoChucNang`
- Migration SaaS tham khảo: `hc_code_saas/deploy/database/202608/20260806_ddl_thue_bao_chuc_nang.sql`.

## Hiện trạng app
- App FE chưa có route `/hethong/goithuebao`.
- App FE sidebar đọc menu từ localStorage key `chucnang`: `hc_code_frontend/web/src/main/webapp/app/layouts/sidebar/sidebar.component.ts`.
- App FE login đang gọi SaaS để lấy menu account: `GET api/saas/taikhoan/chucnang?taiKhoanId=<id>` trong `hc_code_frontend/web/src/main/webapp/app/core/auth/auth-jwt.service.ts`.
- App FE đã hiển thị thông tin gói thuê bao từ `info.loaiThueBao`: `hc_code_frontend/web/src/main/webapp/app/layouts/header/header.component.html`.
- App BE không thấy entity/controller `DmChucNang`; menu chính đang nằm phía SaaS, app BE chủ yếu có `DmSubMenuNv` cho menu nghiệp vụ nội bộ.

## Yêu cầu chức năng
1. Cho phép admin cấu hình danh sách menu app được dùng bởi từng gói thuê bao.
2. Khi user app đăng nhập, danh sách menu trả về phải được lọc theo:
   - quyền account/role hiện tại;
   - gói thuê bao hiện tại của tenant/phòng khám;
   - chỉ lấy menu hệ thống app (`heThong = COMMON` theo sidebar app hiện tại).
3. Route màn hình cấu hình trên app phải dùng đúng đường dẫn `/hethong/goithuebao` để đồng bộ với SaaS.
4. Sidebar app không cần tự lọc thêm theo gói nếu BE/SaaS đã trả menu đúng; tránh duplicate rule trên FE.

## Thiết kế đề xuất
### Backend
- Ưu tiên sửa nguồn API hiện đang cấp menu cho app: `api/saas/taikhoan/chucnang` nếu menu app được quản lý tại SaaS.
- Nếu bắt buộc tác động `hc_code_app`, tạo endpoint proxy/read-only tối thiểu trong app BE để lấy menu gói từ SaaS, không nhân bản bảng `dm_chucnang` vào app vì app BE hiện không có model này.
- Bảng mapping nên reuse thiết kế SaaS: `thue_bao_chuc_nang(khac_id, chucnang_id)` với unique `(khac_id, chucnang_id)`.
- Lưu danh sách menu bằng delete-by-`khacId` rồi insert danh sách mới, giống fix đã làm ở SaaS: `deleteByKhac_Id(Integer khacId)`.

### Frontend
- Copy tối thiểu flow SaaS sang app FE:
  - route `hethong/goithuebao` trong `HeThongModule`;
  - component danh sách gói thuê bao;
  - popup cây menu cấu hình gói thuê bao.
- API popup gọi đúng endpoint đang quản lý menu gói:
  - nếu dùng SaaS trực tiếp: `api/saas/chucnang/goi-thue-bao` và `/goi-thue-bao/add`;
  - nếu app BE có proxy: dùng endpoint app BE tương ứng.
- Tree menu phải lấy danh sách `DmChucNang` hệ app, không lấy nhầm menu SaaS admin. Filter cần có `heThong==COMMON` hoặc rule tương đương.

## Acceptance Criteria
1. Admin mở được `/hethong/goithuebao` trên app FE.
2. Admin chọn một gói thuê bao và thấy cây menu app đúng với menu sidebar app.
3. Lưu menu cho gói lần 1 thành công.
4. Lưu lại cùng gói lần 2 không lỗi unique constraint.
5. User thuộc tenant gói A đăng nhập chỉ thấy menu được gán cho gói A và quyền role/user của họ.
6. Đổi cấu hình gói rồi đăng nhập lại, sidebar phản ánh danh sách menu mới.

## Không làm trong scope này
- Không tự chạy SQL migration vào database thật.
- Không merge branch.
- Không refactor toàn bộ RBAC/menu hiện tại.
- Không thêm dependency FE/BE mới.

## Câu hỏi cần chốt trước khi code
1. Nguồn truth menu gói thuê bao nằm ở SaaS hay cần copy sang app DB tenant?
2. API `api/saas/taikhoan/chucnang` có được phép sửa để lọc theo `loaiThueBao` không?
3. Menu cấu hình cho gói thuê bao app lấy toàn bộ `DmChucNang` `heThong=COMMON` hay còn module khác ngoài `COMMON`?
