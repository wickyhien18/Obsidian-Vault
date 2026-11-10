---
Hub: "[[MOC - AI & ML]]"
created: 2026-11-06
status: In Progress
repo: https://github.com/wickyhien18/Vietnamese-Handwriting-OCR
---

# Ý tưởng dự án

Đồ án nhóm 3 người môn Nhập môn học máy (hạn cuối tháng 11/2026, có vấn đáp): **pipeline đọc chữ viết tay tiếng Việt rồi sửa dấu**.

- **Luồng dữ liệu:** ảnh dòng chữ viết tay → OCR (A) → mô hình sửa dấu (B) → văn bản đúng; C đánh giá, tích hợp và làm demo.
- **Câu hỏi nghiên cứu:** lớp sửa dấu giảm CER/WER của OCR được bao nhiêu, và khi nào nó làm tệ đi (sửa nhầm chữ vốn đúng). Chỉ sửa khi độ tin cậy vượt ngưỡng τ.
- **Ràng buộc từ thầy:** phải tự cài đặt thuật toán (sklearn chỉ làm baseline so sánh); được phép tự tạo LLM nếu khả thi; không dùng đề quá cũ; đề phải có tính thực tiễn thật.
- **Bằng chứng nhu cầu thực tế (đã tra):**
  - Nhập liệu phiếu tuyển sinh lớp 10 ở Tây Ninh hằng năm phải gõ lại bằng tay, tốn nhân lực và chi phí (nghiên cứu CRNN trên phiếu, WER khoảng 0,36).
  - Số hóa hồ sơ bệnh án viết tay (ACIS 2020): CER 6,47%, WER 19,14% trên hồ sơ của 12 bác sĩ.
  - Viettel AI (2025): hệ thống quốc tế thường kém với chữ viết tay tiếng Việt do hệ thống dấu; sản phẩm của họ đạt khoảng 99% với chữ in nhưng chỉ khoảng 90% với chữ viết tay.
  - Bài khảo sát Vietnamese DAR (arXiv 2506.05061): tài liệu hỗn hợp in và viết tay (căn cước, đơn từ, sao kê, bệnh án) vẫn là bài toán mở.
- **Điểm yếu đã biết, cần nói thẳng khi vấn đáp:**
  - UIT-HWDB là dữ liệu **tổng hợp** (dựng từ nét viết trực tuyến VNOnDB), chưa chắc đúng với chữ viết thật.
  - Giả thuyết "lỗi dấu chiếm phần lớn lỗi OCR" **chưa được đo**.
  - Đã có bài AAAI 2025 (arXiv 2410.13305) về post-OCR cho chữ có dấu, nhưng dùng LLM trên sách in lịch sử; chưa kiểm chứng được liệu có công trình gần hơn (chữ viết tay + mô hình nhỏ + ngưỡng tin cậy).

# Tech Stack
| Thành phần | Công nghệ | Lý do chọn |
| ---------- | --------- | ---------- |
| Dữ liệu OCR | UIT-HWDB (line 7.273 ảnh, word 110.745 ảnh, paragraph 1.144 ảnh) | Công khai, có nhãn; ảnh dòng có ngữ cảnh cho bước sửa dấu. Là dữ liệu tổng hợp nên cần tập test thật bổ sung |
| Mô hình OCR (vai A) — dự kiến | CRNN (CNN → BiLSTM → CTC) bằng PyTorch; giải mã CTC greedy + beam search tự cài | Kiến trúc chuẩn cho nhận dạng dòng chữ; phần tự cài là giải mã. **Phụ thuộc thầy cho phép PyTorch autograd** |
| Mô hình sửa dấu (vai B) — dự kiến | Bảng tần suất → bigram + Viterbi tự cài → transformer nhỏ gán nhãn theo âm tiết, kèm hiệu chỉnh độ tin cậy | Tăng dần độ phức tạp; có baseline mạnh để so sánh |
| Đánh giá + demo (vai C — K) | Levenshtein có truy vết tự cài, phân loại lỗi, chỉ số CER/WER, quét ngưỡng τ; demo web tối giản | Đo được đóng góp của lớp sửa dấu và tỉ lệ "sửa hỏng" |
| Hạ tầng | GitHub repo riêng tư (code, hợp đồng dữ liệu, test) + Google Drive (dữ liệu, checkpoint, `ocr_out.jsonl`) + Google Colab | Code cần quản lý phiên bản cho 3 người; dữ liệu/trọng số lớn không đưa lên git |
| Kho văn bản (vai B) | Wikipedia tiếng Việt qua `wikimedia/wikipedia`, cấu hình `20231101.vi`, giấy phép CC BY-SA 3.0 và GFDL theo thẻ dữ liệu | Có sẵn, tải được từ Colab, câu viết chuẩn có dấu để tự sinh cặp nhiễu. Có thể không khớp miền với nhãn UIT-HWDB, đo bằng độ phủ âm tiết |

# Nhật ký kỹ thuật

## Giai đoạn 1 — Chọn đề tài và chốt hướng đi
**Mục tiêu giai đoạn:** Chọn được một đề tài vừa đúng ràng buộc của thầy (tự cài thuật toán, không đề quá cũ, thực tiễn), vừa làm kịp trong khoảng 3,5 tuần với 3 người; chốt phân vai và cách nối các phần để ít ảnh hưởng tiến độ nhau.

**Đã làm:**

- Đặt tiêu chí lọc: dữ liệu lấy được trong khoảng 1 ngày, thuật toán tự cài được, không phải đề kinh điển, chia được việc cho 3 người.
- Lập danh sách 14 đề rồi lọc: giữ lại khôi phục dấu tiếng Việt, phát hiện URL lừa đảo, cảnh báo sinh viên bỏ học, dự báo AQI Hà Nội; thêm 4 đề thị giác máy tính (ảnh thật/ảnh do AI tạo, ảnh giấy tờ chụp lại từ màn hình, OCR chữ viết tay tiếng Việt, chấm trắc nghiệm từ ảnh).
- Kiểm chứng dữ liệu bằng tìm kiếm: PhiUSIIL (235.795 mẫu, 54 feature), Student Dropout (4.424 mẫu, 36 feature, 3 lớp), CIFAKE (120.000 ảnh 32×32, sinh bằng Stable Diffusion 1.4), UIT-HWDB (110.745 ảnh từ, 7.273 ảnh dòng, 1.144 ảnh đoạn), OpenAQ (cần khóa API miễn phí, giới hạn 60 yêu cầu/phút và 2.000/giờ).
- Đề xuất "góc mới" cho từng đề để không dính cách làm cũ (ví dụ: mô hình nhỏ biết khi nào không chắc; độ bền trước tấn công biến đổi URL; xếp hạng top-k cho cảnh báo bỏ học; khoảng dự báo AQI).
- Tìm giao điểm giữa đề NLP và đề thị giác, chọn giao điểm khôi phục dấu × OCR chữ viết tay.
- Chốt phân vai A/B/C và hai hợp đồng dữ liệu; K nhận vai C.
- Dựng bộ khung repo `ml-ocr-diacritics-skeleton.zip`: schema `ocr_out.jsonl`, script kiểm tra, `text_utils.py` (chuẩn hóa NFC, bỏ dấu), corrector giả, 5 test đều pass.

**Quyết định kỹ thuật:**

- **Vấn đề:** chọn đề tài cuối cùng trong số các đề còn lại (đề đơn hay ghép).
- **Đã chọn:** pipeline OCR chữ viết tay tiếng Việt + sửa dấu (ghép hai đề).
- **Vì sao:** hai bài toán nối thành một pipeline tự nhiên (ví dụ benchmark UIT-HWDB có lỗi đọc "bạn" thành "ban"), mỗi người có phần việc độc lập, hợp ý thầy cho phép tự tạo mô hình. Đánh đổi: khối lượng gần gấp đôi một đề đơn, cần PyTorch autograd, dữ liệu là tổng hợp. Đã loại: tiểu đường, giá nhà, cảm xúc review, giao hàng trễ, vỡ nợ, phản hồi sinh viên (quá phổ biến hoặc dữ liệu tổng hợp); phishing URL và typosquatting (nguy cơ dataset quá dễ, dữ liệu giả); ảnh thật/giả AI (phụ thuộc kiếm được ảnh từ bộ sinh mới).

- **Vấn đề:** chia việc thế nào để A, B, C ít ảnh hưởng nhau.
- **Đã chọn:** A làm OCR, B làm mô hình sửa dấu, C làm đánh giá + tích hợp + demo; chỉ nối nhau qua hai hợp đồng, đóng băng từ 8/11: (1) `ocr_out.jsonl` với các trường `id`, `split`, `image_path`, `gt_text`, `ocr_text`, tùy chọn `ocr_conf`; (2) hàm `correct(text)` trả `{"text": str, "conf": [float]}`, một độ tin cậy cho mỗi âm tiết.
- **Vì sao:** C chạy bằng bản giả (OCR giả, corrector giả) từ ngày đầu, B huấn luyện trên dữ liệu nhiễu giả lập, nên A hay B trễ đều không chặn người còn lại. Phụ thuộc mềm duy nhất: sau tuần 1, C báo dạng lỗi OCR thật để B chỉnh bộ làm nhiễu.

- **Vấn đề:** lưu code và dữ liệu ở đâu.
- **Đã chọn:** GitHub repo riêng tư cho code, hợp đồng, test; Google Drive (`ml-project/`) cho dữ liệu, checkpoint, `ocr_out.jsonl`; Colab clone repo và gắn Drive.
- **Vì sao:** cần quản lý phiên bản cho 3 người; GitHub chặn tệp lớn hơn 100 MB nên dữ liệu và trọng số để ở Drive.

- **Vấn đề:** mọi chuỗi tiếng Việt so sánh với nhau phải ở dạng Unicode nào.
- **Đã chọn:** chuẩn hóa tất cả về NFC trước khi so sánh; script kiểm tra báo lỗi nếu `gt_text` hoặc `ocr_text` không phải NFC.
- **Vì sao:** cùng một chữ (ví dụ `ạ`) có thể là một ký tự dựng sẵn hoặc `a` ghép dấu rời; nếu nhãn và đầu ra OCR khác dạng, bộ đo sẽ báo lỗi giả cho chữ vốn đúng.

- **Cập nhật đánh giá (sai lầm đã sửa):** trước đây nhận định khôi phục dấu "không trùng ai" là **sai**: bài toán đã được nghiên cứu nhiều và các mô hình có sẵn đạt khoảng 99% trên văn bản chuẩn. Vì vậy đề không được bảo vệ bằng lý do "mới"; điểm bảo vệ là đo đóng góp của lớp sửa dấu trên chữ viết tay và cơ chế chỉ sửa khi đủ tin cậy.

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 1 — Tạo thư mục bằng `mkdir -p` với ngoặc nhọn không ra thư mục con:** Lệnh `mkdir -p $R/{contracts,scripts,...}` tạo ra một thư mục tên nguyên văn có chứa ngoặc nhọn, các lệnh `touch` sau đó báo "No such file or directory" → Nguyên nhân thật sự: shell chạy lệnh không phải bash nên không có brace expansion → Cách sửa: dùng vòng `for d in ...; do mkdir -p ...; done` liệt kê rõ từng thư mục
- **Bug 2 — `unittest discover` báo "Start directory is not importable":** chạy test ngay sau khi dựng bộ khung thì lỗi, trong khi script kiểm tra riêng vẫn chạy được → Nguyên nhân thật sự: thư mục `tests/` thiếu `__init__.py` nên không phải package để import → Cách sửa: thêm `tests/__init__.py`; chạy lại, 5 test đều pass

**Việc còn tồn đọng từ giai đoạn này:**

- ~~**Cổng 0 (trước 8/11):** hỏi thầy có được dùng PyTorch autograd không. Nếu không được, vai A (CRNN) không làm được và đề này dừng, quay về đề đơn~~ Hiển báo đã xong và được thầy chấp nhận (xem Giai đoạn 3)
- Kiểm tra UIT-HWDB: 7.273 dòng có đủ để huấn luyện CRNN không, có ID người viết để chia tập không, nhãn có đủ dấu không. Nếu không đủ: ghép ảnh từ thành dòng, hoặc dùng OCR có sẵn (cần hỏi thầy)
- Tạo `splits.json` (cần dữ liệu thật), viết bản giả OCR và bộ căn chỉnh Levenshtein + phân loại lỗi (D2, D3)
- **Cổng giả thuyết (13–15/11):** đo lỗi dấu chiếm bao nhiêu % tổng lỗi OCR; nhóm tự đặt ngưỡng trước khi nhìn số liệu
- Tạo repo GitHub riêng tư từ file zip và thư mục `ml-project/` trên Drive, chia quyền cho hai bạn; thay `<OWNER>/<REPO>` trong ô Colab đầu tiên; điền trường `repo` ở đầu note này
- ~~Chốt phân vai A và B với hai bạn~~ Đã chốt ở Giai đoạn 2: Dương = A, Khánh = B, Hiển (K) = C
- **Chưa chốt:** thêm tập test thật khoảng 90 dòng tự viết tay vào kế hoạch của vai C (đã đề xuất, chưa có câu trả lời)
- Tra tài liệu khoảng 30 phút với các từ khóa "Vietnamese handwritten post-OCR correction" và "diacritic restoration OCR" để biết có công trình gần hơn không
- Lịch dự kiến: 6–8/11 giai đoạn 0; 9–15/11 baseline độc lập (mốc 13/11); 16–22/11 mô hình chính + tích hợp đầu-cuối (mốc 20/11); 23–27/11 đánh giá, báo cáo, luyện vấn đáp; 28–30/11 đệm

## Giai đoạn 2 — Thiết lập môi trường và phân công
**Mục tiêu giai đoạn:** Để cả nhóm cài được cùng một môi trường trên Linux, macOS, Windows và Colab/Drive, và để mỗi người biết rõ phần việc, hạn nộp, phần phải tự cài và điều kiện hoàn thành.

**Đã làm:**

- Viết lại `README.md` hoàn toàn bằng tiếng Anh: cài Python và Git theo từng hệ điều hành (Ubuntu/Debian, CachyOS/Arch, Fedora, openSUSE, macOS, Windows), kích hoạt môi trường ảo theo từng shell (bash/zsh, fish, PowerShell, cmd, Git Bash), quy trình Git/GitHub, Drive + Colab, bảng xử lý lỗi.
- Tạo `scripts/setup_local.sh` (Linux/macOS) và `scripts/setup_local.ps1` (Windows): tạo môi trường ảo, cài thư viện, tạo các thư mục `data/`, `checkpoints/`, `outputs/`, chạy 5 test và kiểm tra tệp mẫu.
- Tạo `notebooks/00_setup_drive.ipynb`: gắn Drive, tạo cấu trúc thư mục `ml-project/`, kiểm tra quyền đọc/ghi, kiểm tra token GitHub.
- Tạo `docs/ASSIGNMENTS.md`: Dương = A (OCR), Khánh = B (sửa dấu), Hiển = C (đánh giá, tích hợp, demo), kèm hạn theo ngày, phần phải tự cài, điều kiện hoàn thành, bảng ai chờ ai và các cổng quyết định.
- Tạo `docs/SETUP_CHECKLIST.md`: danh sách việc theo 4 phần (người tạo repo/Drive, từng người trên laptop, trên Colab, báo cáo lại nhóm).
- Kiểm thử script Linux trong một bản sao tạm của repo: tạo được môi trường ảo, cài numpy, 5 test đều pass, tệp mẫu báo OK, các thư mục được tạo.
- Hiển (K) tạo repo GitHub riêng tư `wickyhien18/Vietnamese-Handwriting-OCR` và thư mục Drive cùng tên `Vietnamese-Handwriting-OCR`, báo đã setup xong laptop và Drive của mình; tự chỉnh `README.md` (đường dẫn clone, bỏ phần tạo repo).
- Viết lại `README.md` thành đường thẳng Bước 0 → Bước 9 (sau này gộp bước 8–9 vào bước 7, còn Bước 0 → Bước 7, xem Giai đoạn 4) (có "You should see" ở mỗi bước), thêm cài và đăng nhập GitHub CLI, mục tạo token cho chủ repo; viết lại notebook kiểm tra Drive/GitHub cho khớp (tên thư mục mới, không tự tạo thư mục gốc).

**Quyết định kỹ thuật:**

- **Vấn đề:** người dùng nhiều hệ điều hành khác nhau cần cài giống nhau.
- **Đã chọn:** một script setup cho mỗi họ hệ điều hành, luôn dùng môi trường ảo `.venv`, không cài pip vào hệ thống.
- **Vì sao:** các bản Ubuntu/Debian/Arch gần đây bảo vệ Python hệ thống nên `pip install` ngoài môi trường ảo báo lỗi `externally-managed-environment`; môi trường ảo tránh lỗi này và tránh xung đột phiên bản. Đánh đổi: script PowerShell chưa được chạy thử trên Windows thật.

- **Vấn đề:** Colab clone repo riêng tư, và ai tạo token.
- **Đã chọn:** chủ repo (Hiển) tạo **một** token fine-grained chỉ đọc nội dung, giới hạn đúng một repo, có hạn sau deadline, gửi riêng cho Dương và Khánh; mỗi người lưu vào Secrets của Colab với tên `GITHUB_TOKEN`. Colab chỉ đọc code, việc push làm từ laptop.
- **Vì sao:** repo riêng tư nên `git clone` không xác thực sẽ lỗi. Cộng tác viên không tự tạo được token fine-grained cho repo của người khác (xem Bug 3). Token chỉ đọc và giới hạn một repo giảm thiệt hại nếu bị lộ, và chủ repo thu hồi được bất cứ lúc nào. Đánh đổi: ba người dùng chung một bí mật, và token phải tạo lại khi hết hạn.

- **Vấn đề:** thư mục Drive dùng chung cho 3 người.
- **Đã chọn:** Hiển tạo thư mục `Vietnamese-Handwriting-OCR` trên Drive và chia sẻ quyền Editor; mỗi người thêm shortcut vào My Drive. Notebook kiểm tra không tự tạo thư mục gốc, chỉ báo lỗi nếu không thấy.
- **Vì sao:** để đường dẫn `/content/drive/MyDrive/Vietnamese-Handwriting-OCR` giống nhau cho mọi người. Nếu notebook tự tạo thư mục gốc, người chưa thêm shortcut sẽ vô tình tạo một thư mục rỗng riêng và bài kiểm tra báo đạt giả. Cách shortcut hoạt động trong Colab chưa được kiểm chứng cho dự án này.

- **Vấn đề:** README theo chủ đề khiến người mới phải nhảy qua lại giữa các mục.
- **Đã chọn:** viết lại README thành một đường thẳng từ Bước 0 đến Bước 9, mỗi bước có dòng "You should see" để tự kiểm tra; phần tham chiếu (cấu trúc repo, hợp đồng dữ liệu, xử lý lỗi) để phía sau.
- **Vì sao:** mục tiêu là cả nhóm setup liền mạch mà không cần hỏi thêm.

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 3 — Hướng dẫn tạo token Colab sai cho cộng tác viên:** README bản trước bảo mỗi người tự tạo fine-grained token chỉ đọc cho repo → khi kiểm chứng lại tài liệu GitHub thì phát hiện cộng tác viên không làm được việc đó (chưa có ai chạy theo hướng dẫn sai này) → Nguyên nhân thật sự: mỗi fine-grained token chỉ truy cập được tài nguyên của một chủ sở hữu (user hoặc tổ chức), và GitHub liệt kê "repo mà người dùng là cộng tác viên" trong các trường hợp fine-grained token chưa hỗ trợ; repo thuộc tài khoản cá nhân `wickyhien18` nên chỉ chủ repo mới tạo được → Cách sửa: chủ repo tạo một token chỉ đọc gửi riêng cho hai bạn, README có mục "Maintainer only" hướng dẫn tạo

**Việc còn tồn đọng từ giai đoạn này:**

- Script `setup_local.ps1` chưa chạy trên Windows thật; notebook `00_setup_drive.ipynb` chưa chạy trên Colab thật (mới kiểm tra được tệp JSON hợp lệ); các lệnh cài `gh` theo từng bản Linux chưa chạy thử
- Dương và Khánh chưa setup: Hiển cần mời hai bạn làm cộng tác viên, chia sẻ Drive (Editor), tạo token chỉ đọc và gửi riêng, rồi gửi tin hướng dẫn setup
- **Việc của vai C (Hiển), theo thứ tự:**
  - ~~Trước 8/11: xác nhận hai hợp đồng dữ liệu (`ocr_out.jsonl`, `correct(text)`) với Dương và Khánh; hỏi thầy về PyTorch autograd (Cổng 0)~~ Đã xong (xem Giai đoạn 3)
  - Trước 8/11: tạo `splits.json`. Cần biết cấu trúc UIT-HWDB (tên tệp ảnh, định dạng nhãn, có ID người viết không) để chia tập không rò rỉ
  - Trước 10/11: bản giả OCR trong `src/stubs/` sinh `ocr_out.jsonl` từ nhãn đúng cộng nhiễu dấu, để chạy pipeline khi A chưa xong — code đã viết và test ở Giai đoạn 3, chưa chạy trên dữ liệu thật
  - Trước 12/11: `src/eval/` gồm căn chỉnh Levenshtein có truy vết và bộ phân loại lỗi (dấu thanh / dấu mũ-móc / chữ cái nền / khoảng trắng), có test với ví dụ `bạn` → `ban` — code đã viết và test ở Giai đoạn 3
  - Trước 12/11: nhóm đặt ngưỡng cho cổng giả thuyết (lỗi dấu phải chiếm tối thiểu bao nhiêu % tổng lỗi) **trước khi** nhìn số liệu
  - 13–15/11: chạy cổng giả thuyết trên `ocr_out.jsonl` thật đầu tiên của Dương
- Token chỉ đọc phải tạo lại nếu hết hạn hoặc bị lộ
- Câu hỏi tập test thật khoảng 90 dòng vẫn chưa chốt; chưa ghi lại câu trả lời cụ thể của thầy về PyTorch autograd
- Các hạn trong `docs/ASSIGNMENTS.md` tính từ ngày 6/11, cần xem lại nếu nhóm bắt đầu muộn hơn

## Giai đoạn 3 — Vai C: công cụ dữ liệu và bộ đo lỗi
**Mục tiêu giai đoạn:** Chuẩn bị mọi thứ vai C làm được mà không cần dữ liệu thật, để khi UIT-HWDB về chỉ còn một bước nối (adapter) là chạy được cả chuỗi: chia tập, bản giả OCR, căn chỉnh và phân loại lỗi.

**Đã làm:**

- Hiển báo các bước 1–3 của kế hoạch vai C (mở quyền, xác nhận hợp đồng, hỏi thầy) đã xong và được thầy chấp nhận. Chưa ghi lại câu trả lời cụ thể của thầy cho từng câu.
- Đọc README của bộ dữ liệu (repo `nghiangh/UIT-HWDB-dataset`; đường dẫn cũ `hieunghia-pat/...` chuyển hướng về đây): có link Google Drive riêng cho word, line, paragraph; thống kê 110.745 / 7.273 / 1.144; ảnh mẫu lấy "từ tập huấn luyện". README **không mô tả** định dạng nhãn, ID người viết hay cách chia train/val/test bên trong tệp tải về.
- Viết `scripts/inspect_dataset.py`: in báo cáo cấu trúc từ thư mục hoặc tệp .zip (không giải nén): phân bố đuôi tệp, tên thư mục giống train/val/test, ảnh mẫu, gợi ý ID người viết từ tên tệp, vài dòng đầu của tệp nhãn và số dòng không phải NFC.
- Viết `scripts/make_splits.py`: tạo `splits.json` từ tệp chỉ mục JSONL (`id`, `image_path`, `gt_text`, tùy chọn `writer_id`, `split`).
- Viết `src/stubs/stub_ocr.py`: sinh `ocr_out.jsonl` giả từ nhãn đúng cộng nhiễu dấu (bỏ dấu hoặc đổi dấu thanh), cố định theo `id` nên chạy lại cho kết quả giống hệt.
- Viết `src/eval/align.py` (Levenshtein có truy vết, dùng cho ký tự và danh sách từ, kèm CER/WER) và `src/eval/classify_errors.py` (phân loại lỗi: dấu thanh / dấu mũ-móc-trăng-gạch / cả hai / chữ cái nền / khoảng trắng; tính tỉ lệ lỗi dấu).
- Viết 35 test mới, tổng 40 test, đều pass: bảng Levenshtein của ví dụ `ban` so với `bạn` khớp bảng tính tay, truy vết ra đúng một phép thay thế, các nhãn phân loại, việc chia tập xác định và không phụ thuộc thứ tự, người viết không bị tách sang hai tập, tệp giả qua được script kiểm tra.
- Chạy thử đầu-cuối trên dữ liệu giả (40 dòng): chia tập → bản giả OCR → script kiểm tra báo `OK` → phân loại lỗi chạy được.
- Sửa `README.md` và `docs/SETUP_CHECKLIST.md`: đổi "5 tests" thành cách nói không phụ thuộc số lượng vì số test tăng lên 40.

**Quyết định kỹ thuật:**

- **Vấn đề:** chưa biết cấu trúc UIT-HWDB nhưng muốn viết ngay code chia tập và bản giả
- **Đã chọn:** tách hai tầng: một adapter nhỏ (viết sau khi xem dữ liệu thật) đổi dữ liệu gốc thành chỉ mục JSONL chuẩn; mọi công cụ khác chỉ đọc chỉ mục này
- **Vì sao:** chỉ một chỗ phụ thuộc vào cấu trúc chưa biết, phần còn lại đã kiểm thử. Đánh đổi: thêm một bước adapter

- **Vấn đề:** chia train/val/test thế nào
- **Đã chọn:** ưu tiên chia chính thức của bộ dữ liệu nếu có (thiếu val thì tách val từ train bằng băm); nếu không thì chia theo người viết khi có `writer_id`; cuối cùng mới chia theo băm của `id` theo tỉ lệ 80/10/10
- **Vì sao:** nét chữ của một người không được nằm ở hai tập, nếu không điểm test cao giả; băm không phụ thuộc thứ tự tệp nên chạy lại ra đúng tệp cũ. Đánh đổi: tập nhỏ chia theo băm bị lệch tỉ lệ (40 dòng thử ra 36/3/1), chỉ chuẩn khi số lượng lớn

- **Vấn đề:** quy ước căn chỉnh Levenshtein
- **Đã chọn:** `ref` là nhãn đúng, `hyp` là kết quả OCR; hàng của bảng theo `hyp`, cột theo `ref`; khi truy vết ưu tiên đường chéo, rồi "thừa", rồi "thiếu"
- **Vì sao:** khớp bảng tính tay đã đưa cho Hiển, và kết quả xác định

- **Vấn đề:** lỗi dấu của `đ`
- **Đã chọn:** coi `đ` so với `d` là lỗi dấu loại mũ-móc-trăng-gạch
- **Vì sao:** `đ` không tách được thành chữ nền cộng dấu rời trong Unicode nên cần xử lý riêng

- **Lưu ý quan trọng:** tỉ lệ lỗi dấu của bản giả OCR luôn bằng 100% vì nhiễu chỉ đụng vào dấu, nên **không được dùng số này cho cổng giả thuyết**; chỉ dùng số tính từ OCR thật.

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 4 — Điều kiện tách val khi bộ dữ liệu không có val viết sai logic:** khi đọc lại mã trước khi chạy test thấy một biểu thức lồng nhiều phép chia sẽ không cho đúng tỉ lệ val mong muốn (phát hiện lúc xem lại, chưa kịp chạy) → Nguyên nhân thật sự: ghép hai ý khác nhau vào một điều kiện khi viết vội → Cách sửa: thay bằng một ngưỡng duy nhất `round(100 * val / (train + val))` áp lên giá trị băm; thêm test `test_official_without_val_carves_val_from_train`

**Việc còn tồn đọng từ giai đoạn này:**

- ~~Tải UIT-HWDB-line (link Google Drive trong README của bộ dữ liệu) vào `data/uit_hwdb/` trên Drive; chưa chốt ai tải (Dương theo phân công hoặc Hiển)~~ Hiển tải bằng notebook `00_prepare_data.ipynb` (xem Giai đoạn 4)
- ~~Chạy `scripts/inspect_dataset.py` trên dữ liệu thật và gửi kết quả để viết adapter: tệp nhãn ở đâu, định dạng ra sao, có ID người viết không, có chia sẵn train/val/test không~~ Đã đưa vào `00_prepare_data.ipynb`, báo cáo lưu ở `outputs/dataset_report.txt` (xem Giai đoạn 4)
- ~~Viết adapter đổi dữ liệu gốc thành chỉ mục JSONL, rồi chạy `make_splits.py` và `stub_ocr.py` trên dữ liệu thật; các công cụ mới chưa từng chạy trên dữ liệu UIT-HWDB thật, test hiện chỉ dùng dữ liệu giả~~ Đã viết `scripts/build_index.py` (đoán định dạng nhãn), chưa thấy dữ liệu thật (xem Giai đoạn 4)
- Việc còn lại của vai C: đặt ngưỡng cổng giả thuyết **trước khi** nhìn số liệu (trước 12/11); chạy cổng giả thuyết trên `ocr_out.jsonl` thật của Dương (13–15/11); bảng so sánh các cấu hình, quét ngưỡng độ tin cậy, demo (16–22/11)
- Hiển phải đọc và giải thích được `align.py` và `classify_errors.py` trước vấn đáp. Mã do Claude viết theo yêu cầu; nếu thầy yêu cầu tự viết, dùng các test làm đặc tả để tự viết lại

## Giai đoạn 4 — Rà soát và chuẩn bị dữ liệu cho cả nhóm
**Mục tiêu giai đoạn:** Rà soát toàn bộ file đến lúc này để (1) có sẵn dữ liệu cho cả nhóm, (2) mỗi người có file đầu tiên để bắt tay làm ngay, (3) cập nhật nội dung các file cũ, (4) tận dụng việc Colab đã nối với GitHub bằng token để chạy mọi thứ trong Colab thay vì chép tay.

**Đã làm:**

- Kiểm chứng nguồn: link tải UIT-HWDB-line nằm trong README của bộ dữ liệu trên GitHub (có link Drive riêng cho word, line, paragraph); kho văn bản tiếng Việt lấy từ `wikimedia/wikipedia`, cấu hình `20231101.vi`, giấy phép CC BY-SA 3.0 và GFDL theo thẻ dữ liệu.
- Viết `scripts/build_index.py`: đoán định dạng nhãn (JSON, JSONL, dòng `tên tệp<TAB>chữ`, CSV, hoặc một tệp .txt cạnh mỗi ảnh), ghép nhãn với ảnh, phát hiện chia train/val/test theo tên thư mục và ID người viết theo biểu thức chính quy tùy chọn; nếu dưới 95% ảnh có nhãn thì dừng và in thứ nó thấy để chỉnh.
- Viết `scripts/prepare_corpus.py`: lấy câu từ Wikipedia tiếng Việt, chuẩn hóa NFC, chỉ giữ câu gồm chữ cái tiếng Việt, chữ số và dấu câu cơ bản, dài 4–40 âm tiết, bỏ trùng.
- Viết `src/common/index_io.py` (đọc chỉ mục và `splits.json`).
- Viết 3 notebook: `00_prepare_data.ipynb` (chạy một lần: tải dữ liệu, báo cáo cấu trúc, giải nén ra ổ Colab, tạo chỉ mục, `splits.json`, kho văn bản), `ocr_00_start.ipynb` (Dương: chép ảnh về ổ Colab, kiểm tra ảnh khớp nhãn, GPU) và `corrector_00_start.ipynb` (Khánh: xem kho văn bản, các âm tiết mơ hồ nhất, độ phủ so với nhãn tập train).
- Viết 20 test mới, tổng 60 test, đều pass. Chạy thử logic các notebook trên dữ liệu giả; chưa chạy trên Colab thật.
- Hiển báo `00_prepare_data.ipynb` đã tải dữ liệu, chạy `build_index.py` và tạo `splits.json` thành công, và kho văn bản 300.000 câu đã được tạo trên Drive. Chưa gửi đầu ra của `build_index.py` và `make_splits.py` nên chưa xác nhận số ảnh có nhãn, chế độ chia tập (chính thức, theo người viết hay theo `id`) và có ID người viết hay không.
- Xem phần đầu của corpus (Hiển dán vào cuộc trò chuyện): chữ có dấu đầy đủ, không thấy lỗi mã hóa; thấy một số câu bị cắt ở chữ viết tắt (ví dụ "Ulysses S." tách khỏi "Grant ..."), vài mảnh vụn bắt đầu bằng chữ thường, và các câu liền nhau thuộc cùng một bài. Chưa kiểm toàn bộ 300.000 dòng.
- Sửa `scripts/prepare_corpus.py`: không tách câu sau một chữ cái hoa đơn lẻ ("S.", "A.G.") hoặc khi chữ sau là chữ thường, và bỏ mảnh không bắt đầu bằng chữ hoa, chữ số, dấu nháy hoặc ngoặc; thêm 7 test, tổng 67 test đều pass.
- Tạo bộ file hợp đồng dữ liệu trong `contracts/`: `README.md` (bảng 4 hợp đồng: `dataset_index.jsonl`, `splits.json`, `ocr_out.jsonl`, `correct(text)`), `dataset_index.schema.json`, `splits.schema.json`, `corrector_api.md` (ví dụ cụ thể và 7 quy tắc), hai tệp mẫu trong `examples/`; gửi lại `ocr_out.schema.json` và tệp mẫu của nó.
- Viết `src/common/contract_checks.py` (kiểm tra đầu ra của `correct()`, bản ghi chỉ mục và `splits.json`), `scripts/validate_dataset_index.py` và `tests/test_contract_checks.py`; tổng 87 test đều pass.
- Cập nhật mục hợp đồng dữ liệu trong `README.md` từ 2 thành 4 hợp đồng.
- Cập nhật `README.md` (gộp bước 7, thêm mục phiên đầu tiên và mục chuẩn bị dữ liệu, ô đầu tiên có `BRANCH`, thêm hai dòng xử lý lỗi), `docs/ASSIGNMENTS.md` (quyết định của nhóm, mốc mới, phần đã xong), `docs/SETUP_CHECKLIST.md` (bước 0–7 và phiên đầu tiên).

**Quyết định kỹ thuật:**

- **Vấn đề:** có chờ câu trả lời của thầy về PyTorch autograd hay tự quyết
- **Đã chọn:** nhóm tự quyết dùng PyTorch autograd để huấn luyện CRNN và transformer; các phần phải tự cài (tiền xử lý, giải mã CTC, Viterbi, hiệu chỉnh, căn chỉnh, phân loại lỗi, ngưỡng) giữ nguyên
- **Vì sao:** dự án tự làm nên nhóm tự chịu trách nhiệm; không tự viết backward cho CRNN và transformer trong 3 tuần. Rủi ro: câu trả lời cụ thể của thầy không được ghi lại, nếu thầy cấm thì vai A và phần transformer của B phải đổi

- **Vấn đề:** ai tải dữ liệu và tải bằng cách nào
- **Đã chọn:** Hiển chạy `00_prepare_data.ipynb` một lần trong Colab, kết quả lên Drive; Dương và Khánh không tải lại
- **Vì sao:** Colab đã nối GitHub bằng token nên clone mã mỗi phiên, tải và xử lý ngay trong Colab, không phải chép tay và mọi người dùng cùng một tệp. Đánh đổi: `build_index.py` đoán định dạng nên có thể phải chỉnh khi thấy dữ liệu thật; `gdown` có thể bị Google chặn (đã có cách tải tay)

- **Vấn đề:** đọc ảnh từ đâu khi huấn luyện
- **Đã chọn:** mỗi phiên chép zip về ổ Colab và giải nén ra `/content/uit_hwdb`; `image_path` trong chỉ mục tính tương đối từ thư mục đó
- **Vì sao:** đọc hàng nghìn tệp nhỏ trực tiếp từ Drive chậm; đường dẫn tương đối giúp cả nhóm dùng chung một chỉ mục

- **Vấn đề:** kho văn bản cho vai B
- **Đã chọn:** Wikipedia tiếng Việt; Khánh thêm kho thứ hai nếu độ phủ âm tiết so với nhãn tập train thấp
- **Vì sao:** có sẵn, tải được, câu chuẩn có dấu. Câu mẫu của bộ dữ liệu đọc giống văn báo chí nên có thể lệch miền với Wikipedia (suy đoán, cần đo)

- **Vấn đề:** Colab ghi lại lên GitHub thế nào
- **Đã chọn:** Colab chỉ đọc (token chỉ đọc); tải notebook về, commit từ laptop; để thử mã trên nhánh riêng thì đặt `BRANCH` ở ô đầu tiên
- **Vì sao:** giữ token ở quyền tối thiểu

- **Vấn đề:** để notebook ở đâu để Dương và Khánh mở nhanh
- **Đã chọn:** đặt 4 notebook trong thư mục Drive `notebooks/` (bản gốc vẫn ở repo), mở bằng Open with → Google Colaboratory; mỗi người File → Save a copy in Drive rồi làm trên bản sao; README, checklist và hai notebook khởi động sửa theo
- **Vì sao:** không phải tải notebook từ bản clone mỗi lần, hai bạn vào thẳng từ Drive; Save a copy tránh ghi đè lên file dùng chung. Đánh đổi: bản trên Drive có thể lệch với repo, nên khi notebook đổi Hiển phải tải bản mới lên Drive đè bản cũ

- **Vấn đề:** cổng giả thuyết chỉ dựa vào tỉ lệ lỗi dấu có đủ không
- **Đã chọn:** báo thêm trần lợi ích (CER nhân với tỉ lệ lỗi dấu)
- **Vì sao:** tỉ lệ lớn nhưng CER rất thấp thì sửa dấu cũng không đáng kể

- **Vấn đề:** có sửa bộ tách câu của kho văn bản hay dùng nguyên bản đã chạy
- **Đã chọn:** sửa script (không tách ở chữ viết tắt, bỏ mảnh vụn) và chạy lại ô kho văn bản trước khi Khánh bắt đầu
- **Vì sao:** sửa rẻ và an toàn nhưng lợi ích nhỏ (các mảnh vụn vẫn là tiếng Việt đúng dấu, chỉ làm sai vài ngữ cảnh đầu và cuối câu); chạy lại tốn thời gian tương đương lần đầu nên cần làm trước khi Khánh dùng, để khỏi đổi dữ liệu giữa chừng

- **Vấn đề:** Khánh chia train và val của kho văn bản thế nào
- **Đã chọn:** chia theo khối liên tục (ví dụ 90% đầu làm train, 10% cuối làm val), không xáo ngẫu nhiên từng câu
- **Vì sao:** file xếp theo bài viết, câu liền nhau chung tên riêng và cụm từ lặp; xáo ngẫu nhiên làm val chứa câu hàng xóm của train nên độ chính xác val cao giả

- **Vấn đề:** hợp đồng `correct(text)` ban đầu chưa nói rõ số âm tiết đầu ra và phần được phép sửa
- **Đã chọn:** làm rõ hai quy tắc: số âm tiết không đổi, và chỉ đổi dấu (bỏ dấu hai bên phải ra cùng chuỗi, hoa thường không đổi); thêm kiểm tra bằng `check_correct_output`
- **Vì sao:** phép đo so sánh từng âm tiết giữa đầu vào và đầu ra (bảng sửa đúng, sửa hỏng), và lỗi chữ cái nền là ngoài phạm vi của bộ sửa dấu. Hai quy tắc này khớp với thiết kế gán một nhãn cho mỗi âm tiết nhưng là điều bổ sung sau khi đóng băng 8/11, nên cần Khánh xác nhận

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 5 — Hướng dẫn bật Notebook access ở notebook bất kỳ là sai:** README bản trước bảo tạo secret rồi bật Notebook access trong một notebook bất kỳ, sau đó mới mở notebook kiểm tra → khi tra tài liệu thì thấy bước kiểm tra GitHub sẽ báo lỗi dù token đúng (chưa ai chạy theo hướng dẫn sai này) → Nguyên nhân thật sự: danh sách secret dùng chung cho mọi notebook nhưng quyền truy cập bật riêng cho từng notebook → Cách sửa: gộp thành Bước 7, mở đúng notebook kiểm tra từ Drive rồi thêm secret và bật quyền ngay trong đó; thêm dòng nhắc bật lại với mỗi notebook mới và tách bảng lỗi thành `SecretNotFoundError` và `NotebookAccessError`
- **Bug 6 — Kho văn bản có câu bị cắt ở chữ viết tắt:** trong phần đầu của corpus có các cặp dòng kiểu "…Tổng thống: Ulysses S." rồi "Grant (tại Point Pleasant)…", "Ví dụ của A.G.", và mảnh bắt đầu bằng chữ thường sau dấu `...` → Nguyên nhân thật sự: bộ tách câu cắt mọi chỗ có dấu chấm theo sau là khoảng trắng, không biết chữ cái viết tắt và không kiểm tra chữ sau có phải chữ hoa → Cách sửa: không tách sau chữ cái hoa đơn lẻ hoặc khi chữ sau là chữ thường, bỏ mảnh không bắt đầu như một câu; thêm 7 test
- **Bug 7 — `validate_dataset_index.py` chạy như script thì lỗi `ModuleNotFoundError: No module named 'src'`:** 87 test đều pass nhưng chạy thật `python scripts/validate_dataset_index.py ...` thì lỗi → Nguyên nhân thật sự: test import dưới dạng gói từ thư mục gốc, còn khi chạy như script thì thư mục đầu tiên trong đường dẫn import là `scripts/`, nên không thấy `src` → Cách sửa: chèn thư mục gốc của repo vào `sys.path` ở đầu script; đã chạy thử từ thư mục gốc, từ thư mục khác và với tệp lỗi (thoát với mã 1)

**Việc còn tồn đọng từ giai đoạn này:**

- Hiển chưa gộp các file mới vào `main` và chưa tải 4 notebook lên Drive `notebooks/`: phải làm trước khi chạy `00_prepare_data.ipynb`, vì ô đầu tiên clone nhánh `main` (hoặc đặt `BRANCH` thành `c/eval`)
- Chưa chạy `00_prepare_data.ipynb` trên Colab thật: `build_index.py` chưa từng thấy dữ liệu UIT-HWDB thật nên có thể phải chỉnh; `gdown` chưa thử; `prepare_corpus.py` chưa chạy với Wikipedia thật (cần gói `datasets`)
- Hạn `splits.json` ban đầu là 8/11, nay dời sang 9/11 (kế hoạch ban đầu tính từ 6/11, còn thực tế hôm nay là 8/11)
- Dương và Khánh chưa chạy notebook khởi động của mình
- Chưa chốt tập test thật khoảng 90 dòng; nếu làm thì cần xong khoảng 15/11 để Dương chạy OCR
- `00_prepare_data.ipynb`: ô kho văn bản dùng `os` nhưng chỉ import ở ô tải dữ liệu, nên chạy lại riêng ô này trong phiên mới sẽ lỗi `NameError`; đã thêm `import os` vào ô đó, cần tải bản mới lên Drive `notebooks/`
- Khánh xác nhận hai quy tắc làm rõ của `correct(text)` (cùng số âm tiết, chỉ đổi dấu); Hiển chạy `validate_dataset_index.py` trên `dataset_index.jsonl` và `splits.json` thật rồi đưa các file hợp đồng và mã mới vào repo
- Gộp `prepare_corpus.py` mới vào `main`, chạy lại ô đầu tiên (để clone bản mới) và ô kho văn bản của `00_prepare_data.ipynb`, rồi kiểm tra `wc -l` bằng 300000
- Gửi đầu ra của `build_index.py` và `make_splits.py` để kiểm tra: số ảnh có nhãn so với tổng, `split_counts`, chế độ chia, số người viết. Nếu chế độ là chia theo `id` mà tên tệp có chứa ID người viết thì chia như vậy rò rỉ nét chữ giữa các tập, phải chạy lại `build_index.py` với `--writer-regex`
- Hiển phải đọc và giải thích được `align.py`, `classify_errors.py` và các công cụ dữ liệu trước vấn đáp; cần một script in các cặp ký tự bị OCR nhầm nhiều nhất cho Khánh (trước 15/11)

# Kiến thức đã áp dụng
- [[Unicode Normalization]] (chuẩn hóa NFC trước khi so sánh chuỗi)
- [[Levenshtein Distance]] (đã cài ở `src/eval/align.py`: căn chỉnh OCR với nhãn đúng, truy vết, CER/WER)
