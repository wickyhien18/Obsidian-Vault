---
Hub: "[[MOC - AI & ML]]"
created: 2026-09-09
status: active
repo: https://github.com/wickyhien18/ai-agent-secretary
---

# Ý tưởng dự án

AI Agent đa năng, KHÔNG gắn với codebase Pharmacy Wicky cụ thể: 1 agent duy nhất, nhiều tool — (1) đọc/trả lời về bất kỳ codebase nào được chỉ định, (2) research agent tự search web nhiều bước + tổng hợp có trích dẫn. Orchestrate bằng LangGraph, CLI-based, repo độc lập.

# Tech Stack

|Thành phần|Công nghệ|Lý do chọn|
|---|---|---|
|Orchestration|LangGraph (ReAct/Plan-and-Execute)|Đáp ứng JD: tool-use, multi-step reasoning, memory|
|LLM API|Groq (free tier, rate-limited) — model `openai/gpt-oss-20b`|Máy không có GPU rời, ưu tiên free; đã test và xác nhận model chạy được|
|Vector DB|Chroma|Không cần Docker, setup nhanh hơn pgvector cho việc lưu embedding code|
|Search API|Tavily (free tier)|Chất lượng kết quả cao hơn DuckDuckGo, tối ưu sẵn cho AI agent, phù hợp research agent cần tổng hợp có trích dẫn|

# Nhật ký kỹ thuật

## Giai đoạn 1 — Setup môi trường

**Mục tiêu giai đoạn:** Dựng repo, venv, cài thư viện cốt lõi (langgraph, langchain-core, langchain-groq), xác nhận kết nối LLM chạy được trước khi học/code phần orchestration.

**Đã làm:**

- Test kết nối `test_connection.py` thành công với model `openai/gpt-oss-20b` qua Groq (đổi từ `llama-3.1-8b-instant`)

**Bug gặp phải:**

- **Bug 1 — Model tự nhận sai identity:** Khi hỏi "bạn là ai", `openai/gpt-oss-20b` trả lời tự nhận là ChatGPT do OpenAI phát triển → Nguyên nhân: đây là open-weight model, hiện tượng phổ biến do model được train/distill trên dữ liệu sinh ra từ ChatGPT nên "thừa hưởng" cách tự giới thiệu đó, không phải đang gọi nhầm sang API của OpenAI → Không ảnh hưởng tool-calling/reasoning; có thể sửa bằng system prompt nếu cần agent tự xưng đúng danh tính trong sản phẩm thực tế

**Việc còn tồn đọng từ giai đoạn này:** Không có — giai đoạn 1 hoàn thành.

## Giai đoạn 2 — Học lý thuyết StateGraph + Tool Calling, viết state.py

**Mục tiêu giai đoạn:** Hiểu Node/Edge/Conditional Edge và Tool Calling/Function Calling trong LangGraph, áp dụng vào bài toán agent đa năng (đọc codebase + search web), viết schema state đầu tiên.

**Đã làm:**

- Học lý thuyết StateGraph (Node, Edge, Conditional Edge, cycle) áp dụng vào kiến trúc agent 2-tool
- Học lý thuyết Tool Calling/Function Calling (`@tool`, `bind_tools`, `tool_calls`, `ToolMessage`)
- Hoàn thành bài tập `get_weather`/`get_time` với `bind_tools`: model chọn đúng tool `get_weather` cho câu hỏi về thời tiết

**Bug gặp phải:**

- **Bug 2 — Model tự bịa giá trị tham số khi thiếu thông tin:** Hỏi "What weather today?" (không nêu thành phố) nhưng model tự điền `city: 'San Francisco'` → Nguyên nhân: tham số `city` được khai báo `required` trong schema, model bắt buộc phải điền giá trị nào đó để JSON hợp lệ nên tự đoán 1 city plausible → Cách xử lý: cần thiết kế system prompt yêu cầu model hỏi lại nếu thiếu tham số bắt buộc, hoặc validate ở code trước khi thực thi tool

**Việc còn tồn đọng từ giai đoạn này:**

- Viết `state.py` thật cho agent (2 tool: search_codebase, search_web)
- Quyết định cách xử lý tham số thiếu (hỏi lại người dùng vs default logic) trước khi lắp vào graph thật

## Giai đoạn 3 — Roadmap mở rộng thành Claude Code/Codex-like agent

**Mục tiêu giai đoạn:** Định hướng lại dự án theo hướng agent đa năng như Claude Code/Codex (đọc/ghi file, chạy code trong sandbox), phát triển tăng dần qua các phase, không nhảy cóc.

**Quyết định kỹ thuật:**

- **Vấn đề:** phạm vi "giống Claude Code/Codex" rất rộng, cần chia nhỏ theo mức độ rủi ro
- **Đã chọn:** roadmap 5 phase — Phase 0 (đọc code + search web, read-only, đã code xong khung với stub) → Phase 1 (đọc/ghi file thật, giới hạn trong 1 thư mục) → Phase 2 (chạy code/lệnh trong Docker sandbox cô lập) → Phase 3 (multi-step planning phức tạp hơn) → Phase 4+ (git integration, chạy test tự động)
- **Vì sao:** độ rủi ro tăng dần theo từng phase (từ không thể phá hỏng gì → có thể ghi sai file → có thể chạy code độc hại nếu thiếu sandbox), nên thứ tự này giảm thiểu khả năng agent gây hại trong lúc học/code
- **Xác nhận (Sep 2026):** hoàn thiện Phase 0 thật (thay stub bằng Chroma + Tavily) trước khi bắt đầu Phase 1

**Việc còn tồn đọng:**

- Không còn — Phase 0 (search_codebase + search_web thật) đã hoàn thành

**Bug gặp phải:**

- **Bug 3 — FastEmbed không cài được trên Python 3.14:** `pip install fastembed` chạy được nhưng import vẫn lỗi do `onnxruntime` (dependency của fastembed) chưa có wheel tương thích Python 3.14 → Cách xử lý: chuyển sang `DefaultEmbeddingFunction` của Chroma (model `all-MiniLM-L6-v2`, không cần cài thêm gì) — kết quả test xác nhận semantic search vẫn hoạt động đúng (query tiếng Việt không trùng từ khóa vẫn tìm đúng code liên quan, xếp hạng đúng thứ tự)

**Dịch vụ/API key bên thứ 3 đã dùng:**

- Groq API (LLM inference) — free tier, key trong `.env`
- Tavily API (web search) — free tier, 1,000 credit/tháng, key trong `.env`, đã test thành công với query tiếng Anh, trả về URL + content sạch

**Lưu ý bảo mật quan trọng:**

- Phát hiện qua chính kết quả test `search_web`: đợt công bố lỗ hổng "LangDrained" (Cyera Research, 3/2026) — CVE-2026-34070 ảnh hưởng `load_prompt()`/`load_prompt_from_config()` trong `langchain-core` các bản trước 1.2.22 (path traversal qua config bị đầu độc) → Cần kiểm tra version `langchain-core` đang dùng, upgrade nếu < 1.2.22, trước khi làm Phase 1 (file read/write)
# Kiến thức đã áp dụng
- [[]]