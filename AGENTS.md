# AGENTS.md — HC (HomeClinic / VNPT) System Architecture Reference

> Bản tóm tắt công nghệ, cấu trúc và quy ước code của 5 repo trong cùng workspace.
> Dùng làm "bản đồ" để làm việc trên hệ thống HomeClinic. Chỉ đọc tổng quan, chi tiết đọc trực tiếp trong từng repo.

Workspace gốc: `C:\Users\Khoa\Documents\hc-workspace\`

Hệ thống **HomeClinic (VNPT)**: nền tảng quản lý phòng khám (EMR) + **SaaS multi-tenant** (bán/cho thuê phòng khám theo combo).
Kiến trúc tổng thể: **Backend monolith `hc_code_app` (phòng khám) + Backend SaaS `hc_code_saas` (nền tảng) + Worker job `hc_code_saas_job` + Frontend (monolith & SaaS) + Mobile PWA mockup.**

| Repo | Vai trò | Ngôn ngữ / Framework |
|------|---------|----------------------|
| `hc_code_app` | Monolith phòng khám (khám bệnh, dược, BHYT, LIS, e-prescription, e-invoice) | Java 8, Spring Boot 2.3.0, Maven |
| `hc_code_saas` | Nền tảng SaaS (đăng ký/đơn hàng, tài khoản, RBAC, tenant, đăng nhập JWT/2FA) | Java 8, Spring Boot 2.3.0, Spring Cloud Hoxton |
| `hc_code_saas_job` | Worker job theo lịch (clone schema tenant, xử lý đơn hàng, thống kê sản lượng, quét thuốc hết hạn) | Java 8, Spring Boot 2.2.7 |
| `hc_code_frontend` | Web Angular cho phòng khám (monolith) + mobile PWA mockup | Angular 9 (JHipster), Webpack 4; mobile = HTML/PWA |
| `hc_code_frontend_saas` | Web Angular cho nền tảng SaaS (quản trị đơn hàng, tài khoản, vai trò, CSYT) + mobile PWA mockup | Angular 9 (JHipster), Webpack 4; mobile = HTML/PWA |

---

## 1. hc_code_app — Backend phòng khám (monolith) `hc-common`

- **Stack:** Java 8, Spring Boot **2.3.0.RELEASE**, Maven (**module đơn**, chỉ 1 `pom.xml` ở `hc-common/`), Lombok 1.18.20, MapStruct, Spring Data JPA + RSQL parser, Redis (cache), Kafka + RabbitMQ, Spring Security Crypto, JJWT, MinIO/S3 (object storage), POI/Excel, Freemarker, Swagger.
- **`hc-jasper/`** KHÔNG phải Maven module → chứa **template JasperReports `.jrxml`** (ví dụ `12. BangKeChiPhiKCB.jrxml`). Render thực tế delegate ra external service `ehreport` qua `OnehealthService.renderJasper(...)` (payload XML Base64).
- **Cấu trúc package** (base `hc.common`): `controller/` → `service/` → `service/impl/` → `repository/<domain>/` → `repository/entity/<domain>/` → `model/` (DTO) + `mapper/` (MapStruct) + `config/` (Swagger, Redis, RestTemplate, `config/multitenant/` schema-per-tenant, `config/rabbitmq/`), `constant/` (+ `constant/trangthai/` với nhiều class `TrạngThai*`, `constant/loai/`), `exception/` (`HCException`, `JwtTokenException` + `AdviceController`), `filter/` (`AppFilter` JWT/tenant), `query/` (RSQL), `utils/` (gồm `jdbc/ReportJdbcExecutor`, `ZonedDateTimeDeserializer`), `aop/` (LoggingAop), `kafka/`.
- **Quy ước prefix domain (quan trọng):**
  - `Dm` = Danh mục/dictionary (`DmNhanVien`, `DmKhoaPhong`)
  - `Kb` = Khám bệnh (`KbBenhNhan`, `KbPhieu`, `KbChiDinhDichVu`)
  - `Vt` = Vật tư/dược (`VtVatTu`, `VtTonKho`)
  - `Vp` = Viện phí/thanh toán (`VpPhieuThu`)
  - `Ht` = Hệ thống (`HtCauHinh`, `HtLog`)
  - `Ksk` = Khám sức khỏe, `Rp` = Báo cáo, `Hkd` = tích hợp HKD (VNPT)
- **Quy ước code:**
  - Controller extend `BaseController<D>` → cung cấp `POST /create`, `PUT /update`, `DELETE /delete`, `GET /details?id=`, `GET /search`, `GET /trash`. Trả về `ResponseEntity<BaseResponse<T>>` (`code:"00"` = OK, `"-1"` = lỗi).
  - Entity extend `BaseEntity` (id) hoặc `BaseDmEntity` (thêm `trangthai/ghichu/nguoitao/ngaytao/nguoisua/ngaysua`, có audit). Lombok `@Data @Builder`; `@Nationalized` cho text tiếng Việt; `ZonedDateTime` + `@JsonDeserialize(ZonedDateTimeDeserializer)` cho ngày.
  - Service: interface `XxxService extends BaseService<T>` + impl `XxxServiceImpl extends BaseServiceImpl/BaseDmServiceImpl`, `@Service @Transactional`, search dùng RSQL.
  - Entity/repo theo domain: `repository/entity/danhmuc/DmNhanVien.java`, `repository/danhmuc/DmNhanVienRepository.java extends BaseRepository<E>`.
  - Mapper MapStruct: `@Mapper(componentModel="spring")` trong `hc.common.mapper`.
  - Lỗi: throw `HCException`, message lấy từ `constant/MessageEnum` + i18n `messages_vi.properties`.
- **Domain nghiệp vụ:** khám bệnh, dược (DQG quốc gia), bảo hiểm y tế (QĐ130 XML), LIS phòng lab (Kafka/RabbitMQ), đơn thuốc điện tử (SOAP hPortal), hóa đơn điện tử.
- **Multi-tenant:** schema-per-tenant qua `SchemaContextHolder` / `MultiTenantUtil.choseTenant`.
- **SQL migration:** `deploy/database/YYYYMMDD/<tên>.sql`, convention `<YYYYmmdd>_<ddl/dml>_<nội dung>.sql` + header comment.
- **Profiles/env:** port 8081, config env-driven `app:...` kiểu `${ENV_VAR:default}`; profiles dev/test/production.

## 2. hc_code_saas — Backend nền tảng SaaS — `hc-saas`

- **Công nghệ:** Java 8, Spring Boot **2.3.0.RELEASE**, Spring Cloud Hoxton.SR5, Maven, Lombok + MapStruct, JPA + RSQL, Redis, Kafka, PostgreSQL, JJWT + `onehealth.authentication2factor` (2FA), MinIO, POI, Swagger.
- **Base package** `hc.saas`: `controller/`, `service/` + `service/impl/`, `repository/` + `repository/entity/`, `mapper/` (MapStruct), `model/request/`, `model/sme/`, `model/vncare/`, `query/` (RSQL builder), `utils/` (`JwtTokenFactory`, `PasswordUtil`, `ContextUtil`, `TOPTUtil`), `constant/enums/`, `config/`, `exception/` (`HCException`, `JwtTokenException`), `kafka/`.
- Các domain nghiệp vụ chính: `DonHang`/`SaDonHangCt`, `DmTaiKhoan`, `DmCsyt`, `DmVaiTro`/`DmChucNang`/`DmQuyen`, `HopDong`, `PhongKham`, `KhachHang`, `HtCauHinh`, `SaSanLuongCong`, `DtdtThongKe` (đơn thuốc điện tử). Jackson naming strategy snake_case (`config/SnakeCaseUpperCaseStrategy`).
- **REST naming:** Dùng tiếng Việt **kebab-case**, ví dụ `/public/login`, `/dang-ky-pk-step1`, `/xacnhandonhang`, `/ql-khoitao-donghang`, `/vaitro/add/chucnang`, `/onesme/notify`. Entity path thường `/tên-entity-lowercase`.
- **Layering & quy ước lớp:** Controller `@RestController` `@RequestMapping("/<lowercase>")` extends `BaseController<D>` (CRUD generic `POST /create`, `PUT /update`, `DELETE /delete`, `GET /details`, `GET /search`); Service interface + `*ServiceImpl` extends `BaseServiceImpl<E>` / `BaseDmServiceImpl<E extends BaseDmEntity>` (generic `search()` dùng RSQL); Repository `XxxRepository extends BaseRepository<T>`; Entity `@Entity @Table(name="snake_case")` Lombok `@Data @Builder` extends `BaseDmEntity` (audit) → `BaseEntity`. Service nghiệp vụ đặt tên theo nghiệp vụ tiếng Việt không dấu: `DonHangServiceImpl` (onboarding), `PublicServiceImpl` (login), `RoleAccountServiceImpl`, `HtGoiChucNangServiceImpl`, `HtCsytTrangThaiMenuServiceImpl`.
- **Kiến trúc SaaS:** **schema-per-tenant** Postgres, schema `hc_cli_*` từ template `hc_cli_template_chung`, `hc_cli_template_mienphi`. Đơn hàng `DonHang` (`sa_donhang`) → onboarding `DonHangServiceImpl` (`dangKyStep1/2`, `quanLyKhoiTaoDonHang` → tạo `DmCsyt/DmNhanVien/PhongKham/HopDong` → SME/Kafka provision).
- **RBAC:** `dm_vaitro`, `dm_chucnang`, `dm_quyen`, join `dm_taikhoan_vaitro`, `dm_vaitro_chucnang`, `dm_chucnang_quyen`, `HistoryRoleAccount`; module mới động `ht_goi_chucnang`, `ht_csyt_trangthai_menu`.
- **Auth:** `PublicController /public/login` → `/loginStep2` (2FA modes: TOTP_EMAIL/SMS, OTP_EMAIL/SMS) → JWT access/refresh RS256 (issuer `http://HC.VNCARE.VN`) via `JwtTokenFactory`.
- **Deploy:** `deploy/database/YYYYMM/YYYYMM.sql` (truy vấn iterate `%hc_cli%`); Dockerfile openjdk:8-jdk-alpine, TZ Asia/Ho_Chi_Minh, G1GC volume; Jenkinsfile build→push Nexus→SSH docker-compose `10.159.12.215`. Profiles dev/test/production (`application-dev.yml` port 8083).

## 3. hc_code_saas_job — Worker job theo lịch

- **Java 8, Spring Boot 2.2.7.RELEASE, Maven module đơn** (artifact `jobsaas-0.0.1.jar`). Spring Web, Data JPA, PostgreSQL, Lombok, HTTP client, `passay` password, `commons-text`, `spring-security-crypto`; dependency nội bộ `onehealth.common:authentication2factor`.
- **Base package** `hc.jobsaas`: `config/`, `constant/` (+`constant/enums/`), `controller/`, `job/` (core), `model/`, `repository/` (+`repository/entity/`), `service/` + `service/impl/`, `util/`.
- **Không dùng Quartz/Kafka/RabbitMQ** for scheduling → chỉ **Spring `@Scheduled`** (`@EnableScheduling`).
  - `XuLyDonHangJob` (`@Scheduled` fixedRate ~60s → clone schema tenant `HC_CLI_*`, tạo admin, gửi Email/SMS/Telegram; + cron 01:00 `hoatDongPhongKham()` chuyển trạng thái / gia hạn / tính chi phí nội bộ).
  - `SanLuongJob` (cron 22:00 → gọi Postgres function `fn_thongke_luotkham_ngay` tính sản lượng hàng tháng, gọi HTTP API).
  - `QuetThuocSapHetHanJob` (cron /4min → function `fn_quet_thuochethan` quét thuốc hết hạn → thông báo).
- **Quy ước Job:** `@Component @Slf4j`, kế thừa `BaseJob` (truy vấn DataSource/Hikari config), phương thức entry `schedule()/processDonHang()` + các step `processXxx()`, log theo `log.info("(methodName) bước...")`. Không có package DTO, dùng `model/`.
- **Dùng** `BaseRepository<E>` (kế thừa `JpaSpecificationExecutor` + `PagingAndSortingRepository<Integer>`).
- **Config:** profiles `application-{dev,test,staging,production}.yml`, nhiều placeholder `${DB_URL:...}`. Docker: `openjdk:8-jdk-alpine`, TZ Asia/Ho_Chi_Minh, `-Xms/-Xmx256m`. Branching GitLab/Jenkins.

## 4. hc_code_frontend — Web phòng khám + mobile PWA

### Web (`web/`)
- **Angular 9** (JHipster `emrsso`, `generator-jhipster ^6.8.0`), TypeScript 3.7.5, **Webpack 4** (custom, không dùng Angular CLI), Material + CDK 11, ng-bootstrap, ngx-translate, Highcharts, DevExtreme, SockJS/STOMP (WebSocket). Dev server port 9061.
- **Cấu trúc** `src/main/webapp/app/`:
  - `core/` (auth, user), `account/`, `admin/`, `layouts/`, `shared/` (components, model, util, constants, alert), `blocks/` (config, interceptor), `home/`, `login-page/`.
  - Feature modules lazy-load trong `entities/` theo domain tiếng Việt: `khambenh`, `tiepnhan`, `thungan`, `danhmuc`, `hethong`, `quanly`, `baocao`, `so-y-te`, `traketqua`, `sso`, `public`.
- **Naming:** file `kebab-case.<x>.{ts,html,scss}`; class PascalCase (vd `danh-sach-kham-benh.component.ts` → `DSKhamBenhComponent`); service `<x>.service.ts` `providedIn:'root`; route lowercase theo parent path + kebab module.
- **API config:** `app/app.constants.ts` env-injected `SERVER_API_URL`, `SERVER_WS`, `PHARMACY_URL`, `SSO_*` → webpack.DefinePlugin. Ví dụ prod `https://homeclinic-gw.vncare.vn/`. Services dùng `HttpClient` (e.g. `thungan.service.ts`, `pharmacy.service.ts` với bearer token).
### Mobile (`mobile/`)
- **KHÔNG phải Flutter/React Native** → PWA template "Affan", HTML/Pug/SCSS + Bootstrap 5, build bằng Gulp (`gulpfile.js` pug→html, scss→css). Trang `hc-*.pug` (hc-login, hc-ds-kham, hc-mh-khambenh, ...). Static mockup, chỉ gọi Google Cloud TTS.

## 5. hc_code_frontend_saas — Web SaaS admin + mobile

### Web (`web/`)
- **Angular 9** JHipster (`emrsso`), TypeScript 3.7.5, Material/CDK 11, Webpack custom. Dev port 9062.
- Cấu trúc JHipster: `core/`, `account/`, `admin/`, `layouts/`, `shared/`, `blocks/`, `home/dashboard`.
- Feature modules trong `entities/` theo domain tiếng Việt: `quan-ly-phong-kham`, `quan-ly-hop-dong`, `quan-ly-dich-vu`, `tai-khoan`, `tai-khoan-ttp` (TTP), `vai-tro`, `quyen`, `chucnang`, `csyt`, `khach-hang`, `chuoi-phong-kham`, `dung-chung` (`/hethong`). `shared/` mirror domain.
- Quy ước tên kế JHipster: kebab files, Pascal classes, selector prefix `Jhi-` (`jhi-`), service `dungChung.service.ts`.
- **Auth SaaS:** login 2 bước `/public/login` rồi `/loginStep2` (OTP); JWT ngx-webstorage; role `hasAnyAuthority()` + directive `has-any-authority`. Roles/Quyền là module first-class: `vai-tro`, `quyen`, `chuc-nang`, `goi-chuc-nang`, `tai-khoan-ttp`.
- **Tenant:** tường minh `tenantCode` trong popup `them-sua-tai-khoan`, filter `tenantCode=="*...*"`, dropdown `chi-tiet-don-hang`/`chi-tiet-dich-vu` (default `MB_1`), switch đơn vị `reloadChuyenDonVi(token)`.
### Mobile (`mobile/`)
- Như trên: Affan PWA (Pug/SCSS/Gulp), thêm các trang `hc-page-*` (medicine, patient-search, register, otp, virtual-assistant).

---

## Định tuyến: domain / từ khóa → repo nào

| Domain / Từ khóa | Repo (thư mục) |
|------------------|----------------|
| Khám bệnh phòng khám (kê đơn, bệnh án, chỉ định) | `hc_code_app` (hc-common) |
| Danh mục bệnh viện (dịch vụ, nhân viên, khoa phòng) | `hc_code_app` (hc-common) |
| Dược / vật tư / tồn kho / bán lẻ | `hc_code_app` (hc-common) |
| Viện phí, thanh toán, hóa đơn điện tử e-invoice | `hc_code_app` (hc-common) |
| Bảo hiểm y tế (BHTY / QĐ130 / XML) | `hc_code_app` (hc-common) + `document/baohiem` |
| Đơn thuốc điện tử / liên thông dược quốc gia | `hc_code_app` (hc-common) + `document/` |
| LIS / xét nghiệm (Kafka/RabbitMQ) | `hc_code_app` (hc-common) + `document/lis` |
| Báo cáo, PDF, template Jasper | `hc_code_app` (hc-jasper) + external `ehreport` |
| Đăng nhập, tài khoản, đơn hàng, RBAC, quyền | `hc_code_saas` |
| Quản trị SaaS (tài khoản, đơn hàng, CSYT, hợp đồng) | `hc_code_saas` (backend) + `hc_code_frontend_saas` (web) |
| Worker đăng ký / provision tenant schema | `hc_code_saas_job` |
| Xử lý đơn hàng mới → tạo schema `HC_CLI_*` | `hc_code_saas_job` |
| Báo cáo sản lượng (22:00) / quét thuốc hạn | `hc_code_saas_job` |
| Frontend phòng khám (bệnh nhân, báo cáo) | `hc_code_frontend` (web) |
| Frontend SaaS admin (tenant, người dùng, vai trò, CSYT) | `hc_code_frontend_saas` (web) |
| Mobile mockup (bản demo) | `hc_code_frontend` / `hc_code_frontend_saas` (mobile) |

---

## Lệnh build / run & entry point

| Repo | Entry point / main | Port | Build | Run (dev) |
|------|--------------------|------|-------|-----------|
| `hc_code_app` (`hc-common`) | `hc.common.Application` | **8081** (mọi profile) | `mvn clean package -Dmaven.test.skip=true` (wrapper `mvnw`) | `java -jar target\common-0.0.1.jar --spring.profiles.active=dev` (hoặc script `deploy-local.ps1`) |
| `hc_code_saas` (`hc-saas`) | `hc.saas.Application` | **8083** | `mvn clean package -Dmaven.test.skip=true` | chạy jar với `--spring.profiles.active=dev` |
| `hc_code_saas_job` (`job-saas`) | `hc.jobsaas.JobSaasApplication` | không HTTP nghiệp vụ | `mvn clean package -Dmaven.test.skip=true` (script `build.skiptest.bat`) | chạy jar với profile `dev/test/staging/production` |
| `hc_code_frontend` (`web`) | Angular app `emrsso` | dev **9061** | `npm run webpack:prod` (dev: `npm run webpack:dev`) | `npm run webpack:dev`; proxy → `http://localhost:8080` |
| `hc_code_frontend_saas` (`web`) | Angular app `emrsso` | dev **9062** | `npm run webpack:prod` | `npm run webpack:dev`; proxy → `http://localhost:8080` |

Lưu ý: cả 2 frontend web dùng Angular **9 core (`@angular/core ^9.0.4`) + Material/CDK 11** (bản kết hợp của JHipster `generator-jhipster 6.8.0`), build bằng **Webpack 4.41 custom** (`@ngtools/webpack 9.0.4`, không dùng Angular CLI build), Node dùng `--max_old_space_size=8192`. Entry script: `npm start` = `webpack:dev`. Mobile của 2 repo là PWA tĩnh (template "Affan", Pug/SCSS), build bằng `gulp` (không cần backend).

---

## Lưu ý làm việc chung

- **Không tạo/sửa file trong 5 repo trên** — chỉ đọc và tuân theo ràng buộc hiện tại.
- Ngôn ngữ tiếng Việt không dấu cho tên domain (module, entity, service). Tên class prefix theo domain (`Xxx`, `Dm`...). Backend Java: package `hc.<repo>`; 3 back-end đều Spring Boot Java 8 + Maven + Lombok + MapStruct + RSQL search.
- REST naming backend: tiếng Việt kebab-case; trả về `BaseResponse` code `"00"`.
- SQL migration theo cấu trúc `deploy/database/YYYYMMDD(.sql)` kèm header mô tả.
- Tất cả microservice/Dockerfile base `openjdk:8-jdk-alpine` (hoặc registry nội bộ VNPT), TZ `Asia/Ho_Chi_Minh`, Jenkins CI/CD → Nexus registry + docker-compose SSH deploy.

## Quy tắc bàn giao (bắt buộc với mọi agent, mọi session)

Trước khi bắt đầu BẤT KỲ task nào, luôn kiểm tra `.opencode/progress/` xem có file nào status khác `done` liên quan đến yêu cầu không. Nếu có, đọc toàn bộ file đó trước — nó chứa: đã làm gì, đang ở bước nào, đang chờ gì, việc tiếp theo là gì. Tiếp tục đúng từ đó, không hỏi lại người dùng những thông tin đã có sẵn trong file, không làm lại từ đầu.

Luôn cập nhật đúng progress file sau mỗi bước hoàn thành, khi bị block, và trước khi dừng lại chờ người dùng xử lý việc gì đó — để bất kỳ ai (người, hay agent khác) đọc lại cũng hiểu đúng tình trạng hiện tại.

Dùng lệnh `/work` để bắt đầu hoặc tiếp tục 1 task theo đúng quy trình chuẩn của team.

## Ràng buộc quan trọng — KHÔNG tự động hóa

- Agent KHÔNG được tự chạy/compile/apply SQL migration vào database thật (flyway, liquibase, psql, mysql...). Chỉ tạo file migration theo convention `deploy/database/YYYYMMDD/<tên>.sql`, người dùng tự chạy tay.
- Agent KHÔNG được tự merge code vào bất kỳ nhánh nào (dev/release/test/main...). Merge luôn do người dùng tự thực hiện thủ công qua Pull Request.