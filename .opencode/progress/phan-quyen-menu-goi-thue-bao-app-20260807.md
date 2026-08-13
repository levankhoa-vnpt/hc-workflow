# Progress: phan-quyen-menu-goi-thue-bao-app-20260807

status: approved
next_action: implement-saas-source-of-truth

## Done
- Read existing progress files.
- Created spec file.
- Inspected SaaS reference for `/hethong/goithuebao`.
- Inspected app FE menu flow: login stores `chucnang`, sidebar renders from localStorage.
- Inspected app BE and found no app-side `DmChucNang` model/controller; menu source appears SaaS-side.

## Current conclusion
- Best lazy design: keep `DmChucNang` and `thue_bao_chuc_nang` in SaaS, filter `api/saas/taikhoan/chucnang` by tenant package, and only add app FE route/screen if admin must configure from app UI.

## Blocker
Need confirm source of truth before code:
1. Keep menu package mapping in SaaS only?
2. Or duplicate mapping into app tenant DB?

## Approval
- User approved hướng 1: SaaS source-of-truth.
- Rule: visible menu = role menu ∩ thue_bao_chuc_nang package menu.

