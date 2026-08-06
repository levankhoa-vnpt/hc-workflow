# Progress: phan-quyen-goi-thue-bao-form

status: done
next_action: none

## Bug Fixed
- CauHinhGoiThueBaoModalComponent was missing from EmrssoSharedModule declarations in shared.module.ts. This caused the "Danh sách chức năng" jqxTreeGrid to not render inside the CẤU HÌNH GÓI THUÊ BAO popup.
- Fix: Added import and declaration of CauHinhGoiThueBaoModalComponent in shared.module.ts (matching the pattern of CauHinhVaiTroModalComponent).
- Also removed invalid `[showHeader]="false"` from jqxTreeGrid in cau-hinh-goi-thue-bao.component.html (NG8002: showHeader is not a known property).
- TypeScript compilation (`tsc --noEmit`) passes with no errors.

## Branch
- Repo: hc_code_frontend_saas
- Branch: feature/phan-quyen-menu-cho-goi-thue-bao
- Commit: 66afa06

## Files touched
- web/src/main/webapp/app/shared/shared.module.ts (2 insertions: import + declaration)
- web/src/main/webapp/app/shared/popup/cau-hinh-goi-thue-bao/cau-hinh-goi-thue-bao.component.html (1 line: removed invalid [showHeader]="false")
