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

|           | Dense vector                                                       | Sparse vector (BM25)                                                            |
| --------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| Biểu diễn | Ý nghĩa (semantic), nén thành N số cố định, gần như toàn bộ khác 0 | Từng từ cụ thể có mặt hay không, phần lớn giá trị là 0 (do dùng inverted index) |
| Mạnh ở    | Paraphrase — "phim hoãn chiếu" ≈ "premiere pushed back"            | Tên riêng, từ hiếm — "kapranos"                                                 |
| Yếu ở     | Tên riêng, thuật ngữ hiếm (không có "ý nghĩa" phong phú để mã hoá) | Không hiểu paraphrase, chỉ khớp từ đúng nghĩa đen                               |

### Cơ chế lọc từ trong sparse (BM25) — 2 cơ chế RIÊNG BIỆT, hay bị nhầm làm 1

1. **Stopword removal** — xảy ra ở bước tokenize, TRƯỚC KHI tính BM25. Từ trong danh sách cố định (`"the"`, `"a"`, `"and"`...) bị **loại bỏ hoàn toàn**, không có vị trí/giá trị gì trong sparse vector.
2. **IDF (Inverse Document Frequency)** — áp dụng cho các từ KHÔNG phải stopword. Từ xuất hiện ở nhiều document → IDF thấp (trọng số giảm, nhưng **vẫn được lưu**, không bị loại). Từ hiếm (như "kapranos") → IDF cao.

→ Không phải "the bị BM25 bỏ qua" — mà "the" bị loại bởi stopword (bước riêng), còn các từ phổ biến khác (không phải stopword) chỉ bị giảm trọng số bởi IDF, không bị loại.

### RRF (Reciprocal Rank Fusion) — cách gộp kết quả dense + sparse

- Công thức: `score = 1 / (rank + k)`, trong đó **`rank` tính từ 0** (vị trí #1 = rank 0), **`k` là hằng số cố định = 2** (không phải tham số tự chỉnh, xác nhận từ source code `qdrant-client`)
- **Vì sao không cộng trực tiếp cosine similarity + điểm BM25:** 2 loại điểm này không cùng thang đo (cosine luôn 0-1, BM25 có thể là số bất kỳ tuỳ độ dài văn bản/độ hiếm từ) — cộng trực tiếp vô nghĩa, giống cộng "5kg" với "5°C". Quy về **thứ hạng** trước là cách duy nhất công bằng để so sánh 2 hệ thống chấm điểm khác nhau.

## Câu hỏi

**Hỏi:** Dense và sparse vector khác nhau cốt lõi ở điểm nào? **Bạn trả lời:** Dense lưu ngữ nghĩa, sparse lưu keyword. **Đáp án đầy đủ:** Đúng nhưng nông — sparse không chỉ "lưu keyword" mà lưu **keyword có trọng số theo độ hiếm** (IDF)

**Hỏi:** Dense embedding "học" ý nghĩa từ đâu để 2 câu paraphrase cho vector gần nhau? **Bạn trả lời:** Chưa rõ. **Đáp án đầy đủ:** **Dense embedding "học" ý nghĩa từ đâu:** model được train trên hàng triệu/tỷ cặp câu, dùng kỹ thuật gọi là **contrastive learning** — trong lúc train, model được cho biết "câu A và câu B có ý nghĩa giống nhau" (positive pair) hoặc "khác nhau" (negative pair), và bị điều chỉnh trọng số sao cho **positive pair cho ra vector gần nhau, negative pair cho ra vector xa nhau**. Sau hàng triệu lần điều chỉnh như vậy, model tự "học" được cách biểu diễn ý nghĩa thành không gian vector — không ai lập trình luật "phim = premiere" thủ công, mà model tự suy ra từ dữ liệu train.

**Hỏi**: Vì sao "the" (1000 lần) có trọng số thấp hơn "kapranos" (2 lần) trong BM25? **Bạn trả lời:** Vì "the" quá phổ thông nên BM25 bỏ qua không lưu. **Đáp án đầy đủ:** Sai cơ chế — "the" bị loại bởi **stopword removal** (loại hoàn toàn, TRƯỚC BM25), không phải "BM25 bỏ qua". Các từ phổ biến khác (không phải stopword) vẫn được lưu, chỉ giảm trọng số qua **IDF**

**Hỏi:** RRF dùng công thức gì? Vì sao không cộng trực tiếp cosine + BM25? **Bạn trả lời:** score = 1/(rank+k), rank là thứ tự tìm kiếm, k là trọng số — chưa rõ vì sao. **Đáp án đầy đủ:** `rank` tính từ 0 (không phải "thứ tự" chung chung). `k` là **hằng số cố định = 2**, không phải tham số tự chỉnh. Không cộng trực tiếp vì 2 loại điểm khác thang đo (cosine 0-1 cố định, BM25 bất kỳ) — quy về rank là cách công bằng duy nhất.


## Dự án liên kết
- [[AI-Powered-Search-Engine]]