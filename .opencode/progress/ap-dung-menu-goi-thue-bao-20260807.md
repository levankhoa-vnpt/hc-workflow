# Progress: ap-dung-menu-goi-thue-bao-20260807

status: waiting_review
next_action: user-review

## Yêu cầu đã chốt (OPTION 1 - intersection)
Menu cuối của user app login (`api/saas/tai-khoan/chuc-nang`):
- Gói (`dm_khac`) có `apDungMenu == 1` → menu = **menu vai trò (chucNangTheoVaiTro) ∩ menu gói (thue_bao_chuc_nang)**.
- `apDungMenu == 0` hoặc không có gói/phòng khám → trả **nguyên** menu theo vai trò (như cũ).
- Làm trên **code working tree hiện tại** (branch `feature/phan-quyen-menu-cho-goi-thue-bao`), KHÔNG đụng git.

## File đã đổi
### Backend (hc_code_saas) — repo `hc-saas`
1. `repository/entity/DmKhac.java`: thêm field `apDungMenu` (`@Column(name="ap_dung_menu") Integer`).
2. `service/impl/DmTaiKhoanServiceImpl.java`:
   - Inject `@Autowired ThueBaoChucNangService thueBaoChucNangService;` (đã có qua wildcard import).
   - Rewrite `getChucNangByTaiKhoan`: lấy `chucNucAOTheoVaiTro`, gọi `filterChucNangByGoiThueBao`.
   - Thêm `private List<DmChucNang> filterChucNangByGoiTrueBao(...)`: gate `maPhongKham`/`donHang`/`apDungMenu==KICH_HOAT`/rỗng theo option, rồi filter `chucNangTheoGoiIds.contains(e.getId())`.
- Migration mới `deploy/database/202608/20260807_ddl_dm_khac_ap_dung_menu.sql` (`ADD COLUMN IF NOT EXISTS ap_dung_menu INTEGER NOT NULL DEFAULT 0`). KHÔNG tự chạy DB.

### Frontend (hc_code_frontend_sass) — BỔ SUNG 2026-08-10 (sort danh sách)
- `entities/goi-thue-bao/goi-thue-bao.component.ts`:
  - Thêm `FtType = true; FtSapXep = 'ngayNhap';` (default sort ngày tạo desc).
  - `loadData()`: `sort: [this.FtSapXep, this.FtType ? 'desc' : 'asc']` (trước: `['stt','asc']`).
  - Thêm `sortData(e)` (toggle asc/desc + reload) — convention cùng `quan-ly-hop-dong`.
- `entities/goi-thue-bao/goi-thue-bao.component.html`:
  - Cột có nút sort (click header + icon fa-sort): Mã, Tên, Giá trị, Số tháng CT, Số tháng KM, Số ngày SL, Áp dụng menu, Ngày tạo (lúc nào cũng bấm được, active mầu khi FtSapXep match).
  - Thêm cột mới "Ngày tạo" hiển thị `entity.ngayNhap` (BE: BaseDmEntity.ngayNhap = `ngay_tao`).
  - tsc --noEmit PASS.

### Frontend (hc_code_frontend_sass) — BỔ SUNG 2026-08-10 (fix ẩn dropdown khi EDIT)
- `shared/popup/them-sua-dm-khac/them-sua-dm-khac.component.ts` `onSelect()` dòng 97:
  `this.chaMa = index.chaMa || (index.cha && index.cha.ma) || '';`
  - Trước: đọc `index.chaMa` (row `api/saas/khac` không có field `chaMa`) → `chaMa=''` → khối GOITHUEBAO (dropdown "Áp dụng menu") ẩn trên EDIT.
  - Nay: fallback `index.cha.ma` (='GOITHUEBAO' với mọi gói) → dropdown hiển thị trên EDIT; ADD vẫn chạy (onCreate set trực tiếp).
  - Giữ nguyên dropdown (theo quyết định user), tsc --noEmit PASS.

### Frontend (hc_code_frontend_saas) — đã sẵn có trong working tree
- `them-sua-dm-khac.component.ts/.html`: select "Áp dụng menu", bind + gửi `apDungMenu`.
- `goi-thue-bao.component.html`: cột "Áp dụng menu" (Có/Không).
- `cau-hinh-goi-thue-bao.component.ts/.html`: gate khi `apDungMenu==0` không load/lưu tree.

## Build
- FE (2026-08-10): `tsc --noEmit -p web\tsconfig.json` → exit 0 PASS.
- BE: không có Java/Maven trên máy → chưa verify compile. Code bám sát bản đã chạy trên `release/test` (commit `c11b4df`), all phụ thuộc xác nhận có trong working tree.

## Acceptance mapping
1. Migration file có cột ap_dung_menu default 0 ✓
2. Gói `apDungMenu==1` → menu vai tróc ∩ menu gói ✓
3. Gói `apDungGui==0`/không có → nguyên menu vai tróc ✓
4. Không phá flow admin / role ✓

## Note
- Working tree hiện TẠI branch `feature/phan-quy-menu-cho-goi-thue-bao` — code working tree này CHƯA CÓ merge logic trước đó (nằm ở `release/test`); feature này bổ sung đầy đủ vào nhánh đang đứng.
- Theo yêu cầu KHÔNG thao tác git (không branch mới, không commit).