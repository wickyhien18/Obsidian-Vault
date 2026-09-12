---
created: 2026-09-12
Hub: "[[MOC - AI & ML]]"
status: seed
---

> [!summary] Tóm tắt nhanh
> -
> -
> -
## Định nghĩa

**Memory:** lưu lịch sử hội thoại (`history`), gửi kèm mỗi request — thiết kế **stateless phía server** (client tự giữ và gửi lại toàn bộ lịch sử mỗi lần, giống cách OpenAI/Anthropic API hoạt động)

**Vấn đề nếu KHÔNG có Query Rewriting:** retrieval dùng **nguyên văn** câu hỏi follow-up (vd `"which category does that belong to?"`) để search — câu này không chứa từ khoá thật, hybrid search tìm sai chunk — dù bản thân LLM (nhờ có `history` trong prompt) vẫn hiểu đúng "that" ám chỉ gì. **LLM hiểu đúng ngữ cảnh, nhưng retrieval "mù" trước ngữ cảnh đó** — đây là lý do cần 1 bước LLM riêng viết lại câu hỏi thành dạng độc lập TRƯỚC KHI đưa vào retrieval.

## Câu hỏi

**Hỏi:** Vì sao cần Query Rewriting cho follow-up? Không có thì sao? **Bạn trả lời:** Để câu hỏi sau có ngữ cảnh câu trước; không có sẽ không đồng nhất ngữ cảnh. **Đáp án đầy đủ:** Đúng ý nhưng mơ hồ. Cụ thể: không có rewriting, **retrieval** (không phải LLM) dùng nguyên văn câu hỏi follow-up để search, tìm sai chunk vì thiếu từ khoá thật — dù LLM vẫn hiểu đúng ngữ cảnh nhờ `history`. Vấn đề nằm ở retrieval "mù" trước ngữ cảnh, không phải LLM không hiểu.

## Dự án liên kết
- [[AI-Powered-Search-Engine]]