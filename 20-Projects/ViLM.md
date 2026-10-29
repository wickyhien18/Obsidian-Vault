---
Hub: "[[]]"
created: 2026-10-29
status: seed
repo:
---

# Ý tưởng dự án

Tự huấn luyện **từ đầu (pretrain, không fine-tune)** một mô hình ngôn ngữ rất nhỏ cho tiếng Việt trên Google Colab, rồi xuất sang GGUF và chạy local bằng Ollama trên laptop cá nhân.

- **Phạm vi:** mô hình trả lời câu hỏi đơn giản mức mầm non / cấp 1 (người dùng hỏi, mô hình đáp). Viết truyện ngắn chỉ là hướng phát triển nếu còn thời gian.
- **Mục đích:** làm đẹp CV, trình bày cho thầy giáo, và hiểu sâu cách một LLM được huấn luyện (tokenizer, kiến trúc, vòng lặp huấn luyện, đánh giá).
- **Thời hạn:** khoảng 1 tháng (tính từ 2026-10-29).
- **Điều cần nói thẳng trong báo cáo:** với kiến thức đóng, bảng tra thắng mô hình về độ chính xác và chi phí. Giá trị của dự án nằm ở việc hiểu cách mô hình học và khả năng tổng quát hóa sang cách hỏi mới, nên dùng **bảng tra làm baseline** để so sánh.
- **Hướng dữ liệu:** thu hẹp thế giới theo mức mầm non (con vật, màu, thức ăn, đồ vật, cơ thể, thời tiết...) thay vì dữ liệu rộng. Cảm hứng: TinyStories và BabyLM.

# Tech Stack
| Thành phần | Công nghệ | Lý do chọn |
| ---------- | --------- | ---------- |
| Dữ liệu | Bảng sự thật `facts.csv` + `gen_data.py` (template, thư viện chuẩn Python) — **đã làm** | Đáp án đúng do mình kiểm soát, chấm điểm tự động, không phụ thuộc API hay điều khoản nhà cung cấp |
| Tokenizer | BPE (Hugging Face `tokenizers`), vocab nhỏ, chuẩn hóa NFC — *dự kiến* | Model nhỏ nên vocab phải nhỏ; tiếng Việt cần NFC để không tách một chữ thành hai chuỗi byte |
| Kiến trúc | Llama-architecture qua `transformers` — *dự kiến* | llama.cpp convert được sang GGUF để chạy bằng Ollama |
| Huấn luyện | PyTorch, vòng lặp tự viết, fp16 + GradScaler, Colab T4 — *dự kiến* | Mục tiêu hiểu sâu; T4 không có bf16 tốt; lưu checkpoint lên Drive vì Colab free có thể ngắt phiên |
| Đánh giá | 3 tập kiểm tra (A, B, C) + baseline tra bảng — *dự kiến baseline* | Đo tổng quát hóa cách hỏi, khả năng bịa, khả năng từ chối |
| Triển khai | GGUF + Ollama (Modelfile template thô) — *dự kiến* | Chạy local trên máy không có GPU rời |

# Nhật ký kỹ thuật

## Giai đoạn 0 — Chốt phạm vi

**Mục tiêu giai đoạn:** Quyết định làm gì và không làm gì trong 1 tháng, để không bị cháy thời gian.

**Đã làm:**

- Chốt pretrain từ đầu trên Colab, chạy local bằng Ollama.
- Chốt mục tiêu trước mắt là hỏi–đáp mức mầm non; truyện ngắn đẩy sang "hướng phát triển".
- Chốt vai trò: người dùng hỏi, mô hình đáp.
- Lập kế hoạch 4 tuần: (1) dữ liệu, (2) vòng lặp huấn luyện, (3) thí nghiệm + GGUF/Ollama, (4) báo cáo/slide/repo.

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** pretrain từ đầu hay fine-tune một model có sẵn.
- **Đã chọn:** pretrain từ đầu.
- **Vì sao:** mục tiêu là hiểu sâu cách LLM được huấn luyện. Đánh đổi: mô hình sẽ yếu và chỉ biết thế giới rất hẹp, không dùng được như trợ lý.

- **Vấn đề:** hỏi–đáp hay viết truyện làm mục tiêu đầu.
- **Đã chọn:** hỏi–đáp trước.
- **Vì sao:** theo lựa chọn của người dùng. Đánh đổi: khó hơn truyện, vì cần cả sự thật, và bằng chứng công khai (TinyStories) chủ yếu là về truyện.

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa chốt mục đích ưu tiên khi phải đánh đổi (CV, thầy giáo hay hiểu sâu). Chưa biết số giờ mỗi tuần dành cho dự án.

## Giai đoạn 1 — Dữ liệu (tuần 1)

**Mục tiêu giai đoạn:** Có bộ dữ liệu hỏi–đáp kiểm soát được, đã tách train/val/test, có thống kê để tự kiểm tra bằng mắt.

**Đã làm:**

- Viết bảng sự thật v0: **91 dòng**, 8 thuộc tính (tiếng kêu, màu sắc, ăn gì, số chân, sống ở đâu, vị, công dụng, nên làm gì). *Bản nháp do Claude hỗ trợ soạn; cần tự rà từng dòng và tự giải thích được.*
- Viết `gen_data.py`: sinh cặp hỏi–đáp bằng template, chuẩn hóa NFC, tách tập, tự kiểm tra rò rỉ (dừng ngay nếu có).
- Chạy thử, seed 42. Kết quả: 76 sự thật cho train, 15 sự thật giữ lại; **train 2.721 cặp**, val 143, test_A 262, test_B 120, test_C 15.
- Thống kê: 332 âm tiết khác nhau trong train, câu hỏi trung bình khoảng 6,6 âm tiết, câu đáp khoảng 6,4 âm tiết, khoảng 40 nghìn âm tiết tổng (xấp xỉ token ở mức âm tiết), 5,5% dữ liệu train là câu ngoài phạm vi.

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** sinh dữ liệu bằng LLM tự do hay từ bảng sự thật + template.
- **Đã chọn:** bảng sự thật + template (LLM chỉ dùng sau này, nếu cần, để diễn đạt lại).
- **Vì sao:** đáp án đúng do mình kiểm soát, chấm điểm tự động bằng khớp giá trị, không phụ thuộc giới hạn API. Đánh đổi: độ đa dạng bị chặn bởi số template (tối đa 48 tổ hợp mỗi sự thật), câu văn cứng.

- **Vấn đề:** giọng khi từ chối câu ngoài phạm vi.
- **Đã chọn:** trung tính, "Mình chưa biết." (4 biến thể).
- **Vì sao:** chưa chốt giọng của mô hình (người lớn hay trẻ nhỏ); đổi sau chỉ cần sửa danh sách `REFUSALS` rồi sinh lại.

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 1 — Tập B bị rò gián tiếp qua giá trị lặp:** Quan sát: sự thật bị giữ lại như "con gà sống trong chuồng" có cùng giá trị với "con heo sống trong chuồng" còn nằm trong train → Nguyên nhân thật sự: giá trị đáp án lặp giữa các chủ thể, nên mô hình có thể đoán đúng mà không thật sự biết (48/120 dòng test_B rơi vào trường hợp này) → Cách sửa: thêm cờ `value_seen_in_train` vào test_B và thống kê riêng; khi báo cáo, chỉ tính các dòng `unseen` (72/120) là bằng chứng "bịa hay biết nói chưa biết". Chưa sửa tận gốc bằng cách chọn lại sự thật giữ lại.

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Tự rà 91 sự thật, đọc tay ít nhất 50 cặp sinh ra; sửa dòng sai hoặc mơ hồ (ví dụ "con khỉ ăn chuối", "con chó ăn thịt và xương" có nhiều đáp án đúng, làm chấm điểm khó).
- Mở rộng bảng lên 150–300 dòng (hiện 91). Số cặp train hiện chỉ 2.721, thấp hơn nhiều so với ước tính ban đầu, vì mỗi sự thật chỉ có tối đa 48 tổ hợp; muốn thêm dữ liệu thì cần thêm template, không chỉ thêm sự thật.
- Huấn luyện tokenizer trên Colab (môi trường hiện tại chưa cài `tokenizers`, nên chưa chạy được); đếm số âm tiết khác nhau để chọn vocab.
- Baseline tra bảng để dành cho tuần 3.

# Kiến thức đã áp dụng
- [[Train-Val-Test Split]]
- [[Data Leakage]]
- [[Unicode NFC Normalization]]
- [[Scaling Laws (Chinchilla)]]
