# Progress: phan-quyen-goi-thue-bao-form

status: in_progress
next_action: create-goi-thue-bao-fe-form

## Scope
- Repo: hc_code_saas (BE, login menu already intersects Role∩Package — no change) + hc_code_frontend_saas (new admin form).
- KHÔNG chạm hc_code_app.

## Findings (code đã có)
- Login menu restriction: DmTaiKhoanServiceImpl.getChucNangByTaiKhoan (DmTaiKhoanServiceImpl.java:303-344) = roleMenuCodes ∩ packageMenuCodes.
- Package codes source: DonHangServiceImpl.getPackageCodesByTaiKhoan (DonHangServiceImpl.java:1187) reads DonHang.maDsgoiSanluong (split ';'); ma_goi in quyen_goi_chuc_nang references DmKhac.ma (PK_MAU_COBAN...). FE chi-tiet-don-hang bound maDsgoiSanluong to DmKhac.ma (chi-tiet-don-hang.component.ts:167 bindValue=ma) -> nhất quán.
- Admin gán menu↔gói: CauHinhPhanQuyenGoiChucNangModal (GET /phan-quyen-goi-chuc-nang/chucnang?maGoi=, POST /add/chucnang) + ThemSuaPhanQuyenGoiChucNangModal (create/edit DmKhac package). Both already declared in EmrssoSharedModule.
- Nguồn gói: DmKhac cha.ma=="GOITHUEBAO" (dùng ở chi-tiet-don-hang.component.ts:190).

## Decisions (from user)
- gói rỗng = không hợp lệ (không cần fallback role-only).
- Tạo form Danh sách gói thuê bao riêng, tận dụng DmKhac cha.ma==GOITHUEBAO, copy pattern vai-tro.

## Work steps
- [ ] FE: entities/goi-thue-bao/*.component.{ts,html,scss} (copy vai-tro pattern; REQUEST_URL=api/saas/khac; default filter cha.ma=="GOITHUEBAO"; Thêm/Sửa->ThemSuaPhanQuyenGoiChucNangModal; Cấu hình->CauHinhPhanQuyenGoiChucNangModal).
- [ ] FE: register route path:goithuebao + declarations in dungChung.module.ts.
- [ ] DDL/DML: seed SQL migration (deploy/database/202608) — insert DmChucNang(ma=goithuebao,heThong=SAAS) + gán role admin (dm_vaitro_chucnang) để nav hiện trên sidebar. CHỈ tạo file, KHÔNG chạy vào DB.
- [ ] FE build check (npm run build).
- [ ] commit + push + tạo PR (KHÔNG merge).
- [] Dừng, báo user chạy migration thủ công + merge PR.

## Files touched
- (pending)

## SQL migration (cần user chạy tay trước deploy)
- (pending file name)
