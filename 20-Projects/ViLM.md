---
Hub: "[[MOC - AI & ML]]"
created: 2026-10-29
status: đang làm (giai đoạn 3 — thí nghiệm và xuất mô hình, chưa bắt đầu; giai đoạn 2 đã xong)
repo: https://github.com/wickyhien18/ViLM (private)
---

# Ý tưởng dự án

Tự huấn luyện **từ đầu (pretrain, không fine-tune)** một mô hình ngôn ngữ rất nhỏ cho tiếng Việt trên Google Colab, rồi xuất sang GGUF và chạy local bằng Ollama trên laptop cá nhân.

- **Phạm vi:** mô hình trả lời câu hỏi đơn giản mức mầm non / cấp 1 (người dùng hỏi, mô hình đáp). Viết truyện ngắn chỉ là hướng phát triển nếu còn thời gian.
- **Mục đích:** làm đẹp CV, trình bày cho thầy giáo, và hiểu sâu cách một LLM được huấn luyện (tokenizer, kiến trúc, vòng lặp huấn luyện, đánh giá). Chưa xếp thứ tự ưu tiên khi phải đánh đổi.
- **Thời hạn:** deadline cuối là cuối tháng 11/2026; hiện không có deadline nào khác. Mốc tuần 1: thứ Tư 4/11. Số giờ mỗi tuần chưa chốt (Claude đề xuất khoảng 10).
- **Cách làm việc:** học bằng cách làm (không học trước khóa dài); tự tay thêm sự thật và tự kiểm tra bằng tay; người dùng và Claude cùng giám sát; đầu mỗi giai đoạn ghi danh sách việc cần làm vào note, trong lúc làm Claude ghi nhớ, cuối giai đoạn ghi toàn bộ kết quả vào đúng mục của giai đoạn đó.
- **Điều cần nói thẳng trong báo cáo:** với kiến thức đóng, bảng tra thắng mô hình về độ chính xác và chi phí. Giá trị của dự án nằm ở việc hiểu cách mô hình học và khả năng tổng quát hóa sang cách hỏi mới, nên dùng **bảng tra làm baseline** để so sánh.
- **Hướng dữ liệu:** thu hẹp thế giới theo mức mầm non (con vật, màu, thức ăn, đồ vật, cơ thể, thời tiết, ngày lễ...) thay vì dữ liệu rộng. Cảm hứng: TinyStories và BabyLM.
- **Tài liệu tham khảo của thầy:** `LLM-workshop-2024-main.rar`, gồm 7 phần theo tên thư mục (01_intro, 02_data, 03_architecture, 04_pretraining, 05_weightloading, 06_finetuning, 07_outro), có `the-verdict.txt` và `instruction-data.json`. Đã đọc `README.md`, `requirements.txt`, `03.ipynb`, `04.ipynb` và hai file `supplementary.py`; các phần còn lại chưa đọc. Bài tập 3 (tùy chọn) của notebook 04 là huấn luyện LLM trên văn bản của riêng mình.

# Tech Stack
| Thành phần | Công nghệ | Lý do chọn |
| ---------- | --------- | ---------- |
| Dữ liệu | Bảng sự thật `facts.csv` (tự thêm bằng tay) + `gen_data.py` bản v0 (8 mẫu hỏi, 4 mẫu đáp mỗi thuộc tính, tối đa 40 cặp mỗi sự thật) — **đã làm** | Đáp án đúng do mình kiểm soát, chấm điểm tự động, không phụ thuộc API hay điều khoản nhà cung cấp |
| Môi trường Python | `uv` (thay `venv` + `pip`); phụ thuộc ghi trong `pyproject.toml`, phiên bản khóa trong `uv.lock` — **đã cài (uv 0.12.3)** | Người dùng muốn nhanh và muốn học `uv`; `uv.lock` giúp tái lập môi trường và dùng được khi triển khai |
| Tokenizer | BPE (Hugging Face `tokenizers`), vocab nhỏ, chuẩn hóa NFC — *thư viện đã cài (tokenizers 0.23.3); đã đo vocab 512/1024/2048; chốt kích thước do dữ liệu quyết định (hiện 1057, kích thước thật của cỡ 2048)* | Model nhỏ nên vocab phải nhỏ; tiếng Việt cần NFC để không tách một chữ thành hai chuỗi byte |
| Kiến trúc | Tự viết kiểu workshop của thầy (GPT-2, `model.py`): emb 128, 4 lớp, 4 đầu, ngữ cảnh 64, buộc trọng số, 935.296 tham số — **đã làm** | Mã của thầy có sẵn nên nhanh mà vẫn học được kiến trúc; đánh đổi: xuất GGUF cho Ollama phải tự chuyển định dạng và chưa kiểm chứng llama.cpp có nhận (chưa làm) |
| Huấn luyện | PyTorch, vòng lặp tự viết (AdamW, warmup + cosine, cắt gradient), fp16 + GradScaler, Colab (GPU; loại thực tế chưa ghi lại) — **đã làm**: 10.000 bước hết 304 giây | Mục tiêu hiểu sâu; T4 không có bf16 tốt; lưu checkpoint lên Drive vì Colab free có thể ngắt phiên |
| Đánh giá | 3 tập kiểm tra (A, B, C) chấm bằng `score.py` và `metrics.py` — **đã làm**; baseline tra bảng — *chưa làm* | Đo tổng quát hóa cách hỏi, khả năng bịa, khả năng từ chối |
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

- Chưa chốt số giờ mỗi tuần dành cho dự án (Claude đề xuất khoảng 10). Chưa xếp ưu tiên giữa CV, thầy giáo và hiểu sâu khi phải đánh đổi.

## Giai đoạn 1 — Dữ liệu (tuần 1)

**Mục tiêu giai đoạn:** Có bộ dữ liệu hỏi–đáp kiểm soát được, đã tách train/val/test, có thống kê để tự kiểm tra bằng mắt.

**Đã làm:**

- Bảng sự thật v0: 91 dòng, 8 thuộc tính (tiếng kêu, màu sắc, ăn gì, số chân, sống ở đâu, vị, công dụng, nên làm gì). *Bản nháp do Claude hỗ trợ soạn.*
- `gen_data.py`: sinh cặp hỏi–đáp bằng template, chuẩn hóa NFC, tách tập, tự kiểm tra rò rỉ (dừng ngay nếu có).
- Người dùng tự thêm bằng tay 16 dòng (id 92–107). Bảng hiện có **107 sự thật**; mỗi thuộc tính có ít nhất 12 (ăn gì 13, công dụng 21, màu sắc 12, nên làm gì 12, số chân 12, sống ở đâu 12, tiếng kêu 12, vị 13).
- Chạy lại `gen_data.py` (seed 42) trên bảng 107 dòng: 90 sự thật cho train, 17 sự thật giữ lại; **train 3.147 cặp**, val 165, test_A 304, test_B 136 (80 dòng "unseen", 56 dòng "seen"), test_C 15.
- Thống kê: 382 âm tiết khác nhau trong train, câu hỏi trung bình khoảng 6,8 âm tiết, câu ngoài phạm vi chiếm 4,7% train.
- Đọc tay 50 cặp ngẫu nhiên của train: không thấy lỗi chữ, ngữ pháp hay sự thật; chỉ phát hiện độ lặp (xem mục Quan sát).
- Cài `uv` 0.12.3: thư mục dự án có `pyproject.toml`, `uv.lock` và `.venv`; `tokenizers` 0.23.3 import được qua `uv run`, và `tokenizers>=0.23.3` đã nằm trong `dependencies` của `pyproject.toml` (cài bằng `uv add`).
- Đo tokenizer BPE byte-level (huấn luyện chỉ trên `data/train.jsonl`: 48.879 âm tiết, 382 âm tiết khác nhau) cho ba cỡ vocab: **512** → 1,70 token/âm tiết, 19% âm tiết khác nhau là 1 token; **1024** → 1,34 và 82%; **2048** → 1,33 và 88%. Ở cỡ 2048, BPE chỉ thực hiện 808 trong 1.788 phép gộp dự kiến vì hết cặp để gộp, nên kích thước thật là 1.068 token (260 ký hiệu nền + 808 phép gộp; đã xác nhận bằng `get_vocab_size()`).
- Lưu hai tokenizer vào `tokenizers/tok_1024.json` (kích thước thật 1024) và `tokenizers/tok_2048.json` (kích thước thật 1068). Id token đặc biệt: `<pad>`=0, `<s>`=1, `</s>`=2, `<unk>`=3. Câu mẫu "Hỏi: Con mèo kêu như thế nào?\nĐáp: Con mèo kêu meo meo." thành 18 token ở cả hai cỡ, mỗi chữ thường là một token.
- Chốt tokenizer: sao chép `tokenizers/tok_2048.json` thành `tokenizer.json`; `get_vocab_size()` = 1068; câu "Con mèo kêu meo meo." mã hóa thành 7 id `[38, 292, 484, 324, 597, 597, 17]`.
- Mã hóa dữ liệu thành dãy số (`prepare_data.py`): mỗi cặp được bọc `<s> ... </s>`; train 3.147 cặp thành 71.433 token (trung bình 22,7 token/cặp), val 165 cặp thành 3.712 token (22,5 token/cặp); file `data/train.bin` (142.866 byte) và `data/val.bin` (7.424 byte), mỗi token 2 byte.
- Tách 4 dòng cá nhân (id 94, 97, 103, 104) sang `facts_personal.csv` (không dùng khi train); thêm `facts_personal.csv` và file sao lưu `facts_backup_before_split.csv` vào `.gitignore`. `facts.csv` còn 103 sự thật, `facts_personal.csv` có 4 dòng.
- Sửa dòng 98 (`con rơi` thành `con dơi`) và dòng 107 (thêm "ở" trước `châu Úc`); thêm 2 dòng bù (id 108, 109). `facts.csv` hiện có 105 sự thật, mỗi thuộc tính có 12 dòng (riêng `công dụng` 21). Các số liệu train/val/test, tokenizer và file `.bin` ở các dòng trên là của bảng cũ (107 sự thật).
- Chạy lại toàn bộ trên bảng mới (105 sự thật): 88 sự thật cho train, 17 giữ lại; **train 3.116 cặp**, val 164, test_A 302, test_B 136 (80 dòng `unseen`, 56 dòng `seen`), test_C 15; 377 âm tiết khác nhau, câu hỏi trung bình 6,75 âm tiết, câu ngoài phạm vi 4,8% train. Tokenizer: đặt trần 2048 cho kích thước thật **1.057** (trước 1.068), đặt 1024 vẫn là 1.024; câu mẫu vẫn thành 18 token; các tên riêng không còn trong danh sách chữ bị cắt vụn. Đã sao chép bản 1.057 thành `tokenizer.json`. Mã hóa: train 70.258 token (22,5 token/cặp), val 3.694 token; `data/train.bin` 140.516 byte, `data/val.bin` 7.388 byte.

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** sinh dữ liệu bằng LLM tự do hay từ bảng sự thật + template.
- **Đã chọn:** bảng sự thật + template (LLM chỉ dùng sau này, nếu cần, để diễn đạt lại).
- **Vì sao:** đáp án đúng do mình kiểm soát, chấm điểm tự động bằng khớp giá trị, không phụ thuộc giới hạn API. Đánh đổi: độ đa dạng bị chặn bởi số template (tối đa 48 tổ hợp mỗi sự thật), câu văn cứng.

- **Vấn đề:** giọng khi từ chối câu ngoài phạm vi.
- **Đã chọn:** trung tính, "Mình chưa biết." (4 biến thể).
- **Vì sao:** chưa chốt giọng của mô hình (người lớn hay trẻ nhỏ); đổi sau chỉ cần sửa danh sách `REFUSALS` rồi sinh lại.

- **Vấn đề:** dùng bản nháp sự thật và script kiểm tra của Claude, hay tự làm bằng tay.
- **Đã chọn:** người dùng tự thêm sự thật và tự kiểm tra bằng tay. Bản v1 của Claude (58 sự thật nháp, template bổ sung, `check_data.py`) vẫn còn nhưng chưa dùng.
- **Vì sao:** tự làm phần của mình để hiểu sâu và trả lời được câu "em tự làm phần nào". Đánh đổi: tốn nhiều giờ hơn và dễ sót lỗi, đã gặp Bug 2 và Bug 3.

- **Vấn đề:** có mở rộng template ngay ở tuần 1 không.
- **Đề xuất của Claude (chờ người dùng xác nhận):** chưa mở rộng; khoảng 3 nghìn cặp đủ để chạy vòng lặp huấn luyện ở tuần 2, quay lại sau lần chạy đầu.
- **Vì sao:** mục tiêu tuần 2 là vòng lặp chạy đúng. Đánh đổi: dữ liệu ít và khuôn mẫu hơn.

- **Vấn đề:** quản lý môi trường và thư viện Python bằng `venv` + `pip` hay `uv`.
- **Đã chọn:** `uv` (theo yêu cầu của người dùng).
- **Vì sao:** nhanh hơn, ghi phụ thuộc vào `pyproject.toml` và khóa phiên bản trong `uv.lock`; người dùng muốn học `uv`. Đánh đổi: thêm một công cụ phải học, lệnh khác `pip`.

- **Vấn đề:** chọn cỡ vocab cho tokenizer: 1024 hay 1068 (kích thước thật khi đặt trần 2048).
- **Đã chọn:** để dữ liệu quyết định, tức kích thước thật của cỡ 2048 (`tokenizers/tok_2048.json`): 1.068 ở bảng cũ, **1.057** ở bảng hiện tại.
- **Vì sao:** người dùng muốn bớt trường hợp thiếu chữ khi đoán. Thực tế 1068 giữ nguyên vẹn thêm khoảng 44 chữ hiếm có trong train (cận dưới: 88% so với 82% âm tiết khác nhau là 1 token), nhưng không giúp với chữ chưa có trong train (chúng bị cắt thành mảnh ở cả hai cỡ, không có `<unk>`). Đánh đổi: bám sát dữ liệu hiện tại nên phải huấn luyện lại tokenizer khi thêm nhiều sự thật mới; 1068 không chia hết cho 8 nên có thể chậm hơn chút trên GPU (chưa đo, ảnh hưởng nhỏ với mô hình này).

- **Vấn đề:** bốn dòng của `facts.csv` nói về một người cụ thể (id 94, 97, 103, 104): để trong bảng chính, xóa, hay tách riêng.
- **Đã chọn:** tách sang `facts_personal.csv`, không dùng khi train.
- **Vì sao:** đó là dữ liệu cá nhân của người dùng; tách ra để không đưa vào dữ liệu train, tokenizer và repo, mà vẫn giữ lại cho thí nghiệm sau này. Đánh đổi: bảng chính mất 4 dòng nên `nên làm gì` và `sống ở đâu` còn 11 dòng (dưới mức 12 đã đặt), cần thêm dòng bù; mô hình sẽ không biết các dữ kiện cá nhân này.

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 1 — Tập B bị rò gián tiếp qua giá trị lặp:** Quan sát: sự thật bị giữ lại như "con gà sống trong chuồng" có cùng giá trị với "con heo sống trong chuồng" còn nằm trong train → Nguyên nhân thật sự: giá trị đáp án lặp giữa các chủ thể, nên mô hình có thể đoán đúng mà không thật sự biết (ở bản 91 dòng: 48/120 dòng test_B; ở bản 107 và 105 dòng: 56/136) → Cách sửa: thêm cờ `value_seen_in_train` vào test_B và thống kê riêng; khi báo cáo chỉ tính các dòng `unseen` (hiện 80/136). Chưa sửa tận gốc bằng cách chọn lại sự thật giữ lại.
- **Bug 2 — Hai dòng của `facts.csv` thiếu cột làm `gen_data.py` dừng:** Quan sát: `TypeError: normalize() argument 2 must be str, not None` → Nguyên nhân thật sự: dòng 103 gộp "vị socola" vào một ô (thiếu cột) và dòng 105 thiếu giá trị; `csv.DictReader` điền `None` cho cột thiếu rồi `norm(None)` báo lỗi → Cách sửa: viết đủ 5 cột `id,group,subject,attribute,value` (`103,vị,<chủ thể>,vị,socola` và `105,thức ăn,con gấu trúc,ăn gì,tre`). Script chưa tự kiểm tra số cột; có thể thêm sau.
- **Bug 3 — Chủ thể hoặc giá trị không khớp ngữ pháp của template:** Quan sát: câu sinh ra bị cụt hoặc lặp, ví dụ "Khi 20/11…", "…sống Bắc Ninh.", "…ăn ăn mì tôm." → Nguyên nhân thật sự: chủ thể và giá trị được ghép thẳng vào mẫu câu, nên chủ thể ngày tháng cần "đến ngày/đến dịp", giá trị của "sống ở đâu" cần giới từ "ở", giá trị của "ăn gì" không được bắt đầu bằng "ăn" → Cách sửa: dòng 92–97 và 104 đã sửa (xác nhận khi xem lại bảng); dòng 107 lặp lại lỗi thiếu "ở" (`châu Úc` phải là `ở châu Úc`), đã sửa.
- **Bug 4 — Cảnh báo `VIRTUAL_ENV=venv` không khớp với `.venv`:** Quan sát: `uv run` in cảnh báo `VIRTUAL_ENV=venv does not match the project environment path .venv and will be ignored` → Nguyên nhân thật sự: shell vẫn đang kích hoạt môi trường cũ (tạo bằng `venv`, prompt hiện `(venv)`), trong khi `uv` dùng `.venv` của dự án; `uv` bỏ qua môi trường cũ nên kết quả vẫn đúng → Cách sửa: chạy `deactivate`; từ nay chạy lệnh bằng `uv run`, không `activate`. Đã chạy `deactivate`.
- **Bug 5 — Chỉ số "% âm tiết là 1 token" bị tính thấp:** Quan sát: danh sách "âm tiết bị cắt vụn" có `hỏi`, `hãy`, `bạn`, trong khi câu mẫu cho thấy `Hỏi` là một token → Nguyên nhân thật sự: phép đo thử " " + âm tiết viết thường, nhưng các chữ này trong dữ liệu đứng đầu câu, viết hoa và không có dấu cách phía trước → Cách sửa: coi 82% và 88% là cận dưới; nếu cần số chính xác thì đo trên token thật của từng chữ trong văn bản. Chưa sửa script. Nguyên nhân đã xác nhận bằng cách mã hóa thử hai câu (xem mục Quan sát).
- **Bug 6 — `prepare_data.py` báo `AttributeError: 'str' object has no attribute 'write'`:** Quan sát: lỗi ở dòng `stream.tofile(...)`, không tạo được `data/*.bin` → Nguyên nhân thật sự: `array.tofile()` nhận một file đã mở, không nhận đường dẫn dạng chuỗi; lỗi nằm ở đoạn mã do Claude đưa → Cách sửa: mở file ở chế độ `wb` rồi gọi `stream.tofile(f)`. Phần ghi file đã thử bằng tokenizer giả; chưa chạy với tokenizer thật.
- **Bug 7 — Dòng 98 có chủ thể `con rơi` thay vì `con dơi`:** Quan sát: "con rơi" trong tiếng Việt là cách gọi con ngoài giá thú, không phải con vật; nhóm `bản thân` cũng không đúng → Nguyên nhân thật sự: nhiều khả năng gõ nhầm d/r (người dùng chưa xác nhận trực tiếp ý định, nhưng đã chạy lệnh sửa) → Cách sửa: đổi thành `98,nơi sống,con dơi,sống ở đâu,trong hang`.

**Quan sát (không phải bug):**

- Mỗi sự thật sinh khoảng 33 cặp chỉ khác nhau ở mẫu hỏi hoặc mẫu đáp (3.147 trừ khoảng 150 cặp ngoài phạm vi, chia 90), nên dữ liệu có độ lặp cao và tập val (cắt ngẫu nhiên từ train) chưa đo được tổng quát hóa. Chưa sửa; so sánh `--max-per-fact` 20 và 80 ở tuần 3.
- Tỉ lệ token/âm tiết không thể về 1,0 vì mẫu số chỉ đếm âm tiết, còn tử số có cả dấu câu và xuống dòng (đã xác nhận trên câu mẫu: 13 âm tiết thành 18 token, trong đó 5 token là dấu câu và xuống dòng). Chỉ so sánh tương đối giữa các cỡ vocab.
- Token đổi theo dấu cách đứng trước và chữ hoa (đã xác nhận bằng `tokenizer.json`): "Con mèo kêu meo meo." thành 7 token (`C`, `on`, ...) vì "Con" đứng đầu chuỗi không có dấu cách; "Đáp: Con mèo kêu meo meo." thành 8 token, trong đó ` Con` là một token. Vì vậy khi hỏi mô hình phải viết đúng khuôn lúc train (`Hỏi: ...\nĐáp:`).
- Vocab 1024 và 2048 gần như cho cùng hiệu quả (1,34 và 1,33), vì dữ liệu chỉ có 488 "chữ" khác nhau nên BPE bão hòa quanh 1.068 token (1.057 sau khi đổi bảng).

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Dòng 96 là ý kiến chứ không phải sự thật, chưa quyết định (đã xác định là id 95 của `facts.csv`, xem tồn đọng giai đoạn 2).
- Khi viết script đánh giá và cho mô hình sinh câu trả lời, luôn ghép câu hỏi đúng khuôn `Hỏi: ...\nĐáp:`.
- Nếu sửa hoặc xóa dòng trong `facts.csv` sau này: chạy lại theo thứ tự `gen_data.py`, `build_tokenizer.py`, `prepare_data.py` (tokenizer và các file `.bin` phụ thuộc vào dữ liệu train).
- Mở rộng template (nếu quyết định làm), thí nghiệm `--max-per-fact` 20 và 80, baseline tra bảng: để tuần 3.
- Tùy chọn: thêm kiểm tra số cột vào `gen_data.py` để bắt lỗi như Bug 2.

## Giai đoạn 2 — Mô hình và huấn luyện (tuần 2)

**Mục tiêu giai đoạn:** Dựng mô hình ngôn ngữ nhỏ, tự viết vòng lặp huấn luyện, chạy thử trên máy rồi huấn luyện trên Colab, và đo bằng test A, B, C. **Kết quả:** đạt điều kiện kết thúc (có checkpoint huấn luyện trên Colab cùng ba con số); hoàn thành ngày 6/11, sớm hơn mốc 11/11.

**Đã làm:**

- Đọc mã tham khảo của thầy (workshop của Sebastian Raschka, mã nguồn ghi giấy phép Apache 2.0): `README.md`, `requirements.txt`, `03.ipynb`, `04.ipynb` và hai file `supplementary.py` (bản của phần 03 và bản của phần 04). Nội dung: mô hình kiểu GPT-2 (embedding token và vị trí, khối Transformer chuẩn hóa trước, attention nhiều đầu có mặt nạ nhân quả, feed-forward GELU), vòng lặp huấn luyện đơn giản (AdamW lr 4e-4, weight decay 0,1; không có warmup, cắt gradient hay mixed precision), dữ liệu là một truyện ngắn 5.145 token chia 90/10 theo vị trí, sinh chữ bằng argmax. Notebook 04 tự ghi nhận mô hình bị overfit vì dữ liệu quá nhỏ.
- Cài PyTorch bản CPU bằng `uv` (kho `pytorch-cpu` khai báo trong `pyproject.toml`): `torch` 2.15.0+cpu, `torch.cuda.is_available()` = False. Môi trường dự án `.venv` dùng Python 3.12.
- Thêm `numpy` 2.5.4 bằng `uv add numpy`; import `torch` không còn cảnh báo và việc chuyển tensor sang mảng numpy chạy được.
- `model.py` (`GPTModel`, tự viết kiểu workshop): emb_dim 128, 4 lớp, 4 đầu, ngữ cảnh 64, dropout 0,1, không bias ở q/k/v, buộc trọng số embedding. **935.296 tham số**, khớp phép tính tay (embedding 135.296 + vị trí 8.192 + 4 khối × 197.888 + chuẩn hóa cuối 256); hình dạng logits `(1, 64, 1057)` đúng mong đợi.
- Đo `train.bin`: 3.116 cặp, dài nhất 48 token, trung bình 22,5, không cặp nào dài hơn 64. Loss ban đầu 7,028 so với ln(1057) = 6,963.
- `dataset.py` (bộ nạp dữ liệu): đoạn train lấy ở vị trí bắt đầu ngẫu nhiên, val chia đoạn không chồng (57 đoạn, phủ 3.648/3.694 token). Đã kiểm tra: số token 70.258 và 3.694 khớp bản ghi, id lớn nhất 1056 nhỏ hơn 1057, đích đúng bằng đầu vào dịch một token, chữ giải mã ra đọc được tiếng Việt.
- Học thuộc một batch (`overfit_batch.py`): kiểm tra mặt nạ nhân quả đạt (đổi token từ vị trí 40 trở đi thì đầu ra trước vị trí 40 không đổi); loss 7,03 xuống 0,016 sau 300 bước, đoán đúng 99,6%.
- `train.py` v1: AdamW lr 1e-3 (betas 0,9/0,95), weight decay 0,1 chỉ cho ma trận, warmup 5% số bước rồi giảm cosine về 10%, cắt gradient 1,0, batch 32. Chạy 3.000 bước trên CPU (khoảng 0,2 giây mỗi bước, 616 giây): val thấp nhất 0,4540 ở bước 1200 rồi tăng đến 0,5246, trong khi train giảm đến 0,339.
- `generate.py` (chọn token xác suất cao nhất, dừng ở `</s>`, ghép đúng khuôn `Hỏi: ...\nĐáp:`; có kiểm tra khuôn prompt khớp huấn luyện) trả lời đúng các câu đã học ở cả hai checkpoint. `score.py` chấm test A, B, C kèm khoảng tin cậy 95%. Kết quả hai checkpoint của lần chạy CPU (bộ chấm cũ, số từ chối, bịa và C đếm thiếu; điểm A không đổi): bước 1200 có A 26,5% (80/302), B-unseen đúng 0/80 (từ chối 42, bịa 38), C 15/15; bước 3000 có A 57,3% (173/302), B-unseen đúng 0/80 (từ chối 18, bịa 62), C 15/15.
- `metrics.py` (chấm theo lô, dùng chung với `train.py`): chấm lại checkpoint bước 3000 cho đúng kết quả của `score.py` cũ (173/302; 0/18/62; 15/15; cả hai cùng dùng bộ chấm cũ), nên các điểm A do `train.py` tính đáng tin.
- `train.py` v2: chấm A/B/C mỗi 500 bước, lưu `ckpt_bestA.pt` và mốc định kỳ `ckpt_stepN.pt`, hỗ trợ fp16 + GradScaler trên GPU. Chạy trên Colab (GPU; loại GPU thực tế chưa ghi lại), 10.000 bước hết 304 giây gồm 21 lần chấm (khoảng 0,03 giây mỗi bước, nhanh hơn CPU khoảng 7 lần), checkpoint lưu lên Drive.
- **Kết quả cuối ở bước 10.000** (`ckpt_last` = `ckpt_bestA` = `ckpt_step10000`): val 0,8429 (thấp nhất 0,4592 ở bước 1250), train 0,1601; **A 93,7% (283/302); B-unseen đúng 0/80, từ chối 12, bịa 68; C 15/15** (sau khi sửa lỗi nhận diện từ chối, xem Bug 12; log Colab gốc ghi 11/69 và C 14/15 do bộ chấm cũ). Điểm do bộ chấm của `train.py` tính trên GPU; đã chấm lại `ckpt_step10000.pt` bằng `metrics.py` trên CPU máy cá nhân: A 283/302 khớp tuyệt đối, các số từ chối, bịa và C khớp với bộ chấm cũ trước khi sửa (11/69; 14/15).

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** cách dựng mô hình: (A) tự viết kiểu workshop của thầy (GPT-2) có buộc trọng số embedding đầu vào và đầu ra; (B) dùng lớp Llama của `transformers`; (C) tự viết kiến trúc kiểu Llama (RMSNorm, RoPE, SwiGLU).
- **Đã chọn:** A (Claude đề xuất; người dùng chấp nhận bằng cách chạy tiếp lệnh kiểm tra).
- **Vì sao:** mục đích là học mà không tốn nhiều thời gian; mã của thầy có sẵn nên A vừa nhanh vừa học được kiến trúc. Đánh đổi: nếu muốn chạy bằng Ollama thì phải tự chuyển định dạng; với tokenizer riêng có nguy cơ llama.cpp không nhận ra kiểu pre-tokenizer (theo hiểu biết của Claude, chưa kiểm chứng). Phương án dự phòng: chạy mô hình bằng `generate.py` trên CPU.

- **Vấn đề:** độ dài ngữ cảnh.
- **Đã chọn:** 64.
- **Vì sao:** cặp dài nhất 48 token, không cặp nào vượt 64. Đánh đổi: dư 16 vị trí, chi phí không đáng kể ở cỡ mô hình này.

- **Vấn đề:** cách cắt đoạn để huấn luyện.
- **Đã chọn:** train lấy vị trí bắt đầu ngẫu nhiên; val chia đoạn không chồng, bỏ phần dư 46 token.
- **Vì sao:** đơn giản, mỗi lượt thấy các điểm cắt khác nhau. Đánh đổi: lúc hỏi thật câu luôn bắt đầu ở vị trí 0 còn lúc học thì ở vị trí bất kỳ; mô hình vẫn trả lời đúng nên chưa thành vấn đề. Nếu gặp lỗi lạ khi sinh câu, thử cho đoạn train luôn bắt đầu tại `<s>`.

- **Vấn đề:** chọn checkpoint theo thước đo nào.
- **Đã chọn:** theo điểm A (kèm theo dõi bịa ở B và điểm C), không theo val loss.
- **Vì sao:** xem Bug 9. Đánh đổi: A dao động mạnh giữa các mốc (xem mục Quan sát), nên "A cao nhất qua nhiều mốc" là số lạc quan; ở lần Colab này mốc cao nhất trùng mốc cuối nên không bị ảnh hưởng.

- **Vấn đề:** cách chấm test A, B, C.
- **Đã chọn:** câu sinh ra có chứa `value` (khớp theo ranh giới từ, chuẩn hóa NFC); B chỉ tính dòng `unseen` và chia thành đúng, từ chối, bịa; C đúng khi câu sinh ra là một lời từ chối; thêm khoảng tin cậy 95% (Wilson).
- **Vì sao:** chấm tự động, không phụ thuộc người chấm. Đánh đổi: A là cận trên (value có thể xuất hiện trong câu sai); "bịa" gồm cả chuỗi chữ lộn xộn; test C chỉ 15 dòng nên khoảng tin cậy rộng (15/15 vẫn chỉ cho cận dưới 79,6%).

- **Vấn đề:** chạy lần huấn luyện đầu ở đâu.
- **Đã chọn:** kiểm tra mọi thứ trên CPU máy cá nhân trước, rồi mới lên Colab.
- **Vì sao:** lỗi dễ thấy và sửa ở máy cá nhân; Colab chỉ nhận mã đã chạy đúng. Đánh đổi: tốn thêm một vòng chạy; bù lại Colab nhanh hơn khoảng 7 lần nên 10.000 bước chỉ mất khoảng 5 phút.

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 8 — Cảnh báo `Failed to initialize NumPy: No module named 'numpy'` khi import `torch`:** Quan sát: dòng `UserWarning` xuất hiện nhưng `torch` vẫn chạy và in đúng phiên bản → Nguyên nhân thật sự: `torch` không tự kéo theo `numpy` và môi trường `.venv` chưa có `numpy`, nên phần chuyển đổi giữa tensor và mảng numpy không khởi tạo được → Cách sửa: `uv add numpy`. Đã xác nhận: cài `numpy` 2.5.4, không còn cảnh báo.
- **Bug 9 — Chọn checkpoint theo val loss chọn nhầm:** Quan sát: `ckpt_best` (val thấp nhất, bước 1200) có A 26,5%, thấp hơn `ckpt_last` (bước 3000, val cao hơn) có A 57,3%; trên Colab val tăng từ 0,459 lên 0,843 trong khi A tăng từ khoảng 82% lên 93,7% → Nguyên nhân thật sự: val được cắt ngẫu nhiên từ chính kho dữ liệu của train và chấm mọi token của cả cặp; phần lớn là khuôn câu (nhiều biến thể ngẫu nhiên), phần sự thật chỉ 1–3 token trong khoảng 22, nên val loss gần như mù với việc nhớ sự thật → Cách sửa: `train.py` v2 chấm A/B/C định kỳ, lưu `ckpt_bestA.pt` và các mốc `ckpt_stepN.pt`; không chọn checkpoint theo val loss nữa.
- **Bug 10 — Cột `grad_norm` trong log luôn trống:** Quan sát: cột `grad_norm` của `logs/train_log.csv` rỗng ở lần chạy 300 và 3.000 bước → Nguyên nhân thật sự: `train.py` v1 ghi chuỗi rỗng thay vì giá trị; lỗi nằm ở đoạn mã do Claude đưa → Cách sửa: v2 ghi giá trị `grad_norm` của bước trước đó. Chưa mở file CSV của lần chạy Colab để xác nhận cột có số.
- **Bug 11 — Notebook Colab mở bằng máy không có GPU:** Quan sát: `torch.cuda.is_available()` ra `False`, phiên bản `2.11.0+cpu` → Nguyên nhân thật sự: loại runtime mặc định của Colab là CPU → Cách sửa: Runtime, Change runtime type, chọn GPU. Đã xác nhận: log huấn luyện ghi `thiết bị: cuda | fp16: True`.
- **Bug 12 — Bộ chấm bỏ sót hai trong bốn cách từ chối:** Quan sát: câu "Thủ đô của nước Nhật là gì?" được đáp "Mình chưa học cái này." nhưng bị tính là không từ chối (C 14/15); điểm C trong log Colab từng rơi xuống 3–5/15 ở vài mốc → Nguyên nhân thật sự: dữ liệu train có bốn cách từ chối gần như đều nhau (36–40 cặp mỗi cách: "Mình chưa học cái này.", "Mình không biết.", "Mình chưa biết câu này.", "Mình chưa biết."), còn `is_refusal` chỉ nhận câu có "chưa biết" hoặc trùng đáp án mẫu của test C; mô hình chọn cách nào là tùy từng mốc nên điểm C lật qua lại (giả thuyết; các mốc 1500, 2500, 3500, 5500 không có checkpoint nên không chấm lại được) → Cách sửa: `is_refusal` nhận cả "chưa biết", "không biết", "chưa học". Chấm lại mười mốc đã lưu: C đạt 15/15 ở cả mười mốc (log cũ ghi 12/15 ở bước 7000 và 14/15 ở bước 8000–10000); điểm A không đổi; số từ chối và bịa ở B-unseen đổi nhẹ (bước cuối 12 và 68 thay cho 11 và 69). Các mốc dao động còn lại cần chạy lại để kiểm chứng (việc 1 của giai đoạn 3).
- **Bug 13 — Lịch sử git đã đẩy lên GitHub chứa các dòng cá nhân cũ:** Quan sát: `git push` bị từ chối (non-fast-forward) và `git log origin/main -p` có 8 dòng khớp chuỗi cá nhân; lịch sử cục bộ (5 commit) cũng chứa các dòng cá nhân cũ của `facts.csv` → Nguyên nhân thật sự: trước khi tách dòng cá nhân, các commit cũ đã được đẩy lên một kho GitHub có sẵn; Claude đã giả định chưa có remote vì trường `repo:` còn trống mà không chạy `git remote -v` → Cách sửa: đổi kho sang Private (kho chưa có fork), tạo nhánh `main` sạch một commit (đủ `data/*.bin`, `test_*.jsonl`, `pyproject.toml`, `uv.lock`) và ghi đè lịch sử trên GitHub bằng `git push --force-with-lease`; lịch sử cũ chỉ còn ở nhánh `old-history-LOCAL-ONLY` trên máy cá nhân. Hạn chế còn lại: sau khi ghi đè, các commit cũ có thể vẫn truy cập được bằng mã SHA trong kho (theo tài liệu GitHub), nên **trước khi chuyển kho sang Public (ví dụ để đưa vào CV) phải xóa kho và tạo lại**.

**Quan sát (không phải bug):**

- Val loss và điểm A đi ngược chiều ở lần chạy Colab: val tăng từ 0,459 (bước 1250) lên 0,843 (bước 10.000) còn train giảm đến 0,160, thấp hơn mức sàn ước lượng thô khoảng 0,36 (ln 3.116 chia 22,5), nghĩa là mô hình đã thuộc cả thứ tự các cặp trong chuỗi train cố định; trong lúc đó A tăng. Val loss đo khuôn câu nhiều hơn sự thật, nên chỉ dùng để phát hiện lỗi huấn luyện, không để chọn checkpoint.
- Điểm A dao động mạnh giữa các mốc cách nhau 500 bước: từ bước 1000 đến 7500 nằm trong khoảng 67,5% đến 86,8%; điểm C (theo bộ chấm cũ, nhiều khả năng do Bug 12) rơi xuống 3–5/15 ở bước 1500, 2500, 3500 và 5500 rồi lại 15/15. Từ bước 8000 (lr từ 2e-4 trở xuống) A nằm trong 88,4–93,7% và C trong 13–14/15. Các số từ chối và C của log Colab bị đếm thiếu (Bug 12) nên chưa dùng được để giải thích dao động. Sau khi chấm lại mười mốc đã lưu, C luôn 15/15 còn A vẫn dao động thật (ví dụ 67,5% ở bước 4000 với 22 câu từ chối ở B-unseen), nên giả thuyết "công tắc từ chối" từng ghi ở đây không còn căn cứ; chưa có giải thích cho dao động của A.
- Hai lần chạy (CPU 3.000 bước và Colab 10.000 bước) có loss gần như trùng nhau nhưng A rất khác: 26,5% ở bước 1200 so với 81,8% ở bước 1000; 57,3% ở cuối lịch 3.000 bước so với 86,1% ở bước 3000 của lịch 10.000 bước. Chưa giải thích được. Các khác biệt giữa hai lần chạy: độ dài lịch learning rate, fp16 và GPU, thứ tự batch và dropout ngẫu nhiên. Vì vậy một lần chạy mỗi cấu hình chưa đủ để kết luận khi so sánh các thí nghiệm. Bốn seed ở giai đoạn 3 cho A cuối từ 57,0% đến 93,7%, nên chênh lệch giữa hai lần chạy này phù hợp với dao động giữa các lần chạy (chưa chứng minh).
- B-unseen đúng 0/80 ở mọi mốc vì giá trị đáp án chưa từng xuất hiện trong train; thước đo có ý nghĩa là bịa so với từ chối. Với bộ chấm đã sửa, ở mười mốc đã lưu (bước 1000 đến 10.000) mô hình bịa 58–76 câu trên 80 và từ chối 4–22; ở bước cuối bịa 68/80 (85%). Dữ liệu chỉ dạy từ chối câu ngoài phạm vi (test C) chứ không dạy từ chối khi thiếu sự thật trong phạm vi. Ở lần chạy CPU, mô hình thường trả lời bằng thuộc tính khác của cùng chủ thể (ví dụ "Heo ăn cám" cho câu hỏi tiếng kêu của heo).
- Ở lần chạy CPU, các câu A sai thường là từ chối nhầm câu hỏi mới (ví dụ "Nghe thử xem, con mèo kêu như thế nào?" ra "Mình chưa biết."), trong khi cùng câu hỏi ở dạng đã học thì đáp đúng.
- Ở checkpoint cuối, 13 trong 19 câu A sai thuộc thuộc tính "ăn gì" và đều thuộc đúng hai mẫu hỏi: "Nói xem, … ăn gì nhỉ?" và "Bữa ăn của … có gì?"; cả hai mẫu không xuất hiện trong train (đếm được 0 dòng). Mọi thuộc tính khác đạt từ 95% trở lên. Nghĩa là mô hình tổng quát hóa được nhiều mẫu hỏi mới nhưng yếu ở mẫu xa khuôn đã học (kèm lỗi nhầm chủ thể, ví dụ "Bữa ăn của chó có gì?" ra "Cỏ màu xanh.").
- "Bịa 80/80" ở bước 0 là do định nghĩa (mọi câu không đúng và không từ chối đều tính là bịa, kể cả chữ lộn xộn), không phải mô hình đang bịa; chỉ đọc cột này khi mô hình đã nói được câu hoàn chỉnh.

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

- Chưa giải thích được vì sao hai lần chạy cho A khác nhau nhiều (xem Quan sát); cần chạy nhiều seed (việc 1 của giai đoạn 3).
- GPU của Colab là Tesla T4 (dòng `GPU:` đầu log huấn luyện); cột `grad_norm` đã xác nhận có giá trị ở các log mới (40 giá trị mỗi log).
- Chưa thử mô hình với câu hỏi cách hỏi hoàn toàn khác khuôn dữ liệu (kiểm tra thủ công bằng `generate.py`).
- Đã gom cấu hình vào `config.py` (dùng trong `train.py`) và `score.py` v2 dùng chung bộ chấm với `metrics.py`; `check_model.py` và `overfit_batch.py` vẫn còn bản `cfg` riêng (tùy chọn sửa).
- `.gitignore` đã thêm `checkpoints/`, `logs/`, `*.pt`, `*.tar.gz` và đã kiểm tra không có file nhạy cảm bị git theo dõi; lịch sử git cũ đã được phát hiện chứa dòng cá nhân và đã đẩy lên GitHub từ trước (Bug 13); đã đổi kho sang Private, đang ghi đè bằng nhánh sạch; trước khi mở Public phải xóa và tạo lại kho.
- "Dòng 96" trong ghi chú giai đoạn 1 thật ra là id 95 của `facts.csv` ("đến dịp nghỉ lễ 30/4 và 1/5", nên làm gì: "dành thời gian cho bản thân"), một lời khuyên chủ quan; đề xuất của Claude là giữ như hạn chế có chủ đích vì sửa có thể làm đổi tập B giữ lại (chưa kiểm tra `gen_data.py`), chờ người dùng xác nhận.
- Chưa chấm lại hai checkpoint của lần chạy CPU (bước 1200 và 3000) bằng bộ chấm đã sửa (điểm A của chúng không đổi); các mốc C rơi ở bước 1500, 2500, 3500, 5500 chưa kiểm chứng được vì không có checkpoint.
- Từ giai đoạn trước, còn lại: mở rộng template, thí nghiệm `--max-per-fact` 20 và 80, baseline tra bảng và xuất GGUF/Ollama (đã xếp vào giai đoạn 3).

## Giai đoạn 3 — Thí nghiệm và xuất mô hình (tuần 3)

**Mục tiêu giai đoạn:** Biết kết quả của giai đoạn 2 đáng tin đến đâu (nhiễu giữa các lần chạy), cải thiện hai điểm yếu đã thấy (bịa khi thiếu sự thật, từ chối không ổn định), có baseline tra bảng để so sánh, và chạy được mô hình ở máy cá nhân.

**Đã làm:**

- Kho GitHub `wickyhien18/ViLM` (Private, chưa có fork): lịch sử cũ trên GitHub chứa dòng cá nhân (Bug 13) nên được ghi đè bằng nhánh `main` sạch một commit; Colab lấy mã bằng `git clone` và `git pull` với token chỉ đọc (fine-grained, hết hạn sau 30 ngày), kết quả lưu trên Drive.
- Việc 1 (đo nhiễu): bốn lần chạy 10.000 bước, cùng cấu hình, seed 123, 1, 2, 3, với bộ chấm đã sửa (Bug 12):

| Seed | A cuối | A trong cửa sổ cuối (thấp, trung bình, cao) | B-unseen từ chối | B-unseen bịa | C |
|---|---|---|---|---|---|
| 123 | 93,7% | 88,4, 90,5, 93,7 | 12 | 68 | 15/15 |
| 1 | 80,5% | 70,9, 77,4, 81,8 | 12 | 68 | 15/15 |
| 2 | 83,1% | 78,5, 82,6, 86,4 | 19 | 61 | 15/15 |
| 3 | 57,0% | 56,0, 57,7, 59,9 | 31 | 49 | 15/15 |

  Tổng hợp (trung bình ± độ lệch mẫu, n = 4): A 78,6 ± 15,5 (từ 57,0 đến 93,7); bịa 61,5 ± 9,0 (49 đến 68); từ chối 18,5 ± 9,0 (12 đến 31); C 15,0 ± 0,0.
- Chuẩn bị dữ liệu v2 (đã viết và kiểm tra trên bảng giả, chưa chạy trên dữ liệu thật): `gen_data.py --v2` thêm câu từ chối viết theo các mẫu hỏi quen với chủ thể chưa có trong bảng (mặc định 60 cặp mỗi thuộc tính; các nhóm chủ thể lạ lấy từ danh sách trong mã, tự bỏ chủ thể đã có trong `facts.csv` và cả chủ thể chứa hoặc nằm trong một chủ thể của bảng theo ranh giới từ, ví dụ "con chim cánh cụt" so với "con chim", vì kết quả mong đợi của câu đó không còn rõ ràng) và tạo `test_D` (96 dòng: chủ thể lạ không dùng ở train, mẫu hỏi quen hoặc mẫu giữ lại; mong đợi từ chối). RNG riêng nên test_A, test_B, test_C và `facts_split.csv` giữ nguyên: trên bảng giả, không có `--v2` cho đầu ra giống hệt bản gốc, có `--v2` thì A, B, C vẫn giống hệt. `metrics.py`, `train.py`, `score.py`, `summarize_seeds.py` nhận thêm cột `A_refused` (số câu A bị từ chối) và `D`. Quy trình: nhánh git `data-v2`; `gen_data.py --v2`; `build_tokenizer.py`; chép `tokenizers/tok_2048.json` thành `tokenizer.json`; `prepare_data.py`; kiểm tra `git diff` của `data/test_A/B/C.jsonl` phải không đổi; rồi chạy bốn seed trên Colab và so A, D, từ chối nhầm với kết quả v1 (A 78,6 ± 15,5). Lưu ý: dữ liệu v2 đổi kích thước từ vựng nên checkpoint v1 không nạp được cùng tokenizer v2 (số tham số = 800.000 + 128 × kích thước từ vựng); muốn chấm lại checkpoint v1 phải dùng nhánh `main`. `build_tokenizer.py` (chỉ học từ `train.jsonl`, không dùng test) và `prepare_data.py` không cần sửa cho v2.

- Đã đẩy nhánh `data-v2` lên GitHub và gộp vào `main` (9/11). Từ giờ `main` chứa dữ liệu v2 (kích thước từ vựng 1122), nên checkpoint v1 (từ vựng 1057) không nạp được trên `main`; dữ liệu và mã v1 còn ở commit `d18aaee` (dự kiến gắn thẻ `data-v1` để tái lập kết quả v1). Nhánh `old-history-LOCAL-ONLY` chỉ nằm ở máy cá nhân và không bao giờ được đẩy.
- Bỏ `train_v1.py`: `train.py` (v3) với tham số mặc định (không `--score-every`, không `--amp`) chạy y hệt v1, đã kiểm chứng ở giai đoạn 2 bằng lần chạy thử 200 bước trùng từng chữ số với bản v1 (val 2,1106 ở bước 100 và 1,6019 ở bước 200); từ nay chỉ giữ một file `train.py`.
- Kiểm tra dữ liệu v2 trên máy cá nhân (đạt): `gen_data.py --v2` tạo 417 cặp từ chối mới (417 câu khác nhau), tỉ lệ câu từ chối trong train 16,1%, test_D 96 dòng (mẫu quen 69, mẫu lạ 27); `git diff` của test_A, test_B, test_C rỗng; tokenizer cỡ 2048 có kích thước thật 1122; train 3.533 cặp (78.466 token), val 185 cặp (4.112 token); mô hình 943.616 tham số (= 800.000 + 128 × 1122), cặp dài nhất 48 token, không cặp nào vượt 64; chạy thử 200 bước ghi đủ 13 cột log (có `A_refused`, `D`) và `score.py` in được test_D chia theo mẫu hỏi. `baseline.py` trên dữ liệu v2: A 91,1% (275/302, 27 câu sai đều là từ chối nhầm) với bộ đoán thuộc tính Naive Bayes; sau khi sửa chuẩn hóa loại từ "quả" A lại giảm còn 88,4% (267/302; 35 từ chối nhầm, 0 trả lời sai), nghĩa là nhận định ban đầu của Claude (lỗi do thiếu loại từ) là sai: lỗi nằm ở việc đoán sai thuộc tính trên các mẫu hỏi giữ lại (màu sắc, vị, công dụng thấp nhất; ví dụ "Con quạ thường có màu nào?" đã bị từ chối dù chủ thể có loại từ "con"); đã thêm từ khóa lấy từ tên thuộc tính làm bước đầu, Naive Bayes làm dự phòng (chưa chạy trên dữ liệu thật), B-unseen từ chối 80/80, B-seen bịa 8/56, C 15/15, D 96/96. Tái lập đã xác nhận: ba lần chạy `gen_data.py --v2` với `PYTHONHASHSEED` 0, 1, 2 cho mã băm giống hệt ở `train.jsonl`, `val.jsonl` và `test_D.jsonl`; thống kê test_D đổi trước đó (mẫu quen 76 và lạ 20, rồi 69 và 27) là do cập nhật bộ lọc chủ thể giữa các lần chạy.

- Việc 3 (dữ liệu v2) đã chạy trên Colab (Tesla T4, fp16, 10.000 bước, bốn seed 123, 1, 2, 3, sau khi gộp `data-v2` vào `main`):

| Seed | A cuối | A từ chối nhầm (trên 302) | A trong cửa sổ cuối (thấp, trung bình, cao) | D (trên 96) | B-unseen từ chối | B-unseen bịa | C |
|---|---|---|---|---|---|---|---|
| 123 | 46,4% | 158 | 36,1, 46,2, 53,0 | 86 | 39 | 41 | 15/15 |
| 1 | 93,4% | 15 | 92,1, 93,6, 94,7 | 69 | 14 | 66 | 15/15 |
| 2 | 78,5% | 60 | 72,2, 76,8, 80,8 | 84 | 28 | 52 | 15/15 |
| 3 | 87,7% | 25 | 84,4, 86,5, 87,7 | 66 | 17 | 63 | 15/15 |

  Tổng hợp (n = 4): A 76,5 ± 21,0 (46,4 đến 93,4); A từ chối nhầm 64,5 ± 65,3 (15 đến 158); D 76,2 ± 10,2 (66 đến 86); B-unseen bịa 55,5 ± 11,4 (41 đến 66) và từ chối 24,5 ± 11,4; C 15,0 ± 0,0. So với v1: A 78,6 ± 15,5, bịa 61,5 ± 9,0.

**Quan sát (không phải bug):**

- Dữ liệu v2 không cải thiện điểm A và không thu hẹp độ lệch: A trung bình 76,5 so với 78,6 ở v1 (chênh lệch nhỏ hơn nhiều so với độ lệch), độ lệch tăng từ 15,5 lên 21,0; số câu A bị từ chối nhầm dao động từ 15 đến 158; bịa ở B-unseen giảm từ 61,5 xuống 55,5 nhưng nằm trong nhiễu. Dự đoán của Claude trước khi chạy (A cao hơn và độ lệch nhỏ hơn) đã sai.
- Seed 123 từ tốt nhất ở v1 (A 93,7%) thành kém nhất ở v2 (A 46,4%): xếp hạng seed không chuyển giữa các phiên bản dữ liệu (khởi tạo cũng khác vì kích thước từ vựng đổi từ 1057 sang 1122), nên mỗi lần chạy giống một lần bốc thăm có phương sai lớn.
- A và D đi ngược chiều: (A từ chối nhầm, D) của seed 123, 2, 3, 1 lần lượt là (158, 86), (60, 84), (25, 66), (15, 69); trong lần chạy seed 1, A tăng từ 86,8% (bước 1000, D 84/96) lên 95,4% (bước 7500, D 68/96). Mô hình có một mức độ từ chối chung mà việc thêm câu từ chối theo mẫu quen chưa tách thành hai quyết định (chủ thể lạ so với cách hỏi lạ). Ở bước cuối D chỉ đạt 66 đến 86/96 (trung bình 79%), tức mô hình vẫn trả lời nhiều câu về chủ thể lạ.
- Giả thuyết mới (chưa kiểm chứng): train v2 chỉ có tối đa 43 chủ thể lạ (danh sách trong mã, trừ chủ thể đã có trong bảng hoặc mơ hồ), mô hình có thể ghi nhớ chúng thay vì học "không thuộc tập chủ thể đã biết", nên với chủ thể lạ mới (test D) nó mặc định trả lời. Chưa kiểm tra D chia theo mẫu quen và mẫu lạ (`score.py`), và chưa kiểm tra xác suất mở đầu bằng "Mình" có tách hai nhóm câu không (`refusal_auc.py`, đã viết, chưa chạy).
- Cửa sổ cuối của A ở v2 dao động rộng hơn v1 (ví dụ seed 123: 36,1% đến 53,0%).
- A cuối khác nhau rất nhiều giữa các seed (57,0% đến 93,7%), lớn hơn nhiều so với dao động trong một lần chạy ở cuối lịch (cửa sổ cuối chỉ rộng khoảng 5 đến 11 điểm). Kết quả 93,7% ở giai đoạn 2 là lần chạy tốt nhất trong bốn; con số nên báo cáo là trung bình và khoảng giá trị, không phải 93,7%.
- Seed 3 ổn định ở mức thấp (56,0% đến 59,9%) nên đây không phải nhiễu theo từng mốc mà là mô hình hội tụ về một lời giải kém hơn; nguyên nhân chưa biết.
- Seed có A thấp nhất cũng từ chối nhiều nhất và bịa ít nhất (31 và 49); thứ hạng theo A và theo số từ chối gần như ngược chiều (trừ seed 1), nhưng mới có 4 điểm nên chỉ là xu hướng.
- Chẩn đoán bằng `score.py` ở bước cuối: seed 3 (A 57,0%) có 130 câu A sai, trong đó **124 là từ chối nhầm và chỉ 6 là trả lời sai** (95%); seed 123 (A 93,7%) có 19 câu sai gồm 8 từ chối nhầm và 11 trả lời sai. Lỗi của seed 3 rải ra mọi thuộc tính (ăn gì 20,0%, số chân 50,0%, công dụng 51,6%, màu sắc 60,7%, sống ở đâu 60,0%, vị 68,8%, tiếng kêu 72,5%; riêng nên làm gì 100%), nên A thấp chủ yếu do mô hình từ chối câu hỏi đúng phạm vi nhưng diễn đạt lạ, không phải do quên sự thật.
- Giả thuyết (chưa kiểm chứng): trong train, câu ngoài phạm vi khác câu trong phạm vi cả về nội dung lẫn cách diễn đạt, nên mô hình có thể dùng "cách hỏi quen hay lạ" làm tín hiệu để từ chối; các seed khác nhau ở chỗ dùng tín hiệu nào (nội dung hay cách hỏi). Cách kiểm tra: đo mức trùng cách diễn đạt giữa câu ngoài phạm vi và câu trong phạm vi trong `train.jsonl`. Cách sửa dự kiến: thêm câu ngoài phạm vi viết theo mẫu hỏi trong phạm vi (chủ thể chưa biết), và thêm bộ test D (chủ thể lạ, mẫu hỏi quen) để đo.
- Kết quả kiểm tra mức trùng cách diễn đạt: trong 151 cặp ngoài phạm vi của train chỉ 4 cặp (2,6%) có hai từ đầu trùng với một câu trong phạm vi, nên nội dung và cách hỏi của hai loại câu tách biệt gần như hoàn toàn (xấp xỉ thô); giả thuyết có thêm cơ sở nhưng chưa được chứng minh. Mẫu ngẫu nhiên 12 câu ngoài phạm vi có một câu xuất hiện hai lần, và đếm được chỉ có 40 câu ngoài phạm vi khác nhau trong 151 cặp (mỗi câu có nhiều cách từ chối); và test C có câu rất giống câu trong train ("Thủ đô của nước Nhật là gì?" so với "Thủ đô của nước Pháp là gì?"), nên C 15/15 là bằng chứng yếu cho khả năng từ chối.
- Lệnh `summarize_seeds.py --from-step 0` do Claude đề xuất là sai chỗ: nó tính cả bước 0 (mô hình chưa học) nên A thấp nhất bằng 0 và C thấp nhất bằng 0/15 ở cả bốn lần; phải dùng `--from-step 1000` để xem các mốc C dao động giữa chừng.
- C đạt 15/15 ở cả bốn lần chạy trong cửa sổ cuối (từ bước 8000); chưa kiểm tra giá trị nhỏ nhất của C trên toàn bộ lần chạy (cần chạy `summarize_seeds.py --from-step 0`).
- Hệ quả cho các thí nghiệm sau: với độ lệch cỡ 15 điểm thì ba seed chỉ phân biệt được các cấu hình khác nhau khoảng 30 điểm trở lên; muốn so sánh tinh hơn cần thêm seed hoặc giảm phương sai.

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

Việc cần làm trong giai đoạn 3, theo thứ tự (mốc: thứ Tư 18/11; ước tính của Claude khoảng 10–12 giờ, chưa chắc, vì số giờ mỗi tuần của người dùng chưa chốt; nếu thiếu giờ thì bỏ việc 5 trước):

0. [xong] Đồng bộ mã bằng GitHub: kiểm tra lịch sử git không chứa dữ liệu cá nhân, tạo kho riêng tư, đẩy mã; Colab lấy mã bằng `git clone` và `git pull` thay cho đóng gói và tải file. Xong khi: Colab chạy được `git pull` lấy mã mới nhất từ máy cá nhân. (khoảng 30 phút)
1. [xong, xem mục Đã làm] Đo nhiễu giữa các lần chạy (dùng bốn seed 123, 1, 2, 3 với bộ chấm đã sửa; cũng để kiểm chứng các mốc C dao động): thêm tham số `--seed` vào `train.py`, chạy cùng cấu hình 10.000 bước với ít nhất 3 seed. Xong khi: có A, bịa ở B-unseen và C (ở bước cuối và các mốc 8000–10000) dạng trung bình và độ lệch. Làm đầu tiên vì không so sánh được thí nghiệm nào nếu chưa biết nhiễu. (khoảng 1 giờ)
2. [đã viết `baseline.py`, chờ chạy trên dữ liệu thật] Baseline tra bảng: chương trình đoán chủ thể và thuộc tính từ câu hỏi (khớp chuỗi) rồi trả giá trị trong bảng train; tra không thấy thì từ chối. Chấm bằng cùng bộ chấm. Xong khi: có A, B, C của baseline đặt cạnh mô hình. Đây là phần "nói thẳng" của báo cáo. (khoảng 1–1,5 giờ)
3. [đã chạy bốn seed với dữ liệu v2: không cải thiện, xem mục Đã làm] Dạy mô hình từ chối khi thiếu sự thật trong phạm vi: thêm vào dữ liệu train các cặp "chủ thể đã biết nhưng thuộc tính chưa có" trả lời "Mình chưa biết.", giữ nguyên test B (không đưa vào train). Xong khi: bịa ở B-unseen giảm rõ so với mức nhiễu của việc 1, mà A không giảm quá mức nhiễu đó. **Cần người dùng quyết định trước:** muốn mô hình từ chối kiểu này hay chấp nhận nó đoán. Sau chẩn đoán hai seed, việc này gộp với việc 5: thêm cả câu ngoài phạm vi viết theo mẫu hỏi trong phạm vi và bộ test D; kỳ vọng giảm từ chối nhầm và thu hẹp độ lệch giữa các seed, cần kiểm chứng bằng nhiều seed. (khoảng 2–3 giờ)
4. [hạ ưu tiên: độ lệch 15 đến 21 điểm giữa các seed che mất hiệu ứng nhỏ] Thí nghiệm `--max-per-fact` 20 so với 80 (mỗi cấu hình 3 seed). Xong khi: có bảng A, bịa, C cho từng cấu hình kèm độ lệch. (khoảng 1,5 giờ)
5. (Tùy chọn) Mở rộng mẫu hỏi để tăng đa dạng cách hỏi, rồi chạy lại việc 1 trên dữ liệu mới. Xong khi: A trên bộ test giữ cố định không giảm và tốt hơn mức nhiễu. (khoảng 2 giờ)
6. Cải thiện bộ chấm: thêm tỉ lệ từ chối nhầm trên test A và phân loại lỗi (câu cụt, trả nhầm thuộc tính, từ chối nhầm); gom `score.py` vào `metrics.py`; gom `cfg` vào một chỗ. Xong khi: một bảng lỗi cho checkpoint cuối. (khoảng 45 phút)
7. Xuất mô hình: kiểm chứng llama.cpp có nhận kiến trúc GPT-2 tự viết và tokenizer riêng không (cần đọc tài liệu hiện hành, vì Claude không chắc), thử chuyển GGUF và chạy qua Ollama; nếu không được thì dùng `generate.py` làm phương án chạy local. Xong khi: một câu hỏi–đáp chạy qua Ollama, hoặc ghi rõ lý do dừng. Có rủi ro cao nên làm sau việc 1–4 và đặt hạn mức 3 giờ. (khoảng 2–3 giờ)

Ghi chú cho giai đoạn:

- Khi so sánh hai cấu hình, chỉ kết luận nếu chênh lệch lớn hơn độ lệch giữa các seed (việc 1).
- Chọn checkpoint ở cuối lịch learning rate (lr thấp), đừng chọn giữa chừng khi A và C còn dao động.
- Tuần 4 (báo cáo, slide, repo): gồm cả `.gitignore` cho `checkpoints/` và `logs/`, ghi nguồn mã của thầy (Raschka, Apache 2.0).

# Kiến thức đã áp dụng
- [[Train-Val-Test Split]]
- [[Data Leakage]]
- [[Unicode NFC Normalization]]
- [[Scaling Laws (Chinchilla)]]
- [[Overfitting và Underfitting]]
- [[Learning Rate Schedule (Warmup và Cosine)]]
- [[Mixed Precision Training (fp16)]]
- [[Causal Mask]]
- [[Weight Tying]]
