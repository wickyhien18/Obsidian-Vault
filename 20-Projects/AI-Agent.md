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

# Kiến thức đã áp dụng
- [[]]