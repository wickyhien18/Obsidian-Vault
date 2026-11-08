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

- **Cổng 0 (trước 8/11):** hỏi thầy có được dùng PyTorch autograd không. Nếu không được, vai A (CRNN) không làm được và đề này dừng, quay về đề đơn
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
- Viết lại `README.md` thành đường thẳng Bước 0 → Bước 9 (có "You should see" ở mỗi bước), thêm cài và đăng nhập GitHub CLI, mục tạo token cho chủ repo; viết lại notebook kiểm tra Drive/GitHub cho khớp (tên thư mục mới, không tự tạo thư mục gốc).

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
  - Trước 8/11: xác nhận hai hợp đồng dữ liệu (`ocr_out.jsonl`, `correct(text)`) với Dương và Khánh; hỏi thầy về PyTorch autograd (Cổng 0)
  - Trước 8/11: tạo `splits.json`. Cần biết cấu trúc UIT-HWDB (tên tệp ảnh, định dạng nhãn, có ID người viết không) để chia tập không rò rỉ
  - Trước 10/11: bản giả OCR trong `src/stubs/` sinh `ocr_out.jsonl` từ nhãn đúng cộng nhiễu dấu, để chạy pipeline khi A chưa xong
  - Trước 12/11: `src/eval/` gồm căn chỉnh Levenshtein có truy vết và bộ phân loại lỗi (dấu thanh / dấu mũ-móc / chữ cái nền / khoảng trắng), có test với ví dụ `bạn` → `ban`
  - Trước 12/11: nhóm đặt ngưỡng cho cổng giả thuyết (lỗi dấu phải chiếm tối thiểu bao nhiêu % tổng lỗi) **trước khi** nhìn số liệu
  - 13–15/11: chạy cổng giả thuyết trên `ocr_out.jsonl` thật đầu tiên của Dương
- Token chỉ đọc phải tạo lại nếu hết hạn hoặc bị lộ
- Cổng 0 (hỏi thầy về PyTorch autograd) vẫn chưa làm; câu hỏi tập test thật khoảng 90 dòng vẫn chưa chốt
- Các hạn trong `docs/ASSIGNMENTS.md` tính từ ngày 6/11, cần xem lại nếu nhóm bắt đầu muộn hơn

# Kiến thức đã áp dụng
- [[Unicode Normalization]] (chuẩn hóa NFC trước khi so sánh chuỗi)
- [[Levenshtein Distance]] (sẽ áp dụng ở bước D3: căn chỉnh OCR với nhãn đúng)
