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

## Giai đoạn 4 — Phase 1: File read/write tools

**Mục tiêu giai đoạn:** Thêm khả năng đọc file/thư mục thật (write file để sau), giới hạn trong 1 base_dir để chống path traversal — đúng bài học từ lỗ hổng CVE-2026-34070 đã học.

**Quyết định kỹ thuật:**

- **Vấn đề:** Phase 1 bắt đầu với tool nào, và có giới hạn thư mục ngay từ đầu không
- **Đã chọn:** chỉ `read_file` + `list_directory` trước (chưa ghi); BASE_DIR lấy từ `state["codebase_path"]` (dùng chung với search_codebase, không tạo thư mục `workspace/` riêng — agent chạy ngay trong thư mục project thật); thêm denylist chặn `.env`, `chroma_db`, `.git`
- **Vì sao:** đơn giản, an toàn tuyệt đối cho bước đầu; dùng chung `codebase_path` giữ nhất quán thiết kế với `search_codebase`; denylist tránh agent tự đọc secret của chính nó

**Đã làm:**

- Viết và test thành công `read_file` + `list_directory` trong `agent/file_tools.py` (test qua Python REPL, không cần file test riêng)
- Xác nhận path traversal bị chặn đúng (`../../etc/passwd` bị từ chối)
- Xác nhận denylist hoạt động đúng (`.env` không đọc được)
- Viết và test thành công `write_file` + `edit_file` — ghi file mới, sửa file bằng old_str/new_str duy nhất đều hoạt động đúng
- Xác nhận denylist áp dụng cả cho ghi (`.env` không ghi/sửa được)

**Việc còn tồn đọng:**

- Không còn — Phase 1 (read_file, list_directory, write_file, edit_file) đã hoàn thành, đã ghép vào tools.py + graph.py

## Giai đoạn 5 — Guardrails

**Mục tiêu giai đoạn:** Chống prompt injection (đánh dấu tool result là data qua tag `<tool_result>` + system prompt) và thêm human-in-the-loop approval (`interrupt()`) trước khi thực thi `write_file`/`edit_file`, đúng điều kiện bắt buộc trước khi mở Phase 2 (Docker sandbox).

**Đã làm:**

- Thêm `<tool_result>` tag bọc kết quả tool + system prompt chỉ rõ nội dung trong tag là DATA, không phải instruction
- Thêm `interrupt()` cho `write_file`/`edit_file`, compile graph với `InMemorySaver` checkpointer, `cli.py` xử lý vòng lặp resume qua `Command(resume=...)`
- Test thành công: agent dừng đúng lúc, hiện `[Approval needed]`, chỉ ghi file sau khi duyệt "yes"

**Bug gặp phải:**

- **Bug 4** — **TypedDict** dùng **=** thay vì **:**: `codebase_path = str | None` trong` state.py` (thiếu dấu :) khiến field không được đăng ký vào schema thật của **AgentState** → mọi giá trị `codebase_path` truyền vào `graph.invoke()` bị LangGraph âm thầm bỏ qua, không báo lỗi,` state.get("codebase_path")`luôn None → Cách sửa: đổi = thành : đúng cú pháp TypedDict
- **Bug 5 — Check `query` áp dụng nhầm cho mọi tool:** đoạn code `if not args.get("query", "").strip()` chạy vô điều kiện cho cả 6 tool thay vì chỉ `search_codebase`/`search_web` → `write_file`/`edit_file` (không có tham số `query`) luôn bị chặn nhầm với lỗi "Missing required 'query' parameter", khiến `interrupt()` và tool thật không bao giờ chạy tới, dù model gọi tool đúng hoàn toàn → Cách sửa: thêm điều kiện `if name in NEEDS_QUERY and ...` giới hạn đúng phạm vi 2 tool cần `query`
- **Bug 6 — Model sinh tool-call sai định dạng, gây crash + hallucination:** `openai/gpt-oss-20b` qua Groq thỉnh thoảng sinh output dạng XML giả (`<tool_call><function=...>`) thay vì đúng JSON tool_calls chuẩn, khiến Groq trả lỗi `400 tool_use_failed` ngay ở tầng API; lỗi không được catch khiến graph crash, và ở các lượt không crash, model tự bịa ra lời giải thích sai (hallucination) về nguyên nhân lỗi → Cách sửa: bọc `llm_with_tools.invoke()` trong try/except `groq.BadRequestError`, retry 1 lần (lỗi mang tính stochastic), fallback về câu trả lời báo lỗi rõ ràng nếu retry vẫn thất bại

## Giai đoạn 6 — Phase 2: Docker sandbox cho code execution

**Mục tiêu giai đoạn:** Thêm tool `execute_python` chạy code Python trong container Docker cô lập, đúng điều kiện Guardrails đã hoàn thành trước đó.

**Quyết định kỹ thuật:**

- **Vấn đề:** phạm vi sandbox (loại lệnh cho phép), có chặn mạng không, có cần interrupt() duyệt không
- **Đã chọn:** chỉ chạy Python (chưa cho shell command tùy ý); chặn mạng hoàn toàn (`network_disabled=True`); có interrupt() duyệt trước khi chạy, giống write_file/edit_file
- **Vì sao:** ban đầu định chọn mạng mở + không cần duyệt (tin sandbox đã đủ cô lập), nhưng sau khi chỉ ra rủi ro cụ thể (prompt injection từ read_file/search_web kết hợp mạng mở + không duyệt = có thể rò rỉ dữ liệu ra ngoài dù sandbox cô lập filesystem) đã đổi sang phương án an toàn nhất: chặn mạng VÀ có interrupt

**Đã làm:**

- Viết `agent/docker_tool.py` — `execute_python` dùng `docker.from_env()`, container chạy `detach=True` + `container.wait(timeout=10s)` + `mem_limit="128m"` + `nano_cpus=500_000_000`, không mount volume nào (không thấy được filesystem thật)
- Test thành công: code thường chạy đúng, gọi network bị chặn hoàn toàn, vòng lặp vô hạn bị timeout đúng 10s không treo agent
- Ghép vào `tools.py` (7 tool) và `NEEDS_APPROVAL` trong `graph.py`

**Bug gặp phải:**

- **Bug 7 — `containers.run()` không nhận tham số `timeout`:** tham số `timeout` chỉ áp dụng cho kết nối tới Docker daemon (`docker.from_env(timeout=...)`), không phải timeout cho quá trình chạy trong container → Cách sửa: đổi sang `detach=True` + tự gọi `container.wait(timeout=...)`, `kill()` nếu vượt quá, tự `container.remove(force=True)` trong `finally` (không dùng `remove=True` để tránh race condition mất log trước khi đọc được)
## Giai đoạn 7 — Phase 3: Hierarchical planning (planner/executor)

**Mục tiêu giai đoạn:** Tách "lập kế hoạch nhiều bước" khỏi "thực thi từng bước": `planner` sinh plan một lần (structured output qua Pydantic `Plan`), `executor` xử lý đúng một bước mỗi lượt và lặp trên cùng bước cho tới khi model trả text thuần.

**Quyết định kỹ thuật:**

- **Vấn đề:** Phase 3 bắt đầu với hướng nào: hierarchical planning, reflection/critic, hay replanning
- **Đã chọn:** hierarchical planning (planner/executor) trước; reflection/critic và replanning làm sau
- **Vì sao:** theo pattern Plan-and-Execute, dễ dự đoán hơn ReAct thuần và cho phép hiển thị plan trước khi thực thi

**Đã làm:**

- Thêm field `plan`, `current_step`, `tool_rounds` vào `AgentState`
- Node `planner`, `executor`, `advance_step`; quy tắc: có tool_calls → `act`, text thuần → bước xong → `advance_step`
- Trần `MAX_ROUNDS_PER_STEP = 5`, `MAX_STEPS = 12`
- Test: plan 2 bước đi tuần tự 1/2 → 2/2, `[Approval needed]` xuất hiện đúng một lần, bước 1 đọc đúng `agent/state.py`; câu không cần tool ("hello") cho plan 1 bước và trả lời bình thường

**Bug gặp phải:**

- **Bug 8 — `executor` dùng biến `idx` chưa khai báo:** NameError khi chạy → Nguyên nhân: đoạn code hướng dẫn rút gọn thân hàm (`...`) nên dòng khai báo bị thiếu → Cách sửa: thêm `idx = state["current_step"]` đầu hàm
- **Bug 9 — Graph kết thúc sớm (phát hiện khi review, trước khi chạy):** thiết kế đầu route "không có tool_calls" thẳng tới `END` và chỉ sang bước sau sau 1 lượt tool → bước không cần tool kết thúc cả graph, bước cần nhiều tool bị cắt dở → Cách sửa: executor lặp trên cùng bước, text thuần = bước xong → `advance_step` → `route_after_advance`
- **Bug 10 — Lặp thao tác, chạy rất lâu:** planner chia thừa một hành động thành hai bước (tạo file / ghi nội dung), executor không biết bước trước đã xong nên đọc/ghi lại, `write_file` bị đòi duyệt hai lần cùng nội dung → Cách sửa: planner prompt "fewest steps", executor prompt liệt kê "Already completed steps", thêm `tool_rounds` + `MAX_ROUNDS_PER_STEP`
- **Bug 11 — Bước bị cắt âm thầm khi chạm trần số lượt:** `MAX_ROUNDS_PER_STEP = 3` quá thấp, bước "Read state.py" cần 4 lượt (đọc sai path, list `.`, list `agent`, đọc đúng path) → bị ép sang bước 2 khi chưa đọc được file, agent vẫn báo Done với summary mơ hồ → Cách sửa: nâng lên 5, in cảnh báo khi chạm trần. Bằng chứng đã sửa: bước 1 hoàn thành sau 4 lượt và summary nhắc đúng `tool_rounds` (chỉ có nếu file thật sự được đọc)
# Kiến thức đã áp dụng
- [[]]