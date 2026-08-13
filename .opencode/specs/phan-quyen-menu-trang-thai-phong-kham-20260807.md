# phan-quyen-menu-trang-thai-phong-kham-20260807

## Yeu cau goc
>  tao chuc nang phan quyen menu theo trang thai phong kham. Trang thai lay tu dmkhac cha.ma='TRANG_THAI_PHONG_KHAM'. Mo phong kham co nhieu trang thai (1/2/3 + co the them moi). Lam giong /hethong/goithuebao.

## Repo can cham
- hc_code_saas (backend): entity, repo, service, controller cho mapping trang thai -> chuc nang. Migration SQL.
- hc_code_frontend_saas (web admin): man hinh danh sach trang thai + popup cau hinh.

## Pham vi thay doi

### Backend (hc_code_saas)
**New table: 	rang_thai_phong_kham_chuc_nang**
- khac_id INT FK dm_khac.id (trang thai from TRANG_THAI_PHONG_KHAM)
- chucnang_id INT FK dm_chucnang.id (menu function)
- 	rangthai INT DEFAULT 1 (soft-delete: 1=active, -1=deleted)
- Unique constraint: (khac_id, chucnang_id)

**New entity: TrangThaiPhongKhamChucNang**
- Extends BaseEntity
- Fields: khac (ManyToOne DmKhac), chucNang (ManyToOne DmChucNang), trangThai (Integer)
- Table: 	rang_thai_phong_kham_chuc_nang

**New repository: TrangThaiPhongKhamChucNangRepository**
- getByKhac_Id(Integer khacId) — all rows for a status
- getByKhac_IdAndTrangThaiGreaterThan(Integer khacId, Integer trangThai) — active only

**New service: TrangThaiPhongKhamChucNangService + impl**
- getChucNangByKhacId(Integer khacId) — returns list DmChucNang for a status
- dd(TrangThaiPhongKhamChucNangReq req) — UPSERT pattern (same as ThueBaoChucNangServiceImpl.add)

**New request DTO: TrangThaiPhongKhamChucNangReq**
- Integer khacId
- List<Integer> chucNangIds

**New controller endpoints on DmChucNangController:**
- GET /chucnang/trang-thai-phong-kham?khacId= -> getChucNangByKhacId
- POST /chucnang/trang-thai-phong-kham/add -> addChucNang

**Migration:** deploy/database/202608/20260807_ddl_trang_thai_phong_kham_chuc_nang.sql
- CREATE TABLE + unique constraint + indexes

### Frontend (hc_code_frontend_saas)

**New component: entities/trang-thai-phong-kham/trang-thai-phong-kham.component.ts/.html/.scss**
- Mirror of goi-thue-bao.component but filter dmkhac with cha.ma=="TRANG_THAI_PHONG_KHAM"
- Buttons: Them/Sua/Xoa + Cau Hinh (like goithuebao)
- Uses ThemSuaDMKhacComponent for add/edit with chaMa='TRANG_THAI_PHONG_KHAM'
- Opens CauHinhTrangThaiPhongKhamModalComponent for config

**New popup: shared/popup/cau-hinh-trang-thai-phong-kham/**
- Mirror of CauHinhGoiThueBaoModalComponent
- Calls pi/saas/chucnang/trang-thai-phong-kham?khacId= to load
- Calls pi/saas/chucnang/trang-thai-phong-kham/add to save
- Shows jqxTreeGrid of all chucnang with checkboxes

**Route registration: entities/dungChung.module.ts**
- Add route { path: 'trangthaiphongkham', component: TrangThaiPhongKhamComponent }

## Risk / Luu y
- UPSERT pattern identical to ThueBaoChucNangServiceImpl — safe.
- New table with DEFAULT trangthai=1 — no impact on existing data.
- Menu filtering at login (getChucNangByTaiKhoan) — NOT in this scope yet, depends on how clinic status is resolved from PhongKham.
- Khong tu chay DB.
- Khong merge branch.

## Acceptance Criteria
1. Hien thi danh sach trang thai phong kham tu dmkhac (cha.ma=TRANG_THAI_PHONG_KHAM) thanh cong.
2. Them sua xoa trang thai qua ThemSuaDMKhacComponent thanh cong.
3. Bam Cau Hinh -> thay cay chuc nang, check/uncheck, luu thanh cong.
4. Luu lai khong bi unique constraint.
5. Trang thai co chuc nang gan -> truy van GET tra ve danh sach dung.
