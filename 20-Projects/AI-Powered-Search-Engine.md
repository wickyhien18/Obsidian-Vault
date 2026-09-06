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

**Bug gặp phải:**

- **Bug 12 — Follow-up dùng đại từ ("that article") khiến retrieval trả về chunk không liên quan, LLM trả lời "none relevant":** nguyên nhân là `retrieve_chunks()` chỉ dùng đúng văn bản câu hỏi hiện tại để search — câu hỏi "which category does that article belong to?" không chứa từ khoá thật (musicians, protest, visa...) nên hybrid search không tìm ra chunk đúng, dù bản thân LLM đã hiểu đúng "that" ám chỉ gì. Sửa bằng cách thêm bước **Query Rewriting** (`rewrite_query_with_history()`) — gọi LLM viết lại câu hỏi phụ thuộc ngữ cảnh thành câu độc lập, đầy đủ nghĩa, TRƯỚC khi đưa vào retrieval. Có điều kiện bỏ qua bước này nếu `history` rỗng (lượt hỏi đầu tiên), tránh tốn thêm 1 lệnh gọi LLM không cần thiết.
- **Bug 13 (false alarm, không phải bug thật):** lần test đầu tưởng fix không hoạt động ("not explicitly stated") — hoá ra do server chưa được restart sau khi sửa code, vẫn chạy bản cũ. Rút kinh nghiệm: luôn xác nhận server đã reload đúng code mới trước khi kết luận 1 fix thất bại.
- **Bug 14 (false alarm khác):** 1 lần test cho kết quả category sai (`"entertainment"` thay vì `"tech"`) — do vô tình gõ nhầm input là câu đã bị hệ thống viết lại từ trước (thay vì câu hỏi tự nhiên gốc), khiến bước rewrite chạy lần 2 trên 1 câu vốn đã đủ nghĩa, làm trôi chủ đề. Không phải lỗi code — bài học: cẩn thận khi test thủ công để không tự tạo input sai lệch với kịch bản thật.

**Đánh đổi:** mỗi câu hỏi từ lượt thứ 2 trở đi tốn 2 lần gọi LLM thay vì 1 (viết lại câu hỏi + sinh câu trả lời) — chấp nhận được vì đổi lại retrieval chính xác hơn nhiều.

**Việc còn tồn đọng từ giai đoạn này:** Không có — đã xác nhận hoạt động đúng qua nhiều lần test nhất quán (`curl` và trình duyệt cho cùng kết quả).

## Giai đoạn 11 — Evaluation (đo chất lượng retrieval bằng số liệu)

**Mục tiêu giai đoạn:** Đo khách quan xem hybrid search có thực sự tốt hơn dense-only hay không, bằng con số, thay vì chỉ dựa vào cảm giác qua vài lần test tay.

**Đã làm:**

- Viết `evaluate.py` — xây "golden set" nhỏ (câu hỏi kèm đáp án `article_id` đúng, dựa trên các case đã tự tay xác minh trước đó: "kapranos", "why are musicians protesting")
- Đo 3 chỉ số chuẩn Information Retrieval: **Recall@k** (có tìm ra đáp án đúng không), **MRR** (đáp án đúng đứng thứ mấy), **Precision@k** (trong k kết quả, bao nhiêu % thực sự đúng)
- So sánh trực tiếp dense-only vs hybrid trên cùng dữ liệu

**Kết quả đo được:**

- Recall@5 và MRR: cả 2 phương pháp bằng nhau tuyệt đối (1.00 / 1.000) — không phân biệt được
- Precision@5: "kapranos" → hybrid thắng rõ (1.0 vs 0.8, đúng dự đoán lý thuyết về tên riêng); "why are musicians protesting" → dense-only thắng (0.8 vs 0.6, ngược dự đoán) → trung bình cộng ra **bằng nhau (0.80 cả 2 bên)**

**Quyết định kỹ thuật:**

- **Vấn đề:** golden set chỉ có 2 câu hỏi có đáp án — không đủ để kết luận thống kê đáng tin
- **Đã chọn:** không ép kết luận "hybrid tốt hơn" khi số liệu không ủng hộ rõ ràng — ghi nhận trung thực rằng mỗi phương pháp có điểm mạnh riêng tuỳ loại câu hỏi
- **Vì sao:** kết luận trung thực dựa trên số liệu quan trọng hơn kết luận "đẹp" nhưng không có bằng chứng — đúng tinh thần evaluation thật sự

**Việc còn tồn đọng từ giai đoạn này:**

- Golden set cần mở rộng (10-20 câu, chia nhóm theo loại câu hỏi: tên riêng, paraphrase ngữ nghĩa, câu hỏi thường) để có kết luận đáng tin cậy hơn về việc hybrid tốt hơn ở loại câu hỏi nào cụ thể

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

**Việc còn tồn đọng từ giai đoạn này:**

- Chưa test thực tế xem `[1]`, `[2]` trong answer có khớp đúng với `sources` tương ứng hay không — cần chạy thử qua `/ask` và kiểm tra thủ công
- Chưa đo bằng golden set xem rerank có thực sự cải thiện Precision/MRR so với chỉ dùng RRF hay không — evaluate.py hiện tại KHÔNG bao gồm bước rerank (vì gọi thẳng `vectorstore.similarity_search()`, không qua `retrieve_chunks()`), nên 2 việc này đang tách biệt, cần làm riêng nếu muốn so sánh
- Chưa cập nhật frontend để hiển thị `[1]`, `[2]` dạng link bấm được tới đúng nguồn

**Bug gặp phải (bổ sung sau khi test thật):**

- **Bug 16 — Cross-encoder xếp hạng SAI trên dataset đã tiền xử lý:** test thực tế cho thấy 1 bài KHÔNG liên quan (Jerry Springer opera) được cross-encoder chấm điểm CAO HƠN các bài THỰC SỰ liên quan (visa, file-sharing) cho cùng 1 câu hỏi. Dùng `debug_rerank.py` (script debug riêng, in toàn bộ điểm 15 candidate) xác nhận: đây không phải do chọn sai ngưỡng lọc, mà do bản thân cross-encoder chấm điểm không đáng tin cậy trên dataset này.
- **Nguyên nhân xác định:** text lưu trong Qdrant đã bị tiền xử lý mất dấu câu, viết thường, mất cấu trúc câu tự nhiên (từ bước ingest ban đầu — xem Giai đoạn 3). Model `ms-marco-MiniLM` được train trên text tự nhiên có dấu câu — đưa text đã "băm nát" vào khiến khả năng đánh giá độ liên quan của nó bị nhiễu nặng.
- **Quyết định xử lý:** KHÔNG xoá code rerank — thêm cờ `ENABLE_RERANK` (mặc định `false`) trong `config.py`. Khi tắt, `retrieve_chunks()` dùng lại hybrid RRF thuần (đã kiểm chứng ổn ở Giai đoạn 11). Khi cần, chỉ cần set `ENABLE_RERANK=true` trong `.env` để bật lại, không cần sửa code.
- **Việc còn tồn đọng:** cần sửa `ingest.py` để lưu thêm text gốc (chưa tiền xử lý) làm field riêng, rồi thử lại rerank trên text đó — hiện chưa làm, đây là điều kiện để bật `ENABLE_RERANK=true` một cách đáng tin cậy trong tương lai
- Citation (`[1]`, `[2]`) không bị ảnh hưởng bởi vấn đề này — hoạt động độc lập với việc bật/tắt rerank, vì chỉ đánh số theo thứ tự chunks trả về


# Kiến thức đã áp dụng
- [[Qdrant]]
- [[Kaggle]]