# Progress: phan-quyen-menu-trang-thai-phong-kham-20260807

status: waiting_review
next_action: user-review

## Spec
- .opencode/specs/phan-quyen-menu-trang-thai-phong-kham-20260807.md

## Thanh phan da xong
- Backend: new entity/repository/service/controller (ThueBaoChucNang pattern)
- Migration: 20260807_ddl_trang_thai_phong_kham_chuc_nang.sql
- Frontend list: trang-thai-phong-kham (dm_khac cha.ma='TRANG_THAI_PHONG_KHAM')
- Frontend popup: cau-hinh-trang-thai-phong-kham
- Route: dungChung.module.ts + shared.module.ts
- Build FE: PASS (npx tsc --noEmit)
- Build BE: chua verify (Maven khong co)

## Chua lam (scope 2)
- Menu filtering: ghep vao getChucNangByTaiKhoan (phu thuoc cach resolve trang thai PhongKham -> dm_khac)
- Seed data dm_khac cha TRANG_THAI_PHONG_KHAM (chua thay du lieu)

## Branch
- hc_code_saas: fix/quyen-goi-chucnang-in-query-20260807
- hc_code_frontend_saas: feature/phan-quyen-menu-cho-goi-thue-bao
