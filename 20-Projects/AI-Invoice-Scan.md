---
Hub: "[[]]"
created: 2026-09-07
status: active
repo: https://github.com/wickyhien18/ai-invoice-accounting
---

# Ý tưởng dự án
Hệ thống AI hỗ trợ nhập liệu hóa đơn/chứng từ cho kế toán — đồ án tốt nghiệp nhóm 3 sinh viên năm 4 CNTT (UTC Hà Nội). Mục tiêu: tự động hóa việc đọc, trích xuất, đối chiếu và gán bút toán cho hóa đơn, giảm thời gian nhập liệu thủ công và sai sót cho kế toán viên.

7 nghiệp vụ cốt lõi: (1) OCR số hóa hóa đơn, (2) trích xuất dữ liệu có cấu trúc, (3) hiểu ngữ cảnh (NLP), (4) chấm điểm độ tin cậy, (5) đối chiếu chứng từ (2-way/3-way matching), (6) gán bút toán (GL Coding) theo Thông tư 133/2016/TT-BTC, (7) phân tích & báo cáo xu hướng chi tiêu.

Định hướng kỹ thuật: dùng AI có sẵn qua API (OCR cloud + LLM), không tự train model — đảm bảo máy chạy nhẹ, triển khai được trong ~4 tháng.

# Tech Stack

|Thành phần|Công nghệ|Lý do chọn|
|---|---|---|
|AI Service|Python + FastAPI, Google Cloud Vision API/Tesseract, LLM API (GPT-4o-mini/Claude)|Không cần tự train model, gọi API cloud xử lý OCR/NLP nặng, giữ máy dev nhẹ|
|Backend|Python + FastAPI, SQLAlchemy, Alembic|Đồng nhất ngôn ngữ với AI Service, ORM + migration rõ ràng cho 3 người cùng code|
|Frontend|React + Vite + Tailwind CSS|Yêu cầu ban đầu của người dùng, hệ sinh thái phổ biến, dễ tuyển dụng hỗ trợ sau này|
|Database|MySQL 8.0 (chạy qua Docker)|Đồng bộ phiên bản giữa 3 máy thành viên, tránh lỗi charset/cấu hình lệch nhau|

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
- Cần thống nhất lại tên use case giữa bản vẽ tay của người dùng (VD: "Tải hóa đơn và nhận dạng hóa đơn", "Kiểm tra thông tin và rủi ro của hóa đơn") với danh sách use case đã viết trong Chương 2 — hiện đang lệch tên gọi giữa 2 nơi, cần đồng bộ trước khi hoàn thiện báo cáo

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

# Kiến thức đã áp dụng
- [[]]