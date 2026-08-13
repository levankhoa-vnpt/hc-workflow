# fix-ke-don-focus-va-tiep-nhan-can-nang-20260812

## Bằng chứng / Ảnh màn hình
- Ảnh kèm theo thể hiện màn hình kê đơn: dòng thuốc Gapivell 10mg, các trường lần lượt "Đường dùng, Số ngày, Sáng, Trưa, Chiều, Tối, Số lượng, Cách dùng"; chú thích đỏ: "Khi chọn thuốc chuột sẽ click vào ô đường dùng".
- Khối "Chỉ số"/cân nặng ở vùng tiếp đón trống dù BN đã từng khám có nhập.
- Mapping focus đã verify trong HTML (ke-don-thuoc-2.component.html): `focusTab3Id2` = Đường dùng, `focusTab3Id3` = Số ngày, `focusTab3Id4` = Sáng, `focusTab3Id5` = Trưa, `focusTab3Id6` = Chiều, `focusTab3Id7` = Tối, `focusTab3Id8` = Số lượng, `focusTab3Id9` = Cách dùng.
- Root cause: các nhánh trong `onOpenSoTay()` gọi `onNextInput(2)` → focus vào `focusTab3Id(2+1)` = `focusTab3Id3` = "Số ngày". Cần focus thẳng `focusTab3Id2` (Đường dùng).

## Yêu cầu gốc
1. Kê đơn: sau khi chọn thuốc xong, focus tự nhảy đến ô "Số ngày" thay vì "Đường dùng". Cần sửa để focus nhảy vào "Đường dùng".
2. Tiếp nhận BN cũ: khi tái tiếp nhận bệnh nhân cũ, form tiếp đón không tự lấy cân nặng (và các chỉ số sinh tồn khác) từ lần khám trước.

## Repo cần chạm
- **hc_code_frontend (FE)** — `ke-don-thuoc-2.component.ts` (fix focus kê đơn), `dang-ky-kham.component.ts` (load chiSo BN cũ)
- **hc_code_app (BE)** — `KbChiSoService` / `KbChiSoRepository` / `KbChiSoController` (thêm API lấy chiSo mới nhất theo benhNhanId)

## Phạm vi thay đổi

### 1. Kê đơn: Focus vào "Đường dùng" sau khi chọn thuốc

**Files:**
- `hc_code_frontend/.../shared/components/ke-don-thuoc-2/ke-don-thuoc-2.component.ts`

**Hiện trạng:** Khi chọn thuốc (hoặc popup sổ tay xong / dismiss / không có sổ tay), code gọi `onNextInput(2)` → nhảy đến `focusTab3Id3` = "Số ngày".

**Yêu cầu:** Sau khi chọn thuốc xong (tất cả nhánh trong `onOpenSoTay`), focus vào `focusTab3Id2` = "Đường dùng" thay vì "Số ngày".

**Chi tiết:**
- Trong `onOpenSoTay()`, ở các nhánh gọi `onNextInput(2)` (dismiss sổ tay, 0 records, error) → thay bằng focus vào `focusTab3Id2` trực tiếp.
- Nhánh 1 record sổ tay: sau `layGiaTriMacDinhTheoCauHinh()`, thêm `setTimeout(() => document.getElementById('focusTab3Id2')?.focus(), 100)`.
- Nhánh >1 records (result callback): sau khi set giá trị, thêm focus vào `focusTab3Id2`.
- Nhánh >1 records (dismiss): đã gọi `onNextInput(2)` → thay bằng focus `focusTab3Id2`.

### 2. Tiếp nhận BN cũ: Lấy cân nặng từ lần khám trước

**Files (BE):**
- `KbChiSoRepository.java` — thêm method `findFirstByBenhNhanIdOrderByIdDesc(Integer benhNhanId)`
- `KbChiSoService.java` — thêm `KbChiSo getLatestByBenhNhanId(Integer benhNhanId)`
- `KbChiSoServiceImpl.java` — implement
- `KbChiSoController.java` — thêm endpoint `GET /chiso/bybenhnhan?benhNhanId=...`

**Files (FE):**
- `dang-ky-kham.component.ts` — trong `loadDetailsBN()`, sau khi load xong benhNhanEntity, gọi API mới để lấy chiSo mới nhất của BN → set vào `chiSoEntity` (chỉ set canNang/chieuCao nếu chưa có giá trị hiện tại).

## Rủi ro / điểm cần lưu ý
- Focus change ở ke-don-thuoc-2 có thể ảnh hưởng flow nhập liệu nhanh của dược sĩ quen dùng phím Enter. Cần giữ nguyên behavior Enter key (`keyup.enter` → `onNextInput`), chỉ thay đổi focus ban đầu.
- API `/chiso/bybenhnhan` trả chiSo mới nhất — nếu BN chưa có lần khám nào có chiSo thì trả null (FE xử lý null bằng giữ nguyên giá trị trống).
- `KbChiSo.benhNhanId` tồn tại sẵn trong entity, query theo benhNhanId + order by id desc là an toàn.

## Yêu cầu chi tiết cho /work

### Step 1: BE — Thêm query lấy chiSo mới nhất theo benhNhanId
- `KbChiSoRepository`: thêm `KbChiSo findFirstByBenhNhanIdOrderByIdDesc(Integer benhNhanId)`
- `KbChiSoService`: thêm `KbChiSo getLatestByBenhNhanId(Integer benhNhanId)`
- `KbChiSoServiceImpl`: implement gọi repository, trả null nếu empty
- `KbChiSoController`: thêm `@GetMapping("/bybenhnhan")` gọi service mới

### Step 2: FE — Fix focus ke-don-thuoc-2
- `onOpenSoTay()`: tất cả nhánh → sau khi xử lý xong, focus vào `focusTab3Id2`
- `addThuoc()`: sau khi `focusInputSearchThuoc()` → giữ nguyên (đã đúng, quay về search)
- Chỉ thay đổi focus ở `onOpenSoTay()` (sau khi chọn thuốc + xử lý sổ tay) để nhảy vào "Đường dùng"

### Step 3: FE — Load chiSo BN cũ khi tiếp nhận
- `loadDetailsBN()`: sau callback `getById` benhNhan, gọi API `GET /chiso/bybenhnhan?benhNhanId=...`
- Nếu có chiSo: set `chiSoEntity.canNang`, `chiSoEntity.chieuCao` (và các field sinh tồn khác nếu cần)
- Gọi `onCalBMI()` sau khi set

### Step 4: Verify
- Kê đơn: chọn thuốc từ dropdown → xác nhận focus nhảy vào ô "Đường dùng"
- Tiếp nhận BN cũ (đã từng khám có cân nặng) → xác nhận ô cân nặng được điền sẵn

## Kết quả mong muốn (acceptance criteria)
1. Kê đơn: chọn thuốc → focus vào ô "Đường dùng" (focusTab3Id2) thay vì "Số ngày"
2. Kê đơn: nhấn Enter từ ô "Đường dùng" vẫn chuyển sang ô "Số ngày" bình thường
3. Tiếp nhận BN cũ: ô cân nặng (canNang) tự điền giá trị từ lần khám gần nhất
4. Tiếp nhận BN mới (lần đầu): ô cân nặng vẫn trống
5. FE không crash khi API chiSo trả về null (BN chưa có chiSo)