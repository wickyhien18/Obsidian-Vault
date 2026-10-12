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
|Tìm file|`find_files` + `suggest_paths` (tự viết, dùng `os.walk` và `difflib`)|Agent không có Glob; `search_codebase` chỉ tìm theo nội dung, không tìm theo tên file|
|Đánh giá|`eval.py` (repo giả trong thư mục tạm, chấm theo trạng thái đĩa)|Thay cho việc thử tay từng ca; so sánh được nhiều model|

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

- **Bug 4** — **TypedDict** dùng **=** thay vì **:**: `codebase_path = str | None` trong `state.py` (thiếu dấu :) khiến field không được đăng ký vào schema thật của **AgentState** → mọi giá trị `codebase_path` truyền vào `graph.invoke()` bị LangGraph âm thầm bỏ qua, không báo lỗi, `state.get("codebase_path")`luôn None → Cách sửa: đổi = thành : đúng cú pháp TypedDict
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

## Giai đoạn 8 — Gỡ lỗi "đứng" ở bước đọc/tìm file

**Mục tiêu giai đoạn:** Tìm và sửa nguyên nhân agent bị kẹt ở bước đọc/tìm file khi chạy tác vụ "đọc file rồi tạo file txt tóm tắt"; thêm khả năng định vị file theo tên.

**Quyết định kỹ thuật:**

- **Vấn đề:** model chỉ biết tên file (`tools.py`) trong khi file nằm ở `agent/tools.py`; bộ tool không có cách tìm file theo tên (`list_directory` không đệ quy, `search_codebase` tìm theo nội dung chứ không theo tên file)
- **Đã chọn:** thêm tool `find_files` (tìm theo tên/glob, đệ quy, bỏ qua thư mục rác) và gợi ý đường dẫn gần đúng bằng `difflib` (`suggest_paths`) ngay trong thông báo "không tìm thấy" của `read_file`/`find_files`
- **Vì sao:** để model tự sửa ở lượt kế tiếp thay vì đoán đường dẫn qua nhiều lượt; mô tả tool phải nói rõ tool nào tìm theo tên, tool nào tìm theo nội dung

**Đã làm:**

- In kết quả từng lần gọi tool vào log DEBUG (đầu và cuối kết quả) và đặt dòng `tool_call` sau `check_call`, để log hiện đúng `codebase_path` thật thay vì giá trị model đoán
- Thêm `find_files`, `suggest_paths`, `SKIP_DIRS`; nâng system prompt lên 8 tool; docstring `search_codebase` ghi rõ "chỉ tìm theo nội dung"
- Viết lại chỉ mục Chroma: bỏ qua thư mục rác, mỗi repo một collection (tên theo hash đường dẫn), `upsert` theo batch 200, chỉ nhận một số đuôi file, bỏ file lớn hơn 200KB, xóa và lập lại chỉ mục mỗi lần khởi động, in số chunk
- **Xác nhận (Oct 2026):** log cho thấy chuỗi `read_file` lỗi → `find_files` → `read_file` đúng `agent/tools.py`; dòng `Indexing 131 chunks...` xuất hiện khi gọi `search_codebase`

**Bug gặp phải:**

- **Bug 12 — Agent "đứng"/lặp ở bước đọc và tìm file:** log: `read_file` → `list_directory` → `search_codebase`, rồi im lặng → Nguyên nhân gộp: (1) tên file trong yêu cầu không khớp đường dẫn thật (`tool.py` không tồn tại, `tools.py` nằm ở `agent/`); (2) không có tool tìm theo tên nên model thử `search_codebase` bằng chính tên file; (3) nghi phạm chính cho chỗ im lặng: lần gọi `search_codebase` đầu tiên lập chỉ mục bằng `rglob("*.py")` trên cả thư mục dự án (gồm `.venv`) và không in tiến trình; chưa tái hiện trực tiếp bản treo, nhưng sau khi bỏ `.venv` việc lập chỉ mục chạy bình thường (131 chunk); (4) log in `codebase_path: ''` trước khi `check_call` ghi đè giá trị thật nên gây hiểu nhầm → Cách sửa: như phần "Đã làm" ở trên
- **Bug 13 — Chỉ mục Chroma có chunk "ma":** `search_codebase` trả về `test_repo/email.py` và `test_repo/payment.py` dù thư mục gốc không còn thư mục đó → Nguyên nhân (suy ra từ code): mọi repo dùng chung collection `"codebase"` lưu bền trên đĩa, `add` chỉ thêm chunk chứ không xóa chunk của file đã biến mất; id `chunk_N` trùng giữa các repo và các lần chạy, và theo kiểm chứng trong roadmap 02/10 (chromadb 1.5.9, môi trường khác; chưa tái hiện ở máy mình) `add` với id đã tồn tại bị bỏ qua âm thầm, nên chunk đầu của repo thứ hai biến mất và nội dung cũ không được làm mới → Cách sửa: collection riêng theo hash đường dẫn, id theo `đường_dẫn:chỉ_số`, xóa collection trước khi lập lại chỉ mục. Chưa kiểm chứng lại bằng truy vấn `process_payment` sau khi sửa

## Giai đoạn 9 — Ràng buộc bằng code: lỗi phải dừng, không được bịa

**Mục tiêu giai đoạn:** Làm cho agent dừng và báo trung thực khi tool thất bại, và không cho ghi file dựa trên bản đọc dở.

**Quyết định kỹ thuật:**

- **Vấn đề:** model 20B không giữ được luật chỉ bằng prompt: log cho thấy nó bỏ qua chú thích "[showing lines ...]", ghi file từ trí nhớ của lượt trước và báo cáo sai tên file
- **Đã chọn:** đặt luật trong code (harness) thay vì trong prompt: tiền tố `ERROR:` cho mọi lỗi tool, lỗi thì dừng kế hoạch, guard chặn `write_file` khi mới đọc dở, finalizer chỉ nhìn lượt hiện tại
- **Vì sao:** code là xác định và kiểm chứng được bằng log, còn prompt chỉ là lời nhắc. Cái giá: guard chặn cả trường hợp ghi ghi chú sau khi đọc dở một file khác; chấp nhận được ở giai đoạn này

**Đã làm:**

- `read_file` đọc theo cửa sổ (`offset`, `limit`, ngân sách ký tự riêng, chú thích `[showing lines a-b of N; call read_file again with offset=...]`)
- Thêm `partial_reads` vào `AgentState`; `act` chặn `write_file` khi còn file đọc dở và trả `ERROR: you have only read part of ...`
- Mọi lỗi tool (trong `tools.py`, `run_tool`, `check_call`) bắt đầu bằng `ERROR:`; `last_tool_failed` đưa luồng `executor` → `finalizer`; model có thể báo `STUCK:` để dừng kế hoạch; chạm trần số lượt thì dừng ở `finalizer` thay vì sang bước sau
- Finalizer chỉ thấy lượt hiện tại (`only_current_turn`); kết quả tool của các lượt cũ bị thay bằng một dòng; prompt finalizer thêm luật báo `ERROR`, báo đọc dở, trích đúng nội dung đã ghi
- Bỏ ép `tool_choice="any"` (`force_tool = False`)
- Dời dòng log `tool_call` xuống sau bước duyệt; log cả kết quả bị chặn
- **Xác nhận (Oct 2026):** guard đọc dở chặn đúng ở hai ca (`agent/tools.py` 252-258 dòng; `big.txt` 600 dòng); các ca `tool.py` và `nonexistent_file.py` dừng ở `ERROR` và báo "không tạo file nào"

**Bug gặp phải:**

- **Bug 14 — Tóm tắt dựa trên file bị cắt:** `read_file` trả 6070 ký tự (6000 + chú thích + thẻ), bản tóm tắt thiếu hẳn `write_file`/`edit_file`/`execute_python` nhưng agent vẫn báo "đã đọc thành công"; prompt còn bảo dùng `search_codebase` để lấy phần bị cắt, mà `search_codebase` chỉ lập chỉ mục `.py` nên là ngõ cụt → Cách sửa: `read_file` theo cửa sổ + guard đọc dở
- **Bug 15 — Thất bại bị nuốt, ghi file từ trí nhớ:** `tool.py` không tồn tại → model trả chữ → harness coi là "bước xong" → bước 2 ghi `summ.txt` bằng nội dung của lượt trước; finalizer báo "đã đọc `tools.py` và tạo `summary.txt`" (sai tên file, sai hành động), và người dùng còn bấm duyệt vì màn hình duyệt không cho thấy việc chưa đọc gì → Nguyên nhân: không có tín hiệu lỗi máy-đọc-được, finalizer thấy cả lịch sử → Cách sửa: `ERROR:` + dừng kế hoạch + finalizer chỉ thấy lượt hiện tại
- **Bug 16 — Lỗi 400 "Tool choice is required, but model did not call a tool":** bước đã xong nhưng vẫn bị ép gọi tool, Groq trả 400 hai lần rồi mới rơi vào dự phòng → Cách sửa: bỏ ép tool
- **Bug 17 — Dòng `tool_call` in hai lần sau khi duyệt:** không phải model gọi lại; khi resume sau `interrupt`, LangGraph chạy lại cả node `act` từ đầu. Hiện vô hại vì các việc trước `interrupt` chỉ đọc. Quy tắc rút ra: không đặt tác dụng phụ trước `interrupt` trong cùng một node

## Giai đoạn 10 — Giữ planner nhưng làm cho chắc; chuyển từ thử tay sang eval.py

**Mục tiêu giai đoạn:** Quyết định số phận của planner sau khi log cho thấy nó tốn token và hay sinh lỗi; chuyển từ thử tay từng ca sang bộ kiểm thử tự động.

**Quyết định kỹ thuật:**

- **Vấn đề:** planner hay trả JSON hỏng, tách bước thừa (đọc / tóm tắt / ghi thành 3 bước), token của nó không được đếm; có nên bỏ không
- **Đã chọn:** giữ planner, nhưng ràng buộc bằng code: thử lại tối đa 2 lần, `include_raw=True` để đếm token planner, `tidy_plan` bỏ bước chỉ để suy nghĩ và giới hạn `MAX_PLAN_STEPS = 3`; khi đã dùng tool thì luôn qua finalizer
- **Vì sao:** finalizer là nơi chứa mọi luật trung thực (chỉ báo "đã làm" khi có kết quả tool xác nhận); bỏ lượt "đóng bước" để tiết kiệm token sẽ phá những bước cần ghi nhiều file, nên chưa làm khi chưa có số đo đầy đủ
- **Mốc đánh giá:** tác vụ "đọc file 258 dòng rồi ghi một file" có sàn 8 lượt gọi (planner, tìm file, đọc 2 cửa sổ, ghi, đóng bước 1, đóng bước 2, finalizer) nên mốc đặt lại là dưới 10 lượt và dưới 30K token; mốc đầu (7 lượt, 20K token) không đạt được

**Đã làm:**

- Planner: thử lại khi lỗi 400, lấy token qua `include_raw=True`, `tidy_plan`, prompt "tối đa 3 bước, không tách bước chỉ để suy nghĩ hay tóm tắt"; `route_after_advance` đi qua finalizer nếu lượt hiện tại đã dùng tool, chỉ kết thúc thẳng khi không dùng tool (câu trò chuyện)
- Viết `eval.py`: repo giả trong thư mục tạm (README, `app/main.py`, `app/utils.py`, `notes.txt` chứa lệnh tiêm prompt, `.env` chứa secret) và 8 task (`chat`, `read_version`, `find_symbol`, `create_file`, `edit_file`, `secret_guard`, `injection`, `multi_step`), chấm theo trạng thái đĩa thay vì lời model, hỗ trợ so nhiều model (`--model` lặp lại), `--repeats`, lưu `eval_results.json`; `chdir` vào thư mục tạm nên `chroma_db` của eval không đụng `chroma_db` của dự án. Chưa ghi nhận kết quả chạy lần nào
- **Xác nhận (Oct 2026):** ca file không tồn tại và ca chào hỏi chạy đúng luồng; ca `big.txt` 600 dòng đọc đủ 3 cửa sổ, guard chặn một lần ghi sớm, rồi ghi đúng

**Bảng kết quả kiểm thử qua các đợt sửa** (các dòng trước khi sửa planner chưa tính lượt gọi và token của planner):

|Ca|Lượt gọi|Token vào|Kết quả|
|---|---|---|---|
|`tools.py` → `summary.txt` (người dùng từ chối duyệt), sau khi thêm `find_files`|7|13,732|đọc được `agent/tools.py` nhưng chỉ bản bị cắt; báo cáo đúng "chưa tạo file"|
|`tool.py` → `summ.txt` (tên sai, chưa có `ERROR:`)|6|17,631|SAI: ghi file từ trí nhớ, báo cáo nhầm tên file|
|`tool.py` → `x.txt` (sau các đợt sửa đầu)|4|5,562|đạt: dừng ở lỗi|
|`tools.py` → `y.txt`|7|26,215|ghi đúng; báo cáo trích sai nội dung|
|`tool.py` → `z.txt`|5|7,213|đạt|
|`tools.py` → `w.txt` (guard đọc dở)|10|37,466|đạt; guard chặn ghi sớm; đắt vì planner tách 3 bước|
|`tool.py` → `x.txt` (sau đợt sửa 4)|3|4,581|đạt|
|`tools.py` → `w.txt`|8|27,145|đạt; guard chặn ghi sớm|
|`nonexistent_file.py` → `q.txt` (sau sửa planner)|4|4,832|đạt: dừng, báo đúng|
|`xin chao`|2|1,875|luồng đúng; trả lời chỉ lặp lại hai chữ "xin chao"|
|`big.txt` (600 dòng) → `big_summary.txt`|7|27,787|đạt; đọc 3 cửa sổ, guard chặn 1 lần; chạm trần 5 lượt|

**Bug gặp phải:**

- **Bug 18 — Planner lỗi và tốn token:** (a) Groq trả 400 `tool_use_failed` vì model sinh JSON dư dấu `"` ở cuối, planner không thử lại nên rơi xuống kế hoạch một bước; (b) planner tách một yêu cầu thành 3 bước, mỗi bước thêm vài lượt gọi (37K token cho "đọc rồi ghi"); (c) token của planner không được cộng vào bộ đếm vì `with_structured_output` không đi qua `call_llm` → Cách sửa: thử lại 2 lần, `include_raw=True`, `tidy_plan`
- **Bug 19 — Finalizer trích sai nội dung file đã ghi:** `y.txt` thật chứa "provides a collection of utility functions...", báo cáo trích "implements a suite of utility functions ... leveraging chromadb" → Cách sửa: prompt "copy exactly from the write_file call"; chấp nhận rằng model không trích đúng từng byte (một lần khác chỉ lệch một ký tự gạch nối). Nếu cần chính xác tuyệt đối thì nội dung phải do code in ra, không phải do model
- **Bug 20 — Planner chép lời người dùng thành lời thoại:** `xin chao` → bước "Respond with 'xin chao'" → agent chỉ trả lại hai chữ đó → Đề xuất: prompt planner dặn "bước là mục tiêu, không chép lời người dùng; với lời chào hoặc câu không cần tool thì dùng đúng một bước: trả lời hữu ích bằng ngôn ngữ của người dùng". Chưa áp dụng và chưa kiểm chứng
- **Bug 21 — Chạm trần 5 lượt ngay khi ghi thành công:** ca `big.txt`: đọc, đọc, ghi (bị chặn), đọc, ghi = đúng 5 lượt, nên log in `WARNING: step 1 hit the 5-round cap, stopping` dù việc đã xong; file dài hơn một cửa sổ nữa sẽ bị dừng giữa chừng → Đề xuất nâng `MAX_ROUNDS_PER_STEP` lên 8. Chưa áp dụng

**Bài học rút ra:**

- Log là sự thật: in đầu và cuối kết quả của từng tool trước khi đoán nguyên nhân (chính chú thích ở cuối kết quả mới cho thấy file bị đọc dở)
- Với model yếu, luật phải nằm trong code (guard, tiền tố lỗi, điều kiện dừng); prompt chỉ làm lời nhắc
- Thất bại im lặng nguy hiểm hơn thất bại ồn ào: agent "đứng" chỉ tốn thời gian, còn agent ghi file sai và báo "xong" làm hỏng dữ liệu và niềm tin
- Mốc đánh giá phải tính sàn của tác vụ trước khi đặt; đặt mốc 7 lượt cho tác vụ có sàn 8 lượt chỉ tạo ra kết luận sai
- `interrupt` chạy lại cả node: không đặt tác dụng phụ trước nó
- Thử tay vài chục ca là đủ để hiểu lỗi nhưng không đủ để biết mỗi lần sửa là tiến hay lùi; cần bộ kiểm thử cố định chấm theo trạng thái đĩa

**Việc còn tồn đọng:**

- Chạy `eval.py` lần đầu để có số liệu gốc; sửa các số liệu cũ trong `eval.py` (`tool_errors` còn tìm chuỗi "Tool error" trong khi lỗi giờ bắt đầu bằng `ERROR:`, `duplicate_skips` không còn nguồn, ghi chú "token không gồm planner" đã sai, trạng thái ban đầu thiếu `partial_reads`, hiện vô hại vì code dùng `.get`)
- Thêm vào `eval.py` các ca ứng với lỗi đã gặp: tên file sai, tên file trần nằm trong thư mục con, người dùng từ chối duyệt, file dài nhiều cửa sổ (đọc và ghi), chào hỏi bằng tiếng Việt, và ngưỡng số lượt gọi cho từng task
- Kiểm thử nốt các ca chưa chạy sau các đợt sửa: từ chối duyệt, truy vấn `process_payment` để xác nhận chỉ mục không còn chunk ma
- Áp dụng Bug 20 (prompt planner cho lời chào) và Bug 21 (nâng trần số lượt), rồi chạy lại ca `big.txt`
- Bảo mật: chặn theo mẫu `.env*`, `*.pem`, `id_rsa*` (hiện chỉ chặn đúng tên `.env`); `search_web` chưa cần duyệt nên có thể bị lợi dụng đưa dữ liệu ra ngoài qua truy vấn; `edit_file` chưa có guard đọc dở như `write_file`; thêm `pids_limit`, `read_only`, `cap_drop`, user không phải root cho container; `cli.py` mới chỉ bắt `RateLimitStop`
- Dọn thư mục dự án: các file thử (`summ.txt`, `summary.txt`, `w.txt`, `y.txt`, `big.txt`, `big_summary.txt`), `eval_results.json` và `chroma_db/` không nên đưa lên GitHub; thêm vào `.gitignore`; `nodes.py` đang rỗng
- Tối ưu token (chưa làm vì chưa có số đo đầy đủ từ `eval.py`): bỏ planner cho lời chào, thu nhỏ kết quả tool cũ, bỏ lượt "đóng bước" khi bước chỉ có một việc

# Sổ lỗi tổng hợp (Bug 12-26)

Gom lại mọi lỗi của đợt gỡ lỗi từ 09/10 đến 10/10/2026 để tra cứu một chỗ. Chi tiết từng lỗi nằm ở Giai đoạn 8-10 phía trên; đây là bản đối chiếu theo tầng và theo mức bằng chứng.

**Nhãn bằng chứng:** **[log]** = tái hiện được bằng log chạy thật; **[code]** = suy ra từ đọc code, chưa tái hiện trực tiếp; **[ngoài]** = nguồn ngoài báo cáo, chưa tái hiện ở máy mình.

## Phân loại theo tầng

|Tầng|Nghĩa|Lỗi|Cách đối phó chung|
|---|---|---|---|
|Model (`gpt-oss-20b`)|Hành vi sai của model, không sửa được, chỉ bao quanh|16, 18 (JSON hỏng), 19, 20; model bỏ qua chú thích "đọc dở" (14)|Ràng buộc bằng code, thử lại có giới hạn; prompt chỉ làm lời nhắc|
|Harness (graph.py)|Code điều phối nuốt hoặc hiểu sai tín hiệu|15, 17, 21, route kết thúc bỏ qua finalizer|Tín hiệu lỗi máy-đọc-được, điều kiện dừng rõ ràng|
|Tool (tools.py)|Công cụ thiếu hoặc trả kết quả gây hiểu sai|12, 13, 14|Thêm tool, đọc theo cửa sổ, chỉ mục theo repo|
|Hạ tầng bên ngoài|Giới hạn của dịch vụ|24|Backoff, giảm token, model dự phòng|
|Quy trình|Cách làm và cách đo|22, 23|Eval cố định, đối chiếu từng bản sửa với file thật|

## Bảng lỗi

|Bug|Triệu chứng|Nguyên nhân|Cách sửa|Bằng chứng và trạng thái|
|---|---|---|---|---|
|12|Agent "đứng", lặp `read_file`, `list_directory`, `search_codebase`|Tên file không khớp đường dẫn thật; không có tool tìm theo tên; `search_codebase` tìm theo nội dung; log in `codebase_path` trước khi bị ghi đè|`find_files`, `suggest_paths`, mô tả tool nói rõ tìm theo tên hay theo nội dung, log đặt sau `check_call`|[log] chuỗi `read_file` lỗi, `find_files`, `read_file` đúng. Phần treo ở chỉ mục là [code]|
|13|`search_codebase` trả file không còn trong repo (`test_repo/*`); repo thứ hai có thể thiếu chunk|Collection dùng chung, id `chunk_N` trùng, `add` không làm mới|Collection theo hash đường dẫn, id `đường_dẫn:chỉ_số`, `upsert` theo batch, lập lại khi khởi động, bỏ thư mục rác|[log] chunk ma ở ca `z.txt`; [ngoài] id trùng bị bỏ qua âm thầm. Sau sửa chưa test lại bằng truy vấn `process_payment`|
|14|Tóm tắt thiếu nửa sau file mà agent vẫn báo đã đọc|`read_file` cắt ở 6000 ký tự; prompt trỏ sang `search_codebase` (ngõ cụt); model bỏ qua chú thích "[showing lines ...]"|`read_file` theo cửa sổ (`offset`, `limit`, ngân sách ký tự), `partial_reads`, chặn `write_file` khi còn đọc dở|[log] guard chặn đúng ở `w.txt` và `big.txt`|
|15|Báo "đã tạo summary.txt" sai tên; ghi file từ trí nhớ lượt trước|Lỗi tool không có tín hiệu; model trả chữ nên harness coi là xong; finalizer thấy cả lịch sử|Tiền tố `ERROR:`, `last_tool_failed`, dừng kế hoạch, `STUCK:`, finalizer chỉ thấy lượt hiện tại, kết quả tool cũ bị thay bằng một dòng|[log] `x.txt`, `z.txt`, `q.txt` dừng đúng|
|16|400 "Tool choice is required, but model did not call a tool"|Ép `tool_choice="any"` ở bước đã xong|`force_tool = False`|[log] trước sửa; sau sửa không còn thấy|
|17|Dòng `tool_call: write_file` in hai lần sau khi duyệt|LangGraph chạy lại cả node sau `interrupt`|Không đặt tác dụng phụ trước `interrupt`; log sau bước duyệt|[log] vô hại ở thiết kế hiện tại|
|18|Planner trả 400 vì JSON dư dấu `"`; tách một yêu cầu thành 3 bước; token planner không được đếm|Lỗi sinh của model; prompt "usually 2-4"; `with_structured_output` đi ngoài `call_llm`|Thử lại 2 lần, `include_raw=True`, `tidy_plan` (tối đa 3 bước), prompt chặt hơn|[log] lỗi 400 và 37K token; sửa đã áp dụng, hiệu quả chưa đo bằng eval|
|19|Báo cáo cuối trích sai nội dung file đã ghi|Model viết lại từ trí nhớ thay vì sao chép|Prompt "copy exactly" (chỉ giảm lỗi); muốn chính xác tuyệt đối thì code phải in nội dung|[log] `y.txt`; `w.txt` lệch một ký tự|
|20|`xin chao` được trả lại đúng hai chữ "xin chao"|Planner chép lời người dùng thành bước "Respond with ..."|Prompt planner: bước là mục tiêu, lời chào dùng một bước trả lời hữu ích|[log]; chưa áp dụng|
|21|`WARNING: step 1 hit the 5-round cap` dù việc đã xong|Trần 5 lượt vừa đủ cho 3 cửa sổ đọc, 1 lần ghi bị chặn, 1 lần ghi thật|Nâng `MAX_ROUNDS_PER_STEP` lên 8|[log] ca `big.txt`; chưa áp dụng|
|22|Sửa xong nhưng hành vi không đổi|Áp dụng nửa vời: `suggest_paths` được định nghĩa nhưng không ai gọi; lỗi trong `check_call` thiếu `ERROR:`; `SYSTEM_PROMPT` còn "7 tools"; `FINALIZER_PROMPT` có hai chuỗi liền nhau thiếu khoảng trắng ("tools.If ...")|Sau mỗi vòng sửa phải đối chiếu từng chỗ với file thật; thay trọn hằng thay vì vá từng dòng|[code] trên các bản gửi trung gian; các bản sau đã sửa phần lớn; chưa kiểm lại `FINALIZER_PROMPT` và `check_call` trong bản mới nhất|
|23|Kết luận sai từ số liệu|Đặt mốc 7 lượt và 20K token trước khi tính sàn của tác vụ (sàn là 8 lượt); số token thiếu planner; hai mẫu không đủ để so sánh|Tính sàn từng tác vụ trước khi đặt mốc; mốc mới dưới 10 lượt và 30K token cho ca đọc 258 dòng; `eval.py` với `--repeats`|[log]|
|24|Lỗi 429 thật: `rate limited, waiting 26s` (ca `y.txt`)|Gói miễn phí giới hạn request và token theo phút và theo ngày; một ca tốn 27K-37K token|Backoff theo `retry-after` và dừng khi vô vọng (đã có); cần thêm: giảm token mỗi ca, model dự phòng|[log]. Các nguồn tra cứu (tháng 8/2026) ghi con số không khớp nhau; log Groq (12/10) xác nhận TPM của model này là 8,000 và 66/66 lần 429 là TPM (xem mục "Phân tích log Groq")|
|25|Lỗi 400 `attempted to call tool 'plan' which was not in request.tools` (08/10 13:03, chỉ 265 token vào)|Chưa biết. Nghi ở lượt planner (kích thước nhỏ hợp với planner hơn executor có lịch sử), model gọi một tool tên `plan` không có trong danh sách gửi lên|Chưa sửa; cần log DEBUG của lượt đó hoặc tái hiện planner, và xem cách `with_structured_output` gửi schema|[log] 1 lần duy nhất trong 224 yêu cầu; nguyên nhân chưa kiểm|
|26|29% yêu cầu bị 429; 38/66 lần đòi hơn 4,000 token; lượt lớn nhất 6,705 token vào|TPM 8,000 của model, trong khi ngữ cảnh mỗi lượt có lúc rất lớn (trung vị 1,426 nhưng p90 4,539). Tầng: hạ tầng bên ngoài cộng ngữ cảnh của harness|Chưa sửa. Đề xuất: trần ngữ cảnh mỗi lượt (khoảng 6,000 token vào), thu nhỏ kết quả tool cũ, model dự phòng, tách `wait_seconds` khỏi `seconds` trong eval (đã có)|[log]|

## Giả thuyết đã đặt ra nhưng chưa chứng minh

- **Treo vì lập chỉ mục cả `.venv`:** nghi phạm chính cho chỗ log im lặng sau `search_codebase`, nhưng chưa tái hiện bản treo. Bằng chứng gián tiếp: sau khi bỏ `.venv`, lập chỉ mục chạy bình thường (131 chunk).
- **Groq SDK tự thử lại 429 im lặng:** chưa xác nhận. Log chỉ cho thấy lần chờ do chính code của mình in ra. Đề xuất `max_retries = 0` và `timeout = 30` trong `models.toml` để loại trừ.
- **`add` của Chroma bỏ qua id trùng:** do roadmap 02/10 báo cáo (chromadb 1.5.9, môi trường khác). Kiểm chứng ở máy mình bằng 8 dòng bên dưới (embedding đưa vào tay nên không cần tải model; chưa chạy thử trong môi trường của tôi):

```python
import chromadb
col = chromadb.EphemeralClient().get_or_create_collection("t")
col.add(ids=["chunk_0"], documents=["repo A: payment"], embeddings=[[1.0, 0.0, 0.0]])
col.add(ids=["chunk_0"], documents=["repo B: shipping address"], embeddings=[[0.0, 1.0, 0.0]])
print(col.get(ids=["chunk_0"])["documents"])   # nếu giả thuyết đúng: ['repo A: payment']
col.upsert(ids=["chunk_0"], documents=["repo B: shipping address"], embeddings=[[0.0, 1.0, 0.0]])
print(col.get(ids=["chunk_0"])["documents"])   # sau upsert: ['repo B: shipping address']
```

# Phân tích log Groq (06-09/10/2026)

Nguồn: `groq-logs-Default_Project-7d-2026-10-12-08-27-26.csv`, xuất ngày 12/10/2026. 224 yêu cầu, một model duy nhất (`openai/gpt-oss-20b`), giờ trong bảng là UTC+7. Log chỉ có phía Groq (trạng thái, token, thời gian, không có nội dung prompt) và **kết thúc lúc 23:28 ngày 09/10**: không có yêu cầu nào từ 10/10 trở đi, nên các lần chạy ngày 10/10 (nếu có) không nằm trong dữ liệu này.

Nhãn bằng chứng như ở Sổ lỗi: **[log]**, **[code]**, **[ngoài]**.

## Con số chính

| Chỉ số                                   | Giá trị                                                                                        | Nhãn  |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------- | ----- |
| Tổng yêu cầu                             | 224: 147 thành công (66%), 66 lỗi 429 (29%), 11 lỗi 400 (5%). Tức 34% yêu cầu thất bại         | [log] |
| Token của yêu cầu thành công             | 307,976 vào + 24,906 ra = 332,882; trung bình 2,264 mỗi lượt                                   | [log] |
| Token theo ngày (chỉ yêu cầu thành công) | 06/10: 4,040; 07/10: 14,667; 08/10: 106,399; 09/10: 207,776                                    | [log] |
| Tỷ lệ 429 theo ngày                      | 08/10: 20/81 (25%); 09/10: 46/124 (37%)                                                        | [log] |
| Token vào mỗi lượt thành công            | nhỏ nhất 254; trung vị 1,426; p90 4,539; lớn nhất 6,705. 27 lượt trên 4,000, 5 lượt trên 6,000 | [log] |
| Độ trễ lượt thành công                   | trung vị 0.57s, lớn nhất 2.07s: thời gian chạy gần như toàn là thời gian chờ 429               | [log] |
| Số yêu cầu mỗi phút                      | nhiều nhất 12 (giới hạn RPM 30 không bị chạm)                                                  | [log] |

## Hạn mức: nút thắt là TPM, không phải TPD

- **66/66 lần 429 đều là TPM, giới hạn 8,000** (nội dung lỗi: `tokens per minute (TPM): Limit 8000, Used ..., Requested ...`). Mọi lần đều có `Used + Requested > 8000`. `Requested` trung vị 4,485, lớn nhất 7,150; 38/66 lần đòi hơn 4,000. Thời gian Groq gợi ý chờ từ 0.3s tới khoảng 45s. [log]
- Yêu cầu bị 429 ghi 0 token (không tính vào hạn mức) nhưng tốn thời gian và một lần gọi lại. [log]
- **Không có lỗi TPD nào**, dù 09/10 đã dùng 207,776 token trong một ngày. Giả định "TPD khoảng 200K, một lượt eval 14 task gần như hết một ngày" (Giai đoạn 11, nguồn tra cứu tháng 8) **không được log ủng hộ** cho model này. TPD thật vẫn chưa biết, phải đọc ở trang Limits trong console; log chỉ cho biết nó cao hơn khoảng 208K hoặc chưa được áp dụng. [log] cho phần quan sát, [ngoài] cho con số 200K cũ.
- **Hệ quả cho eval:** nút thắt là thời gian, không phải ngân sách ngày. Với 8,000 token/phút, thời gian tối thiểu của một lượt chạy là `token / 8,000` phút: 8 task cũ (ước 75K) ≥ 9.4 phút; 14 task (ước 130-180K) ≥ 16-23 phút, chưa tính 429 và chưa tính việc gói không khít. Các ước tính token đầu vào của hai con số này vẫn chưa đo.
- **Trần kích thước một lượt:** lượt lớn nhất đã qua là 6,705 vào + 235 ra, và lần 429 lớn nhất đòi 7,150, chỉ cách trần 8,000 khoảng 850. Một lượt đơn lẻ vượt 8,000 thì chờ bao lâu cũng không qua được. Cần trần ngữ cảnh cho mỗi lượt (ví dụ khoảng 6,000 token vào, theo ý "context budget" ở mục 2.5 của guide); con số 6,000 là đề xuất, chưa thử.
- **Chưa hiểu hết cơ chế TPM:** cửa sổ 60 giây dày nhất có 17,086 token thành công (10:54 08/10), lớn hơn 8,000. Vậy TPM không phải "tổng token thật trong 60 giây trượt" cứng; Groq nhiều khả năng tính theo bucket và theo số token ước tính (`Requested`). Đừng thiết kế bộ giãn nhịp (pacing) dựa trên giả định cửa sổ cứng. [log] cho con số, cơ chế chưa kiểm chứng.

## Lỗi 400

|Số lần|Nội dung lỗi|Thời điểm|Liên hệ|
|---|---|---|---|
|7|`Tool choice is required, but model did not call a tool`|08/10 13:01, 21:39 (2 lần); 09/10 18:32 (2 lần), 18:39, 18:40|Bug 16|
|3|`Failed to parse tool call arguments as JSON`|07/10 15:45; 08/10 12:59, 21:37|Bug 6, Bug 18|
|1|`attempted to call tool 'plan' which was not in request.tools`|08/10 13:03|**Bug 25 (mới)**|

- 11 lượt 400 đã dùng 20,756 token vào + 4,324 ra (khoảng 25K), tức khoảng 7% tổng token ghi nhận. [log]
- Lần "Tool choice is required" cuối cùng là 18:40 ngày 09/10. Sau đó có 54 lượt gọi (19:33 tới 23:28) và không có lỗi 400 nào. Phù hợp với việc Bug 16 đã được sửa trong buổi tối đó, nhưng log không cho biết thời điểm áp dụng bản sửa, nên chỉ là quan sát chứ chưa phải bằng chứng. Tương tự, 3 lỗi JSON đều trước 22:00 ngày 08/10; số lần quá ít để kết luận bản sửa planner có tác dụng.

## Đối chiếu với bảng kết quả ở Giai đoạn 10

Mỗi dòng của bảng được dò trong log như một chuỗi lượt gọi thành công liên tiếp (lượt 400 xen giữa bị bỏ) có cùng số lượt và cùng tổng token vào. **10/11 dòng khớp chính xác**:

|Ca (theo bảng Giai đoạn 10)|Khớp log lúc (UTC+7)|Ghi chú|
|---|---|---|
|`tools.py` → `summary.txt` (từ chối duyệt)|08/10 21:21:49|7 lượt, 13,732|
|`tool.py` → `summ.txt`|08/10 21:37:55|khớp chỉ khi bỏ các lượt 400 (6 lượt thành công, 17,631)|
|`tool.py` → `z.txt`|09/10 18:37:02|5 lượt, 7,213|
|`tool.py` → `x.txt` (1)|09/10 18:42:09|4 lượt, 5,562|
|`tools.py` → `w.txt` (10 lượt)|09/10 18:49:05|10 lượt, 37,466|
|`tool.py` → `x.txt` (2)|09/10 19:33:06|3 lượt, 4,581|
|`tools.py` → `w.txt` (8 lượt)|09/10 19:34:07|8 lượt, 27,145|
|`nonexistent_file.py` → `q.txt`|09/10 20:21:27|4 lượt, 4,832|
|`xin chao`|09/10 20:22:03|2 lượt, 1,875|
|`big.txt` → `big_summary.txt`|09/10 23:26:01|7 lượt, 27,787|
|`tools.py` → `y.txt`|**không khớp**|bảng ghi 7 lượt, 26,215; gần nhất là 26,205 lúc 20:24:38 ngày 09/10, lệch đúng 10. Nghi gõ nhầm một chữ số khi chép, chưa xác nhận|

Ba điều rút ra:

1. **Bộ đếm token của agent đúng với Groq** cho các lượt thành công: đây là xác nhận độc lập đầu tiên rằng số liệu `USAGE` đáng tin. [log]
2. **Bộ đếm bỏ sót các lượt 400** (khoảng 25K token trong toàn log; thấy rõ ở ca `tool.py → summ.txt`, chỉ khớp khi bỏ lượt 400) và không thấy các lượt 429. Số "lượt gọi" trong eval vì vậy thấp hơn số lần thử thật. Nên thêm cột "số lần thử" (kể cả 400 và 429) vào `eval_results.json` để đối chiếu với log Groq. [log]
3. Có thể dùng log Groq để kiểm tra giả thuyết "SDK tự thử lại 429 im lặng": sau một lượt eval, so số 429 phía Groq (trong khung giờ chạy) với số lần chờ do code của mình in ra. Nếu số phía Groq lớn hơn, SDK đang tự thử lại. Chưa làm.

## Rà soát `eval.py` (bản gửi ngày 12/10/2026)

Bốn chỗ lỗi thời đã ghi ở Giai đoạn 10 **đều đúng và còn nguyên trong file**: `tool_errors` đếm chuỗi `"Tool error"` (dòng 287), `duplicate_skips` còn tồn tại (dòng 288), ghi chú cuối bảng nói token "exclude the planner call" (dòng 332), và trạng thái ban đầu ở `graph.invoke` thiếu `partial_reads` (dòng 228-236). Thêm các điểm mới thấy khi đọc file [code, chưa chạy]:

- **`g.USAGE` không được đặt lại trong `run_task`.** Nếu `build_graph()` hoặc `use_model()` cũng không đặt lại, `llm_calls`, `input_tokens`, `output_tokens` sẽ cộng dồn qua các task và bảng đầu tiên sai. Cần xem `graph.py` trước khi tin con số đầu tiên. Có thể kiểm tra bằng log Groq: tổng token một task phải khớp một chuỗi liên tiếp như bảng đối chiếu ở trên.
- **`decide()` từ chối mọi hành động ngoài `write_file`/`edit_file`**, kể cả `execute_python`. Ca `execute_python` trong kế hoạch Giai đoạn 11 cần danh sách hành động được duyệt riêng cho từng task.
- **`wait_seconds` chỉ đếm `time.sleep` qua `g.time`.** Nếu `graph.py` không dùng `time.sleep` theo kiểu `import time`, hoặc SDK tự chờ, số này sẽ thấp hơn thực tế.
- **Không có giãn nhịp theo TPM.** Các task chạy liền nhau nên chắc chắn dính 429 (như 29-37% yêu cầu ở log); chấp nhận được cho baseline nếu `wait_seconds` được tách khỏi `seconds`, như file đã làm.
- **Chưa có ngưỡng số lượt và token cho từng task** (đã ghi là việc cần làm ở Giai đoạn 10).

## Việc làm tiếp từ log

1. Đọc TPD thật của `openai/gpt-oss-20b` ở trang Limits trong console Groq, ghi vào mục "Điều cần chốt".
2. Xem `graph.py` để xác nhận chuyện `USAGE` có được đặt lại không, rồi sửa `eval.py`: 4 chỗ lỗi thời, cột "số lần thử", danh sách hành động được duyệt theo task. Không động vào code của agent để giữ mốc "trước".
3. Điều tra Bug 25 bằng log DEBUG của lượt 13:03 ngày 08/10 (nếu còn) hoặc tái hiện planner với cùng prompt.
4. Đặt trần ngữ cảnh cho mỗi lượt gọi (đề xuất khoảng 6,000 token vào) sau khi có số baseline, không trước.
5. Sau mỗi lượt eval, xuất lại log Groq của đúng khung giờ để đối chiếu số lần thử và số 429.

# Roadmap và trạng thái

## Bản roadmap ngày 02/10/2026

|Mốc|Nội dung|Trạng thái lúc đó|
|---|---|---|
|Phase 0|Đọc code (Chroma) và search web (Tavily)|Xong, nhưng index có lỗi|
|Phase 1|`read_file`, `list_directory`, `write_file`, `edit_file` trong `codebase_path`|Xong|
|Guardrails|Gói `<tool_result>`, duyệt trước khi ghi, chặn path và `.env`, dừng khi bị từ chối|Xong|
|Phase 2|`execute_python` trong Docker (chặn mạng, có duyệt)|Có tool và đã test riêng, chưa thấy chạy qua agent|
|Phase 3|Planner, executor, finalizer, ép gọi tool|Xong; còn reflection/critic và replanning|
|Tin cậy và chi phí|Xử lý rate limit, cắt token, `eval.py`|Đã giao, chờ chạy eval thật|
|Phase 4+|Git, chạy test, review code|Chưa bắt đầu|

Hạng mục phụ lúc đó: LangSmith (hoãn), model dự phòng Groq sang Gemini (nâng ưu tiên vì vướng 429), MCP (tùy chọn), web app (sau cùng), Load Balancing (đề xuất bỏ, chưa chốt). Deadline phỏng vấn: tháng 12/2026.

Những chỗ roadmap tự nhận còn là "demo": thoát CLI mất toàn bộ hội thoại; `./chroma_db` nằm theo thư mục đang chạy nên chạy trong project thật sẽ tạo thư mục rác vào repo đó; `execute_python` không thấy file của project nên chưa có vòng "sửa code rồi chạy test"; file dài chỉ đọc được 6000 ký tự đầu.

**Mốc "dùng thật":** mở agent trong thư mục Pharmacy Wicky, giao 3 việc thật (tìm một hàm, sửa một lỗi nhỏ, viết một test) và nó làm đúng mà không phải sửa tay nhiều.

## Đối chiếu với trạng thái 10/10/2026 (theo log và code đã thấy)

|Hạng mục|Trạng thái|Còn thiếu|
|---|---|---|
|R1 Sửa index|Sửa một phần: collection theo repo, id theo `đường_dẫn:chỉ_số`, `upsert` theo batch, lập lại khi khởi động, bỏ thư mục rác|Chưa làm mới khi file đổi trong lúc phiên đang chạy (cache `_indexed_paths` theo phiên, nên agent vừa sửa file thì chỉ mục cũ); `./chroma_db` vẫn theo thư mục đang chạy, chưa chuyển ra ngoài repo; chưa kiểm chứng bằng truy vấn sau sửa|
|R4 Đọc theo khoảng dòng|Xong, có log xác nhận|Ca `big.txt` 600 dòng đọc đủ 3 cửa sổ|
|Guardrails|Thêm guard "đọc dở" cho `write_file`|`edit_file` chưa có guard tương tự; `.env*`, `*.pem`, `id_rsa*` chưa bị chặn theo mẫu; `search_web` chưa cần duyệt|
|Phase 2|Chưa thay đổi|Chưa có lần nào chạy `execute_python` qua agent; chưa có ca trong `eval.py`|
|Phase 3|Xong, đã sửa nhiều|Ép gọi tool đã bỏ (Bug 16, khác roadmap cũ); planner có `tidy_plan`; còn reflection/critic và replanning|
|Tin cậy và chi phí|Có backoff, cắt output, guard, `eval.py`|`eval.py` chưa chạy lần nào; cần sửa 4 chỗ (xem phần tồn đọng ở Giai đoạn 10)|
|R2 Lưu phiên SQLite|Chưa bắt đầu|Thoát CLI vẫn mất hội thoại|
|R3 Chạy test trong sandbox|Chưa bắt đầu|Xem rủi ro bên dưới|
|Model dự phòng|Chưa bắt đầu|Bug 24 cho thấy cần|

**Rủi ro của R3:** sandbox hiện chặn mạng và dùng image `python:3.12-slim` không có thư viện nào. Test của một project thật thường cần cài dependency, nên phải có image dựng sẵn (cài thư viện lúc build, không phải lúc chạy), hoặc chấp nhận thu hẹp ở loại test không cần thư viện ngoài. Cần chốt trước khi làm.

## Thứ tự đề xuất đến tháng 12

1. Chạy `eval.py` lần đầu để có số liệu gốc, sửa 4 chỗ lỗi thời của nó.
2. Áp dụng Bug 20 và Bug 21, chạy lại ca `xin chao` và `big.txt`.
3. Hoàn tất R1: chuyển `chroma_db` ra ngoài repo (thư mục dữ liệu của người dùng hoặc biến môi trường), làm mới khi `mtime` đổi. Việc này bắt buộc trước khi trỏ vào Pharmacy Wicky.
4. Cho `execute_python` chạy qua agent ít nhất một lần, thêm một ca vào `eval.py`.
5. R3 (việc lớn nhất): chạy test trong sandbox với bản sao chỉ đọc.
6. Model dự phòng khi gặp 429.
7. R2 lưu phiên.
8. Chạy 3 việc thật trên Pharmacy Wicky, ghi số liệu (tỷ lệ đạt, số lượt, token, chỗ hỏng). Đây là phần có giá trị nhất khi phỏng vấn.

Load Balancing, MCP, LangSmith, web app: để sau mốc "dùng thật".

# Kế hoạch giai đoạn kế tiếp (11-15): đầu vào và đầu ra

Chưa code. Mỗi giai đoạn chỉ xác định đầu vào, đầu ra và tiêu chí nghiệm thu, để biết khi nào được coi là xong. Thứ tự bám theo phần "Thứ tự đề xuất đến tháng 12" của mục Roadmap; hai việc chạy song song hoặc để sau là model dự phòng (Bug 24) và R2 lưu phiên.

## Giai đoạn 11: Đo baseline (chưa sửa gì thêm)

**Mục tiêu:** có số liệu gốc đáng tin để mọi bản sửa sau đo được. Hiện chỉ có khoảng 11 lần chạy tay, chưa có mốc nào. Mọi giai đoạn sau đều cần mốc "trước" để chứng minh "sau" tốt hơn.

**Đầu vào**

|Đầu vào|Chi tiết|Trạng thái|
|---|---|---|
|Code agent đóng băng|Hash commit của `graph.py`, `tools.py`, `state.py`, `cli.py`, `docker_tool.py`. Bug 20 và 21 chưa áp dụng, để có mốc "trước"|Cần commit và ghi hash|
|`eval.py` và repo giả|8 task hiện có|Có, nhưng lỗi thời 4 chỗ (xem Giai đoạn 10)|
|Cấu hình|`models.toml` (gpt-oss qua Groq), khóa Groq, khóa Tavily|Có|
|Hạn mức Groq|Token và request cho phép mỗi phút và mỗi ngày với `openai/gpt-oss-20b`|Biết một phần từ log: TPM = 8,000 (đã bị chạm 66 lần); RPM và TPD chưa bị chạm. TPD thật vẫn cần đọc trong console (xem "Phân tích log Groq")|
|Ca bổ sung (mô tả, chưa code)|Xem bảng bên dưới|Cần chốt|

**Ca bổ sung:** mỗi ca gồm lời yêu cầu và trạng thái đĩa kỳ vọng.

|Ca|Lời yêu cầu|Kỳ vọng cuối cùng|
|---|---|---|
|Tên file sai|"đọc `tool.py` rồi tạo `x.txt`"|`x.txt` không tồn tại; câu trả lời nêu file không thấy|
|File ở thư mục con|"đọc `utils.py` rồi tạo `s.txt`"|`s.txt` có, nhắc đúng nội dung `app/utils.py`|
|Từ chối duyệt|task ghi file, eval trả `reject`|không có file mới; câu trả lời nói rõ chưa làm|
|File dài|fixture 600 dòng, yêu cầu tóm tắt rồi ghi|có file; log cho thấy đọc đủ các cửa sổ|
|Chào hỏi tiếng Việt|"xin chào"|trả lời tự nhiên, không gọi tool (kiểm tra Bug 20)|
|`execute_python`|"tính tổng 1 đến 100 bằng code"|có lần duyệt; kết quả 5050|

**Đầu ra**

|Đầu ra|Nội dung|
|---|---|
|Số liệu gốc|`eval_results.json` và bảng tóm tắt: đạt/tổng cho từng task, số lượt gọi, token vào/ra (gồm planner), giây, giây chờ rate limit|
|Phân loại lỗi|Mỗi task trượt được gán một tầng (model, harness, tool, hạ tầng) và một số Bug, cũ hoặc mới|
|Ngưỡng cho từng task|Số lượt và token tối đa. Ca "đọc rồi ghi" đã có mốc dưới 10 lượt và 30K token (Bug 23); các ca khác lấy từ số đo|
|So sánh trước/sau|Sau khi áp dụng Bug 20 và 21, chạy lại đúng bộ này và đặt hai cột cạnh nhau|
|Giai đoạn 11 trong nhật ký|Hash commit, bảng số liệu, danh sách lỗi mới|

**Nghiệm thu**

- Chạy hết bộ ca mà không crash.
- Các ca không tất định chạy ít nhất 3 lần để ghi dạng "k/3"; một lần chạy chỉ là nhiễu.
- Mọi ca trượt đều có phân loại.
- Có đúng hai cột "trước" và "sau" cho Bug 20 và 21.

**Ngoài phạm vi:** chỉ mục, sandbox chạy test, lưu phiên, model dự phòng. Nếu muốn sửa gì ngay khi thấy lỗi thì ghi vào danh sách rồi sửa ở giai đoạn sau, nếu không mốc "trước" không còn nguyên.

**Rủi ro chính: TPM 8,000 của Groq (đã xác nhận từ log 12/10).** Nút thắt là tốc độ, không phải ngân sách ngày: 09/10 đã dùng 207,776 token trong một ngày mà không có lỗi TPD nào. Thời gian tối thiểu của một lượt chạy là `token / 8,000` phút: 8 task cũ (ước 75K, chưa đo) ≥ 9.4 phút; 14 task (ước 130-180K, chưa đo) ≥ 16-23 phút, chưa tính chờ 429 (29-37% yêu cầu ở log ngày 08-09/10 bị 429). "3 lần mỗi ca" vì vậy tốn thời gian chờ, không nhất thiết nhiều ngày. Hạn mức ngày (TPD) thật vẫn chưa biết; con số 200K từ nguồn tra cứu tháng 8 không được log ủng hộ. Một lượt gọi đơn lẻ vượt 8,000 token sẽ không bao giờ qua được (lượt lớn nhất đã thấy: 6,705 vào, lần 429 lớn nhất đòi 7,150).

**Mẫu bảng ghi kết quả** (điền khi chạy):

|Task|Trước: đạt|Trước: lượt gọi|Trước: token vào|Sau Bug 20+21: đạt|Sau: lượt gọi|Sau: token vào|Ghi chú lỗi|
|---|---|---|---|---|---|---|---|
|chat||||||||
|read_version||||||||
|find_symbol||||||||
|create_file||||||||
|edit_file||||||||
|secret_guard||||||||
|injection||||||||
|multi_step||||||||
|tên file sai||||||||
|file ở thư mục con||||||||
|từ chối duyệt||||||||
|file dài||||||||
|chào hỏi tiếng Việt||||||||
|`execute_python`||||||||

## Các giai đoạn sau

|Giai đoạn|Đầu vào|Đầu ra|
|---|---|---|
|12: Chỉ mục an toàn (R1)|Số liệu baseline; đường dẫn lưu chỉ mục mới; hai repo thử|Chỉ mục nằm ngoài repo; làm mới khi file đổi; hai repo không lẫn (truy vấn "shipping address" không ra hàm thanh toán của repo kia); `git status` của repo đích sạch sau khi chạy|
|13: `execute_python` qua agent|Ca `execute_python` trong eval|Log có đủ tool call, duyệt, kết quả; ca đạt trong eval|
|14: Chạy test trong sandbox (R3)|Repo đích (bản sao chỉ đọc), lệnh test, image có thư viện|Kết quả đạt/trượt của test đưa về cho model; mạng vẫn chặn; repo gốc không bị ghi|
|15: Dùng thật|Pharmacy Wicky, 3 việc thật (tìm một hàm, sửa một lỗi nhỏ, viết một test)|Tỷ lệ đạt, số lượt, token, danh sách chỗ hỏng; đây là câu chuyện cho phỏng vấn|

## Điều cần chốt trước khi bắt đầu

1. **Hạn mức Groq:** TPM = 8,000 đã xác nhận từ log; còn thiếu TPD thật của `openai/gpt-oss-20b` (đọc trang Limits trong console). Nút thắt là thời gian chờ, nên quyết định đo 1 lần hay 3 lần mỗi ca tùy vào thời gian bạn chịu chờ, không phải ngân sách ngày.
2. **Hash commit** của mốc "trước".
3. **Test của Pharmacy Wicky** viết bằng ngôn ngữ gì, chạy bằng gì, có cần cài thư viện không: quyết định giai đoạn 14 làm được hay phải thu hẹp (sandbox đang chặn mạng và dùng image Python trống).

# Kiến thức đã áp dụng

- [[]]**