---
Hub: "[[MOC - AI & ML]]"
created: 2026-11-06
status: In Progress
repo: https://github.com/wickyhien18/Vietnamese-Handwriting-OCR
---
****
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
  - Giả thuyết "lỗi dấu chiếm phần lớn lỗi OCR" đã được đo ở Giai đoạn 6 trên val: lỗi dấu chỉ chiếm 11,3% tổng lỗi ký tự, trần lợi ích theo số đếm của cổng 1,38 điểm CER (cận dưới của trần chính xác, xem Bug 15), cổng giả thuyết FAIL. Cần nói rõ khi vấn đáp: cổng đo ở mức ký tự; phân tích thăm dò ở mức âm tiết cho kết quả cùng chiều (lỗi chỉ khác dấu chiếm 10,6% lỗi âm tiết).
  - Đã có bài AAAI 2025 (arXiv 2410.13305) về post-OCR cho chữ có dấu, nhưng dùng LLM trên sách in lịch sử; chưa kiểm chứng được liệu có công trình gần hơn (chữ viết tay + mô hình nhỏ + ngưỡng tin cậy).

# Tech Stack
| Thành phần | Công nghệ | Lý do chọn |
| ---------- | --------- | ---------- |
| Dữ liệu OCR | UIT-HWDB (line 7.273 ảnh, word 110.745 ảnh, paragraph 1.144 ảnh) | Công khai, có nhãn; ảnh dòng có ngữ cảnh cho bước sửa dấu. Là dữ liệu tổng hợp nên cần tập test thật bổ sung |
| Mô hình OCR (vai A) | CRNN (6 tầng tích chập → 2 tầng BiLSTM → CTC) bằng PyTorch, hàm mất mát `nn.CTCLoss`; giải mã CTC greedy và prefix beam search tự cài bằng NumPy | Kiến trúc chuẩn cho nhận dạng dòng chữ; phần tự cài là giải mã. Thầy đã đồng ý dùng PyTorch autograd, danh mục ngày 12/11 cũng ghi Phần A dùng "NumPy hoặc PyTorch"; theo Hiển, thầy chỉ nói chung là dùng PyTorch, không nêu từng thành phần, nên phạm vi `nn.CTCLoss`, `nn.LSTM`, `nn.Conv2d`, `AdamW` chưa có xác nhận riêng. Kết quả hiện tại: 25 epoch, val CER 12,27% (beam), Dương xác nhận là lần chạy cuối |
| Mô hình sửa dấu (vai B) — dự kiến | Bảng tần suất → bigram + Viterbi tự cài → transformer nhỏ gán nhãn theo âm tiết, kèm hiệu chỉnh độ tin cậy | Tăng dần độ phức tạp; có baseline mạnh để so sánh |
| Đánh giá + demo (vai C — K) | Levenshtein có truy vết tự cài, phân loại lỗi, chỉ số CER/WER, cổng giả thuyết, bảng sửa đúng/sửa hỏng, quét ngưỡng τ, bootstrap theo đoạn văn (notebook `eval_00_start.ipynb`); demo web tối giản (chưa làm) | Đo được đóng góp của lớp sửa dấu và tỉ lệ "sửa hỏng", kèm khoảng tin cậy |
| Hạ tầng | GitHub repo riêng tư (code, hợp đồng dữ liệu, test) + Google Drive (dữ liệu, checkpoint, `ocr_out.jsonl`) + Google Colab | Code cần quản lý phiên bản cho 3 người; dữ liệu/trọng số lớn không đưa lên git |
| Kho văn bản (vai B) | Wikipedia tiếng Việt qua `wikimedia/wikipedia`, cấu hình `20231101.vi`, giấy phép CC BY-SA 3.0 và GFDL theo thẻ dữ liệu | Có sẵn, tải được từ Colab, câu viết chuẩn có dấu để tự sinh cặp nhiễu. Có thể không khớp miền với nhãn UIT-HWDB, đo bằng độ phủ âm tiết |
| Chia tập | Gom trang thành đoạn văn bằng cửa sổ 5 từ (NFC, chữ thường, bỏ dấu câu) và union-find; giữ test chính thức; val lấy nguyên đoạn văn theo hạt giống 42 | Nhiều trang là người khác nhau chép cùng một đoạn văn, chia theo trang vẫn để val và test trùng khoảng 96% văn bản với train. Đổi lại train mất 14,6% số dòng; val chỉ có 9 đoạn và test 5 đoạn |

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

- ~~**Cổng 0 (trước 8/11):** hỏi thầy có được dùng PyTorch autograd không. Nếu không được, vai A (CRNN) không làm được và đề này dừng, quay về đề đơn~~ Hiển báo đã xong và được thầy chấp nhận (xem Giai đoạn 3) — xác nhận cụ thể về autograd có ở Giai đoạn 5
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
  - ~~Trước 8/11: xác nhận hai hợp đồng dữ liệu (`ocr_out.jsonl`, `correct(text)`) với Dương và Khánh; hỏi thầy về PyTorch autograd (Cổng 0)~~ Đã xong (xem Giai đoạn 3) (đã giải quyết ở Giai đoạn 5: thầy đồng ý autograd
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
- Riêng PyTorch autograd đã có xác nhận ở Giai đoạn 5
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
- Đọc bản `00_prepare_data.ipynb` Hiển đã chạy thật. Bộ dữ liệu: `UIT_HWDB_line.zip` 874,8 MB, 7.273 ảnh `.jpg` và 255 tệp `label.json`, mỗi thư mục trang một tệp (từ điển tên ảnh → chữ). Cấu trúc `UIT_HWDB_line/{train_data,test_data}/<số>/`, thư mục đánh số 1–255, tập test là 6 thư mục 250–255 (201 dòng).
- Kết quả chạy thật: `build_index.py` gắn nhãn 7.229 trên 7.273 ảnh (0 khóa nhãn không khớp, 44 ảnh không có nhãn, tất cả nằm ở train); `make_splits.py` chạy ở chế độ `official+carved_val` cho train 6.279, val 749, test 201; kho văn bản 300.000 câu.
- Nhận xét về nhãn: văn tin tức và giáo khoa, dấu câu tách thành từ riêng (dấu nháy, dấu hai chấm có dấu cách hai bên), có dòng viết hoa toàn bộ, và **cùng một đoạn văn do nhiều người chép** (thư mục 251 và 252 cùng nội dung, ngắt dòng khác nhau).
- Trong Drive có file không do script của mình tạo: `data/corpus/train.txt` (194,2 MB), `val.txt`, `test.txt`, `stats.json` và `outputs/corrector_eval.json`, `corrector_ngram_eval.json`; có vẻ Khánh đã tự chia kho văn bản và bắt đầu đo một mô hình n-gram. Chưa biết kho nào là kho chính.
- Thêm `group_id` (trường tùy chọn, ví dụ thư mục trang) vào chỉ mục và hợp đồng; `build_index.py` có `--group-regex` và báo thêm `no_label_entry`, `empty_label`, `unlabeled_examples`; `make_splits.py` tách val theo nhóm; viết `scripts/check_text_overlap.py`; cập nhật notebook `00_prepare_data.ipynb`, README, `contracts/README.md`, `ASSIGNMENTS.md`. Thêm 11 test, tổng 98 test đều pass, gồm một test mô phỏng đúng bố cục thật của UIT-HWDB-line.
- Cập nhật `README.md` (gộp bước 7, thêm mục phiên đầu tiên và mục chuẩn bị dữ liệu, ô đầu tiên có `BRANCH`, thêm hai dòng xử lý lỗi), `docs/ASSIGNMENTS.md` (quyết định của nhóm, mốc mới, phần đã xong), `docs/SETUP_CHECKLIST.md` (bước 0–7 và phiên đầu tiên).

**Quyết định kỹ thuật:**

- **Vấn đề:** có chờ câu trả lời của thầy về PyTorch autograd hay tự quyết
- **Đã chọn:** nhóm tự quyết dùng PyTorch autograd để huấn luyện CRNN và transformer; các phần phải tự cài (tiền xử lý, giải mã CTC, Viterbi, hiệu chỉnh, căn chỉnh, phân loại lỗi, ngưỡng) giữ nguyên
- **Vì sao:** dự án tự làm nên nhóm tự chịu trách nhiệm; không tự viết backward cho CRNN và transformer trong 3 tuần. Rủi ro: câu trả lời cụ thể của thầy không được ghi lại, nếu thầy cấm thì vai A và phần transformer của B phải đổi
- **Cập nhật:** thầy đồng ý dùng PyTorch autograd; rủi ro "nếu thầy cấm" đóng. Quyết định tự chọn ở trên được xác nhận đúng.

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

- **Vấn đề:** val nên lấy từ đâu khi bộ dữ liệu chỉ chia sẵn train và test
- **Đã chọn:** giữ nguyên chia train/test chính thức; val lấy từ **các trang cuối của train** (nguyên cả trang, theo thứ tự số của thư mục) thay vì rải ngẫu nhiên từng dòng
- **Vì sao:** các dòng cùng một trang có cùng nét chữ và cùng đoạn văn; rải từng dòng làm val chứa dòng hàng xóm của train nên CER val thấp giả, mà cổng giả thuyết lại đo trên val. Chọn khối liên tục vì các trang liền nhau hay chứa cùng một đoạn văn (251 và 252) nên chỉ rò rỉ nhiều nhất ở ranh giới. Đánh đổi: val không còn đại diện ngẫu nhiên cho toàn bộ train

- **Vấn đề:** thư mục trang là gì, có dùng `writer_id` không
- **Đã chọn:** thêm trường mới `group_id` (tùy chọn), không dùng `writer_id`
- **Vì sao:** chưa biết một thư mục ứng với một người viết hay chỉ một trang; gọi là `writer_id` sẽ sai tên và dễ bị hiểu nhầm khi đếm số người viết

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 5 — Hướng dẫn bật Notebook access ở notebook bất kỳ là sai:** README bản trước bảo tạo secret rồi bật Notebook access trong một notebook bất kỳ, sau đó mới mở notebook kiểm tra → khi tra tài liệu thì thấy bước kiểm tra GitHub sẽ báo lỗi dù token đúng (chưa ai chạy theo hướng dẫn sai này) → Nguyên nhân thật sự: danh sách secret dùng chung cho mọi notebook nhưng quyền truy cập bật riêng cho từng notebook → Cách sửa: gộp thành Bước 7, mở đúng notebook kiểm tra từ Drive rồi thêm secret và bật quyền ngay trong đó; thêm dòng nhắc bật lại với mỗi notebook mới và tách bảng lỗi thành `SecretNotFoundError` và `NotebookAccessError`
- **Bug 6 — Kho văn bản có câu bị cắt ở chữ viết tắt:** trong phần đầu của corpus có các cặp dòng kiểu "…Tổng thống: Ulysses S." rồi "Grant (tại Point Pleasant)…", "Ví dụ của A.G.", và mảnh bắt đầu bằng chữ thường sau dấu `...` → Nguyên nhân thật sự: bộ tách câu cắt mọi chỗ có dấu chấm theo sau là khoảng trắng, không biết chữ cái viết tắt và không kiểm tra chữ sau có phải chữ hoa → Cách sửa: không tách sau chữ cái hoa đơn lẻ hoặc khi chữ sau là chữ thường, bỏ mảnh không bắt đầu như một câu; thêm 7 test
- **Bug 7 — `validate_dataset_index.py` chạy như script thì lỗi `ModuleNotFoundError: No module named 'src'`:** 87 test đều pass nhưng chạy thật `python scripts/validate_dataset_index.py ...` thì lỗi → Nguyên nhân thật sự: test import dưới dạng gói từ thư mục gốc, còn khi chạy như script thì thư mục đầu tiên trong đường dẫn import là `scripts/`, nên không thấy `src` → Cách sửa: chèn thư mục gốc của repo vào `sys.path` ở đầu script; đã chạy thử từ thư mục gốc, từ thư mục khác và với tệp lỗi (thoát với mã 1)
- **Bug 8 — Val tách bằng băm của id dòng nên trộn các dòng cùng một trang vào cả train lẫn val:** lần chạy thật in `mode: official+carved_val` (val 749 dòng) → Nguyên nhân thật sự: khi bộ dữ liệu có chia sẵn nhưng không có val, `make_splits.py` băm `id` từng dòng vì chưa có khái niệm nhóm; điều này suy ra từ cách tách chứ chưa kiểm tra trực tiếp trên dữ liệu thật → Cách sửa: thêm `group_id`, tách val theo nhóm (trang cuối của train); thêm test mô phỏng bố cục thật và `check_text_overlap.py` để đo độ trùng văn bản giữa các tập

**Việc còn tồn đọng từ giai đoạn này:**

- Hiển chưa gộp các file mới vào `main` và chưa tải 4 notebook lên Drive `notebooks/`: phải làm trước khi chạy `00_prepare_data.ipynb`, vì ô đầu tiên clone nhánh `main` (hoặc đặt `BRANCH` thành `c/eval`) **[Đã xong một phần]** các script mới đã có trên `main` (ô 5 chạy được `build_index.py` bản mới và các file Giai đoạn 5) và `00_prepare_data.ipynb` bản mới đã chạy; chưa biết 3 notebook còn lại đã lên Drive chưa.
- Chưa chạy `00_prepare_data.ipynb` trên Colab thật: `build_index.py` chưa từng thấy dữ liệu UIT-HWDB thật nên có thể phải chỉnh; `gdown` chưa thử; `prepare_corpus.py` chưa chạy với Wikipedia thật (cần gói `datasets`) **[Đã xong ở Giai đoạn 5]** `build_index.py` đã chạy thật (7.229 trên 7.273 ảnh có nhãn) và `prepare_corpus.py` đã chạy với Wikipedia thật (300.000 câu); chưa rõ `gdown` đã chạy hay file được đặt tay.
- Hạn `splits.json` ban đầu là 8/11, nay dời sang 9/11 (kế hoạch ban đầu tính từ 6/11, còn thực tế hôm nay là 8/11) **[Lỗi thời]** `splits.json` đã được tạo; bản hiện hành là bản chia theo đoạn văn ở Giai đoạn 5.
- Dương và Khánh chưa chạy notebook khởi động của mình **[Chuyển sang Giai đoạn 5]** Dương đã chạy bộ nạp dữ liệu trên chia tập cũ; chưa rõ Khánh; cả hai phải nạp lại bản mới.
- Chưa chốt tập test thật khoảng 90 dòng; nếu làm thì cần xong khoảng 15/11 để Dương chạy OCR **[Chuyển sang Giai đoạn 5]**
- `00_prepare_data.ipynb`: ô kho văn bản dùng `os` nhưng chỉ import ở ô tải dữ liệu, nên chạy lại riêng ô này trong phiên mới sẽ lỗi `NameError`; đã thêm `import os` vào ô đó, cần tải bản mới lên Drive `notebooks/` **[Đã xong]** ô kho văn bản đã chạy được trong notebook bản mới.
- Chạy lại ô 5 của `00_prepare_data.ipynb` (bản mới, có `--group-regex`) và dán kết quả, nhất là `check_text_overlap` (nếu nhiều dòng val hoặc test trùng chữ với train thì phải chia lại theo nhóm văn bản) và `unlabeled_examples` (44 ảnh không nhãn: thiếu khóa trong `label.json` hay nhãn rỗng) **[Đã xong ở Giai đoạn 5]** kết quả `check_text_overlap` dẫn tới Bug 9; `unlabeled_examples` cho `no_label_entry 44` và `empty_label 0` (thiếu khóa trong `label.json`, ví dụ ảnh 10 đến 14 của thư mục 10); **nguyên nhân chi tiết chuyển sang Giai đoạn 5**.
- Báo Dương: `splits.json` sẽ đổi ở tập val (test và train chính thức giữ nguyên); nếu bạn ấy đã chạy huấn luyện thì kết quả val cũ không so sánh được **[Chuyển sang Giai đoạn 5]** `splits.json` đổi ở cả train và val (test giữ nguyên) và mô hình huấn luyện trên train cũ phải huấn luyện lại.
- Hỏi Khánh về `train.txt` / `val.txt` / `test.txt` (194 MB) và hai tệp đánh giá trong Drive: kho nào là kho chính, rồi sửa README và `ASSIGNMENTS.md` cho đúng **[Chuyển sang Giai đoạn 5]** kèm chạy `check_text_overlap.py --corpus` với `train.txt`.
- Tập test chỉ 201 dòng (2,8%) nên CER trên test dao động nhiều; cân nhắc báo kèm khoảng tin cậy **[Chuyển sang Giai đoạn 5]** nay test có 5 đoạn văn và val có 9 đoạn văn.
- Khánh xác nhận hai quy tắc làm rõ của `correct(text)` (cùng số âm tiết, chỉ đổi dấu); Hiển chạy `validate_dataset_index.py` trên `dataset_index.jsonl` và `splits.json` thật rồi đưa các file hợp đồng và mã mới vào repo **[Một nửa đã xong]** `validate_dataset_index.py` đã chạy trên file thật và báo OK (Giai đoạn 5); **phần Khánh xác nhận hai quy tắc chuyển sang Giai đoạn 5.**
- Gộp `prepare_corpus.py` mới vào `main`, chạy lại ô đầu tiên (để clone bản mới) và ô kho văn bản của `00_prepare_data.ipynb`, rồi kiểm tra `wc -l` bằng 300000 **[Đã xong một phần]** ô kho văn bản đã chạy lại và ghi 300.000 câu; chưa xem lại nội dung kho sau khi sửa tách câu, **chuyển sang Giai đoạn 5.**
- Gửi đầu ra của `build_index.py` và `make_splits.py` để kiểm tra: số ảnh có nhãn so với tổng, `split_counts`, chế độ chia, số người viết. Nếu chế độ là chia theo `id` mà tên tệp có chứa ID người viết thì chia như vậy rò rỉ nét chữ giữa các tập, phải chạy lại `build_index.py` với `--writer-regex` **[Đã xong]** đầu ra đã được gửi và đọc: chia theo `group_id`, không có số người viết (`writers 0`); xem Bug 9.
- Hiển phải đọc và giải thích được `align.py`, `classify_errors.py` và các công cụ dữ liệu trước vấn đáp; cần một script in các cặp ký tự bị OCR nhầm nhiều nhất cho Khánh (trước 15/11) **[Chuyển sang Giai đoạn 5]** thêm `check_text_overlap.py` và `make_passage_splits.py`.

## Giai đoạn 5 — Đo trùng văn bản và chia tập theo đoạn văn
**Mục tiêu giai đoạn:** Sau khi đã chia val theo trang, biết văn bản của val và test có lọt vào train hay không; nếu có thì sửa cách chia để CER được đo trên văn bản mô hình chưa thấy, và kiểm tra luôn kho văn bản của vai B.

**Đã làm:**

- Hiển chạy thật ô 5 của `00_prepare_data.ipynb` với `--group-regex`: `build_index.py` ghép 7.229 trên 7.273 ảnh (44 ảnh không có khóa trong `label.json`, ví dụ ảnh 10 đến 14 của thư mục 10; chưa rõ nguyên nhân); `make_splits.py` chạy ở chế độ `official+carved_val_by_group` cho train 6.234 (219 trang), val 794 (30 trang), test 201 (6 trang), không trang nào nằm ở hai tập; `validate_dataset_index.py` báo OK trên file thật.
- `check_text_overlap.py` không có trong 13 file đã giao nên ô 5 báo lỗi thiếu file (Bug 10); viết lại từ mô tả: chuẩn hóa NFC, chữ thường, bỏ dấu câu; ba chỉ số (trùng nguyên dòng, có ít nhất 1 chuỗi 5 từ trùng, có từ 50% cửa sổ 5 từ trở lên trùng); thống kê theo từng trang kèm trang train trùng nhiều nhất; tùy chọn `--corpus` để so với kho văn bản đọc từng dòng; 18 test.
- Đo thật trên chia theo trang: val trùng nguyên dòng 19,8%, có chuỗi 5 từ trùng 97,0%, từ 50% cửa sổ trở lên 96,4% (768 dòng đủ dài); test lần lượt 25,4%, 99,0%, 96,9% (192 dòng đủ dài). Trang test 255 trùng nguyên dòng 91,7% và trang 254 là 71,4% với các trang train.
- So với kho Wikipedia 300.000 câu của Khánh: không dòng val hay test nào trùng nguyên văn; từ 50% cửa sổ trở lên chỉ 0,5% (val) và 0,0% (test); chỉ số "có ít nhất 1 chuỗi" là 19,9% và 34,4% (cụm từ thông dụng). Kho này sạch. Chưa kiểm `train.txt` riêng của Khánh.
- Phân tích cụm trang theo văn bản (chạy ở môi trường riêng, không tạo file cho repo): 255 trang chỉ có 81 đoạn văn, kết quả giống nhau với mọi ngưỡng liên kết từ 0,2 đến 0,6. 32 trang không có bản chép nào; 49 đoạn có từ 2 đến 21 trang; hai đoạn lớn nhất (21 và 20 trang) là đoạn của trang test 254 và 255. Năm đoạn chứa trang test (251 và 252 chung một đoạn), kèm 43 trang train (1.023 trên 7.028 dòng, 14,6%). Với val cũ, 94 trong 219 trang train còn lại vẫn chung đoạn văn với val.
- Đọc hợp đồng dữ liệu (`validate_dataset_index.py`, `contract_checks.py`, `contracts/README.md`): `check_splits` buộc `splits.json` phủ đúng các id của chỉ mục, và `INDEX_OPTIONAL` không cho thêm trường mới như `passage_id`. Đây là ràng buộc quyết định cách biểu diễn các trang bị loại.
- Viết `scripts/make_passage_splits.py`: gom trang thành đoạn văn (hai trang cùng đoạn khi tỉ lệ cửa sổ 5 từ chung, so với trang nhỏ hơn, từ 0,4 trở lên, nối bắc cầu bằng union-find); giữ test chính thức; loại các trang train chung đoạn với test; chọn nguyên đoạn văn làm val theo hạt giống 42 cho đến khoảng 10% số dòng còn lại; ghi `dataset_index.jsonl` (đã lọc), `splits.json` và `passages.json`; tự kiểm tra không đoạn nào nằm ở hai tập, id test trùng test chính thức và các tập phân hoạch đúng chỉ mục; 24 test, gồm một test chạy kết quả qua `check_splits` thật.
- Kết quả thật sau khi chia theo đoạn văn: 81 đoạn (5 đoạn chứa test); train 176 trang, 5.369 dòng, 67 đoạn; val 30 trang, 636 dòng, 9 đoạn; test 6 trang, 201 dòng, 5 đoạn; loại 43 trang, 1.023 dòng; chỉ mục 6.206 dòng, 212 trang; `validate_dataset_index.py` báo OK.
- Đo lại: val trùng nguyên dòng 0,2%, có chuỗi 5 từ trùng 4,7%, từ 50% cửa sổ trở lên 0,2% (621 dòng đủ dài); test lần lượt 0,0%, 5,7%, 2,1% (192 dòng đủ dài); cả hai ở mức KEEP. Còn lại một chút: trang test 253 có 7,1% dòng trùng nhiều với trang train 153 (35 cửa sổ) và trang 250 có 2,8%.
- Sửa `check_text_overlap.py` để cột "top train page" ổn định khi hòa (Bug 11), kiểm lại bằng cách đảo thứ tự nạp trang train theo cả 6 cách.
- Sửa `00_prepare_data.ipynb`: ô 5 cho `build_index.py` ghi `dataset_index_all.jsonl`, chạy `make_passage_splits.py`, rồi kiểm tra bằng `validate_dataset_index.py` và `check_text_overlap.py`; ô 6 thêm lệnh `check_text_overlap.py --corpus`; cập nhật phần đầu và mô tả ô 5. Thêm mục "Which lines the two files contain (since 12 Nov)" vào `contracts/README.md` và đổi tên script tạo hai hợp đồng đầu.
- Đọc "Danh mục 20 bài tập lớn" (TS. Phạm Văn Khánh, 12/11/2026, Hiển gửi). Đề tài của nhóm không nằm trong 20 đề; theo Hiển, thầy đã cho phép. Danh mục chấm thang 100: Phần A cài đặt 20, B chứng minh 25, C bài tập 15, D ứng dụng thực tiễn 30, báo cáo và bảo vệ 10; Phần D cần ít nhất một khâu thu thập và gán nhãn dữ liệu, tinh chỉnh hoặc học tăng cường; Phần A dùng "NumPy hoặc PyTorch"; phải khai báo việc dùng công cụ AI tạo sinh trong phụ lục và hội đồng có thể vấn đáp từng thành viên về mọi chứng minh hay đoạn mã. Dự án ánh xạ gần nhất vào các Đề 6, 2, 3 (quy tắc từ chối ở Đề 20). Chưa xác nhận danh mục này áp dụng nguyên cho đề ngoài danh mục, nhất là Phần B và C.
- Dương đã chạy bộ nạp dữ liệu trên chia tập cũ (train 6.279, val 749, test 201; 50 ảnh đầu không thiếu); chưa nạp lại bản mới.
- Hiển báo thầy đồng ý dùng PyTorch autograd; cập nhật bảng Tech Stack, Cổng 0 và các ghi chú liên quan.

**Quyết định kỹ thuật:**

- **Vấn đề:** đo sự trùng văn bản giữa val/test và train theo cách nào
- **Đã chọn:** trùng nguyên dòng, trùng chuỗi 5 từ và thống kê theo từng trang; quy tắc ba mức dựa trên tỉ lệ dòng có từ 50% cửa sổ 5 từ trở lên trùng
- **Vì sao:** chỉ trùng nguyên dòng bỏ sót cùng đoạn văn nhưng khác chỗ ngắt dòng (như 251 và 252); chỉ cần một chuỗi 5 từ trùng thì cụm quen thuộc trong tin tức làm số cao giả, nên dùng ngưỡng 50% cửa sổ cho quy tắc và vẫn in cả hai số. Bỏ dấu câu trước khi so vì nhãn tách dấu câu thành từ riêng còn Wikipedia thì dính liền

- **Vấn đề:** bao nhiêu phần trăm trùng thì phải chia lại
- **Đã chọn:** đặt mốc trước khi nhìn số: dưới 10% giữ nguyên và ghi chú; từ 10% đến 25% giữ nguyên và báo CER tách theo dòng trùng và không trùng; trên 25% gom trang cùng đoạn văn rồi chia lại
- **Vì sao:** đặt mốc sau khi thấy số thì dễ chọn mốc cho vừa kết quả. Hai mốc là đề xuất chưa có căn cứ thực nghiệm; kết quả 96% đủ rõ nên mốc không ảnh hưởng kết luận. Hành động đề xuất ban đầu cho test (bỏ dòng trùng khỏi train hoặc báo CER tách) hóa ra sai tiền đề: đã cho rằng bỏ dòng trùng sẽ xóa gần hết train, thực tế trùng với test chỉ đến từ 43 trang (14,6%), và không dòng test nào "không trùng" để tách

- **Vấn đề:** 43 trang train chung đoạn văn với test: loại hay giữ
- **Đã chọn:** loại khỏi train và khỏi val, giữ nguyên test chính thức
- **Vì sao:** cả 6 trang test đều có bản chép trong train, nên chỉ có cách loại mới cho test sạch về văn bản. Đổi lại train mất 14,6% số dòng và `splits.json` đổi lần thứ ba. Phương án còn lại (giữ và ghi rõ CER test đo trên văn bản đã thấy) không cho một dòng test sạch nào

- **Vấn đề:** cài đặt chia theo đoạn văn bằng script mới hay sửa trực tiếp `make_splits.py`
- **Đã chọn:** script mới `make_passage_splits.py` trước, gộp logic vào `make_splits.py` sau hạn nộp
- **Vì sao:** sửa `make_splits.py` kéo theo sửa `test_pipeline_tools.py` (đang kiểm tra hành vi "val lấy từ các trang cuối") và có nguy cơ làm vỡ test dưới áp lực thời gian. Đánh đổi: tạm có hai script cùng ghi `splits.json`, dễ nhầm nếu không ghi chú

- **Vấn đề:** biểu diễn các trang bị loại trong hợp đồng
- **Đã chọn:** lọc cả chỉ mục (`dataset_index_all.jsonl` giữ đủ, `dataset_index.jsonl` và `splits.json` chỉ chứa dòng được dùng), `passages.json` ghi trang bị loại; định dạng hợp đồng không đổi
- **Vì sao:** `check_splits` buộc `splits.json` phủ đúng id của chỉ mục, nên không thể chỉ bỏ id khỏi `splits.json`; thêm giá trị `excluded` phải đổi `SPLITS`, schema, validator và bộ nạp của Dương và Khánh, lại cần cả ba người đồng ý

- **Vấn đề:** khi nào báo Dương và Khánh
- **Đã chọn:** báo một lần duy nhất sau khi số liệu chia tập mới khớp
- **Vì sao:** `splits.json` đã đổi hai lần trong ngày, báo từng lần làm họ bỏ kết quả nhiều lần. Kèm cảnh báo: mô hình nào đã huấn luyện trên train cũ đã thấy bản chép của đoạn văn test nên CER test cũ cũng lạc quan, phải huấn luyện lại

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 9 — Chia theo trang vẫn để lộ văn bản val và test:** `check_text_overlap` trên chia theo trang báo 96,4% dòng val và 96,9% dòng test có từ 50% cửa sổ 5 từ trùng với train, trùng nguyên dòng 19,8% và 25,4% → Nguyên nhân thật sự: UIT-HWDB-line có nhiều trang do người khác nhau chép cùng một đoạn văn (255 trang chỉ có 81 đoạn; hai đoạn 21 và 20 trang là đoạn của trang test 254 và 255, mỗi trang có khoảng 19 đến 20 bản chép trong train); chia theo trang chặn được rò rỉ nét chữ nhưng không chặn rò rỉ văn bản → `make_passage_splits.py` chia theo đoạn văn và loại 43 trang chung đoạn với test; đo lại còn 0,2% (val) và 2,1% (test)
- **Bug 10 — `check_text_overlap.py` và test của nó không nằm trong gói file đã giao:** ô 5 báo `can't open file '/content/repo/scripts/check_text_overlap.py'` → Nguyên nhân thật sự (suy từ danh sách file gửi): hai file được tạo ở một lượt trước nhưng không có trong 13 file giao cho Hiển nên không thể chép vào repo → viết lại từ mô tả theo quy tắc đo đã thống nhất, thêm `--corpus` và 18 test
- **Bug 11 — Cột "top train page" đổi giữa hai lần chạy trên cùng dữ liệu:** trang test 255 trùng nhiều nhất với trang train 137 ở lần chạy này và trang 43 ở lần chạy kia, cùng 311 cửa sổ; trang 254 cũng đổi (61 rồi 109) → Nguyên nhân thật sự: nhiều trang train có cùng văn bản nên hòa số cửa sổ, mà thứ tự duyệt lấy từ tập (`set`) không cố định giữa các lần chạy → khi hòa chọn trang có số nhỏ nhất và in `+N tied`; thử đảo thứ tự nạp theo cả 6 cách đều ra cùng kết quả; các tỉ lệ phần trăm không bị ảnh hưởng
- **Bug 12 — Ô 5 chạy tiếp trên file cũ khi script mới chưa có trên `main`:** Colab in `can't open file '/content/repo/scripts/make_passage_splits.py'` nhưng các lệnh sau vẫn chạy và in `OK` và `RESPLIT` của chỉ mục cũ (train 6.234, val 794), dễ tưởng là kết quả bản mới → Nguyên nhân thật sự: ô 1 clone nhánh `main` mà lúc đó `main` chưa có script mới; lệnh `!` trong Colab không dừng khi một lệnh lỗi; `dataset_index.jsonl` và `splits.json` bản cũ vẫn nằm trên Drive nên các lệnh sau đọc được → gộp file vào `main`, chạy lại ô 1 và ô 5 (cho kết quả đúng); chưa sửa gốc, xem tồn đọng

**Việc còn tồn đọng từ giai đoạn này (đã gộp lại các mục chưa xong của Giai đoạn 4; các mục Giai đoạn 4 đã xong hoặc lỗi thời gồm: chạy `00_prepare_data.ipynb` trên Colab thật, dán `check_text_overlap` và `unlabeled_examples`, chạy `validate_dataset_index.py` trên file thật, chạy lại ô kho văn bản (đã chạy, chưa xem lại nội dung kho), gửi đầu ra của `build_index.py` và `make_splits.py`, `import os` trong ô kho văn bản, và hạn `splits.json` 9/11):**

- Báo Dương một lần: `dataset_index.jsonl` và `splits.json` đã đổi (train 5.369, val 636, test 201), nạp lại; CER val cũ bỏ; mô hình nào huấn luyện trên train cũ phải huấn luyện lại vì CER test cũ cũng lạc quan
- Báo Khánh: đánh giá `correct()` trên val và test mới; chạy `check_text_overlap.py --corpus` với `train.txt` của Khánh; chốt kho văn bản chính (kho 300.000 câu đã kiểm, sạch) rồi sửa README và `ASSIGNMENTS.md`; xác nhận hai quy tắc của `correct(text)` (cùng số âm tiết, chỉ đổi dấu)
- Xem 10 đến 20 dòng đầu của `data/corpus/viwiki_sentences.txt` để xác nhận bản `prepare_corpus.py` đã sửa tách câu (không cắt câu sau chữ cái đầu như "S.") là bản đang được dùng trên `main`
- Chưa rõ vì sao 44 ảnh (ví dụ ảnh 10 đến 14 của thư mục 10) không có nhãn: cần xem `label.json` của thư mục 10 (số khóa, tên khóa) để biết là thiếu khóa hay khác định dạng khóa
- Chốt ngưỡng cổng giả thuyết trước khi nhìn số OCR thật (hạn ban đầu 12/11, nhóm chưa chốt): đề xuất đặt đồng thời tỉ lệ lỗi dấu trên tổng lỗi (p) và trần lợi ích (CER nhân p), hành động nếu không đạt, và thống nhất lỗi "cả hai" có tính là lỗi dấu không
- Chốt tập test thật khoảng 90 dòng (cần xong khoảng 15/11 để Dương chạy OCR): nay có thêm lý do vì val chỉ có 9 đoạn và test chỉ có 5 đoạn, nên chỉ một tập viết mới (văn bản ngoài UIT-HWDB và loại khỏi kho của Khánh) mới đo được khả năng tổng quát hóa sang văn bản mới; cần văn bản đồng ý và ẩn danh; tập này cũng đáp ứng yêu cầu thu thập và gán nhãn của Phần D
- Hỏi thầy xem đề ngoài danh mục có phải nộp Phần B (chứng minh) và Phần C (bài tập) không; nếu có thì chưa có gì trong kế hoạch cho hai phần này (có thể mượn các mệnh đề về quy tắc ngưỡng Bayes theo chi phí và cận Hoeffding của Đề 2 và Đề 3); chuẩn bị phụ lục khai báo việc dùng công cụ AI tạo sinh
- Gộp logic `make_passage_splits.py` vào `make_splits.py` sau hạn nộp, sửa `test_pipeline_tools.py` tương ứng và bỏ script trùng để khỏi có hai nguồn ghi `splits.json`
- Làm cho `00_prepare_data.ipynb` dừng khi một lệnh lỗi (Bug 12), ví dụ kiểm tra file tồn tại trước khi chạy tiếp hoặc chạy lệnh trong khối `bash` với `set -e`
- Ghi vào báo cáo: val 9 đoạn và test 5 đoạn nên CER có độ bất định lớn (báo kèm khoảng tin cậy, bootstrap theo đoạn văn); trang test 253 còn 7,1% dòng trùng nhiều với trang train 153
- Hiển phải đọc và giải thích được `align.py`, `classify_errors.py`, `check_text_overlap.py` và `make_passage_splits.py` trước vấn đáp; cần một script in các cặp ký tự bị OCR nhầm nhiều nhất cho Khánh (trước 15/11)
- Lưu bằng chứng việc thầy đồng ý autograd (ảnh chụp tin nhắn hoặc email) vào Drive hoặc `docs/`, để dán vào phụ lục báo cáo.
- Xác nhận phạm vi: `nn.Conv2d`, `nn.LSTM`, `torch.optim.Adam` có được dùng không, và `nn.CTCLoss` chỉ để đối chiếu với hàm tự cài. Thầy mới nói autograd.

## Giai đoạn 6 — Đo OCR mới và cổng giả thuyết
**Mục tiêu giai đoạn:** Có công cụ đo cho vai C chạy được trên dữ liệu thật, đo chất lượng file OCR mới của Dương, và trả lời cổng giả thuyết (lỗi dấu chiếm bao nhiêu phần lỗi OCR, trần lợi ích bao nhiêu) bằng ngưỡng đã chốt trước khi nhìn số, để biết có nên đầu tư tiếp vào lớp sửa dấu hay không.

**Đã làm:**

- Quan sát các lần xuất `ocr_out.jsonl` trước bản cuối. Bản đầu của Dương chạy trên chia tập cũ (train 6.279, val 749, test 201), overfit thử 50 dòng chỉ xuống loss 1,33 và huấn luyện 3 epoch còn loss 2,59. Hiển chạy lại notebook của Dương trên chia tập mới (ô nạp in train 5.369, val 636, test 201) để thay file lệch chia tập; file này đủ 636 dòng val, không trùng id, đúng NFC, nhưng CER 52,58%, WER 97,59%, 208/636 dòng (32,7%) có khoảng trắng bất thường, OCR có 35.703 ký tự so với 43.727 ký tự nhãn (thiếu khoảng 18%), chỉ 238/636 dòng (37,4%) cùng số âm tiết với nhãn. Đây là bản nháp, không dùng cho cổng giả thuyết.
- Dương huấn luyện lại CRNN trên chia tập mới (train 5.369; từ vựng 154 lớp gồm blank): 6 tầng tích chập (64, 128, 256, 256, 512, 512 kênh, BatchNorm ở ba tầng, MaxPool), 2 tầng BiLSTM ẩn 256 chiều với dropout 0,2; ảnh co về cao 32 pixel, đệm bằng nền trắng; độ dài đầu vào của CTC tính riêng cho từng ảnh bằng `(w // 4) + 1`; batch 32, AdamW (lr 1e-3, weight decay 1e-4), `ReduceLROnPlateau` (hệ số 0,5, kiên nhẫn 3) theo val CER, cắt gradient 5,0; 25 epoch, loss train từ 3,4453 xuống 0,0408, val CER (greedy) từ 0,8242 xuống 0,1237 và tốt nhất ở epoch 25 (lưu `best_crnn.pth` và `last_crnn.pth` trong `checkpoints/ocr/`). Hàm mất mát là `nn.CTCLoss` của PyTorch; phần tự cài bằng NumPy là giải mã: greedy và CTC prefix beam search (beam 10, 15 ký tự mạnh nhất mỗi khung).
- Dương xuất `ocr_out_greedy.jsonl`, `ocr_out_beam.jsonl` và `ocr_out.jsonl` (bản sao của beam) vào `outputs/`, mỗi file 837 dòng (val 636 và test 201; dòng test mới chỉ suy luận, chưa chấm). Val CER greedy 0,1237, beam 0,1227. Lần xuất đầu `validate_ocr_out.py` báo lỗi ở cả 837 dòng (Bug 13); sau khi sửa báo OK cho cả hai file.
- Xác nhận từ Dương (theo Hiển, 18/11/2026): lần chạy OCR hiện tại (beam, 25 epoch) là lần cuối; Dương không tinh chỉnh gì theo test; code đã push lên `main`; Dương có thể xuất `ocr_conf` cả hai mức nếu được giao. Vì OCR đã chốt, kết quả cổng giả thuyết bên dưới (lỗi dấu 11,3%, trần lợi ích 1,38 điểm CER, FAIL) là kết quả chính thức của file OCR này.
- Hiển soạn và chạy `eval_00_start.ipynb` (nhánh `c/demo`, 11 phần; mã do trợ lý AI viết theo các quyết định Hiển chốt, Hiển cần đọc hiểu từng ô trước vấn đáp): nối Drive và GitHub; cấu hình đặt trước khi nhìn số; nạp dữ liệu kèm kiểm tra phủ id, nhãn, NFC, đoạn văn; CER/WER thô và chuẩn hóa khoảng trắng; phân loại lỗi và cổng giả thuyết; `correct()` giả kèm kiểm tra hợp đồng; bảng sửa đúng/sửa hỏng đếm hai cách; quét τ trên val với hai ngưỡng sửa hỏng; bootstrap theo đoạn văn và theo trang; chấm test chỉ một lần; ghi `outputs/eval_val{hậu tố}.json`. Kiểm thử trong môi trường giả lập trước (dùng `align.py`, `classify_errors.py`, `contract_checks.py`, `passages.json` thật): chạy hết 11 phần, đường chấm test chạy được lần đầu và bị từ chối ở lần hai hoặc khi thiếu `eval_tau.json`; sau đó Hiển chạy trên Colab thật.
- Kết quả đo file OCR mới (val 636 dòng, 9 đoạn, 30 trang): kiểm tra phủ id, nhãn, NFC và số dòng từng đoạn đều qua; chỉ 2/636 dòng (0,3%) có khoảng trắng bất thường; CER 12,27% và WER 34,30% (thô và chuẩn hóa khoảng trắng bằng nhau; CER 12,27% khớp val CER beam của Dương); 0 dòng rỗng; 441/636 dòng (69,3%) cùng số âm tiết với nhãn; OCR có 43.170 ký tự so với 43.727 ký tự nhãn. Bảng sửa đúng/sửa hỏng: Option 1 bỏ 30,7% số dòng và dùng 6.755 âm tiết (67,2% âm tiết của nhãn), Option 2 dùng 9.888 âm tiết (98,3%).
- Cổng giả thuyết (ngưỡng đặt trước khi chạy ô 5, xem Quyết định kỹ thuật): trên val có 5.365 lỗi ký tự, gồm dấu thanh 361, dấu mũ-móc 142, cả hai 101, chữ cái nền 4.413 và khoảng trắng 348. Tỉ lệ lỗi dấu là (361 + 142 + 101) / 5.365 = 11,3% (ngưỡng 50%); trần lợi ích là 12,27% × 11,3% = 1,38 điểm CER (ngưỡng 3 điểm). **FAIL ở cả hai điều kiện**, cả bản thô lẫn bản chuẩn hóa khoảng trắng (bản chuẩn hóa chỉ khác 1 lỗi dấu thanh). Lỗi chữ cái nền chiếm 82,3% tổng lỗi và khoảng trắng 6,5%. Nếu chỉ đếm theo nhãn của cổng thì CER val xuống khoảng 10,89%, nhưng đó là ước lượng theo một căn chỉnh và trần thật có thể lớn hơn (xem Bug 15 và mục 5d).
- Phân tích thăm dò ở mức âm tiết (mục 5b của `eval_00_start.ipynb`, làm sau khi đã biết kết quả cổng nên không đổi kết luận cổng; kết quả nằm trong `outputs/eval_val_FAKE.json` và không phụ thuộc `correct()` giả): trên 10.059 âm tiết của nhãn có 6.712 âm tiết khớp, 364 âm tiết bị thay chỉ khác dấu, 2.812 bị thay khác chữ cái nền, 171 thiếu và 103 thừa (tổng 3.450 lỗi, đúng bằng WER 34,30%). Âm tiết chỉ khác dấu chiếm 10,6% tổng lỗi âm tiết, gần bằng 11,3% ở mức ký tự, nên kết luận FAIL đứng vững ở cả hai mức (dự đoán ban đầu rằng tỉ lệ ở mức âm tiết có thể cao hơn không được xác nhận). Mức giảm nếu sửa hết các âm tiết chỉ khác dấu (cận dưới của trần, xem Bug 15): WER giảm ít nhất 3,62 điểm (34,30% xuống 30,68%), CER giảm ít nhất 0,85 điểm; số 1,38 điểm của cổng cũng chỉ là cận dưới chứ không phải cận trên; số dòng đúng hoàn toàn tăng từ 26 lên 36 trên 636 dòng (4,1% lên 5,7%). Giới hạn còn lại: val chỉ có 9 đoạn
- Các ô 7 đến 9 (bảng sửa đúng/sửa hỏng, quét τ, bootstrap) mới chạy với `correct()` giả nên chỉ kiểm tra pipeline, không phải kết quả: bản giả chỉ sửa các dòng cùng số âm tiết nên Option 1 và Option 2 cho cùng số sửa đúng (189) và sửa hỏng (462); hai ngưỡng 1% và 2% cùng chọn τ = 0,60 vì bản giả gán độ tin cậy dưới 0,6 cho chữ bị phá. File ghi ra có hậu tố `_DRAFT_FAKE` (lần chạy với `OCR_IS_DRAFT = True`) và `_FAKE` (lần chạy có mục 5b, `OCR_IS_DRAFT = False`); `USE_FAKE` vẫn True nên số ở hai file giống nhau và `eval_val_FAKE.json` là file dẫn chiếu. Kết quả cổng không phụ thuộc `correct()` giả nên vẫn dùng được.
- Tìm vị trí `correct()` thật (theo kết quả `git grep` trên `origin/b/corrector` do Hiển chạy, 19/11/2026): `correct(text)` nằm ở `src/corrector/api.py` (dòng 48); các lớp có phương thức `.correct` ở `src/corrector/correctors.py` (ba lớp) và `src/corrector/tagger.py`; cùng thư mục có `ngram.py`, `viterbi.py`, `calibration.py`, `noise.py`, `evaluate.py`, năm file `tests/test_corrector_*.py` và ba notebook `corrector_01` đến `corrector_03`; `src/stubs/identity_corrector.py` là bản giả của bộ khung. Commit cuối của nhánh là ngày 09/11/2026 (Khánh) và chưa merge vào `main`, nên bản clone `c/demo` chưa có `src/corrector`. Chưa biết `correct()` nạp những file nào và dùng mức mô hình nào mặc định: chờ Khánh trả lời và nội dung `api.py`
- Hiển đã đẩy toàn bộ code mới nhất lên GitHub (theo Hiển). PR #11 (`c/demo` vào `main`) đã merge từ 17/11/2026 lúc 14:32, trước khi có mục 5b, 5c và `print_confusions.py`, nên các thay đổi đó cần commit mới
- Thư trả lời của thầy (theo Hiển): thầy chỉ phản hồi một điều, rằng nhóm cần đảm bảo đúng các yêu cầu của tài liệu danh mục 20 bài tập lớn. Các yêu cầu áp dụng cho dự án: bốn phần A, B, C, D đều bắt buộc (điểm 20, 25, 15, 30, báo cáo và bảo vệ 10); Phần A cài từ đầu bằng NumPy hoặc PyTorch, không dùng hàm huấn luyện dựng sẵn và đối chiếu kết quả với thư viện chuẩn; Phần D được dùng thư viện chuẩn nhưng ít nhất một cài đặt của Phần A phải tham gia so sánh hoặc dùng trực tiếp; kết quả báo dạng trung bình ± độ lệch chuẩn trên ít nhất 5 lần chạy hoặc 5 phần gập, siêu tham số chỉ chọn trên tập kiểm định; dữ liệu có yếu tố con người chỉ thu thập khi có văn bản đồng ý và phải ẩn danh trước khi lưu; sản phẩm gồm báo cáo LaTeX 25 đến 40 trang, kho Git kèm mô tả môi trường và tập lệnh tái lập, datasheet và model card; việc dùng AI phải khai báo trong phụ lục và hội đồng có thể vấn đáp từng thành viên về mọi chứng minh hay đoạn mã. Câu cuối của tài liệu bị cắt khi sao chép ("Không dùng bản AI miễn phí..."), cần đọc đủ

**Quyết định kỹ thuật:**

- **Vấn đề:** chọn ngưỡng nào cho cổng giả thuyết. Hai ngưỡng: P_MIN (tỉ lệ lỗi dấu trên tổng lỗi) với các phương án 0,4; 0,5; 0,6, và CEILING_MIN (CER nhân p, tính bằng điểm CER) với các phương án 2; 3; 5 điểm. Hai ngưỡng không độc lập vì trần lợi ích bằng CER nhân p: với CER 12,27%, 2 / 3 / 5 điểm tương đương p tối thiểu khoảng 16% / 24% / 41%
- **Đã chọn:** P_MIN = 0,5 và CEILING_MIN = 3 điểm CER (0,03 trong mã); lỗi "cả hai" (sai cả dấu thanh lẫn dấu mũ-móc) tính là lỗi dấu, đúng với `classify_errors.py` hiện tại
- **Vì sao:** các phương án đã cân nhắc trước khi chọn: P_MIN 0,4 dễ đạt nhưng thấp hơn một nửa, nên cổng có thể PASS mà giả thuyết "lỗi dấu chiếm phần lớn" vẫn sai, khó bảo vệ khi vấn đáp; 0,5 khớp nghĩa đen của "phần lớn" (hơn một nửa) và dễ giải thích, nhược điểm là số tròn tùy ý và p = 0,47 sẽ FAIL dù trần lợi ích vẫn lớn; 0,6 cho bằng chứng mạnh nếu đạt nhưng mẫu số gồm cả lỗi chữ cái nền và khoảng trắng nên rất dễ loại oan. CEILING_MIN 2 điểm không bỏ cuộc sớm nhưng với CER này chỉ chặn khi OCR rất tốt, và nếu sửa dấu chỉ được khoảng một nửa trần thì lợi ích thật khoảng 1 điểm, khó tách khỏi nhiễu khi val chỉ có 9 đoạn; 3 điểm cân bằng hơn (suy đoán "sửa được khoảng một nửa trần" chưa có số thật); 5 điểm chặt nhưng với CER này gần trùng với P_MIN 0,4 và dễ loại oan khi OCR tốt lên. Đề xuất cặp 0,5 và 3 điểm do trợ lý AI đưa ra, Hiển chọn ngày 17/11/2026 trước khi chạy ô 5; ô 5 giấu toàn bộ số cho đến khi cả hai ngưỡng được đặt. Trước khi chọn, Hiển đã thấy CER, WER và vài dòng mẫu của file OCR mới (ô 4) và vài thống kê ký tự của file OCR nháp trước đó, nhưng chưa thấy tỉ lệ lỗi dấu hay trần lợi ích của file mới. Theo số đếm theo nhãn của cổng, kết quả không phụ thuộc vào lựa chọn này: p = 11,3% thấp hơn cả mức 0,4 và trần lợi ích 1,38 điểm thấp hơn cả mức 2 điểm, nên mọi cặp ngưỡng trong các phương án trên đều FAIL (cần kiểm lại bằng trần chính xác ở mục 5d, xem Bug 15)

- **Vấn đề:** nếu cổng giả thuyết FAIL thì làm gì
- **Đã chọn:** vẫn hoàn thành pipeline và báo như một kết quả âm tính (quyết định ngày 17/11/2026, trước khi chạy ô 5)
- **Vì sao:** câu hỏi nghiên cứu là đo, kết quả âm tính vẫn trả lời được ("sửa dấu giảm tối đa bao nhiêu điểm CER, lỗi chủ yếu là loại gì"); công cụ đo đã xong nên chạy tiếp chỉ cần thay `correct()` giả bằng bản thật; hạn cuối tháng 11 không đủ thời gian đổi hướng; bảng phân loại lỗi giải thích được vì sao sửa dấu bị giới hạn. Hai điều kiện đi kèm: không đổi ngưỡng sau khi thấy p, và phân biệt hai kiểu FAIL (p thấp so với trần lợi ích thấp) trong báo cáo. Đánh đổi: tốn thời gian cho một lớp có trần lợi ích nhỏ; phương án đổi hướng bị loại vì không đủ thời gian và cần cả nhóm đồng ý

- **Vấn đề:** bảng sửa đúng/sửa hỏng đếm thế nào khi số âm tiết của OCR và nhãn khác nhau; khoảng trắng bất thường trong `ocr_text` xử lý thế nào
- **Đã chọn:** đếm cả hai cách và in song song (Option 1: chỉ các dòng cùng số âm tiết, in tỉ lệ dòng bị bỏ; Option 2: căn chỉnh từng từ bằng `align.py`, chỉ đếm các cặp khớp hoặc thay thế); đo khoảng trắng cả bản thô lẫn bản chuẩn hóa, và đưa vào `correct()` bản đã chuẩn hóa
- **Vì sao:** với bản OCR nháp chỉ 37,4% dòng cùng số âm tiết nên Option 1 sẽ bỏ hơn một nửa dòng; với OCR mới là 69,3%, vẫn bỏ 30,7% dòng, đúng ở mốc đã đặt để cân nhắc nâng lên Option 2, còn Option 2 dùng 98,3% âm tiết. In song song cho thấy Option 1 có chệch hay không. Khoảng trắng: bản nháp có 32,7% dòng bất thường nên cần cả hai bản; bản mới chỉ còn 0,3% nên hai bản gần như trùng

- **Vấn đề:** chọn τ trên val và chấm test thế nào để kết quả đáng tin
- **Đã chọn:** quét τ từ 0,00 đến 1,00 (bước 0,05) trên val; so hai ngưỡng sửa hỏng 1% và 2% (chọn τ có CER thấp nhất trong số τ thỏa ngưỡng, hòa thì lấy τ cao hơn; nếu "không sửa" thắng thì kết luận là không nên sửa), rồi đặt `FINAL_CAP` bằng tay để khóa τ; ô chấm test mặc định tắt, chỉ chạy khi có `eval_tau.json`, không còn hậu tố `_DRAFT` hoặc `_FAKE`, và từ chối chạy lần hai khi `eval_test_result.json` đã tồn tại
- **Vì sao:** val sinh ra để chọn tham số, nên so hai ngưỡng trên val là hợp lệ miễn là test chưa bị đụng; chốt chặn trong mã ngăn việc chạy lại test để chỉnh tham số. Báo cáo phải ghi rõ đã so hai ngưỡng trên val rồi mới chọn

- **Vấn đề:** đơn vị lấy mẫu lại khi bootstrap
- **Đã chọn:** đoạn văn là đơn vị chính (9 đoạn val, 5 đoạn test), 1.000 lần lấy mẫu, hạt giống 42, báo khoảng tin cậy 95% của hiệu CER trước và sau; báo thêm kết quả theo trang trên val (30 trang) để đối chiếu
- **Vì sao:** các dòng của một đoạn không độc lập (nhiều người chép cùng văn bản), lấy mẫu theo dòng sẽ cho khoảng tin cậy hẹp giả tạo. Theo trang mịn hơn nhưng các trang cùng đoạn dùng chung văn bản; trên test hai cách gần như trùng (mỗi đoạn chỉ còn 1 trang, riêng một đoạn có 2). Đánh đổi: với 5 đơn vị trên test khoảng tin cậy sẽ rất rộng, đó là kết quả trung thực về độ tin cậy

- **Vấn đề:** chạy thử pipeline khi `correct()` thật của Khánh chưa có
- **Đã chọn:** viết `correct()` giả ngay trong notebook (tự chứa, đúng hợp đồng, nhìn nhãn của chính dòng đang xử lý để tạo đủ các trường hợp sửa đúng, giữ nguyên, sửa hỏng), mọi file kết quả sinh từ bản giả hoặc OCR nháp mang hậu tố `_FAKE` và `_DRAFT`
- **Vì sao:** không phụ thuộc đường dẫn mã của Khánh, và hậu tố cùng chốt chặn ở ô test ngăn việc số từ bản giả lẫn vào kết quả thật. Đánh đổi: bản giả chỉ sửa các dòng cùng số âm tiết nên làm Option 1 và Option 2 cho cùng số sửa đúng và sửa hỏng, không dùng để so sánh hai cách

- **Vấn đề:** xuất `ocr_conf` cả ở mức ký tự lẫn âm tiết như thế nào. Các phương án: (A) thêm trường `ocr_conf_syllable` vào `ocr_out.jsonl`, phải sửa schema và `validate_ocr_out.py` nếu bộ kiểm tra từ chối trường lạ; (B) đổi `ocr_conf` thành đối tượng có `char` và `syllable`, rõ nhất nhưng thay đổi lớn nhất vì mọi nơi đọc `ocr_conf` phải sửa; (C) xuất file riêng `ocr_conf.jsonl`, ghép với `ocr_out.jsonl` bằng `id`, không đổi hợp đồng. Độ tin cậy âm tiết tính từ độ tin cậy ký tự có hai cách: nhỏ nhất hoặc trung bình các ký tự của âm tiết
- **Đã chọn:** phương án C, file `ocr_conf.jsonl` mỗi dòng gồm `id`, `ocr_text` (bản sao để đối chiếu), `char`, `syllable_min` và `syllable_mean`; thử cả hai cách tính âm tiết rồi so sánh trên val, chưa chọn cách nào (quyết định ngày 18/11/2026)
- **Vì sao:** C không đụng vào `ocr_out.jsonl`, `validate_ocr_out.py` và phần đã đo (kết quả cổng), nên ít rủi ro nhất khi bộ đo chưa dùng `ocr_conf`; A và B buộc sửa hợp đồng sát hạn tích hợp. Điều kiện bắt buộc với Dương: dùng đúng checkpoint và giải mã beam hiện tại, và `ocr_text` trong file mới phải giống từng dòng với `ocr_out.jsonl` (nếu khác thì kết quả cổng phải đo lại). Nhỏ nhất thận trọng và hợp để bắt âm tiết có một ký tự đáng ngờ, nhưng một ký tự nhiễu kéo cả âm tiết xuống; trung bình mượt hơn nhưng loãng khi âm tiết dài mà chỉ một chữ sai; chưa có số thật nên thử cả hai. Đánh đổi: thêm một file phải giữ đồng bộ với `ocr_out.jsonl` theo `id`

- **Vấn đề:** sau thư trả lời của thầy, dự án phải đạt đúng tài liệu danh mục. So với tài liệu, dự án còn thiếu: Phần B và C (chưa có trong kế hoạch, chiếm 40 trên 100 điểm); đối chiếu với thư viện chuẩn cho các phần tự cài (hàm mất mát CTC hiện là `nn.CTCLoss` dựng sẵn); kết quả trung bình ± độ lệch chuẩn trên ít nhất 5 lần chạy (hiện chạy một lần kèm khoảng tin cậy bootstrap, hai thứ này khác nhau); văn bản đồng ý và ẩn danh cho dữ liệu người; kiểm điều khoản UIT-HWDB (`report_facts.json` ghi chưa tìm thấy giấy phép); datasheet, model card, báo cáo LaTeX
- **Đã chọn:** lấy tài liệu danh mục làm tiêu chí nghiệm thu, và soạn tin giao việc ngày 19/11/2026. Dương: tự cài CTC loss và kiểm số học với `nn.CTCLoss`, đối chiếu giải mã greedy và beam với thư viện chuẩn, 5 seed nếu kịp (model chính giữ nguyên), Phần B và C cho CTC. Khánh: PR merge `b/corrector` vào `main`, hợp đồng và mức mô hình, kiểm trùng kho văn bản, đối chiếu với thư viện chuẩn, trung bình ± độ lệch chuẩn trên ít nhất 5 lần chạy hoặc 5 phần gập, Phần B và C cho Viterbi bigram, ước lượng hợp lý cực đại và làm mịn, hiệu chỉnh độ tin cậy. Hiển (đề xuất, chờ xác nhận): đối chiếu `align.py` với thư viện chuẩn, Phần B và C cho quy hoạch động Levenshtein, CER, WER và bootstrap, tập test thật, datasheet, model card, báo cáo
- **Vì sao:** Phần B và C chiếm 40 trên 100 điểm và thầy yêu cầu đúng theo đề; tuần 2 phải xong Phần A, B và dữ liệu Phần D nên các hạn rất sát. Bootstrap giữ nguyên làm khoảng tin cậy ở mức đánh giá, còn 5 lần chạy dùng để ước lượng độ dao động của huấn luyện. Đánh đổi: lặp 5 lần huấn luyện OCR tốn GPU (chờ Dương báo thời gian một lần); nếu không kịp huấn luyện lại bằng CTC loss tự cài thì hàm tự cài chỉ dùng để kiểm chứng

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 13 — `validate_ocr_out.py` báo `ocr_conf must be a list` ở cả 837 dòng:** lần xuất đầu của Dương báo lỗi ở từng dòng từ 1 đến 837 → Nguyên nhân thật sự: mã xuất ghi `ocr_conf` là một số thực (độ tin cậy cả dòng), trong khi `ocr_out.schema.json` yêu cầu mảng số → Dương viết lại hàm giải mã để trả về danh sách độ tin cậy rồi xuất lại; `validate_ocr_out.py` báo OK cho cả `ocr_out_greedy.jsonl` và `ocr_out_beam.jsonl`. Chưa rõ danh sách theo ký tự hay theo âm tiết: mã tự nhận diện bằng cách tìm `ocr_conf` trong `contract_checks.py`, mà file này (bản đã gửi) không chứa chữ đó nên nhiều khả năng là theo ký tự (suy luận từ mã, chưa xác nhận với Dương; Dương sẽ xuất độ tin cậy cả hai mức trong file riêng `ocr_conf.jsonl`, xem Quyết định kỹ thuật)
- **Bug 14 — `ocr_out.jsonl` xuất từ chia tập cũ lệch `splits.json` mới:** ô nạp của Dương in `train 6279, val 749, test 201` thay vì 5.369 / 636 / 201, nên file có 749 dòng val không khớp `splits.json` mới → Nguyên nhân thật sự: lúc Dương chạy, Drive còn `dataset_index.jsonl` và `splits.json` bản cũ (cùng họ với Bug 12) → nạp lại bản mới và huấn luyện lại; ô kiểm tra phủ trong `eval_00_start.ipynb` dừng bằng `assert` nếu id không khớp để lỗi này không lọt vào phép đo

**Việc còn tồn đọng từ giai đoạn này (đã gộp lại các mục chưa xong của Giai đoạn 5; các mục Giai đoạn 5 đã xong hoặc lỗi thời gồm: báo Dương nạp lại chia tập mới (Dương đã huấn luyện lại trên train 5.369), chốt ngưỡng cổng giả thuyết, và bootstrap theo đoạn văn (đã có trong `eval_00_start.ipynb`)):**

- Dương đã xác nhận (18/11/2026): OCR hiện tại là bản cuối (khi chạy bộ đo với `correct()` thật thì đặt `OCR_IS_DRAFT = False`), không tinh chỉnh theo test, code đã push lên `main` (nếu `src/` có thay đổi thì merge `main` vào `c/demo` cho khỏi lệch). Còn lại: giao Dương xuất `ocr_conf.jsonl` theo quyết định ở trên (hạn chưa chốt)
- Khi có `ocr_conf.jsonl`: kiểm tra `ocr_text` trong file giống từng dòng với `ocr_out.jsonl`, kiểm tra độ dài và khoảng giá trị [0, 1] của `char`, `syllable_min`, `syllable_mean`; so sánh hai cách tính độ tin cậy âm tiết trên val (độ tin cậy có tách được âm tiết đúng khỏi âm tiết sai không, ví dụ bằng AUROC và bảng tỉ lệ đúng theo nhóm độ tin cậy), rồi ghi kết quả vào nhật ký
- Giao Khánh (tin giao việc soạn 19/11/2026; hạn 20/11 cho phần 1 và 23/11 cho phần 2 là đề xuất, Hiển điều chỉnh): tạo PR merge `b/corrector` vào `main` với test pass và báo dòng import cùng các file mà `correct()` nạp (hiện `correct()` ở `src/corrector/api.py`); báo mức mô hình mặc định và cách chọn từng mức; tự kiểm bằng `check_correct_output` (cùng số âm tiết, chỉ đổi dấu, `conf` một giá trị mỗi âm tiết); chạy `check_text_overlap.py --corpus` với `train.txt` và chốt kho văn bản chính (kho 300.000 câu đã kiểm, sạch) rồi sửa README và `ASSIGNMENTS.md`; theo đề của thầy: đối chiếu từng phần tự cài (n-gram, Viterbi, tagger) với thư viện chuẩn, báo kết quả dạng trung bình ± độ lệch chuẩn trên ít nhất 5 lần chạy hoặc 5 phần gập (cập nhật `corrector_eval.json`), và làm Phần B và C cho phần sửa dấu (tối ưu của Viterbi cho bigram, ước lượng hợp lý cực đại và làm mịn, hiệu chỉnh độ tin cậy, kèm bài tập số học). Sau đó Hiển nối `correct()` vào `eval_00_start.ipynb` (`USE_FAKE = False`, `OCR_IS_DRAFT = False`), quét τ, đặt `FINAL_CAP`, bootstrap và chấm test một lần (file OCR đã có đủ 201 dòng test)
- Giao Dương (tin giao việc soạn 19/11/2026, ngoài việc xuất `ocr_conf.jsonl`; hạn báo kế hoạch 19/11 và nộp 23/11 là đề xuất, Hiển điều chỉnh): tự cài CTC loss (truy hồi forward trong không gian log) và kiểm số học khớp `nn.CTCLoss`, báo dự định dùng để huấn luyện lại hay chỉ để kiểm chứng; đối chiếu bộ giải mã greedy và beam tự cài với một thư viện chuẩn trên val; báo thời gian một lần huấn luyện 25 epoch và khả năng chạy 5 seed (trung bình ± độ lệch chuẩn val CER, model chính giữ nguyên); Phần B và C cho CTC (tính đúng của truy hồi forward, kiểm chứng bằng vét cạn các căn chỉnh trên chuỗi ngắn)
- Phân tích 4.413 lỗi chữ cái nền và viết script in các cặp ký tự bị OCR nhầm nhiều nhất (trước 15/11, đã quá hạn): giải thích vì sao sửa dấu bị giới hạn, đồng thời Khánh dùng để chỉnh bộ sinh nhiễu cho khớp lỗi OCR thật
- Ghi vào báo cáo: val dùng để chọn checkpoint và giảm learning rate nên CER val hơi lạc quan (checkpoint tốt nhất lại là epoch cuối nên ảnh hưởng có thể nhỏ); số báo cáo cuối là CER trên test. Val 9 đoạn và test 5 đoạn nên CER có độ bất định lớn, báo kèm khoảng tin cậy bootstrap; trang test 253 còn 7,1% dòng trùng nhiều với trang train 153
- Đã kiểm theo nội dung hai file Hiển gửi: cả `eval_val_DRAFT_FAKE.json` và `eval_val_FAKE.json` đều có trường `gate`, và `eval_val_FAKE.json` có thêm `exploratory_syllable_level`. Còn lại: ghi lại giờ đặt ngưỡng từ lịch sử phiên bản của notebook Colab làm bằng chứng ngưỡng được chốt trước khi nhìn số
- Chốt tập test thật khoảng 90 dòng (hạn 15/11 đã qua): nay quan trọng hơn vì cổng FAIL, cần kiểm tra kết luận âm tính có đứng vững trên chữ viết thật không; cần văn bản ngoài UIT-HWDB, loại khỏi kho của Khánh, có đồng ý và ẩn danh; tập này cũng đáp ứng yêu cầu thu thập và gán nhãn của Phần D
- Thầy đã yêu cầu nhóm đúng theo tài liệu danh mục nên Phần B (chứng minh) và Phần C (bài tập) là bắt buộc; chuẩn bị phụ lục khai báo việc dùng công cụ AI tạo sinh (gồm mã `eval_00_start.ipynb`, `print_confusions.py` và các bản nhật ký do trợ lý AI soạn); đọc đủ câu cuối của tài liệu danh mục vì bản sao bị cắt ở "Không dùng bản AI miễn phí..."
- Lưu bằng chứng việc thầy cho phép dùng PyTorch và thư trả lời của thầy (ảnh chụp tin nhắn hoặc email) vào Drive hoặc `docs/` để dán vào phụ lục báo cáo. Theo Hiển, thầy đã chấp nhận phạm vi dùng PyTorch chỉ ở mức chung, và thư trả lời (19/11/2026) chỉ yêu cầu nhóm đúng theo tài liệu danh mục, không nêu riêng `nn.Conv2d`, `nn.LSTM`, `AdamW` hay `nn.CTCLoss`. Dương đang huấn luyện bằng các thành phần này (hàm mất mát không tự cài, chỉ phần giải mã tự cài) trong khi đề ghi Phần A không dùng hàm huấn luyện dựng sẵn và phải đối chiếu với thư viện chuẩn, nên khi vấn đáp chỉ nói được thầy đồng ý dùng PyTorch và phải chỉ ra phần tự cài; Dương được giao tự cài CTC loss và kiểm với `nn.CTCLoss`
- Chưa rõ vì sao 44 ảnh (ví dụ ảnh 10 đến 14 của thư mục 10) không có nhãn: cần xem `label.json` của thư mục 10; xem 10 đến 20 dòng đầu của `data/corpus/viwiki_sentences.txt` để xác nhận bản `prepare_corpus.py` đã sửa tách câu là bản đang dùng trên `main`
- Gộp logic `make_passage_splits.py` vào `make_splits.py` sau hạn nộp, sửa `test_pipeline_tools.py` tương ứng; làm cho `00_prepare_data.ipynb` dừng khi một lệnh lỗi (Bug 12)
- Việc của Hiển theo đề (đề xuất 19/11/2026, chờ xác nhận): Phần A: đối chiếu `align.py` (Levenshtein) với một thư viện chuẩn (ví dụ `editdistance` hoặc `jiwer`) trên chuỗi ngẫu nhiên, kiểm tay `classify_errors` và bootstrap; Phần B và C: tính đúng của quy hoạch động Levenshtein, tính chất của CER và WER, khoảng tin cậy bootstrap, kèm bài tập số học (có thể mượn các mệnh đề về ngưỡng Bayes theo chi phí và cận Hoeffding của Đề 2 và Đề 3); Phần D: tập test thật khoảng 90 dòng với văn bản đồng ý và ẩn danh, kiểm điều khoản sử dụng của UIT-HWDB (`report_facts.json` ghi chưa tìm thấy giấy phép, kiểm ngày 17/11/2026), datasheet, model card, báo cáo LaTeX 25 đến 40 trang, kho Git có mô tả môi trường và tập lệnh tái lập
- Hiển phải đọc và giải thích được `align.py`, `classify_errors.py`, `check_text_overlap.py`, `make_passage_splits.py` và `eval_00_start.ipynb` trước vấn đáp

## Giai đoạn 7 — Trần chính xác, Phần B và C của vai đánh giá, rà soát báo cáo
**Mục tiêu giai đoạn:** Sửa sai sót về trần lợi ích của sửa dấu, hoàn thiện Phần B và C của vai đánh giá, rà soát bản nháp báo cáo tiến độ cho nhất quán với số đo và với yêu cầu của đề, và theo dõi việc giao cho Dương và Khánh.

**Đã làm:**

- Notebook `eval_00_start.ipynb` thêm hai mục sau mục 5b. Mục 5c chạy `scripts/print_confusions.py` trên val (chỉ nhận `val` hoặc `train`, có `assert` chặn `test` để bảng nhầm lẫn không dùng chỉnh gì theo test) và ghi `outputs/confusions_<split>_DRAFT.json`. Mục 5d tính trần chính xác của mọi bộ sửa dấu: CER và WER sau khi bỏ dấu cả hai bên, mức giảm tối đa, tỉ lệ lỗi dấu chính xác, đặt cạnh số đếm theo nhãn của cổng và số của mục 5b, kèm kiểm lại cổng bằng số chính xác như một phân tích độ nhạy (không đổi phán quyết đã chốt); kết quả ghi vào `eval_val*.json` (trường `exact_ceiling`). Cả hai mục bị bỏ qua khi `P_MIN` hoặc `CEILING_MIN` còn `None`. Đã kiểm thử trong môi trường giả lập; chưa có kết quả trên Colab thật, Hiển sẽ gửi số liệu sau.
- Script `scripts/print_confusions.py`: các cặp ký tự bị nhầm nhiều nhất theo từng loại lỗi của `classify_errors.py`, ký tự bị bỏ sót và thừa, ký tự kém tin cậy theo tỉ lệ sai; tổng lỗi theo loại khớp với `classify_pair` (đã kiểm). Chưa chạy trên OCR thật.
- Hai file thân bài `phan_B.tex` (B1 quy hoạch động Levenshtein, B2 tính chất độ đo và cận của CER, B3 CER gộp, B4 trần chính xác) và `phan_C.tex` (C1 đến C4, mã kiểm chứng in ngay trong file) cho Phần B và C của vai đánh giá. Biên dịch thử bằng xelatex (12 trang, không lỗi, chưa thử pdflatex với T5). Các đoạn mã chạy lại cho kết quả khớp số ghi trong bài: C1, 2.000 cặp, 0 cặp lệch so với `editdistance`; C2, 5.000 bộ ba, 0 vi phạm; C4(a), 3.000 cặp, 0 vi phạm định lý, số đếm theo nhãn bằng trần chính xác ở 2.359 cặp và nhỏ hơn ở 641 cặp (21,4%). Các chỗ `[điền]` trong Phần C chờ số chạy trên dữ liệu thật.
- Rà soát bản nháp báo cáo tiến độ (LaTeX, 24 trang, đề ngày 20/11/2026, từ Tóm tắt đến Phụ lục D, đã có Phần B gồm 8 mệnh đề và Phần C gồm C1 đến C7 soạn trước): các điểm sai hoặc không nhất quán nêu ở mục tồn đọng.
- Tập viết tay thật: theo Hiển (21/11/2026), đang thu và đã được 45 [chưa rõ: 45 dòng, ảnh hay mẫu; số người viết; số đoạn văn; đã có phiếu đồng ý chưa].
- Theo Hiển (21/11/2026), các mục `[CẦN XÁC NHẬN]` trong báo cáo và các dòng "chưa rõ" trong nhật ký được giữ nguyên cho đến khi có số liệu và xác nhận của Dương và Khánh.

**Quyết định kỹ thuật:**

- **Vấn đề:** báo trần lợi ích của sửa dấu thế nào trong báo cáo và nhật ký, sau khi phát hiện số đếm theo nhãn của cổng không phải cận trên
- **Đã chọn:** dùng trần chính xác (Định lý B4) làm số chính, báo kèm số đếm theo nhãn của cổng như cận dưới; phán quyết cổng đã chốt trước (FAIL) không đổi; thay Mệnh đề 8 trong báo cáo bằng B4
- **Vì sao:** chứng minh của Mệnh đề 8 coi lỗi chữ nền và khoảng trắng là "vẫn còn nguyên" sau sửa dấu, nhưng khoảng cách Levenshtein nhỏ nhất có thể giảm nhờ một căn chỉnh khác (ví dụ ở Bug 15). B4 chứng minh trần chính xác bằng CER của văn bản đã bỏ dấu hai bên, tính được trực tiếp và không phụ thuộc cách phá hòa. Đánh đổi: số chính xác trên OCR cuối chưa có (chờ mục 5d); việc phán quyết FAIL còn đứng vững hay không cần xác nhận lại bằng số chính xác, vì điều kiện p cần đạt 0,5 và trần cần đạt 3 điểm đều đo bằng số đếm theo nhãn

- **Vấn đề:** hình thức giao Phần B và C của vai đánh giá
- **Đã chọn:** B1 đến B4 và C1 đến C4, giao dưới dạng hai file thân bài `.tex` để Hiển `\input` vào file LaTeX chính do Hiển tự tạo; mã kiểm chứng nằm trong `phan_C.tex`, không tạo script riêng
- **Vì sao:** Hiển đã có khung LaTeX và đã có bản Phần B, C soạn trước, nên cần phần ghép được chứ không phải một bản độc lập; bảng ghép: B1 và B2 thay chứng minh Mệnh đề 2 (nhất là bất đẳng thức tam giác), B3 thêm mới, B4 thay Mệnh đề 8, C1 gộp với C1 cũ, C2 đến C4 thêm làm C8 đến C10. Đánh đổi: muốn chạy tự động thì phải chép mã ra file

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 15 — số đếm theo nhãn của cổng bị gọi nhầm là cận trên của trần lợi ích:** nhật ký, tin giao việc cho Khánh ("tối đa giảm khoảng 0,85 đến 1,38 điểm CER") và bản nháp báo cáo (Tóm tắt, mục 7.2, Bảng 6, Mệnh đề 8) đều coi 1,38 điểm (và "CER xuống 10,89%") là mức giảm tối đa của sửa dấu hoàn hảo → Nguyên nhân thật sự: cổng đếm lỗi dấu trên một căn chỉnh tối ưu cụ thể (quy tắc phá hòa của `align.py`: đường chéo, rồi `ins`, rồi `del`), mà một cặp chuỗi có thể có nhiều căn chỉnh tối ưu ngang nhau; số đếm đó chỉ là cận dưới của trần chính xác E = d(h, r) − d(σh, σr) (σ là phép bỏ dấu), chứng minh ở B4. Ví dụ: nhãn `a`, OCR `áo` có d = 2 và căn chỉnh của `align.py` không có lỗi dấu nào, nhưng đổi `áo` thành `ao` giảm d xuống 1. Kiểm bằng vét cạn trên bảng chữ cái nhỏ: 3.000 cặp ngẫu nhiên, 0 vi phạm định lý, số đếm theo nhãn không bao giờ lớn hơn trần chính xác và thấp hơn ở 21,4% số cặp; trên file OCR nháp cũ của lần thử (không phải OCR cuối) trần chính xác là 4,28 điểm so với 2,49 điểm theo nhãn → Đã sửa lời ở nhật ký (các câu 1,38 và 0,85 nay ghi là cận dưới), thêm mục 5d vào notebook, soạn B4; còn phải sửa báo cáo, gửi đính chính cho Khánh và Dương, và điền trần chính xác thật khi có số 5d

**Việc còn tồn đọng từ giai đoạn này (các mục tồn đọng của Giai đoạn 6 vẫn còn hiệu lực, trừ những mục đã thay đổi dưới đây; các dòng cần xác nhận được giữ nguyên theo yêu cầu của Hiển):**

- Hiển chạy lại notebook (mục 5c và 5d) trên Colab và gửi số liệu. Có số thì cập nhật nhật ký, soạn tin đính chính ngắn cho Khánh và Dương về Bug 15, và sửa báo cáo (Tóm tắt, mục 1.5, mục 7.2, Bảng 6, mục 8.7)
- Sửa báo cáo theo rà soát: thay Mệnh đề 8 bằng B4; Tóm tắt ghi "sáu mệnh đề" trong khi mục 8 có tám mệnh đề; câu ở mục 4.4 nói `to_style` được áp dụng khi tính CER (notebook đo không gọi hàm này, cần kiểm bằng `grep` và sửa câu cho đúng cách đo); mục 8.4 ghi T cỡ vài trăm khung trong khi ước từ ảnh (rộng 2048, cao trung bình 140, co về cao 32, `T = w // 4 + 1`) chỉ khoảng 118, cần in số khung thật; Bảng 5 và 6 đã cũ (quét τ và bootstrap đã có trong notebook với `correct()` giả; nhánh `b/corrector` có `calibration.py` và test tương ứng; giải mã tham lam và beam là NumPy viết tay trong notebook của Dương, chờ Dương xác nhận); dấu gạch chéo ngược hiển thị thừa trong PDF (như `\{train_data,test_data\}` và `verify\_partBC.py`); tài liệu tham khảo [5], [6], [7] còn dấu †, kiểm arXiv của [8] và nơi công bố của [1]; câu "chủ yếu do cụm từ thông dụng" ở mục 4.5 chưa kiểm; bổ sung vào mục 7.2 việc Hiển từng thấy vài thống kê ký tự của OCR nháp trước khi chọn ngưỡng; Phụ lục A thêm `phan_B.tex`, `phan_C.tex` và phân tích trần chính xác
- Ghép `phan_B.tex` và `phan_C.tex` vào báo cáo theo bảng ghép ở trên và điền các chỗ `[điền]` sau khi chạy mã trên dữ liệu thật
- Tập viết tay thật: làm rõ đơn vị của "45", số người viết, số đoạn văn và phiếu đồng ý; kiểm 100% số dòng giữa ảnh và nhãn thay vì 20% (tập chỉ 90 đến 150 dòng, nhãn lấy từ văn bản chép nên người viết chép sai sẽ gây nhãn sai)
- Theo dõi phản hồi của Dương và Khánh theo các hạn đã đề xuất (đợt 1: 21/11; đợt 2: 23/11; đợt 3: 25/11); hạn Dương báo kế hoạch 19/11 đã qua
- `print_confusions.py` đã viết (mục 5c) nhưng chưa chạy; chạy xong gửi bảng chữ cái nền cho Dương và ba bảng dấu cho Khánh (thay cho mục tồn đọng cũ về script in các cặp ký tự nhầm)

## Giai đoạn 8 — Tập test viết tay thật: dựng chỉ mục, chạy OCR và chẩn đoán đầu vào (ghi 22/11/2026)
**Mục tiêu giai đoạn:** Kiểm tra trên chữ viết tay thật, ngoài miền UIT-HWDB, xem OCR đã chốt (Giai đoạn 6) tổng quát hóa được đến đâu và kết luận "lỗi dấu chỉ chiếm 11,3% lỗi OCR" còn đúng không (RQ4); đồng thời đáp ứng khâu "thu thập và gán nhãn" bắt buộc của Phần D. Báo riêng, không thay thế test UIT.

**Đã làm:**

- Dữ liệu: 90 ảnh dòng, 9 đoạn × 10 dòng (P01 đến P04 trích "Lão Hạc", P05 đến P09 trích "Chí Phèo" của Nam Cao; theo Hiển, văn bản lấy từ bản Wikisource tiếng Việt, tác giả mất năm 1951 nên hết thời hạn bảo hộ quyền tác giả theo hiểu biết của nhóm; còn phải ghi URL từng truyện và ngày truy cập), do Hiển và Dương chép tay: Dương ảnh 001 đến 045, Hiển ảnh 046 đến 090, nên P05 do cả hai viết (041 đến 045 Dương, 046 đến 050 Hiển). Chụp bằng điện thoại, giấy sinh viên (ô li hoặc kẻ ngang), bút bi; ảnh và file chỉ mục nằm ở Drive `data/handwriting-test/`. Người viết không viết dấu câu. Tập hiện đủ 90 dòng, 45 dòng mỗi người.
- Lệch khỏi kế hoạch ở mục 4.3 của báo cáo: kế hoạch là ít nhất 10 đoạn, mỗi đoạn ít nhất 3 người viết; thực tế là 9 đoạn, 2 người viết, mỗi đoạn một người (trừ P05). Hệ quả: không tách được ảnh hưởng của đoạn văn khỏi ảnh hưởng của người viết; bootstrap theo người viết chỉ có 2 cụm nên vô nghĩa, dùng bootstrap theo đoạn (9 cụm) và báo CER riêng từng người viết như số mô tả.
- Nhãn: `labels.json` (các trường `id`, `image`, `text`, `passage`, `writer`) khớp 90/90 với `lines_to_write.txt` (kiểm bằng script); đúng NFC, không dấu câu, không khoảng trắng đôi, không dòng trùng; kiểu đặt dấu thanh khớp quy ước "hòa" (ví dụ "thỏa" ở `real_070`); 4.218 ký tự nhãn, 1.033 từ (Dương 2.085 ký tự, Hiển 2.133). Nhãn là văn bản được giao chứ không phải chép lại từ ảnh (Bug 17).
- Kiểm tra trùng văn bản, theo ghi chép của Hiển lúc 21/11/2026 23:30 (chưa kiểm lại trong phiên này): so với toàn bộ 7.229 dòng UIT (kể cả 43 trang bị loại): trùng nguyên dòng 0/90, có ít nhất một chuỗi 5 từ 0/90, từ 50% cửa sổ trùng 0/90; so với kho Wikipedia 300.000 câu: 0/90, 1/90, 0/90. Dưới mốc 10% đã chốt ở Giai đoạn 5: giữ nguyên.
- Quy tắc đã ghi trước lần chạy OCR (21/11/2026 23:30): cổng giả thuyết giữ nguyên (tỉ lệ lỗi dấu từ 0,5 và trần lợi ích từ 3 điểm CER); OCR là checkpoint cuối, giải mã beam, tiền xử lý y hệt lúc huấn luyện, chạy một lần, không chỉnh theo kết quả; không dùng tập này để chọn lại OCR, ngưỡng hay τ (τ lấy từ val); bootstrap theo đoạn (9 đơn vị) kèm khoảng tin cậy. Hai chỗ trong ghi chú đó đã đổi: "3 người viết ẩn danh" thành 2 người (Bug 18) và quy tắc "chạy một lần" không còn giữ được (xem Quyết định kỹ thuật và Bug 16).
- `scripts/build_real_index.py` (mới, có `--self-test`): đọc `labels.json`, bỏ trường `writer` (W1 đến W3), gán `writer_id` theo số dòng (`duong` từ 1 đến 45, `hien` từ 46 đến 90, đổi được bằng `--duong-last`), `group_id` là đoạn, `split` là `test`; chỉ ghi đúng sáu trường của `dataset_index.schema.json`; kiểm ảnh tồn tại, ký tự lạ, id trùng; có hàm `strip_punctuation()` (đổi dấu câu và ký hiệu thành khoảng trắng). Đã kiểm đầu ra với schema thật. Chạy thật trên Colab: 90 dòng, 2 người viết, 9 đoạn, `validate_dataset_index.py` báo OK; ghi `real_dataset_index.jsonl` vào Drive.
- OCR trên tập thật: một ô Colab độc lập, không phụ thuộc trạng thái notebook của Dương: dựng lại bộ ký tự từ tập train của `splits.json` (có `assert` 5.369 dòng), cùng kiến trúc CRNN, tiền xử lý (ảnh xám, cao 32 pixel, chuẩn hóa về [-1, 1]), batch 32 đệm nền trắng, nạp `best_crnn.pth`; giải mã greedy và beam (rộng 10) viết tay (hai hàm giải mã trùng bản trong notebook của Dương trên 60 chuỗi ngẫu nhiên; chưa chạy cả ô với PyTorch trong môi trường giả lập). Ghi `real_ocr_out_greedy.jsonl` và `real_ocr_out_beam.jsonl`, không có `ocr_conf` (trường tùy chọn). Ký tự có trong nhãn nhưng không có trong bộ ký tự huấn luyện: "À" (2 lần ở `real_019`, 0,05% ký tự nhãn); giữ nguyên nhãn.
- Lần chạy 0 (đầu vào chưa sửa): nhiều dòng nhãn dài 35 đến 60 ký tự chỉ ra 2 đến 3 ký tự ("kh", "lật", "th"). Chạy ô chẩn đoán đầu vào (đọc ảnh, không chạy mô hình): 20/90 ảnh có EXIF orientation 8 (`real_002, 004, 005, 010, 012, 015, 016, 018, 020, 023, 026, 028, 029, 030, 033, 034, 037, 038, 040, 044`), đều nằm trong 002 đến 044, tức phần Dương chụp; cả 20 ảnh cao hơn rộng. Sau khi xoay lại, 26/90 dòng có số khung `w // 4 + 1` nhỏ hơn số ký tự nhãn (CTC không thể xuất đủ); trung vị 1,22 khung cho mỗi ký tự (từ 0,56 đến 1,83); tỉ lệ rộng/cao trung vị 6,8 (từ 4,2 đến 7,8). Độ sáng trung bình 0,75 (Dương) và 0,53 (Hiển), độ tương phản p95 trừ p5 là 0,41 và 0,30; ảnh của Hiển tối dần trong cùng một đoạn (046 đến 058 giảm từ 0,62 xuống 0,36).
- Lần chạy 1 (chỉ thêm `ImageOps.exif_transpose` vào bước đọc ảnh): beam CER 66,8% (thô) và 66,2% (bỏ dấu câu), WER 93,7% và 93,2%; greedy CER 67,0% và 66,4%, WER 93,5% và 93,1%. Theo người chụp (beam, bỏ dấu câu): Dương 48,7% (45 dòng), Hiển 83,4% (45 dòng). Theo đoạn: P01 45,4%, P02 53,2%, P03 45,9%, P04 48,6%, P05 64,7%, P06 89,9%, P07 89,1%, P08 80,4%, P09 77,1%. 20 ảnh vừa sửa xoay có CER 48,5%, bằng các dòng Dương còn lại (48,8%). 43/90 dòng có dấu câu trong đầu ra OCR; 24 dòng có đầu ra ngắn hơn nửa độ dài nhãn. Các số này tính tạm bằng một hàm Levenshtein viết riêng (không phải `align.py`), chưa phải số chính thức; chưa chạy phân loại lỗi, cổng giả thuyết và bootstrap trên tập này.
- Xem ba ảnh (`real_001`, `real_002`, `real_057`): chữ chỉ chiếm khoảng 35 đến 45% chiều cao ảnh; nền là giấy ô li hoặc kẻ ngang với đường kẻ chạy suốt; `real_057` chụp tối, mực xám, có nét đuôi của dòng trên lọt vào mép trên; `real_001` có hai dòng chữ trong ảnh (Bug 17). Đã soạn ô tạo bảng ảnh đối chiếu (mỗi đoạn một ảnh dài: tên ảnh và nhãn ở trên, ảnh ở dưới, vạch đỏ giữa các dòng, lưu vào `contact_sheets/`); đã chạy trên đủ 9 đoạn sau khi sửa lỗi font (Bug 19).
- Bảng đối chiếu `label_review.csv` (Hiển điền 22/11/2026 sau khi xem `contact_sheets/`, cột `note` để trống): 45 dòng `khop`, 31 `xuong_dong`, 12 `khac_hoa_thuong`, 2 `khac_chu` (`real_047`, `real_057`). Theo người chụp: Dương 18 `khop`, 17 `xuong_dong`, 10 `khac_hoa_thuong`, 0 `khac_chu`; Hiển 27, 14, 2, 2. Tập con sạch (`khop` và `khac_hoa_thuong`) có 57 dòng, 2.310 ký tự (Dương 28 dòng, Hiển 29 dòng); loại 33 dòng theo trạng thái xem từ ảnh và nhãn, không theo CER.
- Chấm chính thức trên tập con sạch bằng đầu ra lần 1 (số chính không phân biệt hoa/thường và bỏ dấu câu; ô đo kiểm từng dòng, `editdistance` và `edit_distance` của `src/eval/align.py` trùng nhau ở cả 57 dòng với cả hai cách chuẩn hóa; kết quả trùng đúng số đã tính tạm trước đó bằng hàm viết riêng; lưu `real_eval_clean.json` trong `handwriting-test/`): beam CER 57,3%, WER 87,2%; greedy CER 58,3%, WER 87,2%; beam có phân biệt hoa/thường CER 60,7%, WER 88,9%. Toàn 90 dòng (cùng cách tính) 63,0% và 92,2%; 31 dòng `xuong_dong` 69,3% và 98,4%. Theo người chụp trên tập sạch: Dương 34,0% (WER 73,3%), Hiển 78,5% (WER 100%); theo đoạn (số dòng sạch): P01 30% (6), P02 34% (4), P03 34% (7), P04 39% (8), P05 55% (6), P06 86% (7), P07 85% (6), P08 75% (7), P09 67% (6). Bootstrap theo đoạn (9 cụm, 5.000 lần, hạt giống 42): khoảng tin cậy 95% của CER gộp từ 42,9% đến 72,0%. Không có dòng nào khớp hoàn toàn với nhãn; 23/57 dòng vẫn có dấu câu trong đầu ra; 15/57 dòng có đầu ra ngắn hơn nửa nhãn. Trạng thái và người chụp lẫn nhau (phần lớn dòng `khac_hoa_thuong` là của Dương), nên không so CER giữa các trạng thái. Chênh giữa Dương và Hiển lẫn với điều kiện chụp (độ sáng 0,75 và 0,53), không diễn giải như khác biệt chữ viết.

**Quyết định kỹ thuật:**

- **Vấn đề:** hợp đồng dữ liệu (đã đóng băng) chỉ cho `split` là `train`, `val` hoặc `test` và cấm trường lạ, trong khi tập thật không thuộc phần nào của UIT-HWDB
- **Đã chọn:** không sửa hợp đồng; dùng file riêng `real_dataset_index.jsonl` và `real_ocr_out_*.jsonl` với `split = "test"`, id bắt đầu bằng `real_`, và không bao giờ ghép vào `splits.json` hay `ocr_out.jsonl`
- **Vì sao:** sửa hợp đồng sát hạn ảnh hưởng validator và mã của Dương và Khánh. Đánh đổi: bộ đo phải đọc file riêng, và "test" ở đây không phải test UIT nên mọi bảng phải ghi rõ

- **Vấn đề:** tính CER thế nào khi nhãn không có dấu câu mà OCR đã học in dấu câu (43/90 dòng có dấu câu trong đầu ra)
- **Đã chọn:** số chính là CER và WER sau khi bỏ dấu câu khỏi đầu ra OCR bằng `strip_punctuation`, báo kèm số thô; tính lại val UIT theo cùng cách để so sánh công bằng. Hiển chốt 22/11/2026: số chính cũng không phân biệt hoa/thường (đổi cả nhãn và đầu ra về chữ thường), báo kèm bản phân biệt hoa/thường làm số phụ
- **Vì sao:** dấu câu OCR chèn thêm bị tính thành lỗi chèn giả; chữ hoa/thường khác nhãn xuất phát từ cách người viết viết, không phải từ mô hình (Bug 17). Đánh đổi: bỏ bớt thông tin về việc mô hình có học đúng chữ hoa hay không

- **Vấn đề:** sửa gì sau lần chạy 0 và 1, khi quy tắc ghi trước là chạy một lần
- **Đã chọn:** chỉ sửa lỗi định dạng đầu vào hoặc lỗi nhãn và ảnh nhìn thấy được trực tiếp trên ảnh và nhãn, không nhìn CER; không chỉnh độ sáng, tương phản hay cách cắt dựa theo CER; nếu muốn thêm bản làm sạch nền thì chốt phương pháp và tham số trước khi chạy; báo cáo ghi mọi lần chạy (0 và 1; lần 1 là bản cuối, không chạy lại OCR) cùng lý do
- **Vì sao:** lần chạy 0 sai vì lỗi xoay (Bug 16) và nhãn không khớp ảnh (Bug 17) thì kết quả không đo được gì về mô hình. Đánh đổi: quy tắc "chạy một lần" đã không giữ được và phải nói thẳng trong báo cáo; số lần nhìn kết quả tăng nên phải nghiêm ngặt ở chỗ không chỉnh theo CER

- **Vấn đề:** có nhờ Dương sửa phần OCR để thêm tiền xử lý ảnh không
- **Đã chọn:** không (Hiển chốt 22/11/2026); OCR và tiền xử lý lúc huấn luyện giữ nguyên, việc chuẩn hóa hướng ảnh nằm ở bước đọc ảnh của ô đo
- **Vì sao:** OCR đã chốt (Giai đoạn 6); đổi tiền xử lý hoặc thêm tăng cường dữ liệu nghĩa là huấn luyện lại và chạy lại 5 hạt giống trong khi hạn là 30/11, và Dương còn việc ưu tiên cao hơn (hàm mất mát CTC tự cài, 5 hạt giống). Đánh đổi: CER trên tập thật sẽ cao và báo như kết quả về dịch chuyển miền, không phải lỗi cần sửa bằng cách tinh chỉnh

- **Vấn đề:** phạm vi việc còn lại của tập 90 dòng, khi CER lần chạy 1 đã khoảng 66% và đường chính (`correct()` của Khánh, quét τ, hàm mất mát CTC tự cài của Dương, sửa báo cáo) chưa xong
- **Đã chọn:** thu hẹp (Hiển chốt 22/11/2026): lập bảng 90 dòng với bốn trạng thái (`khop`, `xuong_dong`, `khac_chu`, `khac_hoa_thuong`) bằng cách xem ảnh; không cắt lại ảnh, không sửa nhãn, không chạy lại OCR; đầu ra lần 1 (đã sửa xoay) là bản cuối; số chính chấm trên tập con sạch (`khop` và `khac_hoa_thuong`), báo riêng số dòng `xuong_dong` và `khac_chu`; bỏ thử các bản làm sạch nền, bỏ phân tích p và trần trên tập thật khi CER vẫn rất cao, bỏ so sánh tỉ lệ rộng/cao với UIT
- **Vì sao:** tập 90 dòng chỉ phục vụ RQ4 và khâu thu thập, gán nhãn của Phần D; kết quả cuối gần như chắc chắn là kết luận về dịch chuyển miền chứ không về lớp sửa dấu, nên thời gian còn lại dành cho đường chính. Đánh đổi: tập chấm nhỏ hơn 90 dòng (số dòng loại ra do ảnh xuống dòng hoặc khác chữ chưa biết); loại dòng theo trạng thái nhìn từ ảnh và nhãn, không theo CER

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 16 — 20 ảnh điện thoại bị lưu xoay nên OCR chỉ ra 2 đến 3 ký tự:** lần chạy 0 cho "kh", "lật", "th" trên các dòng nhãn dài 35 đến 60 ký tự; ô chẩn đoán cho 20 ảnh có EXIF orientation 8, cao hơn rộng, rộng 16 pixel sau khi co về cao 32 và chỉ 5 khung → Nguyên nhân thật sự: ảnh điện thoại chụp dọc rồi xoay bằng thẻ EXIF, `PIL.Image.open` không áp dụng thẻ này nên `preprocess_line_image` nhận một dải dọc; ảnh UIT-HWDB không có thẻ này nên lỗi không hiện ở các phép đo trước → Cách sửa: thêm `ImageOps.exif_transpose` vào bước đọc ảnh và chạy lại (lần 1); 20 ảnh này nay có CER 48,5%, bằng các dòng Dương còn lại (48,8%)
- **Bug 17 — nhãn của tập thật không phải chữ có trong ảnh:** `real_001` có hai dòng chữ trong ảnh ("quay ra" xuống dòng thứ hai) trong khi nhãn là một dòng; `real_001` viết "Có" nhưng nhãn là "có", `real_002` viết "sau" nhưng nhãn là "Sau" → Nguyên nhân thật sự: nhãn sinh từ `lines_to_write.txt` (văn bản giao cho người viết) chứ không phiên âm lại từ ảnh; người viết xuống dòng khi hết chỗ và tự chọn chữ hoa thường. Mới thấy trên 3 ảnh (suy ra chưa kiểm cả 90), chưa biết bao nhiêu dòng bị ảnh hưởng; mô hình đọc một dòng nên không thể đọc đúng ảnh hai dòng, phần lỗi đó không thuộc về mô hình → Cách sửa: chưa xong; lập bảng 90 dòng với bốn trạng thái (khớp, xuống dòng, khác chữ, chỉ khác hoa/thường) bằng cách xem ảnh; không cắt lại ảnh và không sửa nhãn, mà chấm trên tập con sạch và báo riêng số dòng bị loại
- **Bug 18 — trường `writer` W1 đến W3 sai, thực tế chỉ có 2 người viết:** `labels.json` và ghi chú trước ghi 3 người viết ẩn danh xoay vòng theo đoạn (W1, W2, W3, W1, ...); Hiển xác nhận chỉ có Hiển và Dương → Nguyên nhân thật sự: script sinh nhãn gán W1 đến W3 theo đoạn mà không phản ánh người thật; hậu quả là P05 bị chia đôi giữa hai người → Cách sửa: `build_real_index.py` bỏ trường này và gán `writer_id` theo số dòng; báo cáo sửa từ 3 thành 2 người viết (mục 4.3, thẻ dữ liệu, phiếu đồng ý)
- **Bug 19 — ô tạo bảng ảnh đối chiếu báo `OSError: cannot open resource`:** chạy ô trên Colab dừng ở dòng nạp font → Nguyên nhân thật sự: ô nạp font bằng đường dẫn cố định `/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf` (đúng trong môi trường giả lập của mình), mà Colab bản Python 3.13 không có file ở đó → Cách sửa: dùng font DejaVu đi kèm `matplotlib` (`matplotlib.get_data_path()` rồi `fonts/ttf/DejaVuSans.ttf`); đã thử vẽ được chữ tiếng Việt, ô chạy xong cho 9 file `P01.png` đến `P09.png`

**Việc còn tồn đọng từ giai đoạn này (các mục tồn đọng của Giai đoạn 6 và 7 vẫn còn hiệu lực, trừ mục "làm rõ đơn vị của 45, số người viết, số đoạn văn" đã được trả lời ở trên):**

- Chạy ô đo trùng văn bản (cùng định nghĩa cửa sổ 5 từ) của 90 nhãn với toàn bộ dòng UIT, kho 300.000 câu và `train.txt` của Khánh, rồi ghi số chính thức vào mục 4.3 và thẻ dữ liệu thay cho số trong ghi chú 21/11 (đã chạy ngày 24/11/2026 bằng `check_real_text_overlap.py`, xem Giai đoạn 12; chưa xác nhận đã ghi vào báo cáo)
- Ghi số chính thức từ `real_eval_clean.json` vào báo cáo (mục 7 thí nghiệm E6, thẻ dữ liệu nguồn 3, mục 4.3, mục 10.3, mục 1.5); đối chứng thư viện của Phần A (Bảng 5, hàng Levenshtein) đã làm ở mức ký tự trên tập thật (57 dòng, trùng từng dòng), mức từ mới chỉ tính bằng `editdistance`; có thể lặp lại cùng phép kiểm trên 636 dòng val UIT nếu còn thời gian. Không cắt lại ảnh, không sửa nhãn, không chạy lại OCR
- Số chính đã chốt (không phân biệt hoa/thường và bỏ dấu câu); báo kèm số thô và số phân biệt hoa/thường; áp dụng cùng cách cho val UIT khi so sánh
- Không chạy phân loại lỗi, p và trần chính xác trên tập thật (CER còn rất cao, đã quyết định ở Quyết định kỹ thuật); báo cáo ghi kết quả tập thật như kết quả về dịch chuyển miền
- Tính lại CER val UIT cùng cách (không phân biệt hoa/thường, bỏ dấu câu) để so sánh công bằng với tập thật. So sánh tỉ lệ rộng/cao và số khung trên mỗi ký tự với UIT đã bỏ khỏi phạm vi (xem Quyết định kỹ thuật)
- Báo cáo: mục 4.3, thẻ dữ liệu (nguồn 3), mục 10.3 và mục 1.5 đổi từ "kế hoạch" sang số thật, ghi lệch kế hoạch (9 đoạn × 2 người viết, P05 do cả hai), ghi cả lần chạy 0, 1 và cuối cùng lý do, "À" không có trong bộ ký tự huấn luyện, và hạn chế: chỉ hai tác phẩm, giấy ô li, ánh sáng khác nhau giữa hai người nên không so sánh hai người như khác biệt chữ viết
- Phiếu đồng ý và siêu dữ liệu người viết: Hiển chốt 22/11/2026 là không cần phiếu riêng, vì hai người viết là hai thành viên nhóm và là chủ sở hữu dữ liệu; báo cáo (mục 4.3, phụ lục B) sửa cho khớp: nêu rõ lý do, giữ phụ lục B làm mẫu cho trường hợp thu thêm người ngoài nhóm hoặc bỏ phụ lục; siêu dữ liệu chỉ ghi thiết bị (điện thoại), giấy và bút
- Giữ `real_ocr_out_*_run0` và `_run1` làm bản ghi (đầu ra lần 1 là bản dùng để chấm); lưu `contact_sheets/` và `label_review.csv` cùng thư mục `handwriting-test/`

## Giai đoạn 9 — Kết quả vai OCR của Dương: độ tin cậy, đối chiếu giải mã, 5 hạt giống, CTC loss tự cài (ghi 22/11/2026, theo tin nhắn và `role_a_experiments_summary.md` Dương gửi)
**Mục tiêu giai đoạn:** Ghi lại phần việc Dương báo đã xong để đưa vào báo cáo (Bảng 5, Bảng 6, mục 5.1, 6.2.1, 7.4, 10.3) và vấn đáp; đóng các việc tồn đọng của vai OCR ở Giai đoạn 6 và 7. Hiển chưa tự chạy lại các kết quả này.

**Đã làm (theo báo cáo của Dương):**

- `ocr_conf.jsonl`: 837 dòng (636 val và 201 test theo `splits.json` hiện hành), xuất từ `best_crnn.pth` (epoch 25, `model.eval()`, ảnh cao 32 pixel, đệm nền trắng +1,0, chiều dài `(w // 4) + 1`), giải mã beam (beam 10, top_k 15); kèm `ocr_conf_readme.md` trong Drive `outputs/`. Quy ước: `char` là danh sách độ tin cậy cho từng ký tự của `ocr_text` (tính cả khoảng trắng nên `len(char) == len(ocr_text)`), giá trị là `exp(log_prob)` lớn nhất tại khung hình của ký tự được chọn; `syllable_min` và `syllable_mean` tính trên từng âm tiết tách bằng khoảng trắng. `ocr_text` khớp 100% (837/837) với `ocr_out_beam.jsonl`, `validate_ocr_out.py` pass. Khánh dùng file này để quét τ và hiệu chỉnh.
- Đối chiếu bộ giải mã trên val (636 dòng): greedy tự cài (NumPy) khớp 100% với PyTorch, CER 12,37%; beam tự cài (beam 10, top_k 15) CER 12,27%, `pyctcdecode 0.5.0` CER 12,25%, khớp tuyệt đối 96,07% (611/636 dòng); 25 dòng lệch vì bản tự cài chỉ mở rộng 15 ký tự mạnh nhất mỗi khung hình (để nhanh), còn `pyctcdecode` cắt tỉa theo `beam_prune_logp` và xử lý hòa điểm khác.
- 5 hạt giống (42, 123, 456, 789, 2026), cấu hình giữ nguyên (AdamW lr 1e-3, weight_decay 1e-4, batch 32, ReduceLROnPlateau, clip_grad_norm 5,0, 25 epoch, chọn checkpoint bằng val, giải mã beam): val CER beam 12,61%, 12,70%, 12,72%, 12,45%, 13,68%, trung bình 12,83% ± 0,49% (độ lệch chuẩn mẫu, ddof=1); greedy 12,64%, 12,78%, 12,84%, 12,50%, 13,75%. Thời gian mỗi lần 25 epoch: 836,41, 832,47, 844,58, 839,80, 838,89 giây, trung bình 838,4 giây (khoảng 13,97 phút), nên 5 lần khoảng 70 phút trên Colab GPU.
- Hàm mất mát CTC tự cài: `src/ocr/ctc_loss.py`, truy hồi forward trong không gian log, vector hóa theo batch và trạng thái; unit test `tests/test_ctc_loss.py` khớp `torch.nn.CTCLoss` ở float64 (sai số dưới 1e-12), có ca ký tự lặp, `zero_infinity`, gradient autograd, và vét cạn toàn bộ 4^5 = 1.024 đường đi.
- Phần B và C cho CTC: chứng minh quy nạp công thức truy hồi α_t(s) và 2 bài tập tính tay có đáp án chi tiết ở `docs/ctc_theory_and_exercises.md`. Đã đối chiếu (22/11/2026): hai bài tập số học tính lại bằng đệ quy và vét cạn đều đúng (bài 1: p = 0,90, L ≈ 0,105361; bài 2: T = 2 cho p = 0, T = 3 cho p = 0,04, L ≈ 3,218876), và công thức truy hồi trong ký hiệu của báo cáo khớp vét cạn trên 200 ca ngẫu nhiên (sai khác tối đa 1,1 × 10⁻¹⁶). Chứng minh quy nạp đúng về ý nhưng cần sửa bốn điểm khi đưa vào báo cáo: thiếu một dòng định nghĩa trạng thái duy nhất của tiền tố đường đi (cơ sở của việc chia lớp theo trạng thái ở bước trước); câu "mở rộng tối đa 1 ký tự của l'" phải là "tối đa 1 ký tự nhãn (vị trí trên l' tiến 0, 1 hoặc 2)"; ký hiệu 0-based của Dương khác báo cáo (1-based); và "T ≥ L + 1" chỉ đúng khi nhãn có đúng một cặp ký tự lặp (tổng quát T ≥ L + r, r là số cặp liền nhau giống nhau). Đã đưa vào báo cáo dưới dạng một nhận xét về điều kiện nhảy hai bước và hai bài C11, C12.
- 45 dòng viết tay `real_001` đến `real_045` (đã ghi ở Giai đoạn 8).
- Toàn bộ số liệu nằm ở Drive `outputs/role_a_experiments_summary.md`; code và notebook đã đẩy GitHub, Dương báo pass 141/141 unit tests (báo cáo trước ghi 137 kiểm thử ngày 17/11/2026).

**Quyết định kỹ thuật:**

- **Vấn đề:** có huấn luyện lại OCR bằng hàm mất mát CTC tự cài không
- **Đã chọn (theo Dương):** không; giữ nguyên `best_crnn.pth` đã chốt, hàm tự cài dùng để kiểm chứng, đưa vào báo cáo và vấn đáp
- **Vì sao:** giữ kết quả đã chốt (12,27%) và tiết kiệm thời gian. Đánh đổi: mô hình chốt vẫn huấn luyện bằng `nn.CTCLoss`; khi vấn đáp phải nói rõ hàm tự cài đã được kiểm chứng nhưng chưa dùng để huấn luyện, và Bảng 5 phải ghi đúng như vậy

**Việc còn tồn đọng từ giai đoạn này:**

- Dương phải tự giải thích được chứng minh trong `docs/ctc_theory_and_exercises.md` và cả mệnh đề forward CTC do AI soạn trong báo cáo (hạn rà soát 23/11, đề xuất)
- Phụ lục A: khai báo công cụ AI Dương đã dùng (nếu có), phạm vi và gói, vì danh mục yêu cầu khai báo; hỏi Dương
- Hỏi Dương: `best_crnn.pth` (12,27%) thuộc hạt giống nào (không trùng 5 hạt giống: seed 42 cho 12,61%) và 5 hạt giống có chạy trên chia tập mới (5.369, 636, 201) không; 12,27% thấp hơn cả 5 số và thấp hơn trung bình khoảng 1,1 độ lệch chuẩn, và checkpoint chọn bằng val nên số val lạc quan, báo cả hai
- Hỏi Dương: bài kiểm tra so với `nn.CTCLoss` dùng bao nhiêu ca ngẫu nhiên (Bảng 5 yêu cầu 200 ca, sai số dưới 1e-5), và đã đối chiếu ở float32 chưa
- Chưa có: WER val của greedy (Bảng 6) và WER của 5 hạt giống
- Cập nhật báo cáo: Bảng 5 (hàng greedy, beam, CTC loss), mục 5.1, mục 6 ("hai trong tám thuật toán"), mục 6.2.1 (ô `\todo` sai số), Bảng 6, mục 7.4 (ngân sách GPU), mục 10.3, Bảng 10, Bảng 11 (rủi ro "CTC chưa tự cài"), mục 5.3 (137 thành 141 kiểm thử)
- Hiển tự chạy lại `tests/test_ctc_loss.py` để vấn đáp được

## Giai đoạn 10 — Đo bộ sửa dấu thật của Khánh trên tập kiểm định (ghi 22/11/2026, trước khi chấm test)
**Mục tiêu giai đoạn:** Chạy `eval_00_start.ipynb` bản mới (ô 6a nạp `src.corrector.api`, ô 6b phân bố độ tin cậy, ô 7c kiểm `ocr_conf.jsonl`, ô 7d phân rã cạnh sửa ký tự, ô 9b so sánh các mức) với `correct()` thật của Khánh để trả lời RQ2 và RQ3: lớp sửa dấu giảm được bao nhiêu CER, và ngưỡng τ đánh đổi giữa sửa đúng và sửa hỏng ra sao.

**Đã làm:**

- Nạp bộ sửa dấu thật (`src.corrector.api`): mọi mức (mặc định, `frequency`, `bigram`, `tagger`, `identity`) qua kiểm tra hợp đồng trên cả 636 dòng val, 0 vi phạm; mức mặc định cho văn bản và độ tin cậy giống hệt `tagger`. Thời gian chạy 636 dòng: mặc định 7,1 giây, `frequency` 0,2 giây, `bigram` 0,9 giây, `tagger` 3,8 giây.
- Phân bố độ tin cậy (ô 6b): bộ sửa đổi 1.432 trên 9.991 âm tiết (14,3%); 75,0% âm tiết có độ tin cậy từ 0,95; trung vị của các âm tiết bị đổi là 0,735.
- Mỗi mức khi sửa tất cả (τ = 0, cặp âm tiết kiểu 2): `frequency` đổi 3.552, sửa đúng 167, sửa hỏng 1.734 (chính xác 4,7%); `bigram` đổi 2.367, đúng 249, hỏng 636 (10,5%); mặc định (= `tagger`) đổi 1.404, đúng 149, hỏng 225 (10,6%). ECE (tất cả / âm tiết bị đổi / khả sửa, tức chữ nền đã đúng): `frequency` 0,232 / 0,559 / 0,040; `bigram` 0,241 / 0,623 / 0,017; mặc định 0,257 / 0,612 / 0,015.
- Quét τ cho mức mặc định (CER val không sửa 12,27%): τ = 0: CER 13,72%, WER 34,99%, sửa hỏng 3,35%, đúng trừ hỏng −76; τ = 0,50: 13,35%; τ = 0,60: 13,09%, đúng trừ hỏng +5; τ = 0,80: 12,74%, WER 33,84%, đúng trừ hỏng +42; τ = 0,95: 12,52%; τ = 1,00: 12,40%. Với cả hai trần sửa hỏng (1% và 2%) không τ nào đánh bại không sửa: quyết định theo quy tắc đã chốt trước là không sửa, ở mọi mức. Bootstrap theo đoạn (9 cụm): sửa tất cả làm CER tăng 1,45 điểm, khoảng tin cậy 95% của mức giảm từ −1,81 đến −1,21, không lần lấy mẫu nào cho CER giảm; theo trang (30 cụm) từ −1,71 đến −1,15.
- Phân rã cạnh sửa ký tự (ô 7d, xấp xỉ vì đếm theo cặp âm tiết): τ ≥ 0 tổng +604 cạnh sửa (+1,38 điểm, so với +1,45 điểm đo trực tiếp): sửa đúng 149 âm tiết (−152), sửa hỏng 225 (+227), dấu sai thành dấu sai khác 57 (+4), chữ nền sai vẫn sai 973 (+525, +1,20 điểm); τ ≥ 0,80 tổng +194 (+0,44 điểm): 89 (−92), 47 (+47), 22 (+3), 439 (+236). Tổng các nhóm khớp tổng ở cả hai ngưỡng. Kết luận: phần lớn âm tiết bị đổi (khoảng 69% ở τ = 0) có chữ nền sai do OCR; đổi dấu trên chúng vẫn cộng thêm cạnh sửa và lấn át lợi từ các âm tiết sửa đúng.
- Ví dụ sửa hỏng độ tin cậy rất cao (ô 7b): "nhậu" thành "nhau" (0,993), "cạch" thành "cách" (0,998), "nhá" thành "nhà" (0,997): bộ sửa thay một từ hợp lệ nhưng hiếm bằng từ phổ biến hơn. Ví dụ sửa đúng có độ tin cậy thấp (0,29 đến 0,48), nhưng đó là danh sách các độ tin cậy thấp nhất, không đại diện toàn bộ.
- Độ tin cậy của OCR (ô 7c, `ocr_conf.jsonl` qua kiểm tra: `ocr_text` trùng, độ dài và miền giá trị đúng trên 636 dòng): AUROC phân biệt âm tiết đúng với sai là 0,773 (`syllable_min`) và 0,794 (`syllable_mean`); tách đúng với "chỉ sai dấu" là 0,740 và 0,749. Tỉ lệ âm tiết đúng theo dải `syllable_min`: [0,95; 1,00] 90,3% đúng (4.382 âm tiết), nhưng dải thấp không đơn điệu (45,7% ở [0; 0,5) và 38,4% ở [0,5; 0,7)); `syllable_mean` tương tự (đơn điệu từ 0,7 trở lên).

**Quyết định kỹ thuật:**

- **Vấn đề:** chọn τ và mức khi mọi mức đều cho quyết định "không sửa"
- **Đã chọn:** giữ nguyên quy tắc đã chốt trước (CER val thấp nhất ở `FINAL_CAP`, hòa trong 0,05 điểm thì chọn mức đơn giản hơn); khi mọi mức hòa ở "không sửa" thì quy tắc trỏ đến `frequency`, và điều đó không ảnh hưởng kết quả test vì τ là không sửa nên CER test bằng CER OCR thô; báo cáo kết quả val của cả ba mức và nêu rõ quyết định chung là không sửa. `FINAL_CAP`: hai trần cho cùng quyết định, nên chọn 0,02 và ghi cả hai
- **Vì sao:** không đổi quy tắc sau khi thấy số; đánh đổi: nếu chạy test thì kết quả test chỉ lặp lại OCR thô, không thêm thông tin về lợi ích của bộ sửa dấu
- **Vấn đề:** dùng độ tin cậy của OCR làm cổng thứ hai (chỉ sửa khi OCR cũng tin cậy)
- **Đã chọn (đề xuất, chờ Hiển chốt):** không đưa vào kết quả chính vì quyết định chỉ quét τ đã chốt trước ngày 17/11 và thời gian còn lại ngắn; ghi như hướng mở rộng, kèm AUROC 0,77 đến 0,79
- **Vì sao:** thêm một cổng sau khi đã thấy kết quả val là chọn thêm một tham số trên chính val; đánh đổi: bỏ qua cách giảm trực tiếp nhóm "chữ nền sai vẫn sai" chiếm +1,20 điểm

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 20 — thông báo ở ô 8 ngược nghĩa:** ô 8 in "at this cap no correction beats correcting: the conclusion is not to correct" (không sửa thắng nhưng câu chữ nói ngược) → Nguyên nhân thật sự: chữ trong mã gốc của ô 8 (dòng `print` trong `choose_tau`), không ảnh hưởng số → Cách sửa: đổi thành "at this cap no correction beats not correcting: the conclusion is not to correct" (ô 9b đã có câu đúng)

**Việc còn tồn đọng từ giai đoạn này:**

- Đặt `FINAL_CAP = 0.02`, `RUN_TEST = True` và chạy test đúng một lần (201 dòng, 5 đoạn); khoảng tin cậy sẽ rất rộng và không chạy lại để thu hẹp; kết quả test sẽ trùng OCR thô vì quyết định là không sửa
- Ghi vào báo cáo: mục kết quả bộ sửa dấu trên val (E3, E4), Bảng 6, mục 10.3, Bảng 5 (hàng Khánh), Bảng 10, và viết rõ rằng kết quả áp dụng cho OCR hiện tại trên dữ liệu tổng hợp
- Hỏi Khánh những mục chưa thấy báo cáo: ECE trước hiệu chuẩn (chưa đo, cần bản chưa hiệu chuẩn), baseline scikit-learn, kết quả trung bình ± độ lệch chuẩn từ 5 lần chạy, kiểm thử Viterbi với vét cạn, loại 83 dòng trùng nguyên văn khỏi val/test, chọn kho chính (300.000 câu hay `train.txt`), và chạy `check_text_overlap.py --corpus` với `train.txt` trước khi tin các số val
- WER giảm nhẹ ở τ = 0,80 (34,30% xuống 33,84%) chỉ ghi như phân tích thăm dò (tiêu chí chính là CER, chốt trước)

## Giai đoạn 11 — Đưa code tái lập tập thật vào repo và danh sách thông tin còn thiếu của Dương, Khánh (ghi theo yêu cầu của Hiển)
**Mục tiêu giai đoạn:** Các số của tập viết tay thật và các phép đo trùng văn bản đến từ những ô Colab chưa nằm trong repo, nên người chấm không chạy lại được; đưa chúng thành script, và gom các thông tin còn thiếu từ Dương và Khánh (hai bạn đã gộp code vào `main`).

**Đã làm:**

- Năm script mới trong `scripts/` (code và chú thích chỉ bằng tiếng Anh): `run_real_ocr.py` (chạy OCR đã chốt một lần trên tập thật, có `--tag`, có cờ `--no-exif-fix` để tái tạo lần chạy 0, từ chối ghi đè file cũ nếu không có `--force`), `eval_real_clean.py` (chấm tập con sạch từ `label_review.csv` và `real_ocr_out_beam*.jsonl`, dừng nếu `editdistance` và `edit_distance` của `align.py` lệch ở bất kỳ dòng nào), `make_contact_sheets.py` (bảng ảnh đối chiếu theo đoạn), `check_real_text_overlap.py` (dựng chỉ mục tạm với mọi dòng UIT là `train` và 90 dòng thật là `test`, rồi gọi `check_text_overlap.main()`; không sửa file nào của dự án) và `crosscheck_metrics.py` (đối chiếu `edit_distance` với `editdistance` trên một split của `ocr_out_beam.jsonl`, in CER gộp và WER thư viện).
- Kiểm thử: `eval_real_clean.py` chạy trên `label_review.csv` và đầu ra lần 1 cho đúng các số đã báo cáo (CER 57,3%, WER 87,2%, khoảng tin cậy 42,9% đến 72,0%); `make_contact_sheets.py` tạo đúng ảnh đã xoay; `check_real_text_overlap.py` và `crosscheck_metrics.py` chạy trên dữ liệu giả và bắt đúng các dòng trùng, 0 dòng lệch. `run_real_ocr.py` mới biên dịch được, hai bộ giải mã trùng bản đã kiểm trước đó trên 60 chuỗi ngẫu nhiên và bước xoay ảnh cho đúng kích thước (ảnh `real_002`: cao 32, rộng 186 thay vì 16); chưa chạy được cả script với PyTorch và trọng số thật trong môi trường soạn thảo.

**Việc còn tồn đọng từ giai đoạn này:**

- Chạy năm script trong Colab với dữ liệu thật và đẩy lên `main` cùng `label_review.csv`; ghi lại kết quả `crosscheck_metrics.py` trên val (0 dòng lệch) vào báo cáo (Bảng 5, hàng Levenshtein) và kết quả `check_real_text_overlap.py` vào mục 4.3 (đã chạy năm script trong Colab ngày 24/11/2026, kết quả đúng như mong đợi, xem Giai đoạn 12; chưa xác nhận đã đẩy lên `main` và ghi vào báo cáo)
- Gửi hai tin nhắn: Dương (hạt giống của `best_crnn.pth` và 5 hạt giống trên chia tập mới, WER val của greedy và từng hạt giống, bài kiểm CTC ở float32 và với T thật, tái lập 5 hạt giống kèm phiên bản thư viện, khai báo công cụ AI, tự chứng minh lại hai mệnh đề) và Khánh (nhánh gộp, kiểm thử Viterbi với vét cạn, loại làm mịn, ECE trước và sau hiệu chuẩn, baseline scikit-learn, 5 lần chạy của bộ gán nhãn, kho văn bản và 83 dòng trùng, thông tin thẻ mô hình, khai báo công cụ AI, tự chứng minh lại hai mệnh đề)
- Cập nhật Phụ lục D của báo cáo cho khớp các tệp mới

## Giai đoạn 12 — Chạy năm script trên dữ liệu thật, phân tích lỗi tập viết tay thật và soạn tài liệu Phần D (ghi 25/11/2026)
**Mục tiêu giai đoạn:** Giai đoạn 11 đưa năm script vào repo nhưng chưa chạy với dữ liệu thật; giai đoạn này chạy chúng trong Colab và trả lời câu hỏi còn bỏ ngỏ của Phần D: vì sao CER trên tập viết tay thật (57,3%) cao hơn 4 lần val UIT (12,27%), lỗi nằm ở đâu, bộ sửa dấu thật có cứu được không. Đồng thời gom kết quả thành tài liệu nộp cho Phần D (thân bài LaTeX, datasheet, notebook tái lập). Hiển chốt 24/11/2026: chỉ đo lường trên tập 90 dòng sẵn có, không thu thêm dữ liệu hay người viết.

**Đã làm:**

- Chạy năm script trong Colab với dữ liệu thật, không có lỗi code. `run_real_ocr.py` từ chối ghi đè vì `real_ocr_out_greedy_run1.jsonl` đã tồn tại (đúng thiết kế, không xử lý lại ảnh nào); Hiển xác nhận file `run1` được tạo có sửa xoay EXIF. `eval_real_clean.py` trên `real_ocr_out_beam_run1.jsonl` cho đúng số của Giai đoạn 8 (57/90 dòng sạch, 2.310 ký tự; CER 57,3%, WER 87,2%; phân biệt hoa/thường 60,7% và 88,9%; khoảng tin cậy 95% theo đoạn 42,9% đến 72,0%; hai cách tính khoảng cách trùng ở mọi dòng). `make_contact_sheets.py` tạo 9 ảnh P01 đến P09, mỗi ảnh 10 dòng
- `check_real_text_overlap.py` (cửa sổ 5 từ, 90 dòng thật coi là `test`): so với `outputs/dataset_index.jsonl` (6.206 dòng UIT) trùng nguyên dòng, trùng ít nhất một cửa sổ và trùng phần lớn đều 0,0%; so với kho Wikipedia 300.000 câu là 0,0%, 1,1%, 0,0%; so với `train.txt` của Khánh (1.469.903 dòng) là 0,0%, 4,4%, 0,0%; không dòng nào xuất hiện nguyên văn trong hai kho. Kết luận KEEP, không rò rỉ đáng kể
- `crosscheck_metrics.py` trên val: 636 dòng, 0 dòng lệch giữa `edit_distance` và `editdistance`; CER gộp 12,27% (5.365 lỗi trên 43.727 ký tự), WER 34,30% (đúng số của Giai đoạn 6)
- Phân tích lỗi theo người chụp (ô Colab đọc đầu ra `run1`, tập sạch, cùng cách chuẩn hóa với số chính). Dương: 28 dòng, CER 34,0%, CER khi bỏ dấu cả hai bên 28,3%, độ dài đầu ra bằng 1,02 độ dài nhãn, ảnh cao trung vị 569 px, xám trung bình 189/255. Hiển: 29 dòng, CER 78,5%, bỏ dấu 75,6%, độ dài đầu ra 0,50, ảnh cao 1.220 px, xám 134/255. Tỉ lệ rộng/cao 6,7 và 6,8; không dòng nào rỗng. Mười dòng tệ nhất đều của Hiển (CER 88% đến 96%), đầu ra 3 đến 8 ký tự (ví dụ `real_068`: nhãn 49 ký tự, đầu ra "unnn", ảnh 10364×1516; `real_055`: nhãn 59 ký tự, đầu ra "ncn"). Cùng đoạn P05: Dương 28,7%, Hiển 75,8%, nhưng mỗi bên chỉ 3 dòng sạch. Trần chính xác (CER trừ CER bỏ dấu, mệnh đề B4): 5,7 điểm (Dương) và 2,9 điểm (Hiển). Khớp với Giai đoạn 8 (độ sáng 0,75 và 0,53; ảnh Hiển tối dần trong cùng đoạn)
- Thí nghiệm chẩn đoán tiền xử lý (khám phá, mô hình đóng băng, chạy từng ảnh một, tập sạch). Ba biến thể khai báo trước khi chạy, tham số cố định: V0 là tiền xử lý gốc; V1 co ảnh về cao 400 px (Lanczos), chia cho nền làm mờ Gaussian (sigma 40) rồi tự co giãn tương phản (cắt 1%); V2 là V1 cộng cắt sát vùng mực (điểm tối dưới 140, phân vị 1 đến 99 theo chiều dọc và 0,5 đến 99,5 theo chiều ngang, lề 15% chiều cao vùng mực). CER Dương/Hiển: V0 31,2%/78,2%, V1 36,7%/44,7%, V2 36,1%/42,6%; độ dài đầu ra của Hiển 0,50, 0,95, 0,96. Khoảng cách Hiển trừ Dương: 47,0, 8,0, 6,5 điểm (mất khoảng 85%), nhưng Dương tệ đi 4,9 đến 5,5 điểm
- Đo bộ sửa dấu thật (`api.correct`, mức mặc định) trên đầu ra beam `run1` của tập sạch, bốn τ cố định trước (0; 0,6; 0,8; 0,95), giữ cạnh sửa khi độ tin cậy từ τ trở lên (giả định); 0 dòng bị bỏ vì lệch số âm tiết. CER toàn tập sạch: thô 57,3%, τ = 0 là 58,6%, τ = 0,6 là 58,0%, τ = 0,8 là 57,4%, τ = 0,95 là 57,5%, bộ sửa dấu hoàn hảo 53,1%; Dương 34,0% lên 36,3% ở τ = 0, Hiển 78,5% lên 78,8%. Bootstrap theo đoạn (5.000 lần, hạt giống 42) của thay đổi CER: +1,26 điểm [+0,31; +2,17], +0,69 [0,00; +1,36], +0,13 [-0,34; +0,51], +0,17 [-0,05; +0,40]. Không τ nào làm CER giảm; cùng chiều với val (Giai đoạn 10: sửa tất cả tăng 1,45 điểm). Hệ thống cuối trên tập thật là OCR thô, 57,3%
- Tài liệu Phần D: `phan_D.tex` (thân bài để `\input`, 6 bảng, chỉ cần gói `booktabs`; biên dịch thử bằng xelatex, 4 trang, không lỗi), `Datasheet_Tap_chu_viet_tay_that.md` (cấu trúc Gebru và cộng sự, có mục ghi chú cho nhóm ở cuối) và `real_diagnostics.ipynb` (notebook Colab tái lập ba phép đo trên, ghi `real_diagnostics.json`; đã kiểm trên dữ liệu giả, bước nạp mô hình dùng đúng các lời gọi đã chạy trong Colab, chưa chạy toàn bộ trên dữ liệu thật)

**Quyết định kỹ thuật:**

- **Vấn đề:** có chạy chẩn đoán trên tập thật không, khi Giai đoạn 8 đã chốt "bỏ thử các bản làm sạch nền", "không chạy phân loại lỗi, p và trần chính xác trên tập thật" và "không chỉnh theo CER"
- **Đã chọn:** có, như thí nghiệm khám phá (Hiển chốt 24/11/2026: làm phân tích lỗi trước, chỉ đo lường). Quy tắc: mô hình đóng băng; ba biến thể và bốn τ chốt trước khi chạy và không tinh chỉnh; báo cáo tất cả kể cả kết quả xấu; số chính vẫn là 57,3% theo quy trình đóng băng; không chọn biến thể hay τ tốt nhất; tập thật không dùng để chọn τ (τ cuối lấy từ val)
- **Vì sao:** Phần D (30/100 điểm) chấm phân tích lỗi, và CER 57,3% không giải thích được sẽ bị hỏi khi vấn đáp; kết quả đã sửa một kết luận của Giai đoạn 8: khoảng cách giữa hai người chụp chủ yếu là điều kiện chụp, không phải mô hình không đọc nổi chữ của Hiển. Đánh đổi và thừa nhận lệch: V1, V2 được đề xuất sau khi đã thấy CER, độ sáng và đầu ra sụp đổ của Hiển, nên trái tinh thần "không chỉnh theo CER" của Giai đoạn 8 dù tham số chốt trước khi chạy; phải nói thẳng trong báo cáo, chỉ là chẩn đoán, không thay số chính. Trần chính xác cũng đã tính trên tập thật (trái dòng "không chạy trần trên tập thật" của Giai đoạn 8) vì chỉ là phép đo mô tả, không cần chạy lại mô hình

- **Vấn đề:** chuyển các ô Colab chẩn đoán thành script trong `scripts/` (như Giai đoạn 11) hay thành notebook
- **Đã chọn:** một notebook Colab `real_diagnostics.ipynb` (Hiển yêu cầu 25/11/2026), ghi kết quả vào `real_diagnostics.json` và không ghi đè nếu file đã có
- **Vì sao:** dữ liệu và checkpoint nằm trên Drive, chạy trong Colab, người chấm mở được ngay. Đánh đổi: khác cách của Giai đoạn 11 (script chạy từ dòng lệnh), cần giữ notebook trong repo

**Bug gặp phải — đánh số theo thứ tự toàn dự án, không reset theo giai đoạn:**

- **Bug 21 — OCR của Hiển sụp đổ độ dài dù đã sửa xoay:** dòng 30 đến 60 ký tự chỉ ra 3 đến 8 ký tự ("unnn", "ncn", "hhnn"), độ dài đầu ra bằng 0,50 độ dài nhãn, CER 78,5% so với 34,0% của Dương trên cùng mô hình → Nguyên nhân thật sự (một phần, từ thí nghiệm khám phá): điều kiện chụp ảnh của Hiển (tối hơn, xám 134 so với 189; tương phản thấp; độ phân giải cao gấp hơn hai lần); làm phẳng nền khôi phục độ dài đầu ra lên 0,95 và giảm khoảng cách giữa hai người từ 47,0 xuống 6,5 đến 8,0 điểm; chưa tách được thành phần nào có tác dụng, và phần còn lại chưa tách khỏi phong cách chữ → Cách sửa: không sửa mô hình hay tiền xử lý (đã chốt ở Giai đoạn 8); báo kết quả theo người chụp và nói rõ so sánh hai người lẫn điều kiện chụp; V1, V2 không phải cải tiến chung vì Dương tệ đi 4,9 đến 5,5 điểm
- **Bug 22 — CER của cùng một mô hình đổi khi chạy từng ảnh và khi chạy theo lô:** V0 chạy từng ảnh cho Dương 31,2% và Hiển 78,2%, chạy theo lô cho 34,0% và 78,5% → Nguyên nhân thật sự: chưa kiểm chứng; nghi do lô 32 ảnh đệm trắng đến độ rộng của ảnh dài nhất, BiLSTM hai chiều thấy cả phần đệm nên đầu ra phụ thuộc vào ảnh cùng lô → Cách sửa: chưa sửa; số chính vẫn là lần chạy theo lô (đóng băng), V0 chỉ dùng làm mốc trong cùng bảng chẩn đoán; ghi vào báo cáo

**Việc còn tồn đọng từ giai đoạn này:**

- Đẩy năm script và `label_review.csv` lên `main` nếu chưa; ghi vào báo cáo kết quả `crosscheck_metrics.py` trên val (0 dòng lệch, Bảng 5 hàng Levenshtein) và kết quả đo trùng văn bản (mục 4.3, thẻ dữ liệu). Chốt con số UIT: chỉ mục hiện dùng có 6.206 dòng, khác 7.229 trong ghi chú 21/11 (ghi chú đó tính cả 43 trang bị loại), cần ghi đúng một con số kèm lý do
- Ghép `phan_D.tex` vào báo cáo chính, đối chiếu với mục 4.3 và thẻ dữ liệu nguồn 3 để không trùng hoặc lệch số; nói thẳng trong báo cáo việc V1, V2 lệch quy tắc của Giai đoạn 8
- Chạy `real_diagnostics.ipynb` trên Colab thật, đối chiếu số với giai đoạn này, rồi đặt vào repo
- Datasheet: điền các chỗ `[cần xác nhận]` (mẫu điện thoại, cách cắt dòng, tài trợ, đạo đức, vị trí ảnh trong kho) rồi xóa mục "Ghi chú cho nhóm". Hai rủi ro so với danh mục môn học: `writer_id` đang là tên thật (`duong`, `hien`) trong khi danh mục yêu cầu ẩn danh; không có văn bản đồng ý (đã chốt không cần phiếu ở Giai đoạn 8; nếu muốn giảm rủi ro, một đoạn xác nhận ngắn có chữ ký của hai người). Nếu xóa EXIF khi phát hành ảnh, phải xoay ảnh về đúng hướng trước vì hai script đọc dựa vào thẻ này
- Hỏi Dương còn giữ checkpoint của 5 hạt giống không; nếu còn, chạy lên tập thật để có trung bình ± độ lệch chuẩn theo danh mục (hiện tập thật chỉ một checkpoint)
- Chưa đối chiếu giả định về τ (giữ cạnh sửa khi độ tin cậy từ τ trở lên) với ô quét τ trong `eval_00_start.ipynb`; chưa phân tích cạnh sửa đúng/hỏng trên tập thật nên chưa khẳng định cơ chế giống val
- Chưa tách thành phần nào của V1 và V2 (làm phẳng nền, co giãn tương phản, thu nhỏ trung gian, cắt sát) có tác dụng; 6,5 đến 8 điểm còn lại chưa tách khỏi phong cách chữ

# Kiến thức đã áp dụng
- [[Unicode Normalization]] (chuẩn hóa NFC trước khi so sánh chuỗi)
- [[Levenshtein Distance]] (đã cài ở `src/eval/align.py`: căn chỉnh OCR với nhãn đúng, truy vết, CER/WER)
- [[Data Leakage]] (rò rỉ dữ liệu: chia theo trang vẫn để lọt văn bản, chia theo đoạn văn mới sạch)
- [[Union-Find]] (gom trang cùng đoạn văn bằng liên kết bắc cầu ở `scripts/make_passage_splits.py`)
- [[CTC (Connectionist Temporal Classification)]] (hàm mất mát và giải mã greedy, prefix beam search của vai A)
- [[Bootstrap Resampling]] (lấy mẫu lại theo đoạn văn để có khoảng tin cậy ở `eval_00_start.ipynb`)
- [[Domain Shift]] (dịch chuyển miền giữa dữ liệu tổng hợp UIT-HWDB và chữ viết tay thật chụp bằng điện thoại, tập test thật)
- [[Ablation Study]] (ba biến thể tiền xử lý khai báo trước trên mô hình đóng băng để tách ảnh hưởng của điều kiện chụp, tập thật)
- [[Background Normalization]] (làm phẳng nền bằng cách chia cho nền làm mờ, rồi tự co giãn tương phản; V1 và V2 ở Giai đoạn 12)
