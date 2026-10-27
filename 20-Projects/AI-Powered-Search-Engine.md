---
Hub: "[[MOC - AI & ML]]"
created: 2026-09-04
status: active
repo: https://github.com/wickyhien18/AI-Powered-Search-Engine
---

# Ý tưởng dự án

> **Hiện trạng (đọc phần này trước):** dự án hiện là **Perplexity clone**. Hướng ban đầu (search engine trên kho BBC, Giai đoạn 1–17) đã **dừng từ Giai đoạn 18**. Nhật ký giữ nguyên các giai đoạn đó để tra cứu, nhưng các mô tả về semantic search, Qdrant, embedding trong Giai đoạn 1–17 **không còn đúng với bản hiện tại**.

- **Loại hình:** AI search engine trả lời có trích dẫn. Web search lấy nguồn, LLM tổng hợp câu trả lời và gắn `[n]` trỏ về URL gốc. AI là cơ chế lõi, không phải tính năng gắn thêm.
- **Mục đích:** Portfolio project cho vị trí AI Engineer / Junior Backend Engineer, bám sát JD (RAG pipelines, LangChain, Vector Databases, LLM Agents).
- **Hướng đã dừng:** search engine trên kho BBC (semantic/hybrid search bằng embedding + Qdrant), Giai đoạn 1–17, đã deploy (Render + Vercel + Qdrant Cloud).
- **Hướng hiện tại:** Perplexity clone (Giai đoạn 18–20): câu hỏi → web search → LLM tổng hợp có trích dẫn.
- **Trạng thái:** backend v1 chạy end-to-end và đã test với Tavily/Groq thật (Giai đoạn 20); đã lọc footnote Wikipedia khỏi snippet (Giai đoạn 21). Frontend đã cập nhật theo API v2 (Giai đoạn 23) nhưng chưa deploy; rate limit cho `/ask` đã viết nhưng nằm trên nhánh riêng, chưa merge (Giai đoạn 22); chưa có số đo nào cho bản mới (Giai đoạn 24 mới chuẩn bị bộ 20 câu `eval_queries.json`, chưa chạy). Theo trang repo, nhánh `main` trên GitHub vẫn là bản BBC "Wire Search" trong khi CV đã trỏ vào repo và demo; bạn báo sau đó đã merge và deploy v2, chưa xác nhận được (xem Giai đoạn 24).

# Tech Stack

| Thành phần | Công nghệ | Lý do chọn |
| ---------- | --------- | ---------- |
| Backend API | Python + FastAPI + Uvicorn | Ecosystem AI/ML native Python; học 1 lần dùng cho cả AI Engineering sau này |
| Web search | Tavily API (`search_depth="basic"`), bọc trong `search.py` | Free 1.000 credit/tháng, không cần thẻ, trả nội dung đã làm sạch; 1 credit mỗi lần gọi (đã đo ở Giai đoạn 20). Đổi nhà cung cấp chỉ sửa `search.py` |
| LLM | Groq: `openai/gpt-oss-20b` qua `langchain-groq` | Chọn ở Giai đoạn 16 để deploy free tier (bỏ Ollama). Là reasoning model nên `max_tokens=2048`, hết token sẽ trả rỗng (Bug 19) |
| Orchestration | LangChain (bản hiện tại chỉ còn `langchain-groq` và `langchain-core`) | Chuẩn hoá wrapper cho LLM và message (`ChatGroq`, `HumanMessage`/`AIMessage`); nằm trong JD. Bản BBC còn dùng cho chunking, embedding và vector store |
| Lưu hội thoại | SQLite (`chat_history.db`, `db.py`) | Giữ nguyên từ bản BBC. Lưu ý: Render free tier dùng filesystem tạm thời nên dữ liệu có thể mất khi redeploy (Giai đoạn 17) |
| Frontend | Next.js (App Router) + TypeScript | Sát JD thị trường hơn Vite thuần; TypeScript bắt lỗi type khi response shape đổi. Đã cập nhật cho response v2 (Giai đoạn 23: nguồn `title`/`url`, `[n]` bấm được, lỗi 429/503), chưa deploy |
| Deploy | Render (backend) + Vercel (frontend) | Free tier. Bản đang chạy trên đó là bản BBC của Giai đoạn 17; bản mới chưa deploy (chưa thêm `TAVILY_API_KEY` trên Render) |
| Test | `test_backend.py`: 23 test offline | Giả Tavily, Groq và Redis (`fakeredis`), không tốn credit |
| Rate limiting | Upstash Redis (`redis-py`, một script Lua kiểm tra 3 giới hạn) | Bảo vệ quota Tavily (1 credit mỗi lần gọi). Redis ngoài nên bộ đếm sống qua lần khởi động lại của Render. Đang nằm trên nhánh riêng, chưa merge vào `main` |
| Vector DB (đã bỏ) | Qdrant | Mã nguồn mở, tự host dễ (Docker), hỗ trợ hybrid search (dense + sparse) native. Bỏ khỏi luồng chính ở Giai đoạn 18 vì web search không cần ingest và lưu corpus |
| Embedding dense (đã bỏ) | Ollama `nomic-embed-text`, sau đó FastEmbed `BAAI/bge-small-en-v1.5` (Giai đoạn 16) | Chạy local, miễn phí, không cần gọi API ngoài. Không còn dùng trong luồng chính |
| Embedding sparse (đã bỏ) | FastEmbed `Qdrant/bm25` | Bổ sung keyword matching cho hybrid search |
| LLM ban đầu (đã bỏ) | Ollama `llama3` | Chạy local; đánh đổi: chậm trên máy CPU-only. Thay bằng Groq ở Giai đoạn 16 |

**Ghi chú kỹ thuật hiện tại (Perplexity clone):**

- **Luồng `/ask`:** kiểm tra rate limit (có trên nhánh rate limit, chưa merge) → (lượt sau) rewriter viết lại câu hỏi bằng history → Tavily search (`top_k` kết quả, 1–10) → xoá dòng footnote Wikipedia khỏi snippet → bỏ kết quả có `score` dưới `MIN_SCORE` (0.20) hoặc thiếu URL/snippet (kể cả snippet chỉ còn footnote) → đánh số nguồn `[1]`…`[n]` → Groq trả lời có `[n]` → lưu SQLite (chỉ khi thành công) → trả `{query, search_query, answer, sources, conversation_id}`. Không còn nguồn nào thì trả câu báo không tìm thấy và không gọi LLM.
- **Endpoints:** `/health`, `/search` (chỉ web search, không LLM), `/ask`, `/conversations`, `/conversations/{id}`.
- **File đang dùng:** `main.py`, `search.py`, `rate_limit.py`, `config.py`, `db.py`, `test_backend.py`, `requirements.txt`, `requirements-dev.txt`, `probe_scores.py` và `probe_score.py` (đo thang điểm Tavily, gọi Tavily trực tiếp để `MIN_SCORE` không che kết quả). **Thuộc bản BBC, không nằm trong luồng chính:** `ingest.py`, `evaluate.py`, `label_helper.py`, `generate_synthetic_queries.py`.
- **Biến môi trường (`.env`):** `TAVILY_API_KEY`, `GROQ_API_KEY`, `LLM_MODEL` (mặc định `openai/gpt-oss-20b`), `MIN_SCORE` (mặc định 0.20), `CORS_ORIGINS`. Nhóm rate limit (trên nhánh): `REDIS_URL` (để trống thì tắt rate limit), `RATE_LIMIT_IP_PER_MINUTE` (5), `RATE_LIMIT_IP_PER_DAY` (10), `RATE_LIMIT_GLOBAL_PER_DAY` (30), `RATE_LIMIT_FAIL_OPEN` (false), `TRUSTED_PROXY_HOPS` (1), `RATE_LIMIT_DEBUG` (false). Nhóm legacy của bản BBC: `QDRANT_URL`, `QDRANT_API_KEY`, `EMBEDDING_MODEL`, `COLLECTION_NAME`.
- **Chạy và test:** backend chạy bằng uvicorn trên cổng 8000 (các lệnh curl trong nhật ký giả định cổng này); `python test_backend.py` chạy 23 test offline, không tốn credit (cần `pip install -r requirements-dev.txt` để có `fakeredis[lua]`).
- **Cấu trúc thư mục (Giai đoạn 24):** `main.py` ở thư mục gốc (bản bạn gửi) chỉ chuyển tiếp: thêm `backend/` vào `sys.path` rồi `from backend.main import app`, để lệnh `uvicorn main:app` vẫn chạy từ gốc. Tức là bản local đã chuyển code vào `backend/`; chưa xác nhận danh sách file trong `backend/` và chưa thấy thay đổi này trên GitHub `main`.
- **Nguồn nội dung:** v1 chỉ dùng snippet Tavily chọn sẵn. Tải nội dung đầy đủ của trang, tự cắt chunk và xếp hạng (dense + BM25 + RRF) đã được cân nhắc ở Giai đoạn 18 và hoãn thành thí nghiệm sau. Semantic cache bằng Qdrant cũng chưa làm.

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
- Xác nhận hoạt động đúng: Search "kapranos" → đúng 2 bài chứa từ này lên đầu kết quả, điểm số `0.5`/`0.333` khớp chính xác công thức RRF thực tế của Qdrant (`1/(vị_trí + 2)`).

**Quyết định kỹ thuật:**

- **Vấn đề:** implement hybrid search bằng cách gộp thủ công 2 hệ thống search riêng (Qdrant + 1 hệ keyword search khác), hay dùng sparse vector native trong Qdrant
- **Đã chọn:** sparse vector native trong Qdrant
- **Vì sao:** chỉ 1 database, 1 lần query, không cần thêm hệ thống thứ 2 để vận hành/bảo trì

**Bug gặp phải:**

- **Bug 10 — Schema cũ không tương thích:** chạy `ingest.py` mới (schema 2 vector có tên) lên collection cũ (1 vector không tên) → `QdrantVectorStoreError` khi validate lúc khởi tạo `QdrantVectorStore` → sửa bằng cách xoá collection cũ (`DELETE /collections/bbc_news`) rồi ingest lại từ đầu

**Việc còn tồn đọng từ giai đoạn này:** Không có.

---

## Giai đoạn 9 — Tối ưu tốc độ LLM

**Mục tiêu giai đoạn:** Giải quyết vấn đề tồn đọng từ Giai đoạn 5 — `llama3` (8B) sinh câu trả lời quá chậm trên máy CPU-only.

**Đã làm:**

- Đổi `LLM_MODEL` trong `.env` từ `llama3` sang `llama3.2:3b`
- Thêm `num_predict=256` vào `ChatOllama` trong `main.py` để giới hạn cứng số token sinh ra

**Quyết định kỹ thuật:**

- **Vấn đề:** chọn hướng tối ưu nào trong 4 hướng khả thi (đổi model nhỏ hơn, giới hạn num_predict, giảm top_k, thêm streaming)
- **Đã chọn:** đổi model nhỏ hơn + giới hạn num_predict, bỏ qua streaming
- **Vì sao:** đổi model tác động tốc độ lớn nhất với công sức thấp nhất (chỉ đổi `.env`); streaming không giảm tổng thời gian thật, chỉ cải thiện cảm giác chờ — không giải quyết gốc vấn đề

**Bug gặp phải:**

- **Bug 11 — Model nhỏ trả lời "context không chứa đáp án" dù chunk liên quan vẫn được retrieve đúng:** kiểm chứng qua phần `sources` vẫn thấy đúng chunk về visa/file-sharing → nguyên nhân không phải do retrieval sai, mà do `llama3.2:3b` (khả năng suy luận yếu hơn model 8B) hiểu nhầm prompt cũ (_"if the context doesn't contain the answer, say so"_) — không tự ghép được thông tin rải rác ở nhiều chunk khác nhau → sửa bằng cách viết lại prompt, ép rõ model được phép kết hợp thông tin từ nhiều đoạn, chỉ từ chối khi hoàn toàn không có đoạn nào liên quan

**Việc còn tồn đọng từ giai đoạn này:** Không có — đã cân bằng được tốc độ và chất lượng chấp nhận được.

---

## Giai đoạn 10 — Conversation Memory + Query Rewriting

**Mục tiêu giai đoạn:** Cho phép hỏi nhiều lượt liên tiếp (follow-up question), LLM hiểu được ngữ cảnh câu hỏi trước, thay vì mỗi request hoàn toàn độc lập.

**Đã làm:**

- Thêm `history: list[ChatTurn]` vào `AskRequest` — client (frontend) tự giữ và gửi kèm toàn bộ lịch sử mỗi lần gọi `/ask` (thiết kế stateless, giống cách OpenAI/Anthropic API hoạt động)
- Dùng `HumanMessage`/`AIMessage` (LangChain) thay vì ghép chuỗi thô, để LLM phân biệt rõ vai trò từng lượt
- Cập nhật frontend: `turns: ChatTurn[]` thay cho `answer` đơn lẻ, render thành khung chat, gửi `history` (không gồm câu hỏi hiện tại) mỗi lần hỏi
- Đổi giao diện sang dark theme

**Quyết định kỹ thuật:**

- **Vấn đề:** server có nên tự lưu lịch sử hội thoại (session/database) hay để client tự giữ và gửi lại mỗi lần
- **Đã chọn:** client tự giữ, gửi lại toàn bộ `history` mỗi request
- **Vì sao:** giữ server đơn giản (stateless), không cần thêm cơ chế quản lý session — đúng pattern các API chat thực tế (OpenAI, Anthropic) đang dùng
- **Vấn đề khác:** câu hỏi tiếp theo dùng đại từ ("that article") không có từ khoá thật để search (Bug 12)
- **Đã chọn:** thêm bước viết lại câu hỏi bằng LLM (`rewrite_query_with_history()`) trước retrieval
- **Vì sao:** viết lại thành câu độc lập giúp retrieval tìm đúng chunk. Đánh đổi: mỗi câu hỏi từ lượt thứ 2 trở đi tốn 2 lần gọi LLM thay vì 1 (viết lại câu hỏi + sinh câu trả lời) — chấp nhận được vì đổi lại retrieval chính xác hơn nhiều.

**Bug gặp phải:**

- **Bug 12 — Follow-up dùng đại từ ("that article") khiến retrieval trả về chunk không liên quan, LLM trả lời "none relevant":** nguyên nhân là `retrieve_chunks()` chỉ dùng đúng văn bản câu hỏi hiện tại để search — câu hỏi "which category does that article belong to?" không chứa từ khoá thật (musicians, protest, visa...) nên hybrid search không tìm ra chunk đúng, dù bản thân LLM đã hiểu đúng "that" ám chỉ gì. Sửa bằng cách thêm bước **Query Rewriting** (`rewrite_query_with_history()`) — gọi LLM viết lại câu hỏi phụ thuộc ngữ cảnh thành câu độc lập, đầy đủ nghĩa, TRƯỚC khi đưa vào retrieval. Có điều kiện bỏ qua bước này nếu `history` rỗng (lượt hỏi đầu tiên), tránh tốn thêm 1 lệnh gọi LLM không cần thiết.
- **Bug 13 (false alarm, không phải bug thật):** lần test đầu tưởng fix không hoạt động ("not explicitly stated") — hoá ra do server chưa được restart sau khi sửa code, vẫn chạy bản cũ. Rút kinh nghiệm: luôn xác nhận server đã reload đúng code mới trước khi kết luận 1 fix thất bại.
- **Bug 14 (false alarm khác):** 1 lần test cho kết quả category sai (`"entertainment"` thay vì `"tech"`) — do vô tình gõ nhầm input là câu đã bị hệ thống viết lại từ trước (thay vì câu hỏi tự nhiên gốc), khiến bước rewrite chạy lần 2 trên 1 câu vốn đã đủ nghĩa, làm trôi chủ đề. Không phải lỗi code — bài học: cẩn thận khi test thủ công để không tự tạo input sai lệch với kịch bản thật.

**Việc còn tồn đọng từ giai đoạn này:** Không có — đã xác nhận hoạt động đúng qua nhiều lần test nhất quán (`curl` và trình duyệt cho cùng kết quả).

---

## Giai đoạn 11 — Evaluation (đo chất lượng retrieval bằng số liệu)

**Mục tiêu giai đoạn:** Đo khách quan xem hybrid search có thực sự tốt hơn dense-only hay không, bằng con số, thay vì chỉ dựa vào cảm giác qua vài lần test tay.

**Đã làm:**

- Viết `evaluate.py` — xây "golden set" nhỏ (câu hỏi kèm đáp án `article_id` đúng, dựa trên các case đã tự tay xác minh trước đó: "kapranos", "why are musicians protesting")
- Đo 3 chỉ số chuẩn Information Retrieval: **Recall@k** (có tìm ra đáp án đúng không), **MRR** (đáp án đúng đứng thứ mấy), **Precision@k** (trong k kết quả, bao nhiêu % thực sự đúng)
- So sánh trực tiếp dense-only vs hybrid trên cùng dữ liệu
- Kết quả đo: Recall@5 và MRR: cả 2 phương pháp bằng nhau tuyệt đối (1.00 / 1.000) — không phân biệt được
- Kết quả đo: Precision@5: "kapranos" → hybrid thắng rõ (1.0 vs 0.8, đúng dự đoán lý thuyết về tên riêng); "why are musicians protesting" → dense-only thắng (0.8 vs 0.6, ngược dự đoán) → trung bình cộng ra **bằng nhau (0.80 cả 2 bên)**

**Quyết định kỹ thuật:**

- **Vấn đề:** golden set chỉ có 2 câu hỏi có đáp án — không đủ để kết luận thống kê đáng tin
- **Đã chọn:** không ép kết luận "hybrid tốt hơn" khi số liệu không ủng hộ rõ ràng — ghi nhận trung thực rằng mỗi phương pháp có điểm mạnh riêng tuỳ loại câu hỏi
- **Vì sao:** kết luận trung thực dựa trên số liệu quan trọng hơn kết luận "đẹp" nhưng không có bằng chứng — đúng tinh thần evaluation thật sự

**Việc còn tồn đọng từ giai đoạn này:**

- Golden set cần mở rộng (10-20 câu, chia nhóm theo loại câu hỏi: tên riêng, paraphrase ngữ nghĩa, câu hỏi thường) để có kết luận đáng tin cậy hơn về việc hybrid tốt hơn ở loại câu hỏi nào cụ thể

---

## Giai đoạn 12 — Redesign Frontend (giao diện "Wire Search")

**Mục tiêu giai đoạn:** Đổi giao diện từ dark theme đơn giản sang bố cục kiểu ChatGPT/Claude (sidebar + khung chat), nhưng mang thẩm mỹ riêng gắn với chủ đề (tra cứu kho tin tức BBC), tránh giao diện chatbot chung chung.

**Đã làm:**

- Thiết kế theo hướng "wire service/newsroom": nền than chì `#141414`, accent hổ phách `#D9A441`, font `Lora` (serif, masthead) + `IBM Plex Mono` (chỉ dùng cho metadata: category, score, article id)
- Bố cục: sidebar trái (nút "New search" + "Verified queries" lấy đúng 3 câu từ `evaluate.py`, bấm chạy thẳng), khung chat chính dùng đường kẻ mảnh phân cách thay vì bubble bo tròn
- Responsive: dưới 720px, sidebar chuyển thành thanh ngang trên cùng
- Thêm biến `--font-scale` trong CSS, mọi `font-size` nhân qua `calc()` — chỉ cần đổi 1 số để scale toàn bộ cỡ chữ site

**Quyết định kỹ thuật:**

- **Vấn đề:** làm giao diện chatbot chung chung hay gắn thẩm mỹ riêng theo chủ đề dự án
- **Đã chọn:** thẩm mỹ "wire service" riêng, không dùng nền be/serif/cam đất mặc định thường thấy ở giao diện AI
- **Vì sao:** giao diện gắn đúng bối cảnh "tra cứu kho lưu trữ tin tức" giúp project nổi bật hơn khi demo, tránh trông giống bản sao ChatGPT

**Bug gặp phải:**

- **Bug 15 — File mới không lên hiệu lực dù đã "làm hết các bước":** giao diện vẫn hiện bản cũ dù đã giải nén/ghi đè — nguyên nhân nghi ngờ nhiều khả năng nhất là giải nén tạo thư mục lồng nhau (`frontend/frontend/app/...`) do file zip vốn đã chứa sẵn đường dẫn `frontend/app/...`. Xác nhận và xử lý bằng cách kiểm tra `find` + `grep` để định vị đúng file, sau đó tự khắc phục thành công.

**Việc còn tồn đọng từ giai đoạn này:** Không có.

---

## Giai đoạn 13 — Mở rộng Golden Set cho Evaluation

**Mục tiêu giai đoạn:** Golden set cũ (Giai đoạn 11) chỉ có 2 câu hỏi — không đủ để kết luận đáng tin cậy về hybrid vs dense-only. Mở rộng lên nhiều câu hơn, chia rõ theo nhóm loại câu hỏi.

**Đã làm:**

- Viết `label_helper.py` — công cụ chạy hybrid search trên các câu hỏi candidate, in ra đầy đủ text + article_id để TỰ TAY đọc và xác nhận đáp án đúng (không tự bịa ground truth, vì không có quyền truy cập dữ liệu thật để tự xác minh)
- Tự đọc và gán nhãn thủ công cho 10 câu hỏi, chia 3 nhóm: `proper_noun` (kapranos, u2, napster), `paraphrase` (musicians protesting, movie release delayed, illegal downloading lawsuits, government funding), `general` (technology gadgets, australian open, eu stability pact)
- Nâng cấp `evaluate.py`: tính Recall@k/MRR/Precision@k **riêng theo từng nhóm** thay vì gộp chung 1 con số, dùng `defaultdict` gom kết quả theo `category`

**Quyết định kỹ thuật:**

- **Vấn đề:** 2 câu hỏi ban đầu (`"sports championship results"`, `"economic policy changes"`) quá mơ hồ, khó xác định đáp án đúng rạch ròi
- **Đã chọn:** thay bằng câu hỏi cụ thể hơn (`"who won the australian open tennis title"`, `"eu stability pact deficit rules changed"`), dựa trên nội dung thật đã thấy qua `label_helper.py`
- **Vì sao:** câu hỏi mơ hồ tạo nhiễu trong golden set — không thể phân biệt "hệ thống tìm sai" với "câu hỏi vốn không có đáp án rõ ràng"

**Việc còn tồn đọng từ giai đoạn này:**

- Case `"radiohead"` bị loại khỏi golden set vì không tìm thấy đáp án đủ tin cậy trong top 10 kết quả — cần tự `grep` file CSV gốc để xác minh có bài nào thực sự nhắc "radiohead" hay không, trước khi thêm lại

---

## Giai đoạn 14 — Reranking (Cross-Encoder) + Citation theo từng claim

**Mục tiêu giai đoạn:** Vá 2 khoảng trống so với kiến trúc kiểu Perplexity: (1) chưa có bước rerank riêng sau retrieval, (2) câu trả lời không gắn nguồn cụ thể cho từng claim, chỉ trả `sources` rời rạc bên cạnh.

**Đã làm:**

- Thêm **Cross-Encoder rerank** (`fastembed.rerank.cross_encoder.TextCrossEncoder`, model `Xenova/ms-marco-MiniLM-L-6-v2`): `retrieve_chunks()` giờ lấy 15 candidate từ hybrid search (`RERANK_CANDIDATE_POOL`), cho cross-encoder chấm điểm lại từng cặp (query, chunk) cùng lúc, sắp xếp lại rồi cắt còn đúng `top_k`
- Thêm **citation theo từng claim**: đánh số nguồn `[1]`, `[2]`... trong `context_block` theo đúng thứ tự `chunks`/`sources`, ép prompt yêu cầu LLM chèn số nguồn ngay sau mỗi claim (ví dụ `"...lawsuits [2]."`)

**Quyết định kỹ thuật:**

- **Vấn đề:** dùng embedding thường hay cross-encoder cho bước rerank
- **Đã chọn:** cross-encoder
- **Vì sao:** embedding encode câu hỏi và chunk RIÊNG BIỆT rồi so vector (nhanh, chạy được trên toàn bộ 7583 chunk qua Qdrant); cross-encoder đọc CẢ 2 CÙNG LÚC, chính xác hơn nhưng chậm hơn nhiều — chỉ khả thi khi chạy trên tập nhỏ (15 candidate) sau khi đã lọc bằng hybrid search, không thể chạy trên toàn bộ collection

**Bug gặp phải:**

- **Bug 16 — Cross-encoder xếp hạng SAI trên dataset đã tiền xử lý:** test thực tế cho thấy 1 bài KHÔNG liên quan (Jerry Springer opera) được cross-encoder chấm điểm CAO HƠN các bài THỰC SỰ liên quan (visa, file-sharing) cho cùng 1 câu hỏi; `debug_rerank.py` (script debug riêng, in toàn bộ điểm 15 candidate) xác nhận đây không phải do chọn sai ngưỡng lọc → text lưu trong Qdrant đã bị tiền xử lý mất dấu câu, viết thường, mất cấu trúc câu tự nhiên (từ bước ingest ban đầu, xem Giai đoạn 3), trong khi model `ms-marco-MiniLM` được train trên text tự nhiên có dấu câu nên khả năng đánh giá độ liên quan bị nhiễu nặng → KHÔNG xoá code rerank, thêm cờ `ENABLE_RERANK` (mặc định `false`) trong `config.py`; khi tắt, `retrieve_chunks()` dùng lại hybrid RRF thuần (đã kiểm chứng ổn ở Giai đoạn 11), cần thì set `ENABLE_RERANK=true` trong `.env` để bật lại mà không sửa code. Citation (`[1]`, `[2]`) không bị ảnh hưởng, vì chỉ đánh số theo thứ tự chunks trả về.

**Việc còn tồn đọng từ giai đoạn này:**

- ~~Test citation `[1]`/`[2]` khớp đúng sources~~ — đã xác nhận qua test thật (article #0 khớp đúng nội dung [1])
- Chưa đo bằng golden set xem rerank có thực sự cải thiện Precision/MRR so với chỉ dùng RRF hay không — evaluate.py hiện tại KHÔNG bao gồm bước rerank (vì gọi thẳng `vectorstore.similarity_search()`, không qua `retrieve_chunks()`), nên 2 việc này đang tách biệt, cần làm riêng nếu muốn so sánh
- Chưa cập nhật frontend để hiển thị `[1]`, `[2]` dạng link bấm được tới đúng nguồn
- ~~Sửa `ingest.py` để lưu thêm text gốc (chưa tiền xử lý) làm field riêng, rồi thử lại rerank trên text đó~~ — không còn áp dụng, rerank đã bị xoá hẳn ở Giai đoạn 15

---

## Giai đoạn 15 — Xoá bỏ Reranking + Citation

**Mục tiêu giai đoạn:** Sau khi phát hiện Bug 16 (cross-encoder chấm điểm sai trên dataset đã tiền xử lý) và cân nhắc thêm, quyết định tính năng này không đáng giữ ở dạng hiện tại.

**Đã làm:**

- Xoá hoàn toàn code rerank khỏi `main.py`: import `TextCrossEncoder`, các hằng số `RERANK_MODEL_NAME`/`RERANK_CANDIDATE_POOL`/`RERANK_MIN_SCORE`, biến `reranker`, nhánh rẽ trong `retrieve_chunks()` — quay về đúng 1 đường hybrid RRF thuần
- Xoá citation theo từng claim: `context_block` không còn đánh số `[1]`/`[2]`, prompt bỏ đoạn yêu cầu trích dẫn — quay về bản prompt gốc (Giai đoạn 10)
- Bỏ `ENABLE_RERANK` khỏi `config.py` — không còn tính năng để bật/tắt
- Xoá `debug_rerank.py` (không còn cần thiết)
- Cập nhật ngay sau đó: thêm lại `context_block` đánh số `[n]` và đoạn prompt yêu cầu trích dẫn (không thêm lại cross-encoder)

**Quyết định kỹ thuật:**

- **Vấn đề:** giữ code rerank ở trạng thái tắt (`ENABLE_RERANK=false`, như Giai đoạn 14 đã làm) hay xoá hẳn
- **Đã chọn:** xoá hẳn
- **Vì sao:** đánh giá lại thấy tính năng không đủ giá trị để duy trì độ phức tạp code (2 nhánh logic, model phụ cần tải, nguy cơ tái phát Bug 16 nếu ai đó bật nhầm `ENABLE_RERANK=true` mà chưa sửa `ingest.py`) — đơn giản hoá codebase quan trọng hơn giữ 1 tính năng chưa hoạt động đúng
- **Vấn đề khác:** sau khi xoá cả rerank lẫn citation, có nên thêm lại citation không
- **Đã chọn:** thêm lại citation `[1]`/`[2]`, không thêm lại rerank. Kết luận cuối: giữ citation, bỏ hẳn rerank
- **Vì sao:** citation được xác định là hữu ích: giúp tự kiểm tra thủ công claim nào lấy từ nguồn nào, đặc biệt để tự phát hiện khi hybrid RRF lỡ lẫn 1 nguồn kém liên quan (vẫn có thể xảy ra vì hybrid RRF không hoàn hảo). Bug 16 nằm ở việc cross-encoder tự chấm điểm sai, còn citation chỉ đơn thuần đánh số thứ tự nên không dính lỗi đó

**Việc còn tồn đọng từ giai đoạn này:** Không có — retrieval quay về trạng thái ổn định đã kiểm chứng (Giai đoạn 11/13), memory/query rewriting/conversation persistence không bị ảnh hưởng.

---

## Giai đoạn 16 — Chuẩn bị Deploy Free Tier: Bỏ Ollama, chuyển sang FastEmbed + Groq

**Mục tiêu giai đoạn:** Ollama cần CPU/RAM đủ lớn để chạy — free tier của các nền tảng cloud (Render/Fly.io) không đáp ứng nổi. Thay thế cả embedding lẫn LLM để hệ thống deploy được miễn phí.

**Đã làm:**

- Đổi dense embedding: `nomic-embed-text` (qua Ollama) → `BAAI/bge-small-en-v1.5` (qua `FastEmbedEmbeddings`, chạy trực tiếp trong process Python bằng ONNX runtime, không cần server riêng)
- Đổi LLM: `llama3.2:3b` (qua Ollama) → `openai/gpt-oss-20b` (qua Groq API, cloud, miễn phí)
- Thêm `QDRANT_API_KEY` vào `config.py` (tuỳ chọn, cho Qdrant Cloud sau này)
- Đổi `EMBEDDING_DIM` từ 768 → 384 (đúng số chiều của `bge-small-en-v1.5`)
- Xoá collection cũ, `ingest.py` lại toàn bộ dataset với embedding mới

**Quyết định kỹ thuật:**

- **Vấn đề:** cách nào để thay Ollama mà vẫn miễn phí — Groq API (cloud) hay Cloudflare Tunnel (lộ Ollama từ máy cá nhân)
- **Đã chọn:** Groq API
- **Vì sao:** không phụ thuộc máy cá nhân phải bật liên tục, chuyên nghiệp hơn khi demo cho nhà tuyển dụng, đúng kỹ năng "tích hợp LLM provider ngoài" trong JD

**Bug gặp phải:**

- **Bug 17 — Lỗi Ollama "model not found" dù đã đổi sang FastEmbed:** do `ingest.py`/`main.py` chưa được thay file mới, code cũ (`OllamaEmbeddings`) vẫn nhận tên model mới từ `config.py`, gửi nhầm cho Ollama → sửa bằng cách thay đúng file `ingest.py`/`main.py` mới
- **Bug 18 — Groq model `llama-3.1-8b-instant` báo 404:** model đã chuyển sang diện Enterprise-only ("Contact Sales"), không còn mở cho tài khoản free. `llama-3.3-70b-versatile` cũng đã bị Groq ngừng hỗ trợ hẳn. Sửa bằng cách tự gọi `GET /v1/models` với API key thật để lấy danh sách model tài khoản mình thực sự được phép dùng, chọn `openai/gpt-oss-20b` (còn active, giá thấp)
- **Bug 19 — `answer` trả về rỗng dù không lỗi:** `openai/gpt-oss-20b` là REASONING model — tốn token cho phần "suy nghĩ" nội bộ (`reasoning_content` trong `additional_kwargs`) TRƯỚC KHI viết câu trả lời cuối, tất cả tính chung vào `max_tokens`. Giới hạn cũ `max_tokens=256` (kế thừa từ thời Ollama) hết ngân sách giữa lúc model còn đang "suy nghĩ", không kịp sinh `content` → sửa bằng cách tăng `max_tokens` lên `2048`. Xác nhận qua `debug_groq_response.py` (script debug riêng in `additional_kwargs`/`response_metadata`) trước khi kết luận nguyên nhân, không đoán mò.

**Việc còn tồn đọng từ giai đoạn này:**

- ~~Deploy thật lên Qdrant Cloud/Render/Vercel~~ — đã hoàn tất ở Giai đoạn 17

---

## Giai đoạn 17 — Deploy Production (Render + Vercel + Qdrant Cloud)

**Mục tiêu giai đoạn:** Đưa toàn bộ hệ thống lên internet, ai cũng truy cập được qua URL công khai, hoàn toàn miễn phí.

**Đã làm:**

- Backend (FastAPI) → **Render** free tier, region Singapore (gần Việt Nam nhất trong các lựa chọn có sẵn)
- Frontend (Next.js) → **Vercel** free tier
- Vector DB → **Qdrant Cloud** free tier (1GB), region Sydney (Google Cloud Platform — gần Việt Nam nhất trong danh sách provider/region có sẵn)
- Viết `requirements.txt` riêng cho backend, scope đúng những gì `main.py` cần chạy production (không gồm `pandas`/`langchain-text-splitters` — 2 package chỉ `ingest.py` cần, chạy thủ công local, không chạy trên Render)
- Start command trên Render: `uvicorn main:app --host 0.0.0.0 --port $PORT` — bắt buộc dùng `$PORT` do Render tự gán cổng, không cố định như `8000` lúc chạy local

**Quyết định kỹ thuật:**

- **Vấn đề:** ingest lên Qdrant Cloud báo `ResponseHandlingException: The write operation timed out` — batch 50 chunk/lần qua internet thật (thay vì localhost) vượt quá timeout mặc định
- **Đã chọn:** tăng `timeout=60` khi tạo `QdrantClient` (ingest) / `timeout=30` (main.py), đồng thời giảm `batch_size` từ 50 xuống 20
- **Vì sao:** batch nhỏ hơn giảm khả năng 1 request đơn lẻ bị timeout; timeout dài hơn cho đủ thời gian với độ trễ mạng thật

**Bug gặp phải:**

- **Bug 20 — `net::ERR_CONNECTION_REFUSED` gọi `localhost:8000` dù đã deploy:** sau khi deploy cả 2, frontend trên Vercel vẫn cố gọi `localhost:8000` (cổng trên chính máy người xem web, không phải server Render) → lỗi kết nối ngay lập tức.
    - **Nguyên nhân gốc:** biến môi trường dùng để build `API_URL` không có tiền tố `NEXT_PUBLIC_` (ví dụ `process.env.API_URL` thay vì `process.env.NEXT_PUBLIC_API_URL`). Next.js chỉ nhúng biến có tiền tố này vào bundle JS chạy trên trình duyệt — biến không có tiền tố chỉ tồn tại phía server lúc build, trên trình duyệt luôn là `undefined`, khiến code tự rơi về giá trị mặc định `"http://localhost:8000"` bất kể đã set gì trên Vercel dashboard.
    - **Cách sửa:** đổi tên biến thành `NEXT_PUBLIC_API_URL` trong code, set đúng biến này trên Vercel (Project → Settings → Environment Variables) trỏ tới URL Render thật, sau đó **bắt buộc redeploy lại** — biến `NEXT_PUBLIC_*` được nhúng cứng vào bundle lúc build, sửa xong không tự áp dụng nếu không build lại.
    - **Việc liên quan:** xác nhận `allow_origins` trong `CORSMiddleware` (FastAPI) đã có đúng domain Vercel thật, không chỉ `localhost:3000`.

**Việc còn tồn đọng từ giai đoạn này:**

- SQLite (`chat_history.db`) trên Render free tier dùng filesystem tạm thời — dữ liệu conversation có thể mất khi service redeploy/restart. Chấp nhận được cho portfolio demo, nhưng cần lưu ý nếu sau này muốn persistence thật sự bền vững (giải pháp: chuyển sang Postgres free tier của Render hoặc dịch vụ tương tự)
- ~~Render free tier "cold start"~~ — đã xử lý bằng uptime bot (ping định kỳ giữ service không ngủ)

---

## Giai đoạn 18 — Chuyển hướng: Perplexity Clone (Web Search + RAG có trích dẫn)

**Mục tiêu giai đoạn:** Đổi từ search trên kho BBC tĩnh sang RAG trên web trực tiếp, bám đúng kiến trúc Perplexity: câu hỏi → web search → tải nội dung trang → xếp hạng chunk → LLM tổng hợp, trích dẫn `[n]` trỏ về URL. (Đây là kế hoạch ban đầu của giai đoạn; các quyết định cuối giai đoạn đã đơn giản hoá thành web search → snippet → LLM, xem phần Quyết định kỹ thuật.)

**Đã làm:**

- Tạo tài khoản Tavily và API key (lưu trong `.env` dưới tên `TAVILY_API_KEY`); **chưa** thêm biến môi trường trên Render. Groq vẫn đang dùng (LLM `openai/gpt-oss-20b`).
- Chạy thử Tavily 2 lần bằng `try_tavily.py` với câu hỏi "why are musicians protesting". Kết quả:
  - Response gồm: `query`, `follow_up_questions`, `answer`, `images`, `results`, `response_time`, `request_id`. Không có trường `usage`, nên credit phải xem trên dashboard.
  - Mỗi result có `url`, `title`, `content` (snippet), `score`, `raw_content`, `id`. Snippet dài 844–1,501 ký tự (trung bình khoảng 1,237), là các đoạn trích theo câu hỏi nối với nhau bằng `[...]`.
  - Với `include_raw_content=True`: 3/5 kết quả có nội dung đầy đủ (16.7k, 139k, 16.9k ký tự), 2/5 (Reddit, YouTube) trả `null`.
  - Hai lần gọi cùng một câu hỏi cho hai bộ kết quả khác nhau (chỉ 2/5 trùng), chưa rõ nguyên nhân.
  - `score` từ 0.10 đến 0.76; nguồn yếu có score thấp, nhưng mới 10 mẫu nên chưa chọn được ngưỡng lọc.
  - Thời gian phản hồi phía Tavily khoảng 0.94–1.25 giây.

**Quyết định kỹ thuật:**

- **Vấn đề:** nguồn dữ liệu của search engine
- **Đã chọn:** web search API trực tiếp, thay cho corpus BBC
- **Vì sao:** bản chất Perplexity là truy xuất web thời gian thực, kho tĩnh không tái hiện được điều đó
- **Vấn đề khác:** chọn web search API nào
- **Đã chọn:** Tavily
- **Vì sao:** free 1,000 credit/tháng, không cần thẻ, trả nội dung đã làm sạch. Đã cân nhắc và loại: Brave (bỏ free tier từ 2/2026, chỉ còn $5 credit/tháng), Exa ($20 credit dùng một lần), Parallel (chưa kiểm chứng hạn mức free). Lưu ý rủi ro: Tavily bị Nebius mua lại 2/2026, giá/chính sách có thể đổi; số credit mỗi lần gọi khác nhau giữa các nguồn đã đọc, cần xác minh trên trang giá chính thức trước khi dựa vào. (Đã xác minh ở Giai đoạn 20: 1 credit mỗi lần gọi basic.)
- **Vấn đề khác:** Qdrant và ingest BBC còn cần không
- **Đã chọn:** bỏ khỏi luồng chính
- **Vì sao:** chunk lấy mới cho mỗi câu hỏi nên xếp hạng trong bộ nhớ (dense FastEmbed + BM25 + RRF), không cần database lưu lâu dài (kế hoạch lúc đó; xem quyết định về snippet bên dưới). Qdrant chỉ có thể quay lại ở dạng semantic cache, chưa làm
- **Vấn đề khác:** phần nào của bản BBC giữ lại
- **Đã chọn:** giữ memory + query rewriting, SQLite conversations, Groq LLM, Next.js frontend, citation `[n]` (đổi từ `article_id` sang URL)
- **Vì sao:** không ghi lại lý do lúc chốt
- **Vấn đề khác:** kế hoạch fine-tune embedding / QLoRA trên Colab có làm tiếp không
- **Đã chọn:** tạm dừng, làm lại sau khi Perplexity clone xong
- **Vì sao:** ưu tiên hoàn thành Perplexity clone trước
- **Vấn đề khác:** bản đầu của pipeline lấy nội dung từ snippet, từ `raw_content`, hay kết hợp
- **Đã chọn:** chỉ dùng snippet (`search_depth="basic"`, không `include_raw_content`)
- **Vì sao:** đơn giản và nhanh. Hoãn `raw_content` vì có trang dài tới 139k ký tự (cần chunk + xếp hạng), chi phí embed trên Render free tier chưa đo, và khoảng 2/5 trang không có `raw_content`.
- **Vấn đề khác:** xử lý snippet trước khi đưa vào LLM (đưa nguyên / lọc theo `score` / tách theo `[...]` rồi xếp hạng hybrid)
- **Đã chọn:** đưa nguyên snippet của mọi kết quả vào prompt
- **Vì sao:** không có rủi ro vứt nhầm nội dung tốt. Thử BM25 đơn giản (chỉ từ khoá, không phải BM25 của FastEmbed) trên 15 đoạn tách từ file basic cho thấy: đoạn rác của NPR đứng hạng 1 (vì chứa tiêu đề bài), còn hai đoạn Wikipedia (nguồn Tavily chấm cao nhất, 0.736) bị chấm 0 vì dùng "artists"/"boycott" thay cho "musicians"/"protest". Lọc theo `score` cũng chưa có cơ sở chọn ngưỡng (ngưỡng 0.6 sẽ loại bài Chicago Tribune 0.558, bài nói thẳng về chủ đề). Phần dense chưa chạy được trong sandbox nên chưa biết hybrid có cứu được không.

**Việc còn tồn đọng từ giai đoạn này:**

- Thí nghiệm sau: tách snippet theo `[...]` rồi xếp hạng dense + BM25 + RRF, làm thành thí nghiệm có đo lường khi dense chạy được và đã có evaluation cho bản mới
- Ghi số credit Tavily đã dùng cho 2 lần thử (chờ đối chiếu trên dashboard)
- ~~Thiết kế và code pipeline mới~~ — đã code ở Giai đoạn 19, đã test thật ở Giai đoạn 20
- Thử lại cross-encoder rerank trên text web (text tự nhiên có dấu câu, khác lỗi ở Bug 16): mới là dự đoán, cần test mới biết
- Thiết kế lại evaluation (golden set BBC không còn áp dụng)
- Cập nhật README/CV sau khi xong (chưa polish CV chính thức: README, screenshots)
- Gỡ các thành phần trở nên thừa (chưa gỡ): cluster Qdrant Cloud, cron-job.org keep-alive, đoạn ping Qdrant trong `/health` (đã bỏ ở `main.py` mới), `ingest.py`, golden set BBC trong `evaluate.py`
- Fine-tune embedding / QLoRA: tạm dừng, làm sau khi Perplexity clone xong

---

## Giai đoạn 19 — Backend v1 cho Perplexity clone (web search + Groq + citation)

**Mục tiêu giai đoạn:** Có backend chạy end-to-end theo luồng đã chốt ở Giai đoạn 18: câu hỏi → (viết lại nếu là follow-up) → Tavily search → đánh số nguồn → Groq trả lời có `[n]` → lưu SQLite.

**Đã làm:**

- `search.py` (mới): bọc Tavily trong `web_search()`, trả về `title`, `url`, `snippet`, `score`. Bỏ kết quả thiếu URL hoặc snippet rỗng. Lỗi gom thành `WebSearchError`. Đổi nhà cung cấp search về sau chỉ cần sửa file này.
- `main.py` (viết lại): bỏ Qdrant, embeddings, vectorstore; `/ask` chạy luồng mới; giữ query rewriting, memory (`history`) và SQLite conversations.
- `config.py`: thêm `TAVILY_API_KEY`, `CORS_ORIGINS`. Cấu hình Qdrant/embedding giữ ở mục "legacy" để các script đang tạm dừng (`ingest.py`, `evaluate.py`, `generate_synthetic_queries.py`) vẫn import được.
- `db.py`: chuyển comment/docstring sang tiếng Anh (đúng quy ước code chỉ có tiếng Anh), logic không đổi.
- `requirements.txt`: bỏ `fastembed`, `qdrant-client`, `langchain-qdrant`, `langchain-community`; thêm `tavily-python`. Backend deploy nhẹ hơn, không còn tải model embedding lúc khởi động.
- `test_backend.py`: 12 test offline (giả Tavily và Groq, không tốn credit), tất cả pass. Phần test adapter dùng đúng cấu trúc response Tavily thật đã thấy ở thử nghiệm.

**Quyết định kỹ thuật:**

- **Vấn đề:** phạm vi và thứ tự làm backend v1
- **Đã chọn:** (bạn chọn) không streaming ở v1; xoá `chat_history.db` cũ; làm backend trước, test bằng curl, rồi mới frontend
- **Vì sao:** không ghi lại lý do lúc chốt
- **Vấn đề khác:** `/search` làm gì sau khi bỏ Qdrant
- **Đã chọn:** (mình tự chọn, chưa hỏi bạn, có thể đổi) giữ lại nhưng đổi thành "chỉ web search, không LLM"
- **Vì sao:** để kiểm tra Tavily trả gì
- **Vấn đề khác:** response `/ask` có nên cho thấy câu truy vấn đã viết lại không
- **Đã chọn:** (mình tự chọn, có thể đổi) thêm field `search_query`
- **Vì sao:** dễ kiểm tra follow-up
- **Vấn đề khác:** `top_k` nghĩa là gì sau khi chuyển sang web search
- **Đã chọn:** (mình tự chọn, có thể đổi) là số kết quả web, giới hạn 1–10
- **Vì sao:** để một request không đốt quota. Lưu ý: Giai đoạn 20 đã đo thấy `max_results` không ảnh hưởng giá, nên giới hạn này không bảo vệ được quota
- **Vấn đề khác:** khi nào lưu hội thoại vào SQLite
- **Đã chọn:** (mình tự chọn, có thể đổi) chỉ lưu sau khi cả search và LLM thành công
- **Vì sao:** request lỗi không để lại hội thoại rỗng ở sidebar
- **Vấn đề khác:** xử lý lỗi và trường hợp không có nguồn
- **Đã chọn:** (mình tự chọn, có thể đổi) search lỗi trả 502; câu trả lời rỗng (nghi hết token của reasoning model, giống Bug 19) cũng trả 502 thay vì lưu tin nhắn rỗng; không có kết quả web thì trả câu báo không tìm thấy và không gọi LLM
- **Vì sao:** không lưu tin nhắn rỗng, và không bắt LLM đoán khi không có nguồn
- **Vấn đề khác:** rác giao diện trong snippet (navigation, logo, chú thích ảnh)
- **Đã chọn:** (mình tự chọn, có thể đổi) prompt dặn LLM bỏ qua
- **Vì sao:** như đoạn NPR đã thấy ở thử nghiệm
- **Vấn đề khác:** CORS
- **Đã chọn:** (mình tự chọn, có thể đổi) đọc từ biến `CORS_ORIGINS`, mặc định gồm `localhost:3000` và domain Vercel đang deploy
- **Vì sao:** giữ cho môi trường local và frontend đã deploy tiếp tục chạy được

**Việc còn tồn đọng từ giai đoạn này:**

- ~~Chạy thử thật bằng curl với key thật (`/health`, `/search`, `/ask`, follow-up)~~ — đã làm ở Giai đoạn 20
- Chưa kiểm chứng ở thời điểm đó: chưa gọi Tavily/Groq thật từ sandbox (không có key, mạng bị chặn), test offline chỉ chứng minh logic pipeline, không chứng minh chất lượng câu trả lời hay hành vi thật của hai API. Đã chạy thật ở Giai đoạn 20 (bạn chạy trên máy của bạn)
- Xoá `chat_history.db` cũ trước khi chạy (chưa xác nhận đã xoá: id hội thoại vẫn tăng dần qua các lần chạy ở Giai đoạn 20)
- ~~Frontend: đổi hiển thị sources từ `article_id`/`category` sang `title`/`url`, làm `[n]` bấm được~~ — đã làm ở Giai đoạn 23
- Deploy: thêm `TAVILY_API_KEY` trên Render. Cron-job.org ping Qdrant có thể tắt (bản `main.py` mới không còn ping Qdrant trong `/health`)

---

## Giai đoạn 20 — Test thật Perplexity clone: sửa rewriter, bộ lọc score, prompt trả lời

**Mục tiêu giai đoạn:** Chạy backend v1 với Tavily và Groq thật, đo chất lượng bằng cách đối chiếu từng `[n]` với snippet của nguồn được cite, rồi sửa những lỗi nhìn thấy được.

**Đã làm:**

- Test curl thật: `/search` và `/ask` lượt đầu đạt, `[n]` khớp thứ tự `sources`. Lần chạy follow-up đầu tiên sai ở lệnh test, không phải lỗi code: `history` còn nguyên placeholder `<paste the answer here>` và `conversation_id` không phải id vừa nhận.
- Bộ test follow-up cố định: `history_answer.txt` (một câu trả lời mẫu) + `jq`, để phép thử không phụ thuộc kết quả web đổi theo từng ngày.
- `config.py`: thêm `MIN_SCORE` (đọc từ `.env`). `search.py`: bỏ kết quả có score dưới `MIN_SCORE`; kết quả không có score được giữ lại (không đoán được thì không bỏ). `/search` cũng bị lọc. Mặc định lúc đầu 0.05, sau đổi thành 0.20.
- `main.py`, prompt rewriter v2: bỏ lối thoát "câu đã standalone thì giữ nguyên", thay bằng "chỉ giữ nguyên khi history không nói gì về chủ đề đó, kể cả khi đó vẫn bỏ 'tell me more'"; thêm ví dụ thứ 2 (câu hỏi nêu chủ đề chung chung, chi tiết cụ thể nằm trong history). Rewriter trả chuỗi rỗng thì dùng lại câu gốc.
- `main.py`, prompt trả lời: 6 luật chính xác. Bốn luật đầu (cite đúng nguồn chứa claim; hai nguồn lệch số thì nêu cả hai; không gộp các sự kiện khác nhau; chỉ nêu nguyên nhân, trước-sau, phủ định khi nguồn nói rõ) và hai luật mới: đổi "last year/recently" sang năm tuyệt đối theo URL hoặc ngày trong snippet; không khái quát từ vài ví dụ lên "all/every/none". Luật giữ nguyên chữ hedging ("allegedly", "reportedly", "may") được gộp vào luật nguyên nhân.
- Không có nguồn nào vượt ngưỡng thì câu trả lời là "I could not find any relevant web sources for this question." và không gọi LLM.
- `test_backend.py`: 15 test offline pass (12 cũ + 3 mới: tất cả kết quả dưới ngưỡng thì không gọi LLM; kết quả lẫn lộn thì chỉ giữ nguồn trên ngưỡng và `[n]` đánh số lại từ 1; prompt rewriter chứa history và dùng lại câu gốc khi rewrite rỗng). Tắt riêng dòng lọc thì đúng 2 test lọc fail.
- Lượt chạy b3 ("why did Daniel Ek step down as Spotify CEO", lượt đầu nên không qua rewriter) đạt: 2 câu, khớp Reuters `[1]` và `[5]` (hai nguồn này đều là Reuters, bài và video cùng nội dung).
- Đo thang điểm Tavily bằng `probe_scores.py` (4 truy vấn, 37 kết quả):
  - Script gọi Tavily trực tiếp (không qua `search.web_search`) để `MIN_SCORE` không che kết quả; `probe_score.py` là bản rút gọn chỉ gọi truy vấn `first`.
  - Thang điểm không đổi: truy vấn `why are musicians protesting` cho cùng điểm với lần đo đầu ở 3 trang trùng (KXT 0.7581, Wikipedia No Music for Genocide 0.7357, NPR 0.6753). Tập kết quả thì có đổi (In These Times và Harvard Political Review không còn, trang mới xuất hiện). Nghi ngờ "thang điểm có thể đã đổi" ở bản trước của giai đoạn này là sai; điểm của b1 cao hơn a4 nhiều khả năng do truy vấn khác (b1 thêm "Daniel Ek").
  - Truy vấn vô nghĩa `zorbleflax quintanuvo`: 10 kết quả, 0.0072–0.0167; `0.20` cắt sạch cả 10, app sẽ báo không tìm thấy.
  - Nguồn đúng chủ đề thấp nhất: 0.3233 (truy vấn `why are musicians protesting`). Khoảng trống hiện nay là 0.0589 (rác khớp chữ) đến 0.3233 (nguồn đúng), và `0.20` nằm gần trung điểm (≈ 0.19). Giữ `0.20`; comment trong `config.py` đã cập nhật theo 82 kết quả (45 + 37). Biên an toàn phía nguồn tốt hẹp hơn lúc đầu (0.12 thay vì 0.22).
  - Loại rác thứ hai không bị ngưỡng chặn: xem Bug 25.
- Dịch vụ bên thứ ba, Tavily:
  - Dùng `search_depth="basic"`. Theo trang giá chính thức, 1 credit mỗi lần gọi, giá do `search_depth` quyết định; gói free 1.000 credit mỗi tháng. `max_results` không ảnh hưởng giá (đã kiểm chứng bằng số đo thực tế bên dưới), nên `top_k` không bảo vệ được quota.
  - Đã dùng 26/1000 credit (bạn đọc từ dashboard sau lần đo điểm; mình ghi theo mẫu số 1.000 của gói free, coi "26/100" là gõ thiếu một số 0). Từ mốc 15 trở đi mọi lần đếm tay đều khớp dashboard: 6 lần gọi (a4, a5, a6, b1, b2, b3) tăng +6 (giả định mốc 15 đã tính cả a3), rồi 5 lần probe tăng +5 (21 thành 26): `probe_score.py` (1 truy vấn `first`, `max_results=5`) tăng 21 thành 22, `probe_scores.py` (4 truy vấn, `max_results=10`) tăng 22 thành 26 (số 22 do bạn báo ở Giai đoạn 21). Vậy 1 credit mỗi lần gọi, không phụ thuộc `max_results`. Phần trước mốc 15, đếm tay lệch dashboard 2–3, chưa giải thích được.
  - Đơn vị là credit, không phải token.

**Quyết định kỹ thuật:**

- **Vấn đề:** sửa rewriter bằng cách chỉnh prompt hay bằng cách buộc trích nguyên câu trong history rồi mới viết truy vấn
- **Đã chọn:** (bạn chọn) sửa prompt trước (bỏ lối thoát standalone + ví dụ thứ 2)
- **Vì sao:** rẻ nhất, thử trước; cách buộc trích nguyên câu để dành nếu prompt không đủ
- **Vấn đề khác:** ngưỡng lọc `MIN_SCORE`
- **Đã chọn:** (bạn chọn) 0.20, không ghi log các kết quả bị loại
- **Vì sao:** ưu tiên nguồn chuẩn xác và loại rác triệt để; 0.20 nằm gần trung điểm (≈ 0.19) của khoảng trống giữa rác khớp chữ (≤ 0.0589) và nguồn đúng (≥ 0.3233). Đánh đổi: biên an toàn phía nguồn tốt hẹp (0.12); truy vấn mà mọi nguồn tốt chỉ đạt khoảng 0.35 sẽ mất hết nguồn, và score không so sánh được giữa các câu hỏi nên không nên đẩy ngưỡng cao hơn nhiều
- **Vấn đề khác:** sửa prompt trả lời bằng luật chữ (lựa chọn 2) hay bằng cơ chế trích dẫn kiểm chứng được (lựa chọn 1)
- **Đã chọn:** (bạn chọn) lựa chọn 2 trước. Lựa chọn 1 (mỗi câu kèm đoạn trích nguyên văn, server kiểm tra đoạn đó nằm trong snippet nguồn được cite; làm bản chỉ đo, chưa chặn; dùng làm metric cho evaluation mới) để sau
- **Vì sao:** lựa chọn 2 rẻ. Lựa chọn 1 có giới hạn đã thấy ở b2: quote có thật trong snippet nhưng claim vẫn sai (gán nhầm người nói; ngày bài báo bị coi là ngày phát hành), nên cơ chế này không bắt được hai lỗi đó; muốn bắt cần thêm một bước LLM kiểm tra
- **Vấn đề khác:** nguồn trùng (cùng một bài, nhiều URL)
- **Đã chọn:** (bạn chọn) hoãn dedupe
- **Vì sao:** đã thấy ở lần chạy Venice (4/5 nguồn là cùng một bài Euronews và bản Yahoo đăng lại), chưa thấy lại trong b1–b3
- **Vấn đề khác:** các hành vi nhỏ thêm khi sửa code
- **Đã chọn:** (mình tự chọn, chưa hỏi bạn, có thể đổi) kết quả không có `score` thì giữ lại; rewriter trả chuỗi rỗng thì dùng câu gốc; `/search` cũng bị lọc; câu báo không có nguồn đổi thành "relevant web sources"
- **Vì sao:** không đoán được score thì không bỏ (nếu API ngừng trả score, bỏ hết còn tệ hơn); model reasoning có thể trả rỗng khi hết token và search chuỗi rỗng sẽ lỗi; hai lựa chọn còn lại không ghi lý do riêng

**Bug gặp phải:**

- **Bug 21 — Rewriter trả nguyên văn follow-up:** "tell me more about the Spotify protest" được giữ nguyên, nên tìm ra chiến dịch UMAW 2021 thay vì vụ các ban nhạc rút nhạc vì khoản đầu tư quốc phòng của CEO Ek → prompt v1 có luật "standalone thì giữ nguyên" mâu thuẫn với luật "bỏ tell me more" → sửa bằng prompt v2. Kết quả: a4 và b1 đều ra truy vấn có Hotline TNT, King Gizzard, defense và không còn "tell me more"; a6 (đổi chủ đề sang quantum computing) chỉ còn `quantum computing`, không lẫn từ nào của history, nên không hồi quy.
- **Bug 22 — Nguồn rác lọt vào prompt:** truy vấn "Which of the reasons for musicians protesting started first?" trả 5 trang rác (score 0.013–0.022), LLM nhặt chi tiết từ trang rác rồi cite `[5]` → Tavily khớp từ "first" với các trang không liên quan → thêm `MIN_SCORE`. Sau đó một trang Instagram toàn chữ "profile picture" (0.0589) vẫn lọt qua ngưỡng 0.05, nên nâng lên 0.20.
- **Bug 23 — Lỗi của câu trả lời cũ lây sang lượt sau:** ở a5, truy vấn viết lại có "Grammys" nên kết quả đầu là bài Fox News về lịch sử giải Grammy của Macklemore, không liên quan bài hát → rewriter sao chép trung thành chi tiết sai trong history ("Macklemore ... performed at the Grammys", thực tế chỉ Lennox biểu diễn) → chưa sửa. Ở b2 rewriter không chép chi tiết đó nữa, nên đây là không ổn định chứ không phải đã hết.
- **Bug 24 — Snippet Wikipedia nhiều lần chỉ là mục tham khảo (footnote):** b2 viết "released on 7 May 2024", trong khi snippet ở lần chạy a5 ghi đúng 6/5/2024 → Tavily đôi khi trích đúng đoạn mục tham khảo, LLM dựng claim từ tiêu đề và ngày của footnote (ngày một bài báo bị coi là ngày phát hành) → đã sửa ở Giai đoạn 21 (`strip_reference_lines()` trong `search.py`).
- **Bug 25 — Điểm cao không bảo đảm đúng ý người dùng (truy vấn xấu hoặc mơ hồ):** đo bằng `probe_scores.py`, truy vấn `first` trả các trang tên FIRST (an ninh mạng, robot, từ điển) với điểm 0.3898–0.7380, truy vấn `tell me more about it` trả toàn trang giải nghĩa cụm "tell me about it" với điểm 0.4555–0.7566 → `score` đo mức khớp với chữ trong truy vấn chứ không phải ý người dùng, và ngưỡng đủ cao để chặn loại này sẽ cắt luôn nguồn tốt → chưa sửa, chưa có cách rẻ. Phòng thủ còn lại là rewriter (Bug 21) và luật prompt "bỏ qua nguồn nói về chủ đề khác"; lượt hỏi đầu tiên không đi qua rewriter nên không có lớp phòng thủ thứ nhất.
- **Bug 26 — Cite sai nguồn hoặc claim ngoài nguồn:** a3 ("fuels exploitation" cite `[2]` nhưng snippet `[2]` bị cắt trước đoạn đó), a4 (cite Bandcamp sai nguồn), a6 (câu "exponentially more information" nằm ở `[3]` nhưng cite `[1]`; câu "unitary transformations" nằm ở `[2]` nhưng cite `[1][4]`), b2 (số stream Spotify không kiểm chứng được vì snippet không có tên bài) → chưa xác định chắc nguyên nhân; luật prompt "cite đúng nguồn chứa claim" giảm nhưng không loại hẳn (lượt nào cũng còn lỗi) → chưa sửa, hướng để dành là lựa chọn 1 (trích dẫn kiểm chứng được).
- **Bug 27 — Khái quát quá tay và đẩy mạnh nguyên nhân:** a4 ("all of the bands cited the same trigger"), a5 ("led to his removal", bỏ chữ "allegedly"), b1 (từ một ban nhạc giữ nhạc trên Bandcamp lên "the affected bands") → LLM nâng ví dụ lẻ thành quy luật chung → thêm 3 luật chữ (hedging, thời gian tuyệt đối, không khái quát). Kết quả: thời gian tuyệt đối đạt (b1); khái quát vẫn vi phạm (b1); luật hedging chưa kiểm thử được vì nguồn của b3 không có câu nào có từ hedging.
- **Bug 28 — Gộp sự kiện, gán nhầm người nói, mốc thời gian lệch:** a3 gộp Mitchell/Lofgren vào chiến dịch UMAW và dùng "recent/last year" lệch mốc; b2 gán lời của tác giả bài The Conversation cho Macklemore ("Macklemore says ...") → chưa xác định chắc nguyên nhân → luật "không gộp sự kiện" và "đổi sang năm tuyệt đối" đã thêm; chưa có cách bắt lỗi gán nhầm người nói (lựa chọn 1 cũng không bắt được).

**Việc còn tồn đọng từ giai đoạn này:**

- Loại rác thứ hai (truy vấn xấu hoặc mơ hồ, Bug 25): chưa có hướng xử lý; theo dõi xem rewriter có giữ được qua nhiều ca hơn không
- ~~Snippet Wikipedia chỉ là mục tham khảo (Bug 24): chưa quyết hướng xử lý~~ — đã làm ở Giai đoạn 21
- Bug 23 (lỗi lây qua history) chưa có cách sửa rẻ
- Prompt trả lời còn lỗi (Bug 26, 27, 28): quyết định lựa chọn 1 (trích dẫn kiểm chứng được) và giới hạn đã thấy ở b2 xem phần Quyết định kỹ thuật
- Chưa làm: dedupe nguồn trùng, kiểm thử luật hedging
- ~~Rate limit cho `/ask`: mỗi lần gọi tốn 1 credit, site công khai có thể bị gọi liên tục; chưa quyết~~ — đã làm ở Giai đoạn 22 (trên nhánh riêng, chưa merge)
- Chưa kiểm chứng: trên các lượt `/ask` thật (b1–b3) chưa có kết quả nào bị `MIN_SCORE` loại (15 kết quả, thấp nhất 0.657); `0.20` mới được thấy cắt đúng trong lần đo trực tiếp (truy vấn vô nghĩa, 10/10 kết quả). Chất lượng prompt trả lời mới chỉ kiểm bằng đọc tay vài câu trả lời; chưa có metric
- Giữ nguyên từ Giai đoạn 18–19: frontend (sources hiển thị `title`/`url`, `[n]` bấm được), thêm `TAVILY_API_KEY` trên Render, thiết kế lại evaluation, gỡ thành phần thừa của bản BBC, chưa xác nhận đã xoá `chat_history.db` cũ

---

## Giai đoạn 21 — Lọc footnote Wikipedia khỏi snippet (sửa Bug 24)

**Mục tiêu giai đoạn:** Chặn lỗi LLM đọc ngày của một bài báo trong mục tham khảo (References) của Wikipedia thành ngày phát hành bài hát: ở lần chạy b2, câu trả lời viết "released on 7 May 2024" trong khi ngày đúng là 6/5/2024 (Bug 24).

**Đã làm:**

- `search.py`: thêm `strip_reference_lines()` và gọi trong `web_search()` ngay sau khi lấy `content`, trước bước loại nguồn rỗng. Cách làm: tách snippet theo `[...]`, xoá từng dòng footnote trong mỗi đoạn, ghép lại các đoạn còn chữ. Snippet còn 0 ký tự thì bị dòng `if not url or not snippet.strip()` có sẵn loại cả nguồn.
- Nhận dạng footnote: dòng bắt đầu bằng `số.` rồi `↑` luôn là footnote, kể cả khi dòng bị cắt giữa chừng; dòng dạng `8. 1 2 3 "..."` chỉ bị xoá khi có chữ `Retrieved` hoặc `Archived`, để dòng kiểu `1. 2 eggs` được giữ.
- Kiểm chứng trên dữ liệu thật (bạn chạy trên 50 snippet đã lưu trong `chat_history.db`): 5 snippet bị sửa (đều là trang Wikipedia có footnote), 45 snippet còn lại không đổi, nên không có xoá oan. Snippet Hind's Hall sai ngày (1264 thành 0 ký tự) bị bỏ hẳn; snippet Hind's Hall đúng ngày (1229 thành 813 ký tự) vẫn giữ câu "Released as a single on May 6, 2024".
- Mình kiểm lại trong sandbox: chạy `strip_reference_lines()` trên nguyên văn snippet Wikipedia của b2 thì ra rỗng; trên snippet Wikipedia của a3 chỉ còn câu của Yorke, các dòng footnote về Mitchell và Lofgren (nguồn của claim bị gộp ở Bug 28) biến mất. Tắt riêng lời gọi hàm thì đúng test adapter fail.
- `test_backend.py`: thêm 3 test offline (không tốn credit): xoá footnote khỏi snippet; giữ văn bản thường, kể cả dòng đánh số kiểu `1. 2 eggs`; nguồn chỉ có footnote bị adapter bỏ. `python test_backend.py` cho 18/18 PASS (chạy lại ở sandbox của mình cũng 18/18).
- Chạy thật 1 lần (c1: "when was the song Hind's Hall by Macklemore released", lượt đầu nên không qua rewriter): 5 nguồn, score 0.453–0.921, không nguồn nào bị `MIN_SCORE` loại; 0 dòng footnote trong snippet (đếm dòng bắt đầu bằng `số. ↑` bằng `grep`). Câu trả lời đúng: "released on May 6 2024" cite `[1][5]` (Wikipedia và một bài đăng Instagram, đều có ngày 6/5), kèm ngày liệt kê trên Spotify "May 10 2024" cite `[3]` (snippet Spotify ghi May 10, 2024); không còn "7 May 2024". Lưu ý: snippet Wikipedia lần này gồm câu mở đầu, bảng thông tin và phần thân bài, không phải footnote, nên lần chạy này chưa tái hiện đúng ca lỗi gốc và không cho biết bộ lọc đã loại gì; bằng chứng cho bộ lọc vẫn là 5/50 snippet trong `chat_history.db` và test offline. Quan sát thêm: câu hỏi "released" khiến Tavily trích đúng đoạn bảng thông tin có dòng `Released | May 6, 2024`, tức đoạn Tavily trích phụ thuộc cách đặt câu hỏi.
- Tavily: kiểm chứng bằng snippet đã lưu không tốn credit; lần chạy thật c1 tốn 1 credit. Dashboard 28/1000 (bạn báo): 26 sau lần đo điểm, cộng 1 lượt `/ask` nằm giữa b3 và c1 (suy ra từ `conversation_id` nhảy từ 8 lên 10) và cộng c1. Phần suy ra này chưa được bạn xác nhận.

**Quyết định kỹ thuật:**

- **Vấn đề:** xử lý snippet Wikipedia chỉ là mục tham khảo bằng cách bỏ cả snippet, xoá riêng dòng footnote, hay lấy `raw_content` của trang
- **Đã chọn:** (bạn chọn) xoá riêng các dòng footnote, chỉ bỏ nguồn khi xoá xong không còn chữ nào
- **Vì sao:** bỏ cả snippet theo kiểu "phần lớn là dòng tham khảo" có thể mất luôn câu đúng ở snippet lẫn (như a5, câu "Released as a single on May 6, 2024"); lấy `raw_content` tốn thêm độ trễ, giá credit chưa kiểm chứng, và cả trang dài làm prompt phình ra cho model 20B. Đánh đổi: nguồn Wikipedia bị bỏ thì câu trả lời có thể thiếu ngày phát hành, chấp nhận thiếu còn hơn sai

**Việc còn tồn đọng từ giai đoạn này:**

- Giới hạn đã biết: chỉ chữa hai dạng footnote đã thấy (`số. ↑` và `số. 1 2 3 ... Retrieved/Archived`). Mình thử và thấy chưa lọc được: `13. ^ ...` (dấu `^` thay cho `↑`), dòng `↑ ...` không có số, footnote dính sau câu văn trên cùng một dòng, dòng dạng `8. 1 2 3 ...` không có chữ Retrieved/Archived. Ngược lại, một dòng đánh số hợp lệ có `↑` (ví dụ `3. ↑ 5% growth`) sẽ bị xoá oan. Các dạng này chưa thấy trong dữ liệu thật (5/50 snippet bị sửa đều đúng là footnote) nên chưa xử lý
- Chưa chữa: gán nhầm người nói (Bug 28), trang Spotify toàn danh sách quốc gia (rác giao diện), trang không phải Wikipedia trích nhầm danh sách tham khảo theo định dạng khác
- Chạy thật ca lỗi gốc (Tavily trả snippet Wikipedia chỉ có footnote) để xem câu trả lời còn sai ngày không: đã thử một biến thể (c1, hỏi thẳng ngày phát hành) cho đáp án đúng, nhưng snippet Wikipedia lần đó không phải footnote nên chưa tái hiện được ca lỗi; test hiện tại chỉ chứng minh logic lọc, chưa chứng minh chất lượng câu trả lời trên ca lỗi gốc
- Xác nhận `config.py` trong repo là bản có comment sàn nguồn đúng 0.3233 (bản `config_1.py` bạn gửi trước đó đã có; trước đó ghi 0.420)
- Nguồn mạng xã hội được cite như bằng chứng: bài đăng Facebook (b1 `[3]`), Reddit (b1 `[5]`), Instagram (c1 `[5]`, trang này còn lẫn bình luận người dùng và vẫn đạt score 0.453). Chưa gây sai nội dung, nhưng chưa quyết có lọc theo tên miền hay không
- Kiểm tra chất lượng `rewrite_query_with_history` trên nhiều ca hơn (Bug 25: bộ lọc score không chặn được truy vấn xấu, nên chất lượng rewriter quan trọng hơn con số ngưỡng)

---

## Giai đoạn 22 — Rate limit cho /ask và /search (Upstash Redis)

**Mục tiêu giai đoạn:** Chặn việc site công khai bị gọi liên tục làm cạn quota Tavily: mỗi lần gọi `/ask` hoặc `/search` tốn 1 credit (đã đo), gói free chỉ có 1.000 credit mỗi tháng. Cần xong trước khi đặt `TAVILY_API_KEY` trên Render.

**Đã làm:**

- `rate_limit.py` mới: kiểm tra 3 giới hạn cửa sổ cố định TRƯỚC khi chạy bất kỳ lần search nào: `ip_minute` (5 lần mỗi phút mỗi IP), `ip_day` (10 lần mỗi 24 giờ mỗi IP), `global_day` (30 lần mỗi ngày UTC cho tất cả IP). Giới hạn đặt bằng 0 là tắt; `REDIS_URL` để trống là tắt toàn bộ rate limit (dùng khi chạy local và trong test offline).
- Bộ đếm nằm ở Redis (Upstash, qua `redis-py`). Một script Lua kiểm tra cả 3 bộ đếm rồi mới tăng nếu cả 3 đều qua, nên một request bị từ chối không tốn quota và không có đua giữa các instance.
- `main.py`: `/ask` và `/search` dùng chung dependency `enforce_rate_limit`, nên chung một ngân sách. Vượt giới hạn trả 429 với `detail` là object `{code: "rate_limited", limit, retry_after, message}` kèm header `Retry-After`; Redis không dùng được trả 503 `rate_limiter_unavailable`.
- IP khách lấy từ `X-Forwarded-For` đọc từ bên PHẢI với `TRUSTED_PROXY_HOPS` (mặc định 1), vì phần bên trái có thể bị client giả mạo; `RATE_LIMIT_DEBUG=true` ghi log header để đo số proxy thật trên Render.
- `config.py`: thêm `REDIS_URL`, `RATE_LIMIT_*`, `TRUSTED_PROXY_HOPS`, `RATE_LIMIT_DEBUG`. `requirements.txt` còn đúng các gói của luồng web search cộng `redis`; `requirements-dev.txt` (`fakeredis[lua]`, `httpx`) dành cho test.
- `test_backend.py`: thêm 5 test offline dùng `fakeredis` chạy đúng script Lua: giới hạn theo phút tính riêng từng IP; trần toàn cục và việc request bị từ chối không tốn quota; `/ask` trả 429 mà không tới Tavily và `/search` dùng chung ngân sách; đọc IP từ bên phải của `X-Forwarded-For`; Redis hỏng thì đóng theo mặc định và mở khi cấu hình. 23/23 PASS (chạy lại ở sandbox của mình cũng 23/23).
- Mình kiểm lại bằng cách phá từng chốt chặn: bỏ dependency khỏi `/ask` hoặc `/search`, đọc IP từ bên trái, bỏ vòng "kiểm tra hết rồi mới tăng" trong Lua đều làm đúng test tương ứng fail. Thử thêm bằng tay: khoá phút có TTL 60 giây, khoá ngày và khoá toàn cục 86.400 giây; sau khi cửa sổ phút hết hạn thì cho qua lại; `ip_day=10` cho 10 lần rồi chặn; trần toàn cục sang ngày UTC mới thì cho qua lại.
- Dịch vụ bên thứ ba, Upstash Redis: dùng qua `REDIS_URL` (dạng `rediss://`). Mình chưa kiểm chứng giới hạn của gói free.

**Quyết định kỹ thuật:**

- **Vấn đề:** giới hạn theo IP, theo trần toàn cục mỗi ngày, hay cả hai
- **Đã chọn:** (bạn chọn) cả hai, thêm giới hạn IP theo ngày: IP 5 lần mỗi phút, IP 10 lần mỗi ngày, toàn cục 30 lần mỗi ngày
- **Vì sao:** trần toàn cục là mức tối đa cho credit: 30 × 31 ngày = 930 < 1.000; giới hạn IP theo phút chặn script gọi nhanh; giới hạn IP theo ngày chặn một IP ăn hết trần toàn cục làm mọi người khác bị khoá
- **Vấn đề khác:** nơi lưu bộ đếm: bộ nhớ, SQLite hay Redis ngoài
- **Đã chọn:** (bạn chọn) Upstash Redis
- **Vì sao:** bền qua lần khởi động lại của Render (bộ nhớ thì mất, file SQLite trên filesystem tạm của Render free cũng mất, xem Giai đoạn 17) và dùng chung nếu có nhiều instance. Đánh đổi: thêm một dịch vụ ngoài phải quản lý
- **Vấn đề khác:** khi Redis không dùng được (lỗi mạng hoặc hết quota) thì cho request qua hay chặn
- **Đã chọn:** (bạn chọn) chặn, trả 503 (`RATE_LIMIT_FAIL_OPEN=false`)
- **Vì sao:** ưu tiên bảo vệ credit hơn giữ site chạy. Đánh đổi: nếu Upstash lỗi hoặc hết quota thì site ngừng phục vụ cả người dùng thật
- **Vấn đề khác:** thứ tự làm và merge
- **Đã chọn:** (bạn chọn) làm trên nhánh riêng, test local với Upstash trước, đo header IP trên Render sau, chỉ merge vào `main` khi xong hết
- **Vì sao:** không để `main` và site đang chạy bị ảnh hưởng bởi phần chưa đo xong

**Bug gặp phải:** Không có.

**Việc còn tồn đọng từ giai đoạn này:**

- Đo số proxy thật trên Render bằng `RATE_LIMIT_DEBUG=true` trước khi merge: gửi request từ một IP đã biết (ví dụ điện thoại dùng 4G), so `x-forwarded-for` với IP công khai của nó, rồi đặt `TRUSTED_PROXY_HOPS` sao cho phần tử thứ N tính từ cuối đúng là IP đó. Nếu đặt sai, mọi người dùng chung một IP và giới hạn 10 lần mỗi ngày thành trần cho cả site. Đây là rủi ro lớn nhất của giai đoạn này, chưa đo
- ~~Kiểm tra frontend gọi backend trực tiếp từ trình duyệt (CORS cho domain Vercel gợi ý đúng như vậy) chứ không qua route server-side của Next.js; nếu qua server thì mọi request mang IP của Vercel~~ — đúng: `page.tsx` là client component và gọi `fetch` thẳng tới `API_URL` (kết luận từ đọc code, nên kiểm lại bằng tab Network sau khi deploy)
- Chưa thấy kết quả test với Upstash thật (các test hiện có dùng `fakeredis`): cần chạy local với `REDIS_URL` thật, gồm `EVALSHA`/Lua và trường hợp hết quota
- Chưa merge vào `main` và chưa deploy: cần đặt `REDIS_URL` và các biến `RATE_LIMIT_*` trên Render
- ~~Frontend phải xử lý 429 và 503: `detail` của hai mã này là object, trong khi lỗi 502 hiện có `detail` là chuỗi~~ — đã làm ở Giai đoạn 23 (`utils/api.ts`)
- Trần toàn cục 30 mỗi ngày × 31 ngày = 930 credit chỉ chừa 70 credit mỗi tháng cho việc tự test của bạn (đã dùng 28 credit đến thời điểm này); cân nhắc hạ xuống khoảng 25
- Chưa có test cho giá trị mặc định "đóng khi Redis hỏng" trong `config.py`: đổi mặc định sang "mở" thì không test nào fail (mình đã thử)
- Chưa kiểm chứng giới hạn gói free của Upstash; nếu bị spam làm hết quota Redis thì site trả 503 cho mọi người (hệ quả của quyết định đóng)
- `requirements.txt` không còn gói của bản BBC (`langchain-community`, `langchain-qdrant`, `qdrant-client`, `fastembed`, `pandas`, `langchain-ollama`, `kaggle`), nên `ingest.py`, `evaluate.py`, `setup.py` không chạy được từ file này. Muốn chạy `evaluate.py` lấy số cho CV thì phải cài thêm hoặc tạo `requirements-legacy.txt`

---

## Giai đoạn 23 — Frontend v2, README v2 và CI

**Mục tiêu giai đoạn:** Đưa giao diện về đúng với API v2 (nguồn web, `[n]` bấm được, thông báo lỗi 429/503), viết lại README và thêm CI chạy test offline, để các dòng trong CV kiểm chứng được qua demo và GitHub.

**Đã làm:**

- Frontend (bạn sửa): `utils/api.ts` (`askQuestion`, `ApiError`, đọc `detail` đúng theo từng mã: object cho 429/503, chuỗi cho 502); `utils/citations.ts` (`linkCitations`, `isHttpUrl`, `hostname`); `app/page.tsx` (câu trả lời qua `react-markdown`, `[n]` thành link, danh sách nguồn, banner lỗi nằm ngoài vùng chat, đếm ngược cho giới hạn theo phút, trả lại `turns` khi câu hỏi lỗi, nhóm "Try asking" với 3 câu đã chạy thật); `layout.tsx` (đổi tiêu đề); `globals.css`.
- Mình kiểm lại trong sandbox: `tsc --noEmit` ở chế độ strict đạt (react-markdown 10, next 16); 20 kiểm tra hành vi đạt. Với câu trả lời thật c1, `[1]`, `[5]`, `[3]` trỏ đúng Wikipedia, Instagram, Spotify; `[9]` bịa giữ nguyên chữ; `[10]` trỏ đúng nguồn thứ 10; URL có ngoặc, khoảng trắng, `[ ]` vẫn thành link đúng; nguồn cũ kiểu BBC không có `url` không thành link; chặn `javascript:` (trong nguồn lẫn trong link do model viết), ảnh và HTML thô.
- CI: `.github/workflows/tests.yml` (Python 3.12, `pip install -r requirements-dev.txt`, `python test_backend.py`, không cần khoá nào). Mô phỏng từ virtualenv sạch: thoát mã 0 với 23 test PASS, và mã 1 khi cố tình làm hỏng một test.
- README viết lại theo cấu trúc cũ của bạn cho v2, kèm `.env.example`: huy hiệu build chuyển sang huy hiệu GitHub Actions, thêm mục Status (các ô tích), Privacy, Known Limitations, mục Troubleshooting cho 429/503/CORS; bỏ các chỗ chưa điền (`image.png`, `<username>`, "Your Name").

**Quyết định kỹ thuật:**

- **Vấn đề:** sidebar liệt kê mọi hội thoại của mọi người (`GET /conversations` không lọc theo người dùng, `/conversations/{id}` mở cho ai cũng gọi được)
- **Đã chọn:** (bạn chọn) giữ nguyên, ghi rõ trong README mục Privacy rằng hội thoại trên demo là công khai
- **Vì sao:** không ghi lại lý do lúc chốt; hai phương án khác đã đưa ra: ẩn danh sách hội thoại trên production, hoặc mã client trong `localStorage` để backend lọc. Đánh đổi: người xem demo thấy câu hỏi của nhau, nên README dặn không nhập thông tin cá nhân
- **Vấn đề khác:** huy hiệu build trong README
- **Đã chọn:** (bạn chọn viết CI) dùng huy hiệu của GitHub Actions thay cho ảnh tĩnh "build passing"
- **Vì sao:** ảnh tĩnh ghi "passing" trong khi repo chưa có CI nào chạy

**Bug gặp phải:**

- **Bug 29 — Nút ví dụ nối vào hội thoại đang mở:** bấm một câu trong "Try asking" giữa chừng một hội thoại khác sẽ đưa chủ đề cũ cho rewriter và lưu câu hỏi chung hội thoại đó → `runQuery(p)` dùng `turns` và `conversationId` hiện tại → đã đưa cách sửa (tham số `fresh` để ví dụ luôn mở hội thoại mới) và thử typecheck trên bản sao; chưa xác nhận bạn đã áp dụng.
- **Bug 30 — Ví dụ và hội thoại cũ biến mất trên điện thoại:** màn hình ≤ 720px ẩn `.sidebar-quick-prompts`, trong khi màn hình trống vẫn dặn "try an example from the sidebar" → CSS responsive ẩn khối đó → đã đưa cách sửa (đưa 3 ví dụ thành nút ngay trong màn hình trống); chưa xác nhận đã áp dụng.
- **Bug 31 — Chữ "searching the web…" hiện cả khi mở lại hội thoại cũ:** `loadConversation` và `runQuery` dùng chung `loading` → chưa sửa, chỉ là sai chữ.

**Việc còn tồn đọng từ giai đoạn này:**

- Áp dụng cách sửa Bug 29, 30, 31 (nếu chưa)
- Đưa `tests.yml` lên cùng nhánh với code v2: nếu đưa lên `main` bây giờ thì CI đỏ vì `main` chưa có `test_backend.py`; huy hiệu chỉ hiện sau khi workflow chạy trên nhánh mặc định
- README: thêm ảnh chụp màn hình; xác nhận repo có file `LICENSE` (lúc mở danh sách file của repo mình không thấy); tích các ô trong mục Status khi xong từng việc; sửa "Python >= 3.10" thành 3.12 (đã làm trong README mới, CI chỉ chạy 3.12)
- Ghi chú mâu thuẫn: README cũ ghi UptimeRobot giữ server thức, nhật ký (Giai đoạn 17–18) ghi cron-job.org; README mới viết trung tính ("an external uptime monitor"), cần xác nhận dịch vụ thật
- Backend chưa bắt lỗi Groq (hết lượt, quá thời gian): hiện sẽ ra lỗi 500 chung. README dự án Agent của bạn ghi giới hạn 8.000 token mỗi phút của gói free và việc SDK tự retry ngầm; backend này dùng cùng model nên có thể gặp tương tự, chưa quan sát và chưa đo token mỗi request. Nên bắt lỗi Groq thành 502 có thông báo và đo token trước khi giữ mức 5 lần mỗi phút
- Cập nhật CV khi merge: "18 offline tests" thành 23; mục tồn đọng của Giai đoạn 22 (đo hops, test Upstash thật, merge, deploy) vẫn nguyên

---

## Giai đoạn 24 — Đối chiếu CV với dự án và chuẩn bị bộ đo cho bản Perplexity clone

**Mục tiêu giai đoạn:** Điền các placeholder (X/Y) trong mô tả dự án trên CV bằng số thật, và phát hiện chỗ CV không khớp với nhật ký hay repo trước khi đo.

**Đã làm:**

- Đối chiếu từng placeholder của CV với nhật ký:
  - Điền được ngay bằng số có sẵn: ngưỡng `MIN_SCORE=0.20` chọn trên 82 kết quả thật (45 + 37), rác khớp chữ cao nhất 0.0589, nguồn đúng thấp nhất 0.3233; truy vấn vô nghĩa bị cắt 10/10 kết quả; bộ lọc footnote sửa 5/50 snippet đã lưu, 45 snippet còn lại không đổi.
  - Chưa có số: Recall@5 hybrid so với dense-only (dữ liệu duy nhất là 2 câu ở Giai đoạn 11, hai bên bằng nhau: Recall@5 1.00, Precision@5 0.80; kết quả bộ 10 câu ở Giai đoạn 13 chưa được ghi lại); p50 latency (chưa đo bao giờ; 0.94–1.25 giây ở Giai đoạn 18 chỉ là phía Tavily); "cut irrelevant citations from X to Y" (chưa có số trước và sau).
  - Placeholder của dự án Agent (injection cases, run lỗi, 429 retry) không nằm trong nhật ký này.
- Phát hiện 3 chỗ CV không khớp: (1) "18 offline tests" phải là 23; (2) bullet hybrid retrieval trên 2,225 bài thuộc bản BBC đã dừng, không còn trong luồng chính; (3) repo GitHub `main` (mình đọc trang repo) vẫn là bản BBC "Wire Search": README mô tả hybrid search trên kho BBC, có `ingest.py`, `evaluate.py`, `label_helper.py`, không thấy `search.py`, `rate_limit.py`, `test_backend.py` hay CI trong danh sách file. Link demo Vercel mình mở không được (lỗi client), nên chưa biết nó chạy bản nào.
- Đọc `main.py` và `search.py` bạn gửi: `main.py` chỉ là file chuyển tiếp sang `backend.main` (xem mục Cấu trúc thư mục); `search.py` khớp mô tả Giai đoạn 21 (`strip_reference_lines`, lọc `MIN_SCORE`, `WebSearchError`).
- Tạo `eval_queries.json`: 20 câu cố định (10 bình thường, 5 mơ hồ/ngắn, 5 follow-up có `history` cố định). Không câu nào trùng với câu đã dùng để chỉnh prompt hay `MIN_SCORE`. Tên field trong `history` là tạm, chưa khớp `ChatTurn`.
- Rà `README.md` v2 (bản bạn gửi): bỏ tham chiếu tới mục Status không tồn tại, đổi "irrelevant pages" thành "keyword-matching junk pages" ở phần cutoff, bỏ hai câu mình không kiểm chứng được từ nhật ký ("BM25-only" và "40-query results"; bạn chọn bỏ). `eval_queries.json` đổi `history` sang `{role, content}` theo mục API của README. Mục Project Structure chưa sửa: `main.py` gốc chỉ chuyển tiếp sang `backend.main`, cần cây thư mục thật.
- Bạn dán `git ls-files` của repo (sau khi báo đã merge): có `backend/` (`main.py`, `search.py`, `rate_limit.py`, `config.py`, `db.py`, `test_backend.py`, `requirements.txt`, cùng `ingest.py`, `evaluate.py`, `label_helper.py` của v1 và `backend/data/`), `frontend/utils/api.ts`, `frontend/utils/citations.ts` và `env.example`. Không nằm trong danh sách file đã commit: `main.py` ở gốc (bản chuyển tiếp bạn gửi), `.github/workflows/tests.yml`, `requirements-dev.txt`, `probe_scores.py`, `setup.py`. Ngoài ra `frontend/tsconfig.tsbuildinfo` (file build) đang được commit, và `backend/data/tfidf_dataset.csv` đang được commit trong khi README cũ ghi CSV không commit.
- Sửa mục Project Structure, tên file `env.example`, đường dẫn `backend/requirements.txt` và danh sách script v1 trong README theo `git ls-files`.
- Đọc `backend/main.py` và `backend/test_backend.py` bạn gửi: `main.py` import phẳng (`import db`, `from config import ...`) sau khi tự thêm thư mục `backend/` vào `sys.path`, nên `uvicorn backend.main:app` chạy được từ gốc repo mà không cần `main.py` gốc; `test_backend.py` có đúng 23 test (đếm trong file), chạy được bằng `python backend/test_backend.py` và cần `fakeredis[lua]` và `httpx`. README đổi lệnh chạy, lệnh test, lệnh start Render theo đó (`python backend/evaluate.py` của v1 chưa kiểm chứng).
- Soạn lại mô tả dự án trong CV chỉ bằng số đã kiểm chứng (82 kết quả, 0.0589/0.3233, 10/10, 5/50, 23 test, 6 luật prompt) và chỉ ra 4 chỗ đo trong code: thời gian từng bước trong `ask()`, `usage_metadata` của hai lệnh `llm.invoke`, `MIN_SCORE` trong `web_search`, và `answer` kèm `sources[i].snippet` để chấm citation tay.
- Dựng lại CV thành `Giap-Minh-Hien-CV.pdf` (1 trang A4, không dùng template TopCV): dự án Search Engine còn 4 bullet chỉ gồm số đã kiểm chứng, link demo `...-18bt`, bỏ Qdrant khỏi dòng Stack, thêm Redis vào Skills; bullet dự án Agent bỏ các mệnh đề trong ngoặc vuông (injection cases, nguyên nhân run lỗi, cách sửa 429) vì chưa có số. Các mệnh đề đo của Search Engine (p50/p95, nguồn rác bị loại, citation accuracy) chưa đưa vào CV vì chưa đo.
- Chốt trước luật chấm: citation đúng khi claim nằm trong snippet của nguồn được cite (đúng ngoài đời nhưng không có trong snippet vẫn tính sai); nguồn liên quan khi snippet nói đúng chủ đề (câu mơ hồ chấm theo nghĩa phổ biến nhất, ghi rõ).

**Quyết định kỹ thuật:**

- **Vấn đề:** bullet hybrid retrieval trên BBC trong CV
- **Đã chọn:** (bạn chọn) không chạy lại `evaluate.py` của bản BBC; viết lại hoặc bỏ bullet vì thuộc phần cũ của dự án
- **Vì sao:** README đã ghi hybrid không hơn dense-only (Recall@5 bằng nhau), và dự án hiện tại không còn hybrid
- **Vấn đề khác:** đo chỉ số nào cho bản hiện tại
- **Đã chọn:** (bạn chọn) p50/p95 latency của `/ask`; tỉ lệ citation đúng nguồn trên ~20 câu (chấm tay); so sánh `MIN_SCORE=0` với `0.20` trên cùng bộ câu
- **Vì sao:** thay các placeholder chưa có số. Bạn chỉ chọn phương án, không nêu lý do riêng
- **Vấn đề khác:** thành phần bộ câu hỏi
- **Đã chọn:** (bạn chọn) 10 bình thường + 5 mơ hồ/ngắn + 5 follow-up
- **Vì sao:** lý do do mình đề xuất: ở các lượt `/ask` thật trước đó `0.20` chưa cắt nguồn nào trên câu bình thường (thấp nhất 0.657), nên cần câu mơ hồ để thấy tác dụng của ngưỡng. Đánh đổi: tỉ lệ câu mơ hồ ảnh hưởng kết quả, báo cáo phải nêu rõ tỉ lệ
- **Vấn đề khác:** thiết kế phép đo
- **Đã chọn:** (mình tự chọn, chưa hỏi bạn, có thể đổi) gọi Tavily 1 lần mỗi câu (~20 credit), lưu kết quả thô rồi áp hai ngưỡng ngoại tuyến; đo latency chạy local (không rate limit), ghi rõ môi trường, tách thời gian Tavily và Groq, ghi token mỗi lần
- **Vì sao:** chênh lệch giữa hai cấu hình chỉ đến từ ngưỡng, và tiết kiệm credit (đã dùng 28/1000 ở Giai đoạn 22). Đánh đổi: latency local không phản ánh Render
- **Vấn đề khác:** `.github/workflows/tests.yml` không có trong repo, nhưng README nói CI chạy trên mỗi push
- **Đã chọn:** (bạn chọn) bỏ mọi câu về CI khỏi README và CV, thay vì commit workflow
- **Vì sao:** bạn không nêu lý do riêng. Hệ quả: README không còn ghi GitHub Actions, và dòng "Python 3.12 (tested in CI)" đổi về "Python >= 3.10" (mình chưa kiểm chứng 3.10 chạy được; bản gần nhất bạn dùng chưa được ghi lại)

**Bug gặp phải:** Không có.

**Việc còn tồn đọng từ giai đoạn này:**

- Bạn báo đã merge v2 vào `main` và deploy; mình chưa xác nhận được (trang repo mình tải là bản lưu đệm cũ, link demo không mở được). Bản v2 chạy ở `https://ai-powered-search-engine-18bt.vercel.app/` (bạn xác nhận). CV và mục About của GitHub vẫn ghi link cũ `ai-powered-search-engine-rosy.vercel.app`, cần đổi
- Script đo (`backend/eval_run.py`, không sửa code backend, chạy trực tiếp các hàm của `main.py` và `search.py`): đã đủ thông tin để viết, chưa viết (chờ bạn chọn), chưa chạy, chưa có số nào
- Sửa CV: 18 thành 23 test; thay bullet hybrid; Recall@5, p50 và "cut irrelevant citations" chưa có số nên chưa điền được (không dùng "from X to Y" nếu không đo được); không ghi CI vào CV (bạn chọn bỏ mọi câu về CI)
- `.github/workflows/tests.yml` không có trong repo; đã bỏ mọi câu về CI khỏi README. Nếu sau này commit workflow thì thêm lại và ghi CI vào README/CV
- `main.py` ở gốc không nằm trong danh sách file đã commit; cần `git status --short` để biết nó chưa commit hay bị `.gitignore`, và xem lệnh start thực tế đang đặt trên Render (README đã ghi `uvicorn backend.main:app`, chạy được không cần file đó)
- Dọn repo: bỏ `frontend/tsconfig.tsbuildinfo` khỏi git (thêm vào `.gitignore`); xác nhận `backend/data/tfidf_dataset.csv` là dữ liệu gì và sửa mô tả dataset v1 trong README cho khớp
- Placeholder của dự án Agent: cần nhật ký của dự án đó
- Commit README v2 đã rà lên `main` (lần mình xem repo, `main` vẫn hiện README BBC)
- Giữ nguyên các mục tồn đọng của Giai đoạn 22–23 (đo số proxy trên Render, test Upstash thật, merge, deploy, ảnh chụp màn hình README)

# Kiến thức đã áp dụng

- [[RAG]] — Giai đoạn 5, 18–20
- [[Embedding]] — Giai đoạn 3 (bản BBC)
- [[Chunking]] — Giai đoạn 3 (bản BBC)
- [[Vector Database]] — Giai đoạn 3 (bản BBC)
- [[Hybrid Search]] — Giai đoạn 8 (bản BBC)
- [[Conversation Memory]] — Giai đoạn 10
- [[Query Rewriting]] — Giai đoạn 10, 20
- [[Retrieval Evaluation]] — Giai đoạn 11, 13 (Recall@k, MRR, Precision@k)
- [[Cross-Encoder Reranking]] — Giai đoạn 14–15 (đã thử và gỡ)
- [[Citation Grounding]] — Giai đoạn 14–15, 18–20
- [[Web Search API]] — Giai đoạn 18–20
- [[Adapter Pattern]] — Giai đoạn 19 (`search.py`)
- [[Test Doubles]] — Giai đoạn 19–20 (`test_backend.py` giả Tavily và Groq)
- [[Relevance Score Threshold]] — Giai đoạn 20 (`MIN_SCORE`)
- [[Prompt Engineering]] — Giai đoạn 19–20
- [[Regular Expressions]] — Giai đoạn 21 (`strip_reference_lines`)
- [[Rate Limiting]] — Giai đoạn 22 (cửa sổ cố định, script Lua nguyên tử trên Redis)
- [[Continuous Integration]] — Giai đoạn 23 (GitHub Actions chạy test offline)
- [[Evaluation Set Design]] — Giai đoạn 24 (mới thiết kế bộ câu hỏi và luật chấm, chưa chạy)
