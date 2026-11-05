---
Hub: "[[MOC - AI & ML]]"
created: 2026-10-29
status: đang làm (giai đoạn 2 — mô hình và huấn luyện, vừa bắt đầu)
repo:
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
| Kiến trúc | Llama-architecture qua `transformers`, hoặc tự viết kiểu workshop của thầy — *chưa chốt* | `transformers` dễ xuất sang GGUF để chạy bằng Ollama; tự viết thì học được nhiều hơn nhưng phải chuyển định dạng khi xuất |
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

- Dòng 96 là ý kiến chứ không phải sự thật, chưa quyết định.
- Khi viết script đánh giá và cho mô hình sinh câu trả lời, luôn ghép câu hỏi đúng khuôn `Hỏi: ...\nĐáp:`.
- Nếu sửa hoặc xóa dòng trong `facts.csv` sau này: chạy lại theo thứ tự `gen_data.py`, `build_tokenizer.py`, `prepare_data.py` (tokenizer và các file `.bin` phụ thuộc vào dữ liệu train).
- Mở rộng template (nếu quyết định làm), thí nghiệm `--max-per-fact` 20 và 80, baseline tra bảng: để tuần 3.
- Tùy chọn: thêm kiểm tra số cột vào `gen_data.py` để bắt lỗi như Bug 2.

## Giai đoạn 2 — Mô hình và huấn luyện (tuần 2)

**Mục tiêu giai đoạn:** Dựng mô hình ngôn ngữ nhỏ, tự viết vòng lặp huấn luyện, chạy thử trên máy rồi huấn luyện trên Colab, và đo bằng test A, B, C.

**Đã làm:**

- Đọc mã tham khảo của thầy (workshop của Sebastian Raschka, mã nguồn ghi giấy phép Apache 2.0): `README.md`, `requirements.txt`, `03.ipynb`, `04.ipynb` và hai file `supplementary.py` (bản của phần 03 và bản của phần 04). Nội dung: mô hình kiểu GPT-2 (embedding token và vị trí, khối Transformer chuẩn hóa trước, attention nhiều đầu có mặt nạ nhân quả, feed-forward GELU), vòng lặp huấn luyện đơn giản (AdamW lr 4e-4, weight decay 0,1; không có warmup, cắt gradient hay mixed precision), dữ liệu là một truyện ngắn 5.145 token chia 90/10 theo vị trí, sinh chữ bằng argmax. Notebook 04 tự ghi nhận mô hình bị overfit vì dữ liệu quá nhỏ.
- Cài PyTorch bản CPU bằng `uv` (kho `pytorch-cpu` khai báo trong `pyproject.toml`): `torch` 2.15.0+cpu, `torch.cuda.is_available()` = False. Môi trường dự án `.venv` dùng Python 3.12.
- Thêm `numpy` 2.5.4 bằng `uv add numpy`; import `torch` không còn cảnh báo và việc chuyển tensor sang mảng numpy chạy được.

**Quyết định kỹ thuật (nếu có):**

- **Vấn đề:** cách dựng mô hình: (A) tự viết kiểu workshop của thầy (GPT-2) có buộc trọng số embedding đầu vào và đầu ra; (B) dùng lớp Llama của `transformers`; (C) tự viết kiến trúc kiểu Llama (RMSNorm, RoPE, SwiGLU).
- **Đề xuất của Claude (chờ người dùng xác nhận):** A.
- **Vì sao:** mục đích hiện tại là học mà không tốn nhiều thời gian. Mã của thầy có sẵn nên A vừa nhanh vừa học được kiến trúc; B nhanh hơn một chút nhưng học ít về bên trong; C tốn thêm thời gian cho phần chưa cần. Đánh đổi của A: nếu muốn chạy bằng Ollama thì phải tự chuyển định dạng; với tokenizer riêng có nguy cơ llama.cpp không nhận ra kiểu pre-tokenizer (theo hiểu biết của Claude, chưa kiểm chứng). Phương án dự phòng: chạy mô hình bằng script PyTorch trên CPU.

**Bug gặp phải (nếu có) — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 8 — Cảnh báo `Failed to initialize NumPy: No module named 'numpy'` khi import `torch`:** Quan sát: dòng `UserWarning` xuất hiện nhưng `torch` vẫn chạy và in đúng phiên bản → Nguyên nhân thật sự: `torch` không tự kéo theo `numpy` và môi trường `.venv` chưa có `numpy`, nên phần chuyển đổi giữa tensor và mảng numpy không khởi tạo được → Cách sửa: `uv add numpy`. Đã xác nhận: cài `numpy` 2.5.4, không còn cảnh báo.

**Việc còn tồn đọng từ giai đoạn này (nếu có):**

Việc cần làm trong giai đoạn 2, theo thứ tự (mốc: thứ Tư 11/11; ước tính của Claude khoảng 8 giờ):

1. Chốt cách dựng mô hình (xem quyết định bên trên). Xong khi: người dùng xác nhận.
2. Tạo `model.py` từ các khối của thầy (giữ dòng bản quyền): `GPTModel` có buộc trọng số, khởi tạo embedding với độ lệch chuẩn 0,02; cấu hình nhỏ (ví dụ emb_dim 128, 4 lớp, 4 đầu, ngữ cảnh 64). Xong khi: in được số tham số thật và hình dạng đầu ra đúng `(batch, độ dài, vocab)`. (khoảng 45 phút)
3. Đo độ dài thật của các cặp trong `train.bin` để chọn `context_length`. Xong khi: biết cặp dài nhất và số cặp dài hơn ngữ cảnh đã chọn. (khoảng 20 phút)
4. Kiểm tra loss ban đầu xấp xỉ ln(vocab) (khoảng 6,96 với vocab 1057). Xong khi: loss lệch ít; lệch hàng chục là có lỗi. (khoảng 15 phút)
5. Viết bộ nạp dữ liệu đọc `train.bin` và `val.bin` thành các đoạn dài `context_length` (đầu vào và đích lệch nhau một token). Xong khi: in thử một batch và kiểm tra đích đúng bằng đầu vào dịch một token. (khoảng 45 phút)
6. Cho mô hình học thuộc một batch nhỏ. Xong khi: loss giảm gần 0 (nếu không giảm thì có lỗi trong mô hình hoặc vòng lặp). (khoảng 30 phút)
7. Viết vòng lặp huấn luyện: AdamW, warmup rồi giảm cosine, cắt gradient, đánh giá trên toàn bộ val, lưu checkpoint, ghi log loss; chạy thử vài trăm bước trên CPU. Xong khi: loss train và val giảm đều. (khoảng 1,5–2 giờ)
8. Viết hàm sinh câu trả lời: ghép đúng khuôn `Hỏi: ...\nĐáp:`, chọn token xác suất cao nhất, dừng khi gặp `</s>`. Xong khi: mô hình trả lời được ít nhất một câu hỏi đã học. (khoảng 45 phút)
9. Viết script chấm test A, B, C: kiểm tra câu sinh ra có chứa `value`. Báo tỉ lệ đúng của test A, của test B (chỉ tính dòng `unseen`) và tỉ lệ từ chối đúng của test C. Xong khi: có ba con số cho một checkpoint. (khoảng 1 giờ)
10. Huấn luyện trên Colab: thêm mixed precision fp16 với GradScaler, đưa `tokenizer.json`, `data/*.bin` và mã lên Colab, lưu checkpoint lên Drive, chạy lần đầu và ghi đường cong loss. Xong khi: có checkpoint đầu tiên cùng ba con số của bước 9. (khoảng 1,5–2 giờ)

Ghi chú cho cả giai đoạn:

- Thay `tiktoken` của thầy bằng `tokenizer.json` (thư viện `tokenizers`, dùng `encode(...).ids`); đánh giá trên toàn bộ val thay vì 5 batch.
- Không cài nguyên `requirements.txt` của thầy: file có `tensorflow` và `litgpt` (phục vụ nạp trọng số GPT-2 và fine-tuning, dự án không dùng); chỉ cài thứ cần (khi vẽ đồ thị loss thì `uv add matplotlib`). Python của môi trường dự án là 3.12, còn Python hệ thống là 3.14.
- Nếu dùng lại mã của thầy: giữ dòng bản quyền và ghi nguồn (Sebastian Raschka, Apache 2.0) trong báo cáo.

# Kiến thức đã áp dụng
- [[Train-Val-Test Split]]
- [[Data Leakage]]
- [[Unicode NFC Normalization]]
- [[Scaling Laws (Chinchilla)]]
- [[Overfitting và Underfitting]]
