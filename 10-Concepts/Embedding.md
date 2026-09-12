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

1 model AI biến **1 đoạn text** (câu, đoạn văn, hoặc cả tài liệu ngắn) thành **1 vector số cố định chiều** (ví dụ 384 số với `bge-small`). Đây là **document/sentence embedding**, KHÔNG phải token embedding (token embedding là khái niệm riêng, nằm bên trong kiến trúc Transformer để xử lý từng từ, không phải thứ dự án này dùng trực tiếp).

- **Cơ chế "học" ý nghĩa:** model được train bằng **contrastive learning** — cho model xem hàng triệu cặp câu được gắn nhãn "giống nhau" (positive pair) hoặc "khác nhau" (negative pair), model tự điều chỉnh trọng số sao cho positive pair cho vector gần nhau, negative pair cho vector xa nhau. Qua hàng triệu lần điều chỉnh, model tự suy ra cách biểu diễn ý nghĩa thành không gian vector — không ai lập trình luật thủ công.
- **KHÔNG có bước "giải mã vector về lại câu"** — cosine similarity chỉ so sánh 2 vector số để đo độ gần, không có cơ chế nào "dịch ngược" vector thành text. Text gốc được lưu riêng trong `payload` của Qdrant để hiển thị lại khi cần.

## Câu hỏi

**Hỏi:** Embedding là gì? Vì sao 2 câu có ý nghĩa gần nhau lại có vector gần nhau? **Bạn trả lời:** Embedding là kỹ thuật biến 1 từ (token) thành vector để LLM xử lý, 2 câu gần nghĩa thì vector gần nhau theo cosine similarity, khi chuyển về lại câu từ vector. **Đáp án đầy đủ:** Đây là **document/sentence embedding** (biến cả đoạn text thành 1 vector), không phải token embedding. Model học ý nghĩa qua **contrastive learning** (train trên cặp câu giống/khác nghĩa). **Không có bước "chuyển vector về lại câu"** — vector chỉ dùng để so sánh độ gần, text gốc lưu riêng trong payload.

## Dự án liên kết
- [[AI-Powered-Search-Engine]]