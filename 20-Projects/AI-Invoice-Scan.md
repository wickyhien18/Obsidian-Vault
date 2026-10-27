---
Hub: "[[MOC - AI & ML]]"
created: 2026-09-07
status: active
repo: https://github.com/wickyhien18/ai-invoice-accounting
---

# Ý tưởng dự án

Hệ thống AI hỗ trợ nhập liệu hóa đơn/chứng từ cho kế toán — đồ án tốt nghiệp nhóm 3 sinh viên năm 4 CNTT (UTC Hà Nội). Mục tiêu: tự động hóa việc đọc, trích xuất, đối chiếu và gán bút toán cho hóa đơn, giảm thời gian nhập liệu thủ công và sai sót cho kế toán viên.

7 nghiệp vụ cốt lõi: (1) OCR số hóa hóa đơn, (2) trích xuất dữ liệu có cấu trúc, (3) hiểu ngữ cảnh (NLP), (4) chấm điểm độ tin cậy, (5) đối chiếu chứng từ (2-way/3-way matching), (6) gán bút toán (GL Coding) theo Thông tư 133/2016/TT-BTC, (7) phân tích & báo cáo xu hướng chi tiêu.

Định hướng kỹ thuật: dùng AI có sẵn qua API (OCR cloud + LLM), không tự train model — đảm bảo máy chạy nhẹ, triển khai được trong ~4 tháng.

# Tech Stack

|Nhóm|Thành phần|Công nghệ|Lý do chọn|
|---|---|---|---|
|Nền tảng|AI Service|Python 3.12 + FastAPI, Pillow|Cùng ngôn ngữ với backend; Python 3.14 chưa có gói cài sẵn cho một số thư viện|
|Nền tảng|Backend|Python + FastAPI, SQLAlchemy, Alembic, JWT, bcrypt, PyMySQL|ORM + migration rõ ràng cho 3 người cùng code; JWT/bcrypt cho đăng nhập và phân quyền kế toán viên, kế toán trưởng|
|Nền tảng|Frontend|React + Vite + Tailwind CSS (Node.js 20 LTS)|Yêu cầu ban đầu của người dùng, hệ sinh thái phổ biến|
|Nền tảng|Database|MySQL 8.0 (Docker), DECIMAL(18,2) cho số tiền|Đồng bộ phiên bản giữa 3 máy, không sai số làm tròn|
|Nền tảng|Môi trường|Docker Compose, một file `.env` ở root|Đồng bộ cấu hình 3 máy|
|Nền tảng|Mã nguồn|Git + GitHub, ruleset nhánh main (không xóa nhánh, không force push, bắt buộc PR)|Phù hợp quy mô nhóm 3 người|
|Nền tảng|Đầu vào|Đọc trực tiếp file XML hóa đơn điện tử người dùng tải lên|Không có API hoadondientu công khai; XML không cần OCR|
|Nền tảng|Đầu ra|Google Sheet/Excel (bảng kê mẫu 01-2/GTGT), thư viện ghi: openpyxl cho Excel, Google Sheets API cho Sheet (đề xuất, chưa chốt)|Chốt cách ghi tích hợp là Google Sheet/Excel|
|Nền tảng|Sơ đồ|PlantUML, dbdiagram.io (DBML)|Sửa chữ là hình tự cập nhật|
|AI|OCR (NV1)|PaddleOCR (local)|Có mô hình tiếng Việt, đủ phát hiện + nhận dạng, miễn phí không giới hạn trang|
|AI|LLM (NV1, NV4, NV7)|Gemini Flash qua free tier|Trích 5 trường trọng yếu thành JSON, gợi ý định khoản người bán mới, tóm tắt báo cáo; không cần GPU|
|AI|Chuẩn hóa tên công ty, địa chỉ (NV2, NV5)|RapidFuzz|Chạy CPU, miễn phí|
|AI|Phân loại tài liệu đầu vào (NV1)|Logistic Regression (scikit-learn)|Chạy CPU, đơn giản|
|AI|Gợi ý định khoản Nợ/Có (NV4)|TF-IDF + Logistic Regression (scikit-learn)|Chạy CPU; đặc trưng là người bán, thuế suất, tổng tiền, lịch sử định khoản|
|AI|So khớp đơn đặt hàng (NV5)|bge-m3 (sentence-transformers)|Chạy được trên CPU, hỗ trợ đa ngôn ngữ|
|AI|Phát hiện bất thường, hóa đơn trùng (NV6)|Isolation Forest (scikit-learn)|Chạy CPU, thể hiện phần ML khi thuyết trình|

# Nhật ký kỹ thuật

## Giai đoạn 1 — Xác định đề tài và phạm vi nghiệp vụ

**Mục tiêu giai đoạn:** Làm rõ hệ thống làm gì, AI đóng vai trò gì, đâu là phần bắt buộc phải có AI và đâu là phần logic nghiệp vụ thông thường.

**Đã làm:**

- Xác định 7 nghiệp vụ cốt lõi có AI: OCR, trích xuất ML, NLP, confidence scoring, matching, GL Coding, phân tích/báo cáo
- Liệt kê baseline không-AI tương ứng (nhập tay, mapping cố định, đối chiếu chính xác tuyệt đối) làm phương án dự phòng an toàn
- Tìm nguồn tham khảo quốc tế (Ramp, Precoro, Tipalti, HighRadius, Box, EverWorker) và nguồn Việt Nam (MISA AMIS) cho từng nghiệp vụ

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** làm đầy đủ AI thật (tự train OCR/NLP) hay dùng AI có sẵn qua API
- **Đã chọn:** Phương án A — dùng API cloud (Google Vision, LLM API), không train model riêng
- **Vì sao:** phù hợp thời gian 4 tháng và yêu cầu "cấu hình máy không nặng"; chỉ 2/7 nghiệp vụ (OCR + LLM parse) thực sự cần AI, còn lại là logic code thông thường

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa quyết định nền tảng triển khai cuối cùng: web hay app di động — tạm chốt ưu tiên web trước, app là mở rộng sau nếu còn thời gian

## Giai đoạn 2 — Thiết kế kiến trúc & cấu trúc thư mục repo

**Mục tiêu giai đoạn:** Dựng khung dự án ban đầu (monorepo 3 service) để nhóm bắt đầu code, đặt tên repo GitHub.

**Đã làm:**

- Đặt tên repo: `ai-invoice-accounting`
- Tạo cấu trúc thư mục: `ai-service/`, `backend/`, `frontend/`, `database/`, `docs/` (6 thư mục con tương ứng 6 chương báo cáo)
- Viết README.md tổng quan, README riêng cho từng service
- Thêm `.gitignore` chuẩn cho Python + Node

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** đặt tên repo trùng hoặc liên quan đến các dự án cũ (pharmacy-fullstack, steam-tinder) hay không
- **Đã chọn:** `ai-invoice-accounting` — tên độc lập, không liên quan dự án trước
- **Vì sao:** tránh nhầm lẫn ngữ cảnh giữa các đồ án/dự án cá nhân khác nhau

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Lúc này backend/frontend/database vẫn chưa chốt công nghệ cụ thể, README ghi tạm "chưa chốt"

## Giai đoạn 3 — Chốt công nghệ cụ thể và scaffold code

**Mục tiêu giai đoạn:** Chốt chính xác từng công nghệ cho từng thành phần và tạo code khởi tạo (scaffold) chạy được ngay.

**Đã làm:**

- Chốt: Backend = Python FastAPI, Frontend = React + Tailwind CSS, Database = MySQL, AI Service = Python (đồng nhất với backend)
- Tạo scaffold backend: `app/main.py`, `app/database.py`, `requirements.txt` (fastapi, sqlalchemy, pymysql, alembic, JWT, bcrypt...)
- Tạo scaffold AI service: `app/main.py`, `requirements.txt` (fastapi, pillow, pytesseract, google-cloud-vision...)
- Tạo scaffold frontend: Vite + React + Tailwind (`package.json`, `vite.config.js`, `tailwind.config.js`, `App.jsx` chạy được ngay)
- Viết `docker-compose.yml` chạy MySQL 8.0 với charset `utf8mb4` cấu hình sẵn

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** cài MySQL trực tiếp lên từng máy hay chạy qua Docker
- **Đã chọn:** Docker (docker-compose cho MySQL)
- **Vì sao:** đảm bảo cả 3 thành viên chạy đúng 1 phiên bản MySQL giống hệt nhau, tránh lỗi "máy tui chạy được, máy bạn không" do lệch charset/version
- **Vấn đề:** dùng kiểu dữ liệu nào cho số tiền trong MySQL
- **Đã chọn:** `DECIMAL(18,2)`, không dùng FLOAT/DOUBLE
- **Vì sao:** tránh sai số làm tròn — hệ thống kế toán không được phép sai lệch số liệu dù nhỏ

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 1 — Lỗi build wheel pydantic-core/pillow trên Python 3.14:** Chạy `pip install -r requirements.txt` trong `ai-service` báo lỗi build wheel thất bại cho `pillow` và `pydantic-core` → Nguyên nhân thật sự: máy dùng Python 3.14 (quá mới), các thư viện chưa có pre-built wheel, pip phải compile từ source bằng Rust (PyO3) nhưng PyO3 0.22.2 chưa hỗ trợ Python 3.14 → Cách sửa: xóa venv, tạo lại bằng Python 3.12 (`python3.12 -m venv venv`), cài lại `pip install -r requirements.txt`

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa viết endpoint thật cho `/ocr` và `/extract` trong AI Service — mới có `/health` placeholder

## Giai đoạn 4 — Quản lý biến môi trường (.env) và bảo mật cấu hình

**Mục tiêu giai đoạn:** Chuẩn hóa cách quản lý secret/cấu hình giữa 3 service, tránh rối khi 3 thành viên cùng setup máy riêng.

**Đã làm:**

- Ban đầu tạo 3 file `.env.example` riêng (root cho MySQL, backend, ai-service) — sau đó phát hiện gây trùng lặp và dễ lệch dữ liệu
- Consolidate lại thành **1 file `.env.example` duy nhất ở root**, chứa toàn bộ biến cho MySQL + backend + AI service
- Sửa `backend/app/database.py` và `ai-service/app/main.py` để tự động load `.env` gốc qua đường dẫn tương đối (`Path(__file__).resolve().parent.parent.parent / ".env"`)
- Viết chi tiết trong SETUP.md: ý nghĩa từng biến, các lỗi thường gặp khi `.env` bị lệch giữa `DB_USER`/`MYSQL_USER`...

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** giữ 3 file `.env` riêng theo từng service, hay gộp thành 1 file duy nhất
- **Đã chọn:** gộp thành 1 file `.env` duy nhất ở root
- **Vì sao:** 3 file riêng dễ bị lệch giá trị (VD: sửa MySQL_PASSWORD ở root nhưng quên sửa DB_PASSWORD ở backend) → gây lỗi "Access denied" khó debug; 1 file duy nhất loại bỏ hoàn toàn nguy cơ này

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 2 — File `.env.example` cũ bị dùng nhầm:** Người dùng tải nhầm bản `.env.example` cũ (4 dòng, tiếng Việt, tên file `_env.example`) từ một lần tải trước đó → Nguyên nhân thật sự: file cũ vẫn còn trong thư mục uploads của người dùng, không phải lỗi từ phía service → Cách sửa: đối chiếu MD5 hai file để xác nhận khác nhau, hướng dẫn tải lại đúng zip mới nhất chứa file `.env.example` đã consolidate (23 dòng, tiếng Anh)

## Giai đoạn 5 — Chuyển ngôn ngữ tài liệu dự án

**Mục tiêu giai đoạn:** Thử chuẩn hóa toàn bộ tài liệu kỹ thuật (README, SETUP.md, comment code) sang tiếng Anh, sau đó quay lại tiếng Việt theo yêu cầu thực tế của người dùng.

**Đã làm:**

- Dịch toàn bộ README.md, SETUP.md, các README con, comment trong `database.py`/`main.py` sang tiếng Anh
- Quét toàn bộ repo bằng script Python kiểm tra ký tự có dấu để đảm bảo không sót tiếng Việt
- Sau đó quay lại dùng tiếng Việt cho toàn bộ trao đổi và nội dung báo cáo đồ án (Chương 2, 3) theo yêu cầu người dùng

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** ngôn ngữ tài liệu dự án dùng tiếng Anh hay tiếng Việt
- **Đã chọn:** Code/README kỹ thuật giữ tiếng Anh (thời điểm viết); nội dung báo cáo đồ án (Chương 2 trở đi) dùng tiếng Việt
- **Vì sao:** báo cáo đồ án nộp cho hội đồng chấm tại Việt Nam, thuật ngữ kế toán (Nợ/Có, TT133...) cần chính xác bằng tiếng Việt để tránh hiểu nhầm khi bảo vệ

## Giai đoạn 6 — Cấu hình bảo vệ nhánh Git (Branch Ruleset)

**Mục tiêu giai đoạn:** Thiết lập quy tắc bảo vệ nhánh `main` trên GitHub phù hợp quy mô nhóm 3 người.

**Đã làm:**

- Tạo ruleset trên GitHub Settings → Rules cho repo `ai-invoice-accounting`
- Bật 3 rule cốt lõi: Restrict deletions, Block force pushes, Require a pull request before merging
- Bỏ qua các rule nâng cao không cần thiết cho quy mô nhỏ (signed commits, code scanning, status checks CI...)

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** bật đầy đủ mọi rule bảo vệ nhánh hay chỉ chọn lọc
- **Đã chọn:** chỉ 3 rule tối thiểu (xóa nhánh, force push, PR bắt buộc)
- **Vì sao:** nhóm 3 người không có CI/CD, không cần ký commit — bật thừa rule chỉ gây cản trở tốc độ làm việc mà không tăng thêm an toàn thực tế

## Giai đoạn 7 — Cài đặt công cụ hỗ trợ (Docker, MySQL Workbench)

**Mục tiêu giai đoạn:** Hướng dẫn cài đặt môi trường phát triển đầy đủ cho các thành viên nhóm, kể cả người chưa quen dùng terminal.

**Đã làm:**

- Viết hướng dẫn cài Python 3.12, Node.js 20 LTS, Docker theo từng hệ điều hành (Windows/Mac/Linux, riêng CachyOS/Arch)
- Viết hướng dẫn kiểm tra đã cài hay chưa (`--version` cho từng công cụ)
- Viết hướng dẫn cài và kết nối MySQL Workbench tới MySQL chạy trong Docker (`127.0.0.1:3306`) cho người muốn thao tác bằng GUI thay vì terminal

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa cài đặt Docker trên máy người dùng tại thời điểm ghi chú này — đang chờ hoàn tất bước cài đặt trước khi chạy `docker compose up`

## Giai đoạn 8 — Phân tích yêu cầu (Chương 2 báo cáo đồ án)

**Mục tiêu giai đoạn:** Xây dựng Chương 2 (Phân tích & Thiết kế hệ thống) đúng khung lý thuyết Requirement Analysis đã học (dựa theo slide môn học Project 1 — Phân tích yêu cầu).

**Đã làm:**

- Xác định 6 nhóm bên liên quan (stakeholders)
- Soạn bảng câu hỏi phỏng vấn và bảng hỏi khảo sát (elicitation)
- Thu thập danh sách tài liệu liên quan (TT133, Nghị định 70/2025/NĐ-CP, mẫu biểu chứng từ...)
- Đặc tả yêu cầu chức năng (FR-01 → FR-10) và phi chức năng (NFR-01 → NFR-05)
- Ưu tiên hóa theo MoSCoW (Must/Should/Could/Won't have)
- Mô hình hóa yêu cầu đầy đủ 5 kỹ thuật: Use Case Diagram, Activity Diagram, DFD, ERD, User Story & Acceptance Criteria (US-01 → US-14)
- Thêm mục Xác minh yêu cầu và Quản lý yêu cầu theo đúng khung slide

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** thứ tự Chương 2 và Chương 3 nên là Cơ sở lý thuyết trước hay Phân tích thiết kế trước
- **Đã chọn:** đảo lại — Chương 2 = Phân tích & Thiết kế, Chương 3 = Cơ sở lý thuyết
- **Vì sao:** theo yêu cầu trực tiếp của người dùng, khác quy ước thông thường nhưng vẫn triển khai được vì Chương 4-6 không bị ảnh hưởng bởi việc hoán đổi này
- **Vấn đề:** "User Story & Acceptance Criteria" nên là mục riêng đứng trước Đặc tả yêu cầu, hay là 1 trong 5 kỹ thuật con của Mô hình hóa yêu cầu
- **Đã chọn:** xếp vào làm mục con (2.5.5) trong Mô hình hóa yêu cầu
- **Vì sao:** đúng theo thứ tự liệt kê trong slide gốc môn học (Use Case Diagram, Activity Diagram, DFD, ERD, User Story/Acceptance Criteria đều là 5 kỹ thuật ngang hàng)

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Cần rà lại toàn bộ định dạng trích dẫn nguồn trong Chương 2-3 theo đúng chuẩn APA 2 phần (in-text + danh mục cuối bài) thay vì kiểu liệt kê tự do "**Nguồn:** ... — link" đang dùng tạm

## Giai đoạn 9 — Thiết kế cơ sở dữ liệu (ERD) và phát hiện thiếu sót

**Mục tiêu giai đoạn:** Hoàn thiện ERD cho hệ thống, đảm bảo đủ bảng để mọi nghiệp vụ (đặc biệt đối chiếu chứng từ) hoạt động được bằng logic tự động, không chỉ là mô tả suông.

**Đã làm:**

- Thiết kế ban đầu 6 bảng: `users`, `invoices`, `invoice_fields`, `chart_of_accounts`, `journal_entries`, `matching_results`
- Vẽ ERD trực quan (SVG) và Use Case Diagram minh họa

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** bảng `matching_results` chỉ lưu `po_reference` dạng text, không có bảng PO thật để đối chiếu số liệu tự động
- **Đã chọn:** bổ sung bảng `purchase_orders` (id, po_number, vendor, order_date, expected_amount, status), sửa `matching_results.po_reference` thành khóa ngoại `po_id` trỏ đúng bảng này
- **Vì sao:** nếu không có bảng PO thật, hệ thống không thể tự động so sánh số tiền hóa đơn với đơn đặt hàng — tính năng đối chiếu (FR-06) sẽ chỉ là nhập tay, mất hết ý nghĩa "tự động" của đề tài AI
- **Vấn đề:** cách nhập dữ liệu PO vào hệ thống — có nên dùng OCR chụp ảnh như hóa đơn không
- **Đã chọn:** Cách 1 (form nhập liệu trực tiếp) + Cách 2 (import hàng loạt từ Excel/CSV), không dùng OCR cho PO
- **Vì sao:** PO do chính doanh nghiệp tự tạo ra trước khi mua hàng (không phải nhận từ bên ngoài dưới dạng ảnh như hóa đơn), nên nhập liệu trực tiếp hợp lý hơn nhiều so với việc chụp ảnh rồi OCR lại — tránh tốn chi phí API không cần thiết

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 3 — Thiếu bảng dữ liệu để đối chiếu tự động:** ERD ban đầu có `matching_results` nhưng không có nơi lưu dữ liệu PO thật (chỉ có `po_reference` là chuỗi text) → Nguyên nhân thật sự: thiết kế ERD lần đầu chỉ tập trung vào luồng xử lý hóa đơn, bỏ sót rằng đối chiếu cần 2 phía dữ liệu (hóa đơn VÀ đơn đặt hàng) → Cách sửa: bổ sung bảng `purchase_orders`, đổi `po_reference` (text) thành `po_id` (khóa ngoại), thêm cột `amount_difference` để hệ thống tự tính chênh lệch

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Bổ sung FR-09 (nhập PO qua form) và FR-10 (import PO từ Excel/CSV) vào bảng đặc tả yêu cầu và MoSCoW — đã cập nhật văn bản nhưng **chưa vẽ lại ERD trực quan (SVG) và Use Case Diagram** với bảng/use case mới này
    - ✅ **Đã giải quyết ở Giai đoạn 12–13:** ERD chốt lại thành 9 bảng (có `purchase_orders`), use case được sửa và giải thích lại
- Cần thống nhất lại tên use case giữa bản vẽ tay của người dùng (VD: "Tải hóa đơn và nhận dạng hóa đơn", "Kiểm tra thông tin và rủi ro của hóa đơn") với danh sách use case đã viết trong Chương 2 — hiện đang lệch tên gọi giữa 2 nơi, cần đồng bộ trước khi hoàn thiện báo cáo
    - ✅ **Đã giải quyết ở Giai đoạn 12:** use case được sửa/bổ sung kèm giải thích theo đúng nghiệp vụ

## Giai đoạn 10 — Vẽ sơ đồ trực quan trên Figma

**Mục tiêu giai đoạn:** Chuyển các mô hình yêu cầu (Use Case, DFD) từ mô tả văn bản/code DBML sang sơ đồ trực quan để chèn vào báo cáo.

**Đã làm:**

- Kết nối Figma qua tài khoản người dùng (team "Hiển Giáp's team")
- Vẽ DFD mức 1 bằng Mermaid syntax trong FigJam — ban đầu dùng D1-D5 kèm `chart_of_accounts`, sau đó chỉnh lại đúng theo thứ tự đặt tên kho dữ liệu của người dùng (D1 Hóa đơn, D2 Đơn đặt hàng, D3 Đối chiếu, D4 Bút toán) và bổ sung D6 cho danh mục tài khoản (phát hiện thiếu khi đối chiếu với bản vẽ tay)
- Vẽ lại Use Case Diagram theo đúng bố cục ảnh người dùng đã vẽ tay (2 actor, mỗi actor 4 và 3 use case)

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** DFD có nên giữ kho "D5 Tổng hợp" như bản vẽ tay của người dùng hay bỏ đi
- **Đã chọn:** đề xuất bỏ D5, thay bằng luồng output trực tiếp từ tiến trình báo cáo ra Kế toán trưởng — nhưng để người dùng tự quyết định giữ nguyên hay đổi
- **Vì sao:** theo lý thuyết DFD chuẩn, kho dữ liệu (data store) phải là nơi lưu trữ lâu dài; báo cáo tổng hợp thường được tính toán lại mỗi lần từ dữ liệu bút toán, không cần lưu cố định thành kho riêng — quyết định cuối cùng vẫn chưa chốt tại thời điểm ghi chú này
    - ✅ **Chốt ở Giai đoạn 12:** bỏ hẳn DFD khỏi bộ sơ đồ (thầy nhận xét DFD/BFD thừa vì đã có Activity Diagram), nên câu hỏi D5 không còn cần thiết

## Giai đoạn 11 — Gộp báo cáo thành một file duy nhất

**Mục tiêu giai đoạn:** Gộp các phần báo cáo của nhóm thành một file Word thống nhất và bỏ các thông tin không cần thiết.

**Đã làm:**

- Gộp thành `Bao_cao_Nhom15_final.docx` (39 trang) bằng script merge từ file báo cáo nhóm `Project_1.docx`
- Giữ nguyên toàn bộ nội dung đối chiếu đơn đặt hàng (PO) và báo cáo/dashboard — nhóm đã chốt không cắt hai phần này
- Chốt cách ghi tích hợp đầu ra trong toàn bộ tài liệu là **"Google Sheet/Excel"**
- Chốt hướng không kết nối trực tiếp hoadondientu: hóa đơn điện tử không có API công khai, hệ thống nhận file người dùng tải lên, không tra cứu tự động, không xử lý captcha

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** chọn OCR engine và LLM cụ thể nào
- **Đã chọn:** chưa chọn — báo cáo so sánh 3 phương án ở Bảng 2.2: (A) API AI lớn, (B) OCR mã nguồn mở như Tesseract, (C) kết hợp
- **Vì sao:** chưa có số liệu thử nghiệm thật để chọn; tránh nêu độ chính xác không có nguồn
- **Vấn đề:** ngưỡng độ tin cậy dùng cho từng trường
- **Đã chọn:** xanh ≥ 90, vàng 70 đến dưới 90, đỏ < 70
- **Vì sao:** kế toán viên chỉ phải tập trung vào ô vàng/đỏ, ô xanh vẫn được đối chiếu nhưng ít rủi ro hơn

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 4 — Script merge Word lỗi nhiều bước:** thiếu style "Table Grid", sai chữ ký hàm phụ trợ, nhận diện sai trang của mục lục, thừa page break, dư trang trắng cuối file → Nguyên nhân thật sự: file gốc của nhóm không có đủ style/cấu trúc mà script giả định → Cách sửa: sửa từng lỗi trong script, dựng lại file đến khi hết lỗi

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- File Word hiện **đã cũ so với các quyết định ở Giai đoạn 12–14**: còn ERD 5 bảng (cần thay bằng ERD 9 bảng), còn bảng MoSCoW 3 cột, còn mục DFD, chưa có SD1/SD2, use case chưa thay hình mới. Người dùng yêu cầu tạm thời không ghi vào Word
- Bản patch bảng MoSCoW trong script merge bị lỗi cú pháp (chuỗi chứa `\n` trong heredoc), chưa áp dụng được
- Mục lục đang là mục lục tĩnh, danh sách User Story vẫn đang đánh lại số

## Giai đoạn 12 — Chốt MoSCoW và bộ sơ đồ thiết kế

**Mục tiêu giai đoạn:** Chốt cách trình bày MoSCoW và bộ sơ đồ đủ dùng, sửa use case cho đúng nghiệp vụ.

**Đã làm:**

- Đổi bảng MoSCoW: bỏ cột lý do, gom các đặc tả cùng nhóm vào một dòng (Must: FR-01, FR-02, FR-03, FR-04, FR-05, FR-07, xuất Excel/Sheet mẫu 01-2/GTGT, NFR-03/04/05; Should: FR-06, FR-09, FR-08, NFR-01/02; Could: import PO từ Excel/CSV, cảnh báo chi tiêu bất thường; Won't: kết nối trực tiếp hoadondientu, tự hạch toán không qua duyệt, app di động riêng, đọc ngược từ Sheet, tích hợp MISA/KiotViet)
- Rà lại 7 nghiệp vụ theo cách gọi của báo cáo: NV1 Tiếp nhận và bóc tách bằng AI, NV2 Kiểm tra sơ bộ, NV3 Đối chiếu song song và hiệu chỉnh, NV4 Gợi ý định khoản Nợ/Có, NV5 Đối chiếu đơn đặt hàng, NV6 Phê duyệt hai cấp, NV7 Xuất bảng kê và báo cáo
- Sửa lại Use Case Diagram kèm giải thích từng use case (include theo mũi tên dọc để tránh đè nhau)
- Vẽ hai sơ đồ tuần tự SD1, SD2 (bản đầu, sau đó được vẽ lại ở Giai đoạn 14)

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** DFD và BFD có cần giữ không (thầy nhận xét là thừa vì đã có Activity Diagram)
- **Đã chọn:** bỏ DFD và BFD; bộ sơ đồ gồm Use Case, Activity, Sequence (SD1, SD2), ERD, kiến trúc hệ thống; sơ đồ trạng thái hóa đơn là tùy chọn
- **Vì sao:** tránh trùng lặp, mỗi sơ đồ trả lời một câu hỏi khác nhau (ai làm gì / thành phần nào gọi nhau / dữ liệu lưu ở đâu)
- **Vấn đề:** có cần mô hình tuần tự không
- **Đã chọn:** có, vẽ SD1 (tiếp nhận → hiệu chỉnh) và SD2 (gửi duyệt → duyệt/từ chối → xuất)
- **Vì sao:** Activity chỉ cho thấy thứ tự công việc, không cho thấy thành phần nào ghi vào bảng nào

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Slide NV1 ghi ngưỡng 85% trong khi báo cáo dùng 90/70 — cần thống nhất
- Slide NV2 còn câu "tự động tra cứu trên cổng thuế", mâu thuẫn với quyết định không kết nối hoadondientu — cần sửa
- Căn cứ pháp lý mẫu 01-2/GTGT: slide ghi TT119/2014, nguồn lại ghi TT156/2013 — chưa kiểm chứng, cần tra lại
- Câu "3–5 phút/chứng từ" chưa có nguồn — bỏ hoặc ghi rõ là ước tính
- Hai hình minh họa Google Sheet cần ảnh chụp thật, hiện chỉ để chỗ trống
- Câu FR-07 nên sửa thành "lưu trạng thái duyệt của hóa đơn"
- Danh sách "7 nghiệp vụ" ở đầu file nhật ký (OCR, trích xuất, NLP, độ tin cậy, đối chiếu, GL Coding, báo cáo) khác với NV1–NV7 trong báo cáo — nên thống nhất một cách gọi
- Mục "Tech Stack" và Giai đoạn 1 ghi đã chọn Phương án A (API cloud), nhưng báo cáo hiện so sánh 3 phương án mà chưa chọn — nhóm cần quyết định lại cho khớp

## Giai đoạn 13 — Chốt ERD 9 bảng và từ điển dữ liệu

**Mục tiêu giai đoạn:** Chốt ERD cuối, đủ để kiểm tra tính minh bạch sổ sách, kèm từ điển dữ liệu và bảng giải thích.

**Đã làm:**

- Chốt 9 bảng, 5 cụm: `users` | `invoices`, `invoice_fields` | `chart_of_accounts`, `journal_entries` | `item_categories`, `category_aliases` | `purchase_orders`, `matching_results`
- 12 liên kết khóa ngoại; `journal_entries` có hai khóa ngoại cùng trỏ vào `chart_of_accounts` (`debit_account`, `credit_account`); `users` nối vào `invoices` ở 2 chỗ (`uploaded_by`, `approved_by`) và vào `purchase_orders` (`created_by`)
- `invoices` gồm thêm `file_path`, `file_type`, `deduct_group` và các cột trạng thái `status`, `reject_reason`, `approved_by`, `approved_at`, `export_status`, `exported_at`
- Viết DBML cho dbdiagram.io và các bảng giải thích: tổng quan 9 bảng, bảng quan hệ, từ điển từng cột, bảng giá trị `status`
- Thống nhất thứ tự trình bày ERD với thầy: `invoices` → `invoice_fields` → `item_categories`/`category_aliases` → `chart_of_accounts`/`journal_entries` → `purchase_orders`/`matching_results` → `users`

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** trạng thái duyệt lưu ở bảng nào
- **Đã chọn:** lưu ở `invoices` (`status`, `approved_by`, `approved_at`, `reject_reason`); `journal_entries` không có cột trạng thái
- **Vì sao:** duyệt là theo cả hóa đơn, không theo từng dòng bút toán
- **Vấn đề:** duyệt và xuất có chung một trạng thái không
- **Đã chọn:** tách riêng `status` và `export_status`
- **Vì sao:** ghi Sheet lỗi thì hóa đơn vẫn ở `da_duyet`, xuất lại được mà không bị ghi trùng
- **Vấn đề:** có lưu việc chia màn hình đối chiếu vào database không
- **Đã chọn:** không, lưu ở trình duyệt; bỏ cột `is_edited`
- **Vì sao:** chỉ là trạng thái giao diện; việc sửa vẫn truy được nhờ so sánh `extracted_value` với `final_value`
- **Vấn đề:** có lưu file hóa đơn gốc không
- **Đã chọn:** có, thêm lại `file_path` và `file_type`
- **Vì sao:** cần xem lại chứng từ gốc khi duyệt và kiểm tra sổ sách; `file_type` quyết định đọc XML trực tiếp hay gọi AI
- **Vấn đề:** có cần phân nhóm theo mẫu 01-2/GTGT không
- **Đã chọn:** thêm `deduct_group` vào `invoices`
- **Vì sao:** để xếp hóa đơn vào đúng nhóm khi kê khai thuế

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 5 — Làm mất bảng PO khi thay ERD trong báo cáo:** Khi gộp file, ERD 7 bảng của nhóm bị thay bằng bản 5 bảng → Nguyên nhân thật sự: bản 5 bảng được vẽ để gọn nhưng không có `purchase_orders` và `matching_results`, trong khi nhóm đã chốt giữ PO → Cách sửa: dùng ERD 9 bảng chốt ở giai đoạn này; chưa cập nhật vào file Word
- **Bug 6 — Gợi ý sai vị trí lưu trạng thái duyệt:** Ban đầu gợi ý đặt trạng thái duyệt ở `journal_entries` → Nguyên nhân thật sự: nhầm đơn vị duyệt (theo dòng bút toán thay vì theo hóa đơn) → Cách sửa: chuyển toàn bộ sang `invoices`

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa liệt kê giá trị cụ thể của `deduct_group` theo mẫu 01-2/GTGT thật
- `purchase_orders.status` chưa định nghĩa chính thức (đề xuất: đang mở, đã đối chiếu)
- Cần chặn truy cập `file_path`: chỉ người dùng đã đăng nhập mới mở được file hóa đơn (là chứng từ tài chính)
    - ✅ **Đã giải quyết ở Giai đoạn 23:** file gốc chỉ mở khi có token (trả 401 khi không có)
- Chưa viết migration Alembic và model SQLAlchemy cho 9 bảng
- Chưa cập nhật ERD mới vào báo cáo Word

## Giai đoạn 14 — Vẽ lại Activity, SD1, SD2 bằng PlantUML

**Mục tiêu giai đoạn:** Kiểm tra Activity và SD sau khi chốt ERD, vẽ lại cho khớp và có cách vẽ lại được.

**Đã làm:**

- Rà Activity cũ (dạng flowchart không làn) và tìm 6 chỗ lệch ERD/thiết kế chốt: không có swimlane; không rẽ nhánh theo `file_type`; độ tin cậy chỉ 2 nhánh Thấp/Cao; đối chiếu PO luôn chạy; thiếu bước xuất sang Google Sheet/Excel; vòng từ chối không đi qua kế toán viên gửi lại
- Vẽ lại Activity có 3 làn (Kế toán viên, Hệ thống/AI, Kế toán trưởng)
- Đặt tên: **SD1 — Tiếp nhận, bóc tách bằng AI và hiệu chỉnh hóa đơn** (NV1–NV3); **SD2 — Gửi duyệt, phê duyệt hai cấp và xuất bảng kê** (NV4–NV7)
- Xuất 3 file `.puml` và 3 ảnh PNG; viết sẵn lời thuyết trình cho từng sơ đồ và các câu thầy có thể hỏi

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** công cụ vẽ sơ đồ
- **Đã chọn:** PlantUML (dán code vào plantuml.com, xuất PNG/SVG); draw.io là phương án thay thế khi cần kéo thả tay
- **Vì sao:** sửa chữ là hình tự cập nhật, đúng chuẩn UML, có swimlane và khung `alt`/`opt`; thử Mermaid trước nhưng swimlane không đúng chuẩn (các làn xếp cạnh nhau, hình rất dài)

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 7 — Mermaid báo lỗi cú pháp ở SD2:** render sequence diagram thất bại → Nguyên nhân thật sự: dấu `;` trong nội dung tin nhắn bị Mermaid hiểu là dấu ngắt lệnh → Cách sửa: thay `;` bằng dấu phẩy (sau đó chuyển hẳn sang PlantUML)
- **Bug 8 — Ảnh PNG bị chìm chữ khi xem ở chế độ tối:** hộp sáng, chữ sáng trên nền đen → Nguyên nhân thật sự: PNG xuất ra có nền trong suốt, trình xem tô nền đen → Cách sửa: thêm `skinparam backgroundColor white` vào đầu file `.puml`, kiểm tra pixel góc ảnh là trắng

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Activity và SD2 lệch thứ tự: Activity cho hệ thống gợi ý định khoản **trước** khi kế toán viên bấm xác nhận, còn SD2 bắt đầu từ lúc xác nhận rồi mới ghi `journal_entries` — cần chọn một cách và vẽ lại cho khớp
- Tên làn "Hệ thống/AI" đang gộp backend và AI service, tách ra nếu thầy yêu cầu
- Ví dụ bút toán "Nợ 6422 / Có 331" chỉ minh họa cho hóa đơn mua hàng có VAT
- Chưa thay hình mới (use case, Activity, SD1, SD2, ERD) vào báo cáo Word
- Sơ đồ trạng thái hóa đơn (tùy chọn) chưa vẽ
- Chưa có ảnh chụp Google Sheet thật, chưa có số liệu thử nghiệm OCR/LLM
    - ✅ Số liệu thử nghiệm OCR/LLM đã có ở Giai đoạn 26 (chưa có ảnh chụp Google Sheet thật)

## Giai đoạn 15 — Thêm mục "Công nghệ áp dụng" vào báo cáo

**Mục tiêu giai đoạn:** Đưa danh sách model/công nghệ miễn phí mà nhóm đã tra cứu (tháng 10/2026) vào báo cáo Word, để phần công nghệ có căn cứ thay vì để trống.

**Đã làm:**

- Thêm mục **7. Công nghệ áp dụng** vào cuối Chương 2 (sau 6.4 ERD) trong `Project_1.docx`, gồm 7.1 Danh sách công nghệ theo từng khâu nghiệp vụ (13 dòng: OCR, LLM, ML/NLP, FPT.AI Reader chưa rõ chi phí), 7.2 Bộ công nghệ tối thiểu để chạy trọn đồ án (OCR: PaddleOCR/VietOCR + Azure F0 để so sánh; LLM: Gemini Flash free tier; ML riêng: TF-IDF + Logistic Regression, Isolation Forest, bge-m3), 7.3 Lưu ý khi áp dụng
- Cập nhật mục lục tĩnh (4 dòng mới, trang 41–44), kiểm tra bằng cách render PDF: heading 6.4 vẫn ở trang 36 như mục lục cũ
- Giữ nguyên mọi phần khác của file Word

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** viết mục này như kết luận đã chọn công nghệ, hay như danh sách ứng viên
- **Đã chọn:** viết là danh sách ứng viên, ghi rõ chưa chốt phương án OCR/LLM cuối cùng
- **Vì sao:** khớp với Bảng 2.2 trong báo cáo (so sánh 3 phương án mà chưa chọn) và vì free tier thay đổi thường xuyên, chưa có số liệu thử nghiệm thật
- **Vấn đề:** đặt mục mới ở đâu trong báo cáo
- **Đã chọn:** mục 7 của Chương 2, sau phần mô hình hóa yêu cầu
- **Vì sao:** Chương 2 là Phân tích và Thiết kế, công nghệ áp dụng đứng sau yêu cầu và mô hình; nếu nhóm muốn đưa sang Chương 3 hoặc Chương 4 thì chuyển được

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Mục "Tech Stack" ở đầu file này ghi Google Cloud Vision/Tesseract và GPT-4o-mini/Claude, còn danh sách mới chủ yếu là PaddleOCR/VietOCR, Gemini Flash, Azure F0 — cần thống nhất lại cho khớp khi nhóm chốt công nghệ
- Số liệu free tier của Gemini lấy từ bài tổng hợp bên thứ ba (đến 30/09/2026), các nguồn lệch nhau; cần xem hạn mức thực tế tại ai.google.dev/gemini-api/docs/rate-limits trước khi chốt
- Dữ liệu gửi Gemini free tier có thể bị Google dùng cải thiện sản phẩm: chỉ gửi hóa đơn mẫu hoặc đã che mã số thuế và tên doanh nghiệp
- Chưa kiểm tra license từng bản phát hành của các model mã nguồn mở (đặc biệt LayoutLMv3 hạn chế thương mại)
- FPT.AI Reader chưa có bảng giá công khai, cần xem trực tiếp trên reader.fpt.ai
- Mục lục Word vẫn là mục lục tĩnh, số trang cần cập nhật tay khi nội dung đổi

## Giai đoạn 16 — Chọn OCR và bổ sung công nghệ backend/frontend vào báo cáo

**Mục tiêu giai đoạn:** Mục "Công nghệ áp dụng" ở Giai đoạn 15 mới chỉ có công nghệ AI dạng danh sách ứng viên; cần chọn một OCR cụ thể và đưa nốt các công nghệ còn lại (backend, frontend, cơ sở dữ liệu, môi trường) vào báo cáo.

**Đã làm:**

- Sắp xếp lại mục 7 của `Project_1.docx` thành 5 mục: 7.1 Công nghệ sử dụng trong hệ thống (bảng 12 dòng: AI Service gồm OCR/LLM/ML/nền tảng, Backend, Frontend, Cơ sở dữ liệu, Môi trường, Quản lý mã nguồn, Đầu vào XML, Đầu ra Sheet/Excel, Công cụ vẽ sơ đồ), 7.2 Lựa chọn công cụ OCR: PaddleOCR, 7.3 Danh sách công nghệ AI ứng viên (bảng 13 dòng cũ), 7.4 Bộ công nghệ AI tối thiểu, 7.5 Lưu ý khi áp dụng
- Cập nhật mục lục tĩnh (6 dòng, trang 41–46); render PDF kiểm tra, heading 6.4 vẫn ở trang 36
- Sửa bảng "Tech Stack" ở đầu file này cho khớp
- Kiểm tra tài liệu PaddleOCR: tiếng Việt (mã `vi`) nằm trong mô hình `latin_PP-OCRv5_mobile_rec` của PP-OCRv5

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** chọn OCR chính trong PaddleOCR, VietOCR, EasyOCR, Tesseract, Docling, Azure F0, Donut/LayoutLMv3, FPT.AI Reader
- **Đã chọn:** PaddleOCR (chạy local)
- **Vì sao:** có mô hình tiếng Việt, là bộ OCR đủ cả phát hiện vùng chữ lẫn nhận dạng (VietOCR chỉ nhận dạng nên phải ghép thêm bộ phát hiện), mã nguồn mở không giới hạn số trang, hóa đơn thật không phải gửi ra ngoài ở bước đọc chữ, cùng ngôn ngữ Python với AI Service. Đánh đổi: chỉ trả về chữ, cần LLM hoặc quy tắc để ra các trường; Azure F0 giữ lại để đối chứng nếu cần (500 trang/tháng), Tesseract/EasyOCR làm dự phòng, Docling xét thêm nếu bảng dòng hàng hóa đọc kém
- **Vấn đề:** hóa đơn điện tử XML có cần qua OCR không
- **Đã chọn:** đọc trực tiếp file XML, không qua OCR
- **Vì sao:** dữ liệu đã có cấu trúc sẵn, tránh sai số OCR; khớp với `file_type` trong bảng `invoices`

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa có số liệu thử nghiệm: cần chạy PaddleOCR trên bộ hóa đơn mẫu của nhóm và đo tỷ lệ đọc đúng từng trường (số hóa đơn, ngày, mã số thuế, tổng tiền, thuế suất) trước khi coi lựa chọn này là chốt
- LLM (dự kiến Gemini Flash free tier) chưa chốt; dữ liệu gửi lên chỉ là hóa đơn mẫu hoặc đã che mã số thuế và tên doanh nghiệp
- Thư viện ghi Google Sheet và xuất Excel chưa chốt
- Bảng ở mục 7.1 và 7.3 của báo cáo có vài dòng giống nhau về nội dung, nhóm có thể rút gọn 7.3 nếu báo cáo quá dài

## Giai đoạn 17 — Chốt bảng công nghệ áp dụng cho toàn dự án

**Mục tiêu giai đoạn:** Có một bảng công nghệ duy nhất cho cả dự án (nền tảng + AI), mỗi vai trò AI chỉ chọn một công nghệ từ danh sách model miễn phí của nhóm.

**Đã làm:**

- Gộp công nghệ nền tảng (AI Service, Backend, Frontend, Database, môi trường, mã nguồn, đầu vào/ra, sơ đồ) và công nghệ AI thành một bảng ở mục Tech Stack đầu file
- Chọn mỗi vai trò AI một công nghệ: OCR = PaddleOCR; LLM = Gemini Flash (free tier); chuẩn hóa tên = RapidFuzz; phân loại tài liệu = Logistic Regression; gợi ý định khoản = TF-IDF + Logistic Regression; so khớp PO = bge-m3; phát hiện bất thường = Isolation Forest
- Không chọn: PhoBERT, XGBoost/LightGBM, Docling, Donut/LayoutLMv3, Azure F0, Qwen/Gemma/Llama local, EasyOCR, Tesseract, VietOCR, FPT.AI Reader (giữ làm dự phòng hoặc đối chứng)
- Nhóm quyết định **không ghi mục "Công nghệ áp dụng" vào file Word nữa**: mục 7 trong bản `Project_1.docx` đã gửi ở Giai đoạn 15–16 không dùng, giữ file Word gốc của nhóm

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** LLM chọn Gemini Flash qua API hay model local (Qwen-VL, Gemma, Llama qua Ollama)
- **Đã chọn:** Gemini Flash free tier
- **Vì sao:** model local cần GPU hoặc máy đủ RAM, bản nhỏ chạy CPU chậm, mà yêu cầu là máy không nặng. Đánh đổi: dữ liệu gửi lên có thể được Google dùng cải thiện sản phẩm, chỉ gửi hóa đơn mẫu hoặc đã che mã số thuế và tên doanh nghiệp; hạn mức free tier thay đổi thường xuyên
- **Vấn đề:** có dùng PhoBERT và XGBoost/LightGBM không
- **Đã chọn:** chưa dùng
- **Vì sao:** PhoBERT fine-tune nên có GPU; TF-IDF + Logistic Regression đã đủ cho gợi ý định khoản và đơn giản hơn để giải thích khi bảo vệ. Có thể nâng cấp sau nếu độ chính xác chưa đạt

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa thử nghiệm: cần chạy PaddleOCR và Gemini Flash trên bộ hóa đơn mẫu, đo tỷ lệ đọc đúng từng trường
    - ✅ **Đã giải quyết ở Giai đoạn 26:** đo trên 12 ảnh giả, Gemini đúng 97/108 ô, regex 94/108
- Thư viện ghi Sheet/Excel (openpyxl, Google Sheets API) là đề xuất, chưa chốt
- Hạn mức Gemini free tier lấy từ nguồn bên thứ ba (đến 30/09/2026), cần xem lại tại ai.google.dev/gemini-api/docs/rate-limits trước khi dùng
- Chưa kiểm tra license từng bản phát hành của các model mã nguồn mở
- Chưa có dữ liệu lịch sử định khoản để huấn luyện bộ gợi ý định khoản

## Giai đoạn 18 — Tạo bộ hóa đơn giả để thử nghiệm OCR/LLM

**Mục tiêu giai đoạn:** Có dữ liệu thử nghiệm cho PaddleOCR và Gemini Flash khi không lấy được hóa đơn thật từ hoadondientu và việc xin hóa đơn thật khó.

**Đã làm:**

- Viết `gen_invoices.py`: sinh hóa đơn GTGT dữ liệu giả (tên công ty, MST, địa chỉ đều giả, có dòng "DỮ LIỆU GIẢ - CHỈ DÙNG THỬ NGHIỆM") ở 3 kiểu trình bày; xuất PDF, ảnh PNG (khoảng 60% bị làm nhiễu: nghiêng, mờ, nhiễu hạt, bóng đổ, giảm độ phân giải để giống ảnh chụp điện thoại), XML mô phỏng cấu trúc hóa đơn điện tử (đơn giản hóa, chưa kiểm tra với XSD chính thức), và nhãn chuẩn `labels.json`/`labels.csv`
- Viết `score_fields.py`: so kết quả trích xuất với nhãn, tính tỷ lệ đúng từng trường, tách ảnh sạch và ảnh nhiễu
- Chạy thử 40 hóa đơn: tổng tiền khớp từng dòng, số tiền bằng chữ đúng, XML đọc được, script chấm điểm chạy đúng trên dự đoán giả

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** lấy mẫu thử ở đâu khi hoadondientu không lấy được
- **Đã chọn:** sinh hóa đơn giả có nhãn chuẩn làm bộ thử chính
- **Vì sao:** biết đáp án ngay, không lộ dữ liệu doanh nghiệp, gửi Gemini thoải mái, tạo được số lượng tùy ý. Đánh đổi: không phản ánh hết sự đa dạng của hóa đơn thật

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Bộ giả chỉ có 3 kiểu trình bày, đẹp và đều hơn thực tế; kết quả đo trên bộ này cao hơn khi gặp hóa đơn thật, cần thử thêm vài hóa đơn thật đã che và ảnh biên lai MC-OCR (kiểm tra giấy phép trước)
- Chưa chạy PaddleOCR và Gemini Flash thật trên bộ giả, chưa có số liệu
    - ✅ **Đã giải quyết ở Giai đoạn 26**
- Cấu trúc XML là mô phỏng, cần đối chiếu với XML hóa đơn điện tử thật khi viết phần đọc XML
- Nếu báo cáo dùng số liệu từ bộ này phải ghi rõ là dữ liệu tổng hợp, mẫu nhỏ

## Giai đoạn 19 — Viết lại hướng dẫn cài đặt (README.md, SETUP.md) theo công nghệ đã chốt

**Mục tiêu giai đoạn:** Hai file hướng dẫn cho cả nhóm còn theo công nghệ cũ (Google Vision/Tesseract, GPT-4o-mini/Claude); cập nhật theo bảng công nghệ ở Giai đoạn 17 và thêm phần dữ liệu thử nghiệm.

**Đã làm:**

- `README.md`: bảng Tech Stack gộp nền tảng + AI; thêm thư mục `tools/`; thống nhất Python 3.12 và Node.js 20 LTS; thêm bước lấy Gemini API key ([https://aistudio.google.com/apikey](https://aistudio.google.com/apikey)); thêm `alembic upgrade head` vào bước chạy backend; thêm cổng 5173 của frontend; mục Test data và Working with Git (nhánh main được bảo vệ, bắt buộc PR); sửa lỗi định dạng và bỏ các câu còn sót từ cuộc trò chuyện (VD: "matches your earlier error")
- `SETUP.md`: viết lại mục 3.1 AI Service gồm cài PaddlePaddle CPU (`paddlepaddle==3.2.0` từ chỉ mục của paddlepaddle.org.cn), `requirements.txt` (lõi) và `requirements-ml.txt` (scikit-learn, sentence-transformers), torch bản CPU, tải model lần đầu, kiểm tra PaddleOCR; đổi biến môi trường `GOOGLE_APPLICATION_CREDENTIALS`/`LLM_API_KEY` thành `GEMINI_API_KEY`, `LLM_MODEL` là tên model Flash hiện hành; thứ tự khởi động dùng nhiều terminal thay vì `&` (không chạy trên Windows); thêm mục 6 Test data (gen_invoices.py, score_fields.py, cài poppler theo từng hệ điều hành) và mục 7 Troubleshooting (gồm Bug 1 Python 3.14 và lỗi dùng `.env.example` cũ)
- Kiểm tra tài liệu chính thức: `pip install -U google-genai`, khóa lấy tại aistudio.google.com/apikey, biến `GEMINI_API_KEY`; PaddleOCR cài PaddlePaddle trước rồi `pip install paddleocr`

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** tách thư viện AI Service thành một hay hai file requirements
- **Đã chọn:** hai file: `requirements.txt` (FastAPI, Pillow, PaddleOCR, google-genai, RapidFuzz...) và `requirements-ml.txt` (scikit-learn, sentence-transformers)
- **Vì sao:** torch và bge-m3 rất nặng (vài GB); thành viên chỉ làm OCR/LLM không phải tải ngay
- **Vấn đề:** ngôn ngữ của README/SETUP
- **Đã chọn:** giữ tiếng Anh như bản cũ
- **Vì sao:** thống nhất với quyết định ở Giai đoạn 5 (README kỹ thuật tiếng Anh, báo cáo đồ án tiếng Việt); có thể dịch nếu nhóm muốn

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- Không có bug mới ở giai đoạn này

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Các file trong repo phải sửa cho khớp với hai file hướng dẫn: `ai-service/requirements.txt` (bỏ `google-cloud-vision`, `pytesseract`; thêm `paddleocr`, `google-genai`, `rapidfuzz`, `python-dotenv`), tạo `ai-service/requirements-ml.txt`, `.env.example` (bỏ `GOOGLE_APPLICATION_CREDENTIALS`, `LLM_API_KEY`; thêm `GEMINI_API_KEY`, `LLM_MODEL`), `ai-service/app/main.py` nếu có đọc biến cũ, và chép `gen_invoices.py`, `score_fields.py` vào `tools/`
- Chưa kiểm tra cài PaddlePaddle trên Python 3.12 và trên từng hệ điều hành (Windows, Mac, CachyOS) của 3 thành viên; mỗi người cần chạy bước kiểm tra trong SETUP mục 3.1 và báo lỗi
- Tên model Gemini Flash thay đổi theo thời gian; tài liệu mẫu hiện dùng `gemini-3.8-flash`, cần xem model nào có free tier trước khi điền `LLM_MODEL`
    - ✅ **Đã giải quyết ở Giai đoạn 24 (Bug 10):** `gemini-2.5-flash` bị ngừng cấp (404), đã chuyển sang `gemini-3.8-flash` và chạy được
- Biến môi trường cho Google Sheets (service account) chưa thêm vì tính năng xuất Sheet chưa viết
- Migration Alembic cho 9 bảng chưa viết nên `alembic upgrade head` hiện chưa tạo đủ bảng

## Giai đoạn 20 — Thêm kiểu hóa đơn điện tử song ngữ vào bộ hóa đơn giả

**Mục tiêu giai đoạn:** Bộ hóa đơn giả ở Giai đoạn 18 còn đơn giản; cần thêm một kiểu có bố cục giống hóa đơn điện tử thường thấy trên cổng hoadondientu để thử PaddleOCR và Gemini sát thực tế hơn.

**Đã làm:**

- Thêm kiểu trình bày "portal" vào `gen_invoices.py` (cờ `--style portal` để chỉ tạo kiểu này; chế độ mặc định trộn 3 kiểu cũ và kiểu mới): nhãn song ngữ Việt-Anh, khung Mẫu số / Ký hiệu / Số, hàng đánh số cột (1, 2, 3, 4, 5, 6=4x5), dòng trống trong bảng, khung chữ ký số, dòng "Mã của cơ quan thuế" (chỉ với ký hiệu bắt đầu bằng C), mã tra cứu, dòng "Cần kiểm tra, đối chiếu khi lập, giao, nhận hóa đơn"
- Sửa nhãn: `mau_so` = 1 và `ky_hieu` dạng `C26TAA` (6 ký tự, C có mã cơ quan thuế, K không mã, năm 26, T/M, hai chữ cái) thay cho dạng gộp `1C26TAA` cũ; `score_fields.py` chấp nhận cả hai cách ghi; XML thêm thẻ `MCCQT` cho hóa đơn có mã
- Mọi chi tiết nhạy cảm đều giả và ghi rõ: tên công ty, MST, STK, ngân hàng "Mẫu Thử", chữ ký số ghi "CHỮ KÝ SỐ MẪU - KHÔNG CÓ GIÁ TRỊ", mã cơ quan thuế bắt đầu bằng "MẪU-THỬ-", website dùng tên miền `.example`, chìm chữ "MẪU THỬ NGHIỆM" và dòng "DỮ LIỆU GIẢ" ở chân trang
- Cập nhật SETUP.md mục 6 nói về `--style portal`

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** bố cục lấy theo đâu khi không vào được cổng hoadondientu
- **Đã chọn:** dựng theo bố cục chung của hóa đơn điện tử theo Thông tư 78 (nội dung bắt buộc và nhãn song ngữ phổ biến), không sao chép hóa đơn của doanh nghiệp nào
- **Vì sao:** không xem được bản gốc trên cổng; tài liệu tra cứu chỉ xác nhận nội dung bắt buộc (mẫu số, ký hiệu, số, người bán/mua, bảng hàng, thuế, tổng tiền, chữ ký số, mã cơ quan thuế) chứ không mô tả vị trí từng khối nên vị trí là ước lượng
- **Vấn đề:** có làm chữ ký số, mã cơ quan thuế, website trông như thật không
- **Đã chọn:** không, đều ghi rõ là mẫu/giả
- **Vì sao:** tránh tạo ra tài liệu có thể bị nhầm là hóa đơn thật; vẫn đủ để OCR/LLM học cách bỏ qua hoặc đọc các khối này

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Bố cục là ước lượng, chưa đối chiếu với một hóa đơn thật; cần so với 1 đến 2 hóa đơn thật đã che (xin từ người quen) để chỉnh vị trí khối, cỡ chữ, tên cột
- Hóa đơn thật thường có mã QR; chưa vẽ mã QR
- Bộ giả vẫn chỉ có 4 kiểu trình bày; số liệu đo trên bộ này cao hơn khi gặp hóa đơn thật

## Giai đoạn 21 — Đề xuất phân công code cho 3 người

**Mục tiêu giai đoạn:** Chia việc code để cả 3 thành viên đều làm đầy đủ AI, backend, frontend và CSDL, không ai chỉ làm một tầng. Nhóm quyết định bỏ việc vẽ thêm hình sơ đồ để tập trung vào code.

**Đã làm:**

- Soạn `PHAN_CONG.md` (đặt vào `docs/`): chia theo nghiệp vụ thành 3 gói. Gói A: NV1 + NV2 + đăng nhập (bảng users, invoices, invoice_fields; PaddleOCR, Gemini, RapidFuzz). Gói B: NV3 + NV4 (bảng chart_of_accounts, journal_entries, item_categories, category_aliases; TF-IDF + Logistic Regression). Gói C: NV5 + NV6 + NV7 + khung giao diện (bảng purchase_orders, matching_results; bge-m3, Isolation Forest)
- Có danh sách việc từng gói, quy tắc tránh xung đột (hợp đồng API + mock, một migration cho 9 bảng, mỗi người một thư mục, nhánh + PR, review chéo A→B→C→A), lộ trình 8 tuần gợi ý, điều kiện hoàn thành
- Sau đó viết lại `PHAN_CONG.md` gán tên: Giáp Minh Hiển = gói A, Đới Gia Khánh = gói B, Trần Nam Dương = gói C; mỗi người có bảng việc chi tiết (A1–A11, B1–B9, C1–C9) kèm tuần làm, endpoint và sản phẩm bàn giao; danh sách endpoint chung làm hợp đồng; thứ tự làm sớm để không chờ nhau (A3 dữ liệu demo, A7 `llm.py`, C2 khung giao diện); review chéo Hiển→Khánh→Dương→Hiển

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** chia theo tầng (AI, backend, frontend) hay theo nghiệp vụ
- **Đã chọn:** theo nghiệp vụ (cắt dọc)
- **Vì sao:** chia theo tầng thì mỗi người chỉ làm một mảng và không ai làm trọn một tính năng; cắt dọc thì ai cũng có đủ AI, backend, frontend, CSDL để trình bày khi bảo vệ. Đánh đổi: cần thống nhất hợp đồng API sớm và dùng mock để không chờ nhau
- **Vấn đề:** ai nhận gói nào
- **Đã chọn:** Hiển = A, Khánh = B, Dương = C
- **Vì sao:** xếp theo vai trò hiện có trong báo cáo, không theo năng lực code: Hiển giữ repo, nền OCR/LLM và tổng hợp báo cáo, gói A là gói gốc các gói khác dựa vào; Khánh viết user story/use case nên quen màn hình hiệu chỉnh và định khoản; Dương vẽ Activity nên quen luồng duyệt. Đổi được nếu nhóm muốn
- **Vấn đề:** nhiều người tạo migration Alembic song song
- **Đã chọn:** một migration ban đầu cho cả 9 bảng, sau đó chỉ một PR migration tại một thời điểm
- **Vì sao:** tránh nhiều "head" trong Alembic và lệch CSDL giữa các máy

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Nhóm họp xác nhận phân công theo tên, điều chỉnh nếu năng lực lệch nhau
- Điểm công sức (9, 10, 11 đv) là ước lượng thô; gói C nhiều phần ở mức Should/Could (PO, import, dashboard) nên làm Must (duyệt, xuất Sheet/Excel) trước
- Lộ trình 8 tuần là gợi ý, cần chỉnh theo hạn nộp thực tế
- Chưa có hợp đồng API chi tiết (đường dẫn, JSON vào/ra) và bản mock
- Bộ dữ liệu định khoản mẫu cho gói B chưa có; cần tạo vì chưa có lịch sử định khoản thật

## Giai đoạn 22 — Dựng lại thư mục dự án hoàn chỉnh (`ai-invoice-accounting.zip`)

**Mục tiêu giai đoạn:** Nhóm báo thư mục dự án cũ không giống bản đã thiết kế, nên dựng lại một thư mục đầy đủ khớp bảng công nghệ, ERD 9 bảng, tài liệu cài đặt và phân công đã chốt.

**Đã làm:**

- Dựng bộ khung: `docker-compose.yml` (MySQL 8.0, utf8mb4, healthcheck), `.env.example` (một `.env` duy nhất ở gốc), `.gitignore`
- Backend FastAPI: 9 model SQLAlchemy 2.0 theo từ điển dữ liệu, một migration Alembic `0001` tạo đủ 9 bảng, đăng nhập JWT + bcrypt thật (`/api/auth/login`, `/me`), hàm phân quyền `require_role` (chỉ KTT duyệt), `/api/health`, các endpoint hợp đồng của 3 gói trả dữ liệu mẫu có cờ `"mock": true`, script `seed_coa.py` (34 tài khoản TT133) và `seed_demo.py` (2 tài khoản demo, chỉ dùng khi dev), 5 test
- AI service FastAPI: `/health`, 8 endpoint mock (`/ocr`, `/extract`, `/normalize-vendor`, `/classify-gl`, `/classify-doc`, `/anomaly-score`, `/match-po`, `/summarize`), khung `llm.py`, 2 test
- Frontend React + Vite + Tailwind 3.4: đăng nhập, thanh điều hướng, chặn trang hàng chờ duyệt với KTV, 6 trang giữ chỗ gắn nhãn người phụ trách
- `tools/` (sinh và chấm hóa đơn giả), `data_test/` (12 hóa đơn giả mẫu), `docs/PHAN_CONG.md`, README.md và SETUP.md đã sửa
- Đã kiểm: `alembic upgrade/downgrade` trên SQLite, DDL MySQL biên dịch đúng (utf8mb4, InnoDB), pytest backend 5/5 và ai-service 2/2, `npm run build` thành công

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** cách giao khung để 3 người làm song song
- **Đã chọn:** endpoint mock có cờ `"mock": true`, người phụ trách làm xong thì chuyển sang router riêng và xóa khỏi `mock_contract.py`
- **Vì sao:** frontend dùng được ngay từ tuần 1, không ai chờ ai
- **Vấn đề:** SETUP.md ghi sai cơ chế utf8mb4 (`MYSQL_CHARACTER_SET`)
- **Đã chọn:** sửa thành cờ `--character-set-server` / `--collation-server` trong `command` của docker-compose
- **Vì sao:** biến môi trường đó không có trong image MySQL; cờ khởi động mới là cơ chế thật

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Thư mục này dựng lại từ tài liệu, không phải bản repo cũ của nhóm: phải so với lịch sử Git trước khi gộp, không ghi đè code đã có
- Chưa chạy thử với MySQL thật (môi trường dựng không có Docker); chỉ kiểm DDL và migration bằng SQLite
- `seed` tài khoản TT133 cần đối chiếu lại với thông tư
- `LLM_MODEL` (gemini-3.8-flash) phải kiểm tra lại model Flash miễn phí hiện hành
    - ✅ **Đã giải quyết ở Giai đoạn 24 (Bug 10)**
- Cài PaddlePaddle/PaddleOCR chưa thử trên mọi hệ điều hành; `requirements-ml.txt` chưa cài thử
- Gói `mock` còn lại là việc của A, B, C theo `docs/PHAN_CONG.md`

## Giai đoạn 23 — Gói A: đăng nhập, API hóa đơn, đọc XML, kiểm tra sơ bộ

**Mục tiêu giai đoạn:** Thay phần mock của gói A (NV1 phần tiếp nhận và đọc XML, NV2) bằng code thật theo `PHAN_CONG.md` (A2, A4, A5, A8, A9), để Khánh và Dương có hóa đơn thật trong CSDL mà gắn vào.

**Đã làm:**

- Đăng nhập JWT + bcrypt, 2 vai trò (kế toán viên, kế toán trưởng), hàm `require_role("KTT")` dùng chung (`backend/app/security.py`, `routers/auth.py`)
- API hóa đơn `POST /api/invoices`, `GET /api/invoices`, `GET /api/invoices/{id}`, `GET /api/invoices/{id}/file` (`routers/invoices.py`): kiểm tra đuôi file, chữ ký đầu file và dung lượng (≤ 10 MB), lưu file, tạo bản ghi `invoices` ở trạng thái `dang_xu_ly`; file gốc chỉ mở khi có token
- Đọc trực tiếp XML hóa đơn điện tử (`services/xml_parser.py`): dùng `defusedxml`, bỏ namespace, không qua OCR, độ tin cậy 100; hóa đơn nhiều mức thuế suất thì để trống thuế suất và cảnh báo
- Kiểm tra sơ bộ `POST /api/invoices/{id}/check` (`services/checks.py`, viết thuần hàm, có test): MST đúng định dạng, trước thuế + thuế = tổng thanh toán và thuế suất khớp tiền thuế (lệch tối đa 1 đồng), không trùng ký hiệu + số + MST
- Chuẩn hóa tên người bán `POST /normalize-vendor` (`ai-service/app/services/vendor.py`): bỏ dấu, bỏ dạng pháp lý (TNHH, CP...), so khớp RapidFuzz `token_sort_ratio` ≥ 88
- Frontend (`frontend/src/pages/invoices/`): tải nhiều file, danh sách có lọc, trang chi tiết, nút kiểm tra

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** kiểm tra mã số thuế đến mức nào
- **Đã chọn:** chỉ kiểm tra định dạng, không kiểm tra số kiểm tra và không tra cứu cơ quan thuế
- **Vì sao:** nhóm đã chốt không kết nối hoadondientu; MST trong bộ hóa đơn giả là số ngẫu nhiên nên kiểm tra số kiểm tra sẽ báo sai hết. Đánh đổi: không phát hiện được MST đúng định dạng nhưng không tồn tại
- **Vấn đề:** nghi ngờ trùng hóa đơn dựa vào đâu
- **Đã chọn:** bộ ba ký hiệu + số hóa đơn + MST người bán
- **Vì sao:** chỉ trùng số hóa đơn thì chưa chắc là cùng một hóa đơn (người bán khác nhau có thể cùng số); ba trường cùng trùng mới đáng cảnh báo

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- Không có bug mới được ghi lại ở giai đoạn này

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Hóa đơn nhiều mức thuế suất: XML chỉ để trống và cảnh báo, luồng ảnh chưa xử lý riêng
- MST chưa kiểm tra số kiểm tra, chưa đối chiếu cơ quan thuế (theo quyết định không kết nối hoadondientu)

## Giai đoạn 24 — Gói A: OCR, trích xuất bằng Gemini và chấm độ tin cậy (ai-service)

**Mục tiêu giai đoạn:** Làm phần AI của NV1 cho hóa đơn PDF và ảnh (A6, A7): đọc chữ, trích 9 trường, chấm độ tin cậy từng trường để kế toán biết ô nào phải kiểm tra, và để hóa đơn không bị kẹt khi Gemini lỗi.

**Đã làm:**

- OCR (`ai-service/app/services/ocr.py`, `preprocess.py`): PDF có lớp chữ thì lấy chữ trực tiếp; ảnh và PDF scan được tiền xử lý (xoay thẳng bằng phương sai ngang, phóng to, giảm nhiễu, tăng tương phản) rồi qua PaddleOCR; các ô chữ được gom thành dòng theo tọa độ
- Module LLM dùng chung `ai-service/app/llm.py` cho Khánh dùng lại: thử lại tối đa 3 lần, phân biệt hết hạn mức (429) và sai khóa
- Che dữ liệu nhạy cảm trước khi gửi Gemini (`masking.py`): MST, số tài khoản, số điện thoại, email; không che tên công ty
- Trích xuất `POST /extract` (`extractor.py`): Gemini trả JSON 9 trường, song song chạy bộ quy tắc regex (`rules.py`) để đối chiếu
- Chấm độ tin cậy 0–100 cho từng trường, lưu vào `invoice_fields.extracted_value` / `confidence_score`: điểm gốc 50, cộng/trừ theo giá trị có trong văn bản OCR (+20/−25), quy tắc regex đọc cùng (+15/−10), định dạng hợp lệ (+5/−20), ba trường tiền khớp phép cộng (+10/−20), độ tin cậy OCR của dòng chứa giá trị, và mức chắc chắn LLM tự báo (trọng số nhỏ)
- Backend chạy nền gọi `/extract`; hóa đơn chuyển sang `cho_xac_nhan`, hoặc `loi_xu_ly` nếu lỗi, có nút xử lý lại trên giao diện

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** lấy độ tin cậy từ số LLM tự báo hay tự tính
- **Đã chọn:** tự tính bằng công thức cộng/trừ điểm; số LLM báo chỉ là một thành phần trọng số nhỏ
- **Vì sao:** LLM chấm điểm thường lệch; một giá trị LLM bịa ra, không có trong văn bản OCR, vẫn xuống dưới 70 (ô đỏ) dù LLM báo 99 (có test cho trường hợp này). Đánh đổi: các ngưỡng cộng/trừ do nhóm chọn tay, chưa có căn cứ thống kê
- **Vấn đề:** khi Gemini không dùng được (không có khóa, hết hạn mức, máy chủ lỗi) thì xử lý thế nào
- **Đã chọn:** `/extract` rơi về kết quả regex kèm cảnh báo ghi lý do
- **Vì sao:** hóa đơn không bị kẹt ở trạng thái đang xử lý; kế toán vẫn nhận được kết quả có màu cảnh báo

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 9 — Ảnh mãi ở trạng thái "đang xử lý" khi không tải được mô hình PaddleOCR:** Hóa đơn ảnh không bao giờ ra kết quả → Nguyên nhân thật sự: mỗi yêu cầu đều thử tải lại mô hình và chờ hết thời gian → Cách sửa: ghi nhớ lỗi tải trong 5 phút, khởi động nóng mô hình khi bật server, thêm `ocr_ready` ở `/health`, thêm trạng thái `loi_xu_ly`
- **Bug 10 — Gemini không chạy, hệ thống âm thầm dùng regex:** Gemini không được gọi mà không báo lỗi → Nguyên nhân thật sự: `.env` đặt `LLM_MODEL=gpt-4o-mini` (tên model OpenAI dùng với khóa Gemini); sau khi đổi, `gemini-2.5-flash` bị Google ngừng cấp cho tài khoản mới (lỗi 404) → Cách sửa: đổi mặc định và `.env` sang `gemini-3.8-flash`; khi Gemini lỗi, hệ thống dùng regex và ghi lý do vào danh sách cảnh báo của kết quả
- **Bug 11 — Gemini trả 503 "quá tải":** Một số yêu cầu thất bại ngẫu nhiên → Nguyên nhân thật sự: máy chủ Gemini bận → Cách sửa: thử lại tối đa 3 lần; nếu vẫn lỗi thì dùng quy tắc, kết quả vẫn được trả
- **Bug 12 — Một số thư viện được cài mà không thông báo:** Một số thư viện được cài mà không có trong danh sách → Nguyên nhân thật sự: danh sách phụ thuộc chưa được rà soát → Cách sửa: rà lại trên môi trường sạch và liệt kê đầy đủ trong `requirements.txt` và `SETUP.md`

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Gemini bản miễn phí có thể dùng dữ liệu gửi lên để cải thiện sản phẩm; tên công ty không được che, nên chỉ gửi hóa đơn giả hoặc đã che tên
- Hóa đơn nhiều trang: PDF scan được OCR từng trang và ghép dòng, chưa tách riêng bảng dòng hàng
- Ngưỡng cộng/trừ điểm tin cậy chọn tay, cần xem lại bảng hiệu chỉnh khi có số liệu OCR trên hóa đơn thật

## Giai đoạn 25 — Gói A: sửa lỗi đọc ảnh PNG/JPG

**Mục tiêu giai đoạn:** Ban đầu ảnh PNG/JPG chỉ đọc được ngày lập trong khi PDF và XML đọc tốt; sửa bộ quy tắc và cơ chế tự sửa để ảnh đọc được đủ trường.

**Đã làm:**

- Viết lại quy tắc trong `ai-service/app/services/rules.py`: so khớp nhãn trên bản đã bỏ dấu, dấu `:` tùy chọn, sửa nhầm ký tự trong số tiền (`O/0`, `l/1`)
- Dùng nhãn tiếng Anh trong ngoặc làm điểm tựa, đọc ngày theo "…tháng…năm…", so khớp mờ bằng RapidFuzz
- Thêm cơ chế tự sửa số tiền lệch tối đa 2 chữ số bằng phép tính (thuế = trước thuế × thuế suất; tổng = trước thuế + thuế), đánh dấu "computed", độ tin cậy cố định 72 (vàng) kèm cảnh báo
- Thêm test với văn bản OCR bị làm hỏng (mất dấu, mất `:`, nhầm ký tự)

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** ký hiệu hóa đơn bị OCR đọc sai ký tự (`C26M8S` thay vì `C26MBF`) có nên tự sửa không
- **Đã chọn:** không tự sửa, để ô đỏ (độ tin cậy 52,8) cho kế toán kiểm tra
- **Vì sao:** đoán sai sẽ ra một ký hiệu hợp lệ nhưng sai, nguy hiểm hơn một ô đỏ rõ ràng; trong khi số tiền có phép tính để kiểm chứng nên tự sửa được
- **Vấn đề:** tiền thuế/tổng tiền tự tính lại có tin được như giá trị OCR đọc ra không
- **Đã chọn:** đánh dấu "computed" và cố định độ tin cậy 72 (vàng)
- **Vì sao:** giá trị suy từ phép tính không phải chữ đọc trực tiếp từ hóa đơn, nên để vàng cho kế toán kiểm tra

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 13 — Ảnh PNG/JPG chỉ đọc được ngày lập:** PDF và XML đọc tốt, ảnh thì thiếu gần hết trường → Nguyên nhân thật sự: quy tắc regex viết theo văn bản PDF sạch, không chịu được chữ OCR bị mất dấu, mất dấu `:`, nhầm `O/0`, `l/1` → Cách sửa: viết lại quy tắc trong `rules.py` (so khớp nhãn trên bản đã bỏ dấu, `:` tùy chọn, sửa nhầm ký tự trong số tiền), thêm test với văn bản bị làm hỏng
- **Bug 14 — Số hóa đơn đọc thành `0000001`:** Số hóa đơn luôn ra một dãy toàn số 0 → Nguyên nhân thật sự: regex bắt nhầm "Mẫu số (Form): 1" → Cách sửa: thêm điều kiện loại trừ "Mẫu số", chọn ứng viên có nhiều chữ số nhất
- **Bug 15 — Mất số hóa đơn khi OCR làm mất dấu `:`:** Số hóa đơn trống ở một số ảnh → Nguyên nhân thật sự: mẫu yêu cầu có dấu `:` → Cách sửa: thêm mẫu "Số" kèm 6–8 chữ số không cần `:` và mẫu "(No.)"
- **Bug 16 — Tiền thuế đọc thành `11717001717002`:** Sau khi sửa quy tắc, tiền thuế ra một dãy số rất dài → Nguyên nhân thật sự: mẫu số tiền cho phép khoảng trắng nên dính số của hai dòng → Cách sửa: bỏ khoảng trắng khỏi mẫu số
- **Bug 17 — Dòng hàng "Tiền thuê phòng họp" bị nhận làm tiền thuế VAT:** Tiền thuế bị lấy từ dòng hàng → Nguyên nhân thật sự: sau khi bỏ dấu, "thuê" và "thuế" giống nhau → Cách sửa: chỉ nhận "Tiền thuế" khi theo sau là GTGT/VAT hoặc `:`; thêm test riêng
- **Bug 18 — Ngày, thuế suất, tiền thuế, tổng bị bỏ sót và tiền đọc sai 1 chữ số trên ảnh PNG:** Thiếu trường, và có số như 310.548 thay vì 310.568, 4.192.660 thay vì 4.192.668 → Nguyên nhân thật sự: OCR đọc "Ngày" thành "Ngby", "Tiền thuế" thành "Tidn thué"; quy tắc quá cứng → Cách sửa: dùng nhãn tiếng Anh trong ngoặc làm điểm tựa, đọc ngày theo "…tháng…năm…", so khớp mờ bằng RapidFuzz; tự sửa số tiền lệch tối đa 2 chữ số bằng phép tính, đánh dấu "computed", độ tin cậy cố định 72 (vàng) kèm cảnh báo
- **Bug 19 — Ký hiệu hóa đơn đọc thành `C26M8S` (đúng là `C26MBF`):** Ký hiệu sai ký tự → Nguyên nhân thật sự: OCR đọc sai ký tự, Gemini chỉ thấy chữ OCR nên không sửa được → Cách sửa: không sửa tự động (xem quyết định trên); ô được tô đỏ (52,8) để kế toán kiểm tra

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Lỗi OCR đọc sai ký tự (ký hiệu hóa đơn, tên người bán) vẫn chưa có cách sửa tự động; chỉ cảnh báo màu
- Quy tắc được chỉnh theo 12 ảnh giả của nhóm, cần thử lại trên hóa đơn có bố cục khác

## Giai đoạn 26 — Gói A: đo độ chính xác và kiểm thử (A10)

**Mục tiêu giai đoạn:** Có số liệu thật cho PaddleOCR và Gemini trên bộ hóa đơn giả (giải quyết việc tồn đọng ở Giai đoạn 14, 16, 17, 18), và biết độ tin cậy xanh/vàng/đỏ có phản ánh đúng độ chính xác không. Viết phần báo cáo `BAO_CAO_GOI_A.md`.

**Đã làm:**

- Viết `tools/eval_extraction.py` (và `tools/_venv.py` để tự chạy lại bằng Python của `ai-service/venv`); chạy trên máy nhóm với PaddleOCR 3.x, Gemini `gemini-3.8-flash`, 12 ảnh PNG (6 sạch, 6 nhiễu), 9 trường mỗi ảnh = 108 ô
- Đo đọc XML trực tiếp: 12/12 hóa đơn khớp nhãn ở số hóa đơn, MST, tên, tiền trước thuế, thuế suất, tổng thanh toán
- Đo quy tắc regex trên 12 PDF có lớp chữ (không OCR, không LLM): 108/108 ô đúng (trường hợp dễ nhất, không phản ánh ảnh chụp); hiệu chuẩn: xanh 96/96 đúng, vàng 12/12 đúng (toàn là tên người bán), đỏ 0 ô
- Kiểm tra xoay thẳng ảnh: ảnh nghiêng 3° ước lượng lại với sai số ≤ 1°
- Test tự động: backend 21/21, ai-service 21/21, `npm run build` thành công
- Chạy thật qua HTTP: tải PDF và XML qua backend → ai-service → CSDL, phát hiện hóa đơn trùng, file gốc trả 401 khi không có token
- Kết quả đọc ảnh PNG (mẫu nhỏ, hóa đơn giả):

|Cấu hình|Tổng ô đúng|Ảnh sạch|Ảnh nhiễu|Thời gian/ảnh|
|---|---|---|---|---|
|PaddleOCR + regex (có tiền xử lý)|94/108 (87%)|49/54 (91%)|45/54 (83%)|14,2 s|
|PaddleOCR + Gemini (có tiền xử lý)|97/108 (90%)|49/54 (91%)|48/54 (89%)|19,0 s|
|PaddleOCR + regex, không tiền xử lý|chưa đo||||
|PDF ép qua PaddleOCR|chưa đo||||

- Theo trường (Gemini / regex): số ký hiệu 92% / 92%; số hóa đơn 92% / 83%; ngày lập 100% / 100%; MST 100% / 92%; tiền trước thuế 92% / 92%; thuế suất 100% / 92%; tiền thuế 100% / 100%; tổng thanh toán 92% / 92%; **tên người bán 42% / 42%** (5/12)
- Hiệu chuẩn độ tin cậy trên ảnh (số ô, tỉ lệ đúng): xanh ≥ 90: regex 89 ô 97%, Gemini 94 ô 94%; vàng 70–90: regex 15 ô 53%, Gemini 13 ô 69%; đỏ < 70: regex 4 ô 0%, Gemini 1 ô 0%
- Thử chuẩn hóa 12 tên người bán OCR sai với danh sách 6 nhà cung cấp: về đúng 12/12 (điểm khớp 90–100)

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** có nên bỏ bước người duyệt với ô xanh
- **Đã chọn:** giữ bước người duyệt cho mọi ô
- **Vì sao:** ô xanh vẫn sai 3–6% (Gemini 94%, regex 97% đúng), trong khi ô đỏ đều sai (0% đúng) nên cảnh báo màu có tác dụng nhưng chưa đủ để tự động hoàn toàn
- **Vấn đề:** sửa lỗi tên người bán thế nào
- **Đã chọn:** chuẩn hóa theo danh sách nhà cung cấp đã biết bằng RapidFuzz
- **Vì sao:** lỗi nằm ở OCR (bỏ hẳn hoặc đọc sai chữ có dấu phức tạp, ví dụ "Văn phòng phm Mu Th Cường" thay vì "Văn phòng phẩm Mẫu Thử", "Dch vu" thay vì "Dịch vụ"), Gemini chỉ thấy chữ OCR nên không khôi phục được. Đánh đổi: danh sách thử chỉ có 6 tên rất khác nhau nên là điều kiện dễ; nhà cung cấp mới chưa có trong danh sách thì kế toán vẫn phải sửa tay
- **Vấn đề:** ghi số liệu thế nào trong báo cáo
- **Đã chọn:** ghi rõ là hóa đơn giả, mẫu nhỏ, không suy ra độ chính xác trên hóa đơn thật
- **Vì sao:** bộ giả chỉ có vài kiểu trình bày, đẹp và đều hơn thực tế

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 20 — Tên người bán sai ở 7/12 ảnh:** Ô tên người bán chỉ đúng 5/12 → Nguyên nhân thật sự: OCR mất hoặc sai chữ có dấu, Gemini chỉ thấy chữ OCR → Cách sửa: chuẩn hóa theo danh sách nhà cung cấp bằng RapidFuzz; chưa giải quyết được với nhà cung cấp mới
- **Bug 21 — Công cụ debug/đánh giá đòi tự tạo venv ở thư mục gốc:** Công cụ đòi người dùng tự tạo venv ở thư mục gốc → Nguyên nhân thật sự: script dùng Python hệ thống thay vì venv của `ai-service` → Cách sửa: thêm `tools/_venv.py` tự chạy lại bằng Python của `ai-service/venv`
- **Bug 22 — Script đánh giá báo "Connection refused":** Chạy `eval_extraction.py` không ra kết quả → Nguyên nhân thật sự: chưa bật ai-service trước khi chạy; dừng server bằng `Ctrl+Z` chỉ tạm dừng nên cổng vẫn bị chiếm → Cách sửa: ghi rõ trong hướng dẫn bật ai-service rồi mới chạy, dừng server bằng `Ctrl+C`

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Hai cấu hình chưa đo: PaddleOCR + regex không tiền xử lý (để chứng minh tiền xử lý có ích) và PDF ép qua PaddleOCR
- Ở mẫu nhỏ này ảnh sạch không đọc tên tốt hơn ảnh nhiễu (đúng 1/6 so với 4/6), nên chưa kết luận được nhiễu làm OCR kém đi
- Chưa thử trên hóa đơn thật đã che (xin từ người quen) nên kết quả cao hơn thực tế khi gặp hóa đơn thật
- Thời gian 14–19 giây/ảnh trên CPU, chủ yếu do OCR
- Chưa tách bảng dòng hàng cho PDF nhiều trang; nhà cung cấp mới vẫn phải sửa tên tay

## Giai đoạn 27 — Gói A: bộ test khó `data_test_stress/` (nhiều biến thể lệch, nhiễu, nhòe)

**Mục tiêu giai đoạn:** Bộ `data_test/` chỉ có 12 hóa đơn, 3–4 kiểu trình bày và ảnh làm hỏng khá nhẹ (Giai đoạn 18, 26). Cần bộ test không trùng bộ cũ, có nhiều kiểu hỏng ảnh để đo OCR/LLM sát hơn, và có ca lỗi cố ý để thử kiểm tra sơ bộ (NV2) và đọc XML (A5).

**Đã làm:**

- Viết `tools/gen_invoices_stress.py` (dùng lại `gen_invoices.py`): `python tools/gen_invoices_stress.py --n 24 --per 6 --seed 20261018 --out data_test_stress`; mỗi hóa đơn gốc (`ST001`–`ST024`) xuất 1 bản sạch + 6 biến thể = **168 mục kiểm thử**, mỗi mục có đủ `images/<id>.png`, `pdf/<id>.pdf`, `xml/<id>.xml`; bản sạch có PDF lớp chữ, biến thể có PDF scan (chỉ có ảnh, buộc phải OCR)
- 34 biến thể làm hỏng ảnh chia 4 mức (nhẹ, trung bình, khó, cực khó): nghiêng 1,5–15°, quay 90/180/270°, mờ Gauss, mờ chuyển động, nhòe mực, vết nhòe cục bộ, nhiễu hạt, nhiễu muối tiêu, độ phân giải thấp, nén JPEG nặng, bóng đổ, chói sáng, tối, chữ nhạt kiểu giấy nhiệt, cắt lệch mép, chụp chéo (perspective), tem đỏ đè chữ, nếp gấp ngả vàng, ảnh nhị phân, fax, và các kiểu kết hợp
- Nội dung đa dạng: 1 dòng và 12 dòng hàng; tổng tiền hàng tỷ và vài chục nghìn; thuế suất 0/5/8/10%; ký hiệu, số hóa đơn, MST có ký tự dễ nhầm (0/O, 1/I, 8/B, 5/S); ngày đầu/cuối tháng/năm; 14 tên người bán mới; một số tên cũ viết hoa, viết tắt `Cty`, không dấu, kèm `ten_chuan_ky_vong` để thử `/normalize-vendor`
- 8 hóa đơn lỗi cố ý kèm `ky_vong_kiem_tra`: tổng sai, thuế sai, MST 9 chữ số, MST có chữ, lệch 1 đồng (phải thông qua), lệch 2 đồng (phải báo lỗi), MST 13 chữ số (hợp lệ), và bản trùng ST002 (ST024)
- 15 file XML đặc biệt trong `xml_edge/` kèm `xml_edge_labels.json`: namespace mặc định, nhiều thuế suất, thiếu trường, thẻ gốc sai, file cắt cụt, file rỗng, XXE, billion laughs, BOM, số dạng `17,462,720.00`, ngày có giờ, CDATA và ký tự đặc biệt, pretty-print, thuế suất KCT, MST chi nhánh
- Sửa `tools/eval_extraction.py`: thêm `--data data_test_stress` và bảng độ chính xác **theo từng biến thể** trong `report.md`
- Kiểm tra tạm: không trùng số hóa đơn/MST nào của `data_test/`; xem ảnh mẫu bằng mắt; chạy 24 XML sạch và 15 XML đặc biệt qua `backend/app/services/xml_parser.py`; chạy `rule_extract` trên 24 PDF sạch (chưa chạy PaddleOCR và Gemini vì môi trường dựng không có)

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** ghi nhãn cho hóa đơn lỗi cố ý là giá trị đúng hay giá trị hiển thị
- **Đã chọn:** nhãn = đúng những gì hiển thị trên hóa đơn (kể cả MST sai, tổng sai); phần đáng lẽ phải báo lỗi nằm ở `ky_vong_kiem_tra`
- **Vì sao:** phép đo trích xuất (đọc có đúng chữ trên giấy không) và phép đo kiểm tra sơ bộ (có phát hiện hóa đơn sai không) là hai việc khác nhau, tách riêng thì không bị lẫn
- **Vấn đề:** bao nhiêu biến thể trên mỗi hóa đơn
- **Đã chọn:** 1 bản sạch + 6 biến thể rút xoay vòng, mỗi biến thể xuất hiện 3–5 lần trên các hóa đơn và mẫu trình bày khác nhau
- **Vì sao:** cùng một hóa đơn có nhiều biến thể thì so sánh được biến thể nào làm hỏng kết quả; xoay vòng để không biến thể nào bị bỏ sót. Đánh đổi: các biến thể của cùng một hóa đơn có cùng ký hiệu + số + MST nên nạp hết qua `POST /api/invoices` sẽ bị báo trùng, phải đo bằng `eval_extraction.py`
- **Vấn đề:** xử lý các biến thể quay 90/180/270°
- **Đã chọn:** giữ lại, xếp mức "cực khó"
- **Vì sao:** `preprocess.py` chỉ chỉnh nghiêng nhỏ nên nhiều khả năng sai; ghi nhận như một hạn chế, không tính chung vào độ chính xác tổng

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 23 — Script sinh bộ test xóa nhầm ảnh và PDF bản sạch:** Sinh xong chỉ có 144 ảnh thay vì 168 và thiếu toàn bộ PDF `_clean` → Nguyên nhân thật sự: file tạm dùng chung tên với file kết quả của bản `clean` nên bị `unlink()` ở cuối vòng lặp → Cách sửa: file tạm đặt tên `_render_<id>`
- **Bug 24 — `parse_money` đọc sai gấp 100 lần với số dạng `49,632,990.00`:** XML trong `xml_dinh_dang_so_khac` ra tổng thanh toán 4.963.299.000 thay vì 49.632.990, không có cảnh báo → Nguyên nhân thật sự: hàm chỉ nhận riêng dạng `\d+\.\d{1,2}`, mọi dạng khác bị bỏ hết ký tự không phải số nên phần `.00` thành hai chữ số 0 → Cách sửa: **chưa sửa**
- **Bug 25 — MST có chữ cái bị bỏ chữ cái rồi mới kiểm tra:** `0172A69957` thành `017269957` (9 chữ số) → Nguyên nhân thật sự: `canonical("mst_seller")` bỏ mọi ký tự không phải số nên bằng chứng sai định dạng bị méo; hệ thống vẫn báo MST không hợp lệ nhưng giá trị hiển thị cho kế toán đã bị thay → Cách sửa: **chưa sửa**
- **Bug 26 — Regex đọc sai trên 4 PDF sạch của bộ mới (208/216 ô đúng):** ST011 và ST015 (MST sai định dạng) bị lấy thành MST người mua `0109999990`; ST003 (tổng sai) và ST019 (lệch 1 đồng) bị đọc sai tiền trước thuế và tiền thuế → Nguyên nhân thật sự: với MST, nghi do mẫu MST yêu cầu 10 chữ số nên MST 9 chữ số hoặc có chữ không khớp và rơi sang MST đầu tiên khớp được là của người mua; với hai hóa đơn tổng lệch, chưa xác định (nghi `_fuzzy_money` trong `rules.py`) → Cách sửa: **chưa sửa**

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa chạy `eval_extraction.py` trên bộ mới với PaddleOCR và Gemini nên chưa có số liệu độ chính xác theo biến thể; việc này bị giới hạn bởi hạn mức Gemini (xem Giai đoạn 28)
- Sửa Bug 24, 25, 26 rồi thêm test cho các ca này vào `backend/tests` và `ai-service/tests`
- Chưa kiểm tra hành vi `parse_date` với `2026-02-06T08:30:00` ngoài ca đã thấy trong `xml_edge`
- Bộ test vẫn là hóa đơn giả với 4 kiểu trình bày; số liệu sẽ cao hơn thực tế khi gặp hóa đơn thật
- Nếu nạp bộ này vào CSDL qua backend phải nạp mỗi hóa đơn gốc một lần (tránh cảnh báo trùng)

## Giai đoạn 28 — Gói A: tiết kiệm hạn mức Gemini, tránh lỗi 429 (`llm.py`)

**Mục tiêu giai đoạn:** Gemini bản miễn phí giới hạn theo **số yêu cầu** (ghi trong `llm.py`: khoảng 5 yêu cầu/phút và 20 yêu cầu/ngày, không phải giới hạn token), nên khi đo hàng chục ảnh hoặc tải nhiều hóa đơn liên tiếp thì gặp lỗi 429 và hết hạn mức trong ngày. Cần giảm số lượt gọi và tự dừng trước khi bị chặn.

**Đã làm:**

- Sửa `ai-service/app/llm.py`: thêm cache kết quả (cùng model + cùng nội dung gửi thì dùng lại, tắt bằng `LLM_CACHE=0`); giãn cách tối thiểu `LLM_MIN_INTERVAL` giây giữa hai lần gọi (mặc định 13 giây, để không vượt 5 yêu cầu/phút); ngân sách ngày `LLM_DAILY_LIMIT` (mặc định 18, `0` = không giới hạn), hết thì ném `LLMUnavailable` để `extractor.py` chuyển sang regex; số lần thử tối đa `LLM_MAX_ATTEMPTS` (mặc định 2, trước đây 3); mỗi lần gửi yêu cầu đều tính vào ngân sách, kể cả lần lỗi, vì Google cũng tính như vậy
- Ngày tính theo giờ Thái Bình Dương (`America/Los_Angeles`) vì theo ghi chú của bạn hạn mức ngày của Google đặt lại lúc nửa đêm theo giờ đó (khoảng 14–15 giờ Việt Nam); số lượt dùng lưu ở `ai-service/.cache/llm_usage.json` (đổi bằng `LLM_USAGE_FILE`)
- Thêm `tools/llm_usage.py`: xem số lượt đã dùng hôm nay (`python tools/llm_usage.py`), đặt lại cho khớp Google AI Studio (`set 17`), xóa cache (`clear-cache`); chỉ dùng thư viện chuẩn
- Sửa `ai-service/tests/test_invoice_ai.py`: fixture tự tắt cache, giãn cách, ngân sách ngày và trỏ file đếm sang thư mục tạm để các test không ảnh hưởng nhau; thêm 4 test cho `llm.py` (cache, ngân sách ngày dừng trước 429, lần lỗi vẫn tính và số lần thử bị giới hạn, giãn cách giữa các lần gọi)
- Kiểm tra: chạy lại bộ test trên bản repo đã thay 3 file, **28/28 đạt** (PaddleOCR được giả lập, Gemini không được gọi thật); thử `llm_usage.py` xem và đặt số lượt chạy đúng

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** tiết kiệm token hay tiết kiệm số lượt gọi
- **Đã chọn:** giảm số lượt gọi (cache, giãn cách, ngân sách ngày)
- **Vì sao:** hạn mức bản miễn phí tính theo số yêu cầu, nên rút ngắn prompt không giảm nguy cơ 429; mỗi hóa đơn một lần gọi là đơn vị cần tiết kiệm
- **Vấn đề:** khi hết ngân sách thì báo lỗi hay chuyển sang regex
- **Đã chọn:** ném `LLMUnavailable` rồi `extractor.py` rơi về regex kèm cảnh báo
- **Vì sao:** khớp với cơ chế rơi về quy tắc đã có (Giai đoạn 24), hóa đơn không bị kẹt. Đánh đổi: khi hết lượt, kết quả chỉ ở mức regex (87% so với 90% trên 12 ảnh giả)
- **Vấn đề:** giữ khóa trong lúc chờ giãn cách
- **Đã chọn:** giữ khóa (`threading.Lock`) khi chờ, các yêu cầu xếp hàng
- **Vì sao:** đảm bảo không bao giờ gửi hai yêu cầu cách nhau dưới 13 giây dù có nhiều hóa đơn xử lý nền cùng lúc. Đánh đổi: độ trễ cộng dồn (xem việc tồn đọng)

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 27 — Gemini trả 429 khi đo hoặc xử lý nhiều hóa đơn liên tiếp:** hóa đơn rơi về regex, mất phần cải thiện của LLM, và hạn mức trong ngày hết sớm → Nguyên nhân thật sự: mỗi hóa đơn gọi Gemini một lần, lần thử lại cũng tốn lượt, không có cache nên đo lại cùng một ảnh vẫn tốn lượt, và không có giãn cách giữa các lần gọi → Cách sửa: cache, giãn cách 13 giây, ngân sách ngày, giới hạn 2 lần thử như mô tả trên

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Sau khi nhận 429 thật, hệ thống vẫn gọi tiếp ở các yêu cầu sau và mỗi lần đều bị tính lượt (thử 3 lần 429 liên tiếp đều tăng số lượt); nên thêm thời gian nghỉ (ví dụ vài phút) sau 429 thay vì gọi lại ngay
- Giữa các lần thử lại, `time.sleep(2.0 * (attempt + 1))` vẫn chạy cả sau lần thất bại cuối (thừa khoảng 4 giây rồi mới báo lỗi)
- `backend/app/services/ai_client.py` đặt `timeout=180` cho `POST /extract`; mỗi hóa đơn có Gemini phải đợi 13 giây theo hàng, nên tải từ khoảng 13 hóa đơn trở lên cùng lúc (kèm 14 giây OCR mỗi ảnh) có thể quá thời gian chờ và rơi vào `loi_xu_ly` (nút xử lý lại vẫn dùng được); cần tăng `timeout` hoặc giới hạn số hóa đơn xử lý song song
- Đo bộ `data_test_stress` (168 mục) với ngân sách 18 lượt/ngày: chỉ khoảng 18 ảnh đầu dùng LLM, phần còn lại âm thầm chạy regex nên số liệu bị trộn hai cấu hình; cần chia nhiều ngày, chọn một tập con, hoặc chạy `--no-llm` rồi tách riêng; cache giúp chạy lại không tốn lượt
- Cách giảm lượt hiệu quả nhất chưa làm: bỏ qua LLM khi quy tắc regex đã chắc chắn (tất cả ô xanh)
- Giá trị mặc định `LLM_DAILY_LIMIT=18` được lặp ở hai file (`llm.py` và `llm_usage.py`); các biến `LLM_CACHE`, `LLM_MIN_INTERVAL`, `LLM_DAILY_LIMIT`, `LLM_MAX_ATTEMPTS`, `LLM_USAGE_FILE` chưa có trong `.env.example` và `SETUP.md`
- Thư mục `ai-service/.cache` (chứa kết quả LLM và nội dung hóa đơn đã gửi) chưa nằm trong `.gitignore`; cần thêm để không commit nhầm
- Hạn mức "5 yêu cầu/phút, 20 yêu cầu/ngày" và giờ đặt lại là số trong ghi chú của `llm.py`, chưa đối chiếu với trang hạn mức chính thức của Google (cùng việc tồn đọng ở Giai đoạn 15)

## Giai đoạn 29 — Giao diện sáng/tối cho toàn bộ web

**Mục tiêu giai đoạn:** Cho người dùng đổi giữa giao diện sáng và tối, áp dụng cho **toàn bộ web** (cả các trang của Khánh và Dương) mà không phải viết `dark:` ở từng class của từng trang.

**Đã làm:**

- `frontend/tailwind.config.js`: bật `darkMode: "class"`; đổi 13 họ màu có sẵn (slate, gray, red, orange, amber, yellow, green, emerald, sky, blue, indigo, violet, rose) × 11 sắc độ thành biến CSS `--c-<họ>-<sắc độ>` dạng `rgb(var(...) / <alpha-value>)` nên các class có độ trong suốt như `bg-slate-50/50` vẫn dùng được; `:root` giữ màu mặc định của Tailwind, `.dark` đổi sang sắc độ đảo (slate/gray đảo hoàn toàn: 50↔950, 100↔900...; các màu có sắc thái giữ 500–600 đủ đậm để nút chữ trắng vẫn đọc được, `blue-600` thành `blue-500`, từ 600 trở lên sáng lên để đọc trên nền tối)
- Plugin `addBase` trong cùng file: khai báo `colorScheme` sáng/tối, nền và chữ của `body` ở chế độ tối, nền cho ô nhập/chọn chưa có `bg-*` (dùng `:where` để `bg-*` vẫn đè được), màu viền mặc định ở chế độ tối, và `bg-white` được coi là bề mặt (thẻ, thanh điều hướng, ô nhập) nên đổi thành nền tối của thẻ; `text-white` giữ trắng
- `frontend/src/theme.js`: lưu lựa chọn ở `localStorage["theme"]`, mặc định theo hệ điều hành và tự đổi theo khi hệ điều hành đổi nếu người dùng chưa tự chọn; xuất `initTheme`, `setTheme`, hook `useTheme` (`useSyncExternalStore`)
- `frontend/index.html`: đoạn script nhỏ trong `<head>` thêm class `dark` trước khi React chạy, tránh nháy sáng khi tải lại trang
- `frontend/src/components/ThemeToggle.jsx`: nút tròn mặt trời/mặt trăng, `aria-label` tiếng Việt; đặt ở thanh điều hướng (`App.jsx`) và góc phải trang đăng nhập (`Login.jsx`)
- `frontend/src/main.jsx`: gọi `initTheme()` trước khi render
- Kiểm tra (tôi chạy): `npm run build` thành công (CSS 19,09 kB); chạy bản build bằng Chromium (Playwright) với API giả, kiểm các tổ hợp hệ điều hành sáng/tối × lựa chọn đã lưu sáng/tối/chưa chọn; bấm nút đổi rồi tải lại trang; hệ điều hành đổi khi chưa tự chọn; chụp trang đăng nhập, danh sách hóa đơn đủ 6 trạng thái và ngăn chi tiết kèm kết quả kiểm tra sơ bộ ở cả hai chế độ

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** làm chế độ tối bằng `dark:` ở từng class hay đổi cả bảng màu bằng biến CSS
- **Đã chọn:** đổi bảng màu bằng biến CSS ở `tailwind.config.js`
- **Vì sao:** ba người cùng viết giao diện, không ai phải nhớ thêm `dark:`, trang mới tự có chế độ tối. Đánh đổi: chỉ áp dụng cho màu thuộc 13 họ trên; màu cố định (hex, `bg-black`, `bg-white/80`, style inline, màu tùy ý `bg-[#...]`) không đổi theo; tên biến là sắc độ gốc (`--c-slate-50` là nền sáng nhất ở chế độ sáng nhưng là màu tối ở chế độ tối) nên đọc code dễ nhầm (đã có ghi chú trong file)
- **Vấn đề:** lưu lựa chọn ở đâu và mặc định thế nào
- **Đã chọn:** `localStorage`, mặc định theo hệ điều hành
- **Vì sao:** không cần CSDL hay API; khớp với thói quen người dùng. Đánh đổi: lựa chọn chỉ có hiệu lực trong từng trình duyệt
- **Vấn đề:** `bg-white` ở chế độ tối là đen hay màu thẻ
- **Đã chọn:** màu nền tối của thẻ (slate-100 sau khi đảo), không phải đen hoàn toàn
- **Vì sao:** `bg-white` trong code của nhóm luôn là bề mặt (thẻ, nav, bảng) nên cần tách khỏi nền trang để vẫn thấy khung

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 28 — Bấm về giao diện sáng nhưng trang vẫn ra nền tối, chữ trắng khi hệ điều hành đang ở chế độ tối:** (theo ghi chú trong `tailwind.config.js`) → Nguyên nhân thật sự: chưa khai báo `color-scheme: light`, nên trình duyệt dùng mặc định tối của hệ điều hành cho phần chưa đặt màu → Cách sửa: `:root { color-scheme: light }` và `.dark { color-scheme: dark }`; tôi kiểm lại bằng Chromium: hệ điều hành tối + chọn sáng cho ra giao diện sáng thật sự
- **Bug 29 — Viền và ô nhập quá sáng trên nền tối:** (theo ghi chú trong `tailwind.config.js`) → Nguyên nhân thật sự: viền mặc định của Tailwind (gray-200) và ô `input/select/textarea` không có `bg-*` không đi theo bảng màu đã đảo → Cách sửa: đặt lại màu viền cho `.dark *` và nền cho ô nhập bằng `:where(.dark) :is(input, select, textarea)`

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Các file dùng chung (`tailwind.config.js`, `index.html`, `main.jsx`, `App.jsx`) thuộc khung giao diện của Dương (C2 trong `PHAN_CONG.md`, ghi ở đầu `App.jsx`) và thay đổi bảng màu ảnh hưởng mọi trang; nên làm một PR riêng và nhờ Dương, Khánh xem lại trang của mình ở chế độ tối (đặc biệt màn hình đối chiếu có màu độ tin cậy xanh/vàng/đỏ)
- Mới kiểm bằng API giả với các trang gói A; các trang của gói B, C hiện chỉ là chỗ giữ chỗ, chưa kiểm
- Class có độ trong suốt như `bg-white/80` không khớp `[class~="bg-white"]` nên không được đổi nền; màu cố định và style inline cũng không đổi
- Ảnh/PDF hóa đơn gốc vẫn hiện nền sáng (là nội dung, chấp nhận)
- Chưa có test tự động cho `theme.js` (kiểm bằng Playwright thủ công, script không nằm trong repo); nút đổi chưa có `aria-pressed`

## Giai đoạn 30 — Thanh tiến độ khi tải và đọc hóa đơn

**Mục tiêu giai đoạn:** Khi tải hóa đơn PDF/ảnh, người dùng thấy thanh tiến độ chạy theo bước thật (tải lên → đọc chữ → AI → xong) thay vì chỉ thấy "Đang xử lý" suốt vài chục giây, và có thể tải nhiều file mà vẫn theo dõi riêng từng file.

**Đã làm:**

- `ai-service/app/progress.py` (mới): bộ nhớ tiến độ theo `job_id` trong RAM, mỗi mục `(percent, stage, thời điểm)`, khóa bằng `threading.Lock`, tự xóa mục quá 15 phút mỗi lần ghi; `set_progress` bỏ qua nếu không có `job_id`, `get_progress` trả `{"percent", "stage"}` hoặc `None`. Theo chú thích đầu file, `POST /extract` nhận thêm `job_id` tùy chọn và cập nhật qua từng bước thật, `GET /progress/{job_id}` trả bước hiện tại; mất khi khởi động lại dịch vụ nhưng không ảnh hưởng kết quả
- `ai-service/app/routers/invoice_ai.py`: `POST /extract` nhận thêm `job_id` (Form, tối đa 100 ký tự) và gọi `set_progress` ở 3 mốc: 15% "Đang đọc chữ trên tài liệu" (ngay khi nhận file, trước OCR), 55% "Đang trích xuất các trường bằng AI" (trước `extractor.extract_fields`), 95% "Đang lưu kết quả" (sau khi có kết quả); thêm `GET /progress/{job_id}` trả `{"data": {"percent", "stage"}}`, chưa có thì 404
- `backend/app/services/ai_client.py`: `extract_invoice` gửi thêm `job_id` = tên file đã lưu (`path.name`); thêm `get_progress(job_id)` gọi `GET /progress/{job_id}` với timeout 3 giây, lỗi mạng/JSON thì trả `None` (không làm hỏng luồng)
- `backend/app/routers/invoices.py`: thêm `GET /api/invoices/{invoice_id}/progress` (cần đăng nhập). Khi hóa đơn hết `dang_xu_ly` thì trả `percent = 100` với chữ "Hoàn tất" hoặc "Không đọc được hóa đơn" (nếu `loi_xu_ly`); còn đang xử lý thì hỏi ai-service, chưa có thì trả mặc định 5% "Đang chờ xử lý"
- `frontend/src/api/client.js`: thêm `uploadWithProgress(path, formData, onProgress)` dùng `XMLHttpRequest` vì `fetch` không báo được % tải lên; có gắn token, đọc lỗi `detail` giống hàm `api`
- `frontend/src/pages/.../Upload.jsx`: thanh tiến độ cho từng file trong hàng đợi, chia 0–25% là tải lên (`UPLOAD_SHARE = 25`), 25–100% là hệ thống đọc dữ liệu (quy đổi từ `percent` của server); hỏi tiến độ mỗi 1 giây (`POLL_MS`), tối đa 5 phút (`POLL_LIMIT_MS`), chịu được 4 lần lỗi liên tiếp, dừng khi rời trang (`alive`); hàm `watch` chạy song song nên file kế tiếp được tải ngay; XML đọc xong ngay khi tải lên nên bỏ qua bước hỏi tiến độ; thông báo thành công tự mất sau 8 giây; thanh có `role="progressbar"`, `aria-valuenow`, `aria-label`
- Kiểm tra: tôi đọc code 7 file (kể cả `main.py` và `invoice_ai.py` của ai-service), chưa chạy thử riêng phần này; về sau giai đoạn 31 có thêm test ai-service cho `/progress`. Qua đọc code: định dạng `{"data": ...}` của `/progress` khớp với `ai_client.get_progress`; 404 khi chưa có tiến độ làm backend trả mặc định 5%; lỗi OCR/AI không đặt tiến độ nhưng backend chuyển `loi_xu_ly` nên `/progress` của backend trả 100% "Không đọc được hóa đơn"; `job_id` là tên file đã lưu (ký tự chữ, số, `-`, `.`, tối đa khoảng 83 ký tự) nên không vượt giới hạn 100 của ai-service

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** cách đưa tiến độ từ ai-service lên giao diện
- **Đã chọn:** giao diện hỏi lại mỗi giây (polling) qua backend, backend hỏi tiếp ai-service
- **Vì sao:** đơn giản, không cần mở kết nối giữ lâu (SSE/WebSocket) và backend vẫn là cổng duy nhất kiểm tra đăng nhập. Đánh đổi: mỗi file đang đọc phát sinh một request/giây đến backend và một request đến ai-service
- **Vấn đề:** lưu tiến độ ở đâu
- **Đã chọn:** RAM của ai-service, tự xóa sau 15 phút
- **Vì sao:** tiến độ chỉ có ý nghĩa trong vài chục giây, không cần bảng CSDL hay migration. Đánh đổi: ai-service khởi động lại giữa chừng thì tiến độ về mặc định 5% cho đến khi xong (kết quả không mất vì nằm ở CSDL)
- **Vấn đề:** dùng gì làm `job_id`
- **Đã chọn:** tên file đã lưu trên đĩa (có tiền tố uuid 12 ký tự)
- **Vì sao:** ai-service không cần biết `invoice_id`, backend suy ra được từ `inv.file_path`. Đánh đổi: dựa vào tên file duy nhất; cùng một file xử lý lại sẽ dùng lại đúng `job_id` (xem tồn đọng)
- **Vấn đề:** thanh tiến độ cho phần đọc chữ lấy số từ đâu
- **Đã chọn:** 3 mốc % do ai-service báo ở đúng bước đang chạy (15 đọc chữ, 55 AI, 95 lưu), không chạy giả theo thời gian
- **Vì sao:** thanh phản ánh việc đang làm, và khi AI chờ giãn cách giữa các lần gọi (chống 429, giai đoạn 28) người dùng thấy đúng bước đang đứng
- **Vấn đề:** chia thanh giữa tải lên và đọc dữ liệu
- **Đã chọn:** 25% đầu cho tải lên, 75% sau cho đọc dữ liệu
- **Vì sao:** tải lên cục bộ nhanh còn đọc dữ liệu mới là phần lâu. Con số 25 là chọn tay, chưa đo

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- Chưa ghi bug nào cho giai đoạn này (bạn chưa báo lỗi nào khi làm); các điểm yếu tôi thấy khi đọc code nằm ở mục tồn đọng bên dưới

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Bấm “Xử lý lại” dùng lại đúng `job_id` cũ. `/extract` đã đặt lại về 15% khi bắt đầu nên lỗi thanh hiện mức cũ chỉ còn trong khoảng ngắn từ lúc bấm đến lúc tác vụ nền gọi tới ai-service (lúc đó có thể thấy mức 95% "Đang lưu kết quả" của lần trước, tối đa 15 phút trong RAM). Muốn hết hẳn thì backend nên xóa tiến độ cũ hoặc ai-service nhận thêm lệnh xóa
- Chỉ có 3 mốc (15/55/95) nên thanh đứng ở 15% suốt lúc OCR (lâu nhất, nhất là lần đầu nạp PaddleOCR) và ở 55% suốt lúc AI (gồm cả các lần chờ giãn cách và thử lại của giai đoạn 28, có thể hàng chục giây); nhãn "bằng AI" vẫn hiện cả khi `use_llm=false` hoặc luật đủ tin cậy nên không gọi AI. Muốn mượt hơn thì báo thêm mốc bên trong `ocr.run_ocr` (theo trang) và `extract_fields` (đọc chữ xong → luật → AI)
- `/extract` truyền `job_id` qua Form nhưng bộ nhớ tiến độ chỉ nằm trong một tiến trình; nếu chạy ai-service với nhiều worker (`--workers N`) thì `/progress` có thể rơi vào worker khác và trả 404 (giao diện thấy mặc định 5%). Hiện chạy một tiến trình thì không sao; cần ghi chú khi triển khai
- `GET /progress/{job_id}` ở ai-service không cần đăng nhập (ai-service là dịch vụ nội bộ); chỉ lộ tên bước và % của một `job_id` có tiền tố uuid nên rủi ro thấp, nhưng không nên mở ai-service ra ngoài
- `progress.py` có `percent` do ai-service tự báo, giao diện không chặn việc tụt ngược (ví dụ lần thử lại AI báo bước thấp hơn); nếu muốn mượt thì lấy `max` với giá trị đang hiển thị
- Khi `GET /progress` của ai-service lỗi hoặc ai-service đang bận, backend trả mặc định 5% "Đang chờ xử lý" nên người dùng không phân biệt được "đang chờ" với "mất kết nối"; sau 5 phút hoặc 5 lần lỗi giao diện chuyển sang “Đã nhận; xem kết quả trong danh sách” dù hóa đơn có thể vẫn đang xử lý
- Một file đang đọc tốn 1 request/giây đến backend và 1 request đến ai-service; tải nhiều file cùng lúc sẽ nhân lên. Chấp nhận ở quy mô đồ án, cần tăng `POLL_MS` hoặc gộp nhiều `invoice_id` vào một request nếu cần
- Endpoint `/progress` của backend chỉ yêu cầu đăng nhập, không kiểm tra hóa đơn có thuộc người dùng đó không (giống các endpoint khác trong gói A hiện nay)
- `ACCEPT.includes(...)` ở `Upload.jsx` kiểm bằng chuỗi con (đuôi như `.p` vẫn lọt); backend đã chặn bằng phần mở rộng và chữ ký file nên chỉ là chậm báo lỗi
- Test mới chỉ có `test_extract_reports_progress_by_job_id` ở ai-service (404 rồi 95%). Chưa có test: `progress.py` (TTL, `job_id` rỗng), endpoint `/progress` của backend (các trạng thái), `uploadWithProgress`; chưa chạy thử thực tế với nhiều file và Gemini thật

## Giai đoạn 31 — Hiển thị lỗi và kết quả kiểm tra rõ ràng hơn

**Mục tiêu giai đoạn:** Người dùng nhìn là biết mục nào đạt, mục nào sai, mục nào chưa kiểm tra được và sai ở đâu; thông báo lỗi từ backend, ai-service và AI viết bằng tiếng Việt có dấu, nói rõ nguyên nhân thay vì câu chung chung hay mã nội bộ.

**Đã làm:**

- `backend/app/services/checks.py`: `run_checks` trả thêm `issue_details` chia theo từng kiểm tra (`mst`, `totals`, `duplicate`) để giao diện hiện lỗi ngay dưới đúng dòng; `issues` (danh sách phẳng) giữ lại cho tương thích. Câu lỗi nói rõ giá trị: MST "Chưa có MST người bán" hoặc "Giá trị đọc được là '…', cần 10 hoặc 13 chữ số"; tổng tiền ghi cả phép tính (trước thuế + thuế = …, khác tổng ghi trên hóa đơn) và thuế suất (phải là … hóa đơn ghi …); trùng hóa đơn ghi số các hóa đơn trùng (`#12, #15`). `check_totals` trả `None` kèm lý do khi thiếu số liệu (thay vì coi là sai)
- `frontend/src/pages/invoices/InvoiceDetail.jsx`: mỗi kiểm tra có 3 trạng thái, mỗi trạng thái một câu riêng: đạt (✓ xanh), không đạt (✗ đỏ, nói rõ "không đúng"/"không khớp"/"trùng") và chưa kiểm tra được (? xám); nhãn nhỏ "Đạt / Không đạt / Chưa kiểm tra được" cạnh từng dòng; dòng tổng kết ("Có N mục không đạt…", "Chưa có lỗi, nhưng còn mục chưa kiểm tra được", "Tất cả mục kiểm tra đều đạt"); nếu server cũ chưa có `issue_details` thì dùng `issues`. Màn "Không đọc được hóa đơn" nêu nguyên nhân thường gặp (dịch vụ AI tắt hoặc hết hạn mức) và hướng "Xử lý lại"
- Thông báo lỗi tiếng Việt có dấu, đã sửa ở: `invoices.py` (không tìm thấy hóa đơn/file gốc, chỉ nhận PDF/PNG/JPG/XML, file lớn hơn 10 MB, nội dung không khớp đuôi file, ngày không hợp lệ, trạng thái không thể xử lý lại), `auth.py` ("Sai email hoặc mật khẩu"), `security.py` ("Chưa đăng nhập hoặc phiên đã hết hạn", "Không đủ quyền"), `ai_client.py` ("Không gọi được dịch vụ AI…", "Dịch vụ AI trả lỗi <mã>: …", "…sai định dạng"), `xml_parser.py` ("File XML không hợp lệ", "Không phải hóa đơn điện tử (thiếu thẻ gốc HDon)"), `llm.py` (thiếu khóa, hết hạn mức 429, khóa bị từ chối, JSON không hợp lệ, máy chủ Gemini lỗi liên tiếp, hết lượt trong ngày) và `extractor.py` (cảnh báo khi không dùng được Gemini, tắt Gemini, tổng tiền không khớp, đã sửa số theo phép tính)
- `ai-service/tests/test_invoice_ai.py`: cập nhật các điều kiện kiểm tra theo câu chữ mới (`"khác tổng"`, `"Gemini"`, `"hết 2 lượt"`) và thêm `test_extract_reports_progress_by_job_id` (đầu tiên 404, sau `/extract` thì tiến độ 95% và có nhãn bước)
- Kiểm tra (tôi chạy, ghép các file bạn gửi vào bản repo): ai-service 31 test đạt, backend 21 test đạt, `npm run build` frontend thành công. Chưa nhìn giao diện `CheckResult` bằng trình duyệt ở cả hai chế độ sáng/tối và chưa chạy với dữ liệu thật

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** kiểm tra "chưa đủ dữ liệu" nên hiện là đạt hay sai
- **Đã chọn:** thêm trạng thái thứ ba "chưa kiểm tra được" (xám, dấu ?), riêng cho tổng tiền khi thiếu số
- **Vì sao:** báo "không khớp" khi chỉ là thiếu số sẽ làm kế toán sửa nhầm; báo "đạt" thì nguy hiểm hơn. Đánh đổi: giao diện có 3 màu/3 câu để duy trì (xem tồn đọng về mục trùng hóa đơn chưa dùng trạng thái này)
- **Vấn đề:** gắn lỗi vào từng kiểm tra bằng cách nào mà không phá API cũ
- **Đã chọn:** thêm trường `issue_details` và giữ `issues`
- **Vì sao:** giao diện hiển thị lỗi ngay dưới dòng tương ứng, nơi nào còn dùng `issues` (tính năng khác, test cũ) vẫn chạy
- **Vấn đề:** chỉ sửa thông báo ở giao diện hay sửa tận nơi sinh ra lỗi
- **Đã chọn:** sửa chữ ngay ở backend/ai-service/llm
- **Vì sao:** thông báo đi tới người dùng qua nhiều lớp (ai-service → backend → giao diện), chữ có dấu từ gốc thì không cần dịch lại ở từng lớp. Đánh đổi: phải sửa cả các test đang so khớp chữ (đã sửa)

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- Chưa ghi bug nào cho giai đoạn này (bạn chưa báo lỗi nào khi làm); các điểm yếu tôi thấy khi đọc code nằm ở mục tồn đọng bên dưới

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Mục "Trùng hóa đơn" báo "✓ Không trùng hóa đơn đã có" cả khi không kiểm tra được: `find_duplicates` trả danh sách rỗng khi thiếu ký hiệu, số hóa đơn hoặc MST (ví dụ hóa đơn đang xử lý hoặc lỗi đọc), nên giao diện hiện đạt và có thể ra dòng tổng "Tất cả mục kiểm tra đều đạt" dù chưa kiểm tra. Nên cho `run_checks` trả thêm `duplicate_checked` và giao diện hiện trạng thái "chưa kiểm tra được" như mục tổng tiền
- MST thiếu (`None`) hiện nhãn "MST người bán không đúng định dạng" kèm dòng "Chưa có MST người bán"; nhãn chính nên là "chưa kiểm tra được" hoặc "thiếu MST" cho nhất quán với tổng tiền
- `GET /api/invoices/{id}` luôn kèm `checks`, kể cả khi hóa đơn đang `dang_xu_ly`/`loi_xu_ly` và các trường đều rỗng, nên ngăn chi tiết có thể hiện kết quả kiểm tra vô nghĩa trong lúc chưa đọc xong
- Còn câu chưa có dấu: `xml_parser.py` (cảnh báo nhiều mức thuế suất, "Thieu truong {tên_trường}" lộ tên trường nội bộ như `mst_seller`), `invoices.py` dòng gợi ý tên người bán ("Ten nguoi ban giong …", đang chỉ nằm trong `issues`, không vào `issue_details` nên không hiện trong giao diện), và 4 thông báo của ai-service trong `invoice_ai.py` (file lớn, file rỗng, cần gửi file hoặc text, không có tiến độ) vì `ai_client` nhúng nguyên 200 ký tự đầu của câu trả lời ai-service vào lỗi
- Thông báo 409 khi xử lý lại in nguyên mã trạng thái nội bộ ("Hóa đơn đang ở trạng thái dang_xu_ly…"); nên đổi thành nhãn tiếng Việt giống `StatusBadge`
- Chưa kiểm tra `invoice_data.process_invoice` (chưa có file): không biết lỗi `AIServiceError` có được lưu vào `reject_reason`/hiện cho người dùng không; nếu chỉ ghi log thì các câu lỗi mới của `ai_client` không bao giờ tới giao diện và màn "Không đọc được hóa đơn" vẫn dùng câu chung
- Chưa có test cho `check_totals`/`run_checks` với trường hợp thiếu số và cho nhánh `issue_details` ở giao diện; test ai-service so khớp chữ tiếng Việt nên sẽ vỡ nếu sửa câu chữ lần nữa (nên so theo mã lỗi nếu muốn bền)

## Giai đoạn 32 — Đánh dấu trang đang mở trên thanh điều hướng

**Mục tiêu giai đoạn:** Người dùng nhìn là biết mình đang ở trang nào: mục đang mở trên thanh điều hướng nổi bật và tiêu đề tab trình duyệt cũng ghi tên trang.

**Đã làm:**

- `frontend/src/App.jsx`: hàm `linkClass` cho `NavLink` — mục đang mở có nền xanh nhạt, chữ đậm, viền (`bg-blue-100 font-semibold text-blue-800 ring-1 ring-blue-300`), mục khác chỉ là chữ xanh (`text-blue-700 hover:bg-slate-100`); thuộc tính `aria-current="page"` do `NavLink` tự đặt; mục "Tải hóa đơn" (`/`) thêm `end` để không sáng ở mọi trang
- Cùng file: `useEffect` theo `pathname` đặt `document.title` thành "<tên trang> · AI Invoice" (tìm mục trong `NAV`; `/` khớp đúng, các mục khác khớp theo tiền tố), không khớp thì chỉ ghi "AI Invoice"
- Kiểm tra (tôi chạy): `npm run build` thành công; mở bản build bằng Chromium (Playwright, API giả) ở `/`, `/review`, `/entries`, `/po`, cả hệ điều hành sáng và tối — mỗi trang đúng một mục có `aria-current="page"` và tiêu đề tab đúng tên trang. Màu mục đang mở: chữ xanh đậm trên nền xanh nhạt ở chế độ sáng, chữ xanh nhạt trên nền xanh đậm ở chế độ tối (tương phản đủ rõ nhìn bằng mắt qua ảnh chụp thanh điều hướng); mục khác đậm 400, mục đang mở đậm 600

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** đánh dấu mục đang mở chỉ bằng màu chữ hay thêm nền và đậm
- **Đã chọn:** nền + chữ đậm + viền, không chỉ dựa vào màu
- **Vì sao:** không phụ thuộc phân biệt màu (người khó phân biệt màu vẫn thấy), và vẫn rõ ở cả hai giao diện sáng/tối nhờ bảng màu đã đảo ở giai đoạn 29
- **Vấn đề:** tự tính mục đang mở hay dùng `NavLink`
- **Đã chọn:** dùng `NavLink` kèm `end` cho `/`
- **Vì sao:** thư viện tự đặt `aria-current`, trình đọc màn hình biết trang hiện tại; không phải tự so sánh đường dẫn cho class

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- Chưa ghi bug nào cho giai đoạn này (bạn chưa báo lỗi nào khi làm); các điểm yếu tôi thấy khi đọc code nằm ở mục tồn đọng bên dưới

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- `App.jsx` thuộc khung giao diện của Dương (chú thích đầu file: C9); gộp chung PR riêng với phần giao diện sáng/tối của giai đoạn 29 và nhờ Dương xem lại
- Trang đăng nhập cũng bị đặt tiêu đề tab "Tải hóa đơn · AI Invoice" (vì `useEffect` chạy trước lệnh `return <Login />`); nên đặt "Đăng nhập · AI Invoice" khi chưa có `user`
- Tiền tố `startsWith` sẽ khớp nhầm nếu sau này thêm đường dẫn như `/policy` (khớp `/po`); đổi sang so khớp theo đoạn đường dẫn (`pathname === to || pathname.startsWith(to + "/")`)
- Đường dẫn không có trong `Routes` (ví dụ gõ `/abc`) hiện trang trống, tiêu đề chỉ ghi "AI Invoice"; chưa có trang 404
- Người dùng vai KTV vẫn thấy mục "Hàng chờ duyệt" trên thanh rồi bị chuyển về `/` khi bấm (không giải thích vì sao); nên ẩn mục này với KTV hoặc báo "Chỉ kế toán trưởng được vào"
- Thanh điều hướng chưa kiểm ở màn hình hẹp (điện thoại); `flex-wrap` có xuống dòng nhưng chưa xem bằng mắt

# Kiến thức đã áp dụng

- [[]]