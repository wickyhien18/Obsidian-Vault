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

- **Recall@k:** có tìm ra được đáp án đúng trong top-k không? (Có/Không, không quan tâm thứ hạng)
- **MRR (Mean Reciprocal Rank):** đáp án đúng đứng thứ mấy? (đứng #1 tốt hơn #5, tính bằng `1/rank`)
- **Precision@k:** trong k kết quả trả về, bao nhiêu % thực sự đúng?

**Ví dụ Recall cao nhưng Precision thấp** (case thật): search `"kapranos"`, đáp án đúng `{233, 386}`, `top_k=5`. Nếu trả về `[233, 999, 888, 777, 666]` → Recall = 100% (có `233`), nhưng Precision chỉ 20% (1/5 đúng, 4/5 là rác).

**Kết quả thật đã đo:** Recall@5 và MRR bằng nhau tuyệt đối giữa dense-only và hybrid (1.00/1.000) — không phân biệt được 2 phương pháp. Precision@5 mới lộ ra khác biệt thật: hybrid thắng rõ ở câu hỏi chứa tên riêng, nhưng thua nhẹ ở câu hỏi ngữ nghĩa thuần — **không có bằng chứng "hybrid luôn tốt hơn"**, cần golden set lớn hơn để kết luận chắc chắn.

## Câu hỏi

**Hỏi:** Recall@k, MRR, Precision@k đo gì khác nhau? Ví dụ Recall cao Precision thấp? **Bạn trả lời:** Chưa tìm hiểu. **Đáp án đầy đủ:** Xem mục 8 — ví dụ cụ thể: trả về `[233, 999, 888, 777, 666]` cho "kapranos" (đáp án `{233,386}`) → Recall 100%, Precision 20%.
## Dự án liên kết
- [[AI-Powered-Search-Engine]]