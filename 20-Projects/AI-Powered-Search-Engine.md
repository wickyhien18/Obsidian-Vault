---
Hub: "[[MOC - AI & ML]]"
created: 2026-09-04
status: active
repo: https://github.com/wickyhien18/AI-Powered-Search-Engine
---

# Ý tưởng dự án

**AI-Powered Search Engine** — search engine mà bản thân cơ chế tìm kiếm dựa hoàn toàn vào AI (embedding + vector similarity), khác với AI-Integrated (AI chỉ là tính năng gắn thêm vào hệ thống search truyền thống).

# Tech Stack

|Thành phần|Công nghệ|Lý do chọn|
|---|---|---|
|Backend API|Python + FastAPI + Uvicorn|Ecosystem AI/ML native Python; học 1 lần dùng cho cả AI Engineering sau này|
|Orchestration|LangChain|Chuẩn hoá chunking, embedding wrapper, vector store wrapper — nằm trong JD|
|Vector DB|Qdrant|Mã nguồn mở, tự host dễ (Docker), hỗ trợ hybrid search (dense + sparse) native|
|Embedding (dense)|Ollama — `nomic-embed-text`|Chạy local, miễn phí, không cần gọi API ngoài|
|Embedding (sparse)|FastEmbed — `Qdrant/bm25`|Bổ sung keyword matching cho hybrid search|
|LLM (RAG generation)|Ollama — `llama3`|Chạy local; đánh đổi: chậm trên máy CPU-only|
|Frontend|Next.js (App Router) + TypeScript|Sát JD thị trường hơn Vite thuần; TypeScript bắt lỗi type khi response shape đổi|

# Nhật ký kỹ thuật

## Giai đoạn 1 — Setup môi trường

**Mục tiêu giai đoạn:** Chuẩn bị đầy đủ công cụ nền tảng (Python, FastAPI, Docker, Qdrant, Ollama) chạy được trên máy trước khi viết bất kỳ logic nào.

**Đã làm:**

- Tạo Python venv riêng cho project
- Cài FastAPI + Uvicorn, xác nhận chạy được endpoint `/health`
- Cài Docker, chạy Qdrant qua container
- Cài Ollama, pull model embedding `nomic-embed-text`

**Bug gặp phải:**

- **Bug 1 — `externally-managed-environment`:** `pip install` báo lỗi do PEP 668 trên Arch/CachyOS → nguyên nhân là venv chưa activate đúng, sau đó phát hiện thêm venv thiếu sẵn `pip` do thiếu `ensurepip` → sửa bằng `python -m ensurepip --upgrade` để cài `pip` vào trong venv

**Việc còn tồn đọng từ giai đoạn này:** Không có.

---

## Giai đoạn 2 — Chuẩn bị dataset

**Mục tiêu giai đoạn:** Có dữ liệu sạch, sẵn sàng để ingest, không tốn công tự crawl/tự dọn dữ liệu ở bước đầu học pipeline.

**Đã làm:**

- Chọn dataset BBC Articles Cleaned (Kaggle) — 2225 bài báo, 2 cột `text` + `category`
- Setup `.env` + `python-dotenv` để quản lý Kaggle credentials, thêm `.env` vào `.gitignore`
- Viết `setup.py` dùng `KaggleApi` để tải dataset tự động

**Quyết định kỹ thuật:**

- **Vấn đề:** chọn dataset tiếng Anh hay tiếng Việt cho bước học đầu tiên
- **Đã chọn:** tiếng Anh (BBC Articles)
- **Vì sao:** `nomic-embed-text` support tiếng Anh tốt hơn nhiều so với tiếng Việt — ưu tiên học đúng cơ chế pipeline (chunk → embed → search) trước, tránh vướng thêm vấn đề chất lượng embedding đa ngôn ngữ cùng lúc

**Bug gặp phải:**

- **Bug 2 — Kaggle đổi hệ thống token:** nhận được chuỗi token thay vì file `kaggle.json` kiểu cũ → xử lý bằng biến môi trường `KAGGLE_USERNAME`/`KAGGLE_KEY` trong `.env` thay vì file JSON

**Việc còn tồn đọng từ giai đoạn này:** Không có.

---

## Giai đoạn 3 — Ingest Pipeline

**Mục tiêu giai đoạn:** Biến dữ liệu CSV thô thành vector có thể tìm kiếm được, lưu trong Qdrant.

**Đã làm:**

- Viết `config.py` tập trung config dùng chung (`QDRANT_URL`, `EMBEDDING_MODEL`, `COLLECTION_NAME`...)
- Viết `ingest.py`: load CSV → chunk bằng `RecursiveCharacterTextSplitter` (`chunk_size=500`, `chunk_overlap=50`) → embed qua Ollama → lưu Qdrant theo batch 50
- Kết quả: 2225 bài báo → 7583 chunk

**Quyết định kỹ thuật:**

- **Vấn đề:** cắt nhỏ văn bản kiểu gì để không mất ngữ cảnh mà vẫn đủ nhỏ cho embedding
- **Đã chọn:** `RecursiveCharacterTextSplitter` thay vì cắt cố định theo ký tự (naive split)
- **Vì sao:** naive split cắt mù, dễ làm gãy giữa từ (ví dụ "against" → "aga" + "inst"); recursive split ưu tiên cắt theo đoạn/câu/từ trước, chỉ cắt thô khi cần thiết

**Bug gặp phải:**

- **Bug 3 — Qdrant `Connection refused`:** container Qdrant không tự khởi động lại sau khi tắt máy/Docker → sửa bằng `docker start qdrant`, về sau thêm `--restart unless-stopped` để tự khởi động
- **Bug 4 — Duplicate dữ liệu khi chạy `ingest.py` 2 lần:** `add_documents()` mặc định tạo ID ngẫu nhiên mỗi lần gọi → chạy lại tạo ra bản sao thay vì ghi đè → sửa bằng deterministic ID (`uuid5` hash từ `article_id + vị_trí_chunk + nội_dung`), giúp re-run trở thành upsert thay vì insert

**Việc còn tồn đọng từ giai đoạn này:** Không có.

---

## Giai đoạn 4 — Search API

**Mục tiêu giai đoạn:** Xây endpoint để người dùng gửi câu hỏi và nhận lại các chunk liên quan nhất.

**Đã làm:**

- Viết endpoint `/search` trong `main.py`: embed query → `client.query_points()` → trả kết quả kèm text + metadata

**Bug gặp phải:**

- **Bug 5 — `article_id`/`category` trả về `null`:** `QdrantVectorStore` lưu metadata lồng trong key `"metadata"`, không phải ở top-level payload → code ban đầu tìm sai chỗ (`payload["article_id"]`) → sửa bằng `payload.get("metadata", {}).get("article_id")`

**Việc còn tồn đọng từ giai đoạn này:** Không có.

---

## Giai đoạn 5 — RAG (Retrieval-Augmented Generation)

**Mục tiêu giai đoạn:** Thay vì chỉ trả chunk thô, để LLM đọc các chunk đó và tổng hợp thành câu trả lời thật.

**Đã làm:**

- Thêm endpoint `/ask`: retrieval (dùng lại logic của `/search`) → augmentation (ghép chunk thành `context_block`, nhét vào prompt) → generation (`ChatOllama` với model `llama3`)
- Prompt ép LLM chỉ trả lời dựa trên context được cung cấp, giảm hallucination
- Response trả cả `answer` và `sources` để có thể truy ngược nguồn

**Quyết định kỹ thuật:**

- **Vấn đề:** dùng model nào để sinh câu trả lời
- **Đã chọn:** `llama3` qua Ollama (chạy local)
- **Vì sao:** giữ toàn bộ pipeline chạy local, miễn phí, nhất quán với `nomic-embed-text` đã dùng — đánh đổi: chậm trên máy không có GPU rời

**Bug gặp phải:** Không có (chạy đúng ngay lần đầu, kết quả grounded — kiểm chứng qua test "why are musicians protesting?" trả lời đúng dựa trên 2 chủ đề có thật trong `sources`).

**Việc còn tồn đọng từ giai đoạn này:**

- `llama3` sinh chậm trên máy CPU-only — quyết định hoãn tối ưu lại sau. Hướng xử lý dự kiến: đổi model nhỏ hơn, quantization thấp hơn, giới hạn `num_predict`, hoặc thêm streaming để cải thiện cảm giác chờ

---

## Giai đoạn 6 — Frontend

**Mục tiêu giai đoạn:** Có giao diện để nhập câu hỏi, thay vì gọi API bằng `curl`/Swagger.

**Đã làm:**

- Dựng Next.js App Router + TypeScript, gọi `/ask` qua `fetch`
- Định nghĩa `interface AskResponse` khớp đúng shape response của `main.py`

**Quyết định kỹ thuật:**

- **Vấn đề:** dùng React + Vite hay Next.js
- **Đã chọn:** Next.js
- **Vì sao:** phổ biến hơn trong JD thị trường thực tế
- **Vấn đề khác:** JavaScript hay TypeScript
- **Đã chọn:** TypeScript
- **Vì sao:** bắt lỗi type ngay lúc compile nếu response shape của backend đổi (ví dụ gõ nhầm tên field), thay vì lỗi âm thầm lúc chạy trên trình duyệt

**Bug gặp phải:**

- **Bug 6 — CORS chặn request:** trình duyệt chặn request từ Next.js dev server (port 3000) tới FastAPI (port 8000) do khác origin → sửa bằng `CORSMiddleware` trong `main.py`, khai báo đúng origin
- **Bug 7 — `Module not found: Can't resolve 'fs'`:** file `config/env.ts` dùng `loadEnvConfig` (đọc file bằng Node `fs`, chỉ chạy được server-side) nhưng bị import vào component có `'use client'` (chạy trình duyệt, không có filesystem) → sửa bằng cơ chế `NEXT_PUBLIC_` prefix có sẵn của Next.js, bỏ hẳn `loadEnvConfig`
- **Bug 8 — Lẫn lộn file Vite và Next.js:** unzip nhầm file Next.js vào thư mục cũ còn sót file Vite (`src/`, `index.html`, `vite.config.js`) → xoá sạch thư mục, unzip lại vào thư mục trống

**Việc còn tồn đọng từ giai đoạn này:** Không có.

---

## Giai đoạn 7 — VS Code chạy full-stack bằng F5

**Mục tiêu giai đoạn:** Chạy đồng thời backend (Uvicorn) và frontend (Next.js dev server) chỉ bằng 1 phím F5, không cần mở 2 terminal thủ công.

**Đã làm:**

- Viết `.vscode/launch.json` với 2 config riêng (`debugpy` cho FastAPI, `node-terminal` cho `npm run dev`), gộp thành 1 `"compounds"` tên "Run Full Stack"

**Bug gặp phải:**

- **Bug 9 — F5 chỉ chạy backend:** dropdown Run & Debug đang chọn config "Backend (FastAPI)" riêng lẻ, không phải "Run Full Stack" → sửa bằng cách chọn đúng entry "Run Full Stack" trong dropdown trước khi nhấn F5

**Việc còn tồn đọng từ giai đoạn này:** Không có.

---

## Giai đoạn 8 — Hybrid Search

**Mục tiêu giai đoạn:** Khắc phục điểm yếu của semantic search thuần — tìm kém với tên riêng/từ hiếm (ví dụ "kapranos") vì embedding không mã hoá được "ý nghĩa" của một cái tên.

**Đã làm:**

- Thêm sparse vector (BM25 qua FastEmbed, model `Qdrant/bm25`) song song với dense vector đã có
- Đổi schema collection Qdrant từ 1 vector không tên → 2 vector có tên (`"dense"`, `"sparse"`)
- Cấu hình `QdrantVectorStore` với `retrieval_mode=RetrievalMode.HYBRID` để tự động embed + search cả 2 loại vector, gộp kết quả bằng RRF (Reciprocal Rank Fusion)

**Quyết định kỹ thuật:**

- **Vấn đề:** implement hybrid search bằng cách gộp thủ công 2 hệ thống search riêng (Qdrant + 1 hệ keyword search khác), hay dùng sparse vector native trong Qdrant
- **Đã chọn:** sparse vector native trong Qdrant
- **Vì sao:** chỉ 1 database, 1 lần query, không cần thêm hệ thống thứ 2 để vận hành/bảo trì

**Bug gặp phải:**

- **Bug 10 — Schema cũ không tương thích:** chạy `ingest.py` mới (schema 2 vector có tên) lên collection cũ (1 vector không tên) → `QdrantVectorStoreError` khi validate lúc khởi tạo `QdrantVectorStore` → sửa bằng cách xoá collection cũ (`DELETE /collections/bbc_news`) rồi ingest lại từ đầu

**Xác nhận hoạt động đúng:** Search "kapranos" → đúng 2 bài chứa từ này lên đầu kết quả, điểm số `0.5`/`0.333` khớp chính xác công thức RRF thực tế của Qdrant (`1/(vị_trí + 2)`).

**Việc còn tồn đọng từ giai đoạn này:** Không có.

# Kiến thức đã áp dụng
- [[Qdrant]]
- [[Kaggle]]